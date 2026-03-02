# Season 3, Episode 9: eps3.8_stage3.torrent

## Episode Overview

The penultimate episode of Season 3 introduces "Stage 3" -- not another attack, but a recovery operation. The `.torrent` file extension references the BitTorrent protocol used for peer-to-peer file distribution, symbolizing the decentralized distribution of information and the power of mass disclosure. Elliot works to reverse the 5/9 encryption by recovering the original encryption keys, while concepts of anonymous information distribution, dead man's switches, and industrial control system protocols come to the forefront. The episode parallels real-world events around WikiLeaks, SecureDrop, and the tension between secrecy and transparency.

---

## Hacks & Techniques

### BitTorrent Protocol for Mass Data Distribution

BitTorrent is a peer-to-peer (P2P) file sharing protocol designed for efficient distribution of large files. Unlike client-server models where a single server must handle all download requests, BitTorrent distributes the load across all participating peers:

**How BitTorrent works:**

```
Traditional Download:
[Server] ---> [Client A]
[Server] ---> [Client B]      Server bandwidth = bottleneck
[Server] ---> [Client C]

BitTorrent Download:
[Seeder] ---> [Peer A] <---> [Peer B]
    |              |              |
    +-----------> [Peer C] <-----+

Each peer downloads different pieces and shares them with others
More peers = faster download (opposite of client-server)
```

**BitTorrent protocol components:**

- **Torrent file (.torrent)**: Contains metadata including file names, sizes, piece hashes (SHA-1), and tracker URLs
- **Tracker**: Server that coordinates peers; knows which peers have which pieces
- **DHT (Distributed Hash Table)**: Trackerless peer discovery using Kademlia protocol -- makes the system decentralized and censorship-resistant
- **Pieces**: Files are divided into fixed-size pieces (typically 256KB-4MB), each independently verifiable via its SHA-1 hash
- **Seeder**: Peer that has the complete file and only uploads
- **Leecher**: Peer that is still downloading

```bash
# Create a torrent file for mass distribution
mktorrent -v -a udp://tracker.example.com:1337/announce \
    -a udp://tracker2.example.com:6969/announce \
    -l 22 \          # Piece size: 2^22 = 4MB
    -c "E Corp financial records recovery keys" \
    -o ecorp_keys.torrent \
    /path/to/key_files/

# Start seeding using transmission-cli
transmission-cli ecorp_keys.torrent -w /path/to/key_files/ -u 0

# Using aria2 for multi-protocol download
aria2c --seed-time=0 ecorp_keys.torrent

# Magnet link (trackerless, DHT-based)
# magnet:?xt=urn:btih:<info_hash>&dn=ecorp_keys&tr=udp://tracker.example.com:1337
```

**WikiLeaks insurance file parallel:**

WikiLeaks has historically released "insurance files" -- large, AES-256 encrypted archives distributed via BitTorrent. The encryption key is only released if WikiLeaks is compromised or its operators are arrested. This serves as a dead man's switch:

```
Insurance File Distribution:
1. Create archive of sensitive documents
2. Encrypt with AES-256 (strong symmetric encryption)
3. Distribute encrypted archive via BitTorrent (thousands of copies)
4. Hold decryption key in reserve
5. If trigger event occurs: release key
6. All torrent holders can now decrypt the archive
```

### Reversing the 5/9 Encryption

Elliot's primary objective is to reverse the 5/9 hack by recovering the encryption keys used to encrypt E Corp's financial data:

**The encryption problem:**

- During the 5/9 hack, fsociety encrypted E Corp's financial databases using strong encryption (likely AES-256)
- The encryption keys were generated and potentially stored in HSMs
- Without the keys, the encrypted data is mathematically unrecoverable
- E Corp has been operating without access to its own financial records since 5/9

**Recovery approaches:**

1. **HSM key recovery**: Access the HSMs that generated the original encryption keys
2. **Key escrow**: Determine if copies of the keys were escrowed (backed up) anywhere
3. **Shamir's Secret Sharing**: Reconstruct keys that were split among multiple key holders
4. **Brute force**: Mathematically infeasible for AES-256 (2^256 possible keys)

### Key Recovery from HSMs and Shamir's Secret Sharing

**Shamir's Secret Sharing** is a cryptographic scheme that splits a secret (such as an encryption key) into N shares, of which any K shares (where K <= N) are sufficient to reconstruct the secret. Fewer than K shares reveal no information about the secret.

