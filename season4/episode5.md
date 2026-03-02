# Episode 5: eps4.4_405methodnotallowed.h

## Season 4, Episode 5 | "405 Method Not Allowed"

**Air Date:** November 3, 2019
**Directed by:** Sam Esmail

### Overview

This is the famous "silent episode" of Mr. Robot, presented almost entirely without dialogue and filmed to appear as a single continuous take. The episode title references HTTP status code 405, which indicates that the request method is known by the server but not supported for the target resource. The episode is a masterclass in physical penetration testing, as Elliot and Darlene break into the Virtual Realty data center to install network taps and exfiltrate data. Every technique shown is grounded in real-world physical security testing methodology.

---

## Hacks & Techniques

### 1. Physical Penetration Testing of Virtual Realty Data Center

Physical penetration testing (physical pentest) evaluates the physical security controls of a facility. This episode depicts a comprehensive physical intrusion operation against a hardened data center.

#### Physical Pentest Phases

1. **Reconnaissance:**
   - Identify facility location, layout, entry/exit points, and security measures.
   - Study employee schedules, shift changes, and delivery windows.
   - Map security camera coverage, blind spots, and patrol patterns.
   - Identify the type of access control system (card readers, biometrics, PIN pads).

2. **Planning:**
   - Develop entry strategy based on identified weaknesses.
   - Prepare tools and equipment (badge cloner, lock picks, network taps, storage devices).
   - Establish communication plan and abort procedures.
   - Plan extraction route and timeline.

3. **Execution:**
   - Gain physical access through the chosen method.
   - Navigate to the target area while avoiding detection.
   - Complete the objective (data exfiltration, device installation, evidence gathering).
   - Exit without being detected or raising alarms.

4. **Reporting:**
   - Document all security failures exploited.
   - Provide evidence (photos, video, access logs).
   - Recommend remediation measures.

### 2. RFID/NFC Badge Cloning

The episode shows badge cloning as the primary method of bypassing electronic access control systems. RFID (Radio-Frequency Identification) and NFC (Near-Field Communication) badges are ubiquitous in corporate and data center environments.

#### RFID Frequency Bands

| Frequency | Standard | Range | Common Use |
|---|---|---|---|
| **125 kHz (LF)** | HID Prox, EM4100 | ~10 cm | Legacy access control (no encryption) |
| **13.56 MHz (HF)** | MIFARE, iCLASS, DESFire | ~5 cm | Modern access control, transit cards |
| **860-960 MHz (UHF)** | EPC Gen2 | ~12 m | Inventory tracking, vehicle access |

#### Proxmark3

The Proxmark3 is the premier RFID research and cloning device:

