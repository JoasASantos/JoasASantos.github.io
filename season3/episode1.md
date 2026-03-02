# Season 3, Episode 1: eps3.0_power-saver-mode.h

## Episode Overview

The Season 3 premiere opens with Elliot struggling to regain control after Mr. Robot's actions during the gap in his memory. Stage 2 -- the plan to physically destroy E Corp's paper records by detonating UPS batteries in their storage facility -- is already in motion. Elliot must figure out the full scope of Stage 2 while contending with the Dark Army's grip on the plan. The `.h` file extension references a C/C++ header file, symbolizing that this episode lays the foundational declarations for everything that follows this season.

---

## Hacks & Techniques

### UPS Firmware Manipulation (Stage 2 Core Attack)

Stage 2 revolves around a firmware attack targeting Uninterruptible Power Supply (UPS) units installed in E Corp facilities that house paper-based financial records. The attack modifies the UPS firmware to disable thermal protection circuits, causing lithium-ion batteries to enter **thermal runaway** -- an uncontrolled, self-heating chain reaction that leads to fire and explosion.

**How the attack works:**

1. **Reconnaissance**: Identify the UPS make and model deployed across E Corp facilities. Many enterprise environments standardize on a single vendor (e.g., APC, Eaton, Vertiv).
2. **Firmware reverse engineering**: Extract the existing firmware from a sample UPS unit via JTAG, UART, or the vendor's management interface. Disassemble the firmware binary to understand the battery management system (BMS) logic.
3. **Malicious firmware creation**: Modify the BMS routines to disable overcharge protection, override temperature cutoff thresholds, and suppress alarm reporting to the network management card.
4. **Deployment**: Push the modified firmware to target UPS units through the network management interface (typically SNMP or a proprietary web interface). Many UPS network cards accept unsigned firmware, making this feasible.
5. **Trigger**: On command (or at a scheduled time), the modified firmware forces continuous charging at maximum voltage, driving cells into thermal runaway.

**Thermal runaway mechanics:**

- Lithium-ion cells become unstable above approximately 150 degrees Celsius
- Once one cell enters thermal runaway, it heats adjacent cells, causing a cascade
- The resulting fire produces toxic gases (hydrogen fluoride) and temperatures exceeding 600 degrees Celsius
- Enterprise rack-mounted UPS systems contain enough energy to ignite surrounding equipment and building materials

```
// Pseudocode: Modified BMS firmware logic
#define MAX_CHARGE_VOLTAGE 4.2    // Normal per-cell max
#define MODIFIED_MAX_VOLTAGE 5.5  // Force overcharge
#define TEMP_CUTOFF_DISABLED 1    // Bypass thermal protection

void battery_charge_loop() {
    while (1) {
        set_charge_voltage(MODIFIED_MAX_VOLTAGE);
        if (TEMP_CUTOFF_DISABLED) {
            // Suppress all thermal alarms
            clear_alarm_register(THERMAL_ALARM);
            disable_fan_control();
        }
        // Do not report actual voltage/temp to NMC
        report_nominal_values_to_nmc();
        sleep(CHARGE_INTERVAL);
    }
}
```

### HSM (Hardware Security Module) Access Attempt

Elliot recognizes that E Corp's encryption keys -- used during the 5/9 hack to encrypt their financial data -- may be recoverable from Hardware Security Modules. HSMs are tamper-resistant hardware devices designed to securely generate, store, and manage cryptographic keys.

**HSM characteristics relevant to the plot:**

- **Tamper-evident and tamper-resistant**: Physical intrusion attempts trigger zeroization (key destruction)
- **FIPS 140-2/3 certified**: Meet federal standards for cryptographic module security
- **Role-based access control**: Requires M-of-N authentication (multiple key custodians) for sensitive operations
- **Audit logging**: All key operations are cryptographically logged
- **Key hierarchy**: Master keys protect working keys; master keys never leave the HSM in plaintext

