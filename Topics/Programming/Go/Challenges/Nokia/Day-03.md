# **Day 3: Functions, Packages, and Error Handling**

## **Concept:** 
Defining functions, organizing code into packages, and Go's idiomatic error handling using `error` types and `if err != nil` checks.

## **Challenge:** 
Create a `networkutils` package with a function that takes a network interface name (string) and an IP address (string), and returns a formatted string like "Interface \<name\> configured with \<IP\>" or an error if the IP is invalid.

## **Code Example:**
```go
// networkutils/networkutils.go
package networkutils

import (
    "fmt"
    "strings"
)

// ConfigureInterface simulates configuring an interface.
// It returns a success message or an error if the IP is malformed.
func ConfigureInterface(name, ipAddress string) (string, error) {
    if strings.Count(ipAddress, ".") != 3 {
        return "", fmt.Errorf("invalid IPv4 address format for %s: %s", name, ipAddress)
    }
    return fmt.Sprintf("Interface %s configured with %s", name, ipAddress), nil
}

// main.go
package main

import (
    "fmt"
    "log"
    "day3/networkutils" // Assuming your module name is day3
)

func main() {
    msg, err := networkutils.ConfigureInterface("ethernet-1/1", "10.0.0.1/24")
    if err != nil {
        log.Printf("Error configuring interface: %v", err)
    } else {
        fmt.Println(msg)
    }

    msg, err = networkutils.ConfigureInterface("loopback0", "invalid-ip")
    if err != nil {
        log.Printf("Error configuring interface: %v", err)
    } else {
        fmt.Println(msg)
    }
}
```

### **Instructions:**
1.  Create a directory `day3`.
2.  Inside `day3`, create a directory `networkutils`.
3.  Save the first code block as `networkutils/networkutils.go`.
4.  Save the second code block as `main.go` in the `day3` directory.
5.  Run `go mod init day3` in the `day3` directory.
6.  Run `go run main.go`.

