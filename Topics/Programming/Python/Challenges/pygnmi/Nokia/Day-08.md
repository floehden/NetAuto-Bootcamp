## Day 8: Configuration Backup and Restore (Nokia SR Linux)

### Concepts:

  * **Backup Configuration**: Using `gNMI Get` to retrieve the running configuration of the router. This usually involves querying the `config` datatype for relevant paths.
  * **Restore Configuration**: Using `gNMI Set` with the `replace` operation to apply a previously backed-up configuration. The `replace` operation is crucial here as it ensures the entire target path is overwritten, effectively "restoring" the state.
  * **Serialization**: Saving the configuration to a file (e.g., JSON) and loading it back.
  * **Considerations**: SR Linux offers robust native YANG models for configuration. We'll leverage these to get more comprehensive configuration sections.

### Containerlab Setup (`day8_srl_lab.yaml`):

Same as Day 1 or any single-node lab:

```yaml
name: day8_srl_backup_restore_lab
topology:
  nodes:
    srl1:
      kind: srlinux
      image: ghcr.io/nokia/srlinux:latest
```

Deploy the lab:

```bash
sudo containerlab deploy --topo day8_srl_backup_restore_lab.yaml
```

**Important:** Get the `SRL_IP` for `day8_srl_backup_restore_lab-srl1` using `docker inspect`.

### Code Examples:

**1. `day8_srl_backup_config.py`**
This script will back up the hostname, system, and interface configurations to a JSON file.

```python
from pygnmi.client import gNMIclient
import json
import os
import time

SRL_IP = "172.20.0.2" # **UPDATE THIS WITH YOUR SRL1 IP**
GNMI_PORT = 57400
USERNAME = "admin"
PASSWORD = "admin"
BACKUP_FILE = "srl1_config_backup.json"

if __name__ == "__main__":
    print("Waiting for SR Linux to initialize gNMI for backup...")
    time.sleep(10) # Give SR Linux a moment

    with gNMIclient(
        target=(SRL_IP, GNMI_PORT),
        username=USERNAME,
        password=PASSWORD,
        insecure=True
    ) as gc:
        print(f"Connecting to {SRL_IP}:{GNMI_PORT} to backup configuration...")
        try:
            # Paths for configuration data (mixture of OpenConfig and native SR Linux)
            # For comprehensive backup, it's often better to query top-level modules.
            # SR Linux often makes its top-level config available via simple paths.
            config_paths = [
                "/system", # OpenConfig /system module
                "/interface", # OpenConfig /interface module
                "/network-instance", # OpenConfig /network-instance module (for VRFs, routing protocols)
                "/qos" # Example: if you have QoS configs
                # You might need to refine these or add more specific native paths
                # depending on what exact configuration you want to back up.
                # For SR Linux, '/network-instance[name=default]' is common for global configs.
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
                                    # The 'path' value in update_item already contains the full path elements.
                                    # We can directly use this to represent the data subtree.
                                    # For top-level paths, 'path' might be empty, and 'val' is the whole module.
                                    
                                    # Convert val to JSON, and store it.
                                    # A full configuration backup often means getting the *entire* tree
                                    # under a module.
                                    
                                    # The 'path' in update_item is a list of dicts. We convert it to string
                                    # for dictionary key, then store the value.
                                    path_str_elems = [elem.get('elem', '') for elem in update_item['path']['elem']]
                                    path_str_keys = []
                                    for elem in update_item['path']['elem']:
                                        if 'key' in elem:
                                            key_parts = []
                                            for k, v in elem['key'].items():
                                                key_parts.append(f"{k}={v}")
                                            path_str_keys.append(f"[{','.join(key_parts)}]")
                                        else:
                                            path_str_keys.append("") # Placeholder for non-keyed elements

                                    # Combine path elements and keys. This creates a representation
                                    # like "/interfaces/interface[name=ethernet-1/1]/..."
                                    full_structured_path = ""
                                    for i, elem_name in enumerate(path_str_elems):
                                        if elem_name:
                                            full_structured_path += f"/{elem_name}"
                                        if i < len(path_str_keys) and path_str_keys[i]:
                                            full_structured_path += path_str_keys[i]
                                    
                                    # If the path is empty, it means we got the root of the requested path
                                    # e.g., if we requested /system, the update path might be empty, and 'val' is the whole system config.
                                    if not full_structured_path and path: # Use the requested path as key
                                        backup_data[path[0]] = update_item['val']
                                    else: # Use the path from the update_item
                                        backup_data[full_structured_path] = update_item['val']
                                else:
                                    print(f"  Warning: No 'val' or 'path' in update item for path {path}: {update_item}")
                else:
                    print(f"  No data received for path: {path}")

            with open(BACKUP_FILE, 'w') as f:
                json.dump(backup_data, f, indent=2)
            print(f"\nConfiguration backed up to {BACKUP_FILE}")

        except Exception as e:
            print(f"Error backing up configuration: {e}")

```

