# Season 2, Episode 6: eps2.4_m4ster-s1ave.aes

## Episode Overview

This episode's title combines two concepts: master-slave database replication architecture and AES (Advanced Encryption Standard) encryption. The episode is notable for its surreal dream sequence in which Elliot envisions himself in a 1990s sitcom, processing his psychological conflict with Mr. Robot. Beneath the surface-level narrative, the technical themes of data replication and encryption reflect the show's ongoing concerns with data integrity, redundancy, and the protection of information.

---

## Hacks & Techniques

### 1. AES Encryption: Advanced Encryption Standard

AES is the most widely used symmetric encryption algorithm in the world. Selected by NIST in 2001 as the replacement for DES (Data Encryption Standard), AES is used to protect classified government information, secure financial transactions, encrypt communications, and protect stored data.

**AES Fundamentals:**

- **Type**: Symmetric block cipher (same key for encryption and decryption)
- **Block size**: 128 bits (16 bytes) -- always, regardless of key size
- **Key sizes**: 128, 192, or 256 bits
- **Structure**: Substitution-permutation network (SPN)
- **Rounds**: 10 rounds (AES-128), 12 rounds (AES-192), 14 rounds (AES-256)

**AES-256:**

AES-256 uses a 256-bit key, providing the highest level of security in the AES family. The key space is 2^256, which is astronomically large -- there are more possible AES-256 keys than atoms in the observable universe.

**Each Round of AES Consists of Four Operations:**

1. **SubBytes**: Each byte is replaced using a substitution table (S-box), providing non-linearity
2. **ShiftRows**: Rows of the state array are cyclically shifted by different offsets
3. **MixColumns**: Columns are mixed using a mathematical transformation (not applied in the final round)
4. **AddRoundKey**: The round key (derived from the main key via key schedule) is XORed with the state

```
Plaintext (128 bits)
        |
   AddRoundKey (Round 0 -- initial round key)
        |
   +-----------+
   | SubBytes  |
   | ShiftRows |     x 13 rounds (for AES-256)
   | MixColumns|
   | AddRoundKey|
   +-----------+
        |
   SubBytes
   ShiftRows          Final round (no MixColumns)
   AddRoundKey
        |
   Ciphertext (128 bits)
```

### 2. AES Modes of Operation

Since AES operates on fixed 128-bit blocks, a mode of operation is required to encrypt messages longer than one block. The choice of mode has critical security implications.

**CBC (Cipher Block Chaining):**

```
Plaintext Block 1    Plaintext Block 2    Plaintext Block 3
       |                    |                    |
       v                    v                    v
   XOR with IV         XOR with C1          XOR with C2
       |                    |                    |
       v                    v                    v
   AES Encrypt         AES Encrypt          AES Encrypt
       |                    |                    |
       v                    v                    v
  Ciphertext C1        Ciphertext C2        Ciphertext C3
```

- **How it works**: Each plaintext block is XORed with the previous ciphertext block before encryption
- **IV (Initialization Vector)**: A random value used for the first block to ensure identical plaintexts produce different ciphertexts
- **Pros**: Widely supported, errors do not propagate beyond two blocks
- **Cons**: Sequential encryption (cannot be parallelized), vulnerable to padding oracle attacks if not implemented carefully (see POODLE, Lucky 13)
- **Use cases**: Disk encryption, file encryption, legacy TLS

**CTR (Counter Mode):**

```
Counter 0        Counter 1        Counter 2
    |                |                |
    v                v                v
AES Encrypt     AES Encrypt     AES Encrypt
    |                |                |
    v                v                v
XOR with P0     XOR with P1     XOR with P2
    |                |                |
    v                v                v
Ciphertext C0   Ciphertext C1   Ciphertext C2
```

- **How it works**: A counter value is encrypted, and the output is XORed with the plaintext
- **Pros**: Parallelizable (both encryption and decryption), no padding needed, random access to any block
- **Cons**: Counter must NEVER be reused with the same key (nonce reuse is catastrophic)
- **Use cases**: High-performance encryption, network protocols, streaming encryption

**GCM (Galois/Counter Mode):**

