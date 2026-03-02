# Episode 4: eps4.3_404notfound.h

## Season 4, Episode 4 | "404 Not Found"

**Air Date:** October 27, 2019
**Directed by:** Sam Esmail

### Overview

The episode title references HTTP status code 404, the most universally recognized error code, meaning the requested resource could not be found. In the context of the episode, key people, information, and safe communication channels become "not found" as the Dark Army intensifies counter-surveillance operations. Elliot must navigate increasingly dangerous territory using lateral movement techniques, counter-surveillance measures, and encrypted communications while dealing with the threat of SIM swapping attacks that compromise mobile security.

---

## Hacks & Techniques

### 1. HTTP 404 Not Found Status Code

HTTP 404 indicates that the server cannot find the requested resource. Unlike 403 (forbidden), the server has no knowledge of whether the resource existed previously or if access should be granted.

- **Soft 404 vs Hard 404:** A soft 404 returns a 200 status code but displays a "not found" page, while a hard 404 returns the proper status code.
- **Security Implications:** Custom 404 pages that leak server information (version numbers, technology stack) are a reconnaissance vector.
- **Thematic Meaning:** People and information Elliot needs have disappeared. The Dark Army is making resources "404" by eliminating them.

### 2. Dark Army Counter-Surveillance and Signal Jamming

The Dark Army employs sophisticated counter-surveillance techniques to monitor, disrupt, and neutralize threats to their operations.

#### Counter-Surveillance Techniques

- **Surveillance Detection Routes (SDR):** Predetermined routes designed to expose anyone following the target through unpredictable turns, stops, and doubling back.
- **Electronic Counter-Surveillance:**
  - RF (Radio Frequency) sweeps to detect bugs, transmitters, and hidden cameras.
  - Non-Linear Junction Detectors (NLJD) to find electronic components hidden in walls or furniture.
  - Thermal imaging to detect powered electronic devices.
- **Signal Jamming:** Active disruption of wireless communications in a specific area.

#### Signal Jamming Technology

| Type | Frequency Range | Target |
|---|---|---|
| **Cell Phone Jammer** | 700-2600 MHz | GSM, LTE, 5G cellular signals |
| **Wi-Fi Jammer** | 2.4 GHz, 5 GHz | 802.11 a/b/g/n/ac/ax |
| **GPS Jammer** | 1575.42 MHz (L1) | GPS navigation and tracking |
| **Broadband Jammer** | 20 MHz - 6 GHz | All wireless communications |

#### Counter-Surveillance Equipment

- **RF Detector:** Detects active radio transmissions from bugs and trackers.
- **Spectrum Analyzer:** Visualizes all radio frequency activity in an area to identify anomalous transmissions.
- **Camera Detector:** Uses infrared LEDs to detect the reflection from camera lenses (including hidden cameras).
- **TSCM (Technical Surveillance Countermeasures):** Professional-grade equipment for sweeping rooms and vehicles for surveillance devices.

### 3. Lateral Movement Techniques

Once inside a network, attackers must move laterally to reach their objectives. The episode depicts several critical lateral movement techniques used in enterprise environments.

#### PsExec

PsExec is a lightweight telnet replacement from Microsoft's Sysinternals suite that allows process execution on remote systems.

- **Mechanism:** Uses SMB (port 445) to copy a service binary to the target, creates and starts a Windows service, and establishes a named pipe for I/O.
- **Requirements:** Administrator credentials on the target system, SMB access.
- **Detection:** Creates Windows Event ID 7045 (Service Install) and Event ID 4624 (Logon Type 3 - Network).

#### WMI (Windows Management Instrumentation)

WMI provides a standardized interface for managing Windows systems remotely.

- **Mechanism:** Uses DCOM (port 135) or WinRM (port 5985/5986) to execute commands.
- **Advantage:** Does not write files to disk (when used for command execution), making it harder to detect.
- **Detection:** WMI Event ID 5861 (WMI Activity), process creation under `WmiPrvSE.exe`.

#### Pass-the-Hash (PtH)

An attack technique that authenticates to a remote system using the NTLM hash of a password without knowing the actual plaintext password.

- **Mechanism:** Windows NTLM authentication uses the hash directly in the authentication protocol, so possessing the hash is equivalent to possessing the password for network authentication.
- **Impact:** A single compromised administrator hash can provide access to every system where that account has privileges.

#### Pass-the-Ticket (PtT)

A Kerberos-based lateral movement technique that uses stolen Kerberos tickets to authenticate to services.

- **Ticket Granting Ticket (TGT):** Stolen TGT allows requesting service tickets for any service the user can access.
- **Service Ticket (TGS):** Stolen service ticket allows direct access to the specific service.
- **Golden Ticket:** A forged TGT created using the KRBTGT account hash, providing unrestricted domain access.
- **Silver Ticket:** A forged service ticket created using a service account hash, providing access to that specific service.

### 4. SIM Swapping Attacks on Mobile Carriers

SIM swapping (also called SIM hijacking) is a social engineering attack against mobile carriers to transfer a victim's phone number to an attacker-controlled SIM card.