Elliot's challenge is that even with system-level access to E Corp's network, the HSM's keys cannot simply be copied. The keys are bound to the hardware and require proper authentication ceremonies with multiple authorized personnel present.

```bash
# Example: Interacting with a PKCS#11 interface to an HSM
# List available slots and tokens
pkcs11-tool --module /usr/lib/softhsm/libsofthsm2.so --list-slots

# Attempt to list objects (requires PIN)
pkcs11-tool --module /usr/lib/softhsm/libsofthsm2.so --list-objects --login --pin ????

# Export public key (private key cannot be exported from a real HSM)
pkcs11-tool --module /usr/lib/softhsm/libsofthsm2.so --read-object --type pubkey --id 01
```

### Dark Army Operational Security and Compartmentalization

The Dark Army operates with strict intelligence-style compartmentalization:

- **Cell structure**: Operatives know only their immediate handler and their specific task. No operative has visibility into the full operation.
- **Need-to-know basis**: Information is distributed only to those who require it to complete their assignment.
- **Cutouts**: Intermediaries are used between layers of the organization to prevent direct links between leadership (Whiterose) and field operatives.
- **Operational silence**: Communications are kept to an absolute minimum. When contact is necessary, it occurs through pre-arranged methods.
- **Plausible deniability**: Each layer of the organization can be sacrificed without exposing upper layers.

### Tor and Anonymity Tools

Communications between Dark Army operatives and fsociety members rely on anonymity networks:

- **Tor (The Onion Router)**: Routes traffic through multiple encrypted relays, concealing the user's IP address and location
- **Onion services (.onion addresses)**: Allow hosting of services that are only accessible through the Tor network, hiding both client and server
- **Tails OS**: A live operating system that routes all traffic through Tor and leaves no trace on the host machine

```bash
# Starting Tor service
sudo systemctl start tor

# Verifying Tor circuit
curl --socks5-hostname localhost:9050 https://check.torproject.org/api/ip

# Using torsocks to route arbitrary applications through Tor
torsocks ssh user@onionaddress.onion

# Generating a Tor onion service (in torrc)
# HiddenServiceDir /var/lib/tor/hidden_service/
# HiddenServicePort 80 127.0.0.1:8080
```

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **Custom UPS firmware** | Modified battery management system firmware to disable thermal protections |
| **JTAG/UART debugger** | Hardware interfaces for firmware extraction from UPS controllers |
| **IDA Pro / Ghidra** | Disassemblers for reverse engineering UPS firmware binaries |
| **Tor** | Anonymized communications between operatives |
| **Tails OS** | Amnesic live operating system for operational security |
| **PKCS#11 tools** | Interface for interacting with HSM cryptographic tokens |
| **SNMP** | Protocol used for UPS network management card communication |

---

## Commands Shown

```bash
# UPS network management card interaction via SNMP
snmpget -v2c -c public <ups_ip> 1.3.6.1.4.1.318.1.1.1.2.2.1.0  # Battery temperature
snmpget -v2c -c public <ups_ip> 1.3.6.1.4.1.318.1.1.1.2.2.2.0  # Battery voltage
snmpset -v2c -c private <ups_ip> 1.3.6.1.4.1.318.1.1.1.7.2.4.0 i 2  # Firmware update trigger

# Firmware extraction and analysis
binwalk -e firmware.bin                    # Extract filesystem from firmware image
strings firmware.bin | grep -i "version"   # Identify firmware version strings
hexdump -C firmware.bin | head -100        # Examine firmware header

# Tor hidden service setup
tor --hash-password "mypassword"           # Generate hashed password for Tor control
cat /var/lib/tor/hidden_service/hostname   # Retrieve .onion address
```

---

## Real-World Parallels

