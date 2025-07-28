**Day 25: Advanced gNMI Use Cases & Best Practices**

  * **Introduction:** Authentication with gNMI (TLS, username/password metadata). Error handling for gRPC streams. Using libraries like `ygot` for robust path and data modeling. Combining gNMI with eAPI for hybrid automation.

  * **Code Example: Authentication and Generic Path Building**

    ```go
    // gnmi_advanced.go
    package main

    import (
        "context"
        "fmt"
        "log"
        "time"

        "google.golang.org/grpc"
        "google.golang.org/grpc/credentials/insecure"
        "google.golang.org/grpc/metadata" // For authentication metadata
        gpb "github.com/openconfig/gnmi/proto/gnmi"
        "github.com/openconfig/ygot/ygot" // For building gNMI paths more easily
    )

    const (
        targetAddress = "10.0.0.10:6030" // Replace with your cEOS container IP
        username      = "admin"
        password      = "admin"
    )

    func main() {
        ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
        defer cancel()

        // Add username and password to context metadata for authentication
        ctx = metadata.AppendToOutgoingContext(ctx, "username", username, "password", password)

        opts := []grpc.DialOption{
            grpc.WithTransportCredentials(insecure.NewCredentials()), // Use insecure for lab
            grpc.WithBlock(),
        }

        fmt.Printf("Attempting to connect to gNMI server at %s with credentials...\n", targetAddress)
        conn, err := grpc.DialContext(ctx, targetAddress, opts...)
        if err != nil {
            log.Fatalf("Failed to dial gNMI server: %v", err)
        }
        defer conn.Close()
        fmt.Println("Successfully connected to gNMI server.")

        client := gpb.NewgNMIClient(conn)

        // Example: Get hostname using ygot path building
        // You'll need the OpenConfig paths for this to work correctly
        // Arista EOS typically supports OpenConfig models
        hostnamePath, err := ygot.StringToPath("System/Hostname", ygot.New)))
        if err != nil {
            log.Fatalf("Error building path: %v", err)
        }

        req := &gpb.GetRequest{
            Path: []*gpb.Path{hostnamePath},
            Type: gpb.GetRequest_STATE,
        }

        fmt.Println("Getting system hostname...")
        resp, err := client.Get(ctx, req)
        if err != nil {
            log.Fatalf("Failed to get hostname via gNMI: %v", err)
        }

        if resp != nil && len(resp.GetNotification()) > 0 {
            for _, notif := range resp.GetNotification() {
                for _, update := range notif.GetUpdate() {
                    if strVal := update.GetVal().GetStringVal(); strVal != "" {
                        fmt.Printf("Hostname: %s\n", strVal)
                    }
                }
            }
        } else {
            fmt.Println("No hostname found.")
        }

        // --- Example: Using gNMI for a 'health check' via Get multiple paths ---
        // Get CPU utilization (OpenConfig path)
        cpuPath, err := ygot.StringToPath("System/Cpu/Utilization/Avergae", ygot.NewNode()) // Simplified, actual path might differ
        if err != nil {
            log.Fatalf("Error building CPU path: %v", err)
        }

        healthReq := &gpb.GetRequest{
            Path: []*gpb.Path{hostnamePath, cpuPath}, // Request multiple paths
            Type: gpb.GetRequest_STATE,
        }

        fmt.Println("Performing health check (hostname and CPU)...")
        healthResp, err := client.Get(ctx, healthReq)
        if err != nil {
            log.Fatalf("Failed health check via gNMI: %v", err)
        }

        if healthResp != nil && len(healthResp.GetNotification()) > 0 {
            fmt.Println("Health Check Results:")
            for _, notif := range healthResp.GetNotification() {
                for _, update := range notif.GetUpdate() {
                    fmt.Printf("  Path: %s, Value: %+v\n", ygot.PathToString(update.GetPath()), update.GetVal())
                }
            }
        }
    }
    ```

## **Challenge 25: Full Network Health Check and Configuration Rollback**

1.  **Comprehensive Health Check:** Extend the `gnmi_advanced.go` program.
        * Connect to one cEOS node using gNMI with username/password authentication.
        * Use `Get` RPCs to retrieve the following operational state:
            * Hostname
            * CPU Utilization (e.g., `/system/cpu/utilization/state/average` or similar path supported by cEOS)
            * Memory Utilization (e.g., `/system/memory/state/usage` or similar)
            * Status of a specific interface (e.g., `Ethernet1` - `admin-status`, `oper-status`, `last-change`).
        * Print a concise health report for the device.
2.  **Configuration and Rollback (Conceptual/Simulated):**
        * After the health check, use a `Set` RPC to change the description of `Loopback0` to "Configured by Go GNMI Test".
        * Immediately follow this with another `Set` RPC using a `Delete` operation on the Loopback0 interface to effectively remove its configuration.
        * **Self-reflection:** Discuss (mentally or in comments) how you would implement a robust *actual* rollback mechanism (e.g., saving config before changes, committing, and reverting if issues occur). Note that gNMI Set is powerful but doesn't inherently have transactional capabilities like a full NMS.
