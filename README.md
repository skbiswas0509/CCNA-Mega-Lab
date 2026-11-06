# Cisco CCNA 200-301 Mega Lab: Enterprise Network Architecture & Configuration Guide

Welcome to the **Cisco CCNA Mega Lab** repository. This project showcases a fully functional, enterprise-grade multi-tier network deployment built in Cisco Packet Tracer. Designed to cover the entire Cisco CCNA (200-301) syllabus, this lab integrates core switching, high-availability Layer 3 routing, first-hop redundancy, comprehensive infrastructure security, network services, dual-stack IPv6, and centralized wireless control into a unified infrastructure.

---

## 1. Network Architecture & Topology

The topology models a full enterprise infrastructure comprising a redundant **Internet Edge**, a high-speed **Core Layer**, and two distinct branch locations (**Office A** and **Office B**) utilizing a classic **3-Tier Campus Architecture** (Core, Distribution, Access).

```
                            +-------------------+       +-------------------+
                            |  ISP A (Internet) |       |  ISP B (Internet) |
                            +---------+---------+       +---------+---------+
                                      |                           |
                                  (DHCP)                      (DHCP)
                                      |                           |
                                +-----+---------------------------+-----+
                                |             Edge Router R1            |
                                +-------+-----------------------+-------+
                                        | 10.0.0.33/30          | 10.0.0.37/30
                                        |                       |
                  +---------------------+---+               +---+---------------------+
                  |   Core Switch CSW1      |<=============>|   Core Switch CSW2      |
                  +----+----+---------------+  P01: .41/.42 +----+----+---------------+
                       |    |    |    | 10.0.0.40/30             |    |    |    |
                       |    |    +-------------------------+----+    |    |
                       |    +----------------------------------+     |    |
                       |                                       |    |     |    |
             +---------+---------+                   +---------+----+-----+----+
             | DSW-A1   | DSW-A2 |                   | DSW-B1   | DSW-B2 |
             +----+-----+----+---+                   +----+-----+----+---+
                  \     /    /                            \     /    /
                   \   /    /                              \   /    /
             +------+--+---+-----+                   +------+--+---+-----+
             | Access Switches   |                   | Access Switches   |
             | ASW-A1, A2, A3    |                   | ASW-B1, B2, B3    |
             +----------+--------+                   +----------+--------+
                        |                                       |
             +----------+--------+                   +----------+--------+
             | End Hosts / WLC1  |                   | End Hosts / SRV1  |
             | LWAP1 / Phones/PCs|                   | LWAP2 / SRV1 / PCs|
             +-------------------+                   +-------------------+
```

### Architectural Highlights
* **Internet Edge & Edge Routing**: Router `R1` provides WAN connectivity with primary and floating backup static routes over ISP links, along with NAT/PAT translation services.
* **Core Layer (`CSW1`, `CSW2`)**: High-speed Cisco Catalyst multilayer switches configured with Layer 3 routed interfaces, a Layer 3 EtherChannel interconnect, and single-area OSPFv2.
* **Distribution Layer (`DSW-A1/A2`, `DSW-B1/B2`)**: Inter-office routing, SVI termination, Layer 2 PAgP/LACP EtherChannel trunking, and HSRPv2 gateway redundancy.
* **Access Layer (`ASW-A1`..`A3`, `ASW-B1`..`B3`)**: Edge port delivery with 802.1Q trunking, dynamic VLAN assignment, Voice VLANs, BPDU Guard, PortFast, DHCP Snooping, and Dynamic ARP Inspection (DAI).

---

## 2. IP Addressing & VLAN Subnetting Matrix

