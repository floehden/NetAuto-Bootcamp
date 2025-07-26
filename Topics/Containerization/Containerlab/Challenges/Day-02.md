# Day 2: Introducing Nokia SR Linux and Mixed Topology

## **Objective:** 
Deploy a lab with Nokia SR Linux, understand its specific access methods, and create a mixed topology.

1.  **Create a simple SR Linux topology:**
Create `day2-srl-lab.yaml`:

```yaml
name: day2-srl-lab
topology:
  nodes:
    srl1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest # Or specify your downloaded version, e.g., srlabs/srlinux:23.10.1
    srl2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest # Or specify your downloaded version
  links:
  - endpoints: ["srl1:e1-1", "srl2:e1-1"]
```

*Note: SR Linux interfaces are typically `e1-X` or `ethernet-1/X`.*

2.  **Deploy the lab:**

```bash
sudo clab deploy -t day2-srl-lab.yaml
```

3.  **Access SR Linux nodes:**
Default credentials for SR Linux are `admin`/`NokiaSrl1!`.

* **SSH (preferred):**
```bash
sudo clab inspect -t day2-srl-lab.yaml # Get management IPs
ssh admin@<srl1-mgmt-ip>
ssh admin@<srl2-mgmt-ip>
```
* **SR Linux CLI (inside container):**
```bash
docker exec -it clab-day2-srl-lab-srl1 sr_cli
```
* **Bash shell (inside container):**
```bash
docker exec -it clab-day2-srl-lab-srl1 bash
```

4.  **Basic Configuration (inside srl1 CLI):**

```
enter candidate
set interface ethernet-1/1 admin-state enable
set interface ethernet-1/1 subinterface 0 admin-state enable
set interface ethernet-1/1 subinterface 0 ipv4 address 10.0.0.1/24
commit now
show interface ethernet-1/1
quit
```

5.  **Basic Configuration (inside srl2 CLI):**

```
enter candidate
set interface ethernet-1/1 admin-state enable
set interface ethernet-1/1 subinterface 0 admin-state enable
set interface ethernet-1/1 subinterface 0 ipv4 address 10.0.0.2/24
commit now
show interface ethernet-1/1
quit
```

6.  **Test Connectivity (from srl1 CLI):**

```
ping 10.0.0.2
```

You should see successful pings.

7.  **Mixed Topology Example:**
Create `day2-mixed-lab.yaml`:

```yaml
name: day2-mixed-lab
topology:
    nodes:
        ceos1:
            kind: arista_ceos
            image: ceos:4.34.0F
        srl1:
            kind: nokia_srlinux
            image: ghcr.io/nokia/srlinux:latest
    links:
        - endpoints: ["ceos1:eth2", "srl1:e1-2"] # Note: Interface numbers can differ
```

Deploy and configure IPs on the connected interfaces (e.g., `ceos1:eth2` and `srl1:e1-2`) in the same subnet to test connectivity.

8.  **Destroy the labs:**

```bash
sudo clab destroy -t day2-srl-lab.yaml
sudo clab destroy -t day2-mixed-lab.yaml
```
