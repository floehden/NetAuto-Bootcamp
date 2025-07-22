# **Day 11: Introduction to Arista eAPI and GoeAPI Library**

## **Introduction:** 
eAPI as a JSON-RPC based API. Understanding the request and response structure. Introducing `goeapi`, the official Go client library for Arista eAPI.

## **Installation:** 
`go get github.com/aristanetworks/goeapi`

## **Code Example: Basic `goeapi` `show version`**

```go
// show_version.go
package main

import (
    "fmt"
    "os"

    "github.com/aristanetworks/goeapi"
)

func main() {
    // You'll need to create a .eapi.conf file in your home directory
    // Example ~/.eapi.conf:
    // [connection:ceos1]
    // host=<ceos1_container_ip>
    // username=admin
    // password=admin
    // transport=http # or https

    node, err := goeapi.Connect("ceos1") // "ceos1" refers to the connection name in .eapi.conf
    if err != nil {
        fmt.Printf("Error connecting to node: %v\n", err)
        os.Exit(1)
    }

    cmds := []string{"show version"}
    response, err := node.RunCommands(cmds)
    if err != nil {
        fmt.Printf("Error running commands: %v\n", err)
        os.Exit(1)
    }

    // The response is a slice of maps (interface{} for dynamic JSON)
    // You'll typically unmarshal this into a specific struct for better handling
    fmt.Printf("Show Version Response: %+v\n", response)

    // For now, let's just print the raw response. Later we'll parse it.
}
```

## **Challenge 11:** 
Deploy a Containerlab topology with one cEOS device with eAPI enabled. Create the `~/.eapi.conf` file with the correct IP and credentials. Run the `show_version.go` example and observe the raw JSON output.

