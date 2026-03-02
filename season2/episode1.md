# Season 2, Episodes 1 & 2 (Double Premiere): eps2.0_unm4sk-pt1.tc / eps2.0_unm4sk-pt2.tc

## Episode Overview

The Season 2 double premiere picks up weeks after the 5/9 hack. fsociety needs a new base of operations and targets the smart home of E Corp general counsel Susan Jacobs ("Madam Executioner"). Darlene and the team exploit her extensively automated smart home to make the house uninhabitable, forcing her to leave so they can occupy it. Meanwhile, Elliot attempts to live a rigidly structured, analog life to suppress Mr. Robot, and the show introduces the femtocell hack that will become the central technical plot of the season.

---

## Hacks & Techniques

### 1. Smart Home / IoT Reconnaissance and Enumeration

fsociety's first step is mapping all the Internet of Things (IoT) devices in Susan Jacobs' luxury home. Modern smart homes expose a large attack surface because every connected device is a potential entry point.

**Device Enumeration via Network Scanning:**

The team performs network discovery to identify all connected devices on the home network. Smart home systems typically expose services through UPnP (Universal Plug and Play) and SSDP (Simple Service Discovery Protocol), which broadcast device capabilities without authentication.

Typical devices found in a high-end smart home like Jacobs':

- **Smart home hub/controller** (Crestron, Control4, Savant, or similar)
- **Smart thermostat** (Nest, Ecobee, or integrated HVAC controller)
- **Security/alarm system** (networked panel with cloud connectivity)
- **Audio/video system** (whole-home audio with network streaming)
- **Smart lighting** (Lutron, Philips Hue, or hub-integrated)
- **Smart locks and access control**
- **IP cameras**
- **Network-connected appliances**

### 2. UPnP/SSDP Protocol Exploitation

UPnP is a set of networking protocols that allows devices to discover each other on a local network and establish functional network services. SSDP is the discovery protocol within UPnP. These protocols were designed for convenience, not security, and are notoriously vulnerable.

**How UPnP/SSDP Discovery Works:**

1. A new device joins the network and sends an M-SEARCH multicast message to `239.255.255.250:1900`
2. All UPnP-enabled devices respond with their service descriptions
3. The querying device can then access XML device description files
4. These descriptions reveal device type, manufacturer, firmware version, and available control URLs
5. Control commands can be sent without authentication in many implementations

**Key Vulnerabilities:**

- **No authentication**: UPnP was designed for trusted local networks and typically requires no credentials
- **XML External Entity (XXE) attacks**: Malformed XML in UPnP requests can lead to information disclosure
- **Buffer overflows**: Many UPnP implementations (especially libupnp) have known buffer overflow vulnerabilities
- **Port mapping manipulation**: Attackers can create port forwarding rules to expose internal services to the internet
- **Service description enumeration**: Device descriptions reveal firmware versions and model numbers, enabling targeted exploits

### 3. Default Credential Attacks on IoT Devices

A huge percentage of IoT devices ship with default usernames and passwords that users never change. This is one of the most reliably exploitable vulnerabilities in consumer and commercial IoT.

**Common Default Credential Patterns:**

| Device Type | Common Defaults |
|---|---|
| Crestron controllers | admin / admin, crestron / crestron |
| IP cameras | admin / admin, admin / 12345, root / root |
| Smart home hubs | admin / password, installer / installer |
| Network routers | admin / admin, admin / (blank) |
| Alarm panels | installer / 1234, master / 1234 |

**Attack Approach:**

The team systematically tries default credentials against every discovered device. For embedded devices with web interfaces, this often involves HTTP basic authentication or simple POST-based login forms. Many industrial-grade home automation systems (like Crestron) were designed for professional installers and assume the network itself is the security boundary.

### 4. Smart Home Hub Exploitation (Crestron-Like System)

Susan Jacobs' home appears to use a professional-grade home automation controller similar to Crestron or Control4. These systems are central command units that integrate and control all other smart devices.

**Attack Surface of Home Automation Controllers:**

- **Web-based administration panels** often accessible without encryption
- **Telnet/SSH services** for installer configuration (frequently with default or no credentials)
- **CIP (Crestron Internet Protocol)** or proprietary control protocols with no authentication
- **SNMP services** for network management with default community strings
- **Firmware update mechanisms** that may not validate update authenticity

**Once the hub is compromised, the attacker controls everything:**

- HVAC system: Can set extreme temperatures, disable heating/cooling
- Alarm system: Can arm/disarm, trigger false alarms, disable monitoring
- Audio system: Can play audio at maximum volume at any hour
- Lighting: Can control all lights, create strobing effects
- Locks: Can lock/unlock doors
- Cameras: Can disable surveillance or access feeds

### 5. Thermostat Manipulation

After gaining access to the HVAC controller, fsociety manipulates the thermostat to make the house uncomfortable. Smart thermostats often have APIs or web interfaces that allow temperature scheduling and override.

