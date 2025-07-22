# **Day 4: Structs, Slices, and Maps**

## **Concept:** 
Defining custom data structures (`structs`), working with dynamic arrays (`slices`), and key-value pairs (`maps`). These are crucial for handling network device data.

## **Challenge:** 
Define a `Router` struct with fields like `Name`, `IPAddress`, and a `map[string]string` for `Interfaces` (interface name to IP). Create a slice of `Router` structs and print their details.

## **Code Example:**
```go
// main.go
package main

import "fmt"

type Router struct {
    Name        string
    IPAddress   string
    Interfaces  map[string]string
}

func main() {
    routers := []Router{
        {
            Name:      "srl1",
            IPAddress: "192.168.0.1",
            Interfaces: map[string]string{
                "ethernet-1/1": "10.0.0.1/24",
                "ethernet-1/2": "10.0.0.5/24",
            },
        },
        {
            Name:      "srl2",
            IPAddress: "192.168.0.2",
            Interfaces: map[string]string{
                "ethernet-1/1": "10.0.0.2/24",
            },
        },
    }

    for _, router := range routers {
        fmt.Printf("Router Name: %s, Management IP: %s\n", router.Name, router.IPAddress)
        fmt.Println("  Interfaces:")
        for iface, ip := range router.Interfaces {
            fmt.Printf("    - %s: %s\n", iface, ip)
        }
    }
}
```

