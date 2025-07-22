# **Day 13: NETCONF with Go - Edit Operations**

## **Concept:** 
Using NETCONF to modify the configuration of SR Linux, focusing on `<edit-config>` and `<commit>` operations.

## **Challenge:** 
Write a Go program to configure a loopback interface on `srl1` using NETCONF.

## **Code Example:**
```go
// main.go
package main

import (
    "fmt"
    "log"
    "time"

    "github.com/Juniper/go-netconf/netconf"
)

func main() {
    session, err := netconf.DialSSH("clab-srl-2-nodes-srl1:830",
        &netconf.SSHClientConfig{
            User:     "admin",
            Password: "NokiaSrl1!",
            Timeout:  10 * time.Second,
        })
    if err != nil {
        log.Fatalf("Failed to dial NETCONF: %v", err)
    }
    defer session.Close()

    // Configuration to apply (XML format)
    configXML := `
    \<config\>
    \<interface xmlns="urn:nokia.com:srlinux-interfaces"\>
    \<name\>loopback0\</name\>
    \<admin-state\>enable\</admin-state\>
    \<subinterface\>
    \<index\>0\</index\>
    \<admin-state\>enable\</admin-state\>
    \<ipv4\>
    \<address\>
    \<ip-prefix\>1.1.1.1/32\</ip-prefix\>
    \</address\>
    \</ipv4\>
    \</subinterface\>
    \</interface\>
    \</config\>
    \`

    // Edit the configuration
    _, err = session.EditConfig("candidate", configXML)
    if err != nil {
        log.Fatalf("Failed to edit config: %v", err)
    }
    fmt.Println("Configuration applied to candidate datastore.")

    // Commit the changes
    _, err = session.Commit()
    if err != nil {
        log.Fatalf("Failed to commit config: %v", err)
    }
    fmt.Println("Configuration committed successfully.")

    // Verify by getting the config again (optional)
    reply, err := session.GetConfig("running", "<filter><interface xmlns=\"urn:nokia.com:srlinux-interfaces\"><name>loopback0</name></interface></filter>")
    if err != nil {
        log.Fatalf("Failed to get config after commit: %v", err)
    }
    fmt.Println("\n--- New Running Configuration for loopback0 ---")
    fmt.Println(reply.Data)
}
```

### **Instructions:**
1.  Ensure your `srl_2_nodes.clab.yaml` is deployed.
2.  Run `go mod tidy`.
3.  Save the code as `main.go`.
4.  Run `go run main.go`.
