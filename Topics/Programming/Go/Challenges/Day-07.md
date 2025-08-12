# **Day 7: Initial cEOS Configuration with Containerlab**

## **Introduction:** 
Providing startup configurations to cEOS nodes in Containerlab. Using `startup-config` field in the YAML. Accessing cEOS CLI.

## **Code Example: cEOS with Initial Config**

* Create `initial_config.clab.yaml`:

```yaml
name: ceos-configured-lab
topology:
  nodes:
    ceos1:
      kind: arista_ceos
      image: ceos:4.34.0F # e.g., ceos:4.30.6M
      startup-config: |
        hostname ceos1
        no aaa root
        username admin privilege 15 role network-admin secret sha512 $6$RxQ5ae0GOW6SAiCU$7qzQNGX2pSIWBYGF8Xh30lo/s418/diYEEZj9rPrTJiAkYv0s6AvjpTfUHMGz.a58Hg29Yy/nV0Zvplux0
        interface Ethernet1
          no switchport
          ip address 10.0.0.1/30
          description "Link to ceos2"
        management api http-commands
          no shutdown
          protocol http
    ceos2:
      kind: arista_ceos
      image: ceos:4.34.0F
      startup-config: |
        hostname ceos2
        no aaa root
        username admin privilege 15 role network-admin secret sha512 $6$RxQ5ae0GOW6SAiCU$7qzQNGX2pSIWBYGF8Xh30lo/s418/diYEEZj9rPrTJiAkYv0s6AvjpTfUHMGz.a58Hg29Yy/nV0Zvplux0
        interface Ethernet1
          no switchport
          ip address 10.0.0.2/30
          description "Link to ceos1"
        management api http-commands
          no shutdown
          protocol http
  links:
    - endpoints: ["ceos1:eth1", "ceos2:eth1"]
```

* Deploy: `sudo containerlab deploy -t initial_config.clab.yaml --reconfigure`

## **Challenge 7:** 
Deploy `initial_config.clab.yaml`. SSH into `ceos1` (Containerlab usually maps port 22 to a random high port on the host, you can find it with `docker port <container_name>`). Verify the `Ethernet1` configuration and ping `ceos2`'s `Ethernet1` IP. Destroy the lab.
