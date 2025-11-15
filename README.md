# Homelab-Siem

In this project I designed, built up, and deployed a mock virtual company network, to test cyber security skills and to practice offensive red teaming skills. Note this is still a work in progress; there is still missing documentation & demo videos I will add to this README in the future.  

# Design

I designed this project from the ground up based on security principles. Desinging with the CIA triad, zero trust, and .. in mind.
During design I used Ciso Packet Tracer to plan the network architecture. I eventually settled on the network you see in the image below (although this was iterated on during actual network implementation). 

<img width="1137" height="816" alt="Screenshot 2025-11-14 164149" src="https://github.com/user-attachments/assets/5ef6e0ac-3874-4406-b39f-99b99870d52d" />

### There are 4 main subnets:
1. **ATTACK_NET**:<br>
This subnet contains machines used for conducting attacks against systems on the internal subnets, specifically the Kali Linux machine. In a real-world environment, this traffic would normally originate from the external network (WAN), but a dedicated attack subnet is used here for control and isolation. The subnet IP range is **10.10.10.0/24**.
2. **DMZ_NET**:<br>
This subnet contains systems that are part of the Demilitarized Zone (DMZ), primarily the external firewall. The subnet IP range is **172.16.0.0/24**.
3. **SOC_NET**:<br>
This subnet serves as the security operations network and includes systems such as the SOC workstation and the SOC admin machine. The subnet IP range is **192.168.2.0/24**.
4. **LAN_NET**:<br>
This subnet is the main corporate network. It contains machines such as employee laptops, the database server, and the domain controller. The subnet IP range is **192.168.1.0/24**.

## Machines:
