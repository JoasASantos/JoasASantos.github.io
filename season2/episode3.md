# Season 2, Episode 4: eps2.2_init_1.asec

## Episode Overview

The episode title references `init 1`, the Linux command to switch to single-user mode (runlevel 1). Single-user mode is a minimal system state used for maintenance and recovery -- only the root user is active, networking is disabled, and most services are stopped. This mirrors Elliot's isolated state as he helps Ray migrate his dark web site to a new server, discovering in the process the horrifying nature of Ray's marketplace. The technical focus is on dark web infrastructure migration -- moving a Tor hidden service from one server to another while maintaining operational continuity and anonymity.

---

## Hacks & Techniques

### 1. Linux init 1 / Single-User Mode

The `init` system is the first process started by the Linux kernel (PID 1) and manages all other processes. The traditional SysV init system uses numbered runlevels to define system states.

**Linux Runlevels:**

| Runlevel | State | Description |
|---|---|---|
| 0 | Halt | System shutdown |
| 1 / S | Single-user | Minimal mode, root only, no networking |
| 2 | Multi-user | Multi-user without networking (Debian: with networking) |
| 3 | Multi-user | Full multi-user with networking (text mode) |
| 4 | Undefined | Available for custom use |
| 5 | Graphical | Multi-user with networking and GUI |
| 6 | Reboot | System reboot |

**Single-User Mode (Runlevel 1):**

```bash
# Switch to single-user mode
init 1
# or
telinit 1
# or with systemd
systemctl isolate rescue.target
```

**Characteristics of Single-User Mode:**

- Only the root filesystem is mounted (read-only initially in many distributions)
- No network services are running
- No other users can log in
- Only a root shell is provided
- Used for filesystem repair, password recovery, and system maintenance
- The metaphorical equivalent of "working in isolation" -- which is exactly Elliot's state

**Why Single-User Mode Matters for the Episode:**

Elliot is operating in his own "single-user mode" -- isolated from his former life, running minimal processes, doing maintenance on himself. When Ray asks him to help with a server migration, Elliot is essentially being asked to come out of single-user mode and interact with the network again.

### 2. Dark Web Site Migration

Ray's dark web marketplace needs to be migrated to a new server. This is a complex operation that involves transferring the web application, database, and Tor hidden service configuration while maintaining anonymity and minimizing downtime.

**Migration Planning:**

A dark web site migration involves several interdependent components:

1. **Database migration**: Export all data from the old server and import to the new one
2. **Web application migration**: Transfer all application files, configurations, and dependencies
3. **Tor hidden service migration**: Transfer the hidden service private key to maintain the same `.onion` address
4. **Security hardening**: Ensure the new server is properly configured and hardened
5. **DNS/routing**: Update any internal routing or proxy configurations
6. **Testing**: Verify everything works before cutting over

**Critical Consideration -- The .onion Address:**

The `.onion` address of a Tor hidden service is derived from its private key. To maintain the same address after migration (so existing users can still find the site), the private key file must be transferred to the new server. This file is the most sensitive component of the entire operation -- if compromised, an attacker could impersonate the hidden service.

### 3. Database Migration with mysqldump

The marketplace's database (likely MySQL/MariaDB) contains all user accounts, listings, transaction records, and messages. The `mysqldump` utility creates a logical backup that can be transferred and imported on the new server.

**Full Database Export:**

```bash
# Dump all marketplace databases
mysqldump -u root -p --all-databases --single-transaction \
  --routines --triggers --events > marketplace_full_dump.sql

# Dump a specific database with compression
mysqldump -u root -p marketplace_db --single-transaction | \
  gzip > marketplace_db.sql.gz

# For InnoDB tables, --single-transaction ensures consistency
# without locking tables (important for minimal downtime)
mysqldump -u root -p --single-transaction --quick \
  --lock-tables=false marketplace_db > marketplace_db.sql
```

**Database Dump Contents:**

The SQL dump file contains:
- `CREATE DATABASE` and `CREATE TABLE` statements
- `INSERT` statements for all data rows
- Index and constraint definitions
- Stored procedures, triggers, and events
- Character set and collation settings

