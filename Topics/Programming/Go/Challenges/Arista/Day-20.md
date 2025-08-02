# **Day 20: Putting It All Together: Automated Lab Deployment & Configuration**

## **Introduction:** 
Building a script that automates the entire process: deploying a Containerlab, configuring devices from templates/JSON, and verifying the state.

## **Code Example: Full Automation Workflow**

This will be a more complex script, combining elements from previous days.

1.  **Define Topology (YAML):** `automation_lab.clab.yaml` with a few cEOS nodes.
2.  **Define Device Configurations (JSON/Templates):** Individual files for each device.
3.  **Go Program:**
        * Reads the topology YAML.
        * Deploys the Containerlab topology using `os/exec` to run `containerlab deploy`.
        * Waits for devices to boot up (e.g., a simple `time.Sleep` or more robust health check).
        * For each device:
            * Reads its specific configuration (from JSON or template).
            * Uses `goeapi` to connect and push the configuration.
            * Uses `goeapi` to run `show` commands to verify the configuration (e.g., `show ip interface brief`, `show bgp summary`).
            * Reports success or failure.
        * Optionally, `defer` the `containerlab destroy` command.

<!-- end list -->

```go
// automate_lab.go
package main

import (
	//"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"os"
	"os/exec"
	"strings"
	"sync"

	//"text/template"
	"flag"
	"time"
	//"github.com/aristanetworks/goeapi"
)

// Structs for parsing JSON config
type Interface struct {
	Name        string `json:"name"`
	IP          string `json:"ip"`
	Description string `json:"description"`
}

type DeviceConfig struct {
	Hostname   string      `json:"hostname"`
	LoopbackIP string      `json:"loopback_ip"`
	Interfaces []Interface `json:"interfaces"`
}

// Function to deploy Containerlab
func deployLab(topoFile string) error {
	fmt.Printf("Deploying Containerlab topology: %s\n", topoFile)
	cmd := exec.Command("sudo", "containerlab", "deploy", "-t", topoFile)
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	return cmd.Run()
}

// Function to destroy Containerlab
func destroyLab(topoFile string) error {
	fmt.Printf("Destroying Containerlab topology: %s\n", topoFile)
	cmd := exec.Command("sudo", "containerlab", "destroy", "-t", topoFile, "--cleanup")
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	return cmd.Run()
}

// Function to get container IP (simplistic, assumes standard Docker bridge network)
func getContainerIP(containerName string) (string, error) {
	cmd := exec.Command("docker", "inspect", "-f", "{{.NetworkSettings.IPAddress}}", containerName)
	output, err := cmd.Output()
	if err != nil {
		return "", fmt.Errorf("failed to get IP for %s: %w", containerName, err)
	}
	return strings.TrimSpace(string(output)), nil
}

// Function to configure a device
func configureDevice(nodeName, nodeIP, configFilePath string, wg *sync.WaitGroup, errorsChan chan error) {
	defer wg.Done()
	fmt.Printf("Configuring %s (%s)...\n", nodeName, nodeIP)

	// Generate .eapi.conf dynamically (or use env vars/direct connect)
	// For this tutorial, assume .eapi.conf is pre-configured or use dummy values if needed
	// A better approach would be to pass connection details directly to goeapi if possible
	// or generate a temporary .eapi.conf
	// For simplicity, let's assume we can directly connect using username/password
	// (This might require modifying goeapi or directly using HTTP client for eAPI)

	// Let's use a simplified direct connection approach if goeapi supports it, otherwise a mock
	// For now, let's just log and mock success/failure
	// In a real scenario, you'd configure the goeapi.Connect method to use the IP and credentials

	// MOCKING goeapi.Connect and Configure
	// In a real application, replace this with actual goeapi calls:
	// node, err := goeapi.Connect(nodeName) // Or create a direct connection
	// if err != nil { errorsChan <- fmt.Errorf("failed to connect to %s: %w", nodeName, err); return }

	configData, err := ioutil.ReadFile(configFilePath)
	if err != nil {
		errorsChan <- fmt.Errorf("failed to read config file %s for %s: %w", configFilePath, nodeName, err)
		return
	}

	var deviceCfg DeviceConfig
	err = json.Unmarshal(configData, &deviceCfg)
	if err != nil {
		errorsChan <- fmt.Errorf("failed to parse JSON config for %s: %w", nodeName, err)
		return
	}

	// Generate commands from parsed config
	var configCommands []string
	configCommands = append(configCommands, fmt.Sprintf("hostname %s", deviceCfg.Hostname))
	configCommands = append(configCommands, fmt.Sprintf("interface Loopback0"))
	configCommands = append(configCommands, fmt.Sprintf("ip address %s", deviceCfg.LoopbackIP))
	configCommands = append(configCommands, fmt.Sprintf("description \"Configured by automation script\""))
	configCommands = append(configCommands, "no shutdown") // Ensure loopback is up

	for _, iface := range deviceCfg.Interfaces {
		configCommands = append(configCommands, fmt.Sprintf("interface %s", iface.Name))
		configCommands = append(configCommands, fmt.Sprintf("ip address %s", iface.IP))
		configCommands = append(configCommands, fmt.Sprintf("description \"%s\"", iface.Description))
		configCommands = append(configCommands, "no switchport")
		configCommands = append(configCommands, "no shutdown")
	}

	// Simulate sending commands via eAPI
	fmt.Printf("Sending %d configuration commands to %s...\n", len(configCommands), nodeName)
	time.Sleep(2 * time.Second) // Simulate network/device processing time

	// In a real scenario:
	// _, err = node.Configure(configCommands)
	// if err != nil { errorsChan <- fmt.Errorf("failed to configure %s: %w", nodeName, err); return }

	fmt.Printf("Configuration for %s successful.\n", nodeName)

	// Verify configuration
	// In a real scenario:
	// showCmds := []string{"show running-config interface Loopback0", "show ip interface brief"}
	// showResp, err := node.RunCommands(showCmds)
	// if err != nil { errorsChan <- fmt.Errorf("failed to verify config on %s: %w", nodeName, err); return }
	// fmt.Printf("Verification output for %s: %+v\n", nodeName, showResp)

	fmt.Printf("Verification for %s successful (mock).\n", nodeName)
}

func main() {
	topoFile := flag.String("topo", "automation_lab.clab.yaml", "Containerlab Topology file")
	defer func() {
		if r := recover(); r != nil {
			fmt.Printf("Recovered from panic: %v\n", r)
		}
		// Uncomment to automatically destroy the lab after run
		if err := destroyLab(*topoFile); err != nil {
			fmt.Printf("Error destroying lab: %v\n", err)
		}
	}()

	// 1. Deploy the Containerlab topology
	if err := deployLab(*topoFile); err != nil {
		fmt.Printf("Error deploying lab: %v\n", err)
		os.Exit(1)
	}

	// Give devices time to boot up and services to start
	fmt.Println("Waiting 30 seconds for devices to boot up...")
	time.Sleep(30 * time.Second)

	// 2. Prepare device configurations and get IPs
	deviceNodes := map[string]string{
		"clab-automation-lab-ceos1": "config_ceos1.json",
		"clab-automation-lab-ceos2": "config_ceos2.json",
	}

	var wg sync.WaitGroup
	errorsChan := make(chan error, len(deviceNodes))

	for containerName, configFile := range deviceNodes {
		ip, err := getContainerIP(containerName)
		if err != nil {
			fmt.Printf("Could not get IP for %s: %v. Skipping configuration.\n", containerName, err)
			continue
		}
		fmt.Printf("Found IP for %s: %s\n", containerName, ip)

		wg.Add(1)
		go configureDevice(containerName, ip, configFile, &wg, errorsChan)
	}

	wg.Wait()
	close(errorsChan)

	hasErrors := false
	for err := range errorsChan {
		fmt.Printf("Automation Error: %v\n", err)
		hasErrors = true
	}

	if hasErrors {
		fmt.Println("\nAutomation completed with errors.")
	} else {
		fmt.Println("\nAutomation completed successfully!")
	}
}
```

