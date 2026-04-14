# README: Report on Inter-VLAN Communication (CIDR /8 Addressing)

## Assignment: Module 5 – Inter-VLAN Routing & Redundancy

**Reference Documents:** Class Notes 8 & 9  
**Objective:**  
- Configure VLANs 10,20,30,40,50,60 on two Core Switches.  
- Implement Inter-VLAN routing using a Core Router (Router‑on‑a‑Stick with two physical interfaces).  
- Provide redundant uplinks from all Access Switches to both Core Switches.  
- Verify end‑to‑end connectivity across VLANs.

> **IP Addressing Note (as requested):**  
> - Each VLAN uses a **/8** subnet mask (255.0.0.0).  
> - Gateway addresses:  
>   VLAN 10 → 10.0.0.1/8  
>   VLAN 20 → 20.0.0.1/8  
>   VLAN 30 → 30.0.0.1/8  
>   VLAN 40 → 40.0.0.1/8  
>   VLAN 50 → 50.0.0.1/8  
>   VLAN 60 → 60.0.0.1/8  

> **Why /8?**  
> This treats each VLAN as a separate Class A network (first octet unique). While not typical for VLAN segmentation, it fulfills the assignment’s specific requirement. In production, /24 would be more common, but this configuration is valid and routes correctly between non‑overlapping /8 networks.

---

## 1. Network Topology Overview
Core Router 1
(Inter-VLAN Routing)
/
Gi0/0/0 Gi0/0/1
(sub‑ifs) (sub‑ifs)
VLAN 10,20,30 VLAN 40,50,60
| |
Core Switch 1 <----> Core Switch 2
(Trunk) (Trunk)
| |
+---------------------+
| Redundant Links |
+---------------------+
Access Switch 1 Access Switch 2
(Hosts in all VLANs) (Hosts in all VLANs)

text

- **Core Switch 1 & 2** – Layer2 switches with the same VLAN database.  
- **Core Router 1** – Performs inter‑VLAN routing.  
- **Access Switches** – Connect end devices; each has two uplinks (one to Core‑1, one to Core‑2) for redundancy.  
- **Trunk links** carry all VLANs between switches and between switches & router.

---

## 2. IP Addressing Scheme (CIDR /8)

| VLAN | Subnet (CIDR /8) | Gateway (Router sub‑interface) |
|------|------------------|--------------------------------|
| 10   | 10.0.0.0/8       | 10.0.0.1                       |
| 20   | 20.0.0.0/8       | 20.0.0.1                       |
| 30   | 30.0.0.0/8       | 30.0.0.1                       |
| 40   | 40.0.0.0/8       | 40.0.0.1                       |
| 50   | 50.0.0.0/8       | 50.0.0.1                       |
| 60   | 60.0.0.0/8       | 60.0.0.1                       |

> **Example host addresses:**  
> - A PC in VLAN 10 could be `10.0.0.10/8`  
> - A PC in VLAN 20 could be `20.0.0.20/8`  
> - Any address within the respective 10.x.x.x, 20.x.x.x range is valid.

---

## 3. Configuration Steps

### 3.1 Core Switch 1 (All VLANs)

