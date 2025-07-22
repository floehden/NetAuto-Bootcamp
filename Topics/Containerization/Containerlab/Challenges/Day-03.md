# Day 3: Custom Startup Configurations

## **Objective:** Learn how to apply custom configurations to your nodes upon deployment. This is crucial for building repeatable labs.

1.  **Prepare cEOS startup config:**
Create `ceos1-config.cfg`:

```
!
interface Ethernet1
    no shutdown
    ip address 192.168.1.1/24
!
router bgp 65001
    router-id 1.1.1.1
    neighbor 192.168.1.2 remote-as 65002
!
```

2.  **Prepare SR Linux startup config:**
Create `srl1-config.cli`:

```
set / interface ethernet-1/1 admin-state enable
set / interface ethernet-1/1 subinterface 0 admin-state enable
set / interface ethernet-1/1 subinterface 0 ipv4 address 192.168.1.2/24
set / system network-instance default protocols bgp admin-state enable
set / system network-instance default protocols bgp autonomous-system 65002
set / system network-instance default protocols bgp router-id 2.2.2.2
set / system network-instance default protocols bgp neighbor 192.168.1.1 peer-as 65001
```

3.  **Update `day2-mixed-lab.yaml` to use custom configs:**

```yaml
name: day3-custom-config-lab
topology:
    nodes:
    ceos1:
        kind: ceos
        image: arista/ceos:latest
        startup-config: ./ceos1-config.cfg # Path to your config file
    srl1:
        kind: nokia_srlinux
        image: srlabs/srlinux:latest
        startup-config: ./srl1-config.cli # Path to your config file
    links:
    - endpoints: ["ceos1:eth1", "srl1:e1-1"]
```

4.  **Deploy and verify:**

```bash
sudo clab deploy -t day3-custom-config-lab.yaml
```

SSH into `ceos1` and `srl1` and check `show ip interface brief` (cEOS) and `show interface ethernet-1/1` (SR Linux) and BGP summaries to confirm configurations were applied.

5.  **Test BGP Peering (from ceos1 CLI):**

```
show ip bgp summary
```

You should see the BGP peering with `192.168.1.2` in an `Established` state.

6.  **Test BGP Peering (from srl1 CLI):**

```
show network-instance default protocols bgp summary
```

You should see the BGP peering with `192.168.1.1` in an `Up` state.

7.  **Destroy the lab:**

```bash
sudo clab destroy -t day3-custom-config-lab.yaml
```

