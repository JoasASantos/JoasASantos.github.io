# Episode 1: eps1.0_hellofriend.mov

## Overview

The pilot episode introduces Elliot Alderson, a cybersecurity engineer at Allsafe Cybersecurity, and his vigilante hacking activities. Elliot exposes Ron, the owner of a coffee shop, as the operator of a child exploitation website by analyzing Tor exit node traffic. The episode also depicts a massive DDoS attack against E Corp (Evil Corp) and the discovery of a sophisticated rootkit on their servers, drawing Elliot into the orbit of the mysterious fsociety hacking collective.

---

## Hacks & Techniques

### 1. Tor Exit Node Traffic Analysis

Elliot operates a Tor exit node from his apartment, allowing him to monitor unencrypted traffic leaving the Tor network. While Tor encrypts data within its relay circuit, traffic exiting through the final node (the exit node) reverts to its original form if the underlying connection is not encrypted with TLS/SSL. Elliot passively captures this traffic and correlates it to identify Ron as someone hosting illegal content.

**How it works:**

- Elliot configured his machine as a Tor exit relay, meaning traffic from Tor users passes through his node before reaching the public internet.
- Any HTTP (non-HTTPS) traffic, DNS queries, or unencrypted protocol data is visible in plaintext at the exit node.
- By logging source/destination metadata and performing timing correlation, Elliot links anonymous Tor sessions to Ron's server infrastructure.
- This is a well-documented weakness in Tor's threat model: exit nodes can observe and potentially manipulate unencrypted traffic.

**Key concepts:**

- **Tor circuit**: Entry guard -> Middle relay -> Exit node -> Destination
- **Exit node sniffing**: Capturing plaintext data at the last hop
- **Traffic correlation**: Matching timing patterns between Tor sessions and destination access logs
- **Passive surveillance vs. active MITM**: Elliot performs passive observation rather than injecting content

### 2. Password Cracking via Dictionary Attacks

Elliot describes cracking Ron's password using common dictionary-based attacks. He mocks Ron's weak password choices, noting that most people use predictable patterns based on personal information.

**Attack methodology:**

- **Dictionary attacks**: Using wordlists containing common passwords, leaked credential databases, and custom-generated lists based on target reconnaissance.
- **Rule-based mutations**: Applying transformations to dictionary words (e.g., appending numbers, substituting letters with symbols like "a" -> "@", "s" -> "$").
- **Hybrid attacks**: Combining dictionary words with brute-force masks (e.g., `password` + 4 digits).

**Tools referenced:**

- **John the Ripper**: An open-source password cracker supporting numerous hash formats. Capable of dictionary, brute-force, and incremental modes. Highly extensible with custom rules.
- **Hashcat**: A GPU-accelerated password recovery tool. Supports over 300 hash types and multiple attack modes (dictionary, combinator, mask, rule-based, hybrid).

### 3. OSINT and Social Media Reconnaissance for Password Guessing

Before resorting to computational cracking, Elliot performs open-source intelligence (OSINT) gathering on his targets. He researches their social media profiles, public records, and online presence to build a profile of likely password candidates.

**Common OSINT sources for password guessing:**

- **Social media profiles**: Pet names, children's names, spouse names, birthdays, anniversaries, favorite sports teams, alma maters
- **Public records**: Address history, phone numbers, family members
- **Data breach databases**: Previously leaked passwords from other services (credential stuffing)
- **LinkedIn/professional profiles**: Company names, job titles, professional milestones
- **Personal blogs and forums**: Hobbies, favorite media, catchphrases

**Why this works:**

Research consistently shows that a significant percentage of users incorporate personal information into their passwords. A 2019 Google/Harris Poll survey found that 59% of Americans use a name or birthday in their passwords.

### 4. RUDY (R-U-Dead-Yet) DDoS Attack Against E Corp

The major incident in the pilot is a distributed denial-of-service (DDoS) attack targeting E Corp's infrastructure, handled by Allsafe as their security contractor. The attack is identified as a RUDY attack, a Layer 7 (application layer) slow HTTP POST attack.

**How RUDY works:**

1. The attacker initiates a legitimate HTTP POST request to the target web server.
2. The `Content-Length` header is set to an extremely large value, signaling that a large payload is incoming.
3. Instead of sending the full payload, the attacker transmits the POST body one byte at a time at extremely slow intervals (e.g., one byte every 10-100 seconds).
4. The server keeps the connection open, waiting for the complete payload as promised by the Content-Length header.
5. Multiple concurrent slow POST connections exhaust the server's connection pool, preventing legitimate users from connecting.

**Why Layer 7 attacks are effective:**

- They mimic legitimate traffic patterns, making detection harder than volumetric (Layer 3/4) attacks.
- They require far fewer resources from the attacker than bandwidth-based floods.
- Traditional DDoS mitigation appliances focused on traffic volume may miss them entirely.
- Each malicious connection consumes server-side resources (threads, memory, file descriptors) disproportionately.

**Mitigation strategies:**

