# Day 4: Managing Lab Lifecycle and Network Modes

## **Objective:** 
Explore more `clab` commands for managing labs and understand different network modes.

1.  **Deploy a lab (e.g., `day3-custom-config-lab.yaml` from yesterday).**

```bash
sudo clab deploy -t day3-custom-config-lab.yaml
```

2.  **Inspect running labs:**

```bash
sudo clab inspect # Shows all running labs
sudo clab inspect -t day3-custom-config-lab.yaml # Details for a specific lab
```

3.  **List containers:**

```bash
docker ps -a --filter label=containerlab
```

This shows all Docker containers managed by Containerlab.

4.  **Stopping and Starting a lab:**

```bash
sudo clab stop -t day3-custom-config-lab.yaml # Stops containers but keeps state
sudo clab start -t day3-custom-config-lab.yaml # Starts stopped containers
```

*Note: `clab stop` will not remove the containers, allowing you to resume later. `clab destroy` removes everything.*

5.  **Different Network Modes:**
Containerlab defaults to creating a Docker bridge network for management interfaces. You can also leverage:

* **Host Networking:** For advanced scenarios where you need direct access to host network interfaces.
```yaml
# Example for host networking (careful, can conflict with host IPs)
name: host-net-lab
topology:
    nodes:
    ceos1:
        kind: arista_ceos
        image: ceos:4.34.0F
        network-mode: host # This configures host networking for the management interface
    links:
    # Data plane links still work as usual
    - endpoints: ["ceos1:eth1", "ceos2:eth1"]
```
*Note: `network-mode: host` affects the management interface. Data plane links (`endpoints`) still create virtual Ethernet pairs.*

6.  **Removing unused images and containers:**
After destroying labs, it's good practice to clean up Docker:

```bash
docker system prune -f
```

    This removes stopped containers, networks not used by at least one container, dangling images, and build cache.

