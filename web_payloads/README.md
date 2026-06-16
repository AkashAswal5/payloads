# Web Vulnerability Payloads Collection

> **Comprehensive collection of web application security payloads and bypass techniques**

This repository contains an extensive collection of payloads for various web vulnerabilities, organized by category. All payloads are sourced from [PayloadsAllTheThings](https://swisskyrepo.github.io/PayloadsAllTheThings/).

## ⚠️ Disclaimer

**FOR EDUCATIONAL AND AUTHORIZED SECURITY TESTING ONLY**

These payloads are provided for:
- Learning web application security
- Authorized penetration testing
- Security research
- Developing secure applications

**NEVER** use these payloads against systems you don't own or have explicit written permission to test.

## 📚 Vulnerability Categories

### Injection Vulnerabilities
- [XSS (Cross-Site Scripting)](./xss/README.md)
  - Reflected XSS
  - Stored XSS
  - DOM-based XSS
  - Filter Bypass Techniques
  - Polyglots
  - WAF Bypass
  - CSP Bypass
- [SQL Injection](./sqli/README.md)
- [NoSQL Injection](./nosqli/README.md)
- [Command Injection](./command-injection/README.md)
- [LDAP Injection](./ldap-injection/README.md)
- [XPATH Injection](./xpath-injection/README.md)
- [XXE (XML External Entity)](./xxe/README.md)
- [SSTI (Server-Side Template Injection)](./ssti/README.md)
- [CRLF Injection](./crlf-injection/README.md)
- [CSV Injection](./csv-injection/README.md)

### File & Upload Vulnerabilities
- [File Inclusion (LFI/RFI)](./file-inclusion/README.md)
- [Insecure File Upload](./file-upload/README.md)
- [Directory Traversal](./directory-traversal/README.md)
- [Zip Slip](./zip-slip/README.md)

### Authentication & Session
- [Account Takeover](./account-takeover/README.md)
- [MFA Bypass](./mfa-bypass/README.md)
- [JWT Vulnerabilities](./jwt/README.md)
- [OAuth Misconfiguration](./oauth/README.md)
- [CSRF (Cross-Site Request Forgery)](./csrf/README.md)
- [Brute Force & Rate Limit Bypass](./brute-force/README.md)

### Server-Side Vulnerabilities
- [SSRF (Server-Side Request Forgery)](./ssrf/README.md)
- [Insecure Deserialization](./deserialization/README.md)
- [SSI (Server Side Include Injection)](./ssi/README.md)

### Access Control
- [IDOR (Insecure Direct Object Reference)](./idor/README.md)
- [CORS Misconfiguration](./cors/README.md)
- [Open Redirect](./open-redirect/README.md)

### Logic & Design Flaws
- [Business Logic Errors](./business-logic/README.md)
- [Race Condition](./race-condition/README.md)
- [Mass Assignment](./mass-assignment/README.md)
- [Type Juggling](./type-juggling/README.md)
- [Prototype Pollution](./prototype-pollution/README.md)

### API & GraphQL
- [GraphQL Injection](./graphql/README.md)
- [HTTP Parameter Pollution](./hpp/README.md)
- [Request Smuggling](./request-smuggling/README.md)

### Miscellaneous
- [Clickjacking](./clickjacking/README.md)
- [Tabnabbing](./tabnabbing/README.md)
- [Web Cache Deception](./web-cache-deception/README.md)
- [DOM Clobbering](./dom-clobbering/README.md)
- [XS-Leak](./xs-leak/README.md)
- [DNS Rebinding](./dns-rebinding/README.md)

## 🚀 Quick Start

Each vulnerability category has its own directory with:
- `README.md` - Detailed explanation and methodology
- `payloads.txt` - Ready-to-use payloads
- `bypass-techniques.md` - Filter and WAF bypass methods
- `examples/` - Real-world examples and scenarios

## 📖 Usage Example

```bash
# Navigate to a specific vulnerability
cd xss/

# View payloads
cat payloads.txt

# View bypass techniques
cat bypass-techniques.md
```

## 🛡️ Testing Framework

Recommended tools for testing:
- Burp Suite
- OWASP ZAP
- Nuclei
- SQLmap
- XSStrike
- Commix

## 📝 Contributing

Feel free to contribute additional payloads, bypass techniques, or corrections.

## 📚 Resources

- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [HackTricks](https://book.hacktricks.xyz/)

## 📄 License

Educational purposes only. Use responsibly and ethically.

---

**Last Updated:** 2026-06-16
**Source:** PayloadsAllTheThings by SwissKyRepo
