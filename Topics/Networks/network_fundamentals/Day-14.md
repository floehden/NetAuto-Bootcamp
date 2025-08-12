# RADIUS and TACACS+ Access Control Protocols 

Understand the RADIUS and TACACS+ protocols as part of AAA, and configure them on Cisco devices for secure access control.

---

## 1) RADIUS vs TACACS+

**RADIUS (Remote Authentication Dial-In User Service):**

* Open standard protocol (RFC 2865) for centralized authentication, authorization, and accounting.
* Combines authentication and authorization in a single process.
* Encrypts only the password field in access requests.
* Typically uses UDP ports 1812 (authentication) and 1813 (accounting).
* Widely used for network access (VPN, wireless, wired 802.1X).

**TACACS+ (Terminal Access Controller Access-Control System Plus):**

* Cisco-proprietary protocol for AAA.
* Separates authentication, authorization, and accounting.
* Encrypts the entire packet for secure communication.
* Uses TCP port 49.
* Often used for administrative access control on network devices.

**Comparison:**

| Feature        | RADIUS              | TACACS+               |
| -------------- | ------------------- | --------------------- |
| Standard       | Open (RFC)          | Cisco Proprietary     |
| Protocol       | UDP                 | TCP                   |
| Encryption     | Password only       | Entire packet         |
| AAA Separation | Combined auth+authz | Full separation       |
| Use Case       | Network access      | Device administration |

---

## 2) Lab Topology – 1 Cisco Router and AAA Servers

**Devices:**

* Cisco Router R1
* AAA Server running RADIUS
* AAA Server running TACACS+

**IP Scheme:**

* R1: 192.168.1.1
* RADIUS Server: 192.168.1.100
* TACACS+ Server: 192.168.1.101

---

## 3) Step-by-Step Configuration

**Cisco Router (R1) – RADIUS Configuration:**

```
R1> enable
R1# configure terminal
aaa new-model
username admin privilege 15 secret LocalP@ss123
radius server LAB_RADIUS
 address ipv4 192.168.1.100 auth-port 1812 acct-port 1813
 key RadiusKey123
aaa authentication login default group radius local
aaa authorization exec default group radius local
aaa accounting exec default start-stop group radius
line vty 0 4
 login authentication default
 transport input ssh
```

**Cisco Router (R1) – TACACS+ Configuration:**

```
tacacs server LAB_TACACS
 address ipv4 192.168.1.101
 key TacacsKey123
aaa authentication login default group tacacs+ local
aaa authorization exec default group tacacs+ local
aaa accounting exec default start-stop group tacacs+
```

---

## 4) Verification

* Test SSH logins using RADIUS and TACACS+ credentials.
* Review logs on RADIUS and TACACS+ servers to confirm authentication and authorization.
* Disconnect AAA servers to confirm local fallback.

---

## 5) Troubleshooting

* Ensure correct IP, port, and key for each server.
* Verify reachability with `ping`.
* Check method lists applied to VTY lines.

---

## References

1. Cisco. *Configuring RADIUS and TACACS+ for Device Administration.* [https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec\_usr\_tacacs/configuration/xe-17/sec-usr-tacacs-xe-17-book.html](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_usr_tacacs/configuration/xe-17/sec-usr-tacacs-xe-17-book.html)
2. Cisco. *RADIUS Configuration Guide, Cisco IOS XE.* [https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec\_usr\_radius/configuration/xe-17/sec-usr-radius-xe-17-book.html](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_usr_radius/configuration/xe-17/sec-usr-radius-xe-17-book.html)
