# Season 3, Episode 2: eps3.1_undo.gz

## Episode Overview

Elliot, now working inside E Corp as a cybersecurity engineer, races to undo Stage 2 from within the corporate network. The `.gz` (gzip) extension symbolizes compression and reversal -- Elliot is trying to decompress the damage and restore what was corrupted. He attempts to deploy clean firmware patches to the compromised UPS systems while navigating corporate change management processes and the ever-present risk of Tyrell Wellick, who now holds a position of power within E Corp as CTO. Meanwhile, the Dark Army watches from the shadows, and secure communications become critical.

---

## Hacks & Techniques

### Reversing Stage 2 from Inside E Corp

Elliot's approach to neutralizing Stage 2 involves working from the inside -- using his legitimate access as an E Corp employee to identify and remediate the compromised UPS units before the attack can be triggered.

**Elliot's remediation strategy:**

1. **Inventory**: Identify all UPS units across E Corp facilities using the asset management database and network scanning
2. **Firmware verification**: Compare firmware hashes on deployed units against known-good firmware images from the vendor
3. **Clean firmware deployment**: Push verified firmware to all compromised units through the centralized management platform
4. **Monitoring**: Establish alerting for any further unauthorized firmware changes

```bash
# Network scan for UPS management cards (typically on port 80, 443, or 161)
nmap -sV -p 80,443,161 --script=snmp-info 10.0.0.0/8 -oA ups_scan

# Query UPS firmware version via SNMP
snmpwalk -v2c -c public <ups_ip> 1.3.6.1.4.1.318.1.1.1.7.2

# Calculate firmware hash for comparison
sha256sum /path/to/vendor/firmware.bin
sha256sum /path/to/extracted/firmware.bin

# Batch firmware update script
for ip in $(cat ups_targets.txt); do
    echo "[*] Updating firmware on $ip"
    curl -k -u admin:password -F "file=@clean_firmware.bin" \
        https://$ip/cgi-bin/firmware_update
    echo "[+] $ip updated"
done
```

### Deploying Clean Firmware Patches

The firmware patching process for enterprise UPS systems involves several challenges:

- **Authentication**: Each UPS network management card requires credentials. Default credentials are common but may have been changed.
- **Network reachability**: UPS management interfaces may be on isolated management VLANs not directly accessible from standard workstations.
- **Firmware signing**: If the original attack exploited a lack of firmware signing, the clean firmware can be deployed through the same unsigned update mechanism.
- **Verification**: After deployment, each unit must be verified to confirm the clean firmware is running and thermal protections are restored.

```bash
# Verify firmware integrity post-update
snmpget -v2c -c public <ups_ip> 1.3.6.1.4.1.318.1.1.1.7.2.3.0  # Firmware revision
snmpget -v2c -c public <ups_ip> 1.3.6.1.4.1.318.1.1.1.2.2.1.0  # Battery temp (should be normal)
snmpget -v2c -c public <ups_ip> 1.3.6.1.4.1.318.1.1.1.2.2.2.0  # Battery voltage (should be nominal)
```

### Network Segmentation: IT vs. OT/SCADA

A critical plot element is the network architecture separating E Corp's corporate IT network from their Operational Technology (OT) and SCADA systems that control physical infrastructure including UPS units.

**IT/OT segmentation model:**

```
[Corporate IT Network]
    |
    | (Firewall / DMZ)
    |
[IT/OT DMZ - Historian, Jump Servers]
    |
    | (Industrial Firewall - Strict ACLs)
    |
[OT/SCADA Network]
    |--- PLCs (Programmable Logic Controllers)
    |--- UPS Management Cards
    |--- BMS (Building Management Systems)
    |--- HVAC Controls
```

**Key segmentation challenges for Elliot:**

- OT networks often have flat architectures with minimal internal segmentation
- Jump servers or bastion hosts may be the only authorized path between IT and OT
- Industrial protocols (Modbus, BACnet, SNMP) often lack authentication
- Changes to OT systems typically require formal change management approval and maintenance windows

### Active Directory Permissions and Change Management

As an E Corp employee, Elliot must navigate the corporate IT governance structure:

- **Active Directory groups**: Access to OT systems requires membership in specific security groups (e.g., `OT-Admins`, `UPS-Firmware-Mgmt`)
- **Privileged Access Management (PAM)**: Credentials for OT systems may be vaulted in solutions like CyberArk or BeyondTrust, requiring checkout and approval workflows
- **Change Advisory Board (CAB)**: Firmware updates to critical infrastructure require change requests, impact assessments, and management approval
- **Maintenance windows**: Production OT systems can typically only be modified during scheduled downtime

```powershell
# Check current AD group memberships
whoami /groups

# Query AD for OT admin groups
Get-ADGroupMember -Identity "OT-Admins" -Recursive | Select-Object Name

# Request elevated access through PAM
# (Typically done through a web portal, not command line)

# Check change management system for open change windows
# Elliot must either submit a legitimate change request or find a way around the process
```

