# Episode 3: eps1.2_d3bug.mkv

## Overview

Elliot uses his hacking skills for a personal vendetta, targeting Fernando Vera, a violent drug dealer who supplies drugs to Elliot's neighbor Shayla. The episode showcases OSINT techniques applied to drug trafficking networks, email compromise, social media analysis, and the continued investigation of the E Corp rootkit's command-and-control infrastructure. DeepSound steganography is used again for secure data storage, and Android phone exploitation through rooting and spyware installation is depicted.

---

## Hacks & Techniques

### 1. Fernando Vera Surveillance and Hacking

Elliot launches a targeted hacking campaign against Fernando Vera, a local drug dealer. This demonstrates how the same offensive security techniques used against corporate targets or nation-states can be applied to individuals, leveraging the target's digital footprint to build a comprehensive intelligence picture.

**Attack phases:**

1. **Target identification**: Establishing Vera's digital presence across social media, messaging platforms, and communication channels
2. **Network mapping**: Identifying associates, suppliers, customers, and the organizational hierarchy of Vera's drug operation
3. **Vulnerability assessment**: Finding exploitable weaknesses in Vera's digital security practices
4. **Exploitation**: Gaining access to Vera's accounts, communications, and operational data
5. **Intelligence gathering**: Extracting evidence of criminal activity for potential use

**Elliot's approach highlights a key reality of cybersecurity**: most individuals, even those engaged in criminal activity requiring operational security, have poor digital hygiene. Vera's use of social media and unencrypted communications creates extensive attack surface.

### 2. OSINT on Drug Dealer's Network

Elliot performs comprehensive open-source intelligence gathering to map Vera's drug trafficking network. This involves correlating data from multiple sources to build a complete picture of operations, personnel, and infrastructure.

**OSINT methodology applied:**

- **Social graph analysis**: Mapping connections between Vera's associates through social media friend/follow lists, tagged photos, check-ins, and shared group memberships
- **Geolocation intelligence (GEOINT)**: Extracting GPS coordinates from photo EXIF metadata, correlating check-in data with known drug distribution locations, and analyzing movement patterns
- **Communication pattern analysis**: Identifying phone numbers, email addresses, and social media accounts associated with the network through public posts, profile information, and data broker records
- **Financial intelligence**: Identifying Venmo/CashApp transactions, Bitcoin wallet addresses, or other financial traces visible in public posts or forums
- **Temporal analysis**: Establishing schedules, routines, and operational patterns from timestamped social media activity

**OSINT tools and sources (implied):**

| Source | Intelligence Value |
|--------|-------------------|
| Instagram/Twitter | Associate identification, location data, lifestyle indicators |
| Facebook | Social network mapping, group memberships, event attendance |
| Public records | Addresses, property ownership, court records, arrest history |
| Google dorking | Cached pages, exposed documents, indexed profiles |
| Spokeo/Pipl | Aggregated personal data from multiple sources |
| EXIF data | GPS coordinates, device information, timestamps |

### 3. Email Compromise and Social Media Analysis

Elliot gains access to Vera's email accounts, providing a treasure trove of intelligence about his operations. Email compromise remains one of the most impactful forms of unauthorized access because email serves as the hub of most people's digital identity.

**Email compromise techniques:**

- **Credential reuse**: Testing passwords obtained from data breaches against email accounts (credential stuffing)
- **Password reset exploitation**: Using knowledge of personal information (gathered through OSINT) to answer security questions
- **Phishing**: Sending crafted emails designed to capture login credentials
- **Session hijacking**: Intercepting authentication tokens on unsecured networks

**Intelligence extracted from email access:**

- Contact lists revealing network members
- Transaction records and financial communications
- Location information from email headers and content
- Attached documents with operational details
- Password reset emails revealing other accounts
- Two-factor authentication codes (if email-based 2FA is used)

**Social media analysis capabilities with compromised credentials:**

- Direct message history revealing private communications
- Deleted posts recoverable through account data exports
- Private/restricted content not visible publicly
- Account activity logs showing login locations and devices

### 4. Rootkit C2 (Command and Control) Infrastructure Discovery

The investigation of the E Corp rootkit advances as Elliot traces its command-and-control infrastructure, the servers that send commands to the rootkit and receive exfiltrated data.

