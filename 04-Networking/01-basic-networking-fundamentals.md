# Networking Fundamentals — Complete Notes
**Learning Path:** AWS Cloud | DevOps | SRE  
**Topics:** IPv4, Classful Addressing, Classless Addressing, CIDR, Subnetting, IP Ranges, Switches, Routers, and Routing

---

## Table of Contents

1. What Is Networking?
2. What Is an IP Address?
3. IPv4 Address Structure
4. Binary, Bits, and Bytes
5. Why Does Each IPv4 Octet Range from 0 to 255?
6. Classful IP Addressing
7. IP Address Classes A, B, C, D, and E
8. Public and Private IP Addresses
9. Loopback IP Address
10. Network Bits and Host Bits
11. Subnet Masks
12. Classless Addressing
13. CIDR Notation
14. How to Calculate the Number of Host Bits
15. How to Calculate the Number of IP Addresses
16. How to Calculate IP Block Size and Range
17. Worked Subnetting Examples
18. Switches and Routers
19. How Routing Works
20. Default Gateway and Route Tables
21. Networking in AWS
22. Linux Networking Commands
23. Common Networking Mistakes
24. Networking Interview Questions and Answers
25. Practice Exercises

---

# 1. What Is Networking?

Computer networking is the process of connecting two or more devices so they can communicate and exchange data.

Examples of network devices include:

- Computers
- Servers
- Mobile phones
- Routers
- Switches
- Printers
- Cloud servers

### Real-world example

Suppose you open a website hosted on an AWS EC2 instance.

Your computer sends a request through a network. Routers forward the request toward its destination. The server processes the request and sends a response back.

The communication involves several technologies:

- IP addressing
- DNS
- Routing
- TCP or UDP
- HTTP or HTTPS
- Firewalls and security rules

### Why is networking important for DevOps?

Networking is essential when working with:

- AWS VPCs and subnets
- EC2 instances
- Load balancers
- Kubernetes networking
- Docker networking
- DNS configuration
- Application connectivity
- Security groups and NACLs
- CI/CD pipelines
- Production troubleshooting

For example, if an application is running on an EC2 instance but users cannot access it, you may need to investigate its IP address, route table, security group, application port, and DNS configuration.

---

# 2. What Is an IP Address?

An IP address is a logical address used to identify a network interface and enable communication using the Internet Protocol.

Example:

`192.168.1.10`

An IPv4 address contains four decimal numbers separated by dots.

Each number is called an **octet**.

Example:

| Octet 1 | Octet 2 | Octet 3 | Octet 4 |
|---|---|---|---|
| 192 | 168 | 1 | 10 |

An IPv4 address contains 32 bits in total.

**Important:** An IP address identifies a network interface, not necessarily an entire physical device. A device can have multiple network interfaces and multiple IP addresses.

---

# 3. IPv4 Address Structure

An IPv4 address contains:

- 32 bits
- 4 octets
- 8 bits per octet

Therefore:

`8 + 8 + 8 + 8 = 32 bits`

Example:

`192.168.1.10`

Its binary representation is:

`11000000.10101000.00000001.00001010`

Each group of eight binary digits represents one octet.

| Decimal octet | Binary representation |
|---:|---|
| 192 | 11000000 |
| 168 | 10101000 |
| 1 | 00000001 |
| 10 | 00001010 |

Therefore:

`192.168.1.10 = 11000000.10101000.00000001.00001010`

This is a 32-bit IPv4 address.

---

# 4. Binary, Bits, and Bytes

## 4.1 What is a bit?

A bit is the smallest unit of digital information.

A bit can have only two values:

- `0`
- `1`

Computers represent information using binary numbers.

## 4.2 What is a byte?

One byte contains eight bits.

`1 byte = 8 bits`

An IPv4 octet contains eight bits, which is why each octet can represent values from 0 to 255.

## 4.3 Binary place values

The place values of an eight-bit binary number are:

| Bit position | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Decimal value | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

Each position represents a power of two.

From left to right:

`128, 64, 32, 16, 8, 4, 2, 1`

The maximum value is obtained by adding all eight place values:

`128 + 64 + 32 + 16 + 8 + 4 + 2 + 1 = 255`

The minimum value is:

`00000000 = 0`

The maximum value is:

`11111111 = 255`

Therefore:

**The decimal value of an IPv4 octet ranges from 0 to 255.**

## 4.4 How to convert binary to decimal

Example:

`11000000`

Assign the place values:

| Binary bit | 1 | 1 | 0 | 0 | 0 | 0 | 0 | 0 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Place value | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

Add the values corresponding to the bits set to 1:

`128 + 64 = 192`

Therefore:

`11000000 = 192`

Another example:

`10101000`

The bits set to 1 represent:

`128 + 32 + 8 = 168`

Therefore:

`10101000 = 168`

## 4.5 How to convert decimal to binary

Example: Convert `10` to binary.

The place values are:

`128, 64, 32, 16, 8, 4, 2, 1`

We can represent 10 as:

`8 + 2 = 10`

Set the bits corresponding to 8 and 2 to 1.

Result:

`00001010`

Therefore:

`10 = 00001010`

### Important binary values to memorize