| Branch / Subnet | VLAN ID | Subnet CIDR | Default Gateway (HSRP VIP) | Primary Active Switch | Secondary Standby Switch | Description |
| :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| **Office A - Mgmt** | `99` | `10.0.0.0/28` | `10.0.0.1` | DSW-A1 (`10.0.0.2`) | DSW-A2 (`10.0.0.3`) | Switch SVI, WLC1, LWAP1 |
| **Office A - PCs** | `10` | `10.1.0.0/24` | `10.1.0.1` | DSW-A1 (`10.1.0.2`) | DSW-A2 (`10.1.0.3`) | Office A Workstations |
| **Office A - Phones**| `20` | `10.2.0.0/24` | `10.2.0.1` | DSW-A2 (`10.2.0.3`) | DSW-A1 (`10.2.0.2`) | IP Telephony Voice VLAN |
| **Office A - Wi-Fi** | `40` | `10.6.0.0/24` | `10.6.0.1` | DSW-A2 (`10.6.0.3`) | DSW-A1 (`10.6.0.2`) | Wireless Client Subnet |
| **Office B - Mgmt** | `99` | `10.0.0.16/28`| `10.0.0.17` | DSW-B1 (`10.0.0.18`) | DSW-B2 (`10.0.0.19`) | Switch SVI, LWAP2 |
| **Office B - PCs** | `10` | `10.3.0.0/24` | `10.3.0.1` | DSW-B1 (`10.3.0.2`) | DSW-B2 (`10.3.0.3`) | Office B Workstations |
| **Office B - Phones**| `20` | `10.4.0.0/24` | `10.4.0.1` | DSW-B2 (`10.4.0.3`) | DSW-B1 (`10.4.0.2`) | IP Telephony Voice VLAN |
| **Office B - Servers**| `30` | `10.5.0.0/24` | `10.5.0.1` | DSW-B2 (`10.5.0.3`) | DSW-B1 (`10.5.0.2`) | Central Server SRV1 (`10.5.0.4`)|
| **Native VLAN** | `1000`| Unused | N/A | N/A | N/A | Security Native Trunk VLAN |

### Point-to-Point Inter-Switch Links
* **R1 to CSW1**: `10.0.0.32/30` (R1 G0/0: `.33`, CSW1 G1/0/1: `.34`)
* **R1 to CSW2**: `10.0.0.36/30` (R1 G0/1: `.37`, CSW2 G1/0/1: `.38`)
* **CSW1 to CSW2 (PortChannel 1)**: `10.0.0.40/30` (CSW1: `.41`, CSW2: `.42`)
* **CSW1 to Distribution**: `10.0.0.44/30` (DSW-A1), `10.0.0.48/30` (DSW-A2), `10.0.0.52/30` (DSW-B1), `10.0.0.56/30` (DSW-B2)
* **CSW2 to Distribution**: `10.0.0.60/30` (DSW-A1), `10.0.0.64/30` (DSW-A2), `10.0.0.68/30` (DSW-B1), `10.0.0.72/30` (DSW-B2)
* **Loopback Addresses (/32)**: R1 (`10.0.0.76`), CSW1 (`10.0.0.77`), CSW2 (`10.0.0.78`), DSW-A1 (`10.0.0.79`), DSW-A2 (`10.0.0.80`), DSW-B1 (`10.0.0.81`), DSW-B2 (`10.0.0.82`)

---

## 3. Technology Implementation Breakdown

### Part 1: Initial Setup & Security Hardening
* **Hostnames**: Configured across all 13 network devices (`R1`, `CSW1/2`, `DSW-A1/A2/B1/B2`, `ASW-A1..A3/B1..B3`).
* **Privileged EXEC Security**: Configured type 9 scrypt hashing on Catalyst 3650 multilayer switches (`enable algorithm-type scrypt secret jeremysitlab`) and type 5 MD5 hashing on access switches and router `R1` (`enable secret jeremysitlab`).
* **Local User Accounts**: Created user `cisco` with secret `ccna` (type 9 on multilayer switches, type 5 on access switches/routers).
* **Console Line Security**: Configured `line console 0` with `login local`, 30-minute inactivity timeout (`exec-timeout 30 0`), and `logging synchronous`.

### Part 2: Layer 2 Infrastructure, Trunking & VTPv2
* **EtherChannel Aggregation**:
  * **Office A**: Configured PAgP Layer 2 Port-Channel 1 between `DSW-A1` and `DSW-A2` using `channel-group 1 mode desirable`.
  * **Office B**: Configured LACP Layer 2 Port-Channel 1 between `DSW-B1` and `DSW-B2` using `channel-group 1 mode active`.
