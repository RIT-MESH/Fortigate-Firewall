# FortiGate Transparent Mode 
## 1. Overview

This lab demonstrates how to:

* Understand FortiGate operation modes:

  * **NAT mode** (default)
  * **Transparent (TP) mode**
* Convert a FortiGate from NAT mode to Transparent mode
* Configure management access in Transparent mode
* Verify the FortiGate mode from CLI
* Integrate the FortiGate in Transparent mode with a Cisco router doing NAT

---

## 2. Theory

### 2.1 FortiGate Operation Modes

FortiGate supports two main operation modes:

#### NAT Mode

* **Default mode** when a new FortiGate is installed.
* FortiGate behaves like a **router**:

  * Performs **routing**, **NAT**, **VPN**, and other L3/L4 functions.
* **Each interface is in a different subnet**.
* Commonly used when FortiGate is the **edge firewall** connected directly to the ISP.

#### Transparent (TP) Mode

* Used when FortiGate is placed **behind an existing router or firewall**.
* FortiGate behaves like a **Layer 2 bridge/switch**:

  * It forwards traffic at **Layer 2** but can still inspect and filter traffic.
* **All data interfaces share the same subnet**.
* You **cannot assign normal IP addresses to data interfaces**.
* You **must assign a management IP** (called `manageip`) for admin access (GUI/SSH/etc.).

Typical use cases:

* Inline security device between core switch and existing firewall.
* Inserting FortiGate into a production network **without changing IP addressing**.
* Doing inspection/policies in environments where routing must stay unchanged.

---

## 3. FortiGate – Transparent Mode Configuration
<img width="732" height="437" alt="image" src="https://github.com/user-attachments/assets/9566d0f5-5929-40f0-9a8a-00b2af0caa96" />

> Example hostname used: `Ritz`
> Example management IP: `192.168.1.99/24`
> Gateway: `192.168.1.1` (router/firewall in same subnet)

### 3.1 Step 1 – Set hostname (optional but recommended)

```bash
config system global
    set hostname Ritz
end
```

### 3.2 Step 2 – Disable FortiLink (required before switching to TP mode)

```bash
config system interface
    edit fortilink
        set fortilink disable
    next
end
```

> If `fortilink` is not present in your config, you can skip this step.

### 3.3 Step 3 – Change operation mode to Transparent and set management IP

```bash
config system settings
    set opmode transparent
    set manageip 192.168.1.99/24
    set gateway 192.168.1.1
end
```

### 3.4 Step 4 – Configure interface access (HTTP/SSH/ping)

```bash
config system interface
    edit port2
        set allowaccess http ping ssh
    next
end
```

---

## 4. Verifying FortiGate Operation Mode

```bash
get system status
```

Check for:

```
Hostname: FGT-TP
Operation Mode: Transparent
```

---

## 5. Cisco Router Configuration (Edge NAT Router)

### What this section is doing (Theory)

The Cisco router is acting as the **main Layer 3 gateway and NAT device** for the network. Because the FortiGate is in Transparent mode, it does not route or NAT; it only inspects and filters traffic while bridging it at Layer 2. The router is responsible for:

* Providing the **default gateway** for the LAN (192.168.1.1)
* Performing **NAT overload (PAT)** so LAN devices can reach external networks
* Handling the **routing** between the LAN and the upstream provider/network

In Transparent mode, the FortiGate sits **inline between the LAN and this router**, inspecting traffic without changing IPs. This keeps the router fully responsible for all L3/NAT functions while allowing the FortiGate to enforce security policies.

Below is the configuration that enables the router to perform these tasks.

### 5.1 Interface configuration

**What this step does:**
This configures the router’s physical interfaces. FastEthernet0/0 becomes the LAN gateway for devices inside the network, and FastEthernet1/0 becomes the upstream/WAN-facing interface. Marking them as `ip nat inside` and `ip nat outside` prepares the router for NAT.

```bash
conf t

interface FastEthernet0/0
    ip address 192.168.1.1 255.255.255.0
    no shutdown
    ip nat inside

interface FastEthernet1/0
    ip address 192.168.122.240 255.255.255.0
    no shutdown
    ip nat outside
```

### 5.2 Default route

**What this step does:**
This tells the router where to send all traffic that is not destined for the local LAN. In this lab, all unknown traffic is forwarded to the upstream next-hop (192.168.122.1). This enables internet or external network access for the LAN.

```bash
ip route 0.0.0.0 0.0.0.0 192.168.122.1
```

### 5.3 NAT configuration

**What this step does:**
This creates a simple NAT overload setup so that devices in the 192.168.1.0/24 subnet can share a single public/upstream IP. The access list identifies the internal LAN, and the NAT rule translates their source addresses using the outside interface IP. This is essential because the FortiGate in Transparent mode does not perform NAT.

```bash
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 interface FastEthernet1/0 overload
```

### 5.4 Verifying NAT

**What this step does:**
This shows active NAT translations on the router. It confirms whether LAN traffic is being translated correctly and helps verify that end‑to‑end connectivity is working through the FortiGate and out to the upstream network.

```bash
show ip nat translations
```

---

## 6. Traffic Flow Summary

1. LAN host sends traffic to 192.168.1.1 (Cisco router).
2. Traffic passes through FortiGate (Transparent mode).
3. Router performs NAT.
4. Traffic goes upstream.
5. Return traffic passes the FortiGate again.

---

## 7. Troubleshooting Commands

### 7.1 On FortiGate

```bash
get system status
show system interface
execute ping 192.168.1.1
execute ping 8.8.8.8
```

### 7.2 On Cisco Router

```bash
show ip interface brief
show ip nat translations
show ip nat statistics
show ip route
ping 192.168.1.99
ping 8.8.8.8
```

---

## 8. Notes & Best Practices

* Plan management IP and gateway before enabling Transparent mode.
* Policies are still required in TP mode.
* Logging and UTM functions work normally.
* Take backups before and after changes.
