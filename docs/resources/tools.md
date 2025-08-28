# Essential Tools

Quick reference for penetration testing tools used in service enumeration.

## Core Tools

### Nmap - Network Scanner
```bash
# Install: apt-get install nmap
nmap -sV -sC target          # Service detection + scripts
nmap -p1-1000 target         # Port range scan
nmap --script vuln target    # Vulnerability scan
```

### Hydra - Password Attacks
```bash
# Install: apt-get install hydra
hydra -L users.txt -P pass.txt service://target
hydra -l admin -P rockyou.txt ssh://target
```

### Metasploit - Exploitation Framework
```bash
# Install: curl https://raw.githubusercontent.com/.../msfinstall && ./msfinstall
search mysql
use exploit/multi/mysql/mysql_udf_payload
```

## Service-Specific Tools

### Database Tools
```bash
# MySQL
mysql -h target -u root -p
mysql -h target -u root -e "SHOW DATABASES;"

# MSSQL
sqlcmd -S target -U sa -P password
impacket-mssqlclient sa:password@target

# Oracle
sqlplus username/password@target:1521/SID
```

### Mail Tools
```bash
# SMTP enumeration
smtp-user-enum -M VRFY -U users.txt -t target
swaks --to user@target --from test@test.com --server target

# IMAP/POP3
curl -k 'imaps://target' --user user:pass
```

### File Service Tools
```bash
# SMB
enum4linux target
smbclient -L //target -N
crackmapexec smb target

# NFS
showmount -e target
mount -t nfs target:/export /mnt/nfs
```

### Network Tools
```bash
# SNMP
snmpwalk -c public -v1 target
onesixtyone -c community.txt target
snmp-check target
```

## Quick Commands

### Common Port Scans
```bash
# Top 1000 ports
nmap -T4 -A target

# All TCP ports
nmap -p- target

# UDP scan
nmap -sU -p 161,162,123,69 target
```

### Authentication Testing
```bash
# Multiple services
hydra -L users.txt -P passwords.txt protocol://target

# Common protocols
hydra -l admin -P rockyou.txt ssh://target
hydra -l sa -P passwords.txt mssql://target
hydra -l root -P passwords.txt mysql://target
```

## Utility Commands

### Banner Grabbing
```bash
# Netcat
nc target 80
nc target 25

# Telnet  
telnet target 25

# OpenSSL
openssl s_client -connect target:443
```

### One-liners
```bash
# Bash port scan
for port in {1..1000}; do timeout 1 bash -c "</dev/tcp/target/$port" && echo "Port $port is open"; done 2>/dev/null

# HTTP title
curl -s target | grep -o '<title>.*</title>'
```

## Installation

### Kali Linux
```bash
apt-get update
apt-get install -y nmap hydra enum4linux smbclient showmount onesixtyone snmp crackmapexec
```

### Ubuntu/Debian
```bash
apt-get update && apt-get upgrade -y
apt-get install -y nmap hydra smbclient nfs-common snmp-mibs-downloader
```

## Quick Reference

| Service | Primary Tool | Command |
|---------|--------------|---------|
| **Discovery** | nmap | `nmap -sV target` |
| **SMB** | enum4linux | `enum4linux target` |
| **SNMP** | snmpwalk | `snmpwalk -c public -v1 target` |
| **Database** | mysql/sqlcmd | `mysql -h target -u root -p` |
| **Mail** | smtp-user-enum | `smtp-user-enum -M VRFY -U users.txt -t target` |
| **Auth** | hydra | `hydra -L users.txt -P pass.txt service://target` |
