# FortiGate Policy Examples – NAT, VIP, DNAT, Custom Services

This document provides clean, ready-to-use FortiGate firewall policy examples covering:

> **Note:** Sections 1–14 show clean configs for easy copy/paste. Section 15 contains fully commented versions of these configs (line-by-line explanations) so you can clearly understand what each command does.

* Source NAT (SNAT)
* Destination NAT (DNAT)
* VIP (Virtual IP port forwarding)
* Central NAT
* Custom service creation
* Common rule templates

Use this as a reference for labs or real deployments.

---

# 1. Source NAT (SNAT) – Outbound Internet Access

Source NAT translates internal private IPs to the FortiGate’s WAN IP.

## 1.1 Interface-Based NAT (Easy Mode)

```
config firewall policy
    edit 1
        set name "LAN-to-WAN"
        set srcintf "lan"
        set dstintf "wan1"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set service "ALL"
        set nat enable
    next
end
```

**Used when:** WAN interface has a public IP.

---

## 1.2 Central NAT Example

```
config firewall central-snat-map
    edit 1
        set src-addr 192.168.10.0/24
        set dst-addr 0.0.0.0/0
        set protocol 6
        set nat-ip 203.0.113.10
    next
end
```

Ensure Central NAT is enabled:

```
config system settings
    set central-nat enable
end
```

---

# 2. Destination NAT (DNAT) – Using VIPs

DNAT allows external users to access internal servers.

## 2.1 Basic VIP – Full Port Forwarding

```
config firewall vip
    edit "WEB-SERVER"
        set extip 203.0.113.10
        set mappedip 192.168.10.20
    next
end
```

### Policy

```
config firewall policy
    edit 2
        set name "WAN-to-Web"
        set srcintf "wan1"
        set dstintf "lan"
        set srcaddr "all"
        set dstaddr "WEB-SERVER"
        set action accept
        set service "HTTP" "HTTPS"
    next
end
```

---

## 2.2 VIP with Port Forwarding

Example: Forward WAN port 2222 → Internal server port 22 (SSH)

```
config firewall vip
    edit "SSH-Port-Forward"
        set extip 203.0.113.10
        set mappedip 192.168.10.21
        set extintf "wan1"
        set portforward enable
        set extport 2222
        set mappedport 22
        set protocol tcp
    next
end
```

### Policy

```
config firewall policy
    edit 3
        set name "WAN-SSH"
        set srcintf "wan1"
        set dstintf "lan"
        set srcaddr "all"
        set dstaddr "SSH-Port-Forward"
        set action accept
        set service "SSH"
    next
end
```

---

# 3. Static NAT (One-to-One NAT)

Outside IP always maps to the same inside IP.

```
config firewall vip
    edit "OneToOne-NAT"
        set extip 203.0.113.40
        set mappedip 192.168.10.40
        set extintf "wan1"
    next
end
```

Policy:

```
config firewall policy
    edit 4
        set srcintf "wan1"
        set dstintf "lan"
        set srcaddr "all"
        set dstaddr "OneToOne-NAT"
        set action accept
        set service ALL
    next
end
```

---

# 4. Custom Services

Used when you need non-standard ports.

## 4.1 Create a Custom TCP/UDP Service

Example: App runs on port TCP/8080

```
config firewall service custom
    edit "APP-8080"
        set tcp-portrange 8080
    next
end
```

Use in policy:

```
set service "APP-8080"
```

---

# 5. Common Firewall Policy Templates

## 5.1 LAN → WAN (Internet Access)

```
config firewall policy
    edit 10
        set srcintf "lan"
        set dstintf "wan1"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set service "ALL"
        set nat enable
    next
end
```

---

## 5.2 LAN → DMZ

```
config firewall policy
    edit 11
        set srcintf "lan"
        set dstintf "dmz"
        set srcaddr "LAN-SUBNET"
        set dstaddr "DMZ-SERVERS"
        set service ALL
        set action accept
    next
end
```

---

## 5.3 DMZ → LAN (Restricted)

