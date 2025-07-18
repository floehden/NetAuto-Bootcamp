## Day 8: Configuration Backup and Restore

### Concepts:

  * **Backup Configuration**: Using `gNMI Get` to retrieve the running configuration of the router. This usually involves querying the `config` datatype for relevant paths.
  * **Restore Configuration**: Using `gNMI Set` with the `replace` operation to apply a previously backed-up configuration. The `replace` operation is crucial here as it ensures the entire target path is overwritten, effectively "restoring" the state.
  * **Serialization**: Saving the configuration to a file (e.g., JSON) and loading it back.
  * **Considerations**: Understanding that a full configuration backup and restore via gNMI can be complex, as it depends on the YANG modules supported and whether the device's gNMI implementation exposes a complete configuration tree. For Arista cEOS, getting the *full* running configuration via OpenConfig might be challenging, so we'll focus on a subset of configuration or use native paths if available for a "full" config equivalent.

### Containerlab Setup (`day8_lab.yaml`):

Same as Day 1 or any single-node lab:

```yaml
name: day8_gnmi_backup_restore_lab
topology:
  nodes:
    ceos1:
      kind: ceos
      image: arista/ceos:latest
```

Deploy the lab:

```bash
sudo containerlab deploy --topo day8_gnmi_backup_restore_lab.yaml
```

**Important:** Get the `CEOS_IP` for `day8_gnmi_backup_restore_lab-ceos1` using `docker inspect`.

### Code Examples:

**1. `day8_backup_config.py`**
This script will back up the hostname and interface configurations to a JSON file.

```python
from pygnmi.client import gNMIclient
import json
import os

CEOS_IP = "172.20.0.2" # **UPDATE THIS WITH YOUR CEOS1 IP**
GNMI_PORT = 6030
USERNAME = "admin"
PASSWORD = "admin"
BACKUP_FILE = "ceos1_config_backup.json"

if __name__ == "__main__":
    with gNMIclient(
        target=(CEOS_IP, GNMI_PORT),
        username=USERNAME,
        password=PASSWORD,
        insecure=True
    ) as gc:
        print(f"Connecting to {CEOS_IP}:{GNMI_PORT} to backup configuration...")
        try:
            # Paths for configuration data (OpenConfig)
            # We'll back up hostname and all interface configurations.
            # Note: A truly "full" backup via gNMI is highly dependent on YANG model coverage.
            # For cEOS, you might need to query different OpenConfig modules or Arista native.
            # Example paths for config:
            config_paths = [
                "openconfig-system:system/config/hostname",
                "openconfig-interfaces:interfaces/interface/config",
                "openconfig-interfaces:interfaces/interface/openconfig-if-ethernet:ethernet/config/switched-vlan/config/vlan-mode", # To get no switchport
                "openconfig-interfaces:interfaces/interface/openconfig-interfaces:subinterfaces/subinterface/openconfig-ip:ipv4/addresses/address/config"
            ]

            backup_data = {}
            for path in config_paths:
                print(f"Getting config for path: {path}")
                result = gc.get(path=[path], encoding="json_ietf", datatype="config")
                if result and 'notification' in result and result['notification']:
                    for notification in result['notification']:
                        if 'update' in notification:
                            for update_item in notification['update']:
                                if 'path' in update_item and 'val' in update_item:
                                    # Store data with its full path for easy restoration
                                    # The path needs to be stringified for dictionary key
                                    full_path = "/".join([elem.get('name', '') if 'name' in elem else elem.get('elem', '') for elem in update_item['path']['elem'] if elem])
                                    # Clean up paths to match typical OpenConfig structure
                                    full_path = full_path.replace('openconfig-interfaces:interfaces', '/interfaces') \
                                                           .replace('openconfig-system:system', '/system') \
                                                           .replace('openconfig-if-ethernet:ethernet', '/ethernet') \
                                                           .replace('openconfig-interfaces:subinterfaces', '/subinterfaces') \
                                                           .replace('openconfig-ip:ipv4', '/ipv4')

                                    # Pygnmi returns structured paths, reconstruct a string representation
                                    # to use as a key in our backup JSON
                                    path_str = ""
                                    for elem in update_item['path']['elem']:
                                        if 'name' in elem:
                                            path_str += f"/{elem['name']}"
                                            if 'key' in elem:
                                                for k, v in elem['key'].items():
                                                    path_str += f"[{k}={v}]"
                                        elif 'elem' in elem:
                                            path_str += f"/{elem['elem']}"
                                    path_str = path_str.replace('/config', '/config') # Keep /config as part of the path if it's there
                                    path_str = path_str.replace('/state', '/state')

                                    # Remove leading / if it's there from the transformed path
                                    if path_str.startswith('/'):
                                        path_str = path_str[1:]

                                    backup_data[path_str] = update_item['val']
                                else:
                                    print(f"  Warning: No 'val' or 'path' in update item: {update_item}")
                else:
                    print(f"  No data received for path: {path}")

            with open(BACKUP_FILE, 'w') as f:
                json.dump(backup_data, f, indent=2)
            print(f"\nConfiguration backed up to {BACKUP_FILE}")

        except Exception as e:
            print(f"Error backing up configuration: {e}")

```

