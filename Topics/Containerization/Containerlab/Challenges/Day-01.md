# Day 1: Getting Started and Basic Topology

## **Objective:** 
Install Containerlab, verify installation, and deploy a simple two-node lab with Arista cEOS.

1.  **Install Containerlab:**
The easiest way is to use the official installation script:

```bash
curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"
```

This script will install Docker (if not present) and Containerlab. Log out and log back in for `sudo-less` Docker operation to take effect.

2.  **Verify Installation:**

```bash
clab version
docker info
```
You should see the Containerlab version and Docker daemon information.

3.  **Create your first topology (Arista cEOS):**
Create a file named `day1-ceos-lab.yaml`:

```yaml
name: day1-ceos-lab
topology:
  nodes:
    ceos1:
      kind: ceos
      image: arista/ceos:latest # Or specify your downloaded version, e.g., arista/ceos:4.34.1F
    ceos2:
      kind: ceos
      image: arista/ceos:latest # Or specify your downloaded version
  links:
    - endpoints: ["ceos1:eth1", "ceos2:eth1"]
```

4.  **Deploy the lab:**

```bash
sudo clab deploy -t day1-ceos-lab.yaml
```

You will see output indicating the creation of containers and links.

5.  **Access the nodes:**
Containerlab provides easy access. The default credentials for cEOS are `admin`/`admin`.

```bash
sudo clab inspect -t day1-ceos-lab.yaml # To see management IPs
ssh admin@<ceos1-mgmt-ip>
ssh admin@<ceos2-mgmt-ip>
```

Alternatively, you can use `docker exec`:

```bash
docker exec -it clab-day1-ceos-lab-ceos1 Cli
```

6.  **Basic Configuration (inside ceos1 CLI):**

```
enable
configure terminal
interface ethernet 1
  no shutdown
  ip address 10.0.0.1/24
exit
show ip interface brief
```

7.  **Basic Configuration (inside ceos2 CLI):**

```
enable
configure terminal
interface ethernet 1
  no shutdown
  ip address 10.0.0.2/24
exit
show ip interface brief
```

8.  **Test Connectivity (from ceos1 CLI):**

```
ping 10.0.0.2
```

You should see successful pings.

9.  **Destroy the lab:**
```bash
sudo clab destroy -t day1-ceos-lab.yaml
```
