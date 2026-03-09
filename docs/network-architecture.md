# Network Architecture

The network is built on VMware Workstation and segmented into four subnets, all routed and filtered through a central pfSense firewall enforcing a default-deny policy.

## Subnets

| Network | Subnet | Purpose |
|---|---|---|
| ATTACK_NET | 10.10.10.0/24 | Simulated external attacker network (Kali Linux) |
| DMZ_NET | 172.16.0.0/24 | Demilitarized zone hosting the public-facing DVWA webserver |
| LAN_NET | 192.168.10.0/24 | Corporate network - Domain Controller, database server, workstations |
| SOC_NET | 192.168.20.0/24 | Security operations - Splunk SIEM server and SOC analyst workstation |

## Machines

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

*Additional network topology and layout diagrams will be added to this document as the environment evolves.*
