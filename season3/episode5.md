# Season 3, Episode 5: eps3.4_runtime-error.r00 (The Single-Take Episode)

## Episode Overview

This is the landmark single-take episode of Mr. Robot -- filmed (or edited) to appear as one continuous, unbroken shot. The `.r00` extension references the first volume of a RAR archive split across multiple files, symbolizing how the events of this episode are one continuous, unbroken sequence. Stage 2 executes: the modified UPS firmware triggers thermal runaway in batteries across 71 E Corp buildings (the plan was expanded from the original single-building target), causing fires, explosions, and catastrophic loss of life. Elliot races in real time to stop the attack, navigating E Corp's network and physical infrastructure in a desperate attempt to push emergency firmware patches before it is too late.

---

## Hacks & Techniques

### Stage 2 Execution: UPS Thermal Runaway Attack

The attack Elliot has been trying to prevent finally executes. The modified firmware on UPS units across 71 E Corp buildings simultaneously triggers thermal runaway in their lithium-ion battery banks.

**Attack execution sequence:**

```
T-0:00  Trigger signal sent (either timed or remote command)
T-0:05  Modified firmware disables BMS thermal cutoffs
T-0:10  Charging circuits forced to maximum voltage (5.5V per cell vs 4.2V nominal)
T-0:30  Cell temperatures begin rising past safe limits
T-2:00  First cells reach 150°C - electrolyte decomposition begins
T-3:00  Thermal runaway initiates - exothermic chain reaction
T-3:30  Adjacent cells cascade into thermal runaway
T-5:00  Battery enclosures fail - venting of flammable gases
T-6:00  Ignition of vented gases - fire spreads to surrounding equipment
T-8:00  Building fire suppression systems overwhelmed
T-10:00 Structural damage begins in server rooms and records storage
```

**Scale of the attack:**

- Originally designed to target a single E Corp building housing paper records
- Dark Army expanded the target list to 71 buildings without Elliot's knowledge
- Each building contains multiple UPS systems with large battery banks
- The simultaneous nature of the attacks overwhelms emergency response

### Expansion from 1 to 71 Buildings

The escalation from one building to 71 represents a fundamental shift in the attack's purpose:

- **Original plan (1 building)**: Destroy paper records to make the 5/9 encryption irreversible. A targeted infrastructure attack with minimal casualties.
- **Expanded plan (71 buildings)**: Mass casualty event designed to destabilize society. The paper records are secondary; the human cost is the primary objective.

This expansion was possible because:
- All E Corp facilities used the same UPS vendor and management infrastructure
- Centralized firmware management meant a single compromise could propagate to all sites
- Elliot's original attack methodology scaled without modification

### Elliot's Race to Stop Stage 2

Elliot attempts emergency remediation in real time, working through E Corp's systems to push clean firmware before the attack completes:

**Step 1: Identify compromised systems**

```bash
# Emergency scan for UPS units with modified firmware
# Compare firmware checksums against known-good values
for ip in $(cat /opt/ecorp/ups_inventory.txt); do
    fw_hash=$(snmpget -v2c -c public $ip 1.3.6.1.4.1.318.1.1.1.7.2.3.0 2>/dev/null | \
              awk '{print $NF}')
    if [ "$fw_hash" != "$KNOWN_GOOD_HASH" ]; then
        echo "COMPROMISED: $ip - Hash: $fw_hash" >> compromised_ups.txt
    fi
done
```

**Step 2: Emergency firmware push**

```bash
# Push clean firmware to all identified compromised units
# Skip change management - this is an emergency
while read ip; do
    echo "[EMERGENCY] Pushing clean firmware to $ip"
    curl -k --connect-timeout 5 -u admin:$UPS_PASS \
        -F "file=@/opt/ecorp/clean_firmware_v3.2.1.bin" \
        https://$ip/cgi-bin/firmware_update &
done < compromised_ups.txt
wait
echo "[*] All firmware push attempts completed"
```

**Step 3: Emergency shutdown commands**

```bash
# If firmware update is too slow, attempt emergency UPS shutdown
# This cuts power but prevents thermal runaway
for ip in $(cat compromised_ups.txt); do
    # SNMP command to initiate graceful shutdown
    snmpset -v2c -c private $ip 1.3.6.1.4.1.318.1.1.1.7.2.2.0 i 2
    # Backup: Try to disable charging circuit
    snmpset -v2c -c private $ip 1.3.6.1.4.1.318.1.1.1.3.2.1.0 i 1
done
```

### Network Pivoting and Lateral Movement

Elliot must traverse multiple network segments to reach all compromised UPS management interfaces:

**Lateral movement path:**

```
[Elliot's Workstation - Corporate IT]
    |
    v (Credential: AD domain creds)
[Jump Server / Bastion Host]
    |
    v (Credential: OT service account)
[OT Management VLAN]
    |
    v (Credential: UPS admin creds)
[UPS Network Management Cards - Building 1...71]
```

**Techniques used for lateral movement:**

