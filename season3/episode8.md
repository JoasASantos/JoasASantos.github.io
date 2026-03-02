# Season 3, Episode 8: eps3.7_dont-delete-me.ko

## Episode Overview

A deeply personal episode for Elliot, grappling with survivor's guilt and the consequences of the Stage 2 attack. The `.ko` file extension (kernel object) references Linux loadable kernel modules -- code that operates at the deepest level of the operating system, much as this episode operates at the deepest emotional level of the series. The technical themes explore kernel-level rootkits (the deepest form of system compromise), digital identity and fingerprinting (the question of who we really are online), and data recovery (the possibility of retrieving what was thought to be permanently lost).

---

## Hacks & Techniques

### Linux Kernel Object Files (.ko)

A `.ko` file is a **loadable kernel module (LKM)** in Linux. Kernel modules extend the kernel's functionality at runtime without requiring a reboot. They operate in **kernel space** (ring 0), the most privileged execution level:

```
Privilege Levels (x86):
Ring 3: User applications (least privilege)
Ring 2: Device drivers (rarely used in modern OS)
Ring 1: Device drivers (rarely used in modern OS)
Ring 0: Kernel (maximum privilege) <-- .ko modules run here
Ring -1: Hypervisor (if virtualized)
Ring -2: SMM (System Management Mode)
```

**Legitimate kernel module operations:**

```bash
# List currently loaded kernel modules
lsmod

# Get information about a module
modinfo <module_name>

# Load a kernel module
sudo insmod /path/to/module.ko
# or
sudo modprobe <module_name>

# Remove a kernel module
sudo rmmod <module_name>

# View kernel module dependencies
modprobe --show-depends <module_name>

# View kernel log for module load/unload messages
dmesg | grep -i "module"
```

### Kernel Rootkits

A **kernel rootkit** is malicious code that operates at ring 0 (kernel level), giving it complete control over the operating system. Because it runs with the same privileges as the kernel itself, a kernel rootkit can hide its presence from any user-space security tool.

**Kernel rootkit capabilities:**

1. **Syscall hooking**: Intercept and modify system call behavior
2. **File hiding**: Make files invisible to all user-space tools (ls, find, etc.)
3. **Process hiding**: Hide malicious processes from process listings (ps, top, /proc)
4. **Network connection hiding**: Conceal network connections from netstat, ss
5. **Log tampering**: Modify or suppress kernel log messages
6. **Credential theft**: Intercept keystrokes and authentication credentials
7. **Backdoor access**: Provide persistent remote access that survives reboots

**Syscall hooking technique:**

```c
// Simplified kernel rootkit: syscall table hooking
// This intercepts the getdents64 syscall to hide files

#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/syscalls.h>

// Original syscall pointer
static asmlinkage long (*original_getdents64)(unsigned int fd,
    struct linux_dirent64 __user *dirent, unsigned int count);

// Hooked version - filters out entries matching hidden filename
static asmlinkage long hooked_getdents64(unsigned int fd,
    struct linux_dirent64 __user *dirent, unsigned int count) {

    long ret = original_getdents64(fd, dirent, count);

    // Iterate through directory entries and remove hidden ones
    // (Simplified - actual implementation handles kernel/user memory carefully)
    struct linux_dirent64 *d;
    long offset = 0;

    while (offset < ret) {
        d = (struct linux_dirent64 *)((char *)dirent + offset);
        if (strstr(d->d_name, "HIDDEN_PREFIX") != NULL) {
            // Remove this entry by shifting subsequent entries
            memmove(d, (char *)d + d->d_reclen, ret - offset - d->d_reclen);
            ret -= d->d_reclen;
        } else {
            offset += d->d_reclen;
        }
    }
    return ret;
}

// Module initialization - hook the syscall
static int __init rootkit_init(void) {
    // Locate syscall table (various techniques exist)
    // Replace getdents64 entry with hooked version
    // Disable write protection on syscall table page
    // ...
    return 0;
}

// Module cleanup
static void __exit rootkit_exit(void) {
    // Restore original syscall
}

module_init(rootkit_init);
module_exit(rootkit_exit);
MODULE_LICENSE("GPL");  // Ironically, rootkits often claim GPL to avoid tainted kernel warnings
```

