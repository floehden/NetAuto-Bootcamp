### **Day 16: Configuration Management - Replacing Configurations**

**Objective:** Understand how to completely replace the device's configuration using `load_replace_candidate()`. This is a more drastic change, so use with caution in production.

**Concepts:**

  * `load_replace_candidate()`: Replaces the entire running configuration with the provided configuration.
  * **Caution:** This can be destructive if not used carefully, as it removes any configuration not explicitly defined in the candidate.

**Challenge:** Create a minimal configuration for `arista1` that only includes a hostname and the management interface configuration, and apply it using `load_replace_candidate()`. Then, add back the loopback interface from Day 3.

**Code Examples:**

1.  **`day4_replace_config.py`:**
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

    # Minimal replacement configuration
    replace_config_initial = """
    hostname arista-replaced
    interface Management0
      ip address YOUR_ARISTA1_IP/24 # Adjust subnet as per your Containerlab setup
      no shutdown
    """

    # Configuration to add back the loopback
    add_loopback_config = """
    interface Loopback1
      description "Managed by NAPALM - Added back"
      ip address 10.0.0.1/32
    """

    try:
        device.open()

        print("\n--- Replacing configuration with minimal config ---")
        device.load_replace_candidate(config=replace_config_initial)
        diff_initial = device.compare_config()
        print(diff_initial)

        if diff_initial:
            print("\nCommitting initial replacement configuration...")
            device.commit_config()
            print("Initial replacement committed. Waiting a moment...")
            time.sleep(5) # Give the device time to apply

            print("\n--- Adding back Loopback1 ---")
            device.load_merge_candidate(config=add_loopback_config)
            diff_loopback = device.compare_config()
            print(diff_loopback)

            if diff_loopback:
                print("\nCommitting loopback configuration...")
                device.commit_config()
                print("Loopback configuration committed.")
            else:
                print("No loopback changes, discarding.")
                device.discard_config()
        else:
            print("No changes for initial replacement, discarding.")
            device.discard_config()

    except Exception as e:
        print(f"Error: {e}")
        if device.is_opened:
            device.discard_config()
            print("Discarding candidate configuration due to error.")
    finally:
        if device.is_opened:
            device.close()
```
