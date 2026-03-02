# Season 3, Episode 3: eps3.2_legacy.so

## Episode Overview

This episode begins setting up one of Season 3's most critical technical operations: the FBI femtocell hack. The `.so` file extension (shared object, a Linux dynamic library) references legacy code and inherited systems -- fitting for an episode dealing with legacy telecommunications vulnerabilities and inherited loyalties. Darlene's role as an FBI informant introduces HUMINT (Human Intelligence) tradecraft into the show's primarily technical narrative, while the Dark Army continues to operate with layers of operational security that make them nearly untraceable.

---

## Hacks & Techniques

### FBI Femtocell Hack Setup

A **femtocell** is a small, low-power cellular base station typically designed for home or small office use to improve cellular coverage. It connects to the carrier's core network via a standard broadband internet connection. The plan involves deploying a modified femtocell inside or near an FBI field office to intercept cellular communications.

**How a femtocell hack works:**

1. **Acquisition**: Obtain a commercial femtocell unit (e.g., from AT&T, Verizon, or similar carrier)
2. **Firmware extraction**: Dump the firmware via UART/JTAG debug interfaces
3. **Root access**: Exploit vulnerabilities to gain root shell access on the femtocell's embedded Linux system
4. **Traffic interception**: Modify the firmware to capture and forward all voice, SMS, and data traffic passing through the device
5. **Deployment**: Place the modified femtocell in proximity to the target. Mobile phones will automatically connect to it if it presents a stronger signal than legitimate towers.

**Technical architecture of a femtocell:**

```
[Target Mobile Phone]
        |
        | (Radio interface - LTE/3G/2G)
        |
[Modified Femtocell]
        |
        | (IPsec tunnel over broadband - intercepted)
        |
[Carrier Core Network / EPC]
        |
[Attacker's Collection Server]
```

### Femtocell Technical Details (DEF CON 21 Research)

The show's femtocell hack draws directly from real security research presented at DEF CON 21 (2013) by researchers including iSEC Partners (now NCC Group):

**Key research findings referenced:**

- **iSEC Partners (DEF CON 21, 2013)**: Demonstrated that commercial femtocells from multiple carriers contained exploitable vulnerabilities allowing root access
- **Firmware signing bypass**: Many femtocells either did not verify firmware signatures or used weak verification that could be bypassed
- **IPsec tunnel manipulation**: The secure tunnel between femtocell and carrier core network could be intercepted or redirected
- **IMSI disclosure**: Connected phones' IMSI (International Mobile Subscriber Identity) numbers were exposed in cleartext within the femtocell's local processing

```bash
# Femtocell firmware extraction via UART
screen /dev/ttyUSB0 115200    # Connect to UART debug port

# Once root shell is obtained:
cat /proc/cpuinfo             # Identify processor architecture
mount                         # Identify filesystem layout
iptables -L -n                # View firewall rules
tcpdump -i any -w capture.pcap  # Begin capturing all traffic

# Firmware modification for persistent access
dd if=/dev/mtd0 of=firmware_backup.bin   # Backup original firmware
# Modify firmware to include traffic capture and exfiltration
dd if=modified_firmware.bin of=/dev/mtd0  # Flash modified firmware
```

**Signal strength attack:**

The modified femtocell is configured to broadcast at a higher power level than surrounding legitimate cell towers, causing target phones to perform a handover to the rogue base station:

```
Legitimate Tower Signal: -85 dBm (weaker)
Modified Femtocell Signal: -65 dBm (stronger)

Phone behavior: Automatically connects to strongest signal
Result: All cellular traffic routed through attacker-controlled device
```

### Burner Phones and Dead Drops

Characters employ classic counter-surveillance tradecraft alongside digital techniques:

**Burner phone operational security:**

- **Purchase with cash**: No credit card or loyalty card association
- **Activation without identity**: Use prepaid SIM cards that do not require ID verification (jurisdiction-dependent)
- **Limited use**: Each phone is used for a minimal number of communications, then destroyed
- **Battery removal**: Batteries are removed (or phones powered off and stored in Faraday bags) when not actively in use to prevent location tracking
- **Pattern discipline**: Never carry a burner phone alongside a personal phone; never use a burner from home or work locations

**Dead drop methodology:**

