# Season 2, Episode 5: eps2.3_logic-b0mb.hc

## Episode Overview

Named after the logic bomb -- a piece of code inserted into a system that triggers a malicious function when certain conditions are met -- this episode advances the femtocell construction subplot and features social engineering of Angela to infiltrate the FBI. The femtocell, being built by fsociety to intercept FBI cellular communications, takes shape as a realistic device based on Software Defined Radio (SDR) technology. Meanwhile, Angela is manipulated into a position where she can provide physical access to FBI facilities, setting the stage for the Season 2 centerpiece hack.

---

## Hacks & Techniques

### 1. Logic Bomb: Concept and Mechanics

A logic bomb is malicious code deliberately inserted into a software system that lies dormant until a predetermined condition is satisfied. Unlike viruses or worms, logic bombs do not replicate -- they wait silently until triggered.

**Trigger Conditions Can Include:**

- **Date/time trigger**: The bomb activates on a specific date (e.g., Friday the 13th, an employee's termination date)
- **User action trigger**: Activated when a specific user logs in or performs an action
- **Absence trigger**: Activates when a specific condition is NOT met (e.g., when an employee's ID is removed from the payroll system)
- **Counter trigger**: Activates after a system event occurs a certain number of times
- **External signal**: Activated by a command from an external server

**Logic Bomb Structure:**

```python
# Conceptual logic bomb example (for educational purposes)
import datetime
import os

def check_trigger():
    """Check if trigger condition is met"""
    # Time-based trigger
    if datetime.date.today() >= datetime.date(2026, 3, 15):
        return True
    # Absence-based trigger (e.g., check if specific user still exists)
    if not user_exists_in_system("john.doe"):
        return True
    return False

def payload():
    """Malicious action executed when trigger fires"""
    # Could be: data deletion, encryption, exfiltration, etc.
    pass

# The logic bomb check runs silently as part of normal operations
if check_trigger():
    payload()
```

**Characteristics of Logic Bombs:**

- **Stealth**: They are embedded within legitimate code and do not draw attention during normal operation
- **Persistence**: They can remain dormant for months or years before triggering
- **Insider threat**: Often planted by employees or contractors with legitimate system access
- **Difficult to detect**: Static code analysis may not flag the trigger conditions as malicious
- **Deniability**: The attacker may be long gone by the time the bomb triggers

### 2. Real-World Logic Bomb Examples

**Notable Cases:**

- **Roger Duronio (UBS PaineWebber, 2002)**: A systems administrator planted a logic bomb that deleted files on approximately 2,000 servers. He also purchased put options on UBS stock, expecting the attack to drive the price down. He was sentenced to 97 months in prison.

- **Yung-Hsun Lin (Medco Health Solutions, 2004)**: A Unix systems administrator planted a logic bomb triggered by the absence of his name from the employee database. The bomb was designed to delete critical system data if he was laid off. It was discovered before it could detonate.

- **South Korean Banking Attack (2013)**: A logic bomb triggered simultaneously on multiple South Korean bank and broadcaster networks, wiping hard drives across approximately 48,000 computers. Attributed to North Korean threat actors.

- **Siemens Contract Worker (2019)**: A contract worker planted logic bombs in spreadsheet programs he had been hired to maintain, ensuring the spreadsheets would malfunction periodically so he would be called back for paid repair work.

### 3. Femtocell Construction: Hardware and Software

The femtocell being built by fsociety is a rogue cellular base station designed to intercept FBI agents' phone communications. The show depicts the construction process with remarkable technical accuracy.

**What Is a Femtocell:**

A femtocell is a small, low-power cellular base station typically designed to improve indoor cell coverage for homes or small offices. It connects to the cellular carrier's core network via the internet (broadband backhaul). When modified or purpose-built for interception, a femtocell becomes an IMSI catcher -- a surveillance device that forces nearby phones to connect to it instead of legitimate cell towers.

**How a Rogue Femtocell Works:**

1. **Cell Selection**: Mobile phones continuously monitor signal strength from nearby base stations and connect to the strongest one
2. **Power Advantage**: The rogue femtocell, placed close to the target, provides a stronger signal than distant legitimate towers
3. **Downgrade Attack**: The femtocell can force the phone to use older, weaker encryption (A5/1) or no encryption (A5/0) instead of modern standards
4. **Interception**: Once connected, all voice calls, SMS messages, and data pass through the rogue station
5. **Transparent Proxy**: The femtocell relays traffic to a legitimate tower, so the target does not notice any service disruption

### 4. Software Defined Radio (SDR) Components

The femtocell construction involves Software Defined Radio, which implements radio functions in software rather than hardware, providing flexibility to operate on different frequencies and protocols.

**Key SDR Platforms Referenced:**

**OpenBTS (Open Base Transceiver Station):**
- Open-source software that implements the GSM protocol stack
- Turns an SDR into a functional GSM base station
- Originally designed for providing cellular service in remote areas
- Can be configured for 2G GSM operation on various frequency bands

**OsmocomBB (Open Source Mobile Communications Baseband):**
- Open-source GSM baseband implementation
- Allows direct interaction with the GSM air interface
- Supports both base station (BTS) and mobile station (MS) functionality
- Used in security research for protocol analysis and testing

**YateBTS:**
- Another open-source GSM/LTE base station implementation
- Supports both GSM and LTE protocols
- Includes a built-in core network (HLR, MSC, SGSN)
- More user-friendly than OpenBTS for setting up test networks

**SDR Hardware Components:**

| Component | Purpose | Examples |
|---|---|---|
| **SDR Transceiver** | Transmit and receive radio signals | USRP B200/B210, bladeRF, HackRF One |
| **Antenna** | Radiate and receive RF energy | Broadband antenna for cellular bands (700-2600 MHz) |
| **RF Amplifier** | Boost transmit power for range | External PA for 1-5 watt output |
| **GPS Disciplined Oscillator** | Provide accurate clock reference for GSM timing | GPSDO module |
| **Duplexer/Filter** | Separate transmit and receive frequencies | Band-specific cavity filters |
| **Single Board Computer** | Run the SDR software stack | Raspberry Pi, Intel NUC, or laptop |

### 5. IMSI Catcher / Stingray Device Principles

An IMSI catcher (commercially known as a "Stingray" after the Harris Corporation product) is a device that masquerades as a legitimate cell tower to intercept mobile communications.

**IMSI (International Mobile Subscriber Identity):**

The IMSI is a unique identifier stored on a phone's SIM card. It consists of:
- **MCC**: Mobile Country Code (3 digits)
- **MNC**: Mobile Network Code (2-3 digits)
- **MSIN**: Mobile Subscriber Identification Number (up to 10 digits)

**IMSI Catcher Attack Flow:**

```
Normal Operation:
[Phone] -----> [Legitimate Tower] -----> [Carrier Network]

IMSI Catcher Attack:
[Phone] -----> [IMSI Catcher/Femtocell] -----> [Legitimate Tower] -----> [Carrier Network]
                      |
                      v
               [Attacker captures:
                - IMSI numbers
                - Phone calls
                - SMS messages
                - Data traffic
                - Location data]
```

**Attack Techniques:**

- **Identity capture**: Collecting IMSI/IMEI numbers of all phones in range
- **Location tracking**: Determining the physical location of target phones
- **Call/SMS interception**: Recording voice calls and text messages
- **Downgrade attacks**: Forcing connections to use weak or no encryption
- **Silent SMS**: Sending invisible messages to trigger phone responses for tracking
- **Denial of service**: Selectively blocking calls or data for specific subscribers

### 6. Social Engineering of Angela for FBI Infiltration

The episode advances the social engineering of Angela Moss to gain physical access to the FBI's New York field office. This multi-stage manipulation campaign involves:

**Pretexting:**

Creating a false scenario that gives Angela a reason to be present in the FBI building. The pretext must be believable and give her enough time and freedom of movement to deploy devices.

**Key Social Engineering Principles Used:**

- **Authority**: Leveraging Angela's position at E Corp to create a legitimate reason for FBI interaction
- **Reciprocity**: Building a relationship where Angela feels obligated to help
- **Urgency**: Creating time pressure that prevents careful consideration
- **Social proof**: Using the behavior of others (FBI agents' routines) to normalize Angela's presence
- **Commitment and consistency**: Once Angela agrees to help, she is psychologically committed to following through

**Physical Access Considerations:**

- Understanding FBI building security: badge access, security checkpoints, visitor procedures
- Identifying the target floor/room where FBI agents working the E Corp case are located
- Planning device placement locations (femtocell requires proximity to target phones)
- Timing the operation to coincide with predictable staff movements

---

## Tools Used

| Tool | Purpose |
|---|---|
| **OpenBTS** | Open-source GSM base station software |
| **OsmocomBB** | GSM baseband processor software |
| **YateBTS** | GSM/LTE base station with integrated core network |
| **USRP / bladeRF / HackRF** | Software Defined Radio hardware platforms |
| **GNU Radio** | Signal processing framework for SDR |
| **Wireshark (with GSM plugins)** | Protocol analysis for GSM traffic |
| **gr-gsm** | GNU Radio blocks for GSM signal processing |
| **SIM card reader/writer** | Programming SIM cards for test network |

---

## Commands Shown

**OpenBTS Configuration:**

```bash
# Install OpenBTS dependencies
sudo apt-get install autoconf libtool libosip2-dev libortp-dev \
  libusb-1.0-0-dev libsqlite3-dev

# Clone and build OpenBTS
git clone https://github.com/RangeNetworks/openbts.git
cd openbts
./autogen.sh
./configure
make
sudo make install

# Configure OpenBTS for a test network
# Edit /etc/OpenBTS/OpenBTS.db
sqlite3 /etc/OpenBTS/OpenBTS.db \
  "UPDATE CONFIG SET VALUESTRING='901' WHERE KEYSTRING='GSM.Identity.MCC'"
sqlite3 /etc/OpenBTS/OpenBTS.db \
  "UPDATE CONFIG SET VALUESTRING='55' WHERE KEYSTRING='GSM.Identity.MNC'"
```

**YateBTS Setup:**

```bash
# Install YateBTS
sudo apt-get install yate yate-bts

# Configuration file: /usr/local/etc/yate/ybts.conf
# Key settings:
# Radio.Band = 900          # GSM 900 MHz band
# Radio.C0 = 51             # ARFCN channel number
# Identity.MCC = 001        # Test network MCC
# Identity.MNC = 01         # Test network MNC
# Identity.ShortName = Test # Network name displayed on phones

# Start YateBTS
sudo systemctl start yate
```

**SDR Signal Analysis with GNU Radio:**

```bash
# Scan for GSM base stations in the area
grgsm_scanner --band GSM900

# Capture GSM downlink traffic
grgsm_livemon -f 945.2e6 -s 2000000

# Decode captured GSM traffic
tshark -r gsm_capture.pcap -V -Y "gsm_a.dtap"
```

**Logic Bomb Detection (Defensive):**

```bash
# Search for time-based triggers in code
grep -rn "datetime\|time.time\|crontab\|schedule\|Timer" /path/to/codebase/

# Monitor for unauthorized file system changes
aide --check

# Audit scheduled tasks
crontab -l
ls -la /etc/cron.*
systemctl list-timers --all

# Check for suspicious code patterns
grep -rn "os.remove\|shutil.rmtree\|DELETE FROM\|DROP TABLE" /path/to/codebase/
```

---

## Real-World Parallels

### IMSI Catchers in Law Enforcement and Espionage

- **Harris Corporation Stingray**: The most well-known commercial IMSI catcher, used extensively by US law enforcement agencies. The FBI has required local police departments to sign non-disclosure agreements about Stingray use, and cases have been dismissed rather than reveal the technology in court.

- **IMSI Catchers Found Near the White House (2018)**: The Department of Homeland Security confirmed the discovery of unauthorized cell-site simulators (IMSI catchers) operating near the White House and other sensitive areas in Washington, D.C. These were suspected to be operated by foreign intelligence services.

- **iSEC Partners DEF CON Research**: Researchers from iSEC Partners demonstrated practical femtocell hacking at DEF CON, showing how commercially available femtocells could be modified to intercept cellular traffic. This research is directly referenced by Mr. Robot's technical consultants.

### SDR-Based Cellular Security Research

- **Chris Paget at DEF CON 18 (2010)**: Demonstrated a $1,500 IMSI catcher built from commodity hardware and open-source software, showing that cellular interception was no longer limited to well-funded agencies.

- **Karsten Nohl's GSM Research**: Security researcher Karsten Nohl has extensively documented vulnerabilities in the GSM protocol, including cracking the A5/1 encryption algorithm and demonstrating practical interception attacks.

### Logic Bombs in the Wild

- **US Critical Infrastructure**: The Department of Justice has prosecuted multiple cases of logic bombs planted in critical infrastructure systems by disgruntled employees, including power plants, financial systems, and government networks.

- **Nation-State Logic Bombs**: Reports suggest that multiple nations have pre-positioned logic bombs in each other's critical infrastructure networks as a deterrent or first-strike capability -- essentially cyber weapons waiting to be triggered during a conflict.

### Social Engineering in Real Penetration Testing

- Professional penetration testers regularly use social engineering to gain physical access to secure facilities. Techniques shown in the episode -- pretexting, tailgating, creating legitimate-seeming reasons for access -- are standard tools in the social engineering toolkit.

- **Kevin Mitnick**, one of the most famous hackers in history, has written extensively about how social engineering was his primary tool. He argues that it is easier to trick a person into giving you access than to hack your way in technically.

---

## Tool Links

| Tool | Link |
|---|---|
| OpenBTS | http://openbts.org/ |
| YateBTS | https://yatebts.com/ |
| GNU Radio | https://www.gnuradio.org/ |
| USRP (Ettus Research) | https://www.ettus.com/ |
| HackRF | https://greatscottgadgets.com/hackrf/ |
| Wireshark | https://www.wireshark.org/ |

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Logic bomb (dormant malicious code) | T1053 | Scheduled Task/Job | https://attack.mitre.org/techniques/T1053/ |
| Femtocell as additional hardware | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| IMSI catcher (adversary-in-the-middle) | T1557 | Adversary-in-the-Middle | https://attack.mitre.org/techniques/T1557/ |
| Social engineering of Angela (phishing/pretexting) | T1566 | Phishing | https://attack.mitre.org/techniques/T1566/ |
| Network sniffing via femtocell | T1040 | Network Sniffing | https://attack.mitre.org/techniques/T1040/ |
| Cellular communications interception | T1573 | Encrypted Channel | https://attack.mitre.org/techniques/T1573/ |
| Data destruction (logic bomb payload) | T1485 | Data Destruction | https://attack.mitre.org/techniques/T1485/ |

---

## References and Further Reading

- **DEF CON 21 - iSEC Partners Femtocell Research (2013)**: "I Can Hear You Now: Traffic Interception and Remote Mobile Phone Cloning with a Compromised CDMA Femtocell" by Doug DePerry and Tom Ritter.
- **DEF CON 18 - Chris Paget IMSI Catcher Demo (2010)**: Demonstrated building a $1,500 IMSI catcher with commodity hardware.
- **Karsten Nohl - GSM Cracking**: Research on cracking A5/1 encryption used in GSM networks.
- **United States v. Roger Duronio (2006)**: DOJ prosecution of UBS PaineWebber logic bomb case.
- **DHS Report on IMSI Catchers Near White House (2018)**: Confirmation of unauthorized cell-site simulators near sensitive government facilities.
- **SANS Institute - Logic Bombs and Insider Threats**: https://www.sans.org/white-papers/ - Papers on detection and prevention of insider-planted logic bombs.

---

## Search Tags

```
tags: [logic-bomb, femtocell, imsi-catcher, sdr, openbts, yatebts, gnu-radio, hackrf, usrp, social-engineering, wireshark, insider-threat]
season: 2
episode: 5
mitre: [T1053, T1200, T1557, T1566, T1040, T1573, T1485]
```
