# Cisco Device Security & Configuration Management

**Goal:** Learn how to secure network devices and manage configurations effectively using Cisco routers.

---

## 1) Device Security & Configuration Management

**Why Security Matters:**

* Network devices are critical entry points to the network.
* Without proper security, unauthorized users can disrupt operations or access sensitive data.

**Common Security Measures:**

* **Passwords:** Secure access to user and privileged EXEC modes.
* **Banners:** Display legal or security warnings before login.
* **Secure Protocols:** Use SSH instead of Telnet for encrypted remote access.
* **Role-Based Access Control:** Limit what commands users can run.
* **Configuration Backups:** Regularly save and store configuration files securely.
* **Configuration Recovery:** Have a process for restoring devices in case of failure.

**Configuration Management Concepts:**

* **Running Configuration:** Active settings in RAM.
* **Startup Configuration:** Saved settings in NVRAM.
* **Backup & Restore:** Export configs to a TFTP/FTP/SCP server.

---

## 2) Lab Topology – 2 Cisco Routers

**Devices:**

* Cisco Router R1
* Cisco Router R2

---

## 3) Step-by-Step Configuration

**Cisco Router (R1) – Secure Access & Save Config:**

```
R1> enable
R1# configure terminal
! Set enable secret password
enable secret StrongP@ss123
! Set console password
line console 0
 password ConsoleP@ss
 login
 exit
! Set SSH access
hostname R1
ip domain-name lab.local
crypto key generate rsa modulus 2048
ip ssh version 2
username admin privilege 15 secret SSHp@ss123
line vty 0 4
 login local
 transport input ssh
 exit
! Save configuration
end
write memory
```

**Cisco Router (R2) – Secure Access & Save Config:**

```
R2> enable
R2# configure terminal
! Set enable secret password
enable secret StrongP@ss123
! Set console password
line console 0
 password ConsoleP@ss
 login
 exit
! Set SSH access
hostname R2
ip domain-name lab.local
crypto key generate rsa modulus 2048
ip ssh version 2
username admin privilege 15 secret SSHp@ss123
line vty 0 4
 login local
 transport input ssh
 exit
! Save configuration
end
write memory
```

**Backup Configuration to TFTP Server:**

```
copy running-config tftp:
Address or name of remote host []? 192.168.100.10
Destination filename [r1-config]? r1-backup
```

**Restore Configuration from TFTP Server:**

```
copy tftp: running-config
Address or name of remote host []? 192.168.100.10
Source filename []? r1-backup
```

---

## 4) Verification

* Test SSH access: `ssh admin@<device-ip>`
* Check saved configuration: `show running-config` and `show startup-config`
* Confirm banners and passwords are working.

---

## 5) Troubleshooting

* Ensure RSA keys are generated before enabling SSH.
* Verify VTY lines are set to accept SSH.
* Confirm passwords meet complexity requirements.

---

## References

1. Cisco. *Secure Device Access Configuration Guide.* [https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec\_usr\_cfg/configuration/xe-17/sec-usr-cfg-xe-17-book.html](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_usr_cfg/configuration/xe-17/sec-usr-cfg-xe-17-book.html)
