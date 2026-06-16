# Network Security Tools Reference Guide

## Installation and Usage Guide for Essential Network Security Tools

---

## Network Scanning & Reconnaissance

### Nmap - Network Mapper
**Description**: The industry standard for network discovery and security auditing

**Installation**:
```bash
# Debian/Ubuntu
sudo apt-get install nmap

# Red Hat/CentOS
sudo yum install nmap

# macOS
brew install nmap

# Windows
# Download from https://nmap.org/download.html
```

**Basic Usage**:
```bash
nmap -sS -sV -O target.com                    # Stealth SYN scan with version detection
nmap -p- --min-rate 10000 target.com          # Fast full port scan
nmap --script vuln target.com                 # Vulnerability scanning
```

**Advanced Features**:
- NSE (Nmap Scripting Engine) with 600+ scripts
- OS detection and version detection
- Service enumeration
- Firewall/IDS evasion techniques

---

### Masscan - Ultra-fast Port Scanner
**Description**: The fastest Internet port scanner, can scan entire Internet in under 6 minutes

**Installation**:
```bash
# From source
git clone https://github.com/robertdavidgraham/masscan
cd masscan
make
sudo make install

# Debian/Ubuntu
sudo apt-get install masscan
```

**Basic Usage**:
```bash
masscan 0.0.0.0/0 -p80,443 --rate 100000      # Scan Internet for web servers
masscan 192.168.1.0/24 -p0-65535 --rate 10000 # Fast local scan
```

---

### Wireshark / tshark - Network Protocol Analyzer
**Description**: World's most popular network protocol analyzer

**Installation**:
```bash
# Debian/Ubuntu
sudo apt-get install wireshark tshark

# macOS
brew install wireshark

# Windows
# Download from https://www.wireshark.org/download.html
```

**tshark Usage**:
```bash
tshark -i eth0 -w capture.pcap                # Capture to file
tshark -r capture.pcap -Y "http"              # Filter HTTP traffic
tshark -r capture.pcap -Y "dns.qry.name"      # DNS queries
```

---

## Password & Authentication Attacks

### Hydra - Fast Network Logon Cracker
**Description**: Supports numerous protocols for online password cracking

**Installation**:
```bash
# Debian/Ubuntu
sudo apt-get install hydra

# macOS
brew install hydra

# From source
git clone https://github.com/vanhauser-thc/thc-hydra
cd thc-hydra
./configure && make && sudo make install
```

**Basic Usage**:
```bash
hydra -L users.txt -P passwords.txt ssh://192.168.1.100
hydra -l admin -P passwords.txt ftp://192.168.1.100
hydra -L users.txt -P passwords.txt http-post-form://target.com/login:"user=^USER^&pass=^PASS^:F=incorrect"
```

**Supported Protocols**: SSH, FTP, HTTP(S), SMB, MySQL, PostgreSQL, RDP, VNC, SMTP, POP3, IMAP, and 50+ more

---

### John the Ripper - Password Cracker
**Description**: Fast password cracker supporting hundreds of hash types

**Installation**:
```bash
# Debian/Ubuntu
sudo apt-get install john

# From source (Jumbo version)
git clone https://github.com/openwall/john
cd john/src
./configure && make
```

**Basic Usage**:
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --show hashes.txt                        # Show cracked passwords
john --format=Raw-MD5 hashes.txt              # Specific hash format
```

---

### Hashcat - Advanced Password Recovery
**Description**: World's fastest password cracker, GPU-accelerated

**Installation**:
```bash
# Debian/Ubuntu
sudo apt-get install hashcat

# macOS
brew install hashcat

# Windows
# Download from https://hashcat.net/hashcat/
```

**Basic Usage**:
```bash
hashcat -m 0 hashes.txt wordlist.txt          # MD5 hashes
hashcat -m 1000 hashes.txt wordlist.txt       # NTLM hashes
hashcat -m 22000 wifi.hc22000 wordlist.txt    # WPA/WPA2
hashcat -a 3 -m 0 hashes.txt ?d?d?d?d?d?d     # Mask attack (6 digits)
```

---

## Man-in-the-Middle Tools

### Bettercap - Swiss Army Knife for Network Attacks
**Description**: Modern, powerful MITM framework

**Installation**:
```bash
# Using Go
go install github.com/bettercap/bettercap@latest

# Debian/Ubuntu (from repo)
sudo apt-get install bettercap

# Pre-built binaries
# Download from https://github.com/bettercap/bettercap/releases
```

**Basic Usage**:
```bash
sudo bettercap -iface eth0
# Interactive console commands:
# net.probe on
# net.show
# set arp.spoof.targets 192.168.1.100
# arp.spoof on
# net.sniff on
```

---

### Ettercap - Comprehensive MITM Suite
**Description**: Feature-rich MITM attack tool with GUI

**Installation**:
```bash
# Debian/Ubuntu
sudo apt-get install ettercap-graphical

# macOS
brew install ettercap
```

**Basic Usage**:
```bash
ettercap -G                                   # GUI mode
ettercap -T -i eth0 -M arp /// ///           # Text mode, full ARP poisoning
ettercap -T -i eth0 -M arp /192.168.1.100/ /192.168.1.1/  # Target specific hosts
```

---

## Wireless Attack Tools

### Aircrack-ng Suite - WiFi Security Auditing
**Description**: Complete suite for WiFi network security assessment

**Installation**:
```bash
# Debian/Ubuntu
sudo apt-get install aircrack-ng

# macOS
brew install aircrack-ng

