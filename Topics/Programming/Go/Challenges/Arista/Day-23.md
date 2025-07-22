# **Day 23: gNMI `Set` RPC for Configuration**

## **Introduction:** 
Using the `Set` RPC to configure devices. Understanding `Update`, `Replace`, and `Delete` operations. Building `SetRequest` messages.

## **Code Example: Configuring a Loopback Interface via gNMI**

```go
// gnmi_set_loopback.go
package main

import (
    "context"
    "fmt"
    "log"
    "time"

    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"
    gpb "github.com/openconfig/gnmi/proto/gnmi"
    "google.golang.org/protobuf/proto"
    "google.golang.org/protobuf/encoding/prototext" // For debugging Protobuf messages
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

    // Define the gNMI path for the configuration
    // This configures Loopback1 with IP 192.168.100.1/32
    // Data format is typically JSON_IETF
    configPath := &gpb.Path{
        Elem: []*gpb.PathElem{
            {Name: "interfaces"},
            {Name: "interface", Key: map[string]string{"name": "Loopback1"}},
        },
    }

    // Construct the configuration data as JSON_IETF bytes
    // This JSON corresponds to the OpenConfig 'interface' model.
    // Ensure you match the schema expected by the device.
    configJSON := `
    {
        "name": "Loopback1",
        "config": {
            "name": "Loopback1",
            "type": "OTHER",
            "enabled": true
        },
        "subinterfaces": {
            "subinterface": [
                {
                    "index": 0,
                    "config": {
                        "index": 0
                    },
                    "ipv4": {
                        "addresses": {
                            "address": [
                                {
                                    "ip": "192.168.100.1",
                                    "config": {
                                        "ip": "192.168.100.1",
                                        "prefix-length": 32
                                    }
                                }
                            ]
                        }
                    }
                }
            ]
        }
    }`

    // Create a TypedValue for the JSON configuration
    val := &gpb.TypedValue{
        Value: &gpb.TypedValue_JsonIetfVal{
            JsonIetfVal: []byte(configJSON),
        },
    }

    // Create the Update notification
    update := &gpb.Update{
        Path: configPath,
        Val:  val,
    }

    // Create the SetRequest with the Update operation
    req := &gpb.SetRequest{
        Update: []*gpb.Update{update}, // Use Update to merge/add config
        // You could use Replace to overwrite, or Delete to remove
    }

    fmt.Println("Sending SetRequest to configure Loopback1...")
    // For debugging the request message itself:
    // fmt.Println("SetRequest:\n", prototext.Format(req))

    resp, err := client.Set(ctx, req)
    if err != nil {
        log.Fatalf("Failed to set gNMI data: %v", err)
    }

    fmt.Println("SetRequest successful. Response:")
    for _, result := range resp.GetResults() {
        fmt.Printf("  Path: %s, Op: %s\n", result.GetPath(), result.GetOp())
    }

    // Verify configuration using eAPI or another gNMI Get
    // (Not shown here, but crucial in a real scenario)
}
```

## **Challenge 23:**
1.  Deploy a fresh cEOS lab instance.
2.  Write a Go program using gNMI `Set` to configure an `Ethernet2` interface with an IP address (e.g., `10.20.0.1/24`) and enable it.
3.  After the `Set` operation, use `goeapi` (from Module 3) or a gNMI `Get` request (from Day 22) to verify that the `Ethernet2` interface has been configured correctly on the cEOS device.
