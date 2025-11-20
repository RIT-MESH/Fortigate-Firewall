# Switching Basics + Advanced STP + EtherChannel Reference

This document is a complete, all‑in‑one reference covering switching fundamentals, VLANs, spanning tree (STP/RSTP/MST), and EtherChannel/LACP. Designed for both beginners and advanced engineers, this guide includes configuration examples, verification commands, and real‑world troubleshooting.

---

# 1. Switching Basics

## 1.0 Basic Switching Topology (Diagram)

```
       +-----------+
       |  Switch 1 |
       |  (Core)   |
       +-----+-----+
             |
       Trunk g0/1
             |
       +-----+-----+
       |  Switch 2 |
       | Access L2 |
       +--+-----+--+
          |     |
     VLAN10  VLAN20
        |       |
      PC1     Server
```

## 1.1 Access Ports

Used to connect end devices (PCs, printers, servers).

```
interface e0/1
 switchport mode access
 switchport access vlan 10
```

## 1.2 Trunk Ports

Used between switches, or switch → firewall.

```
interface e0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 switchport trunk native vlan 10
```

## 1.3 Show VLAN Info

```
show vlan brief
show interfaces trunk
```

## 1.4 MAC Address Table

```
show mac address-table
```

## 1.5 ARP

```
show ip arp
```

---

# 2. VLAN Fundamentals

## 2.1 Create VLANs

```
vlan 10
 name Users
vlan 20
 name Servers
```

## 2.2 Assign Ports to VLAN

```
interface e0/2
 switchport mode access
 switchport access vlan 20
```

## 2.3 Trunk Uplink

```
interface g0/1
 switchport mode trunk
 switchport trunk allowed vlan 1,10,20,30
```

---

# 3. Spanning Tree Protocol (STP)

## 3.0 STP Root & Blocking Example (Diagram)

```
      (Root Bridge)
         Switch A
        priority 4096
             |
     Forwarding port
             |
             |
     Blocking port
             X
             |
        Switch B
```

## 3.1 STP Modes

* **STP (802.1D):** slow convergence
* **RSTP (802.1w):** fast convergence
* **MST (802.1s):** scalable multi-instance

Enable RSTP:

```
spanning-tree mode rapid-pvst
```

## 3.2 STP Port Roles

* **Root Port:** best path to root bridge
* **Designated Port:** forwarding on each segment
* **Alternate Port:** backup path
* **Backup Port:** rarely seen

## 3.3 STP Port States

* Discarding
* Learning
* Forwarding

## 3.4 STP Root Bridge Configuration

```
spanning-tree vlan 1,10,20 priority 4096
```

Lower value = higher priority.

## 3.5 STP Protection Features

### BPDU Guard (edge ports)

```
interface e0/3
 spanning-tree bpduguard enable
```

### Root Guard

```
interface g0/1
 spanning-tree guard root
```

### Loop Guard

```
interface g0/2
 spanning-tree guard loop
```

### Portfast

```
interface e0/5
 spanning-tree portfast
```

## 3.6 STP Verification

```
show spanning-tree
show spanning-tree detail
show spanning-tree interface g0/1
```

---

# 4. EtherChannel / LACP

## 4.0 EtherChannel (LACP) Diagram

```
             +-------------------+
             |     Switch A      |
             |-------------------|
             |  g0/1   g0/2      |
             +---+------+--------+
                 |      |
   LACP Po1 <----+      +----> LACP Po1
                 |      |
             +---+------+--------+
             |     Switch B      |
             |-------------------|
             |  g0/1   g0/2      |
             +-------------------+
```

## 4.1 EtherChannel Modes

* **on** = static, no negotiation
* **active/passive** = LACP (802.3ad)

## 4.2 LACP Example Between Two Switches

```
interface range g0/1 - 2
 channel-group 1 mode active

interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

## 4.3 Static EtherChannel

```
interface range g0/3 - 4
 channel-group 2 mode on