**Direct Kernel Object Manipulation (DKOM):**

DKOM is a more sophisticated rootkit technique that directly modifies kernel data structures rather than hooking function pointers:

```
Process Hiding via DKOM:

Linux Kernel Task List (doubly-linked list):
init_task <--> process_A <--> [HIDDEN_PROCESS] <--> process_B <--> ...

After DKOM:
init_task <--> process_A <--> process_B <--> ...
                                ^
                                |
                    [HIDDEN_PROCESS still runs but is
                     unlinked from the task list]

Result: ps, top, /proc/* all iterate the task list
        and will never see the hidden process
```

```c
// DKOM process hiding (conceptual)
// Remove a task from the kernel's task list

void hide_process(pid_t target_pid) {
    struct task_struct *task;

    for_each_process(task) {
        if (task->pid == target_pid) {
            // Unlink from task list (process still executes)
            list_del(&task->tasks);

            // Also hide from PID hash table
            // detach_pid(task, PIDTYPE_PID);

            printk(KERN_INFO "Process %d hidden\n", target_pid);
            break;
        }
    }
}
```

**Detecting kernel rootkits:**

```bash
# Check for modified syscall table
# Compare runtime syscall addresses against System.map
cat /boot/System.map-$(uname -r) | grep sys_call_table
cat /proc/kallsyms | grep sys_call_table

# Check for hidden kernel modules
# Compare lsmod output against /sys/module/
ls /sys/module/ | wc -l
lsmod | wc -l
# Discrepancy may indicate hidden modules

# Use rkhunter (Rootkit Hunter)
sudo rkhunter --check --sk

# Use chkrootkit
sudo chkrootkit

# Verify kernel integrity
# Compare running kernel against known-good hash
sha256sum /boot/vmlinuz-$(uname -r)

# Check for unexpected kernel memory modifications
# (Requires specialized forensic tools like Volatility)
vol.py -f memory.lime --profile=LinuxProfile linux_check_syscall
vol.py -f memory.lime --profile=LinuxProfile linux_hidden_modules
vol.py -f memory.lime --profile=LinuxProfile linux_check_creds
```

### Digital Identity and Browser Fingerprinting

The episode explores the concept of digital identity -- how individuals are identified and tracked online even without traditional authentication:

**Browser fingerprinting components:**

```javascript
// Browser fingerprinting collects multiple attributes to create a unique identifier

fingerprint = {
    // Hardware
    screenResolution: screen.width + "x" + screen.height,
    colorDepth: screen.colorDepth,
    cpuCores: navigator.hardwareConcurrency,
    deviceMemory: navigator.deviceMemory,
    gpuRenderer: getWebGLRenderer(),  // Canvas/WebGL fingerprint

    // Software
    userAgent: navigator.userAgent,
    language: navigator.language,
    timezone: Intl.DateTimeFormat().resolvedOptions().timeZone,
    platform: navigator.platform,

    // Browser configuration
    cookiesEnabled: navigator.cookieEnabled,
    doNotTrack: navigator.doNotTrack,
    plugins: getPluginList(),
    mimeTypes: getMimeTypes(),

    // Behavioral
    canvasFingerprint: getCanvasHash(),      // Draw hidden canvas, hash result
    webglFingerprint: getWebGLHash(),        // Render 3D scene, hash result
    audioFingerprint: getAudioContextHash(), // Process audio signal, hash result
    fontList: getAvailableFonts(),           // Detect installed fonts

    // Network
    publicIP: getPublicIP(),
    localIP: getWebRTCLocalIP(),  // WebRTC can leak local IP even through VPN
    connectionType: navigator.connection?.type
};

// Combined fingerprint entropy can uniquely identify ~94% of browsers
// (Per EFF's Panopticlick/Cover Your Tracks research)
```

