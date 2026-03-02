# Season 2, Episode 7: eps2.5_h4ndshake.sme

## Episode Overview

The episode title references handshake protocols -- the negotiation processes that establish secure communications between two parties. In networking, handshakes are foundational: they authenticate identities, agree on encryption parameters, and establish the ground rules for communication. Thematically, the episode revolves around trust negotiations and betrayals. A critical plot point is Elliot's decision to anonymously report Ray's dark web site (which hosts human trafficking, among other horrifying content) to the FBI, using anonymous channels while knowing the FBI actively seeks to identify anonymous sources.

---

## Hacks & Techniques

### 1. Handshake Protocols: Technical Deep Dive

A handshake in networking is a structured exchange of messages between two parties to establish parameters for communication. There are several critical handshake types, each serving different purposes.

**TCP Three-Way Handshake:**

The foundation of all TCP/IP communication. Every web request, email, file transfer, and database connection begins with this handshake.

```
Client                          Server
  |                               |
  |  1. SYN (seq=x)              |
  |------------------------------>|
  |                               |
  |  2. SYN-ACK (seq=y, ack=x+1) |
  |<------------------------------|
  |                               |
  |  3. ACK (seq=x+1, ack=y+1)   |
  |------------------------------>|
  |                               |
  |  [Connection Established]     |
  |  [Data Transfer Begins]       |
```

- **Step 1 (SYN)**: Client sends a TCP segment with the SYN flag set and a random initial sequence number
- **Step 2 (SYN-ACK)**: Server responds with SYN and ACK flags, its own sequence number, and acknowledgment of the client's sequence number
- **Step 3 (ACK)**: Client acknowledges the server's sequence number, completing the handshake

**Attacks Against the TCP Handshake:**

- **SYN Flood**: Sending many SYN packets without completing the handshake, exhausting the server's connection table (a classic DDoS technique)
- **TCP Reset Attack**: Injecting RST packets to tear down established connections
- **Sequence Number Prediction**: If sequence numbers are predictable, an attacker can inject packets into an established connection

**TLS/SSL Handshake (TLS 1.3):**

The TLS handshake establishes an encrypted channel on top of a TCP connection. TLS 1.3 (the current version) simplified the handshake significantly.

```
Client                                    Server
  |                                         |
  |  1. ClientHello                         |
  |    - Supported cipher suites            |
  |    - Key share (ECDHE)                  |
  |    - Supported TLS versions             |
  |---------------------------------------->|
  |                                         |
  |  2. ServerHello                         |
  |    - Selected cipher suite              |
  |    - Key share (ECDHE)                  |
  |    - {EncryptedExtensions}              |
  |    - {Certificate}                      |
  |    - {CertificateVerify}               |
  |    - {Finished}                         |
  |<----------------------------------------|
  |                                         |
  |  3. {Finished}                          |
  |---------------------------------------->|
  |                                         |
  |  [Encrypted Application Data]           |
  |<--------------------------------------->|
```

- **1-RTT**: TLS 1.3 completes in a single round trip (vs. two in TLS 1.2)
- **0-RTT**: For resumed connections, TLS 1.3 supports sending data in the first message
- **Forward Secrecy**: Mandatory ECDHE key exchange ensures past sessions cannot be decrypted even if the server's private key is compromised
- **Simplified Cipher Suites**: Only AEAD ciphers (AES-GCM, ChaCha20-Poly1305) are allowed

**WPA Four-Way Handshake (Wi-Fi):**

The WPA/WPA2 four-way handshake authenticates a device to a Wi-Fi network and establishes the session encryption keys.

```
Access Point (Authenticator)          Client (Supplicant)
         |                                    |
         |  1. ANonce (AP's random nonce)     |
         |----------------------------------->|
         |                        [Client computes PTK from:
         |                         PMK + ANonce + SNonce +
         |                         MAC addresses]
         |  2. SNonce + MIC                   |
         |<-----------------------------------|
         |  [AP computes PTK,                 |
         |   verifies MIC]                    |
         |  3. GTK (encrypted) + MIC          |
         |----------------------------------->|
         |                                    |
         |  4. ACK                            |
         |<-----------------------------------|
         |                                    |
         | [Encrypted communication begins]   |
```

