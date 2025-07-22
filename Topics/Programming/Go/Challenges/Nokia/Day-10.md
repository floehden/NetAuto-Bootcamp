# **Day 10: Inventory Management in Go for Containerlab**

## **Concept:** 
Programmatically obtaining information about deployed Containerlab nodes (IPs, names) to build an inventory in Go, which is essential for scaling automation.

## **Challenge:** 
Write a Go program that parses the `srl_2_nodes.clab.yaml` file (or directly queries Containerlab if possible, but parsing YAML is more direct for this exercise) to extract the hostnames and expected management IPs of the SR Linux nodes.

## **Code Example:**
```go
// main.go
package main

import (
    "fmt"
    "io/ioutil"
    "log"

    "gopkg.in/yaml.v2"
)

// Simplified struct for parsing Containerlab topology
type ContainerlabTopology struct {
    Name     string `yaml:"name"`
    Topology struct {
        Nodes map[string]struct {
            Kind string `yaml:"kind"`
            // In a real scenario, you might add 'mgmt_ipv4' or other fields
        } `yaml:"nodes"`
    } `yaml:"topology"`
}

func main() {
    yamlFile := "srl_2_nodes.clab.yaml" // Ensure this file exists from Day 9

    data, err := ioutil.ReadFile(yamlFile)
    if err != nil {
        log.Fatalf("Error reading YAML file: %v", err)
    }

    var topo ContainerlabTopology
    err = yaml.Unmarshal(data, &topo)
    if err != nil {
        log.Fatalf("Error unmarshalling YAML: %v", err)
    }

    fmt.Printf("Containerlab Lab Name: %s\n", topo.Name)
    fmt.Println("Discovered SR Linux Nodes:")
    for nodeName, nodeDetails := range topo.Topology.Nodes {
        if nodeDetails.Kind == "srl" {
            // Containerlab nodes typically resolve to <lab_name>-<node_name> in Docker DNS
            // For management, you'd usually connect to the container's management IP.
            // For simplicity, we'll use the Containerlab-assigned hostname in this context.
            fmt.Printf("  - Name: %s, Kind: %s, Management Hostname: clab-%s-%s\n",
                nodeName, nodeDetails.Kind, topo.Name, nodeName)
        }
    }
}
```

### **Instructions:**
1.  Ensure `srl_2_nodes.clab.yaml` is present from Day 9.
2.  Run `go mod init day10`.
3.  Run `go get gopkg.in/yaml.v2`.
4.  Save the code as `main.go`.
5.  Run `go run main.go`.
