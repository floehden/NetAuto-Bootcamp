
# **Day 4: Functions and Error Handling**

## **Introduction** 
* Defining functions, parameters, return values. 
* Multiple return values (common for error handling).
* Basic error handling using `if err != nil`. The `defer` keyword.

## **Code Example: Ping Function**

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
