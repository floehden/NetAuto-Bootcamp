# **Day 6: Introduction to Containerlab**

## **Introduction:** 
* What is Containerlab? 
* Why use it for network labs? 
* Basic topology definition (YAML). Deploying and destroying labs.

## **Pre-requisite:** 
Ensure Docker is installed and `arista/ceos` image is imported.


## **Code Example Nokia (Containerlab Topology):**
```yaml
# srl_single.clab.yaml
name: srl-single
topology:
  nodes:
    srl1:
      kind: srl
      image: ghcr.io/nokia/srlinux # Use an appropriate SR Linux image version
      startup-config: |
        system {
            name {
            host-name srl1
            }
        }
      ports:
      - 57400:57400 # gNMI/gRPC
      - 830:830 # NETCONF over SSH
      - 22:22 # SSH
```
### **Instructions:**
1.  Save the YAML as `srl_single.clab.yaml`.
2.  Deploy the lab: `sudo containerlab deploy -t srl_single.clab.yaml`.
3.  Access the CLI: `docker exec -it clab-srl-single-srl1 sr_cli`.
4.  Check status: `show system information`.
5.  Destroy the lab when done: `sudo containerlab destroy -t srl_single.clab.yaml`.



## **Code Example Arista: Simple Containerlab Topology (YAML)**

* Create `simple_lab.clab.yaml`:

```yaml
name: simple-arista-lab
topology:
  nodes:
    ceos1:
      kind: arista_ceos
      image: ceos:4.34.0F # e.g., ceos:4.30.6M
    ceos2:
      kind: arista_ceos
      image: ceos:4.34.0F
  links:
    - endpoints: ["ceos1:eth1", "ceos2:eth1"]
```

* Deploy: `sudo containerlab deploy -t simple_lab.clab.yaml --reconfigure` 
* Destroy: `sudo containerlab destroy -t simple_lab.clab.yaml --cleanup`

## **Challenge 6:** 
Deploy the `simple_lab.yaml`. Use `sudo containerlab graph -t simple_lab.clab.yaml` to visualize the lab. Log into `ceos1` via `docker exec -it clab-simple-arista-lab-ceos1 Cli` and verify interfaces. Then destroy the lab.
