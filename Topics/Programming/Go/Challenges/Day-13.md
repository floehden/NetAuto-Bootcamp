# **Day 13: Sending Configuration Commands via eAPI**

## **Introduction:** 
Using `goeapi` to send configuration commands to cEOS. The `Configure` method. Understanding the `config` context in EOS.

## **Code Example: Configuring a Loopback Interface**

```go
package main

// configure_loopback.go

import (
	"fmt"
	"os"

	"github.com/aristanetworks/goeapi"
)

func main() {
	node, err := goeapi.ConnectTo("ceos1")
	if err != nil {
		fmt.Printf("Error connecting to node: %v\n", err)
		os.Exit(1)
	}

	configCmds := []string{
		"enable",
		"configure",
		"interface Loopback0",
		"ip address 1.1.1.1/32",
		"description \"Configured by GoLang\"",
	}

	response, err := node.RunCommands(configCmds, "text")
	if err != nil {
		fmt.Printf("Error configuring device: %v\n", err)
		os.Exit(1)
	}

	fmt.Printf("Configuration successful: %+v\n", response)
}
```

For verification go the the router via the Cli
```bash
docker exec -it clab-ceos-configured-lab-ceos1 Cli
ceos1>en
ceos1#show running-config interface Loopback0
interface Loopback0
   description "Configured by GoLang"
   ip address 1.1.1.1/32
ceos1#exit
```

## **Challenge 13:** 
Deploy a fresh cEOS lab. Write a Go program using `goeapi` to configure `Ethernet1` with an IP address (e.g., `10.10.10.1/24`) and a description. After configuration, use `goeapi` to retrieve and print the `show ip interface brief` output to confirm.

