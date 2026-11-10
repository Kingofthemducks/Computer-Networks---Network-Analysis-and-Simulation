# Computer Networks – Network Analysis & Simulation

A practical networking project exploring network protocols, packet analysis, traffic monitoring, network configuration, and security through **Wireshark** and **Cisco Packet Tracer**.

The project covers both theoretical networking concepts and hands-on simulations, demonstrating how devices communicate, how network traffic can be inspected, and how different network components affect performance and security.

## Project Overview

This project is divided into two main areas:

- **Network Protocol Analysis:** Examining TCP, ICMP, HTTP, packet statistics, and traffic filtering using Wireshark.
- **Network Design & Simulation:** Building and testing network topologies, configuring routers, exploring firewall behaviour, and comparing hubs with switches using Cisco Packet Tracer.

---

# Part 1 – Network Protocol Analysis

## 1. TCP Flags

TCP (Transmission Control Protocol) uses control flags to manage connections, acknowledge received data, and handle connection termination.

The six main TCP flags examined in this project are:

| Flag | Purpose |
|---|---|
| SYN | Initiates a TCP connection and synchronises sequence numbers. |
| ACK | Acknowledges received data or connection messages. |
| FIN | Indicates that a device has finished sending data and wishes to close the connection. |
| RST | Immediately resets or rejects a TCP connection. |
| PSH | Requests that received data be pushed to the application without unnecessary buffering delays. |
| URG | Indicates that the urgent pointer field is significant and urgent data is present. |

### TCP Three-Way Handshake

<img src="./screenshots/Meditaken%20LockScreen%20on%20S22%20ultra.jpg" width="300"> <img src="./screenshots/lockscreen%20of%20meditaken%20with%20message.png" width="300">

A TCP connection is normally established through a three-step handshake:

1. **SYN:** The client sends a connection request to the server.
2. **SYN-ACK:** The server acknowledges the request and sends its own synchronisation message.
3. **ACK:** The client acknowledges the server, completing the handshake.

This process establishes a reliable connection before normal data transmission begins.

**Screenshot – TCP SYN, SYN-ACK and ACK packets**

<img src="./screenshots/SYN-ACK%20Flag.png" width="800"> 

`![TCP Three-Way Handshake](./screenshots/tcp-handshake.png)`

### TCP Connection Termination

TCP connections are normally closed through a four-step process:

1. The client sends a FIN packet.
2. The server responds with an ACK.
3. The server sends its own FIN packet.
4. The client responds with an ACK.

This allows both sides to independently close their connection and finish transmitting outstanding data.

**Screenshot – TCP FIN and ACK packets**

`![TCP Connection Termination](./screenshots/tcp-termination.png)`

---

## 2. ICMP – Ping and Echo Requests

ICMP (Internet Control Message Protocol) is used for network diagnostics and error reporting. It operates at the Internet Layer of the TCP/IP model.

One of its most common uses is the `ping` command, which checks whether a destination is reachable.

In this project, I used Google's public DNS address (`8.8.8.8`) as the destination for an ICMP Echo Request.

Important packet fields include:

| Field | Description |
|---|---|
| Destination | The IP address receiving the Echo Request. |
| Length | The total packet length, depending on the displayed protocol layer. |
| Type 8 | ICMP Echo Request. |
| Type 0 | ICMP Echo Reply. |
| Identifier | Helps match requests with their corresponding replies. |
| Sequence Number | Identifies the order of individual Echo Requests. |
| TTL | Limits the number of network hops a packet can traverse. |
| Response Time | Measures how long the destination takes to respond. |

The successful reception of an Echo Reply indicates that the destination is reachable through the tested network path.

**Screenshot – ICMP Echo Request and Echo Reply in Wireshark**

`![ICMP Ping Analysis](./screenshots/icmp-ping.png)`

---

## 3. ICMP Traceroute

Traceroute is a network diagnostic technique used to identify the routers that packets pass through on their way to a destination.

It works by manipulating the **Time to Live (TTL)** value in the IP header.

Each router that forwards a packet decreases its TTL by one. When the TTL reaches zero, the router discards the packet and normally returns an ICMP Time Exceeded message.