| Decimal | Binary |
|---:|---|
| 0 | 00000000 |
| 1 | 00000001 |
| 2 | 00000010 |
| 4 | 00000100 |
| 8 | 00001000 |
| 16 | 00010000 |
| 32 | 00100000 |
| 64 | 01000000 |
| 128 | 10000000 |
| 255 | 11111111 |

These values are particularly useful when calculating subnet masks.

---

# 5. Classful IP Addressing

Classful addressing is an older method of dividing IPv4 addresses into fixed network classes.

The five classes are:

- Class A
- Class B
- Class C
- Class D
- Class E

Originally, Classes A, B, and C were used for different network sizes.

Class D was reserved for multicast, and Class E was reserved for experimental or future use.

Modern IP networks primarily use **CIDR (Classless Inter-Domain Routing)** instead of relying on fixed address classes.

However, understanding classful addressing is useful for networking fundamentals and interviews.

---

# 6. IP Address Classes A, B, C, D, and E

## 6.1 Class A

Class A addresses have a first-octet range of:

`1–126` for traditional unicast allocations.

The historical default subnet mask is:

`255.0.0.0`

CIDR equivalent:

`/8`

Example:

`10.20.30.40`

Under classful addressing, the network portion is the first 8 bits, and the remaining 24 bits represent the host portion.

| Component | Value |
|---|---|
| Default mask | 255.0.0.0 |
| CIDR | /8 |
| Network bits | 8 |
| Host bits | 24 |
| Historical block size | 16,777,216 addresses |

The first-octet value `127` is reserved for loopback purposes rather than ordinary Class A unicast addressing.

The traditional Class A range is commonly described as `1.0.0.0` through `126.255.255.255`, subject to special-purpose reservations.

## 6.2 Class B

Class B addresses have a first-octet range of:

`128–191`

The historical default subnet mask is:

`255.255.0.0`

CIDR equivalent:

`/16`

Example:

`172.20.10.5`

| Component | Value |
|---|---|
| Default mask | 255.255.0.0 |
| CIDR | /16 |
| Network bits | 16 |
| Host bits | 16 |
| Historical block size | 65,536 addresses |

The first 16 bits represent the network portion under the traditional classful interpretation.

## 6.3 Class C

Class C addresses have a first-octet range of:

`192–223`

The historical default subnet mask is:

`255.255.255.0`

CIDR equivalent:

`/24`

Example:

`192.168.1.10`

| Component | Value |
|---|---|
| Default mask | 255.255.255.0 |
| CIDR | /24 |
| Network bits | 24 |
| Host bits | 8 |
| Historical block size | 256 addresses |

The first 24 bits represent the network portion under the traditional classful interpretation.

## 6.4 Class D

Class D addresses have a first-octet range of:

`224–239`

Full range:

`224.0.0.0 – 239.255.255.255`

Purpose:

**IP multicast**

Multicast allows one sender to communicate with a group of interested receivers.

Class D addresses are not ordinary host addresses assigned to individual interfaces in the same way as unicast addresses.

## 6.5 Class E

Class E addresses have a first-octet range of:

`240–255`

Full range:

`240.0.0.0 – 255.255.255.255`

They are reserved for experimental or future use, with special-purpose exceptions and restrictions.

They are not used as ordinary public unicast addresses.

## 6.6 Classful addressing summary

| Class | First-octet range | Historical default mask | CIDR | General purpose |
|---|---|---|---|---|
| A | 1–126 | 255.0.0.0 | /8 | Large networks |
| B | 128–191 | 255.255.0.0 | /16 | Medium networks |
| C | 192–223 | 255.255.255.0 | /24 | Smaller networks |
| D | 224–239 | Not applicable | Not applicable | Multicast |
| E | 240–255 | Not applicable | Not applicable | Experimental/reserved |

**Remember:** The historical class determines a default network boundary. Modern CIDR addressing allows a network to use a prefix length independent of the traditional class.

For example:

`192.168.1.0/26`

This is a classless subnet with 64 total IPv4 addresses, even though the address begins with a first octet associated historically with Class C.

---

# 7. Public and Private IP Addresses

IPv4 addresses can be used in public networks or private networks.

## 7.1 Private IPv4 address ranges

The following three ranges are reserved for private IPv4 networking by RFC 1918.

| Private range | CIDR | Address count |
|---|---|---:|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | 16,777,216 |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | 1,048,576 |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | 65,536 |

Private addresses are commonly used in:

- Home networks
- Corporate networks
- AWS VPCs
- Kubernetes clusters
- Internal application networks

Private IPv4 addresses are not globally routable across the public internet.

Devices using private addresses commonly access the internet through NAT or another suitable network gateway.

## 7.2 Public IP address

A public IP address is globally routable in principle, subject to routing policy and special-purpose reservations.

Examples of common uses:

- Public websites
- Internet-facing load balancers
- Public EC2 endpoints
- Internet-facing network services

A public IP address alone does not guarantee connectivity. Routing, firewall rules, application configuration, and other network controls must also permit the traffic.

---

# 8. Loopback IP Address

A loopback address allows a device to communicate with itself through its local networking stack.

The IPv4 loopback range is:

`127.0.0.0/8`

