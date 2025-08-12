# Network Automation Protocols

Network automation relies on standard protocols to enable communication, configuration, monitoring, and management of network devices. Understanding these protocols is crucial for any network engineer or automation specialist.

## Course Overview

This course will guide you through the most important network automation protocols in today’s infrastructure. You’ll gain both conceptual understanding and hands-on experience with each protocol, so you can:

* **Automate repetitive tasks** and reduce human error
* **Integrate heterogeneous devices** into unified automation workflows
* **Leverage modern telemetry** for proactive network monitoring
* **Securely manage configurations** across your network

**Prerequisites:**

* Fundamental networking knowledge (IP, routing, switching)
* Comfort with CLI on network devices
* Basic scripting or programming experience is helpful but not required

---

## Table of Contents

| Day               | Topic    | Description                                                       |
| ----------------- | -------- | ----------------------------------------------------------------- |
| [Day 1](ssh.md) | SSH      | Secure CLI access; the foundation for remote network automation.  |
| [Day 2](netconf.md) | NETCONF  | Model-driven configuration using YANG and secure transactions.    |
| [Day 3](restconf.md) | RESTCONF | Modern RESTful APIs for config/monitoring with YANG over HTTP(S). |
| [Day 4](snmp.md) | SNMP     | Classic protocol for device monitoring and status polling.        |
| [Day 5](gnmi.md) | gNMI     | Streaming telemetry & automation via gRPC and Protocol Buffers.   |

---

## Lab Setup

In order to get hands-on with all of the protocols, we need to spin up a simple 1-router Containerlab with Nokia SR Linux.

### Topology

1. Save this YAML file:

```yaml
# clab-protocol-demo.clab.yaml
name: protocol-lab

topology:
  nodes:
    srl1:
      kind: srl
      image: ghcr.io/nokia/srlinux:latest
```

2. Deploy the lab:

```bash
containerlab deploy -t clab-ssh-demo.yaml
```

3. Inspect the topology to get the IP address:

```bash
containerlab inspect -t clab-ssh-demo.yaml
```

Deploy this lab to get complete hands-on experience with all the protocols.

---

## Additional Resources

* **gNMI Specification (OpenConfig)** – [https://www.openconfig.net/docs/gnmi/gnmi-specification/](https://www.openconfig.net/docs/gnmi/gnmi-specification/)
* **RFC 6241: NETCONF** – [https://datatracker.ietf.org/doc/html/rfc6241](https://datatracker.ietf.org/doc/html/rfc6241)
* **RFC 8040: RESTCONF** – [https://datatracker.ietf.org/doc/html/rfc8040](https://datatracker.ietf.org/doc/html/rfc8040)
* **Extreme Networks RESTCONF Guide** – [https://documentation.extremenetworks.com/restconf\_31.6/GUID-E6C98F14-2CE0-4103-B1B7-F7052ECBE364.shtml](https://documentation.extremenetworks.com/restconf_31.6/GUID-E6C98F14-2CE0-4103-B1B7-F7052ECBE364.shtml)
* **RFC 4253: SSH Transport Layer Protocol** – [https://datatracker.ietf.org/doc/html/rfc4253](https://datatracker.ietf.org/doc/html/rfc4253)
* **SNMP Basics: What It Is and How It Works (Fortra)** – [https://www.fortra.com/resources/articles/snmp-basics-what-it-and-how-it-works](https://www.fortra.com/resources/articles/snmp-basics-what-it-and-how-it-works)

---

# Final ToDo

Post about your journey and what you learned on platforms like LinkedIn, Twitter, or any other of your favourite platforms.
Follow up on your journey and share it with others!
Use the hashtags **#NetAutoBootcamp** **#NetworkAutomation**
You can also tag us on LinkedIn with **@netauto-group-rheinmain**