The process works as follows:

1. Traceroute sends a packet with a TTL of 1.
2. The first router decreases the TTL to zero and returns an ICMP Time Exceeded message (Type 11).
3. The next packet is sent with a TTL of 2, allowing it to reach the second router.
4. The TTL continues increasing until the destination is reached.
5. The destination responds, allowing traceroute to identify the end of the route.

Common ICMP message types observed during this process:

| ICMP Type | Description |
|---|---|
| Type 8 | Echo Request |
| Type 11 | Time Exceeded |
| Type 0 | Echo Reply |

By examining these packets, it is possible to identify individual network hops and measure the response time of each router along the route.

**Screenshot – Traceroute packet analysis**

`![ICMP Traceroute](./screenshots/icmp-traceroute.png)`

---

## 4. Network Traffic Monitoring

Wireshark provides graphical statistics that help visualise network activity over time.

The **Packets per Second (PPS)** graph displays the number of captured packets during individual time intervals.

Key graph features include:

- **Time axis:** Displays elapsed time during the packet capture.
- **Packets axis:** Shows the number of packets captured per interval.
- **Line and bar graphs:** Provide different ways to visualise traffic patterns.
- **Display filters:** Allow specific protocols or packet types to be selected.
- **Zoom and drag:** Help investigate particular periods of network activity.
- **Logarithmic scale:** Makes traffic patterns with large variations easier to inspect.
- **Time of day:** Displays timestamps relative to the time of day rather than only elapsed capture time.
- **Automatic updates:** Refreshes the graph as new packets are captured.
- **Legend:** Identifies the displayed data and its corresponding colours.
- **Interval:** Determines the time period represented by each data point.

Shorter intervals are useful for identifying sudden traffic spikes and fluctuations, whereas longer intervals provide a smoother overview of network activity.

**Screenshot – Wireshark Packets per Second graph**

`![Packets per Second](./screenshots/packets-per-second.png)`

---

## 5. HTTP – Hypertext Transfer Protocol

HTTP is an application-layer protocol used for communication between web clients and servers.

When a user visits a website, the browser sends an HTTP request to the server. The server processes the request and returns an HTTP response.

### HTTP Request Analysis

The HTTP request examined in this project used the `GET` method.

Important request fields include:

| Field | Description |
|---|---|
| Request Method | Specifies the requested operation, such as GET. |
| Request URI | Identifies the requested resource, such as `/`. |
| HTTP Version | Specifies the protocol version, such as HTTP/1.1. |

The GET method is commonly used to retrieve resources such as webpages, images, and other server content.

### HTTP Response Analysis

The server responds with an HTTP response containing information about the result of the request.

Important response fields include:

| Field | Description |
|---|---|
| Response Version | The HTTP version used by the server. |
| Status Code | A numerical result describing the request outcome. |
| Status Description | A short explanation of the status code. |

Common HTTP status codes include:

- **200 OK:** The request was successful.
- **404 Not Found:** The requested resource could not be found.
- **500 Internal Server Error:** The server encountered an error while processing the request.

### Client–Server Communication

The basic communication process is:

1. The browser sends an HTTP GET request.
2. The server receives and processes the request.
3. The server returns an HTTP response containing a status code and, where appropriate, the requested content.
4. The browser receives and displays the returned resource.

**Screenshot – HTTP GET request in Wireshark**

`![HTTP Request](./screenshots/http-request.png)`

**Screenshot – HTTP response and status code**

`![HTTP Response](./screenshots/http-response.png)`

---

## 6. Network Traffic Statistics

Wireshark provides additional statistics that help analyse packet behaviour and network performance.

### Packet Size

Packet size affects how much information can be transmitted in each packet.

Larger packets can carry more data with relatively less protocol overhead, while smaller packets may require more headers to transmit the same amount of information.

In my capture, noticeable packet-size increases occurred around the 41-second and 54-second marks. These peaks coincided with periods of downloading activity.

### Transmission Intervals

Transmission intervals represent the time between packets.