The most commonly used loopback address is:

`127.0.0.1`

It is often called `localhost`.

Example:

```bash
ping 127.0.0.1
```

This tests the local machine's IP networking stack.

You can also use:

```bash
curl http://127.0.0.1:8080
```

This attempts to connect to a service listening on port 8080 on the local machine.

### Important distinction

Consider an application running on an EC2 instance.

If the application listens only on:

`127.0.0.1:8080`

It is listening on the instance's loopback interface. Other machines generally cannot connect to that listener through the instance's private or public IP address.

If the application needs to accept network connections, it must listen on an appropriate network interface or address, such as:

`0.0.0.0:8080`

Here, `0.0.0.0` means all local IPv4 interfaces when used as a listening address.

It is not a substitute for a routable destination address.

**DevOps interview point:** A service can be running successfully but remain inaccessible remotely because it is bound only to `127.0.0.1`.

---

# 9. Network Bits and Host Bits

This is one of the most important concepts in IP addressing.

An IPv4 address contains 32 bits.

Those bits are divided into:

1. Network bits
2. Host bits

The subnet mask or CIDR prefix identifies the boundary between them.

## 9.1 Network bits

Network bits identify the IP subnet to which an address belongs.

## 9.2 Host bits

Host bits identify an address within that subnet.

For ordinary IPv4 subnets, the network address is obtained by applying a bitwise AND operation between the IP address and its subnet mask.

## 9.3 Example: 192.168.1.10/24

IP address:

`192.168.1.10`

CIDR:

`/24`

The `/24` means:

- 24 network bits
- 8 host bits

Calculation:

`32 - 24 = 8 host bits`

Binary structure:

`NNNNNNNN.NNNNNNNN.NNNNNNNN.HHHHHHHH`

Where:

- `N` = network bit
- `H` = host bit

For `/24`, the first three octets belong to the network portion.

The final octet belongs to the host portion.

Example:

| Property | Value |
|---|---|
| IP address | 192.168.1.10 |
| CIDR | /24 |
| Network address | 192.168.1.0 |
| Broadcast address | 192.168.1.255 |
| First usable address | 192.168.1.1 |
| Last usable address | 192.168.1.254 |
| Total addresses | 256 |
| Traditional usable host addresses | 254 |

---

# 10. Subnet Masks

A subnet mask identifies which IPv4 bits belong to the network portion.

Example:

`255.255.255.0`

Binary representation:

`11111111.11111111.11111111.00000000`

There are 24 consecutive 1 bits, followed by 8 zero bits.

Therefore:

`255.255.255.0 = /24`

### Common subnet masks

| CIDR | Subnet mask | Network bits | Host bits | Total addresses | Traditional usable hosts |
|---|---|---:|---:|---:|---:|
| /8 | 255.0.0.0 | 8 | 24 | 16,777,216 | 16,777,214 |
| /16 | 255.255.0.0 | 16 | 16 | 65,536 | 65,534 |
| /20 | 255.255.240.0 | 20 | 12 | 4,096 | 4,094 |
| /24 | 255.255.255.0 | 24 | 8 | 256 | 254 |
| /25 | 255.255.255.128 | 25 | 7 | 128 | 126 |
| /26 | 255.255.255.192 | 26 | 6 | 64 | 62 |
| /27 | 255.255.255.224 | 27 | 5 | 32 | 30 |
| /28 | 255.255.255.240 | 28 | 4 | 16 | 14 |
| /29 | 255.255.255.248 | 29 | 3 | 8 | 6 |
| /30 | 255.255.255.252 | 30 | 2 | 4 | 2 |

The usable-host figures in this table use the traditional convention of subtracting the network and broadcast addresses. `/31` and `/32` require separate treatment.

---

# 11. Classless Addressing

Classless addressing is the modern method of assigning and routing IPv4 address blocks.

It allows networks to use different prefix lengths rather than being restricted to the historical Class A, B, or C boundaries.

For example, consider:

`192.168.1.0/26`

This subnet contains 64 total IPv4 addresses.

It has:

- 26 network bits
- 6 host bits

Total addresses:

`2^6 = 64`

Traditional usable host addresses:

`64 - 2 = 62`

Compare that with a `/24` subnet:

`192.168.1.0/24`

It contains 256 total addresses and traditionally 254 usable host addresses.

Classless addressing allows network administrators to allocate address blocks according to actual requirements.

### Why is classless addressing useful?

It helps with:

- Efficient IP address allocation
- Subnetting
- Route aggregation
- Designing AWS VPCs
- Creating public and private subnets
- Avoiding unnecessary address waste

**Important:** A classless network's prefix is explicitly specified. Do not infer its actual network size from the first octet.

---

# 12. CIDR Notation

CIDR stands for **Classless Inter-Domain Routing**.

CIDR notation combines an IPv4 address with a prefix length.

Example:

`192.168.1.0/24`

Here:

- `192.168.1.0` is the address.
- `/24` is the prefix length.
- The prefix specifies 24 network bits.
- The remaining 8 bits are host bits.

## 12.1 How to calculate network bits

The number after the slash gives the number of network bits.

Examples:

