# **Day 24: gNMI `Subscribe` for Streaming Telemetry**

## **Introduction:** 
The power of gNMI: streaming telemetry. Understanding `ONCE`, `POLL`, and `STREAM` subscriptions. Handling continuous updates.

## **Code Example: Streaming Interface Counters**

```go
// gnmi_subscribe_counters.go
package main

import (
    "context"
    "fmt"
    "io"
    "log"
    "time"

    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"
    gpb "github.com/openconfig/gnmi/proto/gnmi"
)

const (
    targetAddress = "10.0.0.10:6030" // Replace with your cEOS container IP
    username      = "admin"
    password      = "admin"
    intervalSecs  = 10 // Subscription interval in seconds
)

func main() {
    ctx, cancel := context.WithCancel(context.Background())
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

    // Define the gNMI path for interface Ethernet1 counters
    path := &gpb.Path{
        Elem: []*gpb.PathElem{
            {Name: "interfaces"},
            {Name: "interface", Key: map[string]string{"name": "Ethernet1"}},
            {Name: "state"},
            {Name: "counters"},
        },
    }

    // Create the SubscriptionList
    subscribeList := &gpb.SubscriptionList{
        Mode: gpb.SubscriptionList_STREAM, // STREAM for continuous updates
        // QOS:       &gpb.QOSMarking{Marking: 0}, // Optional QoS
        // Use once per X seconds
        Subscription: []*gpb.Subscription{
            {
                Path:              path,
                Mode:              gpb.SubscriptionMode_ON_CHANGE, // Or SAMPLE for fixed intervals
                SampleInterval:    uint64(intervalSecs * time.Second.Nanoseconds()),
                SuppressRedundant: true,
            },
        },
    }

    fmt.Printf("Subscribing to Ethernet1 counters (interval: %d seconds)...\n", intervalSecs)
    stream, err := client.Subscribe(ctx)
    if err != nil {
        log.Fatalf("Failed to create gNMI subscribe stream: %v", err)
    }

    // Send the subscription request
    if err := stream.Send(&gpb.SubscribeRequest{Request: subscribeList}); err != nil {
        log.Fatalf("Failed to send subscribe request: %v", err)
    }

    // Goroutine to receive updates
    go func() {
        for {
            resp, err := stream.Recv()
            if err == io.EOF {
                fmt.Println("Stream closed by server.")
                return
            }
            if err != nil {
                log.Printf("Error receiving from stream: %v", err)
                return
            }

            notification := resp.GetUpdate()
            if notification == nil {
                continue
            }

            fmt.Printf("\n--- gNMI Update Received at %s ---\n", time.Now().Format(time.RFC3339))
            for _, update := range notification.GetUpdate() {
                fmt.Printf("  Path: %s\n", update.GetPath())
                if jsonVal := update.GetVal().GetJsonIetfVal(); jsonVal != nil {
                    fmt.Printf("  Value: %s\n", string(jsonVal))
                } else {
                    fmt.Printf("  Value: %+v\n", update.GetVal())
                }
            }
        }
    }()

    fmt.Println("Press Ctrl+C to stop subscription...")
    // Keep the main goroutine alive
    select {}
}
```

## **Challenge 24:**

1.  Deploy your cEOS lab with gNMI enabled and `Ethernet1` configured.
2.  Run the `gnmi_subscribe_counters.go` program.
3.  While it's running, SSH into `ceos1` and generate some traffic on `Ethernet1` (e.g., `ping 10.0.0.2` if `ceos2` is connected, or send some packets with `sudo tcpdump -i eth1 -w /dev/null`).
4.  Observe the streaming updates in your Go program's output. Specifically, try to identify and print the `in-octets` and `out-octets` values.
