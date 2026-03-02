# Episode 1: eps4.0_401unauthorized.h

## Season 4, Episode 1 | "401 Unauthorized"

**Air Date:** October 6, 2019
**Directed by:** Sam Esmail

### Overview

The Season 4 premiere opens with Elliot intensifying his campaign against the Deus Group, the shadowy cabal of elites who control the world's financial systems. The episode title references HTTP status code 401, signaling that Elliot is attempting to access systems and information for which he lacks authorization. Elliot begins meticulous reconnaissance on Deus Group members while navigating increasingly dangerous territory with the Dark Army.

---

## Hacks & Techniques

### 1. HTTP 401 Unauthorized Status Code Reference

The episode title is a direct reference to the HTTP 401 status code, which indicates that the request lacks valid authentication credentials for the target resource. This frames the entire episode thematically: Elliot is an unauthorized actor attempting to gain access to the most protected systems and people in the world.

- **401 vs 403:** A 401 response means "you have not authenticated," while 403 means "you have authenticated but lack permission." The distinction is important: Elliot has not yet established any foothold.
- **WWW-Authenticate Header:** When a server returns 401, it typically includes a `WWW-Authenticate` header specifying the authentication scheme (Basic, Bearer, Digest, etc.).

### 2. Virtual Machine Detection and Escape

Elliot demonstrates awareness of virtual machine detection techniques, recognizing when a system may be running inside a sandboxed or virtualized environment. This is critical for both offensive operators (who want to detect analysis environments) and defenders (who use VMs to analyze malware).

#### VM-Specific MAC Address Detection

Virtual machines use specific MAC address prefixes assigned to hypervisor vendors:

| Hypervisor | MAC Prefix (OUI) |
|---|---|
| VMware | `00:0C:29`, `00:50:56` |
| VirtualBox | `08:00:27` |
| Hyper-V | `00:15:5D` |
| Parallels | `00:1C:42` |
| QEMU/KVM | `52:54:00` |

#### Registry Key Detection (Windows)

Malware and red teamers check for VM-related registry keys:

- `HKLM\SOFTWARE\VMware, Inc.\VMware Tools`
- `HKLM\SOFTWARE\Oracle\VirtualBox Guest Additions`
- `HKLM\SYSTEM\CurrentControlSet\Services\VBoxGuest`
- `HKLM\HARDWARE\DEVICEMAP\Scsi\Scsi Port 0\Scsi Bus 0\Target Id 0\Logical Unit Id 0` (Identifier containing "VBOX" or "VMWARE")

#### CPUID-Based Detection

The x86 `CPUID` instruction can reveal hypervisor presence:

- **CPUID Leaf 0x1:** Bit 31 of ECX indicates hypervisor presence (Hypervisor Present Bit).
- **CPUID Leaf 0x40000000:** Returns the hypervisor vendor string (e.g., `VMwareVMware`, `Microsoft Hv`, `KVMKVMKVM`).

#### Pafish (Paranoid Fish)

Pafish is an open-source tool that employs multiple VM and sandbox detection techniques:

- Timing attacks (RDTSC instruction timing differences)
- Process enumeration (checking for `vmtoolsd.exe`, `VBoxService.exe`)
- Device driver checks
- Firmware SMBIOS string analysis
- Mouse movement detection (sandboxes often lack real user input)

### 3. OSINT Reconnaissance on Deus Group Members

Elliot conducts extensive Open Source Intelligence (OSINT) gathering on Deus Group members, specifically targeting **Freddy Lomax**, to build profiles that will enable social engineering attacks.

#### OSINT Methodology Applied

1. **Target Identification:** Enumerate all known Deus Group members from available intelligence.
2. **Digital Footprinting:** Search for public records, social media profiles, corporate filings, property records, and financial disclosures.
3. **Relationship Mapping:** Identify connections between targets, their businesses, their families, and their routines.
4. **Pattern Analysis:** Determine schedules, travel patterns, communication habits, and security postures.

#### OSINT Sources Typically Used