#### SIM Swap Attack Chain

1. **Target Selection:** Identify the victim and gather personal information through OSINT (full name, phone number, address, last 4 of SSN, account PIN).
2. **Carrier Contact:** Call the mobile carrier's customer support line while impersonating the victim.
3. **Identity Verification Bypass:** Use gathered personal information to pass the carrier's identity verification questions.
4. **Number Transfer:** Request that the victim's number be transferred to a new SIM card (controlled by the attacker).
5. **Exploitation:** Once the number is transferred:
   - Intercept SMS-based 2FA codes.
   - Receive password reset links sent via SMS.
   - Access bank accounts, email, and cryptocurrency wallets.
   - Impersonate the victim via phone calls and text messages.

#### Impact of SIM Swapping

- **SMS 2FA Bypass:** Any account relying on SMS for two-factor authentication becomes vulnerable.
- **Account Takeover:** Email, banking, cryptocurrency, and social media accounts can be compromised.
- **Identity Theft:** The attacker controls the victim's phone number and can impersonate them.

### 5. Encrypted Communications

With communications increasingly compromised, the characters rely on encrypted messaging to coordinate.

#### Signal Protocol

The Signal Protocol (used by the Signal app, WhatsApp, and Facebook Messenger's secret conversations) provides:

- **End-to-End Encryption (E2EE):** Messages are encrypted on the sender's device and decrypted only on the recipient's device.
- **Double Ratchet Algorithm:** Combines the Extended Triple Diffie-Hellman (X3DH) key agreement protocol with a ratcheting mechanism that provides forward secrecy and future secrecy.
- **Forward Secrecy:** Compromise of long-term keys does not compromise past session keys.
- **Deniability:** The protocol provides cryptographic deniability, meaning messages cannot be cryptographically proven to have been sent by a particular party.

#### OTR (Off-the-Record) Messaging

OTR is a cryptographic protocol for instant messaging encryption:

- **Perfect Forward Secrecy:** New encryption keys for each conversation.
- **Deniable Authentication:** Either party can forge messages after a conversation, providing plausible deniability.
- **Malleable Encryption:** After a message is received and decrypted, the recipient can forge a message that appears to decrypt correctly, preventing cryptographic proof of authorship.

---

## Tools Used

| Tool | Purpose |
|---|---|
| **PsExec (Sysinternals)** | Remote command execution via SMB |
| **Mimikatz** | Credential extraction: hashes, tickets, plaintext passwords |
| **CrackMapExec** | Multi-purpose lateral movement and credential testing |
| **Impacket (wmiexec.py, psexec.py, smbexec.py)** | Python-based lateral movement tools |
| **Rubeus** | Kerberos attack tooling (.NET) |
| **Signal** | End-to-end encrypted messaging |
| **Spectrum Analyzer** | RF signal analysis and detection |
| **SDR (Software-Defined Radio)** | Radio signal interception and analysis |

---

## Commands Shown

### PsExec Usage

```bash
# Execute command on remote system
psexec.exe \\target-ip -u administrator -p password cmd.exe

# Impacket's psexec.py (Linux)
python3 psexec.py domain/admin:password@target-ip

# Execute command without interactive shell
psexec.exe \\target-ip -u admin -p pass -s cmd /c "whoami"
```

### WMI Lateral Movement

```bash
# Windows: WMI command execution
wmic /node:target-ip /user:admin /password:pass process call create "cmd.exe /c whoami > C:\temp\output.txt"

# Impacket's wmiexec.py (Linux)
python3 wmiexec.py domain/admin:password@target-ip

# PowerShell WMI execution
Invoke-WmiMethod -ComputerName target-ip -Credential $cred -Class Win32_Process -Name Create -ArgumentList "powershell -enc <base64_payload>"
```

### Pass-the-Hash with Mimikatz

```
# Extract NTLM hashes
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords

# Pass-the-Hash
mimikatz # sekurlsa::pth /user:administrator /domain:corp.local /ntlm:aad3b435b51404eeaad3b435b51404ee:hash /run:cmd.exe

# With CrackMapExec
crackmapexec smb target-ip -u administrator -H ntlm_hash --exec-method smbexec -x "whoami"
```

### Pass-the-Ticket with Rubeus

```
# Extract Kerberos tickets
Rubeus.exe dump

# Pass-the-Ticket
Rubeus.exe ptt /ticket:base64_encoded_ticket

# Request TGT with hash (overpass-the-hash)
Rubeus.exe asktgt /user:admin /rc4:ntlm_hash /ptt

# Golden Ticket with Mimikatz
mimikatz # kerberos::golden /user:administrator /domain:corp.local /sid:S-1-5-21-... /krbtgt:krbtgt_hash /ptt
```

### Encrypted Communications Setup

```bash
# Generate OTR keys (using otr library)
otr genkey

# Signal CLI for automated encrypted messaging
signal-cli -u +1234567890 send -m "Meeting at the usual place" +0987654321

# Verify Signal safety numbers
signal-cli -u +1234567890 trust -v SAFETY_NUMBER +0987654321
```

---

## Real-World Parallels

### SIM Swapping Attacks

- **Jack Dorsey (Twitter CEO, 2019):** Twitter CEO Jack Dorsey's account was compromised through a SIM swap attack, allowing attackers to post offensive tweets from his account.
- **Michael Terpin (2018):** Cryptocurrency investor lost $24 million in a SIM swap attack. He sued AT&T for $224 million for failing to protect his account.
- **SS7 Exploitation:** In 2017, German banks confirmed that attackers used SS7 protocol vulnerabilities to intercept SMS-based 2FA codes and steal money from customer accounts. SS7 is the signaling protocol used globally for mobile phone network operations.

### Lateral Movement in Real Attacks

- **NotPetya (2017):** Used a combination of EternalBlue (SMB exploit), Mimikatz (credential theft), PsExec, and WMI for devastating lateral movement that spread across global networks within hours, causing over $10 billion in damages.
- **SolarWinds (2020):** The SUNBURST attackers used sophisticated lateral movement techniques including SAML token forging (Golden SAML), a variant of pass-the-ticket attacks for cloud environments.
- **APT29 (Cozy Bear):** Known for methodical lateral movement using WMI, PowerShell remoting, and stolen Kerberos tickets to move through government networks.

### Signal Jamming Incidents

- **GPS Jamming in Conflict Zones:** GPS jamming has been documented in areas around Syria, with Russian forces using GPS spoofing and jamming to interfere with navigation systems.
- **Cell Phone Jammers in Prisons:** Many countries use cell phone jammers in prisons to prevent inmates from making unauthorized calls, though this also disrupts nearby legitimate communications.
- **IMSI Catchers (Stingrays):** Law enforcement uses devices that simulate cell towers to intercept communications, a form of authorized "jamming" that forces phones to connect to the rogue tower.

## Tool Links

- [PsExec](https://learn.microsoft.com/en-us/sysinternals/downloads/psexec) - Remote command execution via SMB
- [Mimikatz](https://github.com/gentilkiwi/mimikatz) - Credential extraction: hashes, tickets, plaintext passwords
- [Impacket](https://github.com/fortra/impacket) - Python tools for lateral movement (wmiexec, psexec, smbexec)
- [Signal](https://signal.org/) - End-to-end encrypted messaging
- [SigPloit](https://github.com/SigPloiter/SigPloit) - SS7 protocol exploitation (reference to SS7 attacks)
- [Cobalt Strike](https://www.cobaltstrike.com/) - C2 framework for red teaming
- [BloodHound](https://github.com/BloodHoundAD/BloodHound) - Active Directory attack path visualization
- [PowerSploit](https://github.com/PowerShellMafia/PowerSploit) - PowerShell post-exploitation modules

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Lateral movement via PsExec | T1021 | Remote Services | https://attack.mitre.org/techniques/T1021/ |
| Pass-the-Hash / Pass-the-Ticket | T1550 | Use Alternate Authentication Material | https://attack.mitre.org/techniques/T1550/ |
| Credential extraction with Mimikatz | T1558 | Steal or Forge Kerberos Tickets | https://attack.mitre.org/techniques/T1558/ |
| SIM swapping of mobile carriers | T1566 | Phishing | https://attack.mitre.org/techniques/T1566/ |
| Counter-surveillance and signal jamming | T1562 | Impair Defenses | https://attack.mitre.org/techniques/T1562/ |
| Encrypted communications (Signal) | T1573 | Encrypted Channel | https://attack.mitre.org/techniques/T1573/ |
| Golden Ticket / Silver Ticket | T1558 | Steal or Forge Kerberos Tickets | https://attack.mitre.org/techniques/T1558/ |
| Remote command execution | T1059 | Command and Scripting Interpreter | https://attack.mitre.org/techniques/T1059/ |

## References and Further Reading

- NotPetya (2017) - Combined use of EternalBlue, Mimikatz, PsExec, and WMI for devastating lateral movement
- SolarWinds SUNBURST (2020) - Golden SAML as a Pass-the-Ticket variant for cloud environments
- Jack Dorsey SIM Swap (2019) - Twitter CEO account compromise via SIM swap
- Michael Terpin Case (2018) - $24 million loss in SIM swap attack
- SS7 Exploitation (2017) - German banks confirm SMS 2FA interception via SS7
- CVE-2017-0144 (MS17-010/EternalBlue) - SMB exploitation used in lateral movement
- NIST SP 800-63B - Deprecation of SMS-based 2FA
- Signal Protocol Technical Documentation: https://signal.org/docs/

## Search Tags

```
tags: [psexec, mimikatz, impacket, signal, lateral-movement, pass-the-hash, pass-the-ticket, SIM-swap, golden-ticket, kerberos, counter-surveillance, signal-jamming, encrypted-comms]
season: 4
episode: 4
mitre: [T1021, T1550, T1558, T1566, T1562, T1573, T1059]
```