**Fingerprinting countermeasures:**

```bash
# Tor Browser: Designed to make all users look identical
# - Standardized window size
# - Disabled WebGL, restricted canvas access
# - Uniform user agent string
# - No plugins

# Firefox privacy settings (about:config)
# privacy.resistFingerprinting = true  (enables fingerprinting protection)
# webgl.disabled = true
# media.peerconnection.enabled = false  (disables WebRTC IP leak)
# dom.battery.enabled = false

# Browser extensions
# - uBlock Origin (blocks tracking scripts)
# - Canvas Blocker (randomizes canvas fingerprint)
# - NoScript (blocks JavaScript-based fingerprinting)
```

### File Carving and Data Recovery

The episode's title "dont-delete-me" connects to the theme of data that persists even after deletion. File carving recovers files from raw disk data based on file signatures rather than filesystem metadata:

**How file carving works:**

```
Disk after file "deletion":
[MBR][File A (allocated)][Deleted File B (unallocated but data remains)][File C (allocated)][Free Space]

File system says: "File B doesn't exist" (directory entry removed)
File carving says: "I see JPEG header (FF D8 FF) at offset 0x50000 and footer (FF D9) at offset 0x5F000"
Result: File B recovered regardless of filesystem state
```

**File carving tools:**

```bash
# Autopsy (GUI frontend for The Sleuth Kit)
# Open-source digital forensics platform
autopsy
# Web-based interface for case management, timeline analysis, keyword search

# The Sleuth Kit (command-line forensic tools)
# List filesystem contents (including deleted files)
fls -r -d /evidence/disk_image.raw
# -r: recursive
# -d: show deleted entries only

# Recover a specific file by inode
icat /evidence/disk_image.raw <inode_number> > recovered_file.dat

# File carving with Scalpel
# Edit /etc/scalpel/scalpel.conf to enable desired file types
scalpel /evidence/disk_image.raw -o /evidence/carved_files/
# Carves files based on header/footer signatures

# File carving with PhotoRec
photorec /evidence/disk_image.raw
# Interactive tool that recovers files from disk images
# Supports over 480 file formats

# Foremost - another file carving tool
foremost -t all -i /evidence/disk_image.raw -o /evidence/foremost_output/

# Bulk_extractor - extracts features without parsing filesystem
bulk_extractor -o /evidence/bulk_output/ /evidence/disk_image.raw
# Extracts emails, URLs, credit card numbers, etc. from raw data
```

**The Sleuth Kit analysis pipeline:**

```bash
# Step 1: Identify partitions
mmls /evidence/disk_image.raw
# Output:
# 000:  Meta    0000000000   0000000000   0000000001   Primary Table (#0)
# 001:  -----   0000000000   0000002047   0000002048   Unallocated
# 002:  000:00  0000002048   0001953791   0001951744   Linux (0x83)

# Step 2: Identify filesystem type
fsstat -o 2048 /evidence/disk_image.raw

# Step 3: List all files (including deleted)
fls -r -o 2048 /evidence/disk_image.raw > file_listing.txt

# Step 4: Create a timeline
fls -r -m "/" -o 2048 /evidence/disk_image.raw > bodyfile.txt
mactime -b bodyfile.txt -d > timeline.csv

# Step 5: Recover specific deleted files
icat -o 2048 /evidence/disk_image.raw 12345 > recovered_document.pdf

# Step 6: Search for strings across unallocated space
blkls -o 2048 /evidence/disk_image.raw | strings > unallocated_strings.txt
```

**Recovering deleted files from ext4 filesystem:**

