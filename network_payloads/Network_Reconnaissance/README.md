# Network Reconnaissance - Complete Guide

## 📖 What is Network Reconnaissance?

Network reconnaissance is the first phase of any penetration test or security assessment. It involves gathering information about target networks, hosts, services, and potential vulnerabilities without direct exploitation.

## 🎯 When to Use Network Reconnaissance

- **Initial Assessment**: Before any penetration testing
- **Network Mapping**: Understanding network topology
- **Asset Discovery**: Finding all devices on a network
- **Service Enumeration**: Identifying running services and versions
- **Vulnerability Assessment**: Finding potential security weaknesses

## 🔍 Types of Reconnaissance

### 1. Passive Reconnaissance
- **Definition**: Gathering information without directly interacting with the target
- **When to Use**: Stealth is critical, initial information gathering
- **Tools**: OSINT tools, Shodan, search engines, WHOIS
- **Techniques**:
  - DNS lookups without zone transfers
  - Public database searches
  - Social media intelligence
  - Cached web pages

### 2. Active Reconnaissance
- **Definition**: Directly interacting with target systems
- **When to Use**: Authorized testing, need detailed information
- **Tools**: Nmap, Masscan, Netdiscover
- **Techniques**:
  - Port scanning
  - Service fingerprinting
  - OS detection
  - Network mapping

## 🛠️ Tool Selection Guide

### When to Use Nmap
- **Detailed service/version detection needed**
- **NSE scripts for specific enumeration**
- **OS fingerprinting required**
- **Firewall/IDS evasion needed**
- **Small to medium networks (< 10,000 hosts)**

**Best For**:
```bash
nmap -sS -sV -O -p- --script=vuln target.com  # Comprehensive scan
nmap -T4 -A 192.168.1.0/24                    # Quick network overview
```

### When to Use Masscan
- **Large networks (thousands of hosts)**
- **Speed is priority over accuracy**
- **Initial port discovery on Internet ranges**
- **No service detection needed**

**Best For**:
```bash
masscan 0.0.0.0/0 -p80,443 --rate 100000      # Internet-wide scan
masscan 10.0.0.0/8 -p1-65535 --rate 10000     # Fast internal scan
```

### When to Use Netdiscover
- **Local network discovery**
- **Layer 2 reconnaissance**
- **Finding hosts that don't respond to ping**
- **Quick ARP-based discovery**

**Best For**:
```bash
netdiscover -r 192.168.1.0/24                 # Active ARP scan
netdiscover -p                                # Passive monitoring
```

## 📊 Reconnaissance Methodology

### Phase 1: Network Discovery (Layer 2/3)
**Objective**: Find all live hosts

```bash
# Method 1: Ping Sweep (Fast but can miss firewalled hosts)
nmap -sn 192.168.1.0/24

# Method 2: ARP Scan (Most reliable on local network)
arp-scan -l
netdiscover -r 192.168.1.0/24

# Method 3: Broadcast ICMP (Sometimes reveals hidden hosts)
ping -b 192.168.1.255
```

**When to Use Each**:
- **Ping Sweep**: Remote networks, quick overview
- **ARP Scan**: Local network, most reliable
- **Broadcast**: Legacy networks, find all responders

### Phase 2: Port Scanning
**Objective**: Identify open ports and services

```bash
# Quick Scan (Top 100 ports - 1-2 minutes)
nmap -F 192.168.1.100

# Standard Scan (Top 1000 ports - 5-10 minutes)
nmap 192.168.1.100

# Comprehensive Scan (All 65535 ports - 30+ minutes)
nmap -p- 192.168.1.100

# Fast Comprehensive (Using Masscan + Nmap)
masscan 192.168.1.100 -p1-65535 --rate 10000 -oL ports.txt
# Then detailed scan on found ports with nmap
```

**Choosing Scan Type**:
- **Quick**: Time-limited, initial assessment
- **Standard**: Balanced approach, most pentests
- **Comprehensive**: Thorough assessment, critical systems

### Phase 3: Service Enumeration
**Objective**: Identify service versions and details

```bash
# Service Version Detection
nmap -sV 192.168.1.100

# Aggressive Service Detection (more probes)
nmap -sV --version-intensity 9 192.168.1.100

# Service + OS + Scripts + Traceroute
nmap -A 192.168.1.100
```

**Version Intensity Levels**:
- **0**: Light (fast, less accurate)
- **5**: Default (balanced)
- **9**: Aggressive (slow, most accurate)

### Phase 4: Vulnerability Scanning
**Objective**: Find known vulnerabilities

