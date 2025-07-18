
## **Module 5: GoLang and gNMI for Streaming Telemetry & Configuration (Days 21-25)**

**Prerequisites for Module 5:**

  * **Arista cEOS with gNMI enabled:** In your Containerlab YAML, you need to enable the gNMI server. Add the following to your cEOS node's `startup-config`:

    ```yaml
    # ... inside ceos node startup-config ...
    gnmi
      no shutdown
      transport grpc
        no shutdown
        port 6030
        vrf management
      !
    ```

    *Note: gNMI typically runs on port 50051, but Arista EOS uses 6030 by default for gRPC. Ensure your firewall/security groups allow access to this port if running on a cloud VM.*

  * **Go gNMI Client Libraries:** You'll need `go get` these:

      * `go get github.com/openconfig/gnmi/proto/gnmi`
      * `go get google.golang.org/grpc`
      * `go get google.golang.org/grpc/credentials/insecure` (for unencrypted connections)
      * `go get google.golang.org/protobuf/proto`
      * `go get github.com/openconfig/ygot/ygot` (useful for path building)
      * `go get github.com/aristanetworks/glog` (used by some Arista examples, though standard log is fine)

**Day 21: Introduction to gNMI and Basic Connection**

  * **Introduction:** What is gNMI? Why use it over eAPI? (Streaming Telemetry, Declarative Configuration, Protobuf/gRPC, schema-driven). gNMI RPC types: `Capabilities`, `Get`, `Set`, `Subscribe`.

  * **Enabling gNMI on cEOS:** Review the `startup-config` change required.

  * **Code Example: Connecting to a gNMI Server**

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

  * **Challenge 21:**

    1.  Update your `automation_lab.clab.yaml` (from Day 20) to enable gNMI on both `ceos1` and `ceos2`.
    2.  Deploy the lab.
    3.  Find the IP address of `clab-automation-lab-ceos1`.
    4.  Update `gnmi_connect.go` with the correct IP and run the program. Verify that it connects and prints capabilities.

**Day 22: gNMI `Get` RPC for Operational State**

  * **Introduction:** Using the `Get` RPC to fetch specific operational state or configuration. Understanding gNMI Paths. Retrieving interface status.

  * **Code Example: Get Interface State**

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

  * **Challenge 22:**

    1.  Ensure your cEOS lab is deployed with `Ethernet1` configured and up (e.g., from Day 7's `initial_config.clab.yaml`).
    2.  Modify `gnmi_get_interface.go` to fetch the operational state for `Loopback0`.
    3.  Print the `Loopback0` IP address and its operational status (up/down) from the gNMI response. You might need to adjust the gNMI path and how you extract the values based on the exact JSON structure returned by cEOS for `Loopback0` state.

**Day 23: gNMI `Set` RPC for Configuration**

  * **Introduction:** Using the `Set` RPC to configure devices. Understanding `Update`, `Replace`, and `Delete` operations. Building `SetRequest` messages.

  * **Code Example: Configuring a Loopback Interface via gNMI**

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

  * **Challenge 23:**

    1.  Deploy a fresh cEOS lab instance.
    2.  Write a Go program using gNMI `Set` to configure an `Ethernet2` interface with an IP address (e.g., `10.20.0.1/24`) and enable it.
    3.  After the `Set` operation, use `goeapi` (from Module 3) or a gNMI `Get` request (from Day 22) to verify that the `Ethernet2` interface has been configured correctly on the cEOS device.

**Day 24: gNMI `Subscribe` for Streaming Telemetry**

  * **Introduction:** The power of gNMI: streaming telemetry. Understanding `ONCE`, `POLL`, and `STREAM` subscriptions. Handling continuous updates.

  * **Code Example: Streaming Interface Counters**

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

  * **Challenge 24:**

    1.  Deploy your cEOS lab with gNMI enabled and `Ethernet1` configured.
    2.  Run the `gnmi_subscribe_counters.go` program.
    3.  While it's running, SSH into `ceos1` and generate some traffic on `Ethernet1` (e.g., `ping 10.0.0.2` if `ceos2` is connected, or send some packets with `sudo tcpdump -i eth1 -w /dev/null`).
    4.  Observe the streaming updates in your Go program's output. Specifically, try to identify and print the `in-octets` and `out-octets` values.

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

  * **Challenge 25: Full Network Health Check and Configuration Rollback**

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

-----

This new gNMI module significantly enhances the tutorial by covering a modern, powerful network management interface. Remember to adapt the IP addresses, usernames, and passwords to your specific Containerlab setup. Enjoy your journey into advanced network automation with Go\!