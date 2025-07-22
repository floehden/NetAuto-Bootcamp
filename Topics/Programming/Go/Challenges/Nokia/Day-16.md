
# **Day 16: gNMI with Go - Set Operations**

## **Concept:** 
Using gNMI `<Set>` requests to modify device configuration. Understanding `UPDATE`, `REPLACE`, and `DELETE` operations.

## **Challenge:** 
Use a gNMI `<Set>` request to change the hostname of `srl1` to something like "my-srl-router". Verify the change.

## **Code Example:**
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

