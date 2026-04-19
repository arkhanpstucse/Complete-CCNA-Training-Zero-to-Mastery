# Branch Configuration and Routing Report (Final with Updated VLAN & WAN IPs)

## Updated VLAN Schema (Core Router)

The following VLANs are configured on the **Core Router** (`CORE_RTR`). Each VLAN uses a **192.168.x.0/24** subnet.

| VLAN Name | VLAN ID | Default Gateway | Subnet            |
|-----------|---------|----------------|-------------------|
| vlan_MZ   | 10      | 192.168.10.1   | 192.168.10.0/24   |
| vlan_IT   | 20      | 192.168.20.1   | 192.168.20.0/24   |
| vlan_MGMT | 30      | 192.168.30.1   | 192.168.30.0/24   |
| Vlan_HR   | 40      | 192.168.40.1   | 192.168.40.0/24   |
| vlan_FIN  | 50      | 192.168.50.1   | 192.168.50.0/24   |
| vlan_DMZ  | 60      | 192.168.60.1   | 192.168.60.0/24   |

**MZ Zone Server** is placed in `vlan_MZ` with IP `192.168.10.10/24` (gateway `192.168.10.1`).

---

## WAN Link Between CTG Branch and Core Router

| Device      | Interface | IP Address    | Mask         |
|-------------|-----------|---------------|--------------|
| CT_GRANCH   | Gig0/1    | 10.0.0.1      | 255.0.0.0    |
| CORE_RTR    | Gig0/0    | 10.0.0.2      | 255.0.0.0    |

> **Note:** The WAN link uses a `/8` subnet (`10.0.0.0/8`). Both routers share this large network.

---

## Branch Internal Network (CTG Branch)

| Device            | Interface         | IP Address / Mask        | Purpose                        |
|-------------------|-------------------|--------------------------|--------------------------------|
| CT_GRANCH         | VLAN 1 (internal) | 192.168.70.1 /24         | Branch default gateway         |
| Branch User (PC)  | NIC               | 192.168.70.100 /24       | Client in branch network       |

---

## Complete IP Addressing Plan

| Device            | Interface            | IP Address / Mask          | Purpose                        |
|-------------------|----------------------|----------------------------|--------------------------------|
| CT_GRANCH         | Vlan1                | 192.168.70.1 /24           | Branch gateway                 |
| CT_GRANCH         | Gig0/1               | 10.0.0.1 /8                | WAN link to Core Router        |
| CORE_RTR          | Gig0/0               | 10.0.0.2 /8                | WAN link to Branch             |
| CORE_RTR          | Vlan10 (vlan_MZ)     | 192.168.10.1 /24           | Gateway for MZ Zone            |
| CORE_RTR          | Vlan20 (vlan_IT)     | 192.168.20.1 /24           | Gateway for IT Zone            |
| CORE_RTR          | Vlan30 (vlan_MGMT)   | 192.168.30.1 /24           | Gateway for MGMT Zone          |
| CORE_RTR          | Vlan40 (Vlan_HR)     | 192.168.40.1 /24           | Gateway for HR Zone            |
| CORE_RTR          | Vlan50 (vlan_FIN)    | 192.168.50.1 /24           | Gateway for FIN Zone           |
| CORE_RTR          | Vlan60 (vlan_DMZ)    | 192.168.60.1 /24           | Gateway for DMZ Zone           |
| MZ_SRV            | Ethernet0            | 192.168.10.10 /24          | MZ Zone server                 |
| Branch User (PC)  | NIC                  | 192.168.70.100 /24         | Client in branch network       |

---

## Configuration Steps

### 1. Build CTG Branch
- Physical connections: Branch switch ports (`F0/0``F0/606`) connected to end devices.
- Basic hostname and console settings on `CT_GRANCH`.

