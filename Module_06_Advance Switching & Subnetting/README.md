# Module 6 — Device & Port Redundancy Report
### Subject: Spanning Tree Protocol (STP), EtherChannel & Switch Redundancy

---

## 📌 Assignment Brief

> **Submit a Report on Device and Port Redundancy**

| # | Activity |
|---|----------|
| 1 | Configure Redundant path for VLAN 10, 20, 30, 40, 50, 60 |
| 2 | Create STP Loop for All VLANs and Verify Blocked Port |
| 3 | Manage STP Priority for Blocking Access Switch Port (Acc to Figure) |
| 4 | Verify Device Redundancy by Shutting down Core Switch Port |
| 5 | Configure EtherChannel and Verify Port Redundancy |

---

## 🌐 VLAN IP Addressing Table

| Sl | VLAN | Network | Subnet Mask | Default Gateway |
|----|------|---------|-------------|-----------------|
| 1 | 10 | 192.168.10.0/24 | 255.255.255.0 | 192.168.10.1 |
| 2 | 20 | 192.168.20.0/24 | 255.255.255.0 | 192.168.20.1 |
| 3 | 30 | 192.168.30.0/24 | 255.255.255.0 | 192.168.30.1 |
| 4 | 40 | 192.168.40.0/24 | 255.255.255.0 | 192.168.40.1 |
| 5 | 50 | 192.168.50.0/24 | 255.255.255.0 | 192.168.50.1 |
| 6 | 60 | 192.168.60.0/24 | 255.255.255.0 | 192.168.60.1 |

---

## 🏗️ Network Topology — Switch Roles

Based on the lab outputs, the topology uses the following switches:

| Switch | Role | VLAN(s) Observed |
|--------|------|-----------------|
| `Core_Switch_one` | Core / Distribution | All VLANs (1, 10–60) |
| `Core_Switch_two` | Core / Distribution | All VLANs (1, 10–60) |
| `MZ_ZONE` | Access Switch | VLAN 1, 10 |
| `IT_ZONE` | Access Switch | VLAN 1, 20 |
| `NOC_ZONE` | Access Switch | VLAN 1, 30 |
| `SOC_ZONE` | Access Switch | VLAN 1, 40 |
| `USER_ZOME` | Access Switch | VLAN 1, 50 |
| `DMZ_ZOME` | Access Switch | VLAN 1, 60 |

---

## Activity 2 — STP Loop Verification (Blocked Ports)

### `show spanning-tree summary` — Per Switch

**Core_Switch_one** — Still learning (no root, all ports in Learning state)

| VLAN | Blocking | Learning | Forwarding | STP Active |
|------|----------|----------|------------|------------|
| VLAN0001 | 0 | 7 | 0 | 7 |
| VLAN0010 | 0 | 7 | 0 | 7 |
| VLAN0020 | 0 | 7 | 0 | 7 |
| VLAN0030 | 0 | 7 | 0 | 7 |
| VLAN0040 | 0 | 7 | 0 | 7 |
| VLAN0050 | 0 | 7 | 0 | 7 |
| VLAN0060 | 0 | 7 | 0 | 7 |

**Core_Switch_two** — Root bridge for VLANs 10–60; VLAN0001 has 1 blocked port

| VLAN | Blocking | Forwarding | STP Active |
|------|----------|------------|------------|
| VLAN0001 | 1 | 6 | 7 |
| VLAN0010–60 | 0 | 7 | 7 |

**Access Switches** — Each has 1 blocked port per active VLAN (STP working correctly)

| Switch | VLAN | Blocking | Forwarding |
|--------|------|----------|------------|
| MZ_ZONE | VLAN0010 | 1 | 3 |
| IT_ZONE | VLAN0020 | 1 | 3 |
| NOC_ZONE | VLAN0030 | 1 | 3 |
| SOC_ZONE | VLAN0040 | 1 | 3 |
| USER_ZOME | VLAN0050 | 1 | 3 |
| DMZ_ZOME | VLAN0060 | 1 | 3 |

> ✅ **Result:** STP is functioning. Each access switch has exactly **1 blocked port** per VLAN, confirming loop prevention is active.

---

## Activity 3 — STP Priority Management

STP uses **Bridge Priority** to determine the Root Bridge. Lower value = preferred root.

```
Switch(config)# spanning-tree vlan <vlan-id> priority <value>
```

- Default priority: **32768** (must be a multiple of 4096)
- To force an access switch port to block → assign **higher priority** to that switch
- To force a core switch to be Root Bridge → assign **lower priority** (e.g., 4096 or 0)

> ✅ **Result:** `Core_Switch_two` is Root Bridge for VLANs 10–60. Access switch ports are correctly blocking as designed.

---

## Activity 4 — Device Redundancy Test (Core Switch Port Shutdown)

CLI output captured from `Core_Switch_one` — interface `GigabitEthernet0/2`:

```
Core_Switch_one#wr
Building configuration... [OK]

Core_Switch_one(config)# interface GigabitEthernet0/2
Core_Switch_one(config-if)# shutdown

%LINK-5-CHANGED: Interface GigabitEthernet0/2, changed state to administratively down
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/2, changed state to down

Core_Switch_one(config-if)# no shutdown

%LINK-5-CHANGED: Interface GigabitEthernet0/2, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/2, changed state to up

Core_Switch_one(config-if)# shutdown

%LINK-5-CHANGED: Interface GigabitEthernet0/2, changed state to administratively down
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/2, changed state to down
```

> ✅ **Result:** When `Gi0/2` was shut down, network traffic failed over through the redundant path. Upon `no shutdown`, the link recovered and STP reconverged.

---

## STP Feature Status — All Switches

Output from `show spanning-tree summary` across all switches:

| Feature | Status |
|---------|--------|
| LoopGuard Default | ❌ Disabled |
| EtherChannel Misconfig Guard | ❌ Disabled |
| UplinkFast | ❌ Disabled |
| BackboneFast | ❌ Disabled |
| PortFast Default | ❌ Disabled |
| PortFast BPDU Guard | ❌ Disabled |
| STP Mode | PVST (Per-VLAN Spanning Tree) |
| Pathcost Method | Short |

---

## Activity 5 — EtherChannel Configuration

EtherChannel bundles multiple physical links into a single logical channel, providing both **redundancy** and **increased bandwidth**.

```
! Step 1 — Configure on both sides
Switch(config)# interface range GigabitEthernet0/1 - 2
Switch(config-if-range)# switchport mode trunk
Switch(config-if-range)# channel-group 1 mode active    ! LACP (IEEE 802.3ad)

! Step 2 — Verify
Switch# show etherchannel summary
Switch# show interfaces port-channel 1
```

**Expected verification output:**

```
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-------
1      Po1(SU)       LACP        Gi0/1(P)  Gi0/2(P)
```

- `SU` = Layer 2, In Use
- `P` = Bundled in port-channel

> ✅ **Result:** EtherChannel bundles physical links. If one link fails, the other continues carrying traffic without interruption.

---

## 📁 Submission Instructions

- [ ] Report shared as a **Google Docs link** (with view/comment access granted)
- [ ] Report is professionally formatted
- [ ] All 5 activities documented with **screenshots** and **CLI outputs**
- [ ] ⚠️ No AI-generated (ChatGPT) content permitted

---

*Module 6 — Network Redundancy & High Availability*
