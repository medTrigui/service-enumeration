# SMTP Enumeration Cheat Sheet

## Table of Contents
1. [Overview](#overview)
2. [Ports & Protocols](#ports--protocols)
3. [Quick Discovery](#quick-discovery)
4. [Essential Enumeration Workflow](#essential-enumeration-workflow)
   - [Banner Grabbing](#1-banner-grabbing)
   - [Command Discovery](#2-command-discovery)
   - [User Enumeration](#3-user-enumeration)
   - [Open Relay Testing](#4-open-relay-testing)
5. [Manual Interaction Techniques](#manual-interaction-techniques)
   - [SMTP Commands](#smtp-commands)
   - [User Verification Methods](#user-verification-methods)
6. [Automated Enumeration](#automated-enumeration)
7. [Authentication Testing](#authentication-testing)
8. [Advanced Techniques](#advanced-techniques)
9. [Common Vulnerabilities](#common-vulnerabilities)
10. [Quick Reference Commands](#quick-reference-commands)
11. [Testing Checklist](#testing-checklist)
12. [Critical Security Issues](#critical-security-issues)

---

## Overview

**SMTP (Simple Mail Transfer Protocol)** is the standard protocol for sending emails between mail servers and from clients to servers. SMTP services often reveal extensive information about users, system configuration, and mail infrastructure, making them valuable targets for reconnaissance.

### Key Functions
- **Mail Transfer**: Send emails between mail servers
- **Mail Submission**: Accept emails from clients for delivery
- **User Verification**: Validate email addresses and users
- **Mail Routing**: Forward emails to appropriate destinations

### Why SMTP Matters in Pentesting
- **User Enumeration**: VRFY and EXPN commands reveal valid users
- **Information Disclosure**: Banners expose software versions and configuration
- **Open Relay Detection**: Misconfigured servers allow mail abuse
- **Authentication Testing**: Weak credentials for mail submission
- **Network Mapping**: Mail server topology and routing information

### Common Attack Vectors
- **User Enumeration**: VRFY, EXPN, RCPT TO commands
- **Banner Grabbing**: Version and software disclosure
- **Open Relay Abuse**: Unauthorized mail sending
- **Authentication Attacks**: Brute force mail submission
- **Information Gathering**: Mail server configuration exposure

---

## Ports & Protocols

```
25/tcp   - SMTP (Simple Mail Transfer Protocol)
465/tcp  - SMTPS (SMTP over SSL/TLS - deprecated)
587/tcp  - SMTP Submission (STARTTLS preferred)
2525/tcp - Alternative SMTP (non-standard)
```

### Protocol Variations
| Port | Protocol | Purpose | Security |
|------|----------|---------|----------|
| **25** | SMTP | Mail transfer between servers | Usually plaintext |
| **465** | SMTPS | SSL/TLS encrypted (deprecated) | Encrypted |
| **587** | SMTP Submission | Client mail submission | STARTTLS |

---

## Quick Discovery

```bash
# Port scan for SMTP services
nmap -p 25,465,587,2525 --open -sV target

# SMTP-specific scripts
nmap -p 25,465,587 --script=smtp-enum-users,smtp-commands,smtp-open-relay target

# Banner grabbing
nc target 25
telnet target 25
```

---

## Essential Enumeration Workflow

### 1. Banner Grabbing

#### Manual Banner Collection
```bash
# Netcat connection
nc -nv target 25

# Telnet connection
telnet target 25

# SSL/TLS connections
openssl s_client -connect target:465
openssl s_client -connect target:587 -starttls smtp

# Look for information disclosure in banners:
# 220 mail.example.com ESMTP Postfix (Ubuntu)
# 220 mail.company.com Microsoft ESMTP MAIL Service ready
```

#### Automated Banner Analysis
```bash
# Nmap banner grabbing
nmap -p 25,465,587 -sV --script=banner target

# Service version detection
nmap -p 25 -sV target
```

### 2. Command Discovery

#### Supported Command Enumeration
```bash
# Manual command discovery
echo "EHLO test.com" | nc target 25
echo "HELP" | nc target 25

# Look for available commands:
# 250-VRFY
# 250-EXPN
# 250-AUTH LOGIN PLAIN
# 250-STARTTLS
```

#### Nmap Command Discovery
```bash
# SMTP commands script
nmap -p 25 --script=smtp-commands target

# Authentication methods
nmap -p 25,587 --script=smtp-auth-methods target
```

### 3. User Enumeration

#### VRFY Command (Verify)
```bash
# Manual VRFY testing
echo "VRFY root" | nc target 25
echo "VRFY admin" | nc target 25
echo "VRFY user" | nc target 25

# Automated VRFY enumeration
for user in root admin user guest; do
    echo "VRFY $user" | nc target 25
done

# Response analysis:
# 250 2.1.5 root <root@target.com>     # User exists
# 550 5.1.1 No such user               # User doesn't exist
# 252 2.1.5 Cannot verify user         # Inconclusive
```

#### EXPN Command (Expand)
```bash
# Manual EXPN testing
echo "EXPN root" | nc target 25
echo "EXPN admin" | nc target 25

# Test common mailing lists
echo "EXPN all" | nc target 25
echo "EXPN everyone" | nc target 25
```

#### RCPT TO Method
```bash
# User enumeration via RCPT TO
echo -e "EHLO test.com\nMAIL FROM:<test@test.com>\nRCPT TO:<root@target.com>" | nc target 25

# Response analysis:
# 250 2.1.5 root@target.com... Recipient ok    # User exists
# 550 5.1.1 root@target.com... User unknown    # User doesn't exist
```

### 4. Open Relay Testing

#### Basic Open Relay Test
```bash
# Test external to external relay
echo -e "EHLO test.com\nMAIL FROM:<attacker@external.com>\nRCPT TO:<victim@external.com>\nDATA\nSubject: Relay Test\n\nThis is a relay test.\n.\nQUIT" | nc target 25

# Response analysis:
# 250 Message accepted for delivery     # VULNERABLE: Open relay
# 554 Relay access denied              # Protected against relay
```

#### Advanced Relay Testing
```bash
# Multiple relay test scenarios
# Local to external
echo -e "MAIL FROM:<local@target.com>\nRCPT TO:<external@test.com>" | nc target 25

# External to local
echo -e "MAIL FROM:<external@test.com>\nRCPT TO:<local@target.com>" | nc target 25

# Localhost variations
echo -e "MAIL FROM:<test@localhost>\nRCPT TO:<external@test.com>" | nc target 25
```

---

## Manual Interaction Techniques

### SMTP Commands
```bash
# Complete SMTP session example
telnet target 25

220 mail.target.com ESMTP ready
EHLO attacker.com
250-mail.target.com Hello attacker.com
250-SIZE 10240000
250-VRFY
250-ETRN
250-ENHANCEDSTATUSCODES
250 8BITMIME

VRFY root
250 2.1.5 root <root@target.com>

VRFY admin
550 5.1.1 admin... User unknown

QUIT
221 2.0.0 Bye
```

### User Verification Methods

#### VRFY Method
```bash
# Systematic user enumeration
users=("root" "admin" "administrator" "user" "guest" "mail" "postmaster" "www-data" "apache" "nginx")

for user in "${users[@]}"; do
    echo "VRFY $user" | nc target 25 | grep -E "250|550"
done
```

#### EXPN Method
```bash
# Mailing list expansion
lists=("all" "everyone" "staff" "users" "admin" "root")

for list in "${lists[@]}"; do
    echo "EXPN $list" | nc target 25
done
```

#### RCPT TO Method
```bash
# Email address validation
domains=("target.com" "localhost" "127.0.0.1")

for domain in "${domains[@]}"; do
    echo -e "EHLO test.com\nMAIL FROM:<test@test.com>\nRCPT TO:<root@$domain>" | nc target 25
done
```

---

## Automated Enumeration

### smtp-user-enum
```bash
# Install smtp-user-enum
# apt-get install smtp-user-enum

# VRFY method enumeration
smtp-user-enum -M VRFY -U users.txt -t target

# EXPN method enumeration
smtp-user-enum -M EXPN -U users.txt -t target

# RCPT method enumeration
smtp-user-enum -M RCPT -U users.txt -t target

# Specify custom port
smtp-user-enum -M VRFY -U users.txt -t target -p 587
```

### Nmap NSE Scripts
```bash
# User enumeration script
nmap -p 25 --script=smtp-enum-users --script-args smtp-enum-users.methods={VRFY,EXPN,RCPT} target

# Open relay detection
nmap -p 25 --script=smtp-open-relay target

# SMTP commands discovery
nmap -p 25 --script=smtp-commands target

# NTLM information disclosure
nmap -p 25,587 --script=smtp-ntlm-info target
```

### Python Automation Script
```python
#!/usr/bin/env python3
import socket
import sys

def smtp_vrfy(target, port, users):
    for user in users:
        try:
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            s.settimeout(5)
            s.connect((target, port))
            
            banner = s.recv(1024).decode()
            print(f"Banner: {banner.strip()}")
            
            s.send(f"VRFY {user}\r\n".encode())
            result = s.recv(1024).decode()
            
            if "250" in result:
                print(f"[+] User exists: {user}")
            elif "550" in result:
                print(f"[-] User not found: {user}")
            else:
                print(f"[?] Uncertain: {user} - {result.strip()}")
                
            s.close()
        except:
            print(f"[!] Error connecting to {target}:{port}")

# Usage
users = ["root", "admin", "user", "guest", "postmaster"]
smtp_vrfy("target", 25, users)
```

---

## Authentication Testing

### SMTP AUTH Testing
```bash
# Check for authentication methods
echo "EHLO test.com" | nc target 587

# Look for AUTH methods:
# 250-AUTH LOGIN PLAIN CRAM-MD5

# Manual authentication testing
echo -e "EHLO test.com\nAUTH LOGIN\n$(echo -n 'username' | base64)\n$(echo -n 'password' | base64)" | nc target 587
```

### Brute Force Authentication
```bash
# Hydra SMTP brute force
hydra -L users.txt -P passwords.txt smtp://target:587

# Medusa SMTP brute force
medusa -h target -u admin -P passwords.txt -M smtp -n 587

# Custom authentication testing
for user in admin user root; do
    for pass in password admin 123456; do
        echo "Testing $user:$pass"
        echo -e "EHLO test.com\nAUTH LOGIN\n$(echo -n $user | base64)\n$(echo -n $pass | base64)" | nc target 587
    done
done
```

---

## Advanced Techniques

### SSL/TLS Testing
```bash
# SMTPS (port 465) connection
openssl s_client -connect target:465

# STARTTLS (port 587) connection
openssl s_client -connect target:587 -starttls smtp

# Cipher enumeration
nmap --script ssl-enum-ciphers -p 465,587 target

# Certificate analysis
echo | openssl s_client -connect target:465 2>/dev/null | openssl x509 -noout -text
```

### Information Disclosure
```bash
# Extract detailed banner information
echo "EHLO test.com" | nc target 25 | grep -E "220|250"

# Check for information leakage
echo "HELP" | nc target 25

# Test for verbose error messages
echo "INVALID_COMMAND" | nc target 25
```

### Mail Header Injection Testing
```bash
# Test for header injection vulnerabilities
echo -e "EHLO test.com\nMAIL FROM:<test@test.com>\nRCPT TO:<user@target.com>\nDATA\nSubject: Test\nCc: admin@target.com\n\nTest message\n.\nQUIT" | nc target 25
```

---

## Common Vulnerabilities

### User Enumeration via VRFY/EXPN
```bash
# VULNERABILITY: VRFY command enabled
echo "VRFY root" | nc target 25
# Response: 250 2.1.5 root <root@target.com>

# VULNERABILITY: EXPN command enabled
echo "EXPN admin" | nc target 25
# Response: 250 2.1.5 admin <admin@target.com>
```

### Open Mail Relay
```bash
# VULNERABILITY: Open relay configuration
echo -e "EHLO test.com\nMAIL FROM:<attacker@external.com>\nRCPT TO:<victim@external.com>\nDATA\nRelay test\n.\nQUIT" | nc target 25
# Response: 250 Message accepted for delivery
```

### Information Disclosure
```bash
# VULNERABILITY: Verbose banner information
nc target 25
# Response: 220 mail.company.com ESMTP Postfix (Ubuntu) ready

# VULNERABILITY: Software version exposure
echo "EHLO test.com" | nc target 25
# Response: 250-mail.company.com Hello, I am glad to meet you
```

### Weak Authentication
```bash
# VULNERABILITY: Weak or default credentials
echo -e "EHLO test.com\nAUTH LOGIN\n$(echo -n 'admin' | base64)\n$(echo -n 'admin' | base64)" | nc target 587
# Response: 235 2.7.0 Authentication successful
```

---

## Quick Reference Commands

### Discovery
```bash
nmap -p 25,465,587 --script=smtp-enum-users,smtp-commands target
nc target 25
smtp-user-enum -M VRFY -U users.txt -t target
```

### User Enumeration
```bash
echo "VRFY root" | nc target 25
echo "EXPN admin" | nc target 25
echo -e "EHLO test\nMAIL FROM:<test@test.com>\nRCPT TO:<user@target.com>" | nc target 25
```

### Open Relay Testing
```bash
echo -e "EHLO test.com\nMAIL FROM:<ext@test.com>\nRCPT TO:<ext2@test.com>\nDATA\nRelay test\n.\nQUIT" | nc target 25
```

### SSL/TLS Testing
```bash
openssl s_client -connect target:465
openssl s_client -connect target:587 -starttls smtp
nmap --script ssl-enum-ciphers -p 465,587 target
```

---

## Testing Checklist

#### Discovery & Detection
- [ ] Port 25/465/587 open and responding
- [ ] SMTP service banner analysis
- [ ] Supported command enumeration
- [ ] SSL/TLS configuration assessment

#### User Enumeration
- [ ] VRFY command availability and testing
- [ ] EXPN command functionality
- [ ] RCPT TO method user validation
- [ ] Automated user enumeration

#### Configuration Testing
- [ ] Open relay detection
- [ ] Authentication method identification
- [ ] STARTTLS support verification
- [ ] Mail routing analysis

#### Security Assessment
- [ ] Banner information disclosure
- [ ] User enumeration vulnerabilities
- [ ] Authentication mechanism strength
- [ ] SSL/TLS cipher analysis

#### Authentication Testing
- [ ] Default credential testing
- [ ] SMTP AUTH brute force
- [ ] Weak password identification
- [ ] Account lockout testing

---

## Critical Security Issues

### User Enumeration Vulnerability
- **Impact**: User account discovery, reconnaissance data
- **Detection**: Successful VRFY/EXPN commands
- **Exploitation**: `echo "VRFY root" | nc target 25`

### Open Mail Relay
- **Impact**: Mail abuse, reputation damage, blacklisting
- **Detection**: Successful external-to-external mail relay
- **Exploitation**: Spam distribution, phishing campaigns

### Information Disclosure
- **Impact**: Software fingerprinting, version identification
- **Detection**: Verbose banners, detailed error messages
- **Exploitation**: Version-specific attack identification

### Weak Authentication
- **Impact**: Unauthorized mail access, account compromise
- **Detection**: Successful authentication with weak credentials
- **Exploitation**: Mail access, credential harvesting

### SSL/TLS Weaknesses
- **Impact**: Man-in-the-middle attacks, credential interception
- **Detection**: Weak ciphers, outdated TLS versions
- **Exploitation**: Traffic interception, credential theft

---

## Common SMTP Response Codes

### Success Codes
```
220 - Service ready
250 - Requested action completed
251 - User not local; will forward
252 - Cannot verify user; will attempt delivery
354 - Start mail input
```

### Error Codes
```
421 - Service not available
450 - Mailbox unavailable (temporary)
550 - Mailbox unavailable (permanent)
551 - User not local
552 - Exceeded storage allocation
553 - Mailbox name not allowed
554 - Transaction failed
```

---

## User Enumeration Wordlists

### Common Usernames
```
root
admin
administrator
user
guest
postmaster
webmaster
mail
nobody
www-data
apache
nginx
ftp
ssh
test
demo
service
```

### Email Address Formats
```
username@domain.com
username@localhost
username@target
admin@target.com
root@target.com
postmaster@target.com
webmaster@target.com
info@target.com
support@target.com
```

---

## Advanced Mail Server Testing

### Mail Server Fingerprinting
```bash
# Identify mail server software
echo "EHLO test.com" | nc target 25 | grep -i "postfix\|sendmail\|exchange\|exim"

# Version detection
nmap -p 25 -sV --script=smtp-vuln-* target
```

### Mail Routing Analysis
```bash
# Trace mail routing
echo -e "EHLO test.com\nMAIL FROM:<test@test.com>\nRCPT TO:<user@target.com>\nDATA\nReceived: trace\n.\nQUIT" | nc target 25

# Multiple domain testing
for domain in target.com localhost 127.0.0.1; do
    echo "RCPT TO:<test@$domain>" | nc target 25
done
```