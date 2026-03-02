# Episode 7: eps4.6_407proxyauthenticationrequired.h

## Season 4, Episode 7 | "407 Proxy Authentication Required"

**Air Date:** November 17, 2019
**Directed by:** Sam Esmail

### Overview

The episode title references HTTP status code 407, which requires the client to authenticate with an intermediary proxy server before the request can proceed. This mirrors the episode's themes of operating through intermediaries, proxy layers, and the need to authenticate one's true identity versus projected identities. The episode explores proxy chains for anonymity, surveillance camera exploitation, and the ongoing theme of identity manipulation.

---

## Hacks & Techniques

### 1. Proxy Chains and Traffic Obfuscation

To hide their activities from both the Dark Army and law enforcement, characters route their traffic through multiple proxy layers.

#### Proxy Chain Architecture

```
Operator's Machine
       |
  [SOCKS5 Proxy 1] (VPS in Country A)
       |
  [Tor Entry Node]
       |
  [Tor Relay Nodes] (3+ hops through Tor network)
       |
  [Tor Exit Node]
       |
  [SOCKS5 Proxy 2] (VPS in Country B)
       |
  [VPN Server] (VPS in Country C)
       |
  Target Server
```

#### Proxychains

Proxychains is a Linux tool that forces any TCP connection to go through specified proxy servers:

- **Dynamic Chaining:** Tries each proxy in order; if one is dead, it skips to the next.
- **Strict Chaining:** All proxies must be available and are used in exact order.
- **Random Chaining:** Randomly selects proxies from the list for each connection, making traffic patterns less predictable.

#### Tor (The Onion Router)

Tor routes traffic through a minimum of three volunteer-operated relays, encrypting the data in layers:

- **Entry/Guard Node:** Knows the user's IP but not the destination.
- **Middle Relay:** Knows neither the source nor the destination.
- **Exit Node:** Knows the destination but not the user's IP.
- **Onion Routing:** Each relay decrypts one layer of encryption, revealing only the next hop.

#### VPN Chains

Multiple VPN connections stacked on top of each other:

- **VPN over VPN:** Connect to VPN1, then within that tunnel, connect to VPN2.
- **Benefit:** No single VPN provider can see both the source and destination.
- **Limitation:** Significant latency; requires trust that VPN providers do not log or cooperate.

#### SOCKS5 Proxies

SOCKS5 is a proxy protocol that handles any type of traffic:

- **No Encryption by Default:** SOCKS5 itself does not encrypt traffic (must be combined with SSH tunnels or TLS).
- **UDP Support:** Unlike SOCKS4, SOCKS5 supports UDP traffic.
- **Authentication:** Supports username/password authentication.
- **DNS Resolution:** Can resolve DNS through the proxy, preventing DNS leaks.

### 2. HTTP 407 Proxy Authentication

The 407 status code is functionally similar to 401 (Unauthorized) but specifically for proxy servers:

- **Proxy-Authenticate Header:** The proxy includes this header in the 407 response, specifying the authentication scheme.
- **Proxy-Authorization Header:** The client includes this header in subsequent requests with authentication credentials.
- **Corporate Environments:** 407 is commonly encountered in corporate networks where all web traffic must pass through an authenticated proxy.

### 3. Surveillance Camera System Compromise

The episode features the compromise of surveillance camera systems, a technique that provides real-time intelligence on target locations.

#### IP Camera Attack Surface

```
Attack Vectors for IP Cameras:
+------------------------------------------+
| 1. Default Credentials                   |
|    admin:admin, admin:12345, root:root   |
+------------------------------------------+
| 2. Firmware Vulnerabilities              |
|    Buffer overflows, command injection   |
+------------------------------------------+
| 3. RTSP Stream Hijacking                 |
|    rtsp://camera-ip:554/stream           |
+------------------------------------------+
| 4. ONVIF Protocol Exploitation           |
|    Web service enumeration and control   |
+------------------------------------------+
| 5. UPnP Misconfiguration                |
|    Exposed to internet via port mapping  |
+------------------------------------------+
| 6. Cloud API Vulnerabilities             |
|    Insecure cloud management platforms   |
+------------------------------------------+
```

