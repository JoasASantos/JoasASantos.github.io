# Episode 12: eps4.11_412preconditionfailed.h

## Season 4, Episode 12 | "412 Precondition Failed"

**Air Date:** December 22, 2019
**Directed by:** Sam Esmail

### Overview

The episode title references HTTP status code 412, which indicates that a precondition in the request headers was evaluated to false by the server. The client set expectations (preconditions) that the server could not meet. This mirrors the episode's themes of identity, failed assumptions, and revelations. The core technical motif of this episode is the `whoami` command, a fundamental system command that asks the most basic question: "Who am I?" This question resonates deeply with Elliot's journey throughout the entire series.

---

## Hacks & Techniques

### 1. whoami Command - Linux and Windows

The `whoami` command is one of the simplest yet most important commands in both Linux and Windows. It returns the current user's identity, answering the fundamental question: who is the user currently executing commands on this system?

#### Linux whoami

```bash
$ whoami
elliot
```

On Linux, `whoami` prints the effective username associated with the current user ID. It is equivalent to `id -un`.

#### Windows whoami

```cmd
C:\> whoami
ECORP\elliot.alderson
```

On Windows, `whoami` displays the domain and username of the currently logged-in user.

### 2. whoami /all and Related Commands

The `/all` flag on Windows provides comprehensive identity information including user name, SID, group memberships, privileges, and logon ID.

#### whoami /all (Windows)

```cmd
C:\> whoami /all

USER INFORMATION
----------------
User Name           SID
=================== =============================================
ecorp\elliot.alderson S-1-5-21-3623811015-3361044348-30300820-1013

GROUP INFORMATION
-----------------
Group Name                           Type      SID                      Attributes
==================================== ========= ======================== ============================
Everyone                             Well-known S-1-1-0                  Mandatory, Enabled by default
BUILTIN\Administrators               Alias      S-1-5-32-544            Mandatory, Enabled by default
BUILTIN\Users                        Alias      S-1-5-32-545            Mandatory, Enabled by default
NT AUTHORITY\INTERACTIVE             Well-known S-1-5-4                  Mandatory, Enabled by default
NT AUTHORITY\Authenticated Users     Well-known S-1-5-11                 Mandatory, Enabled by default
ecorp\Domain Admins                  Group      S-1-5-21-...-512        Mandatory, Enabled by default
ecorp\IT-Security                    Group      S-1-5-21-...-1105       Mandatory, Enabled by default

PRIVILEGES INFORMATION
----------------------
Privilege Name                Description                        State
============================= ================================== ========
SeDebugPrivilege              Debug programs                     Enabled
SeBackupPrivilege             Back up files and directories      Enabled
SeRestorePrivilege            Restore files and directories      Enabled
SeShutdownPrivilege           Shut down the system               Enabled
SeChangeNotifyPrivilege       Bypass traverse checking           Enabled
SeRemoteShutdownPrivilege     Force shutdown from remote system  Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set     Enabled
```

#### Other whoami Flags (Windows)

```cmd
:: Display user name and SID only
whoami /user

:: Display group memberships only
whoami /groups

:: Display current privileges
whoami /priv

:: Display logon ID
whoami /logonid

:: Output in list format
whoami /all /fo list

:: Output in CSV format
whoami /all /fo csv
```

### 3. id Command (Linux)

The Linux `id` command provides more detailed identity information than `whoami`:

```bash
$ id
uid=1000(elliot) gid=1000(elliot) groups=1000(elliot),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),116(lxd)

# Display specific user's information
$ id root
uid=0(root) gid=0(root) groups=0(root)

# Display only the effective user ID
$ id -u
1000

# Display only the effective user name
$ id -un
elliot

# Display only the effective group ID
$ id -g
1000

# Display all group IDs
$ id -G
1000 4 24 27 30 46 116

# Display all group names
$ id -Gn
elliot adm cdrom sudo dip plugdev lxd
```

### 4. groups Command (Linux)

