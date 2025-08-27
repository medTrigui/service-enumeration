# Database Services

Database services are critical targets in penetration testing due to their potential for containing sensitive information and providing pathways for privilege escalation.

## Overview

Database enumeration focuses on:

- Service discovery and version identification
- Authentication bypass and credential attacks
- Information disclosure vulnerabilities
- Privilege escalation techniques
- Data extraction methods

## Common Database Services

### MySQL (Port 3306)
Open-source relational database management system commonly found in web applications.

**Key Attack Vectors:**
- Default and weak credentials
- UDF (User Defined Function) exploitation
- File system access via SQL queries
- Binary log analysis

[View MySQL Cheat Sheet](mysql.md)

### Oracle TNS (Port 1521)
Enterprise database system with extensive functionality and complex security model.

**Key Attack Vectors:**
- SID enumeration and brute forcing
- TNS listener exploitation
- SYSDBA privilege escalation
- PL/SQL injection vulnerabilities

[View Oracle TNS Cheat Sheet](oracle-tns.md)

## General Methodology

### 1. Discovery
```bash
# Port scanning
nmap -p 1433,1521,3306,5432 target

# Service detection
nmap -sV -p <ports> target
```

### 2. Enumeration
```bash
# Banner grabbing
nc target port

# Specialized tools
nmap --script=database-* target
```

### 3. Authentication
```bash
# Default credentials
hydra -C defaults.txt service://target

# Brute force attacks
hydra -L users.txt -P passwords.txt service://target
```

### 4. Exploitation
- Leverage identified vulnerabilities
- Execute commands through database functions
- Extract sensitive information
- Escalate privileges

## Security Considerations

When testing database services:

- **Authorization** - Ensure proper testing authorization
- **Data sensitivity** - Be cautious with production data
- **Service availability** - Avoid disrupting database operations
- **Logging** - Be aware of extensive database logging

## Tools and Resources

### Essential Tools
- **nmap** - Service discovery and enumeration
- **hydra** - Authentication attacks
- **sqlmap** - SQL injection testing
- **Metasploit** - Exploitation framework

### Database-Specific Tools
- **mysql** - MySQL client
- **sqlplus** - Oracle client
- **psql** - PostgreSQL client
- **sqsh** - Sybase/MSSQL client

Select a specific database service above to access detailed enumeration and exploitation techniques.
