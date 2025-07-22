# **Day 5: JSON and YAML Serialization/Deserialization**

## * **Concept:** 
Reading and writing structured data in JSON and YAML formats, common in network configurations and APIs. Using Go's `encoding/json` and external YAML libraries (`gopkg.in/yaml.v2`).

## * **Challenge:** 
Take the `routers` slice from Day 4, serialize it into a JSON string, and then deserialize a YAML string representing a similar router configuration back into a `Router` struct.

## * **Code Example:**
```go
// main.go
package main

import (
    "encoding/json"
    "fmt"
    "log"

    "gopkg.in/yaml.v2" // You'll need to run 'go get gopkg.in/yaml.v2'
)

type Router struct {
    Name        string            `json:"name" yaml:"name"`
    IPAddress   string            `json:"ip_address" yaml:"ip_address"`
    Interfaces  map[string]string `json:"interfaces" yaml:"interfaces"`
}

func main() {
    // Data to serialize to JSON
    routers := []Router{
        {
            Name:      "srl1",
            IPAddress: "192.168.0.1",
            Interfaces: map[string]string{
                "ethernet-1/1": "10.0.0.1/24",
                "ethernet-1/2": "10.0.0.5/24",
            },
        },
    }

    // Marshal to JSON
    jsonData, err := json.MarshalIndent(routers, "", "  ")
    if err != nil {
        log.Fatalf("Error marshalling JSON: %v", err)
    }
    fmt.Println("--- JSON Output ---")
    fmt.Println(string(jsonData))

    // YAML data to deserialize
    yamlData := `
    - name: srl3
        ip\_address: 192.168.0.3
        interfaces:
            ethernet-1/1: 10.0.0.10/24
            loopback0: 10.3.3.3/32
    \`
    var newRouters []Router
    err = yaml.Unmarshal([]byte(yamlData), \&newRouters)
    if err \!= nil {
      log.Fatalf("Error unmarshalling YAML: %v", err)
    }
    fmt.Println("\\n--- YAML Unmarshalled Output ---")
    for \_, r := range newRouters {
        fmt.Printf("Router: %+v\\n", r)
    }
}
```

### **Instructions:**
1.  Run `go mod init day5` (or similar)
2.  Run `go get gopkg.in/yaml.v2`
3.  Save the code as `main.go`.
4.  Run `go run main.go`.

