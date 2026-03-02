# Episode 6: eps4.5_406notacceptable.h

## Season 4, Episode 6 | "406 Not Acceptable"

**Air Date:** November 10, 2019
**Directed by:** Sam Esmail

### Overview

The episode title references HTTP status code 406, which indicates that the server cannot produce a response matching the acceptable values defined in the request's headers. Thematically, the outcomes of events in this episode are "not acceptable" to the characters. The episode focuses on FBI surveillance evasion, forensic analysis techniques, and the Dark Army's lethal operational security including kill chains and dead man's switches that ensure their operations remain uncompromised.

---

## Hacks & Techniques

### 1. FBI Surveillance Evasion and Counter-Surveillance

Characters must operate while under active FBI surveillance, requiring sophisticated evasion techniques.

#### Surveillance Evasion Tradecraft

- **Surveillance Detection Route (SDR):**
  - A carefully planned route designed to force anyone following you to reveal themselves.
  - Includes choke points (narrow alleys, one-way streets), stops at locations where loitering would be unusual, and sudden direction changes.
  - Should appear natural and purposeful, not evasive.

- **Communication Security:**
  - Avoid personal mobile devices (tracked via cell tower triangulation and IMSI catchers).
  - Use burner phones purchased with cash, activated away from home.
  - Use prepaid SIM cards and rotate frequently.
  - Communicate through dead drops (physical or digital) instead of real-time messaging.

- **Digital Counter-Surveillance:**
  - Disable Wi-Fi and Bluetooth on devices to prevent tracking via probe requests.
  - Use Faraday bags to block all radio signals to/from mobile devices.
  - Boot from Tails OS (amnesic live system) to leave no digital traces.
  - Use Tor for any internet activity.

- **Physical Evasion:**
  - Change appearance (clothing, hats, glasses) to defeat facial recognition and visual surveillance.
  - Use public transit with multiple transfers to complicate vehicle surveillance.
  - Enter buildings with multiple exits and leave through a different exit.
  - Time movements to coincide with crowds, events, or rush hours.

#### FBI Surveillance Methods (What is Being Evaded)

- **Mobile Surveillance Teams:** Trained agents in multiple vehicles and on foot, rotating positions.
- **Fixed Surveillance Posts:** Cameras and agents positioned near known locations (home, workplace).
- **Electronic Surveillance:** Phone taps, IMSI catchers (Stingray devices), email monitoring.
- **Financial Tracking:** Monitoring credit card transactions, bank withdrawals, and wire transfers.
- **CCTV Access:** Access to both public and private camera systems.

### 2. GPS Tracker Detection

Finding and neutralizing GPS tracking devices planted on vehicles or carried unknowingly.

#### Types of GPS Trackers

| Type | Placement | Power Source | Detection Method |
|---|---|---|---|
| **Magnetic Mount** | Vehicle undercarriage, wheel well | Internal battery | Physical inspection |
| **OBD-II Port** | Plugged into vehicle diagnostic port | Vehicle power | Visual check of port |
| **Hardwired** | Connected to vehicle electrical system | Vehicle power | Electrical inspection |
| **Personal** | In bag, clothing, or carried items | Internal battery | RF sweep |

#### Detection Techniques

- **Physical Inspection:**
  - Check wheel wells, undercarriage, bumpers, and behind panels.
  - Look for anything that appears out of place, magnetically attached, or has an antenna.
  - Use a mirror and flashlight for hard-to-reach areas.

- **RF Detection:**
  - GPS trackers transmit location data via cellular networks (GSM/LTE).
  - Use an RF detector or spectrum analyzer to identify unauthorized transmissions.
  - Check for periodic burst transmissions (trackers often transmit at intervals to conserve battery).

- **Electronic Sweep:**
  - Non-Linear Junction Detectors (NLJD) can detect electronic components even when powered off.
  - Professional TSCM teams use specialized equipment for comprehensive sweeps.

### 3. Log Analysis and Forensics

The episode features forensic investigation techniques, highlighting how digital evidence is analyzed and how attackers try to cover their tracks.

