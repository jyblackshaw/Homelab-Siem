# Homelab-Siem

In this project I designed, built up, and deployed a mock virtual company network, to test cyber security skills and to practice offensive red teaming skills. Note this is still a work in progress; there is still missing documentation & demo videos I will add to this README in the future.  

# Design

I designed this project from the ground up based on security principles. Desinging with the CIA triad, zero trust, and .. in mind.
During design I used Ciso Packet Tracer to plan the network architecture. I eventually settled on the network you see in the image below (although this was iterated on during actual network implementation). 

<img width="1137" height="816" alt="Screenshot 2025-11-14 164149" src="https://github.com/user-attachments/assets/5ef6e0ac-3874-4406-b39f-99b99870d52d" />

## There are 4 main subnets:
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
| **DC-Server** | Domain Controller providing authentication, DNS, and DHCP services for the corporate network. Some services are still under development. | Windows Server | **LAN_NET:** Static — 192.168.1.10 | LAN_NET |
| **DB-Server** | Internal database server hosting MySQL data for DVWA. Located within the LAN and configured to accept connections only from the Web-Server. | Ubuntu Server LTS | **LAN_NET:** Static — 192.168.1.50 | LAN_NET |
| **Win10-Client** | Employee workstation representing a standard end-user device within the corporate network. Will eventually receive DHCP configuration from the DC-Server. | Windows 10/11 Pro | **LAN_NET:** Dynamic | LAN_NET |
| **Splunk-Server** | SIEM server responsible for collecting, indexing, and correlating security logs from all network segments, enabling SOC monitoring and threat detection. | Ubuntu Server LTS | **SOC_NET:** Static — 192.168.2.100 | SOC_NET |
| **SOC-ADMIN** | Secure management workstation used for SOC operations, including Splunk access and incident investigation. | Windows 10/11 Pro | **SOC_NET:** Static — 192.168.2.50 | SOC_NET |
