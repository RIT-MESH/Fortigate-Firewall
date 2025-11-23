# FortiGate Static Routing – Default Route and Network-Specific Routes

This document explains how static routing works on FortiGate using:

* Your topology
* GUI workflow 
* CLI configuration
* Flow diagrams and verification steps

Static routes are used when:

* The network is small
* Paths are predictable
* You need a default route toward the internet
* You must manually define how to reach remote networks

Unlike dynamic routing, **static routing does not learn routes automatically**.

---

# 1. Lab Topology

```
                 Internet Simulator (NAT1)
                       192.168.112.0/24
                             │
                             │ Port3
                  ┌────────────────────────┐
                  │       FortiGate        │
                  │ Port1: 41.141.1.0/24   │
                  │ Port3: 192.168.112.x   │
                  └──────────┬─────────────┘
                             │ 41.141.1.0/24
                             ▼
                    ┌─────────────────┐
                    │     R1 Router   │
                    │ f0/0: 41.141.1.1│
                    │ f1/0: 41.141.2.1│
                    └──────────┬──────┘
                               │
                               │ 41.141.2.0/24
                               ▼
                          ┌────────────┐
                          │    PC1     │
                          │ 41.141.2.x │
                          └────────────┘
```

---

# 2. Why Static Routes?

Static routes are manually defined paths that tell FortiGate:

* **Where to send internet traffic** (default route)
* **How to reach remote networks** beyond a next-hop router

They are simple, predictable, and preferred in small networks.

---

# 3. Creating a Default Route 

A default route directs all unknown destinations to a gateway.

**Transcript summary:**

* WAN gateway is **192.168.112.2** (Port3 network)
* This route is needed for internet access

<img width="740" height="274" alt="image" src="https://github.com/user-attachments/assets/b44c56dc-c5d3-4311-88a8-9119b9aff216" />


### CLI 

```
config router static
    edit 1
        set dst 0.0.0.0/0
        set device port3
        set gateway 192.168.112.2
    next
end
```

### GUI Workflow

1. Network → Static Routes → Create New
2. Destination: `0.0.0.0/0`
3. Interface: **Port3**
4. Gateway: **192.168.112.2**
5. Save

### Test

```
execute ping 8.8.8.8
```

If gateway is correct, internet will respond.

---

# 4. Creating a Route to a Remote Network

To reach **41.141.2.0/24**, traffic must go through Router R1’s LAN-facing interface:

* Gateway = **41.141.1.1** (R1 f0/0)
* Interface = **Port1** (connected to R1)

### CLI 

```
config router static
    edit 2
        set dst 41.141.2.0/24
        set device port1
        set gateway 41.141.1.1
    next
end
```

### GUI Workflow

1. Network → Static Routes → Create New
2. Destination: `41.141.2.0/24`
3. Interface: **Port1**
4. Gateway: **41.141.1.1**
5. Save

### Test

```
execute ping 41.141.2.1
```

If static route is correct, FortiGate will reach the remote network.

---

# 5. Static Route Processing Flow

```
Incoming Traffic
      │
      ▼
Does a more specific static route exist?
      │ Yes
      ▼
Forward using that route
      │ No
      ▼
Is there a default route?
      │ Yes
      ▼
Forward through gateway (0.0.0.0/0)
      │ No
      ▼
Drop packet (no route available)
```

FortiGate always prefers:

1. **Most specific** route (longest prefix)
2. Then fallback to **default route**

---

# 6. Verifying Static Routes

### FortiGate – Routing Table

```
get router info routing-table static
```

Example output from your file:

```
S* 0.0.0.0/0 [10/0] via 192.168.112.2, port3
S  41.141.2.0/24 [10/0] via 41.141.1.1, port1
```

### Ping Tests

```
execute ping 8.8.8.8
execute ping 41.141.2.1
```

### Sniffer for troubleshooting

```
diag sniff packet any 'icmp' 4
```

---

# 7. Common Static Route Mistakes

### ✔ Wrong gateway

Make sure the gateway is **inside the interface’s subnet**.

### ✔ Missing firewall policy

Routing works, but policy still must allow traffic.

### ✔ Wrong interface selection

Route will never work if it points to the wrong port.

### ✔ Overlapping subnets

May cause FortiGate to pick the wrong route.

---

# 8. Summary

* Static routes define manual paths to networks.
* Default route sends traffic toward the internet.
* More specific routes take priority.
* FortiGate requires correct gateway + interface.
* Routing and firewall policies work together.
