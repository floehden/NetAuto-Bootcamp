# **Day 12: Parsing eAPI Responses with Go Structs**

## **Introduction:** 
Defining Go structs to match JSON response structures. Using `json.Unmarshal` to parse eAPI responses into Go objects.

## **Code Example: Parsing `show version`**

```go
// parse_version.go
package main

import (
    "encoding/json"
    "fmt"
    "os"

    "github.com/aristanetworks/goeapi"
)

// Define a struct to match the "show version" JSON output structure
type ShowVersionResponse struct {
    Architecture       string `json:"architecture"`
    InternalBuildID    string `json:"internalBuildId"`
    InternalBuildVersion string `json:"internalBuildVersion"`
    ModelName          string `json:"modelName"`
    SoftwareImageVersion string `json:"softwareImageVersion"`
    SystemMACAddress   string `json:"systemMacAddress"`
    Uptime             float64 `json:"uptime"`
}

func main() {
    node, err := goeapi.Connect("ceos1")
    if err != nil {
        fmt.Printf("Error connecting to node: %v\n", err)
        os.Exit(1)
    }

    cmds := []string{"show version"}
    response, err := node.RunCommands(cmds)
    if err != nil {
        fmt.Printf("Error running commands: %v\n", err)
        os.Exit(1)
    }

    // eAPI returns a slice of results, even for single commands
    if len(response) > 0 {
        var versionResp ShowVersionResponse
        // The response element is usually a map[string]interface{}
        // We need to marshal it back to JSON bytes and then unmarshal into our struct
        jsonBytes, err := json.Marshal(response[0].(map[string]interface{})["result"])
        if err != nil {
            fmt.Printf("Error marshalling result: %v\n", err)
            os.Exit(1)
        }
        if err := json.Unmarshal(jsonBytes, &versionResp); err != nil {
            fmt.Printf("Error unmarshalling JSON: %v\n", err)
            os.Exit(1)
        }
        fmt.Printf("Software Version: %s\n", versionResp.SoftwareImageVersion)
        fmt.Printf("System MAC: %s\n", versionResp.SystemMACAddress)
        fmt.Printf("Uptime (seconds): %.0f\n", versionResp.Uptime)
    } else {
        fmt.Println("No response received for show version.")
    }
}
```

## **Challenge 12:** 
Using `goeapi`, get the output of `show interfaces brief`. Define a Go struct to parse the key information (e.g., interface name, status, IP address). Print a formatted summary of each interface.