Helper files for automation_lab.go 
automation_lab.clab.yaml
```yaml
name: automation-lab
topology:
  nodes:
    ceos1:
      kind: arista_ceos
      image: ceos:4.34.0F
      startup-config: |
        no aaa root
        username admin privilege 15 role network-admin secret sha512 $6$RxQ5ae0GOW6SAiCU$7qzQNGX2pSIWBYGF8Xh30lo/s418/diYEEZj9rPrTJiAkYv0s6AvjpTfUHMGz.a58Hg29Yy/nV0Zvplux0
        management api http-commands
          no shutdown
          protocol http
    ceos2:
      kind: arista_ceos
      image: ceos:4.34.0F
      startup-config: |
        no aaa root
        username admin privilege 15 role network-admin secret sha512 $6$RxQ5ae0GOW6SAiCU$7qzQNGX2pSIWBYGF8Xh30lo/s418/diYEEZj9rPrTJiAkYv0s6AvjpTfUHMGz.a58Hg29Yy/nV0Zvplux0
        management api http-commands
          no shutdown
          protocol http
  links:
    - endpoints: ["ceos1:eth1", "ceos2:eth1"]
```

config_ceos1.json:
```json
{
    "hostname": "leaf1",
    "loopback_ip": "1.1.1.1/32",
    "interfaces": [
        {
            "name": "Ethernet1",
            "ip": "10.0.0.1/30",
            "description": "Link to ceos2"
        }
    ]
}
```

config_ceos2.json:
```json
{
    "hostname": "leaf2",
    "loopback_ip": "2.2.2.2/32",
    "interfaces": [
        {
            "name": "Ethernet1",
            "ip": "10.0.0.2/30",
            "description": "Link to ceos1"
        }
    ]
}
```

## **Challenge 20:** 
Implement the full automation workflow described above. Ensure your `automation_lab.clab.yaml` and device JSON configuration files are correctly set up. Run the `automate_lab.go` program and verify that the cEOS routers are deployed and configured as expected. Add a final verification step to your Go program that pings `Loopback0` of `ceos2` from `ceos1` (you'll need to figure out how to send CLI commands via eAPI for ping, or run `docker exec` for ping).