### 4. Secure File Transfer with scp and rsync

Transferring files between the old and new servers must be done securely, ideally over Tor to maintain anonymity.

**SCP (Secure Copy Protocol):**

```bash
# Copy the database dump to the new server
scp -i /path/to/key marketplace_full_dump.sql.gz \
  user@newserver:/var/backups/

# Copy the web application directory
scp -r -i /path/to/key /var/www/marketplace/ \
  user@newserver:/var/www/

# SCP over Tor using torsocks
torsocks scp -i /path/to/key marketplace_full_dump.sql.gz \
  user@hiddenservice.onion:/var/backups/
```

**rsync for Efficient Transfer:**

```bash
# Sync web application files with compression and progress
rsync -avz --progress /var/www/marketplace/ \
  user@newserver:/var/www/marketplace/

# Incremental sync (only changed files)
rsync -avz --delete /var/www/marketplace/ \
  user@newserver:/var/www/marketplace/

# rsync over Tor
rsync -avz -e "torsocks ssh -i /path/to/key" \
  /var/www/marketplace/ user@hiddenservice.onion:/var/www/marketplace/

# Exclude unnecessary files during transfer
rsync -avz --exclude='*.log' --exclude='cache/' --exclude='tmp/' \
  /var/www/marketplace/ user@newserver:/var/www/marketplace/
```

**rsync vs scp:**

- `rsync` is preferred for large transfers because it only sends differences (delta transfer)
- `rsync` can resume interrupted transfers
- `rsync` supports compression during transfer (`-z` flag)
- `scp` is simpler for single-file transfers

### 5. Tor Hidden Service Configuration and Migration

The most critical part of the migration is transferring the Tor hidden service configuration, which allows the new server to respond to the same `.onion` address.

**Tor Configuration File (torrc):**

```bash
# /etc/tor/torrc on the new server

# Basic Tor configuration
SocksPort 9050
Log notice file /var/log/tor/notices.log

# Hidden service configuration
HiddenServiceDir /var/lib/tor/marketplace/
HiddenServicePort 80 127.0.0.1:8080
HiddenServicePort 443 127.0.0.1:8443

# Security hardening
HiddenServiceMaxStreams 100
HiddenServiceMaxStreamsCloseCircuit 1

# Optional: v3 onion service (more secure)
HiddenServiceVersion 3
```

**Key Files in HiddenServiceDir:**

```
/var/lib/tor/marketplace/
├── hostname            # Contains the .onion address
├── private_key         # RSA private key (v2) -- CRITICAL
├── hs_ed25519_public_key   # Ed25519 public key (v3)
├── hs_ed25519_secret_key   # Ed25519 private key (v3) -- CRITICAL
└── authorized_clients/     # Client authorization keys (if used)
```

**Migration of Hidden Service Keys:**

```bash
# On the old server: securely copy the hidden service directory
# This MUST be done over an encrypted channel

# Create an encrypted archive of the hidden service keys
tar czf - /var/lib/tor/marketplace/ | \
  gpg --symmetric --cipher-algo AES256 > hs_backup.tar.gz.gpg

# Transfer the encrypted archive
scp hs_backup.tar.gz.gpg user@newserver:/tmp/

# On the new server: decrypt and extract
gpg --decrypt /tmp/hs_backup.tar.gz.gpg | tar xzf - -C /

# Set correct ownership and permissions
chown -R debian-tor:debian-tor /var/lib/tor/marketplace/
chmod 700 /var/lib/tor/marketplace/
chmod 600 /var/lib/tor/marketplace/private_key

# Restart Tor to load the hidden service
systemctl restart tor
```

### 6. HiddenServiceDir and HiddenServicePort Configuration

**HiddenServiceDir:**

This directive specifies the directory where Tor stores the hidden service's cryptographic keys and hostname file. Each hidden service needs its own directory.

```bash
# Single hidden service
HiddenServiceDir /var/lib/tor/marketplace/
HiddenServicePort 80 127.0.0.1:8080

# Multiple hidden services on the same server
HiddenServiceDir /var/lib/tor/service1/
HiddenServicePort 80 127.0.0.1:8081

HiddenServiceDir /var/lib/tor/service2/
HiddenServicePort 80 127.0.0.1:8082
```

