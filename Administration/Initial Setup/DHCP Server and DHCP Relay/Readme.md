# DHCP Server and DHCP Relay

This document lists every command and configuration used to build the full network path:

**Cloud (Internet) → R1 → FortiGate → LAN (PC + Webterm)**
<img width="844" height="347" alt="image" src="https://github.com/user-attachments/assets/165d37fb-5d6e-445b-99fa-719a093d4eac" />

Including interface setup, routing, firewall policies, and DHCP.

---

## 1. R1 Configuration

### 1.1 Internet Interface (f1/1 → Cloud / VMnet8 NAT)

```
conf t
interface f1/1
 no shutdown
 ip address dhcp
end
```

### 1.2 Default Route

```
conf t
ip route 0.0.0.0 0.0.0.0 f1/1
end
```

### 1.3 LAN Interface (f1/0 → FortiGate port3)

```
conf t
interface f1/0
 ip address 172.16.10.10 255.255.255.0
 no shutdown
end
```

---

## 2. FortiGate Interface Configuration

### 2.1 WAN Side (port3 → R1)

```
config system interface
 edit "port3"
  set ip 172.16.10.1 255.255.255.0
  set allowaccess ping http https
 next
end
```

### 2.2 LAN Side (port1 → PC + Webterm)

```
config system interface
 edit "port1"
  set ip 192.168.1.99 255.255.255.0
  set allowaccess ping http https
 next
end
```

---

## 3. FortiGate Static Route (towards Internet via R1)

```
config router static
 edit 1
  set gateway 172.16.10.10
  set device "port3"
 next
end
```

---

## 4. FortiGate Firewall Policies

### 4.1 LAN → WAN

```
config firewall policy
 edit 1
  set name "LAN-to-R1"
  set srcintf "port1"
  set dstintf "port3"
  set srcaddr "all"
  set dstaddr "all"
  set schedule "always"
  set service "ALL"
  set action accept
  set nat enable
 next
end
```

### 4.2 WAN → LAN (for testing / ping)

```
config firewall policy
 edit 3
  set name "R1-to-LAN"
  set srcintf "port3"
  set dstintf "port1"
  set srcaddr "all"
  set dstaddr "all"
  set schedule "always"
  set service "ALL"
  set action accept
  set nat disable
 next
end
```

---

## 5. DHCP on FortiGate (for PC + Webterm)

### Option A: FortiGate as DHCP Server

```
config system dhcp server
 edit 1
  set interface "port1"
  set default-gateway 192.168.1.99
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

### Option B: DHCP Relay to R1

```
config system interface
 edit "port1"
  set dhcp-relay-service enable
  set dhcp-relay-ip "172.16.10.10"
 next
end
```

---

## 6. PC (VPCS) Configuration

### DHCP

```
dhcp
```

### Manual

```
ip 192.168.1.10 255.255.255.0 192.168.1.99
```

### Testing

```
ping 192.168.1.99
ping 172.16.10.1
ping 8.8.8.8
```

---

## 7. Webterm (Linux) Configuration

```
dhclient eth0
ip a
route -n
ping 192.168.1.99
ping 8.8.8.8
```

Access FortiGate GUI:

```
http://192.168.1.99
```

---

## 8. Cloud (GNS3) Configuration

In GNS3:

**Cloud1 → Configure → Ethernet interfaces**

Select:

```
VMnet8 (NAT)
```

This maps to **eth1** after the VMware restore.

Reconnect:

```
Cloud (VMnet8) → R1 f1/1
```

---

## 9. Full Network Flow

Internet → Cloud (VMnet8 NAT) → R1 f1/1 → R1 f1/0 → FortiGate port3 → FortiGate port1 → Switch → PC + Browser

---

## 10. Troubleshooting Commands (Easy and Practical)

Below are the essential troubleshooting commands used during setup, explained in simple language.

---

### 🔍 **10.1 R1 (Cisco Router) Troubleshooting**

#### **Check all interface status**

```
show ip int brief
```

*Shows IP addresses and UP/DOWN status of all interfaces.*

#### **Test connectivity to FortiGate, LAN, or Internet**

```
ping <IP>
```

Examples:

```
ping 192.168.1.99     (FortiGate LAN)
ping 172.16.10.1      (FortiGate WAN)
ping 8.8.8.8          (Internet)
```

#### **Check default gateway**

```
show ip route
```

*Confirms if 0.0.0.0/0 is pointing to f1/1.*

#### **Check DHCP result on WAN interface**

```
show dhcp lease
```

*Shows if R1 successfully received an IP from Cloud (VMnet8).*

---

### 🔍 **10.2 FortiGate Troubleshooting**

#### **Check all interfaces and IPs**

```
show system interface
```

#### **Ping from FortiGate (LAN, WAN, Internet)**

```
execute ping 192.168.1.10
execute ping 172.16.10.10
execute ping 8.8.8.8
```

#### **Check firewall policies**

```
show firewall policy
```

#### **Check routes on FortiGate**

```
get router info routing-table all
```

#### **Check ARP table (devices connected)**

```
get system arp
```

#### **Check DHCP behavior (relay/server)**

```
diagnose sniff packet port1 'port 67 or port 68' 4
```

*Useful when PC cannot get DHCP.*

---

### 🔍 **10.3 PC / VPCS Troubleshooting**

#### **Request DHCP IP**

```
dhcp
```

#### **Check assigned IP**

```
show ip
```

#### **Test connectivity**

```
ping 192.168.1.99
ping 172.16.10.1
ping 8.8.8.8
```

---

### 🔍 **10.4 Webterm (Linux) Troubleshooting**

#### **Check interfaces**

```
ip a
```

#### **Check routing table**

```
route -n
```

#### **Renew DHCP**

```
dhclient eth0
```

#### **Check DNS resolution**

```
cat /etc/resolv.conf
```

#### **Ping tests**

```
ping 192.168.1.99
ping google.com
```

---

### 🔍 **10.5 GNS3 Cloud Troubleshooting**

#### **Verify Cloud → VMnet8 mapping**

Make sure Cloud uses:

```
VMnet8 (NAT)
```

Not VMnet0 or VMnet1.

#### **If R1 DHCP fails**

Restart VMware network services in Windows:

```
net stop vmnetdhcp
net stop vmnat
net start vmnetdhcp
net start vmnat
```

---

This completes the full command list *plus* troubleshooting steps in an easy and practical format.