### Tyrell Wellick as Insider Threat

Tyrell's appointment as E Corp CTO represents **privilege escalation through social position** rather than technical exploitation:

- **Legitimate access**: As CTO, Tyrell has broad access to systems, facilities, and personnel across E Corp
- **Authority**: He can approve changes, override controls, and access restricted areas without triggering security alerts
- **Trust position**: His role places him above suspicion for many security monitoring systems that focus on lower-level employees
- **Policy override**: He can create exceptions to security policies, whitelist IP addresses, or authorize emergency changes

**Insider threat indicators that should raise flags:**

- Accessing systems outside his normal job function
- Working unusual hours or from unusual locations
- Bypassing change management procedures
- Requesting access to highly granular technical systems (a CTO typically delegates this)
- Communicating through unofficial channels

### Monitor Darkly — LCD Monitor Firmware Backdoor

Darlene, now cooperating with the FBI, backdoors Elliot's computer while he sleeps. But she doesn't install malware on his computer — she inserts a device into the back of his LCD monitor that exploits the **display controller firmware**. This lets the FBI see everything displayed on Elliot's screen without leaving any trace on his computer's filesystem or memory.

**Technical details:**

- Based on the **"Monitor Darkly"** research presented at **DEF CON 24 (2016)** by **Ang Cui and Jatin Kataria** (Red Balloon Security)
- Targets the display controller firmware (OSD controller) used in millions of LCD monitors
- When given access to HDMI or USB debug ports on the monitor, the exploit can:
  - Capture everything displayed on screen
  - Inject/manipulate pixels on the display
  - Record screen contents to external storage
- Since it targets only monitor hardware, it leaves **zero traces** on the connected computer
- Standard forensic analysis of the computer would find nothing
- Elliot eventually figures out he's being watched by setting a phishing trap: writing a fake email with a URL on screen and baiting the FBI into following it

```
Attack Architecture:
[FBI Implant Device] --> [Monitor Debug Port (HDMI/USB)]
        |                         |
   Sends captured          Modifies display
   frames to FBI           controller firmware
        |                         |
   [FBI Surveillance]      [Captures all screen
    receives real-time      content invisibly]
        |
   Computer OS has
   ZERO awareness
```

**Real-world:** Ang Cui demonstrated on Dell U2410 monitors that firmware could be modified to capture contents, inject pixels, or alter displayed content — all invisible to the host OS. The research showed that many monitors use similar display controllers, making this a widespread potential attack surface.

### Signal Encrypted Messaging with Disappearing Messages

Characters use Signal for secure communications, leveraging its security properties:

- **End-to-end encryption**: Uses the Signal Protocol (Double Ratchet algorithm with X3DH key agreement)
- **Disappearing messages**: Messages automatically delete after a configurable timer (30 seconds to 1 week)
- **Forward secrecy**: Each message uses a unique encryption key; compromising one key does not decrypt past messages
- **Sealed sender**: Hides sender metadata from the Signal service itself
- **No message logs on server**: Signal servers store minimal metadata

```
Signal Protocol Stack:
    [Plaintext Message]
         |
    [Double Ratchet Encryption] -- Per-message key derivation
         |
    [X3DH Key Agreement] -- Initial key exchange
         |
    [Curve25519 / AES-256-GCM / HMAC-SHA256]
         |
    [TLS 1.3 Transport]
         |
    [Signal Server (stores almost nothing)]
```

**Disappearing message considerations:**

- Messages are deleted from both sender and recipient devices after the timer expires
- Screenshots and notifications can still capture content before deletion
- If a device is seized before the timer expires, messages may be recoverable through forensic imaging
- Disappearing messages provide operational security but not perfect secrecy

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **Nmap** | Network scanning to locate UPS management interfaces |
| **SNMP tools (snmpwalk, snmpget)** | Query and manage UPS units over SNMP |
| **curl** | HTTP-based firmware upload to UPS management cards |
| **Active Directory** | Enterprise identity and access management |
| **Signal** | End-to-end encrypted messaging with disappearing messages |
| **CyberArk / PAM** | Privileged access management for OT credentials |
| **sha256sum** | Firmware integrity verification |

---

## Commands Shown

```bash
# Decompress/extract gzip files (thematic to episode title)
gzip -d archive.gz
zcat archive.gz | less

# Network reconnaissance for UPS devices
nmap -sU -p 161 --script=snmp-brute 10.0.0.0/8

# SNMP enumeration of UPS systems
snmpwalk -v2c -c public <target> 1.3.6.1.4.1.318

# Active Directory queries via LDAP
ldapsearch -x -H ldap://dc01.ecorp.local -b "dc=ecorp,dc=local" \
    "(memberOf=CN=OT-Admins,OU=Groups,DC=ecorp,DC=local)"

# Firmware hash verification
diff <(sha256sum clean_firmware.bin) <(sha256sum extracted_firmware.bin)

# Signal CLI (command-line interface for Signal)
signal-cli -u +1234567890 send -m "Meeting at the usual place" +0987654321
signal-cli -u +1234567890 send -m "Details attached" --expire-in-seconds 30 +0987654321
```

