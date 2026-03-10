# SSH Hardening

Documentation of SSH key-based authentication setup, bastion host configuration, and access controls across the lab environment.

## Overview

All SSH access is managed from the **SOC-Admin** bastion host (192.168.20.50, Windows). A single ed25519 key pair is used to authenticate to all Linux servers. Password authentication will be disabled once MFA is configured.

## Key Pair Generation

On SOC-Admin (PowerShell):

```powershell
ssh-keygen -t ed25519
```

A passphrase was set on the private key for an additional layer of protection.

Keys stored at `C:\Users\Analyst\.ssh\`:
- `id_ed25519` — private key (stays on SOC-Admin)
- `id_ed25519.pub` — public key (copied to each server)

## Public Key Distribution

From SOC-Admin, the public key was copied to each server using `scp`:

```powershell
scp C:\Users\Analyst\.ssh\id_ed25519.pub dvwa-admin-user@172.16.0.200:/tmp/id_ed25519.pub
scp C:\Users\Analyst\.ssh\id_ed25519.pub soc_analyst@192.168.20.100:/tmp/id_ed25519.pub
scp C:\Users\Analyst\.ssh\id_ed25519.pub db@192.168.10.50:/tmp/id_ed25519.pub
```

On each Linux server, the key was installed:

```bash
mkdir -p ~/.ssh && chmod 700 ~/.ssh
cat /tmp/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
rm /tmp/id_ed25519.pub
```

## SSH Access Summary

| Server | IP | Subnet | Username | Command from SOC-Admin |
|---|---|---|---|---|
| Web-Server (DVWA) | 172.16.0.200 | DMZ_NET | dvwa-admin-user | `ssh dvwa-admin-user@172.16.0.200` |
| Splunk-Server | 192.168.20.100 | SOC_NET | soc_analyst | `ssh soc_analyst@192.168.20.100` |
| DB-Server | 192.168.10.50 | LAN_NET | db | `ssh db@192.168.10.50` |

## Firewall Rules

SSH access is scoped to SOC-Admin only. The following pfSense rules allow port 22 traffic from the bastion host:

| Source | Destination | Port | Purpose |
|---|---|---|---|
| 192.168.20.50 | 192.168.20.100 | 22 | SOC-Admin to Splunk-Server (SOC_NET, no cross-subnet rule needed) |
| 192.168.20.50 | 172.16.0.200 | 22 | SOC-Admin to Web-Server (SOC_NET → DMZ_NET) |
| 192.168.20.50 | 192.168.10.50 | 22 | SOC-Admin to DB-Server (SOC_NET → LAN_NET) |

No other hosts are permitted to initiate SSH connections to these servers.

## MFA (Planned)

*To be configured — multi-factor authentication for SSH using TOTP or similar.*

## SSH Log Forwarding to Splunk

*To be configured — SSH auth logs will be forwarded to Splunk for monitoring and alerting.*
