
# **Day 22: gNMI `Get` RPC for Operational State**

## **Introduction:** 
Using the `Get` RPC to fetch specific operational state or configuration. Understanding gNMI Paths. Retrieving interface status.

## **Code Example: Get Interface State**

```go
// gnmi_get_interface.go
package main

import (
    "context"
    "fmt"
    "log"
    "time"

    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"
    gpb "github.com/openconfig/gnmi/proto/gnmi"
    "google.golang.org/protobuf/proto" // For unmarshalling typed data
)

const (
    targetAddress = "10.0.0.10:6030" // Replace with your cEOS container IP
    username      = "admin"
    password      = "admin"
)

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()

    opts := []grpc.DialOption{
        grpc.WithTransportCredentials(insecure.NewCredentials()),
        grpc.WithBlock(),
    }

    conn, err := grpc.DialContext(ctx, targetAddress, opts...)
    if err != nil {
        log.Fatalf("Failed to dial gNMI server: %v", err)
    }
    defer conn.Close()
    client := gpb.NewgNMIClient(conn)

    // Define the gNMI path for interface state
    // Paths are relative to a root. For OpenConfig, usually "/interfaces/interface[name=<iface_name>]/state"
    // Arista EOS supports OpenConfig paths.
    // Let's get the state for Ethernet1
    path := &gpb.Path{
        Elem: []*gpb.PathElem{
            {Name: "interfaces"},
            {Name: "interface", Key: map[string]string{"name": "Ethernet1"}},
            {Name: "state"},
        },
    }

    req := &gpb.GetRequest{
        Path: []*gpb.Path{path},
        Type: gpb.GetRequest_STATE, // Request operational state
    }

    fmt.Printf("Getting state for Ethernet1...\n")
    resp, err := client.Get(ctx, req)
    if err != nil {
        log.Fatalf("Failed to get gNMI data: %v", err)
    }

    // Process the response
    if resp != nil && len(resp.GetNotification()) > 0 {
        for _, notif := range resp.GetNotification() {
            for _, update := range notif.GetUpdate() {
                fmt.Printf("Path: %s\n", update.GetPath())
                fmt.Printf("Val: %s\n", update.GetVal().GetStringVal()) // Value might be JSON_IETF or others
                // For complex data, you'd need to unmarshal the TypedValue
                // For example, if it's JSON_IETF:
                if jsonVal := update.GetVal().GetJsonIetfVal(); jsonVal != nil {
                    fmt.Println("JSON Value:", string(jsonVal))
                    // You'd typically unmarshal this JSON into a Go struct
                } else if stringVal := update.GetVal().GetStringVal(); stringVal != "" {
                    fmt.Println("String Value:", stringVal)
                }
            }
        }
    } else {
        fmt.Println("No data received for Ethernet1 state.")
    }
}
```

## **Challenge 22:**

1.  Ensure your cEOS lab is deployed with `Ethernet1` configured and up (e.g., from Day 7's `initial_config.clab.yaml`).
2.  Modify `gnmi_get_interface.go` to fetch the operational state for `Loopback0`.
3.  Print the `Loopback0` IP address and its operational status (up/down) from the gNMI response. You might need to adjust the gNMI path and how you extract the values based on the exact JSON structure returned by cEOS for `Loopback0` state.

