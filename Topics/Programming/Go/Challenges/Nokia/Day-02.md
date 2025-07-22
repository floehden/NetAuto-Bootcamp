# **Day 2: Variables, Data Types, and Control Flow**

## **Concept:** 
Understanding Go's basic data types (strings, integers, booleans), declaring variables, and fundamental control flow statements (if/else, for loops).
## **Challenge:**
 Write a Go program that takes an IP address as a string, checks if it's a valid IPv4 address (simple check, e.g., using `strings.Contains` for dots), and prints whether it's valid or not.
## **Code Example:**

```go
// main.go
package main

import (
    "fmt"
    "strings"
)

func main() {
    ipAddress := "192.168.1.1"
    if isValidIPv4(ipAddress) {
        fmt.Printf("%s is a valid IPv4 address (basic check).\n", ipAddress)
    } else {
        fmt.Printf("%s is not a valid IPv4 address (basic check).\n", ipAddress)
    }

    ipAddress = "2001:db8::1"
    if isValidIPv4(ipAddress) {
        fmt.Printf("%s is a valid IPv4 address (basic check).\n", ipAddress)
    } else {
        fmt.Printf("%s is not a valid IPv4 address (basic check).\n", ipAddress)
    }
}

func isValidIPv4(ip string) bool {
    // A very basic check: does it contain three dots?
    // In a real-world scenario, you'd use net.ParseIP
    return strings.Count(ip, ".") == 3
}
```

