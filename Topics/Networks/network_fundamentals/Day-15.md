# Troubleshooting Methodologies & Diagnostic Tools 

Learn structured troubleshooting approaches and diagnostic tools for network devices to quickly identify and resolve issues.

---

## 1) Troubleshooting Methodologies

**Common Approaches:**

1. **Top-Down:** Start from the application layer down to the physical layer.
2. **Bottom-Up:** Start from the physical layer and move up the OSI model.
3. **Divide and Conquer:** Test at a midpoint in the path to narrow down the issue.
4. **Follow the Path:** Trace the packet path and test each hop.
5. **Swap and Test:** Replace suspected faulty components with known working ones.
6. **Comparative Analysis:** Compare with a working system or baseline configuration.

**Best Practices:**

* Document each step and test result.
* Change one variable at a time.
* Confirm the fix before closing the case.
* Use network diagrams for context.

---

## 2) Common Diagnostic Tools

* **ping:** Tests reachability and measures latency.
* **traceroute / trace:** Identifies path and possible delays.
* **show commands:** Device-specific commands to check status (e.g., `show ip interface brief`).
* **debug commands:** Detailed logging for troubleshooting (use with caution).
* **packet capture:** Using tools like Wireshark for deep traffic inspection.
* **log analysis:** Review syslogs and event logs for clues.
* **SNMP monitoring:** Continuous health tracking.

---

## 3) Lab Example – Cisco & Arista

**Scenario:** A PC cannot reach a server across the network.

**Cisco Router Diagnostic Steps:**

```
R1> enable
R1# ping 192.168.1.10
R1# traceroute 192.168.1.10
R1# show ip route
R1# show interface gigabitEthernet0/0
R1# debug ip icmp
```

**Arista Switch Diagnostic Steps:**

```
S1> enable
S1# ping 192.168.1.10
S1# traceroute 192.168.1.10
S1# show ip route
S1# show interfaces Ethernet1
S1# monitor session capture
```

---

## 4) Verification

* Issue is identified and resolved (e.g., wrong VLAN, missing route).
* Verify by retesting connectivity and ensuring logs show no errors.

---

## 5) Troubleshooting Checklist

* [ ] Confirm physical connectivity (cables, interfaces up).
* [ ] Verify correct IP addressing and subnet mask.
* [ ] Check VLAN and trunk configurations.
* [ ] Confirm routing table entries.
* [ ] Test reachability to each hop.
* [ ] Review logs for anomalies.

---

## References

1. Cisco. *Troubleshooting IP Connectivity.* [https://www.cisco.com/c/en/us/support/docs/ip/troubleshooting/13714-12.html](https://www.cisco.com/c/en/us/support/docs/ip/troubleshooting/13714-12.html)
2. Arista. *EOS User Manual – Troubleshooting Tools.* [https://www.arista.com/en/um-eos/eos-troubleshooting](https://www.arista.com/en/um-eos/eos-troubleshooting)