- **How it works**: Combines CTR mode encryption with Galois field multiplication for authentication
- **Provides both**: Confidentiality (encryption) AND integrity/authenticity (authentication tag)
- **AEAD**: Authenticated Encryption with Associated Data -- can authenticate unencrypted header data while encrypting the payload
- **Tag**: Produces a 128-bit authentication tag that detects any tampering
- **Pros**: Fast, parallelizable, provides authentication, widely adopted
- **Cons**: Nonce reuse is catastrophic (reveals auth key), tag size matters for security
- **Use cases**: TLS 1.3, IPsec, SSH, modern protocols

```bash
# AES-256-GCM encryption with OpenSSL
openssl enc -aes-256-gcm -in plaintext.txt -out encrypted.bin \
  -K $(hexdump -n 32 -e '"%02x"' /dev/urandom) \
  -iv $(hexdump -n 12 -e '"%02x"' /dev/urandom)
```

### 3. AES in Practice

**Common Applications of AES:**

| Application | Mode Typically Used | Key Size |
|---|---|---|
| Full disk encryption (LUKS, BitLocker, FileVault) | XTS (CBC variant) | 256-bit |
| TLS 1.3 | GCM, CCM | 128 or 256-bit |
| IPsec VPN | GCM, CBC | 256-bit |
| SSH | CTR, GCM | 128 or 256-bit |
| File encryption (GPG, 7-Zip) | CFB, CBC | 256-bit |
| Database encryption (TDE) | CBC, GCM | 256-bit |
| Wi-Fi (WPA2/WPA3) | CCMP (CTR+CBC-MAC) | 128-bit |

### 4. Master-Slave Database Replication

Master-slave replication is a database architecture pattern where data is written to a primary (master) server and automatically copied to one or more secondary (slave) servers. This provides data redundancy, read scalability, and high availability.

**Architecture:**

```
                    [Write Queries]
                          |
                          v
                    +----------+
                    |  MASTER  |
                    | (Primary)|
                    +----------+
                     /    |    \
                    /     |     \        Binary Log Replication
                   v      v      v
             +-------+ +-------+ +-------+
             | SLAVE | | SLAVE | | SLAVE |
             |  (1)  | |  (2)  | |  (3)  |
             +-------+ +-------+ +-------+
                ^         ^         ^
                |         |         |
           [Read Queries distributed across slaves]
```

**How MySQL Master-Slave Replication Works:**

1. **Binary Log (binlog)**: The master records all data-changing operations in its binary log
2. **I/O Thread**: Each slave has an I/O thread that connects to the master and reads binlog events
3. **Relay Log**: The slave writes received events to its relay log
4. **SQL Thread**: The slave's SQL thread reads the relay log and replays the events against the local database

**Types of Replication:**

- **Statement-Based Replication (SBR)**: The actual SQL statements are replicated. Compact but can have non-deterministic behavior.
- **Row-Based Replication (RBR)**: The actual changed rows are replicated. More data but deterministic.
- **Mixed Replication**: Uses statement-based by default and switches to row-based when necessary.

### 5. Setting Up MySQL Master-Slave Replication

**Master Configuration (`/etc/mysql/mysql.conf.d/mysqld.cnf`):**

```ini
[mysqld]
# Unique server ID (must be different for each server)
server-id = 1

# Enable binary logging
log_bin = /var/log/mysql/mysql-bin.log

# Database to replicate (optional, replicates all if omitted)
binlog_do_db = marketplace_db

# Binary log format
binlog_format = ROW

# Retention period for binary logs
expire_logs_days = 7

# Sync binary log to disk on each write (for durability)
sync_binlog = 1
```

**Slave Configuration:**

```ini
[mysqld]
server-id = 2
relay-log = /var/log/mysql/mysql-relay-bin.log
read_only = 1
```

**Setting Up Replication:**

