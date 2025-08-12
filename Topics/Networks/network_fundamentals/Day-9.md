# OSPF

**Goal:** Understand OSPF (Open Shortest Path First) concepts and implement them on a simulated network with four Arista routers.

---

## 1) OSPF

OSPF is a link-state Interior Gateway Protocol (IGP) that uses the Shortest Path First (SPF) algorithm to determine the best path to each network in a topology. It supports variable-length subnet masks (VLSM), route summarization, and converges quickly after network changes.

**Key Points:**

* Operates at Layer 3 of the OSI model.
* Uses areas to organize and optimize routing (Area 0 is the backbone).
* Exchanges link-state advertisements (LSAs) to build a complete topology map.
* Path cost is calculated based on interface bandwidth.

**Advantages:**

* Fast convergence.
* Efficient hierarchical design.
* Scales for medium to large networks.

---

## 2) Lab Topology – 4 Arista Routers

**Devices:**

* R1, R2, R3, (Cisco in GNS3 )
* Arranged in a square topology with R1–R2–R3, 

**IP Scheme:**

* R1–R2: 10.0.12.0/30
* R2–R3: 10.0.23.0/30

* Loopbacks:

  * R1: 192.168.1.0/24
  * R2: 192.168.2.0/24
  * R3: 192.168.3.0/24

  <p align="center">
  <img src="img/routes.png" alt="Static Routing Lab">
</p>

---

## 3) Step-by-Step Configuration (Arista EOS)

**On R1:**

```
configure terminal
hostname R1
interface Ethernet1/0
 ip address 10.0.12.1 255.255.255.252
 no shutdown
interface Loopback0
 ip address 192.168.1.1 255.255.255.0
router ospf 1
 network 10.0.12.0 0.0.0.3 area 0
 network 192.168.1.0 0.0.0.255 area 0
end
write memory
```

**On R2:**

```
configure terminal
hostname R2
interface Ethernet1/0
 ip address 10.0.12.2 255.255.255.252
 no shutdown
interface Ethernet1/1
 ip address 10.0.23.1 255.255.255.252
 no shutdown
interface Loopback0
 ip address 192.168.2.1 255.255.255.0
router ospf 1
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.23.0 0.0.0.3 area 0
 network 192.168.2.0 0.0.0.255 area 0
end
write memory
```

**On R3:**

```
configure terminal
hostname R3
interface GigabitEthernet0/0/0
 ip address 10.0.23.2 255.255.255.252
 no shutdown
interface Loopback0
 ip address 192.168.3.1 255.255.255.0
router ospf 1
 network 10.0.23.0 0.0.0.3 area 0
 network 192.168.3.0 0.0.0.255 area 0
end
write memory
```
---

## 4) Verification

* Use `ping` from each router to reach all loopbacks.
* Check OSPF neighbor relationships:

```
show ip ospf neighbor
```

* Check routing table for OSPF-learned routes:

```
show ip route 
```
For example:
```
O        10.0.23.0/30 [110/20] via 10.0.12.2, 00:00:14, Ethernet1/0
O        192.168.2.1/32 [110/11] via 10.0.12.2, 00:00:14, Ethernet1/0
```

---

## 5) Troubleshooting

* Ensure all interfaces are `no shutdown`.
* Verify correct subnet masks.
* Ensure all routers are in the same OSPF area for this lab.

---

## Challenge

Try same configuration, IP scheme but with Arista
