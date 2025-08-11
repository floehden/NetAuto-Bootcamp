# **Day 14: Retrieving Operational Data (Getters)**

## **Objective:** 
Explore NAPALM's "getter" methods to retrieve various operational states from the cEOS router.

## **Concepts:**

  * `get_facts()`
  * `get_interfaces()`
  * `get_interfaces_ip()`
  * `get_lldp_neighbors()`
  * `get_mac_address_table()`
  * `get_arp_table()`

## **Challenge:** 
Retrieve and print the IP addresses of all interfaces and the LLDP neighbors of `arista1`.

## **Code Examples:**

1.  **`day2_getters.py`:**
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

    print("\n--- Interfaces ---")
    interfaces = device.get_interfaces()
    print(json.dumps(interfaces, indent=2))

    print("\n--- Interfaces IP ---")
    interfaces_ip = device.get_interfaces_ip()
    print(json.dumps(interfaces_ip, indent=2))

    print("\n--- LLDP Neighbors ---")
    lldp_neighbors = device.get_lldp_neighbors()
    print(json.dumps(lldp_neighbors, indent=2))

    # Additional getters to explore:
    # print("\n--- MAC Address Table ---")
    # mac_table = device.get_mac_address_table()
    # print(json.dumps(mac_table, indent=2))

    # print("\n--- ARP Table ---")
    # arp_table = device.get_arp_table()
    # print(json.dumps(arp_table, indent=2))

except Exception as e:
    print(f"Error: {e}")
finally:
    if device.is_alive():
        device.close()
```
