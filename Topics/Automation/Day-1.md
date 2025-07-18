# Device Automation with Netmiko

Multi-vendor library to simplify CLI connections to network devices

Network automation to screen-scraping devices is primarily concerned with gathering output from show commands and with making configuration changes.

## Why Netmiko?

Network automation to screen-scraping devices is primarily concerned with gathering output from show commands and with making configuration changes. Netmiko aims to accomplish both of these operations and to do it across a very broad set of platforms. It seeks to do this while abstracting away low-level state control (i.e. eliminate low-level regex pattern matching to the extent practical)

This is where the basics starts and in order to get complete hands -on will need virtual lab where you can configure some devices. This lab will use containerlab.

## Install and setup Containerlab

This will quickly install clab:
```bash
curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"
```
### Topology
1. Save this yaml file

```yaml
# clab-protocol-demo.clab.yaml
name: protocol-lab

topology:
  nodes:
    srl1:
      kind: srl
      image: ghcr.io/nokia/srlinux:latest
```
2. Deploy the lab

```bash
containerlab deploy -t clab-ssh-demo.yaml
```
3. Inspect the topology to get the ip address

```bash
containerlab inspect -t clab-ssh-demo.yaml
```

Deploy this lab to get a complete hands on netmiko lab

## Example Usage

This is a simple script that uses netmiko to connect to a nokia device and backup running configuration

1. Define connection details
2. Generate a timestamp
3. Connect via Netmiko
4. Retrieve the running config
5. Save it to a file named config_backup_<timestamp>.txt
6. Disconnect


```python
#!/usr/bin/env python3
# simple_backup_nokia_srl.py
# Beginner-friendly script to backup Nokia SR Linux configuration (with paging disabled)

import datetime
from netmiko import ConnectHandler

# 1. Define the router connection details
#    Updated for your SR Linux containerlab instance
device = {
    'device_type': 'nokia_srl',    # Nokia SR Linux driver
    'host': '172.20.20.2',          # Router IP address
    'username': 'admin',            # Login username
    'password': 'NokiaSrl1!',       # Login password
}

# 2. Build a timestamp for file naming
now = datetime.datetime.now()
timestamp = now.strftime('%Y%m%d_%H%M%S')  # e.g. 20250717_162530

# 3. Connect to the router over SSH
print('Connecting to the router...')
connection = ConnectHandler(**device)

# 4. Disable paging so the full output returns in one go
print('Disabling paging...')
connection.send_command_timing('screen-length 0 temporary')

# 5. Ask the router to show its configuration file in JSON
print('Retrieving configuration JSON...')
config_output = connection.send_command_timing('bash cat /etc/opt/srlinux/config.json')

# 6. Save the configuration to a file
filename = f'config_backup_{timestamp}.json'
print(f'Saving configuration in {filename}...')
with open(filename, 'w') as file:
    file.write(config_output)

# 7. Close the SSH connection
connection.disconnect()
print('Done! Router configuration backed up.')
```
Output shall look like this, the config file will be saved in working directory.

```bash
Connecting to the router...
Disabling paging...
Retrieving configuration...
Saving configuration in config_backup_20250717_140101.txt...
Done! Router configuration backed up.
```

## Challenge

Write a script which:
1. Use input() to ask for your SR-OS “show” command.

2. Connect with Netmiko, disable paging (screen-length 0 temporary), run the command with send_command_timing(), print the output, and disconnect.
---
## Reference

- Netmiko GitHub Repository: https://github.com/ktbyers/netmiko
- Official Netmiko Documentation: https://netmiko.readthedocs.io/en/latest/