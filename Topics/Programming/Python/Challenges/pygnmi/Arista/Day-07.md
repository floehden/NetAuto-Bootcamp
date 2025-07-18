# Day 7: Comprehensive Automation Scenario and Cleanup

### Concepts:

  * **End-to-End Automation**: Combining all learned `pygnmi` operations to achieve a specific network automation goal.
  * **Modular Code**: Structuring your Python scripts for reususability.
  * **Lab Teardown**: Properly cleaning up Containerlab resources.

### Containerlab Setup (`day7_lab.yaml`):

```yaml
name: day7_gnmi_full_lab
topology:
  nodes:
    ceos1:
      kind: ceos
      image: arista/ceos:latest
    ceos2:
      kind: ceos
      image: arista/ceos:latest
    ceos3:
      kind: ceos
      image: arista/ceos:latest
  links:
    - endpoints: ["ceos1:eth1", "ceos2:eth1"]
    - endpoints: ["ceos2:eth2", "ceos3:eth1"]
    - endpoints: ["ceos1:eth2", "ceos3:eth2"] # Triangle topology
```

Deploy the lab:

```bash
sudo containerlab deploy --topo day7_gnmi_full_lab.yaml
```

Get IPs for `ceos1`, `ceos2`, `ceos3` using `docker inspect` or `clab inspect`.

### Code Examples:

**1. `day7_full_lab_automation.py`**
This script will:

1.  Configure interfaces on `ceos1`, `ceos2`, `ceos3`.
2.  Set up basic OSPF routing.
3.  Verify OSPF neighbor states.
4.  Continuously stream interface operational status.

<!-- end list -->