- Request timeout enforcement (killing connections with abnormally slow transfer rates)
- Connection rate limiting per source IP
- Web application firewalls (WAFs) with slow-rate detection
- Reverse proxy buffering (e.g., Nginx buffering full requests before passing to backend)

### 5. Rootkit Discovery on E Corp Servers

While investigating the DDoS attack, Elliot discovers a suspicious DAT file on E Corp's servers that turns out to be a rootkit, a piece of malware designed to maintain persistent, stealthy, privileged access to a compromised system. This rootkit is far more sophisticated than the DDoS attack and suggests a deeper, more serious compromise.

**Rootkit characteristics observed:**

- Hidden processes not visible through standard system utilities
- Modified system binaries to conceal the rootkit's presence
- Network connections to external command-and-control (C2) infrastructure
- Persistence mechanisms surviving reboots
- A DAT file serving as the rootkit's configuration or payload container

**Types of rootkits (for context):**

| Type | Level | Description |
|------|-------|-------------|
| User-mode | Ring 3 | Replaces/hooks userspace binaries and libraries |
| Kernel-mode | Ring 0 | Modifies kernel modules, syscall table, or VFS |
| Bootkits | Pre-OS | Infects MBR/VBR/UEFI to load before the OS |
| Firmware | Hardware | Embeds in device firmware (NIC, HDD, BIOS/UEFI) |

### 6. IRC for Anonymous Communication

The fsociety group uses IRC (Internet Relay Chat) for coordination, a historically significant communication platform for hacker groups and hacktivist collectives.

**Why IRC is favored by hacking groups:**

- Decentralized architecture with self-hosted servers
- No mandatory registration or identity verification
- Can be accessed through Tor for additional anonymity
- Supports encrypted channels (SSL/TLS connections, OTR plugins)
- Minimal logging by default (depends on server/client configuration)
- Low bandwidth requirements; works over constrained connections

### 7. VPN and Proxy Usage

Elliot maintains strict operational security by routing his traffic through VPNs and proxy chains to obscure his real IP address and location.

**Layered anonymity architecture:**

- **VPN (Virtual Private Network)**: Encrypts all traffic between the client and VPN server, masking the source IP from the destination.
- **Proxy chains**: Routing traffic through multiple proxy servers sequentially, each only aware of the previous and next hop.
- **Tor over VPN**: Connecting to Tor through a VPN so the ISP cannot see Tor usage, and the Tor entry node sees the VPN IP instead of the real IP.

---

## Tools Used

| Tool | Purpose | Category |
|------|---------|----------|
| Tor | Anonymous routing, exit node operation | Anonymity/Privacy |
| John the Ripper | Password hash cracking (CPU-based) | Password Cracking |
| Hashcat | Password hash cracking (GPU-accelerated) | Password Cracking |
| RUDY (R-U-Dead-Yet) | Layer 7 slow HTTP POST DDoS | Denial of Service |
| IRC client | Anonymous group communication | Communication |
| VPN client | Traffic encryption and IP masking | Anonymity/Privacy |
| Proxy chains | Multi-hop traffic routing | Anonymity/Privacy |
| Wireshark / tcpdump | Tor exit node traffic capture (implied) | Network Analysis |

---

## Commands Shown

### Process analysis to investigate suspicious activity
```bash
# List all running processes with full details
ps aux

# Filter processes for specific suspicious keywords
ps aux | grep [suspicious_process_name]

# Display active network connections and listening ports
netstat -tulnp

# Alternative with ss (modern replacement for netstat)
ss -tulnp

# Search for the suspicious DAT file across the filesystem
find / -name "*.dat" -type f 2>/dev/null

# Examine file metadata
file suspicious_file.dat
strings suspicious_file.dat | head -50
```

### Tor exit node configuration (conceptual)
```bash
# Tor exit relay configuration in torrc
ExitRelay 1
ExitPolicy accept *:80     # Allow HTTP exit traffic
ExitPolicy accept *:443    # Allow HTTPS exit traffic
ExitPolicy reject *:*      # Reject all other exit traffic
```

### Password cracking examples
```bash
# John the Ripper - dictionary attack with rules
john --wordlist=/usr/share/wordlists/rockyou.txt --rules=best64 hashes.txt

# Hashcat - dictionary attack with rule file (GPU-accelerated)
hashcat -m 0 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt -r rules/best64.rule

# Hashcat - hybrid attack (dictionary + 4-digit mask)
hashcat -m 0 -a 6 hashes.txt /usr/share/wordlists/rockyou.txt ?d?d?d?d
```

### Network investigation during DDoS
```bash
# Monitor active connections to identify RUDY slow POST pattern
netstat -an | grep ESTABLISHED | awk '{print $5}' | sort | uniq -c | sort -rn | head -20

# Check for connections with abnormally long durations
ss -tn state established '( dport = :80 or dport = :443 )'

# Watch connection states in real time
watch -n 1 'netstat -an | grep :80 | awk "{print \$6}" | sort | uniq -c | sort -rn'
```

