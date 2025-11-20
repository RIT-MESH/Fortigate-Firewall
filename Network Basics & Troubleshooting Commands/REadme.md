# Full Network Basics, Configuration & Troubleshooting Reference

This document contains **all FortiGate, Switch, and Router commands**, fully structured, and consolidated. It includes:

* FortiGate basics, interface configs, DHCP, redundant WAN, troubleshooting
* Switch (Cisco L2) VLAN, STP, EtherChannel, troubleshooting
* Router (Cisco L3) WAN, DHCP relay/server, routing, redundant WAN, troubleshooting
* End‑to‑end workflow, templates, and ping matrix

---

# 1. FORTIGATE

## 1.1 Basics

```
show                     # Display current config
config                   # Enter config mode
end                      # Exit config mode
execute                  # System-level ops
get                      # Show system info
```

---

## 1.2 Essential Commands

### Interface Information

```
get system interface                      # Show all interfaces, IPs, status
get system interface physical             # Hardware-level info (speed/duplex)
execute ping <IP>                         # Connectivity test
execute traceroute <IP>                   # Path check
```

### Routing

```
get router info routing-table all         # Full routing table
get router info bgp summary               # BGP status
get router info ospf neighbor             # OSPF neighbors
```

### Firewall policies

```
show firewall policy                      # List rules in numerical order
```

---

## 1.3 Core Configuration Blocks

### System interface

```
config system interface                   # Interface config tree
    edit port1
        set ip 192.168.1.99/24            # Assign IP
        set allowaccess ping http https ssh
        set role lan
    next
end
```

### System global

```
config system global
    set hostname FGT-LAB
    set gui-theme dark
    set admintimeout 30
end
```

### Admin configuration

```
config system admin
    edit admin2
        set password StrongPass
        set access-profile super_admin
    next
end
```

---

## 1.4 FortiGate Configuration Modes

### Global Mode

```
config global            # Manage system-wide & HA settings
end
```

### VDOM Mode

```
config vdom
edit root
end
```

---

## 1.5 Global Useful Commands

```
get system status                       # Serial, version, uptime
get system performance status           # CPU, RAM
execute traceroute <IP>                 # Trace path
execute ping-option repeat 5 <IP>       # Enhanced ICMP test
```

---

## 1.6 Interface Configuration Deep Dive

### Physical Interface

```
config system interface
    edit port1
        set ip 192.168.1.99 255.255.255.0
        set allowaccess ping http https ssh
    next
end
```

### VLAN Interface

```
config system interface
    edit vlan10
        set interface port1
        set vlanid 10
        set ip 192.168.10.1 255.255.255.0
    next
end
```

### Redundant Interface (active/passive ports)

```
config system interface
    edit RED1
        set type redundant
        set member port9 port10
        set ip 192.168.20.1 255.255.255.0
    next
end
```

### Aggregate (LACP)

```
config system interface
    edit AGG1
        set type aggregate
        set member port3 port4
        set lacp-mode active
        set ip 10.10.10.1 255.255.255.0
    next
end
```

---

## 1.7 DHCP Server

```
config system dhcp server
    edit 1
        set interface vlan10
        config ip-range
            edit 1
                set start-ip 192.168.10.50
                set end-ip 192.168.10.200
            next
        end
        set netmask 255.255.255.0
        set default-gateway 192.168.10.1
    next
end
```

---

## 1.8 DHCP Relay

```
config system interface
    edit lan
        set ip 192.168.10.1 255.255.255.0
        set dhcp-relay-service enable
        set dhcp-relay-ip 10.0.0.10
    next
end
```

---

## 1.9 Redundant WAN (SD‑WAN)

```
config system virtual-wan-link
    set status enable
    config members
        edit 1
            set interface wan1
            set gateway 203.0.113.1
        next
        edit 2
            set interface wan2
            set gateway 198.51.100.1
        next
    end
    config health-check
        edit google
            set server 8.8.8.8
        next
    end
    config service
        edit 1
            set src all
            set dst all
            set priority-members 1 2
        next
    end
end
```

---

## 1.10 Logs & Debug Tools

```
diagnose debug enable
diagnose debug console timestamp enable
diagnose debug flow filter addr <IP>
diagnose debug flow trace start 20
diagnose debug disable
```

---

## 1.11 FortiGate Troubleshooting (Unified)

### NAT

```
show firewall central-nat
show firewall policy
diagnose firewall iprope lookup <src> <dst> <protocol>
```

### Sessions

```
diag sys session list
diag sys session filter src <IP>
```