```bash
$ groups
elliot adm cdrom sudo dip plugdev lxd

$ groups root
root : root

$ groups elliot
elliot : elliot adm cdrom sudo dip plugdev lxd
```

### 5. System Access Management Concepts

The `whoami` command connects to broader system access management concepts that are foundational to security.

#### Identity and Access Management (IAM)

```
AUTHENTICATION (AuthN): "Who are you?"
+------------------------------------------+
| Methods:                                  |
| - Username/Password                       |
| - Certificates (X.509)                   |
| - Biometrics (fingerprint, face, iris)   |
| - Hardware tokens (YubiKey, SmartCard)   |
| - Multi-factor authentication (MFA)     |
+------------------------------------------+
              |
              v
AUTHORIZATION (AuthZ): "What can you do?"
+------------------------------------------+
| Models:                                   |
| - DAC (Discretionary Access Control)     |
| - MAC (Mandatory Access Control)         |
| - RBAC (Role-Based Access Control)       |
| - ABAC (Attribute-Based Access Control)  |
+------------------------------------------+
              |
              v
ACCOUNTING: "What did you do?"
+------------------------------------------+
| Mechanisms:                               |
| - Audit logs                              |
| - Session recording                       |
| - Activity monitoring                     |
| - SIEM integration                        |
+------------------------------------------+
```

#### Privilege Escalation Context

Understanding who you are is the first step in determining what escalation is needed:

```
Low-Privilege User        Standard User            Administrator/Root
(Guest, Service Account)  (Domain User)            (Full Control)
         |                      |                        |
    whoami reveals:        whoami reveals:           whoami reveals:
    Limited groups         Standard groups           Admin groups
    No special privs       Basic privileges          All privileges
         |                      |                        |
    Need: Kernel exploit   Need: UAC bypass,         Goal achieved:
    or service exploit     Token manipulation,       Full system
                           or misconfiguration       control
```

#### Windows Security Identifiers (SIDs)

Understanding SIDs is critical for access management:

| SID | Account |
|---|---|
| `S-1-5-18` | LOCAL SYSTEM |
| `S-1-5-19` | LOCAL SERVICE |
| `S-1-5-20` | NETWORK SERVICE |
| `S-1-5-32-544` | BUILTIN\Administrators |
| `S-1-5-32-545` | BUILTIN\Users |
| `S-1-5-21-...-500` | Domain Administrator |
| `S-1-5-21-...-512` | Domain Admins group |
| `S-1-5-21-...-519` | Enterprise Admins group |

### 6. Process Termination and System Shutdown

The episode's themes of ending and finality connect to process and system lifecycle management.

#### Process Management Commands

```bash
# Linux: List running processes
ps aux
ps -ef

# Find specific process
ps aux | grep process_name
pgrep -l process_name

# Get process tree
pstree -p

# Send signals to processes
kill PID           # Send SIGTERM (graceful shutdown request)
kill -9 PID        # Send SIGKILL (force termination, cannot be caught)
kill -15 PID       # Send SIGTERM explicitly
kill -STOP PID     # Pause process
kill -CONT PID     # Resume process

# Kill by name
killall process_name
pkill process_name

# Kill all processes for a user
pkill -u username
```

```powershell
# Windows: List running processes
Get-Process
tasklist

# Find specific process
Get-Process -Name "process_name"
tasklist /FI "IMAGENAME eq process_name.exe"

# Terminate process
Stop-Process -Name "process_name" -Force
taskkill /IM process_name.exe /F
taskkill /PID 1234 /F

# Terminate process and all children
taskkill /IM process_name.exe /T /F
```

#### System Shutdown Commands

```bash
# Linux: Shutdown
shutdown -h now        # Halt immediately
shutdown -h +10        # Shutdown in 10 minutes
shutdown -r now        # Reboot immediately
poweroff               # Power off immediately
reboot                 # Reboot immediately
init 0                 # Switch to runlevel 0 (halt)
init 6                 # Switch to runlevel 6 (reboot)
systemctl poweroff     # systemd power off
systemctl reboot       # systemd reboot
```

