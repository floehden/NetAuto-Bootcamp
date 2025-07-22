
# **Day 6: Introduction to Containerlab**

## **Introduction:** 
* What is Containerlab? 
* Why use it for network labs? 
* Basic topology definition (YAML). Deploying and destroying labs.

## **Pre-requisite:** Ensure Docker is installed and `arista/ceos` image is imported.

## **Code Example: Simple Containerlab Topology (YAML)**

* Create `simple_lab.yaml`:

```yaml
name: simple-arista-lab
topology:
    nodes:
    ceos1:
        kind: ceos
        image: arista/ceos:<your_ceos_version> # e.g., arista/ceos:4.30.6M
    ceos2:
        kind: ceos
        image: arista/ceos:<your_ceos_version>
    links:
    - endpoints: ["ceos1:eth1", "ceos2:eth1"]
```

* Deploy: `sudo containerlab deploy -t simple_lab.yaml --reconfigure` 
* Destroy: `sudo containerlab destroy -t simple_lab.yaml --cleanup`

## **Challenge 6:** 
Deploy the `simple_lab.yaml`. Use `sudo containerlab graph -t simple_lab.yaml` to visualize the lab. Log into `ceos1` via `docker exec -it clab-simple-arista-lab-ceos1 Cli` and verify interfaces. Then destroy the lab.