```python
from pygnmi.client import gNMIclient
import json
import time
import threading

# --- IMPORTANT: UPDATE WITH YOUR CEOS IPs ---
CEOS1_IP = "172.20.0.2"
CEOS2_IP = "172.20.0.3"
CEOS3_IP = "172.20.0.4"
# ---

GNMI_PORT = 6030
USERNAME = "admin"
PASSWORD = "admin"

DEVICES = {
    "ceos1": CEOS1_IP,
    "ceos2": CEOS2_IP,
    "ceos3": CEOS3_IP
}

def connect_gnmi(ip, port, username, password, insecure=True):
    return gNMIclient(target=(ip, port), username=username, password=password, insecure=insecure)

def configure_interface(gc, device_name, interface_name, ip_address, prefix_length):
    print(f"Configuring {interface_name} on {device_name} with {ip_address}/{prefix_length}")
    updates = [
        {
            "path": f"openconfig-interfaces:interfaces/interface[name={interface_name}]/openconfig-if-ethernet:ethernet/config/switched-vlan/config/vlan-mode",
            "value": "openconfig-if-ethernet:VLAN_MODE_UNSET"
        },
        {
            "path": f"openconfig-interfaces:interfaces/interface[name={interface_name}]/config/enabled",
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
    try:
        response = gc.set(update=updates, encoding="json_ietf")
        print(f"  {device_name} {interface_name} config response: {json.dumps(response, indent=2)}")
        return True
    except Exception as e:
        print(f"  Failed to configure {interface_name} on {device_name}: {e}")
        return False

def configure_ospf_interface(gc, device_name, interface_name, area_id):
    print(f"Configuring OSPF on {interface_name} on {device_name} in area {area_id}")
    # Note: OpenConfig for OSPF is complex and device-specific for full implementation.
    # Arista cEOS might require specific paths or use native YANG.
    # This is a simplified example based on common OpenConfig OSPF paths.
    # You may need to adapt these paths for specific cEOS versions/models.
    updates = [
        {
            "path": f"openconfig-network-instance:network-instances/network-instance[name=default]/openconfig-network-instance-types:PROTOTCOL_TYPE_OSPF/openconfig-ospf:ospfv2/global/config/router-id",
            "value": "1.1.1.1" if device_name == "ceos1" else ("2.2.2.2" if device_name == "ceos2" else "3.3.3.3")
        },
        {
            "path": f"openconfig-network-instance:network-instances/network-instance[name=default]/openconfig-network-instance-types:PROTOTCOL_TYPE_OSPF/openconfig-ospf:ospfv2/areas/area[identifier={area_id}]/interfaces/interface[id={interface_name}]/config/interface-id",
            "value": interface_name
        },
        {
            "path": f"openconfig-network-instance:network-instances/network-instance[name=default]/openconfig-network-instance-types:PROTOTCOL_TYPE_OSPF/openconfig-ospf:ospfv2/areas/area[identifier={area_id}]/interfaces/interface[id={interface_name}]/config/enabled",
            "value": True
        }
        # A more complete OSPF configuration would involve setting process ID, authentication, etc.
    ]
    try:
        response = gc.set(update=updates, encoding="json_ietf")
        print(f"  {device_name} OSPF config response: {json.dumps(response, indent=2)}")
        return True
    except Exception as e:
        print(f"  Failed to configure OSPF on {device_name}: {e}")
        return False

def get_ospf_neighbors(gc, device_name):
    print(f"Checking OSPF neighbors on {device_name}...")
    path = ["openconfig-network-instance:network-instances/network-instance[name=default]/openconfig-network-instance-types:PROTOTCOL_TYPE_OSPF/ospfv2/areas/area/interfaces/interface/neighbors/neighbor/state"]
    try:
        result = gc.get(path=path, encoding="json_ietf", datatype="state")
        if result and 'notification' in result and result['notification']:
            for notification in result['notification']:
                if 'update' in notification:
                    for update in notification['update']:
                        if 'path' in update and 'val' in update:
                            print(f"  {device_name} OSPF Neighbor: {json.dumps(update['val'], indent=2)}")
        else:
            print(f"  No OSPF neighbor data found on {device_name}.")
        return True
    except Exception as e:
        print(f"  Error getting OSPF neighbors on {device_name}: {e}")
        return False

def subscribe_interface_oper_status(gc, device_name, interface_name):
    print(f"Subscribing to {interface_name} operational status on {device_name}...")
    path = [f"openconfig-interfaces:interfaces/interface[name={interface_name}]/state/oper-status"]
    subscription_params = {
        "subscription": [
            {
                "path": path[0],
                "mode": "STREAM",
                "sample_interval": 5000000000 # 5 seconds
            }
        ]
    }
    try:
        for response in gc.subscribe2(subscribe=subscription_params):
            if 'update' in response:
                update = response['update']['update'][0]
                status = update['val'] if 'val' in update else update['value']
                print(f"  [{device_name} - {interface_name}] OPER STATUS: {status} at {time.strftime('%H:%M:%S')}")
            elif 'sync_response' in response and response['sync_response']:
                print(f"  {device_name} - {interface_name} subscription initial sync complete.")
            elif 'error' in response:
                print(f"  {device_name} - {interface_name} subscription error: {response['error']}")
    except Exception as e:
        print(f"  Error in {device_name} - {interface_name} subscription thread: {e}")

if __name__ == "__main__":
    # Define IP addresses for the links
    # ceos1-eth1 (10.0.0.1/30) - ceos2-eth1 (10.0.0.2/30)
    # ceos2-eth2 (10.0.0.5/30) - ceos3-eth1 (10.0.0.6/30)
    # ceos1-eth2 (10.0.0.9/30) - ceos3-eth2 (10.0.0.10/30)

    # Step 1: Configure Interfaces
    print("--- Day 7: Configuring Interfaces ---")
    with connect_gnmi(DEVICES["ceos1"], GNMI_PORT, USERNAME, PASSWORD) as gc1:
        configure_interface(gc1, "ceos1", "Ethernet1", "10.0.0.1", 30)
        configure_interface(gc1, "ceos1", "Ethernet2", "10.0.0.9", 30)

    with connect_gnmi(DEVICES["ceos2"], GNMI_PORT, USERNAME, PASSWORD) as gc2:
        configure_interface(gc2, "ceos2", "Ethernet1", "10.0.0.2", 30)
        configure_interface(gc2, "ceos2", "Ethernet2", "10.0.0.5", 30)

    with connect_gnmi(DEVICES["ceos3"], GNMI_PORT, USERNAME, PASSWORD) as gc3:
        configure_interface(gc3, "ceos3", "Ethernet1", "10.0.0.6", 30)
        configure_interface(gc3, "ceos3", "Ethernet2", "10.0.0.10", 30)

    time.sleep(5) # Give devices time to process config and bring up links

    # Step 2: Configure OSPF (simplified for demonstration, may need manual CLI for full config)
    print("\n--- Day 7: Configuring OSPF (Simplified) ---")
    with connect_gnmi(DEVICES["ceos1"], GNMI_PORT, USERNAME, PASSWORD) as gc1:
        configure_ospf_interface(gc1, "ceos1", "Ethernet1", 0) # Area 0
        configure_ospf_interface(gc1, "ceos1", "Ethernet2", 0)
    with connect_gnmi(DEVICES["ceos2"], GNMI_PORT, USERNAME, PASSWORD) as gc2:
        configure_ospf_interface(gc2, "ceos2", "Ethernet1", 0)
        configure_ospf_interface(gc2, "ceos2", "Ethernet2", 0)
    with connect_gnmi(DEVICES["ceos3"], GNMI_PORT, USERNAME, PASSWORD) as gc3:
        configure_ospf_interface(gc3, "ceos3", "Ethernet1", 0)
        configure_ospf_interface(gc3, "ceos3", "Ethernet2", 0)

    time.sleep(10) # Give OSPF time to converge

    # Step 3: Verify OSPF Neighbors
    print("\n--- Day 7: Verifying OSPF Neighbors ---")
    for device_name, ip_address in DEVICES.items():
        with connect_gnmi(ip_address, GNMI_PORT, USERNAME, PASSWORD) as gc:
            get_ospf_neighbors(gc, device_name)

    # Step 4: Stream Interface Operational Status in parallel threads
    print("\n--- Day 7: Streaming Interface Operational Status ---")
    subscription_threads = []
    for device_name, ip_address in DEVICES.items():
        # Subscribe to Ethernet1 status on all devices for cross-link visibility
        thread = threading.Thread(target=lambda: subscribe_interface_oper_status(
            connect_gnmi(ip_address, GNMI_PORT, USERNAME, PASSWORD), device_name, "Ethernet1"
        ))
        thread.daemon = True # Allow main program to exit even if threads are running
        subscription_threads.append(thread)
        thread.start()

    try:
        print("\nStreaming interface status for 60 seconds. Press Ctrl+C to exit early.")
        time.sleep(60) # Stream for 60 seconds
    except KeyboardInterrupt:
        print("\nStopping streaming and exiting.")
    finally:
        print("Automation complete. Cleaning up lab...")
        # Containerlab cleanup (manual step for the user)
        # sudo containerlab destroy --topo day7_gnmi_full_lab.yaml

```

### Challenge 7:

  * Run the `day7_full_lab_automation.py` script. Observe the configuration, OSPF neighbor output, and streaming telemetry.
  * While the script is running, manually shut down `Ethernet1` on `ceos1` via its CLI (`docker exec -it day7_gnmi_full_lab-ceos1 Cli` then `configure terminal`, `interface Ethernet1`, `shutdown`). Observe how the `pygnmi` script's streaming output reacts. Then bring it back up (`no shutdown`).
  * **Bonus**: Extend the `day7_full_lab_automation.py` script to include a function that retrieves and prints the `Arista-ext:router-id` for each cEOS device using a *native Arista gNMI path*.

### Lab Teardown:

After completing the tutorial, destroy your Containerlab environment:

```bash
sudo containerlab destroy --topo day7_gnmi_full_lab.yaml
```