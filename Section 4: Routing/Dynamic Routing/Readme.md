# FortiGate Dynamic Routing – RIP, OSPF, and BGP

This document explains how to configure **Dynamic Routing on FortiGate**, using:

* Your full transcript (GUI + CLI explanation)
* Your topology diagram
* Cisco router configuration
* All FortiGate CLI examples from the uploaded file (Dynamic Routing.txt)

This guide covers: **RIP, OSPF, and BGP** routing between FortiGate and a Cisco router.

---



https://github.com/user-attachments/assets/3257c3d5-1b31-4d1f-9118-d126e2a26b2d



https://github.com/user-attachments/assets/ce894c8d-a3e1-4089-b5b9-a1b1f6a247bb

# 1. Lab Topology

```
                         Internet Simulator (NAT1)
                               192.168.112.0/24
                                     │
                                     │
                        ┌─────────────────────────┐
                        │       FortiGate         │
                        │  Port3: 192.168.112.x   │
                        │  Port1: 41.141.1.0/24   │
                        └───────────┬────────────┘
                                    │
                                    │ 41.141.1.0/24
                                    ▼
                         ┌───────────────────┐
                         │       R1 Router   │
                         │ f0/0: 41.141.1.1  │
                         │ f1/0: 41.141.2.1  │
                         └───────────┬──────┘
                                     │
                                     │ 41.141.2.0/24
                                     ▼
                         ┌───────────────────┐
                         │       PC1         │
                         │ e0: 41.141.2.x    │
                         └───────────────────┘
```

This topology is used to demonstrate **dynamic routing protocols**:

* RIP v2
* OSPFv2
* BGP

---
<img width="789" height="276" alt="image" src="https://github.com/user-attachments/assets/29b081ab-6a9f-4da4-a1c5-28618f4fc921" />


# 2. Before Starting – Router Interface Setup

Cisco Router R1 must be configured first:

```
conf t
interface f0/0
 ip address 41.141.1.1 255.255.255.0
 no shutdown
!
interface f1/0
 ip address 41.141.2.1 255.255.255.0
 no shutdown
end
wr mem
```

This provides:

* A **shared network** between FortiGate and R1 (41.141.1.0/24)
* A **remote network** behind R1 (41.141.2.0/24)

---

# 3. RIP Configuration

## 3.1 RIP Theory

RIP is a simple distance‑vector routing protocol.

* Uses hop count as metric
* Max 15 hops
* Slow convergence
* Mostly used in labs

FortiGate RIP supports:

* Version 1 or 2 (use **RIP v2**)
* Network advertisement
* Route redistribution

---

## 3.2 FortiGate RIP Configuration

(From your uploaded file)

```
config router rip
    config network
        edit 1
            set prefix 41.141.1.0 255.255.255.0
        next
    end
    config redistribute "connected"
        set status enable
    end
    config interface
        edit "port3"
            set receive-version 2
            set send-version 2
        next
    end
end
```

---

## 3.3 Cisco Router RIP Configuration

```
router rip
 version 2
 network 41.141.1.0
 network 41.141.2.0
 redistribute connected
```

---

## 3.4 Verifying RIP

FortiGate:

```
get router info routing-table all
```

Cisco:

```
show ip route rip
```

FortiGate GUI:

* Dashboard → Routing Monitor → Look for type **RIP**

---

# 4. OSPF Configuration

## 4.1 OSPF Theory

OSPF is a fast, link‑state interior gateway protocol.

* Uses areas
* Builds LSDB (link-state database)
* Forms adjacencies using **Hello** packets
* Adapts quickly to changes

Requirements:

* Matching area IDs
* Matching networks
* Unique router IDs

---

## 4.2 FortiGate OSPF Configuration

```
config router ospf
    set router-id 1.1.1.1
    config area
        edit 0.0.0.0
        next
    end
    config network
        edit 1
            set prefix 41.141.1.0 255.255.255.0
        next
    end
    config redistribute "connected"
        set status enable
    end
end
```

---

## 4.3 Cisco Router OSPF Configuration

```
router ospf 1
 router-id 2.2.2.2
 network 41.141.1.0 0.0.0.255 area 0
 network 41.141.2.0 0.0.0.255 area 0
 redistribute connected
```

---

## 4.4 Verify OSPF

FortiGate:

```
get router info ospf neighbor
```

Cisco:

```
show ip ospf neighbor
show ip route ospf
```

FortiGate GUI:

* Routing Monitor → Type **OSPF** shows learned routes

---

# 5. BGP Configuration

## 5.1 BGP Theory

BGP is an exterior gateway protocol.

* Uses AS numbers
* Forms TCP sessions (port 179)
* Exchanges NLRI (Network Layer Reachability Information)
* Used for ISP, enterprise edge, multihoming

Requirements:

* Matching **AS numbers** for iBGP
* Neighbor IP must be reachable
* Must advertise networks explicitly

---

## 5.2 FortiGate BGP Configuration

```
config router bgp
    set as 10
    config neighbor
        edit "41.141.1.1"
            set remote-as 10
        next
    end
    config network
        edit 1
            set prefix 41.141.1.0 255.255.255.0
        next
    end
    config redistribute "connected"
        set status enable
    end
end
```

---

## 5.3 Cisco Router BGP Configuration

```
router bgp 10
 neighbor 41.141.1.2 remote-as 10
 network 41.141.1.0 mask 255.255.255.0
 network 41.141.2.0 mask 255.255.255.0
 redistribute connected
```

When the neighbor comes up:

```
%BGP-5-ADJCHANGE: neighbor 41.141.1.2 Up
```

---

## 5.4 Verify BGP

FortiGate:

```
get router info bgp summary
```

Cisco:

```
show ip bgp summary
show ip route bgp
```

FortiGate GUI:

* Routing Monitor → Type **BGP** shows learned routes

---

# 6. Troubleshooting Dynamic Routing

## Useful Commands

### FortiGate

```
diag sniff packet any 'host 41.141.1.1' 4
get router info routing-table all
diag debug enable
diag debug application rip -1
diag debug application ospf -1
diag debug application bgp -1
```

### Cisco

```
show ip protocols
show ip route
show ip ospf interface
show ip bgp summary
```

---

# 7. Summary

* RIP is simple but slow; useful for testing
* OSPF is fast and scalable
* BGP is used for enterprise edge and ISP scenarios
* FortiGate integrates seamlessly with Cisco routers
* Routing Monitor helps visualize learned routes
