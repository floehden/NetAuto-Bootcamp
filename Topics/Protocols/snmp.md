# SNMP

SNMP stands for "Simple Network Management Protocol." It’s an application layer protocol included in the internet protocol suite, a set of the most commonly used communication protocols online.

SNMP originated in the 1980s at the time when organizational networks were growing in both size and complexity. Today, it’s one of the most widely accepted protocols for network monitoring. Here’s a look at how SNMP works and why it matters to network professionals.

## What is the SNMP Protocol used for?

SNMP is used to collect data related to network changes or to determine the status of network-connected devices. Collecting this data can help IT professionals keep their finger on the pulse of all their managed devices and applications. Every device within the network can be queried in real time with SNMP, TCP, and other types of probes for their performance metrics. When thresholds for certain values are exceeded, software can alert system administrators of the issue, allowing them to drill into the data and troubleshoot a solution.

## How SNMP Works?

SNMP works by sending messages, called protocol data units (PDUs), to devices within your network that “speak” SNMP. These messages are called SNMP Get-Requests. Using these requests, network administrators can track virtually any data values they specify. All of the information SNMP tracks can be provided to a product that asks for it. That product can either display or store the data, depending on an administrator’s preferences.

| Command      | What it Does                              | Who Uses It     |
| ------------ | ----------------------------------------- | --------------- |
| **GET**      | Retrieve a value (e.g., CPU usage)        | Manager → Agent |
| **GET-NEXT** | Retrieve the next value in a list/table   | Manager → Agent |
| **SET**      | Change/set a value on the device          | Manager → Agent |
| **TRAP**     | Unsolicited alert/notification            | Agent → Manager |
| **INFORM**   | Like TRAP, but requires acknowledgment    | Agent → Manager |
| **GET-BULK** | Efficiently retrieve lots of data at once | Manager → Agent |

image here

example uscase:

SNMP Manager sends a GET request for the interface status OID.

SNMP Agent receives the request and responds with the current value.

If something critical happens (e.g., link down), the SNMP Agent sends a TRAP to the manager immediately—no request needed.

## Reference 

https://www.fortra.com/resources/articles/snmp-basics-what-it-and-how-it-works
