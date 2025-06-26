# Addressing and DHCP

IN the work of networking, just like our world for data to reach its correct destination addressing is a must.

There are 2 kind of addressing
1. Physical address (Permananet)
2. Logical address (Temporary)

## Physcial Addressing

This address is permanent and also unique. It can also be known as MAc address, LAN card address, Hardware address, Network adapter address, Machine address, Burin in (Bia) etc.

## Logical Address

This temporary kind of address and also LAyer 3 kind of addressing which is the Network layer on OSI and TCP/IP model. 

### IP addressing

IP address or internal protocol address is the address which is used on "internet" level. An IP address is a numerical identifier. The computer network to which it is assigned uses it to communicate with the outside world. It’s also used to identify a host or network interface and display its location. There are 2 kind of IP version IPv4 and IPV6

IPv4 is 32 bit and IPv6 is 128 bit which is quiet complex as compared to IPV4 and Due to IPv4 address exhaustion, the networking world has increasingly adopted IPv6, which provides an enormous address space (2128 addresses). IPv6 does not use the class-based system at all. Instead, it focuses on hierarchical addressing and aggregation for routing efficiency. 

IP addressing is further divided into 2 parts

1. Private ip
2. Public ip

#### Private IP Address

The network router assigns a private IP address to the devices connected to it. This allows devices from the same network to communicate and interact with each other — all without connecting to the internet. With the help of private IP addresses, a router can forward data traffic on its own network.

This means private addresses help to increase security within a network. To give a practical example: When using your wireless connection at home, you can send documents from your computer to your printer for printing — all without the risk of outsiders gaining access to them.

#### Public IP Address

ISPs (internet service providers) generally assign each network and the associated router a public address. It’s used to establish a connection to the internet and communicate outside of your own network. As such, it’s required for you to go online.

The public address is accessible from around the world and helps other devices communicate with your network. In short, a public IP address ensures content from websites, emails, and streaming providers as well as other data from the internet is correctly forwarded to you.

#### IP4 Classes

In the IPv4 IP address space, there are five classes: A, B, C, D and E. Each class has a specific range of IP addresses (and ultimately dictates the number of devices you can have on your network). Primarily, class A, B, and C are used by the majority of devices on the Internet. Class D and class E are for special uses.

The list below shows the five available IP classes, along with the number of networks each can support and the maximum number of hosts (devices) that can be on each of those networks. The four octets that make up an IPv4 address are conventionally represented by a.b.c.d - such as 127.10.20.30.

| Class | First Octet Range | Default Subnet Mask | Network : Host Bits | # of Networks | Hosts per Network  | Typical Use         |
| :---: | :---------------: | :-----------------: | :-----------------: | :-----------: | :----------------- | :------------------ |
|   A   |      1 – 126      |      255.0.0.0      |        8 : 24       |      126      | 2²⁴–2 = 16 777 214 | Very large networks |
|   B   |     128 – 191     |     255.255.0.0     |       16 : 16       |     16 384    | 2¹⁶–2 = 65 534     | Medium networks     |
|   C   |     192 – 223     |    255.255.255.0    |        24 : 8       |   2 097 152   | 2⁸–2 = 254         | Small networks      |
|   D   |     224 – 239     |         n/a         |         n/a         |      n/a      | n/a                | Multicast groups    |
|   E   |     240 – 254     |         n/a         |         n/a         |      n/a      | n/a                | Experimental        |

####  Classless Inter-Domain Routing (CIDR)

CIDR replaces the rigid structure of IP classes by allowing subnetting based on arbitrary bit-length prefixes. For example, instead of assigning a full Class C block (which includes 256 IP addresses), a network administrator can allocate only 32 IPs using a CIDR block like 192.168.1.0/27.

#### Subnetting

A subnet is a physical segment of a network in which IP addresses with the same network address are used. These subnets can be connected to each other via routers and then form a large coherent network. There are not enough IP addresses. Subnetting allows networks to be separated from each other and private IP addresses can also be assigned twice.

NEED TO THINK

## DHCP

Dynamic Host Configuration Protocol  automatically configures network devices with IP address configuration information. This can help by simplifying and centralizing the allocation of IP configurations. Without DHCP, each time  a client added to a network, it needs to be configured its network interface with information about the network its been connected to. It removes the burden of configuring IP address manually. It reduces human error and efforts.

### How DHCP works

The four steps in lease generation are:

The DHCP client broadcasts a DHCPDISCOVER packet. The only computers that respond are computers that have the DHCP Server role, or computers or routers that are running a DHCP relay agent. In the last case, the DHCP relay agent forwards the message to the DHCP server that you have configured to relay requests.
A DHCP Server responds with a DHCPOFFER packet, which contains a potential address for the client. If multiple DHCP servers receive the DHCPDISCOVER packet, then multiple DHCP servers can respond.
The client receives the DHCPOFFER packet. If the client receives multiple DHCPOFFER packets, it selects the first response. The client then sends a DHCPREQUEST packet that contains a server identifier. This informs the DHCP servers that receive the broadcast which server’s DHCPOFFER the client has chosen to accept.
The DHCP servers receive the DHCPREQUEST. Servers that the client hasn't accepted use this message as the notification that the client has declined that server’s offer. The chosen server stores the IP address-client information in the DHCP database and responds with a DHCPACK message. If the DHCP server can't provide the address that was offered in the initial DHCPOFFER, the DHCP server sends a DHCPNAK message.

### DHCP lease renewal

When the DHCP lease reaches 50 percent of the lease time, the client automatically attempts to renew the lease. This process occurs in the background. It's possible for a computer to have the same DHCP-assigned IP address for a long time. This is because the computer renews the lease multiple times.

To attempt to renew the IP address lease, the client sends a unicast DHCPREQUEST message. The server that originally leased the IP address sends a DHCPACK message back to the client. This message contains any new parameters that have changed since the original lease was created. Note that these packets do not broadcast, because at this point the client has an IP address that it can use for unicast communications.

If the DHCP client cannot contact the DHCP server, then the client waits until 87.5 percent of the lease time expires. At this point, the client sends a DHCPREQUEST broadcast (rather than a unicast) to obtain a renewal, and the request goes to all DHCP servers, not just the server that provided the original lease. However, this broadcast request is for a renewal, not a new lease.

Because client computers might be moved while they are turned off (for example, a laptop computer that is plugged into a new subnet), client computers also attempt renewal during the startup process, or when the computer detects a network change. If renewal is successful, the lease period resets.