**2. `day8_srl_restore_config.py`**
This script will read from the backup JSON file and apply the configuration. For restoration, we often use `replace` on high-level branches (like `/system` or `/interface[name=...]`) to ensure consistency.

```python
from pygnmi.client import gNMIclient
import json
import os
import time

SRL_IP = "172.20.0.2" # **UPDATE THIS WITH YOUR SRL1 IP**
GNMI_PORT = 57400
USERNAME = "admin"
PASSWORD = "admin"
BACKUP_FILE = "srl1_config_backup.json"

if __name__ == "__main__":
    if not os.path.exists(BACKUP_FILE):
        print(f"Error: Backup file '{BACKUP_FILE}' not found. Please run day8_srl_backup_config.py first.")
        exit(1)

    with open(BACKUP_FILE, 'r') as f:
        backup_data = json.load(f)

    with gNMIclient(
        target=(SRL_IP, GNMI_PORT),
        username=USERNAME,
        password=PASSWORD,
        insecure=True
    ) as gc:
        print(f"Connecting to {SRL_IP}:{GNMI_PORT} to restore configuration from {BACKUP_FILE}...")
        try:
            replaces_operations = []

            # For SR Linux, replacing entire modules or top-level containers works well.
            # The backup_data keys are already the paths we can use.
            for path_str, value_data in backup_data.items():
                # Reconstruct the pygnmi path structure from the string key.
                # This needs to handle both /module and /module/container[key=val]/...
                # Simple parser, assuming well-formed OpenConfig/native paths:
                path_elements = []
                current_path_segment = ""
                in_key = False
                key_dict = {}

                for i, char in enumerate(path_str):
                    if char == '/' and not in_key:
                        if current_path_segment:
                            if key_dict: # If previous segment had a key
                                path_elements[-1]['key'] = key_dict
                                key_dict = {}
                            path_elements.append({"elem": current_path_segment})
                        current_path_segment = ""
                    elif char == '[' and not in_key: # Start of a key
                        if current_path_segment: # Add the element before the key
                             path_elements.append({"elem": current_path_segment})
                        current_path_segment = ""
                        in_key = True
                    elif char == ']' and in_key: # End of a key
                        in_key = False
                        if '=' in current_path_segment:
                            k, v = current_path_segment.split('=', 1)
                            key_dict[k] = v.strip('"') # Remove quotes if present
                        # The element that the key belongs to is the LAST one added to path_elements
                        # If current_path_segment is the element name, add it first.
                        if path_elements and 'elem' in path_elements[-1]: # For non-keyed, like /system/hostname
                            path_elements[-1]['key'] = key_dict
                        elif path_elements and 'name' in path_elements[-1]: # For keyed elements like /interface[name=ethernet-1/1]
                             path_elements[-1]['key'] = key_dict
                        else: # New keyed element, e.g., the "interface" in /interface[name=x]
                            # This scenario is complex if the "name" part is already parsed as an element.
                            # A simpler, more robust approach might be to hardcode for top-level paths or use a proper YANG parser.
                            # For now, let's assume keys apply to the immediately preceding named element.
                            pass # Will refine below.
                        key_dict = {}
                        current_path_segment = ""
                    else:
                        current_path_segment += char
                
                # Add the last segment
                if current_path_segment:
                    path_elements.append({"elem": current_path_segment})
                    if key_dict:
                        path_elements[-1]['key'] = key_dict
                
                # The above parser is still simplified. For robustness, if you parse a
                # path like "/interfaces/interface[name=ethernet-1/1]", the pygnmi client
                # expects: [{"elem": "interfaces"}, {"name": "interface", "key": {"name": "ethernet-1/1"}}]
                # My simple parser above might produce: [{"elem": "interfaces"}, {"elem": "interface", "key": {"name": "ethernet-1/1"}}]
                # The 'name' vs 'elem' distinction for list elements is important.

                # Let's rebuild the path based on typical OpenConfig structure where applicable
                # A more practical approach for SR Linux is often to use the specific module as the path:
                # e.g., Set({"path": "/system", "value": {"hostname": "new-hostname"}})
                # So if backup_data stores the top-level structure, we can use that.

                # Simplified restore logic: Iterate over top-level module paths in backup_data
                # and replace them entirely. This assumes the backup_data for that path is the
                # full subtree to be restored.
                
                # Example: "/system" in backup_data maps to the entire system config.
                # Then we can do: replace = {"path": "/system", "value": backup_data["/system"]}
                
                # This needs careful construction of the `value` based on what the backup_data holds.
                # If backup_data contains flattened individual configs, we need to restructure.
                # If it holds full module subtrees, it's easier.

                # Assuming `backup_data` stores top-level modules as keys, and their values are
                # the JSON subtree for that module's config:
                if path_str.startswith("/"): # It's a top-level module
                    # For SR Linux, you can often replace an entire top-level config branch.
                    # This is the most reliable way to "restore" a module's config.
                    # Convert the string path back to pygnmi's list-of-dict path format.
                    # Example: "/system" -> [{"elem": "system"}]
                    # "/interfaces/interface[name=ethernet-1/1]" -> [{"elem": "interfaces"}, {"name": "interface", "key": {"name": "ethernet-1/1"}}]

                    pygnmi_path = []
                    parts = path_str.split('/')
                    for part in parts:
                        if not part: continue # Skip empty string from leading /
                        if '[' in part and ']' in part: # It's a keyed list element
                            name = part.split('[')[0]
                            key_str = part.split('[')[1].rstrip(']')
                            key_parts = {}
                            for k_v in key_str.split(','):
                                k, v = k_v.split('=')
                                key_parts[k] = v
                            pygnmi_path.append({"name": name, "key": key_parts})
                        else: # Simple element
                            pygnmi_path.append({"elem": part})

                    replaces_operations.append({
                        "path": pygnmi_path,
                        "value": value_data # This is the JSON subtree to replace with
                    })

            # Perform the set operation with 'replace'
            if replaces_operations:
                print(f"Attempting to restore {len(replaces_operations)} top-level configuration blocks...")
                response = gc.set(replace=replaces_operations, encoding="json_ietf")
                print("\n--- Restore Operation Response ---")
                print(json.dumps(response, indent=2))
                print("Configuration restoration initiated. Verifying...")
                time.sleep(5) # Give device time to apply config

                # Verify a key configuration element, e.g., hostname
                path_verify = ["/system/state/hostname"]
                result_verify = gc.get(path=path_verify, encoding="json_ietf", datatype="state")
                if result_verify and 'notification' in result_verify and result_verify['notification']:
                    for notification in result_verify['notification']:
                        if 'update' in notification:
                            for item in notification['update']:
                                if 'path' in item and 'val' in item:
                                    print(f"Verified Hostname after restore: {item['val']}")
            else:
                print("No replace operations generated for restoration.")

        except Exception as e:
            print(f"Error restoring configuration: {e}")

```