### UPS/Battery Attacks
- **Samsung Galaxy Note 7 (2016)**: Battery design flaws caused thermal runaway in consumer devices, demonstrating the real danger of lithium-ion battery failures. Samsung recalled 2.5 million units after numerous fires and explosions.
- **UPS vulnerability research**: Security researchers have demonstrated vulnerabilities in UPS network management cards at conferences like DEF CON and Black Hat, showing that many enterprise UPS systems accept unsigned firmware updates.
- **Armis TLStorm (2022)**: Researchers at Armis discovered critical vulnerabilities in APC Smart-UPS devices that could allow remote code execution and firmware manipulation -- almost exactly matching the Stage 2 attack vector.

### HSM Security
- **DigiNotar breach (2011)**: Attackers compromised a Certificate Authority but were unable to extract keys from properly configured HSMs, demonstrating the hardware security boundary.
- **PKCS#11 security research**: Academic work has shown that the PKCS#11 API standard has logical flaws that can sometimes allow key extraction through sequences of legitimate API calls (Clulow, 2003; Bortolozzo et al., 2010).

### Dark Army Operational Model
- **APT groups (Fancy Bear, Lazarus Group, APT41)**: Real-world state-sponsored hacking groups operate with similar compartmentalization, using front companies and cutouts to maintain deniability.
- **Numbers stations and dead drops**: Intelligence agencies have historically used pre-arranged, one-way communication methods similar to the Dark Army's operational protocols.

## Tool Links

- [Tor](https://www.torproject.org/) - Anonymity network used for communications between operatives
- [Tails OS](https://tails.net/) - Amnesic operating system for operational security
- [Binwalk](https://github.com/ReFirmLabs/binwalk) - Firmware image analysis and extraction
- [GnuPG](https://gnupg.org/) - Encryption for secure communications
- [Wireshark](https://www.wireshark.org/) - Network traffic analysis and SNMP protocols
- [Nmap](https://nmap.org/) - Network scanning to identify UPS devices
- [Shodan](https://www.shodan.io/) - Search for internet-exposed UPS devices

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| UPS firmware manipulation | T1195 | Supply Chain Compromise | https://attack.mitre.org/techniques/T1195/ |
| UPS device reconnaissance | T1046 | Network Service Discovery | https://attack.mitre.org/techniques/T1046/ |
| Communication via Tor | T1572 | Protocol Tunneling | https://attack.mitre.org/techniques/T1572/ |
| Encrypted communication | T1573 | Encrypted Channel | https://attack.mitre.org/techniques/T1573/ |
| HSM and cryptographic key access | T1005 | Data from Local System | https://attack.mitre.org/techniques/T1005/ |
| Firmware extraction via JTAG/UART | T1200 | Hardware Additions | https://attack.mitre.org/techniques/T1200/ |
| Physical destruction via thermal runaway | T1561 | Disk Wipe | https://attack.mitre.org/techniques/T1561/ |
| Dark Army compartmentalization | T1583 | Acquire Infrastructure | https://attack.mitre.org/techniques/T1583/ |

## References and Further Reading

- **Armis TLStorm (CVE-2022-22805, CVE-2022-22806, CVE-2022-0715)**: Critical vulnerabilities in APC Smart-UPS devices allowing remote code execution and firmware manipulation -- attack vector nearly identical to Stage 2
- **NIST SP 800-147**: BIOS Protection Guidelines -- relevant for firmware protection against unauthorized modifications
- **DigiNotar CA Breach (2011)**: Demonstrated the importance of HSMs in protecting cryptographic keys
- **Clulow, J. (2003)**: "On the Security of PKCS#11" -- academic research on logical flaws in the PKCS#11 API
- **Bortolozzo et al. (2010)**: "Attacking and Fixing PKCS#11 Security Tokens" -- key extraction through sequences of legitimate API calls
- **Samsung Galaxy Note 7 Battery Failure Analysis (2016)**: Technical analysis of thermal runaway in lithium-ion batteries

## Search Tags

```
tags: [UPS, firmware, thermal-runaway, HSM, PKCS11, Tor, Tails, JTAG, UART, SNMP, Dark-Army, compartmentalization]
season: 3
episode: 1
mitre: [T1195, T1046, T1572, T1573, T1005, T1200, T1561, T1583]
```
