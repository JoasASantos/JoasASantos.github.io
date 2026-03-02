# Episode 10: eps4.9_410gone.h

## Season 4, Episode 10 | "410 Gone"

**Air Date:** December 8, 2019
**Directed by:** Sam Esmail

### Overview

The episode title references HTTP status code 410, which indicates that the requested resource is permanently gone. Unlike 404 (not found), 410 explicitly states the resource existed previously but has been intentionally and permanently removed. This perfectly encapsulates the episode's focus: the Deus Group's wealth is permanently gone, redistributed through automated scripts, and the evidence of how it was done is being systematically and permanently destroyed through anti-forensic techniques.

---

## Hacks & Techniques

### 1. Wealth Redistribution Execution via Automated Scripts

Following the successful hack of the Deus Group's financial accounts, automated scripts handle the actual distribution of stolen funds to the general public.

#### Automated Distribution Architecture

```
+------------------+     +-------------------+     +------------------+
| Master Control   |---->| Distribution      |---->| Recipient        |
| Script           |     | Servers (Multiple)|     | Accounts         |
| (Orchestrator)   |     | (Load Balanced)   |     | (Millions)       |
+------------------+     +-------------------+     +------------------+
        |                         |                        |
   Monitors                 Processes                 Receives
   progress               transactions              micro-transfers
   and errors             in parallel               across banks
```

#### Distribution Script Concepts

```python
#!/usr/bin/env python3
"""
Fund Distribution - Automated Wealth Redistribution
Distributes funds across millions of recipient accounts
"""

import asyncio
import aiohttp
import hashlib
from typing import List, Dict

class WealthRedistributor:
    def __init__(self, source_funds: float, recipients: List[Dict]):
        self.total_funds = source_funds
        self.recipients = recipients
        self.per_recipient = source_funds / len(recipients)
        self.completed = 0
        self.failed = 0

    async def distribute(self):
        """Execute parallel distribution across all recipients"""
        semaphore = asyncio.Semaphore(1000)  # Limit concurrent transfers

        async def transfer_with_limit(recipient):
            async with semaphore:
                return await self._execute_transfer(recipient)

        tasks = [transfer_with_limit(r) for r in self.recipients]
        results = await asyncio.gather(*tasks, return_exceptions=True)

        for result in results:
            if isinstance(result, Exception):
                self.failed += 1
            else:
                self.completed += 1

        return {
            'total': len(self.recipients),
            'completed': self.completed,
            'failed': self.failed,
            'amount_per_recipient': self.per_recipient
        }

    async def _execute_transfer(self, recipient: Dict):
        """Execute a single transfer via banking API"""
        transfer_payload = {
            'destination_account': recipient['account'],
            'destination_bank': recipient['routing'],
            'amount': self.per_recipient,
            'currency': 'USD',
            'reference': hashlib.sha256(
                f"REDIST-{recipient['id']}".encode()
            ).hexdigest()[:16]
        }
        # Transfer execution logic
        return transfer_payload


if __name__ == "__main__":
    # Load recipient list and execute
    asyncio.run(main())
```

### 2. Anti-Forensics: Log Deletion and Timestomping

After the operation, all traces must be eliminated. Anti-forensics encompasses techniques used to prevent, disrupt, or mislead forensic analysis.

#### Log Deletion

Removing or tampering with logs is the most basic anti-forensic technique:

##### Windows Log Clearing

- **Security Log:** Contains authentication, authorization, and audit events.
- **System Log:** Contains service events, driver failures, and system changes.
- **Application Log:** Contains application-specific events.
- **PowerShell Log:** Contains PowerShell command execution history.
- **Sysmon Log:** If Sysmon is installed, contains detailed process, network, and file activity.

##### Linux Log Locations

| Log File | Contents |
|---|---|
| `/var/log/auth.log` | Authentication events (SSH logins, sudo usage) |
| `/var/log/syslog` | General system messages |
| `/var/log/kern.log` | Kernel messages |
| `/var/log/apache2/access.log` | Web server access logs |
| `/var/log/mysql/mysql.log` | Database query logs |
| `/var/log/cron` | Scheduled task execution |
| `~/.bash_history` | User command history |
| `/var/log/wtmp` | Login records |
| `/var/log/btmp` | Failed login records |
| `/var/log/lastlog` | Last login information |