**HiddenServicePort:**

This maps external-facing ports (what clients connect to) to internal services.

```bash
# Format: HiddenServicePort VIRTUAL_PORT TARGET
HiddenServicePort 80 127.0.0.1:8080    # HTTP
HiddenServicePort 443 127.0.0.1:8443   # HTTPS (rarely needed, Tor provides encryption)
HiddenServicePort 22 127.0.0.1:22      # SSH (for admin access over Tor)
HiddenServicePort 5222 127.0.0.1:5222  # XMPP (for chat services)
```

**Security Considerations:**

- The target address should always be `127.0.0.1` (localhost) to prevent accidental exposure
- The web application should only bind to localhost, never to `0.0.0.0`
- Firewall rules should block all incoming traffic except from Tor
- The hidden service directory must have strict permissions (700)

### 7. Web Server Configuration for Hidden Service

```bash
# Nginx configuration for the marketplace behind Tor
# /etc/nginx/sites-available/marketplace

server {
    listen 127.0.0.1:8080;
    server_name localhost;

    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";

    # Do NOT add server version info
    server_tokens off;

    root /var/www/marketplace;
    index index.php index.html;

    # PHP processing
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # Deny access to sensitive files
    location ~ /\.(ht|git) {
        deny all;
    }

    # Access log (consider disabling for anonymity)
    access_log off;
    error_log /var/log/nginx/marketplace_error.log crit;
}
```

### 8. Database Import on New Server

```bash
# Import the database dump on the new server
mysql -u root -p < marketplace_full_dump.sql

# Or import compressed dump
gunzip < marketplace_db.sql.gz | mysql -u root -p marketplace_db

# Verify import
mysql -u root -p -e "USE marketplace_db; SHOW TABLES; SELECT COUNT(*) FROM users;"

# Update database connection settings in the application
vim /var/www/marketplace/config/database.php
```

---

## Tools Used

| Tool | Purpose |
|---|---|
| **mysqldump** | Database export/backup utility |
| **mysql** | Database import and management |
| **scp** | Secure file transfer between servers |
| **rsync** | Efficient file synchronization |
| **torsocks** | Route arbitrary traffic through Tor |
| **Tor** | Anonymous hidden service hosting |
| **gpg** | Encryption of sensitive files during transfer |
| **Nginx** | Web server for the marketplace application |
| **systemctl** | Service management for Tor and web services |

---

## Commands Shown

**Complete Migration Sequence:**

```bash
# === ON OLD SERVER ===

# Step 1: Dump the database
mysqldump -u root -p --single-transaction marketplace_db | \
  gzip > /tmp/marketplace_db.sql.gz

# Step 2: Archive web application files
tar czf /tmp/marketplace_www.tar.gz -C /var/www marketplace/

# Step 3: Securely archive Tor hidden service keys
tar czf - /var/lib/tor/marketplace/ | \
  gpg -c --cipher-algo AES256 -o /tmp/tor_hs.tar.gz.gpg

# Step 4: Transfer all archives to new server
torsocks scp /tmp/marketplace_db.sql.gz \
  /tmp/marketplace_www.tar.gz \
  /tmp/tor_hs.tar.gz.gpg \
  admin@newserver.onion:/tmp/

# === ON NEW SERVER ===

# Step 5: Import database
gunzip < /tmp/marketplace_db.sql.gz | mysql -u root -p

# Step 6: Extract web application
tar xzf /tmp/marketplace_www.tar.gz -C /var/www/

# Step 7: Restore Tor hidden service keys
gpg -d /tmp/tor_hs.tar.gz.gpg | tar xzf - -C /
chown -R debian-tor:debian-tor /var/lib/tor/marketplace/
chmod 700 /var/lib/tor/marketplace/

# Step 8: Configure and start services
systemctl restart nginx
systemctl restart php-fpm
systemctl restart tor

# Step 9: Verify hidden service is accessible
curl --socks5-hostname 127.0.0.1:9050 http://$(cat /var/lib/tor/marketplace/hostname)
```

**Switching to Single-User Mode:**

