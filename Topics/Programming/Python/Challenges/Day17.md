
# **Day 17: Understanding Configuration Rollback**

## **Objective:** Learn how to use NAPALM's rollback feature in case of problematic configuration changes.

## **Concepts:**

  * `commit_config(revert_in=SECONDS)`: Commits changes, but automatically rolls back if a `commit_config()` with `confirmed=True` is not issued within `SECONDS`.
  * `rollback_config()`: Reverts the last committed configuration.

## **Challenge:**
Apply a configuration that would break connectivity (e.g., changing the management IP or shutting down an interface), use `revert_in` to automatically roll back, and then manually trigger a rollback for a different change.

## **Code Examples:**

1.  **`day5_rollback.py`:**
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

# Configuration to test revert_in
break_config = """
interface Management0
    shutdown
"""

# Configuration for manual rollback test
temp_config = """
interface Loopback100
    description "Temporary interface for rollback test"
    ip address 100.0.0.1/32
"""

try:
    device.open()

    print("\n--- Testing revert_in (auto-rollback) ---")
    print("Applying config to shutdown Management0. This should revert in 30 seconds.")
    device.load_merge_candidate(config=break_config)
    diff_break = device.compare_config()
    print(diff_break)

    if diff_break:
        revert_in = 30  # seconds
        device.commit_config(revert_in=revert_in)
        print("Configuration committed with auto-revert in 30 seconds. DO NOT RE-RUN YET.")
        print("You will lose connectivity and then it should come back.")
        time.sleep(revert_in + 5) # Wait for rollback to occur
        print("Connectivity should be restored now. Checking current config...")
        # Re-open connection as it might have dropped
        device.close()
        device.open()
        current_config_after_revert = device.get_config()['running']
        print("\n--- Running Config After Revert Attempt ---")
        print(current_config_after_revert)
        if "shutdown" in current_config_after_revert:
            print("ERROR: Management0 is still shut down! Revert failed.")
        else:
            print("SUCCESS: Management0 is no longer shut down.")

    else:
        print("No changes for break config, skipping auto-revert test.")

    print("\n--- Testing manual rollback ---")
    print("Applying a temporary interface for manual rollback.")
    device.load_merge_candidate(config=temp_config)
    diff_temp = device.compare_config()
    print(diff_temp)

    if diff_temp:
        device.commit_config()
        print("Temporary config committed. Verifying...")
        temp_int_check = device.get_interfaces_ip().get('Loopback100')
        if temp_int_check:
            print("Loopback100 exists. Now performing manual rollback...")
            device.rollback_config()
            print("Rollback initiated. Checking config again...")
            # Re-open connection to refresh state
            device.close()
            device.open()
            temp_int_check_after_rollback = device.get_interfaces_ip().get('Loopback100')
            if not temp_int_check_after_rollback:
                print("SUCCESS: Loopback100 removed by rollback.")
            else:
                print("ERROR: Loopback100 still present after rollback.")
        else:
            print("Temporary interface not found after commit, something went wrong.")
    else:
        print("No changes for temporary config, skipping manual rollback test.")


except Exception as e:
    print(f"Error: {e}")
    if device.is_alive():
        device.discard_config()
        print("Discarding candidate configuration due to error.")
finally:
    if device.is_alive():
        device.close()
```
