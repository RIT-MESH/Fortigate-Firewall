# Transparent Mode

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

> Example hostname used: `FGT-TP`
> Example management IP: `192.168.1.99/24`
> Gateway: `192.168.1.1` (router/firewall in same subnet)

### 3.1 Step 1 – Set hostname (optional but recommended)

```bash
config system global
    set hostname FGT-TP
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

### 5.1 Interface configuration

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

```bash
ip route 0.0.0.0 0.0.0.0 192.168.122.1
```

### 5.3 NAT configuration

```bash
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 interface FastEthernet1/0 overload
```

### 5.4 Verifying NAT

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