```powershell
# Windows: Shutdown
shutdown /s /t 0       # Shutdown immediately
shutdown /r /t 0       # Reboot immediately
shutdown /l            # Log off
shutdown /s /t 300     # Shutdown in 5 minutes
shutdown /a            # Abort scheduled shutdown
Stop-Computer          # PowerShell shutdown
Restart-Computer       # PowerShell reboot
```

---

## Tools Used

| Tool | Purpose |
|---|---|
| **whoami** | User identity verification (Linux/Windows) |
| **id** | Detailed user and group identity (Linux) |
| **groups** | Group membership enumeration (Linux) |
| **net user / net group** | Windows user and group management |
| **Get-ADUser** | Active Directory user enumeration (PowerShell) |
| **BloodHound** | Active Directory attack path visualization |
| **PowerView** | Active Directory enumeration from compromised host |

---

## Commands Shown

### Identity Enumeration (Post-Exploitation Context)

```bash
# First commands after gaining access to a system:

# Linux
whoami                    # Who am I?
id                        # Full identity info
hostname                  # What system am I on?
uname -a                  # OS information
cat /etc/passwd           # All users on system
cat /etc/group            # All groups on system
sudo -l                   # What can I run as sudo?
cat /etc/sudoers          # Full sudo configuration
ls -la /home/             # Home directories (users)
last                      # Recent logins
w                         # Currently logged in users

# Windows
whoami /all               # Full identity info
hostname                  # System name
systeminfo                # OS and patch information
net user                  # All local users
net localgroup administrators  # Local admin group members
net group /domain         # Domain groups
net group "Domain Admins" /domain  # Domain admin members
net user elliot.alderson /domain   # Specific user details
cmdkey /list              # Stored credentials
```

### Active Directory Identity Enumeration

```powershell
# PowerShell AD Module
Get-ADUser -Identity "elliot.alderson" -Properties *
Get-ADUser -Filter * -Properties MemberOf | Select-Object Name, MemberOf
Get-ADGroupMember -Identity "Domain Admins"
Get-ADPrincipalGroupMembership -Identity "elliot.alderson"

# PowerView (PowerSploit)
Import-Module PowerView.ps1
Get-DomainUser -Identity "elliot.alderson"
Get-DomainGroup -MemberIdentity "elliot.alderson"
Find-DomainUserLocation
Get-DomainGroupMember -Identity "Domain Admins"

# BloodHound data collection
Import-Module SharpHound.ps1
Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\temp\
# Then import into BloodHound GUI for attack path visualization
```

### Signal Handling and Process Lifecycle

```bash
# Linux signals reference
kill -l    # List all signals

# Key signals:
# SIGHUP  (1)  - Hangup / reload configuration
# SIGINT  (2)  - Interrupt (Ctrl+C)
# SIGQUIT (3)  - Quit with core dump
# SIGKILL (9)  - Force kill (cannot be caught or ignored)
# SIGTERM (15) - Graceful termination request
# SIGSTOP (19) - Pause process (cannot be caught)
# SIGCONT (18) - Continue paused process

# Process lifecycle
# Start -> Running -> (Signal) -> Terminating -> Zombie -> Reaped
```

---

## Real-World Parallels

### Identity in Cybersecurity

- **Zero Trust Architecture:** Modern security frameworks (NIST SP 800-207) are built around the principle of "never trust, always verify." Every access request requires identity verification, regardless of network location. The `whoami` question is asked continuously, not just at login.
- **Identity-Based Attacks:** The most devastating breaches often involve identity compromise. The SolarWinds attack used forged SAML tokens (Golden SAML) to impersonate any user, essentially answering "whoami" with any identity the attacker chose.
- **Kerberoasting:** Attackers request Kerberos service tickets for accounts with SPNs and crack them offline. The goal is to become someone else, to change the answer to `whoami`.

### Privilege Escalation in Real Attacks

