# Complete Payload Index

## 📑 All Available Payloads by Category

### ✅ Completed Payloads (Comprehensive Collections)

#### 1. XSS (Cross-Site Scripting) - `xss/`
- ✓ Basic alert payloads
- ✓ Image tag exploits
- ✓ SVG-based XSS
- ✓ Event handler payloads
- ✓ HTML5 tags
- ✓ Remote JS loading
- ✓ Filter bypasses (encoding, case, tag breaking)
- ✓ Polyglot XSS
- ✓ WAF bypass techniques
- ✓ CSP bypass
- ✓ Angular/AngularJS XSS
- ✓ Data grabbers (cookie stealing)
- ✓ Keyloggers
- ✓ Blind XSS
- ✓ PostMessage XSS
- **Files:** `payloads.txt`, `README.md`

#### 2. SQL Injection - `sqli/`
- ✓ MySQL injection
- ✓ MSSQL injection
- ✓ PostgreSQL injection
- ✓ Oracle injection
- ✓ SQLite injection
- ✓ Union-based exploitation
- ✓ Error-based exploitation
- ✓ Boolean-based blind
- ✓ Time-based blind
- ✓ SQLmap usage guide
- ✓ WAF bypass techniques
- ✓ Polyglot payloads
- **Files:** `payloads.txt`

#### 3. Command Injection - `command-injection/`
- ✓ Basic command separators
- ✓ Command chaining
- ✓ Bypass techniques
- ✓ Blacklist filter bypass
- ✓ Space filter bypass
- ✓ Reverse shell payloads (Bash, Netcat, Python, PHP, Perl, Ruby)
- ✓ Data exfiltration
- ✓ Time-based detection
- **Files:** `payloads.txt`

#### 4. SSRF (Server-Side Request Forgery) - `ssrf/`
- ✓ Localhost variations
- ✓ AWS metadata endpoints
- ✓ Google Cloud metadata
- ✓ Azure metadata
- ✓ Digital Ocean metadata
- ✓ Bypass techniques (encoding, IPv6, DNS rebinding)
- ✓ Protocol exploitation (Gopher, Dict, File)
- ✓ Port scanning
- ✓ Advanced exploitation (Redis, Memcache, SMTP)
- **Files:** `payloads.txt`

