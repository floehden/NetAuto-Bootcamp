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
  * **Docker Installed:** Containerlab relies on Docker. Ensure it's installed and running on your Linux machine (or WSL2 on Windows, or OrbStack on macOS with a Linux VM).
  * **Containerlab Installed:** Follow the official Containerlab installation guide: [https://containerlab.dev/install/](https://containerlab.dev/install/)
  * **Arista cEOS-lab Image:** You'll need to download the cEOS-lab image from the [Arista support portal](https://www.arista.com/en/support/software-download) (requires an Arista account, free lab versions are available). Import it into Docker: `docker import cEOS-lab-<version>.tar.xz ceos:<version>` (e.g., `ceos:4.30.6M`).
  * **Go Installed:** Download and install Go from [https://golang.org/doc/install](https://golang.org/doc/install).


## **Module 1: GoLang Fundamentals**
| Day | Description | 
| -------- | ------- |
| 1 | [Getting Started with GoLang](/Topics/Programming/Go/Challenges/Day-01.md) | 
| 2 | [Variables, Data Types, and Operators](/Topics/Programming/Go/Challenges/Day-02.md) |
| 3 | [Control Flow (If/Else, For Loops, Switch)](/Topics/Programming/Go/Challenges/Day-03.md) | 
| 4 | [Functions, Packages and Error Handling](/Topics/Programming/Go/Challenges/Day-04.md) | 
| 5 | [Slices, Maps, and Structs](/Topics/Programming/Go/Challenges/Day-05.md) | 

## **Module 2: Containerlab Setup & Basic Router Interaction (Days 6-10)**
| Day | Description | 
| -------- | ------- | 
| 6 | [Introduction to Containerlab](/Topics/Programming/Go/Challenges/Day-06.md) | 
| 7 | [Initial cEOS Configuration with Containerlab](/Topics/Programming/Go/Challenges/Day-07.md) | 
| 8 | [GoLang Basic File I/O for Config Files](/Topics/Programming/Go/Challenges/Day-08.md) | 
| 9 | [Structuring Go Projects and Modules](/Topics/Programming/Go/Challenges/Day-09.md) | 
| 10 | [Basic Interaction with cEOS (Manual SSH/eAPI prep)](/Topics/Programming/Go/Challenges/Day-10.md)  |

### Extras (SSH with Nokia SRL) 
| Extra | Description | 
| -------- | ------- | 
| M2.E 1 | [SSH/CLI Automation with Go](/Topics/Programming/Go/Challenges/M2-E-01.md) | 
| M2.E 2 | [Containerlab Multi-Node Topology (SRL)](/Topics/Programming/Go/Challenges/M2-E-02.md) | 
| M2.E 3 | [Inventory Management in Go for Containerlab](/Topics/Programming/Go/Challenges/M2-E-03.md) | 

## **Module 3: GoLang and Arista eAPI (Days 11-15)**

| Day | Description | 
| -------- | ------- | 
| 11 | [Introduction to Arista eAPI and GoeAPI Library](/Topics/Programming/Go/Challenges/Day-11.md) | 
| 12 | [Parsing eAPI Responses with Go Structs](/Topics/Programming/Go/Challenges/Day-12.md) | 
| 13 | [Sending Configuration Commands via eAPI](/Topics/Programming/Go/Challenges/Day-13.md) |
| 14 | [Advanced eAPI Operations: Multiple Commands & Error Handling ](/Topics/Programming/Go/Challenges/Day-14.md) | 
| 15 | [Templating Configurations with Go's `text/template`](/Topics/Programming/Go/Challenges/Day-15.md) |

### Extras (with Nokia SRL and NETCONF) (TEST IT)
| Extra | Description | 
| -------- | ------- | 
| M3.E 01 | [NETCONF with Go - Basic Get Operations](/Topics/Programming/Go/Challenges/M3-E-01.md) | 
| M3.E 02 | [NETCONF with Go - Edit Operations](/Topics/Programming/Go/Challenges/M3-E-01.md) | 

## **Module 4: Advanced GoLang for Network Automation (Days 16-20)**

| Day | Description | 
| -------- | ------- | 
| 16 | [Concurrency with Goroutines and Channels](/Topics/Programming/Go/Challenges/Day-16.md) | 
| 17 | [Building a Simple CLI Tool with `flag`](/Topics/Programming/Go/Challenges/Day-17.md) |
| 18 | [Working with JSON and YAML Configuration Files (External Data](/Topics/Programming/Go/Challenges/Day-18.md) | 
| 19 | [Building a Network Discovery and Reporting Tool](/Topics/Programming/Go/Challenges/Day-19.md) | 
| 20 | [Putting It All Together: Automated Lab Deployment & Configuration](/Topics/Programming/Go/Challenges/Day-20.md) | 

## **Module 5: GoLang and gNMI for Streaming Telemetry & Configuration (Days 21-24)**
| Day | Description | 
| -------- | ------- | 
| 21 | [Introduction to gNMI and Basic Connection](/Topics/Programming/Go/Challenges/Day-21.md) | 
| 22 | [gNMI `Get` RPC for Operational State](/Topics/Programming/Go/Challenges/Day-22.md) |
| 23 | [gNMI `Set` RPC for Configuration](/Topics/Programming/Go/Challenges/Day-23.md) |
| 24 | [gNMI `Subscribe` for Streaming Telemetry](/Topics/Programming/Go/Challenges/Day-24.md) | 


For further informations on using Golang with gNMI look at the [documentation](https://gnmic.openconfig.net/user_guide/golang_package/intro/) and especially in the example of [CAPABILITIES](https://gnmic.openconfig.net/user_guide/golang_package/examples/capabilities/), [GET](https://gnmic.openconfig.net/user_guide/golang_package/examples/get/), [SET](https://gnmic.openconfig.net/user_guide/golang_package/examples/set/) and [SUBSCRIBE](https://gnmic.openconfig.net/user_guide/golang_package/examples/subscribe/).


## Project
| Project | Description | 
| -------- | ------- | 
| Final 01 | [Building a Generic Network Client in Go](/Topics/Programming/Go/Challenges/final-project-02.md) | 
| Final 02 | [Building a Simple Automation Tool & Next Steps](/Topics/Programming/Go/Challenges/final-project-02.md) | 


## Create a CLI
If you are interested in creating a CLI with Cobra go to [Creating a CLI](/Topics/Programming/Go/Challenges/CLI/readme.md)

## Create a Terraform provider (coming soon!)
If you are interested in creating a Terraform Provider go to [Creating a Terraform-Provider](/Topics/Programming/Go/Challenges/Terraform/readme.md)</br>
For further informations go to https://developer.hashicorp.com/terraform/tutorials/providers-plugin-framework

## Create a Kubernetes Operator (coming soon!)
If you are interested in creating a Kubernetes Operator go to [Creating a Kubernetes Operator](/Topics/Programming/Go/Challenges/Kubernetes-Operator/readme.md) </br>
For further informations go to https://book.kubebuilder.io/