- **Physical dead drops**: Pre-arranged locations where information (USB drives, SD cards, written messages) is left for retrieval by another party
- **Digital dead drops**: Shared accounts (email drafts, cloud storage) where messages are composed but never sent, only saved as drafts
- **Timing protocols**: Specific schedules for drop placement and retrieval to minimize surveillance risk

```bash
# Digital dead drop using a shared email draft folder
# Both parties have credentials to the same account
# Messages are saved as drafts, never sent (no email metadata generated)

# Dead drop via shared cloud storage with encryption
gpg --symmetric --cipher-algo AES256 -o deadrop_message.gpg message.txt
# Upload encrypted file to a shared anonymous cloud account
# Recipient downloads and decrypts with the pre-shared passphrase
gpg --decrypt deadrop_message.gpg
```

### Double Agent HUMINT Tradecraft (Darlene as FBI Informant)

Darlene's position as both an fsociety member and FBI informant introduces classical intelligence tradecraft:

**Handler-agent relationship:**

- **Handler**: FBI Agent Dom DiPierro manages Darlene as a Confidential Human Source (CHS)
- **Meetings**: Conducted in controlled environments (safe houses) or through brief, planned encounters (brush passes)
- **Communication protocols**: Pre-arranged signals for requesting emergency meetings or aborting contact
- **Intelligence reporting**: Darlene provides actionable intelligence on fsociety and potentially the Dark Army

**Double agent risks:**

- **Divided loyalty**: Darlene's true allegiance is ambiguous -- she may be feeding the FBI genuine intelligence, disinformation, or a mixture
- **Operational exposure**: If the Dark Army discovers Darlene is informing, the consequences are lethal
- **Information flow control**: Each side only receives information Darlene chooses to share, giving her significant power (and risk)
- **Polygraph and verification**: Law enforcement agencies typically require periodic verification of informant reliability

**Counter-intelligence considerations:**

- The Dark Army employs counter-intelligence practices and would be monitoring for informants
- Behavioral changes in known associates (Darlene) would be flagged
- Communication pattern analysis could reveal unauthorized contacts with law enforcement

### Dark Army Operational Security Layers

The Dark Army's OPSEC model employs multiple independent security layers:

