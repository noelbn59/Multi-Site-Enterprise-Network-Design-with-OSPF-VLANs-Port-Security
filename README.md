# Multi-Site-Enterprise-Network-Design-with-OSPF-VLANs-Port-Security
Multi-area OSPF network design with VLAN segmentation, inter-VLAN routing, DHCP per department, trunking, STP, and port security — built and verified in Cisco Packet Tracer/GNS3.


# Multi-Site Enterprise Network Design with OSPF, VLANs & Port Security

## Overview
This project simulates a multi-site enterprise network for a fictional company with a head office and two branch locations. It uses multi-area OSPF for scalable routing between sites, VLANs to logically separate departments (Sales, HR, IT), inter-VLAN routing for controlled communication, DHCP for automatic IP assignment per department, and port security to protect access-layer switch ports from unauthorized devices. Built and verified in Cisco Packet Tracer.

## Tools Used
- Cisco Packet Tracer (v8.x)
- Cisco IOS (Router: 2911 series, Switches: 2960 series, L3 Switch: 3560 series)
- Wireshark (optional, for traffic verification)

## Network Topology
![Network Topology](topology-diagram.png)

- **3 sites**: Head Office (Area 0) and two branch offices (Area 1 and Area 2), connected via serial WAN links
- **1 Layer 3 switch** at Head Office handling inter-VLAN routing
- **Multiple Layer 2 switches** per site handling department-level access ports
- **3 VLANs per site**: Sales (VLAN 10), HR (VLAN 20), IT (VLAN 30)
- End devices (PCs) representing employees in each department at each site

## IP Addressing Scheme

| Department/VLAN | VLAN ID | Network Address | Subnet Mask | Usable Range | Gateway |
|---|---|---|---|---|---|
| Sales | 10 | 192.168.10.0 | /24 | .1 – .254 | 192.168.10.1 |
| HR | 20 | 192.168.20.0 | /24 | .1 – .254 | 192.168.20.1 |
| IT | 30 | 192.168.30.0 | /24 | .1 – .254 | 192.168.30.1 |
| WAN Link (HO–Branch1) | – | 10.0.0.0 | /30 | .1 – .2 | – |
| WAN Link (HO–Branch2) | – | 10.0.0.4 | /30 | .1 – .2 | – |

## Configuration Steps

1. **VLAN Creation** — Created VLANs 10, 20, and 30 on all access-layer switches for Sales, HR, and IT.
2. **Trunking** — Configured 802.1Q trunk links between access switches and the Layer 3 switch, allowing only the required VLANs on each trunk.
3. **Inter-VLAN Routing** — Configured SVIs (Switched Virtual Interfaces) on the Layer 3 switch for each VLAN, enabling controlled routing between departments.
4. **Multi-Area OSPF** — Configured OSPF with the Head Office in Area 0 and each branch office in its own area (Area 1, Area 2), connected via Area Border Routers.
5. **DHCP** — Configured separate DHCP pools per VLAN so each department at each site receives IP addresses automatically, using `ip helper-address` where the DHCP server is not local to the subnet.
6. **Spanning Tree Protocol (STP)** — Verified STP is active on all switches to prevent Layer 2 loops, with the Layer 3 switch set as the root bridge.
7. **Port Security** — Enabled port security on access ports, restricting each port to a maximum of 1–2 MAC addresses and setting a violation action of `shutdown` to block unauthorized devices.

## Key Configuration Snippets

```
! VLAN and SVI configuration on L3 switch
interface Vlan10
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 192.168.30.10
!
interface Vlan20
 ip address 192.168.20.1 255.255.255.0
!
ip dhcp pool SALES
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8

! OSPF configuration
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
 area 1 range 192.168.0.0 255.255.0.0

! Port Security on access switch
interface FastEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
```

## Verification
- `show vlan brief` — confirms VLANs are created and correct ports are assigned
- `show ip route ospf` — confirms OSPF routes from other areas are learned correctly
- `show ip ospf neighbor` — confirms OSPF adjacency is formed between routers
- `ping` between hosts in different VLANs and across sites — confirms end-to-end connectivity
- `show ip dhcp binding` — confirms DHCP is assigning correct addresses per VLAN
- `show port-security interface Fa0/1` — confirms port security is active and shows secured MAC address
- `show spanning-tree` — confirms no loops and correct root bridge election

## What I Learned
This project helped me understand how multi-area OSPF improves scalability by summarizing routes at area boundaries instead of flooding the entire network with routing updates. I also learned how DHCP relay (`ip helper-address`) works when the DHCP server isn't on the same subnet as the requesting client, and how port security protects access-layer switches from unauthorized or rogue devices by limiting and locking down MAC addresses per port.

## References
- Topology and configuration structure inspired by [CCNA_Complex_Network_Design](https://github.com/HoosseinRahimi/CCNA_Complex_Network_Design)
- Cisco official documentation on OSPF, DHCP, and Port Security (cisco.com)

---
**Author:** Noel Binu
**Certification:** CCNA (Cisco Certified Network Associate)
**Tools:** Cisco Packet Tracer / GNS3