- **Read:** Identifies card type, frequency, and data stored on the card.
- **Clone:** Copies card data to a blank writable card (T5577 for LF, MIFARE Classic for HF).
- **Simulate:** Emulates a card without needing a physical clone.
- **Sniff:** Captures communications between a card and reader.
- **Brute Force:** Tests keys for encrypted cards (MIFARE Classic's CRYPTO1 has known vulnerabilities).

#### ACR122U

The ACR122U is a USB NFC reader/writer commonly used for HF (13.56 MHz) card operations:

- Works with MIFARE Classic, MIFARE DESFire, NTAG, and other ISO 14443 cards.
- Can read, write, and clone NFC cards.
- Supported by libnfc and nfc-tools on Linux.
- More affordable and portable than the Proxmark3 but limited to HF frequencies.

#### Badge Cloning Process

```
Step 1: Identify Target Badge
+------------------+
| Security Guard   |
| Badge: HID Prox  |
| Freq: 125 kHz    |
+------------------+
         |
Step 2: Covert Reading (Proxmark3)
+------------------+
| proxmark3> lf    |
| hid read         |
| Tag ID: 2004..   |
| FC: 44 CN: 1337  |
+------------------+
         |
Step 3: Clone to Blank Card
+------------------+
| proxmark3> lf    |
| hid clone        |
| 2004...          |
| Written to T5577 |
+------------------+
         |
Step 4: Use Cloned Badge
+------------------+
| *beep* Access    |
| Granted          |
+------------------+
```

### 3. Lock Picking and Physical Bypass Techniques

When electronic access control is not the only barrier, physical lock bypass becomes necessary.

#### Lock Picking Fundamentals

- **Pin Tumbler Locks:** The most common lock type. Contains spring-loaded pins of varying heights that must align at the shear line when the correct key is inserted.
- **Single Pin Picking (SPP):** Using a tension wrench and pick to set each pin individually at the shear line.
- **Raking:** Using a serrated pick (rake) with rapid in-and-out motion while applying tension to set multiple pins simultaneously.
- **Bump Keys:** A specially cut key that, when struck ("bumped"), momentarily forces all pins to jump, allowing the lock to turn.

#### Common Lock Picking Tools

| Tool | Purpose |
|---|---|
| **Tension Wrench** | Applies rotational pressure to the lock cylinder |
| **Hook Pick** | Sets individual pins (SPP technique) |
| **Rake Pick (Bogota, Snake)** | Rapidly sets multiple pins |
| **Diamond Pick** | Versatile pick for various pin positions |
| **Bump Key** | Percussion-based bypass |
| **Bypass Tool** | Reaches past the lock mechanism entirely |
| **Electric Pick Gun** | Automated vibration-based pick |
| **Disc Detainer Pick** | Specialized for disc detainer locks (Abloy style) |

#### Physical Bypass Techniques

- **Latch Slipping:** Using a credit card or shim to push back a spring-loaded latch on a door.
- **Under-Door Tools:** Reaching under a door gap to pull an interior handle.
- **REX (Request to Exit) Sensor Bypass:** Many doors have motion or infrared sensors on the inside for egress. These can sometimes be triggered from outside using compressed air under the door or a device slid underneath.
- **Hinge Pin Removal:** If hinges are on the accessible side of the door, removing hinge pins allows the door to be opened.

### 4. Tailgating Through Secured Doors

Tailgating (also called piggybacking) is following an authorized person through a secured door without presenting credentials.

#### Tailgating Techniques

- **Direct Tailgating:** Walking closely behind an authorized person as they badge in.
- **Hands Full Method:** Carrying boxes, equipment, or coffee to make the authorized person feel compelled to hold the door.
- **The Grateful Follower:** Arriving at the door just as someone exits, thanking them as you slip in.
- **Group Blend:** Joining a group of employees returning from lunch or a smoke break and walking in with the crowd.
- **Prop the Door:** Finding doors that have been propped open by employees (especially loading docks and smoking areas).

#### Anti-Tailgating Countermeasures

- **Mantraps:** Two-door entry systems where only one person can enter the interstitial space at a time.
- **Turnstiles:** Rotating barriers that allow only one person per badge swipe.
- **Tailgate Detection Systems:** Overhead sensors (infrared arrays, stereo cameras) that count the number of people entering per badge swipe and trigger alarms if a mismatch is detected.
- **Security Guards:** Human observation at entry points.

### 5. Booting from External Media to Bypass OS Authentication

Once physical access to a server or workstation is obtained, booting from external media bypasses the installed operating system's authentication entirely.

#### Boot Attack Methodology

1. **Access the System:** Gain physical access to the target computer.
2. **Modify Boot Order:** Enter BIOS/UEFI settings (F2, F12, DEL, ESC during POST) and change the boot order to prioritize USB or CD/DVD.
3. **Boot Live OS:** Boot from a prepared USB drive containing a live Linux distribution.
4. **Mount Internal Drives:** Access the internal hard drive's file system directly, bypassing all OS-level access controls.
5. **Extract Data:** Copy SAM/SYSTEM files (Windows) for offline password cracking, or directly read sensitive files.

#### Live Boot Distributions for Forensics/Hacking

| Distribution | Purpose |
|---|---|
| **Kali Linux (Live)** | Comprehensive penetration testing toolkit |
| **Hiren's Boot CD/PE** | System recovery and password reset |
| **Kon-Boot** | Bypass Windows/macOS login without modifying passwords |
| **CAINE** | Computer forensics live distribution |
| **Tails** | Anonymous live OS routing through Tor |

### 6. Network Tap Installation

A critical objective of the intrusion is installing a network tap to passively capture traffic from the data center's network.

#### Throwing Star LAN Tap

The Throwing Star LAN Tap (by Great Scott Gadgets) is a passive Ethernet tap:

- **Passive Design:** No power supply required; works by passively splitting the signal.
- **Full Duplex Capture:** Captures traffic in both directions on a 10/100 Mbps Ethernet link.
- **Undetectable:** Because it is passive, it does not generate any network traffic and cannot be detected through network scanning.
- **Limitation:** Only works with 10/100 Mbps connections. Gigabit Ethernet uses all four pairs for bidirectional communication and requires an active tap.

#### Network Tap Placement

```
Normal Connection:
[Server] ----Ethernet---- [Switch]

With Throwing Star LAN Tap:
[Server] ----[TAP]---- [Switch]
               |
            [Monitor Port]
               |
          [Capture Device]
          (Laptop/Pi with tcpdump)
```

#### Alternative Network Taps

- **Active Taps:** Regenerate the signal (works with Gigabit), but require power and are detectable.
- **SPAN/Mirror Port:** Configure a switch to copy traffic from one port to another (requires switch access).
- **ARP Spoofing:** Software-based "tapping" that redirects traffic through the attacker's machine (detectable, not passive).

### 7. Data Exfiltration via Portable Storage

Once network access is established or data is identified, exfiltrating it using portable storage devices.

#### Exfiltration Methods

- **USB Storage:** High-capacity USB drives (1TB+ available) for bulk data copy.
- **External SSD:** Higher read/write speeds for time-sensitive operations.
- **microSD Cards:** Extremely small and easy to conceal.
- **Network-Based:** If a network tap with remote access is installed, data can be exfiltrated over the network later.

#### Data Exfiltration Commands

```bash
# Mount the target system's drive
mount /dev/sda1 /mnt/target

# Copy specific files
cp -r /mnt/target/path/to/sensitive/data /media/usb/exfil/

# Create compressed archive of target data
tar czf /media/usb/exfil/data.tar.gz /mnt/target/var/lib/database/

# Use dd for full disk imaging
dd if=/dev/sda of=/media/usb/disk_image.raw bs=4M status=progress
```

### 8. Security Camera Avoidance and Alarm System Bypass

Navigating the data center requires avoiding or neutralizing surveillance and alarm systems.

#### Camera Avoidance Techniques

- **Blind Spot Mapping:** During reconnaissance, identify camera positions and calculate blind spots based on camera type and lens angle.
- **Timing:** Move during camera sweeps (for PTZ cameras) when the camera is pointed away.
- **IR LED Flooding:** Infrared LEDs can blind IR-sensitive cameras without visible light.
- **Physical Obstruction:** Temporarily obstructing camera view with strategically placed objects.

#### Alarm System Bypass

- **Motion Sensor Types:**
  - **PIR (Passive Infrared):** Detects body heat. Can be bypassed by moving very slowly or using thermal-blocking material.
  - **Microwave:** Detects movement through Doppler radar. Harder to bypass.
  - **Dual-Tech:** Combines PIR and microwave; both must trigger for an alarm.
  - **Ultrasonic:** Uses sound waves; rarely used in modern systems.
- **Door/Window Sensors:** Magnetic reed switches. Can be bypassed by placing a magnet on the sensor while opening the door.
- **Glass Break Sensors:** Detect the specific frequency of breaking glass.

### 9. 3D-Printed Fingerprint Bypass

To access the biometric-locked server room at Virtual Realty, Elliot and Darlene:

1. Photograph a security guard's fingerprint (from a glass or surface)
2. Process the image in Photoshop to enhance contrast and detail
3. 3D-print a replica on flexible material
4. Press the replica against the biometric scanner to gain access

This technique has been demonstrated by researchers to fool both capacitive and optical fingerprint scanners. In 2014, the Chaos Computer Club (CCC) cloned a German politician's fingerprint from a press conference photo.

### 10. ELAN Building Lighting System Hack

Elliot notices "ELAN" brand on electrical infrastructure, connects via IP, and exploits default credentials to access the building's lighting control system. He reconfigures it to kill all lights for their escape. Many building automation systems (ELAN, Control4, Crestron, Savant) ship with default or weak credentials and are accessible on the local network.

### 11. Surveillance Camera Firmware Disable

A firmware update is pushed to all video surveillance cameras, rendering them inoperable for approximately 40 minutes. This exploits the fact that IP cameras accept unsigned firmware and reboot during updates.

### 12. KVM Access to Servers

KVM (Keyboard, Video, Mouse) access provides direct console-level control of servers without going through network-based remote management.

#### KVM Attack Methodology

- **Local KVM Switch:** Data centers use KVM switches to manage multiple servers from a single console. Gaining access to a KVM switch provides control over every connected server.
- **IPMI/iLO/iDRAC:** Out-of-band management interfaces (Intelligent Platform Management Interface, HP Integrated Lights-Out, Dell iDRAC) provide KVM-over-IP access. Default credentials on these interfaces are a well-known vulnerability.
- **Crash Cart:** A portable workstation with a monitor, keyboard, and mouse brought to a server for direct console access.

---

## Tools Used

| Tool | Purpose |
|---|---|
| **Proxmark3** | RFID/NFC badge reading, cloning, and simulation |
| **ACR122U** | NFC card reading and writing |
| **Lock Pick Set** | Physical lock bypass |
| **Bump Keys** | Percussion-based lock opening |
| **Throwing Star LAN Tap** | Passive network traffic capture |
| **USB Rubber Ducky** | Keystroke injection for rapid command execution |
| **Kali Linux Live USB** | Bootable penetration testing environment |
| **Kon-Boot** | OS authentication bypass |
| **IR LED Blaster** | Camera blinding tool |
| **Portable External SSD** | High-speed data exfiltration |

---

## Commands Shown

### Proxmark3 Badge Cloning

```bash
# Identify card type and frequency
proxmark3> hw tune
proxmark3> lf search
proxmark3> hf search

# Read HID Prox card (125 kHz)
proxmark3> lf hid read
# Output: TAG ID: 2004xxxxxxxx (H10301 FC:44 CN:1337)

# Clone to T5577 blank card
proxmark3> lf hid clone -r 2004xxxxxxxx

# Read MIFARE Classic (13.56 MHz)
proxmark3> hf mf rdbl --blk 0 -k FFFFFFFFFFFF

# Dump all sectors of MIFARE Classic
proxmark3> hf mf dump

# Write cloned data to blank MIFARE card
proxmark3> hf mf restore
```

### Network Tap Traffic Capture

```bash
# Capture traffic from the network tap using tcpdump
tcpdump -i eth0 -w /media/usb/capture.pcap -c 100000

# Capture with specific filters
tcpdump -i eth0 -w /media/usb/capture.pcap 'port 443 or port 80 or port 3306'

# Use tshark for more detailed capture
tshark -i eth0 -w /media/usb/capture.pcap -f "host 10.0.0.1"

# Real-time analysis of captured traffic
tcpdump -i eth0 -A 'port 80' | grep -i 'password\|login\|user'
```

### Boot from External Media

```bash
# Create bootable Kali Linux USB
dd if=kali-linux-live-amd64.iso of=/dev/sdb bs=4M status=progress && sync

# Once booted, mount target drives
fdisk -l                          # List all drives
mount /dev/sda2 /mnt/target      # Mount Windows partition

# Extract Windows SAM database for password cracking
cp /mnt/target/Windows/System32/config/SAM /media/usb/
cp /mnt/target/Windows/System32/config/SYSTEM /media/usb/

# Crack passwords offline with samdump2
samdump2 SYSTEM SAM
```

### KVM and IPMI Access

```bash
# Scan for IPMI interfaces on the network
nmap -sU -p 623 --script ipmi-version 10.0.0.0/24

# Access IPMI interface with default credentials
ipmitool -I lanplus -H 10.0.0.100 -U ADMIN -P ADMIN chassis status

# Get IPMI user list
ipmitool -I lanplus -H 10.0.0.100 -U ADMIN -P ADMIN user list

# Dump IPMI password hashes
ipmitool -I lanplus -H 10.0.0.100 -U ADMIN -P ADMIN user list
# Then use Metasploit: auxiliary/scanner/ipmi/ipmi_dumphashes
```

---

## Real-World Parallels

### Physical Penetration Testing

- **Deviant Ollam:** Renowned physical security expert and lock picker who has documented numerous physical penetration test findings, including badge cloning, tailgating, and lock bypass techniques in data centers and corporate offices.
- **DEFCON Lockpicking Village:** Annual demonstration of how physical security often lags behind digital security, with attendees learning to pick locks in minutes.
- **Red Team Operations:** Military and intelligence agencies conduct physical red team assessments of their own facilities, testing everything from perimeter security to interior access controls.

### RFID Cloning Incidents

- **Hotel Lock Hacking (2012):** Cody Brocious demonstrated at Black Hat that Onity hotel locks (used in millions of hotel rooms worldwide) could be opened with a simple device plugged into the lock's DC port.
- **Tesla Key Fob Cloning (2018):** Researchers at KU Leuven demonstrated cloning Tesla Model S key fobs in seconds using a Proxmark3 and a Raspberry Pi, exploiting the weak 40-bit encryption used in the DST40 RFID protocol.
- **MIFARE Classic Hacking (2008):** Researchers cracked the CRYPTO1 cipher used in MIFARE Classic cards, affecting transit systems and access control installations worldwide (including the London Oyster Card).

### Data Center Physical Security Breaches

- **Hosted.com / C I Host (2007):** Armed attackers physically entered a Chicago data center and stole servers, demonstrating that even data centers can be targets of physical intrusion.
- **British Airways Data Center Power Loss (2017):** While not a security breach, the incident highlighted the catastrophic consequences of physical infrastructure failures, leading to the cancellation of over 700 flights.

### Network Tapping

- **NSA QUANTUM Program:** Leaked documents revealed that the NSA had programs for intercepting and modifying network hardware during shipping to install network taps (Operation SHOTGIANT targeted Huawei infrastructure).
- **Greek Wiretapping Scandal (2004-2005):** Unknown attackers installed lawful intercept software on Vodafone Greece's network to wiretap the phones of over 100 senior Greek government officials, including the Prime Minister.

## Tool Links

- [Proxmark3](https://proxmark.com/) - RFID/NFC badge reading, cloning, and simulation
- [Throwing Star LAN Tap](https://greatscottgadgets.com/throwingstar/) - Passive Ethernet network traffic capture
- [USB Rubber Ducky](https://shop.hak5.org/products/usb-rubber-ducky) - Keystroke injection for rapid command execution
- [Nmap](https://nmap.org/) - Network scanning for rogue device and IPMI detection
- [Tails](https://tails.net/) - Anonymous live operating system routed via Tor
- [Wireshark](https://www.wireshark.org/) - Analysis of traffic captured via network tap
- [Deviant Ollam resources](https://www.deviantollam.com/) - Physical security and lock picking expert
- [Lockpick tools (TOOOL)](https://toool.us/) - Lockpicker organization for physical security research
- [Hashcat](https://hashcat.net/hashcat/) - Offline password cracking (SAM database)
- [ELAN](https://www.elancontrolsystems.com/) - Building automation system exploited via default credentials

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Physical penetration of data center | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| RFID/NFC badge cloning | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| Tailgating through secured doors | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| External media boot for authentication bypass | T1091 | Replication Through Removable Media | https://attack.mitre.org/techniques/T1091/ |
| Passive network tap installation | T1040 | Network Sniffing | https://attack.mitre.org/techniques/T1040/ |
| Data exfiltration via portable storage | T1048 | Exfiltration Over Alternative Protocol | https://attack.mitre.org/techniques/T1048/ |
| Camera and alarm system evasion | T1564 | Hide Artifacts | https://attack.mitre.org/techniques/T1564/ |
| KVM/IPMI server access | T1021 | Remote Services | https://attack.mitre.org/techniques/T1021/ |
| SAM database extraction for cracking | T1005 | Data from Local System | https://attack.mitre.org/techniques/T1005/ |
| Lock picking and physical bypass | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| 3D-printed fingerprint bypass | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| Building lighting system takeover via default creds | T1078 | Valid Accounts | https://attack.mitre.org/techniques/T1078/ |
| Surveillance camera firmware disable | T1562 | Impair Defenses | https://attack.mitre.org/techniques/T1562/ |

## References and Further Reading

- Deviant Ollam - DEFCON presentations on physical penetration of data centers and corporate offices
- DEFCON Lockpicking Village - Physical security bypass demonstrations
- Tesla Key Fob Cloning (KU Leuven, 2018) - Key fob cloning using Proxmark3
- MIFARE Classic Hacking (2008) - CRYPTO1 cipher break affecting transit and access control systems
- Hotel Lock Hacking (Cody Brocious, Black Hat 2012) - Onity lock vulnerabilities
- NSA QUANTUM/SHOTGIANT - Network hardware interception in transit
- Greek Wiretapping Scandal (2004-2005) - Espionage via lawful intercept software
- C I Host Data Center Physical Breach (2007) - Physical intrusion at a Chicago data center

## Search Tags

```
tags: [proxmark3, throwing-star-LAN-tap, USB-rubber-ducky, RFID-cloning, badge-cloning, lock-picking, physical-pentest, network-tap, tailgating, data-exfiltration, KVM, IPMI, boot-bypass, tails, 3D-print-fingerprint, biometric-bypass, ELAN, building-automation, camera-firmware, default-credentials]
season: 4
episode: 5
mitre: [T1200, T1091, T1040, T1048, T1564, T1021, T1005, T1078, T1562]
```
