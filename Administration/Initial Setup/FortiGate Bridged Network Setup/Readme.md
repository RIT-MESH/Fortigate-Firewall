# GNS3 FortiGate Bridged Network Setup Summary

Below is the clean, final summary of the steps we followed to make the FortiGate reachable from your local machine using a bridged adapter and a new Cloud node.

---
```
        Wi-Fi Router (DHCP)
               |
       [ VMware Bridged ]
               |
        +----------------+
        |    GNS3 VM     |
        |  ens34/ens33   |
        +----------------+
               |
          Cloud (eth2)
               |
     +---------------------+
     |    FortiGate VM     |
     |  port3 (WAN - DHCP) |
     +---------------------+
               |
     port1 (LAN - WebTerm)
               |
          webterm-1
```

---

# **Step 1: Add a Bridged Adapter to the GNS3 VM (with VMnet0 check)**

### **1. Open VMware Workstation**

Make sure your GNS3 VM is **powered off**.

### **2. Add a new network adapter**

* Go to:
  **GNS3 VM → Settings → Add… → Network Adapter**
* Select:
  **Bridged (Automatic)**
* Enable:
  **Replicate physical network connection state** (important for Wi-Fi users)

### **3. Verify VMware Bridge is actually mapped to VMnet0**

This part is important and you did it manually:

#### **Open “Virtual Network Editor” → Configure Adapters**

* Find **VMnet0**
* Make sure VMnet0 is set to **Bridge to your Wi-Fi adapter**
  (Example: *Intel(R) Wi-Fi 6 AX201*, *Qualcomm Wireless*, etc.)

If VMnet0 is not mapped manually, VMware sometimes bridges to the wrong adapter.

### **4. Confirm inside GNS3 VM settings**

Your VM should now show adapters like:

| Adapter               | Mode                    |
| --------------------- | ----------------------- |
| Network Adapter 1     | Host-only               |
| Network Adapter 2     | NAT                     |
| Network Adapter 3     | Custom (VMnet0)         |
| **Network Adapter 4** | **Bridged (Automatic)** |

### **5. Start the GNS3 VM**

Now the VM will receive traffic correctly from your real Wi-Fi router.


---

# 2. Verify Bridged Adapter Inside the GNS3 VM (Linux)

Inside the VM terminal:

```
ip a
```

You confirmed that the new bridged adapter appears as:

* `ens34` or `ens33` (dynamic Wi-Fi bridged interface)

This interface received an IP from your **home Wi-Fi network**.

---

# 3. Create a New Cloud Node in GNS3

* Delete old Cloud nodes.
* Drag a **new Cloud** into the topology.
* Right‑click → **Configure → Ethernet Interfaces**.
* Use the dropdown and select the interface corresponding to the bridged adapter:

  * `ens34` (your Wi-Fi bridged link)
* Add it as **eth2** on the Cloud.

The Cloud now shows:

* **eth2 = bridged adapter**

---

# 4. Connect FortiGate to the Cloud

* Connect **FortiGate port3 → Cloud eth2**.

Topology:

```
webterm → port1 (LAN) → FortiGate → port3 (WAN) → Cloud (eth2)
```

---

# 5. Configure FortiGate Port3 as DHCP Client

Inside FortiGate CLI:

```
config system interface
edit port3
set mode dhcp
set allowaccess ping https ssh http
end
```

Then verify:


```
diag ip address list
get system interface
```

Port3 received a valid **home Wi-Fi network IP**, e.g.:

```
192.168.128.119
```

---

# 6. Access FortiGate GUI from Your Local Browser

Open:

```
https://192.168.x.x
```

(Log in with `admin` and no password.)

This confirmed the bridged Cloud → GNS3 VM → FortiGate path works correctly.

---

# Final Result

Your FortiGate is now reachable directly from your laptop through the bridged network, just like real hardware. All traffic flows from Wi-Fi → VM → Cloud node → FortiGate.

---