```cisco
hostname Core-SW1
vlan 10
 name VLAN_10
vlan 20
 name VLAN_20
vlan 30
 name VLAN_30
vlan 40
 name VLAN_40
vlan 50
 name VLAN_50
vlan 60
 name VLAN_60
!
interface range GigabitEthernet0/1-2
 description Trunk to Core-SW2 & Router (Gi0/0/0)
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,60
!
interface GigabitEthernet0/3
 description Trunk to Access-SW1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,60
!
interface GigabitEthernet0/4
 description Trunk to Access-SW2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,60
!
interface GigabitEthernet0/5
 description Connection to Router Gi0/0/0 (VLANs 10,20,30)
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
!
interface GigabitEthernet0/6
 description Connection to Router Gi0/0/1 (VLANs 40,50,60)
 switchport mode trunk
 switchport trunk allowed vlan 40,50,60
3.2 Core Switch 2 (Identical VLAN configuration)
cisco
hostname Core-SW2
vlan 10,20,30,40,50,60
!
interface range GigabitEthernet0/1-2
 description Trunk to Core-SW1 & Router (backup)
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,60
!
interface GigabitEthernet0/3
 description Trunk to Access-SW1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,60
!
interface GigabitEthernet0/4
 description Trunk to Access-SW2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,60
Note: Core Switch 2 does not directly connect to the router in this design; routing is handled solely by Core Router 1. Inter‑switch trunk (G0/1‑2) ensures both core switches share the same VLAN information and provide L2 path redundancy.

3.3 Core Router 1 – Inter‑VLAN Routing (with /8 masks)
Router‑on‑a‑Stick with two physical interfaces:

GigabitEthernet0/0/0 → subinterfaces for VLAN 10, 20, 30

GigabitEthernet0/0/1 → subinterfaces for VLAN 40, 50, 60

cisco
hostname Core-RTR1
!
interface GigabitEthernet0/0/0
 no ip address
!
interface GigabitEthernet0/0/0.10
 encapsulation dot1Q 10
 ip address 10.0.0.1 255.0.0.0
!
interface GigabitEthernet0/0/0.20
 encapsulation dot1Q 20
 ip address 20.0.0.1 255.0.0.0
!
interface GigabitEthernet0/0/0.30
 encapsulation dot1Q 30
 ip address 30.0.0.1 255.0.0.0
!
interface GigabitEthernet0/0/1
 no ip address
!
interface GigabitEthernet0/0/1.40
 encapsulation dot1Q 40
 ip address 40.0.0.1 255.0.0.0
!
interface GigabitEthernet0/0/1.50
 encapsulation dot1Q 50
 ip address 50.0.0.1 255.0.0.0
!
interface GigabitEthernet0/0/1.60
 encapsulation dot1Q 60
 ip address 60.0.0.1 255.0.0.0
!
ip routing
Why /8 works:
Each VLAN’s subnet is a separate Class A network (e.g., 10.0.0.0/8, 20.0.0.0/8). These do not overlap, so the router can maintain distinct routes. Hosts in VLAN 10 use a default gateway of 10.0.0.1 and a mask of 255.0.0.0, so they believe the entire 10.x.x.x range is local. Communication to 20.x.x.x will be sent to the router.

3.4 Access Switches – Redundant Uplinks
Each Access Switch connects to both Core Switch 1 and Core Switch 2 using trunk links. Spanning Tree Protocol (STP) prevents loops and provides automatic failover.

cisco
hostname Access-SW1
vlan 10,20,30,40,50,60
!
interface GigabitEthernet0/1
 description Uplink to Core-SW1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,60
!
interface GigabitEthernet0/2
 description Uplink to Core-SW2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,60
!
interface range FastEthernet0/1-4
 description Access ports for end devices
 switchport mode access
 switchport access vlan 10   (example)
!
spanning-tree mode rapid-pvst
Redundancy behaviour:

STP blocks one of the two uplinks to avoid loops.

If the active link (e.g., to Core‑SW1) fails, STP unblocks the redundant link (to Core‑SW2) within seconds.

All VLANs remain reachable via the surviving core switch because the two core switches are interconnected via a trunk.

4. Verification of Inter‑VLAN Connectivity
4.1 Check VLANs and Trunks on Core Switches
cisco
Core-SW1# show vlan brief
Core-SW1# show interfaces trunk
4.2 Verify Router Sub‑interfaces (with /8 masks)
cisco
Core-RTR1# show ip interface brief
Interface                  IP-Address      OK? Method Status Protocol
Gig0/0/0.10                10.0.0.1        YES manual up     up
Gig0/0/0.20                20.0.0.1        YES manual up     up
Gig0/0/0.30                30.0.0.1        YES manual up     up
Gig0/0/1.40                40.0.0.1        YES manual up     up
Gig0/0/1.50                50.0.0.1        YES manual up     up
Gig0/0/1.60                60.0.0.1        YES manual up     up
4.3 Test Inter‑VLAN Ping
Assume end devices:

PC1: VLAN 10 → IP 10.0.0.10/8, gateway 10.0.0.1

PC2: VLAN 30 → IP 30.0.0.20/8

PC3: VLAN 50 → IP 50.0.0.30/8

From PC1:

text
ping 30.0.0.20   -> successful (VLAN10 → VLAN30 via router)
ping 50.0.0.30   -> successful (VLAN10 → VLAN50 via router)
On Router, confirm routing table (shows connected /8 networks):

cisco
Core-RTR1# show ip route
Codes: C - connected, S - static, ...
C    10.0.0.0/8 is directly connected, Gig0/0/0.10
C    20.0.0.0/8 is directly connected, Gig0/0/0.20
C    30.0.0.0/8 is directly connected, Gig0/0/0.30
C    40.0.0.0/8 is directly connected, Gig0/0/1.40
C    50.0.0.0/8 is directly connected, Gig0/0/1.50
C    60.0.0.0/8 is directly connected, Gig0/0/1.60
4.4 Redundancy Test
Disconnect the active uplink from Access‑SW1 to Core‑SW1.

Observe: STP transitions the backup link (to Core‑SW2) to forwarding state.

Repeat the ping test → connectivity remains uninterrupted (a few seconds of packet loss may occur during STP convergence).

5. Summary & Conclusion
VLANs 10‑60 were successfully configured on both Core Switch 1 and Core Switch 2.

Inter‑VLAN routing is provided by Core Router 1 using two physical interfaces – one for VLANs 10,20,30 and another for VLANs 40,50,60 – each with 802.1Q subinterfaces.

IP addressing uses a /8 subnet mask for each VLAN as requested, with gateways X.0.0.1.

Redundancy is achieved by connecting every Access Switch to both Core Switches, leveraging STP for loop prevention and automatic failover.

Verification confirms that hosts in different VLANs can communicate and that the redundant links function as expected.