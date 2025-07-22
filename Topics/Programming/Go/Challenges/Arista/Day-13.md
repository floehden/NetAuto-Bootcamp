# **Day 13: Sending Configuration Commands via eAPI**

## **Introduction:** 
Using `goeapi` to send configuration commands to cEOS. The `Configure` method. Understanding the `config` context in EOS.

## **Code Example: Configuring a Loopback Interface**

```go
// configure_loopback.go
package main

import (
    "fmt"
    "os"

    "github.com/aristanetworks/goeapi"
)

func main() {
    node, err := goeapi.Connect("ceos1")
    if err != nil {
        fmt.Printf("Error connecting to node: %v\n", err)
        os.Exit(1)
    }

    configCmds := []string{
        "interface Loopback0",
        "ip address 1.1.1.1/32",
        "description \"Configured by GoLang\"",
    }

    response, err := node.Configure(configCmds)
    if err != nil {
        fmt.Printf("Error configuring device: %v\n", err)
        os.Exit(1)
    }

    fmt.Printf("Configuration successful: %+v\n", response)

    // Verify configuration
    showCmds := []string{"show running-config interface Loopback0"}
    showResponse, err := node.RunCommands(showCmds)
    if err != nil {
        fmt.Printf("Error verifying config: %v\n", err)
        os.Exit(1)
    }
    fmt.Println("\n--- Verified Loopback0 Configuration ---")
    fmt.Println(showResponse[0].(map[string]interface{})["result"].(map[string]interface{})["output"])
}
```

## **Challenge 13:** 
Deploy a fresh cEOS lab. Write a Go program using `goeapi` to configure `Ethernet1` with an IP address (e.g., `10.10.10.1/24`) and a description. After configuration, use `goeapi` to retrieve and print the `show ip interface brief` output to confirm.

