# **Day 18: Working with JSON and YAML Configuration Files (External Data)**

## **Introduction:** 
Reading configuration data from external JSON files. Marshalling and unmarshalling JSON in Go.

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
1.  Run `go mod init day18-1` (or similar)
2.  Run `go get gopkg.in/yaml.v2`
3.  Save the code as `main.go`.
4.  Run `go run main.go`.

## **Code Example 2: Device Configuration from JSON**

Create `device_config.json`:

```json
{
    "hostname": "leaf3",
    "ip_address": "10.0.0.3",
    "loopback_ip": "3.3.3.3/32",
    "interfaces": [
        {
            "name": "Ethernet1",
            "ip": "10.0.3.1/30",
            "description": "Link to spine1"
        },
        {
            "name": "Ethernet2",
            "ip": "10.0.3.5/30",
            "description": "Link to spine2"
        }
    ]
}
```

`read_json_config.go`:

```go
package main

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "os"
)

type Interface struct {
    Name        string `json:"name"`
    IP          string `json:"ip"`
    Description string `json:"description"`
}

type DeviceConfig struct {
    Hostname    string      `json:"hostname"`
    IPAddress   string      `json:"ip_address"`
    LoopbackIP  string      `json:"loopback_ip"`
    Interfaces  []Interface `json:"interfaces"`
}

func main() {
    jsonData, err := ioutil.ReadFile("device_config.json")
    if err != nil {
        fmt.Printf("Error reading JSON file: %v\n", err)
        os.Exit(1)
    }

    var config DeviceConfig
    err = json.Unmarshal(jsonData, &config)
    if err != nil {
        fmt.Printf("Error unmarshalling JSON: %v\n", err)
        os.Exit(1)
    }

    fmt.Printf("Device Hostname: %s\n", config.Hostname)
    fmt.Printf("Loopback IP: %s\n", config.LoopbackIP)
    fmt.Println("Configuring Interfaces:")
    for _, iface := range config.Interfaces {
        fmt.Printf("  - %s: IP %s, Description: %s\n", iface.Name, iface.IP, iface.Description)
        // In a real scenario, you'd generate commands and send via eAPI
    }
}
```

## **Challenge 18:** 
Create a Containerlab topology with a single cEOS node. Write a Go program that reads device configuration from a JSON file (similar to `device_config.json` but specific to your lab node). Use `goeapi` and the parsed JSON data to configure the hostname, a loopback interface, and at least one Ethernet interface on the cEOS router.

