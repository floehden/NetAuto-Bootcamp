# **Day 3: Control Flow (If/Else, For Loops, Switch)**

## **Introduction** 
Conditional statements (`if`, `else if`, `else`), looping constructs (`for` loop variations), `switch` statements for multiple conditions.

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

## **Code Example: Device Status Check**

```go
// status_check.go
package main

import (
	"fmt"
	"time"
)

func main() {
    cpuUsage := 85
    memoryUsage := 60
    interfaceStatus := "up"

    if cpuUsage > 80 {
        fmt.Println("High CPU usage warning!")
    } else if cpuUsage > 50 {
        fmt.Println("Moderate CPU usage.")
    } else {
        fmt.Println("CPU usage normal.")
    }

    if memoryUsage > 75 {
        fmt.Println("High memory usage warning!")
    }

    switch interfaceStatus {
    case "up":
        fmt.Println("Interface is up and running.")
    case "down":
        fmt.Println("Interface is down!")
    default:
        fmt.Println("Unknown interface status.")
    }

    fmt.Println("Counting from 1 to 5:")
    for i := 1; i <= 5; i++ {
        fmt.Println(i)
        time.Sleep(500 * time.Millisecond)
    }
}
```

## **Challenge 3:** 
Write a program that simulates checking interface status for 5 interfaces. If an interface is "down", print an alert message and the interface number. If it's "up", print "Interface X is healthy."
