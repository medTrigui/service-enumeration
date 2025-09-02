# Network Services

Network services provide core infrastructure functionality and often expose valuable system information for reconnaissance and exploitation.

## Overview

Network service enumeration focuses on:

- System and device identification
- Configuration disclosure
- Management interface access
- Information leakage
- Administrative access vectors

## Common Network Services

### SNMP (Ports 161, 162)
Simple Network Management Protocol provides network device monitoring and management capabilities.

**Key Attack Vectors:**
- Community string enumeration
- MIB walking for information disclosure
- Write access for configuration changes
- Version-specific vulnerabilities

[View SNMP Cheat Sheet](snmp.md)

## General Methodology

### 1. Discovery
```bash
# Port scanning
nmap -sU -p 161,162 target

# Service detection
nmap -sU -sV -p 161,162 target
```

### 2. Enumeration
```bash
# Community string testing
onesixtyone -c community.txt target

# MIB walking
snmpwalk -c public -v1 target
```

### 3. Information Gathering
```bash
# System information
snmpwalk -c public -v1 target 1.3.6.1.2.1.1

# Network interfaces
snmpwalk -c public -v1 target 1.3.6.1.2.1.2.2.1
```

### 4. Exploitation
- Leverage exposed information
- Attempt configuration changes
- Extract sensitive data
- Map network infrastructure

## Security Considerations

When testing network services:

- **Network impact** - Be cautious with network-wide services
- **Device stability** - Avoid overwhelming network devices
- **Information sensitivity** - Network topology data can be sensitive
- **Monitoring detection** - Network services are often monitored

## Common Information Exposed

### SNMP Data
- System information and versions
- Network interface configurations
- Routing table information
- Running processes and services
- Installed software
- User accounts
- Network shares and mounted filesystems

## Exploitation Techniques

### Read Access
- System reconnaissance
- Network mapping
- Service identification
- Vulnerability assessment

### Write Access
- Configuration modification
- Service manipulation
- Access control bypass
- Denial of service

## Tools and Resources

### Essential Tools
- **nmap** - Service discovery and scripting
- **onesixtyone** - SNMP community string enumeration
- **snmpwalk** - MIB tree walking
- **snmp-check** - Comprehensive SNMP enumeration

### Specialized Tools
- **snmpget** - Specific OID retrieval
- **snmpset** - SNMP value modification
- **snmptranslate** - OID translation
- **snmpdf** - Disk space enumeration

Select a specific network service above to access detailed enumeration and exploitation techniques.

