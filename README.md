# vlsm-static-routing-wan-lab
Engineered and designed lab network using VLSM to create subnets with a different number of hosts per LAN. Static routing configurations &amp; ICMP connectivity testing finalized successful network traffic.
# Networking Lab via Cisco network equipment: VLSM Subnetting and Static Routing Across Four LANs

## Project Overview

This project demonstrates the design, configuration, troubleshooting, and verification of a multi-LAN network using Cisco Packet Tracer.

The network consists of two Cisco routers connected via point-to-point WAN link. The connection of gateways provides connectivity between four labeled LANs: California, Texas, New York, and Florida.

The objective I set for myself was to implement efficient IPv4 addressing using Variable Length Subnet Masking (VLSM), then configure all necccessary static routes on both gateways, establish default routes, and verify end-to-end connectivity.

## Network Topology

The lab includes:

- 2 Cisco 2911 routers (R1 and R2)
- 4 Layer 2 switches
- 4 end-user PCs
- 4 LAN subnets
- 1 point-to-point WAN subnet

R1 provides connectivity to the California and Texas LANs, while R2 provides connectivity to the New York and Florida LANs.

The routers communicate via the `192.168.5.240/30` network. I'll put a pic below:

<img width="945" height="633" alt="CCNA VLSM Topology" src="https://github.com/user-attachments/assets/afc5374a-b9f1-436a-b31c-db8a4adf8f53" />


## IPv4 Addressing and VLSM Design

The network was subnetted from the `192.168.5.0/24` address using VLSM to assign different blocks of address space according to host requirements per LAN.

| Location | Network Address | Subnet Mask | Required Hosts | Usable Hosts |
|---|---|---|---|---|
| Texas | 192.168.5.0/25 | 255.255.255.128 | 64 | 126 |
| California | 192.168.5.128/26 | 255.255.255.192 | 45 | 62 |
| New York | 192.168.5.192/27 | 255.255.255.224 | 14 | 30 |
| Florida | 192.168.5.224/28 | 255.255.255.240 | 9 | 14 |
| WAN | 192.168.5.240/30 | 255.255.255.252 | 2 | 2 |

Using VLSM allowed each LAN to receive a subnet appropriate for its host requirements while minimizing the possibility of wasted IPs and overall poor distribution

## Router Interface Configuration

Each router was configured with three GigabitEthernet interfaces. I made sure the G0 interface numbers weren't in order with the IP scheme to better mimic enterprise networks that may not be as organized as a lab.

**Router R1**

| Interface | IP Address | Purpose |
|---|---|---|
| G0/0 | 192.168.5.190/26 | California LAN |
| G0/1 | 192.168.5.126/25 | Texas LAN |
| G0/2 | 192.168.5.241/30 | WAN connection to R2 |

**Router R2**

| Interface | IP Address | Purpose |
|---|---|---|
| G0/0 | 192.168.5.222/27 | New York LAN |
| G0/1 | 192.168.5.238/28 | Florida LAN |
| G0/2 | 192.168.5.242/30 | WAN connection to R1 |

## Static Routing Configuration

Static routing was implemented to provide communication between LANs connected to different routers.

<img width="650" height="701" alt="CCNA VLSM routing table   last resort gateway config R1" src="https://github.com/user-attachments/assets/7da61d0f-ca75-435d-8217-9f2f9482c936" />


```text
ip route 192.168.5.192 255.255.255.224 192.168.5.242
ip route 192.168.5.224 255.255.255.240 192.168.5.242
```

<img width="741" height="745" alt="CCNA VLSM routing table   last resort gateway config R2" src="https://github.com/user-attachments/assets/8add83ea-2051-4c72-95ab-6d9cd0db4d21" />


```text
ip route 192.168.5.0 255.255.255.128 192.168.5.241
ip route 192.168.5.128 255.255.255.192 192.168.5.241
```

