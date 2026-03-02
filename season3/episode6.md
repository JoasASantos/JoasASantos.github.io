# Season 3, Episode 6: eps3.5_kill-process.inc

## Episode Overview

The aftermath of the 71-building Stage 2 attack dominates this episode. The `.inc` file extension (include file, used in PHP and other languages) represents the inclusion of new elements into the narrative: the forensic investigation, the attribution battle, and the Dark Army's false flag operation to frame Iran for the attack. Digital forensics teams struggle with the scale of the investigation, while the Dark Army plants evidence to misdirect attribution. This episode is a deep dive into the post-incident world of forensics, attribution, and information warfare.

---

## Hacks & Techniques

### Aftermath of the 71-Building Attack

The Stage 2 attack resulted in:

- Physical destruction of 71 E Corp facilities
- Significant casualties among building occupants
- Destruction of paper financial records (the ostensible primary objective)
- Massive public panic and economic destabilization
- A national security incident requiring federal investigation coordination

**Incident scope:**

```
Affected Sites: 71 E Corp buildings across multiple states
Attack Vector: Modified UPS firmware causing thermal runaway
Physical Damage: Server rooms, records storage, surrounding infrastructure
Casualties: Thousands (specific numbers referenced later in the season)
Digital Evidence: UPS network management logs, firmware images, network traffic
Investigative Agencies: FBI, DHS, CISA, ATF (arson/explosives)
```

### Digital Forensics and Attribution Challenges

Forensic investigators face enormous challenges in attributing the attack:

**Evidence collection at scale:**

- 71 separate crime scenes, each requiring forensic teams
- Fire and explosion damage destroys much of the digital evidence
- Surviving UPS controllers must be forensically imaged before evidence degrades
- Network logs from E Corp's SIEM and flow collectors must be preserved and analyzed
- Chain of custody must be maintained across thousands of evidence items

**Attribution challenge framework:**

```
Level 1: What happened? (Technical reconstruction)
    - Modified firmware caused thermal runaway
    - Firmware was deployed through UPS network management interfaces

Level 2: How did it happen? (Attack path reconstruction)
    - Network intrusion path from corporate IT to OT
    - Credential theft and lateral movement
    - Firmware development and testing

Level 3: Who did it? (Attribution)
    - IP addresses, domains, infrastructure used
    - Code analysis (language, coding style, compiler artifacts)
    - Operational patterns (working hours, holidays observed)
    - Cui bono? (Who benefits?)

Level 4: On whose authority? (State sponsorship)
    - Political context and geopolitical motivations
    - Intelligence community assessments
    - Evidence of state resources and capabilities
```

### Forensic Disk Imaging of Compromised UPS Controllers

Forensic teams must create bit-for-bit images of the UPS controllers' storage to preserve evidence:

```bash
# Forensic imaging of UPS controller flash memory
# Connect via JTAG/UART to surviving UPS controllers

# Create forensic image using dd
dd if=/dev/mtd0 of=/evidence/ups_building_12_controller_a.img bs=512 conv=noerror,sync

# Calculate hash for chain of custody
sha256sum /evidence/ups_building_12_controller_a.img > /evidence/ups_building_12_controller_a.img.sha256
md5sum /evidence/ups_building_12_controller_a.img >> /evidence/ups_building_12_controller_a.img.sha256

# Use dc3dd for forensic imaging with built-in hashing
dc3dd if=/dev/mtd0 of=/evidence/ups_controller.img hash=sha256 log=/evidence/imaging_log.txt

# Using FTK Imager for GUI-based forensic imaging
# (Common in law enforcement forensic labs)

# Analyze firmware image
binwalk /evidence/ups_controller.img
strings /evidence/ups_controller.img | grep -i "version\|build\|date\|author"
```

**Firmware analysis for attribution:**

```bash
# Compare compromised firmware against clean vendor firmware
vbindiff clean_firmware.bin compromised_firmware.bin

# Extract and analyze modified code sections
radare2 -A compromised_firmware.bin
# [r2] afl    # List all functions
# [r2] pdf @main  # Disassemble main function
# [r2] /x 90909090  # Search for NOP sleds

# Identify compiler and build environment
file compromised_firmware.bin
readelf -h compromised_firmware.bin    # If ELF format
```

### Dark Army False Flag Operation Framing Iran

