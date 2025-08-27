# SNMP Enumeration Cheat Sheet

## Table of Contents
1. [Overview](#overview)
2. [Ports & Protocols](#ports--protocols)
3. [Quick Discovery](#quick-discovery)
4. [Essential Enumeration Workflow](#essential-enumeration-workflow)
5. [SNMP Versions & Security](#snmp-versions--security)
6. [Windows-Specific Enumeration](#windows-specific-enumeration)
7. [Common OID Reference](#common-oid-reference)
8. [Automated Tools](#automated-tools)
9. [Quick Reference Commands](#quick-reference-commands)
10. [Testing Checklist](#testing-checklist)
11. [Critical Security Issues](#critical-security-issues)

---

## Overview

**SNMP (Simple Network Management Protocol)** is a network protocol for monitoring and managing network devices including routers, switches, servers, and IoT devices. It operates over UDP and provides extensive system information through a hierarchical database structure.

### Why SNMP Matters in Pentesting
- **Information Disclosure**: Reveals extensive system information
- **Weak Authentication**: Often uses default community strings  
- **Network Mapping**: Exposes network topology and devices
- **Credential Harvesting**: May contain usernames and system details
- **Legacy Security**: Older versions lack proper encryption

### Common Attack Vectors
- **Default Community Strings**: "public" and "private" often unchanged
- **Community String Brute Force**: Weak authentication mechanisms
- **Information Enumeration**: System users, processes, network configuration
- **Write Access Exploitation**: Device reconfiguration capabilities

---

## Ports & Protocols

```
161/udp - SNMP (Simple Network Management Protocol)
162/udp - SNMP Trap (asynchronous notifications)
```

### SNMP Versions
| Version | Authentication | Encryption | Security Level |
|---------|---------------|------------|----------------|
| **SNMPv1** | Community strings | None | Very Low |
| **SNMPv2c** | Community strings | None | Very Low |  
| **SNMPv3** | Username/Password | AES/3DES | High |

---

## Quick Discovery

```bash
# UDP port scan for SNMP
nmap -sU --open -p 161 target

# SNMP service detection
nmap -sU -p 161 --script=snmp-info target

# Comprehensive SNMP discovery
nmap -sU -p 161 --script=snmp-* target
```

---

## Essential Enumeration Workflow

### Community String Discovery
```bash
# Test common default strings
snmpwalk -c public -v1 target
snmpwalk -c private -v1 target
snmpwalk -c community -v1 target

# Brute force community strings
onesixtyone -c community.txt target
onesixtyone -i targets.txt -c community.txt

# Create community wordlist
echo -e "public\nprivate\ncommunity\nmanager\nadmin\ndefault\nsnmp\nmonitor" > community.txt

# Nmap brute force
nmap -sU -p 161 --script=snmp-brute target
```

### Complete MIB Enumeration
```bash
# Walk entire MIB tree
snmpwalk -c public -v1 target

# Walk specific branches
snmpwalk -c public -v1 target 1.3.6.1.2.1.1    # System info
snmpwalk -c public -v1 target 1.3.6.1.2.1.25   # Host resources
snmpwalk -c public -v1 target 1.3.6.1.4.1.77    # Windows-specific

# Bulk operations (SNMPv2c)
snmpbulkwalk -c public -v2c target

# BRAA for fast enumeration
braa public@target:.1.3.6.*
```

### System Information Gathering
```bash
# System description
snmpget -c public -v1 target 1.3.6.1.2.1.1.1.0

# System uptime
snmpwalk -c public -v1 target 1.3.6.1.2.1.1.3.0

# Hostname
snmpwalk -c public -v1 target 1.3.6.1.2.1.1.5

# System contact
snmpget -c public -v1 target 1.3.6.1.2.1.1.4.0
```

---

## SNMP Versions & Security

### SNMPv1/v2c (Insecure)
```bash
# SNMPv1 enumeration
snmpwalk -v1 -c public target

# SNMPv2c enumeration  
snmpwalk -v2c -c public target

# VULNERABILITY: Plaintext community strings, no encryption
```

### SNMPv3 (Secure)
```bash
# SNMPv3 with authentication
snmpwalk -v3 -u username -A password -l authNoPriv target

# SNMPv3 with authentication and encryption
snmpwalk -v3 -u username -A authpass -X privpass -l authPriv target
```

### Write Access Testing
```bash
# Test with private community string
snmpset -c private -v1 target 1.3.6.1.2.1.1.4.0 s "Test Contact"

# Verify write access
snmpget -c private -v1 target 1.3.6.1.2.1.1.4.0
```

---

## Windows-Specific Enumeration

### User Account Information
```bash
# Windows user accounts
snmpwalk -c public -v1 target 1.3.6.1.4.1.77.1.2.25

# Enumerate specific users
for i in {1..20}; do 
    snmpget -c public -v1 target 1.3.6.1.4.1.77.1.2.25.1.1.$i
done
```

### Process and Service Information
```bash
# Running programs/processes
snmpwalk -c public -v1 target 1.3.6.1.2.1.25.4.2.1.2

# Process paths
snmpwalk -c public -v1 target 1.3.6.1.2.1.25.4.2.1.4

# Windows services via Nmap
nmap -sU -p 161 --script=snmp-win32-services target
```

### Network and Share Information
```bash
# Windows hostname
snmpwalk -c public -v1 target 1.3.6.1.2.1.1.5

# Windows share information (method 1)
snmpwalk -c public -v1 target 1.3.6.1.4.1.77.1.2.3.1.1

# Windows share information (method 2)
snmpwalk -c public -v1 target 1.3.6.1.4.1.77.1.2.27

# Windows shares via Nmap
nmap -sU -p 161 --script=snmp-win32-shares target

# TCP listening ports
snmpwalk -c public -v1 target 1.3.6.1.2.1.6.13.1.3
```

### Software Information
```bash
# Installed software names
snmpwalk -c public -v1 target 1.3.6.1.2.1.25.6.3.1.2

# Software installation paths
snmpwalk -c public -v1 target 1.3.6.1.2.1.25.6.3.1.4

# System processes count
snmpwalk -c public -v1 target 1.3.6.1.2.1.25.1.6.0
```

### Network Configuration
```bash
# Network interfaces
snmpwalk -c public -v1 target 1.3.6.1.2.1.2.2.1.2

# IP addresses
snmpwalk -c public -v1 target 1.3.6.1.2.1.4.20.1.1

# TCP connections
snmpwalk -c public -v1 target 1.3.6.1.2.1.6.13.1.3

# Network statistics via Nmap
nmap -sU -p 161 --script=snmp-netstat target
```

---

## Common OID Reference

### System Information OIDs
```
1.3.6.1.2.1.1.1.0    - System Description
1.3.6.1.2.1.1.3.0    - System Uptime
1.3.6.1.2.1.1.4.0    - System Contact
1.3.6.1.2.1.1.5.0    - System Name/Hostname
1.3.6.1.2.1.1.6.0    - System Location
```

### Windows-Specific OIDs
```
1.3.6.1.4.1.77.1.2.25     - Windows User Accounts
1.3.6.1.2.1.25.4.2.1.2    - Running Programs/Processes
1.3.6.1.2.1.25.4.2.1.4    - Process Paths
1.3.6.1.4.1.77.1.2.3.1.1  - Windows Share Information
1.3.6.1.4.1.77.1.2.27     - Windows Share Information (alt)
1.3.6.1.2.1.6.13.1.3      - TCP Local Ports
1.3.6.1.2.1.25.6.3.1.2    - Software Names
1.3.6.1.2.1.25.1.6.0      - System Processes Count
```

### Network Information OIDs
```
1.3.6.1.2.1.2.2.1.2       - Network Interfaces
1.3.6.1.2.1.4.20.1.1      - IP Addresses
1.3.6.1.2.1.4.21.1.1      - Routing Table
1.3.6.1.2.1.4.22.1.2      - ARP Table
1.3.6.1.2.1.7.5.1.2       - UDP Listeners
```

---

## Automated Tools

### snmp-check
```bash
# Comprehensive SNMP enumeration
snmp-check target
snmp-check target -c public
snmp-check target -c private

# Specify custom community string
snmp-check target -c custom_community
```

### Nmap NSE Scripts
```bash
# System description
nmap -sU -p 161 --script=snmp-sysdescr target

# Process enumeration
nmap -sU -p 161 --script=snmp-processes target

# Network interface information
nmap -sU -p 161 --script=snmp-interfaces target

# SNMP brute force
nmap -sU -p 161 --script=snmp-brute target

# Windows-specific scripts
nmap -sU -p 161 --script=snmp-win32-services target
nmap -sU -p 161 --script=snmp-win32-shares target
```

### Mass SNMP Scanning
```bash
# Scan multiple targets
onesixtyone -i targets.txt -c community.txt

# Nmap across ranges
nmap -sU -p 161 --script=snmp-info 192.168.1.0/24

# Save results for analysis
onesixtyone -i targets.txt -c community.txt -o snmp_results.txt
```

---

## Quick Reference Commands

### Discovery
```bash
nmap -sU -p 161 --script=snmp-info target
onesixtyone -c community.txt target
snmp-check target
```

### Basic Enumeration
```bash
snmpwalk -c public -v1 target
snmpwalk -c public -v1 target 1.3.6.1.2.1.1
snmpget -c public -v1 target 1.3.6.1.2.1.1.1.0
```

### Windows-Specific
```bash
snmpwalk -c public -v1 target 1.3.6.1.4.1.77.1.2.25    # Users
snmpwalk -c public -v1 target 1.3.6.1.2.1.25.4.2.1.2   # Processes
snmpwalk -c public -v1 target 1.3.6.1.4.1.77.1.2.27    # Shares
```

### Network Information
```bash
snmpwalk -c public -v1 target 1.3.6.1.2.1.4.20.1.1     # IP addresses
snmpwalk -c public -v1 target 1.3.6.1.2.1.6.13.1.3     # TCP ports
snmpwalk -c public -v1 target 1.3.6.1.2.1.2.2.1.2      # Interfaces
```

---

## Testing Checklist

#### Discovery & Detection
- [ ] UDP port 161 open and responding
- [ ] SNMP version identification
- [ ] Community string enumeration
- [ ] SNMPv3 user discovery (if applicable)

#### Authentication Testing
- [ ] Default community string testing
- [ ] Community string brute force
- [ ] SNMPv3 authentication testing
- [ ] Write access verification

#### Information Enumeration
- [ ] System information gathering
- [ ] User account enumeration
- [ ] Process and service discovery
- [ ] Network configuration extraction
- [ ] Installed software identification

#### Windows-Specific Testing
- [ ] User account enumeration
- [ ] Share information gathering
- [ ] Running process identification
- [ ] Service enumeration

#### Security Assessment
- [ ] Community string strength
- [ ] SNMP version security
- [ ] Information disclosure assessment
- [ ] Write access implications

---

## Critical Security Issues

### Default Community Strings
- **Impact**: Complete system information disclosure
- **Detection**: Successful authentication with "public"/"private"
- **Exploitation**: `snmpwalk -c public -v1 target`

### Information Disclosure
- **Impact**: Reconnaissance data, user enumeration, network mapping
- **Detection**: Successful SNMP queries returning sensitive data
- **Exploitation**: User account discovery, process enumeration

### Write Access Vulnerability
- **Impact**: Device reconfiguration, service disruption
- **Detection**: Successful SNMP SET operations
- **Exploitation**: `snmpset -c private -v1 target 1.3.6.1.2.1.1.4.0 s "Test"`

### SNMPv1/v2c Usage
- **Impact**: Credential interception, man-in-the-middle attacks
- **Detection**: SNMP version identification
- **Exploitation**: Network traffic analysis, credential harvesting

### Weak Community Strings
- **Impact**: Unauthorized access, information disclosure
- **Detection**: Successful brute force attacks
- **Exploitation**: Complete SNMP access with guessable credentials

---

## Community String Wordlists

### Common Default Strings
```
public
private
community
manager
admin
default
snmp
monitor
guest
cisco
read
write
test
security
network
```

### Wordlist Locations
- SecLists: `/SecLists/Discovery/SNMP/snmp_default_pass.txt`
- SecLists: `/SecLists/Discovery/SNMP/snmp_default_community_strings.txt`
- Custom list: Create based on target environment

---

## Advanced Techniques

### Custom OID Queries
```bash
# Query specific OIDs
snmpget -c public -v1 target 1.3.6.1.2.1.1.1.0    # sysDescr
snmpget -c public -v1 target 1.3.6.1.2.1.1.5.0    # sysName

# Walk OID ranges
for oid in $(seq 1 100); do
    snmpget -c public -v1 target 1.3.6.1.2.1.1.$oid.0 2>/dev/null
done
```

### SNMP Write Operations
```bash
# Modify system contact (if write access available)
snmpset -c private -v1 target 1.3.6.1.2.1.1.4.0 s "Attacker Contact"

# Test for dangerous write access
snmpset -c private -v1 target 1.3.6.1.2.1.1.4.0 s "Test"
```

### Information Extraction
```bash
# Extract potentially sensitive data
snmpwalk -c public -v1 target 1.3.6.1.4.1.77.1.2.25 | grep -i admin
snmpwalk -c public -v1 target 1.3.6.1.2.1.25.4.2.1.2 | grep -i sql
```