**2. `day8_restore_config.py`**
This script will read from the backup JSON file and apply the configuration using `gNMI Set` with `replace` for top-level paths, and `update` for specific values. *Note: Full `replace` of the entire config tree is usually not recommended or practical via gNMI directly, as it would delete unmanaged elements. We'll use `replace` on specific sub-trees or `update` for individual values.*

For simplicity and safety, this example uses `update` for most paths and `replace` only for a specific top-level element if needed (like an entire interface definition).

```python
from pygnmi.client import gNMIclient
import json
import os
import time

CEOS_IP = "172.20.0.2" # **UPDATE THIS WITH YOUR CEOS1 IP**
GNMI_PORT = 6030
USERNAME = "admin"
PASSWORD = "admin"
BACKUP_FILE = "ceos1_config_backup.json"

if __name__ == "__main__":
    if not os.path.exists(BACKUP_FILE):
        print(f"Error: Backup file '{BACKUP_FILE}' not found. Please run day8_backup_config.py first.")
        exit(1)

    with open(BACKUP_FILE, 'r') as f:
        backup_data = json.load(f)

    with gNMIclient(
        target=(CEOS_IP, GNMI_PORT),
        username=USERNAME,
        password=PASSWORD,
        insecure=True
    ) as gc:
        print(f"Connecting to {CEOS_IP}:{GNMI_PORT} to restore configuration from {BACKUP_FILE}...")
        try:
            updates = []
            replaces = [] # Use replaces for full object replacement if needed

            for path_str, value in backup_data.items():
                # Reconstruct the pygnmi path structure from the string key
                # This reconstruction is simplified and assumes a certain structure
                # A robust solution would need a more sophisticated path parsing.
                # For this example, we'll try to convert back to the 'elem' list format.

                # Example: "/interfaces/interface[name=Ethernet1]/config/description"
                # Needs to become: [{"name": "interfaces"}, {"name": "interface", "key": {"name": "Ethernet1"}}, {"elem": "config"}, {"elem": "description"}]

                # Let's simplify this by only handling specific known paths from our backup for 'update'
                # For a full config restoration, you'd iterate through the backup_data carefully
                # and construct 'update' or 'replace' operations based on the path structure.

                # A safer approach for a tutorial: apply as updates for specific config elements.
                # For example, to restore just hostname and a specific interface's config:
                if "/system/config/hostname" in path_str:
                    updates.append({
                        "path": "openconfig-system:system/config/hostname",
                        "value": value
                    })
                elif "/interfaces/interface" in path_str and "/config" in path_str:
                    # This part is tricky. A single 'update' might overwrite specific parts
                    # A 'replace' on a full interface might be better if you're sure you want
                    # to replace the entire interface config.
                    # Let's try to parse the path to get interface name and then form the update.
                    parts = path_str.split('/')
                    interface_name = None
                    for part in parts:
                        if part.startswith("interface[name="):
                            interface_name = part.split('=')[1].rstrip(']')
                            break
                    
                    if interface_name:
                        # Constructing specific OpenConfig paths for restoration
                        # This requires knowing the exact structure of the backed up data
                        # For example, if 'value' contains the whole interface config subtree
                        # then a 'replace' on "openconfig-interfaces:interfaces/interface[name=Ethernet1]"
                        # would be appropriate.
                        
                        # Given our backup strategy saves individual config items, we'll iterate
                        # through the backup_data and form individual updates.

                        # The logic here needs to be precise based on how the 'value' is stored
                        # in the backup. If it's a scalar value (like hostname or description),
                        # direct update works. If it's a complex JSON structure, you might need
                        # to replace the whole subtree.

                        # For demonstration, let's assume we want to restore simple scalar values
                        # like description and enable status. For IPs, it's more complex as they are keys.

                        # Instead of trying to reconstruct the entire path, let's target
                        # specific parts. For a true restore, the backup should be structured
                        # to allow `replace` operations on coherent config blocks.
                        pass # We'll handle this more granularly below.
                
            # Manual construction of updates for demonstration based on the backup_data format
            # This is more robust than trying to auto-reconstruct arbitrary paths.
            restoration_updates = []
            for full_oc_path, val in backup_data.items():
                # Reconstruct the pygnmi 'path' list of dictionaries
                # This is a critical step and needs to accurately reflect the YANG path.
                # Example: "system/config/hostname" -> [{"elem": "system"}, {"elem": "config"}, {"elem": "hostname"}]
                # Example: "interfaces/interface[name=Ethernet1]/config/description" -> [{"elem": "interfaces"}, {"elem": "interface", "key": {"name": "Ethernet1"}}, {"elem": "config"}, {"elem": "description"}]

                # Simple path parser: (This is a simplified parser and might need enhancement)
                parsed_path_elems = []
                current_path_str = ""
                i = 0
                while i < len(full_oc_path):
                    char = full_oc_path[i]
                    if char == '/':
                        if current_path_str:
                            parsed_path_elems.append({"elem": current_path_str})
                        current_path_str = ""
                    elif char == '[': # Handle keys
                        if current_path_str:
                            parsed_path_elems.append({"name": current_path_str}) # "name" for keyed list
                        key_str_start = i
                        while i < len(full_oc_path) and full_oc_path[i] != ']':
                            i += 1
                        key_str = full_oc_path[key_str_start+1:i] # "name=Ethernet1"
                        key_parts = key_str.split('=')
                        if len(key_parts) == 2:
                            parsed_path_elems[-1]['key'] = {key_parts[0]: key_parts[1]}
                        current_path_str = ""
                    else:
                        current_path_str += char
                    i += 1
                if current_path_str:
                    parsed_path_elems.append({"elem": current_path_str})

                # Prepend OpenConfig module names where appropriate (simplified)
                final_path_elems = []
                for elem_dict in parsed_path_elems:
                    if 'elem' in elem_dict:
                        if elem_dict['elem'] == 'system':
                            final_path_elems.append({"elem": "openconfig-system:system"})
                        elif elem_dict['elem'] == 'interfaces':
                            final_path_elems.append({"elem": "openconfig-interfaces:interfaces"})
                        elif elem_dict['elem'] == 'ethernet':
                            final_path_elems.append({"elem": "openconfig-if-ethernet:ethernet"})
                        elif elem_dict['elem'] == 'subinterfaces':
                            final_path_elems.append({"elem": "openconfig-interfaces:subinterfaces"})
                        elif elem_dict['elem'] == 'ipv4':
                            final_path_elems.append({"elem": "openconfig-ip:ipv4"})
                        else:
                            final_path_elems.append(elem_dict)
                    elif 'name' in elem_dict: # For keyed lists like 'interface'
                         if elem_dict['name'] == 'interface':
                            final_path_elems.append({"name": "openconfig-interfaces:interface", "key": elem_dict['key']})
                         elif elem_dict['name'] == 'address':
                            final_path_elems.append({"name": "openconfig-ip:address", "key": elem_dict['key']})
                         else:
                            final_path_elems.append(elem_dict)

                restoration_updates.append({
                    "path": final_path_elems,
                    "value": val
                })
            
            # Perform the set operation
            if restoration_updates:
                response = gc.set(update=restoration_updates, encoding="json_ietf")
                print("\n--- Restore Operation Response ---")
                print(json.dumps(response, indent=2))
                print("Configuration restoration initiated. Verifying...")
                time.sleep(5) # Give device time to apply config

                # Verify a key configuration element, e.g., hostname
                path_verify = ["openconfig-system:system/state/hostname"]
                result_verify = gc.get(path=path_verify, encoding="json_ietf", datatype="state")
                if result_verify and 'notification' in result_verify and result_verify['notification']:
                    for notification in result_verify['notification']:
                        if 'update' in notification:
                            for item in notification['update']:
                                if 'path' in item and 'val' in item:
                                    print(f"Verified Hostname after restore: {item['val']}")
            else:
                print("No updates generated for restoration.")

        except Exception as e:
            print(f"Error restoring configuration: {e}")

```