#### Splunk

Splunk is an enterprise-grade platform for searching, monitoring, and analyzing machine-generated data:

- **Data Ingestion:** Collects logs from servers, network devices, applications, and security tools.
- **Search Processing Language (SPL):** Powerful query language for analyzing log data.
- **Dashboards:** Real-time visualization of security events and anomalies.
- **SIEM Capability:** Used as a Security Information and Event Management platform.

#### ELK Stack (Elasticsearch, Logstash, Kibana)

Open-source alternative to Splunk for log management and analysis:

- **Elasticsearch:** Distributed search and analytics engine for storing and querying log data.
- **Logstash:** Data processing pipeline for ingesting, transforming, and forwarding logs.
- **Kibana:** Web interface for visualizing Elasticsearch data with dashboards and charts.
- **Beats:** Lightweight data shippers (Filebeat, Winlogbeat, Packetbeat) for collecting logs.

#### Windows Event Log Analysis

Critical Windows Event IDs for security forensics:

| Event ID | Log | Description |
|---|---|---|
| **4624** | Security | Successful logon |
| **4625** | Security | Failed logon attempt |
| **4648** | Security | Logon using explicit credentials |
| **4672** | Security | Special privileges assigned |
| **4688** | Security | New process created |
| **4698** | Security | Scheduled task created |
| **4720** | Security | User account created |
| **7045** | System | New service installed |
| **1102** | Security | Audit log cleared |

### 4. Dark Army Kill Chain and Dead Man's Switches

The Dark Army operates with extreme operational security, including kill chain procedures and dead man's switches that activate when operators are compromised.

#### Kill Chain Concept

In the Dark Army's context, the "kill chain" refers to their procedure for eliminating compromised operatives and destroying evidence:

1. **Detection:** Identify that an operative has been compromised or captured.
2. **Assessment:** Determine the scope of potential intelligence exposure.
3. **Containment:** Isolate the compromised operative from the rest of the network.
4. **Elimination:** Terminate the operative and destroy all evidence linking back to the organization.
5. **Reconstitution:** Activate replacement operatives and rebuild compromised infrastructure.

#### Dead Man's Switch

A dead man's switch is a mechanism that triggers automatically when an operator fails to perform a regular action (proving they are alive and free):

- **Digital Dead Man's Switch:**
  - Requires regular check-in (e.g., entering a password on a website every 24 hours).
  - If the check-in is missed, pre-programmed actions execute automatically.
  - Actions may include: releasing encrypted information, wiping devices, sending alerts, or activating self-destruct protocols.

- **Implementation Approaches:**
  - Scheduled tasks that must be canceled regularly.
  - Encrypted containers that require regular re-authentication to prevent key destruction.
  - Cloud-based services that trigger webhooks on missed check-ins.
  - Hardware devices with timers that activate physical destruction mechanisms.

### 5. Auto-Destruct Encrypted Communications

The Dark Army uses communication systems designed to automatically destroy messages after reading or after a set time period.

#### Self-Destructing Message Implementations

- **Signal Disappearing Messages:** Messages automatically deleted after a configurable time period (from 5 seconds to 4 weeks) on both sender and recipient devices.
- **Burn-on-Read:** Messages that are destroyed immediately after being read by the recipient.
- **Ephemeral Key Exchange:** Communication sessions where encryption keys are destroyed after each message, making retroactive decryption impossible.

#### Secure Communication Protocols

- **Forward Secrecy:** Each message uses a unique session key; compromise of one key does not compromise other messages.
- **Zero-Knowledge Architecture:** The server never has access to plaintext messages or encryption keys.
- **Metadata Minimization:** Reducing or eliminating metadata (who communicates with whom, when, and how often).

---

## Tools Used

| Tool | Purpose |
|---|---|
| **Splunk** | Enterprise log analysis and SIEM |
| **ELK Stack** | Open-source log management and visualization |
| **Wireshark** | Network traffic analysis for forensic investigation |
| **RF Bug Detector** | Detection of GPS trackers and listening devices |
| **Spectrum Analyzer** | Radio frequency analysis |
| **Faraday Bag** | Signal blocking for mobile devices |
| **Tails OS** | Amnesic operating system for anonymous operations |
| **Signal** | Encrypted messaging with disappearing messages |
| **Non-Linear Junction Detector** | Detection of hidden electronic components |