```sql
-- On the MASTER: Create replication user
CREATE USER 'repl_user'@'%' IDENTIFIED BY 'secure_password';
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'%';
FLUSH PRIVILEGES;

-- On the MASTER: Get current binary log position
SHOW MASTER STATUS;
-- +------------------+----------+
-- | File             | Position |
-- +------------------+----------+
-- | mysql-bin.000003 |      154 |
-- +------------------+----------+

-- On the SLAVE: Configure and start replication
CHANGE MASTER TO
  MASTER_HOST='master_ip',
  MASTER_USER='repl_user',
  MASTER_PASSWORD='secure_password',
  MASTER_LOG_FILE='mysql-bin.000003',
  MASTER_LOG_POS=154;

START SLAVE;

-- Verify replication status
SHOW SLAVE STATUS\G
-- Key fields to check:
-- Slave_IO_Running: Yes
-- Slave_SQL_Running: Yes
-- Seconds_Behind_Master: 0
```

### 6. High Availability Concepts

**Beyond Simple Master-Slave:**

- **Master-Master Replication**: Both servers can accept writes, with changes replicated bidirectionally. Risk of conflicts requires conflict resolution strategies.
- **Group Replication**: MySQL Group Replication provides virtually synchronous replication with automatic failover and conflict detection.
- **Galera Cluster**: Synchronous multi-master replication for MySQL/MariaDB with automatic node joining and state transfer.
- **Read Replicas**: Multiple read-only slaves distribute read queries, improving performance for read-heavy workloads.

**Failover Scenarios:**

```
Normal Operation:            After Master Failure:
[Master] --> [Slave 1]       [Master] X (down)
         --> [Slave 2]       [Slave 1] promoted to new Master
         --> [Slave 3]       [Slave 2] --> repoints to Slave 1
                             [Slave 3] --> repoints to Slave 1
```

---

## Tools Used

| Tool | Purpose |
|---|---|
| **OpenSSL** | AES encryption/decryption operations |
| **MySQL/MariaDB** | Database management and replication |
| **GPG** | File encryption using AES-256 |
| **LUKS/cryptsetup** | Linux disk encryption with AES |
| **mysqldump** | Database backup for replication initialization |
| **mysqlbinlog** | Binary log analysis and point-in-time recovery |

---

## Commands Shown

**AES Encryption with OpenSSL:**

```bash
# Encrypt a file with AES-256-CBC
openssl enc -aes-256-cbc -salt -pbkdf2 \
  -in sensitive_data.txt \
  -out sensitive_data.enc

# Decrypt
openssl enc -d -aes-256-cbc -pbkdf2 \
  -in sensitive_data.enc \
  -out sensitive_data.txt

# Encrypt with a specific key and IV (hex)
openssl enc -aes-256-cbc -nosalt \
  -K 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef \
  -iv 0123456789abcdef0123456789abcdef \
  -in plaintext.txt -out ciphertext.bin

# Generate a random 256-bit key
openssl rand -hex 32

# Generate a random 128-bit IV
openssl rand -hex 16
```

**AES Encryption in Python:**

```python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
import os

# AES-256-GCM encryption
key = get_random_bytes(32)  # 256-bit key
nonce = get_random_bytes(12)  # 96-bit nonce for GCM

cipher = AES.new(key, AES.MODE_GCM, nonce=nonce)

# Encrypt with authenticated data
header = b"authenticated but not encrypted"
cipher.update(header)

plaintext = b"This is the secret message"
ciphertext, tag = cipher.encrypt_and_digest(plaintext)

# Decryption
cipher_dec = AES.new(key, AES.MODE_GCM, nonce=nonce)
cipher_dec.update(header)
decrypted = cipher_dec.decrypt_and_verify(ciphertext, tag)
```

**MySQL Replication Commands:**

```bash
# Check replication status on slave
mysql -e "SHOW SLAVE STATUS\G" | grep -E "Running|Behind|Error"

# Monitor binary log events on master
mysqlbinlog --no-defaults /var/log/mysql/mysql-bin.000003 | head -50

# Skip a problematic replication event
mysql -e "STOP SLAVE; SET GLOBAL sql_slave_skip_counter = 1; START SLAVE;"

# Check master status
mysql -e "SHOW MASTER STATUS\G"

# Compare master and slave data consistency
pt-table-checksum --replicate=percona.checksums h=master_ip
```

**Linux Full Disk Encryption with AES:**