**C2 infrastructure analysis:**

- **Beacon analysis**: Monitoring the rootkit's periodic connections to external servers, noting timing intervals, destination IPs, and payload sizes
- **DNS analysis**: Examining DNS queries for domain generation algorithm (DGA) patterns, fast-flux hosting, or DNS tunneling used for covert communication
- **Traffic decryption**: Attempting to intercept or decrypt the encrypted communication channel between the rootkit and its C2 servers
- **Infrastructure mapping**: Using passive DNS databases, WHOIS records, and hosting provider information to trace the C2 servers to their operators
- **Geolocation**: Determining the physical locations of C2 servers to assess the likely origin of the attack

**C2 architecture patterns identified:**

```
Rootkit on E Corp Server
    |
    |--- DNS Query to DGA domain (changes daily)
    |--- HTTPS POST to C2 IP (encrypted beacon)
    |
    v
C2 Server (bulletproof hosting)
    |
    |--- Receives exfiltrated data
    |--- Sends commands/updates
    |--- Proxied through multiple jurisdictions
```

### 5. DeepSound Steganography for Data Storage on CDs

Elliot uses DeepSound steganography to store sensitive data hidden within audio files, which are then burned to CDs for offline storage. This represents a layered approach to data security combining steganography, encryption, and air-gapped storage.

**Data storage methodology:**

1. **Data preparation**: Sensitive files (evidence, credentials, intelligence) are collected and organized
2. **Encryption**: Data is encrypted with AES-256 before embedding, providing cryptographic protection
3. **Steganographic embedding**: Encrypted data is hidden within audio files using DeepSound's LSB (least significant bit) technique
4. **Physical storage**: Audio files are burned to CDs, creating an air-gapped (offline) backup that cannot be remotely accessed
5. **Labeling and organization**: CDs are labeled as normal music albums to avoid suspicion during physical searches

**Security advantages of this approach:**

- **Plausible deniability**: CDs appear to contain normal music; casual inspection reveals nothing suspicious
- **Air-gapped storage**: Physical media cannot be accessed remotely, protecting against network-based attacks
- **Layered defense**: Even if the steganography is detected, the underlying data is still AES-256 encrypted
- **Physical control**: The owner has complete custody of the storage medium
- **Forensic resistance**: Standard forensic tools focused on file system analysis may not detect steganographic content in audio files

### 6. Android Phone Rooting and Spyware Installation

The episode depicts the process of rooting an Android phone and installing surveillance software to monitor a target's communications, location, and activities.

**Android rooting process:**

Rooting an Android device means gaining superuser (root) access to the operating system, bypassing manufacturer and carrier restrictions. This provides complete control over the device, including the ability to install software that operates with unrestricted system-level privileges.

**SuperSU:**

- A root access management tool for Android that acts as a gatekeeper for superuser permissions
- Allows the user to grant or deny root access to individual applications
- Logs all root access requests for auditing
- Required for spyware that needs system-level access to intercept calls, messages, and other protected data

**FlexiSPY:**

FlexiSPY is a commercial surveillance application (sometimes called "stalkerware" or a "monitoring tool") that, once installed on a rooted device, provides comprehensive surveillance capabilities:

| Capability | Description |
|-----------|-------------|
| Call interception | Record and listen to phone calls in real time |
| SMS/MMS capture | Read all text messages sent and received |
| GPS tracking | Real-time location monitoring with geofencing |
| Ambient recording | Remotely activate microphone for room surveillance |
| Camera capture | Remotely activate camera for photo/video |
| App monitoring | Read messages from WhatsApp, Signal, Telegram, etc. |
| Keylogging | Capture all keystrokes including passwords |
| Screen capture | Periodic screenshots of device activity |
| Browser history | Complete web browsing history and bookmarks |
| Email access | Read all emails sent and received |

**Installation requirements:**

- Physical access to the target device (at least briefly)
- Device must be rooted (SuperSU or equivalent)
- Unknown sources must be enabled in Android settings
- Google Play Protect must be disabled to prevent detection
- The spy app runs as a system service, surviving reboots and factory reset (on some devices)

---

## Tools Used