```
config firewall policy
    edit 12
        set srcintf "dmz"
        set dstintf "lan"
        set srcaddr "DMZ-SERVERS"
        set dstaddr "LAN-MGMT"
        set service "SSH" "RDP"
        set action accept
    next
end
```

---

# 6. Troubleshooting NAT & VIP

## 6.1 Check VIP

```
show firewall vip
```

## 6.2 Check Firewall Policies

```
show firewall policy
```

## 6.3 Test DNAT from FortiGate

```
execute telnet <mapped_ip> <port>
execute telnet <vip_public_ip> <external_port>
```

## 6.4 Check Session Table

```
diag sys session list
```

## 6.5 Debug Policy Lookup

```
diag firewall iprope lookup <src_ip> <dst_ip> <protocol>
```

---

# 7. Quick Notes

* VIP = DNAT (external → internal)
* SNAT = internal → external
* Full NAT uses both
* VIP requires **matching policy**
* NAT must be enabled in outbound rules

---

# 8. Firewall Address Objects

Address objects make policies readable and reusable.

## 8.1 Host, Subnet, Range, FQDN

```bash
config firewall address
    edit "LAN-Subnet"
        set subnet 192.168.10.0 255.255.255.0
    next
    edit "WEB-Server"
        set subnet 192.168.10.20 255.255.255.255
    next
    edit "DMZ-Range"
        set type iprange
        set start-ip 192.168.20.10
        set end-ip 192.168.20.50
    next
    edit "Google-FQDN"
        set type fqdn
        set fqdn "www.google.com"
    next
end
```

## 8.2 Address Groups

```bash
config firewall addrgrp
    edit "DMZ-SERVERS"
        set member "WEB-Server" "DMZ-Range"
    next
end
```

Use in policy:

```bash
set srcaddr "LAN-Subnet"
set dstaddr "DMZ-SERVERS"
```

---

# 9. Service Groups

## 9.1 Custom Services

```bash
config firewall service custom
    edit "WEB-8080"
        set tcp-portrange 8080
    next
    edit "DB-5432"
        set tcp-portrange 5432
    next
end
```

## 9.2 Service Group

```bash
config firewall service group
    edit "APP-STACK"
        set member "HTTP" "HTTPS" "WEB-8080" "DB-5432"
    next
end
```

Use in policy:

```bash
set service "APP-STACK"
```

---

# 10. VIP Groups

VIP groups let you group multiple VIPs and use them in one policy.

## 10.1 Create Multiple VIPs

```bash
config firewall vip
    edit "WEB-1"
        set extip 203.0.113.11
        set mappedip 192.168.10.11
    next
    edit "WEB-2"
        set extip 203.0.113.12
        set mappedip 192.168.10.12
    next
end
```

## 10.2 Create VIP Group

```bash
config firewall vipgrp
    edit "WEB-FARM"
        set member "WEB-1" "WEB-2"
    next
end
```

## 10.3 Policy Using VIP Group

```bash
config firewall policy
    edit 20
        set name "WAN-to-WEB-FARM"
        set srcintf "wan1"
        set dstintf "dmz"
        set srcaddr "all"
        set dstaddr "WEB-FARM"
        set action accept
        set service "HTTP" "HTTPS"
    next
end
```

---

# 11. NAT64 / NAT46 (IPv4–IPv6 Translation)

> NOTE: Exact commands can vary by FortiOS version; this is a lab-style example.

## 11.1 NAT64 – IPv6 Clients to IPv4 Server

IPv6 clients reach an IPv4-only server via a NAT64 VIP.

```bash
config firewall vip46
    edit "NAT64-WEB"
        set extip6 2001:db8:100::10          # IPv6 visible to clients
        set mappedip 192.0.2.10              # IPv4 server
        set extintf "wan1"
        set portforward enable
        set extport 80
        set mappedport 80
        set protocol tcp
    next
end
```

Policy:

```bash
config firewall policy6
    edit 30
        set srcintf "wan1"
        set dstintf "dmz"
        set srcaddr "all"
        set dstaddr "NAT64-WEB"
        set action accept
        set service "HTTP"
    next
end
```

## 11.2 NAT46 – IPv4 Clients to IPv6 Server

