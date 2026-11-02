# OSI Model – Networking Fundamentals

## 1. What is the OSI Model?

OSI stands for **Open Systems Interconnection**.

The OSI model is a conceptual framework that explains how data travels from one computer to another over a network.

It divides network communication into **7 layers**. Each layer performs a specific responsibility and works with the layers above and below it.

### Why do we need the OSI Model?

Imagine opening a website:

1. You enter `https://www.google.com` in your browser.
2. Your browser prepares an HTTP/HTTPS request.
3. Data is divided into segments and packets.
4. The network determines where the packets should go.
5. The physical network sends bits through cables or wireless signals.
6. The destination server receives and processes the data.
7. The response travels back to your browser.

The OSI model helps us understand each stage.

## 2. The Seven Layers of the OSI Model

| Layer | Name | Main Responsibility | Examples |
|---|---|---|---|
| 7 | Application | Provides network services to applications | HTTP, HTTPS, DNS, SMTP, SSH |
| 6 | Presentation | Data format, encryption and compression | TLS, JPEG, UTF-8 |
| 5 | Session | Establishes and manages communication sessions | Session management, RPC |
| 4 | Transport | End-to-end delivery, ports and reliability | TCP, UDP |
| 3 | Network | Logical addressing and routing | IPv4, IPv6, ICMP, routers |
| 2 | Data Link | Frames, MAC addressing and local delivery | Ethernet, Wi-Fi, switches |
| 1 | Physical | Transmits raw bits as signals | Cables, fiber, radio signals |

### Easy way to remember

**Top to Bottom:**

All People Seem To Need Data Processing

- A – Application
- P – Presentation
- S – Session
- T – Transport
- N – Network
- D – Data Link
- P – Physical

## 3. Layer 7 – Application Layer

The Application Layer is the layer closest to the end user.

It provides network services that applications use to communicate.

### Examples

| Protocol | Purpose | Default Port |
|---|---|---|
| HTTP | Website communication | 80 |
| HTTPS | Secure website communication | 443 |
| DNS | Resolves domain names to IP addresses | 53 |
| SSH | Secure remote server access | 22 |
| FTP | File transfer | 21 (control) |
| SMTP | Sending email | 25 (server-to-server) |
| DHCP | Automatically assigns IP configuration | UDP 67/68 |

### Practical example

When you enter:

`https://www.google.com`

Your browser uses application-layer protocols to communicate with the website.

**DevOps example:**
- Nginx receives HTTP/HTTPS requests.
- An application server handles application requests.
- DNS resolves the domain name.

## 4. Layer 6 – Presentation Layer

The Presentation Layer is responsible for representing data in a format that the receiving system can understand.

### Main responsibilities

1. Data translation and formatting.
2. Encryption and decryption.
3. Compression and decompression.

### Examples

- UTF-8 – Character encoding.
- JPEG – Image format.
- PNG – Image format.
- Data serialization formats such as JSON.

### Encryption example

When you access an HTTPS website, TLS protects communication through encryption.

**Important:** TLS is commonly associated with the OSI Presentation Layer in conceptual explanations, but in real TCP/IP implementations TLS operates between application protocols and the transport layer. OSI layer mapping is not always exact.

## 5. Layer 5 – Session Layer

The Session Layer establishes, manages and terminates communication sessions between applications.

### Main responsibilities

- Session establishment.
- Session maintenance.
- Session termination.
- Session recovery and synchronization concepts.

### Example

Imagine a user logging into an application.

The application establishes a session to maintain the user's authenticated state across requests.

Examples of session-related mechanisms include:
- Application session IDs.
- RPC session management.
- Authentication sessions.

**Important:** In modern networking, session functionality is often implemented inside applications and libraries rather than as a separate network layer.

## 6. Layer 4 – Transport Layer

The Transport Layer provides communication between processes running on different systems.

It uses **port numbers** to identify the destination application or service.

The two important protocols are:

1. TCP – Transmission Control Protocol.
2. UDP – User Datagram Protocol.

### TCP

TCP is connection-oriented and provides reliable, ordered byte-stream delivery.

Main features:
- Connection establishment using a three-way handshake.
- Acknowledgements.
- Retransmission of lost data.
- Sequencing and reassembly.
- Flow control and congestion control.