### 2. Connect Internal Branch Network
- Assign IP to internal interface:
  ```cisco
  interface Vlan1
   ip address 192.168.70.1 255.255.255.0
   no shutdown
* Configure branch user with static IP�192.168.70.100/24, gateway�192.168.70.1.
3. Verify Internal Reachability
* From branch user:�ping 192.168.70.1�? successful.
* From router:�show ip interface brief�confirms Vlan1 up/up.
4. Connect CTG Branch with Core Router
* Physical cable between�CT_GRANCH Gig0/1�and�CORE_RTR Gig0/0.
* Configure WAN IPs:
cisco
! On CT_GRANCH
interface GigabitEthernet0/1
 ip address 10.0.0.1 255.0.0.0
 no shutdown

! On CORE_RTR
interface GigabitEthernet0/0
 ip address 10.0.0.2 255.0.0.0
 no shutdown
5. Configure Core Router VLANs (SVIs)
* Create VLANs and assign IP gateways on�CORE_RTR:
cisco
vlan 10
 name vlan_MZ
vlan 20
 name vlan_IT
vlan 30
 name vlan_MGMT
vlan 40
 name Vlan_HR
vlan 50
 name vlan_FIN
vlan 60
 name vlan_DMZ

interface Vlan10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface Vlan20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
interface Vlan30
 ip address 192.168.30.1 255.255.255.0
 no shutdown
interface Vlan40
 ip address 192.168.40.1 255.255.255.0
 no shutdown
interface Vlan50
 ip address 192.168.50.1 255.255.255.0
 no shutdown
interface Vlan60
 ip address 192.168.60.1 255.255.255.0
 no shutdown
* Connect MZ server to an access port in VLAN 10, assign IP�192.168.10.10/24�with gateway�192.168.10.1.
6. Configure Static Routing
On CT_GRANCH (Branch Router)
* Add a default route pointing to Core Router:
cisco
ip route 0.0.0.0 0.0.0.0 10.0.0.2
* Optional:�Explicit routes for each VLAN subnet (not required if using default route, but shown for clarity):
cisco
ip route 192.168.10.0 255.255.255.0 10.0.0.2
ip route 192.168.20.0 255.255.255.0 10.0.0.2
ip route 192.168.30.0 255.255.255.0 10.0.0.2
ip route 192.168.40.0 255.255.255.0 10.0.0.2
ip route 192.168.50.0 255.255.255.0 10.0.0.2
ip route 192.168.60.0 255.255.255.0 10.0.0.2
On CORE_RTR (Core Router)
* Add return route to branch subnet:
cisco
ip route 192.168.70.0 255.255.255.0 10.0.0.1
7. Verify Routing Tables
CT_GRANCH
cisco
show ip route
Expected output:
text
C    192.168.70.0/24 is directly connected, Vlan1
C    10.0.0.0/8 is directly connected, GigabitEthernet0/1
S*   0.0.0.0/0 [1/0] via 10.0.0.2
(Optional explicit routes appear as S for each 192.168.x.0/24)
CORE_RTR
cisco
show ip route
Expected output:
text
C    10.0.0.0/8 is directly connected, Gig0/0
C    192.168.10.0/24 is directly connected, Vlan10
C    192.168.20.0/24 is directly connected, Vlan20
C    192.168.30.0/24 is directly connected, Vlan30
C    192.168.40.0/24 is directly connected, Vlan40
C    192.168.50.0/24 is directly connected, Vlan50
C    192.168.60.0/24 is directly connected, Vlan60
S    192.168.70.0/24 [1/0] via 10.0.0.1
8. Reach MZ Zone Server from Branch User
* From branch user (192.168.70.100):
bash
ping 192.168.10.10
* Expected result: Successful replies (0% loss).
* Additional tests to other VLANs (e.g.,�ping 192.168.20.10�for IT server if present).

Verification Commands Summary
Command
Purpose
show ip interface brief�(both routers)
Check interface status and IPs
show ip route
Display routing table
ping 10.0.0.2�(from CT_GRANCH)
Test WAN link connectivity
ping 192.168.70.1�(from branch user)
Test internal gateway reachability
ping 192.168.10.10�(from branch user)
Test endtoend connectivity to MZ server
traceroute 192.168.10.10�(from branch user)
Show path via core router

Troubleshooting Notes
* If branch user cannot ping�192.168.10.10:
o Verify WAN interfaces are up (no shutdown).
o Check static routes:�show running-config | include ip route.
o Ensure core router VLAN 10 SVI is up (show ip interface brief).
o Confirm MZ server has correct IP and gateway (192.168.10.1).
o Check for firewall or ACL blocking ICMP.
* Because WAN uses�10.0.0.0/8, be careful not to overlap with any other�10.x.x.x�subnets in the network.

Conclusion
The CTG Branch (192.168.70.0/24) is successfully connected to the Core Router via a�10.0.0.0/8�WAN link. The Core Router hosts six VLANs with gateways in the�192.168.x.0/24�range. Static routing (default route from branch, specific return route on core) enables branch users to reach the MZ Zone server (192.168.10.10) and any other server in the core VLANs. All activity objectives are met.
Report prepared by:�Network Engineering Team
Date:�April 19, 2026