```bash
config firewall vip64
    edit "NAT46-APP"
        set extip 203.0.113.50               # IPv4 seen by clients
        set mappedip6 2001:db8:200::50       # IPv6 server
        set extintf "wan1"
        set portforward enable
        set extport 443
        set mappedport 443
        set protocol tcp
    next
end
```

Policy:

```bash
config firewall policy
    edit 31
        set srcintf "wan1"
        set dstintf "dmz"
        set srcaddr "all"
        set dstaddr "NAT46-APP"
        set action accept
        set service "HTTPS"
    next
end
```

---

# 12. IPv6 Policy Examples

## 12.1 IPv6 Interface on LAN

```bash
config system interface
    edit "lan"
        set ip6 2001:db8:10::1/64
        set allowaccess ping https http
    next
end
```

## 12.2 IPv6 Address Objects

```bash
config firewall address6
    edit "LANv6-Subnet"
        set ip6 2001:db8:10::/64
    next
    edit "Serverv6"
        set ip6 2001:db8:10::20/128
    next
end
```

## 12.3 IPv6 Policy LAN → WAN

```bash
config firewall policy6
    edit 40
        set srcintf "lan"
        set dstintf "wan1"
        set srcaddr "LANv6-Subnet"
        set dstaddr "all"
        set service "ALL"
        set action accept
    next
end
```

---

# 13. HA-Aware NAT/VIP Configs

In HA (Active–Passive) clusters:

* Use **floating IPs / virtual MACs** for VIPs and WAN addresses where possible.
* VIPs and policies are synchronized automatically between units.

## 13.1 Enabling Session Pickup (Optional)

```bash
config system ha
    set session-pickup enable
    set session-pickup-nat enable
end
```

## 13.2 VIP on HA Cluster

VIP config is created once and syncs to all units:

```bash
config firewall vip
    edit "HA-WEB"
        set extip 203.0.113.100
        set mappedip 192.168.10.100
        set extintf "wan1"
    next
end
```

Ensure:

* Same **wan1** IP/VRRP/floating IP is reachable via the active unit.
* Upstream router sends traffic to the cluster’s shared address.

---

# 14. Real-World Multi-Tier NAT Scenarios

## 14.1 Internet → FortiGate → DMZ Reverse Proxy → App Server

**Goal:**

* Public IP 203.0.113.10 → DMZ NGINX reverse proxy → Internal app 10.0.0.10.

### Step 1: VIP to DMZ Reverse Proxy

```bash
config firewall vip
    edit "VIP-REVERSE-PROXY"
        set extip 203.0.113.10
        set mappedip 192.168.20.10          # NGINX in DMZ
        set extintf "wan1"
        set portforward enable
        set extport 443
        set mappedport 443
        set protocol tcp
    next
end
```

### Step 2: Policy WAN → DMZ

```bash
config firewall policy
    edit 50
        set name "WAN-to-Proxy"
        set srcintf "wan1"
        set dstintf "dmz"
        set srcaddr "all"
        set dstaddr "VIP-REVERSE-PROXY"
        set action accept
        set service "HTTPS"
    next
end
```

### Step 3: DMZ → LAN Policy (Proxy to App)

```bash
config firewall address
    edit "APP-SERVER"
        set subnet 10.0.0.10 255.255.255.255
    next
end

config firewall policy
    edit 51
        set name "Proxy-to-App"
        set srcintf "dmz"
        set dstintf "lan"
        set srcaddr "VIP-REVERSE-PROXY"    # or DMZ host object
        set dstaddr "APP-SERVER"
        set service "HTTP" "HTTPS"
        set action accept
    next
end
```

---

## 14.2 Double NAT (ISP CPE + FortiGate)

**Scenario:**

* FortiGate WAN is private (10.x.x.x) behind ISP router.
* You still want inbound access via VIP.

**High-level steps:**

1. On ISP router: Port forward from public IP → FortiGate WAN IP.
2. On FortiGate: VIP from WAN IP → internal server.

FortiGate VIP example:

