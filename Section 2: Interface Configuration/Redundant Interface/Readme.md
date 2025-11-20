# Redundant Interface Lab README

<img width="677" height="317" alt="image" src="https://github.com/user-attachments/assets/45a2d435-f8d0-4da8-99e2-7e0bd52f3843" />

---

## 1. Topology Overview

```
PC1 ---- Switch ---- Port9
                      Port10
                         |
                     FortiGate
                         |
                      Port1
                         |
                     Admin PC (192.168.1.99/24)
```

### Subnets

* **PC LAN (Redundant Interface):** 192.168.10.0/24

  * Gateway: **192.168.10.1** (FortiGate redundant interface)
* **Admin Network:** 192.168.1.0/24

  * Admin PC: **192.168.1.99**

### Goal

Create a redundant interface on the FortiGate using **port9** and **port10** so that:

* Only one port is active at a time
* The other port remains on standby
* Failover happens instantly if the active link goes down

---

## 2. What a Redundant Interface Is

A redundant interface groups two physical interfaces for failover.

### Properties

* No LACP or aggregation protocol
* Only one link is active at any time
* Standby port activates when the active port fails
* Bandwidth equals one link (not combined)

This setup is useful when the switch does not support LACP or when a simple failover link is needed.

---

## 3. FortiGate CLI Configuration

```
config system interface
    edit "REDUNDANT"
        set vdom "root"
        set ip 192.168.10.1 255.255.255.0
        set allowaccess ping
        set type redundant
        set member "port9" "port10"
        set role lan
    next
end
```

### Explanation

* **type redundant** creates a failover logical interface
* **member ports:** port9 (active) and port10 (standby)
* **192.168.10.1** becomes the gateway for PC1
* **allowaccess ping** allows testing from PCs

---

## 4. GUI Configuration Steps

### Step 1: Log in

1. Connect Admin PC to Port1
2. Open browser
3. Go to:

```
https://192.168.1.99
```

4. Log in

### Step 2: Create Redundant Interface

1. Go to **Network > Interfaces**
2. Click **Create New → Interface**
3. Set:

   * **Name:** REDUNDANT
   * **Type:** Redundant
   * **Role:** LAN
   * **IP:** 192.168.10.1/24
   * **Allow Access:** Ping (and others if needed)
4. Under **Physical Interface Members** select:

   * port9
   * port10
5. Click **OK**

### Step 3: Verify Status

1. Go to **Network > Interfaces**
2. REDUNDANT should be **up**
3. port9 should show **active**
4. port10 should show **standby**

### Step 4: Firewall Policy (if you want traffic to go somewhere)

1. Go to **Policy & Objects → Firewall Policy**
2. Click **Create New**
3. Set:

   * Incoming: **REDUNDANT**
   * Outgoing: Port1 or WAN
   * Source: 192.168.10.0/24
   * Destination: desired target
4. Service: ALL
5. Action: ACCEPT
6. Click **OK**

### Step 5: Test Failover

1. Ping 192.168.10.1 from PC1
2. Unplug cable from **port9**
3. In GUI, port10 becomes **active**
4. Ping continues with little or no interruption
5. Reconnect port9 and observe failback

---

## 5. Switch Requirements

* Do **not** configure LACP or port-channel
* Both switch ports must be:

  * Access mode
  * VLAN 10

Example:

```
interface e0/6
  switchport mode access
  switchport access vlan 10

interface e0/7
  switchport mode access
  switchport access vlan 10
```

---

## 6. PC Configuration

### PC1

* IP: 192.168.10.x
* Mask: 255.255.255.0
* Gateway: 192.168.10.1

### Admin PC

* IP: 192.168.1.99
* Mask: 255.255.255.0
* Gateway: 192.168.1.1

---

## 7. Topology Diagram (Text Version)

```
                 Admin PC (192.168.1.99)
                         |
                       Port1
                       FGT
              +---------+---------+
              |                   |
        port9 (active)      port10 (standby)
              \                 /
                 \           /
                    Switch
                       |
                     PC1
```

## 8. Port1 and Admin-PC Connection Configuration

### 8.1 Port1 Configuration (CLI)

```
config system interface
    edit "port1"
        set ip 192.168.1.99 255.255.255.0
        set allowaccess http https ping ssh
        set role lan
    next
end
```

**Explanation:**

* FortiGate Port1 IP is **192.168.1.99/24** (management address)
* Admin PC will use this as its gateway and GUI access IP

---

### 8.2 Port1 Configuration (GUI)

1. Go to **Network > Interfaces**
2. Click **port1**
3. Set:

   * **IP Address:** `192.168.1.99/24`
   * **Role:** LAN
   * **Allow Access:** HTTPS, HTTP, PING, SSH
4. Click **OK**

---

### 8.3 Admin PC Configuration

* **IP:** 192.168.1.10
* **Mask:** 255.255.255.0
* **Gateway:** 192.168.1.99
* **DNS:** 8.8.8.8 or firewall

---

### 8.4 Connecting to the GUI from Admin PC

1. Open a web browser
2. Navigate to:

```
https://192.168.1.99
```

3. Log in to the FortiGate dashboard

---

### 8.5 Connectivity Testing

From Admin PC:

```
ping 192.168.1.99
```

If successful, the Admin PC can reach the FortiGate management interface.