The Dark Army executes a sophisticated false flag operation to redirect attribution toward Iran:

**False flag techniques employed:**

1. **Language artifacts**: Embed Farsi/Persian language strings in malware code, comments, and configuration files
2. **Infrastructure overlap**: Route attack traffic through Iranian IP ranges or compromised Iranian servers
3. **Tooling overlap**: Use or imitate known Iranian APT tools (e.g., tools associated with APT33/Elfin, APT34/OilRig, or APT35/Charming Kitten)
4. **Working hour patterns**: Schedule automated actions during Iranian business hours (UTC+3:30)
5. **Targeting patterns**: Make the attack appear consistent with known Iranian strategic interests
6. **Code reuse**: Include code snippets from publicly analyzed Iranian malware samples

```python
# Example: Embedding false attribution indicators in malware

# Farsi strings planted in firmware modification code
# (Would appear in forensic string analysis)
false_strings = [
    "\u0628\u0631\u0646\u0627\u0645\u0647 \u0645\u0648\u0641\u0642 \u0628\u0648\u062f",  # "Program was successful" in Farsi
    "\u062d\u0645\u0644\u0647 \u0634\u0631\u0648\u0639 \u0634\u062f",  # "Attack started" in Farsi
    "C:\\Users\\Mohammad\\Desktop\\project\\",  # Farsi name in file path
]

# Timestamp manipulation to match Iranian working hours
# Iran Standard Time (IRST) = UTC+3:30
import datetime
iranian_working_hour = datetime.time(10, 30)  # 10:30 AM IRST = 7:00 AM UTC
# Set file timestamps to Iranian business hours
```

**Attribution indicators that forensic analysts would examine:**

| Indicator | Genuine | Planted (False Flag) |
|-----------|---------|---------------------|
| Language strings | Consistent throughout | Mixed or inconsistent |
| Compilation timestamps | Match timezone consistently | May have inconsistencies |
| Infrastructure | Established over time | Recently acquired or hijacked |
| TTPs | Match known actor profile | Cherry-picked from public reports |
| Code quality | Consistent style | Mixed styles suggesting copy-paste |
| Operational mistakes | Rare, consistent | Deliberately "careless" |

### Evidence Planting: Mimicking Other APT TTPs

The Dark Army studies publicly available threat intelligence reports to replicate known Iranian APT Tactics, Techniques, and Procedures (TTPs):

**Mimicked TTPs from known Iranian groups:**

- **APT33 (Elfin/Refined Kitten)**: Known for targeting energy and aviation sectors with custom droppers
- **APT34 (OilRig/Helix Kitten)**: Known for DNS tunneling C2, custom PowerShell backdoors
- **APT35 (Charming Kitten)**: Known for credential harvesting and spearphishing

```bash
# Planting false indicators in malware configuration
# Include domains mimicking Iranian APT infrastructure patterns
# Use naming conventions seen in public APT34 reports

# Example: Creating a fake C2 configuration that mimics APT34
cat << 'EOF' > fake_c2_config.json
{
    "c2_servers": [
        "update-service.ddns.net",
        "cdn-storage.hopto.org",
        "mail-verify.zapto.org"
    ],
    "dns_tunnel": {
        "domain": "ns1.iran-update.com",
        "subdomain_encoding": "base32"
    },
    "working_hours": {
        "start": "07:00",
        "end": "16:00",
        "timezone": "Asia/Tehran"
    }
}
EOF
```

### Olympic Destroyer Malware Parallel

The Dark Army's false flag operation mirrors the real-world Olympic Destroyer attack:

- **Olympic Destroyer (2018)**: Malware that targeted the Pyeongchang Winter Olympics opening ceremony. The malware contained multiple layers of false flags designed to misdirect attribution:
  - Rich header data pointed to Lazarus Group (North Korea)
  - Code similarities pointed to Chinese APT groups
  - Some indicators pointed to Russian groups
  - Ultimate attribution: GRU (Russian military intelligence) Unit 74455

The Olympic Destroyer case demonstrated that sophisticated actors can deliberately plant attribution indicators to create confusion and misdirection.

### Evidence Destruction: Secure Deletion

The Dark Army and other actors attempt to destroy digital evidence to hinder the investigation:

