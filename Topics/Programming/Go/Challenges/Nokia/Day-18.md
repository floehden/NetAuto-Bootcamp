# **Day 18: Building a Generic Network Client in Go**

## **Concept:** 
Creating a reusable Go package that encapsulates different network interaction methods (SSH, NETCONF, gNMI) for a generic network device.

## **Challenge:** 
Create a `networkclient` package with an interface (e.g., `NetworkDevice`) and implementations for SR Linux using SSH and gNMI. The interface should define methods like `GetHostname()`, `SetHostname()`, `GetInterfaceState()`.

## **Code Example (Conceptual, building on previous days):**
```go
// networkclient/device.go
package networkclient

import "fmt"

// NetworkDevice defines the interface for interacting with a network device.
type NetworkDevice interface {
    Connect() error
    Close() error
    GetHostname() (string, error)
    SetHostname(hostname string) error
    GetInterfaceState(ifaceName string) (string, error) // Simplified state
    // Add more methods as needed (e.g., GetConfig, SetConfig, Subscribe)
}

// SRLGNMIClient implements NetworkDevice for SR Linux via gNMI
type SRLGNMIClient struct {
    // gNMI client related fields
    Target   string
    Username string
    Password string
    // grpc.ClientConn and gpb.GNMIClient
}

func (c *SRLGNMIClient) Connect() error {
    // Implement gNMI connection logic from Day 15/16
    fmt.Printf("Connecting to %s via gNMI...\n", c.Target)
    return nil
}
func (c *SRLGNMIClient) Close() error {
    fmt.Println("Closing gNMI connection.")
    return nil
}
func (c *SRLGNMIClient) GetHostname() (string, error) {
    fmt.Println("Getting hostname via gNMI.")
    return "example-srl-gnmi", nil // Placeholder
}
func (c *SRLGNMIClient) SetHostname(hostname string) error {
    fmt.Printf("Setting hostname to %s via gNMI.\n", hostname)
    return nil
}
func (c *SRLGNMIClient) GetInterfaceState(ifaceName string) (string, error) {
    fmt.Printf("Getting state for %s via gNMI.\n", ifaceName)
    return "up", nil // Placeholder
}

// SRLSSHClient implements NetworkDevice for SR Linux via SSH
type SRLSSHClient struct {
    // SSH client related fields
    Target   string
    Username string
    Password string
    // golang.org/x/crypto/ssh.Client
}

func (c *SRLSSHClient) Connect() error {
    // Implement SSH connection logic from Day 7/8
    fmt.Printf("Connecting to %s via SSH...\n", c.Target)
    return nil
}
func (c *SRLSSHClient) Close() error {
    fmt.Println("Closing SSH connection.")
    return nil
}
func (c *SRLSSHClient) GetHostname() (string, error) {
    fmt.Println("Getting hostname via SSH.")
    return "example-srl-ssh", nil // Placeholder
}
func (c *SRLSSHClient) SetHostname(hostname string) error {
    fmt.Printf("Setting hostname to %s via SSH.\n", hostname)
    return nil
}
func (c *SRLSSHClient) GetInterfaceState(ifaceName string) (string, error) {
    fmt.Printf("Getting state for %s via SSH.\n", ifaceName)
    return "up", nil // Placeholder
}


// main.go
package main

import (
    "fmt"
    "log"
    "day18/networkclient" // Assuming module name is day18
)

func main() {
    // Using the gNMI client implementation
    gnmiClient := &networkclient.SRLGNMIClient{
        Target: "clab-srl-2-nodes-srl1:57400",
        Username: "admin",
        Password: "NokiaSrl1!",
    }
    if err := gnmiClient.Connect(); err != nil {
        log.Fatalf("Error connecting via gNMI: %v", err)
    }
    defer gnmiClient.Close()

    hostname, err := gnmiClient.GetHostname()
    if err != nil {
        log.Printf("Error getting hostname: %v", err)
    } else {
        fmt.Printf("Hostname (gNMI): %s\n", hostname)
    }
    gnmiClient.SetHostname("new-srl-gnmi-host")

    fmt.Println()

    // Using the SSH client implementation
    sshClient := &networkclient.SRLSSHClient{
        Target: "clab-srl-2-nodes-srl1:22",
        Username: "admin",
        Password: "NokiaSrl1!",
    }
    if err := sshClient.Connect(); err != nil {
        log.Fatalf("Error connecting via SSH: %v", err)
    }
    defer sshClient.Close()

    hostname, err = sshClient.GetHostname()
    if err != nil {
        log.Printf("Error getting hostname: %v", err)
    } else {
        fmt.Printf("Hostname (SSH): %s\n", hostname)
    }
    sshClient.SetHostname("new-srl-ssh-host")
}
```
### **Instructions:**
1.  Create a directory `day18`.
2.  Inside `day18`, create a directory `networkclient`.
3.  Save the first code block as `networkclient/device.go`.
4.  Save the second code block as `main.go` in the `day18` directory.
5.  Run `go mod init day18`.
6.  Run `go run main.go`. (Note: The `Connect`, `GetHostname`, etc., methods are placeholders and need to be fully implemented using the SSH and gNMI libraries from previous days for a functional client).