#### Timestomping

Timestomping is the modification of file timestamps to mislead forensic analysis. Every file has multiple timestamps that can be manipulated:

##### NTFS File Timestamps (Windows - MACE)

- **M** - Modified: When the file content was last changed.
- **A** - Accessed: When the file was last opened/read.
- **C** - Changed (MFT): When the Master File Table entry was last changed.
- **E** - Entry Created: When the file was originally created.

##### Timestomping with Metasploit

The `timestomp` command in Meterpreter modifies all four MACE timestamps:

```
meterpreter > timestomp evil.exe -m "01/01/2019 12:00:00"
meterpreter > timestomp evil.exe -a "01/01/2019 12:00:00"
meterpreter > timestomp evil.exe -c "01/01/2019 12:00:00"
meterpreter > timestomp evil.exe -e "01/01/2019 12:00:00"

# Or modify all timestamps at once
meterpreter > timestomp evil.exe -z "01/01/2019 12:00:00"

# Copy timestamps from a legitimate file
meterpreter > timestomp evil.exe -f C:\\Windows\\System32\\notepad.exe
```

##### Detecting Timestomping

- **$STANDARD_INFORMATION vs $FILE_NAME:** NTFS stores timestamps in two attributes. `timestomp` typically only modifies `$STANDARD_INFORMATION`. Comparing both reveals manipulation.
- **$MFT Analysis:** The Master File Table records can reveal discrepancies.
- **Journal Analysis:** The NTFS $UsnJrnl (change journal) and $LogFile record file operations independently of file timestamps.

### 3. Secure Data Wiping

Simply deleting files does not remove data from disk. Secure wiping overwrites the data to prevent recovery.

#### shred (Linux)

```bash
# Overwrite file 3 times with random data, then zero, then delete
shred -vfz -n 3 sensitive_file.txt

# Shred and remove
shred -vfzu sensitive_file.txt

# Shred an entire partition
shred -vfz -n 3 /dev/sdb1
```

#### wevtutil (Windows Event Log Utility)

```powershell
# Clear specific event logs
wevtutil cl Security
wevtutil cl System
wevtutil cl Application
wevtutil cl "Microsoft-Windows-PowerShell/Operational"
wevtutil cl "Microsoft-Windows-Sysmon/Operational"

# Clear all event logs at once
for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1" 2>nul

# PowerShell equivalent
Get-WinEvent -ListLog * | ForEach-Object {
    try { [System.Diagnostics.Eventing.Reader.EventLogSession]::GlobalSession.ClearLog($_.LogName) }
    catch {}
}
```

#### DoD 5220.22-M Standard

The US Department of Defense's standard for data sanitization (from the National Industrial Security Program Operating Manual):

| Pass | Operation | Data Written |
|---|---|---|
| **Pass 1** | Write | All zeros (0x00) |
| **Pass 2** | Write | All ones (0xFF) |
| **Pass 3** | Write | Random data |
| **Verification** | Read | Verify overwrite success |

#### Gutmann Method

Peter Gutmann's 35-pass overwrite method, designed to defeat magnetic force microscopy:

- **Passes 1-4:** Random data
- **Passes 5-31:** Specific patterns designed to flip magnetic domains in various encoding schemes (MFM, RLL)
- **Passes 32-35:** Random data

> **Note:** Modern research suggests that for contemporary hard drives, a single pass of random data is sufficient to prevent data recovery. The Gutmann method was designed for older drive technologies. For SSDs, secure erase via the ATA Secure Erase command is the recommended approach.

### 4. Digispark USB Attack Preparation

Elliot is shown holding a small USB device that he will use in his assault on the Washington Township Nuclear Power Plant. Upon close inspection, the text **"Digispark"** is visible on the silkscreen of the board near the USB copper contact patches. Security expert **Trammell Hudson** identified this prop as a **Digispark** — an Arduino-compatible ATtiny85-based microcontroller that acts as a USB Human Interface Device (HID) for keystroke injection attacks.

#### What is the Digispark?

| Specification | Detail |
|---|---|
| **Microcontroller** | Atmel ATtiny85 |
| **Clock Speed** | 16.5 MHz |
| **Flash Memory** | 8 KB (~6,012 bytes usable) |
| **SRAM** | 512 bytes |
| **USB** | Built-in USB Type-A (software USB via V-USB) |
| **Dimensions** | ~18 x 22 mm |
| **Cost** | $2–$6 USD |

