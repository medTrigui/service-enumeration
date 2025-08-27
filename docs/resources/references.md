# References and Resources

Additional resources for service enumeration and penetration testing.

## Official Documentation

### Service Protocols
- **RFC 821** - Simple Mail Transfer Protocol (SMTP)
- **RFC 1939** - Post Office Protocol Version 3 (POP3)
- **RFC 3501** - Internet Message Access Protocol Version 4 (IMAP4)
- **RFC 1157** - Simple Network Management Protocol (SNMP)
- **RFC 1094** - Network File System Protocol (NFS)

### Database Documentation
- **MySQL Documentation** - https://dev.mysql.com/doc/
- **Oracle Database Documentation** - https://docs.oracle.com/database/
- **PostgreSQL Documentation** - https://www.postgresql.org/docs/
- **Microsoft SQL Server** - https://docs.microsoft.com/en-us/sql/

## Security Resources

### Vulnerability Databases
- **National Vulnerability Database (NVD)** - https://nvd.nist.gov/
- **Common Vulnerabilities and Exposures (CVE)** - https://cve.mitre.org/
- **Exploit Database** - https://www.exploit-db.com/
- **SecurityFocus** - https://www.securityfocus.com/vulnerabilities

### Testing Methodologies
- **OWASP Testing Guide** - https://owasp.org/www-project-web-security-testing-guide/
- **NIST SP 800-115** - Technical Guide to Information Security Testing and Assessment
- **PTES** - Penetration Testing Execution Standard
- **OSSTMM** - Open Source Security Testing Methodology Manual

## Tool Documentation

### Network Scanning
- **Nmap Reference Guide** - https://nmap.org/book/
- **Nmap Scripting Engine (NSE)** - https://nmap.org/nsedoc/
- **Masscan Documentation** - https://github.com/robertdavidgraham/masscan

### Service-Specific Tools
- **Enum4linux** - https://github.com/CiscoCXSecurity/enum4linux
- **SNMPwalk Manual** - https://linux.die.net/man/1/snmpwalk
- **Hydra Documentation** - https://github.com/vanhauser-thc/thc-hydra
- **Metasploit Unleashed** - https://www.metasploit.com/unleashed/

## Default Credentials

### Database Services
```
MySQL:
- root / (blank)
- root / root
- root / mysql
- root / password

Oracle:
- sys / change_on_install
- system / manager
- scott / tiger
- hr / hr

MSSQL:
- sa / (blank)
- sa / sa
- sa / password
```

### Network Devices
```
SNMP Communities:
- public
- private
- community
- manager
- admin

Common Router/Switch:
- admin / admin
- admin / password
- cisco / cisco
- root / (blank)
```

## Port Reference

### Common Service Ports
```
21    - FTP
22    - SSH
23    - Telnet
25    - SMTP
53    - DNS
69    - TFTP
80    - HTTP
110   - POP3
111   - RPC
135   - MS-RPC
139   - NetBIOS
143   - IMAP
161   - SNMP
389   - LDAP
443   - HTTPS
445   - SMB
465   - SMTPS
587   - SMTP Submission
993   - IMAPS
995   - POP3S
1433  - MSSQL
1521  - Oracle TNS
2049  - NFS
3306  - MySQL
3389  - RDP
5432  - PostgreSQL
5900  - VNC
```

## Command Cheat Sheets

### Nmap Quick Reference
```bash
# Host discovery
nmap -sn network/24

# TCP SYN scan
nmap -sS target

# Service version detection
nmap -sV target

# OS detection
nmap -O target

# Script scanning
nmap -sC target

# Aggressive scan
nmap -A target

# UDP scan
nmap -sU target

# Fast scan
nmap -F target

# Custom ports
nmap -p 1-100,443,8080 target
```

### PowerShell Equivalents
```powershell
# Port scan
1..1024 | % {echo ((new-object Net.Sockets.TcpClient).Connect("target",$_)) "Port $_ is open"} 2>$null

# Banner grab
$client = New-Object System.Net.Sockets.TcpClient("target", 80)
$stream = $client.GetStream()
$writer = New-Object System.IO.StreamWriter($stream)
$reader = New-Object System.IO.StreamReader($stream)
```

## Legal and Ethical Guidelines

### Authorization Requirements
- **Written permission** from system owners
- **Scope definition** and boundaries
- **Time windows** for testing activities
- **Emergency contacts** and procedures

### Best Practices
1. **Test only authorized systems**
2. **Document all activities**
3. **Avoid service disruption**
4. **Protect sensitive data**
5. **Report findings responsibly**

### Legal Frameworks
- **Computer Fraud and Abuse Act (CFAA)** - United States
- **Computer Misuse Act** - United Kingdom
- **Criminal Code** - Canada
- **Cybersecurity Law** - European Union

## Professional Certifications

### Entry Level
- **CompTIA Security+**
- **CompTIA PenTest+**
- **EC-Council CEH** (Certified Ethical Hacker)

### Advanced Level
- **Offensive Security OSCP** (Certified Professional)
- **GIAC GPEN** (Penetration Tester)
- **EC-Council LPT** (Licensed Penetration Tester)

### Expert Level
- **Offensive Security OSEE** (Expert Exploitation)
- **GIAC GXPN** (Exploit Researcher and Advanced Penetration Tester)
- **Crest CRT** (Registered Tester)

## Training Resources

### Online Platforms
- **Hack The Box** - https://www.hackthebox.eu/
- **TryHackMe** - https://tryhackme.com/
- **VulnHub** - https://www.vulnhub.com/
- **OverTheWire** - https://overthewire.org/wargames/

### Books
- "The Web Application Hacker's Handbook" by Dafydd Stuttard
- "Penetration Testing: A Hands-On Introduction to Hacking" by Georgia Weidman
- "Metasploit: The Penetration Tester's Guide" by David Kennedy
- "Nmap Network Scanning" by Gordon Fyodor Lyon

### Video Training
- **Cybrary** - https://www.cybrary.it/
- **Pluralsight** - Security courses
- **Udemy** - Penetration testing courses
- **SANS** - Professional training courses

## Community Resources

### Forums and Communities
- **Reddit r/netsec** - https://reddit.com/r/netsec
- **Information Security Stack Exchange** - https://security.stackexchange.com/
- **Null Byte** - https://null-byte.wonderhowto.com/
- **Exploit Development** - Various Discord/Slack communities

### Conferences
- **DEF CON** - Las Vegas, Nevada
- **Black Hat** - Multiple locations
- **BSides** - Local security conferences
- **OWASP AppSec** - Application security

## Contribution Guidelines

If you'd like to contribute to these cheat sheets:

1. **Fork the repository**
2. **Create feature branch**
3. **Add valuable content**
4. **Test all commands**
5. **Submit pull request**

### Content Standards
- Accurate and tested commands
- Clear explanations
- Proper attribution
- Legal and ethical considerations
- Professional presentation
