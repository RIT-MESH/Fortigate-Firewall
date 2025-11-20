# Complete Network Setup (VLAN 10 / 20 / 30 + L3 Switch + FortiGate)

This document covers the full configuration for the following network topology:

* **L3 Switch performing inter‑VLAN routing**
* **FortiGate acting as WAN/Firewall**
* **Three VLANs (10, 20, 30)**
* **Admin PC, PC2, PC3 connected to different VLANs**
* **Trunk link between switch and FortiGate**

---

## 1. Network Topology

<img width="936" height="516" alt="image" src="https://github.com/user-attachments/assets/4eb2be72-b970-4e2d-85d4-711f07f559ad" />
This section explains the overall physical and logical layout of the network.

```
                Firefox (Admin PC)
                  VLAN 10 (192.168.10.0/24)
                          |
                       g0/1 (Access 10)
                          |
                 +---------------------+
                 |       SW2-1         |
                 +---------------------+
             g0/2 (Acc20)      g0/3 (Acc30)
               PC2               PC1
            VLAN20             VLAN30

                       g0/0 (TRUNK)
                              |
                        FortiGate port1
                        VLAN10/20/30 TAGGED
                        VLAN10 GW: 192.168.10.1
```

---

## 2. VLAN & Subnet Plan

| VLAN | Subnet          | Purpose        | Gateway        |
| ---- | --------------- | -------------- | -------------- |
| 10   | 192.168.10.0/24 | Admin / FG GUI | 192.168.10.100 |
| 20   | 192.168.20.0/24 | PC2            | 192.168.20.100 |
| 30   | 192.168.30.0/24 | PC1            | 192.168.30.100 |

---

## 3. Switch Configuration (L3 Switch)

### 3.1 Create VLANs

```bash
conf t
vlan 10
exit
vlan 20
exit
vlan 30
exit
end
```

### 3.2 Create SVI Interfaces (Gateways)

```bash
interface vlan 10
 ip address 192.168.10.100 255.255.255.0
 no shutdown
 exit
interface vlan 20
 ip address 192.168.20.100 255.255.255.0
 no shutdown
 exit
interface vlan 30
 ip address 192.168.30.100 255.255.255.0
 no shutdown
 exit
end
```

### 3.3 Assign Access Ports

```bash
interface g0/1
 switchport mode access
 switchport access vlan 10
 exit
interface g0/2
 switchport mode access
 switchport access vlan 20
 exit
interface g0/3
 switchport mode access
 switchport access vlan 30
 exit
end
```

### 3.4 Configure TRUNK to FortiGate

```bash
interface g0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 exit
end
```

### 3.5 Enable Routing

```bash
conf t
ip routing
end
```

---

## 4. FortiGate Configuration

### 4.1 Set Hostname

```bash
config system global
 set hostname FGT
end
```

### 4.2 Configure VLAN Interfaces

#### VLAN 10

```bash
config system interface
 edit "vlan10"
  set interface "port1"
  set vlanid 10
  set ip 192.168.10.1 255.255.255.0
  set allowaccess ping http https ssh
  set role lan
 next
end
```

#### VLAN 20

```bash
config system interface
 edit "vlan20"
  set interface "port1"
  set vlanid 20
  set ip 192.168.20.1 255.255.255.0
  set allowaccess ping
  set role lan
 next
end
```

#### VLAN 30

```bash
config system interface
 edit "vlan30"
  set interface "port1"
  set vlanid 30
  set ip 192.168.30.1 255.255.255.0
  set allowaccess ping
  set role lan
 next
end
```

---

## 5. Static Routes on FortiGate

```bash
config router static
 edit 1
  set dst 192.168.20.0 255.255.255.0
  set gateway 192.168.10.100
  set device "port1"
 next
 edit 2
  set dst 192.168.30.0 255.255.255.0
  set gateway 192.168.10.100
  set device "port1"
 next
end
```

---

## 6. PC IP Addressing

### VLAN 10

```
IP: 192.168.10.10
GW: 192.168.10.100
DNS: 8.8.8.8
```

### VLAN 20

```
IP: 192.168.20.10
GW: 192.168.20.100
DNS: 8.8.8.8
```

### VLAN 30

```
IP: 192.168.30.10
GW: 192.168.30.100
DNS: 8.8.8.8
```

