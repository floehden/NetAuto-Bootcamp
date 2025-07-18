## Day 5: Streaming Telemetry with gNMI Subscribe (Nokia SR Linux)

### Concepts:

  * **Subscribe RPC**: Receiving continuous streams of data from the device.
  * **Subscription Modes**:
      * **STREAM**: Continuous stream of updates as they occur.
      * **ONCE**: A single snapshot of data, then terminates.
      * **POLL**: On-demand snapshots triggered by the client.
  * **QoS (Quality of Service)**: Setting DSCP values for telemetry traffic.

### Containerlab Setup (`day5_srl_lab.yaml`):

```yaml
name: day5_srl_gnmi_lab
topology:
  nodes:
    srl1:
      kind: srlinux
      image: ghcr.io/nokia/srlinux:latest
      # No special configuration needed for subscribe, as gNMI is enabled by default.
```

### Code Examples:

**1. `day5_srl_subscribe_interface_counters.py` (STREAM mode)**
This script subscribes to interface counter updates and prints them as they change.

```python
from pygnmi.client import gNMIclient
import json
import time

SRL_IP = "172.20.0.2" # **UPDATE THIS WITH YOUR SRL1 IP**
GNMI_PORT = 57400
USERNAME = "admin"
PASSWORD = "admin"

if __name__ == "__main__":
    with gNMIclient(
        target=(SRL_IP, GNMI_PORT),
        username=USERNAME,
        password=PASSWORD,
        insecure=True
    ) as gc:
        print(f"Connecting to {SRL_IP}:{GNMI_PORT} to subscribe to interface counters (STREAM)...")
        try:
            # OpenConfig path for interface statistics (counters).
            # For SR Linux, you might find more comprehensive data using native paths.
            # Example OpenConfig path for a specific interface's statistics:
            # path = ["/interfaces/interface[name=ethernet-1/1]/state/statistics"]

            # Let's use a native SR Linux path for more comprehensive counters
            # or try openconfig path for all interfaces' state/statistics.
            # SR Linux usually provides counters under /state/statistics.
            path = ["/interfaces/interface/state/statistics"] # OpenConfig path
            # Alternatively, native SR Linux path example:
            # path = ["srl_nokia-interfaces:interface[name=ethernet-1/1]/statistics"]

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
                    for update_data in response['update']['update']:
                        if 'path' in update_data and 'val' in update_data:
                            interface_name = None
                            for elem in update_data['path']['elem']:
                                if 'key' in elem and 'name' in elem['key']:
                                    interface_name = elem['key']['name']
                            if interface_name:
                                print(f"Interface: {interface_name} Counters:")
                                print(json.dumps(update_data['val'], indent=2))
                            else:
                                print(f"Path: {update_data['path']['elem']}")
                                print(json.dumps(update_data['val'], indent=2))
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
After running the subscription script, you can generate traffic on `srl1` by:

1.  Accessing the SR Linux CLI: `docker exec -it day5_srl_gnmi_lab-srl1 /usr/bin/sr_cli`
2.  Configure an IP on `ethernet-1/1` (e.g., `10.0.0.1/24`) if you haven't.
3.  Ping an IP on that subnet that doesn't exist (e.g., `ping 10.0.0.2`). This will generate outbound packets and should trigger counter updates.

### Challenge 5:

  * Modify `day5_srl_subscribe_interface_counters.py` to subscribe to the CPU utilization of `srl1` using an appropriate OpenConfig path (e.g., `/system/cpu/state/utilization`).
  * Experiment with `ONCE` and `POLL` subscription modes. For `POLL`, implement a loop that sends poll requests every 5 seconds.

