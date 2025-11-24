# FortiGate Policy‑Based Routing (PBR) – Multi‑WAN Traffic Steering

This document explains **Policy‑Based Routing (PBR)** on FortiGate using:

* Your full transcript (GUI + CLI)
* Two‑WAN topology
* Policy flow diagrams
* CLI example from *Policy_Based_Routing.txt*

PBR lets you route traffic **based on conditions**, not only on routing table decisions.

---

# 1. What Is Policy‑Based Routing?

Normally, FortiGate forwards traffic based on its **routing table**.

But PBR lets you override this by matching:

* Source IP / user
* Destination IP
* Service / port
* Protocol
* Incoming interface

**PBR always takes precedence over the routing table**.

---

# 2. Lab Topology

```
        Firefox1              Firefox2
          │                       │
          │                       │
        ┌─────────Switch───────────┐
        │  LAN 172.16.10.0/24      │
        └──────────────┬───────────┘
                       │ Port3 (LAN)
                 ┌─────────────────────┐
                 │     FortiGate      │
                 │ Port1 → WAN1        │
                 │ Port2 → WAN2        │
                 └────────┬────────────┘
                          │
          ┌───────────────┴───────────────┐
          │                               │
     NAT1 Cloud (WAN1 ISP)        Cloud1 (WAN2 ISP)
     192.168.122.0/24             192.168.1.0/24
```

**Goal:**

* WAN1 (Port1) = Primary internet
* WAN2 (Port2) = Backup internet
* **BUT:** Only traffic from Firefox1 is forced through WAN2 using PBR.

---

# 3. Multi‑WAN Static Routes Setup

Two default routes:

* Route #1 → WAN1 (priority 0)
* Route #2 → WAN2 (priority 10)

The lower priority wins → WAN1 becomes primary.

FortiGate Routing Monitor:

```
WAN1 → Primary
WAN2 → Secondary
```

---

# 4. Firewall Policies for WAN Outbound Traffic

Two firewall policies:

```
LAN → WAN1 (allow, NAT enabled)
LAN → WAN2 (allow, NAT enabled)
```

Both must exist, because: **PBR only decides direction; firewall policy decides permission.**

---

<img width="818" height="324" alt="image" src="https://github.com/user-attachments/assets/317af9de-b387-4e92-a299-03a816aa4a78" />

# 5. Creating a Policy‑Based Route (GUI)

### Step‑by‑Step

1. Go to **Network → Policy Routes**
2. Create New
3. Incoming interface → **LAN (port3)**
4. Source → IP of Firefox1 (example: 172.16.10.20)
5. Destination → all
6. Protocol → TCP or ALL
7. **Outgoing interface → WAN2**
8. Gateway → WAN2 gateway (ex: 192.168.1.1)
9. Save

Now only Firefox1 uses WAN2.
All other devices still use WAN1.

---

# 6. CLI Configuration (from uploaded file)

```
config router policy
    edit 1
        set input-device "port3"
        set src "172.16.10.20/255.255.255.255"
        set protocol 6
        set start-port 80
        set end-port 80
        set gateway 192.168.1.1
        set output-device "port2"
    next
end
```

This PBR rule says:

* Traffic coming from **172.16.10.20**
* Using **TCP port 80**
* Should exit through **port2 (WAN2)**
* Using gateway **192.168.1.1**

---

# 7. How PBR Overrides the Routing Table

```
Incoming packet from LAN
          │
          ▼
Does a policy‑route match?
          │ Yes
          ▼
Forward to WAN2 (ignore routing table)
          │ No
          ▼
Use normal routing table (WAN1 primary)
```

---

# 8. Monitoring & Verification

### 8.1 FortiView Sessions

Filter by source IP of Firefox1.
You should see:

```
Outbound Interface: port2 (WAN2)
```

### 8.2 Logs → Forward Traffic

Look for entries showing destination interface **WAN2**.

### 8.3 CLI

```
diag sys session filter src 172.16.10.20
diag sys session list
```

### 8.4 PC Tests

```
ping 8.8.8.8
curl http://neverssl.com
```

Traffic to port 80 should go through WAN2.

---

# 9. Advanced PBR Options

You can route based on:

* Protocol (TCP/UDP/ICMP)
* Destination IP ranges
* Services
* Device MAC (using objects)
* Internet Service Database (ISDB)

Examples:

* Send **Microsoft Update** traffic through WAN2
* Send **Office365** through WAN1
* Send **port 443** traffic through WAN1 but **port 80** traffic through WAN2

---

# 10. Summary

* PBR allows routing based on **criteria**, not routing table.
* PBR takes priority over static/dynamic routes.
* Routing table still determines all other traffic.
* Multi‑WAN failover + PBR gives flexible traffic control.
