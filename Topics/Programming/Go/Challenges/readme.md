# Learning Programming Golang

**Target Audience:** Network engineers, DevOps engineers, and anyone interested in modern network automation using Go.

**Learning Objectives:**

By the end of this tutorial, you will be able to:

  * Understand Go fundamentals relevant to network automation.
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
  * **Arista cEOS-lab Image:** You'll need to download the cEOS-lab image from the [Arista support portal](https://www.arista.com/en/support/software-download) (requires an Arista account, free lab versions are available). Import it into Docker: `docker import cEOS-lab-<version>.tar.xz ceos:<version>` (e.g., `ceos:4.30.6M`).
  * **Go Installed:** Download and install Go from [https://golang.org/doc/install](https://golang.org/doc/install).


## **Module 1: GoLang Fundamentals**
| Day | Description | 
| -------- | ------- |
| 1 | [Getting Started with GoLang](/Topics/Programming/Go/Challenges/Arista/Day-01.md) | 
| 2 | [Variables, Data Types, and Operators](/Topics/Programming/Go/Challenges/Arista/Day-02.md) |
| 3 | [Control Flow (If/Else, For Loops, Switch)](/Topics/Programming/Go/Challenges/Arista/Day-03.md) | 
| 4 | [Functions and Error Handling](/Topics/Programming/Go/Challenges/Arista/Day-04.md) | 
| 5 | [Slices, Maps, and Structs](/Topics/Programming/Go/Challenges/Arista/Day-05.md) | 

## **Module 2: Containerlab Setup & Basic Router Interaction (Days 6-10)**
| Day | Description | State |
| -------- | ------- | ------- |
| 6 | [Introduction to Containerlab](/Topics/Programming/Go/Challenges/Arista/Day-06.md) | works |
| 7 | [Initial cEOS Configuration with Containerlab](/Topics/Programming/Go/Challenges/Arista/Day-07.md) | works |
| 8 | [GoLang Basic File I/O for Config Files](/Topics/Programming/Go/Challenges/Arista/Day-08.md) | works |
| 9 | [Structuring Go Projects and Modules](/Topics/Programming/Go/Challenges/Arista/Day-09.md) | works |
| 10 | [Basic Interaction with cEOS (Manual SSH/eAPI prep)](/Topics/Programming/Go/Challenges/Arista/Day-10.md)  | need further testing |

## **Module 3: GoLang and Arista eAPI (Days 11-15)**

| Day | Description | 
| -------- | ------- | 
| 11 | [Introduction to Arista eAPI and GoeAPI Library](/Topics/Programming/Go/Challenges/Arista/Day-11.md) | 
| 12 | [Parsing eAPI Responses with Go Structs](/Topics/Programming/Go/Challenges/Arista/Day-12.md) | 
| 13 | [Sending Configuration Commands via eAPI](/Topics/Programming/Go/Challenges/Arista/Day-13.md) |
| 14 | [Advanced eAPI Operations: Multiple Commands & Error Handling ](/Topics/Programming/Go/Challenges/Arista/Day-14.md) | 
| 15 | [Templating Configurations with Go's `text/template`](/Topics/Programming/Go/Challenges/Arista/Day-15.md) |

## **Module 4: Advanced GoLang for Network Automation (Days 16-20)(Untested)**

| Day | Description | 
| -------- | ------- | 
| 16 | [Concurrency with Goroutines and Channels](/Topics/Programming/Go/Challenges/Arista/Day-16.md) | 
| 17 | [Building a Simple CLI Tool with `flag`](/Topics/Programming/Go/Challenges/Arista/Day-17.md) |
| 18 | [Working with JSON Configuration Files (External Data)](/Topics/Programming/Go/Challenges/Arista/Day-18.md) | 
| 19 | [Building a Network Discovery and Reporting Tool](/Topics/Programming/Go/Challenges/Arista/Day-19.md) | 
| 20 | [Putting It All Together: Automated Lab Deployment & Configuration](/Topics/Programming/Go/Challenges/Arista/Day-20.md) | 

## **Module 5 (Arista only): GoLang and gNMI for Streaming Telemetry & Configuration (Days 21-26)**
| Day | Description | State |
| -------- | ------- | ------- | 
| 21 | [Introduction to gNMI and Basic Connection](/Topics/Programming/Go/Challenges/Arista/Day-21.md) |   not tested |
| 22 | [gNMI `Get` RPC for Operational State](/Topics/Programming/Go/Challenges/Arista/Day-22.md) |  not tested |
| 23 | [gNMI `Set` RPC for Configuration](/Topics/Programming/Go/Challenges/Arista/Day-23.md) |  not tested |
| 24 | [gNMI `Subscribe` for Streaming Telemetry](/Topics/Programming/Go/Challenges/Arista/Day-24.md) |  not tested |
| 25 | [Advanced gNMI Use Cases & Best Practices](/Topics/Programming/Go/Challenges/Arista/Day-25.md) |  not tested |

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


## Create a CLI
If you are interested in creating a CLI with Cobra go to [Creating a CLI](/Topics/Programming/Go/Challenges/CLI/readme.md)

## Create a Terraform provider (coming soon!)
If you are interested in creating a Terraform Provider go to [Creating a Terraform-Provider](/Topics/Programming/Go/Challenges/Terraform/readme.md)</br>
For further informations go to https://developer.hashicorp.com/terraform/tutorials/providers-plugin-framework

## Create a Kubernetes Operator (coming soon!)
If you are interested in creating a Kubernetes Operator go to [Creating a Kubernetes Operator](/Topics/Programming/Go/Challenges/Kubernetes-Operator/readme.md) </br>
For further informations go to https://book.kubebuilder.io/
