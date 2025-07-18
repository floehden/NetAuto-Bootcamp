# Automating Networks with Ansible

Ansible is an open-source IT automation engine that lets you describe your infrastructure as code and automate virtually any task—across cloud, on-premises, and network devices—without installing agents on the managed nodes.

## Why Ansible?

* **Agentless Architecture**: Uses SSH (Linux/Unix) or WinRM (Windows) to connect, with no persistent agent required on managed hosts.
* **Simplicity**: Human-readable YAML playbooks describe desired state.
* **Extensible**: Thousands of built-in modules and a rich plugin ecosystem.
* **Scalable**: Push-based execution from a centralized control node.

## Core Components

1. **Control Node**: Machine running Ansible CLI (`ansible`, `ansible-playbook`).
2. **Inventory**: Static (INI/YAML) or dynamic lists of managed hosts.
3. **Modules**: Discrete units of work (file copy, package install, configuration tasks) executed remotely.
4. **Playbooks**: YAML files defining one or more plays that map hosts to ordered tasks.
5. **Roles**: Reusable collections of tasks, variables, templates, and files.

## Example Usage

Ansible is user friendly and can be used in day to day life of a network/devops engineer. 

Most common use case would be configuration of routers and other network devices. Updating the software usually when the devices are alot.

In this example we fetch the running configuration of nokia router and save it in local directory

```yaml
# backup_srlinux_config.yml
---
- name: Backup SR Linux Running Configuration
  hosts: nokia
  gather_facts: no
  connection: ansible.netcommon.httpapi

  vars:
    ansible_network_os: nokia.srlinux.srlinux
    # generate a timestamp on the control host
    timestamp: "{{ lookup('pipe', 'date +%Y%m%d_%H%M%S') }}"

  tasks:
    - name: Retrieve full running configuration
      nokia.srlinux.get:
        paths:
          - path: "/"            # root of the data tree
            datastore: running
      register: running_cfg

    - name: Save running config to timestamped file on controller
      copy:
        content: "{{ running_cfg.result[0] }}"
        dest: "config_backup_{{ timestamp }}.txt"
      delegate_to: localhost
      run_once: true

```
Install the ansible galaxy collection of Nokia SR linux and Execute it by,

```bash
ansible-galaxy collection install nokia.srlinux
ansible-playbook -i inventory.ini backup_srlinux_config.yml
```
Output:

```bash
PLAY [Backup SR Linux Running Configuration] *****************************************************************************************

TASK [Retrieve full running configuration] *******************************************************************************************
ok: [clab-srlceos01-srl]

TASK [Save running config to timestamped file on controller] *************************************************************************
changed: [clab-srlceos01-srl -> localhost]

PLAY RECAP ***************************************************************************************************************************
clab-srlceos01-srl         : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

## Common Use Cases

* **Configuration Management**: Ensure system packages, services, and configuration files are consistent.
* **Application Deployment**: Automate multi-tier app rollout and updates.
* **Orchestration**: Coordinate tasks across multiple systems (e.g., rolling restarts).
* **Network Automation**: Push configurations, gather state, and integrate with device APIs.

---
## Challenge

1. Use ansible to configure nginx on ubuntu machine
2. Use ansible to update packages.

## Reference

* Official Ansible Documentation: [https://docs.ansible.com/ansible/latest/index.html](https://docs.ansible.com/ansible/latest/index.html)

