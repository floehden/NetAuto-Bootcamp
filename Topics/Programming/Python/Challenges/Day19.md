# **Day 19: Building a Simple Automation Script & Cleanup**

**Objective:** Combine the learned concepts into a practical script and properly clean up your Containerlab environment.

## **Concepts:**

  * Scripting multiple NAPALM operations.
  * Containerlab lab destruction.

## **Challenge:**

1.  Create a Python script that:
      * Connects to `arista1`.
      * Retrieves and prints the running configuration.
      * Adds a new VLAN (e.g., VLAN 100, name "AUTOMATION\_VLAN").
      * Verifies the VLAN creation by retrieving facts or running configuration.
      * Removes the VLAN.
      * Verifies the VLAN removal.
      * Closes the connection.
2.  Destroy your Containerlab lab.

## **Code Examples:**

1.  **`day7_automation_script.py`:**

```python
from napalm import get_network_driver
import json
import time

DEVICE_IP = "YOUR_ARISTA1_IP"
USERNAME = "admin"
PASSWORD = "admin"

driver = get_network_driver("eos")
device = driver(
    hostname=DEVICE_IP,
    username=USERNAME,
    password=PASSWORD,
    optional_args={"secret": PASSWORD}
)

def print_section_header(title):
    print(f"\n{'='*5} {title} {'='*5}")

try:
    device.open()
    print("Connected to Arista cEOS.")

    print_section_header("Current Running Configuration")
    initial_config = device.get_config()['running']
    print(initial_config)

    # --- Add VLAN ---
    print_section_header("Adding VLAN 100")
    add_vlan_config = """
    vlan 100
        name AUTOMATION_VLAN
    """
    device.load_merge_candidate(config=add_vlan_config)
    diff_add_vlan = device.compare_config()
    print(diff_add_vlan)
    if diff_add_vlan:
        device.commit_config()
        print("VLAN 100 added. Verifying...")
        time.sleep(2) # Give device time to update state
        current_config_after_add = device.get_config()['running']
        if "vlan 100" in current_config_after_add and "name AUTOMATION_VLAN" in current_config_after_add:
            print("VLAN 100 successfully created.")
        else:
            print("VLAN 100 not found in config after add, potential issue.")
    else:
        print("No changes for VLAN 100, might already exist.")

    # --- Remove VLAN ---
    print_section_header("Removing VLAN 100")
    remove_vlan_config = """
    no vlan 100
    """
    device.load_merge_candidate(config=remove_vlan_config)
    diff_remove_vlan = device.compare_config()
    print(diff_remove_vlan)
    if diff_remove_vlan:
        device.commit_config()
        print("VLAN 100 removed. Verifying...")
        time.sleep(2) # Give device time to update state
        current_config_after_remove = device.get_config()['running']
        if "vlan 100" not in current_config_after_remove:
            print("VLAN 100 successfully removed.")
        else:
            print("VLAN 100 still found in config after remove, potential issue.")
    else:
        print("No changes for VLAN 100 removal, might not exist.")

except Exception as e:
    print(f"An error occurred during automation: {e}")
    if device.is_opened:
        device.discard_config() # Always good to discard if an error occurs during config changes
        print("Discarding candidate configuration due to error.")
finally:
    if device.is_alive():
        device.close()
        print("\nConnection closed.")

```

2.  **Destroy Containerlab lab:**

```bash
sudo containerlab destroy -t day1_topology.clab.yml --cleanup
```

    This will shut down and remove the Docker containers, cleaning up your lab environment.

## **Going Further:**

  * **Error Handling Refinements:** Implement more specific exception handling for different NAPALM errors.
  * **Data Validation:** After retrieving data, add checks to ensure the data is as expected.
  * **Configuration Templating:** Use Jinja2 to create dynamic configurations, especially for larger labs or different device roles.
  * **Inventory Management:** Integrate with a tool like Netbox or a simple YAML inventory file to manage device details (IPs, credentials).
  * **Automation Frameworks:** Explore how NAPALM integrates with frameworks like Ansible (via `napalm-ansible`) or Nornir for more complex automation workflows.
  * **Multiple Devices:** Expand your Containerlab topology to include multiple cEOS routers and practice automating configurations across them.
  * **Custom Getters/Setters:** For very specific, vendor-unique data or configurations not covered by NAPALM's standard getters/setters, you can extend the driver.
  * **NAPALM-CLI:** Experiment with the `napalm` command-line tool for quick checks.

  ## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%2019%20of%20the%20NetAuto%20Bootcamp%20on%20Python%20Programming!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @netauto-group-rheinmain

