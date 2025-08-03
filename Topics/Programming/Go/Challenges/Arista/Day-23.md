# **Day 23: gNMI `Set` RPC for Configuration**

## **Introduction:** 
Using the `Set` RPC to configure devices. Understanding `Update`, `Replace`, and `Delete` operations. Building `SetRequest` messages.

## **Code Example: Configuring a Loopback Interface via gNMI**

```go
// gnmi_set_loopback.go
package main

import (
	"context"
	"fmt"
	"os"

	"github.com/openconfig/gnmic/pkg/api"
	"google.golang.org/protobuf/encoding/prototext"
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

	// create a gNMI SetRequest
	var updates []api.GNMIOption

	updates = append(updates, api.Update(
		api.Path("/system/config/hostname"),
		api.Value("ceos0000001", "json_ietf"),
	))

	setReq, err := api.NewSetRequest(updates...)

	if err != nil {
		fmt.Println(err, "failed to create Set request")
		os.Exit(1)
	}

	// send the created gNMI SetRequest to the created target
	setResp, err := target.Set(ctx, setReq)
	if err != nil {
		fmt.Println(err, "failed to send Set request")
		fmt.Println(prototext.Format(setResp))
		os.Exit(1)
	}

	fmt.Printf("gNMI Set Response: %s", prototext.Format(setResp))
}
```

## **Challenge 23:**
1.  Deploy a fresh cEOS lab instance.
2.  Write a Go program using gNMI `Set` to configure an `Ethernet2` interface with an IP address (e.g., `10.20.0.1/24`) and enable it.
3.  After the `Set` operation, use `goeapi` (from Module 3) or a gNMI `Get` request (from Day 22) to verify that the `Ethernet2` interface has been configured correctly on the cEOS device.
