#  gNMI (gRPC Network Management Interface)

gRPC-based protocol for the
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

## Example usage

Priya is a network automation engineer working with hundreds of routers in a cloud data center. She needs to monitor interface status and push new configurations quickly and reliably. Instead of using traditional CLI scripts, Priya leverages **gNMI**—a high-performance, model-driven protocol for managing devices at scale.

gNMI allows Priya to:
- Retrieve configuration and operational data (GET).
- Push configuration changes (SET).
- Subscribe to real-time telemetry (SUBSCRIBE).
- Use a single, secure, and efficient protocol for all tasks.

To query interface status on a Nokia SR Linux router, Priya uses the [OpenConfig gNMI CLI tool](https://github.com/openconfig/gnmi):

### 1. Install gNMI CLI

```bash
bash -c "$(curl -sL https://get-gnmic.openconfig.net)"
```
### 2. SSH into the device and enable gnmi server
```bash
A:admin@srl# enter candidate

# 1) Enable the gNMI instance
set system grpc-server gnmi admin-state enable

# 2) Tell it which services to run (the ‘gnmi’ service)
set system grpc-server gnmi services [gnmi]

# 3) Bind it to port 57400
set system grpc-server gnmi port 57400

# 4) (Optional if you’ve already got a cert bound)  
#    Ensure it’s using the default self-signed profile
set system grpc-server gnmi tls-profile default-tls-profile

commit stay
```

### 3. Run a GET Request

```bash
gnmic -a 172.20.20.2:57401 \
      -u admin \
      -p NokiaSrl1! \
      --insecure \
      get --path /interface \
      --encoding ASCII
```
