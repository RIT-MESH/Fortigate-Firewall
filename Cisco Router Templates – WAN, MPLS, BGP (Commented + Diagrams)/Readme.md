# Cisco Router Templates – WAN, MPLS, BGP (Commented + Diagrams)

This document is a Cisco router templates for **WAN, MPLS, and BGP**, with diagrams placed next to the related configurations for easy understanding.

---

# 1. Base Router Template

## 1.1 Topology Context (Generic Router)

This base config applies to most Cisco routers in WAN/MPLS/BGP labs.

```bash
hostname R1                              # Set router hostname
no ip domain-lookup                      # Avoid DNS lookup on typos
service timestamps debug datetime msec   # Timestamps on debug output
service timestamps log datetime msec     # Timestamps on logs
!
username admin privilege 15 secret cisco123   # Local admin user
!
line console 0
 exec-timeout 0 0                        # Never auto-logout on console
 logging synchronous                     # Keep output readable
line vty 0 4
 password cisco                          # VTY password
 login                                   # Enable password auth
 transport input ssh                     # Allow SSH only
!
ip cef                                   # Enable Cisco Express Forwarding
```

---

# 2. WAN Templates

## 2.0 WAN Topology Diagram

This diagram matches the WAN interface templates (PPPoE, static, DHCP).

```text
   Customer LAN              WAN Link              ISP / MPLS Core
  192.168.10.0/24        public/PPPoE/DHCP         (varies)
        |                       |                      |
      [LAN]                 [WAN]                  [Edge]
        |                      |                       
      Cisco R1  g0/1    g0/0 / Dialer1        ISP / PE Router
```

## 2.1 PPPoE / Dialer Interface (Commented)

```bash
interface Dialer1
 ip address negotiated                   # Get IP via PPP negotiation
 encapsulation ppp                       # Use PPP over Dialer
 dialer pool 1                           # Bind to dialer pool 1
 ppp chap hostname user1                 # CHAP username
 ppp chap password 0 pass1               # CHAP password
 ppp pap sent-username user1 password pass1  # PAP credentials (if used)
!
interface GigabitEthernet0/0
 description WAN-PPPoE-Access           # Physical link towards modem/ONT
 pppoe enable group global              # Enable PPPoE client on this port
 pppoe-client dial-pool-number 1        # Use Dialer1 via pool 1
 no shutdown
```

## 2.2 Static WAN (Ethernet) (Commented)

```bash
interface GigabitEthernet0/0
 description WAN-Uplink-Static          # WAN link towards ISP
 ip address 203.0.113.2 255.255.255.252 # Static public IP
 no shutdown
```

## 2.3 DHCP WAN (Commented)

```bash
interface GigabitEthernet0/0
 description WAN-Uplink-DHCP            # WAN link gets IP via DHCP
 ip address dhcp                        # Request IP from ISP DHCP
 no shutdown
```

---

# 3. MPLS Templates

## 3.0 MPLS Core Topology Diagram

This diagram matches the **MPLS P and PE router** templates.

```text
          +--------+           +--------+           +--------+
 Customer |  CE-A  |---LAN----|  PE1   |---MPLS----|   P    |
  Site A  +--------+          |(Provider Edge)     +--------+
                                  |  
                                  | MPLS core
                                  |  
          +--------+          +--------+
 Customer |  CE-B  |---LAN----|  PE2   |
  Site B  +--------+          +--------+
```

## 3.1 MPLS Core Router (P Router) – Commented

```bash
ip cef                                  # Enable CEF for MPLS operation
mpls label protocol ldp                  # Use LDP for label distribution
mpls ip                                  # Enable MPLS globally
!
interface g0/0
 description MPLS-Core-Link-1
 ip address 10.0.0.1 255.255.255.0       # Core link IP
 mpls ip                                 # Enable MPLS on this interface
 no shutdown
!
interface g0/1
 description MPLS-Core-Link-2
 ip address 10.0.1.1 255.255.255.0       # Another core link
 mpls ip                                 # MPLS on this link too
 no shutdown
```

## 3.2 MPLS PE Router (Provider Edge) – Commented

```bash
ip cef                                  # Required for MPLS & VPN
mpls ip                                  # Enable MPLS
mpls label protocol ldp                  # Use LDP for labels
!
interface g0/0
 description MPLS-Core-to-P
 ip address 10.0.0.2 255.255.255.0       # Connected to P/another PE
 mpls ip                                 # Run MPLS here
 no shutdown
!
# Define VRF for customer A
ip vrf CustomerA
 rd 65000:1                              # Route Distinguisher
 route-target export 65000:1             # VPN routes exported with RT
 route-target import 65000:1             # VPN routes imported with RT
!
# Customer-facing interface in VRF
interface g0/1
 description CustomerA-LAN
 ip vrf forwarding CustomerA             # Bind interface to VRF
 ip address 192.168.10.1 255.255.255.0   # Gateway for customer LAN
 no shutdown
```

## 3.3 MPLS L3VPN – PE to PE Routing (BGP vpnv4 AF) – Commented