- Short intervals can indicate frequent packet transmission and increased network activity.
- Irregular intervals may indicate jitter or inconsistent traffic patterns.
- Longer gaps can indicate reduced activity or interruptions in transmission.

### Network Congestion

Network congestion occurs when the amount of traffic exceeds the available network capacity.

Potential consequences include:

- Increased latency.
- Packet loss.
- TCP retransmissions.
- Reduced effective throughput.

A sudden increase in traffic around the 10-second mark was visible in my capture and provided an example of an unusual traffic pattern worth investigating.

### Time Interval Analysis

Different graph intervals provide different levels of detail.

**Short intervals (approximately 0.1–10 seconds):**
- Useful for identifying sudden traffic spikes.
- Help investigate packet loss and short-term congestion.
- Provide more detail when analysing downloads and streaming activity.

**Long intervals (10 seconds or more):**
- Provide a smoother overview of network performance.
- Help identify longer-term traffic patterns.
- Are useful for observing recurring activity, such as scheduled downloads or backups.

**Screenshot – Packet size and network traffic statistics**

`![Network Statistics](./screenshots/network-statistics.png)`

---

## 7. Wireshark Traffic Filters

Wireshark filtering allows network traffic to be narrowed down to specific protocols, devices, ports, and communication patterns.

There are two main types of filters.

### Capture Filters

Capture filters are applied **before packet capturing begins**.

They determine which packets Wireshark records, reducing the amount of data collected.

They use Berkeley Packet Filter (BPF) syntax and can filter traffic based on:

- IP addresses.
- Network subnets.
- Protocols.
- Port numbers.

### Display Filters

Display filters are applied **after packets have been captured**.

They hide packets that do not match the selected conditions without deleting them from the capture.

They use Wireshark's own filtering syntax and are useful when investigating specific traffic after a capture has already been completed.

### Advanced Display Filter Examples

**Filtering HTTP and HTTPS traffic**

```wireshark
tcp.port == 80 || tcp.port == 443
```

Displays TCP traffic using ports 80 and 443, commonly associated with HTTP and HTTPS.

**Filtering communication between two specific devices**

```wireshark
ip.src == 192.168.42.178 && ip.dst == 95.100.104.29
```

Displays packets travelling from the specified source IP address to the specified destination IP address.

**Excluding ARP and ICMP traffic**

```wireshark
!(arp || icmp)
```

Hides ARP and ICMP packets, making it easier to focus on other protocols, such as HTTP, DNS, or FTP.

Combining conditions with logical operators makes it possible to investigate specific network interactions more efficiently.

**Screenshot – Advanced Wireshark display filters**

`![Wireshark Display Filters](./screenshots/wireshark-filters.png)`

---

# Part 2 – Network Design & Simulation

Cisco Packet Tracer was used to build network topologies, configure devices, and test communication between different network components.

## 8. Small Homogeneous Network – Router Configuration

I created a small network consisting of three routers, three switches, and three PCs.

The routers were interconnected using serial connections, while the remaining devices were connected using Copper Straight-Through cables.

### Network Configuration

Each router was configured with:

- Serial interfaces for communication with neighbouring routers.
- FastEthernet interfaces for its local network.
- IP addresses assigned to the relevant interfaces.
- RIP routing to exchange network information.

Example addressing scheme:

| Network | Router Interface | Example IP Address |
|---|---|---|
| Network 1 | FastEthernet 0/0 | 192.168.1.1 |
| Network 2 | FastEthernet 0/0 | 192.168.2.1 |
| Network 3 | FastEthernet 0/0 | 192.168.3.1 |

Serial interfaces were assigned addresses from the `11.0.0.0` network to allow communication between routers.

### RIP Configuration

Routing Information Protocol (RIP) was configured to allow routers to exchange routing information and learn about networks that were not directly connected to them.

This allowed devices on separate local networks to communicate without manually configuring every remote route.

### Connectivity Testing

After configuring the routers and PCs, I used the `ping` command from the command prompt to test communication between devices on different networks.

A successful test with four replies and no packet loss indicated that the devices could communicate across the configured network.

**Screenshot – Three-router network topology**

`![Three Router Network](./screenshots/three-router-network.png)`

**Screenshot – Router configuration and RIP**

