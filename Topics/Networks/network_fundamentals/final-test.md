# Final Network Challenge – End-of-Course Assessment

**Goal:** Apply all theoretical and practical concepts from the course to troubleshoot, configure, and validate a multi-vendor network.

---

## Scenario Overview

You are the network engineer for a small enterprise that has recently expanded. The network is a mix of Cisco routers and Arista switches. Following a network upgrade, multiple connectivity issues have been reported.

**Devices:**

* **2 Cisco Routers:** R1 (HQ), R2 (Branch)
* **2 Arista Switches:** S1 (HQ), S2 (Branch)
* **1 Server:** Located at Branch (S2)
* **1 Client PC:** Located at HQ (S1)

**Network Setup:**

* R1 connected to S1, R2 connected to S2.
* S1 and S2 connected via a trunk link between HQ and Branch.
* Server on VLAN 20 (S2), Client PC on VLAN 10 (S1).
* Inter-VLAN routing intended to be handled by R1.
* Static routing configured between R1 and R2 for site-to-site communication.

---

## Reported Issues

1. Client PC cannot reach the Server.
2. Ping from R1 to the Server works.
3. Ping from R2 to the Client PC fails.
4. Some devices show inconsistent VLAN membership.

---

## Your Tasks

1. **VLAN Verification:**

   * Ensure all devices have correct VLAN assignments.
   * Confirm trunk ports are properly configured and allow required VLANs.

2. **Routing Validation:**

   * Verify inter-VLAN routing on R1.
   * Check static routes between R1 and R2.

3. **Connectivity Testing:**

   * Use `ping` and `traceroute` from multiple points in the network.
   * Identify where communication fails.

4. **Configuration Review:**

   * Use `show` commands on both Cisco and Arista devices.
   * Look for misconfigurations, shutdown interfaces, or incorrect IP addressing.

5. **Resolution:**

   * Fix identified problems.
   * Ensure end-to-end connectivity between Client PC and Server.

6. **Documentation:**

   * Record the troubleshooting process.
   * Document final working configurations.

---

## Success Criteria

* Client PC can ping and access the Server.
* All VLANs and trunk links are correctly configured.
* Routing tables are accurate on both Cisco and Arista devices.
* Network diagrams and configuration documentation are up to date.

---

## References

1. Cisco. *Troubleshooting IP Connectivity.* [https://www.cisco.com/c/en/us/support/docs/ip/troubleshooting/13714-12.html](https://www.cisco.com/c/en/us/support/docs/ip/troubleshooting/13714-12.html)
2. Arista. *EOS User Manual – Troubleshooting Tools.* [https://www.arista.com/en/um-eos/eos-troubleshooting](https://www.arista.com/en/um-eos/eos-troubleshooting)
