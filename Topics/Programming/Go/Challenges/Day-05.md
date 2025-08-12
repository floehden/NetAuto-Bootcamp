# **Day 5: Slices, Maps, and Structs**

## **Introduction:** 
Dynamic arrays (slices), key-value pairs (maps), and custom data structures (structs). How to iterate over them.

## **Code Example: Network Inventory**

```go
// inventory.go
package main

import "fmt"

type Device struct {
    Hostname    string
    IPAddress   string
    Location    string
    Model       string
    Interfaces  []string
}

func main() {
    // Slice of strings
    var vlans []string
    vlans = append(vlans, "VLAN10", "VLAN20", "VLAN30")
    fmt.Println("VLANs:", vlans)

    // Map of device IPs to hostnames
    deviceMap := make(map[string]string)
    deviceMap["10.0.0.1"] = "leaf1"
    deviceMap["10.0.0.2"] = "leaf2"
    fmt.Println("Device Map:", deviceMap)

    // Struct for a network device
    router := Device{
        Hostname:   "core-router-1",
        IPAddress:  "172.16.0.1",
        Location:   "Data Center A",
        Model:      "Arista 7050S",
        Interfaces: []string{"Ethernet1", "Ethernet2", "Loopback0"},
    }
    fmt.Printf("Router Hostname: %s, Model: %s\n", router.Hostname, router.Model)
    fmt.Println("Router Interfaces:")
    for _, iface := range router.Interfaces {
        fmt.Println("- ", iface)
    }
}
```

## **Challenge 5:** 
Define a `NetworkSegment` struct with fields like `Name`, `VLANID`, and a slice of `Device` structs that belong to this segment. Populate an array (slice) of `NetworkSegment`s and print out all devices within each segment.

