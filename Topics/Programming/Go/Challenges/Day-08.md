# **Day 8: GoLang Basic File I/O for Config Files**

## **Introduction:** 
Reading and writing files in Go. Reading topology and configuration data from files.

## **Code Example: Reading a Topology File**

```go
// read_topo.go
package main

import (
    "fmt"
    "os"
    "io/ioutil"
)

func main() {
    filePath := "initial_config.clab.yaml" // Use the YAML file from Day 7

    content, err := ioutil.ReadFile(filePath)
    if err != nil {
        fmt.Printf("Error reading file: %v\n", err)
        return
    }
    fmt.Println("--- Topology File Content ---")
    fmt.Println(string(content))
}
```

## **Challenge 8:** 
Create a simple text file `device_list.txt` with a list of device hostnames (one per line). Write a Go program that reads this file and prints each hostname, prefixed with "Processing device: ".

