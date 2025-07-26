# Challenges
Containerlab is a powerful, open-source CLI tool for orchestrating and managing container-based networking labs. It allows network engineers, developers, and students to spin up virtual network topologies composed of various Network Operating Systems (NOS) like Arista cEOS and Nokia SR Linux, using a declarative YAML file. This "lab-as-code" approach simplifies the creation, management, and versioning of complex network environments, making it ideal for testing, learning, and CI/CD integration.

## **Why Containerlab?**

  * **Lightweight and Fast:** Leverages Docker (or Podman) containers, which are more resource-efficient and quicker to spin up than traditional VMs.
  * **Declarative Topologies:** Define your entire lab in a simple YAML file, making it easy to understand, reproduce, and share.
  * **Multi-Vendor Support:** Supports a wide range of containerized NOS, including Arista cEOS, Nokia SR Linux, Cisco XRd, Juniper cJunosEvolved, VyOS, and more.
  * **Automation Friendly:** Integrates seamlessly with automation tools like Ansible, Python, and others for configuration and testing.
  * **Open Source & Community Driven:** Benefits from an active community that contributes to its development and provides support.

## Challenges with Containerlab

While Containerlab offers significant advantages, it also presents some challenges:

1.  **Image Acquisition:** Unlike some consumer-grade simulators, you often need to acquire official container images for proprietary NOS like Arista cEOS and Nokia SR Linux from vendor support portals. This usually requires a valid account or license.
2.  **Resource Consumption:** While lighter than VMs, complex topologies with many nodes can still consume significant CPU and RAM, especially if not properly managed.
3.  **Troubleshooting:** Debugging issues within the containerized network environment can sometimes be more complex than with traditional VMs, requiring familiarity with Docker commands (e.g., `docker logs`, `docker exec`).
4.  **Initial Setup:** Setting up the Linux host with Docker and Containerlab can have its nuances, though the provided installation scripts simplify this considerably.
5.  **Configuration Persistence:** By default, containers are ephemeral. You need to explicitly manage startup configurations to persist changes across lab deployments, typically by mounting configuration files into the containers.
6.  **Interface Naming Conventions:** Different NOS might have varying interface naming conventions within their containers, requiring attention when defining links in the Containerlab topology.
7.  **Limited GUI:** Containerlab is primarily a CLI tool. While it can integrate with external visualization tools, it doesn't offer a built-in graphical interface for topology design or interaction.

## **Prerequisites:**

* A Linux host (Ubuntu 20.04/22.04 or similar distribution recommended) or WSL2 on Windows.

* `sudo` privileges.

* Docker (Containerlab's setup script can install it).

* Arista cEOS container images. You will need to download these from their respective vendor portals.

* **Arista cEOS:** Typically available as a `.tar.xz` file from Arista's support portal. After downloading, import it into Docker:
  ```bash
  docker import cEOS-lab-<version>.tar.xz arista/ceos:<version>
  ```
  (Replace `<version>` with your downloaded image version, e.g., `4.34.1F`)
  
## **Simple Example Use Case: Verifying BGP Peering and Route Exchange**

Throughout this tutorial, we've implicitly used a simple use case:

  * Deploying a network topology (e.g., two routers or a spine-leaf fabric).
  * Configuring IP addresses on interfaces.
  * Configuring BGP peering between the devices.
  * Verifying the BGP session state.
  * Pinging between devices to confirm reachability.

This entire workflow, from lab deployment to configuration and verification, can be automated with Containerlab and tools like Ansible, making it perfect for:

  * **Learning:** Spin up complex topologies to understand routing protocols without expensive hardware.
  * **Testing:** Validate configuration changes or new features in a safe, isolated environment.
  * **Proof-of-Concept:** Quickly demonstrate network designs or automation scripts.
  * **CI/CD:** Integrate network tests into your software delivery pipeline to ensure network configurations are always correct.


## Overview
| Day | Description |
| ----- | ----- |
| Day 01 | [Getting Started and Basic Topology](/Topics/Containerization/Containerlab/Challenges/Day-01.md) |
| Day 02 | [Introducing Nokia SR Linux and Mixed Topology](/Topics/Containerization/Containerlab/Challenges/Day-02.md) |
| Day 03 | [Custom Startup Configurations](/Topics/Containerization/Containerlab/Challenges/Day-03.md) |
| Day 04 | [Managing Lab Lifecycle and Network Modes](/Topics/Containerization/Containerlab/Challenges/Day-04.md) |
| Day 05 | [Lab as Code - Advanced Topologies and Customizations](/Topics/Containerization/Containerlab/Challenges/Day-05.md) |
| Day 06 | [Automation with Containerlab and External Tools](/Topics/Containerization/Containerlab/Challenges/Day-06.md) |
| Day 07 | [Advanced Use Cases and Community Resources](/Topics/Containerization/Containerlab/Challenges/Day-07.md) |
