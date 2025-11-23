# FortiGate Firewall Policies – Allowing or Denying Traffic Between Interfaces

This document explains how FortiGate firewall policies work. It combines:

* GUI workflow 
* CLI configuration
* Topology diagram
* Traffic flow logic
* Best practices

A policy determines **what traffic is allowed or denied** when it enters one interface and exits another.
By default, FortiGate **denies all traffic** unless a policy explicitly allows it.

---

# 1. Lab Topology (LAN → DMZ)


https://github.com/user-attachments/assets/3274c445-8f3c-4c9d-852c-58b1f4c0bf42

https://github.com/user-attachments/assets/0b0f1bf3-5a6b-45ed-bdfd-cc9dac0e3a8b




```
                     Internet (NAT Cloud)
                           │
                           ▼
                      FortiGate
                 ┌───────┬────────┬───────┐
                 │ Port2 │ Port3  │ Port1 │
                 └───┬───┘        └───┬───┘
                     │                │
            LAN 192.168.1.0/24      DMZ 172.16.1.0/24
                     │                │
            Firefox Client           Web Server
            PC1                     (Apache)
```

**Goal:** Allow LAN users to access the Web Server inside the DMZ.

By default, LAN cannot reach DMZ because FortiGate does not allow inter‑interface traffic unless a policy is created.

---

# 2. Policy Processing Flow

```
Incoming traffic → Match policy? → Action → Forward or Deny
```

Detailed flow:

```
Packet arrives on Source Interface
        │
        ▼
Is there a matching policy for:
- srcintf
- dstintf
- source IP
- destination IP
- service (port)?
        │
   ┌────┴─────┐
   │ Yes      │ No
   ▼          ▼
Allow        Deny (implicit deny)
        │
Rewrite NAT if enabled
        │
Forward to destination
```

If no policy matches, the packet is silently dropped.

---
<img width="739" height="338" alt="image" src="https://github.com/user-attachments/assets/2927f7f9-548c-4b8f-8ca6-a494ed8cbe54" />

# 3. GUI Workflow for Creating a LAN → DMZ Web Policy

### Step 1 — Identify your interfaces

* LAN = port2
* DMZ = port1

### Step 2 — Identify destination object (web server)

Example:

```
WEB_SERVER = 172.16.1.20
```

### Step 3 — Create the Policy

Policy & Objects → Firewall Policy → Create New

* **Name:** LAN_TO_DMZ_WEB
* **Incoming Interface:** LAN (port2)
* **Outgoing Interface:** DMZ (port1)
* **Source:** all (or LAN address object)
* **Destination:** WEB_SERVER address object
* **Service:** HTTP (and PING if needed for testing)
* **NAT:** Disabled (we’re routing, not translating)
* **Logging:** Enabled

### Step 4 — Test

From LAN client:

* Access web page via browser
* Ping the server (if PING was allowed)

If the policy is correctly configured, traffic flows.

---

# 4. CLI Configuration (From Policy.txt)

This is the exact CLI structure from your uploaded file.

```
config firewall policy
    edit 1
        set name "LAN_TO_DMZ_WEB"
        set srcintf "port1"
        set dstintf "port2"
        set srcaddr "all"
        set dstaddr "WEB_SERVER"
        set action accept
        set schedule "always"
        set service "HTTP" "PING"
        set logtraffic all
    next
end
```

**Note:** Adjust `srcintf` and `dstintf` based on your topology.

---

# 5. How to Verify Your Policy

### CLI

```
diag sys session list | grep 172.16.1.20
```

Shows active sessions hitting the web server.

### GUI

Dashboard → FortiView → Sessions

* Look for source IP (client)
* Destination IP (web server)
* Interface pair LAN → DMZ

---

# 6. Best Practices

### ✔ Use address objects instead of “all”

This limits exposure and increases control.

### ✔ Order policies correctly

FortiGate reads them **top → bottom**.

### ✔ Enable logging for troubleshooting

Especially when learning.

### ✔ Allow only required services

Use HTTP/HTTPS, not ALL.

### ✔ Never use NAT for internal zone-to-zone policies

These are routed internally.

---

# 7. Summary

* FortiGate blocks all traffic unless a policy permits it.
* Policies must match interface, IP, and service.
* LAN → DMZ traffic requires an explicit firewall rule.
* Use objects and limited services to stay secure.

