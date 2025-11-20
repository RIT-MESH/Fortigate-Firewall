# Network Basics & Troubleshooting Commands (FortiGate, Router, Switch)

This README provides a clean, practical reference for essential configuration and troubleshooting commands used in FortiGate firewalls, Routers (Cisco-style), and Switches. It is meant for quick use in labs, GNS3, Packet Tracer, or real devices.

---

# 1. FortiGate Basics

## 1.1 Common Navigation Commands

```
show        ← Display running configuration
config      ← Enter configuration mode
end         ← Exit config mode
execute     ← Run operational system commands
get         ← Show status/info for components
```

---

# 2. FortiGate Essential Commands

## 2.1 Interface Information

```
get system interface                 ← Show all interfaces
get system interface physical        ← Show hardware port status
execute ping <IP>                    ← Ping from FortiGate
execute traceroute <IP>              ← Trace route
```

## 2.2 Routing

```
get router info routing-table all    ← Full routing table
get router info bgp summary          ← BGP status (if used)
get router info ospf neighbor        ← OSPF neighbors
```

## 2.3 Firewall Policies

```
show firewall policy                 ← View firewall rules
```

## 2.4 Logs & Debug

```
diagnose debug enable

# Check flows
diagnose debug flow filter addr <IP>
diagnose debug flow trace start 20

diagnose debug disable
```

## 2.5 System Monitoring

```
get system status                    ← Version + System info
get hardware nic <port>              ← Hardware-level interface info
```

---

# 3. Router (Cisco) Basics

## 3.1 Navigation

```
show running-config
show startup-config
configure terminal
exit
```

## 3.2 Interface & IP Commands

```
show ip interface brief              ← Summary of all interfaces
show interfaces                      ← Detailed interface info
interface g0/0
 ip address X.X.X.X Y.Y.Y.Y
 no shutdown
```

## 3.3 Routing Commands

```
show ip route                        ← Routing table
show ip protocols                    ← Dynamic routing protocol details
show ip ospf neighbor                ← OSPF neighbors
show ip bgp summary                  ← BGP status
```

## 3.4 Connectivity Troubleshooting

```
ping <IP>
traceroute <IP>
```

---

# 4. Switch Basics (Layer 2 / Cisco Style)

## 4.1 Interface & VLAN Commands

```
show vlan brief                      ← VLAN status
show interfaces status               ← Port status summary
show mac address-table               ← MAC table
```

## 4.2 Access & Trunk Configuration

```
# Access Port
interface e0/1
 switchport mode access
 switchport access vlan 10

# Trunk Port
interface e0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

## 4.3 Spanning Tree

```
show spanning-tree                   ← View STP status
```

---

# 5. End-to-End Troubleshooting Workflow

## Step 1: Check Connectivity

* From PC:

```
ping gateway
ping remote network
```

* From Router:

```
ping <IP>
traceroute <IP>
```

* From FortiGate:

```
execute ping <IP>
```

---

## Step 2: Check Interface Status

* FortiGate:

```
get system interface
```

* Router:

```
show ip interface brief
```

* Switch:

```
show interfaces status
```

---

## Step 3: Check VLAN & Trunking

* Switch:

```
show vlan brief
show interfaces trunk
```

---

## Step 4: Check Routing

* FortiGate:

```
get router info routing-table all
```

* Router:

```
show ip route
```

---

## Step 5: Check Firewall Policies (FortiGate)

```
show firewall policy
```

---

## Step 6: Use Debug Tools (FortiGate)

```
diagnose debug reset
diagnose debug flow filter clear
diagnose debug flow filter addr <IP>
diagnose debug flow trace start 20
```

---

# 6. Quick Cheat Sheet

### FortiGate

* `execute ping` → Connectivity test
* `get system interface` → Interface status
* `diagnose debug flow` → Packet path debugging

### Router

* `show ip int brief` → Fast interface check
* `show ip route` → Routing table

### Switch

* `show vlan brief` → VLAN info
* `show mac address-table` → MAC learning

---

If you want, I can also create versions for:

* Advanced routing (BGP, OSPF, redistribution)
* FortiGate NAT, VIP, SD-WAN
* GNS3 lab-ready templates
* A printable PDF cheat sheet

---

# 7. Additional FortiGate Useful Sections

## 7.1 NAT Troubleshooting

```
show firewall central-nat
show firewall policy
get router info routing-table all

# Test SNAT behavior
diagnose firewall iprope lookup <src_ip> <dst_ip> <protocol>
```

## 7.2 Check Session Table

```
diag sys session list
# Filter by source
diag sys session filter src <IP>
# Filter by destination
diag sys session filter dst <IP>
```

## 7.3 Clear Sessions

```
diag sys session clear
```

---

# 8. Additional Router (Cisco) Troubleshooting

## 8.1 ARP & Neighbor Issues

```
show ip arp
clear ip arp
```

## 8.2 Interface Errors

```
show interfaces counters errors
show interfaces | include error
```

## 8.3 Routing Protocol Debug

```
debug ip routing
! OSPF
debug ip ospf events
debug ip ospf adj
! EIGRP
debug eigrp packets
debug eigrp fsm
```

---

# 9. Additional Switch (Cisco) Troubleshooting

## 9.1 Port Security

```
show port-security
show port-security interface <int>
```

## 9.2 Errors & Duplex Issues

```
show interfaces <int> counters errors
show interfaces <int> status
```

## 9.3 Trunk Problems

```
show interfaces trunk
show spanning-tree vlan <ID>
```

---

# 10. Common Real-World Troubleshooting Scenarios

## Scenario 1: PC cannot reach gateway

**Check:**

* Wrong VLAN on switch
* Wrong default gateway on PC
* Redundant interface down
* Policy blocking traffic (FortiGate)

**Commands:**

```
# Switch
show vlan brief