* **Trunk Port Configuration**:
  * Configured explicit 802.1Q trunking on all inter-switch links (`switchport mode trunk`).
  * Explicitly disabled Dynamic Trunking Protocol (`switchport nonegotiate`).
  * Set unused **VLAN 1000** as the Native VLAN across all trunk links.
  * Restricted allowed VLANs per branch (`10,20,40,99` for Office A; `10,20,30,99` for Office B).
* **VLAN Trunking Protocol (VTP)**:
  * Established VTP Domain `JeremysITLab` running `vtp version 2`.
  * `DSW-A1` and `DSW-B1` operate as VTP Servers; all Access switches configured as `vtp mode client`.
* **Access & Voice Port Provisioning**:
  * Access ports assigned to workstation VLANs (`10`), server VLANs (`30`), and LWAP management VLANs (`99`).
  * IP Telephony access ports configured with data VLAN 10 and voice VLAN 20 (`switchport voice vlan 20`).
  * WLC1 trunk link configured on `ASW-A1` (`F0/2`) allowing VLANs `40,99` with Native VLAN 99.
  * Unused interfaces administratively disabled (`shutdown`).

### Part 3: Layer 3 Routing & HSRPv2 High Availability
* **Core Interconnects**: Enabled IP Routing (`ip routing`) on all Core and Distribution switches.
* **Layer 3 EtherChannel**: Configured non-switching routed ports on `CSW1` and `CSW2` (`no switchport`) bound to Layer 3 `Port-Channel 1` with IP subnet `10.0.0.40/30`.
* **First Hop Redundancy Protocol (HSRPv2)**:
  * Deployed HSRP Version 2 across all subnets for default gateway redundancy.
  * **Active Load Sharing**: `DSW-A1`/`DSW-B1` active for Groups 1 & 2 (Mgmt & PCs); `DSW-A2`/`DSW-B2` active for Groups 3 & 4 (Phones & Servers/Wi-Fi).
  * Priority set to `105` on active gateways with preemption enabled (`standby <group> preempt`).

### Part 4: Spanning Tree Protocol (RSTP) & STP Security
* **Rapid PVST+ Migration**: Transitioned all switches from standard PVST+ to Rapid PVST+ (`spanning-tree mode rapid-pvst`).
* **Root Bridge Optimization**:
  * Primary STP Root configured with `priority 0` to align with the HSRP Active gateway for each VLAN.
  * Secondary STP Root configured with `priority 4096` to align with the HSRP Standby gateway.
* **Edge Port Protection**:
  * Configured `spanning-tree portfast` and `spanning-tree bpduguard enable` on host-facing interfaces.
  * Configured `spanning-tree portfast trunk` on `ASW-A1` interface `F0/2` connecting to WLC1.

### Part 5: OSPFv2 Routing & WAN Edge Redundancy
* **OSPFv2 Single-Area 0**: Configured OSPF Process ID `1` in Area `0` across `R1`, `CSW1/2`, and `DSW-A1/A2/B1/B2`.
* **Router ID Standardization**: Manually assigned OSPF Router IDs to match each device's Loopback 0 IPv4 address (`router-id 10.0.0.X`).
* **Adjacency Optimization**:
  * Activated `ip ospf network point-to-point` on physical point-to-point Layer 3 links.
  * Designated Loopback 0 and host-facing SVIs (VLAN 10, 20, 30, 40) as `passive-interface` to suppress unnecessary OSPF hello packets.
* **Floating Static Default Routes**:
  * Primary Internet default route on `R1`: `ip route 0.0.0.0 0.0.0.0 203.0.113.1` (via ISP A).
  * Backup floating static route on `R1`: `ip route 0.0.0.0 0.0.0.0 203.0.113.5 2` (AD = 2 via ISP B).
  * Injected default route into OSPF domain using `default-information originate`.