| CIDR | Network bits |
|---|---:|
| /8 | 8 |
| /16 | 16 |
| /24 | 24 |
| /26 | 26 |
| /28 | 28 |

## 12.2 How to calculate host bits

Formula:

`Host bits = 32 - Prefix length`

Examples:

For `/24`:

`32 - 24 = 8 host bits`

For `/26`:

`32 - 26 = 6 host bits`

For `/20`:

`32 - 20 = 12 host bits`

## 12.3 How to calculate total IP addresses

Formula:

`Total IPv4 addresses = 2^(number of host bits)`

Example: `/24`

Host bits:

`32 - 24 = 8`

Total addresses:

`2^8 = 256`

Example: `/26`

Host bits:

`32 - 26 = 6`

Total addresses:

`2^6 = 64`

Example: `/20`

Host bits:

`32 - 20 = 12`

Total addresses:

`2^12 = 4096`

## 12.4 Traditional usable-host formula

For ordinary IPv4 subnets where the network and broadcast addresses are reserved:

`Usable hosts = 2^(host bits) - 2`

Example: `/26`

`2^6 - 2 = 62`

Example: `/24`

`2^8 - 2 = 254`

### Exceptions

- `/31` can provide two addresses for a point-to-point link under RFC 3021.
- `/32` identifies a single IPv4 address.
- Special-purpose ranges may have additional restrictions.
- AWS reserves five IPv4 addresses in each IPv4 subnet, so AWS usable address counts differ from the traditional host calculation.

---

# 13. How to Calculate the Number of Host Bits

Always use this formula:

`Host bits = 32 - CIDR prefix`

Examples:

### Example 1: /24

`32 - 24 = 8`

Host bits = 8.

### Example 2: /26

`32 - 26 = 6`

Host bits = 6.

### Example 3: /27

`32 - 27 = 5`

Host bits = 5.

### Example 4: /20

`32 - 20 = 12`

Host bits = 12.

### Example 5: /16

`32 - 16 = 16`

Host bits = 16.

## Quick reference

| CIDR | Host bits | Total addresses |
|---|---:|---:|
| /16 | 16 | 65,536 |
| /20 | 12 | 4,096 |
| /24 | 8 | 256 |
| /25 | 7 | 128 |
| /26 | 6 | 64 |
| /27 | 5 | 32 |
| /28 | 4 | 16 |
| /29 | 3 | 8 |
| /30 | 2 | 4 |
| /31 | 1 | 2 |
| /32 | 0 | 1 |

As the prefix number increases, the subnet becomes smaller.

For example:

- `/24` contains 256 addresses.
- `/25` contains 128 addresses.
- `/26` contains 64 addresses.
- `/27` contains 32 addresses.

Each time the prefix increases by one, the address count is halved.

---

# 14. How to Calculate IP Block Size and Range

This is a key subnetting skill.

Consider:

`192.168.1.0/26`

We want to calculate:

1. Network address
2. Broadcast address
3. First usable IP
4. Last usable IP
5. Total IP addresses

## Step 1: Identify the prefix

`/26`

Network bits:

`26`

Host bits:

`32 - 26 = 6`

## Step 2: Calculate the total number of addresses

`2^6 = 64`

The subnet contains 64 total addresses.

## Step 3: Calculate the block size

When the subnet boundary falls inside an octet, a useful formula is:

`Block size = 256 - subnet mask value in the changing octet`

For `/26`, the subnet mask is:

`255.255.255.192`

The changing octet is the fourth octet.

Therefore:

`256 - 192 = 64`

Block size = 64.

## Step 4: Identify the subnet ranges

The ranges in the fourth octet increase by 64:

- 0–63
- 64–127
- 128–191
- 192–255

These are four separate `/26` subnets within the same `/24` address block.

## Step 5: Calculate the network and broadcast addresses

Suppose the given IP is:

`192.168.1.70/26`

The fourth octet is 70.

The block ranges are:

- 0–63
- 64–127
- 128–191
- 192–255

Since 70 falls between 64 and 127:

Network address:

`192.168.1.64`

Broadcast address:

`192.168.1.127`

First usable address:

`192.168.1.65`

Last usable address:

`192.168.1.126`

Total addresses:

`64`

Traditional usable hosts:

`64 - 2 = 62`

### Final answer

| Property | Value |
|---|---|
| Given IP | 192.168.1.70/26 |
| Subnet mask | 255.255.255.192 |
| Block size | 64 |
| Network address | 192.168.1.64 |
| First usable IP | 192.168.1.65 |
| Last usable IP | 192.168.1.126 |
| Broadcast address | 192.168.1.127 |
| Total addresses | 64 |
| Traditional usable hosts | 62 |

---

# 15. Worked Subnetting Examples

## Example 1: Calculate the range for 192.168.1.10/24

Given:

`192.168.1.10/24`

### Step 1: Calculate host bits

`32 - 24 = 8`

### Step 2: Calculate total addresses

`2^8 = 256`

### Step 3: Calculate the subnet mask

`255.255.255.0`

### Step 4: Calculate the range

The network is:

`192.168.1.0/24`

