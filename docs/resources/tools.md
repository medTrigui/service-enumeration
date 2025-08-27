# Essential Tools

Comprehensive list of tools used in service enumeration and penetration testing.

## Network Discovery

### Nmap
**Description:** Network exploration tool and security scanner
**Installation:** `apt-get install nmap`
**Key Features:**
- Port scanning and service detection
- OS fingerprinting
- Vulnerability scanning with NSE scripts
- Custom timing and stealth options

**Common Usage:**
```bash
# Basic scan
nmap target

# Service version detection
nmap -sV target

# Script scanning
nmap -sC target

# Comprehensive scan
nmap -sS -sV -sC -A target
```

### Masscan
**Description:** High-speed port scanner
**Installation:** `apt-get install masscan`
**Key Features:**
- Extremely fast scanning capabilities
- Internet-scale scanning
- Custom packet crafting

**Common Usage:**
```bash
# Fast port scan
masscan -p1-65535 target --rate=1000

# Specific ports
masscan -p22,80,443 target --rate=100
```

## Service-Specific Tools

### Database Tools

#### MySQL Client
**Installation:** `apt-get install mysql-client`
```bash
# Connect to MySQL
mysql -h target -u username -p

# Execute commands
mysql -h target -u root -e "SHOW DATABASES;"
```

#### Oracle Client (SQLPlus)
**Installation:** Download from Oracle website
```bash
# Connect to Oracle
sqlplus username/password@target:1521/SID

# SYSDBA access
sqlplus / as sysdba
```

### Mail Service Tools

#### SMTP User Enum
**Installation:** `apt-get install smtp-user-enum`
```bash
# User enumeration
smtp-user-enum -M VRFY -U users.txt -t target
smtp-user-enum -M EXPN -U users.txt -t target
```

#### Swaks
**Installation:** `apt-get install swaks`
```bash
# SMTP testing
swaks --to user@target --from test@test.com --server target
```

### File Service Tools

#### SMB Tools
```bash
# Enum4linux
apt-get install enum4linux
enum4linux target

# SMBClient
apt-get install smbclient
smbclient -L //target -N

# CrackMapExec
pip3 install crackmapexec
crackmapexec smb target
```

#### NFS Tools
```bash
# Showmount
apt-get install nfs-common
showmount -e target

# Mount
mount -t nfs target:/export /mnt/nfs
```

### Network Service Tools

#### SNMP Tools
```bash
# SNMP Walk
apt-get install snmp-mibs-downloader snmp
snmpwalk -c public -v1 target

# Onesixtyone
apt-get install onesixtyone
onesixtyone -c community.txt target

# SNMP Check
apt-get install snmp-check
snmp-check target
```

## Authentication Testing

### Hydra
**Description:** Network authentication cracker
**Installation:** `apt-get install hydra`
**Supported Protocols:** SSH, FTP, HTTP, SMB, SNMP, MySQL, PostgreSQL, etc.

**Common Usage:**
```bash
# SSH brute force
hydra -L users.txt -P passwords.txt ssh://target

# MySQL brute force
hydra -L users.txt -P passwords.txt mysql://target

# SMTP brute force
hydra -L users.txt -P passwords.txt smtp://target
```

### Medusa
**Description:** Parallel login brute-forcer
**Installation:** `apt-get install medusa`

**Common Usage:**
```bash
# SSH attack
medusa -h target -U users.txt -P passwords.txt -M ssh

# FTP attack
medusa -h target -U users.txt -P passwords.txt -M ftp
```

## Exploitation Frameworks

### Metasploit
**Description:** Penetration testing framework
**Installation:** `curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall && chmod 755 msfinstall && ./msfinstall`

**Common Modules:**
```bash
# Search for exploits
search mysql
search oracle
search smtp

# Use specific exploit
use exploit/multi/mysql/mysql_udf_payload
```

### SearchSploit
**Description:** Exploit database search tool
**Installation:** `apt-get install exploitdb`

**Usage:**
```bash
# Search for exploits
searchsploit mysql
searchsploit "oracle tns"
searchsploit smtp
```

## Utility Tools

### Netcat
**Description:** Network utility for reading/writing network connections
**Installation:** `apt-get install netcat`

**Usage:**
```bash
# Banner grabbing
nc target 80
nc target 25

# Listen for connections
nc -lvp 4444
```

### Telnet
**Description:** Network protocol client
**Installation:** `apt-get install telnet`

**Usage:**
```bash
# Connect to services
telnet target 25
telnet target 110
telnet target 143
```

### OpenSSL
**Description:** SSL/TLS toolkit
**Installation:** `apt-get install openssl`

**Usage:**
```bash
# SSL banner grabbing
openssl s_client -connect target:443

# SMTP STARTTLS
openssl s_client -connect target:25 -starttls smtp
```

## Custom Scripts and One-liners

### Bash One-liners
```bash
# Port scan with bash
for port in {1..1000}; do timeout 1 bash -c "</dev/tcp/target/$port" && echo "Port $port is open"; done 2>/dev/null

# HTTP title grabbing
curl -s target | grep -o '<title>.*</title>'

# Banner grab multiple ports
for port in 21 22 23 25 53 80 110 111 135 139 143 443 993 995 1433 1521 3306 5432; do echo "Port $port: $(nc -w 1 target $port < /dev/null)"; done
```

### Python Scripts
```python
# Simple port scanner
import socket
for port in range(1, 1001):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(1)
    result = sock.connect_ex(('target', port))
    if result == 0:
        print(f"Port {port} is open")
    sock.close()
```

## Tool Installation Scripts

### Kali Linux
```bash
# Update package list
apt-get update

# Install essential tools
apt-get install -y nmap hydra enum4linux smbclient showmount onesixtyone snmp

# Install additional tools
apt-get install -y crackmapexec smtp-user-enum swaks medusa
```

### Ubuntu/Debian
```bash
# Update system
apt-get update && apt-get upgrade -y

# Install from repositories
apt-get install -y nmap hydra smbclient nfs-common snmp-mibs-downloader

# Manual installations
wget https://github.com/CiscoCXSecurity/enum4linux/archive/master.zip
unzip master.zip && cd enum4linux-master
```

## Tool Categories Summary

| Category | Primary Tools | Secondary Tools |
|----------|---------------|-----------------|
| **Discovery** | nmap, masscan | zmap, unicornscan |
| **SMB** | enum4linux, smbclient | crackmapexec, smbmap |
| **SNMP** | snmpwalk, onesixtyone | snmp-check, snmpget |
| **Database** | mysql, sqlplus | sqsh, psql |
| **Mail** | smtp-user-enum, swaks | sendemail, mailx |
| **Authentication** | hydra, medusa | patator, crowbar |
| **General** | nc, telnet | socat, curl |