### Challenge 8:

1.  **Initial Configuration**: Before running `day8_srl_backup_config.py`, manually configure some distinct settings on `srl1` using its CLI (`/usr/bin/sr_cli`):
      * Change the hostname (e.g., `srl-backup-test`).
      * Add a description to `ethernet-1/1` (`set /interface ethernet-1/1 description "Backup Test Interface"`).
      * Configure an IP address on `ethernet-1/2` (e.g., `192.168.100.1/24`). Ensure it's enabled.
      * Add a static route: `set /network-instance[name=default]/static-routes/route[prefix=172.16.0.0/24]/next-hop[index=0]/ip-address 10.0.0.2` (assuming `10.0.0.2` is a reachable next-hop, e.g., `srl2`).
      * `commit and quit`
2.  **Backup**: Run `day8_srl_backup_config.py`. Verify that `srl1_config_backup.json` is created and contains the hostname, interface, and static route configuration you set. Note that the structure in the JSON might be nested, reflecting the YANG model.
3.  **Modify Configuration**: Manually change the hostname again (e.g., `srl-modified`), delete the description from `ethernet-1/1`, and delete the static route via CLI.
      * `enter candidate`
      * `set /system name srl-modified`
      * `delete /interface ethernet-1/1 description`
      * `delete /network-instance[name=default]/static-routes/route[prefix=172.16.0.0/24]`
      * `commit and quit`
4.  **Restore**: Run `day8_srl_restore_config.py`. Observe the output.
5.  **Verify Restoration**: After the script completes, use `docker exec -it day8_srl_backup_restore_lab-srl1 /usr/bin/sr_cli` and verify that the hostname, `ethernet-1/1` description, and the static route have been restored to their state from the backup file.

### Important Considerations for Backup and Restore:

  * **Full Configuration vs. Partial**: Nokia SR Linux, with its strong YANG adherence, is better suited for module-level backups than some other vendors. By querying `/system`, `/interface`, `/network-instance`, etc., you often get a complete subtree.
  * **Native vs. OpenConfig**: For full control and more stable paths, relying on Nokia's native YANG models (e.g., `srl_nokia-interfaces`, `srl_nokia-network-instance`) is often preferred for configuration, although OpenConfig is widely supported for state.
  * **Idempotency**: `replace` operations are powerful but must be used carefully. Replacing a top-level container means deleting everything under it first, then recreating with the provided JSON. Ensure your backup data for that path is truly comprehensive if using `replace`.
  * **Commit Behavior**: gNMI `Set` operations on SR Linux are transactional. If multiple updates/replaces are sent in one `Set` request, they are committed together.

### Lab Teardown:

```bash
sudo containerlab destroy --topo day8_srl_backup_restore_lab.yaml
```
