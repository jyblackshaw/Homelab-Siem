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
| W11-Client | Windows 11 Enterprise | DHCP | LAN_NET | Corporate workstation |
| Splunk-Server | Ubuntu Server | 192.168.20.100 | SOC_NET | Splunk SIEM - log aggregation and alerting |
| SOC-Admin | Windows 11 | 192.168.20.50 | SOC_NET | SOC analyst workstation and bastion host |

### Machine Descriptions

**pfSense** <br>
The central security gateway of the network. All inter-subnet traffic passes through pfSense, where firewall rules enforce access control between zones. Also handles NAT routing for internet access, DNS resolution for all subnets (forwarding queries on behalf of hosts), DHCP for non-corporate subnets, and port forwarding from WAN to DMZ services.

**Kali-Attacker** <br>
Offensive security workstation used for red team exercises against the lab environment. Runs tools including Hydra, Burp Suite, John the Ripper, Nmap, and Netcat. Firewall rules restrict it to only reaching the DVWA webserver in the DMZ, simulating an external attacker with access limited to public-facing services.

**Web-Server (DVWA)** <br>
 Hosts the Damn Vulnerable Web Application on Apache with PHP. This is the primary attack target in the lab. It connects to a remote MariaDB database on CORP_NET for its backend and forwards logs to the Splunk server on SOC_NET. Port forwarding on pfSense exposes it through the WAN interface, simulating a public-facing web server.

**DB-Server** <br>
Hosts the MariaDB database used by DVWA, including both DVWA's application tables and custom fake data (customer PII, credit cards, employee records, credentials) used for SQL injection and data exfiltration exercises. Sits on CORP_NET and only accepts database connections from the DVWA webserver on port 3306.

**DC-Server** <br>
Active Directory Domain Controller for `corp.homelab.local`. Provides centralized identity management, DNS, and DHCP for CORP_NET. Manages user accounts, security groups, OUs, and Group Policy Objects that enforce password policies, account lockout, least-privilege access, and audit logging across domain-joined machines.

**W11-Client** <br>
Domain-joined corporate workstation representing a standard employee endpoint. Receives its IP via DHCP from the DC, authenticates users against Active Directory, and has GPOs applied that restrict local admin rights based on security group membership. Used to test and verify access controls for different roles (IT admin vs HelpDesk vs standard user).

**Splunk-Server** <br>
Centralized SIEM responsible for collecting, indexing, and correlating security logs from endpoints across all subnets. Receives logs via Splunk Universal Forwarders on port 9997. Used to build SPL queries, correlation searches, alert rules, and dashboards for detecting attacks performed in the lab.

**SOC-Admin** <br>
The security analyst's primary workstation and the central management point for the lab. Used to access the Splunk web interface for log analysis and alerting, the pfSense web GUI for firewall management, and SSH into Linux servers for administration.SSH access to servers is scoped to this machine only via firewall rules.

## Key Security Principles

- **Default-Deny Firewall Policy:** All traffic is blocked unless explicitly allowed. Each rule is scoped to specific source, destination, protocol, and port.
- **Network Segmentation:** Subnets isolate attacker, DMZ, corporate, and SOC environments. Lateral movement between zones is restricted by firewall rules.
- **Separated Web and Database Tiers:** DVWA runs on a webserver in the DMZ while its MariaDB database sits on a separate server in the LAN, connected only on port 3306.
- **Centralized Logging:** Endpoints forward logs to the Splunk SIEM on SOC_NET for monitoring, correlation, and alerting.
- **SSH Hardening:** Key-based authentication via ed25519, managed from the SOC-Admin bastion host.
- **Centralized DNS Resolution:** All hosts are manually configured to use the pfSense gateway on their subnet as their DNS server rather than an external resolver. pfSense forwards queries to the internet on behalf of each host. This allows DNS firewall rules to be scoped to the gateway IP only, blocking any direct DNS traffic to external servers. It also gives pfSense full visibility into every DNS query across the network, which can be logged and forwarded to Splunk for monitoring. The exception is CORP_NET - domain-joined machines use the Domain Controller (192.168.10.10) as their DNS server, which resolves internal domain names for CORP_NET hosts. The DB-Server is not domain-joined and uses pfSense directly. The DC itself forwards any external DNS queries it cannot resolve to pfSense.

| Subnet | DNS Server |
|---|---|
| DMZ_NET | 172.16.0.1 |
| CORP_NET | 192.168.10.1 |
| SOC_NET | 192.168.20.1 |

On Linux hosts this is set in `/etc/resolv.conf`. On Windows hosts it is configured in the network adapter's IPv4 DNS settings.

*Additional network topology and layout diagrams will be added to this document as the environment evolves.*
