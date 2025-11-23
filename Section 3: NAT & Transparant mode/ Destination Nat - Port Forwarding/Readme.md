# FortiGate DNAT (VIP) – Publishing Internal Servers to the Internet

This document explains **DNAT (Destination NAT)** on FortiGate, combining:

* GUI workflow (from transcript)
* CLI configuration (from DNAT(VIP).txt)
* Flow diagrams
* Policy processing charts

DNAT is used to publish internal servers so external clients can reach them.
---

https://github.com/user-attachments/assets/8ee0cefc-5dae-49e2-8554-bf12a3152cb3



# 1. What Is DNAT?

DNAT maps a **public IP/port** to an **internal private server**.
This is how you publish:

* Web servers (HTTP/HTTPS)
* FTP servers
* Mail servers
* Any internal hosted application

When traffic hits the firewall’s public IP, FortiGate translates the destination to the internal server.

---

# 2. Topology Diagram (Lab Setup)

```
              Internet (Cloud)
                   │
            41.141.12.0/24
                   │
        ┌────────────────────┐
        │      Router R1     │
        │ f1/0: 41.141.12.1  │
        │ f0/0: 41.141.1.2   │
        └──────────┬─────────┘
                   │ 41.141.1.0/24
        ┌──────────┴────────────┐
        │       FortiGate       │
        │ port2 (WAN): 41.141.1.1
        │ port1 (LAN): 192.168.1.1
        └──────────┬────────────┘
                   │ 192.168.1.0/24
        ┌──────────┴────────────┐
        │        Switch         │
        └───────┬───────────────┘
         Firefox Client       PC1
```

This is the DNAT lab topology used for publishing an internal web server.

---

# 2. DNAT Flow Diagram

```
      Internet Client
      (41.141.1.x)
            │
            ▼
   ┌─────────────────┐
   │  Public IP on   │
   │ FortiGate WAN   │
   │   41.141.1.2    │
   └───────┬─────────┘
           │ Incoming HTTP
           ▼
   ┌─────────────────┐
   │   VIP Object    │
   │ WEB-SERVER      │
   │ maps to         │
   │ 192.168.1.20    │
   └───────┬─────────┘
           │ DNAT Translation
           ▼
   ┌─────────────────┐
   │ Internal Server │
   │ 192.168.1.20    │
   │ Apache Web App  │
   └─────────────────┘
```

---

# 2.1 DNAT Traffic Flow (Step-by-Step Based on Topology)

1. External client sends HTTP request to **41.141.1.1** (FortiGate WAN IP).
2. Traffic reaches **R1**, which routes it toward the FortiGate.
3. FortiGate receives the packet on **port2 (WAN)**.
4. VIP matches the destination IP and rewrites it to **192.168.1.20**.
5. Packet is forwarded to LAN via **port1**.
6. Internal Apache server responds.
7. FortiGate performs reverse DNAT (private → public).
8. Response goes back through **R1 → Cloud (Internet)**.

---

# 3. VIP Processing Flowchart

```
Incoming packet on WAN IP
        │
        ▼
Is destination matching a VIP?
        │Yes
        ▼
Rewrite Destination IP → Mapped IP
Rewrite Destination Port → Mapped Port (if port forwarding)
        │
        ▼
Apply inbound firewall policy
        │
        ▼
Forward packet to internal server
```
<img width="959" height="293" alt="image" src="https://github.com/user-attachments/assets/66615617-f835-4c3d-9c66-d5603d59872d" />

---

# 4. GUI Workflow 

### Step 1 — Verify WAN interface has a static public IP

Network → Interfaces → Select WAN → Confirm IP e.g. **41.141.1.2**

### Step 2 — Create VIP (Virtual IP)

Policy & Objects → Virtual IPs → Create New

* Name: *WEB-SERVER*
* Interface: WAN (port2)
* External IP: **41.141.1.2**
* Mapped IP: **192.168.1.20** (web server)
* Service: HTTP

### Step 3 — Create inbound policy

Policy & Objects → Firewall Policy → Create New

* Incoming Interface: WAN
* Outgoing Interface: LAN
* Source: all
* Destination: **WEB-SERVER VIP object**
* Service: HTTP
* NAT: Disabled

### Step 4 — Test

From external client:

```
http://41.141.1.2
```

External user reaches internal Apache server.

---

# 5. Port Forwarding Example

To publish internal port 80 using external port 8080:

* Enable **port forwarding** in VIP
* External port: 8080
* Mapped port: 80

Client uses:

```
http://41.141.1.2:8080
```

---

# 6. CLI Configuration 

## 6.1 VIP Creation

```
config firewall vip
    edit "WEB-SERVER"
        set extip 41.141.1.2
        set mappedip "192.168.1.20"
        set extintf "any"
        set color 6
    next
end
```

## 6.2 DNAT Firewall Policy

```
config firewall policy
    edit 2
        set name "WEB SERVER"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "web"
        set action accept
        set schedule "always"
        set service "HTTP"
        set logtraffic all
    next
end
```

## 6.3 Port Forwarding

```
config firewall vip
    edit "WEB-SERVER"
        set extip 41.141.1.2
        set mappedip "192.168.1.20"
        set extintf "any"
        set portforward enable
        set extport 8080
        set mappedport 80
    next
end
```

---

# 7. Verification

### CLI

```
diag sys session list | grep 192.168.1.20
```

Shows NAT-rewritten sessions.

### GUI

Dashboard → FortiView Sessions

* See external client IP
* See WAN → LAN mapping
* Confirm VIP used

---

# 8. Summary

* DNAT publishes internal servers to the internet.
* VIP defines *public → private* mapping.
* Firewall policy controls who can access the server.
* Port forwarding allows custom external ports.

