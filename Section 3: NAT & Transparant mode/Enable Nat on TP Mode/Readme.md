# FortiGate Transparent Mode with NAT

This guide explains how to enable source NAT on a FortiGate that is operating in Transparent Mode. In transparent mode, the FortiGate normally behaves like a Layer-2 bridge, but NAT can still be enabled when needed. This setup is useful when the FortiGate must sit inline without acting as the gateway, yet still provide internet access by translating traffic.

---

## 1. What You Are Achieving


https://github.com/user-attachments/assets/e592c8c3-5ea3-424c-b742-2bc4485b7714

https://github.com/user-attachments/assets/a6524d72-55fd-4ab0-881b-2c0ab6c251b1



In Transparent Mode:

* The FortiGate does not participate in routing.
* Data interfaces do not have IP addresses.
* A dedicated management IP is used for administration.

To enable NAT in this mode, you:

1. Assign two management IPs

   * One reachable from the LAN
   * One reachable from the WAN
2. Create an IP pool for NAT
3. Apply NAT in a firewall policy
4. Add a default route so the FortiGate knows where to send outbound traffic

This allows LAN devices to reach the internet through the FortiGate even though it is operating as a transparent bridge.

---

## 2. Step-by-Step Configuration

Below is the complete and clean version of the configuration from the uploaded file.

---

## 2.1 Configure Management IPs

Two management IPs are set:

* First for administrative access (LAN side)
* Second for NAT gateway reachability (WAN side)

```
config system settings
    set manageip 192.168.1.99/24 192.168.122.240/24
end
```

This allows the FortiGate to bridge traffic while still being reachable from both networks.

---

## 2.2 Create an IP Pool for NAT

This pool defines which IP address will be used for NAT. Here the WAN-side IP is used as the NAT source.

```
config firewall ippool
    edit 1
        set type overload
        set startip 192.168.122.240
        set endip 192.168.122.240
    next
end
```

Because this is overload mode, multiple LAN devices can share the same translated IP.

---

## 2.3 Create the NAT Policy

This firewall policy allows LAN-to-WAN traffic and applies NAT using the IP pool.

```
config firewall policy
    edit 1
        set name "INTERNET POLICY"
        set srcintf "port1"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set logtraffic all
        set ippool enable
        set poolname "1"
        set nat enable
    next
end
```

Important points:

* Transparent mode still uses normal firewall policies.
* NAT only works when explicitly enabled inside the policy.
* The IP pool must be attached to the policy.

---

## 2.4 Add Default Route

Transparent mode still requires a route for outbound NAT traffic.

```
config router static
    edit 1
        set gateway 192.168.122.1
    next
end
```

The FortiGate uses this route to deliver NAT-translated packets to the next hop on the WAN side.

---

## 3. How the Traffic Flows

1. A LAN device sends traffic to the upstream gateway.
2. Traffic passes through the FortiGate.
3. The firewall policy matches and applies NAT using the IP pool.
4. Traffic is forwarded to the WAN upstream router.
5. Return traffic flows back through the FortiGate and is un-NATed.
6. The original connection is delivered back to the LAN host.

The FortiGate bridges traffic at Layer-2 but still performs NAT at Layer-3 inside the policy.

---

## 4. Notes & Recommendations

* NAT in transparent mode is not common but works when properly configured.
* The management IPs do not participate in routing; they only allow the FortiGate to reach both sides.
* Ensure the WAN-side gateway matches the network where the NAT pool address exists.
* Enable logging to validate NAT hits during testing.
