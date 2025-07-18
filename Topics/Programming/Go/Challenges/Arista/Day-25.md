# **Day 25: Configuration Backup and Restore with gNMI**

  * **Introduction:** The critical role of configuration backup and restore in network operations. How gNMI's `Get` with `CONFIG` type and `Set` with `REPLACE` operation enable declarative full-configuration management.

  * **Key Concepts:**

      * **Backup:** Fetching the entire running configuration as structured data (JSON\_IETF).
      * **Restore:** Applying a full configuration, effectively overwriting the current state (`REPLACE` operation). This is powerful and must be used with caution.
      * **Root Path (`/`):** When backing up or restoring the *entire* configuration, you typically use a root path (`/`) to specify the whole config tree.

  * **Code Example: Backup and Restore Configuration**

    ```go
    // gnmi_config_b_r.go
    package main

    import (
        "context"
        "fmt"
        "io/ioutil"
        "log"
        "time"

        "google.golang.org/grpc"
        "google.golang.org/grpc/credentials/insecure"
        "google.golang.org/grpc/metadata"
        gpb "github.com/openconfig/gnmi/proto/gnmi"
        "google.golang.org/protobuf/proto" // For potential debug printing of protobuf messages
    )

    const (
        targetAddress  = "10.0.0.10:6030" // Replace with your cEOS container IP
        username       = "admin"
        password       = "admin"
        backupFile     = "ceos_config_backup.json"
        restoreFile    = "ceos_config_to_restore.json" // Could be the same as backupFile
    )

    func getGNMIClient(ctx context.Context, addr, user, pass string) (gpb.gNMIClient, *grpc.ClientConn, error) {
        opts := []grpc.DialOption{
            grpc.WithTransportCredentials(insecure.NewCredentials()),
            grpc.WithBlock(),
        }

        // Add authentication metadata to the context
        authCtx := metadata.AppendToOutgoingContext(ctx, "username", user, "password", pass)

        fmt.Printf("Attempting to connect to gNMI server at %s...\n", addr)
        conn, err := grpc.DialContext(authCtx, addr, opts...)
        if err != nil {
            return nil, nil, fmt.Errorf("failed to dial gNMI server: %w", err)
        }
        fmt.Println("Successfully connected to gNMI server.")
        return gpb.NewgNMIClient(conn), conn, nil
    }

    // backupConfig fetches the full configuration and saves it to a file
    func backupConfig(ctx context.Context, client gpb.gNMIClient, filePath string) error {
        fmt.Printf("\n--- Starting Configuration Backup to %s ---\n", filePath)

        // Request the entire configuration tree from the root path
        configPath := &gpb.Path{
            Elem: []*gpb.PathElem{}, // Empty path represents the root
        }

        req := &gpb.GetRequest{
            Path: []*gpb.Path{configPath},
            Type: gpb.GetRequest_CONFIG, // Crucial: Request configuration data
            Encoding: gpb.Encoding_JSON_IETF, // Request JSON_IETF encoding
        }

        resp, err := client.Get(ctx, req)
        if err != nil {
            return fmt.Errorf("failed to get configuration: %w", err)
        }

        if resp == nil || len(resp.GetNotification()) == 0 {
            return fmt.Errorf("no configuration data received")
        }

        var configBytes []byte
        // Iterate through notifications and updates to find the config data
        for _, notif := range resp.GetNotification() {
            for _, update := range notif.GetUpdate() {
                if update.GetPath().String() == "/" { // Check for the root path update
                    if jsonVal := update.GetVal().GetJsonIetfVal(); jsonVal != nil {
                        configBytes = jsonVal
                        break
                    }
                }
            }
            if configBytes != nil {
                break
            }
        }

        if configBytes == nil {
            return fmt.Errorf("could not find root configuration JSON_IETF in response")
        }

        if err := ioutil.WriteFile(filePath, configBytes, 0644); err != nil {
            return fmt.Errorf("failed to write backup to file: %w", err)
        }

        fmt.Printf("Configuration successfully backed up to %s\n", filePath)
        return nil
    }

    // restoreConfig reads configuration from a file and applies it to the device
    func restoreConfig(ctx context.Context, client gpb.gNMIClient, filePath string) error {
        fmt.Printf("\n--- Starting Configuration Restore from %s ---\n", filePath)

        configData, err := ioutil.ReadFile(filePath)
        if err != nil {
            return fmt.Errorf("failed to read configuration file: %w", err)
        }

        // The path for a full restore (replace) is the root
        configPath := &gpb.Path{
            Elem: []*gpb.PathElem{}, // Empty path for root
        }

        // Create a TypedValue for the JSON configuration
        val := &gpb.TypedValue{
            Value: &gpb.TypedValue_JsonIetfVal{
                JsonIetfVal: configData,
            },
        }

        // Create the SetRequest with a REPLACE operation at the root
        // This will replace the entire configuration of the device with the provided data.
        req := &gpb.SetRequest{
            Replace: []*gpb.Update{
                {
                    Path: configPath,
                    Val:  val,
                },
            },
        }

        fmt.Println("Sending SetRequest (REPLACE operation) to restore configuration...")
        resp, err := client.Set(ctx, req)
        if err != nil {
            return fmt.Errorf("failed to restore configuration: %w", err)
        }

        fmt.Println("Configuration restore successful. Response:")
        for _, result := range resp.GetResults() {
            fmt.Printf("  Path: %s, Op: %s\n", result.GetPath(), result.GetOp())
        }

        fmt.Printf("Configuration restored from %s\n", filePath)
        return nil
    }

    func main() {
        ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second) // Increased timeout
        defer cancel()

        client, conn, err := getGNMIClient(ctx, targetAddress, username, password)
        if err != nil {
            log.Fatalf("Error getting gNMI client: %v", err)
        }
        defer conn.Close()

        // --- Step 1: Backup Current Configuration ---
        if err := backupConfig(ctx, client, backupFile); err != nil {
            log.Fatalf("Backup failed: %v", err)
        }
        fmt.Println("Waiting a moment after backup...")
        time.Sleep(2 * time.Second)

        // --- Step 2: Make a "disruptive" change to the device config ---
        fmt.Println("\n--- Making a disruptive change to the device (e.g., remove Loopback0) ---")
        deletePath := &gpb.Path{
            Elem: []*gpb.PathElem{
                {Name: "interfaces"},
                {Name: "interface", Key: map[string]string{"name": "Loopback0"}},
            },
        }
        deleteReq := &gpb.SetRequest{
            Delete: []*gpb.Path{deletePath},
        }
        _, err = client.Set(ctx, deleteReq)
        if err != nil {
            fmt.Printf("Warning: Failed to delete Loopback0 (may not exist): %v\n", err)
        } else {
            fmt.Println("Loopback0 interface deleted (or attempt made).")
        }
        fmt.Println("Waiting a moment after disruptive change...")
        time.Sleep(2 * time.Second)

        // --- Step 3: Restore Configuration ---
        // For demonstration, we'll restore from the same backup file.
        // In a real scenario, you might have a specific known-good config file.
        if err := restoreConfig(ctx, client, backupFile); err != nil {
            log.Fatalf("Restore failed: %v", err)
        }

        // --- Step 4: Verify Restoration (Optional but Recommended) ---
        fmt.Println("\n--- Verifying Restoration (e.g., check Loopback0 again) ---")
        // You would typically use a gNMI Get or eAPI call here to confirm the state.
        // For this example, let's get the Loopback0 config again to see if it's back.
        loopback0Path := &gpb.Path{
            Elem: []*gpb.PathElem{
                {Name: "interfaces"},
                {Name: "interface", Key: map[string]string{"name": "Loopback0"}},
            },
        }
        verifyReq := &gpb.GetRequest{
            Path: []*gpb.Path{loopback0Path},
            Type: gpb.GetRequest_CONFIG,
            Encoding: gpb.Encoding_JSON_IETF,
        }
        verifyResp, err := client.Get(ctx, verifyReq)
        if err != nil {
            fmt.Printf("Error during verification Get: %v\n", err)
        } else if verifyResp != nil && len(verifyResp.GetNotification()) > 0 {
            fmt.Println("Loopback0 configuration after restore:")
            for _, notif := range verifyResp.GetNotification() {
                for _, update := range notif.GetUpdate() {
                    if jsonVal := update.GetVal().GetJsonIetfVal(); jsonVal != nil {
                        fmt.Println(string(jsonVal))
                    }
                }
            }
        } else {
            fmt.Println("Loopback0 not found after restore. Restoration might have failed or config was empty.")
        }

        fmt.Println("\nAutomation for backup/restore completed.")
    }
    ```

  * **Challenge 25:**

    1.  Ensure your Containerlab with one cEOS node (gNMI enabled) is deployed and has a simple base configuration (e.g., hostname, a Loopback0 interface with an IP).
    2.  Run the `gnmi_config_b_r.go` program.
    3.  Verify that a `ceos_config_backup.json` file is created and contains the full configuration.
    4.  Observe the deletion of `Loopback0` (or the attempt) and then its re-appearance after the restore.
    5.  **Bonus:** Try to modify the `ceos_config_backup.json` file manually (e.g., change the Loopback0 IP address or description), save it as `ceos_config_to_restore.json`, and then run the script using `ceos_config_to_restore.json` for the restore operation. Confirm the new configuration is applied.
