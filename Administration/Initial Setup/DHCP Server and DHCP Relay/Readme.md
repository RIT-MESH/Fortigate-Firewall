

# DHCP Server and DHCP Relay – Full Network Lab Configuration

Below is the entire network build, including **R1, FortiGate, Cloud (VMnet8), PC, Webterm**, and complete troubleshooting commands.

<img width="844" height="347" alt="image" src="https://github.com/user-attachments/assets/165d37fb-5d6e-445b-99fa-719a093d4eac" />


Everything is arranged cleanly so you can scroll, read, and study easily.


---

# 1. R1 (Cisco Router) Configuration

## 1.1 Internet Interface (f1/1 → Cloud / VMnet8 NAT)

```
conf t                    # Enter configuration mode
interface f1/1            # Select interface connected to Cloud (VMnet8)
 no shutdown              # Enable the interface
 ip address dhcp          # Get IP from VMnet8 NAT
end                       # Exit configuration
```

## 1.2 Default Route

```
conf t                                   # Enter config mode
ip route 0.0.0.0 0.0.0.0 f1/1            # Default route → Internet via f1/1
end                                      # Exit config
```

## 1.3 LAN Interface (f1/0 → FortiGate port3)

```
conf t                                   # Enter config mode
interface f1/0                           # LAN-facing interface\ nip address 172.16.10.10 255.255.255.0   # Static IP for router LAN
 no shutdown                             # Enable interface
end                                      # Exit config
```

---

# 2. FortiGate Interface Configuration

## 2.1 WAN Side (port3 → R1)

```
config system interface
 edit "port3"
  set ip 172.16.10.1 255.255.255.0       # Set WAN IP
  set allowaccess ping http https        # Allow management & ping
 next
end
```

## 2.2 LAN Side (port1 → PC + Webterm)

```
config system interface
 edit "port1"
  set ip 192.168.1.99 255.255.255.0      # Set LAN IP
  set allowaccess ping http https        # Allow GUI + ping from LAN
 next
end
```

---

# 3. FortiGate Static Route (towards Internet via R1)

```
config router static
 edit 1
  set gateway 172.16.10.10               # Next hop = R1
  set device "port3"                     # Traffic exits via WAN port3
 next
end
```

---

# 4. FortiGate Firewall Policies

## 4.1 LAN → WAN (Allow users to browse Internet)

```
config firewall policy
 edit 1
  set name "LAN-to-R1"
  set srcintf "port1"                    # From LAN
  set dstintf "port3"                    # To WAN/R1
  set srcaddr "all"
  set dstaddr "all"
  set schedule "always"
  set service "ALL"
  set action accept
  set nat enable                          # Enable NAT for Internet access
 next
end
```

## 4.2 WAN → LAN (optional for testing)

```
config firewall policy
 edit 3
  set name "R1-to-LAN"
  set srcintf "port3"                    # From R1
  set dstintf "port1"                    # To LAN
  set srcaddr "all"
  set dstaddr "all"
  set schedule "always"
  set service "ALL"
  set action accept                       # Allow traffic inside
  set nat disable
 next
end
```

---

# 5. DHCP on FortiGate

## Option A: FortiGate as DHCP Server

```
config system dhcp server
 edit 1
  set interface "port1"                  # Serve IPs on LAN port1
  set default-gateway 192.168.1.99       # FortiGate LAN IP
  set netmask 255.255.255.0
  config ip-range
   edit 1
    set start-ip 192.168.1.10
    set end-ip 192.168.1.200
   next
  end
 next
end
```

## Option B: DHCP Relay (Forward to R1)

```
config system interface
 edit "port1"
  set dhcp-relay-service enable
  set dhcp-relay-ip "172.16.10.10"       # R1 as DHCP server
 next
end
```

> Use either DHCP Server OR Relay, not both.

---

# 6. PC (VPCS) Configuration

## DHCP Mode

```
dhcp                                        # Get IP from FortiGate or R1
```

## Manual Mode

```
ip 192.168.1.10 255.255.255.0 192.168.1.99  # Manual IP + gateway
```

## Testing

```
ping 192.168.1.99                           # Gateway
ping 172.16.10.1                             # FortiGate WAN
ping 8.8.8.8                                 # Internet
```

---

# 7. Webterm (Linux) Configuration

```
dhclient eth0                                # Request IP via DHCP
ip a                                         # Check IP
route -n                                     # Check gateway
ping 192.168.1.99                            # Ping gateway
ping 8.8.8.8                                 # Test Internet
```

Access FortiGate GUI:

```
http://192.168.1.99
```

---

# 8. GNS3 Cloud Setup

In Cloud settings choose:

```
VMnet8 (NAT)                                 # Provides DHCP + Internet
```

Then connect:

```
Cloud → R1 f1/1
```

---

# 9. Full Network Flow

```
Internet → Cloud (VMnet8 NAT) → R1 f1/1 → R1 f1/0 → FortiGate port3 → FortiGate port1 → Switch → PC/Webterm
```

---

# 10. Troubleshooting Commands

## R1 (Cisco)

```
show ip int brief                            # Interface status
show ip route                                # Routing table
show dhcp lease                              # DHCP info from VMnet8
ping <IP>                                     # Connectivity test
```

## FortiGate

```
show system interface                         # Show interface configs
execute ping <IP>                             # Ping from FortiGate
show firewall policy                          # View policies
get router info routing-table all             # Routing table
get system arp                                # ARP entries
diagnose sniff packet port1 'port 67 or 68' 4 # Capture DHCP traffic
```

## PC (VPCS)

```
dhcp                                          # Get IP
show ip                                       # Display IP
ping <IP>                                     # Test
```

## Webterm (Linux)

```
ip a                                          # Show interfaces
route -n                                      # Show routing table
dhclient eth0                                 # Renew DHCP
cat /etc/resolv.conf                          # Check DNS
```

## GNS3 Cloud / Windows

Restart VMware NAT services if DHCP fails:

```
net stop vmnetdhcp
net stop vmnat
net start vmnetdhcp
net start vmnat
```

---

