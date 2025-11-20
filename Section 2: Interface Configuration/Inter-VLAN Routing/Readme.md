# Inter-VLAN Routing on FortiGate

This document explains how to configure Inter-VLAN Routing (IVR) on a FortiGate firewall using the VLAN interfaces and firewall policies found in the provided configuration file.

## 1. Overview

Three VLANs are created on the FortiGate. Each VLAN has its own gateway IP and is carried over a trunk link to a switch. The goal is to let devices in these VLANs communicate with each other.
# Network Topology
<img width="834" height="500" alt="image" src="https://github.com/user-attachments/assets/524ab862-d0a1-4680-ab48-c415b6c12d76" />

```
                +--------------------+
                |      Internet      |
                +---------+----------+
                          |
                     +----+----+
                     | FortiGate|
                     |          |
           port1.10  |  vlan10  |
           port1.20  |  VLAN20  |
           port1.30  |  VLAN30  |
                     +----+----+
                          |
                 Trunk (VLAN 10,20,30)
                          |
                    +-----+------+
                    |   Switch   |
                    +-----+------+
                      |    |    |
                 VLAN10 VLAN20 VLAN30
                      |    |    |
                    PCs in each VLAN
```

### VLANs in this setup

* VLAN10
* VLAN20
* VLAN30

Inter-VLAN routing happens on the FortiGate through either a firewall policy or a VLAN zone.

## 2. Requirements

Before starting, make sure you have:

* A FortiGate with VLAN interfaces created and assigned to a physical port
* A switch with a trunk toward the FortiGate that carries VLANs 10, 20 and 30
* Access ports for end devices in the correct VLANs
* "Multiple Interface Policy" enabled

  * System > Feature Visibility > enable "Multiple Interface Policy"

## 3. Option 1: IVR using a Firewall Policy

This option uses a single policy that allows traffic between all VLANs.

### Configuration (CLI)

```bash
config firewall policy
    edit 1
        set name "IVR"
        set srcintf "vlan10" "VLAN20" "VLAN30"
        set dstintf "vlan10" "VLAN20" "VLAN30"
        set srcaddr "VLAN10" "VLAN20" "VLAN30"
        set dstaddr "VLAN10" "VLAN20" "VLAN30"
        set action accept
        set schedule "always"
        set service "ALL"
        set logtraffic all
        set nat enable
    next
end
```

### What this does

* Allows any VLAN to reach any other VLAN
* Enables logging
* NAT can be optional. You can disable it if you don’t want NAT inside your LAN

## 4. Option 2: IVR using a VLAN Zone

This option groups the VLAN interfaces into a zone. Any interface inside the zone is allowed to communicate with the others automatically.

### Configuration (CLI)

```bash
config system zone
    edit "VLANS"
        set intrazone allow
        set interface "vlan10" "VLAN20" "VLAN30"
    next
end
```

### What this does

* Creates a zone called VLANS
* Adds VLAN10, VLAN20 and VLAN30 to the zone
* Allows communication between them without extra policies

## 5. Verification

After completing the setup, test connectivity.

### Check interface status

```bash
get system interface
```

### Ping from FortiGate

```bash
execute ping <PC_in_VLAN10>
execute ping <PC_in_VLAN20>
execute ping <PC_in_VLAN30>
```

### Ping between PCs

Test communication between VLANs from each client.

### Check logs

Go to: Log & Report > Forward Traffic
Filter by the policy name "IVR" if using Option 1.

## 6. Troubleshooting

If inter-VLAN communication is not working, check the following.

### 1. VLAN interface status

```bash
get system interface
```

Make sure interfaces are up and have correct IPs.

### 2. Gateway settings on PCs

Verify PCs use the correct FortiGate VLAN IP as their default gateway.

### 3. Policy order

If using a firewall policy, make sure it’s in the correct position in the policy table.

### 4. Switch configuration

* The port toward FortiGate must be a trunk
* VLANs 10, 20 and 30 must be tagged
* Access ports must be assigned to the correct VLANs

### 5. Debug flow

```bash
diagnose debug reset
diagnose debug flow filter clear
diagnose debug flow filter addr <source_IP>
diagnose debug flow show console enable
diagnose debug enable
diagnose debug flow trace start 20
# Run your tests
diagnose debug disable
```