**Key Terms:**
- **PMK** (Pairwise Master Key): Derived from the Wi-Fi password and SSID
- **PTK** (Pairwise Transient Key): Session key derived from the PMK and nonces
- **GTK** (Group Temporal Key): Key for broadcast/multicast traffic
- **ANonce/SNonce**: Random values from AP and client
- **MIC** (Message Integrity Code): Proves knowledge of the PMK without revealing it

**Attacks Against the WPA Handshake:**

- **KRACK Attack (Key Reinstallation Attack)**: Discovered in 2017, forces nonce reuse in the four-way handshake, allowing decryption of traffic
- **Handshake Capture + Offline Dictionary Attack**: Capture the four-way handshake and attempt to derive the PMK offline by testing passwords
- **PMKID Attack**: Extract the PMKID from the first frame of the handshake and crack it offline (does not require a full handshake capture)

### 2. Elliot Reports Ray's Dark Web Site to the FBI

After discovering that Ray's marketplace hosts human trafficking, Elliot decides to report it to the FBI. This creates a paradox: he must communicate with law enforcement while being a wanted hacker himself, requiring sophisticated anonymous communication techniques.

**The Dilemma:**

- Elliot wants the FBI to shut down Ray's site
- Elliot cannot reveal his own identity as he is involved in the 5/9 hack
- The FBI actively uses techniques to de-anonymize tipsters, especially for dark web cases
- The tip must contain enough specific information to be actionable without revealing how Elliot obtained it

### 3. Anonymous Reporting Methods

**Using Tor for Anonymous Communication:**

```bash
# Access FBI tip page through Tor
torbrowser-launcher

# Or via command line with torsocks
torsocks curl https://tips.fbi.gov

# Using Tor SOCKS proxy directly
curl --socks5-hostname 127.0.0.1:9050 https://tips.fbi.gov
```

**Anonymous Email Services:**

- **ProtonMail**: End-to-end encrypted email based in Switzerland. Can be accessed over Tor. Does not require personally identifying information to create an account.
- **Guerrilla Mail**: Disposable email service for one-time communications
- **Tutanota**: Another encrypted email provider with anonymous registration

**Creating an Anonymous ProtonMail Account:**

1. Access ProtonMail's `.onion` address through Tor Browser
2. Create account without providing any identifying information
3. Do not use any username connected to other identities
4. Compose the tip with only information that could plausibly come from a general user who stumbled onto the site
5. Send from a public Wi-Fi location (never from a personally associated network)
6. Never access this account from a non-Tor connection
7. Destroy the account after use

**Other Anonymous Tip Channels:**

- **SecureDrop**: Open-source whistleblower submission system used by many news organizations. Operates as a Tor hidden service with airgapped processing.
- **FBI Tips Website**: `tips.fbi.gov` -- while the FBI will attempt to identify the tipster, using Tor provides protection
- **IC3 (Internet Crime Complaint Center)**: `ic3.gov` -- accepts cybercrime complaints
- **Phone tips via burner phone**: Prepaid phones purchased with cash from locations without surveillance cameras

### 4. Operational Security for Anonymous Reporting

**OPSEC Measures:**

1. **Network anonymity**: Only access reporting channels through Tor, never directly
2. **Physical anonymity**: If using a public network, choose a location without cameras and avoid patterns (do not use the same location repeatedly)
3. **Content sanitization**: Do not include any information that reveals how you know what you know
4. **Writing style**: Vary your writing style from your normal patterns (stylometry is a real identification technique)
5. **Timing**: Do not submit tips at times that correlate with your known schedule
6. **Device security**: Use a clean device (Tails OS booted from USB) that leaves no traces
7. **No cross-contamination**: Never use the anonymous identity for anything else

### 5. FBI De-anonymization Techniques

The FBI has developed sophisticated techniques for identifying anonymous internet users, especially in dark web investigations.

**Network Investigative Techniques (NITs):**

NITs are FBI-developed exploits deployed against Tor users. The most notable example:

