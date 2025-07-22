# Day 5: Lab as Code - Advanced Topologies and Customizations

## **Objective:** 
Explore more complex topology features and customization options.

1.  **Multiple Links and Link Properties:**
Containerlab allows you to define multiple links between nodes and add specific properties.
Create `day5-advanced-links-lab.yaml`:

```yaml
name: day5-advanced-links-lab
topology:
    nodes:
    ceos1:
        kind: ceos
        image: arista/ceos:latest
    srl1:
        kind: nokia_srlinux
        image: srlabs/srlinux:latest
    ceos2:
        kind: ceos
        image: arista/ceos:latest
    links:
    - endpoints: ["ceos1:eth1", "srl1:e1-1"]
        labels:
        link-type: "point-to-point" # Custom label
    - endpoints: ["srl1:e1-2", "ceos2:eth1"]
        mtu: 9000 # Set custom MTU for the link
    - endpoints: ["ceos1:eth2", "ceos2:eth2"] # Direct link between ceos nodes
```

Deploy, configure IPs, and verify connectivity and link properties (e.g., `show interface ethernet 1/2 mtu` on SR Linux).

2.  **Host Links:**
Connect a node's interface directly to the host's network namespace. This is useful for sniffing traffic on a specific link with Wireshark on the host.
Create `day5-host-link-lab.yaml`:

```yaml
name: day5-host-link-lab
topology:
    nodes:
    ceos1:
        kind: ceos
        image: arista/ceos:latest
    links:
    - endpoints: ["ceos1:eth1", "host:ceos1-to-host"] # Creates a veth pair on the host
```

Deploy the lab:

```bash
sudo clab deploy -t day5-host-link-lab.yaml
```

On your host, you should see a new interface named `ceos1-to-host`. You can then use `tcpdump` or Wireshark on this interface.

```bash
ip a show ceos1-to-host
sudo tcpdump -i ceos1-to-host
```

Inside `ceos1`, configure `eth1` with an IP and try to ping your host's `ceos1-to-host` IP (if configured).

3.  **Mounting Directories:**
You can mount host directories into your container nodes, which is useful for injecting configuration files, scripts, or collecting logs.
Update `day3-custom-config-lab.yaml` (or create a new one):

```yaml
name: day5-mounted-config-lab
topology:
    nodes:
    ceos1:
        kind: ceos
        image: arista/ceos:latest
        startup-config: ./ceos1-config.cfg
        extra-hosts: # Add custom host entries (optional)
        - "myhost:172.20.20.1"
        binds: # Mount host directories
        - "./scripts:/opt/scripts" # Mount local 'scripts' folder to '/opt/scripts' in container
    srl1:
        kind: nokia_srlinux
        image: srlabs/srlinux:latest
        startup-config: ./srl1-config.cli
        binds:
        - "./logs:/var/log/custom-logs" # Mount local 'logs' folder to '/var/log/custom-logs' in container
    links:
    - endpoints: ["ceos1:eth1", "srl1:e1-1"]
```

Create `scripts` and `logs` directories in your lab folder. Place a dummy script in `scripts`. After deployment, you can `docker exec -it` into the containers and verify the mounted directories.

4.  **Destroy the labs.**