**How Shamir's Secret Sharing works:**

```
Original Key: K = "3a7b...f921" (256-bit AES key)

Split into 5 shares (3-of-5 threshold):
Share 1: "a1b2...c3d4" (held by Key Custodian A)
Share 2: "e5f6...g7h8" (held by Key Custodian B)
Share 3: "i9j0...k1l2" (held by Key Custodian C)
Share 4: "m3n4...o5p6" (held by Key Custodian D)
Share 5: "q7r8...s9t0" (held by Key Custodian E)

Any 3 of these 5 shares can reconstruct the original key K.
2 or fewer shares provide zero information about K.
```

**Mathematical basis (polynomial interpolation):**

```python
# Shamir's Secret Sharing using a polynomial over a finite field
# Secret S is the constant term of a random polynomial of degree (K-1)

# For a 3-of-5 scheme:
# f(x) = S + a1*x + a2*x^2  (degree 2 polynomial, random a1, a2)
# Share_i = (i, f(i)) for i = 1..5

# Recovery: Given any 3 points, Lagrange interpolation recovers f(x)
# S = f(0) = the original secret

# Python implementation using the 'secretsharing' library
from secretsharing import PlaintextToHexSecretSharer

# Split a secret into 5 shares with threshold 3
shares = PlaintextToHexSecretSharer.split_secret(
    "3a7bf921deadbeef",  # The secret (encryption key)
    3,                    # Threshold (minimum shares needed)
    5                     # Total shares
)
print("Shares:", shares)

# Reconstruct from any 3 shares
recovered = PlaintextToHexSecretSharer.recover_secret(shares[:3])
print("Recovered secret:", recovered)
```

```bash
# Using ssss (Shamir's Secret Sharing Scheme) command-line tool
# Split a secret
echo "AES256_KEY_HERE" | ssss-split -t 3 -n 5
# Output: 5 shares, any 3 can reconstruct

# Combine shares to recover secret
ssss-combine -t 3
# Enter 3 shares when prompted
# Output: Original secret
```

### SecureDrop-Style Anonymous Submission Platforms

The episode references anonymous submission platforms similar to SecureDrop, used for whistleblowers to submit information to journalists or organizations without revealing their identity:

**SecureDrop architecture:**

```
[Whistleblower]
    |
    | (Tor Browser)
    |
[Tor Network] (3+ relay hops)
    |
    | (.onion address)
    |
[SecureDrop Source Interface] (Tor hidden service)
    |
    | (Air-gapped transfer via USB)
    |
[SecureDrop Journalist Workstation] (Tails OS, air-gapped)
    |
[Journalist reviews submissions]
```

**Security properties:**

- **Source anonymity**: The journalist never learns the whistleblower's identity (unless voluntarily disclosed)
- **Server anonymity**: The SecureDrop server's physical location is hidden behind Tor
- **Air gap**: Submissions are transferred from the internet-connected server to the journalist workstation via USB, preventing network-based exfiltration
- **No metadata**: Tor prevents network-level identification of the source
- **Encryption**: All submissions are encrypted with the journalist's PGP key

```bash
# SecureDrop setup (simplified)
# Requires two physical servers: Application Server and Monitor Server

# Source interface accessible only via Tor
# http://<onion_address>.onion/

# Journalist installs SecureDrop Workstation (based on Qubes OS)
# Or uses Tails OS on air-gapped machine

# PGP key generation for journalist
gpg --full-generate-key --expert
# Use 4096-bit RSA key
# Upload public key to SecureDrop application server

# Submissions are encrypted at rest on the server
# Only decryptable on the air-gapped journalist workstation
```

### Dead Man's Switches for Automated Information Release

A **dead man's switch** is a mechanism that automatically triggers an action (typically releasing information) if the operator fails to periodically confirm they are safe and free:

**Technical implementations:**

