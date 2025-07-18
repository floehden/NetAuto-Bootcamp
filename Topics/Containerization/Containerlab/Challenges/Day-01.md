7-Day Containerlab Course: Arista cEOS & Nokia SR LinuxThis course is designed to provide a hands-on learning experience with Containerlab, focusing specifically on building and managing network topologies with Arista cEOS and Nokia SR Linux virtual network operating systems. Each day includes key concepts, practical examples, and a challenge to test your understanding.Day 1: Introduction to Containerlab & Basic SetupKey ConceptsWhat is Containerlab? A tool for orchestrating container-based networking labs. It allows you to build virtual network topologies using various Network Operating Systems (NOS) running as containers.Why use Containerlab?Lightweight: Uses containers, which are more resource-efficient than traditional VMs.Fast Deployment: Spin up complex topologies in seconds.Reproducible: Define your lab in a YAML file for consistent deployments.Automated: Integrates well with CI/CD pipelines and automation tools.Installation: Requires Docker (or Podman) and the containerlab binary.Basic Commandsclab deploy -t <file.yaml>: Deploys the lab.clab destroy -t <file.yaml>: Destroys the lab.clab inspect: Shows details about running labs.clab graph: Generates a visual graph of the lab.First Simple cEOS TopologyThis example creates a lab with two Arista cEOS nodes connected via a single link. It demonstrates the basic structure of a Containerlab topology file.Topology Diagram+-------+       +-------+
| ceos1 |-------| ceos2 |
+-------+       +-------+
Topology File: ceos-basic.yamlname: ceos-basic
topology:
  nodes:
    ceos1:
      kind: ceos
      image: arista/ceos:latest
      startup-config: |
        hostname ceos1
        interface Ethernet1
          no switchport
          ip address 10.0.0.1/24
        interface Management0
          ip address 192.168.1.101/24
          no shutdown
    ceos2:
      kind: ceos
      image: arista/ceos:latest
      startup-config: |
        hostname ceos2
        interface Ethernet1
          no switchport
          ip address 10.0.0.2/24
        interface Management0
          ip address 192.168.1.102/24
          no shutdown
  links:
    - endpoints: ["ceos1:eth1", "ceos2:eth1"]