```bash
router bgp 65000                         # Provider AS
 bgp log-neighbor-changes                # Log BGP neighbor events
!
 address-family ipv4 unicast
  # Typically iBGP or IGP redistribution for core
 exit-address-family
!
 address-family vpnv4                    # VPNv4 for L3VPN labels + routes
  neighbor 10.0.0.1 send-community both  # Send extended + standard communities
  neighbor 10.0.0.1 activate             # Activate VPNv4 to remote PE/Route Reflector
 exit-address-family
!
 address-family ipv4 vrf CustomerA       # Per-VRF address-family
  redistribute connected                 # Inject connected VRF routes into VPN
 exit-address-family
```

---

# 4. BGP Templates

## 4.0 BGP Peering Diagram (ISP Edge)

This diagram matches the **eBGP** edge router config.

```text
  Customer LAN              Customer Edge R1            ISP Edge R2
 192.168.10.0/24        192.168.10.1/24 |         203.0.113.1/30
       PC ---- LAN ----[R1-AS65001]-----WAN------[R2-AS65000]----Core
```

## 4.1 EBGP – ISP Edge Router – Commented

```bash
router bgp 65001                         # Local AS
 bgp log-neighbor-changes                # Log changes
 neighbor 203.0.113.1 remote-as 65000    # eBGP neighbor (ISP)
!
 address-family ipv4 unicast
  network 192.168.10.0 mask 255.255.255.0   # Advertise customer LAN
  neighbor 203.0.113.1 activate             # Activate neighbor in this AFI
 exit-address-family
```

## 4.2 IBGP – Internal Peering (Commented)

```bash
router bgp 65000                         # Same AS on both routers (iBGP)
 neighbor 10.1.1.2 remote-as 65000       # Peer within same AS
 neighbor 10.1.1.2 update-source Loopback0 # Use loopback as source for stability
!
interface Loopback0
 ip address 10.1.1.1 255.255.255.255     # Stable router ID / peering address
```

## 4.3 BGP Route Reflector Template – Commented

```bash
router bgp 65000                         # Service provider / core AS
 bgp cluster-id 1                        # Route-reflector cluster ID
 neighbor 10.1.1.2 remote-as 65000       # iBGP neighbor (client)
 neighbor 10.1.1.2 route-reflector-client # Mark as RR client
 neighbor 10.1.1.3 remote-as 65000       # Another iBGP neighbor
 neighbor 10.1.1.3 route-reflector-client # Also RR client
```

### 4.3.1 BGP RR Topology Diagram

```text
           +-----------+        iBGP         +-----------+
           |   R2 RR   |<------------------->|   R3 RR   |
           | (RR Core) |                     | (Optional)|
           +-----+-----+                      +----------+
                 |
          iBGP (client)
                 |
       +---------+---------+
       |      R1 Client    |
       |   (Spoke / PE)    |
       +-------------------+
```

---

# 5. VRF Templates (Commented)

## 5.0 VRF / L3VPN Customer Diagram

```text
   CustomerA Site-A LAN           MPLS / Provider Core        CustomerA Site-B LAN
  192.168.100.0/24                     (VRF-A)                 192.168.200.0/24
          |                                 |                           |
        CE-A --- PE1 (VRF CustomerA) --- MPLS --- PE2 (VRF CustomerA) --- CE-B
```

## 5.1 VRF Definition – Commented

```bash
ip vrf CustomerA
 rd 65000:1                              # Unique RD for VRF CustomerA
 route-target import 65000:1             # Import VPN routes tagged 65000:1
 route-target export 65000:1             # Export routes with same RT
```

## 5.2 VRF Interface – Commented

```bash
interface g0/2
 description CustomerA-LAN
 ip vrf forwarding CustomerA             # Bind this interface to VRF
 ip address 192.168.100.1 255.255.255.0  # VRF interface IP (default gateway)
```

## 5.3 VRF Static Route – Commented

```bash
ip route vrf CustomerA 0.0.0.0 0.0.0.0 192.168.100.254
# Default route inside VRF CustomerA pointing to CE router
```

---

# 6. Verification Commands (Commented)

## 6.1 WAN Verification

```bash
show ip interface brief                  # Check interface status & IPs
show interfaces description              # Verify WAN interface descriptions
show pppoe session                       # For PPPoE sessions
show interfaces dialer 1                 # Dialer interface details
```

## 6.2 MPLS Verification

```bash
show mpls interfaces                     # Check MPLS enabled interfaces
show mpls ldp neighbors                  # LDP neighbors (label distribution)
show mpls forwarding-table               # LFIB – label forwarding table
```

## 6.3 BGP Verification

```bash
show ip bgp summary                      # BGP neighbor states / prefixes
show ip bgp                              # BGP table
show ip bgp neighbors                    # Detailed neighbor info
show ip route bgp                        # Routes learned via BGP
```

## 6.4 VRF Verification

```bash
show ip vrf                              # List VRFs
show ip route vrf CustomerA              # Routing table for VRF CustomerA
show ip bgp vpnv4 all                    # L3VPN routes
```

---

End of README