- **Public Records:** SEC filings, court records, property databases, corporate registrations
- **Social Media:** LinkedIn (corporate connections), Instagram/Twitter (personal habits and locations)
- **WHOIS Data:** Domain registrations linked to business entities
- **Financial Records:** Bloomberg, Reuters, offshore company registries (e.g., Panama Papers databases)
- **Dark Web:** Leaked databases, underground forums, paste sites

### 4. Social Engineering via Impersonation

Elliot and Darlene engage in social engineering through impersonation, a technique that involves assuming a false identity to manipulate targets into revealing information or granting access.

#### Key Impersonation Principles

- **Authority:** Impersonating someone in a position of power (executive, IT administrator, law enforcement) to compel compliance.
- **Urgency:** Creating a time-sensitive scenario that discourages the target from verifying the impersonator's identity.
- **Familiarity:** Using personal details gathered through OSINT to establish trust and credibility.
- **Pretext Development:** Building a complete backstory that can withstand casual scrutiny, including fake business cards, spoofed caller IDs, and fabricated email accounts.

### 5. Physical Surveillance Techniques

The episode depicts physical surveillance operations used to track Deus Group members and understand their security measures.

#### Surveillance Tradecraft

- **Static Surveillance:** Observing a fixed location (target's home, office, or frequented establishment) from a concealed position.
- **Mobile Surveillance:** Following a target on foot or by vehicle while maintaining cover.
- **Counter-Surveillance Detection Routes (SDR):** Techniques used to determine if one is being followed, involving random turns, doubling back, and stopping at unusual locations.
- **Pattern of Life (PoL) Analysis:** Documenting a target's daily routine over days or weeks to identify predictable behaviors and vulnerabilities.
- **Photography/Video:** Discreet documentation of targets, their associates, vehicles, and security details.

---

## Tools Used

| Tool | Purpose |
|---|---|
| **Pafish** | Virtual machine and sandbox detection |
| **OSINT Framework** | Structured approach to open-source intelligence gathering |
| **Maltego** | Relationship mapping and link analysis for target profiling |
| **Shodan** | Internet-connected device reconnaissance |
| **theHarvester** | Email, subdomain, and name enumeration from public sources |
| **Social-Engineer Toolkit (SET)** | Spear-phishing and impersonation attack automation |
| **Recon-ng** | Web reconnaissance framework for OSINT |

---

## Commands Shown

### VM Detection via CPUID (Assembly/C)

```c
#include <stdio.h>
#include <cpuid.h>

int main() {
    unsigned int eax, ebx, ecx, edx;
    __cpuid(1, eax, ebx, ecx, edx);
    if (ecx & (1 << 31)) {
        printf("Hypervisor detected!\n");
        // Get hypervisor vendor string
        __cpuid(0x40000000, eax, ebx, ecx, edx);
        char vendor[13];
        memcpy(vendor, &ebx, 4);
        memcpy(vendor + 4, &ecx, 4);
        memcpy(vendor + 8, &edx, 4);
        vendor[12] = '\0';
        printf("Hypervisor vendor: %s\n", vendor);
    }
    return 0;
}
```

### MAC Address Check (Linux)

```bash
# Check for VM MAC address prefixes
ip link show | grep -iE "00:0c:29|00:50:56|08:00:27|00:15:5d|00:1c:42|52:54:00"
```

### Registry Check for VM (Windows PowerShell)

```powershell
# Check for VMware Tools
Get-ItemProperty -Path "HKLM:\SOFTWARE\VMware, Inc.\VMware Tools" -ErrorAction SilentlyContinue

# Check for VirtualBox Guest Additions
Get-ItemProperty -Path "HKLM:\SOFTWARE\Oracle\VirtualBox Guest Additions" -ErrorAction SilentlyContinue

# Check SMBIOS data
Get-WmiObject Win32_ComputerSystem | Select-Object Manufacturer, Model
```

### OSINT Reconnaissance Commands

```bash
# theHarvester for email and subdomain enumeration
theHarvester -d deusgroup.com -b google,linkedin,twitter -l 500

# Recon-ng for structured OSINT
recon-ng
> marketplace install all
> modules load recon/domains-hosts/google_site_web
> options set SOURCE deusgroup.com
> run

# WHOIS lookup
whois deusgroup.com

# Subdomain enumeration
amass enum -d deusgroup.com -passive
```

---

## Real-World Parallels

### VM Detection in Malware

Sophisticated malware families routinely employ VM detection to evade analysis:

- **Emotet:** Checks for VM artifacts before executing payloads, including process names and registry keys.
- **TrickBot:** Uses CPUID and timing checks to detect sandbox environments.
- **Dridex:** Employs multi-layered VM detection including mouse movement analysis and screen resolution checks.

### OSINT in Real Attacks

- **APT28 (Fancy Bear):** Russian state-sponsored group extensively uses OSINT to profile targets before launching spear-phishing campaigns against government and military personnel.
- **Lazarus Group:** North Korean hackers use LinkedIn and other professional networks to identify and target employees at financial institutions and cryptocurrency exchanges.
- **Social Media Exploitation:** The 2020 Twitter hack began with OSINT gathering on Twitter employees, identifying those with access to internal admin tools.

### Social Engineering Incidents

- **Kevin Mitnick:** The famous hacker relied heavily on impersonation and social engineering, often calling companies while impersonating IT staff to obtain passwords and access codes.
- **Frank Abagnale:** Famously impersonated a Pan Am pilot, doctor, and attorney, demonstrating how authority-based impersonation can bypass even rigorous verification procedures.
- **RSA Breach (2011):** Attackers used targeted social engineering emails with a malicious Excel attachment to compromise RSA's SecurID two-factor authentication tokens.

## Tool Links

- [Pafish (Paranoid Fish)](https://github.com/a0rtega/pafish) - VM and sandbox detection tool
- [Maltego](https://www.maltego.com/) - Relationship mapping and link analysis for target profiling
- [Shodan](https://www.shodan.io/) - Internet-connected device reconnaissance
- [SpiderFoot](https://github.com/smicallef/spiderfoot) - Automated OSINT tool
- [Sherlock](https://github.com/sherlock-project/sherlock) - Username search across platforms
- [Amass](https://github.com/owasp-amass/amass) - Subdomain enumeration and infrastructure mapping
- [SET (Social Engineering Toolkit)](https://github.com/trustedsec/social-engineer-toolkit) - Social engineering attack automation
- [Nmap](https://nmap.org/) - Network scanning and service enumeration

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| VM/Sandbox detection | T1497 | Virtualization/Sandbox Evasion | https://attack.mitre.org/techniques/T1497/ |
| OSINT reconnaissance on Deus Group members | T1589 | Gather Victim Identity Information | https://attack.mitre.org/techniques/T1589/ |
| Subdomain and infrastructure enumeration | T1592 | Gather Victim Host Information | https://attack.mitre.org/techniques/T1592/ |
| Active target reconnaissance | T1595 | Active Scanning | https://attack.mitre.org/techniques/T1595/ |
| Social engineering through impersonation | T1566 | Phishing | https://attack.mitre.org/techniques/T1566/ |
| Physical surveillance and information gathering | T1598 | Phishing for Information | https://attack.mitre.org/techniques/T1598/ |
| System registry checks for VM detection | T1082 | System Information Discovery | https://attack.mitre.org/techniques/T1082/ |

## References and Further Reading

- ESET Research: "Pafish - Paranoid Fish" - Sandbox and virtual machine detection techniques
- MITRE ATT&CK - Virtualization/Sandbox Evasion: https://attack.mitre.org/techniques/T1497/
- APT28 (Fancy Bear) OSINT campaigns - CrowdStrike Intelligence Reports
- Kevin Mitnick, "The Art of Deception" (Wiley, 2002) - Classic reference on social engineering
- Frank Abagnale, "Catch Me If You Can" - Impersonation and social engineering in practice
- RSA Breach Analysis (2011) - Anatomy of an advanced social engineering attack
- Twitter Hack 2020 - OSINT and social engineering against tech platform employees
- CVE-2017-7921 - Example of device reconnaissance via Shodan

## Search Tags

```
tags: [pafish, maltego, shodan, spiderfoot, sherlock, amass, SET, OSINT, VM-detection, social-engineering, impersonation, reconnaissance, physical-surveillance]
season: 4
episode: 1
mitre: [T1497, T1589, T1592, T1595, T1566, T1598, T1082]
```
