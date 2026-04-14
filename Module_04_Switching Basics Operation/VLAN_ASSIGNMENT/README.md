# VLAN Configuration and Verification – Module 4 Assignment

**Course:** Network Administration  
**Reference:** Class Note 8  
**Topic:** VLAN Configuration and Verification in a Multi-Switched Environment

---

## 📋 Overview

This assignment covers the complete configuration and verification of Virtual Local Area Networks (VLANs) across a multi-switched network topology consisting of one **Access Switch** and two **Core Switches**.

---

## 🏗️ Network Topology

```
[ End Devices ]
      |
 [ Access Switch (AS1) ]   ←── VLANs: 10, 20, 30 (Port Assignments)
      | (Trunk: 802.1Q)
 [ Core Switch 1 (CS1) ]   ←── VLANs: 10, 20, 30 (SVIs + Routing)
      | (Trunk: 802.1Q)
 [ Core Switch 2 (CS2) ]   ←── VLANs: 40, 50, 60 (SVIs + Routing)
```

---

## 📁 Files in This Submission

| File | Description |
|------|-------------|
| `VLAN_Report_Module4.docx` | Full lab report with configurations, tables, and verification |
| `README.md` | This file – project overview and quick reference |

---

## 🔧 VLAN Plan

### Core Switch 1 (CS1) — VLANs 10, 20, 30

| VLAN ID | Name             | Subnet            | SVI IP             |
|---------|------------------|-------------------|--------------------|
| 10      | VLAN_Management  | 192.168.10.0/24   | 192.168.10.254/24  |
| 20      | VLAN_Sales       | 192.168.20.0/24   | 192.168.20.254/24  |
| 30      | VLAN_Engineering | 192.168.30.0/24   | 192.168.30.254/24  |

### Core Switch 2 (CS2) — VLANs 40, 50, 60

| VLAN ID | Name          | Subnet            | SVI IP             |
|---------|---------------|-------------------|--------------------|
| 40      | VLAN_HR       | 192.168.40.0/24   | 192.168.40.254/24  |
| 50      | VLAN_Finance  | 192.168.50.0/24   | 192.168.50.254/24  |
| 60      | VLAN_Guest    | 192.168.60.0/24   | 192.168.60.254/24  |

---

## ✅ Activity Checklist

- [x] Configure VLANs in Access Switch according to tables
- [x] Assign Ports to VLANs according to tables
- [x] Verify VLAN and check intra-VLAN connectivity
- [x] Configure VLAN 10, 20, 30 in Core Switch 1
- [x] Ensure VLANs working in Multi-Switched Environment (CS1)
- [x] Configure VLAN 40, 50, 60 in Core Switch 2
- [x] Ensure VLANs working in Multi-Switched Environment (CS2)
- [x] README.md included

---

## 🖥️ Key Commands Reference

### Create a VLAN
```
Switch(config)# vlan <id>
Switch(config-vlan)# name <VLAN_NAME>
Switch(config-vlan)# exit
```

### Assign Port to VLAN (Access Mode)
```
Switch(config)# interface <interface>
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan <id>
```

### Configure Trunk Port
```
Switch(config)# interface <interface>
Switch(config-if)# switchport trunk encapsulation dot1q
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan <id-list>
```

### Configure SVI (Layer 3)
```
Switch(config)# interface vlan <id>
Switch(config-if)# ip address <ip> <mask>
Switch(config-if)# no shutdown
```

### Verification Commands
```
Switch# show vlan brief
Switch# show interfaces trunk
Switch# show ip interface brief
Switch# show ip route
Switch# show vtp status
Switch# show spanning-tree vlan <id>
```

---

## 📌 Key Concepts

- **VLAN**: Logical network segmentation at Layer 2
- **Trunk Port**: Carries multiple VLANs between switches using 802.1Q tagging
- **Access Port**: Carries traffic for a single VLAN to end devices
- **SVI (Switched Virtual Interface)**: Layer 3 interface for inter-VLAN routing
- **VTP**: VLAN Trunking Protocol — propagates VLAN info across switches
- **802.1Q**: IEEE standard for VLAN tagging on Ethernet frames

---

## 📝 Notes

- Intra-VLAN communication (same VLAN) works at Layer 2 without routing
- Inter-VLAN communication requires Layer 3 routing via SVIs or a router-on-a-stick
- Cross-VLAN isolation (e.g., Guest VLAN 60 cannot reach Management VLAN 10) is enforced via ACLs on SVIs
- Always save configuration: `Switch# write memory`

---

*Submitted as part of Module 4 – Network Administration Lab | Reference: Class Note 8*
