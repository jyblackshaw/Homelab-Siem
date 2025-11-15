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

| VM Name | Role / Function | OS | IP Configuration | Network Adapter(s) |
| :--- | :--- | :--- | :--- | :--- |
| **pfSense** | **Central Security Gateway (Firewall)** Acts as the single point of defense, enforcing routing, **access control (firewall rules)**, and network segmentation between all subnets (WAN, ATTACK, DMZ, SOC, LAN). | pfSense CE | **ATTACK\_NET**: 10.10.10.1<br>**DMZ\_NET**: 172.16.0.1<br>**LAN\_NET**: 192.168.1.1<br>**SOC\_NET**: 192.168.2.1 | 4 (one for each internal subnet) |
| **Kali-Attacker** | **External Threat Emulation** Used for **Offensive (Red Team) exercises** to launch controlled attacks (e.g., port scanning, brute-force, web app attacks) against the targets in the DMZ and LAN. | Kali Linux | **ATTACK\_NET**: Static: 10.10.10.10 | ATTACK\_NET |
| **Web-Server** | **Vulnerable Web Host (DMZ)** Public-facing server hosting the **Damn Vulnerable Web Application (DVWA)**. It is the primary target for web application attacks. Generates logs and contains forwarder for the SOC. | Ubuntu Server LTS | **DMZ\_NET**: Static: 172.16.0.100 | DMZ\_NET |
| **DC-Server** | **Domain Controller (LAN)** Manages central authentication, DNS, and DHCP services for the internal corporate `LAN_NET`. Some of the functionality is still a WIP. | Windows Server | **LAN\_NET**: Static: 192.168.1.10 | LAN\_NET |
| **DB-Server** | **Internal Database Host** Stores sensitive user data (MySQL) of the DVWA. It is secured within the `LAN_NET` and configured to be accessible only by the Web-Server. | Ubuntu Server LTS | **LAN\_NET**: Static: 192.168.1.50 | LAN\_NET |
| **Win10-Client** | **Employee Workstation (LAN)** Simulates a standard end-user machine in the internal corporate network, will eventually use DHCP from the DC-Server. | Windows 10/11 Pro | **LAN\_NET**: Dynamic | LAN\_NET |
| **Splunk-Server** | **SIEM Platform (SOC)** Hosts Splunk server. It centrally collects, indexes, and correlates all security logs from the firewall and servers to allow real-time threat detection. | Ubuntu Server LTS | **SOC\_NET**: Static: 192.168.2.100 | SOC\_NET |
| **SOC-ADMIN** | **Secure Management Workstation** Used exclusively for security management tasks, accessing the Splunk interface, and investigating incidents. | Windows 10/11 Pro | **SOC\_NET**: Static: 192.168.2.50 | SOC\_NET |