**Techniques:**

- Setting the heating system to maximum during summer
- Disabling cooling systems
- Overriding manual thermostat controls so the homeowner cannot fix the temperature locally
- Locking out the thermostat interface with a PIN or configuration change

### 6. Alarm System Manipulation

The home alarm system is triggered repeatedly to harass the homeowner:

- Triggering false alarms at odd hours
- Sending false alerts to the monitoring company
- Disabling the system so the homeowner feels insecure
- Manipulating zone configurations

### 7. Audio System Hijacking

The whole-home audio system is used to play disturbing audio throughout the house:

- Exploiting DLNA/UPnP media renderer capabilities
- Pushing audio content to networked speakers
- Setting volume to maximum and disabling local volume controls
- Playing content on a loop through scheduled automation routines

### 8. Physical Eviction Through Digital Harassment

The combined effect of all these attacks constitutes a "digital eviction" -- making the home environment so hostile that the occupant voluntarily leaves. This is a sophisticated form of cyber-physical attack:

1. **Thermostat**: House becomes unbearably hot or cold
2. **Alarm**: Constant false alarms disrupt sleep and daily life
3. **Audio**: Disturbing sounds play at random times
4. **Lighting**: Lights flash or turn on/off unpredictably
5. **Locks**: Potential lockout scenarios

The homeowner, unable to regain control of her own home, eventually leaves -- at which point fsociety moves in and establishes their base of operations.

### 9. Femtocell Planning Introduction

The premiere introduces the concept that will drive Season 2's main technical plot: building a femtocell to intercept FBI cellular communications. At this stage, Darlene and the team begin discussing the need to monitor what the FBI knows about fsociety, which will eventually lead to the femtocell hack executed in Episode 8.

A femtocell is a small, low-power cellular base station designed to improve indoor cell coverage. When modified, it can act as a rogue base station that intercepts all cellular traffic from nearby phones -- essentially a DIY IMSI catcher.

---

## Tools Used

| Tool | Purpose |
|---|---|
| **Nmap** | Network scanning and device enumeration |
| **UPnP/SSDP scanner** | Discovery of smart home devices via multicast |
| **Shodan** (conceptual) | Identifying IoT device types and known vulnerabilities |
| **Default credential databases** | Lists of factory-default usernames and passwords |
| **Crestron Toolbox** (or similar) | Professional home automation programming/control software |
| **Browser/curl** | Accessing web-based device administration panels |
| **Wireshark** | Analyzing network traffic between smart home devices |

---

## Commands Shown

**UPnP/SSDP Device Discovery:**

```bash
# Send M-SEARCH discovery request to find all UPnP devices
# Targets the SSDP multicast address on port 1900
python -c "
import socket
msg = \
'M-SEARCH * HTTP/1.1\r\n' \
'HOST: 239.255.255.250:1900\r\n' \
'MAN: \"ssdp:discover\"\r\n' \
'MX: 3\r\n' \
'ST: ssdp:all\r\n' \
'\r\n'
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM, socket.IPPROTO_UDP)
sock.settimeout(5)
sock.sendto(msg.encode(), ('239.255.255.250', 1900))
while True:
    try:
        data, addr = sock.recvfrom(65507)
        print(f'[+] Device found at {addr}')
        print(data.decode())
    except socket.timeout:
        break
"
```

**Nmap Network Scan for IoT Devices:**

```bash
# Comprehensive scan of home network to find all connected devices
nmap -sV -sC -O --open 192.168.1.0/24

# Targeted scan for common IoT ports
nmap -p 80,443,8080,8443,23,22,1900,5000,8008,9090 192.168.1.0/24

# UPnP-specific NSE scripts
nmap --script upnp-info -p 1900 192.168.1.0/24
```

**Accessing Crestron-Like Controller:**

```bash
# Connect to Crestron controller via Telnet (default port 41795)
telnet 192.168.1.100 41795

# Common Crestron commands once connected
> VER           # Check firmware version
> IPCONFIG      # View network configuration
> PROGCOMMENTS  # View program information
> SHOWHW        # Show hardware configuration
```

**Thermostat API Manipulation:**

```bash
# Example: Setting thermostat to maximum heat via REST API
curl -X PUT http://192.168.1.105/tstat \
  -H "Content-Type: application/json" \
  -d '{"tmode": 1, "t_heat": 95.0}'

# Disable cooling mode
curl -X PUT http://192.168.1.105/tstat \
  -H "Content-Type: application/json" \
  -d '{"tmode": 1, "override": 1}'
```

**Default Credential Brute Force with Hydra:**

```bash
# Brute force web login on smart home hub
hydra -L /usr/share/seclists/Usernames/default-credentials.txt \
      -P /usr/share/seclists/Passwords/default-credentials.txt \
      192.168.1.100 http-get /

# Telnet brute force against IoT device
hydra -l admin -P /usr/share/wordlists/iot-defaults.txt \
      192.168.1.100 telnet
```

