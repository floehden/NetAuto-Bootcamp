# Day 3: Custom Startup Configurations

## **Objective:** Learn how to apply custom configurations to your nodes upon deployment. This is crucial for building repeatable labs.

1.  **Prepare cEOS 1 startup config:**
Create `ceos1-config.cfg`:

```
!
interface Ethernet1
    no shutdown
    no switchport
    ip address 192.168.1.1/24
!
router bgp 65001
    router-id 1.1.1.1
    neighbor 192.168.1.2 remote-as 65002
!
```

2.   **Prepare cEOS 2 startup config:**
Create `ceos2-config.cfg`:

```
!
interface Ethernet1
    no shutdown
    no switchport
    ip address 192.168.1.2/24
!
router bgp 65001
    router-id 2.2.2.2
    neighbor 192.168.1.1 remote-as 65002
!
```


3.  **Update `day2-mixed-lab.yaml` to use custom configs:**

```yaml
name: day3-custom-config-lab
topology:
  nodes:
    ceos1:
      kind: arista_ceos
      image: ceos:4.34.0F
      startup-config: ./ceos1-config.cfg # Path to your config file
    ceos2:
      kind: arista_ceos
      image: ceos:4.34.0F
      startup-config: ./ceos2-config.cfg # Path to your config file
  links:
      - endpoints: ["ceos1:eth1", "ceos2:eth1"]
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

