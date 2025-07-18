
## Day 2: Reading Operational Data with gNMI Get (Nokia SR Linux)

### Concepts:

  * **Get RPC**: Retrieving current state and configuration data from the device.
  * **gNMI Paths**: Specifying data points using OpenConfig or native Nokia SR Linux YANG paths.
  * **Encoding**: Understanding JSON, JSON\_IETF, PROTO, ASCII encodings.

### Containerlab Setup (`day2_srl_lab.yaml`):

Same as Day 1. You can reuse the lab or destroy/redeploy.

### Code Examples:

**1. `day2_srl_get_interfaces.py`**
This script gets operational state of all interfaces.

```python
from pygnmi.client import gNMIclient
import json

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
        print(f"Connecting to {SRL_IP}:{GNMI_PORT} to get interface state...")
        try:
            # OpenConfig path for all interfaces' state data
            # SR Linux often exposes more granular data under specific interface paths.
            # Let's get summary interface state.
            path = ["/interfaces/interface/state"] # OpenConfig path

            result = gc.get(path=path, encoding="json_ietf", datatype="state")
            print("\n--- Interface Operational State ---")

            if result and 'notification' in result and result['notification']:
                for notification in result['notification']:
                    if 'update' in notification:
                        for update in notification['update']:
                            # For json_ietf, 'val' contains the JSON data
                            # The 'path' element can be quite verbose, let's simplify.
                            if 'path' in update and 'val' in update:
                                interface_name = None
                                for elem in update['path']['elem']:
                                    if 'key' in elem and 'name' in elem['key']:
                                        interface_name = elem['key']['name']
                                if interface_name:
                                    print(f"Interface: {interface_name}")
                                    print(json.dumps(update['val'], indent=2))
                                else:
                                    print(f"Path: {update['path']['elem']}")
                                    print(json.dumps(update['val'], indent=2))
            else:
                print("No data received for interfaces.")
        except Exception as e:
            print(f"Error getting interfaces: {e}")

```

**2. `day2_srl_get_system_hostname.py`**
This script gets the system hostname.

```python
from pygnmi.client import gNMIclient
import json

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
        print(f"Connecting to {SRL_IP}:{GNMI_PORT} to get system hostname...")
        try:
            # OpenConfig path for system hostname (config and state)
            path = ["/system/state/hostname"] # OpenConfig path
            result = gc.get(path=path, encoding="json_ietf", datatype="state")
            print("\n--- System Hostname ---")
            if result and 'notification' in result and result['notification']:
                for notification in result['notification']:
                    if 'update' in notification:
                        for update in notification['update']:
                            if 'path' in update and 'val' in update:
                                # The hostname is a simple string value
                                print(f"Hostname: {update['val']}")
            else:
                print("No hostname data received.")
        except Exception as e:
            print(f"Error getting hostname: {e}")

```

### Challenge 2:

  * Run both `day2_srl_get_interfaces.py` and `day2_srl_get_system_hostname.py`.
  * Find the Nokia SR Linux native gNMI path for interface statistics (e.g., `srl_nokia-interfaces:interface/statistics`). You can explore SR Linux documentation or the SR Linux CLI with `info from state <path>`.
  * Modify `day2_srl_get_interfaces.py` to fetch specific statistics (e.g., `in-octets`, `out-octets`) for `ethernet-1/1` using an SR Linux native path.


