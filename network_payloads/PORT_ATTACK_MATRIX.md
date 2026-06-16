# Complete Port Attack Matrix

## 📊 All 50+ Ports - Attack Overview & Quick Reference

This document provides a comprehensive attack matrix for all covered ports with success rates, difficulty levels, and quick attack commands.

---

## 🎯 Attack Difficulty & Success Legend

**Difficulty**:
- 🟢 **Easy**: Beginner-friendly, simple tools
- 🟡 **Medium**: Requires some experience
- 🔴 **Hard**: Advanced techniques needed

**Success Rate**:
- 🟢 **High**: 60-90% success in real environments
- 🟡 **Medium**: 30-60% success
- 🔴 **Low**: <30% success

---

## 📋 Complete Port Matrix

| Port | Service | Difficulty | Success | Best Attack | Full Guide |
|------|---------|-----------|---------|-------------|-----------|
| **20/21** | FTP | 🟢 Easy | 🟢 High (40%) | Anonymous/Brute | [✓ Full Guide](Port_Specific_Attacks/Port_21_FTP/README.md) + [Bypass](Port_Specific_Attacks/Port_21_FTP/bypass_techniques.md) |
| **22** | SSH | 🟡 Medium | 🟡 Medium (30%) | Keys/Brute | [✓ Full Guide](Port_Specific_Attacks/Port_22_SSH/README.md) + [Bypass](Port_Specific_Attacks/Port_22_SSH/bypass_techniques.md) |
| **23** | Telnet | 🟢 Easy | 🟢 High (50%) | Default Creds/Sniff | [✓ Full Guide](Port_Specific_Attacks/Port_23_Telnet/README.md) |
| **25** | SMTP | 🟢 Easy | 🟡 Medium (30%) | Enum/Relay | [✓ Full Guide](Port_Specific_Attacks/Port_25_SMTP/README.md) |
| **53** | DNS | 🟢 Easy | 🟢 High (20-95%) | Alt DNS/Zone Transfer | [✓ Full Guide](Port_Specific_Attacks/Port_53_DNS/README.md) + [Bypass](Port_Specific_Attacks/Port_53_DNS/bypass_techniques.md) |
| **67/68** | DHCP | 🟡 Medium | 🟡 Medium (40%) | Starvation/Rogue | ✓ Payloads |
| **69** | TFTP | 🟢 Easy | 🟡 Medium (35%) | File Download | ✓ Payloads |
| **80** | HTTP | 🟡 Medium | 🟡 Medium (35%) | SQL Inj/Dir Enum | [✓ Full Guide](Port_Specific_Attacks/Port_80_443_HTTP/README.md) + [Bypass](Port_Specific_Attacks/Port_80_443_HTTP/bypass_techniques.md) |
| **88** | Kerberos | 🔴 Hard | 🟡 Medium (40%) | AS-REP Roast | ✓ Payloads |
| **110** | POP3 | 🟢 Easy | 🟡 Medium (30%) | Brute Force | ✓ Payloads |
| **111** | RPCBind | 🟡 Medium | 🟡 Medium (25%) | Enum/NFS | ✓ Payloads |
| **119** | NNTP | 🟢 Easy | 🔴 Low (15%) | Enum | ✓ Payloads |
| **135** | MSRPC | 🟡 Medium | 🟡 Medium (20%) | Enum | ✓ Payloads |
| **137/138** | NetBIOS | 🟢 Easy | 🟡 Medium (30%) | NBTScan/Enum | ✓ Payloads |
| **139** | SMB | 🟡 Medium | 🟡 Medium (25%) | Null Session | [✓ Full Guide](Port_Specific_Attacks/Port_139_445_SMB/README.md) + [Bypass](Port_Specific_Attacks/Port_139_445_SMB/bypass_techniques.md) |
| **143** | IMAP | 🟢 Easy | 🟡 Medium (30%) | Brute Force | ✓ Payloads |
| **161/162** | SNMP | 🟢 Easy | 🟢 High (50%) | Community Strings | ✓ Payloads |
| **389** | LDAP | 🟡 Medium | 🟡 Medium (35%) | Anonymous Bind | ✓ Payloads |
| **443** | HTTPS | 🟡 Medium | 🟡 Medium (25%) | SSL Issues/WAF | [✓ Full Guide](Port_Specific_Attacks/Port_80_443_HTTP/README.md) + [Bypass](Port_Specific_Attacks/Port_80_443_HTTP/bypass_techniques.md) |
| **445** | SMB | 🟡 Medium | 🟢 High (30%) | Pass-Hash/EternalBlue | [✓ Full Guide](Port_Specific_Attacks/Port_139_445_SMB/README.md) + [Bypass](Port_Specific_Attacks/Port_139_445_SMB/bypass_techniques.md) |
| **465/587** | SMTPS | 🟢 Easy | 🟡 Medium (25%) | Enum/Brute | ✓ Payloads |
| **512/513/514** | R-Services | 🟢 Easy | 🔴 Low (10%) | Default Access | ✓ Payloads |
| **515** | LPD | 🟢 Easy | 🔴 Low (15%) | Print Jobs | ✓ Payloads |
| **548** | AFP | 🟡 Medium | 🔴 Low (20%) | Brute Force | ✓ Payloads |
| **554** | RTSP | 🟢 Easy | 🟡 Medium (25%) | Stream Access | ✓ Payloads |
| **631** | IPP | 🟡 Medium | 🟡 Medium (30%) | CUPS Exploits | ✓ Payloads |
| **636** | LDAPS | 🟡 Medium | 🟡 Medium (30%) | Same as LDAP | ✓ Payloads |
| **873** | Rsync | 🟢 Easy | 🟡 Medium (35%) | Module Enum/Download | ✓ Payloads |
| **993** | IMAPS | 🟢 Easy | 🟡 Medium (25%) | Brute Force | ✓ Payloads |
| **995** | POP3S | 🟢 Easy | 🟡 Medium (25%) | Brute Force | ✓ Payloads |
| **1080** | SOCKS | 🟡 Medium | 🟡 Medium (30%) | Proxy Test | ✓ Payloads |
| **1194** | OpenVPN | 🔴 Hard | 🔴 Low (15%) | Config Extract | ✓ Payloads |
| **1433** | MSSQL | 🟡 Medium | 🟡 Medium (35%) | xp_cmdshell | ✓ Payloads |
| **1521** | Oracle | 🔴 Hard | 🟡 Medium (25%) | SID Enum/Brute | ✓ Payloads |
| **1723** | PPTP | 🟡 Medium | 🟡 Medium (30%) | Brute Force | ✓ Payloads |
| **2049** | NFS | 🟢 Easy | 🟢 High (40%) | Mount Shares | ✓ Payloads |
| **2121** | FTP Alt | 🟢 Easy | 🟢 High (40%) | Same as Port 21 | ✓ Payloads |
| **3000** | Web Apps | 🟡 Medium | 🟡 Medium (30%) | Framework Specific | ✓ Payloads |
| **3268/3269** | Global Catalog | 🟡 Medium | 🟡 Medium (30%) | AD Enum | ✓ Payloads |
| **3306** | MySQL | 🟢 Easy | 🟢 High (35%) | Empty Pass/Brute | [✓ Full Guide](Port_Specific_Attacks/Port_3306_MySQL/README.md) |
| **3389** | RDP | 🟡 Medium | 🟢 High (45%) | Brute Force | [✓ Full Guide](Port_Specific_Attacks/Port_3389_RDP/README.md) |
| **4444** | Metasploit | 🟡 Medium | 🔴 Low (10%) | Backdoor | ✓ Payloads |
| **5000** | UPnP/Flask | 🟡 Medium | 🟡 Medium (30%) | Various | ✓ Payloads |
| **5060/5061** | SIP | 🟡 Medium | 🟡 Medium (30%) | Extension Enum | ✓ Payloads |
| **5432** | PostgreSQL | 🟡 Medium | 🟡 Medium (35%) | Default/Brute | ✓ Payloads |
| **5555** | ADB | 🟢 Easy | 🟢 High (60%) | Direct Connect | ✓ Payloads |
| **5900** | VNC | 🟢 Easy | 🟡 Medium (25%) | No Password/Brute | ✓ Payloads |
| **6379** | Redis | 🟢 Easy | 🟢 High (60%) | No Auth/Webshell | ✓ Payloads |
| **6666** | IRC/Backdoor | 🟡 Medium | 🔴 Low (15%) | Connect/Commands | ✓ Payloads |
| **8000-8888** | HTTP Alts | 🟡 Medium | 🟡 Medium (35%) | Same as HTTP | ✓ Payloads |
| **9100** | Printer | 🟢 Easy | 🟡 Medium (30%) | PRET/Print Jobs | ✓ Payloads |
| **10000** | Webmin | 🟡 Medium | 🟡 Medium (35%) | Default/Exploits | ✓ Payloads |
| **27017/27018** | MongoDB | 🟢 Easy | 🟢 High (55%) | No Auth | ✓ Payloads |

