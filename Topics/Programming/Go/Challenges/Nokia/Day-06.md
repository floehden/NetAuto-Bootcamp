# **Day 6: Introduction to Containerlab**

## **Concept:** 
Understanding Containerlab's role in creating virtual network labs with containers, its YAML topology definition, and basic commands.

## **Challenge:** 
Create a simple Containerlab topology with one Nokia SR Linux node. Deploy it, access its CLI, and check its basic status.
## **Code Example (Containerlab Topology):**
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

