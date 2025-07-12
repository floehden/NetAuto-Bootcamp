#  gNMI (gRPC Network Management Interface)

a gRPC-based protocol for the
modification and retrieval of configuration from a target device, as well as the
control and generation of telemetry streams from a target device to a data
collection system. The intention is that a single gRPC service definition can
cover both configuration and telemetry - allowing a single implementation on the
target, as well as a single NMS element to interact with the device via
telemetry and configuration RPCs.

gNMI is gRPC Network Management Interface developed by Google. gNMI provides the mechanism to
install, manipulate, and delete the configuration of network devices, and also to view operational data. The
content provided through gNMI can be modeled using YANG.
gRPC is a remote procedure call developed by Google for low-latency, scalable distributions with mobile
clients communicating to a cloud server. gRPC carries gNMI, and providesthe meansto formulate and transmit
data and operation requests.
When a gNMI service failure occurs, the gNMI broker (GNMIB) will indicate an operational change of state
from up to down, and all RPCs will return a service unavailable message until the database is up and running.
Upon recovery, the GNMIB will indicate a change of operation state from down to up, and resume normal
handling of RPCs

## How does gNMI work?

The keynoperation of gnmi to retrieve data from the device

| RPC Name      | Purpose                            | Initiator       |
| ------------- | ---------------------------------- | --------------- |
| **Get**       | Retrieve current state/config data | Client → Server |
| **Set**       | Change or update configuration     | Client → Server |
| **Subscribe** | Receive live telemetry streams     | Client ↔ Server |

gNMI Client: Usually a network controller, automation platform, or monitoring tool (e.g., OpenConfig collector, custom app).

gNMI Server: Runs on the network device (router, switch, etc.), exposing config and telemetry.

## Reference
https://www.openconfig.net/docs/gnmi/gnmi-specification/