```bash
# NSE Vulnerability Scripts
nmap --script=vuln 192.168.1.100

# Specific vulnerability checks
nmap --script=smb-vuln-ms17-010 192.168.1.100
nmap --script=ssl-heartbleed 192.168.1.100

# Safe scripts only
nmap --script=safe 192.168.1.100
```

## 🎭 Evasion Techniques

### Firewall Detection and Bypass

**1. Detect Firewall**:
```bash
# ACK Scan (Firewall mapping)
nmap -sA 192.168.1.100

# Compare results:
# Filtered = Firewall present
# Unfiltered = No firewall OR stateless firewall
```

**2. Bypass Techniques**:

**Fragmentation** - Breaks packets into small fragments
```bash
nmap -f 192.168.1.100                         # 8-byte fragments
nmap --mtu 16 192.168.1.100                   # 16-byte fragments
```
**When to Use**: Stateful firewalls that don't reassemble fragments

**Source Port Manipulation** - Use trusted ports
```bash
nmap -g 53 192.168.1.100                      # Source port 53 (DNS)
nmap --source-port 80 192.168.1.100           # Source port 80 (HTTP)
```
**When to Use**: Firewalls allow traffic from specific ports

**Decoy Scanning** - Hide among fake sources
```bash
nmap -D RND:10 192.168.1.100                  # 10 random decoys
nmap -D 192.168.1.5,ME,192.168.1.6 192.168.1.100
```
**When to Use**: IDS evasion, attribution hiding

**Timing Manipulation** - Slow down to avoid detection
```bash
nmap -T0 192.168.1.100                        # Paranoid (5 min between probes)
nmap -T1 192.168.1.100                        # Sneaky (15 sec between)
nmap --scan-delay 10s 192.168.1.100           # Custom 10s delay
```
**When to Use**: Sensitive targets with IDS/IPS

## 🔒 Stealth Scanning Techniques

### SYN Scan (Half-Open)
```bash
nmap -sS 192.168.1.100
```
- **Stealth Level**: Medium
- **Speed**: Fast
- **Logged**: Usually not by application, sometimes by firewall
- **When to Use**: Default choice, requires root

### FIN/NULL/Xmas Scans
```bash
nmap -sF 192.168.1.100  # FIN scan
nmap -sN 192.168.1.100  # NULL scan
nmap -sX 192.168.1.100  # Xmas scan
```
- **Stealth Level**: High
- **Speed**: Medium
- **Bypass**: Some firewalls ignore these
- **When to Use**: Bypassing simple firewalls, RFC-compliant systems

### Idle Scan (Zombie)
```bash
nmap -sI zombie.host:80 192.168.1.100
```
- **Stealth Level**: Maximum
- **Speed**: Slow
- **Bypass**: Complete source IP hiding
- **When to Use**: Need absolute anonymity, have idle zombie host

## 🎯 DNS Reconnaissance Techniques

### Basic DNS Enumeration
```bash
# Query all record types
dig target.com ANY
nslookup -type=any target.com

# Specific records
dig target.com MX                             # Mail servers
dig target.com NS                             # Name servers
dig target.com TXT                            # TXT records
dig target.com SOA                            # Start of Authority
```

### Zone Transfer Attack
**What**: Request full DNS database from server
**When to Use**: If DNS server allows transfers (misconfiguration)

```bash
# Test zone transfer
dig @ns1.target.com target.com axfr
host -l target.com ns1.target.com
nmap --script dns-zone-transfer target.com
```

**Why This Works**: Misconfigured DNS servers allow anyone to request full zone data

### Subdomain Enumeration
**Techniques**:

**1. Brute Force**:
```bash
gobuster dns -d target.com -w subdomains.txt
ffuf -w wordlist.txt -u http://FUZZ.target.com
```

**2. Passive Discovery**:
```bash
sublist3r -d target.com                       # Multiple sources
amass enum -passive -d target.com             # OSINT only
```

**3. Certificate Transparency**:
```bash
# Search CT logs for subdomains
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value'
```

## 📡 SNMP Enumeration

### Why SNMP is Valuable
- Reveals system information
- Shows network configuration
- Lists running processes
- Exposes installed software
- Provides routing tables

### SNMP Reconnaissance Flow
```bash
# 1. Discover SNMP services
nmap -sU -p 161 192.168.1.0/24

# 2. Brute force community strings
onesixtyone -c community.txt 192.168.1.100
nmap -sU -p 161 --script=snmp-brute 192.168.1.100

# 3. Enumerate with found community
snmpwalk -v 2c -c public 192.168.1.100

# 4. Extract specific information
snmpwalk -v 2c -c public 192.168.1.100 .1.3.6.1.2.1.1        # System
snmpwalk -v 2c -c public 192.168.1.100 .1.3.6.1.2.1.25.4.2.1.2  # Processes
snmpwalk -v 2c -c public 192.168.1.100 .1.3.6.1.2.1.25.6.3.1.2  # Software
```