```python
# Dead man's switch concept
# If the operator fails to "check in" within the specified interval,
# the system automatically releases encrypted data or decryption keys

import time
import smtplib
from datetime import datetime, timedelta

CHECK_IN_INTERVAL = timedelta(hours=24)
last_check_in = datetime.now()

def check_in(auth_token):
    """Operator confirms they are safe"""
    global last_check_in
    if verify_token(auth_token):
        last_check_in = datetime.now()
        return True
    return False

def monitor():
    """Continuously checks if operator has checked in"""
    while True:
        if datetime.now() - last_check_in > CHECK_IN_INTERVAL:
            trigger_release()
            break
        time.sleep(3600)  # Check every hour

def trigger_release():
    """Release decryption keys or publish documents"""
    # Option 1: Email decryption key to journalists
    send_key_to_journalists(decryption_key)

    # Option 2: Post decryption key to blockchain (immutable)
    post_to_blockchain(decryption_key)

    # Option 3: Seed torrent with decrypted documents
    start_seeding(decrypted_documents_torrent)

    # Option 4: Publish to multiple paste sites
    publish_to_pastebin(decryption_key)
```

**Real-world implementations:**

- **Encrypted file + delayed key release**: Distribute encrypted files widely; release key only if triggered
- **Blockchain-based**: Store encrypted data in blockchain transactions; release key to dedicated smart contract
- **Multi-party**: Distribute shares of the key using Shamir's Secret Sharing to trusted parties who release their shares if triggered
- **Canary-based**: A regularly updated public statement ("warrant canary") whose absence signals that a trigger event has occurred

### NetFlow Traffic Analysis for De-Anonymization

**NetFlow** records metadata about network traffic flows, providing a high-level view of network communications without capturing full packet content:

**NetFlow record contents:**

```
Source IP | Dest IP | Src Port | Dst Port | Protocol | Bytes | Packets | Start Time | End Time
10.0.1.50 | 192.168.1.1 | 45234 | 443 | TCP | 125000 | 87 | 14:30:01 | 14:30:45
```

**De-anonymization through traffic analysis:**

Even when traffic content is encrypted (e.g., Tor), traffic patterns can reveal information:

```
Tor De-anonymization via Traffic Correlation:

[User] --> [Entry Node] --> [Relay] --> [Exit Node] --> [Destination]

Attack: If adversary controls (or monitors) both entry and exit:
- Correlate timing and volume of traffic entering Tor with traffic exiting Tor
- Match patterns to link user to destination

NetFlow analysis enables this:
- Collect NetFlow data from ISPs and major network peering points
- Correlate flow timing: flow entering Tor at time T, flow exiting at time T+delta
- Volume correlation: 500KB entering, ~500KB exiting (accounting for Tor overhead)
- Long-term pattern analysis: User consistently accesses destination at specific times
```

```bash
# NetFlow analysis tools
# nfdump - process and analyze NetFlow data
nfdump -R /var/netflow/data/ -t '2019/11/15.14:00:00-2019/11/15.15:00:00' \
    -s srcip/bytes -n 20
# Top 20 source IPs by bytes in the specified time window

# Flow correlation for Tor de-anonymization
nfdump -R /var/netflow/ -t '2019/11/15.14:30:00-2019/11/15.14:31:00' \
    'dst ip 10.0.0.0/8 and dst port 9001'
# Identify Tor relay connections in the time window

# SiLK (System for Internet-Level Knowledge)
rwfilter --start-date=2019/11/15 --end-date=2019/11/15 \
    --proto=6 --dport=9001 --type=out --pass=stdout | \
    rwstats --fields=sip --value=bytes --count=20
```

### SCADA/ICS Protocols: Modbus and DNP3

The episode references industrial control system protocols that were relevant to the original 5/9 attack and Stage 2:

**Modbus protocol:**

- Developed in 1979 by Modicon (now Schneider Electric)
- No authentication, no encryption in original specification
- Master-slave architecture: master (SCADA) polls slaves (PLCs/RTUs)
- Commonly runs over TCP port 502 (Modbus TCP) or serial (Modbus RTU)

```bash
# Modbus interaction using modbus-cli
modbus read 192.168.1.100 0 10       # Read 10 holding registers starting at address 0
modbus write 192.168.1.100 0 100 200  # Write values to registers

# Using mbtget for Modbus TCP
mbtget -a 1 -r 0 -n 10 192.168.1.100   # Read registers from unit 1

# Metasploit Modbus modules
msfconsole
use auxiliary/scanner/scada/modbusdetect
set RHOSTS 192.168.1.0/24
run

# Python Modbus interaction
python3 -c "
from pymodbus.client import ModbusTcpClient
client = ModbusTcpClient('192.168.1.100')
client.connect()
result = client.read_holding_registers(0, 10, unit=1)
print(result.registers)
client.close()
"
```

