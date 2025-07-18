# Day 6: Advanced Topics - Error Handling and Custom YANG Paths

### Concepts:

  * **Robust Error Handling**: Implementing `try-except` blocks for network operations.
  * **Custom/Native YANG Paths**: Working with vendor-specific YANG models and paths when OpenConfig is not sufficient or available. Arista's native paths are prefixed with `eos_native:`.
  * **Combining Operations**: Performing multiple Get/Set operations in a single script.

### Containerlab Setup (`day6_lab.yaml`):

```yaml
name: day6_gnmi_lab
topology:
  nodes:
    ceos1:
      kind: ceos
      image: arista/ceos:latest
    ceos2:
      kind: ceos
      image: arista/ceos:latest
  links:
    - endpoints: ["ceos1:eth1", "ceos2:eth1"]
```

Deploy the lab:

```bash
sudo containerlab deploy --topo day6_gnmi_lab.yaml
```

**Configure IPs on `ceos1` and `ceos2` for `eth1` in CLI:**

  * **ceos1 CLI**:
    ```
    enable
    configure terminal
    interface Ethernet1
    no switchport
    ip address 10.0.0.1/30
    end
    ```
  * **ceos2 CLI**:
    ```
    enable
    configure terminal
    interface Ethernet1
    no switchport
    ip address 10.0.0.2/30
    end
    ```

### Code Examples:

**1. `day6_robust_config_and_status.py`**
This script attempts to configure an IP on `Ethernet2` on both `ceos1` and `ceos2`, and then verifies their link status. It includes basic error handling.

```python
from pygnmi.client import gNMIclient
import json

CEOS1_IP = "172.20.0.2" # **UPDATE THIS WITH YOUR CEOS1 IP**
CEOS2_IP = "172.20.0.3" # **UPDATE THIS WITH YOUR CEOS2 IP**
GNMI_PORT = 6030
USERNAME = "admin"
PASSWORD = "admin"

DEVICES = {
    "ceos1": CEOS1_IP,
    "ceos2": CEOS2_IP
}

def configure_interface_ip(gc, device_name, interface_name, ip_address, prefix_length):
    print(f"\n--- Configuring {interface_name} on {device_name} with {ip_address}/{prefix_length} ---")
    update_path = [
        {
            "path": f"openconfig-interfaces:interfaces/interface[name={interface_name}]/openconfig-interfaces:subinterfaces/subinterface[index=0]/openconfig-ip:ipv4/config/enabled",
            "value": True
        },
        {
            "path": f"openconfig-interfaces:interfaces/interface[name={interface_name}]/openconfig-interfaces:subinterfaces/subinterface[index=0]/openconfig-ip:ipv4/addresses/address[ip={ip_address}]/config/ip",
            "value": ip_address
        },
        {
            "path": f"openconfig-interfaces:interfaces/interface[name={interface_name}]/openconfig-interfaces:subinterfaces/subinterface[index=0]/openconfig-ip:ipv4/addresses/address[ip={ip_address}]/config/prefix-length",
            "value": prefix_length
        }
    ]
    # For Arista, also need to remove switchport mode and ensure interface is up
    update_no_switchport = [
        {
            "path": f"openconfig-interfaces:interfaces/interface[name={interface_name}]/openconfig-if-ethernet:ethernet/config/switched-vlan/config/vlan-mode",
            "value": "openconfig-if-ethernet:VLAN_MODE_UNSET" # This might be vendor specific, remove switchport or set mode
        },
         {
            "path": f"openconfig-interfaces:interfaces/interface[name={interface_name}]/config/enabled",
            "value": True
        }
    ]

    try:
        # First try to set no switchport and enable
        gc.set(update=update_no_switchport, encoding="json_ietf")
        print(f"Set no switchport and enabled for {interface_name} on {device_name}.")
        # Then set IP address
        response = gc.set(update=update_path, encoding="json_ietf")
        print(f"Configuration response: {json.dumps(response, indent=2)}")
        print(f"Successfully configured {interface_name} on {device_name}.")
        return True
    except Exception as e:
        print(f"Failed to configure {interface_name} on {device_name}: {e}")
        return False

def get_interface_oper_status(gc, device_name, interface_name):
    print(f"\n--- Getting operational status of {interface_name} on {device_name} ---")
    path = [f"openconfig-interfaces:interfaces/interface[name={interface_name}]/state/oper-status"]
    try:
        result = gc.get(path=path, encoding="json_ietf", datatype="state")
        if result and 'notification' in result and result['notification']:
            for notification in result['notification']:
                if 'update' in notification:
                    for item in notification['update']:
                        if 'path' in item and 'val' in item:
                            status = item['val']
                            print(f"Operational status of {interface_name} on {device_name}: {status}")
                            return status
        print(f"Could not retrieve operational status for {interface_name} on {device_name}.")
        return None
    except Exception as e:
        print(f"Error getting operational status for {interface_name} on {device_name}: {e}")
        return None

if __name__ == "__main__":
    for device_name, ip_address in DEVICES.items():
        with gNMIclient(
            target=(ip_address, GNMI_PORT),
            username=USERNAME,
            password=PASSWORD,
            insecure=True
        ) as gc:
            # Configure IP on Ethernet2 (example)
            configure_interface_ip(gc, device_name, "Ethernet2", f"172.16.0.{1 if device_name == 'ceos1' else 2}", 24)
            # Get operational status of Ethernet1 (link between ceos1 and ceos2)
            get_interface_oper_status(gc, device_name, "Ethernet1")

```

### Challenge 6:

  * Using an Arista native gNMI path (e.g., something under `eos_native:/Smash/routing/status/route`), try to retrieve the routing table of `ceos1`.
  * Implement robust error handling in `day6_robust_config_and_status.py` to catch specific gRPC errors (e.g., `grpc.RpcError`) and provide more informative messages to the user.
  * Extend `day6_robust_config_and_status.py` to also configure an OSPF process on both `ceos1` and `ceos2` using `pygnmi` and OpenConfig paths (if supported by cEOS for OSPF config via gNMI). Verify the OSPF neighbor state.