#### 5. File Inclusion (LFI/RFI) - `file-inclusion/`
- ✓ Linux LFI targets
- ✓ Windows LFI targets
- ✓ Path traversal
- ✓ Bypass techniques (null byte, encoding)
- ✓ PHP wrappers (php://, data://, expect://, phar://, zip://)
- ✓ LFI to RCE (log poisoning, /proc/self/environ, session files)
- ✓ RFI payloads
- ✓ Interesting file lists
- **Files:** `payloads.txt`

#### 6. XXE (XML External Entity) - `xxe/`
- ✓ Basic XXE payloads
- ✓ Blind XXE (OOB, error-based)
- ✓ SSRF via XXE
- ✓ DOS attacks (Billion Laughs)
- ✓ Protocol exploitation
- ✓ SOAP/SVG XXE
- ✓ Windows file access
- ✓ XInclude attacks
- ✓ Office document XXE
- ✓ Bypass techniques
- **Files:** `payloads.txt`

#### 7. SSTI (Server-Side Template Injection) - `ssti/`
- ✓ Detection payloads
- ✓ Jinja2 (Python/Flask)
- ✓ Tornado (Python)
- ✓ Twig (PHP/Symfony)
- ✓ Freemarker (Java)
- ✓ Velocity (Java)
- ✓ Smarty (PHP)
- ✓ Mako (Python)
- ✓ Pug/Jade (NodeJS)
- ✓ ERB (Ruby)
- ✓ Handlebars (JavaScript)
- ✓ Razor (ASP.NET)
- ✓ Bypass techniques
- **Files:** `payloads.txt`

#### 8. Directory Traversal - `directory-traversal/`
- ✓ Basic traversal patterns
- ✓ Linux targets
- ✓ Windows targets
- ✓ URL encoding
- ✓ Double URL encoding
- ✓ Unicode encoding
- ✓ Mixed slashes
- ✓ Null byte injection
- ✓ Filter bypasses
- ✓ Interesting file lists
- **Files:** `payloads.txt`

#### 9. JWT Vulnerabilities - `jwt/`
- ✓ Algorithm confusion (RS256→HS256)
- ✓ None algorithm bypass
- ✓ Weak secret brute force
- ✓ KID header injection
- ✓ JKU header injection
- ✓ JWK header injection
- ✓ Payload manipulation
- ✓ Timestamp bypass
- ✓ Tools and commands
- ✓ Testing checklist
- **Files:** `payloads.txt`

#### 10. NoSQL Injection - `nosqli/`
- ✓ MongoDB authentication bypass
- ✓ Boolean-based injection
- ✓ MongoDB operators
- ✓ Blind NoSQL injection
- ✓ JavaScript injection ($where)
- ✓ JSON API payloads
- ✓ CouchDB injection
- ✓ Cassandra injection
- ✓ Bypass techniques
- ✓ Tools and testing
- **Files:** `payloads.txt`

#### 11. CSRF (Cross-Site Request Forgery) - `csrf/`
- ✓ Basic CSRF (GET/POST)
- ✓ JSON CSRF
- ✓ CSRF token bypass
- ✓ SameSite cookie bypass
- ✓ Clickjacking + CSRF
- ✓ Advanced techniques
- ✓ Content-Type tricks
- ✓ Exploitation examples
- ✓ Testing checklist
- **Files:** `payloads.txt`

### 📋 Additional Vulnerabilities (Directories Created)

The following vulnerability categories have directories created and are ready for payload addition:

- `cors/` - CORS Misconfiguration
- `deserialization/` - Insecure Deserialization
- `idor/` - Insecure Direct Object Reference
- `graphql/` - GraphQL Injection
- `ldap-injection/` - LDAP Injection
- `xpath-injection/` - XPATH Injection
- `crlf-injection/` - CRLF Injection
- `open-redirect/` - Open Redirect
- `race-condition/` - Race Condition
- `prototype-pollution/` - Prototype Pollution
- `mass-assignment/` - Mass Assignment
- `type-juggling/` - Type Juggling
- `clickjacking/` - Clickjacking
- `business-logic/` - Business Logic Errors
- `brute-force/` - Brute Force & Rate Limit
- `mfa-bypass/` - MFA Bypass
- `oauth/` - OAuth Misconfiguration
- `ssi/` - Server Side Include Injection
- `csv-injection/` - CSV Injection
- `xslt/` - XSLT Injection
- `tabnabbing/` - Tabnabbing
- `web-cache-deception/` - Web Cache Deception
- `dom-clobbering/` - DOM Clobbering
- `xs-leak/` - XS-Leak
- `dns-rebinding/` - DNS Rebinding
- `request-smuggling/` - Request Smuggling
- `hpp/` - HTTP Parameter Pollution
- `zip-slip/` - Zip Slip
- `account-takeover/` - Account Takeover
- `file-upload/` - Insecure File Upload

## 🔍 How to Use This Collection

### 1. Navigate to Specific Vulnerability
```bash
cd xss/
cat payloads.txt
```

### 2. Search for Specific Technique
```bash
grep -r "bypass" .
grep -r "RCE" .
```

### 3. Quick Reference
```bash
cat QUICK_REFERENCE.md
```

## 📊 Payload Statistics

- **Total Vulnerability Categories:** 40+
- **Completed Comprehensive Payloads:** 11
- **Total Payload Files:** 11+
- **Lines of Payloads:** 2000+
- **Coverage:** Critical web vulnerabilities

## 🎯 Recommended Testing Order

1. **Start with Detection**
   - XSS: `<script>alert(1)</script>`
   - SQLi: `' OR '1'='1`
   - Command: `; whoami`

2. **Test for Bypass**
   - Try encoding variations
   - Test different contexts
   - Use filter bypass techniques

3. **Escalate to Exploitation**
   - Data exfiltration
   - Remote code execution
   - Privilege escalation

## 🛠️ Integration with Tools

### Burp Suite
```
1. Load payloads into Intruder
2. Use as active scan insertion points
3. Manual testing in Repeater
```

### OWASP ZAP
```
1. Import as fuzzing payloads
2. Custom active scan rules
3. Manual request modification
```

### Command Line
```bash
# Use with curl
curl "http://target.com/page?id=$(cat sqli/payloads.txt)"

# Use with ffuf
ffuf -w xss/payloads.txt -u http://target.com/FUZZ
```

## 📚 Additional Resources

- `README.md` - Main documentation
- `QUICK_REFERENCE.md` - Quick testing guide
- `PAYLOAD_INDEX.md` - This file
- Individual `/README.md` files in each category

---

**Last Updated:** 2026-06-16  
**Source:** PayloadsAllTheThings  
**Maintained for:** Educational and authorized security testing