---

## Commands Shown

### Splunk SPL Queries for Forensics

```spl
# Search for failed login attempts from a specific source
index=security sourcetype=WinEventLog:Security EventCode=4625
| stats count by src_ip, user
| sort -count

# Detect lateral movement via PsExec
index=security sourcetype=WinEventLog:System EventCode=7045
| search ServiceName="PSEXESVC"
| table _time, ComputerName, ServiceName, ServiceFileName

# Find log clearing events (evidence of cover-up)
index=security sourcetype=WinEventLog:Security EventCode=1102
| table _time, ComputerName, SubjectUserName

# Hunt for suspicious process execution
index=security sourcetype=WinEventLog:Security EventCode=4688
| search (NewProcessName="*powershell*" OR NewProcessName="*cmd*")
| where CommandLine LIKE "%encoded%"
| table _time, ComputerName, SubjectUserName, NewProcessName, CommandLine

# Detect data exfiltration via unusual outbound connections
index=network sourcetype=firewall action=allowed direction=outbound
| stats sum(bytes_out) as total_bytes by dest_ip
| where total_bytes > 1073741824
| sort -total_bytes
```

### ELK Stack Setup and Queries

```bash
# Elasticsearch query for failed logins
curl -X GET "localhost:9200/winlogbeat-*/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        {"match": {"event.code": "4625"}},
        {"range": {"@timestamp": {"gte": "now-24h"}}}
      ]
    }
  },
  "aggs": {
    "by_source_ip": {
      "terms": {"field": "source.ip"}
    }
  }
}'

# Kibana KQL query
event.code: 4625 and source.ip: 10.0.0.*
```

### GPS Tracker Detection

```bash
# Using RTL-SDR to scan for GPS tracker transmissions
rtl_power -f 800M:1GHz:100k -g 50 -i 10 -e 1h scan_results.csv

# Analyze with heatmap
python3 heatmap.py scan_results.csv output.png

# Use gqrx for real-time SDR spectrum analysis
gqrx  # GUI application - tune to common GSM frequencies (850/900/1800/1900 MHz)
```

### Windows Event Log Forensics

```powershell
# Query for security-relevant events
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4624} -MaxEvents 100 |
  Select-Object TimeCreated, @{N='User';E={$_.Properties[5].Value}},
  @{N='LogonType';E={$_.Properties[8].Value}},
  @{N='SourceIP';E={$_.Properties[18].Value}}

# Find audit log clearing
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=1102}

# Export event logs for analysis
wevtutil epl Security C:\forensics\security.evtx

# Parse with PowerShell
Get-WinEvent -Path C:\forensics\security.evtx |
  Where-Object {$_.Id -eq 4688} |
  Format-Table TimeCreated, Message -AutoSize
```

---

## Real-World Parallels

### FBI Surveillance Programs

- **COINTELPRO (1956-1971):** The FBI's counterintelligence program conducted extensive surveillance on civil rights leaders, political organizations, and activists. Techniques included physical surveillance, wiretapping, mail interception, and informant infiltration.
- **Stingray / IMSI Catchers:** Law enforcement agencies use cell-site simulators to track mobile phones, intercept communications, and identify devices in a specific area. Their use was often kept secret, even from judges.
- **NSA PRISM Program:** Revealed by Edward Snowden, PRISM provided the government with access to data from major tech companies, demonstrating the scope of digital surveillance capabilities.

### Dead Man's Switches in Practice

- **WikiLeaks Insurance Files:** Julian Assange published encrypted "insurance" files, with the implication that decryption keys would be released if anything happened to him, functioning as a dead man's switch for information disclosure.
- **Dread Pirate Roberts (Silk Road):** Ross Ulbricht allegedly had mechanisms to alert others or destroy evidence if he was arrested, though they were neutralized by the FBI's careful arrest operation in a public library.

