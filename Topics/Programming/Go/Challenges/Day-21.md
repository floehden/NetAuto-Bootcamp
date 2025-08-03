# **Day 21: Introduction to gNMI and Basic Connection**

## **Introduction:** 
What is gNMI? Why use it over eAPI? (Streaming Telemetry, Declarative Configuration, Protobuf/gRPC, schema-driven). gNMI RPC types: `Capabilities`, `Get`, `Set`, `Subscribe`. </br>
</br>
**Enabling gNMI on cEOS:** Review the `startup-config` change required.

## **Code Example: Connecting to a gNMI Server**

```yaml
gnmi-lab.clab.yaml
name: gnmi-lab
topology:
  nodes:
    ceos1:
      kind: arista_ceos
      image: ceos:4.34.0F

    ceos2:
      kind: arista_ceos
      image: ceos:4.34.0F
      
  links:
    - endpoints: ["ceos1:eth1", "ceos2:eth1"]
```


```go
// gnmi_connect.go
package main

import (
	"context"
	"fmt"
	"os"

	"github.com/openconfig/gnmic/pkg/api"
)

const (
	targetAddress = "172.20.20.2:6030" // Replace with your cEOS container IP and gNMI port
	username      = "admin"
	password      = "admin"
)

func main() {
	target, err := api.NewTarget(
		api.Address(targetAddress),
		api.Username(username),
		api.Password(password),
		api.SkipVerify(false),
		api.Insecure(true),
	)
	if err != nil {
		fmt.Println(err, "failed to create gnmi client")
		os.Exit(1)
	}

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	// create a gNMI client
	err = target.CreateGNMIClient(ctx)
	if err != nil {
		fmt.Println(err, "failed to create gnmi client")
		os.Exit(1)
	}
	defer target.Close()

	// send the created gNMI GetRequest to the created target
	capResp, err := target.Capabilities(ctx)
	if err != nil {
		fmt.Println(err, "failed to execute get request")
		os.Exit(1)
	}

	fmt.Println("gNMI Capabilities:")
	fmt.Printf("  Supported Models: %+v\n", capResp.GetSupportedModels())
	fmt.Printf("  Supported Versions: %+v\n", capResp.GetGNMIVersion())
	fmt.Printf("  Supported Encodings: %+v\n", capResp.GetSupportedEncodings())
}
```

### **Instructions**

1.  Deploy the `gnmi-lab.clab.yaml`.
2.  Find the IP address of `clab-automation-lab-ceos1`.
3.  Update `gnmi_connect.go` with the correct IP and run the program. Verify that it connects and prints capabilities.

## Challanges
* Update the Code, that it works for a SR Linux Topology
* Hints: look at the differences btween the connections in the documentation on gnmi of [SRLinux](https://containerlab.dev/manual/kinds/srl/#__tabbed_1_3) and [ceos](https://containerlab.dev/manual/kinds/ceos/#__tabbed_1_4) at the containerlab documentatation 