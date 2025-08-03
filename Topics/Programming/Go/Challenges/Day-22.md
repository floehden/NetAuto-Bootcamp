
# **Day 22: gNMI `Get` RPC for Operational State**

## **Introduction:** 
Using the `Get` RPC to fetch specific operational state or configuration. Understanding gNMI Paths. Retrieving interface status.

## **Code Example: Get System Name (Nokia SRL)**

```go
// gnmi_get_name.go
package main

import (
	"context"
	"fmt"
	"os"

	"github.com/openconfig/gnmic/pkg/api"
)

const (
	targetAddress = "172.20.20.2:57400" // Replace with your cEOS container IP and gNMI port
	username      = "admin"
	password      = "NokiaSrl1!"
)

func main() {
	target, err := api.NewTarget(
		api.Address(targetAddress),
		api.Username(username),
		api.Password(password),
		api.SkipVerify(true),
		api.Insecure(false),
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
	// create a GetRequest
	getReq, err := api.NewGetRequest(
		api.Path("/system/name/host-name"),
		api.Encoding("json_ietf"),
	)
	if err != nil {
		fmt.Println(err, "failed to execute get request")
		os.Exit(1)
	}

	// send the created gNMI GetRequest to the created target
	getResp, err := target.Get(ctx, getReq)
	if err != nil {
		fmt.Println(err, "failed to execute get request")
		os.Exit(1)
	}

	for _, notif := range getResp.Notification {
		for _, update := range notif.Update {
			fmt.Printf("Path: %s\n", update.Path)
			fmt.Printf("Return value: %s\n", update.Val.GetJsonIetfVal())
		}
	}
}

```


## **Code Example: Get Interface State (Arista ceos)**

```go
// gnmi_get_interface.go
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
	// create a GetRequest
	getReq, err := api.NewGetRequest(
		api.Path("/interfaces/interface[name=Ethernet1]/state"),
		api.Encoding("json_ietf"),
	)
	if err != nil {
		fmt.Println(err, "failed to execute get request")
		os.Exit(1)
	}

	// send the created gNMI GetRequest to the created target
	getResp, err := target.Get(ctx, getReq)
	if err != nil {
		fmt.Println(err, "failed to execute get request")
		os.Exit(1)
	}

	for _, notif := range getResp.Notification {
		for _, update := range notif.Update {
			fmt.Printf("Path: %s\n", update.Path)
			fmt.Printf("Return value: %s\n", update.Val.GetJsonIetfVal())
		}
	}
}

```

## **Challenge 22:**

1.  Ensure your cEOS lab is deployed with `Ethernet1` configured and up (e.g., from Day 7's `initial_config.clab.yaml`).
2.  Modify `gnmi_get_interface.go` to fetch the operational state for `Loopback0`.
3.  Print the `Loopback0` IP address and its operational status (up/down) from the gNMI response. You might need to adjust the gNMI path and how you extract the values based on the exact JSON structure returned by cEOS for `Loopback0` state.