| Property | Value |
|---|---|
| Network address | 192.168.1.0 |
| First usable IP | 192.168.1.1 |
| Last usable IP | 192.168.1.254 |
| Broadcast address | 192.168.1.255 |
| Total addresses | 256 |
| Traditional usable hosts | 254 |

---

## Example 2: Calculate the range for 192.168.1.70/26

Given:

`192.168.1.70/26`

Host bits:

`32 - 26 = 6`

Total addresses:

`2^6 = 64`

Subnet mask:

`255.255.255.192`

Block size:

`256 - 192 = 64`

The fourth-octet range containing 70 is 64–127.

| Property | Value |
|---|---|
| Network address | 192.168.1.64 |
| First usable IP | 192.168.1.65 |
| Last usable IP | 192.168.1.126 |
| Broadcast address | 192.168.1.127 |
| Total addresses | 64 |
| Traditional usable hosts | 62 |

---

## Example 3: Calculate the range for 192.168.1.100/27

Given:

`192.168.1.100/27`

### Step 1: Calculate host bits

`32 - 27 = 5`

### Step 2: Calculate total addresses

`2^5 = 32`

### Step 3: Calculate the subnet mask

`255.255.255.224`

### Step 4: Calculate block size

`256 - 224 = 32`

The subnet ranges are:

- 0–31
- 32–63
- 64–95
- 96–127
- 128–159
- 160–191
- 192–223
- 224–255

The address 100 belongs to the 96–127 range.

| Property | Value |
|---|---|
| Network address | 192.168.1.96 |
| First usable IP | 192.168.1.97 |
| Last usable IP | 192.168.1.126 |
| Broadcast address | 192.168.1.127 |
| Total addresses | 32 |
| Traditional usable hosts | 30 |

---

## Example 4: Calculate the range for 172.16.20.15/20

This example is important because the subnet boundary does not fall at an octet boundary.

Given:

`172.16.20.15/20`

### Step 1: Calculate host bits

`32 - 20 = 12`

### Step 2: Calculate total addresses

`2^12 = 4096`

### Step 3: Calculate the subnet mask

`/20 = 255.255.240.0`

### Step 4: Calculate the block size

The changing octet is the third octet.

Its subnet mask value is 240.

`256 - 240 = 16`

The third-octet subnet ranges are:

- 0–15
- 16–31
- 32–47
- 48–63
- 64–79
- and so on.

The third octet of the given IP is 20.

Therefore, it belongs to the 16–31 range.

### Step 5: Calculate the address range

| Property | Value |
|---|---|
| Given IP | 172.16.20.15/20 |
| Subnet mask | 255.255.240.0 |
| Network address | 172.16.16.0 |
| First usable IP | 172.16.16.1 |
| Last usable IP | 172.16.31.254 |
| Broadcast address | 172.16.31.255 |
| Total addresses | 4,096 |
| Traditional usable hosts | 4,094 |

**Important lesson:** When a subnet prefix falls inside an octet, calculate the block size in that octet rather than assuming the subnet boundary is at the end of an octet.

---

# 16. Special Case: /31 and /32

## /31

A `/31` contains two IPv4 addresses.

Host bits:

`32 - 31 = 1`

Total addresses:

`2^1 = 2`

On point-to-point links that support RFC 3021, both addresses can be used as endpoint addresses without reserving a traditional network and broadcast address.

## /32

A `/32` contains one IPv4 address.

Host bits:

`32 - 32 = 0`

Total addresses:

`2^0 = 1`

A `/32` often identifies one specific IPv4 address, such as a host route.

---

# 17. Switches and Routers

Switches and routers help devices communicate, but they perform different jobs.

## 17.1 What is a switch?

A network switch primarily connects devices within a local network.

Examples:

- Computers in an office
- Servers in a data center
- Devices within a VLAN

A typical Layer 2 switch forwards Ethernet frames using MAC addresses.

It learns which MAC addresses are reachable through its ports by examining incoming frames and maintains a MAC address table.

Example:

Computer A → Switch → Computer B

If both devices are in the same VLAN and IP subnet, they can typically communicate through the switch without sending their traffic through a router.

## 17.2 What is a router?

A router forwards IP packets between networks.

Examples:

- Office LAN to the internet
- One IP subnet to another
- One VPC network to another through an appropriate routing connection

Routers use routing tables to select where packets should go.

Example:

Network A → Router → Network B

A router is commonly the default gateway for devices on a local network.

## 17.3 Switch vs router

| Feature | Switch | Router |
|---|---|---|
| Primary purpose | Connect devices within a LAN | Connect different IP networks |
| Typical OSI layer | Layer 2 | Layer 3 |
| Main forwarding information | MAC address table | IP routing table |
| Typical unit forwarded | Ethernet frame | IP packet |
| Separates IP subnets by itself | Not a typical Layer 2 switch | Yes |
| Connects a LAN to another routed network | Not by itself | Yes |
| Common example | Office Ethernet switch | Home internet router |

**Important:** Some switches support Layer 3 routing. The table describes the common distinction between a Layer 2 switch and an IP router.

## 17.4 Real-world example

Suppose you have:

- Laptop A: `192.168.1.10/24`
- Laptop B: `192.168.1.20/24`
- Server C: `192.168.2.10/24`

Laptop A wants to communicate with Laptop B.

