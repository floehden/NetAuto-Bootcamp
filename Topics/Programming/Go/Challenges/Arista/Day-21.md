# **Day 21: Introduction to gNMI and Basic Connection**

## **Introduction:** 
What is gNMI? Why use it over eAPI? (Streaming Telemetry, Declarative Configuration, Protobuf/gRPC, schema-driven). gNMI RPC types: `Capabilities`, `Get`, `Set`, `Subscribe`. </br>
</br>
**Enabling gNMI on cEOS:** Review the `startup-config` change required.

## **Code Example: Connecting to a gNMI Server**

```go
// gnmi_connect.go
package main

import (
    "context"
    "fmt"
    "log"
    "time"

    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"
    gpb "github.com/openconfig/gnmi/proto/gnmi" // Alias to avoid conflicts
)

const (
    targetAddress = "10.0.0.10:6030" // Replace with your cEOS container IP and gNMI port
    username      = "admin"
    password      = "admin"
)

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()

    // Set up gRPC dial options
    opts := []grpc.DialOption{
        grpc.WithTransportCredentials(insecure.NewCredentials()), // Use insecure for lab environment
        grpc.WithBlock(), // Block until the connection is established
    }

    fmt.Printf("Attempting to connect to gNMI server at %s...\n", targetAddress)
    conn, err := grpc.DialContext(ctx, targetAddress, opts...)
    if err != nil {
        log.Fatalf("Failed to dial gNMI server: %v", err)
    }
    defer conn.Close()
    fmt.Println("Successfully connected to gNMI server.")

    // Create a gNMI client
    client := gpb.NewgNMIClient(conn)

    // Perform a simple Capabilities request to verify connection
    fmt.Println("Requesting gNMI capabilities...")
    capResp, err := client.Capabilities(ctx, &gpb.CapabilityRequest{})
    if err != nil {
        log.Fatalf("Failed to get gNMI capabilities: %v", err)
    }

    fmt.Println("gNMI Capabilities:")
    fmt.Printf("  Supported Models: %+v\n", capResp.GetSupportedModels())
    fmt.Printf("  Supported Versions: %+v\n", capResp.GetSupportedGnmiVersions())
    fmt.Printf("  Supported Encodings: %+v\n", capResp.GetSupportedEncodings())
}
```

## **Challenge 21:**

1.  Update your `automation_lab.clab.yaml` (from Day 20) to enable gNMI on both `ceos1` and `ceos2`.
2.  Deploy the lab.
3.  Find the IP address of `clab-automation-lab-ceos1`.
4.  Update `gnmi_connect.go` with the correct IP and run the program. Verify that it connects and prints capabilities.