#### Camera Discovery and Enumeration

- **Shodan:** Search engine for internet-connected devices. Queries like `webcam`, `RTSP`, `camera`, or `port:554` reveal exposed cameras.
- **Censys:** Similar to Shodan, provides insights into internet-connected device configurations.
- **ONVIF Device Manager:** Discovers and interacts with ONVIF-compliant cameras on a network.

#### RTSP (Real-Time Streaming Protocol)

RTSP is the standard protocol for IP camera streams:

- **Default Port:** 554 (TCP)
- **Stream URLs:** Vary by manufacturer but often follow predictable patterns.
- **Authentication:** Often Basic authentication (base64-encoded, not encrypted) or no authentication at all.

### 4. Default Credentials on IP Cameras (CCTV Vulnerabilities)

One of the most common and devastating vulnerabilities in surveillance systems is the use of default or weak credentials.

#### Common Default Credentials by Manufacturer

| Manufacturer | Default Username | Default Password |
|---|---|---|
| **Hikvision** | admin | 12345 |
| **Dahua** | admin | admin |
| **Axis** | root | pass |
| **Samsung** | admin | 4321 |
| **Bosch** | (none) | (none) |
| **Vivotek** | root | (none) |
| **Foscam** | admin | (none) |
| **Ubiquiti** | ubnt | ubnt |

#### CCTV Vulnerability Categories

- **Authentication Bypass:** Many cameras have backdoor accounts or firmware vulnerabilities that bypass authentication entirely.
- **Hardcoded Credentials:** Some cameras contain credentials embedded in firmware that cannot be changed.
- **Command Injection:** Web interfaces that pass user input directly to system commands.
- **Path Traversal:** Accessing files outside the web root to read configuration files or credentials.
- **Firmware Extraction:** Dumping and analyzing firmware to find hidden credentials and vulnerabilities.

### 5. Identity Manipulation and Social Engineering

The episode continues the series' exploration of identity manipulation as a core hacking technique.

#### Identity Manipulation Techniques

