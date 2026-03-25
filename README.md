# Homelab-SOC: Enterprise Network Security Lab

A segmented virtual enterprise network built from the ground up for practicing offensive and defensive cybersecurity. The environment simulates a real corporate network with isolated subnets, a centralized firewall, Active Directory, a vulnerable web application, and a SIEM - designed to demonstrate the full lifecycle of an attack from initial exploitation to detection and response.

## Network Architecture

<!-- Network diagram -->
<img width="1137" height="816" alt="Network Diagram" src="links/screenshots/network-topo.png" />

## Project Sections

Below are the various sections of this project. Each covers a different aspect of the environment - from how the network is built, to the attacks performed against it, to the defenses put in place to detect and contain those attacks. Click into each section for a detailed writeup.

**[Architecture](docs/)** - Network layout, subnet design, machine roles, firewall configuration, and infrastructure setup <br>
**[Attacks](attacks/)** - Offensive exercises performed against the environment, with step-by-step writeups showing exploitation techniques and attack chains <br>
**[Defenses](defenses/)** - Detection, monitoring, and containment strategies used to identify and respond to the attacks <br>

## Architecture

| Document | Description |
|---|---|
| [Network Architecture](docs/network-architecture.md) | Subnet layout, machine inventory, and key security principles |
| [Firewall Rules](docs/firewall-rules.md) | pfSense rule sets for each subnet and port forwarding |
| [Active Directory](docs/active-directory.md) | Domain Controller setup, OUs, GPOs, and domain-joined hosts |
| [Database & Webserver Setup](docs/database-setup.md) | MariaDB installation, DVWA configuration, and remote database architecture |
| [Splunk Setup](docs/splunk-setup.md) | SIEM deployment, forwarder configuration, indexes, and dashboards |
| [SSH Hardening](docs/ssh-hardening.md) | Key-based authentication, bastion host setup, and access controls |

## Attack Writeups

| Writeup | Description |
|---|---|
| [Command Injection → DB Compromise](attacks/command-injection.md) | Full attack chain from DVWA input field to database credential theft and password cracking |
| [Brute Force](attacks/brute-force.md) | Brute force attack on admin login |
| [XSS (Reflected & Stored)](attacks/xss.md) | *Planned* |
| [Privilege Escalation]() | *Planned* |

## Defense Documentation

| Document | Description |
|---|---|
| [Log Analysis](defenses/log-analysis.md) | SPL queries, correlation searches, and Splunk alert dashboards |
| [Attack Detection](defenses/attack-detection.md) | Detection and investigation of attacks performed against the environment |
| [Alerts & Dashboards](defenses/alerts-dashboards.md) | Splunk alert rules, dashboards, and real-time monitoring |
| EDR / AV | *Planned* |
| [Vulnerability Scanning](defenses/vulnerability-scanning.md) | OpenVAS scanning, findings, and remediation |
| [IPS / IDS](defenses/ips-ids.md) | Suricata on pfSense - intrusion detection and prevention |

## Status

> **Note:** Much of the environment is built and functional, but documentation for several sections is still in progress.

Current priorities:
- Completing DVWA attack modules and writeups
- Active Directory GPO configuration
- Splunk dashboards & alerts
- Suricata IPS/IDS on pfSense
- EDR / AV
- Vulnerability Scanning
- In depth packet/log analysis.
- Putting DB & DC on seperate subnets.
- Policy Implementations (GRC).

---

*Built by Julian Yeshen Blackshaw*
