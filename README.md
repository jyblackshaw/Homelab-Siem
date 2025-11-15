# Homelab-Siem

In this project I designed, built up, and deployed a mock virtual company network, to test cyber security skills and to practice offensive red teaming skills. Note this is still a work in progress; there is still missing documentation & demo videos I will add to this README in the future.  

# Design

I designed this project from the ground up based on security principles. Desinging with the CIA triad, zero trust, and .. in mind.
During design I used Ciso Packet Tracer to plan the network architecture. I eventually settled on the network you see in the image below (although this was iterated on during actual network implementation). 

<img width="1137" height="816" alt="Screenshot 2025-11-14 164149" src="https://github.com/user-attachments/assets/5ef6e0ac-3874-4406-b39f-99b99870d52d" />

There are 4 main subnets:
1. ATTACK_NET: This subnet contains machines used for attacking other machines on the internal subnets, specifically the Kali-Linux machine. In a real scenario this network would just be all external traffic (WAN), but we use a designated attack subnet for control purposes. The subnet IP range is 10.10.10.x/24.
2. DMZ_NET: This subnet contains machines that are apart of the Demilitarized Zone, namely the external firewall. The subnet IP range is 172.16.0.x/24.
3. SOC_NET: This subnet is the security network, and involves the SOC and SOC-admin machines. The subnet IP range is 192.168.2.x/24.
4. LAN_NET: This subnet is the main coporate network. It contain machines like employee laptops, the database server, and domain controller. The subnet IP range is 192.168.1.x/24.

## Machines:
