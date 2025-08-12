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
	"regexp"
	"strconv"
	"strings"

	"github.com/aristanetworks/goeapi"
)

// OutputData represents the structure of each object in the JSON array
type OutputData struct {
	Output string `json:"output"`
}

// SystemInfo represents the parsed information from the "output" string
type SystemInfo struct {
	HardwareVersion      string
	SerialNumber         string
	HardwareMACAddress   string
	SystemMACAddress     string
	SoftwareImageVersion string
	Architecture         string
	InternalBuildVersion string
	InternalBuildID      string
	ImageFormatVersion   string
	ImageOptimization    string
	KernelVersion        string
	Uptime               string
	TotalMemoryKB        int
	FreeMemoryKB         int
}

func main() {
	// You'll need to create a .eapi.conf file in your home directory
	// Example ~/.eapi.conf:
	// [connection:ceos1]
	// host=172.20.20.2
	// username=admin
	// password=admin
	// enablepwd=passwd
	// transport=http
	node, err := goeapi.ConnectTo("ceos1")
	if err != nil {
		fmt.Printf("Error connecting to node: %v\n", err)
		os.Exit(1)
	}

	cmds := []string{"show version"}
	response, err := node.RunCommands(cmds, "text")
	if err != nil {
		fmt.Printf("Error running commands: %v\n", err)
		os.Exit(1)
	}

	var rawTextOutput string // Variable to hold the extracted raw text

	if len(response.Result) > 0 {

		resultMap := response.Result[0] // This will now be of type map[string]interface{}

		if outputVal, ok := resultMap["output"]; ok { // Accessing the map
			if outputStr, ok := outputVal.(string); ok {
				rawTextOutput = outputStr
			} else {
				fmt.Printf("Value for 'output' key is not a string (type: %T). Map content: %+v\n", outputVal, resultMap)
				os.Exit(1)
			}
		} else if outputVal, ok := resultMap["response"]; ok { // Sometimes it's "response"
			if outputStr, ok := outputVal.(string); ok {
				rawTextOutput = outputStr
			} else {
				fmt.Printf("Value for 'response' key is not a string (type: %T). Map content: %+v\n", outputVal, resultMap)
				os.Exit(1)
			}
		} else {
			fmt.Printf("Neither 'output' nor 'response' key found in the result map. Map content: %+v\n", resultMap)
			os.Exit(1)
		}
	} else {
		fmt.Println("No results returned for the command.")
		os.Exit(0)
	}

	// The rest of your code remains the same...
	var transformedOutput []OutputData
	if rawTextOutput != "" {
		transformedOutput = []OutputData{{Output: rawTextOutput}}
	} else {
		fmt.Println("Extracted raw text output is empty. Cannot proceed with parsing.")
		os.Exit(1)
	}

	jsonBytes, err := json.Marshal(transformedOutput)
	if err != nil {
		fmt.Printf("Error marshalling transformed output: %v\n", err)
		os.Exit(1)
	}
	// fmt.Printf("Raw JSON Response for parsing: %s\n", jsonBytes)

	var outputDatas []OutputData
	err = json.Unmarshal(jsonBytes, &outputDatas)
	if err != nil {
		fmt.Printf("Error unmarshaling JSON: %v\n", err)
		return
	}

	if len(outputDatas) == 0 {
		fmt.Println("No data found in JSON response.")
		return
	}

	outputString := outputDatas[0].Output
	systemInfo, err := parseSystemInfo(outputString)
	if err != nil {
		fmt.Printf("Error parsing system info: %v\n", err)
		return
	}

	// fmt.Printf("%+v\n", systemInfo)
	fmt.Println("\n--- Parsed System Information ---")
	if systemInfo.HardwareVersion != "" {
		fmt.Printf("  Hardware version: %s\n", systemInfo.HardwareVersion)
	} else {
		fmt.Println("  Hardware version: (Not provided)") // Indicate if empty
	}
	fmt.Printf("  Serial number: %s\n", systemInfo.SerialNumber)
	fmt.Printf("  Hardware MAC address: %s\n", systemInfo.HardwareMACAddress)
	fmt.Printf("  System MAC address: %s\n", systemInfo.SystemMACAddress)
	fmt.Printf("\n")
	fmt.Printf("  Software image version: %s\n", systemInfo.SoftwareImageVersion)
	fmt.Printf("  Architecture: %s\n", systemInfo.Architecture)
	fmt.Printf("  Internal build version: %s\n", systemInfo.InternalBuildVersion)
	fmt.Printf("  Internal build ID: %s\n", systemInfo.InternalBuildID)
	fmt.Printf("  Image format version: %s\n", systemInfo.ImageFormatVersion)
	fmt.Printf("  Image optimization: %s\n", systemInfo.ImageOptimization)
	fmt.Printf("\n")
	fmt.Printf("  Kernel version: %s\n", systemInfo.KernelVersion)
	fmt.Printf("\n")
	fmt.Printf("  Uptime: %s\n", systemInfo.Uptime)
	fmt.Printf("  Total memory: %d kB\n", systemInfo.TotalMemoryKB)
	fmt.Printf("  Free memory: %d kB\n", systemInfo.FreeMemoryKB)
	fmt.Println("-------------------------------")
}

// parseSystemInfo extracts information from the multi-line output string
func parseSystemInfo(output string) (SystemInfo, error) {
	info := SystemInfo{}

	getValue := func(pattern string) string {
		re := regexp.MustCompile(pattern)
		matches := re.FindStringSubmatch(output)
		if len(matches) > 1 {
			return strings.TrimSpace(matches[1])
		}
		return ""
	}

	info.HardwareVersion = getValue(`Hardware version: (.*)`)
	info.SerialNumber = getValue(`Serial number: (.*)`)
	info.HardwareMACAddress = getValue(`Hardware MAC address: (.*)`)
	info.SystemMACAddress = getValue(`System MAC address: (.*)`)
	info.SoftwareImageVersion = getValue(`Software image version: (.*)`)
	info.Architecture = getValue(`Architecture: (.*)`)
	info.InternalBuildVersion = getValue(`Internal build version: (.*)`)
	info.InternalBuildID = getValue(`Internal build ID: (.*)`)
	info.ImageFormatVersion = getValue(`Image format version: (.*)`)
	info.ImageOptimization = getValue(`Image optimization: (.*)`)
	info.KernelVersion = getValue(`Kernel version: (.*)`)
	info.Uptime = getValue(`Uptime: (.*)`)

	totalMemoryStr := getValue(`Total memory: (\d+) kB`)
	if totalMemoryStr != "" {
		totalMemory, err := strconv.Atoi(totalMemoryStr)
		if err != nil {
			return info, fmt.Errorf("failed to parse Total memory: %w", err)
		}
		info.TotalMemoryKB = totalMemory
	}

	freeMemoryStr := getValue(`Free memory: (\d+) kB`)
	if freeMemoryStr != "" {
		freeMemory, err := strconv.Atoi(freeMemoryStr)
		if err != nil {
			return info, fmt.Errorf("failed to parse Free memory: %w", err)
		}
		info.FreeMemoryKB = freeMemory
	}

	return info, nil
}
```

## **Challenge 12:** 
Using `goeapi`, get the output of `show interfaces brief`. Define a Go struct to parse the key information (e.g., interface name, status, IP address). Print a formatted summary of each interface.

