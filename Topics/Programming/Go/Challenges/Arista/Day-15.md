# **Day 15: Templating Configurations with Go's `text/template`**

## **Introduction:** 
Using Go's built-in templating engine (`text/template`) to generate dynamic configurations based on input data. This is crucial for scalable automation.

## **Code Example: Interface Template**

Create `interface_template.txt`:

```
interface {{.InterfaceName}}
    description "{{.Description}}"
    ip address {{.IPAddress}}/{{.PrefixLength}}
    no shutdown
```

`generate_config.go`:

```go
package main

import (
    "bytes"
    "fmt"
    "os"
    "text/template"
)

type InterfaceData struct {
    InterfaceName string
    Description   string
    IPAddress     string
    PrefixLength  int
}

func main() {
    data := InterfaceData{
        InterfaceName: "Ethernet2",
        Description:   "Uplink to Core",
        IPAddress:     "10.0.1.1",
        PrefixLength:  24,
    }

    tmpl, err := template.ParseFiles("interface_template.txt")
    if err != nil {
        fmt.Printf("Error parsing template: %v\n", err)
        os.Exit(1)
    }

    var buf bytes.Buffer
    err = tmpl.Execute(&buf, data)
    if err != nil {
        fmt.Printf("Error executing template: %v\n", err)
        os.Exit(1)
    }

    fmt.Println("--- Generated Configuration ---")
    fmt.Println(buf.String())

    // In a real scenario, you would then send buf.String() to the device via eAPI.
}
```

## **Challenge 15:** 
Create a template for configuring a BGP neighbor, including `remote-as`, `neighbor IP`, and `update-source`. Define a struct to hold the necessary BGP neighbor data. Generate a BGP configuration for two different neighbors and print the generated config.