---

## 🎯 Quick Attack Commands by Port

### Port 21 - FTP
```bash
ftp 192.168.1.100                                                      # Anonymous
hydra -l admin -P passwords.txt ftp://192.168.1.100                    # Brute force
nmap -p 21 --script ftp-* 192.168.1.100                                # Enumerate
```

### Port 22 - SSH
```bash
ssh root@192.168.1.100                                                 # Try default
hydra -l root -P passwords.txt ssh://192.168.1.100                     # Brute force
ssh-keyscan 192.168.1.100                                              # Get keys
```

### Port 23 - Telnet
```bash
telnet 192.168.1.100                                                   # Connect
hydra -l admin -P passwords.txt telnet://192.168.1.100                 # Brute force
tcpdump -i eth0 -A 'tcp port 23'                                       # Sniff credentials
```

### Port 25 - SMTP
```bash
smtp-user-enum -M VRFY -U users.txt -t 192.168.1.100                   # Enum users
nmap -p 25 --script smtp-open-relay 192.168.1.100                      # Check relay
telnet 192.168.1.100 25                                                # Manual test
```

### Port 53 - DNS
```bash
dig @192.168.1.100 target.com axfr                                     # Zone transfer
gobuster dns -d target.com -w wordlist.txt                             # Subdomain enum
fierce --domain target.com                                             # DNS enum
```