### TCP Three-Way Handshake

1. SYN – Client requests a connection.
2. SYN-ACK – Server acknowledges and responds.
3. ACK – Client acknowledges the server.

```text
Client                         Server
  |                               |
  | -------- SYN ---------------> |
  |                               |
  | <------ SYN-ACK ------------- |
  |                               |
  | -------- ACK ---------------> |
  |                               |
  |       Connection Ready        |
```

### UDP

UDP is connectionless and does not guarantee delivery, ordering or retransmission.

Features:
- Low protocol overhead.
- No TCP-style connection handshake.
- Suitable for latency-sensitive communication.
- Application can implement its own reliability if required.

### TCP vs UDP

| TCP | UDP |
|---|---|
| Connection-oriented | Connectionless |
| Reliable delivery | No delivery guarantee |
| Ordered byte stream | Datagrams |
| Retransmission supported | No built-in retransmission |
| More protocol overhead | Less protocol overhead |
| HTTP/1.1, HTTP/2, SSH | DNS commonly, VoIP, streaming |

**Note:** HTTP/3 uses QUIC over UDP.

### What is a port?

An IP address identifies a network interface or destination host, while a port identifies a service or application endpoint.

Example:

`192.168.1.10:443`

- `192.168.1.10` – IP address.
- `443` – HTTPS service port.

### Important ports for DevOps

| Port | Protocol / Service |
|---|---|
| 22 | SSH |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 8080 | Common alternative HTTP/application port |
| 3000 | Common development application port |
| 9090 | Common Prometheus port |
| 6443 | Kubernetes API server |

Ports are conventions; applications can be configured to listen on different ports.

## 7. Layer 3 – Network Layer

The Network Layer handles logical addressing and routing between different networks.

It determines how packets can travel from the source network to the destination network.

### Main responsibilities

- Logical IP addressing.
- Routing.
- Packet forwarding.
- Path selection.
- Fragmentation in IPv4 where applicable.

### Important protocols

- IPv4.
- IPv6.
- ICMP.
- Routing protocols such as OSPF and BGP.

### What is an IP address?

An IP address is a logical address used to identify a network interface in an IP network.

Example:

`192.168.1.10`

### What is a router?

A router forwards IP packets between networks using routing information.

### Practical example

Suppose:

- EC2-A private IP: `10.0.1.10`
- EC2-B private IP: `10.0.2.10`

If they belong to different subnets in the same VPC, the VPC local route allows communication, provided Security Groups and Network ACLs allow the traffic.

If they belong to different VPCs, you need a connectivity mechanism such as:
- VPC Peering.
- Transit Gateway.
- VPN.

### AWS networking connection

| Networking concept | AWS service or feature |
|---|---|
| Logical network | VPC |
| Network segmentation | Subnet |
| Routing | Route Table |
| Internet connectivity | Internet Gateway |
| Outbound internet for private IPv4 instances | NAT Gateway |
| Private VPC-to-VPC connectivity | VPC Peering |
| Centralized VPC connectivity | Transit Gateway |

## 8. Layer 2 – Data Link Layer

The Data Link Layer handles communication between devices on the same local network link.

It works with frames and MAC addresses.

### Main responsibilities

- Framing.
- MAC addressing.
- Local network delivery.
- Error detection (for example, Ethernet frame FCS).
- Media access control.

### What is a MAC address?

MAC stands for Media Access Control.

It is a link-layer address associated with a network interface.

Example:

`00:1A:2B:3C:4D:5E`

### What is a switch?

A Layer 2 switch forwards Ethernet frames using its MAC address table.

It learns which MAC addresses are reachable through which switch ports.

### What is ARP?

ARP stands for Address Resolution Protocol.

In IPv4 local networks, ARP helps discover the MAC address associated with a local IPv4 address.

Example:

A computer wants to send traffic to `192.168.1.20`.

It checks whether it knows the destination MAC address. If not, it uses ARP to discover it.

**Important:** ARP is used for IPv4 address resolution on a local link. IPv6 uses Neighbor Discovery Protocol (NDP).

## 9. Layer 1 – Physical Layer