```bash
# Using extundelete for ext4 recovery
extundelete /dev/sda1 --restore-all --output-dir /recovery/

# Using debugfs for ext4
debugfs /dev/sda1
# debugfs: ls -d    (list deleted files)
# debugfs: logdump -i <inode>  (check journal for inode)
# debugfs: dump <inode> /recovery/recovered_file

# Journal analysis for recently deleted files
# ext4 journals may contain copies of recently modified inodes
jls /evidence/disk_image.raw
jcat /evidence/disk_image.raw <journal_block>
```

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **insmod / modprobe** | Linux kernel module loading |
| **rkhunter** | Rootkit detection scanner |
| **chkrootkit** | Rootkit detection tool |
| **Volatility** | Memory forensics framework for rootkit detection |
| **Autopsy** | GUI digital forensics platform |
| **The Sleuth Kit (TSK)** | Command-line forensic analysis tools |
| **PhotoRec** | File carving and recovery tool |
| **Scalpel** | High-performance file carving tool |
| **Foremost** | File carving based on headers/footers |
| **bulk_extractor** | Feature extraction from disk images |
| **extundelete** | ext4 filesystem file recovery |
| **Tor Browser** | Anti-fingerprinting web browser |

---

## Commands Shown

```bash
# Kernel module operations
lsmod | grep suspicious_module
modinfo suspicious_module
sudo rmmod suspicious_module

# Rootkit detection
sudo rkhunter --update && sudo rkhunter --check
sudo chkrootkit
cat /proc/modules | sort > loaded_modules.txt
ls /sys/module/ | sort > sysfs_modules.txt
diff loaded_modules.txt sysfs_modules.txt

# File recovery with The Sleuth Kit
mmls disk_image.raw
fls -r -d -o 2048 disk_image.raw
icat -o 2048 disk_image.raw <inode> > recovered_file

# File carving
scalpel disk_image.raw -o carved_output/
photorec disk_image.raw
foremost -t doc,pdf,jpg,png -i disk_image.raw -o foremost_output/

# Memory forensics for rootkit detection
vol.py -f memory.dump --profile=LinuxProfile linux_check_syscall
vol.py -f memory.dump --profile=LinuxProfile linux_hidden_modules
vol.py -f memory.dump --profile=LinuxProfile linux_pslist
vol.py -f memory.dump --profile=LinuxProfile linux_psxview  # Cross-reference process lists

# Browser fingerprint checking
# Visit https://coveryourtracks.eff.org/ (EFF's fingerprinting test)
# Visit https://browserleaks.com/ (comprehensive fingerprint test)
```

---

## Real-World Parallels

### Kernel Rootkits
- **Rustock rootkit (2006-2011)**: One of the most sophisticated kernel rootkits ever deployed, Rustock infected an estimated 1-2 million machines and operated the world's largest spam botnet. It used advanced DKOM techniques to hide from all user-space detection tools and was only disrupted by a coordinated takedown led by Microsoft.
- **Azazel rootkit**: Open-source userland rootkit that hooks libc functions to hide files, processes, and network connections. While not kernel-level, it demonstrates the concept of syscall-level hiding.
- **Diamorphine**: A modern, open-source Linux kernel rootkit used for educational purposes that demonstrates syscall hooking, process hiding, and module hiding techniques.
- **Hacking Team rootkits**: The Italian surveillance company's products included kernel-level rootkits for both Windows and Linux, leaked in the 2015 breach.

### Browser Fingerprinting
- **EFF Panopticlick/Cover Your Tracks**: The Electronic Frontier Foundation's research demonstrated that browser fingerprinting can uniquely identify 94% of browsers without using cookies, using only passively collected configuration data.
- **FingerprintJS**: A commercial fingerprinting library used by thousands of websites for fraud detection, demonstrating the widespread deployment of fingerprinting technology.
- **Apple ITP (Intelligent Tracking Prevention)**: Safari's anti-tracking features represent the browser industry's response to fingerprinting, though researchers continue to find new fingerprinting vectors.