- **Operation Torpedo (2012)**: The FBI seized a Tor hidden service hosting child exploitation material and replaced it with a NIT (a browser exploit) that caused visitors' computers to send their real IP addresses to an FBI server.

- **Playpen Investigation (2015)**: The FBI operated a seized child exploitation site for 13 days while deploying a NIT that exploited a Firefox vulnerability in the Tor Browser. This resulted in over 8,700 real IP addresses being collected and hundreds of prosecutions.

**How NITs Work:**

```
1. FBI seizes/compromises a Tor hidden service
2. FBI deploys exploit code (NIT) on the site
3. User visits the site through Tor Browser
4. NIT exploits a browser vulnerability
5. Malicious code executes outside Tor
6. User's real IP address is sent to FBI server
7. FBI subpoenas the ISP for subscriber info
```

**Traffic Correlation Attacks:**

- The FBI and NSA can potentially perform traffic correlation by monitoring both ends of a Tor circuit
- If an adversary can observe traffic entering Tor (at the entry node) and exiting Tor (at the exit node or hidden service), they can correlate timing and volume patterns
- This requires significant infrastructure but is within the capability of nation-state actors

**Browser Fingerprinting:**

- Even through Tor, browser characteristics (screen resolution, installed fonts, WebGL capabilities, JavaScript behavior) can create a unique fingerprint
- The Tor Browser is specifically designed to minimize fingerprinting, but zero-day vulnerabilities can bypass these protections

**Stylometric Analysis:**

- Writing style analysis can potentially identify anonymous authors by analyzing vocabulary, sentence structure, punctuation patterns, and other linguistic features
- The FBI has used linguistic analysis in cases like the Unabomber investigation

### 6. Silk Road Investigation Parallels

Ray's dark web marketplace and its takedown closely parallel the Silk Road investigation:

**How the FBI Found Silk Road:**

1. **OPSEC Failure**: Ross Ulbricht posted about Silk Road on a Bitcoin forum using an email address (`rossulbricht@gmail.com`) that he also used elsewhere
2. **Server Discovery**: An FBI agent claimed to have found the Silk Road server's real IP address through a "misconfigured" login page (the exact method remains controversial)
3. **Undercover Operations**: FBI agents operated as vendors and administrators within the marketplace
4. **Controlled Purchases**: Agents made purchases to establish the criminal nature of the marketplace
5. **Subpoenas and Surveillance**: Traditional law enforcement tools were used once initial leads were established

**Controversial Aspects:**

- It has been debated whether the FBI's discovery of the Silk Road server was through the claimed "leaky CAPTCHA" or through a parallel construction using NSA intelligence
- The Silk Road case raised significant questions about the Fourth Amendment rights of Tor users

### 7. MagSpoof Hotel Keycard Cloning

Darlene clones a maid's hotel keycard using a **MagSpoof** device, created by security researcher Samy Kamkar. MagSpoof uses an electromagnet to wirelessly emulate magnetic stripe cards. Instead of physically cloning the card, it captures the magnetic stripe data and generates the same electromagnetic pattern the card reader expects. The show's writers contacted Samy Kamkar, who created a custom version of the device specifically for the show.

**How MagSpoof Works:**

- MagSpoof can emulate any magnetic stripe card without physically touching the reader
- Uses an Arduino-based board (ATtiny85) with a copper coil
- Generates electromagnetic pulses that mimic the signal produced by a physical card swipe
- The device is small enough to conceal in a hand or sleeve
- Open-sourced on GitHub by Samy Kamkar

```
MagSpoof Components:
[Arduino/ATtiny85] --> [H-bridge driver] --> [Copper coil]
       |                                        |
  Stores card          Generates electromagnetic
  data digitally       field to emulate swipe
```

**Technical Details:**

Traditional magnetic stripe cards encode data using the F2F (Frequency/double Frequency) encoding scheme across up to three tracks. When a card is swiped, the magnetic field changes are read by the card reader's head. MagSpoof replaces this physical swipe by generating the same electromagnetic field changes using a copper coil driven by an H-bridge circuit. The card data is stored digitally on the microcontroller, and the coil pulses are timed to match the expected F2F encoding — making the card reader unable to distinguish between a real swipe and MagSpoof's emulation.

