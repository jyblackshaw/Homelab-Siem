# Homelab-SIEM

A segmented virtual enterprise network built from the ground up for practicing offensive and defensive cybersecurity. The environment simulates a real corporate network with isolated subnets, a centralized firewall, Active Directory, a vulnerable web application, and a SIEM - designed to demonstrate the full lifecycle of an attack from initial exploitation to detection and response.

## Network Architecture

The network is built on VMware Workstation and segmented into four subnets, all routed and filtered through a central pfSense firewall enforcing a default-deny policy.

<!-- Network diagram -->
<img width="1137" height="816" alt="Network Diagram" src="https://github.com/user-attachments/assets/5ef6e0ac-3874-4406-b39f-99b99870d52d" />

### Subnets

| Network | Subnet | Purpose |
|---|---|---|
| ATTACK_NET | 10.10.10.0/24 | Simulated external attacker network (Kali Linux) |
| DMZ_NET | 172.16.0.0/24 | Demilitarized zone hosting the public-facing DVWA webserver |
| LAN_NET | 192.168.10.0/24 | Corporate network - Domain Controller, database server, workstations |
| SOC_NET | 192.168.20.0/24 | Security operations - Splunk SIEM server and SOC analyst workstation |

### Machines

| VM | OS | IP | Subnet | Role |
|---|---|---|---|---|
| pfSense | pfSense CE | Gateway on each subnet | All | Central firewall and router |
| Kali-Attacker | Kali Linux | 10.10.10.10 | ATTACK_NET | Penetration testing workstation |
| Web-Server (DVWA) | Ubuntu Server | 172.16.0.200 | DMZ_NET | DVWA web application (Apache + PHP) |
| DB-Server | Ubuntu Server | 192.168.10.50 | LAN_NET | MariaDB backend for DVWA |
| DC-Server | Windows Server | 192.168.10.10 | LAN_NET | Active Directory Domain Controller |
| Win10-Client | Windows 10/11 | DHCP | LAN_NET | Corporate workstation |
| Splunk-Server | Ubuntu Server | 192.168.20.100 | SOC_NET | Splunk SIEM - log aggregation and alerting |
| SOC-Admin | Windows 10/11 | 192.168.20.50 | SOC_NET | SOC analyst workstation and bastion host |

## Key Security Principles

- **Default-Deny Firewall Policy:** All traffic is blocked unless explicitly allowed. Each rule is scoped to specific source, destination, protocol, and port.
- **Network Segmentation:** Subnets isolate attacker, DMZ, corporate, and SOC environments. Lateral movement between zones is restricted by firewall rules.
- **Separated Web and Database Tiers:** DVWA runs on a webserver in the DMZ while its MariaDB database sits on a separate server in the LAN, connected only on port 3306.
- **Centralized Logging:** Endpoints forward logs to the Splunk SIEM on SOC_NET for monitoring, correlation, and alerting.
- **SSH Hardening:** Key-based authentication via ed25519, managed from the SOC-Admin bastion host.

## Documentation

| Document | Description |
|---|---|
| [Firewall Rules](docs/firewall-rules.md) | pfSense rule sets for each subnet with rationale |
| [Database Setup](docs/database-setup.md) | MariaDB installation, DVWA configuration, and remote database architecture |
| [Splunk Setup](docs/splunk-setup.md) | SIEM deployment, forwarder configuration, indexes, and dashboards |
| [Active Directory](docs/active-directory.md) | Domain Controller setup, OUs, GPOs, and domain-joined Linux hosts |
| [SSH Hardening](docs/ssh-hardening.md) | Key-based authentication, bastion host setup, and access controls |

## Attack Writeups

| Writeup | Description |
|---|---|
| [Command Injection → DB Compromise](attacks/command-injection.md) | Full attack chain from DVWA input field to database credential theft and password cracking |
| [SQL Injection](attacks/sql-injection.md) | *Planned* |
| [Brute Force](attacks/brute-force.md) | *Planned* |
| [XSS (Reflected & Stored)](attacks/xss.md) | *Planned* |

## Defense Documentation

| Document | Description |
|---|---|
| [Network Segmentation](defenses/network-segmentation.md) | How subnet isolation and firewall rules contain lateral movement |
| [IDS Monitoring](defenses/ids-monitoring.md) | Intrusion detection configuration and alert tuning |
| [Log Analysis & Alerting](defenses/log-analysis.md) | SPL queries, correlation searches, and Splunk alert dashboards |

## Status

This project is actively being built out. Current priorities:
- Completing DVWA attack modules and writeups
- Active Directory GPO configuration and domain-joining Linux hosts
- Splunk forwarder deployment and dashboard creation
- Tightening SOC_NET firewall rules

---

*Built by Julian Yeshen Blackshaw*
