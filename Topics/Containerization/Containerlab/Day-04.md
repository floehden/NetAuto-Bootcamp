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

4.  **Saving the current running configuration as inital configuration**

```bash
containerlab save -t day3-custom-config-lab.yaml
```

5.  **Removing unused images and containers:**
After destroying labs, it's good practice to clean up Docker:

```bash
docker system prune -f
```

This removes stopped containers, networks not used by at least one container, dangling images, and build cache.

