# Password Management

All credentials across the lab environment are stored in a KeePass database on my host machine. This includes service accounts, SSH passphrases, MariaDB users, DVWA logins, Active Directory accounts, and firewall admin credentials. This vault is protected by a very complex 30 character master password. This allows convience and security for this lab, but in a real enterprise network a centralized secrets manager or privileged access management (PAM) solution would be used instead to support multi-user access, audit logging, and automated credential rotation. This may be something I implement at a future date.

![KeePass database](/links/screenshots/keypass/KeePassXC.jpeg)

## Why KeePass

- Offline — the database file stays local, no cloud sync
- Encrypted with AES-256
- Organises credentials by category (servers, services, domain accounts, etc.)

## What's Stored

| Category | Examples |
|---|---|
| SSH | Key passphrase for ed25519 pair |
| Database | MariaDB `dvwa` user, root credentials |
| Web Apps | DVWA admin login |
| Network | pfSense admin credentials |
| Active Directory | Domain admin and service accounts |
| Splunk | Splunk web UI credentials |
