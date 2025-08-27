# Service Enumeration Cheat Sheets

[![Version](https://img.shields.io/badge/version-2.0.0-blue.svg)](https://github.com/yourusername/service-enumeration/releases)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Contributions](https://img.shields.io/badge/contributions-welcome-orange.svg)](CONTRIBUTING.md)

Professional penetration testing reference guides for systematic service discovery and exploitation.

## Available Cheat Sheets

| Service | Port(s) | Description |
|---------|---------|-------------|
| [MySQL](mysql.md) | 3306 | Database enumeration, credential attacks, file operations |
| [Oracle TNS](oracle-tns.md) | 1521 | SID enumeration, SYSDBA escalation, ODAT framework |
| [SMTP](smtp.md) | 25, 465, 587 | User enumeration, open relay testing, authentication bypass |
| [IMAP/POP3](imap-pop3.md) | 143, 993, 110, 995 | Email access, credential attacks, SSL testing |
| [SNMP](snmp.md) | 161, 162 | Community strings, MIB walking, Windows enumeration |
| [SMB](smb.md) | 445, 139 | Share enumeration, null sessions, RID cycling |
| [NFS](nfs.md) | 2049, 111 | Mount operations, UID manipulation, privilege escalation |

## Usage

Each cheat sheet follows a consistent methodology:

1. **Discovery** - Port scanning and service detection
2. **Enumeration** - Information gathering and reconnaissance
3. **Authentication** - Credential attacks and testing
4. **Exploitation** - Attack execution and access gaining
5. **Post-Exploitation** - Persistence and lateral movement

## Quick Examples

```bash
# MySQL enumeration
nmap -p 3306 --script=mysql-* target
hydra -L users.txt -P passwords.txt mysql://target

# SMTP user enumeration
smtp-user-enum -M VRFY -U users.txt -t target

# SMB enumeration
smbclient -L -N //target
netexec smb target --shares -u guest -p ''

# SNMP enumeration
onesixtyone -c community.txt target
snmpwalk -c public -v1 target
```

## Installation

```bash
git clone https://github.com/yourusername/service-enumeration.git
cd service-enumeration
```

## Contributing

Contributions are welcome. Please submit pull requests with:
- New exploitation techniques
- Updated tool commands
- Additional service coverage
- Bug fixes and improvements

## License

MIT License - See [LICENSE](LICENSE) for details.

## Disclaimer

These resources are for educational and authorized testing purposes only. Ensure proper authorization before testing any systems.