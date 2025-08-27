# Service Enumeration Cheat Sheets

Professional penetration testing reference guides for systematic service discovery and exploitation.

## Overview

This documentation provides comprehensive cheat sheets for enumerating and exploiting various network services commonly encountered during penetration testing engagements. Each guide follows a consistent methodology covering discovery, enumeration, authentication testing, exploitation, and post-exploitation techniques.

## Methodology

All cheat sheets follow this proven approach:

1. **Discovery** - Port scanning and service detection
2. **Enumeration** - Information gathering and reconnaissance
3. **Authentication** - Credential attacks and testing
4. **Exploitation** - Attack execution and access gaining
5. **Post-Exploitation** - Persistence and lateral movement

## Service Categories

### Database Services

Critical database systems that often contain sensitive information and provide pathways for privilege escalation.

- **[MySQL](database/mysql.md)** - Open-source relational database management system
- **[Oracle TNS](database/oracle-tns.md)** - Enterprise database with extensive attack surface

### Mail Services

Email infrastructure services that can reveal user information and provide attack vectors.

- **[SMTP](mail/smtp.md)** - Simple Mail Transfer Protocol for email transmission
- **[IMAP/POP3](mail/imap-pop3.md)** - Email retrieval protocols for client access

### Network Services

Core network services that provide system information and management capabilities.

- **[SNMP](network/snmp.md)** - Simple Network Management Protocol for device monitoring

### File Services

Network file sharing protocols that often have misconfigurations leading to unauthorized access.

- **[SMB](file/smb.md)** - Server Message Block protocol for Windows file sharing
- **[NFS](file/nfs.md)** - Network File System for Unix/Linux environments

## Quick Reference

Each cheat sheet includes:

- **Port information** - Default and alternative ports
- **Discovery commands** - Nmap scripts and scanning techniques
- **Enumeration tools** - Specialized tools for information gathering
- **Authentication attacks** - Credential testing and brute force techniques
- **Exploitation methods** - Proven attack vectors and payloads
- **Post-exploitation** - Techniques for maintaining access and escalation

## Getting Started

1. Identify target services through port scanning
2. Select the appropriate cheat sheet
3. Follow the methodology from discovery to exploitation
4. Adapt commands to your specific environment

## Tools Required

Most techniques require standard penetration testing tools:

- Nmap for service discovery
- Hydra for credential attacks
- Metasploit for exploitation
- Custom scripts and utilities
- Service-specific tools (detailed in each guide)

## Disclaimer

These resources are intended for authorized penetration testing and educational purposes only. Always ensure proper authorization before testing any systems. Unauthorized access to computer systems is illegal and unethical.