- **Dirty COW (CVE-2016-5376):** A Linux kernel race condition that allowed any local user to gain root privileges, changing the answer to `whoami` from a regular user to `root`.
- **PrintNightmare (CVE-2021-34527):** A Windows Print Spooler vulnerability that allowed authenticated users to execute code as SYSTEM, the highest-privilege Windows identity.
- **UAC Bypass Techniques:** Numerous methods exist to bypass Windows User Account Control, escalating from a standard user to an administrator without triggering the UAC prompt.

### The Philosophical "Who Am I?" in Hacking Culture

- **Hacker Pseudonyms:** The hacking community has a long tradition of identity exploration through handles and aliases (Kevin Mitnick was "Condor," Kevin Poulsen was "Dark Dante").
- **Anonymous:** The hacktivist collective deliberately built an identity based on anonymity, with the Guy Fawkes mask symbolizing the rejection of individual identity in favor of collective action.
- **Mr. Robot's Core Theme:** The entire series is built around the question of identity. Elliot's `whoami` moment is not just technical but existential, representing the ultimate convergence of the show's technical and narrative themes.

### Process Termination in Security

- **Kill Switch:** The concept of forcefully terminating a process (kill -9) is analogous to the kill switches used in malware emergencies. Marcus Hutchins' WannaCry kill switch was effectively a `kill -9` for a global malware outbreak.
- **Emergency System Shutdown:** Nuclear facilities, power plants, and other critical infrastructure have physical and digital emergency shutdown procedures (SCRAM buttons for reactors, Emergency Stop for industrial machinery) that parallel the `shutdown` and `kill` commands.

## Tool Links

- [BloodHound](https://github.com/BloodHoundAD/BloodHound) - Active Directory attack path visualization
- [PowerSploit](https://github.com/PowerShellMafia/PowerSploit) - Active Directory enumeration (PowerView)
- [Mimikatz](https://github.com/gentilkiwi/mimikatz) - Credential extraction and identity manipulation
- [Impacket](https://github.com/fortra/impacket) - Python tools for enumeration and lateral movement
- [Metasploit](https://www.metasploit.com/) - Exploitation and privilege escalation framework
- [Nmap](https://nmap.org/) - Network scanning and service enumeration

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Identity enumeration (whoami, id) | T1087 | Account Discovery | https://attack.mitre.org/techniques/T1087/ |
| Group and privilege discovery | T1087 | Account Discovery | https://attack.mitre.org/techniques/T1087/ |
| Active Directory enumeration (BloodHound) | T1087 | Account Discovery | https://attack.mitre.org/techniques/T1087/ |
| Privilege escalation | T1068 | Exploitation for Privilege Escalation | https://attack.mitre.org/techniques/T1068/ |
| UAC bypass | T1548 | Abuse Elevation Control Mechanism | https://attack.mitre.org/techniques/T1548/ |
| System information discovery | T1082 | System Information Discovery | https://attack.mitre.org/techniques/T1082/ |
| Process discovery | T1057 | Process Discovery | https://attack.mitre.org/techniques/T1057/ |
| System shutdown/reboot | T1529 | System Shutdown/Reboot | https://attack.mitre.org/techniques/T1529/ |

## References and Further Reading

- NIST SP 800-207 - Zero Trust Architecture: continuous identity verification
- SolarWinds SUNBURST (2020) - Golden SAML for impersonation of any user
- CVE-2016-5376 (Dirty COW) - Local user to root privilege escalation on Linux
- CVE-2021-34527 (PrintNightmare) - Code execution as SYSTEM on Windows
- Kerberoasting - Kerberos ticket attack for identity compromise
- Kevin Mitnick ("Condor") - Identity exploration in hacker culture
- Anonymous (hacktivism) - Collective identity and anonymity
- SANS Institute - Post-exploitation identity enumeration best practices

## Search Tags

```
tags: [bloodhound, powersploit, mimikatz, impacket, whoami, identity, privilege-escalation, active-directory, UAC-bypass, process-management, system-shutdown, SID, kerberos, post-exploitation]
season: 4
episode: 12
mitre: [T1087, T1068, T1548, T1082, T1057, T1529]
```