# FortiGate
get system interface
execute ping <PC>
```

---

## Scenario 2: Two networks cannot communicate

**Possible causes:**

* Missing route
* Policy not created
* Wrong subnet mask
* Trunk not carrying VLANs

**Commands:**

```
# Router
show ip route

# FortiGate
get router info routing-table all
show firewall policy

# Switch
show interfaces trunk
```

---

## Scenario 3: Internet not working

**Check:**

* Default route
* NAT rule
* DNS
* ISP link

**Commands:**

```
# FortiGate
get router info routing-table all
show firewall policy
show firewall central-nat
execute ping 8.8.8.8
execute ping google.com
```

---

# 11. Quick Templates

## 11.1 FortiGate VLAN Template

```
config system interface
    edit vlan10
        set vdom root
        set ip 192.168.10.1 255.255.255.0
        set interface port1
        set vlanid 10
    next
end
```

## 11.2 Cisco Router Static Route Template

```
ip route 0.0.0.0 0.0.0.0 <next-hop>
```

## 11.3 Cisco Switch Trunk Template

```
interface g0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

---

# 12. Very Useful Ping Matrix

Use this matrix when diagnosing multi-device issues:

| Test From            | Test To                | Purpose |
| -------------------- | ---------------------- | ------- |
| PC → Gateway         | Connectivity, VLAN OK  |         |
| PC → Other PC        | Inter-VLAN or L2 issue |         |
| PC → Router          | Routing + VLAN OK      |         |
| PC → FortiGate       | L2/L3 path OK          |         |
| FortiGate → PC       | Reverse-path + routing |         |
| FortiGate → Internet | WAN + NAT              |         |
| Router → FortiGate   | Routing neighbor check |         |

---

# 13. Need Even More?

I can add:

* SD-WAN basics
* BGP complete lab
* OSPF multi-area
* Redundant WAN setup
* DHCP relay + servers
* GNS3 full sample topologies

Tell me what you want next.

---

# 14. FortiGate Core Configuration Blocks (Most Used)

Below are the **main FortiGate configuration trees** you will use constantly.

## 14.1 `config system interface`

Used to configure:

* Physical interfaces
* VLAN interfaces
* Redundant interfaces
* Aggregate (LACP) interfaces
* Loopback interfaces

### Example

```
config system interface
    edit port1
        set ip 192.168.1.99 255.255.255.0
        set allowaccess ping http https ssh
        set role lan
    next
end
```

---

## 14.2 `config system global`

Used for **system-wide settings** such as hostname, management ports, GUI preferences, DNS, inspection mode.

### Example

```
config system global
    set hostname FGT-LAB
    set timezone 04
    set admintimeout 30
    set gui-theme dark
end
```

---

## 14.3 `config system admin`

Used to create, modify, or delete admin accounts.

### Example

```
config system admin
    edit admin2
        set password Forti123
        set access-profile super_admin
        set two-factor email
    next
end
```

---

# 14. FortiGate Configuration Modes (GLOBAL vs CONFIG)

FortiGate has **two major configuration layers**: **Global Mode** and **VDOM Mode**.
Some commands only work in one of these modes depending on device setup.

## 14.1 Global Configuration Mode

Used when multiple VDOMs exist. Lets you manage system-wide settings.

### Enter Global Mode

```
config global
```

### Exit Global Mode

```
end
```

### Common Global Mode Commands

```
config system global              ← system-wide settings
config system admin               ← admin accounts
config router static              ← static routes (depending on vdom)
config system ha                  ← HA cluster settings
```

---

## 14.2 VDOM (Virtual Domain) Mode

Each VDOM acts like its own firewall.
If your unit uses VDOMs, you must enter the specific VDOM before running interface or policy commands.

### Enter a VDOM

```
config vdom
edit <vdom-name>
```

Example:

```
config vdom
edit root
```

### Exit VDOM

```
end
```

---

# 15. FortiGate Interface Configuration Deep Dive

Below are the most important interface-related config blocks.

## 15.1 Configure a Physical Interface

```
config system interface
    edit port1
        set ip 192.168.1.99 255.255.255.0
        set allowaccess ping http https ssh
        set role lan
    next
end
```

## 15.2 Configure a VLAN Interface

```
config system interface
    edit vlan10
        set vdom root
        set interface port1
        set vlanid 10
        set ip 192.168.10.1 255.255.255.0
    next
end
```

## 15.3 Configure a Redundant Interface

```
config system interface
    edit RED1
        set type redundant
        set member port9 port10
        set ip 192.168.20.1 255.255.255.0
    next
end
```

## 15.4 Configure an Aggregate (LACP) Interface

```
config system interface
    edit LACP-AGG
        set type aggregate
        set member port3 port4
        set lacp-mode active
        set ip 10.10.10.1 255.255.255.0
    next
end
```

## 15.5 Configure DHCP Server on an Interface

```
config system dhcp server
    edit 1
        set interface vlan10
        set lease-time 86400
        config ip-range
            edit 1
                set start-ip 192.168.10.50
                set end-ip 192.168.10.200
            next
        end
        set default-gateway 192.168.10.1
        set netmask 255.255.255.0
    next
end
```

---

# 16. FortiGate Global Useful Commands

```
get system status                ← firmware, serial, uptime
get system performance status    ← CPU & memory usage
get router info bgp summary      ← BGP details
get router info ospf interface   ← OSPF interface summary
execute traceroute <IP>          ← trace path
execute ping-option repeat 5     ← ping with options
```

---

# 17. FortiGate Debug Levels (Very Useful)

## Set debug level

```
diagnose debug enable
diagnose debug console timestamp enable
```

## Disable debug (always do this!)

```
diagnose debug disable
```

---