```bash
# Traditional SysV init
init 1
# or
telinit 1

# systemd equivalent
systemctl isolate rescue.target

# From GRUB bootloader (for recovery)
# Edit kernel line and append: init=/bin/bash
# or: single
# or: systemd.unit=rescue.target
```

---

## Real-World Parallels

### Dark Web Marketplace Migrations

Real dark web marketplaces frequently need to migrate infrastructure:

- **Server seizure evasion**: When operators suspect law enforcement is closing in, they migrate to new infrastructure
- **Hosting provider changes**: "Bulletproof hosting" providers in jurisdictions with lax enforcement are used, but these relationships can break down
- **DDoS attacks**: Competing marketplaces or extortionists frequently DDoS dark web sites, necessitating infrastructure changes
- **AlphaBay** reportedly migrated servers multiple times during its operation to evade detection

### Tor Hidden Service Security

- **Freedom Hosting (2013)**: The FBI compromised the largest Tor hidden service hosting provider by exploiting a Firefox vulnerability in the Tor Browser. This demonstrates that even with Tor, operational security failures can lead to identification.

- **Tor Hidden Service Protocol Weaknesses**: Researchers have demonstrated attacks against the hidden service protocol, including:
  - **Sybil attacks on the DHT**: Placing malicious nodes at positions in the DHT where hidden service descriptors are stored
  - **Traffic correlation attacks**: Monitoring enough Tor relays to correlate entry and exit traffic
  - **Application-layer leaks**: Hidden services that leak their real IP through application-level errors

### init 1 and System Recovery

- Single-user mode remains a critical tool for system administrators recovering from failed updates, corrupted configurations, or compromised systems
- In forensic contexts, booting into single-user mode (or from external media) is often the first step in examining a potentially compromised system
- Modern Linux distributions using systemd have replaced the numeric runlevel system with "targets," but the concept of an isolated recovery mode persists as `rescue.target` and `emergency.target`

---

## Tool Links

| Tool | Link |
|---|---|
| Tor | https://www.torproject.org/ |
| Tor Hidden Services | https://community.torproject.org/onion-services/ |
| GnuPG/GPG | https://gnupg.org/ |
| MySQL | https://www.mysql.com/ |

---

## MITRE ATT&CK Mapping

| Episode Technique | MITRE ATT&CK ID | Name | Link |
|---|---|---|---|
| Hidden service migration (infrastructure) | T1583 | Acquire Infrastructure | https://attack.mitre.org/techniques/T1583/ |
| Secure file transfer (scp/rsync) | T1105 | Ingress Tool Transfer | https://attack.mitre.org/techniques/T1105/ |
| Encrypted channel usage (Tor + GPG) | T1573 | Encrypted Channel | https://attack.mitre.org/techniques/T1573/ |
| Database migration (mysqldump) | T1005 | Data from Local System | https://attack.mitre.org/techniques/T1005/ |
| Protocol tunneling (torsocks) | T1572 | Protocol Tunneling | https://attack.mitre.org/techniques/T1572/ |
| Remote service via Tor | T1133 | External Remote Services | https://attack.mitre.org/techniques/T1133/ |

---

## References and Further Reading

- **Freedom Hosting Takedown (2013)**: FBI operation that compromised the largest Tor hidden service hosting provider using a Firefox exploit in the Tor Browser.
- **Tor Project - Onion Service Protocol Documentation**: https://community.torproject.org/onion-services/ - Official documentation on how onion services work.
- **SANS Reading Room - Dark Web Infrastructure**: https://www.sans.org/white-papers/ - Papers on dark web hosting, migration, and operational security.
- **AlphaBay Takedown (2017)**: DOJ case documenting infrastructure migration patterns used by dark web marketplace operators.
- **DEF CON 25 - Tor Hidden Service Attacks**: Presentations on Sybil attacks against the DHT and traffic correlation techniques.

---

## Search Tags

```
tags: [tor, hidden-service, mysql, mysqldump, scp, rsync, gpg, torsocks, dark-web, migration, init, single-user-mode]
season: 2
episode: 4
mitre: [T1583, T1105, T1573, T1005, T1572, T1133]
```