Both belong to:

`192.168.1.0/24`

Laptop A can send traffic directly to Laptop B at Layer 2, assuming both are reachable within the same local network.

Now Laptop A wants to communicate with Server C.

Server C belongs to:

`192.168.2.0/24`

That is a different subnet.

Laptop A sends the packet to its default gateway, which routes the packet toward the destination network.

### Key takeaway

- Same local subnet: devices can generally communicate directly at Layer 2.
- Different IP subnets: communication normally requires routing.

---

# 18. How Routing Works

Routing is the process of selecting a path for IP packets to travel from a source network toward a destination network.

Routers use routing tables to make forwarding decisions.

## 18.1 Example network

Suppose an EC2 instance has:

`10.0.1.10/24`

Its subnet is:

`10.0.1.0/24`

The subnet's route table contains:

| Destination | Target |
|---|---|
| 10.0.0.0/16 | Local |
| 0.0.0.0/0 | Internet Gateway |

The first route represents the VPC's local network routing.

The second route is a default route toward the Internet Gateway.

For an appropriate internet-bound destination, the route table can select the Internet Gateway route. However, internet access also requires suitable public IP addressing, security rules, and other necessary configuration.

## 18.2 What is a routing table?

A routing table contains destination prefixes and the corresponding next hop or target.

A simplified Linux routing table might look like this:

```text
Destination       Gateway          Interface
10.0.1.0/24       0.0.0.0          eth0
0.0.0.0/0         10.0.1.1         eth0
```

The first entry represents a directly connected network.

The second represents a default route through a gateway.

Actual routing-table output varies by operating system and configuration.

## 18.3 What is a default route?

A default route is used when no more specific matching route is available.

For IPv4, the default route is:

`0.0.0.0/0`

It matches every IPv4 destination, but a more specific matching route normally takes precedence.

## 18.4 Longest-prefix match

Routers generally choose the matching route with the longest prefix.

This is called **longest-prefix match**.

Suppose a routing table contains:

| Destination | Target |
|---|---|
| 10.0.0.0/8 | Route A |
| 10.1.0.0/16 | Route B |
| 10.1.2.0/24 | Route C |
| 0.0.0.0/0 | Default route |

For destination:

`10.1.2.50`

All four prefixes match, but `/24` is the most specific.

Therefore, the router selects:

`10.1.2.0/24 → Route C`

This principle is important when troubleshooting overlapping routes in Linux, AWS, and enterprise networks.

---

# 19. Networking in AWS

AWS networking uses many of the same fundamentals described above.

## 19.1 VPC

A Virtual Private Cloud (VPC) is a logically isolated virtual network in AWS.

Example:

`10.0.0.0/16`

This IPv4 CIDR block contains 65,536 addresses.

Host bits:

`32 - 16 = 16`

Total addresses:

`2^16 = 65,536`

## 19.2 Subnets

A subnet divides a VPC's address space into smaller network ranges.

Example:

VPC:

`10.0.0.0/16`

Public subnet:

`10.0.1.0/24`

Private subnet:

`10.0.2.0/24`

Each `/24` contains 256 IPv4 addresses.

In AWS, five IPv4 addresses in each subnet are reserved by AWS.

Therefore:

`256 - 5 = 251`

A `/24` AWS subnet has 251 IPv4 addresses available for assignment to resources, subject to service-specific constraints.

## 19.3 Route tables

A route table controls where network traffic from a subnet is directed.

Example:

| Destination | Target | Purpose |
|---|---|---|
| 10.0.0.0/16 | Local | Communication within the VPC |
| 0.0.0.0/0 | Internet Gateway | Route IPv4 traffic toward the internet |

A private subnet commonly uses a NAT Gateway route for appropriate outbound internet access, rather than a direct route to an Internet Gateway.

Example:

| Destination | Target |
|---|---|
| 10.0.0.0/16 | Local |
| 0.0.0.0/0 | NAT Gateway |

A NAT Gateway does not automatically provide unsolicited inbound internet access to private instances.

## 19.4 Security groups

Security groups act as stateful virtual firewalls for supported AWS resources.

For example, a web server security group may allow inbound HTTPS traffic on TCP port 443.

Security groups also have outbound rules that control traffic initiated by the associated resource.

Return traffic for permitted connections is automatically allowed by the stateful behavior of security groups.

## 19.5 Network ACLs

Network ACLs are associated with subnets and act as stateless network filters.

They have inbound and outbound rules. Because they are stateless, both directions of traffic must be considered when designing connectivity.

## 19.6 AWS networking troubleshooting example

Suppose you launch an EC2 instance and cannot access its website.

Check:

1. Is the instance running?
2. Is the application running?
3. Is the application listening on the correct address and port?
4. Does the instance have the required IP addressing?
5. Does the subnet route table contain the necessary route?
6. Is an Internet Gateway attached and correctly configured, if public internet access is required?
7. Does the security group permit inbound traffic?
8. Do NACL rules permit the required traffic in both directions?
9. Is DNS configured correctly?
10. Is the application returning a valid response?

Do not assume that assigning a public IP address automatically makes an application reachable.

---

# 20. Linux Networking Commands