**DNP3 (Distributed Network Protocol 3):**

- Used primarily in electric utilities, water/wastewater, and oil/gas
- More feature-rich than Modbus, supports unsolicited reporting and secure authentication (SA)
- Commonly runs on TCP port 20000
- DNP3 Secure Authentication (SA) adds challenge-response authentication but is not widely deployed

```bash
# DNP3 traffic analysis in Wireshark
tshark -r capture.pcap -Y "dnp3" -T fields -e dnp3.ctl.dir -e dnp3.al.func

# Common DNP3 function codes:
# 0x01 = Read
# 0x02 = Write
# 0x03 = Select
# 0x04 = Operate
# 0x0D = Cold Restart
# 0x0E = Warm Restart
```

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **BitTorrent (transmission, aria2)** | Peer-to-peer file distribution |
| **ssss** | Shamir's Secret Sharing command-line implementation |
| **SecureDrop** | Anonymous whistleblower submission platform |
| **Tor** | Anonymous communication network |
| **GPG/PGP** | Encryption for secure document handling |
| **nfdump / SiLK** | NetFlow analysis for traffic pattern investigation |
| **modbus-cli / pymodbus** | Modbus protocol interaction |
| **Wireshark / tshark** | Protocol analysis for Modbus and DNP3 traffic |
| **Metasploit** | SCADA module scanning |

---

## Commands Shown

```bash
# Create and distribute a torrent
mktorrent -a udp://tracker.example.com:1337 -o release.torrent /path/to/files/
transmission-cli release.torrent

# Shamir's Secret Sharing
echo "MASTER_KEY_256BIT_HEX" | ssss-split -t 3 -n 5 -w token
ssss-combine -t 3

# SecureDrop source submission (via Tor Browser)
# Navigate to: http://<onion_address>.onion
# Submit documents through the web interface
# Record the codename for future check-ins

# NetFlow analysis
nfdump -R /data/netflow/ -s dstip/flows -n 50 'proto tcp and dst port 443'
nfdump -R /data/netflow/ -A srcip,dstip -s record/bytes -n 20

# Modbus scanning
nmap -sV -p 502 --script modbus-discover 10.0.0.0/24
mbtget -a 1 -r 0 -n 100 192.168.1.100

# DNP3 scanning
nmap -sV -p 20000 --script dnp3-info 10.0.0.0/24

# Dead man's switch check-in (example cron job)
# 0 */12 * * * curl -s -X POST https://deadman.example.onion/checkin -d "token=SECRET"
```

---

## Real-World Parallels

### WikiLeaks Insurance Files
- **WikiLeaks insurance files (2010-present)**: WikiLeaks has distributed multiple AES-256 encrypted "insurance files" via BitTorrent, ranging from 1.4GB to 87GB. These files serve as dead man's switches -- if WikiLeaks is shut down or its operators compromised, the decryption keys would theoretically be released. The largest insurance file was distributed in 2016 via magnet link and seeded by thousands of peers worldwide.

### SecureDrop Adoption
- **SecureDrop**: Originally created by Aaron Swartz and Kevin Poulsen (as DeadDrop), now maintained by the Freedom of the Press Foundation. Used by The New York Times, The Washington Post, The Guardian, ProPublica, and dozens of other news organizations worldwide. The platform has been used for major investigative journalism stories.
- **GlobaLeaks**: An alternative open-source whistleblowing platform with similar goals but different architecture.

### Shamir's Secret Sharing in Practice
- **DNSSEC root key ceremony**: The Internet's DNSSEC root zone signing key is protected using a form of secret sharing. Trusted Community Representatives from around the world each hold a physical key card, and a quorum is required for key operations. This is one of the highest-profile real-world uses of threshold cryptography.
- **Cryptocurrency custody**: Companies like Coinbase and other exchanges use Shamir's Secret Sharing (or related threshold signature schemes) to protect private keys controlling billions of dollars in cryptocurrency.

### Traffic Analysis and Tor De-Anonymization
- **Carnegie Mellon / SEI Tor de-anonymization (2014)**: Researchers at Carnegie Mellon's Software Engineering Institute developed a technique to de-anonymize Tor users using traffic correlation, which was reportedly used by the FBI to identify users of the Silk Road 2.0 and other hidden services. This led to significant controversy and a debate about research ethics.
- **NSA XKeyscore + Tor**: Leaked NSA documents showed that the NSA was collecting NetFlow-like data on Tor relays and attempting traffic correlation attacks, validating the threat model depicted in the show.

