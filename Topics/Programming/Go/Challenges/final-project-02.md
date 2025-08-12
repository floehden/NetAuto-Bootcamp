# ** Final Project 02: Building a Simple Automation Tool & Next Steps**

##  **Concept:** 
Integrating all learned concepts into a practical, command-line automation tool. Discussing packaging Go applications, error handling best practices, and continuous learning.

## **Challenge:** 
Create a Go application that takes arguments (e.g., node name, command, protocol) and performs the requested operation using the `networkclient` package. Package it into a single executable.

## **Code Example (Conceptual):**
```go
// main.go
package main

import (
    "flag"
    "fmt"
    "log"
    "final/networkclient" // Assuming final is the module from previous task
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
### **Instructions:**
1.  Ensure your `srl_2_nodes.clab.yaml` is deployed.
2.  Ensure your `final/networkclient` package is fully implemented.
3.  Save the code as `main.go`.
4.  Compile the tool: `go build -o network-tool main.go`.
5.  Run it: `./network-tool -node srl1 -action get-hostname -protocol gnmi`.
6.  Run it: `./network-tool -node srl1 -action set-hostname -value "new-test-host" -protocol gnmi`.
7.  Verify: `./network-tool -node srl1 -action get-hostname -protocol gnmi`.


**Important Notes:**

  * **Error Handling:** The code examples provide basic error handling. In production, you would implement more robust error handling, logging, and retry mechanisms.
  * **Authentication:** Using `ssh.InsecureIgnoreHostKey()` and hardcoded passwords (`NokiaSrl1!`) is for lab environments only. For production, implement proper SSH key-based authentication, environment variables, or secure credential management. For gNMI, proper TLS certificates should be used.
  * **YANG Paths:** Be precise with YANG paths for gNMI and NETCONF. Refer to the Nokia SR Linux YANG documentation for exact paths. You can also use `sr_cli tools dump model` on SR Linux to see the loaded YANG modules and their structure.
  * **Containerlab Management IPs:** Containerlab typically sets up a Docker bridge network where nodes are resolvable by their Containerlab-assigned hostnames (`clab-<lab_name>-<node_name>`). Use these hostnames for connecting from your Go programs running on the host.
  * **Idempotency:** When performing configuration changes, aim for idempotent operations (applying the same config multiple times yields the same result without error).
  * **Community and Resources:** Actively engage with the Go and network automation communities. Refer to official Go documentation, Nokia SR Linux documentation, and OpenConfig specifications.
