**Target Audience:** Network engineers, DevOps engineers, and anyone interested in modern network automation using Go.

**Prerequisites:**

  * Basic understanding of networking concepts (IP addressing, routing, interfaces).
  * Familiarity with the Linux command line.
  * Basic programming concepts (variables, loops, functions).
  * Docker installed on your system.
  * Go installed on your system (latest stable version recommended).
  * Containerlab installed on your system.

**Learning Objectives:**

By the end of this tutorial, you will be able to:

  * Understand Go fundamentals relevant to network automation.
  * Set up and manage Nokia SR Linux labs using Containerlab.
  * Interact with Nokia SR Linux via SSH/CLI using Go.
  * Automate configuration and state retrieval using NETCONF with Go.
  * Leverage gNMI for configuration, operational state, and streaming telemetry with Go.
  * Work with JSON and YAML data formats in Go for network configurations.
  * Build simple Go applications for network automation tasks.

-----

## **20-Day GoLang for Nokia SR Linux Automation Tutorial with Containerlab**

### **Module 1: GoLang Fundamentals for Network Automation (Days 1-5)**

**Day 1: GoLang Basics & Setup**

  * **Concept:** Introduction to Go, its philosophy, and why it's suitable for network automation (concurrency, performance, strong typing). Setting up your Go environment.
  * **Challenge:** Install Go, set up your GOPATH, and write your first "Hello, Network\!" program.
  * **Code Example:**
    ```go
    // main.go
    package main

    import "fmt"

    func main() {
        fmt.Println("Hello, Network Automation with Go!")
    }
    ```
    **Instructions:**
    1.  Create a directory `day1`.
    2.  Save the code as `main.go` inside `day1`.
    3.  Open your terminal in the `day1` directory.
    4.  Run `go run main.go`.

**Day 2: Variables, Data Types, and Control Flow**

  * **Concept:** Understanding Go's basic data types (strings, integers, booleans), declaring variables, and fundamental control flow statements (if/else, for loops).
  * **Challenge:** Write a Go program that takes an IP address as a string, checks if it's a valid IPv4 address (simple check, e.g., using `strings.Contains` for dots), and prints whether it's valid or not.
  * **Code Example:**
    ```go
    // main.go
    package main

    import (
        "fmt"
        "strings"
    )

    func main() {
        ipAddress := "192.168.1.1"
        if isValidIPv4(ipAddress) {
            fmt.Printf("%s is a valid IPv4 address (basic check).\n", ipAddress)
        } else {
            fmt.Printf("%s is not a valid IPv4 address (basic check).\n", ipAddress)
        }

        ipAddress = "2001:db8::1"
        if isValidIPv4(ipAddress) {
            fmt.Printf("%s is a valid IPv4 address (basic check).\n", ipAddress)
        } else {
            fmt.Printf("%s is not a valid IPv4 address (basic check).\n", ipAddress)
        }
    }

    func isValidIPv4(ip string) bool {
        // A very basic check: does it contain three dots?
        // In a real-world scenario, you'd use net.ParseIP
        return strings.Count(ip, ".") == 3
    }
    ```

**Day 3: Functions, Packages, and Error Handling**

  * **Concept:** Defining functions, organizing code into packages, and Go's idiomatic error handling using `error` types and `if err != nil` checks.
  * **Challenge:** Create a `networkutils` package with a function that takes a network interface name (string) and an IP address (string), and returns a formatted string like "Interface \<name\> configured with \<IP\>" or an error if the IP is invalid.
  * **Code Example:**
    ```go
    // networkutils/networkutils.go
    package networkutils

    import (
        "fmt"
        "strings"
    )

    // ConfigureInterface simulates configuring an interface.
    // It returns a success message or an error if the IP is malformed.
    func ConfigureInterface(name, ipAddress string) (string, error) {
        if strings.Count(ipAddress, ".") != 3 {
            return "", fmt.Errorf("invalid IPv4 address format for %s: %s", name, ipAddress)
        }
        return fmt.Sprintf("Interface %s configured with %s", name, ipAddress), nil
    }

    // main.go
    package main

    import (
        "fmt"
        "log"
        "day3/networkutils" // Assuming your module name is day3
    )

    func main() {
        msg, err := networkutils.ConfigureInterface("ethernet-1/1", "10.0.0.1/24")
        if err != nil {
            log.Printf("Error configuring interface: %v", err)
        } else {
            fmt.Println(msg)
        }

        msg, err = networkutils.ConfigureInterface("loopback0", "invalid-ip")
        if err != nil {
            log.Printf("Error configuring interface: %v", err)
        } else {
            fmt.Println(msg)
        }
    }
    ```
    **Instructions:**
    1.  Create a directory `day3`.
    2.  Inside `day3`, create a directory `networkutils`.
    3.  Save the first code block as `networkutils/networkutils.go`.
    4.  Save the second code block as `main.go` in the `day3` directory.
    5.  Run `go mod init day3` in the `day3` directory.
    6.  Run `go run main.go`.

**Day 4: Structs, Slices, and Maps**

  * **Concept:** Defining custom data structures (`structs`), working with dynamic arrays (`slices`), and key-value pairs (`maps`). These are crucial for handling network device data.
  * **Challenge:** Define a `Router` struct with fields like `Name`, `IPAddress`, and a `map[string]string` for `Interfaces` (interface name to IP). Create a slice of `Router` structs and print their details.
  * **Code Example:**
    ```go
    // main.go
    package main

    import "fmt"

    type Router struct {
        Name        string
        IPAddress   string
        Interfaces  map[string]string
    }

    func main() {
        routers := []Router{
            {
                Name:      "srl1",
                IPAddress: "192.168.0.1",
                Interfaces: map[string]string{
                    "ethernet-1/1": "10.0.0.1/24",
                    "ethernet-1/2": "10.0.0.5/24",
                },
            },
            {
                Name:      "srl2",
                IPAddress: "192.168.0.2",
                Interfaces: map[string]string{
                    "ethernet-1/1": "10.0.0.2/24",
                },
            },
        }

        for _, router := range routers {
            fmt.Printf("Router Name: %s, Management IP: %s\n", router.Name, router.IPAddress)
            fmt.Println("  Interfaces:")
            for iface, ip := range router.Interfaces {
                fmt.Printf("    - %s: %s\n", iface, ip)
            }
        }
    }
    ```

