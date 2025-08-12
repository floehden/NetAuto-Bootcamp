# Cisco Router CLI & Interface Configuration

This guide explains how to navigate the Command Line Interface (CLI) of Cisco routers and configure interfaces.

---

## 1) Cisco Router CLI & Interfaces

**CLI Overview:**

* The CLI is a text-based interface used for configuring, monitoring, and troubleshooting routers.
* Common CLI modes:

  * **User EXEC mode:** Limited monitoring commands.
  * **Privileged EXEC mode:** Access to all monitoring and configuration commands.
  * **Global Configuration mode:** System-wide configuration.
  * **Interface Configuration mode:** Interface-specific settings.

**Interface Basics:**

* Interfaces are physical or virtual ports on a router.
* Each interface can be assigned an IP address, speed/duplex settings, and operational states.
* Common interface types:

  * **Ethernet** (e.g., FastEthernet, GigabitEthernet)
  * **Loopback** (virtual interface for testing/management)

**Key Cisco Commands:**

* `show running-config` – Displays active configuration.
* `show ip interface brief` – Summarizes interface status and IP addresses.
* `interface` – Enters interface configuration mode.
* `ip address` – Assigns an IP address.
* `no shutdown` – Enables the interface.

---

## 2) Lab Topology – 2 Cisco Routers

**Devices:**

* Cisco Router R1
* Cisco Router R2

**IP Scheme:**

* R1–R2 link: 10.0.1.0/30

  * R1: 10.0.1.1/30
  * R2: 10.0.1.2/30

---

## 3) Step-by-Step Configuration

**Cisco Router R1:**

```
R1> enable
R1# configure terminal
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip address 10.0.1.1 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit
R1# write memory
```

**Cisco Router R2:**

```
R2> enable
R2# configure terminal
R2(config)# interface GigabitEthernet0/0
R2(config-if)# ip address 10.0.1.2 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit
R2# write memory
```

---

## 4) Verification

* Use `ping` to test connectivity between R1 and R2.
* Use `show ip interface brief` to check interface status.

---

## 5) Troubleshooting

* Ensure interfaces are in **up/up** state.
* Verify IP addresses are correct and in the same subnet.
* Check cable or simulated link connections.

---

## Reference

* Cisco. *Using the Command-Line Interface.* [https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/fundamentals/configuration/15-s/fun-15-s-book/cf\_cli.html](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/fundamentals/configuration/15-s/fun-15-s-book/cf_cli.html)