`![RIP Configuration](./screenshots/rip-configuration.png)`

**Screenshot – Successful ping between different networks**

`![Network Connectivity Test](./screenshots/network-ping-test.png)`

---

## 9. Firewall Configuration and Testing

I created a smaller network consisting of one switch, three PCs, and one server configured to demonstrate firewall behaviour.

The objective was to test how firewall rules can restrict communication between devices.

### Configuration

1. Connected the PCs and server to the same switch.
2. Assigned each device a different IP address.
3. Configured the server's firewall settings.
4. Created a rule denying ICMP traffic.
5. Enabled the firewall and tested connectivity using ping commands.

### Testing Results

After applying the firewall rule, I attempted to ping the other devices from one of the PCs.

The other devices returned request timeouts, while the PC could still successfully ping itself.

This demonstrated how firewall rules can restrict certain types of network traffic and prevent devices from communicating through specific protocols.

**Screenshot – Firewall configuration**

`![Firewall Configuration](./screenshots/firewall-configuration.png)`

**Screenshot – Ping tests with firewall enabled**

`![Firewall Testing](./screenshots/firewall-testing.png)`

---

## 10. Hubs vs Switches

I built two separate network simulations in Cisco Packet Tracer to compare the behaviour of hubs and switches.

Each topology consisted of three PCs connected to a central networking device.

I assigned each PC a unique IP address within the same network and tested communication between devices using ping requests.

### Hub Simulation

A hub operates at the Physical Layer (Layer 1) of the OSI model.

When a hub receives a frame, it repeats the incoming signal to all other connected ports rather than forwarding it only to the intended destination.

**Main limitations of hubs:**

- All connected devices receive the repeated traffic.
- They share a single collision domain.
- They operate in half-duplex mode.
- Collisions can occur when devices transmit simultaneously.
- Traffic is easier for other connected devices to intercept.
- Unnecessary traffic consumes available network bandwidth.

### Switch Simulation

A network switch operates primarily at the Data Link Layer (Layer 2).

Unlike a hub, a switch learns device MAC addresses and uses its MAC address table to forward frames towards the appropriate destination port.

This improves network efficiency by reducing unnecessary traffic sent to other devices.

### Comparison

| Feature | Hub | Switch |
|---|---|---|
| OSI Layer | Layer 1 | Layer 2 |
| Forwarding | Repeats signals to other ports | Forwards frames using MAC addresses |
| Collision Domains | One shared domain | Separate collision domains per port |
| Duplex | Half-duplex | Typically full-duplex |
| Efficiency | Lower | Higher |
| Security | Traffic is easily exposed to connected devices | Better traffic separation |

Switches are generally much more efficient and secure than traditional hubs.

Additional security can be achieved through network segmentation, VLANs, and appropriate switch security configurations.

**Screenshot – Hub network simulation**

`![Hub Network](./screenshots/hub-network.png)`

**Screenshot – Switch network simulation**

`![Switch Network](./screenshots/switch-network.png)`

---

# Technologies & Tools

| Technology | Purpose |
|---|---|
| Wireshark | Packet capture, protocol inspection, traffic filtering, and statistics. |
| Cisco Packet Tracer | Network design, device configuration, and network simulation. |
| TCP/IP | Understanding network communication and transport protocols. |
| ICMP | Network diagnostics, ping, and traceroute. |
| HTTP | Analysing client–server web communication. |
| RIP | Dynamic routing between interconnected networks. |
| Ethernet | Wired network communication between devices. |

# Skills Demonstrated

Through this project, I explored and practised:

- TCP connection establishment and termination.
- Packet-level analysis of network protocols.
- ICMP diagnostics and traceroute.
- HTTP client–server communication.
- Network traffic monitoring and statistical analysis.
- Wireshark capture and display filtering.
- IP addressing and subnet-based network design.
- Router configuration and dynamic routing using RIP.
- Firewall configuration and connectivity testing.
- Understanding the differences between hubs and switches.
- Identifying network congestion and potential performance issues.

# Author

**Laszlo Major**

Copyright © 2026 Laszlo Major. All rights reserved.