- **Document Forgery:**
  - Creating or modifying identity documents (driver's licenses, employee badges, business cards).
  - Using high-quality printers and laminators for physical documents.
  - Photoshop or GIMP for digital document modification.

- **Digital Identity Creation:**
  - Creating convincing fake social media profiles with history and connections.
  - Establishing fake business entities with websites, phone numbers, and email addresses.
  - Using AI-generated profile photos (StyleGAN) to create realistic but non-existent people.

- **Voice Manipulation:**
  - Voice changers for phone-based social engineering.
  - AI voice synthesis (deepfake audio) to impersonate specific individuals.
  - Caller ID spoofing to display any desired phone number.

- **Behavioral Mimicry:**
  - Studying the target's mannerisms, vocabulary, and communication style.
  - Using insider knowledge and jargon to establish credibility.
  - Projecting confidence and authority to discourage verification.

---

## Tools Used

| Tool | Purpose |
|---|---|
| **Proxychains** | Route traffic through multiple proxy servers |
| **Tor** | Anonymous network routing through onion relays |
| **OpenVPN / WireGuard** | VPN tunnel establishment |
| **Shodan** | Internet-connected device discovery (cameras) |
| **Nmap** | Network scanning for camera discovery |
| **ONVIF Device Manager** | IP camera discovery and management |
| **VLC Media Player** | RTSP stream viewing |
| **Hydra** | Brute-force credentials on camera login pages |
| **Burp Suite** | Web application testing on camera interfaces |
| **Cameradar** | Automated RTSP stream discovery and access |

---

## Commands Shown

### Proxychains Configuration and Usage

```bash
# /etc/proxychains4.conf
# Dynamic chain - skip dead proxies
dynamic_chain

# Proxy list
[ProxyList]
socks5 192.168.1.100 1080 user1 pass1
socks5 10.0.0.50 9050
socks5 172.16.0.25 1080 user2 pass2

# Usage with any tool
proxychains4 nmap -sT -Pn target-ip
proxychains4 curl https://target-website.com
proxychains4 ssh user@target-server
```

### Tor Configuration

```bash
# Install and start Tor
sudo apt install tor
sudo systemctl start tor

# Tor SOCKS proxy runs on 127.0.0.1:9050
# Use with proxychains or directly

# Route traffic through Tor
torsocks curl https://check.torproject.org

# Change Tor circuit (get new exit node)
sudo killall -HUP tor

# Tor hidden service configuration (/etc/tor/torrc)
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:80
```

### SSH SOCKS5 Proxy

```bash
# Create SOCKS5 proxy through SSH tunnel
ssh -D 9050 -f -C -q -N user@proxy-server

# Chain SSH tunnels
ssh -L 9051:localhost:9050 user@hop1 -t ssh -D 9050 user@hop2

# Dynamic port forwarding through multiple hops
ssh -J user@hop1,user@hop2 -D 1080 user@final-hop
```

### IP Camera Discovery and Access

```bash
# Discover cameras on local network using Nmap
nmap -sV -p 554,80,443,8080,8443 --script rtsp-url-brute 192.168.1.0/24

# Shodan CLI search for exposed cameras
shodan search "RTSP/1.0" --fields ip_str,port,org
shodan search "webcamXP" --fields ip_str,port
shodan search "Server: IP Webcam Server" --fields ip_str,port

# Access RTSP stream with VLC
vlc rtsp://admin:12345@camera-ip:554/Streaming/Channels/101

# Common RTSP URL patterns
# Hikvision: rtsp://user:pass@ip:554/Streaming/Channels/101
# Dahua:     rtsp://user:pass@ip:554/cam/realmonitor?channel=1&subtype=0
# Axis:      rtsp://user:pass@ip:554/axis-media/media.amp

# Brute-force camera credentials
hydra -l admin -P /usr/share/wordlists/rockyou.txt camera-ip http-get /
hydra -l admin -P passwords.txt camera-ip rtsp

# Use Cameradar for automated RTSP discovery
cameradar -t 192.168.1.0/24 -p 554,8554
```

### Camera Firmware Analysis

```bash
# Download and extract firmware
wget http://firmware-url/camera_firmware.bin
binwalk -e camera_firmware.bin

# Search for credentials in extracted firmware
grep -rn "password\|passwd\|admin" _camera_firmware.bin.extracted/

# Find hardcoded credentials
strings camera_firmware.bin | grep -i "password\|admin\|root"
```

---

## Real-World Parallels

### Proxy Chains and Anonymity

- **Silk Road:** Ross Ulbricht operated the Silk Road dark web marketplace exclusively through Tor, demonstrating both the power and limitations of onion routing for anonymity. His eventual identification came from operational security mistakes, not Tor compromise.
- **APT Groups:** Advanced Persistent Threat groups routinely chain multiple proxies across jurisdictions to complicate attribution. APT28 (Fancy Bear) has been documented using VPN services, Tor, and compromised infrastructure in multiple countries as proxy chains.
- **Tor Vulnerabilities:** While Tor provides strong anonymity, attacks such as traffic correlation (observing both entry and exit nodes) and Tor exit node eavesdropping have been demonstrated. In 2014, the FBI de-anonymized Tor users on the Silk Road 2.0 through a combination of techniques.

### IP Camera Vulnerabilities

- **Mirai Botnet (2016):** The Mirai malware infected hundreds of thousands of IoT devices, primarily IP cameras and DVRs with default credentials, creating a massive botnet that launched the largest DDoS attacks in history (including the Dyn DNS attack that took down Twitter, Netflix, and Reddit).
- **Hikvision Backdoor (2017):** A critical vulnerability (CVE-2017-7921) was discovered in Hikvision cameras that allowed remote attackers to gain full administrator access by sending a specially crafted request, bypassing all authentication. This affected millions of cameras worldwide.
- **Verkada Breach (2021):** Hackers gained access to over 150,000 surveillance cameras from security startup Verkada, viewing live feeds from hospitals, prisons, schools, and Tesla factories. The initial access was through a super admin account exposed on the internet.
- **Ring Camera Hacks (2019):** Multiple incidents of attackers accessing Ring home security cameras using credential stuffing attacks (reused passwords from other breaches), speaking to families through the camera's speaker.

### Identity Manipulation

- **Deepfake Audio in CEO Fraud (2019):** Criminals used AI-generated voice technology to impersonate the CEO of a German energy company, convincing a UK subsidiary's managing director to wire $243,000 to a Hungarian bank account. This was one of the first known cases of AI voice deepfakes used in financial fraud.
- **Operation Newscaster (Iran):** Iranian state-sponsored hackers created an elaborate network of fake social media profiles, including fake journalists and recruiters, to build relationships with and extract intelligence from US military and government officials over several years.

## Tool Links

- [Proxychains](https://github.com/haad/proxychains) - Traffic routing through multiple proxy servers
- [Tor](https://www.torproject.org/) - Anonymous routing via the onion network
- [Shodan](https://www.shodan.io/) - Internet-connected device discovery (cameras)
- [Censys](https://censys.io/) - Device discovery and monitoring platform
- [Nmap](https://nmap.org/) - Network scanning for camera discovery
- [Hashcat](https://hashcat.net/hashcat/) - Credential brute-force (contextual reference)
- [Sherlock](https://github.com/sherlock-project/sherlock) - Digital identity search across platforms
- [GnuPG](https://gnupg.org/) - Communication encryption and verification

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Proxy chains for traffic obfuscation | T1572 | Protocol Tunneling | https://attack.mitre.org/techniques/T1572/ |
| Routing via Tor | T1573 | Encrypted Channel | https://attack.mitre.org/techniques/T1573/ |
| Surveillance camera compromise | T1133 | External Remote Services | https://attack.mitre.org/techniques/T1133/ |
| Default credentials on IP cameras | T1078 | Valid Accounts | https://attack.mitre.org/techniques/T1078/ |
| Camera credential brute-force | T1110 | Brute Force | https://attack.mitre.org/techniques/T1110/ |
| Identity manipulation and forgery | T1036 | Masquerading | https://attack.mitre.org/techniques/T1036/ |
| Camera firmware reconnaissance | T1082 | System Information Discovery | https://attack.mitre.org/techniques/T1082/ |
| SSH SOCKS5 proxy tunneling | T1572 | Protocol Tunneling | https://attack.mitre.org/techniques/T1572/ |

## References and Further Reading

- Silk Road and Tor - Power and limitations of onion routing for anonymity
- Mirai Botnet (2016) - Massive infection of IP cameras and DVRs with default credentials
- CVE-2017-7921 (Hikvision Backdoor) - Authentication bypass on millions of cameras
- Verkada Breach (2021) - Access to 150,000+ surveillance cameras via super admin account
- Ring Camera Hacks (2019) - Credential stuffing on home security cameras
- Deepfake Audio CEO Fraud (2019) - AI use for voice impersonation in financial fraud
- Operation Newscaster (Iran) - Elaborate network of fake profiles for espionage
- APT28 Proxy Chain Attribution - Documented use of VPN, Tor, and compromised infrastructure

## Search Tags

```
tags: [proxychains, tor, shodan, censys, nmap, RTSP, IP-camera, proxy-chain, SOCKS5, VPN, identity-manipulation, deepfake, default-credentials, CCTV, surveillance]
season: 4
episode: 7
mitre: [T1572, T1573, T1133, T1078, T1110, T1036, T1082]
```