```bash
# Standard file deletion (NOT secure - data remains on disk)
rm sensitive_file.txt

# Secure deletion using shred (GNU coreutils)
# Overwrites file with random data multiple times before deletion
shred -vfz -n 7 sensitive_file.txt
# -v: verbose (show progress)
# -f: force (change permissions if needed)
# -z: final overwrite with zeros to hide shredding
# -n 7: 7 random overwrite passes

# Secure deletion using srm (secure-delete package)
srm -vz sensitive_file.txt
# Uses Gutmann 35-pass method by default

# Secure deletion of entire directories
srm -rvz /path/to/evidence_directory/

# Secure deletion of free space on a drive
sfill -vz /mount/point

# Disk wiping for complete evidence destruction
dd if=/dev/urandom of=/dev/sda bs=1M status=progress
# or
nwipe /dev/sda    # NIST 800-88 compliant wiping

# SSD considerations:
# Traditional overwriting is unreliable on SSDs due to wear leveling
# ATA Secure Erase is the recommended method for SSDs
hdparm --user-master u --security-set-pass password /dev/sda
hdparm --user-master u --security-erase password /dev/sda

# Memory forensics countermeasure - clear RAM
# (Prevents cold boot attacks)
smem -f        # Secure memory wiper
```

**Limitations of secure deletion:**

- SSDs with wear leveling make traditional overwriting unreliable
- TRIM commands may leave data in unmapped blocks
- Backup systems, snapshots, and replication may preserve copies
- Network logs on remote systems cannot be locally deleted
- Forensic techniques may recover data from partially overwritten magnetic media

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **dd / dc3dd** | Forensic disk imaging of UPS controllers |
| **FTK Imager** | GUI-based forensic imaging tool |
| **Binwalk** | Firmware image analysis and extraction |
| **Radare2 / Ghidra** | Reverse engineering of modified firmware |
| **Wireshark** | Network traffic analysis for forensic evidence |
| **shred / srm** | Secure file deletion |
| **nwipe** | Disk wiping for evidence destruction |
| **SIEM (Splunk/QRadar)** | Log analysis and event correlation |
| **MITRE ATT&CK** | Framework for TTP analysis and attribution |

---

## Commands Shown

```bash
# Forensic evidence preservation
dd if=/dev/sda of=/evidence/disk_image.raw bs=64K conv=noerror,sync
sha256sum /evidence/disk_image.raw | tee /evidence/hash_log.txt

# Firmware comparison and analysis
diff <(xxd clean_firmware.bin) <(xxd compromised_firmware.bin)
strings -n 8 compromised_firmware.bin > firmware_strings.txt

# Timeline analysis using log2timeline/Plaso
log2timeline.py /evidence/timeline.plaso /evidence/disk_image.raw
psort.py -o l2tcsv /evidence/timeline.plaso -w /evidence/timeline.csv

# Search for Farsi/Arabic strings (false flag indicators)
strings compromised_firmware.bin | grep -P '[\x{0600}-\x{06FF}]'

# Secure evidence destruction
shred -vfz -n 7 /tmp/staging/*
srm -rvz /tmp/tools/
dd if=/dev/zero of=/dev/sdb bs=1M status=progress

# Memory acquisition for forensic analysis
sudo insmod /opt/tools/lime.ko "path=/evidence/memory.lime format=lime"

# Volatility memory forensics
vol.py -f /evidence/memory.lime --profile=LinuxUbuntu_5_4_0-x64 linux_pslist
vol.py -f /evidence/memory.lime --profile=LinuxUbuntu_5_4_0-x64 linux_bash
```

---

## Real-World Parallels

### Olympic Destroyer (2018)
- The most prominent real-world example of a cyber false flag operation. Russian GRU Unit 74455 (Sandworm) deployed malware during the Pyeongchang Winter Olympics that contained multiple layers of fabricated attribution indicators pointing to North Korea and China. This directly parallels the Dark Army's operation to frame Iran for the Stage 2 attack. Kaspersky Lab published detailed analysis showing how the false flags were constructed.

### Attribution Challenges in Practice
- **Sony Pictures hack (2014)**: Initially attributed to North Korea, though some researchers questioned the attribution. The case highlighted how difficult definitive attribution is even with significant intelligence resources.
- **DNC hack (2016)**: Multiple threat intelligence firms attributed the attack to Russian groups (APT28/APT29), but the "Guccifer 2.0" persona was created as a false front to muddy attribution.
- **ShadowBrokers (2016-2017)**: Released NSA hacking tools with unclear attribution, demonstrating how stolen tools can complicate attribution when those tools are reused by other actors.