```bash
# SSH tunneling through jump server
ssh -L 10161:ups-mgmt-vlan:161 elliot@jumpserver.ecorp.internal

# SOCKS proxy for accessing multiple OT network hosts
ssh -D 1080 elliot@jumpserver.ecorp.internal
proxychains nmap -sV -p 80,443,161 10.50.0.0/16

# Port forwarding for specific UPS management interfaces
ssh -L 8443:10.50.1.100:443 -L 8444:10.50.1.101:443 elliot@jumpserver.ecorp.internal

# Using PsExec for Windows lateral movement
psexec.py ecorp/elliot@workstation02.ecorp.internal

# WMI-based remote execution
wmiexec.py ecorp/elliot@ot-server01.ecorp.internal "cmd /c snmpset ..."
```

### Physical Security Bypass

The single-take format showcases Elliot physically moving through E Corp facilities, encountering and bypassing physical security controls:

**Badge access systems:**

- **RFID proximity cards**: Standard corporate badge readers operating on 125kHz (HID Prox) or 13.56MHz (iCLASS, MIFARE)
- **Tailgating/piggybacking**: Following an authorized person through a badge-controlled door without swiping
- **Badge cloning**: If Elliot had access to an RFID cloner (e.g., Proxmark3), he could clone an authorized badge

```bash
# Proxmark3 badge cloning (if available)
proxmark3> lf hid read          # Read a low-frequency HID card
proxmark3> lf hid clone         # Clone the card to a blank
proxmark3> lf hid sim           # Simulate the card with Proxmark
```

**Server room access:**

- Combination locks, key locks, or biometric readers
- Some server rooms use mantrap (airlock) entries requiring two-factor authentication
- Emergency bypass procedures may be invocable during active incidents

**Tailgating technique:**

- Carry equipment (laptop bag, tool box) to appear as if you belong
- Follow closely behind an authorized person through a controlled door
- Time entry with a group of people to make badge-checking difficult
- Use the pretext of an emergency to convince someone to hold the door

### Real-Time SOC Incident Response

As the attack unfolds, E Corp's Security Operations Center (SOC) would initiate incident response procedures:

**Standard incident response timeline:**

```
Detection (T+0):
- SIEM alerts on multiple simultaneous UPS anomalies
- Building fire alarms trigger across multiple facilities
- Network monitoring shows unusual SNMP traffic patterns

Triage (T+2 min):
- SOC analysts assess scope and severity
- Initial classification: Critical severity, multi-site incident
- Incident Commander designated

Containment (T+5 min):
- Attempt to isolate affected UPS network segments
- Emergency shutdown commands issued
- Physical security notified for building evacuations

Eradication (T+15 min):
- Identify root cause (compromised firmware)
- Begin firmware remediation on unaffected units
- Preserve forensic evidence where possible

Recovery (T+60 min - days):
- Replace destroyed UPS units
- Restore power to unaffected areas
- Verify clean firmware on all remaining units
```

**SIEM correlation rules that should fire:**

```
Rule: Multiple UPS thermal alarms within 60-second window
Severity: CRITICAL
Condition: count(ups_thermal_alarm) > 5 within 60s
Action: Page incident response team, auto-isolate UPS VLAN

Rule: Unauthorized firmware update to UPS systems
Severity: HIGH
Condition: ups_firmware_change NOT IN approved_change_window
Action: Alert SOC, block source IP, preserve logs

Rule: Mass simultaneous UPS voltage anomalies
Severity: CRITICAL
Condition: ups_voltage > 4.5V_per_cell across > 3 systems
Action: Emergency shutdown all UPS charging, evacuate facilities
```

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **SNMP tools** | Emergency UPS management and firmware deployment |
| **SSH tunneling** | Network pivoting through segmented networks |
| **Proxychains** | SOCKS proxy for routing tools through tunnels |
| **curl** | HTTP-based firmware upload to UPS management cards |
| **Nmap** | Emergency network scanning for UPS devices |
| **PsExec / WMIExec** | Lateral movement across Windows systems |
| **Proxmark3** | RFID badge cloning (physical access) |
| **SIEM (Splunk/QRadar)** | Security event correlation and alerting |

---

## Commands Shown

```bash
# Emergency UPS firmware check across all buildings
parallel -j 50 'snmpget -v2c -c public {} 1.3.6.1.4.1.318.1.1.1.7.2.3.0' \
    :::: ups_all_buildings.txt > firmware_audit.txt

# Emergency shutdown via SNMP
snmpset -v2c -c private <ups_ip> 1.3.6.1.4.1.318.1.1.1.7.2.2.0 i 2

# Network pivoting via SSH
ssh -J jumpserver.ecorp.internal -L 8443:10.50.1.100:443 elliot@ot-bastion

# Real-time log monitoring during incident
tail -f /var/log/syslog | grep -i "ups\|thermal\|battery\|alarm"

# Emergency network isolation (firewall rule)
iptables -I FORWARD -d 10.50.0.0/16 -j DROP  # Block all traffic to OT VLAN
iptables -I FORWARD -s 10.50.0.0/16 -j DROP  # Block all traffic from OT VLAN
# Exception for emergency firmware push
iptables -I FORWARD -s 10.0.1.50 -d 10.50.0.0/16 -p tcp --dport 443 -j ACCEPT
```

