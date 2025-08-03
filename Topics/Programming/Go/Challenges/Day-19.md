# **Day 19: Building a Network Discovery and Reporting Tool**

## **Introduction:** 
Combining previous concepts to build a more complex tool. Discovering information from devices and generating reports.

## **Code Example: Device Inventory Script**

```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
    "sync"
    "time"

    "github.com/aristanetworks/goeapi"
)

type DeviceInventory struct {
    Hostname     string `json:"hostname"`
    SoftwareVersion string `json:"software_version"`
    SystemMAC    string `json:"system_mac"`
    ManagementIP string `json:"management_ip"`
    Status       string `json:"status"`
    Error        string `json:"error,omitempty"`
}

func collectDeviceInfo(nodeName string, nodeIP string, wg *sync.WaitGroup, results chan DeviceInventory) {
    defer wg.Done()

    inventory := DeviceInventory{
        Hostname:     nodeName,
        ManagementIP: nodeIP,
        Status:       "Failed",
    }

    // Temporarily override .eapi.conf logic for direct IP connection for simplicity in this example
    // In a real tool, you might dynamically generate .eapi.conf or use goeapi's direct connection options if available
    // For this example, let's assume we establish a direct connection or mock it
    // A more robust solution would involve dynamically creating a connection profile or using a custom transport in goeapi

    // Simulating eAPI connection and data retrieval
    // In a real scenario, you'd connect using goeapi.Connect() and run commands
    // For now, let's just mock some success/failure
    if nodeName == "ceos-unreachable" {
        inventory.Error = "Device unreachable"
        results <- inventory
        return
    }

    fmt.Printf("Collecting info from %s (%s)...\n", nodeName, nodeIP)
    time.Sleep(3 * time.Second) // Simulate API call delay

    // Mocking successful data retrieval
    inventory.Status = "Success"
    inventory.SoftwareVersion = "4.30.6M"
    inventory.SystemMAC = "00:11:22:33:44:55"

    results <- inventory
}

func main() {
    // Assume a Containerlab lab is deployed with these nodes
    // Make sure your .eapi.conf has entries for these
    nodes := map[string]string{
        "ceos1": "172.17.0.2", // Replace with actual container IP or hostname
        "ceos2": "172.17.0.3",
        "ceos-unreachable": "172.17.0.4", // Simulate an unreachable device
    }

    var wg sync.WaitGroup
    results := make(chan DeviceInventory, len(nodes))

    for name, ip := range nodes {
        wg.Add(1)
        go collectDeviceInfo(name, ip, &wg, results)
    }

    wg.Wait()
    close(results)

    var inventoryReport []DeviceInventory
    for inv := range results {
        inventoryReport = append(inventoryReport, inv)
    }

    jsonReport, err := json.MarshalIndent(inventoryReport, "", "  ")
    if err != nil {
        fmt.Printf("Error marshalling inventory report: %v\n", err)
        os.Exit(1)
    }

    fmt.Println("\n--- Network Inventory Report ---")
    fmt.Println(string(jsonReport))
}
```

## **Challenge 19:** 
Deploy a Containerlab topology with three cEOS nodes. For each node, use `goeapi` (not mocked data) to retrieve its `show version` and `show ip interface brief` output. Combine this information into a `DeviceInventory` struct for each device. Generate a single JSON report containing the inventory for all devices.