1. **Layer 1 - Identity**: Operatives use false identities, never their real names or documentation
2. **Layer 2 - Communication**: All communications are encrypted, ephemeral, and conducted through anonymizing networks
3. **Layer 3 - Compartmentalization**: Strict need-to-know; operatives know only their immediate task and handler
4. **Layer 4 - Physical security**: Counter-surveillance detection routes (SDRs), meeting protocols, and clean rooms for sensitive discussions
5. **Layer 5 - Elimination**: Operatives who are compromised or have completed their usefulness are eliminated to prevent information leakage (the Dark Army's most extreme OPSEC measure)

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **Commercial femtocell** | Base hardware for rogue cellular base station |
| **UART/JTAG interfaces** | Debug access for firmware extraction |
| **tcpdump** | Packet capture on the femtocell's network interfaces |
| **Burner phones** | Disposable communication devices |
| **GPG** | Symmetric encryption for dead drop communications |
| **Faraday bags** | RF shielding to prevent phone tracking when not in use |
| **Screen/minicom** | Serial terminal for UART communication |

---

## Commands Shown

```bash
# UART connection to femtocell debug port
screen /dev/ttyUSB0 115200
# Alternative: minicom -D /dev/ttyUSB0 -b 115200

# Firmware extraction from flash memory
dd if=/dev/mtd0 of=/tmp/femtocell_firmware.bin bs=1M

# Analyze femtocell firmware
binwalk femtocell_firmware.bin
binwalk -e femtocell_firmware.bin   # Extract embedded filesystems

# Network traffic capture on femtocell
tcpdump -i eth0 -w /tmp/capture.pcap
tcpdump -i rmnet0 -w /tmp/cellular_capture.pcap  # Capture cellular-side traffic

# Check connected devices (IMSI numbers)
cat /proc/net/ipsec_eroute          # View IPsec tunnels
sqlite3 /var/db/femtocell.db "SELECT imsi,imei FROM connected_devices;"

# GPG encryption for dead drop communications
gpg --gen-key                        # Generate key pair
gpg -e -r recipient@example.com message.txt   # Encrypt message
gpg -d message.txt.gpg              # Decrypt message

# Secure file deletion after dead drop retrieval
shred -vfz -n 5 message.txt
```

---

## Real-World Parallels

### Femtocell Security Research
- **DEF CON 21 (2013)**: iSEC Partners presented "I Can Hear You Now: Traffic Interception and Remote Mobile Phone Cloning with a Compromised CDMA Femtocell," demonstrating real-world femtocell exploitation.
- **DEF CON 22 (2014)**: Further research showed that femtocells from multiple carriers and manufacturers shared common vulnerability classes.
- **FCC enforcement**: The FCC has taken action against unauthorized use of femtocells and signal boosters, recognizing the security implications.

### IMSI Catchers and Stingrays
- **Harris Corporation Stingray**: Law enforcement agencies use purpose-built IMSI catchers (also called cell-site simulators) that function similarly to a modified femtocell, forcing target phones to connect.
- **ACLU Stingray tracking**: The ACLU has documented the use of Stingray devices by over 75 law enforcement agencies across the United States.
- **Carpenter v. United States (2018)**: Supreme Court case addressing cell-site location information privacy, relevant to the surveillance themes in this episode.

### Intelligence Tradecraft
- **Aldrich Ames / Robert Hanssen**: CIA and FBI double agents who operated for years, demonstrating both the power and the eventual failure of double agent tradecraft.
- **Dead drop operations**: The FBI's arrest of Russian sleeper agents in 2010 (Operation Ghost Stories) revealed extensive use of dead drops and digital dead drops for communication.
- **Confidential informant programs**: FBI and DEA informant handling procedures mirror the Darlene-DiPierro relationship depicted in the show.

## Tool Links

- [OpenBTS](http://openbts.org/) - Open-source GSM base station software used in femtocell research
- [GNU Radio](https://www.gnuradio.org/) - Signal processing framework for software-defined radio
- [USRP (Ettus Research)](https://www.ettus.com/) - Software-defined radio hardware for cellular research
- [HackRF](https://greatscottgadgets.com/hackrf/) - SDR platform for RF spectrum analysis
- [Wireshark](https://www.wireshark.org/) - Network protocol analysis and packet capture
- [GnuPG](https://gnupg.org/) - Encryption for dead drop communications
- [Binwalk](https://github.com/ReFirmLabs/binwalk) - Femtocell firmware analysis and extraction
- [Tor](https://www.torproject.org/) - Anonymity network for Dark Army communications

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Femtocell as interception device | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| Firmware extraction via UART/JTAG | T1059 | Command and Scripting Interpreter | https://attack.mitre.org/techniques/T1059/ |
| Cellular communications interception | T1040 | Network Sniffing | https://attack.mitre.org/techniques/T1040/ |
| Digital dead drops for communication | T1102 | Web Service | https://attack.mitre.org/techniques/T1102/ |
| Use of burner phones to avoid tracking | T1036 | Masquerading | https://attack.mitre.org/techniques/T1036/ |
| Communication encryption with GPG | T1573 | Encrypted Channel | https://attack.mitre.org/techniques/T1573/ |
| Secure evidence deletion | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| Darlene as dual HUMINT source | T1557 | Adversary-in-the-Middle | https://attack.mitre.org/techniques/T1557/ |

## References and Further Reading

- **iSEC Partners (DEF CON 21, 2013)**: "I Can Hear You Now: Traffic Interception and Remote Mobile Phone Cloning with a Compromised CDMA Femtocell"
- **DEF CON 22 (2014)**: Additional research on vulnerabilities in femtocells from multiple manufacturers
- **Carpenter v. United States (2018)**: US Supreme Court decision on cell-site location information privacy
- **ACLU Stingray Tracking Project**: Documentation of IMSI catcher use by law enforcement agencies in the US
- **Operation Ghost Stories (2010)**: FBI case of Russian agents using physical and digital dead drops
- **Chris Paget (DEF CON 18, 2010)**: IMSI catcher demonstration for $1,500

## Search Tags

```
tags: [femtocell, IMSI-catcher, UART, JTAG, firmware, GPG, dead-drop, burner-phone, HUMINT, Darlene, FBI, Dark-Army, OPSEC]
season: 3
episode: 3
mitre: [T1200, T1059, T1040, T1102, T1036, T1573, T1070, T1557]
```