### Part 6: Core Infrastructure Services
* **Router-Based DHCP Server**: Configured 7 dedicated DHCP pools on `R1` (`A-Mgmt`, `A-PC`, `A-Phone`, `B-Mgmt`, `B-PC`, `B-Phone`, `Wi-Fi`) with excluded host ranges (`10.0.0.1` - `10.0.0.10`), domain name `jeremysitlab.com`, and DNS server `10.5.0.4`.
* **WLC LWAP Discovery**: Embedded DHCP Option 43 (`option 43 ip 10.0.0.7`) in management pools for LWAP controller discovery.
* **DHCP Relay**: Configured `ip helper-address 10.0.0.76` on all distribution SVI interfaces to forward client broadcast requests to `R1`.
* **DNS Services (`SRV1`)**: Configured DNS A-Records for `jeremysitlab.com`, `google.com`, `youtube.com`, and CNAME record for `www.jeremysitlab.com`.
* **NTP Time Synchronization**: `R1` configured as Stratum 5 master (`ntp master 5`); all switches configured as NTP clients authenticated via MD5 key 1 (`ntp authentication-key 1 md5 ccna`, `ntp trusted-key 1`).
* **Management & Logging**:
  * Configured SNMP Read-Only community string `SNMPSTRING` (`snmp-server community SNMPSTRING RO`).
  * Enabled Syslog logging to `SRV1` at debugging level (`logging 10.5.0.4`, `logging trap debugging`) with an 8192-byte logging buffer (`logging buffered 8192`).
* **FTP IOS Maintenance**: Downloaded updated Cisco IOS image (`c2900-universalk9-mz.SPA.155-3.M.bin`) from `SRV1` via FTP, updated boot path (`boot system flash:`), verified reboot, and deleted legacy image.
* **Secure Management (SSHv2)**: Generated 4096-bit RSA keys (`crypto key generate rsa`), enforced `ip ssh version 2`, configured VTY line restriction (`transport input ssh`), and applied Access Class ACL 1 permitting only Office A PC subnet (`10.1.0.0/24`).

### Part 7: Security Hardening & First-Hop Defense
* **Extended Access Control Lists**:
  * Configured extended ACL `OfficeA_to_OfficeB` on `DSW-A1` & `DSW-A2` SVI VLAN 10 (`inbound`) close to the traffic source.
  * Permits ICMP traffic from Office A PCs (`10.1.0.0/24`) to Office B PCs (`10.3.0.0/24`), blocks all other IP traffic between the two PC subnets, and permits all remaining enterprise traffic.
* **Port Security**:
  * Enabled on access port `F0/1` across all access switches.
  * Set `switchport port-security violation restrict` and sticky MAC learning (`switchport port-security mac-address sticky`).
  * Adjusted maximum MAC limits: `maximum 1` for single-host/LWAP/Server ports; `maximum 2` for dual Phone/PC ports.
* **DHCP Snooping**:
  * Globally enabled and enforced per active VLAN (`ip dhcp snooping vlan 10,20,40,99` / `10,20,30,99`).
  * Disabled Option 82 insertion (`no ip dhcp snooping information option`) to prevent packet drop by Cisco IOS DHCP server.
  * Trusted uplink ports `G0/1` and `G0/2` (`ip dhcp snooping trust`).
  * Enforced rate limits on untrusted access ports (`ip dhcp snooping limit rate 15` pps; `100` pps on WLC link).
* **Dynamic ARP Inspection (DAI)**:
  * Enforced per-VLAN DAI (`ip arp inspection vlan ...`) with explicit trust on distribution uplinks `G0/1-2`.
  * Activated validation checks for source MAC, destination MAC, and IP header alignment (`ip arp inspection validate dst-mac src-mac ip`).

### Part 8: Dual-Stack IPv6 Routing
* **IPv6 Global Routing**: Enabled `ipv6 unicast-routing` on `R1`, `CSW1`, and `CSW2`.
* **Addressing Standards**:
  * Explicit IPv6 addresses on WAN interfaces: `2001:db8:a::2/64` and `2001:db8:b::2/64`.
  * EUI-64 stateless autoconfiguration on core LAN links: `2001:db8:a1::/64 eui-64` and `2001:db8:a2::/64 eui-64`.
  * Link-Local IPv6 activation on `CSW1`/`CSW2` Layer 3 PortChannel 1 using `ipv6 enable`.
* **Static IPv6 Default Routing**:
  * Primary recursive route: `ipv6 route ::/0 2001:db8:a::1`.
  * Backup fully-specified floating route: `ipv6 route ::/0 g0/1/0 2001:db8:b::1 2` (AD = 2).

