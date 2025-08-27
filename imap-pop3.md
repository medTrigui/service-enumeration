# IMAP & POP3 Enumeration Cheat Sheet

## Table of Contents
1. [Overview](#overview)
2. [Ports & Protocols](#ports--protocols)
3. [Quick Discovery](#quick-discovery)
4. [Essential Enumeration Workflow](#essential-enumeration-workflow)
   - [Service Detection](#1-service-detection)
   - [Banner Analysis](#2-banner-analysis)
   - [Authentication Testing](#3-authentication-testing)
   - [Email Access](#4-email-access)
5. [Manual Interaction Techniques](#manual-interaction-techniques)
   - [IMAP Commands](#imap-commands)
   - [POP3 Commands](#pop3-commands)
   - [SSL/TLS Connections](#ssltls-connections)
6. [Credential Attacks](#credential-attacks)
   - [Brute Force Attacks](#brute-force-attacks)
   - [Password Spraying](#password-spraying)
7. [Email Enumeration & Extraction](#email-enumeration--extraction)
8. [Scripted Enumeration](#scripted-enumeration)
9. [Common Vulnerabilities](#common-vulnerabilities)
10. [Quick Reference Commands](#quick-reference-commands)
11. [Testing Checklist](#testing-checklist)
12. [Critical Security Issues](#critical-security-issues)

---

## Overview

**IMAP (Internet Message Access Protocol)** and **POP3 (Post Office Protocol v3)** are email retrieval protocols that allow clients to access emails stored on mail servers. Both protocols are commonly found in corporate environments and can expose sensitive information.

### Key Functions
- **Email Retrieval**: Download emails from mail servers
- **Authentication**: User credential validation
- **Mailbox Management**: IMAP supports server-side folder operations
- **Multi-Device Access**: IMAP allows synchronized access across devices

### Why IMAP/POP3 Matter in Pentesting
- **Credential Exposure**: Often use weak or default passwords
- **Information Disclosure**: Email content reveals sensitive data
- **Lateral Movement**: Valid credentials may work on other services
- **Business Intelligence**: Email metadata exposes organizational structure
- **Plaintext Protocols**: Unencrypted traffic exposes credentials

### Common Attack Vectors
- **Brute Force Attacks**: Weak password policies
- **Credential Stuffing**: Reused credentials across services
- **Plaintext Sniffing**: Unencrypted authentication
- **Information Harvesting**: Email content and metadata extraction
- **Configuration Exploitation**: Debug logging, anonymous access

---

## Ports & Protocols

```
110/tcp  - POP3 (Post Office Protocol v3)
143/tcp  - IMAP (Internet Message Access Protocol)
993/tcp  - IMAPS (IMAP over SSL/TLS)
995/tcp  - POP3S (POP3 over SSL/TLS)
```

### Protocol Comparison
| Feature | IMAP | POP3 |
|---------|------|------|
| **Server Storage** | Yes (persistent) | No (download & delete) |
| **Multiple Clients** | Yes (synchronized) | No (single client) |
| **Folder Support** | Yes | No |
| **Partial Download** | Yes | No |
| **Default Port** | 143 (993 SSL) | 110 (995 SSL) |
| **Complexity** | High | Low |

---

## Quick Discovery

```bash
# Port scan for email services
nmap -p 110,143,993,995 --open -sV target

# IMAP capabilities detection
nmap -p 143,993 --script=imap-capabilities target

# Comprehensive email service scan
nmap -p 110,143,993,995 -sC -sV target
```

---

## Essential Enumeration Workflow

### 1. Service Detection

#### Banner Grabbing
```bash
# IMAP banner
nc target 143

# POP3 banner
nc target 110

# Check for common responses:
# * OK IMAP4rev1 service ready
# +OK POP3 server ready
```

#### Capability Enumeration
```bash
# IMAP capabilities
echo "A1 CAPABILITY" | nc target 143

# POP3 capabilities  
echo "CAPA" | nc target 110

# Look for authentication methods:
# AUTH=PLAIN, AUTH=LOGIN, AUTH=CRAM-MD5
```

### 2. Banner Analysis

#### Service Identification
```bash
# Detailed service fingerprinting
nmap -p 143,993 --script=imap-ntlm-info target
nmap -p 110,995 --script=pop3-ntlm-info target

# Version detection
nmap -sV -p 110,143,993,995 target
```

### 3. Authentication Testing

#### Default Credentials
```bash
# Common default accounts to test:
# admin:admin
# admin:password
# test:test
# guest:guest
# user:user
```

#### Manual Authentication
```bash
# IMAP login test
echo "A1 LOGIN admin admin" | nc target 143

# POP3 login test
(echo "USER admin"; echo "PASS admin") | nc target 110
```

### 4. Email Access

#### IMAP Email Enumeration
```bash
# Connect and authenticate
telnet target 143
A1 LOGIN username password
A2 LIST "" "*"              # List all mailboxes
A3 SELECT INBOX             # Select inbox
A4 SEARCH ALL               # List all email IDs
A5 FETCH 1 BODY[TEXT]       # Get email content
A6 LOGOUT
```

#### POP3 Email Enumeration
```bash
# Connect and enumerate
telnet target 110
USER username
PASS password
STAT                        # Get message count
LIST                        # List all messages
RETR 1                      # Retrieve first message
QUIT
```

---

## Manual Interaction Techniques

### IMAP Commands
```bash
# Essential IMAP commands for enumeration
A1 CAPABILITY               # Show server capabilities
A2 LOGIN user pass          # Authenticate
A3 LIST "" "*"              # List all folders
A4 SELECT INBOX             # Select mailbox
A5 SEARCH ALL               # List message IDs
A6 FETCH 1 BODY[HEADER]     # Get headers only
A7 FETCH 1 BODY[TEXT]       # Get message body
A8 FETCH 1 BODY[]           # Get full message
A9 CREATE "Testing"         # Create folder (if writable)
A10 LOGOUT                  # Disconnect
```

### POP3 Commands
```bash
# Essential POP3 commands
CAPA                        # Server capabilities
USER username               # Set username
PASS password               # Set password
STAT                        # Message statistics
LIST                        # List all messages
LIST 1                      # Info for message 1
RETR 1                      # Retrieve message 1
TOP 1 10                    # Get first 10 lines of message 1
DELE 1                      # Mark message 1 for deletion
RSET                        # Reset session
QUIT                        # Close connection
```

### SSL/TLS Connections
```bash
# IMAPS connection
openssl s_client -connect target:993

# POP3S connection
openssl s_client -connect target:995

# Ignore certificate errors
openssl s_client -connect target:993 -verify_return_error

# Test specific SSL/TLS versions
openssl s_client -connect target:993 -tls1_2
```

---

## Credential Attacks

### Brute Force Attacks

#### Hydra
```bash
# IMAP brute force
hydra -L users.txt -P passwords.txt imap://target

# POP3 brute force  
hydra -L users.txt -P passwords.txt pop3://target

# IMAPS/POP3S brute force
hydra -L users.txt -P passwords.txt imaps://target
hydra -L users.txt -P passwords.txt pop3s://target

# Single user, multiple passwords
hydra -l admin -P passwords.txt imap://target

# Throttled attack (slower)
hydra -L users.txt -P passwords.txt -t 1 -W 3 imap://target
```

#### Medusa
```bash
# IMAP brute force
medusa -h target -U users.txt -P passwords.txt -M imap

# POP3 brute force
medusa -h target -U users.txt -P passwords.txt -M pop3

# Single user attack
medusa -h target -u admin -P passwords.txt -M imap
```

### Password Spraying
```bash
# Test common passwords against user list
hydra -L users.txt -p 'Password123' imap://target
hydra -L users.txt -p 'password' imap://target
hydra -L users.txt -p 'admin' imap://target

# Seasonal passwords
hydra -L users.txt -p 'Summer2024!' imap://target
```

---

## Email Enumeration & Extraction

### Using cURL
```bash
# IMAP with cURL
curl -k 'imaps://target' --user username:password

# POP3 with cURL
curl -k 'pop3s://target' --user username:password

# List specific folder
curl -k 'imaps://target/INBOX' --user username:password

# Download specific message
curl -k 'imaps://target/INBOX;UID=1' --user username:password
```

### Email Content Analysis
```bash
# Search for specific content in IMAP
A1 LOGIN user pass
A2 SELECT INBOX
A3 SEARCH TEXT "password"           # Search for "password"
A4 SEARCH FROM "admin"              # Search by sender
A5 SEARCH SUBJECT "confidential"    # Search by subject
A6 SEARCH SINCE "01-Jan-2024"       # Search by date
```

### Bulk Email Extraction
```bash
# Extract all emails from INBOX
for i in {1..10}; do
    echo "A$i FETCH $i BODY[]" | nc target 143
done

# Save specific email
echo "A1 LOGIN user pass
A2 SELECT INBOX  
A3 FETCH 1 BODY[]" | nc target 143 > email_1.txt
```

---

## Scripted Enumeration

### Python IMAP Script
```python
#!/usr/bin/env python3
import imaplib
import sys

def test_imap_login(host, user, password):
    try:
        server = imaplib.IMAP4(host, 143)
        server.login(user, password)
        server.select("INBOX")
        status, messages = server.search(None, "ALL")
        print(f"[+] Login successful: {user}:{password}")
        print(f"[+] Messages found: {len(messages[0].split())}")
        server.logout()
        return True
    except:
        print(f"[-] Login failed: {user}:{password}")
        return False

# Usage
test_imap_login("target", "admin", "password")
```

### Bash Email Extractor
```bash
#!/bin/bash
# Extract all emails from IMAP

HOST=$1
USER=$2
PASS=$3

echo "A1 LOGIN $USER $PASS
A2 SELECT INBOX
A3 SEARCH ALL" | nc $HOST 143 | grep "SEARCH" | cut -d' ' -f3- | tr ' ' '\n' | while read msgid; do
    echo "A4 FETCH $msgid BODY[]" | nc $HOST 143 > "email_${msgid}.txt"
done
```

---

## Common Vulnerabilities

### Plaintext Authentication
```bash
# VULNERABILITY: Credentials sent in cleartext
# Detection: Successful login over ports 110/143
echo "A1 LOGIN admin password" | nc target 143

# Mitigation: Use SSL/TLS (ports 993/995)
```

### Weak Authentication Mechanisms
```bash
# Check supported authentication methods
echo "A1 CAPABILITY" | nc target 143

# Look for insecure methods:
# AUTH=PLAIN (base64 encoded plaintext)
# No SSL/TLS requirement
```

### Information Disclosure
```bash
# Debug logging enabled
# Look for verbose server responses
# Banner information disclosure

# Example vulnerable banner:
# * OK [CAPABILITY IMAP4rev1 AUTH=PLAIN] Dovecot v2.3.16 ready
```

### Anonymous Access
```bash
# Test for anonymous/guest access
echo "A1 LOGIN anonymous ''" | nc target 143
echo "A1 LOGIN guest guest" | nc target 143

# VULNERABILITY: Unauthorized email access
```

---

## Quick Reference Commands

### Discovery
```bash
nmap -p 110,143,993,995 --script=imap-capabilities,pop3-capabilities target
nc target 143
nc target 110
```

### Authentication
```bash
hydra -L users.txt -P passwords.txt imap://target
medusa -h target -U users.txt -P passwords.txt -M imap
echo "A1 LOGIN user pass" | nc target 143
```

### Email Access
```bash
curl -k 'imaps://target' --user user:pass
telnet target 143
openssl s_client -connect target:993
```

### SSL/TLS Testing
```bash
openssl s_client -connect target:993
openssl s_client -connect target:995
nmap --script ssl-enum-ciphers -p 993,995 target
```

---

## Testing Checklist

#### Discovery & Detection
- [ ] Port scan for 110,143,993,995
- [ ] Banner grabbing and service identification
- [ ] Capability enumeration (AUTH methods)
- [ ] SSL/TLS configuration assessment

#### Authentication Testing
- [ ] Default credential testing
- [ ] Brute force attacks (rate-limited)
- [ ] Password spraying with common passwords
- [ ] Anonymous/guest access attempts

#### Protocol Analysis
- [ ] Plaintext vs encrypted connections
- [ ] Supported authentication mechanisms
- [ ] SSL/TLS certificate validation
- [ ] Protocol version identification

#### Email Enumeration
- [ ] Mailbox/folder listing
- [ ] Message count and metadata
- [ ] Email content extraction
- [ ] Sensitive information discovery

#### Security Assessment
- [ ] Debug logging configuration
- [ ] Information disclosure in banners
- [ ] SSL/TLS cipher strength
- [ ] Rate limiting effectiveness

---

## Critical Security Issues

### Plaintext Authentication
- **Impact**: Credential exposure, session hijacking
- **Detection**: Successful authentication over ports 110/143
- **Exploitation**: Network sniffing, credential harvesting

### Weak SSL/TLS Configuration
- **Impact**: Man-in-the-middle attacks, credential theft
- **Detection**: Weak ciphers, outdated TLS versions
- **Exploitation**: SSL strip attacks, certificate bypass

### Information Disclosure
- **Impact**: Service fingerprinting, reconnaissance data
- **Detection**: Verbose banners, debug information
- **Exploitation**: Version-specific exploit identification

### Weak Authentication
- **Impact**: Unauthorized email access, data theft
- **Detection**: Successful brute force attacks
- **Exploitation**: Email content extraction, lateral movement

### Anonymous Access
- **Impact**: Unauthorized email access without credentials
- **Detection**: Successful anonymous login attempts
- **Exploitation**: Complete mailbox access, data exfiltration

---

## Advanced Techniques

### Protocol Downgrade Attacks
```bash
# Force plaintext over encrypted ports
openssl s_client -connect target:993 -cipher NULL

# Test for STARTTLS downgrade
echo "A1 CAPABILITY" | nc target 143
```

### Email Forensics
```bash
# Extract email headers for analysis
A1 FETCH 1 BODY[HEADER]

# Look for internal IP addresses, domains
# Analyze email routing information
# Extract attachment metadata
```

### Persistence Techniques
```bash
# Email forwarding rules (if IMAP supports)
# Search for sensitive keywords across all emails
# Monitor for new emails with specific content

# Continuous monitoring script
while true; do
    echo "A1 LOGIN user pass; A2 SELECT INBOX; A3 SEARCH ALL" | nc target 143
    sleep 300
done
```