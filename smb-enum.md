# SMB Enumeration

---

## 1. Ports & Protocols

| Port | Protocol         | Description                        |
|------|------------------|------------------------------------|
| 137  | UDP              | NetBIOS Name Service (nbtstat)     |
| 138  | UDP              | NetBIOS Datagram Service           |
| 139  | TCP              | NetBIOS Session Service (SMB over NetBIOS) |
| 445  | TCP              | SMB over TCP (Direct, modern)      |

- **SMB** (Server Message Block): File/printer sharing, inter-process communication.
- **NetBIOS**: Session layer protocol, often used with SMB for legacy support.

---

## 2. SMB Versions & Features

| Version   | OS/Support                        | Key Features/Notes                                 |
|-----------|-----------------------------------|----------------------------------------------------|
| CIFS      | Windows NT 4.0, Samba             | NetBIOS interface, legacy, insecure                |
| SMB 1.0   | Windows 2000, Samba               | Direct TCP, vulnerable, deprecated                 |
| SMB 2.0   | Vista/2008                        | Perf. upgrades, message signing, caching           |
| SMB 2.1   | Win7/2008R2                       | Locking improvements                               |
| SMB 3.0   | Win8/2012, Samba 4                | Multichannel, encryption, remote storage           |
| SMB 3.1.1 | Win10/2016                        | Integrity checks, AES-128 encryption               |

- **CIFS ≈ SMB1**; modern systems use SMB2/3.
- **Samba**: Open-source SMB/CIFS for Linux/Unix.

---

## 3. Scanning & Discovery

### Nmap

```bash
nmap -v -p 139,445 --script smb-os-discovery,smb-enum-shares,smb-enum-users <target>
nmap -sV -sC -p139,445 <target>
```

- `-p 139,445`: Scan SMB/NetBIOS ports.
- `-sC`: Default scripts (includes some SMB checks).
- `--script smb-*`: Use NSE scripts for deep SMB enumeration.

**Useful NSE Scripts:**
- `smb-os-discovery`: OS/domain info via SMB
- `smb-enum-shares`: List shares
- `smb-enum-users`: List users
- `smb-enum-domains`: List domains
- `smb-brute`: Brute-force SMB logins

### NetBIOS Name Scan

```bash
nbtscan -r <target>/24
```
- Reveals NetBIOS names, MACs, and roles.

---

## 4. Manual Enumeration Tools

### Windows Built-in

```cmd
net view \\<host> /all
```
- Lists shares (including admin shares).

### Linux Tools

#### smbclient

```bash
smbclient -N -L //<target>         # List shares (null session)
smbclient //<target>/<share>       # Connect to share
```
- `-N`: No password (null session)
- `-L`: List shares

**smbclient commands:**  
`ls`, `get`, `put`, `help`, `!<cmd>` (run local shell command)

#### rpcclient

```bash
rpcclient -U "" <target>
```
- Null session, interactive shell

| Command             | Description                        |
|---------------------|------------------------------------|
| srvinfo             | Server info                        |
| enumdomains         | List domains                       |
| querydominfo        | Domain/server/user info            |
| netshareenumall     | List all shares                    |
| netsharegetinfo <s> | Info about a specific share        |
| enumdomusers        | List domain users                  |
| queryuser <RID>     | Info about a specific user         |
| querygroup <RID>    | Info about a group                 |

#### Brute-force User RIDs

```bash
for i in $(seq 500 1100); do rpcclient -N -U "" <target> -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo ""; done
```

#### SMBMap

```bash
smbmap -H <target>
```
- Lists shares, permissions, comments.

#### CrackMapExec

```bash
crackmapexec smb <target> --shares -u '' -p ''
```
- Enumerates shares, users, OS, signing, SMBv1 status.

#### Enum4linux-ng

```bash
./enum4linux-ng.py <target> -A
```
- Comprehensive enumeration (users, shares, groups, policies, OS, etc.)

---

## 5. Samba/SMB Server Config (Linux)

**Config file:** `/etc/samba/smb.conf`

| Setting                | Description                                      |
|------------------------|--------------------------------------------------|
| [sharename]            | Name of the share                                |
| workgroup              | Workgroup/domain                                 |
| path                   | Directory path                                   |
| server string          | Server description                               |
| unix password sync     | Sync UNIX/SMB passwords                          |
| usershare allow guests | Allow guest access                               |
| map to guest           | Map bad logins to guest                          |
| browseable             | Show in share list                               |
| guest ok               | Allow anonymous access                           |
| read only              | Read-only share                                  |
| create mask            | File permissions                                 |
| directory mask         | Directory permissions                            |