**Day 5: JSON and YAML Serialization/Deserialization**

  * **Concept:** Reading and writing structured data in JSON and YAML formats, common in network configurations and APIs. Using Go's `encoding/json` and external YAML libraries (`gopkg.in/yaml.v2`).
  * **Challenge:** Take the `routers` slice from Day 4, serialize it into a JSON string, and then deserialize a YAML string representing a similar router configuration back into a `Router` struct.
  * **Code Example:**
    ```go
    // main.go
    package main

    import (
        "encoding/json"
        "fmt"
        "log"

        "gopkg.in/yaml.v2" // You'll need to run 'go get gopkg.in/yaml.v2'
    )

    type Router struct {
        Name        string            `json:"name" yaml:"name"`
        IPAddress   string            `json:"ip_address" yaml:"ip_address"`
        Interfaces  map[string]string `json:"interfaces" yaml:"interfaces"`
    }

    func main() {
        // Data to serialize to JSON
        routers := []Router{
            {
                Name:      "srl1",
                IPAddress: "192.168.0.1",
                Interfaces: map[string]string{
                    "ethernet-1/1": "10.0.0.1/24",
                    "ethernet-1/2": "10.0.0.5/24",
                },
            },
        }

        // Marshal to JSON
        jsonData, err := json.MarshalIndent(routers, "", "  ")
        if err != nil {
            log.Fatalf("Error marshalling JSON: %v", err)
        }
        fmt.Println("--- JSON Output ---")
        fmt.Println(string(jsonData))

        // YAML data to deserialize
        yamlData := `
    ```

<!-- end list -->

  - name: srl3
    ip\_address: 192.168.0.3
    interfaces:
    ethernet-1/1: 10.0.0.10/24
    loopback0: 10.3.3.3/32
    \`
    var newRouters []Router
    err = yaml.Unmarshal([]byte(yamlData), \&newRouters)
    if err \!= nil {
    log.Fatalf("Error unmarshalling YAML: %v", err)
    }
    fmt.Println("\\n--- YAML Unmarshalled Output ---")
    for \_, r := range newRouters {
    fmt.Printf("Router: %+v\\n", r)
    }
    }
    ```
    **Instructions:**
    1.  Run `go mod init day5` (or similar)
    2.  Run `go get gopkg.in/yaml.v2`
    3.  Save the code as `main.go`.
    4.  Run `go run main.go`.

    ```

-----

### **Module 2: Containerlab and Basic SR Linux Interaction (Days 6-10)**

**Day 6: Introduction to Containerlab**

  * **Concept:** Understanding Containerlab's role in creating virtual network labs with containers, its YAML topology definition, and basic commands.
  * **Challenge:** Create a simple Containerlab topology with one Nokia SR Linux node. Deploy it, access its CLI, and check its basic status.
  * **Code Example (Containerlab Topology):**
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
    **Instructions:**
    1.  Save the YAML as `srl_single.clab.yaml`.
    2.  Deploy the lab: `sudo containerlab deploy -t srl_single.clab.yaml`.
    3.  Access the CLI: `docker exec -it clab-srl-single-srl1 sr_cli`.
    4.  Check status: `show system information`.
    5.  Destroy the lab when done: `sudo containerlab destroy -t srl_single.clab.yaml`.

**Day 7: SSH/CLI Automation with Go**

  * **Concept:** Using Go to establish SSH connections to network devices and execute CLI commands. Introduction to SSH libraries like `golang.org/x/crypto/ssh`.
  * **Challenge:** Write a Go program that connects to the SR Linux node (from Day 6's Containerlab setup) via SSH, executes `show system information`, and prints the output.
  * **Code Example:**
    ```go
    // main.go
    package main

    import (
        "fmt"
        "io/ioutil"
        "log"
        "time"

        "golang.org/x/crypto/ssh"
    )

    func main() {
        // SSH client config
        config := &ssh.ClientConfig{
            User: "admin",
            Auth: []ssh.AuthMethod{
                ssh.Password("NokiaSrl1!"), // Default password for SR Linux
            },
            HostKeyCallback: ssh.InsecureIgnoreHostKey(), // NOT for production!
            Timeout:         5 * time.Second,
        }

        // Connect to the SSH server (replace with your SR Linux management IP/hostname)
        // For Containerlab, typically the node name resolves to its IP in the docker network
        conn, err := ssh.Dial("tcp", "clab-srl-single-srl1:22", config)
        if err != nil {
            log.Fatalf("Failed to dial: %v", err)
        }
        defer conn.Close()

        // Create a new session
        session, err := conn.NewSession()
        if err != nil {
            log.Fatalf("Failed to create session: %v", err)
        }
        defer session.Close()

        // Run the command
        output, err := session.CombinedOutput("sr_cli show system information")
        if err != nil {
            log.Fatalf("Failed to run command: %v", err)
        }

        fmt.Println(string(output))
    }
    ```
    **Instructions:**
    1.  Ensure your `srl_single.clab.yaml` from Day 6 is deployed.
    2.  Run `go mod init day7`.
    3.  Run `go get golang.org/x/crypto/ssh`.
    4.  Save the code as `main.go`.
    5.  Run `go run main.go`.

**Day 8: Building a More Robust SSH Client**

  * **Concept:** Enhancing the SSH client with functions for executing multiple commands, handling command prompts, and structured output parsing.
  * **Challenge:** Modify the Day 7 code to accept a slice of commands and execute them sequentially, printing the output for each. Consider how you might "expect" certain prompts if doing interactive CLI.
  * **Code Example (Conceptual - highly simplified interactive SSH):**
    ```go
    // main.go
    package main

    import (
        "bytes"
        "fmt"
        "io"
        "log"
        "time"

        "golang.org/x/crypto/ssh"
    )

    // SSHClient represents an SSH client connection.
    type SSHClient struct {
        client *ssh.Client
        session *ssh.Session
        stdin io.WriteCloser
        stdout io.Reader
        stderr io.Reader
    }

    // NewSSHClient establishes an SSH connection.
    func NewSSHClient(addr, user, pass string) (*SSHClient, error) {
        config := &ssh.ClientConfig{
            User: user,
            Auth: []ssh.AuthMethod{
                ssh.Password(pass),
            },
            HostKeyCallback: ssh.InsecureIgnoreHostKey(),
            Timeout:         10 * time.Second,
        }

        client, err := ssh.Dial("tcp", addr, config)
        if err != nil {
            return nil, fmt.Errorf("failed to dial: %w", err)
        }

        session, err := client.NewSession()
        if err != nil {
            client.Close()
            return nil, fmt.Errorf("failed to create session: %w", err)
        }

        // Set up stdin, stdout, and stderr for interactive use
        stdin, _ := session.StdinPipe()
        stdout, _ := session.StdoutPipe()
        stderr, _ := session.StderrPipe()

        if err := session.Shell(); err != nil {
            session.Close()
            client.Close()
            return nil, fmt.Errorf("failed to start shell: %w", err)
        }

        return &SSHClient{
            client: client,
            session: session,
            stdin: stdin,
            stdout: stdout,
            stderr: stderr,
        }, nil
    }

    // RunCommand sends a command and returns the output.
    func (c *SSHClient) RunCommand(cmd string) (string, error) {
        // Simple write and read, in a real scenario you'd need to handle prompts
        // and read until a known prompt is seen.
        _, err := fmt.Fprintf(c.stdin, "%s\n", cmd)
        if err != nil {
            return "", fmt.Errorf("failed to write command: %w", err)
        }

        // Give the device time to process and send output
        time.Sleep(500 * time.Millisecond)

        var buf bytes.Buffer
        _, err = io.CopyN(&buf, c.stdout, 1024) // Read up to 1KB, adjust as needed
        if err != nil && err != io.EOF {
             // EOF is expected if the command finishes before the buffer is full
            return "", fmt.Errorf("failed to read output: %w", err)
        }
        return buf.String(), nil
    }

    // Close closes the SSH session and client.
    func (c *SSHClient) Close() {
        if c.session != nil {
            c.session.Close()
        }
        if c.client != nil {
            c.client.Close()
        }
    }

    func main() {
        // Ensure srl1 is deployed via Containerlab
        client, err := NewSSHClient("clab-srl-single-srl1:22", "admin", "NokiaSrl1!")
        if err != nil {
            log.Fatalf("Error creating SSH client: %v", err)
        }
        defer client.Close()

        commands := []string{
            "sr_cli show system information",
            "sr_cli show interface brief",
        }

        for _, cmd := range commands {
            fmt.Printf("\n--- Executing command: %s ---\n", cmd)
            output, err := client.RunCommand(cmd)
            if err != nil {
                log.Printf("Error running command '%s': %v", cmd, err)
            } else {
                fmt.Println(output)
            }
        }
    }
    ```
    **Note:** This `RunCommand` is a simplified example. For robust CLI automation, you'd typically use a library that handles "expect" patterns to wait for prompts, like `go-expect`.

**Day 9: Containerlab Multi-Node Topology**

  * **Concept:** Building more complex network topologies with multiple SR Linux nodes and interconnections using Containerlab.
  * **Challenge:** Create a Containerlab topology with two SR Linux nodes connected by an `ethernet-1/1` interface on both ends. Configure basic IP addresses on these interfaces using the `startup-config` block in the YAML.
  * **Code Example (Containerlab Topology):**
    ```yaml
    # srl_2_nodes.clab.yaml
    name: srl-2-nodes
    topology:
      nodes:
        srl1:
          kind: srl
          image: ghcr.io/nokia/srlinux
          startup-config: |
            system {
              name {
                host-name srl1
              }
            }
            interface ethernet-1/1 {
              admin-state enable
              subinterface 0 {
                admin-state enable
                ipv4 {
                  address 10.0.0.1/24
                }
              }
            }
        srl2:
          kind: srl
          image: ghcr.io/nokia/srlinux
          startup-config: |
            system {
              name {
                host-name srl2
              }
            }
            interface ethernet-1/1 {
              admin-state enable
              subinterface 0 {
                admin-state enable
                ipv4 {
                  address 10.0.0.2/24
                }
              }
            }
      links:
        - endpoints: ["srl1:ethernet-1/1", "srl2:ethernet-1/1"]
    ```
    **Instructions:**
    1.  Save the YAML as `srl_2_nodes.clab.yaml`.
    2.  Deploy the lab: `sudo containerlab deploy -t srl_2_nodes.clab.yaml`.
    3.  Verify connectivity from `srl1`: `docker exec -it clab-srl-2-nodes-srl1 sr_cli "ping 10.0.0.2"`.
    4.  Destroy the lab when done: `sudo containerlab destroy -t srl_2_nodes.clab.yaml`.

**Day 10: Inventory Management in Go for Containerlab**

  * **Concept:** Programmatically obtaining information about deployed Containerlab nodes (IPs, names) to build an inventory in Go, which is essential for scaling automation.
  * **Challenge:** Write a Go program that parses the `srl_2_nodes.clab.yaml` file (or directly queries Containerlab if possible, but parsing YAML is more direct for this exercise) to extract the hostnames and expected management IPs of the SR Linux nodes.
  * **Code Example:**
    ```go
    // main.go
    package main

    import (
        "fmt"
        "io/ioutil"
        "log"

        "gopkg.in/yaml.v2"
    )

    // Simplified struct for parsing Containerlab topology
    type ContainerlabTopology struct {
        Name     string `yaml:"name"`
        Topology struct {
            Nodes map[string]struct {
                Kind string `yaml:"kind"`
                // In a real scenario, you might add 'mgmt_ipv4' or other fields
            } `yaml:"nodes"`
        } `yaml:"topology"`
    }

    func main() {
        yamlFile := "srl_2_nodes.clab.yaml" // Ensure this file exists from Day 9

        data, err := ioutil.ReadFile(yamlFile)
        if err != nil {
            log.Fatalf("Error reading YAML file: %v", err)
        }

        var topo ContainerlabTopology
        err = yaml.Unmarshal(data, &topo)
        if err != nil {
            log.Fatalf("Error unmarshalling YAML: %v", err)
        }

        fmt.Printf("Containerlab Lab Name: %s\n", topo.Name)
        fmt.Println("Discovered SR Linux Nodes:")
        for nodeName, nodeDetails := range topo.Topology.Nodes {
            if nodeDetails.Kind == "srl" {
                // Containerlab nodes typically resolve to <lab_name>-<node_name> in Docker DNS
                // For management, you'd usually connect to the container's management IP.
                // For simplicity, we'll use the Containerlab-assigned hostname in this context.
                fmt.Printf("  - Name: %s, Kind: %s, Management Hostname: clab-%s-%s\n",
                    nodeName, nodeDetails.Kind, topo.Name, nodeName)
            }
        }
    }
    ```
    **Instructions:**
    1.  Ensure `srl_2_nodes.clab.yaml` is present from Day 9.
    2.  Run `go mod init day10`.
    3.  Run `go get gopkg.in/yaml.v2`.
    4.  Save the code as `main.go`.
    5.  Run `go run main.go`.

-----

### **Module 3: Model-Driven Automation with Go (Days 11-15)**

**Day 11: Introduction to NETCONF and YANG**

  * **Concept:** Understanding NETCONF as a programmatic interface for network devices, its XML-based messaging, and the role of YANG data models in defining configurations and operational state.
  * **Challenge:** Explore some basic SR Linux YANG models (e.g., `srl_nokia-interfaces.yang`) by either downloading them from the SR Linux documentation or directly from a running SR Linux node (`/opt/srlinux/models/`). Identify the XML paths for configuring an interface.
  * **No Go code for this day, focus on conceptual understanding and YANG exploration.**

**Day 12: NETCONF with Go - Basic Get Operations**

  * **Concept:** Using a Go NETCONF library (e.g., `github.com/Juniper/go-netconf/netconf`) to connect to SR Linux and perform `<get-config>` operations.
  * **Challenge:** Connect to one of your SR Linux nodes via NETCONF and retrieve the entire running configuration. Parse the XML response.
  * **Code Example:**
    ```go
    // main.go
    package main

    import (
        "fmt"
        "log"
        "os"
        "time"

        "github.com/Juniper/go-netconf/netconf" // go get github.com/Juniper/go-netconf/netconf
    )

    func main() {
        // Ensure srl1 is deployed via Containerlab with NETCONF enabled (default on SR Linux)
        session, err := netconf.DialSSH("clab-srl-2-nodes-srl1:830",
            &netconf.SSHClientConfig{
                User:     "admin",
                Password: "NokiaSrl1!",
                Timeout:  10 * time.Second,
            })
        if err != nil {
            log.Fatalf("Failed to dial NETCONF: %v", err)
        }
        defer session.Close()

        reply, err := session.GetConfig("running")
        if err != nil {
            log.Fatalf("Failed to get config: %v", err)
        }

        fmt.Println("--- Running Configuration ---")
        fmt.Println(reply.Data) // reply.Data will contain the XML configuration
    }
    ```
    **Instructions:**
    1.  Ensure your `srl_2_nodes.clab.yaml` is deployed.
    2.  Run `go mod init day12`.
    3.  Run `go get github.com/Juniper/go-netconf/netconf`.
    4.  Save the code as `main.go`.
    5.  Run `go run main.go`.

**Day 13: NETCONF with Go - Edit Operations**

  * **Concept:** Using NETCONF to modify the configuration of SR Linux, focusing on `<edit-config>` and `<commit>` operations.
  * **Challenge:** Write a Go program to configure a loopback interface on `srl1` using NETCONF.
  * **Code Example:**
    ```go
    // main.go
    package main

    import (
        "fmt"
        "log"
        "time"

        "github.com/Juniper/go-netconf/netconf"
    )

    func main() {
        session, err := netconf.DialSSH("clab-srl-2-nodes-srl1:830",
            &netconf.SSHClientConfig{
                User:     "admin",
                Password: "NokiaSrl1!",
                Timeout:  10 * time.Second,
            })
        if err != nil {
            log.Fatalf("Failed to dial NETCONF: %v", err)
        }
        defer session.Close()

        // Configuration to apply (XML format)
        configXML := `
    ```

\<config\>
\<interface xmlns="urn:nokia.com:srlinux-interfaces"\>
\<name\>loopback0\</name\>
\<admin-state\>enable\</admin-state\>
\<subinterface\>
\<index\>0\</index\>
\<admin-state\>enable\</admin-state\>
\<ipv4\>
\<address\>
\<ip-prefix\>1.1.1.1/32\</ip-prefix\>
\</address\>
\</ipv4\>
\</subinterface\>
\</interface\>
\</config\>
\`

````
    // Edit the configuration
    _, err = session.EditConfig("candidate", configXML)
    if err != nil {
        log.Fatalf("Failed to edit config: %v", err)
    }
    fmt.Println("Configuration applied to candidate datastore.")

    // Commit the changes
    _, err = session.Commit()
    if err != nil {
        log.Fatalf("Failed to commit config: %v", err)
    }
    fmt.Println("Configuration committed successfully.")

    // Verify by getting the config again (optional)
    reply, err := session.GetConfig("running", "<filter><interface xmlns=\"urn:nokia.com:srlinux-interfaces\"><name>loopback0</name></interface></filter>")
    if err != nil {
        log.Fatalf("Failed to get config after commit: %v", err)
    }
    fmt.Println("\n--- New Running Configuration for loopback0 ---")
    fmt.Println(reply.Data)
}
```
**Instructions:**
1.  Ensure your `srl_2_nodes.clab.yaml` is deployed.
2.  Run `go mod tidy`.
3.  Save the code as `main.go`.
4.  Run `go run main.go`.
````

**Day 14: Introduction to gNMI and Protocol Buffers**

  * **Concept:** Understanding gNMI as a modern, high-performance API based on gRPC and Protocol Buffers for configuration, operational state, and streaming telemetry. Overview of `.proto` files and code generation.
  * **Challenge:** Locate the gNMI `.proto` files (often available in OpenConfig or vendor-specific repos). Understand how they define data structures and services. (No Go code for this day, focus on conceptual understanding).

**Day 15: gNMI with Go - Capabilities and Get Operations**

  * **Concept:** Using Go's gRPC library and generated gNMI client code to perform gNMI `<Capabilities>` and `<Get>` operations on SR Linux.
  * **Challenge:** Connect to `srl1` via gNMI, retrieve its capabilities, and then use a `<Get>` request to fetch the hostname or interface state.
  * **Code Example:**
    ```go
    // main.go
    package main

    import (
        "context"
        "crypto/tls"
        "fmt"
        "log"
        "time"

        "google.golang.org/grpc"
        "google.golang.org/grpc/credentials"

        gpb "github.com/openconfig/gnmi/proto/gnmi" // go get github.com/openconfig/gnmi
    )

    func main() {
        // Ensure srl1 is deployed via Containerlab
        // SR Linux gNMI default port is 57400
        target := "clab-srl-2-nodes-srl1:57400"
        username := "admin"
        password := "NokiaSrl1!"

        // InsecureSkipVerify is NOT for production environments.
        // For production, you'd use proper TLS certificates.
        creds := credentials.NewTLS(&tls.Config{
            InsecureSkipVerify: true,
        })

        opts := []grpc.DialOption{
            grpc.WithTransportCredentials(creds),
            grpc.WithPerRPCCredentials(&auth{
                username: username,
                password: password,
            }),
        }

        conn, err := grpc.Dial(target, opts...)
        if err != nil {
            log.Fatalf("Failed to dial gNMI: %v", err)
        }
        defer conn.Close()

        client := gpb.NewGNMIClient(conn)
        ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
        defer cancel()

        // 1. Get Capabilities
        capResp, err := client.Capabilities(ctx, &gpb.CapabilityRequest{})
        if err != nil {
            log.Fatalf("Failed to get capabilities: %v", err)
        }
        fmt.Println("--- gNMI Capabilities ---")
        fmt.Printf("gNMI Version: %s\n", capResp.GNMIVersion)
        fmt.Println("Supported Models:")
        for _, model := range capResp.SupportedModels {
            fmt.Printf("  - Name: %s, Organization: %s, Version: %s\n", model.Name, model.Organization, model.Version)
        }
        fmt.Println("Supported Encodings:", capResp.SupportedEncodings)

        // 2. Get Hostname (example path using OpenConfig or SR Linux specific YANG)
        // Path for hostname might vary. Using a common one or you can derive from YANG
        // For SR Linux, `system { name { host-name ... } }` is generally the path.
        // The gNMI path equivalent for OpenConfig `hostname` might be: /system/config/hostname
        // For SRL, you might use: /system/name/host-name
        getPath := &gpb.Path{
            Elem: []*gpb.PathElem{
                {Name: "system"},
                {Name: "name"},
                {Name: "host-name"},
            },
        }

        getReq := &gpb.GetRequest{
            Path:    []*gpb.Path{getPath},
            Type:    gpb.GetRequest_CONFIG, // Or GetRequest_STATE for operational data
            Encoding: gpb.Encoding_JSON_IETF, // SR Linux prefers JSON_IETF
        }

        getResp, err := client.Get(ctx, getReq)
        if err != nil {
            log.Fatalf("Failed to get data: %v", err)
        }

        fmt.Println("\n--- Get Hostname Response ---")
        for _, notification := range getResp.Notification {
            for _, update := range notification.Update {
                fmt.Printf("Path: %s, Value: %s\n", gpb.PathToString(update.Path), update.Val.GetJsonIetfVal())
            }
        }
    }

    // auth implements credentials.PerRPCCredentials for gNMI authentication
    type auth struct {
        username string
        password string
    }

    func (a *auth) GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error) {
        return map[string]string{
            "username": a.username,
            "password": a.password,
        }, nil
    }

    func (a *auth) RequireTransportSecurity() bool {
        return true // Or false if you want to allow insecure connections (not recommended for production)
    }
    ```
    **Instructions:**
    1.  Ensure your `srl_2_nodes.clab.yaml` is deployed.
    2.  Run `go mod init day15`.
    3.  Run `go get google.golang.org/grpc github.com/openconfig/gnmi/proto/gnmi`.
    4.  Save the code as `main.go`.
    5.  Run `go run main.go`.

-----

### **Module 4: Advanced GoLang and Network Automation Patterns (Days 16-20)**

**Day 16: gNMI with Go - Set Operations**

  * **Concept:** Using gNMI `<Set>` requests to modify device configuration. Understanding `UPDATE`, `REPLACE`, and `DELETE` operations.
  * **Challenge:** Use a gNMI `<Set>` request to change the hostname of `srl1` to something like "my-srl-router". Verify the change.
  * **Code Example:**
    ```go
    // main.go
    package main

    import (
        "context"
        "crypto/tls"
        "fmt"
        "log"
        "time"

        "google.golang.org/grpc"
        "google.golang.org/grpc/credentials"

        gpb "github.com/openconfig/gnmi/proto/gnmi"
    )

    // auth implements credentials.PerRPCCredentials for gNMI authentication
    type auth struct {
        username string
        password string
    }

    func (a *auth) GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error) {
        return map[string]string{
            "username": a.username,
            "password": a.password,
        }, nil
    }

    func (a *auth) RequireTransportSecurity() bool {
        return true
    }

    func main() {
        target := "clab-srl-2-nodes-srl1:57400"
        username := "admin"
        password := "NokiaSrl1!"

        creds := credentials.NewTLS(&tls.Config{InsecureSkipVerify: true})
        opts := []grpc.DialOption{
            grpc.WithTransportCredentials(creds),
            grpc.WithPerRPCCredentials(&auth{username: username, password: password}),
        }

        conn, err := grpc.Dial(target, opts...)
        if err != nil {
            log.Fatalf("Failed to dial gNMI: %v", err)
        }
        defer conn.Close()

        client := gpb.NewGNMIClient(conn)
        ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
        defer cancel()

        // Path for hostname: /system/name/host-name
        hostnamePath := &gpb.Path{
            Elem: []*gpb.PathElem{
                {Name: "system"},
                {Name: "name"},
                {Name: "host-name"},
            },
        }

        // Create a SetRequest to update the hostname
        setReq := &gpb.SetRequest{
            Update: []*gpb.Update{
                {
                    Path: hostnamePath,
                    Val:  &gpb.TypedValue{Value: &gpb.TypedValue_StringVal{StringVal: "my-srl-router"}},
                },
            },
        }

        _, err = client.Set(ctx, setReq)
        if err != nil {
            log.Fatalf("Failed to set hostname: %v", err)
        }
        fmt.Println("Hostname updated successfully.")

        // Verify the change by getting the hostname again
        getReq := &gpb.GetRequest{
            Path: []*gpb.Path{hostnamePath},
            Type: gpb.GetRequest_CONFIG,
            Encoding: gpb.Encoding_JSON_IETF,
        }

        getResp, err := client.Get(ctx, getReq)
        if err != nil {
            log.Fatalf("Failed to get hostname after set: %v", err)
        }
        fmt.Println("\n--- Verified Hostname ---")
        for _, notification := range getResp.Notification {
            for _, update := range notification.Update {
                fmt.Printf("New Hostname: %s\n", update.Val.GetStringVal())
            }
        }
    }
    ```

**Day 17: gNMI with Go - Streaming Telemetry**

  * **Concept:** Subscribing to streaming telemetry data from SR Linux using gNMI `<Subscribe>` operations. Understanding different subscription modes (ONCE, TARGET\_DEFINED, STREAM).
  * **Challenge:** Subscribe to interface state changes on `srl1` (e.g., `oper-state`) and print updates as they occur.
  * **Code Example:**
    ```go
    // main.go
    package main

    import (
        "context"
        "crypto/tls"
        "fmt"
        "io"
        "log"
        "time"

        "google.golang.org/grpc"
        "google.golang.org/grpc/credentials"

        gpb "github.com/openconfig/gnmi/proto/gnmi"
    )

    // auth implements credentials.PerRPCCredentials for gNMI authentication
    type auth struct {
        username string
        password string
    }

    func (a *auth) GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error) {
        return map[string]string{
            "username": a.username,
            "password": a.password,
        }, nil
    }

    func (a *auth) RequireTransportSecurity() bool {
        return true
    }

    func main() {
        target := "clab-srl-2-nodes-srl1:57400"
        username := "admin"
        password := "NokiaSrl1!"

        creds := credentials.NewTLS(&tls.Config{InsecureSkipVerify: true})
        opts := []grpc.DialOption{
            grpc.WithTransportCredentials(creds),
            grpc.WithPerRPCCredentials(&auth{username: username, password: password}),
        }

        conn, err := grpc.Dial(target, opts...)
        if err != nil {
            log.Fatalf("Failed to dial gNMI: %v", err)
        }
        defer conn.Close()

        client := gpb.NewGNMIClient(conn)
        ctx, cancel := context.WithCancel(context.Background()) // Use WithCancel for streaming
        defer cancel()

        // Path to subscribe to (e.g., all operational state for ethernet-1/1)
        subscribePath := &gpb.Path{
            Elem: []*gpb.PathElem{
                {Name: "interface", Key: map[string]string{"name": "ethernet-1/1"}},
                {Name: "oper-state"},
            },
        }

        subscribeReq := &gpb.SubscribeRequest{
            Request: &gpb.SubscribeRequest_Subscribe{
                Subscribe: &gpb.SubscriptionList{
                    Mode: gpb.SubscriptionList_STREAM, // STREAM for continuous updates
                    UpdatesOnly: true, // Only send updates, not initial state
                    Subscription: []*gpb.Subscription{
                        {
                            Path: subscribePath,
                            Mode: gpb.SubscriptionMode_ON_CHANGE, // Or SAMPLE for periodic
                            SampleInterval: 5 * 1000 * 1000 * 1000, // 5 seconds in nanoseconds for SAMPLE mode
                        },
                    },
                },
            },
        }

        subClient, err := client.Subscribe(ctx)
        if err != nil {
            log.Fatalf("Failed to create subscribe client: %v", err)
        }

        if err := subClient.Send(subscribeReq); err != nil {
            log.Fatalf("Failed to send subscribe request: %v", err)
        }

        fmt.Println("Subscribed to interface oper-state. Waiting for updates (Ctrl+C to stop)...")

        for {
            resp, err := subClient.Recv()
            if err == io.EOF {
                fmt.Println("Server closed the stream.")
                return
            }
            if err != nil {
                log.Fatalf("Failed to receive subscribe response: %v", err)
            }

            switch v := resp.Response.(type) {
            case *gpb.SubscribeResponse_Update:
                for _, update := range v.Update.Update {
                    fmt.Printf("Timestamp: %v, Path: %s, Value: %s\n",
                        time.Unix(0, v.Update.Timestamp),
                        gpb.PathToString(update.Path),
                        update.Val.GetStringVal()) // Assuming string value for oper-state
                }
            case *gpb.SubscribeResponse_SyncResponse:
                fmt.Println("Synchronization finished.")
            default:
                fmt.Printf("Received unknown response type: %T\n", v)
            }
        }
    }
    ```
    **Instructions:**
    1.  Ensure your `srl_2_nodes.clab.yaml` is deployed.
    2.  Run `go mod tidy`.
    3.  Save the code as `main.go`.
    4.  Run `go run main.go`.
    5.  While the program is running, log into `srl1`'s CLI and change the `admin-state` of `ethernet-1/1` to `disable` and then `enable` to see updates:
        ```bash
        docker exec -it clab-srl-2-nodes-srl1 sr_cli
        # In SR Linux CLI:
        enter candidate
        set /interface ethernet-1/1 admin-state disable
        commit
        set /interface ethernet-1/1 admin-state enable
        commit
        exit
        ```

**Day 18: Building a Generic Network Client in Go**

  * **Concept:** Creating a reusable Go package that encapsulates different network interaction methods (SSH, NETCONF, gNMI) for a generic network device.
  * **Challenge:** Create a `networkclient` package with an interface (e.g., `NetworkDevice`) and implementations for SR Linux using SSH and gNMI. The interface should define methods like `GetHostname()`, `SetHostname()`, `GetInterfaceState()`.
  * **Code Example (Conceptual, building on previous days):**
    ```go
    // networkclient/device.go
    package networkclient

    import "fmt"

    // NetworkDevice defines the interface for interacting with a network device.
    type NetworkDevice interface {
        Connect() error
        Close() error
        GetHostname() (string, error)
        SetHostname(hostname string) error
        GetInterfaceState(ifaceName string) (string, error) // Simplified state
        // Add more methods as needed (e.g., GetConfig, SetConfig, Subscribe)
    }

    // SRLGNMIClient implements NetworkDevice for SR Linux via gNMI
    type SRLGNMIClient struct {
        // gNMI client related fields
        Target   string
        Username string
        Password string
        // grpc.ClientConn and gpb.GNMIClient
    }

    func (c *SRLGNMIClient) Connect() error {
        // Implement gNMI connection logic from Day 15/16
        fmt.Printf("Connecting to %s via gNMI...\n", c.Target)
        return nil
    }
    func (c *SRLGNMIClient) Close() error {
        fmt.Println("Closing gNMI connection.")
        return nil
    }
    func (c *SRLGNMIClient) GetHostname() (string, error) {
        fmt.Println("Getting hostname via gNMI.")
        return "example-srl-gnmi", nil // Placeholder
    }
    func (c *SRLGNMIClient) SetHostname(hostname string) error {
        fmt.Printf("Setting hostname to %s via gNMI.\n", hostname)
        return nil
    }
    func (c *SRLGNMIClient) GetInterfaceState(ifaceName string) (string, error) {
        fmt.Printf("Getting state for %s via gNMI.\n", ifaceName)
        return "up", nil // Placeholder
    }

    // SRLSSHClient implements NetworkDevice for SR Linux via SSH
    type SRLSSHClient struct {
        // SSH client related fields
        Target   string
        Username string
        Password string
        // golang.org/x/crypto/ssh.Client
    }

    func (c *SRLSSHClient) Connect() error {
        // Implement SSH connection logic from Day 7/8
        fmt.Printf("Connecting to %s via SSH...\n", c.Target)
        return nil
    }
    func (c *SRLSSHClient) Close() error {
        fmt.Println("Closing SSH connection.")
        return nil
    }
    func (c *SRLSSHClient) GetHostname() (string, error) {
        fmt.Println("Getting hostname via SSH.")
        return "example-srl-ssh", nil // Placeholder
    }
    func (c *SRLSSHClient) SetHostname(hostname string) error {
        fmt.Printf("Setting hostname to %s via SSH.\n", hostname)
        return nil
    }
    func (c *SRLSSHClient) GetInterfaceState(ifaceName string) (string, error) {
        fmt.Printf("Getting state for %s via SSH.\n", ifaceName)
        return "up", nil // Placeholder
    }


    // main.go
    package main

    import (
        "fmt"
        "log"
        "day18/networkclient" // Assuming module name is day18
    )

    func main() {
        // Using the gNMI client implementation
        gnmiClient := &networkclient.SRLGNMIClient{
            Target: "clab-srl-2-nodes-srl1:57400",
            Username: "admin",
            Password: "NokiaSrl1!",
        }
        if err := gnmiClient.Connect(); err != nil {
            log.Fatalf("Error connecting via gNMI: %v", err)
        }
        defer gnmiClient.Close()

        hostname, err := gnmiClient.GetHostname()
        if err != nil {
            log.Printf("Error getting hostname: %v", err)
        } else {
            fmt.Printf("Hostname (gNMI): %s\n", hostname)
        }
        gnmiClient.SetHostname("new-srl-gnmi-host")

        fmt.Println()

        // Using the SSH client implementation
        sshClient := &networkclient.SRLSSHClient{
            Target: "clab-srl-2-nodes-srl1:22",
            Username: "admin",
            Password: "NokiaSrl1!",
        }
        if err := sshClient.Connect(); err != nil {
            log.Fatalf("Error connecting via SSH: %v", err)
        }
        defer sshClient.Close()

        hostname, err = sshClient.GetHostname()
        if err != nil {
            log.Printf("Error getting hostname: %v", err)
        } else {
            fmt.Printf("Hostname (SSH): %s\n", hostname)
        }
        sshClient.SetHostname("new-srl-ssh-host")
    }
    ```
    **Instructions:**
    1.  Create a directory `day18`.
    2.  Inside `day18`, create a directory `networkclient`.
    3.  Save the first code block as `networkclient/device.go`.
    4.  Save the second code block as `main.go` in the `day18` directory.
    5.  Run `go mod init day18`.
    6.  Run `go run main.go`. (Note: The `Connect`, `GetHostname`, etc., methods are placeholders and need to be fully implemented using the SSH and gNMI libraries from previous days for a functional client).

**Day 19: Concurrency with GoRoutines for Automation**

  * **Concept:** Leveraging Go's powerful concurrency model (goroutines and channels) to perform automation tasks on multiple devices in parallel, significantly speeding up operations.
  * **Challenge:** Using the generic network client (or direct SSH/gNMI calls), iterate through multiple SR Linux nodes in your Containerlab topology and retrieve their hostnames concurrently.
  * **Code Example (Conceptual):**
    ```go
    // main.go
    package main

    import (
        "fmt"
        "log"
        "sync"
        "time"
        "day18/networkclient" // Assuming day18 is the module from previous day
    )

    func main() {
        // List of SR Linux nodes from your Containerlab topology (e.g., from Day 10)
        // For demonstration, hardcoding them.
        nodes := []struct{
            Name string
            Target string
            Protocol string
        }{
            {"srl1", "clab-srl-2-nodes-srl1", "gnmi"},
            {"srl2", "clab-srl-2-nodes-srl2", "gnmi"},
        }

        var wg sync.WaitGroup
        results := make(chan string, len(nodes))

        for _, node := range nodes {
            wg.Add(1)
            go func(nodeName, target, protocol string) {
                defer wg.Done()

                var dev networkclient.NetworkDevice
                if protocol == "gnmi" {
                    dev = &networkclient.SRLGNMIClient{
                        Target: target + ":57400",
                        Username: "admin",
                        Password: "NokiaSrl1!",
                    }
                } else if protocol == "ssh" {
                    dev = &networkclient.SRLSSHClient{
                        Target: target + ":22",
                        Username: "admin",
                        Password: "NokiaSrl1!",
                    }
                } else {
                    results <- fmt.Sprintf("Error for %s: Unsupported protocol %s", nodeName, protocol)
                    return
                }


                err := dev.Connect()
                if err != nil {
                    results <- fmt.Sprintf("Error connecting to %s: %v", nodeName, err)
                    return
                }
                defer dev.Close()

                hostname, err := dev.GetHostname()
                if err != nil {
                    results <- fmt.Sprintf("Error getting hostname for %s: %v", nodeName, err)
                    return
                }
                results <- fmt.Sprintf("Node %s (via %s) hostname: %s", nodeName, protocol, hostname)

            }(node.Name, node.Target, node.Protocol)
        }

        wg.Wait()
        close(results)

        fmt.Println("\n--- Concurrent Hostname Retrieval Results ---")
        for res := range results {
            fmt.Println(res)
        }
    }
    ```
    **Instructions:**
    1.  Ensure your `srl_2_nodes.clab.yaml` is deployed.
    2.  Ensure your `day18/networkclient` package is present and ideally, its methods are somewhat implemented (even if they just print a message for this exercise).
    3.  Save the code as `main.go`.
    4.  Run `go run main.go`.

**Day 20: Building a Simple Automation Tool & Next Steps**

  * **Concept:** Integrating all learned concepts into a practical, command-line automation tool. Discussing packaging Go applications, error handling best practices, and continuous learning.
  * **Challenge:** Create a Go application that takes arguments (e.g., node name, command, protocol) and performs the requested operation using the `networkclient` package. Package it into a single executable.
  * **Code Example (Conceptual):**
    ```go
    // main.go
    package main

    import (
        "flag"
        "fmt"
        "log"
        "day18/networkclient" // Assuming day18 is the module from previous day
    )

    func main() {
        nodeName := flag.String("node", "", "Containerlab node name (e.g., srl1)")
        action := flag.String("action", "", "Action to perform (get-hostname, set-hostname, get-interface-state)")
        protocol := flag.String("protocol", "gnmi", "Protocol to use (gnmi, ssh)")
        value := flag.String("value", "", "Value for set actions (e.g., new hostname)")

        flag.Parse()

        if *nodeName == "" || *action == "" {
            flag.Usage()
            log.Fatal("Node name and action are required.")
        }

        var dev networkclient.NetworkDevice
        target := fmt.Sprintf("clab-srl-2-nodes-%s", *nodeName) // Containerlab convention

        switch *protocol {
        case "gnmi":
            dev = &networkclient.SRLGNMIClient{
                Target: target + ":57400",
                Username: "admin",
                Password: "NokiaSrl1!",
            }
        case "ssh":
            dev = &networkclient.SRLSSHClient{
                Target: target + ":22",
                Username: "admin",
                Password: "NokiaSrl1!",
            }
        default:
            log.Fatalf("Unsupported protocol: %s", *protocol)
        }

        err := dev.Connect()
        if err != nil {
            log.Fatalf("Failed to connect to %s via %s: %v", *nodeName, *protocol, err)
        }
        defer dev.Close()

        switch *action {
        case "get-hostname":
            hostname, err := dev.GetHostname()
            if err != nil {
                log.Fatalf("Error getting hostname: %v", err)
            }
            fmt.Printf("Hostname for %s: %s\n", *nodeName, hostname)
        case "set-hostname":
            if *value == "" {
                log.Fatal("Value is required for set-hostname action.")
            }
            err := dev.SetHostname(*value)
            if err != nil {
                log.Fatalf("Error setting hostname: %v", err)
            }
            fmt.Printf("Hostname for %s set to: %s\n", *nodeName, *value)
        case "get-interface-state":
            // You'd need an additional flag for interface name here
            log.Fatal("get-interface-state requires an interface name (not implemented in this example).")
        default:
            log.Fatalf("Unknown action: %s", *action)
        }
    }
    ```
    **Instructions:**
    1.  Ensure your `srl_2_nodes.clab.yaml` is deployed.
    2.  Ensure your `day18/networkclient` package is fully implemented.
    3.  Save the code as `main.go`.
    4.  Compile the tool: `go build -o network-tool main.go`.
    5.  Run it: `./network-tool -node srl1 -action get-hostname -protocol gnmi`.
    6.  Run it: `./network-tool -node srl1 -action set-hostname -value "new-test-host" -protocol gnmi`.
    7.  Verify: `./network-tool -node srl1 -action get-hostname -protocol gnmi`.

-----

**Important Notes:**

  * **Error Handling:** The code examples provide basic error handling. In production, you would implement more robust error handling, logging, and retry mechanisms.
  * **Authentication:** Using `ssh.InsecureIgnoreHostKey()` and hardcoded passwords (`NokiaSrl1!`) is for lab environments only. For production, implement proper SSH key-based authentication, environment variables, or secure credential management. For gNMI, proper TLS certificates should be used.
  * **YANG Paths:** Be precise with YANG paths for gNMI and NETCONF. Refer to the Nokia SR Linux YANG documentation for exact paths. You can also use `sr_cli tools dump model` on SR Linux to see the loaded YANG modules and their structure.
  * **Containerlab Management IPs:** Containerlab typically sets up a Docker bridge network where nodes are resolvable by their Containerlab-assigned hostnames (`clab-<lab_name>-<node_name>`). Use these hostnames for connecting from your Go programs running on the host.
  * **Idempotency:** When performing configuration changes, aim for idempotent operations (applying the same config multiple times yields the same result without error).
  * **Community and Resources:** Actively engage with the Go and network automation communities. Refer to official Go documentation, Nokia SR Linux documentation, and OpenConfig specifications.

This tutorial provides a structured path to learning Go for Nokia SR Linux automation. Good luck on your 20-day journey\!