```

## 4.4 Show EtherChannel Status

```
show etherchannel summary
show interfaces port-channel 1
```

---

# 5. Real‑World Designs

## 5.1 Switch → FortiGate via LACP

### 5.1.1 Topology Diagram

```
+-------------+            +-----------------+
|  Switch     |  LACP Po10 |   FortiGate     |
|  g0/1,g0/2  +===========>+  port3,port4    |
+-------------+            +-----------------+
      |                            |
   VLANs                        Internal
   10,20,30                     interfaces
```

Switch side:

```
interface range g0/1-2
 channel-group 10 mode active
```

FortiGate side:

```
config system interface
 edit "agg1"
  set type aggregate
  set member port3 port4
  set lacp-mode active
 next
end
```

## 5.2 Core Switch Redundancy with MST

```
spanning-tree mode mst
spanning-tree mst configuration
 name CORE
 revision 1
 instance 1 vlan 10,20
 instance 2 vlan 30,40
```

---

# 6. Troubleshooting Switching / STP / EtherChannel

## 6.1 VLAN Issues

Checklist:

* Is the port an access or trunk port?
* Correct VLAN allowed?
* Native VLAN mismatch?

```
show interfaces trunk
show vlan brief
```

## 6.2 STP Blocking Unexpectedly

```
show spanning-tree interface g0/1
show spanning-tree detail
```

Check for:

* Root bridge location
* Wrong priority
* Loops

## 6.3 EtherChannel Down / Inconsistent

```
show etherchannel summary
show interfaces g0/1 switchport
```

Common issues:

* Speed/duplex mismatch
* VLANs different between members
* One port configured as access, other trunk
* Wrong LACP mode

## 6.4 Broadcast Storm Symptoms

* CPU high
* STP flapping
* MAC table moving constantly

```
show processes cpu
show mac address-table
```

---

# 7. Quick Cheat Sheet

### Access Port

```
int e0/1
 sw mode access
 sw access vlan 10
```

### Trunk Port

```
int g0/1
 sw mode trunk
 sw trunk allowed vlan 10,20,30
```

### EtherChannel (LACP)

```
int range g0/1-2
 channel-group 1 mode active
```

### STP Root Bridge

```
spanning-tree vlan 1 priority 4096
```

### STP Protections

```
spanning-tree bpduguard enable
spanning-tree guard loop
spanning-tree guard root
```

---

# 8. Fully Commented Reference

## 8.1 Access Port (Commented)

```bash
interface e0/1                       # Configure interface e0/1
 switchport mode access              # Set as access (non-trunk) port
 switchport access vlan 10           # Assign this port to VLAN 10
```

## 8.2 Trunk Port with Native VLAN (Commented)

```bash
interface e0/1                       # Trunk uplink
 switchport mode trunk               # Enable trunk mode
 switchport trunk allowed vlan 10,20,30   # Specify allowed VLANs
 switchport trunk native vlan 10     # Native VLAN = untagged VLAN
```

## 8.3 VLAN Creation (Commented)

```bash
vlan 10                              # Create VLAN 10
 name Users                          # Name VLAN
vlan 20
 name Servers
```

## 8.4 EtherChannel LACP (Commented)

```bash
interface range g0/1 - 2             # Select interfaces g0/1 and g0/2
 channel-group 1 mode active         # Enable LACP in active mode

interface port-channel 1             # Logical link
 switchport mode trunk               # Make EtherChannel a trunk
 switchport trunk allowed vlan 10,20,30
```

## 8.5 STP Root Bridge (Commented)

```bash
spanning-tree vlan 1,10,20 priority 4096   # Force this switch as root
```

## 8.6 STP Protections (Commented)

```bash
interface e0/5
 spanning-tree portfast              # Skip listening/learning on edge ports

interface e0/6
 spanning-tree bpduguard enable      # Shut port if BPDU received (protects LAN)

interface g0/1
 spanning-tree guard root            # Prevent switch from becoming root

interface g0/2
 spanning-tree guard loop            # Avoid loops if BPDUs stop unexpectedly
```

---


End of README.
