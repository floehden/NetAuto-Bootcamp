# Learning Programming Golang

**Target Audience:** Network engineers, DevOps engineers, and anyone interested in modern network automation using Go.

**Learning Objectives:**

By the end of this tutorial, you will be able to:

  * Understand Go fundamentals relevant to network automation.
  * Set up and manage Nokia SR Linux labs using Containerlab.
  * Interact with Nokia SR Linux via SSH/CLI using Go.
  * Automate configuration and state retrieval using NETCONF with Go.
  * Leverage gNMI for configuration, operational state, and streaming telemetry with Go.
  * Work with JSON and YAML data formats in Go for network configurations.
  * Build simple Go applications for network automation tasks.

## **Prerequisites:**

  * **Basic Linux Command Line Knowledge:** Familiarity with `cd`, `ls`, `mkdir`, `docker`, `ssh`.
  * **Fundamental Networking Concepts:** Understanding of IP addressing, routing, interfaces, VLANs.
  * **Basic programming concepts** variables, loops, functions
  * **Docker Installed:** Containerlab relies on Docker. Ensure it's installed and running on your Linux machine (or WSL2 on Windows, or OrbStack/Docker Desktop on macOS with a Linux VM).
  * **Containerlab Installed:** Follow the official Containerlab installation guide: [https://containerlab.dev/install/](https://containerlab.dev/install/)
  * **Arista cEOS-lab Image:** You'll need to download the cEOS-lab image from the Arista support portal (requires an Arista account, free lab versions are available). Import it into Docker: `docker import cEOS-lab-<version>.tar.xz arista/ceos:<version>` (e.g., `arista/ceos:4.30.6M`).
  * **Go Installed:** Download and install Go from [https://golang.org/doc/install](https://golang.org/doc/install).


## **Module 1: GoLang Fundamentals **
| Day | Arista | Nokia |
| -------- | ------- | ------- |
| 1 | [Getting Started with GoLang](/Topics/Programming/Go/Challenges/Arista/Day-01.md) | [GoLang Basics & Setup](/Topics/Programming/Go/Challenges/Nokia/Day-01.md) |
| 2 | [Variables, Data Types, and Operators](/Topics/Programming/Go/Challenges/Arista/Day-02.md) |  [Variables, Data Types, and Control Flow](/Topics/Programming/Go/Challenges/Nokia/Day-02.md) |
| 3 | [Control Flow (If/Else, For Loops, Switch)](/Topics/Programming/Go/Challenges/Arista/Day-03.md) |  [Functions, Packages, and Error Handling](/Topics/Programming/Go/Challenges/Nokia/Day-03.md) |
| 4 | [Functions and Error Handling](/Topics/Programming/Go/Challenges/Arista/Day-04.md) |  [Structs, Slices, and Maps](/Topics/Programming/Go/Challenges/Nokia/Day-04.md) |
| 5 | [Slices, Maps, and Structs](/Topics/Programming/Go/Challenges/Arista/Day-05.md) |  [JSON and YAML Serialization/Deserialization](/Topics/Programming/Go/Challenges/Nokia/Day-05.md) |

## **Module 2: Containerlab Setup & Basic Router Interaction (Days 6-10)**
| Description | Arista | Nokia |
| -------- | ------- | ------- |
| 6 | [Introduction to Containerlab](/Topics/Programming/Go/Challenges/Arista/Day-06.md) |  [Introduction to Containerlab](/Topics/Programming/Go/Challenges/Nokia/Day-06.md) |
| 7 | [Initial cEOS Configuration with Containerlab](/Topics/Programming/Go/Challenges/Arista/Day-07.md) |  [SSH/CLI Automation with Go](/Topics/Programming/Go/Challenges/Nokia/Day-07.md) |
| 8 | [GoLang Basic File I/O for Config Files](/Topics/Programming/Go/Challenges/Arista/Day-08.md) |  [Building a More Robust SSH Client](/Topics/Programming/Go/Challenges/Nokia/Day-08.md) |
| 9 | [Structuring Go Projects and Modules](/Topics/Programming/Go/Challenges/Arista/Day-09.md) |  [Containerlab Multi-Node Topology](/Topics/Programming/Go/Challenges/Nokia/Day-09.md) |
| 10 | [Basic Interaction with cEOS (Manual SSH/eAPI prep)](/Topics/Programming/Go/Challenges/Arista/Day-10.md) |  [ Inventory Management in Go for Containerlab](/Topics/Programming/Go/Challenges/Nokia/Day-10.md) |

## **Module 3: GoLang and Arista eAPI / Model-Driven Automation with Go (Days 11-15)**

| Description | Arista | Nokia |
| -------- | ------- | ------- |
| 11 | [Introduction to Arista eAPI and GoeAPI Library](/Topics/Programming/Go/Challenges/Arista/Day-11.md) |  [Introduction to NETCONF and YANG](/Topics/Programming/Go/Challenges/Nokia/Day-11.md) |
| 12 | [Parsing eAPI Responses with Go Structs](/Topics/Programming/Go/Challenges/Arista/Day-12.md) |  [NETCONF with Go - Basic Get Operations](/Topics/Programming/Go/Challenges/Nokia/Day-12.md) |
| 13 | [Sending Configuration Commands via eAPI](/Topics/Programming/Go/Challenges/Arista/Day-13.md) |  [NETCONF with Go - Edit Operations](/Topics/Programming/Go/Challenges/Nokia/Day-13.md) |
| 14 | [Advanced eAPI Operations: Multiple Commands & Error Handling ](/Topics/Programming/Go/Challenges/Arista/Day-14.md) |  [Introduction to gNMI and Protocol Buffers](/Topics/Programming/Go/Challenges/Nokia/Day-14.md) |
| 15 | [Templating Configurations with Go's `text/template`](/Topics/Programming/Go/Challenges/Arista/Day-15.md) |  [gNMI with Go - Capabilities and Get Operations](/Topics/Programming/Go/Challenges/Nokia/Day-15.md) |

## **Module 4: Advanced GoLang for Network Automation (Days 16-20)**

| Description | Arista | Nokia |
| -------- | ------- | ------- |
| 16 | [Concurrency with Goroutines and Channels](/Topics/Programming/Go/Challenges/Arista/Day-16.md) |  [gNMI with Go - Set Operations](/Topics/Programming/Go/Challenges/Nokia/Day-16.md) |
| 17 | [Building a Simple CLI Tool with `cobra` or `flag`](/Topics/Programming/Go/Challenges/Arista/Day-17.md) |  [gNMI with Go - Streaming Telemetry](/Topics/Programming/Go/Challenges/Nokia/Day-17.md) |
| 18 | [Working with JSON Configuration Files (External Data)](/Topics/Programming/Go/Challenges/Arista/Day-18.md) |  [Building a Generic Network Client in Go](/Topics/Programming/Go/Challenges/Nokia/Day-18.md) |
| 19 | [Building a Network Discovery and Reporting Tool](/Topics/Programming/Go/Challenges/Arista/Day-19.md) |  [Concurrency with GoRoutines for Automation](/Topics/Programming/Go/Challenges/Nokia/Day-19.md) |
| 20 | [Putting It All Together: Automated Lab Deployment & Configuration](/Topics/Programming/Go/Challenges/Arista/Day-20.md) |  [Building a Simple Automation Tool](/Topics/Programming/Go/Challenges/Nokia/Day-20.md) |

## **Module 5: GoLang and gNMI for Streaming Telemetry & Configuration (Days 21-26)**
| Description | Arista | 
| -------- | ------- | 
| 21 | [Introduction to gNMI and Basic Connection](/Topics/Programming/Go/Challenges/Arista/Day-21.md) |  
| 22 | [gNMI `Get` RPC for Operational State](/Topics/Programming/Go/Challenges/Arista/Day-22.md) | 
| 23 | [gNMI `Set` RPC for Configuration](/Topics/Programming/Go/Challenges/Arista/Day-23.md) | 
| 24 | [gNMI `Subscribe` for Streaming Telemetry](/Topics/Programming/Go/Challenges/Arista/Day-24.md) | 
| 25 | [Advanced gNMI Use Cases & Best Practices](/Topics/Programming/Go/Challenges/Arista/Day-25.md) | 
| 26 | [Configuration Backup and Restore with gNMI](/Topics/Programming/Go/Challenges/Arista/Day-26.md) | 

**Prerequisites for Module 5:**

* **Arista cEOS with gNMI enabled:** In your Containerlab YAML, you need to enable the gNMI server. Add the following to your cEOS node's `startup-config`:

```yaml
# ... inside ceos node startup-config ...
gnmi
    no shutdown
    transport grpc
    no shutdown
    port 6030
    vrf management
    !
```

*Note: gNMI typically runs on port 50051, but Arista EOS uses 6030 by default for gRPC. Ensure your firewall/security groups allow access to this port if running on a cloud VM.*

* **Go gNMI Client Libraries:** You'll need `go get` these:

    * `go get github.com/openconfig/gnmi/proto/gnmi`
    * `go get google.golang.org/grpc`
    * `go get google.golang.org/grpc/credentials/insecure` (for unencrypted connections)
    * `go get google.golang.org/protobuf/proto`
    * `go get github.com/openconfig/ygot/ygot` (useful for path building)
    * `go get github.com/aristanetworks/glog` (used by some Arista examples, though standard log is fine)


