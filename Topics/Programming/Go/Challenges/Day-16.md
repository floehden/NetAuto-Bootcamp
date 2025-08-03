# **Day 16: Concurrency with Goroutines and Channels**

## **Introduction:** 
Go's powerful concurrency model (goroutines and channels). Running multiple network operations in parallel.

## **Code Example: Concurrent Pings**

```go
// concurrent_ping.go
package main

import (
    "fmt"
    "time"
    "sync" // For waiting on goroutines
)

func pingDevice(ip string, wg *sync.WaitGroup, results chan string) {
    defer wg.Done()
    fmt.Printf("Pinging %s...\n", ip)
    time.Sleep(2 * time.Second) // Simulate network delay
    result := fmt.Sprintf("Ping to %s complete.", ip)
    results <- result // Send result to channel
}

func main() {
    devices := []string{"192.168.1.1", "192.168.1.2", "192.168.1.3", "192.168.1.4"}
    var wg sync.WaitGroup
    results := make(chan string, len(devices)) // Buffered channel

    for _, ip := range devices {
        wg.Add(1)
        go pingDevice(ip, &wg, results)
    }

    wg.Wait() // Wait for all goroutines to finish
    close(results) // Close the channel when all results are sent

    fmt.Println("\n--- Ping Results ---")
    for res := range results {
        fmt.Println(res)
    }
}
```

## **Challenge 16:** 
Modify your eAPI `show version` program to connect to two different cEOS nodes concurrently (you'll need a Containerlab setup with two nodes and corresponding `~/.eapi.conf` entries). Use goroutines and channels to retrieve and print the version information from both devices simultaneously.