---

## Real-World Parallels

### Smart Home Hacking Research

- **DEF CON and Black Hat presentations** have repeatedly demonstrated vulnerabilities in Crestron, Control4, Savant, and other home automation platforms. Researchers have shown full remote takeover of homes through exposed web interfaces and default credentials.

- **Crestron vulnerabilities**: In 2018, researchers at SEC Consult disclosed multiple vulnerabilities in Crestron AM-100 and AM-101 devices, including command injection and hardcoded credentials. Crestron controllers have historically been accessible via Telnet with no authentication.

- **Mirai Botnet (2016)**: The Mirai malware demonstrated on a massive scale how IoT devices with default credentials could be compromised. It infected hundreds of thousands of IP cameras, routers, and DVRs, using them to launch the largest DDoS attacks in history at that time.

- **Shodan IoT Exposure**: The search engine Shodan has consistently shown that millions of IoT devices are directly accessible from the internet with default credentials. Smart home controllers, IP cameras, and industrial control systems are routinely found exposed.

- **Ring Camera Incidents**: Multiple incidents of attackers accessing Ring home security cameras through credential stuffing, allowing them to speak through cameras, spy on families, and manipulate home systems.

### UPnP as an Attack Vector

- **CallStranger (CVE-2020-12695)**: A UPnP vulnerability affecting billions of devices that allowed data exfiltration, DDoS amplification, and port scanning of internal networks.

- **UPnProxy**: Akamai research revealed that attackers were exploiting UPnP in routers to create proxy networks, redirecting traffic through compromised home routers for malicious purposes.

### The "Digital Eviction" Concept

- While the show dramatizes this concept, security researchers have demonstrated that compromising a smart home hub effectively gives an attacker physical control over a living space. The concept of using cyber attacks to create uninhabitable physical conditions is a form of cyber-physical attack that bridges the digital and physical worlds.

- **Nest Thermostat Hacking**: Researchers have demonstrated taking control of Nest thermostats, including the ability to set extreme temperatures and lock out the homeowner. In January 2019, reports emerged of hackers accessing Nest devices to crank up heat, play disturbing audio through cameras, and threaten homeowners.

---

## Tool Links

| Tool | Link |
|---|---|
| Nmap | https://nmap.org/ |
| Shodan | https://www.shodan.io/ |
| Wireshark | https://www.wireshark.org/ |
| Crestron | https://www.crestron.com/ |
| UPnP/miniupnpc | https://miniupnp.tuxfamily.org/ |

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Network scanning and device enumeration | T1046 | Network Service Discovery | https://attack.mitre.org/techniques/T1046/ |
| Default credential attacks on IoT | T1110 | Brute Force | https://attack.mitre.org/techniques/T1110/ |
| Exploiting UPnP/SSDP for discovery | T1046 | Network Service Discovery | https://attack.mitre.org/techniques/T1046/ |
| Smart home hub control (Crestron) | T1190 | Exploit Public-Facing Application | https://attack.mitre.org/techniques/T1190/ |
| Thermostat and HVAC manipulation | T1565 | Data Manipulation | https://attack.mitre.org/techniques/T1565/ |
| Audio system hijack (DLNA/UPnP) | T1219 | Remote Access Software | https://attack.mitre.org/techniques/T1219/ |
| Femtocell planning (rogue base station) | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| Digital eviction via cyber-physical attack | T1529 | System Shutdown/Reboot | https://attack.mitre.org/techniques/T1529/ |

---

## References and Further Reading

- **CVE-2020-12695** (CallStranger - UPnP vulnerability): https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-12695
- **DEF CON - Smart Home Hacking Presentations**: Multiple talks at DEF CON have demonstrated Crestron, Control4, and Savant vulnerabilities including command injection and hardcoded credentials.
- **SEC Consult - Crestron AM-100/AM-101 Vulnerabilities (2018)**: Disclosed command injection, hardcoded credentials, and other flaws in Crestron presentation systems.
- **Mirai Botnet Analysis (2016)**: US-CERT Alert TA16-288A on the Mirai IoT botnet that leveraged default credentials on hundreds of thousands of IoT devices.
- **SANS Institute - IoT Security**: https://www.sans.org/white-papers/ - Multiple papers on IoT attack surfaces and smart home vulnerabilities.
- **Akamai UPnProxy Research**: Research revealing attackers exploiting UPnP in routers to create proxy networks for malicious traffic redirection.

---

## Search Tags

```
tags: [nmap, shodan, wireshark, crestron, upnp, ssdp, iot, smart-home, default-credentials, femtocell, cyber-physical]
season: 2
episode: 1
mitre: [T1046, T1110, T1190, T1565, T1219, T1200, T1529]
```
