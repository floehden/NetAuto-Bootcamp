# **Day 2: Variables, Data Types, and Operators**

## **Introduction:** 
Declaring variables (`var`, `:=`), common data types (int, string, bool, float), arithmetic, comparison, and logical operators. Type inference.

## **Code Example: Network Device Info**

```go
// device_info.go
package main

import "fmt"

func main() {
    deviceName := "my-arista-router"
    ipAddress := "192.168.1.1"
    ports := 24
    isOnline := true

    fmt.Printf("Device: %s\n", deviceName)
    fmt.Printf("IP Address: %s\n", ipAddress)
    fmt.Printf("Number of Ports: %d\n", ports)
    fmt.Printf("Is Online: %t\n", isOnline)

    // Example operation
    uptimeHours := 120
    days := uptimeHours / 24
    fmt.Printf("Uptime: %d days\n", days)
}
```

## **Challenge 2:** 
Create a program that declares variables for a network segment (e.g., `networkName`, `subnet`, `maskLength`, `numberOfHosts`). Calculate and print the number of usable hosts in that subnet.
