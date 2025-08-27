# File Services

File sharing services are common attack vectors due to frequent misconfigurations and the sensitive nature of shared data.

## Overview

File service enumeration focuses on:

- Share enumeration and access testing
- Permission analysis and bypass
- Anonymous access identification
- Credential requirement assessment
- Data extraction opportunities

## Common File Services

### SMB (Ports 445, 139)
Server Message Block protocol provides Windows-based file and printer sharing.

**Key Attack Vectors:**
- Null session enumeration
- Share permission bypass
- RID cycling for user enumeration
- Administrative share access

[View SMB Cheat Sheet](smb.md)

### NFS (Ports 2049, 111)
Network File System provides Unix/Linux distributed file sharing capabilities.

**Key Attack Vectors:**
- Export enumeration and mounting
- UID/GID manipulation
- no_root_squash exploitation
- Weak export permissions

[View NFS Cheat Sheet](nfs.md)

## General Methodology

### 1. Discovery
```bash
# Port scanning
nmap -p 139,445,111,2049 target

# Service detection
nmap -sV -sC -p <ports> target
```

### 2. Enumeration
```bash
# SMB enumeration
enum4linux target
smbclient -L //target -N

# NFS enumeration
showmount -e target
rpcinfo -p target
```

### 3. Access Testing
```bash
# Anonymous access
smbclient //target/share -N
mount -t nfs target:/export /mnt

# Authenticated access
smbclient //target/share -U username
```

### 4. Exploitation
- Extract sensitive files
- Escalate privileges
- Plant backdoors
- Lateral movement

## Security Considerations

When testing file services:

- **Data sensitivity** - File shares often contain sensitive data
- **System integrity** - Avoid modifying critical files
- **Access logging** - File access is typically logged
- **Network performance** - Large file operations can impact network

## Common Misconfigurations

### SMB Issues
- Anonymous access to sensitive shares
- Weak share permissions
- Null session information disclosure
- Administrative share exposure

### NFS Issues
- World-readable exports
- no_root_squash misconfiguration
- Weak export restrictions
- Insecure RPC configuration

## Exploitation Techniques

### SMB Exploitation
- Share enumeration and access
- User and group enumeration
- Password policy extraction
- Administrative access

### NFS Exploitation
- Export mounting and access
- UID manipulation for privilege escalation
- File system modification
- Backdoor placement

## Tools and Resources

### Essential Tools
- **nmap** - Service discovery and scripting
- **enum4linux** - SMB enumeration
- **smbclient** - SMB client access
- **showmount** - NFS export enumeration

### Specialized Tools
- **rpcclient** - RPC client for Windows
- **smbmap** - SMB share enumeration
- **crackmapexec** - SMB and other protocol testing
- **mount** - NFS mounting

Select a specific file service above to access detailed enumeration and exploitation techniques.