### 8. Cantenna (Pringles Can Antenna)

Darlene builds a directional antenna from a Pringles can to connect to the femtocell Angela planted on the FBI floor from a hotel room across the street. A cantenna is a homemade directional Wi-Fi antenna that focuses wireless signal in one direction, extending range significantly (up to ~1 mile line-of-sight).

**How a Cantenna Works:**

A standard DIY cantenna build uses a metal can with a specific diameter and probe length calculated for the target frequency (2.4 GHz for Wi-Fi). The can acts as a waveguide, focusing the radio frequency energy in a single direction rather than radiating it omnidirectionally. The probe (a short piece of copper wire soldered to a connector) is positioned at a precise distance from the closed end of the can, calculated as one-quarter of the guide wavelength.

**Key Design Parameters for 2.4 GHz:**

- Can diameter: ~83mm (a Pringles can is close to ideal)
- Probe length: ~31mm (quarter wavelength at 2.4 GHz)
- Probe distance from back wall: ~45mm (quarter guide wavelength)
- Connector: N-type female connector with copper probe soldered to center pin

**Use in the Episode:**

Darlene positions the cantenna in the hotel room window, aiming it at the FBI building across the street. This directional antenna allows her laptop to connect to the femtocell's Wi-Fi signal at a distance far beyond what a standard laptop Wi-Fi card could achieve, enabling the team to remotely access the FBI's intercepted communications without being physically present in the building.

---

## Tools Used

| Tool | Purpose |
|---|---|
| **Tor Browser** | Anonymous web access for tip submission |
| **Tails OS** | Amnesic operating system for leaving no traces |
| **ProtonMail** | End-to-end encrypted anonymous email |
| **SecureDrop** | Whistleblower submission system |
| **torsocks** | Routing arbitrary applications through Tor |
| **Wireshark** | Analyzing handshake protocols |
| **tcpdump** | Capturing network handshakes |

---

## Commands Shown

**Capturing and Analyzing TCP Handshakes:**

```bash
# Capture TCP handshake with tcpdump
tcpdump -i eth0 -c 10 'tcp[tcpflags] & (tcp-syn|tcp-ack) != 0' -w handshake.pcap

# Analyze with tshark
tshark -r handshake.pcap -Y "tcp.flags.syn==1" -T fields \
  -e ip.src -e ip.dst -e tcp.seq -e tcp.ack

# View TLS handshake details
tshark -r capture.pcap -Y "tls.handshake" -V
```

**TLS Handshake Inspection:**

```bash
# Connect to a server and display TLS handshake details
openssl s_client -connect example.com:443 -state -debug 2>&1 | \
  grep -E "SSL_connect|Cipher|Protocol"

# Show only the negotiated cipher and protocol
openssl s_client -connect example.com:443 2>/dev/null | \
  grep -E "Protocol|Cipher"

# Test specific TLS version
openssl s_client -connect example.com:443 -tls1_3
```

**WPA Handshake Capture (for authorized testing):**

```bash
# Put interface in monitor mode
airmon-ng start wlan0

# Capture WPA handshake
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon

# Force handshake by deauthenticating a client
aireplay-ng -0 1 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon

# Verify handshake was captured
aircrack-ng capture-01.cap
```

**Anonymous Tip Submission via Tor:**

```bash
# Boot Tails OS from USB (clean environment)
# Connect to public Wi-Fi

# Verify Tor connection
torsocks curl https://check.torproject.org/api/ip
# Response: {"IsTor":true,"IP":"x.x.x.x"}

# Access FBI tip site through Tor
torbrowser https://tips.fbi.gov

# Or use ProtonMail onion address
torbrowser https://protonmailrmez3lotccipshtkleegetolb73fuirgj7r4o4vfu7ozyd.onion
```

---

## Real-World Parallels

### TLS Handshake Vulnerabilities

- **Heartbleed (2014)**: A bug in OpenSSL's heartbeat extension allowed attackers to read server memory, potentially exposing private keys. This did not attack the handshake itself but undermined the trust model.

- **POODLE (2014)**: Exploited a vulnerability in the SSL 3.0 protocol's CBC mode padding to decrypt data. Led to the deprecation of SSL 3.0.

