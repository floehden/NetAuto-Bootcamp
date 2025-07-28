# **Day 14: Advanced eAPI Operations: Multiple Commands & Error Handling**

## **Introduction:** 
Sending multiple commands in a single eAPI call. Robust error checking for each command within a batch.

## **Code Example: Batch Configuration and Error Checking**

```go
// batch_config.go
package main

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
		"configure terminal",
		"interface Loopback1",
		"ip address 2.2.2.2/32",
		"description \"Another Loopback\"",
		"interface Ethernet99", // This interface likely doesn't exist
		"ip address 3.3.3.3/24",
	}

	response, err := node.RunCommands(configCmds, "text")
	if err != nil {
		fmt.Printf("Error during batch configuration: %v\n", err)
		// You might need to inspect the 'response' object if available
		// to see which specific command failed if the underlying eAPI
		// library provides that granularity in the error.
	} else {
		fmt.Printf("Batch configuration results: %+v\n", response)
	}

	// For more granular error checking, you might need to send commands one by one
	// or check the "errors" field in the raw JSON response if the API returns it.
	// goeapi often abstracts this, returning an error for the whole batch on first failure.
}
```

## **Challenge 14:** 
Create a Go program that attempts to configure two loopback interfaces (e.g., Loopback0 and Loopback1) and then tries to configure a non-existent interface (e.g., Loopback99). Observe how `goeapi` handles the error and what is returned. Try to implement a mechanism to identify which specific command caused the failure.

