# **Day 24: gNMI `Subscribe` for Streaming Telemetry**

## **Introduction:** 
The power of gNMI: streaming telemetry. Understanding `ONCE`, `POLL`, and `STREAM` subscriptions. Handling continuous updates.

## **Code Example: Streaming Interface Counters**

```go
// gnmi_subscribe_counters.go
package main

import (
	"context"
	"fmt"
	"os"
	"time"

	"github.com/openconfig/gnmic/pkg/api"
	"google.golang.org/protobuf/encoding/prototext"
)

const (
	targetAddress = "172.20.20.2:6030" // Replace with your cEOS container IP and gNMI port
	username      = "admin"
	password      = "admin"
	intervalSecs  = 10
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

	// create a gNMI subscribeRequest
	subReq, err := api.NewSubscribeRequest(
		api.Encoding("json_ietf"),
		api.SubscriptionListMode("stream"),
		api.Subscription(
			api.Path("/system/memory/state/used"),
			api.SubscriptionMode("sample"),
			api.SampleInterval(10*time.Second),
		),
		api.Subscription(
			api.Path("/system/cpus/cpu[index=0]/state/idle/avg"),
			api.SubscriptionMode("sample"),
			api.SampleInterval(intervalSecs*time.Second),
		),
	)
	if err != nil {
		fmt.Println(err, "subscription request")
	}
	fmt.Println(prototext.Format(subReq))
	// start the subscription
	go target.Subscribe(ctx, subReq, "sub1")
	// start a goroutine that will stop the subscription after x seconds
	go func() {
		select {
		case <-ctx.Done():
			return
		case <-time.After(42 * time.Second):
			target.StopSubscription("sub1")
		}
	}()
	subRspChan, subErrChan := target.ReadSubscriptions()
	for {
		select {
		case rsp := <-subRspChan:
			fmt.Println(prototext.Format(rsp.Response))
		case tgErr := <-subErrChan:
			fmt.Printf("subscription %q stopped: %v", tgErr.SubscriptionName, tgErr.Err)
		}
	}

}
```

## **Challenge 24:**

1.  Deploy your cEOS lab.
2.  Run the `gnmi_subscribe_counters.go` program.
3.  While it's running, SSH into `ceos1` and generate some traffic on `Ethernet1` (e.g., `ping 10.0.0.2` if `ceos2` is connected, or send some packets with `sudo tcpdump -i eth1 -w /dev/null`).

