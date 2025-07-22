## **Day 19: Concurrency with GoRoutines for Automation**

## **Concept:** 
Leveraging Go's powerful concurrency model (goroutines and channels) to perform automation tasks on multiple devices in parallel, significantly speeding up operations.

## **Challenge:** 
Using the generic network client (or direct SSH/gNMI calls), iterate through multiple SR Linux nodes in your Containerlab topology and retrieve their hostnames concurrently.

## **Code Example (Conceptual):**
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

### **Instructions:**
1.  Ensure your `srl_2_nodes.clab.yaml` is deployed.
2.  Ensure your `day18/networkclient` package is present and ideally, its methods are somewhat implemented (even if they just print a message for this exercise).
3.  Save the code as `main.go`.
4.  Run `go run main.go`.