| Tool | Purpose | Category |
|------|---------|----------|
| DeepSound | Audio steganography for hiding data in sound files | Steganography |
| SuperSU | Android root access management | Mobile Exploitation |
| FlexiSPY | Commercial spyware/monitoring for Android | Mobile Surveillance |
| Social media platforms | OSINT data sources for target profiling | OSINT |
| Email client access | Reading compromised email accounts | Account Compromise |
| CD burning software | Creating air-gapped physical data backups | Data Storage |
| Wireshark / tcpdump | Analyzing rootkit C2 network traffic | Network Analysis |

---

## Commands Shown

### OSINT and reconnaissance
```bash
# Search for target across multiple social media platforms
# Using theHarvester for email and subdomain enumeration
theharvester -d target_domain.com -b all

# Extract EXIF data from downloaded photos
exiftool downloaded_photo.jpg

# Extract GPS coordinates from image metadata
exiftool -gpslatitude -gpslongitude *.jpg

# Search for email addresses in data breach databases (conceptual)
# Tools like h8mail can check breach databases
h8mail -t target@email.com

# Google dorking for target information
# site:instagram.com "fernando vera"
# site:facebook.com "fernando vera" city_name
```

### Rootkit C2 analysis
```bash
# Capture network traffic on the compromised server
tcpdump -i eth0 -w capture.pcap

# Filter for DNS queries that may reveal DGA domains
tcpdump -i eth0 port 53 -nn

# Analyze packet capture for C2 beacon patterns
tshark -r capture.pcap -Y "ip.dst != 10.0.0.0/8" -T fields -e ip.dst -e tcp.dstport | sort | uniq -c | sort -rn

# Check for suspicious cron jobs (persistence mechanism)
crontab -l
ls -la /etc/cron.*
cat /etc/crontab

# Inspect loaded kernel modules for rootkit components
lsmod | grep -v "^Module"
cat /proc/modules
```

### Android rooting and spyware (conceptual)
```bash
# Enable USB debugging on Android device
# Settings > Developer Options > USB Debugging

# Check device connection via ADB
adb devices

# Push SuperSU to device
adb push SuperSU.apk /sdcard/

# Install SuperSU
adb install SuperSU.apk

# Push and install spyware APK
adb push flexispy.apk /sdcard/
adb shell pm install -r /sdcard/flexispy.apk

# Grant system-level permissions (on rooted device)
adb shell su -c "pm grant com.flexispy.app android.permission.READ_SMS"
adb shell su -c "pm grant com.flexispy.app android.permission.RECORD_AUDIO"
adb shell su -c "pm grant com.flexispy.app android.permission.ACCESS_FINE_LOCATION"

# Hide the app from the launcher
adb shell su -c "pm disable com.flexispy.app/.LauncherActivity"
```

### DeepSound usage
```bash
# DeepSound is a Windows GUI application
# Conceptual workflow:

# 1. Open DeepSound application
# 2. Load carrier audio file (e.g., album_track.wav)
# 3. Add files to hide (evidence, logs, data)
# 4. Set AES-256 encryption password
# 5. Encode - output appears as normal audio file
# 6. Burn encoded audio files to CD

# Verify CD contents appear as normal audio
file /media/cdrom/track01.wav
# Output: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, stereo 44100 Hz
```

---

## Real-World Parallels

### OSINT in Law Enforcement
Law enforcement agencies routinely use OSINT techniques similar to Elliot's approach. The DEA's Special Operations Division uses social media analysis, communication metadata, and public records to map drug trafficking networks. In 2013, the Silk Road investigation by the FBI used OSINT to identify Ross Ulbricht (Dread Pirate Roberts) through forum posts, Stack Overflow questions, and other digital breadcrumbs that connected his real identity to his anonymous persona.

### Commercial Spyware and Stalkerware
The FlexiSPY-style spyware depicted in the show mirrors a real and growing industry. FlexiSPY itself is a real product marketed for "employee monitoring" and "parental control." The NSO Group's Pegasus spyware, exposed extensively by Citizen Lab and Amnesty International, represents the nation-state tier of this technology, capable of zero-click exploitation of fully patched iOS and Android devices. In 2021, the Pegasus Project investigation revealed targeting of journalists, activists, and heads of state across 50+ countries.