```bash
config firewall vip
    edit "VIP-DBL-NAT"
        set extip 10.0.0.2
        set mappedip 192.168.10.50
        set extintf "wan1"
        set portforward enable
        set extport 8443
        set mappedport 443
        set protocol tcp
    next
end
```

Policy:

```bash
config firewall policy
    edit 60
        set srcintf "wan1"
        set dstintf "lan"
        set srcaddr "all"
        set dstaddr "VIP-DBL-NAT"
        set action accept
        set service "HTTPS"
    next
end
```

---

# 15. Commented Code Reference (Easy Understanding)

Below are commented versions of the most important examples above. Use these when you want to remember **why** each line is there.

## 15.1 SNAT – LAN to WAN (Interface NAT)

```bash
config firewall policy              # Enter firewall policy configuration
    edit 1                          # Create/edit policy ID 1
        set name "LAN-to-WAN"      # Friendly name for the policy
        set srcintf "lan"          # Source interface = LAN
        set dstintf "wan1"         # Destination interface = WAN1
        set srcaddr "all"          # Any source address in LAN
        set dstaddr "all"          # Any destination on the internet
        set action accept           # Allow the traffic
        set service "ALL"          # Allow all services (ports) – can restrict later
        set nat enable              # Enable NAT (use WAN IP as source)
    next                            # Move to next policy
end                                 # Exit policy config
```

## 15.2 VIP – DNAT to Internal Web Server

```bash
config firewall vip                 # Configure Virtual IPs (DNAT)
    edit "WEB-SERVER"              # Name of the VIP
        set extip 203.0.113.10      # Public IP seen from internet
        set mappedip 192.168.10.20  # Internal server IP
    next
end

config firewall policy              # Policy to allow traffic to VIP
    edit 2
        set name "WAN-to-Web"      # Policy name
        set srcintf "wan1"         # From WAN interface
        set dstintf "lan"          # To LAN where server lives
        set srcaddr "all"          # Any internet source
        set dstaddr "WEB-SERVER"   # Destination = VIP object
        set action accept           # Allow
        set service "HTTP" "HTTPS" # Only web traffic
    next
end
```

## 15.3 Port-Forward VIP (External 2222 → Internal SSH 22)

```bash
config firewall vip
    edit "SSH-Port-Forward"
        set extip 203.0.113.10      # Public IP
        set mappedip 192.168.10.21  # Internal SSH server
        set extintf "wan1"         # VIP tied to WAN1
        set portforward enable      # Turn on port forwarding
        set extport 2222            # Port users connect to from internet
        set mappedport 22           # Real SSH port on server
        set protocol tcp            # TCP only
    next
end

config firewall policy
    edit 3
        set name "WAN-SSH"
        set srcintf "wan1"         # From internet
        set dstintf "lan"          # To LAN
        set srcaddr "all"          # Any source
        set dstaddr "SSH-Port-Forward" # VIP as destination
        set action accept
        set service "SSH"          # SSH service (port 22)
    next
end
```

## 15.4 Address Objects & Groups

```bash
config firewall address
    edit "LAN-Subnet"
        set subnet 192.168.10.0 255.255.255.0   # Entire LAN /24
    next
    edit "WEB-Server"
        set subnet 192.168.10.20 255.255.255.255 # Single host
    next
    edit "DMZ-Range"
        set type iprange                       # Range of IPs
        set start-ip 192.168.20.10
        set end-ip 192.168.20.50
    next
end

config firewall addrgrp
    edit "DMZ-SERVERS"                         # Group of DMZ servers
        set member "WEB-Server" "DMZ-Range"    # Add objects to group
    next
end
```

## 15.5 Custom Service & Service Group

```bash
config firewall service custom
    edit "APP-8080"
        set tcp-portrange 8080                 # Custom TCP service port 8080
    next
    edit "DB-5432"
        set tcp-portrange 5432                 # DB port 5432
    next
end

config firewall service group
    edit "APP-STACK"                           # Group for app-related ports
        set member "HTTP" "HTTPS" "APP-8080" "DB-5432"
    next
end

# Use in policy:
config firewall policy
    edit 15
        set name "LAN-to-APP"
        set srcintf "lan"
        set dstintf "dmz"
        set srcaddr "LAN-Subnet"
        set dstaddr "DMZ-SERVERS"
        set service "APP-STACK"              # Use the service group
        set action accept
    next
end
```

