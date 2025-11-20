# OSPF & BGP Lab – FortiGate + Cisco Router

This document provides complete, working configurations for OSPF and BGP between FortiGate and Cisco routers, along with verification and troubleshooting commands.

---

## 1. Lab Topology

### 1.1 OSPF Topology

```
      LAN-A (FGT side)
      192.168.10.0/24
             |
         port2 (FGT)
        192.168.10.1

         FortiGate
        port1: 10.0.0.1/30
                 |
            10.0.0.0/30
                 |
        G0/0: 10.0.0.2
          Cisco R1
        G0/1: 192.168.20.1
             |
      LAN-B (R1 side)
      192.168.20.0/24
```

---

### 1.2 BGP Topology

```
FortiGate (AS 65001)        Cisco R1 (AS 65002)
   port1: 10.0.0.1/30  ----  G0/0: 10.0.0.2/30

FGT advertises: 192.168.10.0/24
R1 advertises:  192.168.20.0/24
```

---

## 2. OSPF Configuration – FortiGate

### 2.1 Enable OSPF and Add Interfaces

```
config router ospf
    set router-id 1.1.1.1

    config area
        edit 0.0.0.0
        next
    end

    config ospf-interface
        edit "port1-to-R1"
            set interface "port1"
            set area 0.0.0.0
        next
        edit "port2-LAN-A"
            set interface "port2"
            set area 0.0.0.0
        next
    end

    config network
        edit 1
            set prefix 10.0.0.0 255.255.255.252
            set area 0.0.0.0
        next
        edit 2
            set prefix 192.168.10.0 255.255.255.0
            set area 0.0.0.0
        next
    end
end
```

### 2.2 FortiGate Interface IPs

```
config system interface
    edit "port1"
        set ip 10.0.0.1 255.255.255.252
        set allowaccess ping
    next
    edit "port2"
        set ip 192.168.10.1 255.255.255.0
    next
end
```

---

## 3. OSPF Configuration – Cisco Router (R1)

```
conf t
 hostname R1

 interface GigabitEthernet0/0
  ip address 10.0.0.2 255.255.255.252
  no shutdown

 interface GigabitEthernet0/1
  ip address 192.168.20.1 255.255.255.0
  no shutdown

 router ospf 1
  router-id 2.2.2.2
  network 10.0.0.0 0.0.0.3 area 0
  network 192.168.20.0 0.0.0.255 area 0
end
write memory
```

---

## 4. OSPF Verification

### On FortiGate

```
get router info ospf interface
get router info ospf neighbor
get router info routing-table ospf
execute ping 192.168.20.1
```

### On Cisco R1

```
show ip ospf neighbor
show ip route ospf
ping 192.168.10.1
```

---

## 5. BGP Configuration – FortiGate (AS 65001)

```
config router bgp
    set as 65001
    set router-id 1.1.1.1

    config neighbor
        edit "10.0.0.2"
            set remote-as 65002
        next
    end

    config network
        edit 1
            set prefix 192.168.10.0 255.255.255.0
        next
    end
end
```

---

## 6. BGP Configuration – Cisco Router (AS 65002)

```
conf t
 hostname R1

 interface GigabitEthernet0/0
  ip address 10.0.0.2 255.255.255.252
  no shutdown

 interface GigabitEthernet0/1
  ip address 192.168.20.1 255.255.255.0
  no shutdown

 router bgp 65002
  neighbor 10.0.0.1 remote-as 65001
  network 192.168.20.0 mask 255.255.255.0
end
write memory
```

---

## 7. BGP Verification

### On FortiGate

```
get router info bgp summary
get router info bgp neighbors
get router info routing-table bgp
```

### On Cisco R1

```
show ip bgp summary
show ip bgp
show ip route bgp
```

---

## 8. Troubleshooting Guide

### 8.1 OSPF Issues

* Area mismatch
* Wrong network statements
* Interface shutdown
* Mask mismatch

Commands:

```
FGT: get router info ospf neighbor
R1 : show ip ospf neighbor
```

### 8.2 BGP Issues

* Wrong AS number
* Wrong neighbor IP
* TCP port 179 blocked
* Prefix not in routing table

Commands:

```
FGT: get router info bgp summary
R1 : show ip bgp summary
```

---

## 9. Quick Reference Blocks

### FortiGate OSPF Minimal

```
config router ospf
 set router-id 1.1.1.1
 config network
  edit 1
   set prefix 10.0.0.0 255.255.255.252
   set area 0.0.0.0
  next
 end
end
```

### Cisco OSPF Minimal

```
router ospf 1
 router-id 2.2.2.2
 network 10.0.0.0 0.0.0.3 area 0
```

### FortiGate BGP Minimal

```
config router bgp
 set as 65001
 config neighbor
  edit 10.0.0.2
   set remote-as 65002
  next
 end
end
```

### Cisco BGP Minimal

```
router bgp 65002
 neighbor 10.0.0.1 remote-as 65001
 network 192.168.20.0 mask 255.255.255.0
```

---

