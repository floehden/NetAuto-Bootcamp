# Day 01

**Prerequisites:**

  * **Basic Linux Command Line Knowledge:** Familiarity with `cd`, `ls`, `mkdir`, `docker`, `ssh`.
  * **Fundamental Networking Concepts:** Understanding of IP addressing, routing, interfaces, VLANs.
  * **Docker Installed:** Containerlab relies on Docker. Ensure it's installed and running on your Linux machine (or WSL2 on Windows, or OrbStack/Docker Desktop on macOS with a Linux VM).
  * **Containerlab Installed:** Follow the official Containerlab installation guide: [https://containerlab.dev/install/](https://containerlab.dev/install/)
  * **Arista cEOS-lab Image:** You'll need to download the cEOS-lab image from the Arista support portal (requires an Arista account, free lab versions are available). Import it into Docker: `docker import cEOS-lab-<version>.tar.xz arista/ceos:<version>` (e.g., `arista/ceos:4.30.6M`).
  * **Go Installed:** Download and install Go from [https://golang.org/doc/install](https://golang.org/doc/install).


### **Module 1: GoLang Fundamentals (Days 1-5)**

## **Day 1: Getting Started with GoLang**

  * **Introduction:** Why Go for network automation? Concurrency, performance, static typing, strong standard library. Basic Go program structure (`package main`, `func main()`, `import`, `fmt.Println`).

  * **Code Example: Hello World**

    ```go
    // hello.go
    package main

    import "fmt"

    func main() {
        fmt.Println("Hello, Go!")
    }
    ```

      * To run: `go run hello.go`

  * **Challenge 1:** Write a Go program that prints your name and a simple network-related phrase (e.g., "Automating networks with Go\!").

## **Day 2: Variables, Data Types, and Operators**

  * **Introduction:** Declaring variables (`var`, `:=`), common data types (int, string, bool, float), arithmetic, comparison, and logical operators. Type inference.

  * **Code Example: Network Device Info**

    ```go
    // device_info.go
    package main

    import "fmt"

    func main() {
        deviceName := "my-arista-router"
        ipAddress := "192.168.1.1"
        ports := 24
        isOnline := true

        fmt.Printf("Device: %s\n", deviceName)
        fmt.Printf("IP Address: %s\n", ipAddress)
        fmt.Printf("Number of Ports: %d\n", ports)
        fmt.Printf("Is Online: %t\n", isOnline)

        // Example operation
        uptimeHours := 120
        days := uptimeHours / 24
        fmt.Printf("Uptime: %d days\n", days)
    }
    ```

  * **Challenge 2:** Create a program that declares variables for a network segment (e.g., `networkName`, `subnet`, `maskLength`, `numberOfHosts`). Calculate and print the number of usable hosts in that subnet.

## **Day 3: Control Flow (If/Else, For Loops, Switch)**

  * **Introduction:** Conditional statements (`if`, `else if`, `else`), looping constructs (`for` loop variations), `switch` statements for multiple conditions.

  * **Code Example: Device Status Check**

    ```go
    // status_check.go
    package main

    import "fmt"

    func main() {
        cpuUsage := 85
        memoryUsage := 60
        interfaceStatus := "up"

        if cpuUsage > 80 {
            fmt.Println("High CPU usage warning!")
        } else if cpuUsage > 50 {
            fmt.Println("Moderate CPU usage.")
        } else {
            fmt.Println("CPU usage normal.")
        }

        if memoryUsage > 75 {
            fmt.Println("High memory usage warning!")
        }

        switch interfaceStatus {
        case "up":
            fmt.Println("Interface is up and running.")
        case "down":
            fmt.Println("Interface is down!")
        default:
            fmt.Println("Unknown interface status.")
        }

        fmt.Println("Counting from 1 to 5:")
        for i := 1; i <= 5; i++ {
            fmt.Println(i)
        }
    }
    ```

  * **Challenge 3:** Write a program that simulates checking interface status for 5 interfaces. If an interface is "down", print an alert message and the interface number. If it's "up", print "Interface X is healthy."

## **Day 4: Functions and Error Handling**

  * **Introduction:** Defining functions, parameters, return values. Multiple return values (common for error handling). Basic error handling using `if err != nil`. The `defer` keyword.

  * **Code Example: Ping Function**

    ```go
    // ping.go
    package main

    import (
        "fmt"
        "errors"
    )

    func pingDevice(ip string) (string, error) {
        if ip == "192.168.1.100" { // Simulate a device being unreachable
            return "", errors.New("device unreachable")
        }
        return fmt.Sprintf("Ping to %s successful!", ip), nil
    }

    func main() {
        result, err := pingDevice("192.168.1.1")
        if err != nil {
            fmt.Printf("Error: %v\n", err)
        } else {
            fmt.Println(result)
        }

        result, err = pingDevice("192.168.1.100")
        if err != nil {
            fmt.Printf("Error: %v\n", err)
        } else {
            fmt.Println(result)
        }
    }
    ```

  * **Challenge 4:** Create a function `configureInterface(interfaceName string, ipAddress string, netmask string)` that returns a string representing the configuration command (e.g., "interface Ethernet1\\n ip address 10.0.0.1/24\\n no shutdown") and an error if the IP address or netmask is invalid. Call this function and handle the potential error.

## **Day 5: Slices, Maps, and Structs**

  * **Introduction:** Dynamic arrays (slices), key-value pairs (maps), and custom data structures (structs). How to iterate over them.

  * **Code Example: Network Inventory**

    ```go
    // inventory.go
    package main

    import "fmt"

    type Device struct {
        Hostname    string
        IPAddress   string
        Location    string
        Model       string
        Interfaces  []string
    }

    func main() {
        // Slice of strings
        var vlans []string
        vlans = append(vlans, "VLAN10", "VLAN20", "VLAN30")
        fmt.Println("VLANs:", vlans)

        // Map of device IPs to hostnames
        deviceMap := make(map[string]string)
        deviceMap["10.0.0.1"] = "leaf1"
        deviceMap["10.0.0.2"] = "leaf2"
        fmt.Println("Device Map:", deviceMap)

        // Struct for a network device
        router := Device{
            Hostname:   "core-router-1",
            IPAddress:  "172.16.0.1",
            Location:   "Data Center A",
            Model:      "Arista 7050S",
            Interfaces: []string{"Ethernet1", "Ethernet2", "Loopback0"},
        }
        fmt.Printf("Router Hostname: %s, Model: %s\n", router.Hostname, router.Model)
        fmt.Println("Router Interfaces:")
        for _, iface := range router.Interfaces {
            fmt.Println("- ", iface)
        }
    }
    ```

  * **Challenge 5:** Define a `NetworkSegment` struct with fields like `Name`, `VLANID`, and a slice of `Device` structs that belong to this segment. Populate an array (slice) of `NetworkSegment`s and print out all devices within each segment.

### **Module 2: Containerlab Setup & Basic cEOS Interaction (Days 6-10)**

## **Day 6: Introduction to Containerlab**

  * **Introduction:** What is Containerlab? Why use it for network labs? Basic topology definition (YAML). Deploying and destroying labs.

  * **Pre-requisite:** Ensure Docker is installed and `arista/ceos` image is imported.

  * **Code Example: Simple Containerlab Topology (YAML)**

    Create `simple_lab.yaml`:

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

      * Deploy: `sudo containerlab deploy -t simple_lab.yaml`
      * Destroy: `sudo containerlab destroy -t simple_lab.yaml`

  * **Challenge 6:** Deploy the `simple_lab.yaml`. Use `sudo containerlab graph -t simple_lab.yaml` to visualize the lab. Log into `ceos1` via `docker exec -it clab-simple-arista-lab-ceos1 Cli` and verify interfaces. Then destroy the lab.

## **Day 7: Initial cEOS Configuration with Containerlab**

  * **Introduction:** Providing startup configurations to cEOS nodes in Containerlab. Using `startup-config` field in the YAML. Accessing cEOS CLI.

  * **Code Example: cEOS with Initial Config**

    Create `initial_config.clab.yaml`:

    ```yaml
    name: ceos-configured-lab
    topology:
      nodes:
        ceos1:
          kind: ceos
          image: arista/ceos:<your_ceos_version>
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
          kind: ceos
          image: arista/ceos:<your_ceos_version>
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

      * Deploy: `sudo containerlab deploy -t initial_config.clab.yaml`

  * **Challenge 7:** Deploy `initial_config.clab.yaml`. SSH into `ceos1` (Containerlab usually maps port 22 to a random high port on the host, you can find it with `docker port <container_name>`). Verify the `Ethernet1` configuration and ping `ceos2`'s `Ethernet1` IP. Destroy the lab.

## **Day 8: GoLang Basic File I/O for Config Files**

  * **Introduction:** Reading and writing files in Go. Reading topology and configuration data from files.

  * **Code Example: Reading a Topology File**

    ```go
    // read_topo.go
    package main

    import (
        "fmt"
        "os"
        "io/ioutil"
    )

    func main() {
        filePath := "initial_config.clab.yaml" // Use the YAML file from Day 7

        content, err := ioutil.ReadFile(filePath)
        if err != nil {
            fmt.Printf("Error reading file: %v\n", err)
            return
        }
        fmt.Println("--- Topology File Content ---")
        fmt.Println(string(content))
    }
    ```

  * **Challenge 8:** Create a simple text file `device_list.txt` with a list of device hostnames (one per line). Write a Go program that reads this file and prints each hostname, prefixed with "Processing device: ".

## **Day 9: Structuring Go Projects and Modules**

  * **Introduction:** Go modules for dependency management. Project structure (`main.go`, `go.mod`, utility packages). Best practices for organizing Go code.

  * **Code Example: Simple Module**

    ```
    myproject/
    ├── go.mod
    ├── main.go
    └── utils/
        └── network.go
    ```

    `myproject/go.mod`:

    ```
    module example.com/myproject

    go 1.22
    ```

    `myproject/utils/network.go`:

    ```go
    package utils

    import "fmt"

    func GreetNetworkEngineer(name string) {
        fmt.Printf("Hello, %s! Happy automating!\n", name)
    }
    ```

    `myproject/main.go`:

    ```go
    package main

    import (
        "example.com/myproject/utils" // Import your custom package
        "fmt"
    )

    func main() {
        fmt.Println("Starting network automation project.")
        utils.GreetNetworkEngineer("Alice")
    }
    ```

      * To run: `cd myproject && go mod tidy && go run .`

  * **Challenge 9:** Refactor your `configureInterface` function from Day 4 into a `config` package within your project. Update `main.go` to import and use it.

## **Day 10: Basic Interaction with cEOS (Manual SSH/eAPI prep)**

  * **Introduction:** Understanding how to connect to cEOS. SSH for CLI access. Introduction to eAPI (EOS API) as the programmatic interface. Enabling eAPI on cEOS.

  * **Code Example: Enabling eAPI on cEOS (via startup-config)**

    Update your `initial_config.clab.yaml` (from Day 7) to include:

    ```yaml
    # ... inside each ceos node config ...
            management api http-commands
              no shutdown
              protocol http # Or https, depending on your preference
              # To allow specific sources (optional, but good practice)
              # ip http-server source-interface Management0
              # ip http-server protocol https ssl profile server
    # ...
    ```

      * Deploy the lab with this updated YAML.
      * Verify eAPI is enabled by trying to access it from your host (e.g., using `curl`). You'll need the container's IP address. You can get it with `docker inspect <container_name> | grep "IPAddress"`. Then `curl -k -u admin:admin https://<container_ip>/command-api`. (Note: replace `admin:admin` with your configured username/password and adjust for `http` or `https`).

  * **Challenge 10:** Deploy the lab with eAPI enabled. From your host machine, use `curl` to send a `show version` command to one of your cEOS nodes via eAPI. Parse the JSON output manually to find the EOS version.

## **Module 3: GoLang and Arista eAPI (Days 11-15)**

## **Day 11: Introduction to Arista eAPI and GoeAPI Library**

  * **Introduction:** eAPI as a JSON-RPC based API. Understanding the request and response structure. Introducing `goeapi`, the official Go client library for Arista eAPI.

  * **Installation:** `go get github.com/aristanetworks/goeapi`

  * **Code Example: Basic `goeapi` `show version`**

    ```go
    // show_version.go
    package main

    import (
        "fmt"
        "os"

        "github.com/aristanetworks/goeapi"
    )

    func main() {
        // You'll need to create a .eapi.conf file in your home directory
        // Example ~/.eapi.conf:
        // [connection:ceos1]
        // host=<ceos1_container_ip>
        // username=admin
        // password=admin
        // transport=http # or https

        node, err := goeapi.Connect("ceos1") // "ceos1" refers to the connection name in .eapi.conf
        if err != nil {
            fmt.Printf("Error connecting to node: %v\n", err)
            os.Exit(1)
        }

        cmds := []string{"show version"}
        response, err := node.RunCommands(cmds)
        if err != nil {
            fmt.Printf("Error running commands: %v\n", err)
            os.Exit(1)
        }

        // The response is a slice of maps (interface{} for dynamic JSON)
        // You'll typically unmarshal this into a specific struct for better handling
        fmt.Printf("Show Version Response: %+v\n", response)

        // For now, let's just print the raw response. Later we'll parse it.
    }
    ```

  * **Challenge 11:** Deploy a Containerlab topology with one cEOS device with eAPI enabled. Create the `~/.eapi.conf` file with the correct IP and credentials. Run the `show_version.go` example and observe the raw JSON output.

## **Day 12: Parsing eAPI Responses with Go Structs**

  * **Introduction:** Defining Go structs to match JSON response structures. Using `json.Unmarshal` to parse eAPI responses into Go objects.

  * **Code Example: Parsing `show version`**

    ```go
    // parse_version.go
    package main

    import (
        "encoding/json"
        "fmt"
        "os"

        "github.com/aristanetworks/goeapi"
    )

    // Define a struct to match the "show version" JSON output structure
    type ShowVersionResponse struct {
        Architecture       string `json:"architecture"`
        InternalBuildID    string `json:"internalBuildId"`
        InternalBuildVersion string `json:"internalBuildVersion"`
        ModelName          string `json:"modelName"`
        SoftwareImageVersion string `json:"softwareImageVersion"`
        SystemMACAddress   string `json:"systemMacAddress"`
        Uptime             float64 `json:"uptime"`
    }

    func main() {
        node, err := goeapi.Connect("ceos1")
        if err != nil {
            fmt.Printf("Error connecting to node: %v\n", err)
            os.Exit(1)
        }

        cmds := []string{"show version"}
        response, err := node.RunCommands(cmds)
        if err != nil {
            fmt.Printf("Error running commands: %v\n", err)
            os.Exit(1)
        }

        // eAPI returns a slice of results, even for single commands
        if len(response) > 0 {
            var versionResp ShowVersionResponse
            // The response element is usually a map[string]interface{}
            // We need to marshal it back to JSON bytes and then unmarshal into our struct
            jsonBytes, err := json.Marshal(response[0].(map[string]interface{})["result"])
            if err != nil {
                fmt.Printf("Error marshalling result: %v\n", err)
                os.Exit(1)
            }
            if err := json.Unmarshal(jsonBytes, &versionResp); err != nil {
                fmt.Printf("Error unmarshalling JSON: %v\n", err)
                os.Exit(1)
            }
            fmt.Printf("Software Version: %s\n", versionResp.SoftwareImageVersion)
            fmt.Printf("System MAC: %s\n", versionResp.SystemMACAddress)
            fmt.Printf("Uptime (seconds): %.0f\n", versionResp.Uptime)
        } else {
            fmt.Println("No response received for show version.")
        }
    }
    ```

  * **Challenge 12:** Using `goeapi`, get the output of `show interfaces brief`. Define a Go struct to parse the key information (e.g., interface name, status, IP address). Print a formatted summary of each interface.

## **Day 13: Sending Configuration Commands via eAPI**

  * **Introduction:** Using `goeapi` to send configuration commands to cEOS. The `Configure` method. Understanding the `config` context in EOS.

  * **Code Example: Configuring a Loopback Interface**

    ```go
    // configure_loopback.go
    package main

    import (
        "fmt"
        "os"

        "github.com/aristanetworks/goeapi"
    )

    func main() {
        node, err := goeapi.Connect("ceos1")
        if err != nil {
            fmt.Printf("Error connecting to node: %v\n", err)
            os.Exit(1)
        }

        configCmds := []string{
            "interface Loopback0",
            "ip address 1.1.1.1/32",
            "description \"Configured by GoLang\"",
        }

        response, err := node.Configure(configCmds)
        if err != nil {
            fmt.Printf("Error configuring device: %v\n", err)
            os.Exit(1)
        }

        fmt.Printf("Configuration successful: %+v\n", response)

        // Verify configuration
        showCmds := []string{"show running-config interface Loopback0"}
        showResponse, err := node.RunCommands(showCmds)
        if err != nil {
            fmt.Printf("Error verifying config: %v\n", err)
            os.Exit(1)
        }
        fmt.Println("\n--- Verified Loopback0 Configuration ---")
        fmt.Println(showResponse[0].(map[string]interface{})["result"].(map[string]interface{})["output"])
    }
    ```

  * **Challenge 13:** Deploy a fresh cEOS lab. Write a Go program using `goeapi` to configure `Ethernet1` with an IP address (e.g., `10.10.10.1/24`) and a description. After configuration, use `goeapi` to retrieve and print the `show ip interface brief` output to confirm.

## **Day 14: Advanced eAPI Operations: Multiple Commands & Error Handling**

  * **Introduction:** Sending multiple commands in a single eAPI call. Robust error checking for each command within a batch.

  * **Code Example: Batch Configuration and Error Checking**

    ```go
    // batch_config.go
    package main

    import (
        "fmt"
        "os"

        "github.com/aristanetworks/goeapi"
    )

    func main() {
        node, err := goeapi.Connect("ceos1")
        if err != nil {
            fmt.Printf("Error connecting to node: %v\n", err)
            os.Exit(1)
        }

        configCmds := []string{
            "interface Loopback1",
            "ip address 2.2.2.2/32",
            "description \"Another Loopback\"",
            "interface Ethernet99", // This interface likely doesn't exist
            "ip address 3.3.3.3/24",
        }

        response, err := node.Configure(configCmds)
        if err != nil {
            fmt.Printf("Error during batch configuration: %v\n", err)
            // You might need to inspect the 'response' object if available
            // to see which specific command failed if the underlying eAPI
            // library provides that granularity in the error.
        } else {
            fmt.Printf("Batch configuration results: %+v\n", response)
        }

        // For more granular error checking, you might need to send commands one by one
        // or check the "errors" field in the raw JSON response if the API returns it.
        // goeapi often abstracts this, returning an error for the whole batch on first failure.
    }
    ```

  * **Challenge 14:** Create a Go program that attempts to configure two loopback interfaces (e.g., Loopback0 and Loopback1) and then tries to configure a non-existent interface (e.g., Loopback99). Observe how `goeapi` handles the error and what is returned. Try to implement a mechanism to identify which specific command caused the failure.

## **Day 15: Templating Configurations with Go's `text/template`**

  * **Introduction:** Using Go's built-in templating engine (`text/template`) to generate dynamic configurations based on input data. This is crucial for scalable automation.

  * **Code Example: Interface Template**

    Create `interface_template.txt`:

    ```
    interface {{.InterfaceName}}
      description "{{.Description}}"
      ip address {{.IPAddress}}/{{.PrefixLength}}
      no shutdown
    ```

    `generate_config.go`:

    ```go
    package main

    import (
        "bytes"
        "fmt"
        "os"
        "text/template"
    )

    type InterfaceData struct {
        InterfaceName string
        Description   string
        IPAddress     string
        PrefixLength  int
    }

    func main() {
        data := InterfaceData{
            InterfaceName: "Ethernet2",
            Description:   "Uplink to Core",
            IPAddress:     "10.0.1.1",
            PrefixLength:  24,
        }

        tmpl, err := template.ParseFiles("interface_template.txt")
        if err != nil {
            fmt.Printf("Error parsing template: %v\n", err)
            os.Exit(1)
        }

        var buf bytes.Buffer
        err = tmpl.Execute(&buf, data)
        if err != nil {
            fmt.Printf("Error executing template: %v\n", err)
            os.Exit(1)
        }

        fmt.Println("--- Generated Configuration ---")
        fmt.Println(buf.String())

        // In a real scenario, you would then send buf.String() to the device via eAPI.
    }
    ```

  * **Challenge 15:** Create a template for configuring a BGP neighbor, including `remote-as`, `neighbor IP`, and `update-source`. Define a struct to hold the necessary BGP neighbor data. Generate a BGP configuration for two different neighbors and print the generated config.

## **Module 4: Advanced GoLang for Network Automation (Days 16-20)**

## **Day 16: Concurrency with Goroutines and Channels**

  * **Introduction:** Go's powerful concurrency model (goroutines and channels). Running multiple network operations in parallel.

  * **Code Example: Concurrent Pings**

    ```go
    // concurrent_ping.go
    package main

    import (
        "fmt"
        "time"
        "sync" // For waiting on goroutines
    )

    func pingDevice(ip string, wg *sync.WaitGroup, results chan string) {
        defer wg.Done()
        fmt.Printf("Pinging %s...\n", ip)
        time.Sleep(2 * time.Second) // Simulate network delay
        result := fmt.Sprintf("Ping to %s complete.", ip)
        results <- result // Send result to channel
    }

    func main() {
        devices := []string{"192.168.1.1", "192.168.1.2", "192.168.1.3", "192.168.1.4"}
        var wg sync.WaitGroup
        results := make(chan string, len(devices)) // Buffered channel

        for _, ip := range devices {
            wg.Add(1)
            go pingDevice(ip, &wg, results)
        }

        wg.Wait() // Wait for all goroutines to finish
        close(results) // Close the channel when all results are sent

        fmt.Println("\n--- Ping Results ---")
        for res := range results {
            fmt.Println(res)
        }
    }
    ```

  * **Challenge 16:** Modify your eAPI `show version` program to connect to two different cEOS nodes concurrently (you'll need a Containerlab setup with two nodes and corresponding `~/.eapi.conf` entries). Use goroutines and channels to retrieve and print the version information from both devices simultaneously.

## **Day 17: Building a Simple CLI Tool with `cobra` or `flag`**

  * **Introduction:** Creating command-line interface (CLI) tools in Go. Using the `flag` package for simple arguments or the `cobra` library for more complex CLIs.

  * **Code Example: Basic Flag Usage**

    ```go
    // cli_tool.go
    package main

    import (
        "flag"
        "fmt"
        "os"
    )

    func main() {
        hostname := flag.String("host", "", "Hostname or IP address of the device")
        command := flag.String("cmd", "", "Command to execute on the device")
        username := flag.String("user", "admin", "Username for device access")
        password := flag.String("pass", "admin", "Password for device access")

        flag.Parse() // Parse the command-line arguments

        if *hostname == "" || *command == "" {
            fmt.Println("Usage: cli_tool -host <ip> -cmd <command> [-user <user>] [-pass <pass>]")
            os.Exit(1)
        }

        fmt.Printf("Connecting to %s with user %s\n", *hostname, *username)
        fmt.Printf("Executing command: %s\n", *command)
        // In a real tool, you would use goeapi here with the provided host/user/pass
    }
    ```

      * To run: `go run cli_tool.go -host 10.0.0.1 -cmd "show version"`

  * **Challenge 17:** Extend the CLI tool to accept an additional flag, `--config-file`, which takes a path to a configuration file. If the `--config-file` flag is provided, read the content of that file and print it, simulating sending it to the device.

## **Day 18: Working with JSON Configuration Files (External Data)**

  * **Introduction:** Reading configuration data from external JSON files. Marshalling and unmarshalling JSON in Go.

  * **Code Example: Device Configuration from JSON**

    Create `device_config.json`:

    ```json
    {
        "hostname": "leaf3",
        "ip_address": "10.0.0.3",
        "loopback_ip": "3.3.3.3/32",
        "interfaces": [
            {
                "name": "Ethernet1",
                "ip": "10.0.3.1/30",
                "description": "Link to spine1"
            },
            {
                "name": "Ethernet2",
                "ip": "10.0.3.5/30",
                "description": "Link to spine2"
            }
        ]
    }
    ```

    `read_json_config.go`:

    ```go
    package main

    import (
        "encoding/json"
        "fmt"
        "io/ioutil"
        "os"
    )

    type Interface struct {
        Name        string `json:"name"`
        IP          string `json:"ip"`
        Description string `json:"description"`
    }

    type DeviceConfig struct {
        Hostname    string      `json:"hostname"`
        IPAddress   string      `json:"ip_address"`
        LoopbackIP  string      `json:"loopback_ip"`
        Interfaces  []Interface `json:"interfaces"`
    }

    func main() {
        jsonData, err := ioutil.ReadFile("device_config.json")
        if err != nil {
            fmt.Printf("Error reading JSON file: %v\n", err)
            os.Exit(1)
        }

        var config DeviceConfig
        err = json.Unmarshal(jsonData, &config)
        if err != nil {
            fmt.Printf("Error unmarshalling JSON: %v\n", err)
            os.Exit(1)
        }

        fmt.Printf("Device Hostname: %s\n", config.Hostname)
        fmt.Printf("Loopback IP: %s\n", config.LoopbackIP)
        fmt.Println("Configuring Interfaces:")
        for _, iface := range config.Interfaces {
            fmt.Printf("  - %s: IP %s, Description: %s\n", iface.Name, iface.IP, iface.Description)
            // In a real scenario, you'd generate commands and send via eAPI
        }
    }
    ```

  * **Challenge 18:** Create a Containerlab topology with a single cEOS node. Write a Go program that reads device configuration from a JSON file (similar to `device_config.json` but specific to your lab node). Use `goeapi` and the parsed JSON data to configure the hostname, a loopback interface, and at least one Ethernet interface on the cEOS router.

## **Day 19: Building a Network Discovery and Reporting Tool**

  * **Introduction:** Combining previous concepts to build a more complex tool. Discovering information from devices and generating reports.

  * **Code Example: Device Inventory Script**

    ```go
    package main

    import (
        "encoding/json"
        "fmt"
        "os"
        "sync"
        "time"

        "github.com/aristanetworks/goeapi"
    )

    type DeviceInventory struct {
        Hostname     string `json:"hostname"`
        SoftwareVersion string `json:"software_version"`
        SystemMAC    string `json:"system_mac"`
        ManagementIP string `json:"management_ip"`
        Status       string `json:"status"`
        Error        string `json:"error,omitempty"`
    }

    func collectDeviceInfo(nodeName string, nodeIP string, wg *sync.WaitGroup, results chan DeviceInventory) {
        defer wg.Done()

        inventory := DeviceInventory{
            Hostname:     nodeName,
            ManagementIP: nodeIP,
            Status:       "Failed",
        }

        // Temporarily override .eapi.conf logic for direct IP connection for simplicity in this example
        // In a real tool, you might dynamically generate .eapi.conf or use goeapi's direct connection options if available
        // For this example, let's assume we establish a direct connection or mock it
        // A more robust solution would involve dynamically creating a connection profile or using a custom transport in goeapi

        // Simulating eAPI connection and data retrieval
        // In a real scenario, you'd connect using goeapi.Connect() and run commands
        // For now, let's just mock some success/failure
        if nodeName == "ceos-unreachable" {
            inventory.Error = "Device unreachable"
            results <- inventory
            return
        }

        fmt.Printf("Collecting info from %s (%s)...\n", nodeName, nodeIP)
        time.Sleep(3 * time.Second) // Simulate API call delay

        // Mocking successful data retrieval
        inventory.Status = "Success"
        inventory.SoftwareVersion = "4.30.6M"
        inventory.SystemMAC = "00:11:22:33:44:55"

        results <- inventory
    }

    func main() {
        // Assume a Containerlab lab is deployed with these nodes
        // Make sure your .eapi.conf has entries for these
        nodes := map[string]string{
            "ceos1": "172.17.0.2", // Replace with actual container IP or hostname
            "ceos2": "172.17.0.3",
            "ceos-unreachable": "172.17.0.4", // Simulate an unreachable device
        }

        var wg sync.WaitGroup
        results := make(chan DeviceInventory, len(nodes))

        for name, ip := range nodes {
            wg.Add(1)
            go collectDeviceInfo(name, ip, &wg, results)
        }

        wg.Wait()
        close(results)

        var inventoryReport []DeviceInventory
        for inv := range results {
            inventoryReport = append(inventoryReport, inv)
        }

        jsonReport, err := json.MarshalIndent(inventoryReport, "", "  ")
        if err != nil {
            fmt.Printf("Error marshalling inventory report: %v\n", err)
            os.Exit(1)
        }

        fmt.Println("\n--- Network Inventory Report ---")
        fmt.Println(string(jsonReport))
    }
    ```

  * **Challenge 19:** Deploy a Containerlab topology with three cEOS nodes. For each node, use `goeapi` (not mocked data) to retrieve its `show version` and `show ip interface brief` output. Combine this information into a `DeviceInventory` struct for each device. Generate a single JSON report containing the inventory for all devices.

## **Day 20: Putting It All Together: Automated Lab Deployment & Configuration**

  * **Introduction:** Building a script that automates the entire process: deploying a Containerlab, configuring devices from templates/JSON, and verifying the state.

  * **Code Example: Full Automation Workflow**

    This will be a more complex script, combining elements from previous days.

    1.  **Define Topology (YAML):** `automation_lab.clab.yaml` with a few cEOS nodes.
    2.  **Define Device Configurations (JSON/Templates):** Individual files for each device.
    3.  **Go Program:**
          * Reads the topology YAML.
          * Deploys the Containerlab topology using `os/exec` to run `containerlab deploy`.
          * Waits for devices to boot up (e.g., a simple `time.Sleep` or more robust health check).
          * For each device:
              * Reads its specific configuration (from JSON or template).
              * Uses `goeapi` to connect and push the configuration.
              * Uses `goeapi` to run `show` commands to verify the configuration (e.g., `show ip interface brief`, `show bgp summary`).
              * Reports success or failure.
          * Optionally, `defer` the `containerlab destroy` command.

    <!-- end list -->

    ```go
    // automate_lab.go
    package main

    import (
        "bytes"
        "encoding/json"
        "fmt"
        "io/ioutil"
        "os"
        "os/exec"
        "strings"
        "sync"
        "text/template"
        "time"

        "github.com/aristanetworks/goeapi"
    )

    // Structs for parsing JSON config
    type Interface struct {
        Name        string `json:"name"`
        IP          string `json:"ip"`
        Description string `json:"description"`
    }

    type DeviceConfig struct {
        Hostname    string      `json:"hostname"`
        LoopbackIP  string      `json:"loopback_ip"`
        Interfaces  []Interface `json:"interfaces"`
    }

    // Function to deploy Containerlab
    func deployLab(topoFile string) error {
        fmt.Printf("Deploying Containerlab topology: %s\n", topoFile)
        cmd := exec.Command("sudo", "containerlab", "deploy", "-t", topoFile)
        cmd.Stdout = os.Stdout
        cmd.Stderr = os.Stderr
        return cmd.Run()
    }

    // Function to destroy Containerlab
    func destroyLab(topoFile string) error {
        fmt.Printf("Destroying Containerlab topology: %s\n", topoFile)
        cmd := exec.Command("sudo", "containerlab", "destroy", "-t", topoFile, "--cleanup")
        cmd.Stdout = os.Stdout
        cmd.Stderr = os.Stderr
        return cmd.Run()
    }

    // Function to get container IP (simplistic, assumes standard Docker bridge network)
    func getContainerIP(containerName string) (string, error) {
        cmd := exec.Command("docker", "inspect", "-f", "{{.NetworkSettings.IPAddress}}", containerName)
        output, err := cmd.Output()
        if err != nil {
            return "", fmt.Errorf("failed to get IP for %s: %w", containerName, err)
        }
        return strings.TrimSpace(string(output)), nil
    }

    // Function to configure a device
    func configureDevice(nodeName, nodeIP, configFilePath string, wg *sync.WaitGroup, errorsChan chan error) {
        defer wg.Done()
        fmt.Printf("Configuring %s (%s)...\n", nodeName, nodeIP)

        // Generate .eapi.conf dynamically (or use env vars/direct connect)
        // For this tutorial, assume .eapi.conf is pre-configured or use dummy values if needed
        // A better approach would be to pass connection details directly to goeapi if possible
        // or generate a temporary .eapi.conf
        // For simplicity, let's assume we can directly connect using username/password
        // (This might require modifying goeapi or directly using HTTP client for eAPI)

        // Let's use a simplified direct connection approach if goeapi supports it, otherwise a mock
        // For now, let's just log and mock success/failure
        // In a real scenario, you'd configure the goeapi.Connect method to use the IP and credentials

        // MOCKING goeapi.Connect and Configure
        // In a real application, replace this with actual goeapi calls:
        // node, err := goeapi.Connect(nodeName) // Or create a direct connection
        // if err != nil { errorsChan <- fmt.Errorf("failed to connect to %s: %w", nodeName, err); return }

        configData, err := ioutil.ReadFile(configFilePath)
        if err != nil {
            errorsChan <- fmt.Errorf("failed to read config file %s for %s: %w", configFilePath, nodeName, err)
            return
        }

        var deviceCfg DeviceConfig
        err = json.Unmarshal(configData, &deviceCfg)
        if err != nil {
            errorsChan <- fmt.Errorf("failed to parse JSON config for %s: %w", nodeName, err)
            return
        }

        // Generate commands from parsed config
        var configCommands []string
        configCommands = append(configCommands, fmt.Sprintf("hostname %s", deviceCfg.Hostname))
        configCommands = append(configCommands, fmt.Sprintf("interface Loopback0"))
        configCommands = append(configCommands, fmt.Sprintf("ip address %s", deviceCfg.LoopbackIP))
        configCommands = append(configCommands, fmt.Sprintf("description \"Configured by automation script\""))
        configCommands = append(configCommands, "no shutdown") // Ensure loopback is up

        for _, iface := range deviceCfg.Interfaces {
            configCommands = append(configCommands, fmt.Sprintf("interface %s", iface.Name))
            configCommands = append(configCommands, fmt.Sprintf("ip address %s", iface.IP))
            configCommands = append(configCommands, fmt.Sprintf("description \"%s\"", iface.Description))
            configCommands = append(configCommands, "no switchport")
            configCommands = append(configCommands, "no shutdown")
        }

        // Simulate sending commands via eAPI
        fmt.Printf("Sending %d configuration commands to %s...\n", len(configCommands), nodeName)
        time.Sleep(2 * time.Second) // Simulate network/device processing time

        // In a real scenario:
        // _, err = node.Configure(configCommands)
        // if err != nil { errorsChan <- fmt.Errorf("failed to configure %s: %w", nodeName, err); return }

        fmt.Printf("Configuration for %s successful.\n", nodeName)

        // Verify configuration
        // In a real scenario:
        // showCmds := []string{"show running-config interface Loopback0", "show ip interface brief"}
        // showResp, err := node.RunCommands(showCmds)
        // if err != nil { errorsChan <- fmt.Errorf("failed to verify config on %s: %w", nodeName, err); return }
        // fmt.Printf("Verification output for %s: %+v\n", nodeName, showResp)

        fmt.Printf("Verification for %s successful (mock).\n", nodeName)
    }

    func main() {
        topoFile := "automation_lab.clab.yaml"
        defer func() {
            if r := recover(); r != nil {
                fmt.Printf("Recovered from panic: %v\n", r)
            }
            // Uncomment to automatically destroy the lab after run
            // if err := destroyLab(topoFile); err != nil {
            //     fmt.Printf("Error destroying lab: %v\n", err)
            // }
        }()

        // 1. Deploy the Containerlab topology
        if err := deployLab(topoFile); err != nil {
            fmt.Printf("Error deploying lab: %v\n", err)
            os.Exit(1)
        }

        // Give devices time to boot up and services to start
        fmt.Println("Waiting 30 seconds for devices to boot up...")
        time.Sleep(30 * time.Second)

        // 2. Prepare device configurations and get IPs
        deviceNodes := map[string]string{
            "clab-automation-lab-ceos1": "config_ceos1.json",
            "clab-automation-lab-ceos2": "config_ceos2.json",
        }

        var wg sync.WaitGroup
        errorsChan := make(chan error, len(deviceNodes))

        for containerName, configFile := range deviceNodes {
            ip, err := getContainerIP(containerName)
            if err != nil {
                fmt.Printf("Could not get IP for %s: %v. Skipping configuration.\n", containerName, err)
                continue
            }
            fmt.Printf("Found IP for %s: %s\n", containerName, ip)

            wg.Add(1)
            go configureDevice(containerName, ip, configFile, &wg, errorsChan)
        }

        wg.Wait()
        close(errorsChan)

        hasErrors := false
        for err := range errorsChan {
            fmt.Printf("Automation Error: %v\n", err)
            hasErrors = true
        }

        if hasErrors {
            fmt.Println("\nAutomation completed with errors.")
        } else {
            fmt.Println("\nAutomation completed successfully!")
        }
    }

    ```

Helper files for automation_lab.go 
automation_lab.clab.yaml
```yaml
name: automation-lab
topology:
  nodes:
    ceos1:
      kind: ceos
      image: arista/ceos:<your_ceos_version>
      startup-config: |
        no aaa root
        username admin privilege 15 role network-admin secret sha512 $6$RxQ5ae0GOW6SAiCU$7qzQNGX2pSIWBYGF8Xh30lo/s418/diYEEZj9rPrTJiAkYv0s6AvjpTfUHMGz.a58Hg29Yy/nV0Zvplux0
        management api http-commands
          no shutdown
          protocol http
    ceos2:
      kind: ceos
      image: arista/ceos:<your_ceos_version>
      startup-config: |
        no aaa root
        username admin privilege 15 role network-admin secret sha512 $6$RxQ5ae0GOW6SAiCU$7qzQNGX2pSIWBYGF8Xh30lo/s418/diYEEZj9rPrTJiAkYv0s6AvjpTfUHMGz.a58Hg29Yy/nV0Zvplux0
        management api http-commands
          no shutdown
          protocol http
  links:
    - endpoints: ["ceos1:eth1", "ceos2:eth1"]
```

config_ceos1.json:
```json
{
    "hostname": "leaf1",
    "loopback_ip": "1.1.1.1/32",
    "interfaces": [
        {
            "name": "Ethernet1",
            "ip": "10.0.0.1/30",
            "description": "Link to ceos2"
        }
    ]
}
```

config_ceos2.json:
```json
  {
      "hostname": "leaf2",
      "loopback_ip": "2.2.2.2/32",
      "interfaces": [
          {
              "name": "Ethernet1",
              "ip": "10.0.0.2/30",
              "description": "Link to ceos1"
          }
      ]
  }
```

  * **Challenge 20:** Implement the full automation workflow described above. Ensure your `automation_lab.clab.yaml` and device JSON configuration files are correctly set up. Run the `automate_lab.go` program and verify that the cEOS routers are deployed and configured as expected. Add a final verification step to your Go program that pings `Loopback0` of `ceos2` from `ceos1` (you'll need to figure out how to send CLI commands via eAPI for ping, or run `docker exec` for ping).

-----

**Important Notes:**

  * **Error Handling:** In real-world applications, error handling should be much more robust, including retry mechanisms, specific error type checking, and detailed logging.
  * **Security:** For production environments, avoid hardcoding credentials. Use environment variables, secure configuration files, or secrets management tools. Always use HTTPS for eAPI communication.
  * **Containerlab IPs:** Containerlab assigns IP addresses to cEOS nodes on the `docker0` bridge (or your custom Docker network). You'll need to use `docker inspect <container_name>` to find these IPs for `~/.eapi.conf` or direct connections.
  * **`goeapi` library:** The `goeapi` library might abstract some lower-level eAPI details. For advanced scenarios, you might interact directly with eAPI via `net/http` and `encoding/json` if `goeapi` doesn't provide the necessary functionality.
  * **Cleanup:** Always remember to destroy your Containerlab labs after use to free up resources (`sudo containerlab destroy -t <your_topology_file.yaml>`).
