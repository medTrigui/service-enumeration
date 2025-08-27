# SMB Enumeration Cheat Sheet

## Ports & Protocols
```
139/tcp - NetBIOS Session Service (SMB over NetBIOS)
445/tcp - SMB over TCP (Direct, modern)
```

## Quick Discovery
```bash
# Port scan
nmap -p 139,445 --open -sV target

# Service detection
nmap -p 139,445 -sC -sV target
```

---

## Essential Enumeration Workflow

### 1. Host Enumeration
```bash
# NetExec - Get OS info, SMB version, signing status
netexec smb target

# Example output shows: OS, domain, SMB version, signing enabled/disabled
```

### 2. Share Enumeration

#### Null Session Attacks
```bash
# smbclient - List shares (unauthenticated)
smbclient -L -N //target
# VULNERABILITY: Anonymous share enumeration allowed

# NetExec variations
netexec smb target --shares
netexec smb target -u guest -p '' --shares
netexec smb target -u '' -p '' --shares
```

#### With Credentials
```bash
# NetExec with valid creds
netexec smb target -u username -p password --shares

# smbclient with creds
smbclient -L //target -U username%password
```

### 3. Share Access

#### Null Session Access
```bash
# Access share without authentication
smbclient //target/share -N
# VULNERABILITY: Anonymous access to sensitive shares

# Common shares to test
smbclient //target/IPC$ -N
smbclient //target/C$ -N
smbclient //target/ADMIN$ -N
```

#### Authenticated Access
```bash
# With username/password
smbclient //target/share -U username%password

# Kerberos authentication
smbclient.py 'domain/user:pass@target' -k -no-pass
```

---

## File Enumeration Techniques

### smbclient Commands
```bash
# Once connected to a share:
ls                    # List files
cd directory         # Change directory
get filename         # Download file
put filename         # Upload file
help                 # Show all commands
!command             # Execute local shell command
```

### Automated File Discovery
```bash
# NetExec spider module
netexec smb target -u user -p pass -M spider_plus

# Output saved to /tmp/nxc_spider_plus/target.json
```

### Mass File Search
```bash
# MANSPIDER - Search for sensitive files
manspider --threads 256 target -u username -p password --filetype pdf,doc,docx,xls,xlsx
```

---

## User Enumeration

### RID Cycling
```bash
# Impacket lookupsid
lookupsid.py guest@target -no-pass

# NetExec RID brute force
netexec smb target -u guest -p '' --rid-brute

# Manual RID cycling with rpcclient
rpcclient -U "" target
rpcclient $> lookupsids S-1-5-21-[DOMAIN-SID]-500
```

### SAM Remote Protocol
```bash
# Dump user information via SAM
samrdump.py domain/user:pass@target
```

### Manual User Enumeration
```bash
# rpcclient interactive session
rpcclient -U "" target

# Common rpcclient commands:
enumdomusers         # List domain users
queryuser [RID]      # Get user details
enumdomgroups        # List domain groups
querygroup [RID]     # Get group details
```

---

## Vulnerability Scanning
```bash
# Nmap SMB vulnerability scripts
nmap --script smb-vuln* -p 139,445 target

# Common vulnerabilities to check:
# - MS17-010 (EternalBlue)
# - MS08-067 (Conficker)
# - MS06-025
```

---

## Common Default Shares

| Share    | Type | Description | Risk Level |
|----------|------|-------------|------------|
| ADMIN$   | Disk | Remote admin | HIGH |
| C$       | Disk | Default root share | HIGH |
| IPC$     | IPC  | Inter-process communication | MEDIUM |
| NETLOGON | Disk | Logon server share | MEDIUM |
| SYSVOL   | Disk | Domain policies | MEDIUM |

---

## Attack Scenarios

### Scenario 1: Null Session Share Access
```bash
# Discovery
smbclient -L -N //target

# Access attempt
smbclient //target/Users -N
# If successful: VULNERABILITY - Anonymous access to user directories
```

### Scenario 2: Guest Account Enumeration
```bash
# Share enumeration
netexec smb target -u guest -p '' --shares

# User enumeration
netexec smb target -u guest -p '' --rid-brute
# If successful: VULNERABILITY - Information disclosure via guest account
```

### Scenario 3: Authenticated Enumeration
```bash
# Full enumeration with valid creds
netexec smb target -u user -p pass --shares
netexec smb target -u user -p pass -M spider_plus
samrdump.py domain/user:pass@target
```

---

## Key Security Issues to Look For

### High-Risk Configurations
- **Anonymous share listing**: `smbclient -L -N //target` succeeds
- **Guest account enabled**: Authentication with guest:'' works
- **Writable shares**: Upload capabilities without authentication
- **Sensitive file exposure**: Config files, backups in accessible shares
- **SMBv1 enabled**: Legacy protocol with known vulnerabilities

### Common Misconfigurations
```bash
# Check for writable shares
echo "test" | smbclient //target/share -N -c "put - test.txt"

# Look for backup files
# .bak, .old, .backup extensions in shares

# Check for configuration files
# web.config, database.conf, etc.
```

---

## Advanced Techniques

### Kerberos Authentication
```bash
# When NTLM is disabled
netexec smb target -k -u user -p pass

# Impacket with Kerberos
smbclient.py 'domain/user:pass@target' -k -no-pass
```

### Password Spraying
```bash
# Test common passwords against user list
netexec smb target -u users.txt -p 'Password123' --continue-on-success
```

---

## Quick Reference Commands

### Initial Discovery
```bash
nmap -p 139,445 --open target
netexec smb target
```

### Share Enumeration
```bash
smbclient -L -N //target
netexec smb target -u guest -p '' --shares
```

### File Access
```bash
smbclient //target/share -N
smbclient //target/share -U user%pass
```

### User Enumeration
```bash
lookupsid.py guest@target -no-pass
netexec smb target -u guest -p '' --rid-brute
```

### Vulnerability Scan
```bash
nmap --script smb-vuln* -p 139,445 target
```

---

## Testing Checklist

#### Discovery
- [ ] Port 139/445 open
- [ ] SMB version identification
- [ ] OS fingerprinting
- [ ] Domain information

#### Authentication Testing
- [ ] Null session access
- [ ] Guest account access
- [ ] Anonymous share listing
- [ ] Default credentials

#### Share Enumeration
- [ ] List all available shares
- [ ] Test access to each share
- [ ] Identify writable shares
- [ ] Check share permissions

#### File Discovery
- [ ] Enumerate files in accessible shares
- [ ] Search for sensitive files
- [ ] Download interesting files
- [ ] Check for backup files

#### User Enumeration
- [ ] RID cycling attack
- [ ] SAM remote protocol
- [ ] Domain user listing
- [ ] User detail extraction

#### Vulnerability Assessment
- [ ] SMB vulnerability scan
- [ ] SMBv1 detection
- [ ] Signing requirements
- [ ] Known CVE checks

---

## Critical Vulnerabilities

### Anonymous Access
- **Impact**: Information disclosure, potential data theft
- **Detection**: `smbclient -L -N //target` succeeds
- **Exploitation**: Access sensitive shares without authentication

### Guest Account Enabled
- **Impact**: User enumeration, share access
- **Detection**: `netexec smb target -u guest -p ''` succeeds
- **Exploitation**: RID cycling, share enumeration

### Writable Shares
- **Impact**: File upload, potential malware deployment
- **Detection**: Successful file upload to share
- **Exploitation**: Upload webshells, malicious executables