---

## Real-World Parallels

### Tor Exit Node Surveillance
In 2007, Swedish security researcher Dan Egerstad demonstrated this exact technique by operating Tor exit nodes and capturing login credentials for embassies and government agencies worldwide. He obtained passwords for over 100 email accounts belonging to foreign embassies, proving that Tor does not encrypt traffic beyond the exit node. This incident underscored the critical importance of end-to-end encryption (HTTPS, SSH) even when using anonymity networks.

### RUDY and Layer 7 DDoS Attacks
Layer 7 DDoS attacks have become increasingly prevalent in real-world cyber campaigns. The Slowloris tool (2009) pioneered the slow HTTP attack concept, and RUDY extended it to POST requests. Notable incidents include attacks against financial institutions during Operation Ababil (2012-2013) by the Izz ad-Din al-Qassam Cyber Fighters, which combined volumetric and application-layer techniques.

### Password Reuse and Credential Stuffing
The emphasis on OSINT-driven password guessing reflects real-world attack patterns. Major breaches at LinkedIn (2012, 117 million credentials), Adobe (2013, 153 million), and Collection #1-5 (2019, 2.2 billion records) have created massive datasets that attackers use for credential stuffing attacks, where leaked passwords from one service are tested against other services.

### Rootkit Discovery
The discovery of the DAT file rootkit parallels real-world incidents such as the Sony BMG rootkit scandal (2005), where Sony installed XCP rootkit software on customers' computers through music CDs, and the more sophisticated Uroburos/Turla rootkit attributed to Russian intelligence services, which remained undetected on government networks for years.

---

## Tool Links

- **Tor**: https://www.torproject.org/
- **John the Ripper**: https://www.openwall.com/john/
- **Hashcat**: https://hashcat.net/hashcat/
- **RUDY (R-U-Dead-Yet)**: https://github.com/sahilchaddha/rudern (PoC concept)
- **Slowloris** (referenced in parallel): https://github.com/gkbrk/slowloris
- **Wireshark**: https://www.wireshark.org/
- **tcpdump**: https://www.tcpdump.org/
- **Proxychains**: https://github.com/haad/proxychains

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Tor exit node traffic sniffing | T1040 | Network Sniffing | https://attack.mitre.org/techniques/T1040/ |
| Password cracking via dictionary attacks | T1110 | Brute Force | https://attack.mitre.org/techniques/T1110/ |
| OSINT for password guessing | T1589 | Gather Victim Identity Information | https://attack.mitre.org/techniques/T1589/ |
| RUDY DDoS attack against E Corp | T1499 | Endpoint Denial of Service | https://attack.mitre.org/techniques/T1499/ |
| RUDY Layer 7 slow HTTP POST | T1071 | Application Layer Protocol | https://attack.mitre.org/techniques/T1071/ |
| Rootkit on E Corp servers | T1014 | Rootkit | https://attack.mitre.org/techniques/T1014/ |
| Rootkit C2 encrypted communication | T1573 | Encrypted Channel | https://attack.mitre.org/techniques/T1573/ |
| IRC for anonymous communication | T1102 | Web Service | https://attack.mitre.org/techniques/T1102/ |
| VPN and proxy chain usage | T1572 | Protocol Tunneling | https://attack.mitre.org/techniques/T1572/ |
| Social media reconnaissance | T1593 | Search Open Websites/Domains | https://attack.mitre.org/techniques/T1593/ |

---

## References and Further Reading

- **CVE-2014-6271 (Shellshock)** - referenced in the broader season context: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
- **Dan Egerstad Tor Exit Node Research (2007)** - Wired article on Tor exit node eavesdropping: https://www.wired.com/2007/09/rogue-nodes-turn-tor-anonymizer-into-eavesdroppers-paradise/
- **RUDY and Slow HTTP Attacks - OWASP**: https://owasp.org/www-community/attacks/Slow_HTTP_attack
- **Operation Ababil DDoS Campaigns (2012-2013)**: https://www.akamai.com/blog
- **Sony BMG Rootkit Scandal (2005)** - EFF analysis: https://www.eff.org/deeplinks/2005/11/sony-bmg-rootkit-scandal
- **Turla/Uroburos APT Group Analysis** - Kaspersky: https://www.kaspersky.com/resource-center/threats/turla
- **DEF CON Conference Archives** - Tor security talks: https://www.defcon.org/
- **SANS White Papers on Password Cracking Techniques**: https://www.sans.org/white-papers/
- **Tor Project - Exit Relay Best Practices**: https://community.torproject.org/relay/setup/exit/

---

## Search Tags

```
tags: [Tor, John the Ripper, Hashcat, RUDY, DDoS, rootkit, IRC, VPN, proxy chains, Wireshark, tcpdump, password cracking, OSINT, exit node sniffing, Layer 7 attack, traffic analysis]
season: 1
episode: 1
mitre: [T1040, T1110, T1589, T1499, T1071, T1014, T1573, T1102, T1572, T1593]
```