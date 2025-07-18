
# **Day 15: Configuration Management - Merging Configurations**

**Objective:** Learn how to apply partial configurations to the cEOS router using `load_merge_candidate()` and `commit_config()`.

**Concepts:**

  * `load_merge_candidate()`: Merges a new configuration with the existing running configuration.
  * `commit_config()`: Applies the candidate configuration to the running configuration.
  * `compare_config()`: Shows the difference between the running and candidate configurations.
  * `discard_config()`: Discards the candidate configuration.

**Challenge:** Add a loopback interface and a static route to `arista1` using `load_merge_candidate()`.

**Code Examples:**

1.  **`day3_merge_config.py`:**
```python
    from napalm import get_network_driver
    import json

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

    # Configuration to merge
    new_config = """
    interface Loopback1
      description "Managed by NAPALM"
      ip address 10.0.0.1/32
    ip route 192.168.10.0/24 Null0 10
    """

    try:
        device.open()
        print("Loading merge candidate configuration...")
        device.load_merge_candidate(config=new_config)

        print("\n--- Configuration Difference ---")
        diff = device.compare_config()
        print(diff)

        if diff:
            print("\nCommitting configuration...")
            device.commit_config()
            print("Configuration committed successfully.")
        else:
            print("No changes detected, discarding candidate.")
            device.discard_config()

    except Exception as e:
        print(f"Error: {e}")
        # Important: Discard candidate config in case of error
        if device.is_opened:
            device.discard_config()
            print("Discarding candidate configuration due to error.")
    finally:
        if device.is_opened:
            device.close()
```
