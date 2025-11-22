# FortiGate NAT – IP Pools (GUI + CLI)

This document covers **only NAT IP Pools** on FortiGate. It combines:

* GUI workflow (from your transcript)
* CLI configuration (from your NAT.txt file)
* Clear explanations for all four NAT types
* Ping/session verification notes

This is NOT the full NAT course. Only IP Pools.

---

# 1. What Are IP Pools?

IP Pools let the FortiGate translate internal IP addresses to one or more external IPs.
They define **which public IP(s)** will be used when NAT is enabled in a firewall policy.

FortiGate supports **four** IP Pool modes:

1. **Overload (PAT)**
2. **One-to-One NAT**
3. **Fixed Port Range (FPR)**
4. **Port Block Allocation (PBA)**

Each type affects:

* How many internal clients can NAT
* How many external IPs are needed
* How ports are assigned

---

# 2. NAT IP Pool Types Explained

## 2.1 Overload (PAT)

This is the most common NAT type.

* One public IP can serve **many internal clients**.
* Uses **Port Address Translation**.
* Internal clients get different source ports, but all share the same public IP.

### Use case:

General internet access when you only have **one public IP**.

---

## 2.2 One-To-One NAT

This is **static NAT**.

* Each internal IP is mapped to **exactly one** external IP.
* No port translation.

### Requirements:

If you have 5 internal clients, you need **5 public IPs**.

### Use case:

Servers that must keep the same public IP.

---

## 2.3 Fixed Port Range (FPR)

A PAT type where you define:

* Which internal IP range will use the pool
* Which external IP(s) they can translate to

### Use case:

You want to control **which internal hosts** are allowed to go out using specific public IPs.

---

## 2.4 Port Block Allocation (PBA)

Also a PAT method, but you control **port consumption**.

* You define how many ports each user gets
* You define how many blocks each user can use

### Use case:

Large networks where you want to prevent a single client from consuming all NAT ports.

---

# 3. NAT Flow Diagram (High‑Level)

```
           ┌──────────────────────┐
           │   Internal Clients   │
           │  (192.168.x.x LAN)   │
           └──────────┬───────────┘
                      │
                      ▼
           ┌──────────────────────┐
           │   FortiGate (TP)     │
           │  Policy + IP Pool    │
           └──────────┬───────────┘
                      │ NAT Applied Here
                      ▼
           ┌──────────────────────┐
           │   Public IP Range    │
           │ (Used by IP Pools)   │
           └──────────┬───────────┘
                      │
                      ▼
           ┌──────────────────────┐
           │   Upstream Router    │
           │  (Default Gateway)   │
           └──────────┬───────────┘
                      │
                      ▼
           ┌──────────────────────┐
           │        Internet       │
           └──────────────────────┘
```

This shows **where** NAT happens. The FortiGate translates the source IP using the IP Pool before forwarding traffic to the upstream router.

---

# 3.1 Policy Processing Flowchart

```
                 Incoming Packet
                        │
                        ▼
             ┌────────────────────┐
             │  Does a policy     │
             │      match?        │
             └───────┬────────────┘
                     │Yes
                     ▼
             ┌────────────────────┐
             │   Is NAT enabled   │
             │   in this policy?  │
             └───────┬────────────┘
                     │Yes
                     ▼
          ┌──────────────────────────────┐
          │ Apply IP Pool (Overload,     │
          │ One‑to‑One, FPR, PBA)        │
          └───────────┬──────────────────┘
                      ▼
             ┌────────────────────┐
             │ Forward to WAN     │
             │   (Next hop)       │
             └────────────────────┘
```


---

# 3. GUI Workflow 

### Overload (PAT)

1. Go to **Policy & Objects → IP Pools**
2. Click **Create New**
3. Choose **Overload**
4. Enter:

   * External Start IP
   * External End IP (same as start when using 1 IP)
5. Save
6. Go to **Firewall Policy**
7. Edit the internet policy
8. Under **NAT**, choose **Dynamic IP Pool**
9. Select the pool you created

### One-To-One NAT

1. Same steps, but choose **One-to-One**
2. Enter full public IP range
3. Map in the firewall policy

### Fixed Port Range

1. Choose **Fixed Port Range**
2. Enter:

   * External IP range
   * Internal source IP range
3. Apply to firewall policy
4. Only clients inside the internal range get NATed

### Port Block Allocation

1. Choose **Port Block Allocation**
2. Configure:

   * Block size (ports per block)
   * Blocks per user
3. Apply to firewall policy

---

# 4. NAT Type Comparison Diagram

```
+-------------------+--------------------------+---------------------------+
| NAT Type          | External IP Requirement  | Port Behavior             |
+-------------------+--------------------------+---------------------------+
| Overload (PAT)    | 1 IP for many users      | Many ports per user       |
| One-to-One        | 1 IP per internal host   | No port translation       |
| Fixed Port Range  | One IP, restricted users | Port ranges allowed       |
| Port Block Alloc. | One IP, multi users      | Controlled port blocks    |
+-------------------+--------------------------+---------------------------+
```

---

# 4. CLI Configurations 

Below is the full CLI version. 

## 4.1 Overload (PAT)

```
config firewall ippool
    edit "Overload"
        set startip 192.168.122.240
        set endip 192.168.122.240
    next
end

config firewall policy
    edit 1
        set name "INTERNET"
        set srcintf "port1"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set logtraffic all
        set ippool enable
        set poolname "Overload"
        set nat enable
    next
end
```

---

## 4.2 One-To-One NAT

```
config firewall ippool
    edit "One2One"
        set type one-to-one
        set startip 192.168.122.240
        set endip 192.168.122.241
    next
end

config firewall policy
    edit 3
        set name "INTERNET"
        set srcintf "port1"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set ippool enable
        set poolname "One2One"
        set nat enable
    next
end
```

---

## 4.3 Fixed Port Range (FPR)

```
config firewall ippool
    edit "FPR"
        set type fixed-port-range
        set startip 192.168.122.240
        set endip 192.168.122.240
        set source-startip 192.168.10.99
        set source-endip 192.168.10.100
    next
end

config firewall policy
    edit 3
        set name "INTERNET"
        set srcintf "port1"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set ippool enable
        set poolname "FPR"
        set nat enable
    next
end
```

---

## 4.4 Port Block Allocation (PBA)

```
config firewall ippool
    edit "PBA"
        set type fixed-port-range
        set startip 192.168.122.240
        set endip 192.168.122.240
        set num-blocks-per-user 8
        set block-size 128
    next
end

config firewall policy
    edit 3
        set name "INTERNET"
        set srcintf "port1"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set ippool enable
        set poolname "PBA"
        set nat enable
    next
end
```

---

# 5. Verification Steps

## CLI

```
get system session list | grep <protocol/ip>
```

Look for:

* Source IP
* Translated Source IP
* Ports used

## GUI

**Dashboard → FortiView Sessions**
You will see:

* Internal IP
* Public IP it was translated to
* Ports
* Destination

---

# 6. Summary

* Overload = Most common, uses ports to multiplex many clients into one IP.
* One-to-One = Each client needs its own public IP.
* Fixed Port Range = NAT only applies to a defined internal range.
* Port Block Allocation = Limits how many ports each client can use.

