# Routing Redundancy Configuration Report

## Assignment: Module 9



# Objective
The objective of this assignment is to implement and verify routing redundancy in an enterprise network environment using:

1. HSRP (Hot Standby Router Protocol) for Core Router Redundancy
2. Floating Static Routes for Branch Router Redundancy
3. OSPF Cost Manipulation for Dynamic Route Redundancy
4. End-to-End Reachability Testing
5. Failover Verification after Core Router Shutdown

---

# Network Topology Overview
The topology contains:

- Two Core Routers
- Multiple VLANs in the Head Office
- CTG Branch Router
- RAJ Branch Router
- OSPF Dynamic Routing
- Redundant WAN Links
- HSRP Gateway Redundancy
- Floating Static Backup Routes

### VLAN Information

| VLAN | Network |
|---|---|
| VLAN 10 | 10.0.0.0/8 |
| VLAN 20 | 20.0.0.0/8 |
| VLAN 30 | 30.0.0.0/8 |
| VLAN 40 | 40.0.0.0/8 |
| VLAN 50 | 50.0.0.0/8 |
| VLAN 60 | 60.0.0.0/8 |
| CTG Branch | 100.0.0.0/8 |
| RAJ Branch | 200.0.0.0/8 |

---

# Part 1: Configure HSRP for Core Router Redundancy

## Purpose
HSRP provides gateway redundancy by creating:

- One Active Router
- One Standby Router
- One Virtual Gateway IP

If the active router fails, the standby router automatically becomes active.

---

# Actual HSRP Configuration Used in the Topology

## Core_Router_01

```bash
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.0.0.2 255.0.0.0
 standby 10 ip 10.0.0.1
 standby 10 priority 150
 standby 10 preempt
!
interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 20.0.0.2 255.0.0.0
 standby 20 ip 20.0.0.1
 standby 20 priority 150
 standby 20 preempt
 standby preempt
!
interface FastEthernet0/0.30
 encapsulation dot1Q 30
 ip address 30.0.0.2 255.0.0.0
 ip helper-address 10.0.0.11
 standby 30 ip 30.0.0.1
 standby 30 priority 150
 standby 30 preempt
 standby preempt
!
interface FastEthernet1/0.40
 encapsulation dot1Q 40
 ip address 40.0.0.2 255.0.0.0
 standby 40 ip 40.0.0.1
 standby 40 priority 120
 standby 40 preempt
 standby preempt
!
interface FastEthernet1/0.50
 encapsulation dot1Q 50
 ip address 50.0.0.2 255.0.0.0
 standby 50 ip 50.0.0.1
 standby 50 priority 120
 standby 50 preempt
 standby preempt
!
interface FastEthernet1/0.60
 encapsulation dot1Q 60
 ip address 60.0.0.2 255.0.0.0
 standby 60 ip 60.0.0.1
 standby 60 priority 120
 standby 60 preempt
 standby preempt
```

---

## Core_Router_02

```bash
interface FastEthernet0/0.40
 encapsulation dot1Q 40
 ip address 40.0.0.3 255.0.0.0
 standby 40 ip 40.0.0.1
 standby 40 priority 150
 standby 40 preempt
 standby preempt
!
interface FastEthernet0/0.50
 encapsulation dot1Q 50
 ip address 50.0.0.3 255.0.0.0
 standby 50 ip 50.0.0.1
 standby 50 priority 150
 standby 50 preempt
 standby preempt
!
interface FastEthernet0/0.60
 encapsulation dot1Q 60
 ip address 60.0.0.3 255.0.0.0
 standby 60 ip 60.0.0.1
 standby 60 priority 150
 standby 60 preempt
 standby preempt
!
interface FastEthernet1/0.10
 encapsulation dot1Q 10
 ip address 10.0.0.3 255.0.0.0
 standby 10 ip 10.0.0.1
 standby 10 priority 120
 standby 10 preempt
!
interface FastEthernet1/0.20
 encapsulation dot1Q 20
 ip address 20.0.0.3 255.0.0.0
 standby 20 ip 20.0.0.1
 standby 20 priority 120
 standby 20 preempt
 standby preempt
!
interface FastEthernet1/0.30
 encapsulation dot1Q 30
 ip address 30.0.0.3 255.0.0.0
 standby 30 ip 30.0.0.1
 standby 30 priority 120
 standby 30 preempt
 standby preempt
```

---

## HSRP Priority Distribution