### Challenge 8:

1.  **Initial Configuration**: Before running `day8_backup_config.py`, manually configure some distinct settings on `ceos1` using its CLI:
      * Change the hostname (e.g., `ceos-backup-test`).
      * Add a description to `Ethernet1` (e.g., `Backup Test Interface`).
      * Configure an IP address on `Ethernet2` (e.g., `192.168.100.1/24`).
2.  **Backup**: Run `day8_backup_config.py`. Verify that `ceos1_config_backup.json` is created and contains the hostname and interface configuration you set.
3.  **Modify Configuration**: Manually change the hostname again (e.g., `ceos-modified`) and delete the description from `Ethernet1` via CLI.
4.  **Restore**: Run `day8_restore_config.py`. Observe the output.
5.  **Verify Restoration**: After the script completes, use `docker exec -it day8_gnmi_backup_restore_lab-ceos1 Cli` and verify that the hostname and Ethernet1 description have been restored to their state from the backup file. Check the IP address on Ethernet2 as well.

### Important Considerations for Backup and Restore:

  * **Full Configuration vs. Partial**: Achieving a *complete* configuration backup and restore via gNMI that exactly mirrors `show running-config` is incredibly complex due to the nature of YANG models. YANG typically defines data models, not a flat configuration text. Devices may not expose their entire configuration as a single, restorable gNMI tree. You might need to selectively backup and restore parts of the configuration.
  * **Vendor-Specifics**: Arista cEOS's gNMI implementation (and its underlying EOS-specific YANG models) might provide more direct ways to get/set the full running configuration, potentially via native paths (e.g., something like `eos_native:/Smash/Config/running-config` if it exists and is exposed). Always consult Arista's gNMI documentation.
  * **Idempotency**: `Set` operations with `update` are generally idempotent (applying them multiple times has the same effect). `Replace` is also idempotent in that it ensures the target path matches the provided value.
  * **Data Models**: A deep understanding of OpenConfig and/or vendor-specific YANG models is crucial for correctly constructing paths for backup and restoration. The path parsing logic in `day8_restore_config.py` is simplified and would need to be much more robust for a production system.

### Lab Teardown:

```bash
sudo containerlab destroy --topo day8_gnmi_backup_restore_lab.yaml
```