**Common Community Strings**:
- public (read-only)
- private (read-write)
- community
- manager

## 🚦 Traffic Analysis for Reconnaissance

### Passive Network Monitoring
```bash
# Monitor all traffic
tcpdump -i eth0 -w capture.pcap

# Monitor specific protocols
tcpdump -i eth0 'port 53'                     # DNS
tcpdump -i eth0 'port 80 or port 443'         # HTTP/HTTPS
tcpdump -i eth0 'arp'                         # ARP

# Extract hostnames from traffic
tshark -r capture.pcap -Y "dns.qry.name" -T fields -e dns.qry.name | sort -u
```

### Banner Grabbing
**Purpose**: Identify service versions without using scanner

```bash
# Netcat method
nc -v 192.168.1.100 80
# Then send: HEAD / HTTP/1.0

# Curl method (HTTP/HTTPS)
curl -I http://192.168.1.100
curl -v telnet://192.168.1.100:21             # FTP

# Telnet method
telnet 192.168.1.100 25                       # SMTP
telnet 192.168.1.100 110                      # POP3

# Nmap banner grab
nmap -sV --script=banner 192.168.1.100
```

## 🛡️ Defenses to Bypass

### Common Defense Mechanisms

**1. Rate Limiting**
- **Defense**: Drops packets above threshold
- **Bypass**: Slow scan with timing options
```bash
nmap -T0 --max-rate 10 192.168.1.100
```

**2. Geo-blocking**
- **Defense**: Blocks by source country
- **Bypass**: Use VPN/proxy from allowed country
```bash
proxychains nmap -sT 192.168.1.100
```

**3. Port Knocking**
- **Defense**: Ports open only after specific sequence
- **Bypass**: Discover knock sequence via:
  - Traffic analysis
  - Configuration files
  - Default sequences
```bash
knock target.com 1234 5678 9012
nmap -p 22 target.com
```

**4. Honeypots**
- **Defense**: Fake services to detect attackers
- **Detection**:
  - Unusual service banners
  - Too many open ports
  - Services that shouldn't exist
- **Avoid**: Reconnaissance only, don't exploit

## 📝 Practical Attack Scenarios

### Scenario 1: Corporate Network Assessment
```bash
# 1. Passive OSINT
whois target.com
dig target.com ANY
amass enum -passive -d target.com

# 2. External perimeter scan
nmap -sS -p- --min-rate 5000 target.com -oA external

# 3. Service enumeration
nmap -sV -p <found_ports> target.com

# 4. Vulnerability assessment
nmap --script vuln -p <found_ports> target.com
```

### Scenario 2: Internal Network Discovery
```bash
# 1. Find all hosts
arp-scan -l
nmap -sn 192.168.1.0/24

# 2. Quick port scan on all hosts
masscan 192.168.1.0/24 -p1-1000 --rate 10000

# 3. Detailed scan on interesting hosts
nmap -A -p- 192.168.1.10,192.168.1.20,192.168.1.30

# 4. Service-specific enumeration
enum4linux -a 192.168.1.10
snmpwalk -v 2c -c public 192.168.1.20
```

## 🎓 Best Practices

1. **Always get written authorization** before scanning
2. **Start passive**, move to active only if needed
3. **Use appropriate timing** - aggressive scans alert defenders
4. **Document everything** - save all scan results
5. **Verify findings** - false positives are common
6. **Be aware of impact** - some scans can crash services
7. **Use VPN/proxy** when appropriate for attribution hiding

## 🔍 Information to Extract

From reconnaissance, you should gather:
- ✅ Network topology and IP ranges
- ✅ Live host inventory
- ✅ Open ports and services
- ✅ Service versions and OS
- ✅ Domain names and subdomains
- ✅ Email addresses and usernames
- ✅ Network architecture (routers, firewalls, etc.)
- ✅ Potential vulnerabilities
- ✅ Security controls in place

## ⚠️ Common Mistakes to Avoid

1. **Too aggressive scanning** → Detection and blocking
2. **Not checking UDP** → Missing critical services (DNS, SNMP)
3. **Ignoring IPv6** → Incomplete assessment
4. **Single tool reliance** → Missing findings
5. **No baseline** → Can't detect changes
6. **Assuming closed = secure** → Filtered ports need investigation

## 📚 Additional Resources

- Nmap Reference Guide: https://nmap.org/book/
- NSE Script Documentation: https://nmap.org/nsedoc/
- OSINT Framework: https://osintframework.com/
- DNS Dumpster: https://dnsdumpster.com/