The Digispark is often called a **"budget USB Rubber Ducky"** — it provides the same USB HID keystroke injection capability at a fraction of the price ($2–6 vs ~$80 for the Hak5 Rubber Ducky). When plugged in, the OS automatically trusts it as a keyboard, allowing it to inject pre-programmed keystrokes at high speed.

#### Digispark vs USB Rubber Ducky

| Feature | Digispark ATtiny85 | USB Rubber Ducky (Hak5) |
|---|---|---|
| **Price** | $2–$6 | ~$80 |
| **Microcontroller** | ATtiny85 (8-bit AVR) | Custom 60 MHz 32-bit CPU |
| **Storage** | ~6 KB flash | MicroSD (2 GB+) |
| **Appearance** | Exposed PCB — conspicuous | Looks like normal USB drive — stealthy |
| **Programming** | Arduino C/C++ (DigiKeyboard.h) | DuckyScript |
| **Keystroke Speed** | Moderate | Very fast (~1000+ WPM) |

#### Programming Example (DigiKeyboard)

```cpp
#include "DigiKeyboard.h"

void setup() {
  DigiKeyboard.delay(2000);  // Wait for OS to recognize device
  DigiKeyboard.sendKeyStroke(KEY_R, MOD_GUI_LEFT);  // Win+R
  DigiKeyboard.delay(500);
  DigiKeyboard.print("powershell -w hidden -ep bypass ...");
  DigiKeyboard.sendKeyStroke(KEY_ENTER);
}

void loop() {}
```

> **Context:** Elliot prepares this device here and deploys it in the next episode (S4E11) at the Washington Township Nuclear Power Plant to deliver a ransomware payload via PowerShell and the "fuxor" encryption program.

### 5. Evidence Destruction and Track Covering

A comprehensive approach to eliminating all evidence of the operation.

#### Track Covering Checklist

```
OPERATIONAL SECURITY CLEANUP:

[ ] Clear all system logs (Security, System, Application, PowerShell)
[ ] Clear Sysmon logs if present
[ ] Delete Windows Prefetch files (C:\Windows\Prefetch\)
[ ] Clear browser history and cached data
[ ] Clear recent documents lists
[ ] Delete temporary files (%TEMP%, /tmp/)
[ ] Clear bash/PowerShell command history
[ ] Remove any tools/scripts uploaded to compromised systems
[ ] Remove persistence mechanisms (scheduled tasks, services, registry keys)
[ ] Overwrite free disk space to eliminate deleted file remnants
[ ] Clear DNS cache
[ ] Clear ARP cache
[ ] Remove firewall rule modifications
[ ] Restore any modified system configurations
[ ] Clear USN Journal entries (Windows)
[ ] Remove any created user accounts
[ ] Clear RDP/SSH connection logs
[ ] Wipe memory (to remove encryption keys and credentials)
```

#### Comprehensive Cleanup Script

```bash
#!/bin/bash
# Linux Anti-Forensics Cleanup

# Clear command history
history -c
history -w
> ~/.bash_history
> ~/.zsh_history
unset HISTFILE

# Clear system logs
for log in /var/log/auth.log /var/log/syslog /var/log/kern.log \
           /var/log/daemon.log /var/log/messages; do
    if [ -f "$log" ]; then
        shred -vfzu "$log" 2>/dev/null
        touch "$log"
    fi
done

# Clear login records
> /var/log/wtmp
> /var/log/btmp
> /var/log/lastlog

# Clear temporary files
rm -rf /tmp/* /var/tmp/*

# Clear DNS cache
systemd-resolve --flush-caches 2>/dev/null

# Overwrite free space
dd if=/dev/urandom of=/tmp/wipe bs=1M 2>/dev/null
sync
rm -f /tmp/wipe

# Clear memory (drop caches)
echo 3 > /proc/sys/vm/drop_caches
```

