# **Day 15: gNMI with Go - Capabilities and Get Operations**

## **Concept:** 
Using Go's gRPC library and generated gNMI client code to perform gNMI `<Capabilities>` and `<Get>` operations on SR Linux.

## **Challenge:** 
Connect to `srl1` via gNMI, retrieve its capabilities, and then use a `<Get>` request to fetch the hostname or interface state.

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
### **Instructions:**
1.  Ensure your `srl_2_nodes.clab.yaml` is deployed.
2.  Run `go mod init day15`.
3.  Run `go get google.golang.org/grpc github.com/openconfig/gnmi/proto/gnmi`.
4.  Save the code as `main.go`.
5.  Run `go run main.go`.