Each route specifies a destination network, subnet mask, and next-hop IP address (R1/R2 int's).

## Default Route Configuration

Default static routes were configured on both routers to demonstrate gateway-of-last-resort behavior. I configured this to each gateway to ensure that all unknown traffic (unknown destination IP addresses via packet's IPv4 Headers) is to be forwarded out to the internet. In a real environment, the router would complete this action by receiving the packet, sending it directly towards the ONT provided by the ISP, to be sent to the Internet via fiber optic connection.


**R1**

```text
ip route 0.0.0.0 0.0.0.0 192.168.5.242
```

**R2**

```text
ip route 0.0.0.0 0.0.0.0 192.168.5.241
```

The default route will also be used when a packet's destination does not match a more specific route.

**Lab limitation:** These default routes point toward each other. They demonstrate default-route configuration but do not provide functional Internet access. An upstream network and appropriate routing configuration would be needed for external connectivity.

## End-Device Configuration and Connectivity Testing

Each PC was assigned a static IPv4 address, subnet mask, and default gateway corresponding to its LAN.

Connectivity was tested using ICMP echo requests via the `ping` command.

During testing, initial ping attempts experienced timeouts. Additional attempts successfully received ICMP echo replies with 0% packet loss in the captured test (I repeated this for each host). The initial timeouts occur because devices are still resolving local next-hop (LAN gateway) MAC addresses using ARP.

## Packet-Level Analysis Using Cisco Packet Tracer Simulation Mode

### Objective

To observe and analyze ICMP traffic between devices on separate LANs using Cisco Packet Tracer's Simulation Mode.

<img width="967" height="452" alt="CCNA VLSM Simulation ping from pc1 to pc2 pt  1" src="https://github.com/user-attachments/assets/0d36b20d-17ac-4148-96df-03b806784f1f" />

### Testing Procedure

I initiated an ICMP ping from PC1 (California LAN) to PC2 (New York LAN), allowing me to observe packet forwarding across the network.

The communication path involved:

**PC1 → SW1 → R1 → R2 → SW3 → PC2**

Using Simulation Mode, I inspected the Protocol Data Unit (PDU) information at the switch to examine the Ethernet and IPv4 headers.

### Packet Analysis

The captured PDU displayed the following information:

| Field | Observed Value |
|---|---|
| Source IPv4 address | 192.168.5.129 |
| Destination IPv4 address | 192.168.5.193 |
| Source MAC address | 0060.2F60.2D98 |
| Destination MAC address | 0060.4781.0A01 |
| EtherType | 0x0800 (IPv4) |
| IP Protocol | 0x01 (ICMP) |
| IP TTL | 128 |

### Key Observations

**Layer 2 — Data Link Layer**

The Ethernet frame contained the source MAC address of PC1 and the destination MAC address of R1's G0/0 interface.

Because PC2 resides on a different subnet, PC1 forwarded the packet toward its default gateway rather than addressing the Ethernet frame directly to PC2.

**Layer 3 — Network Layer**

The IPv4 header identified PC1 (`192.168.5.129`) as the source and PC2 (`192.168.5.193`) as the destination.

These IP addresses identify the original endpoints, while the Ethernet source and destination MAC addresses are rewritten as necessary when routers forward packets across different network segments.

**ICMP**

The IPv4 protocol field contained `0x01`, identifying ICMP as the encapsulated protocol used for connectivity testing.

### What I Learned

This exercise reinforced my understanding of how Layer 2 switching and Layer 3 routing work together to deliver packets between different networks.

Simulation Mode provided additional visibility into packet encapsulation and the forwarding process beyond what is available through standard ping testing.

### Supporting Evidence

- Simulation Mode screenshot showing the network topology and packet path
- PDU Details screenshot displaying Ethernet, IPv4, and ICMP header information

## Troubleshooting Experience

One configuration issue encountered during the lab involved an invalid destination network in a static route.

**Incorrect command:**

```text
ip route 192.168.5.193 255.255.255.224 192.168.5.242
```

Cisco IOS returned:

```text
%Inconsistent address and mask
```

The issue occurred because `192.168.5.193` is a usable host address within the `192.168.5.192/27` subnet, not the subnet's network address.

**Corrected command:**

```text
ip route 192.168.5.192 255.255.255.224 192.168.5.242
```

This troubleshooting exercise reinforced the importance of distinguishing network addresses from host addresses when configuring static routes.

## Verification Commands

The following Cisco IOS commands were used to inspect routing and configuration information:

```text
show ip route
show running-config
```

The routing tables were checked for accurate connected (`C`), local (`L`), static (`S`), and default static (`S*`) routes.

All configurations were saved using:

```text
write memory
```

## Key Skills Demonstrated

- IPv4 subnetting and VLSM
- Cisco IOS CLI configuration
- Router interface addressing
- Point-to-point WAN connectivity
- Static and default routing
- Routing table interpretation
- ARP and ICMP fundamentals
- Network troubleshooting
- Configuration verification and persistence
- Technical documentation

## Lessons Learned

This project strengthened my skill in subnetting, and my understanding of how routers forward traffic between separate networks using static routes.

One of my biggest takeaways was learning that static routes require the correct destination network address and subnet mask rather than an arbitrary host address within that subnet.

I also gained practical experience interpreting Cisco routing tables, identifying next-hop addresses, and validating connectivity through hands-on testing.

Most importantly, the lab reinforced that successful network configuration involves more than entering commands. It requires understanding the TRUE ADDRESSING DESIGN, verifying expected behavior, and troubleshooting when results differ from expectations.

## Future Improvements

Potential extensions to this lab include:

- Implementing OSPF to compare dynamic and static routing
- Adding an ISP router to simulate external network connectivity
- Testing connectivity using `tracert` and `traceroute`
- Inspecting ARP tables and MAC address tables
- Implementing the use of VLANs per dept. ranging from the same or different geographic locations
- Implementing ACLs to control traffic between LANs

## Tools and Technologies

**Simulation:** Cisco Packet Tracer  
**Networking:** IPv4, VLSM, Ethernet, ARP, ICMP, Static Routing  
**Devices:** Cisco 2911 routers, Layer 2 switches, PCs  
**CLI:** Cisco IOS

---

*This project is part of my hands-on networking and cybersecurity learning portfolio, focused on developing practical infrastructure configuration, troubleshooting, and network analysis skills.*
