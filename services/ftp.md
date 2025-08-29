# File Transfer Protocol (FTP) Enumeration

File Transfer Protocol (FTP) is one of the oldest protocols on the Internet, running on the application layer of TCP/IP for transferring files between client and server.

## Service Information

**Default Ports:** 21/tcp (control), 20/tcp (data)
**Protocol:** TCP
**Service:** FTP Server (vsftpd, ProFTPd, FileZilla Server)

## Discovery

### Port Scanning
```bash
# Nmap scan for FTP
nmap -p 21 target
nmap -sV -p 21 target

# Service detection with scripts
nmap -sV -p 21 -sC target
nmap -sV -p 21 -A target

# FTP-specific scripts
nmap --script ftp-* target
nmap --script ftp-anon,ftp-bounce,ftp-brute target
```

### Banner Grabbing
```bash
# Netcat
nc -nv target 21

# Telnet
telnet target 21

# OpenSSL (for FTPS)
openssl s_client -connect target:21 -starttls ftp
```

## Enumeration

### Anonymous Access Testing
```bash
# Standard FTP client
ftp target
# Username: anonymous
# Password: (blank) or anonymous

# Passive mode
ftp -p target

# Using wget
wget -m --no-passive ftp://anonymous:anonymous@target

# curl
curl ftp://target --user anonymous:
```

### Service Information
```bash
# FTP client commands for enumeration
ftp> status
ftp> syst
ftp> help
ftp> feat

# Enable debugging
ftp> debug
ftp> trace
```

### Directory Enumeration
```bash
# List directories
ftp> ls
ftp> ls -la
ftp> ls -R

# Navigate directories
ftp> pwd
ftp> cd directory
ftp> cd ..

# Download file listings
ftp> ls > listing.txt
```

## Authentication

### Default Credentials
```
Common defaults:
- anonymous / (blank)
- anonymous / anonymous
- anonymous / guest
- ftp / ftp
- admin / admin
- admin / password
```

### Brute Force Attacks
```bash
# Hydra
hydra -L users.txt -P passwords.txt ftp://target
hydra -l admin -P passwords.txt ftp://target
hydra -t 3 -s 21 -L users.txt -P passwords.txt target ftp

# Medusa
medusa -h target -u admin -P passwords.txt -M ftp
medusa -h target -U users.txt -P passwords.txt -M ftp

# Ncrack
ncrack -p 21 --user admin -P passwords.txt target
```

### Manual Authentication
```bash
# Telnet method
telnet target 21
USER username
PASS password

# FTP client
ftp target
Name: username
Password: password
```

## Exploitation

### Anonymous Upload Testing
```bash
# Test upload capabilities
ftp> put testfile.txt
ftp> get testfile.txt

# Create test file
echo "test" > testfile.txt
ftp> binary
ftp> put testfile.txt
```

### File Operations
```bash
# Download files
ftp> get filename
ftp> mget *.txt

# Upload files
ftp> put localfile.txt
ftp> mput *.txt

# Binary mode (important for executables)
ftp> binary
ftp> ascii
```

### Directory Operations
```bash
# Create directories
ftp> mkdir testdir

# Remove files/directories
ftp> delete filename
ftp> rmdir directoryname

# Rename files
ftp> rename oldname newname
```

### Bulk Download
```bash
# Download entire FTP site
wget -m --no-passive ftp://anonymous:anonymous@target

# Download specific directory
wget -r --no-passive ftp://anonymous:anonymous@target/directory/

# Mirror with curl
curl -T "{file1,file2}" ftp://target --user username:password
```

## Advanced Techniques

### FTP Bounce Attack
```bash
# Using Nmap
nmap -b username:password@target:21 victim_target

# Manual bounce
PORT 192,168,1,100,0,22
```

### TFTP Enumeration
```bash
# TFTP discovery (UDP 69)
nmap -sU -p 69 target

# TFTP client
tftp target
> get filename
> put filename
> status
```

### Mount FTP Locally
```bash
# Install curlftpfs
sudo apt-get install curlftpfs

# Mount FTP
mkdir /mnt/ftp
curlftpfs ftp-user:ftp-pass@target /mnt/ftp/

# Allow other users
curlftpfs -o allow_other ftp-user:ftp-pass@target /mnt/ftp/

# Unmount
fusermount -u /mnt/ftp
```

## Post-Exploitation

### Web Shell Upload
```bash
# Upload PHP web shell (if FTP connected to web server)
ftp> binary
ftp> put shell.php

# Upload ASPX shell
ftp> put shell.aspx

# Upload JSP shell
ftp> put shell.jsp
```

### Configuration File Access
```bash
# Look for sensitive files
ftp> get .htaccess
ftp> get web.config
ftp> get config.php
ftp> get wp-config.php

# Download logs
ftp> get access.log
ftp> get error.log
```

### Privilege Escalation
```bash
# Upload files to writable directories
# Common web directories:
# /var/www/html/
# /var/www/
# C:\inetpub\wwwroot\
# /usr/local/apache2/htdocs/

# Test for world-writable directories
ftp> ls -la
```

## Nmap Scripts

### FTP-Specific Scripts
```bash
# All FTP scripts
nmap --script ftp-* target

# Anonymous access check
nmap --script ftp-anon target

# Brute force
nmap --script ftp-brute target

# Vulnerability checks
nmap --script ftp-vuln-* target

# System information
nmap --script ftp-syst target

# vsftpd backdoor check
nmap --script ftp-vsftpd-backdoor target

# ProFTPd backdoor check
nmap --script ftp-proftpd-backdoor target
```

### Script Tracing
```bash
# Detailed script execution
nmap -sV -p21 -sC -A target --script-trace

# Update NSE database
sudo nmap --script-updatedb
```

## Common Vulnerabilities

### Anonymous Access
- Unauthorized file access
- Information disclosure
- File upload capabilities

### Weak Credentials
- Default usernames and passwords
- Brute force susceptible accounts

### Configuration Issues
- World-writable directories
- Sensitive file exposure
- Excessive permissions

### Known Exploits
```bash
# vsftpd 2.3.4 backdoor
nmap --script ftp-vsftpd-backdoor target

# ProFTPd backdoor
nmap --script ftp-proftpd-backdoor target

# Check for specific CVEs
searchsploit vsftpd
searchsploit proftpd
```

## Tools

### FTP Clients
```bash
# Standard FTP client
ftp target

# sftp (secure)
sftp user@target

# lftp (advanced)
lftp ftp://target

# FileZilla (GUI)
filezilla
```

### Specialized Tools
```bash
# ftp-upload (automated upload)
ftp-upload -h target -u username -p password -f file.txt

# ncftp (enhanced FTP client)
ncftp target

# wget (non-interactive)
wget ftp://target/file.txt
```

## Security Considerations

### Detection Evasion
```bash
# Use passive mode
ftp -p target

# Slow down connections
ftp> quote pasv

# Change transfer mode
ftp> binary
ftp> ascii
```

### Log Analysis
- FTP servers typically log all connections
- Monitor for failed login attempts
- Check upload/download activity
- Review configuration changes

### Defensive Measures
- Disable anonymous access if not needed
- Implement strong authentication
- Use FTPS/SFTP for encryption
- Regular security updates
- File permission auditing
- Rate limiting and fail2ban

## Configuration Files

### Common Locations
```bash
# vsftpd
/etc/vsftpd.conf
/etc/vsftpd/

# ProFTPd
/etc/proftpd.conf
/usr/local/etc/proftpd.conf

# Pure-FTPd
/etc/pure-ftpd/

# FileZilla Server
C:\Program Files\FileZilla Server\
```

### Dangerous Settings
```bash
# vsftpd dangerous configurations
anonymous_enable=YES
anon_upload_enable=YES
anon_mkdir_write_enable=YES
no_anon_password=YES
write_enable=YES
```

## References

- [RFC 959 - File Transfer Protocol](https://tools.ietf.org/html/rfc959)
- [vsftpd Documentation](https://security.appspot.com/vsftpd.html)
- [ProFTPd Documentation](http://www.proftpd.org/)
- [HackTricks FTP](https://book.hacktricks.xyz/pentesting/pentesting-ftp)