### SCADA/ICS Protocol Vulnerabilities
- **Stuxnet (2010)**: Exploited Siemens S7-300 PLCs communicating over proprietary SCADA protocols to manipulate uranium enrichment centrifuges.
- **Ukraine power grid attack (2015)**: Attackers used legitimate ICS protocols (including OPC) to open circuit breakers, causing power outages for 225,000 customers.
- **ICS-CERT advisories**: The US government's ICS-CERT regularly publishes advisories about vulnerabilities in Modbus, DNP3, and other ICS protocol implementations, demonstrating the ongoing security challenges in these systems.

## Tool Links

- [BitTorrent](https://www.bittorrent.com/) - P2P protocol for decentralized file distribution
- [qBittorrent](https://www.qbittorrent.org/) - Open-source BitTorrent client
- [SecureDrop](https://securedrop.org/) - Anonymous submission platform for whistleblowers
- [Tor](https://www.torproject.org/) - Anonymous communication network
- [Tails](https://tails.net/) - Amnesic operating system for SecureDrop workstation
- [GnuPG](https://gnupg.org/) - PGP encryption for secure documents
- [VeraCrypt](https://veracrypt.fr/) - Volume encryption for key protection
- [Metasploit](https://www.metasploit.com/) - SCADA/Modbus scanning modules
- [pymodbus](https://github.com/pymodbus-dev/pymodbus) - Python library for Modbus interaction
- [Wireshark](https://www.wireshark.org/) - Modbus and DNP3 protocol analysis
- [Nmap](https://nmap.org/) - Network scanning for SCADA device discovery
- [Shodan](https://www.shodan.io/) - Search for internet-exposed ICS/SCADA devices
- [Censys](https://censys.io/) - Infrastructure and exposed device search

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Key distribution via BitTorrent | T1567 | Exfiltration Over Web Service | https://attack.mitre.org/techniques/T1567/ |
| Anonymous communication via Tor | T1572 | Protocol Tunneling | https://attack.mitre.org/techniques/T1572/ |
| Encryption key recovery | T1005 | Data from Local System | https://attack.mitre.org/techniques/T1005/ |
| NetFlow traffic analysis for de-anonymization | T1040 | Network Sniffing | https://attack.mitre.org/techniques/T1040/ |
| Modbus/SCADA protocol interaction | T1071 | Application Layer Protocol | https://attack.mitre.org/techniques/T1071/ |
| ICS device scanning | T1046 | Network Service Discovery | https://attack.mitre.org/techniques/T1046/ |
| Dead man's switch for automatic release | T1053 | Scheduled Task/Job | https://attack.mitre.org/techniques/T1053/ |
| Anonymous submission via SecureDrop | T1102 | Web Service | https://attack.mitre.org/techniques/T1102/ |

## References and Further Reading

- **WikiLeaks Insurance Files (2010-2016)**: AES-256 encrypted files distributed via BitTorrent as a dead man's switch
- **SecureDrop (Freedom of the Press Foundation)**: Platform created by Aaron Swartz and Kevin Poulsen, used by NYT, Washington Post, The Guardian
- **DNSSEC Root Key Ceremony**: Real-world use of secret sharing for protecting the internet's DNS root key
- **Carnegie Mellon SEI Tor De-anonymization (2014)**: Controversial research on traffic correlation to de-anonymize Tor users
- **NSA XKeyscore + Tor Documents (Snowden leaks)**: Leaked documents showing NetFlow data collection on Tor relays
- **Stuxnet and ICS Protocol Exploitation (2010)**: Exploitation of proprietary SCADA protocols for centrifuge manipulation
- **Ukraine Power Grid Attack (2015)**: Use of legitimate ICS protocols to open circuit breakers and cause power outages
- **ICS-CERT Advisories**: Regular advisories on vulnerabilities in Modbus and DNP3 implementations

## Search Tags

```
tags: [BitTorrent, SecureDrop, Tor, Shamir-Secret-Sharing, dead-mans-switch, encryption-keys, NetFlow, traffic-analysis, Modbus, DNP3, SCADA, ICS, WikiLeaks, PGP, Tails]
season: 3
episode: 9
mitre: [T1567, T1572, T1005, T1040, T1071, T1046, T1053, T1102]
```
