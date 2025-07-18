# Day 5: Streaming Telemetry with gNMI Subscribe

### Concepts:

  * **Subscribe RPC**: Receiving continuous streams of data from the device.
  * **Subscription Modes**:
      * **STREAM**: Continuous stream of updates as they occur.
      * **ONCE**: A single snapshot of data, then terminates.
      * **POLL**: On-demand snapshots triggered by the client.
  * **QoS (Quality of Service)**: Setting DSCP values for telemetry traffic.

### Containerlab Setup (`day5_lab.yaml`):

```yaml
name: day5_gnmi_lab
topology:
  nodes:
    ceos1:
      kind: ceos
      image: arista/ceos:latest
      # No special configuration needed for subscribe, as gNMI is enabled by default.
      # We'll use the management interface (eth0) for gNMI.
```

### Code Examples:

**1. `day5_subscribe_interface_counters.py` (STREAM mode)**
This script subscribes to interface counter updates and prints them as they change.

```python
from pygnmi.client import gNMIclient
import json
import time

CEOS_IP = "172.20.0.2" # **UPDATE THIS WITH YOUR CEOS1 IP**
GNMI_PORT = 6030
USERNAME = "admin"
PASSWORD = "admin"

if __name__ == "__main__":
    with gNMIclient(
        target=(CEOS_IP, GNMI_PORT),
        username=USERNAME,
        password=PASSWORD,
        insecure=True
    ) as gc:
        print(f"Connecting to {CEOS_IP}:{GNMI_PORT} to subscribe to interface counters (STREAM)...")
        try:
            # OpenConfig path for all interface counters
            # Note: For Arista cEOS, you might get more granular data if you subscribe
            # to a specific interface or a more general path like '/interfaces'.
            # Arista's gNMI implementation might be slightly different for OpenConfig paths
            # versus its native paths for streaming.
            # Let's try a common OpenConfig path first.
            path = ["openconfig-interfaces:interfaces/interface/state/counters"]

            # Subscribe in STREAM mode
            # For pygnmi, 'subscribe2' is often used for streaming data with richer options.
            # The 'sub_mode' specifies the subscription mode.
            subscription_params = {
                "subscription": [
                    {
                        "path": path[0],
                        "mode": "STREAM",
                        "sample_interval": 1000000000 # 1 second in nanoseconds
                    }
                ]
            }

            print("Subscribing to interface counters. Press Ctrl+C to stop.")
            for response in gc.subscribe2(subscribe=subscription_params):
                if 'update' in response:
                    print(f"\n--- Update received at {time.time()} ---")
                    # Process the update message
                    for update in response['update']['update']:
                        print(f"Path: {update['path']['elem']}")
                        if 'val' in update:
                            print(f"Value: {json.dumps(update['val'], indent=2)}")
                        elif 'value' in update: # Sometimes the value is directly in 'value'
                            print(f"Value: {json.dumps(update['value'], indent=2)}")
                elif 'sync_response' in response and response['sync_response']:
                    print("--- Initial synchronization complete ---")
                elif 'error' in response:
                    print(f"Error in subscription: {response['error']}")

        except KeyboardInterrupt:
            print("\nSubscription stopped by user.")
        except Exception as e:
            print(f"Error during subscription: {e}")

```

**How to generate traffic for testing:**
After running the subscription script, you can generate traffic on `ceos1` by:

1.  Accessing the cEOS CLI: `docker exec -it day5_gnmi_lab-ceos1 Cli`
2.  Ping another interface (if connected to another node, or even ping an invalid IP to generate egress traffic).
      * You'd need another cEOS or a host connected to `Ethernet1` or `Ethernet2` to see meaningful traffic.
      * For a simple test, configure an IP on `Ethernet1` (e.g., `10.0.0.1/24`) and ping an IP on that subnet that doesn't exist (e.g., `ping 10.0.0.2`). This will generate outbound packets.

### Challenge 5:

  * Modify `day5_subscribe_interface_counters.py` to subscribe to the CPU utilization of `ceos1` using an appropriate OpenConfig path (e.g., `openconfig-system:system/cpu/state/utilization`).
  * Experiment with `ONCE` and `POLL` subscription modes. For `POLL`, implement a loop that sends poll requests every 5 seconds.