**Dangerous Settings:**
- `browseable = yes`
- `read only = no`
- `writable = yes`
- `guest ok = yes`
- `create mask = 0777`
- `directory mask = 0777`

**Restart Samba:**
```bash
sudo systemctl restart smbd
```

---

## 6. Typical Default Shares

| Share    | Type | Description                  |
|----------|------|-----------------------------|
| ADMIN$   | Disk | Remote admin                |
| C$       | Disk | Default root share          |
| IPC$     | IPC  | Inter-process communication |
| NETLOGON | Disk | Logon server share          |
| SYSVOL   | Disk | Logon server share          |

---

## 7. Attack/Enumeration Flow

### Flowchart

```mermaid
flowchart TD
    A[Start: Port Scan] --> B{Are 139/445 open?}
    B -- No --> Z[Stop: Not SMB]
    B -- Yes --> C[Service/OS Detection|Nmap version scan, NSE scripts]
    C --> D[NetBIOS Name Enumeration|nbtscan, nmblookup]
    D --> E[Share Enumeration|smbclient, rpcclient, SMBMap, CrackMapExec]
    E --> F[User/Group Enumeration|rpcclient, enum4linux-ng, RID brute-force]
    F --> G[Access Shares|smbclient, download files, check permissions]
    G --> H[Check for Null Sessions|anonymous/guest access]
    H --> I[Check for Dangerous Configs|guest access, writable shares, weak permissions]
    I --> J[Document Findings & Exploit as Needed]
```

### Step-by-Step Attack/Enumeration Flow

1. **Port Scan:**  
   - Identify if TCP 139/445 are open.

2. **Service/OS Detection:**  
   - Use Nmap version scan and NSE scripts to fingerprint SMB and OS.

3. **NetBIOS Name Enumeration:**  
   - Use tools like `nbtscan` or `nmblookup` to gather NetBIOS names and workgroup/domain info.

4. **Share Enumeration:**  
   - Enumerate available shares using `smbclient`, `rpcclient`, `SMBMap`, or `CrackMapExec`.

5. **User/Group Enumeration:**  
   - Enumerate users and groups with `rpcclient`, `enum4linux-ng`, or by brute-forcing RIDs.

6. **Access Shares:**  
   - Attempt to access shares, download files, and check permissions using `smbclient` or similar tools.

7. **Check for Null Sessions:**  
   - Test for anonymous or guest access to shares and information.

8. **Check for Dangerous Configurations:**  
   - Look for guest access, writable shares, and weak permissions.

9. **Document Findings & Exploit as Needed:**  
   - Save all output, document misconfigurations, and proceed with exploitation if permitted.

---

## 8. Useful Tables

### SMB Versions

| Version   | Port(s) | Default? | Notes                        |
|-----------|---------|----------|------------------------------|
| SMB1/CIFS | 139     | Legacy   | Deprecated, insecure         |
| SMB2+     | 445     | Modern   | Secure, supports encryption  |

### Common Tools

| Tool           | Use Case                | Command Example                        |
|----------------|------------------------|----------------------------------------|
| nmap           | Port/service scan       | nmap -p139,445 -sC -sV <target>        |
| nbtscan        | NetBIOS names           | nbtscan -r <target>/24                 |
| smbclient      | List/connect shares     | smbclient -N -L //<target>             |
| rpcclient      | Deep enum, users/groups | rpcclient -U "" <target>               |
| smbmap         | Share/perm enum         | smbmap -H <target>                     |
| crackmapexec   | Enum, brute, shares     | crackmapexec smb <target> --shares     |
| enum4linux-ng  | All-in-one enum         | ./enum4linux-ng.py <target> -A         |

---

## 9. Key Points to Remember

- **SMB/NetBIOS are separate but related.**
- **Null sessions** (anonymous access) are often possible on misconfigured systems.
- **Always check for writable shares and guest access.**
- **RID brute-forcing** can reveal users even if listing is blocked.
- **Samba config** can be a goldmine for misconfigurations.
- **Automated tools** are great, but always verify manually for hidden/edge cases.

---

**Tip:**  
Always save your enumeration output for later review and evidence.  
Use `-oN`/`-oG`/`-oA` with nmap, and redirect tool output to files.

---

*This cheat sheet is designed for fast, effective SMB enumeration and exploitation in a pentest/OSCP context.*