## 15.6 VIP Group (Web Farm)

```bash
config firewall vip
    edit "WEB-1"
        set extip 203.0.113.11                 # Public IP for server 1
        set mappedip 192.168.10.11             # Internal server 1
    next
    edit "WEB-2"
        set extip 203.0.113.12                 # Public IP for server 2
        set mappedip 192.168.10.12             # Internal server 2
    next
end

config firewall vipgrp
    edit "WEB-FARM"
        set member "WEB-1" "WEB-2"              # Group both VIPs
    next
end

config firewall policy
    edit 20
        set name "WAN-to-WEB-FARM"
        set srcintf "wan1"                     # From internet
        set dstintf "dmz"                      # To DMZ
        set srcaddr "all"
        set dstaddr "WEB-FARM"                  # VIP group as destination
        set action accept
        set service "HTTP" "HTTPS"
    next
end
```

## 15.7 NAT64 Example (IPv6 → IPv4)

```bash
config firewall vip46
    edit "NAT64-WEB"
        set extip6 2001:db8:100::10            # IPv6 address exposed to clients
        set mappedip 192.0.2.10                # IPv4-only web server
        set extintf "wan1"                     # On WAN1
        set portforward enable
        set extport 80                         # IPv6 clients hit port 80
        set mappedport 80                      # Server also listens on 80
        set protocol tcp
    next
end

config firewall policy6
    edit 30
        set srcintf "wan1"                     # IPv6 side
        set dstintf "dmz"                      # IPv4/dual-stack side
        set srcaddr "all"
        set dstaddr "NAT64-WEB"                 # VIP46 object
        set action accept
        set service "HTTP"
    next
end
```

## 15.8 IPv6 LAN Policy

```bash
config system interface
    edit "lan"
        set ip6 2001:db8:10::1/64              # IPv6 address on LAN
        set allowaccess ping https http        # Allow mgmt from LAN
    next
end

config firewall address6
    edit "LANv6-Subnet"
        set ip6 2001:db8:10::/64              # IPv6 subnet object
    next
end

config firewall policy6
    edit 40
        set srcintf "lan"                      # From LAN
        set dstintf "wan1"                     # To WAN
        set srcaddr "LANv6-Subnet"             # IPv6 LAN
        set dstaddr "all"                      # Any destination
        set service "ALL"                      # All IPv6 services
        set action accept
    next
end
```

## 15.9 HA Session Pickup for NAT/VIP

```bash
config system ha
    set mode a-p                               # Active-passive HA
    set group-name "FGT-HA"
    set session-pickup enable                  # Keep sessions on failover
    set session-pickup-nat enable              # Also keep NAT sessions
end
```

## 15.10 Multi-Tier NAT – Reverse Proxy in DMZ

```bash
# VIP from internet to DMZ reverse proxy
config firewall vip
    edit "VIP-REVERSE-PROXY"
        set extip 203.0.113.10                 # Public IP
        set mappedip 192.168.20.10             # NGINX in DMZ
        set extintf "wan1"
        set portforward enable
        set extport 443                        # External HTTPS
        set mappedport 443                     # Proxy listens on 443
        set protocol tcp
    next
end

# Policy: WAN to DMZ reverse proxy
config firewall policy
    edit 50
        set name "WAN-to-Proxy"
        set srcintf "wan1"
        set dstintf "dmz"
        set srcaddr "all"
        set dstaddr "VIP-REVERSE-PROXY"
        set action accept
        set service "HTTPS"
    next
end

# Address object for internal app server
config firewall address
    edit "APP-SERVER"
        set subnet 10.0.0.10 255.255.255.255   # App in LAN
    next
end

# Policy: DMZ proxy to internal app
config firewall policy
    edit 51
        set name "Proxy-to-App"
        set srcintf "dmz"
        set dstintf "lan"
        set srcaddr "VIP-REVERSE-PROXY"        # Or DMZ host / range
        set dstaddr "APP-SERVER"
        set service "HTTP" "HTTPS"            # App protocols
        set action accept
    next
end
```