```bash
# Create a LUKS encrypted volume using AES-256-XTS
cryptsetup luksFormat --cipher aes-xts-plain64 \
  --key-size 512 --hash sha512 /dev/sdb1

# Open the encrypted volume
cryptsetup luksOpen /dev/sdb1 encrypted_vol

# Create filesystem and mount
mkfs.ext4 /dev/mapper/encrypted_vol
mount /dev/mapper/encrypted_vol /mnt/secure

# Check encryption details
cryptsetup luksDump /dev/sdb1
```

---

## Real-World Parallels

### AES in Global Security

- **AES is the backbone of internet security**: Every HTTPS connection you make likely uses AES-GCM as part of TLS 1.3. Billions of AES encryption/decryption operations happen every second across the internet.

- **Hardware acceleration**: Modern CPUs include dedicated AES-NI (AES New Instructions) that perform AES operations in hardware, achieving throughput of several gigabytes per second. This makes AES encryption essentially "free" from a performance perspective on modern hardware.

- **Government classification**: AES-256 is approved by the NSA for protecting TOP SECRET information. It is the only publicly available cipher approved for this classification level.

- **Quantum resistance**: AES-256 is considered resistant to quantum computing attacks via Grover's algorithm, which would effectively halve the key strength. AES-256 under quantum attack would still provide 128-bit security, which remains strong.

### Database Replication in Production

- **Every major internet service** relies on database replication for availability and performance. Facebook, Google, Amazon, and other tech giants use replication extensively.

- **GitHub outage (2018)**: A 24-hour outage was caused by a failure in MySQL master-slave replication after a network partition. The incident highlighted the complexity of maintaining data consistency across replicas.

- **Split-brain scenarios**: When network partitions occur and multiple nodes believe they are the master, conflicting writes can occur. This is one of the fundamental challenges in distributed systems, described by the CAP theorem.

### Master-Slave Terminology

- The industry has been moving away from "master-slave" terminology toward "primary-replica" or "source-replica" terminology. MySQL 8.0.22+ introduced `CHANGE REPLICATION SOURCE TO` as a replacement for `CHANGE MASTER TO`, and many other database systems have adopted similar changes.

### Encryption in E Corp's World

- In the context of Mr. Robot, encryption is both a tool and a theme. The 5/9 hack encrypted E Corp's financial records, destroying (or hiding) the data that tracks consumer debt. The irony is that the same technology (AES) that protects legitimate data was used to hold it hostage -- a direct parallel to real-world ransomware attacks that use AES or similar algorithms to encrypt victim data.

---

## Tool Links

| Tool | Link |
|---|---|
| OpenSSL | https://www.openssl.org/ |
| GnuPG/GPG | https://gnupg.org/ |
| MySQL | https://www.mysql.com/ |

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| AES encryption of data (E Corp ransomware) | T1486 | Data Encrypted for Impact | https://attack.mitre.org/techniques/T1486/ |
| Encrypted channel for communications | T1573 | Encrypted Channel | https://attack.mitre.org/techniques/T1573/ |
| Data manipulation (replication) | T1565 | Data Manipulation | https://attack.mitre.org/techniques/T1565/ |
| File obfuscation with encryption | T1027 | Obfuscated Files or Information | https://attack.mitre.org/techniques/T1027/ |
| Local system data collection | T1005 | Data from Local System | https://attack.mitre.org/techniques/T1005/ |

---

## References and Further Reading

- **NIST FIPS 197 - AES Standard**: Official NIST specification for the Advanced Encryption Standard.
- **CVE-2014-3566** (POODLE - SSL 3.0 CBC vulnerability): https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-3566
- **GitHub MySQL Replication Outage (2018)**: Post-incident analysis of the 24-hour outage caused by MySQL replication failure.
- **SANS Institute - Encryption Best Practices**: https://www.sans.org/white-papers/ - Papers on AES implementation and key management.
- **Ransomware and AES**: Research papers on how ransomware families like WannaCry, Ryuk, and Conti use AES for file encryption.

---

## Search Tags

```
tags: [aes, encryption, openssl, gpg, mysql, database-replication, master-slave, luks, disk-encryption, ransomware]
season: 2
episode: 6
mitre: [T1486, T1573, T1565, T1027, T1005]
```
