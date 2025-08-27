# Mail Services

Mail services provide essential communication infrastructure and often reveal valuable information about users and system configurations.

## Overview

Mail service enumeration focuses on:

- User enumeration and validation
- Authentication mechanism testing
- Configuration disclosure
- Relay testing and abuse
- SSL/TLS security assessment

## Common Mail Services

### SMTP (Ports 25, 465, 587)
Simple Mail Transfer Protocol handles email transmission between servers.

**Key Attack Vectors:**
- User enumeration via VRFY/EXPN commands
- Open relay testing
- Authentication bypass techniques
- SSL/TLS configuration weaknesses

[View SMTP Cheat Sheet](smtp.md)

### IMAP/POP3 (Ports 143, 993, 110, 995)
Email retrieval protocols for client access to mailboxes.

**Key Attack Vectors:**
- Credential brute forcing
- Plaintext authentication interception
- SSL certificate validation bypass
- Mailbox enumeration and access

[View IMAP/POP3 Cheat Sheet](imap-pop3.md)

## General Methodology

### 1. Discovery
```bash
# Port scanning
nmap -p 25,110,143,465,587,993,995 target

# Service detection
nmap -sV -sC -p <ports> target
```

### 2. Enumeration
```bash
# SMTP user enumeration
smtp-user-enum -M VRFY -U users.txt -t target

# Banner grabbing
telnet target 25
nc target 110
```

### 3. Authentication Testing
```bash
# Credential attacks
hydra -L users.txt -P passwords.txt smtp://target
hydra -L users.txt -P passwords.txt imap://target

# Manual authentication
telnet target 110
```

### 4. Configuration Analysis
- SSL/TLS cipher strength
- Authentication methods supported
- Relay configuration
- Rate limiting implementation

## Security Considerations

When testing mail services:

- **User privacy** - Respect email privacy during testing
- **Service disruption** - Avoid overwhelming mail servers
- **Legal compliance** - Follow applicable privacy laws
- **Rate limiting** - Be aware of connection limits

## Common Vulnerabilities

### SMTP Issues
- Open mail relays
- User enumeration
- Weak authentication
- SSL/TLS misconfigurations

### IMAP/POP3 Issues
- Plaintext authentication
- Weak credentials
- SSL certificate problems
- Unauthorized mailbox access

## Tools and Resources

### Essential Tools
- **nmap** - Service discovery and scripting
- **telnet/nc** - Manual protocol interaction
- **smtp-user-enum** - SMTP user enumeration
- **hydra** - Authentication attacks

### Specialized Tools
- **swaks** - SMTP testing toolkit
- **sendemail** - Command-line email client
- **openssl** - SSL/TLS testing
- **curl** - IMAP/POP3 client functionality

Select a specific mail service above to access detailed enumeration and exploitation techniques.
