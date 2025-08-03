# **Day 4: Functions, Packages and Error Handling**

## **Introduction** 
* Defining functions, parameters, return values. 
* Multiple return values (common for error handling).
* Basic error handling using `if err != nil`. The `defer` keyword.

## **Code Example **

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

## **Code Example 2: Ping Function**
```go
// ping.go
package main

import (
    "fmt"
    "errors"
)

func pingDevice(ip string) (string, error) {
    if ip == "192.168.1.100" { // Simulate a device being unreachable
        return "", errors.New("device unreachable")
    }
    return fmt.Sprintf("Ping to %s successful!", ip), nil
}

func main() {
    result, err := pingDevice("192.168.1.1")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Println(result)
    }

    result, err = pingDevice("192.168.1.100")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Println(result)
    }
}
```

## **Challenge 4:** 
Create a function `configureInterface(interfaceName string, ipAddress string, netmask string)` that returns a string representing the configuration command (e.g., "interface Ethernet1\\n ip address 10.0.0.1/24\\n no shutdown") and an error if the IP address or netmask is invalid. Call this function and handle the potential error.