### Connectivity

```
execute ping <IP>
execute traceroute <IP>
```

### Routing

```
get router info routing-table all
```

---

# 2. SWITCH (Cisco L2)

---

## 2.1 Basics

```
show vlan brief                      # VLAN list
show interfaces status               # Port states
show mac address-table               # L2 forwarding table
```

---

## 2.2 VLANs & Ports

### Access Port

```
interface e0/1
 switchport mode access
 switchport access vlan 10
```

### Trunk Port

```
interface g0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

---

## 2.3 STP

```
show spanning-tree
```

### Protection

```
spanning-tree bpduguard enable
spanning-tree guard root
spanning-tree guard loop
```

---

## 2.4 EtherChannel (LACP)

```
interface range g0/1 - 2
 channel-group 1 mode active
!
interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

---

## 2.5 Switch Troubleshooting

```
show vlan brief                      # Missing VLAN
show interfaces trunk                # Trunk mismatch
show mac address-table               # MAC movement
show interfaces counters errors      # CRC, drops
show spanning-tree vlan 10           # STP state
```

---

# 3. ROUTER (Cisco L3)

---

## 3.1 Router Basics

```
show running-config                   # Active config
show startup-config                   # Boot config
configure terminal                    # Enter config mode
exit                                   # Move back one level
```

---

## 3.2 Interface & IP

```
show ip interface brief               # Status + IP summary
show interfaces                       # Detailed stats

interface g0/0
 ip address X.X.X.X Y.Y.Y.Y
 no shutdown
```

---

## 3.3 Routing Commands

```
show ip route                         # Complete routing table
show ip protocols                     # Protocols running (OSPF/EIGRP)
show ip ospf neighbor                 # OSPF adjacency states
show ip bgp summary                   # BGP peer states & prefixes
```

---

## 3.4 Connectivity

```
ping <IP>
traceroute <IP>
```

---

## 3.5 WAN Types

### Static

```
interface g0/0
 ip address 203.0.113.2 255.255.255.252
```

### DHCP

```
interface g0/0
 ip address dhcp
```

### PPPoE

```
interface Dialer1
 ip address negotiated
 encapsulation ppp
 dialer pool 1
```

---

## 3.6 DHCP Server

```
ip dhcp pool LAN
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
```

---

## 3.7 DHCP Relay

```
interface Vlan10
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 10.0.0.10
```

---

## 3.8 Redundant WAN (IP SLA)

```
ip sla 1
 icmp-echo 8.8.8.8 source-interface g0/0
 frequency 5
ip sla schedule 1 life forever start-time now

track 1 ip sla 1 reachability

ip route 0.0.0.0 0.0.0.0 203.0.113.1 track 1
ip route 0.0.0.0 0.0.0.0 198.51.100.1 200
```

---

## 3.9 Router Troubleshooting

```
show ip arp                           # ARP table
show interfaces counters errors       # CRC/drop
show ip ospf neighbor                 # OSPF state
show ip bgp summary                   # BGP up/down
```

---

# 4. End‑to‑End Troubleshooting Workflow

### Step 1 – Connectivity

```
PC → Gateway
PC → Remote
FGT → Internet
```

### Step 2 – Interfaces

```
FGT: get system interface
RTR: show ip int brief
SW:  show interfaces status
```

### Step 3 – VLAN/Trunk

```
show vlan brief
show interfaces trunk
```

### Step 4 – Routing

```
FGT → get router info routing-table all
RTR → show ip route
```

### Step 5 – Policies (FGT)

```
show firewall policy
```

### Step 6 – Debug (FGT)

```
diag debug flow trace start 20
```

---

# 5. Quick Templates

### FortiGate VLAN

```
config system interface
 edit vlan10
  set interface port1
  set vlanid 10
  set ip 192.168.10.1 255.255.255.0
 next
end
```

### Cisco Router Static Route

```
ip route 0.0.0.0 0.0.0.0 <next-hop>
```

### Cisco Switch Trunk

```
interface g0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

---

# 6. Ping Matrix

| From → To          | Purpose                 |
| ------------------ | ----------------------- |
| PC → Gateway       | VLAN/Interface check    |
| PC → PC            | L2 or L3 issue          |
| PC → Router        | Routing + VLAN OK       |
| PC → FortiGate     | L3 path correctness     |
| FGT → PC           | Reverse path + ARP      |
| FGT → Internet     | WAN + NAT operation     |
| Router → FortiGate | Routing neighbor checks |

---

End of consolidated document.
