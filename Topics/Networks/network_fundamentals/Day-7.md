# Router Architecture, Static Routing & Dynamic Routing Overview – 1-Day Theory Course

**Goal:** Gain a deep understanding of router architecture, the principles of static routing, and an overview of dynamic routing using verified definitions and sources.

---

## 1) Router Architecture

A router is a network device that connects two or more packet-switched networks or subnetworks. It manages traffic between them by forwarding data packets to their intended IP addresses. It also allows multiple devices to share the same internet connection.

Routers operate at Layer 3 of the OSI model and have three operational planes:

* **Control Plane:** Runs routing protocols (e.g., OSPF, BGP), computes routes, and updates the routing table.
* **Data Plane (Forwarding Plane):** Forwards packets using the forwarding information base (FIB).
* **Management Plane:** Interfaces for configuration, monitoring, and administration.

**Key Hardware and Software Components:**

* **CPU:** Runs control plane processes.
* **Memory:**

  * **RAM:** Stores routing table, ARP table, running configuration.
  * **NVRAM:** Stores the startup configuration.
  * **Flash:** Stores operating system images.
* **Interfaces:** Physical and logical ports.

---

## 2) Static Routing

Static routing is the process of manually configuring routes in a router’s routing table. These routes do not change unless manually updated or removed by an administrator.

**Advantages:**

* Predictable and secure.
* No bandwidth used for routing protocol updates.

**Disadvantages:**

* No automatic failover.
* Manual maintenance is required for network changes.

**Example (Cisco IOS):**

```
Router(config)# ip route 192.168.2.0 255.255.255.0 10.1.1.2
```

**Example (Arista EOS):**

```
router(config)# ip route 192.168.2.0/24 10.1.1.2
```

---

## 3) Dynamic Routing Overview

Dynamic routing uses routing protocols to automatically discover and maintain routing information. Routers exchange routing updates and adapt to network topology changes without manual intervention.

**Advantages:**

* Automatic adaptation to changes.
* Suitable for large and complex networks.

**Disadvantages:**

* Higher CPU and memory usage.
* More complex to configure than static routes.

**Types of Dynamic Routing Protocols:**

* **Distance Vector:** Shares entire routing tables periodically (e.g., RIP, EIGRP).
* **Link State:** Advertises link states to build a network map (e.g., OSPF, IS-IS).
* **Path Vector:** Shares paths between autonomous systems (e.g., BGP).

---

## 4) Key Takeaways

* Routers connect separate networks and forward packets based on Layer 3 addressing.
* Static routing offers simplicity but requires manual updates.
* Dynamic routing scales better and adapts automatically.
* Cisco IOS and Arista EOS have similar routing principles but slightly different syntax.

---

## References

1. Cisco. "What is a Router?" [https://www.cisco.com/site/us/en/learn/topics/small-business/what-is-a-router.html](https://www.cisco.com/site/us/en/learn/topics/small-business/what-is-a-router.html)
2. Cisco. *IP Routing Configuration Guide, Cisco IOS XE 17.x.* [https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/ip-routing/b-ip-routing.html](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/ip-routing/b-ip-routing.html)
3. Cisco. "Static Routes" module (IOS XE 17.x). [https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/ip-routing/b-ip-routing/m\_iri-iprouting.html](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/ip-routing/b-ip-routing/m_iri-iprouting.html)
4. Cisco. "Configure a Next Hop IP Address for Static Routes." [https://www.cisco.com/c/en/us/support/docs/dial-access/floating-static-route/118263-technote-nexthop-00.html](https://www.cisco.com/c/en/us/support/docs/dial-access/floating-static-route/118263-technote-nexthop-00.html)
5. Arista. *EOS User Manual — IPv4.* [https://www.arista.com/en/um-eos/eos-ipv4](https://www.arista.com/en/um-eos/eos-ipv4)
6. Arista. *EOS User Manual — Routing Protocols.* [https://www.arista.com/en/um-eos/eos-routing-protocols](https://www.arista.com/en/um-eos/eos-routing-protocols)