CommandsUse these commands to deploy, access, and verify the lab.sudo clab deploy -t ceos-basic.yaml
```bash
# Access ceos1
docker exec -it clab-ceos-basic-ceos1 Cli

# Access ceos2
docker exec -it clab-ceos-basic-ceos2 Cli
Day 1 ChallengeYour Task:Modify the ceos-basic.yaml file to add a third cEOS node named ceos3.Connect ceos3 to ceos1 using a new link (e.g., ceos1:eth2 to ceos3:eth1).Assign appropriate IP addresses to the new link and the management interface of ceos3.Deploy the updated topology, verify connectivity between all nodes, and then destroy it.Day 2: Understanding Topology Files (YAML)Day 2 dives deep into the heart of Containerlab: the YAML topology file. You will learn about the required structure, key properties for defining nodes and links, and the specific interface naming conventions for both Arista cEOS and Nokia SR Linux. This knowledge is fundamental to building any custom lab.Key ConceptsYAML Structure: Containerlab files are written in YAML, which uses indentation to define structure.Root Elements: The main keys are name (required) and topology (required), which contains nodes and links.Nodes Section: Defines each device. Key properties include kind (e.g., 'ceos', 'srl'), image, and optional startup-config.Links Section: Defines connections using endpoints, which lists the two nodes and interfaces to connect.Interface NamingArista cEOS: e.g., Ethernet1, Management0Nokia SR Linux: e.g., e1-1, mgmt0Mixed cEOS and SR Linux TopologyThis example shows the power of Containerlab to mix vendors in a single topology. We connect one Arista cEOS node to one Nokia SR Linux node.Topology Diagram+-------+       +----------+
| ceos1 |-------| srlinux1 |
+-------+       +----------+
Topology File: mixed-lab.yamlname: mixed-lab
topology:
  nodes:
    ceos1:
      kind: ceos
      image: arista/ceos:latest
      startup-config: |
        hostname ceos1
        interface Ethernet1
          no switchport
          ip address 10.0.0.1/24
    srlinux1:
      kind: srl
      image: ghcr.io/nokia/srlinux
      startup-config: |
        set / interface ethernet-1/1 subinterface 0 ipv4 address 10.0.0.2/24
        set / system name srlinux1
  links:
    - endpoints: ["ceos1:eth1", "srlinux1:e1-1"]
Day 2 ChallengeYour Task:Create a topology file (triangle-lab.yaml) with three nodes: ceos1, srlinux1, and ceos2.Connect them in a triangle: ceos1 ↔ srlinux1 ↔ ceos2 ↔ ceos1.Assign a unique IP subnet to each of the three links.Deploy, verify basic pings across each link, and then destroy the lab.Day 3: Arista cEOS SpecificsToday focuses entirely on Arista cEOS. You will learn the essentials for configuring cEOS in Containerlab, including the startup-config syntax for interfaces and routing protocols. The example lab demonstrates how to build a three-router OSPF network, a common and fundamental routing scenario.Key ConceptsImage Acquisition: cEOS images are typically downloaded from Arista's official portal and loaded into Docker manually with docker load.Initial Configuration: Use the startup-config property with standard EOS/IOS-like CLI commands.L3 Interfaces: Remember to use no switchport on interfaces intended for routing.Accessing the CLI: Use docker exec -it <container_name> Cli to get an interactive shell.cEOS Network with OSPFThis lab builds a classic 3-router triangle, with all nodes running OSPF to exchange routing information. Each router advertises a loopback interface.Topology Diagram     +----+
     | r1 |
    /  |  \
   /   |   \
  +----+---+----+
  | r2 |---| r3 |
  +----+---+----+
Topology File: ceos-ospf.yamlname: ceos-ospf
topology:
  nodes:
    r1:
      kind: ceos
      image: arista/ceos:latest
      startup-config: |
        hostname r1
        interface Loopback0
          ip address 1.1.1.1/32
        interface Ethernet1
          no switchport
          ip address 10.0.1.1/24
        interface Ethernet2
          no switchport
          ip address 10.0.2.1/24
        router ospf 1
          network 0.0.0.0/0 area 0
    r2:
      kind: ceos
      image: arista/ceos:latest
      startup-config: |
        hostname r2
        interface Loopback0
          ip address 2.2.2.2/32
        interface Ethernet1
          no switchport
          ip address 10.0.1.2/24
        interface Ethernet2
          no switchport
          ip address 10.0.3.1/24
        router ospf 1
          network 0.0.0.0/0 area 0
    r3:
      kind: ceos
      image: arista/ceos:latest
      startup-config: |
        hostname r3
        interface Loopback0
          ip address 3.3.3.3/32
        interface Ethernet1
          no switchport
          ip address 10.0.2.2/24
        interface Ethernet2
          no switchport
          ip address 10.0.3.2/24
        router ospf 1
          network 0.0.0.0/0 area 0
  links:
    - endpoints: ["r1:eth1", "r2:eth1"]
    - endpoints: ["r1:eth2", "r3:eth1"]
    - endpoints: ["r2:eth2", "r3:eth2"]
Day 3 ChallengeYour Task:Add a fourth cEOS node (r4) to the OSPF lab.Connect r4 to r1 and r2.Configure OSPF on r4 and give it a unique loopback IP.Deploy and verify that all four routers can ping each other's loopback addresses.Day 4: Nokia SR Linux SpecificsWelcome to the world of Nokia SR Linux. This day covers the unique aspects of working with this modern, model-driven NOS. You'll learn the hierarchical set command syntax for configuration and how to build an ISIS routing domain, providing a contrast to the OSPF lab from Day 3.Key ConceptsImage Acquisition: SR Linux images are publicly available on GitHub Container Registry (ghcr.io), making them easy to access.Configuration Model: SR Linux uses a hierarchical, model-driven configuration. All configuration is done with set commands, followed by commit full to apply changes.CLI Access: Use docker exec -it <container_name> sr_cli to enter the interactive CLI.Operational State: Use info from state ... to view operational data, similar to show commands on other platforms.SR Linux Network with ISISThis lab builds the same 3-router triangle as Day 3, but this time using Nokia SR Linux and the ISIS routing protocol. This highlights the differences in configuration syntax and protocol choice.Topology Diagram     +-----+
     | sr1 |
    /   |   \
   /    |    \
  +-----+-----+
  | sr2 |-----| sr3 |
  +-----+-----+
Topology File: srlinux-isis.yamlname: srlinux-isis
topology:
  nodes:
    sr1:
      kind: srl
      image: ghcr.io/nokia/srlinux
      startup-config: |
        set / interface ethernet-1/1 subinterface 0 ipv4 address 10.0.1.1/24
        set / interface ethernet-1/2 subinterface 0 ipv4 address 10.0.2.1/24
        set / interface lo0 subinterface 0 ipv4 address 1.1.1.1/32
        set / router isis instance 1
        set / router isis instance 1 interface ethernet-1/1.0
        set / router isis instance 1 interface ethernet-1/2.0
        set / router isis instance 1 interface lo0.0
        commit full
    sr2:
      kind: srl
      image: ghcr.io/nokia/srlinux
      startup-config: |
        set / interface ethernet-1/1 subinterface 0 ipv4 address 10.0.1.2/24
        set / interface ethernet-1/2 subinterface 0 ipv4 address 10.0.3.1/24
        set / interface lo0 subinterface 0 ipv4 address 2.2.2.2/32
        set / router isis instance 1
        set / router isis instance 1 interface ethernet-1/1.0
        set / router isis instance 1 interface ethernet-1/2.0
        set / router isis instance 1 interface lo0.0
        commit full
    sr3:
      kind: srl
      image: ghcr.io/nokia/srlinux
      startup-config: |
        set / interface ethernet-1/1 subinterface 0 ipv4 address 10.0.2.2/24
        set / interface ethernet-1/2 subinterface 0 ipv4 address 10.0.3.2/24
        set / interface lo0 subinterface 0 ipv4 address 3.3.3.3/32
        set / router isis instance 1
        set / router isis instance 1 interface ethernet-1/1.0
        set / router isis instance 1 interface ethernet-1/2.0
        set / router isis instance 1 interface lo0.0
        commit full
  links:
    - endpoints: ["sr1:e1-1", "sr2:e1-1"]
    - endpoints: ["sr1:e1-2", "sr3:e1-1"]
    - endpoints: ["sr2:e1-2", "sr3:e1-2"]
Day 4 ChallengeYour Task:Add a fourth SR Linux node (sr4) to the ISIS lab.Connect sr4 to sr1 and sr2.Configure ISIS on sr4, give it a unique loopback IP, and ensure it forms adjacencies.Deploy and verify that all four routers can ping each other's loopback addresses.Day 5: Advanced Containerlab FeaturesNow that you're comfortable with the basics, Day 5 introduces advanced features. You'll learn how to connect your lab to external networks using bridges and how to persist configurations using bind mounts. This is key for creating more complex, stateful labs that survive reboots.Key ConceptsExternal Links: Use kind: bridge to create a Docker bridge that your lab nodes can connect to, providing a gateway to your host machine or other networks.Binding Mounts (binds): This powerful feature lets you mount a host directory or file into a container. It's the primary method for persisting configuration files.Lab Management: Explore additional commands like clab graph to visualize your topology (requires Graphviz) and clab tools netns for low-level network namespace debugging.Connecting to Host & Persisting ConfigThis example shows two advanced features: connecting a cEOS node to a host bridge and mounting a local directory to save its startup configuration.Topology Diagram+-------+       +------------+
| ceos1 |-------| Host Bridge|
+-------+       +------------+
Host Directory Setup# Create directory on host before deploying
mkdir ./ceos1-config
Topology File: host-connect.yamlname: host-connect
topology:
  nodes:
    ceos1:
      kind: ceos
      image: arista/ceos:latest
      binds:
        - ./ceos1-config:/mnt/flash
      startup-config: /mnt/flash/startup-config
  links:
    - endpoints: ["ceos1:eth1", "bridge:clab-host-br"]
Day 5 ChallengeYour Task:Create a topology with one SR Linux node.Configure a persistent mount for its configuration file (/etc/opt/srlinux/config.json).Deploy, make a change (e.g., add a loopback), save the config, destroy, and re-deploy to verify persistence.Use clab graph on your host to generate a PNG image of the topology.Day 6: Interoperability and Advanced RoutingDay 6 focuses on a real-world scenario: making different vendors talk to each other. You'll build a lab with both cEOS and SR Linux and configure them to peer using BGP. This day also introduces the concept of route redistribution, a critical skill for complex network integrations.Key ConceptsMulti-Vendor Topologies: Containerlab makes it trivial to mix and match different network operating systems in the same lab.BGP Peering: Learn the basic configuration to establish an eBGP session between a cEOS router and an SR Linux router.Route Redistribution: The technique of sharing routes learned from one routing protocol (like OSPF) into another (like BGP), and vice-versa.AS Numbers: BGP requires each router to be in an Autonomous System, identified by a unique number.Multi-Vendor BGP PeeringA simple but powerful lab showing an Arista cEOS router in AS 65001 peering directly with a Nokia SR Linux router in AS 65002. Each router advertises its loopback IP into BGP.Topology Diagram+----------------+       +----------------+
| AS 65001       |-------| AS 65002       |
| **ceos1** |       | **srlinux1** |
+----------------+       +----------------+
Topology File: multi-vendor-bgp.yamlname: multi-vendor-bgp
topology:
  nodes:
    ceos1:
      kind: ceos
      image: arista/ceos:latest
      startup-config: |
        hostname ceos1
        interface Loopback0
          ip address 1.1.1.1/32
        interface Ethernet1
          no switchport
          ip address 10.0.0.1/24
        router bgp 65001
          neighbor 10.0.0.2 remote-as 65002
          network 1.1.1.1/32
    srlinux1:
      kind: srl
      image: ghcr.io/nokia/srlinux
      startup-config: |
        set / interface ethernet-1/1 subinterface 0 ipv4 address 10.0.0.2/24
        set / interface lo0 subinterface 0 ipv4 address 2.2.2.2/32
        set / router bgp autonomous-system 65002
        set / router bgp neighbor 10.0.0.1 peer-as 65001
        set / router bgp network 2.2.2.2/32
        commit full
  links:
    - endpoints: ["ceos1:eth1", "srlinux1:e1-1"]
Day 6 ChallengeYour Task:Build a 3-node chain: ceos1 ↔ srlinux1 ↔ ceos2.Run BGP between ceos1 and srlinux1.Run OSPF between srlinux1 and ceos2.On srlinux1, redistribute routes between BGP and OSPF.Verify that ceos1 can ping the loopback of ceos2, and vice-versa.Day 7: Troubleshooting and Best PracticesOn the final day, you'll learn the crucial skill of troubleshooting. We'll look at common issues, the tools available to debug them, and best practices to help you build reliable and maintainable labs. A deliberate error in the example lab provides a hands-on troubleshooting exercise.Common IssuesImage Not Found: Double-check that the image exists in your local Docker daemon.YAML Syntax Errors: Indentation must be perfect. Use a linter.Container Not Starting: Use docker logs <container_name> to check for startup errors.Connectivity Issues: Check IPs/masks, ensure interfaces are enabled (no shutdown), and verify routing protocol neighbors.Best PracticesVersion Control: Always keep your lab files in Git.Clear Naming: Use descriptive names for labs, nodes, and interfaces.Modularity: Break large, complex labs into smaller, more manageable files.Destroy Labs: Destroy labs with clab destroy when you're finished to free up system resources.Deliberate Error and TroubleshootingThis lab has an intentional error: the subnet masks on the link between the two nodes do not match. This is a common mistake that prevents connectivity. Your task is to find and fix it.Topology File: broken-lab.yamlname: broken-lab
topology:
  nodes:
    ceos1:
      kind: ceos
      image: arista/ceos:latest
      startup-config: |
        hostname ceos1
        interface Ethernet1
          no switchport
          ip address 10.0.0.1/24
    srlinux1:
      kind: srl
      image: ghcr.io/nokia/srlinux
      startup-config: |
        # INTENTIONAL ERROR: Mismatched subnet mask
        set / interface ethernet-1/1 subinterface 0 ipv4 address 10.0.0.2/25
        commit full
  links:
    - endpoints: ["ceos1:eth1", "srlinux1:e1-1"]
````

Troubleshooting Steps
* Deploy the lab. Pings will fail.
* On ceos1, run show ip interface brief.  Note the /24 mask.
* On srlinux1, run info from state interface ethernet-1/1.0. Note the /25 mask.
* Diagnosis: The mismatched masks mean the nodes believe they are on different subnets.
* Fix: Destroy the lab, correct the mask in the YAML to /24 on srlinux1, and re-deploy.

## Day 7 ChallengeYour Task:This is your final exam! Build a new, multi-vendor topology of your own design. It must include:At least two cEOS nodes and two SR Linux nodes.At least two different routing protocols (e.g., OSPF and BGP).Route redistribution between the protocols.Introduce an intentional but subtle error.Write down the steps you would take to troubleshoot and fix your own broken lab.