The Physical Layer is responsible for transmitting raw bits over a physical medium.

It deals with electrical, optical or radio signals.

### Examples

- Ethernet cables.
- Fiber optic cables.
- Wi-Fi radio signals.
- Network connectors.
- Hubs and repeaters.

### Main responsibilities

- Transmission of bits.
- Signal encoding.
- Physical media characteristics.
- Data transmission speed.
- Connectors and electrical/optical specifications.

Example:

When a computer sends data through an Ethernet cable, the bits are represented by physical signals.

## 10. How Data Travels Through the OSI Model

Let's understand what happens when you open:

`https://example.com`

### Sender side – Encapsulation

Data moves from Layer 7 down to Layer 1.

| Layer | Data Unit | What happens |
|---|---|---|
| Application | Data | HTTP request is created |
| Presentation | Data | Data formatting/encryption concepts |
| Session | Data | Session management |
| Transport | Segment (TCP) | Adds source and destination ports |
| Network | Packet | Adds source and destination IP addresses |
| Data Link | Frame | Adds source and destination MAC addresses |
| Physical | Bits | Transmits signals |

### Receiver side – Decapsulation

The destination receives the bits and processes them from Layer 1 up to Layer 7.

```text
SENDER                                  RECEIVER

Application  ----------------------->   Application
Presentation ----------------------->   Presentation
Session      ----------------------->   Session
Transport    ----------------------->   Transport
Network      ----------------------->   Network
Data Link    ----------------------->   Data Link
Physical     ----------------------->   Physical
```

The sender encapsulates data by adding protocol information at different layers. The receiver removes and processes that information during decapsulation.

**Note:** This is a conceptual view. Real TCP/IP implementations do not always have a separate implementation for each OSI layer.

## 11. OSI Model vs TCP/IP Model

The TCP/IP model is the practical networking model used by the Internet.

| OSI Model | TCP/IP Model |
|---|---|
| Application | Application |
| Presentation | Application |
| Session | Application |
| Transport | Transport |
| Network | Internet |
| Data Link | Network Access / Link |
| Physical | Network Access / Link |

### Key differences

| OSI Model | TCP/IP Model |
|---|---|
| 7 layers | Commonly represented as 4 layers |
| Conceptual reference model | Practical protocol architecture |
| Separates Session and Presentation | Includes these functions in Application |
| Separates Data Link and Physical | Commonly combines them into Link |

## 12. Network Troubleshooting Using the OSI Model

As a DevOps or Cloud Engineer, you should troubleshoot connectivity layer by layer.

| Layer | Common issue | Useful command or tool |
|---|---|---|
| L7 Application | Application returns HTTP 500 | `curl -v https://example.com` |
| L6 Presentation | TLS certificate or encryption issue | `openssl s_client -connect example.com:443` |
| L5 Session | Application session/authentication issue | Application logs |
| L4 Transport | Port blocked or service not listening | `nc -vz IP PORT`, `ss -tulnp` |
| L3 Network | Incorrect IP or routing | `ip addr`, `ip route`, `traceroute` |
| L2 Data Link | MAC/ARP or local link issue | `ip neigh`, `arp -n` |
| L1 Physical | Cable, NIC or signal issue | Interface status, cable/link checks |

### Example: EC2 website is not accessible

Suppose you hosted an Nginx website on an EC2 instance, but the browser cannot open it.

Troubleshoot:

1. **Layer 1:** Is the network interface/link operational?
2. **Layer 2:** Is local network communication working?
3. **Layer 3:** Does the EC2 instance have the correct IP and route? Is the subnet route table correct?
4. **Layer 4:** Is Nginx listening on port 80 or 443? Does the Security Group allow the port?
5. **Layer 5:** Is the application session working?
6. **Layer 6:** Is the TLS certificate valid if using HTTPS?
7. **Layer 7:** Is Nginx/application returning the expected HTTP response?

Useful commands:

```bash
# Check network interfaces and IP addresses
ip addr

# Check routing table
ip route

# Check neighbor/ARP information
ip neigh

# Check listening ports
sudo ss -tulnp

# Check connectivity to a port
nc -vz <EC2_PRIVATE_IP> 80

# Check HTTP response
curl -v http://localhost

# Check HTTPS certificate and handshake
openssl s_client -connect example.com:443
```