### Log Analysis in Incident Response

- **Target Breach (2013):** Security firm FireEye's monitoring tools detected the intrusion, but alerts went unheeded. Post-breach forensics relied heavily on log analysis to reconstruct the attack timeline and identify the scope of data theft.
- **Equifax Breach (2017):** Post-breach investigation used log analysis to determine that attackers had been present in the network for 76 days before detection, exfiltrating data on 147 million people. The breach was partly attributed to failure to monitor logs effectively.
- **SolarWinds (2020):** Detection of the SUNBURST backdoor was achieved through log analysis by FireEye, which noticed anomalous authentication patterns in their own environment.

### Counter-Surveillance in Espionage

- **Aldrich Ames (CIA Mole):** FBI surveillance of CIA officer Aldrich Ames included physical surveillance, financial monitoring, and communications interception. Ames attempted counter-surveillance by using dead drops and coded communications, but ultimately failed to detect the FBI's investigation.
- **Robert Hanssen (FBI Mole):** Hanssen successfully evaded detection for 22 years by understanding FBI surveillance techniques from the inside, using dead drops instead of electronic communications, and carefully managing his financial transactions.

## Tool Links

- [Splunk](https://www.splunk.com/) - Enterprise log analysis and SIEM
- [ELK Stack](https://www.elastic.co/elastic-stack) - Open-source log management and visualization
- [Wireshark](https://www.wireshark.org/) - Network traffic analysis for forensic investigation
- [Tails](https://tails.net/) - Amnesic operating system for anonymous operations
- [Signal](https://signal.org/) - Encrypted messaging with disappearing messages
- [Tor](https://www.torproject.org/) - Anonymous traffic routing via the Tor network
- [Volatility](https://www.volatilityfoundation.org/) - Memory forensic analysis (contextual reference)
- [YARA](https://virustotal.github.io/yara/) - Malware detection rules (contextual reference)

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| FBI surveillance evasion | T1036 | Masquerading | https://attack.mitre.org/techniques/T1036/ |
| GPS tracker detection and neutralization | T1016 | System Network Configuration Discovery | https://attack.mitre.org/techniques/T1016/ |
| Forensic log analysis (Splunk/ELK) | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| Dark Army dead man's switch | T1480 | Execution Guardrails | https://attack.mitre.org/techniques/T1480/ |
| Self-destructing encrypted communications | T1573 | Encrypted Channel | https://attack.mitre.org/techniques/T1573/ |
| Burner phones and disposable SIM cards | T1036 | Masquerading | https://attack.mitre.org/techniques/T1036/ |
| Digital counter-surveillance (Faraday bags) | T1562 | Impair Defenses | https://attack.mitre.org/techniques/T1562/ |
| Windows Event Log analysis for forensics | T1087 | Account Discovery | https://attack.mitre.org/techniques/T1087/ |

## References and Further Reading

- COINTELPRO (1956-1971) - FBI counterintelligence program and surveillance techniques
- Edward Snowden / NSA PRISM - Revelations about the scope of government digital surveillance
- WikiLeaks Insurance Files - Dead man's switch for information disclosure
- Silk Road / Dread Pirate Roberts - Operational security mechanisms and dead man's switch
- Target Breach (2013) - Log analysis failure resulting in massive data theft
- Equifax Breach (2017) - Attackers present for 76 days without log detection
- SolarWinds (2020) - Detection via log analysis of anomalous authentication patterns
- Aldrich Ames / Robert Hanssen - Counter-surveillance in real-world espionage
- Stingray / IMSI Catchers - Use of cell tower simulators by law enforcement agencies

## Search Tags

```
tags: [splunk, ELK-stack, wireshark, tails, signal, tor, counter-surveillance, FBI-evasion, GPS-tracker, log-analysis, dead-mans-switch, faraday-bag, forensics, SIEM, disappearing-messages]
season: 4
episode: 6
mitre: [T1036, T1016, T1070, T1480, T1573, T1562, T1087]
```
