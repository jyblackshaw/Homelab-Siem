# Homelab-Siem

In this project, I designed, built, and deployed a mock virtual company network to practice cybersecurity concepts, including offensive (Red Team) techniques and defensive monitoring. This environment is still a work in progress—additional documentation, diagrams, and demonstration videos will be added in future updates.
## Design

This project was designed from the ground up with core security principles in mind, including the CIA Triad, Zero Trust, and network segmentation.
During the planning phase, I used Cisco Packet Tracer to prototype the network architecture. After several iterations, I finalized the design shown in the diagram below (with further refinements made during actual implementation)

<img width="1137" height="816" alt="Screenshot 2025-11-14 164149" src="https://github.com/user-attachments/assets/5ef6e0ac-3874-4406-b39f-99b99870d52d" />

## Subnets:
1. **ATTACK_NET**:<br>
This subnet contains machines used for conducting attacks against systems on the internal subnets, specifically the Kali Linux machine. In a real-world environment, this traffic would normally originate from the external network (WAN), but a dedicated attack subnet is used here for control and isolation. The subnet IP range is **10.10.10.0/24**.
2. **DMZ_NET**:<br>
This subnet contains systems that are part of the Demilitarized Zone (DMZ), primarily the external firewall. The subnet IP range is **172.16.0.0/24**.
3. **SOC_NET**:<br>
This subnet serves as the security operations network and includes systems such as the SOC workstation and the SOC admin machine. The subnet IP range is **192.168.2.0/24**.
4. **LAN_NET**:<br>
This subnet is the main corporate network. It contains machines such as employee laptops, the database server, and the domain controller. The subnet IP range is **192.168.1.0/24**.

## Machines:

The network is comprised of 9 virtual machines (VMs) deployed in VMware, each serving a specific role within the simulated enterprise and security environment.

| **VM Name** | **Role / Function** | **Operating System** | **IP Configuration** | **Network Adapter(s)** |
| :--- | :--- | :--- | :--- | :--- |
| **pfSense** | Central Security Gateway (Firewall). Acts as the primary security control point, enforcing routing, firewall rules, and network segmentation across all subnets (ATTACK, DMZ, LAN, SOC). Three-legged firewall model. | pfSense CE | **ATTACK_NET:** 10.10.10.1<br>**DMZ_NET:** 172.16.0.1<br>**LAN_NET:** 192.168.1.1<br>**SOC_NET:** 192.168.2.1 | 4 (one per subnet) |
| **Kali-Attacker** | Offensive security workstation used for controlled Red Team activities (e.g., scanning, exploitation, and attack simulation) targeting systems in the DMZ and LAN. | Kali Linux | **ATTACK_NET:** Static — 10.10.10.10 | ATTACK_NET |
| **Web-Server** | Public-facing web server hosting the Damn Vulnerable Web Application (DVWA). Serves as the primary target for web application attacks and forwards security logs to the SOC. | Ubuntu Server LTS | **DMZ_NET:** Static — 172.16.0.100 | DMZ_NET |
| **DC-Server** | Domain Controller providing authentication, DNS, and DHCP services for the corporate network. Still under development. | Windows Server | **LAN_NET:** Static — 192.168.1.10 | LAN_NET |
| **DB-Server** | Internal database server hosting MySQL data for DVWA. Located within the LAN and configured to accept connections only from the Web-Server. | Ubuntu Server LTS | **LAN_NET:** Static — 192.168.1.50 | LAN_NET |
| **Win10-Client** | Employee workstation representing a standard end-user device within the corporate network. Will eventually receive DHCP configuration from the DC-Server. | Windows 10/11 Pro | **LAN_NET:** Dynamic | LAN_NET |
| **Splunk-Server** | SIEM server responsible for collecting, indexing, and correlating security logs from all network segments, enabling SOC monitoring and threat detection. | Ubuntu Server LTS | **SOC_NET:** Static — 192.168.2.100 | SOC_NET |
| **SOC-ADMIN** | Secure management workstation used for SOC operations, including Splunk access and fire rule editing. | Windows 10/11 Pro | **SOC_NET:** Static — 192.168.2.50 | SOC_NET |

**Work In Progress:**
- DC-Server: DHCP functionality for LAN machines not yet configured.
- DB-Server: Currently experiencing issues with DVWA connecting to the MySQL database on the DB-Server. DVWA is temporarily using its internal MySQL instance, but the goal is to resolve the connection issues and transition to the external database as intended.

## Firewall Rules (pfSense)

The security architecture is enforced by the single **pfSense** firewall, managing all traffic between the segmented subnets with a default **DENY ALL** policy. Rules are organized by security flow, detailing the exact protocol and port numbers derived from the active configuration.

| Rule | Source Zone (Machine) | Destination Zone (Machine/Service) | Protocol | Port(s) | Purpose | Policy |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **I. External/Attack to DMZ** | **WAN** (Internet) | DMZ\_NET Address (Web-Server) | TCP | 80, 443 | Allow external access to the public Web-Server (HTTP/S). | **ALLOW** |
| | **ATTACK\_NET** (Kali-Attacker) | 172.16.0.100 (Web-Server) | TCP | 80 (HTTP) | Allow Kali VM to reach the Web-Server. | **ALLOW** |
| **II. DMZ to LAN** | **DMZ\_NET** (Web-Server) | 192.168.1.50 (DB-Server) | TCP | 3306 | Allow the Web-Server to connect to the internal Database (MySQL). | **ALLOW** |
| **III. DMZ to SOC** | **DMZ\_NET** (Web-Server) | 192.168.2.100 (Splunk-Server) | TCP | 9997 | Allow Web-Server to forward logs to the Splunk Universal Forwarder listener. | **ALLOW** |
| **IV. DMZ Outbound** | **DMZ\_NET** (Subnets) | WAN (Internet) | TCP/UDP | 80, 443 | Allow DMZ servers to access the web for system updates. | **ALLOW** |
| | **DMZ\_NET** (Subnets) | WAN (Internet) | UDP | 53 (DNS) | Allow DMZ servers to perform domain name lookups. | **ALLOW** |
| | **DMZ\_NET** (Subnets) | WAN (Internet) | IPv4 ICMP | Any | Allow DMZ servers to send Ping requests for basic network testing. | **ALLOW** |
| **V. LAN Internal Rule** | **LAN\_NET** (Subnets) | WAN (Internet) & DMZ\_NET | IPv4 All | Any | Allow general corporate user access to the Internet and the DMZ Web-Server. | **ALLOW** |
| | **LAN\_NET** (Subnets) | SOC\_NET (Subnets) | IPv4 All | Any | **SECURITY PRINCIPLE:** Block corporate user network access to the SOC network. | **BLOCK** |
| **VI. SOC Internal Rule** | **SOC\_NET** (Subnets) | WAN (Internet) | IPv4 All | Any | Allow SOC team full access to the Internet for threat intelligence and updates. | **ALLOW** |
| | **SOC\_NET** (SOC-ADMIN) | This Firewall (self) | TCP/UDP | 80, 443 | Allow the SOC-ADMIN machine to access the pfSense Web GUI for management. | **ALLOW** |

### **Default Policy** <br>
**The implicit firewall rule is DENY ALL traffic that is not explicitly permitted by the rules above.** This is the foundation of the network's zero-trust approach.

**Work In Progress:** <br>
This project is still a work in progress and some of these rules are subject to change. I will update the README when appropriate.