### Data Recovery and File Carving
- **BTK Killer case (2005)**: Dennis Rader was identified after metadata in a floppy disk he sent to police was recovered using forensic tools, revealing his identity. This case demonstrated the power of digital forensics.
- **Enron investigation (2001)**: Forensic teams recovered millions of deleted emails from Enron servers, providing crucial evidence for prosecution. File carving and deleted data recovery were essential techniques.
- **Silk Road investigation (2013)**: FBI forensic teams recovered deleted files from Ross Ulbricht's laptop, including a journal documenting the creation and operation of the Silk Road marketplace. Data "deletion" did not prevent recovery.
- **NIST CFTT (Computer Forensics Tool Testing)**: NIST validates forensic tools including file carving software to ensure they produce reliable results admissible in court.

## Tool Links

- [Volatility](https://www.volatilityfoundation.org/) - Memory forensics framework for rootkit detection
- [Autopsy](https://www.autopsy.com/) - Digital forensics platform with graphical interface
- [Sleuth Kit](https://www.sleuthkit.org/) - Command-line forensics tools
- [Foremost](https://foremost.sourceforge.net/) - File carving based on file headers/footers
- [Binwalk](https://github.com/ReFirmLabs/binwalk) - Analysis and extraction of data embedded in images
- [YARA](https://virustotal.github.io/yara/) - Detection rules for rootkit and malware identification
- [Tor](https://www.torproject.org/) - Anti-fingerprinting browser for digital identity protection
- [Wireshark](https://www.wireshark.org/) - Network traffic analysis for hidden connection detection

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Kernel rootkit via loadable module | T1014 | Rootkit | https://attack.mitre.org/techniques/T1014/ |
| Syscall hooking to hide files | T1564 | Hide Artifacts | https://attack.mitre.org/techniques/T1564/ |
| DKOM to hide processes | T1564.001 | Hidden Files and Directories | https://attack.mitre.org/techniques/T1564/001/ |
| Persistence via kernel module | T1547 | Boot or Logon Autostart Execution | https://attack.mitre.org/techniques/T1547/ |
| Credential capture via kernel keylogger | T1056 | Input Capture | https://attack.mitre.org/techniques/T1056/ |
| Browser fingerprinting for tracking | T1082 | System Information Discovery | https://attack.mitre.org/techniques/T1082/ |
| Deleted data recovery (file carving) | T1005 | Data from Local System | https://attack.mitre.org/techniques/T1005/ |
| Kernel log tampering | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |

## References and Further Reading

- **Rustock Rootkit Analysis (Microsoft, 2011)**: Analysis of the most sophisticated kernel rootkit ever deployed, infecting 1-2 million machines
- **Diamorphine Rootkit (GitHub)**: Open-source Linux kernel rootkit for educational purposes demonstrating syscall hooking
- **Hacking Team Leak (2015)**: Leak revealing kernel rootkits for Windows and Linux used for surveillance
- **EFF Panopticlick/Cover Your Tracks**: Research demonstrating that fingerprinting can identify 94% of browsers without cookies
- **BTK Killer Digital Forensics Case (2005)**: Criminal case solved through metadata recovered from a floppy disk
- **Silk Road Investigation (FBI, 2013)**: Recovery of deleted files from Ross Ulbricht's laptop as crucial evidence
- **NIST CFTT (Computer Forensics Tool Testing)**: Forensic tool validation program for court admissibility
- **NIST SP 800-86**: Guide to Integrating Forensic Techniques into Incident Response

## Search Tags

```
tags: [kernel-rootkit, LKM, syscall-hooking, DKOM, browser-fingerprinting, file-carving, data-recovery, Volatility, Autopsy, Sleuth-Kit, Foremost, PhotoRec, rkhunter, chkrootkit, Tor-Browser]
season: 3
episode: 8
mitre: [T1014, T1564, T1564.001, T1547, T1056, T1082, T1005, T1070]
```