| VLAN | Active Router | Standby Router |
|---|---|---|
| VLAN 10 | Core_Router_01 | Core_Router_02 |
| VLAN 20 | Core_Router_01 | Core_Router_02 |
| VLAN 30 | Core_Router_01 | Core_Router_02 |
| VLAN 40 | Core_Router_02 | Core_Router_01 |
| VLAN 50 | Core_Router_02 | Core_Router_01 |
| VLAN 60 | Core_Router_02 | Core_Router_01 |

This configuration provides load balancing along with gateway redundancy.

---

## Verification

```bash
Router# show standby brief
```

### Expected Result

- Router 1 should be Active
- Router 2 should be Standby
- Virtual IP should be reachable

---

# Part 2: Configure Floating Static Route for Branch Router Redundancy

## Purpose
Floating Static Routes are backup routes that become active only if the primary route fails.

Administrative Distance (AD) of backup route must be higher than the primary route.

---

## Configuration Example

### Primary Static Route

```bash
Router(config)# ip route 200.0.0.0 255.0.0.0 35.0.0.2
```

### Floating Backup Route

```bash
Router(config)# ip route 200.0.0.0 255.0.0.0 37.0.0.2 150
```

---

## Explanation

| Route Type | Administrative Distance |
|---|---|
| Primary Route | 1 |
| Floating Route | 150 |

The backup route remains inactive until the primary route becomes unavailable.

---

## Verification

```bash
Router# show ip route
```

### Expected Result

- Primary route should appear in routing table
- Backup route should activate after primary failure

---

# Part 3: Configure Different Costs for OSPF Route Redundancy

## Purpose
OSPF selects paths based on cost.

Lower cost path becomes preferred route.
Higher cost path remains backup.

---

## Configuration Example

### Preferred Link

```bash
Router(config)# interface serial 2/0
Router(config-if)# ip ospf cost 10
```

### Backup Link

```bash
Router(config)# interface serial 3/0
Router(config-if)# ip ospf cost 100
```

---

## Explanation

| OSPF Cost | Result |
|---|---|
| Lower Cost | Preferred Path |
| Higher Cost | Backup Path |

---

## Verification

```bash
Router# show ip ospf interface
Router# show ip route ospf
```

### Expected Result

- OSPF should use lower cost path
- Higher cost route should become active if primary path fails

---

# Part 4: Verify Reachability from Head Office to Branch

## Testing Method
Use ping and traceroute commands.

### Ping Test

```bash
PC> ping 200.0.0.1
```

### Traceroute Test

```bash
PC> tracert 200.0.0.1
```

---

## Expected Result

- Successful ping replies
- Stable connectivity between Head Office and Branch
- Traffic should follow preferred routes

---

# Part 5: Verify Reachability after Turning Off One Core Router

## Procedure

1. Shut down one core router
2. Observe HSRP failover
3. Verify OSPF recalculation
4. Verify floating route activation
5. Test connectivity again

---

## Example Shutdown Command

```bash
Router(config)# interface fa0/0
Router(config-if)# shutdown
```

---

## Verification Commands

### HSRP Verification

```bash
Router# show standby brief
```

### OSPF Verification

```bash
Router# show ip route
```

### Connectivity Verification

```bash
PC> ping 200.0.0.1
```

---

# Observations

| Test Scenario | Result |
|---|---|
| Normal operation | Successful |
| Core Router failure | Successful failover |
| OSPF backup path | Activated |
| Floating route | Activated |
| Branch connectivity | Maintained |

---

# Advantages of Routing Redundancy

1. High Availability
2. Automatic Failover
3. Reduced Downtime
4. Improved Network Reliability
5. Better Traffic Management
6. Fault Tolerance

---

# Conclusion

In this assignment, routing redundancy was successfully implemented using:

- HSRP for gateway redundancy
- Floating Static Routes for backup routing
- OSPF cost tuning for path redundancy

The network maintained connectivity even after failure of a core router, proving that redundancy mechanisms improve availability and reliability in enterprise networks.

---

# Commands Summary

## HSRP Commands

```bash
standby 10 ip 10.0.0.254
standby 10 priority 120
standby 10 preempt
show standby brief
```

## Floating Route Commands

```bash
ip route 200.0.0.0 255.0.0.0 35.0.0.2
ip route 200.0.0.0 255.0.0.0 37.0.0.2 150
show ip route
```

## OSPF Commands

```bash
router ospf 1
network 0.0.0.0 255.255.255.255 area 0
ip ospf cost 10
ip ospf cost 100
show ip ospf interface
```

---

# End of Report