### Part 9: Cisco Wireless LAN Controller (WLC) & LWAP Deployment
* **WLC Management**: Accessed Cisco 2504/3504 WLC GUI via HTTPS.
* **Dynamic Interface Provisioning**: Created dynamic interface `Wi-Fi` mapped to VLAN `40` on physical port 1 (`IP: 10.6.0.2/24`, `Gateway: 10.6.0.1`, `DHCP Server: 10.0.0.76`).
* **WLAN Profile Creation**: Configured WLAN ID `1` with Profile Name and SSID `Wi-Fi`, assigned to dynamic interface `Wi-Fi`.
* **Wireless Security**: Enforced WPA2-Enterprise/PSK security policy with AES encryption and Pre-Shared Key `cisco123`.
* **CAPWAP LWAP Association**: Verified CAPWAP discovery and tunnel establishment for lightweight access points `LWAP1` and `LWAP2`.

---

## 4. Master Configuration Snippets

### Edge Router R1 Configuration
```ios
! Hostname & Management Security
hostname R1
enable secret jeremysitlab
username cisco secret ccna
line con 0
 login local
 exec-timeout 30 0
 logging synchronous
exit

! OSPFv2 Single-Area 0 & Passive Interfaces
router ospf 1
 router-id 10.0.0.76
 passive-interface Loopback0
 default-information originate
exit

interface Loopback0
 ip address 10.0.0.76 255.255.255.255
 ip ospf 1 area 0
exit

interface GigabitEthernet0/0
 ip address 10.0.0.33 255.255.255.252
 ip ospf 1 area 0
 ip ospf network point-to-point
 ip nat inside
 no shutdown
exit

interface GigabitEthernet0/1
 ip address 10.0.0.37 255.255.255.252
 ip ospf 1 area 0
 ip ospf network point-to-point
 ip nat inside
 no shutdown
exit

! WAN Connections & Floating Static Routes
interface GigabitEthernet0/0/0
 ip address dhcp
 ip nat outside
 no shutdown
exit

interface GigabitEthernet0/1/0
 ip address dhcp
 ip nat outside
 no shutdown
exit

ip route 0.0.0.0 0.0.0.0 203.0.113.1
ip route 0.0.0.0 0.0.0.0 203.0.113.5 2

! NAT / PAT Configuration
ip nat inside source static 10.5.0.4 203.0.113.113
ip nat pool POOL1 203.0.113.200 203.0.113.207 netmask 255.255.255.248
ip nat inside source list 2 pool POOL1 overload

access-list 2 permit 10.1.0.0 0.0.0.255
access-list 2 permit 10.2.0.0 0.0.0.255
access-list 2 permit 10.3.0.0 0.0.0.255
access-list 2 permit 10.4.0.0 0.0.0.255
access-list 2 permit 10.6.0.0 0.0.0.255

! DHCP Server & Option 43
ip dhcp excluded-address 10.0.0.1 10.0.0.10
ip dhcp excluded-address 10.1.0.1 10.1.0.10
ip dhcp excluded-address 10.2.0.1 10.2.0.10
ip dhcp excluded-address 10.0.0.17 10.0.0.26
ip dhcp excluded-address 10.3.0.1 10.3.0.10
ip dhcp excluded-address 10.4.0.1 10.4.0.10
ip dhcp excluded-address 10.6.0.1 10.6.0.10

ip dhcp pool A-Mgmt
 network 10.0.0.0 255.255.255.240
 default-router 10.0.0.1
 dns-server 10.5.0.4
 domain-name jeremysitlab.com
 option 43 ip 10.0.0.7
exit
```

### Distribution Switch DSW-A1 (HSRP & Security)
```ios
hostname DSW-A1
ip routing
vtp domain JeremysITLab
vtp mode server

! Rapid Spanning Tree & Root Bridge Alignment
spanning-tree mode rapid-pvst
spanning-tree vlan 10,99 priority 0
spanning-tree vlan 20,40 priority 4096

! EtherChannel & Trunking
interface range GigabitEthernet1/0/4-5
 channel-group 1 mode desirable
exit

interface range GigabitEthernet1/0/1-3, Port-Channel1
 switchport mode trunk
 switchport nonegotiate
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 10,20,40,99
exit

! HSRPv2 High Availability Gateways
interface Vlan10
 ip address 10.1.0.2 255.255.255.0
 standby version 2
 standby 2 ip 10.1.0.1
 standby 2 priority 105
 standby 2 preempt
 ip helper-address 10.0.0.76
 ip access-group OfficeA_to_OfficeB in
exit

interface Vlan99
 ip address 10.0.0.2 255.255.255.240
 standby version 2
 standby 1 ip 10.0.0.1
 standby 1 priority 105
 standby 1 preempt
 ip helper-address 10.0.0.76
exit

! Inter-Office Access Control List
ip access-list extended OfficeA_to_OfficeB
 permit icmp 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
 deny ip 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
 permit ip any any
exit
```