### Digital Forensics at Scale
- **Malaysian Airlines MH17 investigation**: Bellingcat and the Joint Investigation Team used digital forensics, metadata analysis, and open-source intelligence to attribute the shoot-down to Russian military units -- a model for large-scale forensic investigations.
- **NIST SP 800-86**: Guide to Integrating Forensic Techniques into Incident Response, the standard reference for federal forensic investigations like the one depicted in this episode.

### Secure Deletion Standards
- **DoD 5220.22-M**: Previously the standard for media sanitization, requiring three overwrite passes.
- **NIST SP 800-88 (Guidelines for Media Sanitization)**: Current standard that recognizes Clear, Purge, and Destroy as sanitization levels, acknowledging that SSD sanitization requires different approaches than HDD sanitization.
- **Gutmann method (1996)**: Peter Gutmann's 35-pass method, once considered the gold standard for secure deletion, now largely considered excessive for modern drives.

## Tool Links

- [Volatility](https://www.volatilityfoundation.org/) - Memory forensics framework for post-incident analysis
- [Autopsy](https://www.autopsy.com/) - Digital forensics platform with graphical interface
- [Sleuth Kit](https://www.sleuthkit.org/) - Command-line forensics tools
- [Binwalk](https://github.com/ReFirmLabs/binwalk) - Firmware image analysis and extraction
- [YARA](https://virustotal.github.io/yara/) - Malware detection and classification rules
- [Splunk](https://www.splunk.com/) - SIEM for log analysis and event correlation
- [ELK Stack](https://www.elastic.co/elastic-stack) - Forensic log analysis and visualization
- [Wireshark](https://www.wireshark.org/) - Network traffic analysis for forensic evidence
- [DBAN](https://dban.org/) - Secure disk wiping for evidence destruction
- [SDelete](https://learn.microsoft.com/en-us/sysinternals/downloads/sdelete) - Secure file deletion on Windows
- [Foremost](https://foremost.sourceforge.net/) - File carving for evidence recovery

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| False flag framing Iran | T1036 | Masquerading | https://attack.mitre.org/techniques/T1036/ |
| Planting false attribution indicators | T1027 | Obfuscated Files or Information | https://attack.mitre.org/techniques/T1027/ |
| Secure evidence destruction | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| Disk and SSD wiping | T1561 | Disk Wipe | https://attack.mitre.org/techniques/T1561/ |
| Secure file deletion (shred/srm) | T1070.004 | File Deletion | https://attack.mitre.org/techniques/T1070/004/ |
| Memory forensics with Volatility | T1005 | Data from Local System | https://attack.mitre.org/techniques/T1005/ |
| Timeline analysis with Plaso | T1082 | System Information Discovery | https://attack.mitre.org/techniques/T1082/ |
| Mimicking Iranian APT TTPs | T1583 | Acquire Infrastructure | https://attack.mitre.org/techniques/T1583/ |

## References and Further Reading

- **Olympic Destroyer Analysis (Kaspersky, 2018)**: Detailed analysis of multi-layered false flags used in the Olympic Destroyer malware
- **Sony Pictures Hack Attribution (2014)**: Case study on attribution challenges in large-scale cyber attacks
- **NIST SP 800-86**: Guide to Integrating Forensic Techniques into Incident Response
- **NIST SP 800-88 Rev. 1**: Guidelines for Media Sanitization -- current standard for secure media sanitization
- **DoD 5220.22-M**: Previous standard for media sanitization with three overwrite passes
- **Gutmann, Peter (1996)**: "Secure Deletion of Data from Magnetic and Solid-State Memory" -- 35-pass method
- **Bellingcat MH17 Investigation**: Model for large-scale digital forensic investigation using OSINT
- **APT33/APT34/APT35 Threat Intelligence Reports (FireEye/Mandiant)**: Public reports on Iranian APT group TTPs

## Search Tags

```
tags: [forensics, attribution, false-flag, Iran, APT33, APT34, APT35, Olympic-Destroyer, evidence-destruction, shred, disk-wipe, Volatility, Binwalk, SIEM, timeline-analysis, firmware-analysis]
season: 3
episode: 6
mitre: [T1036, T1027, T1070, T1561, T1070.004, T1005, T1082, T1583]
```