### Steganography in Criminal Operations
Criminal organizations have been documented using steganography for operational communications. In 2012, the German Federal Criminal Police Office discovered that al-Qaeda operatives used steganography to hide operational documents in pornographic videos on public websites. The hidden files contained details of planned operations and were password-protected in addition to the steganographic concealment.

### Email Compromise as an Intelligence Tool
The 2014 Sony Pictures hack attributed to North Korea (Lazarus Group) demonstrated the devastating intelligence value of email compromise. Attackers exfiltrated and leaked thousands of internal emails, revealing confidential business strategies, employee salary data, unreleased film scripts, and embarrassing personal communications. The 2016 Democratic National Committee email compromise similarly showed how email access can provide comprehensive intelligence about an organization's operations and personnel.

---

## Tool Links

- **DeepSound**: http://jpinsoft.net/deepsound/
- **FlexiSPY**: https://www.flexispy.com/
- **SuperSU**: discontinued (was at chainfire.eu)
- **Wireshark**: https://www.wireshark.org/
- **tcpdump**: https://www.tcpdump.org/
- **Maltego** (OSINT, implied): https://www.maltego.com/
- **Shodan** (reconnaissance, implied): https://www.shodan.io/

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| OSINT on drug dealer network | T1589 | Gather Victim Identity Information | https://attack.mitre.org/techniques/T1589/ |
| Social media reconnaissance | T1593 | Search Open Websites/Domains | https://attack.mitre.org/techniques/T1593/ |
| Email compromise via credential reuse | T1110 | Brute Force | https://attack.mitre.org/techniques/T1110/ |
| Email account access | T1114 | Email Collection | https://attack.mitre.org/techniques/T1114/ |
| Rootkit C2 beacon analysis | T1071 | Application Layer Protocol | https://attack.mitre.org/techniques/T1071/ |
| DNS tunneling for C2 | T1572 | Protocol Tunneling | https://attack.mitre.org/techniques/T1572/ |
| DGA-based C2 infrastructure | T1568 | Dynamic Resolution | https://attack.mitre.org/techniques/T1568/ |
| DeepSound steganography for data hiding | T1027 | Obfuscated Files or Information | https://attack.mitre.org/techniques/T1027/ |
| Android phone rooting (privilege escalation) | T1068 | Exploitation for Privilege Escalation | https://attack.mitre.org/techniques/T1068/ |
| FlexiSPY spyware - input capture | T1056 | Input Capture | https://attack.mitre.org/techniques/T1056/ |
| FlexiSPY spyware - data collection | T1005 | Data from Local System | https://attack.mitre.org/techniques/T1005/ |
| Rootkit persistence via cron jobs | T1053 | Scheduled Task/Job | https://attack.mitre.org/techniques/T1053/ |

---

## References and Further Reading

- **Silk Road Investigation - FBI OSINT Techniques**: https://www.fbi.gov/news/stories/ross-ulbricht-sentenced-may-2015
- **NSO Group Pegasus Spyware - Citizen Lab Research**: https://citizenlab.ca/tag/pegasus/
- **Pegasus Project (2021)**: https://www.theguardian.com/news/series/pegasus-project
- **Sony Pictures Hack (2014) - FBI Analysis**: https://www.fbi.gov/news/press-releases/update-on-sony-investigation
- **DNC Email Compromise (2016)**: https://www.crowdstrike.com/blog/bears-midst-intrusion-democratic-national-committee/
- **Steganography in Al-Qaeda Operations (2012)**: https://www.bka.de/
- **SANS - OSINT Techniques for Network Mapping**: https://www.sans.org/white-papers/
- **FlexiSPY and Stalkerware Research - EFF**: https://www.eff.org/issues/stalkerware
- **Android Rooting Security Implications**: https://source.android.com/docs/security

---

## Search Tags

```
tags: [OSINT, DeepSound, steganography, FlexiSPY, SuperSU, Android rooting, spyware, stalkerware, email compromise, credential stuffing, C2, DNS tunneling, DGA, social media analysis, EXIF, geolocation, drug trafficking, Wireshark, tcpdump]
season: 1
episode: 3
mitre: [T1589, T1593, T1110, T1114, T1071, T1572, T1568, T1027, T1068, T1056, T1005, T1053]
```