### Access Switch ASW-A2 (First-Hop Defense)
```ios
hostname ASW-A2
vtp mode client
spanning-tree mode rapid-pvst

! Trunking to Distribution
interface range GigabitEthernet0/1-2
 switchport mode trunk
 switchport nonegotiate
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 10,20,40,99
 ip dhcp snooping trust
 ip arp inspection trust
exit

! Host Port Security & IP Telephony
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 20
 switchport nonegotiate
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip dhcp snooping limit rate 15
exit

! First-Hop Security Features
ip dhcp snooping
ip dhcp snooping vlan 10,20,40,99
no ip dhcp snooping information option

ip arp inspection vlan 10,20,40,99
ip arp inspection validate dst-mac src-mac ip
```

---

## 5. Verification & Operational Testing Guide

Execute these standard Cisco IOS verification commands to validate operational health across all protocol layers:

| Layer / Feature | Recommended Verification Command | Expected Successful Output Summary |
| :--- | :--- | :--- |
| **Trunking & DTP** | `show interface trunk` | Interfaces in `802.1q` trunking state with Native VLAN 1000 and matching allowed VLAN lists. |
| **EtherChannel** | `show etherchannel summary` | Flags `SU` (Layer 2) or `RU` (Layer 3) on PortChannels, and flag `P` on member physical links. |
| **HSRP Redundancy**| `show standby brief` | Correct Active (`Prio 105`) and Standby (`Prio 100`) roles matched to VIP addresses across VLANs. |
| **Spanning Tree** | `show spanning-tree summary` | Mode `rstp`, with `This bridge is the root` indicated on designated VLAN root switches. |
| **OSPFv2 Routing** | `show ip ospf neighbor` | Adjacencies in `FULL/ -` state over point-to-point links; no stuck `EXSTART`/`EXCHANGE` states. |
| **DHCP Services** | `show ip dhcp binding` | Active leased IPv4 bindings mapped to client MAC addresses across Office A, B, and Wi-Fi pools. |
| **NAT Translations**| `show ip nat translations` | Dynamic PAT entries mapping inside private IPs (`10.X.X.X`) to public pool (`203.0.113.200`..`.207`). |
| **Port Security** | `show port-security interface f0/1` | Status `Secure-up`, Violation Mode `Restrict`, correct sticky MAC address count. |
| **DHCP Snooping** | `show ip dhcp snooping` | Operational on configured VLANs, Option 82 disabled, uplinks trusted. |
| **DAI Status** | `show ip arp inspection vlan <id>`| Dynamic ARP inspection active with source/destination/IP validation enabled. |
| **IPv6 Status** | `show ipv6 interface brief` | Active Link-Local (`FE80::`) and Global Unicast (`2001:DB8::`) IPv6 addresses. |

---

## 6. How to Load and Validate in Packet Tracer

1. Download the Packet Tracer activity file (`.pka`) from the repository or repository releases.
2. Open the file in **Cisco Packet Tracer v8.x** or newer.
3. Apply device configurations sequentially through the CLI or via automated text script paste.
4. **Crucial Scoring Requirement**: To ensure Packet Tracer's automated grading engine awards full points, you **must write running configuration to startup memory** on every device:
   ```ios
   copy running-config startup-config
   ```
   *(or simply `write memory` / `do write`)*.
5. Click **Check Results** in the Packet Tracer Activity window to confirm **100% Completion Score**.

---

## 📜 Author & Acknowledgments
* **Instructor & Original Lab Design**: Jeremy's IT Lab (Jeremy McDowell)
* **Documentation & Implementation**: Engineered for production-level GitHub portfolio presentation.