These commands are useful for DevOps and SRE troubleshooting.

## Display IP addresses

```bash
ip addr
```

Shows the IP addresses configured on network interfaces.

## Display network interfaces

```bash
ip link
```

Shows network interface information and status.

## Display routing table

```bash
ip route
```

Shows IPv4 routing information.

## Test connectivity to a destination

```bash
ping 8.8.8.8
```

Tests IP reachability using ICMP echo requests, if permitted.

A failed ping does not necessarily mean the destination is unavailable; ICMP may be blocked.

## Test HTTP or HTTPS

```bash
curl -I https://example.com
```

Requests HTTP response headers.

Useful for checking application reachability and HTTP response status.

## Check listening ports

```bash
ss -tulnp
```

Displays listening TCP and UDP sockets and associated processes where permissions allow.

## Resolve DNS

```bash
nslookup example.com
```

Queries DNS information using the configured resolver.

Another option:

```bash
dig example.com
```

## Trace the network path

```bash
traceroute example.com
```

Shows the path toward a destination where intermediate routers respond to the probes.

The command may need to be installed separately.

## Check the local loopback interface

```bash
ping 127.0.0.1
```

Tests local loopback networking.

## Test a specific application port

```bash
nc -vz 192.168.1.10 443
```

Tests whether a TCP connection can be established to the specified address and port, if `nc` is installed and the connection is permitted.

---

# 21. Common Networking Mistakes

### Mistake 1: Assuming every IP octet can exceed 255

Incorrect:

`192.168.1.300`

Correct:

Every IPv4 octet must be between 0 and 255.

### Mistake 2: Confusing network bits with host bits

For `/24`:

- Network bits = 24
- Host bits = 8

Formula:

`Host bits = 32 - prefix`

### Mistake 3: Assuming /24 always means a public network

Incorrect.

`192.168.1.0/24` is a private IPv4 subnet.

CIDR notation describes the size of an address block, not whether it is public or private.

### Mistake 4: Assuming the first octet determines the subnet size

This is not true with modern classless addressing.

For example:

`192.168.1.0/26`

The subnet contains 64 addresses, not 256.

### Mistake 5: Forgetting the block size

For a `/26`:

`256 - 192 = 64`

The fourth-octet ranges increase by 64.

### Mistake 6: Confusing a switch with a router

A typical Layer 2 switch forwards Ethernet frames within a local network.

A router forwards IP packets between networks.

### Mistake 7: Assuming a public IP guarantees internet access

Internet access also depends on routing, security controls, and correct application configuration.

### Mistake 8: Assuming 0.0.0.0 and 127.0.0.1 mean the same thing

They have different purposes.

- `127.0.0.1` is the IPv4 loopback address.
- `0.0.0.0` is the unspecified IPv4 address; in a listening socket, it commonly means all local IPv4 interfaces.

---

# 22. Networking Interview Questions and Answers

## Q1. What is an IP address?

An IP address is a logical address used to identify a network interface and support IP communication.

## Q2. How many bits are in IPv4?

IPv4 uses 32 bits, divided into four 8-bit octets.

## Q3. Why does an IPv4 octet range from 0 to 255?

An octet contains eight bits.

The number of possible combinations is:

`2^8 = 256`

Since the values begin at zero, the range is:

`0–255`

## Q4. What is classful addressing?

Classful addressing is the historical IPv4 addressing scheme that divided address space into Classes A, B, C, D, and E.

## Q5. What is classless addressing?

Classless addressing uses explicit CIDR prefix lengths instead of fixed class boundaries.

## Q6. What does /24 mean?

It means that the first 24 bits are the network prefix.

The remaining eight bits are host bits.

## Q7. How many addresses are in a /24 subnet?

Host bits:

`32 - 24 = 8`

Total:

`2^8 = 256`

Traditional usable hosts:

`256 - 2 = 254`

## Q8. How many addresses are in a /26 subnet?

Host bits:

`32 - 26 = 6`

Total:

`2^6 = 64`

Traditional usable hosts:

`64 - 2 = 62`

## Q9. What is a subnet mask?

A subnet mask identifies the network and host portions of an IPv4 address.

Example:

`255.255.255.0 = /24`

## Q10. What is the loopback address?

The most common IPv4 loopback address is:

`127.0.0.1`

It allows a system to communicate with itself.

## Q11. What is the difference between a switch and a router?

A typical Layer 2 switch forwards Ethernet frames using MAC addresses. A router forwards IP packets between networks using routing information.

## Q12. What is a default gateway?

A default gateway is the next-hop device used to forward traffic when no more specific matching route is available.

## Q13. What is a routing table?

A routing table contains destination prefixes and their corresponding next hops or targets.

## Q14. What is the default IPv4 route?

The default IPv4 route is:

`0.0.0.0/0`

It matches any IPv4 destination, but more specific routes take precedence.

## Q15. What is longest-prefix match?

It is the routing principle of selecting the most specific matching destination prefix.

## Q16. What is the difference between a public and private IP address?

Private addresses are reserved for use within private networks and are not globally routable across the public internet. Public addresses can be globally routed, subject to routing policies and other restrictions.

## Q17. How many host bits are present in /27?

`32 - 27 = 5`

