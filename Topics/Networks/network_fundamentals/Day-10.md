# BGP 

Understand Border Gateway Protocol (BGP) concepts and implement them on a simulated network with four Arista routers.

---

## 1) BGP

BGP is a path-vector routing protocol primarily used to exchange routing information between autonomous systems (ASes). It is the protocol that powers the global internet, making routing decisions based on policies and path attributes rather than simply the shortest path.

**Key Points:**

* Operates at Layer 3 of the OSI model.
* External BGP (eBGP) connects different ASes, Internal BGP (iBGP) connects routers within the same AS.
* Uses TCP port 179 for reliable delivery of routing updates.
* Common attributes: AS-Path, Next-Hop, Local Preference, MED.
* Policies can influence route selection.

**Advantages:**

* Scalable to the size of the internet.
* Highly flexible routing policies.
* Supports route aggregation.

---

## 2) Lab Topology – 4 Cisco Routers

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

Normally, when you create a BGP neighbor with an IP, BGP uses the outgoing interface’s IP address as the source of TCP connections.

If you peer to a loopback address instead of the directly connected interface, that loopback doesn’t belong to any physical link so BGP must be told “use my loopback address as the source” when initiating sessions.

Without neighbor <IP> update-source Loopback0, the TCP connection fails because the neighbor expects packets from your loopback IP, but they come from the physical interface IP.

Using loopbacks for BGP is common because:

The peering stays up as long as any path exists between loopbacks.

It’s not tied to one specific physical interface.

---
* Autonomous Systems:

  * R1: AS 65001
  * R2: AS 65002
  * R3: AS 65003

<p align="center">
  <img src="img/routes.png" alt="Static Routing Lab">
</p>

---

## 3) Step-by-Step Configuration 

**On R1:**

```
conf t
hostname R1
!
interface Ethernet1/0
 ip address 10.0.12.1 255.255.255.252
 no shutdown
!
interface Loopback0
 ip address 192.168.1.1 255.255.255.0
!
ip route 192.168.2.1 255.255.255.255 10.0.12.2
ip route 192.168.3.1 255.255.255.255 10.0.12.2
!
router bgp 65001
 neighbor 192.168.2.1 remote-as 65002
 neighbor 192.168.2.1 update-source Loopback0
 neighbor 192.168.2.1 ebgp-multihop 2
 network 192.168.1.0 mask 255.255.255.0
end
wr mem


```

**On R2:**

```
conf t
hostname R2
!
interface Ethernet1/0
 ip address 10.0.12.2 255.255.255.252
 no shutdown
!
interface Ethernet1/1
 ip address 10.0.23.1 255.255.255.252
 no shutdown
!
interface Loopback0
 ip address 192.168.2.1 255.255.255.0
!
ip route 192.168.1.1 255.255.255.255 10.0.12.1
ip route 192.168.3.1 255.255.255.255 10.0.23.2
!
router bgp 65002
 neighbor 192.168.1.1 remote-as 65001
 neighbor 192.168.1.1 update-source Loopback0
 neighbor 192.168.1.1 ebgp-multihop 2
 neighbor 192.168.3.1 remote-as 65003
 neighbor 192.168.3.1 update-source Loopback0
 neighbor 192.168.3.1 ebgp-multihop 2
 !
 network 192.168.2.0 mask 255.255.255.0
 neighbor 192.168.1.1 next-hop-self
 neighbor 192.168.3.1 next-hop-self
end
wr mem
```

**On R3:**

```
conf t
hostname R3
!
interface Ethernet1/0
 ip address 10.0.23.2 255.255.255.252
 no shutdown
!
interface Loopback0
 ip address 192.168.3.1 255.255.255.0
!
ip route 192.168.2.1 255.255.255.255 10.0.23.1
ip route 192.168.1.1 255.255.255.255 10.0.23.1
!
router bgp 65003
 neighbor 192.168.2.1 remote-as 65002
 neighbor 192.168.2.1 update-source Loopback0
 neighbor 192.168.2.1 ebgp-multihop 2
 network 192.168.3.0 mask 255.255.255.0
end
wr mem

```

---

## 4) Verification

* Use `ping` from each router to reach all loopbacks.
* Check BGP neighbor relationships:

```
show ip bgp summary
```

* Check routing table for BGP-learned routes:

```
show ip route bgp
```
For example on R1:

```
B        192.168.2.0/24 [20/0] via 10.0.12.2, 00:03:37
B        192.168.3.0/24 [20/0] via 10.0.12.2, 00:03:06
```

---

## 5) Troubleshooting

* Ensure interfaces are `no shutdown`.
* Verify AS numbers and neighbor IP addresses match on both sides.
* Ensure TCP port 179 is not blocked in simulations.

---

## Challenge

Try same configuration, IP scheme but with Arista