---

## Real-World Parallels

### IT/OT Convergence Attacks
- **Stuxnet (2010)**: The most famous IT/OT attack crossed network boundaries to reach air-gapped SCADA systems controlling Iranian centrifuges. Like Stage 2, it manipulated firmware on physical controllers to cause physical destruction.
- **TRITON/TRISIS (2017)**: Malware targeted Schneider Electric Triconex Safety Instrumented Systems (SIS) in a Saudi petrochemical plant, attempting to disable safety systems -- conceptually similar to disabling UPS thermal protections.
- **Colonial Pipeline (2021)**: Demonstrated how IT-side compromise can have massive OT/physical impact, even when OT systems are not directly attacked.

### Insider Threat Programs
- **Edward Snowden**: Used legitimate system administrator access to exfiltrate classified documents, demonstrating the power of insider access at scale.
- **Robert Hanssen (FBI)**: Exploited his position of trust within the FBI for decades, similar to how Tyrell uses his CTO position.
- **CERT Insider Threat Center**: Carnegie Mellon's research shows that insiders with elevated privileges are responsible for the most damaging incidents.

### Signal Protocol Adoption
- **Signal Protocol**: Now used not only in Signal but also in WhatsApp, Google Messages (RCS), and Facebook Messenger, protecting billions of conversations.
- **Australian Assistance and Access Act (2018)**: Legislation attempting to compel companies to break encryption, highlighting the tension between law enforcement access and communication security that the show explores.

## Tool Links

- [Nmap](https://nmap.org/) - Network scanning to locate UPS management interfaces
- [Signal](https://signal.org/) - End-to-end encrypted messaging with disappearing messages
- [Wireshark](https://www.wireshark.org/) - Network traffic analysis and SNMP protocols
- [BloodHound](https://github.com/BloodHoundAD/BloodHound) - Active Directory relationship and permission analysis
- [PowerView/PowerSploit](https://github.com/PowerShellMafia/PowerSploit) - Active Directory enumeration and exploitation
- [Impacket](https://github.com/fortra/impacket) - Network protocol tools for AD interaction
- [pymodbus](https://github.com/pymodbus-dev/pymodbus) - Modbus industrial protocol interaction
- [BACnet tools](https://sourceforge.net/projects/bacnet/) - Building automation protocol tools
- [Monitor Darkly (Red Balloon Security)](https://redballoonsecurity.com/) - LCD monitor firmware exploitation research

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Network scanning for UPS devices | T1046 | Network Service Discovery | https://attack.mitre.org/techniques/T1046/ |
| Access via Active Directory credentials | T1078 | Valid Accounts | https://attack.mitre.org/techniques/T1078/ |
| Tyrell as insider threat with privilege escalation | T1098 | Account Manipulation | https://attack.mitre.org/techniques/T1098/ |
| IT/OT segmentation and lateral movement | T1021 | Remote Services | https://attack.mitre.org/techniques/T1021/ |
| Encrypted communication via Signal | T1573 | Encrypted Channel | https://attack.mitre.org/techniques/T1573/ |
| Corporate environment reconnaissance | T1082 | System Information Discovery | https://attack.mitre.org/techniques/T1082/ |
| AD group and permission enumeration | T1069 | Permission Groups Discovery | https://attack.mitre.org/techniques/T1069/ |
| Change control bypass | T1562 | Impair Defenses | https://attack.mitre.org/techniques/T1562/ |
| Monitor firmware backdoor for screen capture | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| Monitor Darkly screen surveillance | T1113 | Screen Capture | https://attack.mitre.org/techniques/T1113/ |

## References and Further Reading

- **Stuxnet Analysis (Symantec, 2011)**: "W32.Stuxnet Dossier" -- detailed technical analysis of the most significant IT/OT attack in history
- **TRITON/TRISIS (FireEye/Mandiant, 2017)**: Analysis of the malware that targeted industrial safety systems at a petrochemical plant
- **Colonial Pipeline Incident (CISA, 2021)**: Real-world example of physical impact caused by IT-side compromise
- **NIST SP 800-82**: Guide to Industrial Control Systems Security -- standard reference for OT/SCADA system security
- **CERT Insider Threat Studies**: Carnegie Mellon research on insider threats in corporate environments
- **Signal Protocol Specification**: Technical documentation of the Double Ratchet protocol and X3DH key agreement

## Search Tags

```
tags: [Nmap, SNMP, Active-Directory, Signal, firmware-patching, IT-OT-segmentation, insider-threat, change-management, Tyrell-Wellick, Monitor-Darkly, display-firmware, screen-capture, LCD-backdoor, Ang-Cui, DEF-CON-24, Red-Balloon-Security]
season: 3
episode: 2
mitre: [T1046, T1078, T1098, T1021, T1573, T1082, T1069, T1562, T1200, T1113]
```
