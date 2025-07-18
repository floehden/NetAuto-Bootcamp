# Network Automation with Nornir

Nornir is a pure-Python automation framework designed for network engineers and operators. It provides an extensible, plugin-driven approach to inventory management, task execution, and parallel device interactions without requiring an external orchestrator or agent on target devices.

## Why Nornir?

* **Pythonic Workflow**: Everything is native Python—no DSL required.
* **Extensible Plugin System**: Swap inventory, connection, and task plugins (Netmiko, Scrapli, Napalm, etc.) without rewriting core logic.
* **Parallelism**: Built-in support for executing tasks concurrently across your inventory.
* **Reusability & Testing**: Break automation steps into Python functions, importable and testable in isolation.

## Core Concepts

1. **Inventory**

   * Defines **hosts**, **groups**, and **defaults**.
   * Managed via Inventory plugins (e.g., `SimpleInventory`, `YAMLInventory`).

2. **InitNornir**

   * Bootstraps the Nornir engine by loading inventory and defaults.

3. **Tasks & Plugins**

   * **Tasks** are Python functions that perform actions on devices (send CLI commands, retrieve telemetry, apply configs).
   * **Task Plugins** (e.g., `nornir_netmiko`, `nornir_scrapli`) provide pre-built tasks like `send_command` or `send_config`.

4. **Runner**

   * Invokes tasks across hosts, managing concurrency, result aggregation, and error handling.

5. **Results**

   * Each task run returns a structured result object (`MultiResult`), containing per-host `Result` entries with `.result`, `.failed`, and `.exception` fields.

## Example USage

Similar to ansible and netmiko example we will use nornior to do exact same thing! Backing up the nokia router

1. Register Inventory & Init Nornir: Load your nornir-simple-inventory.yml.
2. Generate Timestamp: One timestamp for all hosts.
3. For Each Host (backup_config):
4. Disable paging (screen-length 0 temporary)
5. Run bash cat /etc/opt/srlinux/config.json to grab the running config
6. Save to config_<hostname>_<YYYYMMDD_HHMMSS>.json
7. Run the Task across all hosts with nr.run(task=backup_config).

```bash
#!/usr/bin/env python3
"""
back-nornir.py

Use Nornir + Scrapli to backup SR Linux config with detailed logs.
"""

import datetime
from nornir import InitNornir
from nornir.core.plugins.inventory import InventoryPluginRegister
from nornir.plugins.inventory.simple import SimpleInventory
from nornir_scrapli.tasks import send_command

print("=== Nornir SR Linux Config Backup Script ===")

# 1) Register the SimpleInventory plugin so Nornir can find it by name
print("Registering SimpleInventory plugin...")
InventoryPluginRegister.register("SimpleInventory", SimpleInventory)
print("Plugin registered successfully.\n")

# 2) Create a shared timestamp for all output files
ts = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
print(f"Timestamp for this run: {ts}\n")

# 3) Initialize Nornir, pointing at your existing inventory file
inventory_file = "/home/netopschic/Documents/practicals/clab/clab-srlceos01/nornir-simple-inventory.yml"
print(f"Initializing Nornir using inventory file:\n  {inventory_file}\n")
nr = InitNornir(
    inventory={
        "plugin": "SimpleInventory",
        "options": {
            "host_file": inventory_file,
        },
    }
)
print("Nornir initialization complete.\n")

def backup_config(task):
    print(f"[{task.host.name}] ➤ Starting backup task")

    # 4) Disable paging so the device streams the full output
    print(f"[{task.host.name}] ➤ Disabling paging...")
    task.run(task=send_command, command="screen-length 0 temporary")
    print(f"[{task.host.name}] ✓ Paging disabled")

    # 5) Retrieve the JSON-config file from disk
    print(f"[{task.host.name}] ➤ Retrieving configuration JSON...")
    result = task.run(task=send_command, command="bash cat /etc/opt/srlinux/config.json")
    print(f"[{task.host.name}] ✓ Retrieved {len(result.result)} bytes of data")

    # 6) Save it locally
    filename = f"config_{task.host.name}_{ts}.json"
    print(f"[{task.host.name}] ➤ Saving configuration to {filename}...")
    with open(filename, "w") as fd:
        fd.write(result.result)
    print(f"[{task.host.name}] ✓ Configuration saved\n")

if __name__ == "__main__":
    print("▶ Running Nornir tasks...\n")
    nr.run(task=backup_config)
    print("▶ All tasks complete.")

```
Execute and install the plugins related to nornior

```bash
pip install nornir nornir-scrapli scrapli
python3 back-nornior.py
```
Output:

```bash
=== Nornir SR Linux Config Backup Script ===
Registering SimpleInventory plugin...
Plugin registered successfully.

Timestamp for this run: 20250717_151054

Initializing Nornir using inventory file:
  /home/netopschic/Documents/practicals/clab/clab-srlceos01/nornir-simple-inventory.yml

Nornir initialization complete.

▶ Running Nornir tasks...

[srl] ➤ Starting backup task
[srl] ➤ Disabling paging...
[srl] ✓ Paging disabled
[srl] ➤ Retrieving configuration JSON...
[srl] ✓ Retrieved 107479 bytes of data
[srl] ➤ Saving configuration to config_srl_20250717_151054.json...
[srl] ✓ Configuration saved

▶ All tasks complete.
```

The config file again will be saved in working directory.

## Reference

* Nornir Project: [https://nornir.readthedocs.io/en/latest/](https://nornir.readthedocs.io/en/latest/)
* GitHub Repository: [https://github.com/nornir-automation/nornir](https://github.com/nornir-automation/nornir)