- **ROBOT (2017)**: "Return of Bleichenbacher's Oracle Threat" -- a 19-year-old vulnerability in RSA key exchange affected major websites including Facebook and PayPal.

- **Raccoon Attack (2020)**: A timing vulnerability in the TLS-DH(E) key exchange that could allow an attacker to compute the shared secret.

### FBI Dark Web Operations

- **Operation Pacifier (2015)**: The FBI's operation against the Playpen dark web site, where agents ran the site for 13 days while deploying NITs. This remains one of the most controversial law enforcement operations in cybercrime history.

- **Hansa Market Takeover (2017)**: Dutch police secretly operated the Hansa dark web marketplace for a month, collecting buyer and vendor identities, before coordinating its shutdown with the AlphaBay takedown.

- **Welcome to Video (2019)**: An international operation that took down a dark web child exploitation site. Blockchain analysis of Bitcoin payments was key to identifying users.

### Anonymous Whistleblowing

- **Edward Snowden (2013)**: Used encrypted email (Lavabit) and secure channels to communicate with journalists. His case highlighted the importance of anonymous communication tools.

- **SecureDrop adoption**: Major news organizations including The New York Times, The Washington Post, and The Guardian now operate SecureDrop instances for anonymous source communication.

- **The Tor Project's role**: Tor was originally developed by the US Naval Research Laboratory and continues to receive US government funding, even as the FBI develops techniques to circumvent it -- a paradox often discussed in privacy circles.

### KRACK Attack (2017)

- Researcher Mathy Vanhoef discovered that the WPA2 four-way handshake could be manipulated to reinstall an already-in-use key, resetting the nonce counter and allowing traffic decryption. This affected virtually every Wi-Fi device in the world and required patches from all major operating system and device vendors.

---

## Tool Links

| Tool | Link |
|---|---|
| Tor | https://www.torproject.org/ |
| Tor Hidden Services | https://community.torproject.org/onion-services/ |
| Signal | https://signal.org/ |
| OpenSSL | https://www.openssl.org/ |
| Wireshark | https://www.wireshark.org/ |
| MagSpoof | https://github.com/samyk/magspoof |

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Anonymous communication via Tor | T1572 | Protocol Tunneling | https://attack.mitre.org/techniques/T1572/ |
| Encrypted channel (TLS handshake) | T1573 | Encrypted Channel | https://attack.mitre.org/techniques/T1573/ |
| FBI NIT (browser exploit) | T1190 | Exploit Public-Facing Application | https://attack.mitre.org/techniques/T1190/ |
| Traffic correlation / network sniffing | T1040 | Network Sniffing | https://attack.mitre.org/techniques/T1040/ |
| Application layer protocol | T1071 | Application Layer Protocol | https://attack.mitre.org/techniques/T1071/ |
| WPA handshake capture | T1040 | Network Sniffing | https://attack.mitre.org/techniques/T1040/ |
| Hotel keycard cloning via MagSpoof | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |

---

## References and Further Reading

- **CVE-2014-0160** (Heartbleed): https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160
- **CVE-2014-3566** (POODLE): https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-3566
- **CVE-2017-13077** (KRACK Attack): https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-13077
- **DEF CON 25 - FBI Dark Web Operations**: Presentations on NIT deployments and Tor de-anonymization techniques.
- **SANS Institute - TLS Security**: https://www.sans.org/white-papers/ - Papers on TLS handshake security and configuration best practices.
- **Operation Pacifier Court Documents**: Legal proceedings revealing FBI's operation of the Playpen hidden service.
- **Raccoon Attack (2020)**: https://raccoon-attack.com/ - Research paper on timing vulnerability in TLS-DH(E).

---

## Search Tags

```
tags: [tor, tls, handshake, tcp, wpa, krack, anonymous-reporting, fbi, nit, opsec, protonmail, wireshark, openssl, MagSpoof, cantenna, magnetic-stripe, Samy-Kamkar, hotel-keycard]
season: 2
episode: 7
mitre: [T1572, T1573, T1190, T1040, T1071, T1200]
```