# From source
git clone https://github.com/aircrack-ng/aircrack-ng
cd aircrack-ng
autoreconf -i
./configure
make && sudo make install
```

**Tools Included**:
- `airmon-ng` - Enable monitor mode
- `airodump-ng` - Capture packets
- `aireplay-ng` - Packet injection
- `aircrack-ng` - WEP/WPA cracking

**Basic Workflow**:
```bash
airmon-ng start wlan0
airodump-ng wlan0mon
airodump-ng -c 6 --bssid XX:XX:XX:XX:XX:XX -w capture wlan0mon
aireplay-ng --deauth 10 -a XX:XX:XX:XX:XX:XX wlan0mon
aircrack-ng -w wordlist.txt capture-01.cap
```

---

### Reaver - WPS PIN Attack Tool
**Description**: Brute force WPS PINs to recover WPA/WPA2 passphrases

**Installation**:
```bash
# Debian/Ubuntu
sudo apt-get install reaver

# From source
git clone https://github.com/t6x/reaver-wps-fork-t6x
cd reaver-wps-fork-t6x/src
./configure && make && sudo make install
```

**Basic Usage**:
```bash
wash -i wlan0mon                              # Scan for WPS
reaver -i wlan0mon -b XX:XX:XX:XX:XX:XX -vv   # Attack WPS
reaver -i wlan0mon -b XX:XX:XX:XX:XX:XX -vv -K  # Pixie dust attack
```

---

## Packet Crafting & Manipulation

### Scapy - Interactive Packet Manipulation
**Description**: Powerful Python-based interactive packet manipulation program

**Installation**:
```bash
# Using pip
pip3 install scapy

# Debian/Ubuntu
sudo apt-get install python3-scapy

# macOS
brew install scapy
```

**Basic Usage**:
```python
from scapy.all import *

# Send packet
send(IP(dst="192.168.1.100")/ICMP())

# Sniff packets
packets = sniff(count=10)

# ARP scan
ans, unans = srp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst="192.168.1.0/24"), timeout=2)
```

---

### Hping3 - Network Tool
**Description**: Command-line oriented TCP/IP packet assembler/analyzer

**Installation**:
```bash
# Debian/Ubuntu
sudo apt-get install hping3

# macOS
brew install hping

# From source
git clone https://github.com/antirez/hping
cd hping
./configure && make && sudo make install
```

**Basic Usage**:
```bash
hping3 -S -p 80 192.168.1.100                 # SYN scan
hping3 -1 192.168.1.100                       # ICMP ping
hping3 -2 -p 53 192.168.1.100                 # UDP scan
hping3 --flood -S -p 80 192.168.1.100         # SYN flood
```

---

## Enumeration Tools

### Enum4linux - SMB Enumeration
**Description**: Tool for enumerating data from Windows and Samba hosts

**Installation**:
```bash
# Debian/Ubuntu
sudo apt-get install enum4linux

# Manual download
wget https://github.com/CiscoCXSecurity/enum4linux/raw/master/enum4linux.pl
chmod +x enum4linux.pl
```

**Basic Usage**:
```bash
enum4linux -a 192.168.1.100                   # All enumeration
enum4linux -U 192.168.1.100                   # User enumeration
enum4linux -S 192.168.1.100                   # Share enumeration
```

---

## Web Tools for Network Testing

### cURL - Command Line URL Transfer
**Description**: Transfer data with URLs, supports many protocols

**Installation**: Usually pre-installed, otherwise:
```bash
# Debian/Ubuntu
sudo apt-get install curl

# macOS (pre-installed)
brew install curl
```

**Network Usage**:
```bash
curl -I http://192.168.1.100                  # Headers only
curl -v http://192.168.1.100                  # Verbose
curl -X POST -d "data" http://192.168.1.100   # POST request
curl --proxy socks5://127.0.0.1:1080 http://target.com
```

---

## Traffic Analysis & Monitoring

### tcpdump - Command-line Packet Analyzer
**Description**: Powerful command-line packet analyzer

**Installation**: Usually pre-installed
```bash
# Debian/Ubuntu
sudo apt-get install tcpdump

# macOS (pre-installed)
```

**Basic Usage**:
```bash
tcpdump -i eth0                               # Capture on eth0
tcpdump -i eth0 -w capture.pcap               # Write to file
tcpdump -r capture.pcap                       # Read from file
tcpdump -i eth0 port 80                       # Filter by port
tcpdump -i eth0 host 192.168.1.100            # Filter by host
```

---

## VPN & Tunneling

### OpenVPN - Open Source VPN
**Installation**:
```bash
# Debian/Ubuntu
sudo apt-get install openvpn

# macOS
brew install openvpn
```

---

## Exploitation Frameworks

### Metasploit Framework
**Description**: World's most used penetration testing framework

**Installation**:
```bash
# Kali Linux (pre-installed)
# Ubuntu/Debian
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall
chmod 755 msfinstall
./msfinstall
```

**Network Module Examples**:
```bash
msfconsole
use auxiliary/scanner/portscan/tcp
use auxiliary/scanner/smb/smb_version
use auxiliary/dos/tcp/synflood
```

---

## Security Distributions

### Kali Linux
Most tools pre-installed. Download from: https://www.kali.org/

### Parrot Security OS
Alternative to Kali: https://www.parrotsec.org/

---

## Additional Resources

- **Nmap Reference Guide**: https://nmap.org/book/man.html
- **Wireshark User Guide**: https://www.wireshark.org/docs/
- **OWASP Testing Guide**: https://owasp.org/www-project-web-security-testing-guide/
- **Penetration Testing Execution Standard**: http://www.pentest-standard.org/

---

**Legal Notice**: These tools should only be used on networks you own or have explicit written permission to test. Unauthorized network scanning and exploitation is illegal.
