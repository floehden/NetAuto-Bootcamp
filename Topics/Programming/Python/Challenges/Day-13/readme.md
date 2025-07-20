# Day 13 Setting up the Lab and First Connection**

## **Prerequisites:**

  * **Linux Environment:** A Linux machine (VM or bare metal) with Docker installed. Ubuntu 22.04 LTS or later is recommended.
  * **Docker:** Ensure Docker is installed and your user can run Docker commands without `sudo`. Refer to Docker's official documentation for installation.
  * **Containerlab:** Install Containerlab. The quick setup script `curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"` is highly recommended.
  * **Arista cEOS-lab Image:** Download the Arista cEOS-lab image (a `.tar.gz` or `.tar.xz` file) from the Arista support portal. You'll need an Arista account to access this. Import the image into Docker:
    ```bash
    docker import cEOS-lab-<VERSION>.tar.xz ceos:latest 
    # Or specify the exact version, e.g., docker import cEOS-lab-4.30.6M.tar.xz ceos:4.30.6M
    ```
  * **Python 3.9+:** Ensure you have Python 3.9 or higher installed.
  * **NAPALM:** Install NAPALM: `pip install napalm`
  * **Basic Python Knowledge:** Familiarity with Python syntax, data structures (dictionaries, lists), and functions.

## Tasks

## **Objective:** 
Get your Containerlab environment running with an Arista cEOS router and establish a basic NAPALM connection.

## **Concepts:**

  * Containerlab topology definition (`.clab.yml`)
  * Arista cEOS basic configuration (eAPI, SSH)
  * NAPALM `get_network_driver` and `open()` method

## **Challenge:** 
Deploy a single cEOS router, configure its management interface, and verify SSH/eAPI access.

**Code Examples:**

1.  **`day1_topology.clab.yml`:**

```yaml
name: arista-napalm-lab
topology:
  nodes:
    arista1:
      kind: ceos
      image: ceos:4.32.0F # Use the tag you imported your cEOS image with
      # You might need to add startup-config to enable eAPI and SSH, 
      # or configure it manually after the container starts.
      # For simplicity, we'll assume default cEOS enables SSH with admin/admin.
      # For eAPI, you might need to add:
      #startup-config: |
      #  management api http-commands
      #    no shutdown
      #    protocol https port 443
      #    protocol http port 80
      #  username admin secret admin
```

2.  **Deploy Containerlab lab:**

```bash
sudo containerlab deploy -t day1_topology.clab.yml --reconfigure
```

      * **Note:** Containerlab automatically assigns IP addresses to the management interface (`eth0`) of cEOS containers within its `clab` Docker network. You'll need to find this IP.
      * To find the IP: `docker inspect -f '{{ .NetworkSettings.Networks.clab.IPAddress }}' clab-arista-napalm-lab-arista1` (replace `clab-arista-napalm-lab-arista1` with your container's full name if it's different).

3.  **`day1_connect.py`:**

```python
from napalm import get_network_driver
import json

# Replace with the actual IP address of your arista1 cEOS container
# You can get this by running: 
# docker inspect -f '{{ .NetworkSettings.Networks.clab.IPAddress }}' clab-arista-napalm-lab-arista1
DEVICE_IP = "YOUR_ARISTA1_IP" 
USERNAME = "admin"
PASSWORD = "admin" # Default password for cEOS lab

# Arista EOS uses 'eos' driver
driver = get_network_driver("eos")
device = driver(
    hostname=DEVICE_IP,
    username=USERNAME,
    password=PASSWORD,
    optional_args={"secret": PASSWORD} # Required for Arista eAPI
)

print(f"Connecting to device {DEVICE_IP}...")
try:
    device.open()
    print("Connection successful!")

    # Example: Get device facts
    facts = device.get_facts()
    print("\n--- Device Facts ---")
    print(json.dumps(facts, indent=2))

except Exception as e:
    print(f"Error connecting or retrieving facts: {e}")
finally:
    if device.is_alive():
        device.close()
        print("Connection closed.")

```

## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%2013%20of%20the%20NetAuto%20Bootcamp%20on%20Python%20Programming!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @netauto-group-rheinmain