---

## 15.11 Central SNAT Map

```bash
config system settings
    set central-nat enable                # Turn on central NAT mode
end

config firewall central-snat-map
    edit 1
        set src-addr 192.168.10.0/24      # Original source subnet
        set dst-addr 0.0.0.0/0            # Any destination (internet)
        set protocol 6                    # Protocol 6 = TCP (0 = any)
        set nat-ip 203.0.113.10           # Public IP used as source
    next
end
```

## 15.12 Static One-to-One NAT 

```bash
config firewall vip
    edit "OneToOne-NAT"
        set extip 203.0.113.40            # Public IP on WAN
        set mappedip 192.168.10.40        # Internal server IP
        set extintf "wan1"               # Interface where VIP lives
    next
end

config firewall policy
    edit 4
        set srcintf "wan1"               # Traffic arriving from internet
        set dstintf "lan"                # Going into LAN
        set srcaddr "all"                # Any external source
        set dstaddr "OneToOne-NAT"       # VIP as destination
        set action accept                 # Allow
        set service ALL                   # All services – can restrict later
    next
end
```

## 15.13 LAN → DMZ and DMZ → LAN Policies

```bash
# LAN → DMZ (users accessing DMZ servers)
config firewall policy
    edit 11
        set srcintf "lan"                 # From LAN users
        set dstintf "dmz"                 # To DMZ
        set srcaddr "LAN-SUBNET"         # LAN subnet object
        set dstaddr "DMZ-SERVERS"        # Group of DMZ servers
        set service ALL                   # All services (can restrict)
        set action accept                 # Allow
    next
end

# DMZ → LAN (restricted management)
config firewall policy
    edit 12
        set srcintf "dmz"                 # From DMZ servers
        set dstintf "lan"                 # To LAN
        set srcaddr "DMZ-SERVERS"        # Only DMZ servers
        set dstaddr "LAN-MGMT"           # Only management subnet/hosts
        set service "SSH" "RDP"          # Only management ports
        set action accept                 # Allow
    next
end
```

## 15.14 Troubleshooting Commands 

```bash
show firewall vip                        # List all VIP/DNAT objects
show firewall policy                     # Show all firewall policies

execute telnet <mapped_ip> <port>       # Test connectivity to real server
execute telnet <vip_public_ip> <port>   # Test connectivity using VIP from FGT

diag sys session list                   # Show session table entries

diag firewall iprope lookup \
    <src_ip> <dst_ip> <protocol>        # See which policy/NAT will match
```

## 15.15 NAT46 Example 

```bash
config firewall vip64
    edit "NAT46-APP"
        set extip 203.0.113.50           # IPv4 address used by clients
        set mappedip6 2001:db8:200::50   # IPv6-only server address
        set extintf "wan1"              # Interface facing IPv4 clients
        set portforward enable
        set extport 443                  # External HTTPS port
        set mappedport 443               # Internal HTTPS port
        set protocol tcp                 # TCP only
    next
end

config firewall policy
    edit 31
        set srcintf "wan1"              # From IPv4 side
        set dstintf "dmz"               # To IPv6/dual-stack side
        set srcaddr "all"               # Any client
        set dstaddr "NAT46-APP"         # VIP64 object
        set action accept
        set service "HTTPS"             # HTTPS service
    next
end
```

## 15.16 Double NAT VIP 

```bash
config firewall vip
    edit "VIP-DBL-NAT"
        set extip 10.0.0.2               # FGT WAN private IP (behind ISP)
        set mappedip 192.168.10.50       # Internal server IP
        set extintf "wan1"              # WAN interface
        set portforward enable
        set extport 8443                 # Port forwarded by ISP to FGT
        set mappedport 443               # Real HTTPS port on server
        set protocol tcp
    next
end

config firewall policy
    edit 60
        set srcintf "wan1"              # From ISP router / internet
        set dstintf "lan"               # To LAN
        set srcaddr "all"
        set dstaddr "VIP-DBL-NAT"       # VIP as destination
        set action accept
        set service "HTTPS"
    next
end
```

---

