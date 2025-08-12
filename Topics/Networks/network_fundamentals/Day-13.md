# AAA (Authentication, Authorization, and Accounting) on Cisco Routers

Understand AAA concepts and implement AAA on Cisco devices for secure, centralized access control.

---

## 1) AAA Overview

**Definition:**

* **Authentication:** Verifies the identity of the user or device.
* **Authorization:** Determines what actions an authenticated user can perform.
* **Accounting:** Records user activities for auditing and billing.

**Why AAA is Important:**

* Centralized management of user access.
* Enhanced security with external authentication servers (e.g., RADIUS, TACACS+).
* Detailed logging of administrative activities.

**AAA Methods:**

* **Local:** User credentials stored on the device.
* **Server-based:** Credentials and policies stored on an external AAA server.

---

## 2) Lab Topology – 1 Cisco Router and 1 AAA Server

**Devices:**

* Cisco Router R1
* AAA Server (running RADIUS or TACACS+)

**IP Scheme:**

* R1: 192.168.1.1
* AAA Server: 192.168.1.100

---

## 3) Step-by-Step Configuration

**Cisco Router (R1) – AAA with Local and RADIUS:**

```
R1> enable
R1# configure terminal
! Enable AAA
aaa new-model
! Local user
username admin privilege 15 secret LocalP@ss123
! Configure RADIUS server
radius server LAB_RADIUS
 address ipv4 192.168.1.100 auth-port 1812 acct-port 1813
 key RadiusKey123
! Apply AAA for login
aaa authentication login default group radius local
! Apply AAA for exec authorization
aaa authorization exec default group radius local
! Apply AAA for accounting
aaa accounting exec default start-stop group radius
! Enable VTY lines to use AAA
line vty 0 4
 login authentication default
 transport input ssh
```

---

## 4) Verification

* Test SSH login from a client.
* Check AAA server logs to confirm authentication, authorization, and accounting entries.
* Disconnect the AAA server to verify local authentication fallback.

---

## 5) Troubleshooting

* Ensure AAA server IP, ports, and key match on device and server.
* Verify the AAA server is reachable using `ping`.
* Confirm that the correct method list is applied to VTY lines.

---

## References

1. Cisco. *AAA Configuration Guide, Cisco IOS XE.* [https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec\_usr\_aaa/configuration/xe-17/sec-usr-aaa-xe-17-book.html](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_usr_aaa/configuration/xe-17/sec-usr-aaa-xe-17-book.html)