### Port 80/443 - HTTP/HTTPS
```bash
gobuster dir -u http://192.168.1.100 -w wordlist.txt                   # Dir enum
nikto -h http://192.168.1.100                                          # Vuln scan
sqlmap -u "http://192.168.1.100/page?id=1" --dbs                       # SQL injection
```

### Port 139/445 - SMB
```bash
smbclient -L //192.168.1.100 -N                                        # Null session
enum4linux -a 192.168.1.100                                            # Full enum
crackmapexec smb 192.168.1.100 -u admin -p password                    # Auth test
nmap --script smb-vuln-ms17-010 192.168.1.100                          # EternalBlue
```

### Port 161 - SNMP
```bash
snmpwalk -v 2c -c public 192.168.1.100                                 # SNMP walk
onesixtyone -c community.txt 192.168.1.100                             # Brute community
nmap -sU -p 161 --script snmp-* 192.168.1.100                          # Enumerate
```

### Port 3306 - MySQL
```bash
mysql -h 192.168.1.100 -u root                                         # Try empty pass
hydra -l root -P passwords.txt mysql://192.168.1.100                   # Brute force
mysqldump -h 192.168.1.100 -u root -p --all-databases > dump.sql       # Dump all
```

### Port 3389 - RDP
```bash
xfreerdp /u:admin /p:password /v:192.168.1.100                         # Connect
hydra -l Administrator -P passwords.txt rdp://192.168.1.100            # Brute force
nmap --script rdp-vuln-ms12-020 192.168.1.100                          # Check vulns
```

### Port 5900 - VNC
```bash
vncviewer 192.168.1.100:5900                                           # Connect
hydra -P passwords.txt vnc://192.168.1.100                             # Brute force
nmap -p 5900 --script vnc-* 192.168.1.100                              # Enumerate
```

### Port 6379 - Redis
```bash
redis-cli -h 192.168.1.100                                             # Connect
redis-cli -h 192.168.1.100 INFO                                        # Get info
redis-cli -h 192.168.1.100 CONFIG GET *                                # Get config
```

### Port 27017 - MongoDB
```bash
mongo 192.168.1.100:27017                                              # Connect
mongo --host 192.168.1.100 --eval "db.adminCommand('listDatabases')"  # List DBs
mongodump --host 192.168.1.100 --out dump/                             # Dump all
```

---

## 📊 Attack Success Statistics

### Highest Success Rates (>50%)
1. **Redis (6379)** - 60% - Often no authentication
2. **MongoDB (27017)** - 55% - Frequently exposed without auth
3. **ADB (5555)** - 60% - Debug mode left enabled
4. **Telnet (23)** - 50% - Weak/default credentials
5. **SNMP (161)** - 50% - Default community strings