```powershell
# Windows Anti-Forensics Cleanup

# Clear all event logs
wevtutil el | ForEach-Object { wevtutil cl "$_" 2>$null }

# Clear PowerShell history
Remove-Item (Get-PSReadlineOption).HistorySavePath -Force -ErrorAction SilentlyContinue

# Clear Prefetch
Remove-Item "C:\Windows\Prefetch\*" -Force -ErrorAction SilentlyContinue

# Clear recent documents
Remove-Item "$env:APPDATA\Microsoft\Windows\Recent\*" -Force -ErrorAction SilentlyContinue

# Clear temp files
Remove-Item "$env:TEMP\*" -Recurse -Force -ErrorAction SilentlyContinue

# Clear DNS cache
ipconfig /flushdns

# Clear ARP cache
netsh interface ip delete arpcache

# Remove any created scheduled tasks
schtasks /delete /tn "BackdoorTask" /f 2>$null

# Disable and clear USN Journal
fsutil usn deletejournal /d C:
```

---

## Tools Used

| Tool | Purpose |
|---|---|
| **Metasploit (timestomp)** | File timestamp manipulation |
| **shred** | Secure file deletion on Linux |
| **wevtutil** | Windows Event Log manipulation |
| **SDelete (Sysinternals)** | Secure file deletion on Windows |
| **BleachBit** | Cross-platform evidence cleaning tool |
| **DBAN (Darik's Boot and Nuke)** | Full disk secure erasure |
| **Eraser** | Windows secure file deletion |
| **cipher /w** | Windows free space overwriting |
| **Custom Python Scripts** | Automated fund distribution |

---

## Commands Shown

### Timestomping

```bash
# Linux: Modify file timestamps
touch -t 201901011200.00 evil_script.py    # Modify access and modification time
touch -d "2019-01-01 12:00:00" evil_script.py

# Using debugfs to modify ext4 creation time (crtime)
debugfs -w /dev/sda1
> set_inode_field /path/to/file crtime 201901011200

# Meterpreter timestomping (see section above)
meterpreter > timestomp evil.exe -z "01/01/2019 12:00:00"
```

### Secure Wiping

```bash
# Linux: Secure delete with shred
shred -vfz -n 7 classified_document.pdf

# Linux: Overwrite free disk space
sfill -v /mount/point

# macOS: Secure empty trash (deprecated but historical)
srm -sz sensitive_file.txt

# Windows: SDelete for secure deletion
sdelete -p 3 -s C:\path\to\sensitive\files\

# Windows: Cipher to wipe free space
cipher /w:C:\

# DBAN: Boot from DBAN media, then:
# Select drive and choose "DoD 5220.22-M" or "Gutmann" method

# SSD Secure Erase via hdparm
hdparm --user-master u --security-set-pass password /dev/sdb
hdparm --user-master u --security-erase password /dev/sdb
```

### Log Manipulation

```bash
# Selective log editing (remove specific entries)
# Using sed to remove lines containing attacker IP
sed -i '/192\.168\.1\.100/d' /var/log/auth.log

# Remove entries from wtmp (login records)
utmpdump /var/log/wtmp | grep -v "attacker_user" > /tmp/clean_wtmp
utmpdump -r /tmp/clean_wtmp > /var/log/wtmp

# Clear specific user's lastlog entry
lastlog -u attacker_user  # View entry
# Then zero out with custom tool

# Windows: Remove specific event by Event ID
# (Requires custom tool - native tools only support full log clearing)
```

---

## Real-World Parallels

### Anti-Forensics in Real Attacks

- **Sony Pictures (2014):** The attackers (attributed to North Korea) used custom wiper malware called "Destover" that overwrote the Master Boot Record (MBR) and file contents of compromised systems, making forensic recovery extremely difficult.
- **Shamoon (2012, 2016, 2018):** Destructive malware targeting Saudi Aramco and other Middle Eastern organizations. Shamoon overwrote hard drives with images (initially a burning American flag, later a photo of the drowned Syrian refugee Alan Kurdi) to make a political statement while destroying data.
- **NotPetya (2017):** While disguised as ransomware, NotPetya was actually a wiper that permanently destroyed data. The encryption key was never stored, making data recovery impossible, even if the ransom was paid.
- **Olympic Destroyer (2018):** Malware deployed during the PyeongChang Winter Olympics that destroyed data and included sophisticated anti-forensic features, including false flag indicators designed to frame North Korea and China when it was actually attributed to Russia.

### Timestomping Detection

- **SANS Digital Forensics:** SANS Institute teaches timestomping detection through MFT analysis, comparing $STANDARD_INFORMATION and $FILE_NAME timestamps. When these diverge, it is a strong indicator of manipulation.
- **Mandiant APT Reports:** Multiple APT groups have been documented using timestomping as part of their TTPs (Tactics, Techniques, and Procedures), including APT28, APT29, and Lazarus Group.

### Secure Data Destruction Standards

- **NIST SP 800-88 (Guidelines for Media Sanitization):** The current US government standard for data destruction, which supersedes DoD 5220.22-M for most purposes. Recommends "Clear," "Purge," and "Destroy" levels depending on the sensitivity of the data.
- **GDPR Right to Erasure:** The European Union's General Data Protection Regulation requires organizations to securely delete personal data upon request, making secure wiping a legal requirement, not just a security practice.
- **Healthcare (HIPAA):** Healthcare organizations must follow specific data destruction procedures for protected health information (PHI).

### Wealth Redistribution and Hacktivism

- **Robin Hood Concept in Hacking:** While no real-world hacker has successfully redistributed stolen wealth on the scale depicted in Mr. Robot, the concept has deep roots in hacktivist ideology.
- **Anonymous Operations:** Hacktivist collective Anonymous has conducted operations targeting financial institutions, though these were disruptive (DDoS) rather than redistributive.
- **Phineas Fisher (2016):** The hacktivist who breached Hacking Team and Gamma Group claimed to have donated stolen money to the Kurdish Rojava revolution, representing one of the rare cases where a hacker explicitly redistributed stolen funds for ideological purposes.

## Tool Links

- [Metasploit](https://www.metasploit.com/) - File timestamp manipulation (timestomp)
- [SDelete](https://learn.microsoft.com/en-us/sysinternals/downloads/sdelete) - Secure file deletion on Windows
- [DBAN](https://dban.org/) - Full disk secure erasure
- [Volatility](https://www.volatilityfoundation.org/) - Memory forensics analysis (detection)
- [Autopsy](https://www.autopsy.com/) - Digital forensics platform (detection)
- [Wireshark](https://www.wireshark.org/) - Network traffic analysis (detection)
- [Digispark](http://digistump.com/products/1) - ATtiny85-based USB HID keystroke injection device
- [Arduino IDE](https://www.arduino.cc/en/software) - Programming environment for Digispark payloads

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| System log deletion | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| File timestomping | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| Secure data wiping (shred, SDelete) | T1485 | Data Destruction | https://attack.mitre.org/techniques/T1485/ |
| Evidence destruction | T1561 | Disk Wipe | https://attack.mitre.org/techniques/T1561/ |
| Automated redistribution via scripts | T1059 | Command and Scripting Interpreter | https://attack.mitre.org/techniques/T1059/ |
| Command history clearing | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| Persistence mechanism removal | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| Free disk space overwriting | T1485 | Data Destruction | https://attack.mitre.org/techniques/T1485/ |
| Digispark USB HID attack preparation | T1091 | Replication Through Removable Media | https://attack.mitre.org/techniques/T1091/ |

## References and Further Reading

- **Sony Pictures Hack (2014)** - Wiper malware "Destover" used for evidence destruction
- **Shamoon (2012, 2016, 2018)** - Destructive malware targeting Saudi Aramco
- **NotPetya (2017)** - Wiper disguised as ransomware with permanent data destruction
- **Olympic Destroyer (2018)** - Sophisticated anti-forensics with false flag attribution
- **NIST SP 800-88** - Guidelines for Media Sanitization
- **SANS Digital Forensics** - Timestomping detection via MFT analysis
- **Phineas Fisher (2016)** - Hacktivist who redistributed stolen funds
- **GDPR Right to Erasure** - Legal requirements for secure data deletion
- **BadUSB (SRLabs, Black Hat 2014)** - Fundamental USB firmware trust model vulnerability
- **FIN7 BadUSB Mailing Campaign (2021-2022)** - FBI flash alert about physical USB keystroke injection attacks using LilyGO BadUSB devices mailed to US businesses

## Search Tags

```
tags: [metasploit, sdelete, DBAN, anti-forensics, timestomping, log-deletion, secure-wiping, evidence-destruction, shred, wevtutil, wealth-redistribution, data-destruction, track-covering, hacktivism, digispark, ATtiny85, USB-HID, BadUSB, keystroke-injection]
season: 4
episode: 10
mitre: [T1070, T1485, T1561, T1059, T1091]
```
