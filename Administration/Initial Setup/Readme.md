# Fortigate Access Guide

This document provides a simple and clear overview of how to access and configure a Fortigate firewall.

> **Note:** Added detailed comments under each command block to explain what the command does and why it is used. of how to access and configure a Fortigate firewall.
> It includes login details, hostname changes, interface configuration and basic CLI operations.

---

## 📌 Access Methods

You can connect to the Fortigate unit in three different ways:

1. **Web GUI (HTTPS/HTTP)**
2. **CLI via SSH**
3. **Console Port (Direct connection)**

> **Note:**
> Fortigate appliances ship with **port1** preconfigured to the default IP address:
> **192.168.1.99**

---

## 🔐 Default Login

* **Username:** `admin`
* **Password:** *(empty / blank)*
* You will be prompted to set a new password on first login.

---

## 🖥️ Change Hostname

> **Purpose:** A clear hostname helps identify the firewall easily, especially in multi-device networks.

Use the following CLI commands to change the hostname:

```bash
config system global          # Enter global system settings
set hostname Ritz             # Set the device hostname
end                           # Save and exit
```

---

## 🌐 Interface Configuration

> **Purpose:** Interface settings control how the firewall connects to the network. Below is a clean, properly spaced, easy-to-read version.

Configure **port1** with a static IP, admin access, and an alias:

```bash
config system interface            # Go to interface configuration
edit port1                         # Select interface port1
set mode static                    # Set static IP mode
set ip 192.168.1.99/24             # Assign IP address and subnet
set allowaccess http ping ssh      # Allow access methods
set alias "LAN1"                   # Friendly name for the interface
set role lan                       # Set interface role
end                                 # Save and exit
```

---

## 📶 Ping From CLI

> **Purpose:** Ping helps verify network connectivity from the firewall to another device.

> **Comment:** Use ping to verify network connectivity between the firewall and another device.

To test basic connectivity from the Fortigate CLI:

```bash
# execute ping <destination_ip>
```

Example:

```bash
Ritz # execute ping 8.8.8.8
```

---

## ✔️ Summary

This README covers:

* How to access a Fortigate device
* Default login information
* Hostname configuration
* Interface setup
* Connectivity testing via ping