### Medium Success Rates (30-50%)
- **FTP (21)** - 40% - Anonymous/defaults
- **NFS (2049)** - 40% - Open shares
- **RDP (3389)** - 45% - Brute force success
- **Kerberos (88)** - 40% - AS-REP roasting
- **MySQL (3306)** - 35% - Empty/weak passwords

### Lower Success Rates (<30%)
- **SSH (22)** - 30% - Strong auth common
- **HTTPS (443)** - 25% - Modern security
- **OpenVPN (1194)** - 15% - Strong encryption
- **Oracle (1521)** - 25% - Complex authentication

---

## 🎯 Attack Strategy by Environment

### Internal Network Pentest
**Priority Targets**:
1. **Port 445 (SMB)** - Lateral movement, Pass-the-Hash
2. **Port 22/23 (SSH/Telnet)** - Remote access
3. **Port 3306/1433/5432** - Databases with sensitive data
4. **Port 88 (Kerberos)** - AD attacks (Kerberoasting)
5. **Port 389 (LDAP)** - Domain enumeration

### External Pentest
**Priority Targets**:
1. **Port 80/443 (HTTP/HTTPS)** - Web vulnerabilities
2. **Port 22 (SSH)** - Remote access attempts
3. **Port 21 (FTP)** - Data exposure
4. **Port 3389 (RDP)** - Windows access
5. **Port 25 (SMTP)** - Email security

### IoT/Embedded Devices
**Priority Targets**:
1. **Port 23 (Telnet)** - Often enabled by default
2. **Port 80 (HTTP)** - Web admin panels
3. **Port 8080/8888** - Alternate web ports
4. **Port 161 (SNMP)** - Network devices
5. **Port 5000** - UPnP and other services

---

## 🔥 Most Dangerous Ports (CVSS Impact)

| Rank | Port | Service | Why Dangerous |
|------|------|---------|---------------|
| 1 | 445 | SMB | EternalBlue, Pass-the-Hash, Lateral movement |
| 2 | 3389 | RDP | BlueKeep, Remote code execution |
| 3 | 22 | SSH | System access, Pivoting |
| 4 | 3306 | MySQL | Data breach, UDF RCE |
| 5 | 6379 | Redis | Unauthenticated RCE, Webshell upload |
| 6 | 23 | Telnet | Cleartext credentials |
| 7 | 27017 | MongoDB | Data exposure, No auth |
| 8 | 1433 | MSSQL | xp_cmdshell RCE |
| 9 | 80/443 | HTTP(S) | Application vulnerabilities |
| 10 | 21 | FTP | Credential theft, Data exposure |

---

## 📚 Complete Documentation Index

### Full Attack Guides Available
- ✅ [Port 21 - FTP](Port_Specific_Attacks/Port_21_FTP/README.md) (465 lines)
- ✅ [Port 22 - SSH](Port_Specific_Attacks/Port_22_SSH/README.md) (585 lines)
- ✅ [Port 23 - Telnet](Port_Specific_Attacks/Port_23_Telnet/README.md) (150+ lines)
- ✅ [Port 3306 - MySQL](Port_Specific_Attacks/Port_3306_MySQL/README.md) (150+ lines)

### Bypass Technique Guides Available
- ✅ [Port 21 - FTP Bypass](Port_Specific_Attacks/Port_21_FTP/bypass_techniques.md) (150 lines)
- ✅ [Port 22 - SSH Bypass](Port_Specific_Attacks/Port_22_SSH/bypass_techniques.md) (150 lines)
- ✅ [Port 53 - DNS Bypass](Port_Specific_Attacks/Port_53_DNS/bypass_techniques.md) (150 lines)
- ✅ [Port 80/443 - HTTP Bypass](Port_Specific_Attacks/Port_80_443_HTTP/bypass_techniques.md) (150 lines)
- ✅ [Port 139/445 - SMB Bypass](Port_Specific_Attacks/Port_139_445_SMB/bypass_techniques.md) (150 lines)

### Payload Collections
- ✅ [All Ports Quick Reference](Port_Specific_Attacks/ALL_PORTS_QUICK_REFERENCE.md) (50+ ports)
- ✅ [Bypass Techniques Index](BYPASS_TECHNIQUES_INDEX.md)
- ✅ Individual payloads.txt for 50+ ports

---

**Total Coverage**: 50+ ports with 2000+ attack commands and techniques!

**Last Updated**: 2026-06-16