---

## Real-World Parallels

### Samsung Galaxy Note 7 (2016)
- Samsung Galaxy Note 7 devices experienced spontaneous lithium-ion battery thermal runaway due to manufacturing defects, not malicious firmware, but the physical result was identical to what Stage 2 achieves intentionally. Samsung recalled 2.5 million devices and the FAA banned the phone from all flights. This real-world event validates the show's portrayal of battery thermal runaway as a genuinely dangerous phenomenon.

### Stuxnet (2010)
- Stuxnet targeted Siemens S7-300 PLCs controlling uranium enrichment centrifuges, modifying their firmware to cause physical destruction while reporting normal operating parameters to monitoring systems. Stage 2 mirrors this approach: the modified UPS firmware causes physical damage (thermal runaway) while reporting nominal values to the network management system. Both attacks demonstrate the cyber-physical attack paradigm where software manipulation causes real-world physical destruction.

### Industrial Control System Attacks
- **TRITON/TRISIS (2017)**: Targeted safety instrumented systems at a Saudi petrochemical facility, attempting to disable safety controls that prevent catastrophic failures -- directly analogous to disabling UPS thermal protections.
- **Ukraine power grid attacks (2015, 2016)**: Demonstrated that cyber attacks on critical infrastructure can cause widespread physical impact affecting thousands of people.
- **Oldsmar water treatment plant (2021)**: An attacker attempted to increase sodium hydroxide levels in a Florida water treatment facility, showing how SCADA manipulation can threaten public safety.

### Incident Response Challenges
- **NotPetya (2017)**: Organizations hit by NotPetya faced challenges similar to E Corp's SOC -- a rapidly spreading, destructive attack across multiple sites with limited time to respond. Maersk lost nearly all of its 49,000 laptops and servers across 600 sites in multiple countries.
- **WannaCry (2017)**: The UK's NHS was hit across multiple hospitals simultaneously, creating the same kind of multi-site crisis response challenge depicted in this episode.

## Tool Links

- [Nmap](https://nmap.org/) - Emergency network scanning for UPS devices
- [Proxychains](https://github.com/haad/proxychains) - SOCKS proxy for routing tools through tunnels
- [Impacket](https://github.com/fortra/impacket) - PsExec and WMIExec for lateral movement
- [Splunk](https://www.splunk.com/) - SIEM for security event correlation
- [ELK Stack](https://www.elastic.co/elastic-stack) - Real-time log analysis during incident
- [Wireshark](https://www.wireshark.org/) - SNMP and network traffic analysis during incident response
- [Metasploit](https://www.metasploit.com/) - Exploitation framework for lateral movement
- [Cobalt Strike](https://www.cobaltstrike.com/) - Adversary operations and lateral movement platform

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Stage 2 execution - physical destruction | T1529 | System Shutdown/Reboot | https://attack.mitre.org/techniques/T1529/ |
| Data and infrastructure destruction | T1485 | Data Destruction | https://attack.mitre.org/techniques/T1485/ |
| Lateral movement via SSH tunneling | T1021 | Remote Services | https://attack.mitre.org/techniques/T1021/ |
| Lateral movement via PsExec/WMIExec | T1569 | System Services | https://attack.mitre.org/techniques/T1569/ |
| Network pivoting | T1090 | Proxy | https://attack.mitre.org/techniques/T1090/ |
| Physical security bypass (tailgating) | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| Emergency network scanning | T1046 | Network Service Discovery | https://attack.mitre.org/techniques/T1046/ |
| Firmware manipulation at scale | T1195 | Supply Chain Compromise | https://attack.mitre.org/techniques/T1195/ |

## References and Further Reading

- **Armis TLStorm (2022)**: CVE-2022-22805, CVE-2022-22806, CVE-2022-0715 -- vulnerabilities in APC Smart-UPS validating the Stage 2 attack vector
- **Stuxnet Dossier (Symantec, 2011)**: Complete technical analysis of the first documented cyber-physical attack
- **TRITON/TRISIS (FireEye, 2017)**: Malware targeting industrial safety systems -- direct parallel with disabling UPS thermal protections
- **NotPetya Analysis (CISA)**: Incident response challenges in simultaneous multi-site attacks
- **WannaCry NHS Impact Report (2017)**: Impact of simultaneous attack across multiple facilities
- **NIST SP 800-61**: Computer Security Incident Handling Guide -- standard incident response procedures
- **Samsung Galaxy Note 7 Root Cause Analysis (2017)**: Official analysis of thermal runaway in lithium-ion batteries

## Search Tags

```
tags: [Stage-2, thermal-runaway, UPS, lateral-movement, SSH-tunneling, Proxychains, PsExec, WMIExec, SIEM, incident-response, physical-security, badge-cloning, Proxmark3, SNMP]
season: 3
episode: 5
mitre: [T1529, T1485, T1021, T1569, T1090, T1200, T1046, T1195]
```
