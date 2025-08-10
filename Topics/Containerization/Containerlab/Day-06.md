# Day 6: Automation with Containerlab and External Tools

## **Objective:** 
Integrate Containerlab with external automation tools like Ansible (basic introduction).

1.  **Prepare a multi-node lab for BGP fabric:**
Create `day6-bgp-fabric.yaml`:

```yaml
name: day6-bgp-fabric
topology:
    nodes:
        leaf1:
            kind: arista_ceos
            image: ceos:4.34.0F
            startup-config: |
                hostname leaf1
                interface Ethernet1
                    no shutdown
                    no switchport
                    ip address 10.0.0.1/31
                interface Ethernet2
                    no shutdown
                    no switchport
                    ip address 10.0.0.3/31
                router bgp 65000
                    router-id 1.1.1.1
                    neighbor 10.0.0.0 remote-as 65000
                    neighbor 10.0.0.2 remote-as 65000
        leaf2:
            kind: arista_ceos
            image: ceos:4.34.0F
            startup-config: |
                hostname leaf2
                interface Ethernet1
                    no shutdown
                    ip address 10.0.0.0/31
                router bgp 65000
                    router-id 2.2.2.2
                    neighbor 10.0.0.1 remote-as 65000
        spine1:
            kind: arista_ceos
            image: ceos:4.34.0F
            startup-config: |
                hostname spine1
                interface Ethernet1
                    no shutdown
                    ip address 10.0.0.2/31
                router bgp 65000
                    router-id 3.3.3.3
                    neighbor 10.0.0.3 remote-as 65000
    links:
        - endpoints: ["leaf1:eth1", "leaf2:eth1"]
        - endpoints: ["leaf1:eth2", "spine1:e1-1"]
```

*This lab shows a simple spine-leaf setup. Leaf1 peers with Leaf2 (cEOS-cEOS) and Spine1 (cEOS-SRL).*

2.  **Deploy the lab:**

```bash
sudo clab deploy -t day6-bgp-fabric.yaml
```

3.  **Basic Ansible Integration (conceptual):**
Containerlab makes it easy to integrate with Ansible by providing access to the management IPs.

* **Generate Ansible inventory:**

```bash
sudo clab inspect -t day6-bgp-fabric.yaml --format json > inventory.json
# You would then parse this JSON to create an Ansible inventory
```

A common pattern is to use a Python script or JQ to extract IPs from `clab inspect` output and create an `ansible_inventory.yaml`.

* **Example Ansible Inventory (`ansible_inventory.yaml`):**

```yaml
all:
    children:
    arista_ceos:
        hosts:
            leaf1:
                ansible_host: <leaf1-mgmt-ip>
            leaf2:
                ansible_host: <leaf2-mgmt-ip>
            spine1:
                ansible_host: <spine1-mgmt-ip>
        vars:
            ansible_user: admin
            ansible_password: admin
            ansible_network_os: arista.eos.eos
```

* **Example Ansible Playbook (`ping_devices.yaml`):**

```yaml
---
- name: Ping Arista cEOS devices
    hosts: arista_ceos
    gather_facts: no
    tasks:
    - name: Test connectivity to cEOS devices
        ansible.builtin.ping:

```

* **Run the playbook:**

```bash
ansible-playbook -i ansible_inventory.yaml ping_devices.yaml
```

(You'll need Ansible installed on your host.)

4.  **Verify BGP peerings**
    Log into leaf1 and spine1 and check BGP status.

5.  **Destroy the lab.**


