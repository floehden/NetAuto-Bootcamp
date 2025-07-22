You're right to want to dive into Junos\! Juniper's offerings like cJunosEvolved and vMX/vSRX are popular in Containerlab. Adding specific challenges for each day will enhance the learning experience.

Here's the expanded 11-day tutorial, incorporating Junos and daily challenges.

-----

## 11-Day Containerlab Tutorial: Arista cEOS, Nokia SR Linux, and Juniper Junos

This tutorial will guide you through the basics of Containerlab, focusing on Arista cEOS, Nokia SR Linux, and Juniper Junos, with practical examples and daily challenges.

**Prerequisites:**

  * A Linux host (Ubuntu 20.04/22.04 or similar distribution recommended) or WSL2 on Windows.

  * `sudo` privileges.

  * Docker (Containerlab's setup script can install it).

  * Arista cEOS, Nokia SR Linux, and Juniper Junos container images. You will need to download these from their respective vendor portals or build them if using `vrnetlab` images.

      * **Arista cEOS:** Typically available as a `.tar.xz` file from Arista's support portal. After downloading, import it into Docker:
        ```bash
        docker import cEOS-lab-<version>.tar.xz arista/ceos:<version>
        ```
        (Replace `<version>` with your downloaded image version, e.g., `4.34.1F`)
      * **Nokia SR Linux:** Usually available as a `.tgz` file. Import similarly:
        ```bash
        docker import srlinux-<version>.tgz srlabs/srlinux:<version>
        ```
        (Replace `<version>` with your downloaded image version, e.g., `23.10.1`)
      * **Juniper cJunosEvolved / vMX / vSRX:** These are often delivered as `qcow2` files and require building into a Docker image using the `vrnetlab` framework. The Containerlab documentation has excellent guides for this. For `cJunosEvolved`, it's often recommended. You'll download a `qcow2` file, and then use `vrnetlab` to package it into a Docker image.
          * **Example for cJunosEvolved (conceptual, actual steps depend on `vrnetlab` version and Junos image):**
            1.  Download `junos-evo-install-*-qemu.qcow2` from Juniper Support.
            2.  Clone `vrnetlab` repository: `git clone https://github.com/vrnetlab/vrnetlab.git`
            3.  Navigate to the `vrnetlab/juniper/vjunosevolved` directory.
            4.  Place your downloaded `qcow2` file in this directory.
            5.  Build the Docker image: `make docker-build-qemu IMAGE=junos-evo-install-*-qemu.qcow2`
            6.  The resulting image will typically be named `vrnetlab/vr-vjunosevolved:<version>`.

-----

### Day 1: Getting Started and Basic Topology (Arista cEOS)

**Objective:** Install Containerlab, verify installation, and deploy a simple two-node lab with Arista cEOS.

1.  **Install Containerlab:**

    ```bash
    curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"
    ```

    Log out and log back in for `sudo-less` Docker operation to take effect.

2.  **Verify Installation:**

    ```bash
    clab version
    docker info
    ```

3.  **Create your first topology (`day1-ceos-lab.yaml`):**

    ```yaml
    name: day1-ceos-lab
    topology:
      nodes:
        ceos1:
          kind: ceos
          image: arista/ceos:latest # Or your downloaded version
        ceos2:
          kind: ceos
          image: arista/ceos:latest # Or your downloaded version
      links:
        - endpoints: ["ceos1:eth1", "ceos2:eth1"]
    ```

4.  **Deploy, Access, Configure, and Test:**

    ```bash
    sudo clab deploy -t day1-ceos-lab.yaml
    sudo clab inspect -t day1-ceos-lab.yaml # Get management IPs
    # ssh admin@<ceos1-mgmt-ip>
    # configure Ethernet1 with 10.0.0.1/24 on ceos1
    # configure Ethernet1 with 10.0.0.2/24 on ceos2
    # ping 10.0.0.2 from ceos1
    ```

5.  **Destroy the lab:**

    ```bash
    sudo clab destroy -t day1-ceos-lab.yaml
    ```

**Day 1 Challenge:**

  * **Challenge:** The `eth` interface names used in cEOS (e.g., `eth1`) often map to `Ethernet1` in the EOS CLI. What happens if you try to use `Ethernet1` directly in the `endpoints` definition in the YAML? Deploy and observe.
  * **Hint:** Containerlab expects the Linux interface name within the container's network namespace, not the NOS-specific CLI name.

-----

### Day 2: Introducing Nokia SR Linux and Mixed Topology

**Objective:** Deploy a lab with Nokia SR Linux, understand its specific access methods, and create a mixed topology.

1.  **Create simple SR Linux topology (`day2-srl-lab.yaml`):**

    ```yaml
    name: day2-srl-lab
    topology:
      nodes:
        srl1:
          kind: nokia_srlinux
          image: srlabs/srlinux:latest
        srl2:
          kind: nokia_srlinux
          image: srlabs/srlinux:latest
      links:
        - endpoints: ["srl1:e1-1", "srl2:e1-1"]
    ```

2.  **Deploy, Access, Configure, and Test:**

    ```bash
    sudo clab deploy -t day2-srl-lab.yaml
    # Access srl1 (ssh admin@<srl1-mgmt-ip> or docker exec -it clab-...-srl1 sr_cli)
    # Configure ethernet-1/1 with 10.0.0.1/24 on srl1
    # Configure ethernet-1/1 with 10.0.0.2/24 on srl2
    # Ping 10.0.0.2 from srl1
    ```

3.  **Mixed Topology Example (`day2-mixed-lab.yaml`):**

    ```yaml
    name: day2-mixed-lab
    topology:
      nodes:
        ceos1:
          kind: ceos
          image: arista/ceos:latest
        srl1:
          kind: nokia_srlinux
          image: srlabs/srlinux:latest
      links:
        - endpoints: ["ceos1:eth2", "srl1:e1-2"]
    ```

    Deploy and configure IPs on `ceos1:eth2` and `srl1:e1-2`.

4.  **Destroy labs:**

    ```bash
    sudo clab destroy -t day2-srl-lab.yaml
    sudo clab destroy -t day2-mixed-lab.yaml
    ```

**Day 2 Challenge:**

  * **Challenge:** SR Linux interfaces have both `ethernet-1/X` (CLI) and `e1-X` (Docker/Linux namespace) naming conventions. If you configure a Layer 3 interface on `srl1:e1-2` and `ceos1:eth2` to be in the same subnet, but then accidentally configure `ethernet-1/3` on SRL (instead of `ethernet-1/2`), what will happen to the ping? How would you quickly troubleshoot this discrepancy using `docker exec` and Linux commands inside the SRL container?
  * **Hint:** Inside the SR Linux container, `ip a` will show the Linux interface names, not the SR Linux CLI names directly.

-----

### Day 3: Custom Startup Configurations

**Objective:** Learn how to apply custom configurations to your nodes upon deployment.

1.  **Prepare `ceos1-config.cfg` and `srl1-config.cli` (as in original tutorial).**

2.  **Update `day3-custom-config-lab.yaml` to use custom configs:**

    ```yaml
    name: day3-custom-config-lab
    topology:
      nodes:
        ceos1:
          kind: ceos
          image: arista/ceos:latest
          startup-config: ./ceos1-config.cfg
        srl1:
          kind: nokia_srlinux
          image: srlabs/srlinux:latest
          startup-config: ./srl1-config.cli
      links:
        - endpoints: ["ceos1:eth1", "srl1:e1-1"]
    ```

3.  **Deploy and verify BGP peering.**

4.  **Destroy the lab.**

**Day 3 Challenge:**

  * **Challenge:** Modify the BGP configuration in `ceos1-config.cfg` to use a different local AS (e.g., `65003`) without changing `srl1-config.cli`. Deploy the lab. What happens to the BGP peering? How would you verify the BGP state from both devices and identify the misconfiguration?
  * **Hint:** Look for BGP neighbor state commands (`show ip bgp summary` on cEOS, `show network-instance default protocols bgp summary` on SR Linux) and inspect logs (`docker logs`).

-----

### Day 4: Managing Lab Lifecycle and Network Modes

**Objective:** Explore more `clab` commands for managing labs and understand different network modes.

1.  **Deploy a lab (e.g., `day3-custom-config-lab.yaml`).**
2.  **Inspect running labs (`clab inspect`, `docker ps`).**
3.  **Stopping and Starting a lab (`clab stop`, `clab start`).**
4.  **Different Network Modes (e.g., `network-mode: host` example).**
5.  **Removing unused images and containers (`docker system prune -f`).**

**Day 4 Challenge:**

  * **Challenge:** Deploy the `day3-custom-config-lab.yaml`. After it's running, intentionally remove a link between `ceos1` and `srl1` from the `day3-custom-config-lab.yaml` file, but *do not* destroy the lab. Now run `sudo clab deploy -t day3-custom-config-lab.yaml` again. What is the behavior of Containerlab? Will it remove the link, or will it ignore the change? How can you ensure the lab matches the updated YAML without a full `destroy` and `deploy`?
  * **Hint:** Containerlab's `deploy` command is generally idempotent for *adding* resources. For *removing* resources, you typically need to `destroy` and `deploy` or manually intervene.

-----

### Day 5: Introduction to Juniper Junos (cJunosEvolved)

**Objective:** Understand how to acquire and deploy a basic Juniper cJunosEvolved node.

**Prerequisites for Day 5:**

  * You *must* have built your cJunosEvolved Docker image by now (e.g., `vrnetlab/vr-vjunosevolved:latest`). If not, refer to the "Prerequisites" section at the top of this document or Containerlab's official documentation for detailed `vrnetlab` instructions.

<!-- end list -->

1.  **Create a simple Junos topology (`day5-junos-lab.yaml`):**

    ```yaml
    name: day5-junos-lab
    topology:
      nodes:
        junos1:
          kind: juniper_vjunosevolved # Or juniper_vmx, juniper_vsrx depending on your image
          image: vrnetlab/vr-vjunosevolved:latest # Or your specific image name/tag
      # No links for now, just a single node
    ```

2.  **Deploy the lab:**

    ```bash
    sudo clab deploy -t day5-junos-lab.yaml
    ```

    *Note: Junos nodes (especially cJunosEvolved, vMX, vSRX) can take several minutes to boot fully, as they are QEMU VMs wrapped in containers. Monitor with `docker logs -f clab-day5-junos-lab-junos1`.*

3.  **Access the Junos node:**

      * Default credentials are typically `root` (no password, or `root`/`Juniper` for some versions, or `admin`/`admin`). Check `vrnetlab` or Junos documentation for your specific image version.
      * **SSH (preferred):**
        ```bash
        sudo clab inspect -t day5-junos-lab.yaml # Get management IP
        ssh root@<junos1-mgmt-ip>
        ```
      * **CLI within container:**
        ```bash
        docker exec -it clab-day5-junos-lab-junos1 cli # Or just 'bash' then 'cli'
        ```

4.  **Basic verification (inside Junos CLI):**

    ```
    show interfaces terse
    show system uptime
    show version
    ```

5.  **Destroy the lab:**

    ```bash
    sudo clab destroy -t day5-junos-lab.yaml
    ```

**Day 5 Challenge:**

  * **Challenge:** Junos nodes, especially `vrnetlab` ones, can take a long time to boot. How can you automate checking if the Junos CLI is available via SSH *before* attempting to log in, using a simple bash script or Python (without going into full Ansible yet)?
  * **Hint:** Consider using `clab inspect` to get the IP, and then tools like `nc` (netcat) or `ssh` with a short timeout to probe the SSH port.

-----

### Day 6: Junos Only Topology and Configuration

**Objective:** Deploy a two-node Junos lab and configure basic connectivity.

1.  **Create a two-node Junos topology (`day6-junos-p2p.yaml`):**

    ```yaml
    name: day6-junos-p2p
    topology:
      nodes:
        junos1:
          kind: juniper_vjunosevolved
          image: vrnetlab/vr-vjunosevolved:latest
        junos2:
          kind: juniper_vjunosevolved
          image: vrnetlab/vr-vjunosevolved:latest
      links:
        - endpoints: ["junos1:et-0/0/0", "junos2:et-0/0/0"] # Or ge-0/0/0, xe-0/0/0 depending on kind
    ```

    *Note: Junos interfaces are typically `et-0/0/X`, `ge-0/0/X`, or `xe-0/0/X`. The `vrnetlab` kind maps `eth1` in Docker to `et-0/0/0`, `eth2` to `et-0/0/1`, etc.*

2.  **Deploy the lab:**

    ```bash
    sudo clab deploy -t day6-junos-p2p.yaml
    ```

3.  **Configure basic IPs and commit (inside Junos CLI for junos1):**

    ```
    edit
    set interfaces et-0/0/0 unit 0 family inet address 10.0.0.1/24
    set interfaces et-0/0/0 unit 0 description "Link to junos2"
    commit and-quit
    show interfaces terse et-0/0/0
    ```

    **Configure junos2 similarly:**

    ```
    edit
    set interfaces et-0/0/0 unit 0 family inet address 10.0.0.2/24
    set interfaces et-0/0/0 unit 0 description "Link to junos1"
    commit and-quit
    show interfaces terse et-0/0/0
    ```

4.  **Test Connectivity:**

    ```
    ping 10.0.0.2 source 10.0.0.1
    ```

5.  **Destroy the lab.**

**Day 6 Challenge:**

  * **Challenge:** Instead of manually configuring, try to apply a startup configuration file to the Junos nodes. Create `junos1-config.conf` and `junos2-config.conf` with the IP configurations, then add `startup-config: ./junos1-config.conf` to the node definitions in the YAML. Deploy the lab and verify.
  * **Hint:** Junos config format is different from Arista and Nokia. You'll need to use `set` commands or `full-configuration` blocks.

-----

### Day 7: Mixed Vendor Topology with Junos, Arista, and Nokia

**Objective:** Build a multi-vendor lab involving Junos, cEOS, and SR Linux, demonstrating interoperability.

1.  **Create a mixed topology (`day7-mixed-vendor-lab.yaml`):**

    ```yaml
    name: day7-mixed-vendor-lab
    topology:
      nodes:
        junos1:
          kind: juniper_vjunosevolved
          image: vrnetlab/vr-vjunosevolved:latest
        ceos1:
          kind: ceos
          image: arista/ceos:latest
        srl1:
          kind: nokia_srlinux
          image: srlabs/srlinux:latest
      links:
        - endpoints: ["junos1:et-0/0/0", "ceos1:eth1"] # Link Junos to cEOS
        - endpoints: ["ceos1:eth2", "srl1:e1-1"]     # Link cEOS to SR Linux
        - endpoints: ["srl1:e1-2", "junos1:et-0/0/1"] # Link SR Linux back to Junos (for a triangle)
    ```

2.  **Deploy the lab:**

    ```bash
    sudo clab deploy -t day7-mixed-vendor-lab.yaml
    ```

3.  **Configure each device:**

      * **junos1:**
          * `et-0/0/0`: `10.0.0.1/24` (towards ceos1)
          * `et-0/0/1`: `10.0.0.5/24` (towards srl1)
      * **ceos1:**
          * `Ethernet1`: `10.0.0.2/24` (towards junos1)
          * `Ethernet2`: `10.0.0.3/24` (towards srl1)
      * **srl1:**
          * `ethernet-1/1`: `10.0.0.4/24` (towards ceos1)
          * `ethernet-1/2`: `10.0.0.6/24` (towards junos1)

4.  **Test Connectivity:**

      * Ping `10.0.0.2` from `junos1`.
      * Ping `10.0.0.3` from `ceos1`.
      * Ping `10.0.0.4` from `srl1`.
      * Test ping across the entire triangle (e.g., `junos1` to `srl1` via `ceos1` - this will require static routes or a routing protocol).

5.  **Destroy the lab.**

**Day 7 Challenge:**

  * **Challenge:** Configure static routes on all three devices so that `junos1` can ping `srl1`'s `ethernet-1/1` interface (10.0.0.4) by traversing `ceos1`. Similarly, ensure full reachability across all directly connected subnets.
  * **Hint:** You'll need routes for `10.0.0.0/24`, `10.0.0.4/24`, and `10.0.0.6/24` on the appropriate nodes.

-----

### Day 8: Lab as Code - Advanced Topologies and Customizations

**Objective:** Explore more complex topology features and customization options.

1.  **Multiple Links and Link Properties:**
    Create `day8-advanced-links-lab.yaml` with custom labels and MTU (as in original Day 5). Deploy and verify.

2.  **Host Links:**
    Connect a node to the host (as in original Day 5). Use this to run `tcpdump` on your host and verify traffic.

3.  **Mounting Directories:**
    Mount local directories into your nodes (as in original Day 5). Test by placing a simple text file on your host and confirming you can `cat` it inside the container.

**Day 8 Challenge:**

  * **Challenge:** Create a simple text file on your host (e.g., `my-notes.txt`) and mount it to `/root/my-notes.txt` inside `junos1`. After deploying the lab, try to modify `my-notes.txt` from *within* the Junos CLI (if possible, though usually not directly in a mounted file) or via `docker exec`. What are the implications of directly modifying mounted files?
  * **Hint:** Direct modification from within the NOS container might be tricky for proprietary systems, but the file system mount itself will be visible. The primary benefit of mounts is *injecting* data.

-----

### Day 9: Automation with Containerlab and External Tools (Ansible Refinement)

**Objective:** Refine Ansible integration and automate initial configuration of the multi-vendor lab.

1.  **Prepare a multi-node BGP fabric (`day9-bgp-fabric.yaml`):**
    This time, include a Junos node as part of the BGP fabric.

    ```yaml
    name: day9-bgp-fabric
    topology:
      nodes:
        leaf1:
          kind: ceos
          image: arista/ceos:latest
        leaf2:
          kind: ceos
          image: arista/ceos:latest
        spine1:
          kind: nokia_srlinux
          image: srlabs/srlinux:latest
        spine2: # New Junos spine
          kind: juniper_vjunosevolved
          image: vrnetlab/vr-vjunosevolved:latest
      links:
        - endpoints: ["leaf1:eth1", "leaf2:eth1"]
        - endpoints: ["leaf1:eth2", "spine1:e1-1"]
        - endpoints: ["leaf2:eth2", "spine1:e1-2"]
        - endpoints: ["leaf1:eth3", "spine2:et-0/0/0"] # Link leaf1 to Junos spine
        - endpoints: ["leaf2:eth3", "spine2:et-0/0/1"] # Link leaf2 to Junos spine
    ```

2.  **Deploy the lab:**

    ```bash
    sudo clab deploy -t day9-bgp-fabric.yaml
    ```

3.  **Generate Ansible Inventory:**

    ```bash
    sudo clab inspect -t day9-bgp-fabric.yaml --format json > inventory.json
    ```

    Manually create `ansible_inventory.yaml` based on `inventory.json` (as discussed in original Day 6), ensuring correct `ansible_network_os` for each vendor.

4.  **Create Ansible Playbooks:**

      * **`01-configure_interfaces.yaml`:** Playbook to configure IP addresses on all links.
      * **`02-configure_bgp.yaml`:** Playbook to configure iBGP (or eBGP, based on your design) between leaves and spines.
      * **`03-verify_connectivity.yaml`:** Playbook to run ping/BGP summary checks.

    *Example snippet for Junos interface config within an Ansible playbook (requires `juniper.device` collection):*

    ```yaml
    - name: Configure Junos interfaces
      hosts: juniper_devices
      connection: local
      gather_facts: no
      tasks:
        - name: Apply interface configuration
          juniper.device.junos_config:
            lines:
              - set interfaces et-0/0/0 unit 0 family inet address 10.0.0.10/31
              - set interfaces et-0/0/1 unit 0 family inet address 10.0.0.12/31
            # Add other necessary config lines
    ```

5.  **Run Playbooks sequentially.**

6.  **Destroy the lab.**

**Day 9 Challenge:**

  * **Challenge:** Write an Ansible playbook that not only configures the interfaces and BGP but also attempts to establish loopback interfaces on all nodes and advertise them into BGP. Then, add a verification step in the playbook to check if the loopback IPs are reachable from all other nodes in the fabric.
  * **Hint:** You'll need to define loopback IP variables in your inventory and ensure BGP is configured to advertise them.

-----

### Day 10: Deeper Dive into Junos Features (Optional: EVPN/VXLAN Setup)

**Objective:** Explore a slightly more complex Junos-specific use case, such as a basic EVPN-VXLAN setup.

1.  **Prepare a Junos EVPN-VXLAN topology (`day10-junos-evpn.yaml`):**
    This will typically involve at least two Junos devices acting as VTEPs and possibly a third as a route reflector.

    ```yaml
    name: day10-junos-evpn
    topology:
      nodes:
        leaf1-junos:
          kind: juniper_vjunosevolved
          image: vrnetlab/vr-vjunosevolved:latest
        leaf2-junos:
          kind: juniper_vjunosevolved
          image: vrnetlab/vr-vjunosevolved:latest
        rr-junos:
          kind: juniper_vjunosevolved
          image: vrnetlab/vr-vjunosevolved:latest
      links:
        - endpoints: ["leaf1-junos:et-0/0/0", "rr-junos:et-0/0/0"]
        - endpoints: ["leaf2-junos:et-0/0/0", "rr-junos:et-0/0/1"]
        - endpoints: ["leaf1-junos:et-0/0/1", "leaf2-junos:et-0/0/1"] # For direct fabric link (optional)
    ```

2.  **Deploy the lab:**

    ```bash
    sudo clab deploy -t day10-junos-evpn.yaml
    ```

3.  **Apply Junos EVPN-VXLAN Configuration (manual or via Ansible):**
    This is complex and beyond full code examples here, but involves:

      * Configuring loopback interfaces for VTEP source.
      * Enabling BGP with EVPN address family and route reflection.
      * Configuring bridge domains and VXLAN encapsulation.
      * Creating a client (e.g., a simple Linux container or another network device) to attach to a VXLAN-enabled interface.

4.  **Verify EVPN State:**

    ```
    show evpn instance
    show evpn database
    show route table <VRF>.evpn.0
    ```

5.  **Destroy the lab.**

**Day 10 Challenge:**

  * **Challenge:** While EVPN-VXLAN is complex, try to configure a minimal setup where `leaf1-junos` and `leaf2-junos` can exchange host routes over EVPN. Don't worry about data plane forwarding yet, focus on control plane verification. Add simple Linux containers (e.g., `kind: linux`, `image: alpine/git`) to `leaf1-junos` and `leaf2-junos` to act as virtual hosts connected to the EVPN bridge.
  * **Hint:** The Containerlab documentation has examples for `linux` nodes. You'll need to use VLANs and possibly bridge configuration on the Junos nodes to connect the Linux hosts into the VXLAN.

-----

### Day 11: Advanced Use Cases, Community, and Troubleshooting

**Objective:** Explore more advanced Containerlab features, community resources, and common troubleshooting techniques.

1.  **Containerlab Graph Visualization:**

    ```bash
    sudo clab graph -t day9-bgp-fabric.yaml
    ```

    Use `dot` to convert to PNG.

2.  **Monitoring and Telemetry Integration (Conceptual):**
    Discuss how to integrate Prometheus/Grafana or other tools by exposing management ports or using `ext-container` for collector nodes.

3.  **CI/CD Integration (Conceptual):**
    Reiterate how Containerlab's "lab-as-code" fits into automated testing pipelines.

4.  **Common Troubleshooting Techniques:**

      * `docker logs -f <container_name>`: Essential for checking NOS boot issues or errors.
      * `docker exec -it <container_name> bash`: Get a shell inside the container to use Linux commands (`ip a`, `ping`, `tcpdump`).
      * `clab inspect`: Verify assigned IPs and ports.
      * **Resource Management:** If nodes fail to start or are sluggish, check host CPU/RAM (`top`, `htop`, `free -h`).
      * **Image Issues:** Ensure images are correctly named and imported (`docker images`).
      * **Interface Mismatches:** Double-check `endpoints` in YAML against actual Linux interface names within containers.

5.  **Explore More Lab Examples:**

      * [Containerlab Lab Examples](https://www.google.com/search?q=https://containerlab.dev/lab-examples/)
      * Pay special attention to the `juniper` examples on the Containerlab site.

6.  **Join the Community:**

      * Containerlab GitHub: [https://github.com/srl-labs/containerlab](https://github.com/srl-labs/containerlab)
      * Containerlab Discord channel.

**Day 11 Challenge:**

  * **Challenge:** You have deployed a complex lab (e.g., `day9-bgp-fabric.yaml`). One of the nodes (e.g., `leaf1`) is not establishing BGP peering with `spine1`. Without destroying the lab, outline a step-by-step troubleshooting process using *only* `clab` and `docker` commands from your host, along with SSH/CLI access to the devices. Focus on identifying the root cause (e.g., interface down, wrong IP, BGP config mismatch, firewall).
  * **Hint:** Think about the layers of troubleshooting: container status, network interface status, IP configuration, routing table, and finally, protocol-specific checks.

-----

This expanded tutorial provides a more challenging and comprehensive learning path, especially with the integration of Junos and specific challenges for each day\! Good luck\!