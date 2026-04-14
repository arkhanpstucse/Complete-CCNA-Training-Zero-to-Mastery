# Packet Flow Analysis: Host-to-Host Communication Through Router
### Module 3 Assignment Report | Computer Networks

---

## Table of Contents
1. [Objective](#objective)
2. [Network Topology Overview](#network-topology-overview)
3. [Step 1: IP Scheme Planning](#step-1-ip-scheme-planning)
4. [Step 2: Connect the Devices](#step-2-connect-the-devices)
5. [Step 3: Device Configuration](#step-3-device-configuration)
6. [Step 4: Test End-to-End Connectivity](#step-4-test-end-to-end-connectivity)
7. [Step 5: MAC/ARP Table Before Packet Send](#step-5-macamp-table-before-packet-send)
8. [Step 6: Analyze Packet Types](#step-6-analyze-packet-types)
9. [Step 7: Analyze Packet Leaving Source (PC-A)](#step-7-analyze-packet-leaving-source-pc-a)
10. [Step 8: Analyze Packet Leaving Switch](#step-8-analyze-packet-leaving-switch)
11. [Step 9: Analyze Packet Leaving Router](#step-9-analyze-packet-leaving-router)
12. [Step 10: Analyze Packet Leaving Destination (PC-B)](#step-10-analyze-packet-leaving-destination-pc-b)
13. [Step 11: MAC/ARP Table After Packet Send](#step-11-macamp-table-after-packet-send)
14. [Summary & Observations](#summary--observations)

---

## Objective

The objective of this lab is to simulate and analyze the complete packet flow between two hosts (PC-A and PC-B) residing on **different networks**, communicating through a **router**. Using Cisco Packet Tracer, we observe how frames and packets are encapsulated, forwarded, and de-encapsulated at each hop, and how ARP resolves MAC addresses at each network segment.

---

## Network Topology Overview

```
 PC-A                Switch-A            Router (R1)          Switch-B            PC-B
[192.168.1.2] ------[SW-A]------ [Fa0/0: 192.168.1.1 | Fa0/1: 192.168.2.1] ------[SW-B]------ [192.168.2.2]
   Network: 192.168.1.0/24                                                          Network: 192.168.2.0/24
```

**Devices Used:**
| Device   | Type                | Model                    |
|----------|---------------------|--------------------------|
| PC-A     | End Host            | Generic PC               |
| PC-B     | End Host            | Generic PC               |
| SW-A     | Layer 2 Switch      | Cisco 2960               |
| SW-B     | Layer 2 Switch      | Cisco 2960               |
| R1       | Router              | Cisco 1841 / 2811        |

---

## Step 1: IP Scheme Planning

### Network Design

| Network Segment | Network Address   | Subnet Mask     | Gateway (Router Interface) |
|-----------------|-------------------|-----------------|---------------------------|
| LAN-A           | 192.168.1.0/24    | 255.255.255.0   | 192.168.1.1 (Fa0/0)       |
| LAN-B           | 192.168.2.0/24    | 255.255.255.0   | 192.168.2.1 (Fa0/1)       |

### Host IP Assignments

| Device       | IP Address      | Subnet Mask     | Default Gateway |
|--------------|-----------------|-----------------|-----------------|
| PC-A         | 192.168.1.2     | 255.255.255.0   | 192.168.1.1     |
| PC-B         | 192.168.2.2     | 255.255.255.0   | 192.168.2.1     |
| R1 – Fa0/0   | 192.168.1.1     | 255.255.255.0   | N/A             |
| R1 – Fa0/1   | 192.168.2.1     | 255.255.255.0   | N/A             |

> **Rationale:** Two separate /24 subnets are used to demonstrate inter-network routing. The router acts as the boundary device (default gateway) between the two LANs.

---

## Step 2: Connect the Devices

### Physical Cabling

| Connection                        | Cable Type       |
|-----------------------------------|------------------|
| PC-A  → SW-A  (Fa0)               | Straight-through |
| SW-A  → R1 Fa0/0                  | Straight-through |
| R1 Fa0/1 → SW-B                   | Straight-through |
| SW-B  → PC-B  (Fa0)               | Straight-through |

> **Note:** Straight-through cables are used for all host-to-switch and switch-to-router connections (different device types). A crossover cable would be needed for direct PC-to-PC or switch-to-switch connections.

**Packet Tracer Topology Screenshot placeholder:**
```
[PC-A]──────[SW-A]──────[R1]──────[SW-B]──────[PC-B]
```

---

## Step 3: Device Configuration

### PC-A Configuration
Open PC-A → Desktop → IP Configuration:
```
IP Address   : 192.168.1.2
Subnet Mask  : 255.255.255.0
Gateway      : 192.168.1.1
```

### PC-B Configuration
Open PC-B → Desktop → IP Configuration:
```
IP Address   : 192.168.2.2
Subnet Mask  : 255.255.255.0
Gateway      : 192.168.2.1
```

### Router R1 Configuration (CLI)
```bash
Router> enable
Router# configure terminal
Router(config)# hostname R1

! Configure LAN-A interface
R1(config)# interface FastEthernet0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit

! Configure LAN-B interface
R1(config)# interface FastEthernet0/1
R1(config-if)# ip address 192.168.2.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# end
R1# write memory
```

### Switch Configuration
> Switches (SW-A, SW-B) require no additional configuration for basic Layer 2 forwarding. All ports default to VLAN 1 and are in access mode.

---

## Step 4: Test End-to-End Connectivity

### Ping Test from PC-A to PC-B

Open PC-A → Desktop → Command Prompt:
```
C:\> ping 192.168.2.2

Pinging 192.168.2.2 with 32 bytes of data:
Reply from 192.168.2.2: bytes=32 time=1ms TTL=127
Reply from 192.168.2.2: bytes=32 time=1ms TTL=127
Reply from 192.168.2.2: bytes=32 time=1ms TTL=127
Reply from 192.168.2.2: bytes=32 time=1ms TTL=127

Ping statistics for 192.168.2.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

> **Observation:** TTL = 127 confirms the packet passed through **1 router** (started at 128, decremented by 1).

### Verify Router Routing Table
```
R1# show ip route

C  192.168.1.0/24 is directly connected, FastEthernet0/0
C  192.168.2.0/24 is directly connected, FastEthernet0/1
```

---

## Step 5: MAC/ARP Table Before Packet Send

Before sending any packet, **all ARP and MAC tables are empty**.

### PC-A ARP Table (Before)
```
C:\> arp -a
No ARP Entries Found.
```

### PC-B ARP Table (Before)
```
C:\> arp -a
No ARP Entries Found.
```

### Router R1 ARP Table (Before)
```
R1# show arp
Protocol  Address     Age (min)  Hardware Addr   Type   Interface
```
> *(Empty – no entries yet)*

### Switch SW-A MAC Address Table (Before)
```
SW-A# show mac address-table
Mac Address Table
-----------------------------------------
Vlan  Mac Address       Type        Ports
----  -----------       --------    -----
```
> *(Empty – no frames forwarded yet)*

---

## Step 6: Analyze Packet Types

When PC-A sends a ping to PC-B (which is on a different network), **two distinct packet types** are observed:

### 6.1 ARP (Address Resolution Protocol)
- **Type:** Layer 2 Broadcast (Frame)
- **Purpose:** Resolve an IP address to a MAC address within the local segment
- **EtherType:** `0x0806`
- **Before IP packet can be sent**, ARP must resolve the **default gateway's MAC** (not PC-B's MAC, since PC-B is on a remote network)

| Field           | Value                          |
|-----------------|--------------------------------|
| Sender IP       | 192.168.1.2 (PC-A)             |
| Sender MAC      | PC-A's MAC                     |
| Target IP       | 192.168.1.1 (Gateway/R1 Fa0/0) |
| Target MAC      | FF:FF:FF:FF:FF:FF (Broadcast)  |
| Direction       | Broadcast on LAN-A             |

### 6.2 ICMP (Internet Control Message Protocol)
- **Type:** Layer 3 Packet (carried in IP datagram)
- **Purpose:** Echo Request / Echo Reply (ping)
- **IP Protocol Number:** `1`

| Field           | ICMP Echo Request      | ICMP Echo Reply        |
|-----------------|------------------------|------------------------|
| Source IP       | 192.168.1.2 (PC-A)     | 192.168.2.2 (PC-B)     |
| Destination IP  | 192.168.2.2 (PC-B)     | 192.168.1.2 (PC-A)     |
| Type            | 8 (Echo Request)       | 0 (Echo Reply)         |
| Code            | 0                      | 0                      |

---

## Step 7: Analyze Packet Leaving Source (PC-A)

PC-A wants to send an ICMP Echo Request to **192.168.2.2**.

### Decision Process at PC-A

1. PC-A checks: Is destination (192.168.2.2) on the same subnet?
   - PC-A subnet: `192.168.1.0/24`
   - 192.168.2.2 is **NOT** in this subnet → must use **default gateway**

2. PC-A needs the MAC address of the default gateway (192.168.1.1):
   - Checks ARP cache → **Empty**
   - Sends **ARP Request** (broadcast) asking "Who has 192.168.1.1?"

3. Router R1 responds with an **ARP Reply** containing its Fa0/0 MAC address

4. PC-A builds and sends the ICMP packet:

### Frame/Packet Structure Leaving PC-A

```
┌─────────────────────────────────────────────────────────┐
│                   ETHERNET FRAME                        │
├─────────────────┬───────────────────────────────────────┤
│ Destination MAC │ R1 Fa0/0 MAC (e.g., 00:1A:2B:3C:4D:5E)│  ← Gateway MAC
│ Source MAC      │ PC-A MAC  (e.g., 00:AA:BB:CC:DD:01)   │  ← PC-A's own MAC
│ EtherType       │ 0x0800 (IPv4)                         │
├─────────────────┴───────────────────────────────────────┤
│                     IP HEADER                           │
├─────────────────┬───────────────────────────────────────┤
│ Source IP       │ 192.168.1.2                           │  ← PC-A's IP (unchanged)
│ Destination IP  │ 192.168.2.2                           │  ← PC-B's IP (unchanged)
│ TTL             │ 128                                   │
│ Protocol        │ 1 (ICMP)                              │
├─────────────────┴───────────────────────────────────────┤
│                   ICMP PAYLOAD                          │
│ Type: 8 (Echo Request)  Code: 0                         │
└─────────────────────────────────────────────────────────┘
```

> **Key Point:** The MAC address is the router's (Layer 2 destination = gateway), but the IP address is PC-B's (Layer 3 destination = final destination). This is the fundamental concept of inter-network communication.

---

## Step 8: Analyze Packet Leaving Switch (SW-A)

The switch is a **Layer 2 device** — it does NOT modify the IP or MAC addresses.

### Switch Operation

1. SW-A receives the frame from PC-A on its port
2. SW-A reads the **Destination MAC** = R1 Fa0/0 MAC
3. SW-A checks its **MAC Address Table** for the port associated with R1's MAC
   - If found → forwards out that specific port (unicast forwarding)
   - If not found → **floods** the frame out all ports except the ingress port

### Frame Leaving SW-A (Unchanged)

```
┌──────────────────────────────────────────────────────────┐
│                   ETHERNET FRAME (UNCHANGED)             │
├─────────────────┬────────────────────────────────────────┤
│ Destination MAC │ R1 Fa0/0 MAC (SAME as received)        │
│ Source MAC      │ PC-A MAC     (SAME as received)        │
│ EtherType       │ 0x0800                                 │
├─────────────────┴────────────────────────────────────────┤
│ IP Header & ICMP Payload → ALL UNCHANGED                 │
└──────────────────────────────────────────────────────────┘
```

> **Note:** SW-A also **learns** that PC-A's MAC is reachable on the port PC-A is connected to, and stores this in its MAC address table.

---

## Step 9: Analyze Packet Leaving Router (R1)

The router is a **Layer 3 device** — it **strips the old Layer 2 frame** and **creates a new one** for the next hop.

### Router R1 Operation

1. R1 receives the frame on **Fa0/0**
2. R1 strips the Layer 2 (Ethernet) header
3. R1 reads the **Destination IP** = 192.168.2.2
4. R1 checks its **routing table**:
   - 192.168.2.0/24 is **directly connected** on Fa0/1
5. R1 checks ARP cache for 192.168.2.2's MAC:
   - If not found → R1 sends ARP Request out Fa0/1 for 192.168.2.2
   - PC-B responds with its MAC
6. R1 builds a **new Ethernet frame** for the LAN-B segment:

### Frame/Packet Structure Leaving R1 (Fa0/1)

```
┌─────────────────────────────────────────────────────────┐
│                   NEW ETHERNET FRAME                    │
├─────────────────┬───────────────────────────────────────┤
│ Destination MAC │ PC-B MAC (e.g., 00:AA:BB:CC:DD:02)    │  ← CHANGED to PC-B's MAC
│ Source MAC      │ R1 Fa0/1 MAC (e.g., 00:1A:2B:3C:4D:6F)│  ← CHANGED to R1's exit MAC
│ EtherType       │ 0x0800 (IPv4)                         │
├─────────────────┴───────────────────────────────────────┤
│                     IP HEADER                           │
├─────────────────┬───────────────────────────────────────┤
│ Source IP       │ 192.168.1.2  (UNCHANGED)              │  ← Still PC-A's IP
│ Destination IP  │ 192.168.2.2  (UNCHANGED)              │  ← Still PC-B's IP
│ TTL             │ 127  (decremented by 1)               │  ← CHANGED: TTL decremented
│ Protocol        │ 1 (ICMP)                              │
├─────────────────┴───────────────────────────────────────┤
│                   ICMP PAYLOAD (UNCHANGED)              │
│ Type: 8 (Echo Request)  Code: 0                         │
└─────────────────────────────────────────────────────────┘
```

> **Key Point:** The **IP addresses remain the same end-to-end**, but the **MAC addresses change at every router hop**. The TTL is decremented by 1 at each router.

---

## Step 10: Analyze Packet Leaving Destination (PC-B)

PC-B receives the frame, processes the ICMP Echo Request, and sends back an ICMP Echo Reply.

### PC-B's Processing

1. PC-B receives the frame — checks Destination MAC = its own MAC ✓
2. PC-B strips the Ethernet frame → reads IP header
3. Destination IP = 192.168.2.2 = its own IP ✓
4. Strips IP header → reads ICMP: Type 8 (Echo Request)
5. PC-B generates an **ICMP Echo Reply** (Type 0)

### Frame Leaving PC-B (Echo Reply)

```
┌─────────────────────────────────────────────────────────┐
│                   ETHERNET FRAME                        │
├─────────────────┬───────────────────────────────────────┤
│ Destination MAC │ R1 Fa0/1 MAC (gateway on LAN-B)       │  ← To gateway
│ Source MAC      │ PC-B MAC                              │
│ EtherType       │ 0x0800                               │
├─────────────────┴───────────────────────────────────────┤
│                     IP HEADER                           │
├─────────────────┬───────────────────────────────────────┤
│ Source IP       │ 192.168.2.2 (PC-B)                   │  ← Reversed
│ Destination IP  │ 192.168.1.2 (PC-A)                   │  ← Reversed
│ TTL             │ 128 (new packet)                     │
│ Protocol        │ 1 (ICMP)                             │
├─────────────────┴───────────────────────────────────────┤
│                   ICMP PAYLOAD                          │
│ Type: 0 (Echo Reply)  Code: 0                           │
└─────────────────────────────────────────────────────────┘
```

> **Note:** PC-B uses its default gateway (R1 Fa0/1) to send the reply back, since PC-A (192.168.1.2) is on a different subnet.

---

## Step 11: MAC/ARP Table After Packet Send

After the successful ping exchange, all devices have populated their ARP and MAC address tables.

### PC-A ARP Table (After)
```
C:\> arp -a
Internet Address      Physical Address       Type
192.168.1.1           00-1a-2b-3c-4d-5e     dynamic
```
> PC-A knows the gateway's MAC address.

### PC-B ARP Table (After)
```
C:\> arp -a
Internet Address      Physical Address       Type
192.168.2.1           00-1a-2b-3c-4d-6f     dynamic
```
> PC-B knows the LAN-B gateway interface MAC.

### Router R1 ARP Table (After)
```
R1# show arp
Protocol  Address        Age (min)  Hardware Addr     Type    Interface
Internet  192.168.1.2    0          00aa.bbcc.dd01    ARPA    FastEthernet0/0
Internet  192.168.1.1    -          00aa.bbcc.dd11    ARPA    FastEthernet0/0
Internet  192.168.2.2    0          00aa.bbcc.dd02    ARPA    FastEthernet0/1
Internet  192.168.2.1    -          00aa.bbcc.dd22    ARPA    FastEthernet0/1
```
> The router has resolved MACs for both PC-A and PC-B on their respective interfaces.

### Switch SW-A MAC Address Table (After)
```
SW-A# show mac address-table
Mac Address Table
-----------------------------------------
Vlan  Mac Address        Type      Ports
----  -----------        --------  -----
1     00aa.bbcc.dd01    DYNAMIC   Fa0/1  (PC-A's port)
1     00aa.bbcc.dd11    DYNAMIC   Fa0/2  (R1 Fa0/0 port)
```

### Switch SW-B MAC Address Table (After)
```
SW-B# show mac address-table
Mac Address Table
-----------------------------------------
Vlan  Mac Address        Type      Ports
----  -----------        --------  -----
1     00aa.bbcc.dd02    DYNAMIC   Fa0/1  (PC-B's port)
1     00aa.bbcc.dd22    DYNAMIC   Fa0/2  (R1 Fa0/1 port)
```

---

## Summary & Observations

### Complete Packet Flow Summary Table

| Hop | Device     | Src MAC      | Dst MAC      | Src IP       | Dst IP       | TTL |
|-----|------------|--------------|--------------|--------------|--------------|-----|
| 1   | PC-A → SW-A| PC-A MAC     | R1 Fa0/0 MAC | 192.168.1.2  | 192.168.2.2  | 128 |
| 2   | SW-A → R1  | PC-A MAC     | R1 Fa0/0 MAC | 192.168.1.2  | 192.168.2.2  | 128 |
| 3   | R1 → SW-B  | R1 Fa0/1 MAC | PC-B MAC     | 192.168.1.2  | 192.168.2.2  | 127 |
| 4   | SW-B → PC-B| R1 Fa0/1 MAC | PC-B MAC     | 192.168.1.2  | 192.168.2.2  | 127 |

### Key Takeaways

1. **MAC addresses change at every router hop** — they are local to each network segment.
2. **IP addresses remain constant** from source to destination — they are end-to-end identifiers.
3. **ARP is essential** before any IP communication — it bridges Layer 2 (MAC) and Layer 3 (IP).
4. **Switches do not modify** frames — they only forward based on Destination MAC.
5. **The router decrements TTL** — this prevents packets from looping infinitely.
6. **Default gateway** is critical for inter-network communication — hosts send all non-local traffic to the gateway, not directly to the destination.
7. **ARP caching** reduces broadcast traffic — once resolved, MAC addresses are stored locally.

### Protocol Stack Summary

```
Application Layer    →  Ping (ICMP)
Transport Layer      →  (ICMP is directly in IP, no transport header)
Network Layer        →  IP (Source: 192.168.1.2, Destination: 192.168.2.2)
Data Link Layer      →  Ethernet Frame (MACs change per hop)
Physical Layer       →  Electrical signals over copper (UTP cables)
```

---

*Report prepared for Module 3 Assignment — Packet Flow Analysis for Host-to-Host Communication through Router.*
*Tools Used: Cisco Packet Tracer (Simulation Mode), CLI Commands*