---

## 7. Traffic Flow Explanation

### PC1 → PC2

```
PC1 → VLAN30 SVI → Switch → VLAN20 SVI → PC2
```

### PC → Internet

```
PC → SVI → Switch → VLAN10 → FortiGate → NAT → Internet
```

### Admin GUI

```
Browser → 192.168.10.1 → FortiGate GUI
```

---

## 8. Required Firewall Policies

```bash
config firewall policy
 edit 1
  set srcintf "vlan10" "vlan20" "vlan30"
  set dstintf "port2"
  set srcaddr all
  set dstaddr all
  set action accept
  set schedule always
  set service ALL
  set nat enable
 next
end
```

---

## 9. Troubleshooting Commands

This section provides common commands to diagnose connectivity issues on the switch, FortiGate, and PCs.

### On Switch

Below are the most important troubleshooting commands with clear explanations.

```
show vlan brief
```

Shows all VLANs configured on the switch and which ports belong to each VLAN.

```
show ip interface brief
```

Displays all Layer‑3 (SVI) interfaces with their IP addresses and up/down status.

```
show run
```

Shows the full running configuration of the switch, including VLANs, ports, and routing.

```
show interfaces trunk
```

Displays trunk ports, allowed VLANs, and tagging status. Used to confirm the switch → FortiGate trunk.

```
ping 192.168.x.x
```

Tests connectivity to gateways, other VLANs, FortiGate, or PCs. Replace x.x.x.x with the target IP.

### On FortiGate

Below are essential troubleshooting commands for diagnosing FortiGate network issues.

```
diag netlink interface list
```

Shows all FortiGate interfaces, their link status, and kernel-level network information.

```
get system interface
```

Displays all configured interfaces, their IP addresses, roles, and administrative status.

```
execute ping 192.168.x.x
```

Tests connectivity from the FortiGate to VLAN gateways, PCs, or external IPs.

```
get router info routing-table all
```

Shows the full routing table, which helps verify default routes, static routes, and learned routes.

### PC Level

```
ping gateway
tracert 8.8.8.8
ipconfig /all
```

---

## 10. Final Checklist

This section verifies all network components are configured properly and working together.

| Item                         | Status |
| ---------------------------- | ------ |
| VLANs created on switch      | ✔      |
| SVI gateways configured      | ✔      |
| Trunk to FortiGate           | ✔      |
| VLAN interfaces on FortiGate | ✔      |
| Static routes added          | ✔      |
| PCs have correct gateway     | ✔      |
| Firewall policy for internet | ✔      |

---

## 11. WAN / ISP Setup

This section configures the ISP/WAN interface on the FortiGate, including DHCP/static IP, NAT, and default routing so internal VLANs can reach the internet.

### 11.1 Configure WAN Interface (port2 example)

```bash
config system interface
    edit "port2"
        set mode dhcp            # If ISP provides IP automatically
        set allowaccess ping
        set role wan
        set alias "WAN"
    next
end
```

If your ISP gives a **static IP**, use:

```bash
config system interface
    edit "port2"
        set mode static
        set ip x.x.x.x y.y.y.y      # Public IP + subnet mask
        set allowaccess ping
        set role wan
    next
end
```

---

### 11.2 Default Route to ISP

FortiGate must forward all outbound traffic to the WAN gateway.

```bash
config router static
    edit 0
        set dst 0.0.0.0 0.0.0.0
        set gateway x.x.x.x        # ISP gateway
        set device "port2"
    next
end
```

If using DHCP on port2, FortiGate automatically receives the default route.

---

### 11.3 NAT for Internet Access

Your internal VLANs must be NAT’ed out of port2.

```bash
config firewall policy
    edit 10
        set srcintf "vlan10" "vlan20" "vlan30"
        set dstintf "port2"
        set srcaddr all
        set dstaddr all
        set action accept
        set schedule always
        set service ALL
        set nat enable
    next
end
```

---

### 11.4 Test WAN Connectivity

```bash
execute ping 8.8.8.8
execute traceroute 8.8.8.8
```

If ping works, VLAN clients will also reach the internet.

---

### 11.5 Example End-to-End Traffic Flow

```
PC → SVI Gateway (Switch) → FortiGate VLAN10 → NAT on port2 → ISP → Internet
```

---