Also check AWS controls:
- Security Group inbound rules.
- Security Group outbound rules.
- Network ACL inbound and outbound rules.
- Subnet route table.
- Internet Gateway or NAT Gateway, depending on the traffic direction.
- VPC Flow Logs.

## 13. Important Interview Questions and Answers

### Q1. What is the OSI model?

The OSI model is a seven-layer conceptual framework that explains how data communication happens between networked systems.

### Q2. How many layers are in the OSI model?

Seven layers: Application, Presentation, Session, Transport, Network, Data Link and Physical.

### Q3. Which layer is responsible for routing?

Layer 3 – Network Layer.

### Q4. Which layer uses IP addresses?

Layer 3 – Network Layer.

### Q5. Which layer uses MAC addresses?

Layer 2 – Data Link Layer.

### Q6. Which layer uses port numbers?

Layer 4 – Transport Layer.

### Q7. What is the difference between TCP and UDP?

TCP provides reliable, ordered delivery using acknowledgements and retransmissions. UDP provides connectionless datagram delivery without built-in delivery or ordering guarantees.

### Q8. Which layer does a switch work on?

A traditional Ethernet switch operates at Layer 2. Layer 3 switches can also perform routing.

### Q9. Which layer does a router work on?

A router primarily operates at Layer 3.

### Q10. What is encapsulation?

Encapsulation is the process of adding protocol headers (and sometimes trailers) as data moves down the network stack.

### Q11. What is decapsulation?

Decapsulation is the process of processing and removing protocol information as data moves up the network stack at the receiving system.

### Q12. What is the difference between an IP address and a MAC address?

An IP address is a logical network-layer address used for communication across networks. A MAC address is a link-layer address used for local network delivery.

### Q13. What is the difference between a frame, packet and segment?

- Segment: Transport-layer data unit, commonly used for TCP.
- Packet: Network-layer data unit.
- Frame: Data-link-layer data unit.

For UDP, the transport-layer data unit is called a datagram.

### Q14. Why is DNS considered an Application Layer protocol?

DNS provides a domain-name resolution service to applications. It commonly uses UDP port 53 and can also use TCP port 53.

### Q15. What happens when you enter a website URL?

The system resolves the domain name, establishes the required transport connection (depending on the protocol), performs TLS negotiation when HTTPS is used, sends the application request, and receives the response.

### Q16. How do you troubleshoot a network issue using the OSI model?

Start from the physical/link connectivity and work upward through IP routing, transport ports, TLS/session behavior and application responses. Use commands and logs appropriate to each layer.

## 14. Quick Revision Table

| Layer | Name | Key concept | Address / Data |
|---|---|---|---|
| 7 | Application | Network services | Application data |
| 6 | Presentation | Format, encryption | Application data |
| 5 | Session | Session management | Application data |
| 4 | Transport | TCP / UDP, ports | Segment / Datagram |
| 3 | Network | Routing, IP | Packet |
| 2 | Data Link | MAC, frames | Frame |
| 1 | Physical | Signals, cables, bits | Bits |

## 15. Important Points to Remember for AWS DevOps Interviews

- IP address belongs to Layer 3.
- MAC address belongs to Layer 2.
- TCP and UDP belong to Layer 4.
- Port numbers identify service endpoints.
- Routers forward packets between networks.
- Layer 2 switches forward frames using MAC addresses.
- DNS resolves domain names to IP addresses.
- Security Groups and Network ACLs control traffic in AWS.
- Security Groups are stateful; Network ACLs are stateless.
- VPC route tables determine where network traffic is sent.
- VPC Flow Logs help investigate accepted and rejected IP traffic.
- OSI is a conceptual model; actual Internet networking follows the TCP/IP architecture.

---

## Learning Outcome

After studying this topic, you should be able to:

- Explain all seven OSI layers.
- Differentiate IP address, MAC address and port number.
- Explain TCP vs UDP.
- Understand encapsulation and decapsulation.
- Understand how switches and routers work.
- Map networking concepts to AWS VPC components.
- Troubleshoot basic EC2 connectivity problems using the OSI model.