There are five host bits.

## Q18. How many total addresses are in /27?

`2^5 = 32`

There are 32 total addresses.

Traditionally, 30 are usable for ordinary hosts.

## Q19. What is the block size of a /26 subnet?

The subnet mask is:

`255.255.255.192`

Block size:

`256 - 192 = 64`

## Q20. An EC2 instance has a public IP but cannot be reached. What do you check?

Check the application listener, security group, subnet route table, Internet Gateway, public IP configuration, NACLs, operating-system firewall, and DNS if applicable.

---

# 23. Practice Exercises

Try solving these questions before checking the answers.

### Exercise 1

How many bits are in an IPv4 address?

### Exercise 2

Why does an IPv4 octet range from 0 to 255?

### Exercise 3

What is the traditional default subnet mask for Class B?

### Exercise 4

What does `/26` mean?

### Exercise 5

How many host bits are present in `/27`?

### Exercise 6

How many total addresses are in a `/27` subnet?

### Exercise 7

What is the subnet mask for `/26`?

### Exercise 8

Calculate the block size for `/26`.

### Exercise 9

Find the network address for:

`192.168.1.70/26`

### Exercise 10

Find the network address for:

`192.168.1.100/27`

### Exercise 11

Find the network address for:

`172.16.20.15/20`

### Exercise 12

What is the most commonly used IPv4 loopback address?

### Exercise 13

What is the difference between a switch and a router?

### Exercise 14

What is the default IPv4 route?

### Exercise 15

What does longest-prefix match mean?

---

# 24. Practice Exercise Answers

### Answer 1

IPv4 contains 32 bits.

### Answer 2

An octet contains eight bits:

`2^8 = 256` possible values.

The values range from 0 to 255.

### Answer 3

Class B's historical default subnet mask is:

`255.255.0.0`

CIDR:

`/16`

### Answer 4

`/26` means 26 network bits and six host bits.

### Answer 5

`32 - 27 = 5`

### Answer 6

`2^5 = 32` total addresses.

### Answer 7

`255.255.255.192`

### Answer 8

`256 - 192 = 64`

### Answer 9

`192.168.1.64`

### Answer 10

`192.168.1.96`

### Answer 11

`172.16.16.0`

### Answer 12

`127.0.0.1`

### Answer 13

A typical Layer 2 switch connects devices within a local network. A router forwards packets between IP networks.

### Answer 14

`0.0.0.0/0`

### Answer 15

Longest-prefix match selects the most specific matching route for a destination IP address.

---

# 25. Final Revision Cheat Sheet

## IPv4 fundamentals

- IPv4 address length: 32 bits
- Number of octets: 4
- Bits per octet: 8
- Minimum octet value: 0
- Maximum octet value: 255

## Classful addressing

- Class A: first-octet range 1–126 for traditional unicast allocations; default `/8`
- Class B: first-octet range 128–191; default `/16`
- Class C: first-octet range 192–223; default `/24`
- Class D: first-octet range 224–239; multicast
- Class E: first-octet range 240–255; experimental/reserved

## Important formulas

**Host bits**

`Host bits = 32 - prefix length`

**Total IPv4 addresses**

`Total addresses = 2^(host bits)`

**Traditional usable host count**

`Usable hosts = 2^(host bits) - 2`

This last formula applies to ordinary subnets that reserve network and broadcast addresses, not to `/31` and `/32` cases.

**Block size**

`Block size = 256 - subnet mask value in the changing octet`

Use the octet where the subnet mask is neither 255 nor 0.

## Important subnet sizes

| CIDR | Total addresses | Traditional usable hosts |
|---|---:|---:|
| /24 | 256 | 254 |
| /25 | 128 | 126 |
| /26 | 64 | 62 |
| /27 | 32 | 30 |
| /28 | 16 | 14 |
| /29 | 8 | 6 |
| /30 | 4 | 2 |
| /31 | 2 | Point-to-point exception |
| /32 | 1 | Single-address prefix |

## Networking devices

- Switch: forwards Ethernet frames within a local network.
- Router: forwards IP packets between networks.
- Routing table: determines the next hop or target for a destination.
- Default route: `0.0.0.0/0`.
- Loopback: `127.0.0.1`.
- Private IPv4 ranges: `10.0.0.0/8`, `172.16.0.0/12`, and `192.168.0.0/16`.

## AWS networking

- VPC: logically isolated virtual network.
- Subnet: smaller IP address range within a VPC.
- Route table: controls traffic forwarding.
- Internet Gateway: supports internet connectivity for appropriately configured VPC resources.
- NAT Gateway: allows suitable outbound connectivity from private networks.
- Security group: stateful virtual firewall.
- Network ACL: stateless subnet-level network filter.

---

## Learning Recommendation

Before moving on to advanced AWS networking, practise these three skills until you can perform them without referring to notes:

1. Convert an IPv4 octet between binary and decimal.
2. Calculate the network address, broadcast address, and address range from a CIDR block.
3. Explain how a switch, router, and routing table work together when traffic travels between two different subnets.

Once these are clear, move on to **subnetting practice, AWS VPC CIDR planning, route tables, and troubleshooting EC2 connectivity**.