# **Day 18: Advanced Operations & Error Handling**

## **Objective:** 
Explore more advanced NAPALM features and implement robust error handling.

## **Concepts:**

  * `ping()` and `traceroute()`
  * `get_users()`
  * Error handling with `try...except...finally` blocks.

## **Challenge:**

1.  Ping a non-existent IP address and handle the expected failure gracefully.
2.  Attempt to trace a route to `8.8.8.8`.
3.  Retrieve the configured users on `arista1`.

## **Code Examples:**

1.  **`day6_advanced_ops.py`:**
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

    try:
        device.open()

        print("\n--- Testing Ping (successful) ---")
        ping_result = device.ping("8.8.8.8") # Or an IP reachable from your cEOS
        print(json.dumps(ping_result, indent=2))

        print("\n--- Testing Ping (unsuccessful/non-existent IP) ---")
        try:
            ping_fail_result = device.ping("1.1.1.1") # Assuming this is not reachable
            print(json.dumps(ping_fail_result, indent=2))
        except Exception as e:
            print(f"Expected error during ping to non-existent IP: {e}")

        print("\n--- Testing Traceroute ---")
        try:
            traceroute_result = device.traceroute("8.8.8.8")
            print(json.dumps(traceroute_result, indent=2))
        except Exception as e:
            print(f"Error during traceroute: {e}")

        print("\n--- Getting Users ---")
        users = device.get_users()
        print(json.dumps(users, indent=2))

    except Exception as e:
        print(f"An unexpected error occurred during operation: {e}")
    finally:
        if device.is_opened:
            device.close()
    ```

## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%2018%20of%20the%20NetAuto%20Bootcamp%20on%20Python%20Programming!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @netauto-group-rheinmain