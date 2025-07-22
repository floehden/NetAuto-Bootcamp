# **Day 17: gNMI with Go - Streaming Telemetry**

## **Concept:** 
Subscribing to streaming telemetry data from SR Linux using gNMI `<Subscribe>` operations. Understanding different subscription modes (ONCE, TARGET\_DEFINED, STREAM).

## **Challenge:** 
Subscribe to interface state changes on `srl1` (e.g., `oper-state`) and print updates as they occur.

## **Code Example:**
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

### **Instructions:**
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

