# Quick Start Guide

Get started with service enumeration using proven methodologies and tools.

## Prerequisites

Ensure you have the following tools installed:

```bash
# Essential tools
apt-get update
apt-get install nmap hydra smbclient nfs-common snmp

# Optional but recommended
apt-get install enum4linux nbtscan onesixtyone snmp-mibs-downloader
```

## Basic Workflow

### 1. Discovery Phase

Start with comprehensive port scanning:

```bash
# Quick scan for common ports
nmap -sS -O -sV target

# Comprehensive scan
nmap -sS -A -O -sV -p- target

# UDP scan for specific services
nmap -sU -p 161,162,123,69 target
```

### 2. Service Identification

Identify specific services and versions:

```bash
# Banner grabbing
nmap -sV -p <ports> target

# Script scanning
nmap --script=default,safe -p <ports> target
```

### 3. Enumeration

Use service-specific enumeration techniques:

```bash
# SMB enumeration
enum4linux target
smbclient -L //target -N

# SNMP enumeration
onesixtyone -c community.txt target
snmpwalk -c public -v1 target

# DNS enumeration
dnsrecon -d domain.com
```

### 4. Authentication Testing

Test for weak or default credentials:

```bash
# Hydra password attacks
hydra -L users.txt -P passwords.txt service://target

# Service-specific auth testing
smbclient //target/share -U username
```

## Common Port Reference

| Port | Service | Common Tools |
|------|---------|--------------|
| 21 | FTP | nmap, hydra, ftp |
| 22 | SSH | nmap, hydra, ssh |
| 25 | SMTP | nmap, smtp-user-enum |
| 53 | DNS | nmap, dnsrecon, dig |
| 80/443 | HTTP/HTTPS | nmap, nikto, dirb |
| 110/995 | POP3 | nmap, hydra |
| 111 | RPC | nmap, rpcinfo |
| 135 | MSRPC | nmap, rpcclient |
| 139/445 | SMB | nmap, enum4linux, smbclient |
| 143/993 | IMAP | nmap, hydra |
| 161/162 | SNMP | nmap, onesixtyone, snmpwalk |
| 389 | LDAP | nmap, ldapsearch |
| 993/995 | IMAPS/POP3S | nmap, openssl |
| 1433 | MSSQL | nmap, sqsh |
| 1521 | Oracle | nmap, sqlplus |
| 2049 | NFS | nmap, showmount |
| 3306 | MySQL | nmap, mysql |
| 5432 | PostgreSQL | nmap, psql |

## Methodology Templates

### Network Reconnaissance

```bash
# Host discovery
nmap -sn network/24

# Port discovery
masscan -p1-65535 target --rate=1000

# Service enumeration
nmap -sV -sC -p <discovered_ports> target
```

### Service-Specific Testing

Follow this pattern for each discovered service:

1. **Basic enumeration**
2. **Version identification**
3. **Vulnerability scanning**
4. **Authentication testing**
5. **Exploitation attempts**

### Documentation

Document findings systematically:

- Service versions and configurations
- Identified vulnerabilities
- Successful authentication attempts
- Exploitation results
- Remediation recommendations

## Next Steps

Once you've completed basic reconnaissance:

1. Review service-specific cheat sheets
2. Perform targeted enumeration
3. Test authentication mechanisms
4. Attempt exploitation
5. Document all findings

Select the appropriate cheat sheet based on discovered services to continue with detailed enumeration and exploitation techniques.
