# AWS Networking Fundamentals — Part 1
## VPC | Subnetting | Internet Gateway | Route Tables

**Learning Path:** AWS Cloud / DevOps / SRE  
**Level:** Beginner to Intermediate  
**Purpose:** Practical learning, AWS troubleshooting, and interview preparation

---

# Table of Contents

1. [Introduction to AWS Networking](#1-introduction-to-aws-networking)
2. [Amazon VPC](#2-amazon-vpc)
3. [IP Addressing Fundamentals](#3-ip-addressing-fundamentals)
4. [CIDR Notation](#4-cidr-notation)
5. [Subnetting](#5-subnetting)
6. [Public and Private Subnets](#6-public-and-private-subnets)
7. [Internet Gateway](#7-internet-gateway-igw)
8. [Route Tables](#8-route-tables)
9. [How VPC, Subnets, IGW, and Route Tables Work Together](#9-how-vpc-subnets-igw-and-route-tables-work-together)
10. [Practical AWS Scenario](#10-practical-aws-scenario)
11. [Troubleshooting](#11-troubleshooting)
12. [Interview Questions and Answers](#12-interview-questions-and-answers)
13. [Key Takeaways](#13-key-takeaways)

---

# 1. Introduction to AWS Networking

## 1.1 What Is Networking?

Networking allows computers, servers, applications, and other devices to communicate with each other.

For example:

- A user accesses a website from a laptop.
- An EC2 instance communicates with a database.
- An application server downloads software updates.
- An application connects to an external API.

To communicate, devices need network addresses and a way to determine where traffic should go.

Important networking components include:

- IP addresses
- Subnets
- Routers
- Switches
- Route tables
- Firewalls
- Gateways
- DNS

## 1.2 What Is AWS Networking?

AWS networking provides the infrastructure required to connect cloud resources securely.

Examples:

- EC2 instances communicate with each other.
- Public web servers receive requests from internet users.
- Private application servers communicate with databases.
- Private resources access AWS services through VPC endpoints.
- Applications communicate with external services through appropriate network paths.

Important AWS networking components:

| Component | Purpose |
|---|---|
| VPC | Provides an isolated virtual network |
| Subnet | Divides a VPC IP address range into smaller ranges |
| Internet Gateway | Connects a VPC to the internet when routing and addressing permit |
| Route Table | Determines where network traffic should go |
| Security Group | Controls allowed inbound and outbound traffic for supported resources |
| Network ACL | Controls inbound and outbound traffic at the subnet boundary |
| NAT Gateway | Allows private-subnet resources to initiate connections to external destinations |
| VPC Endpoint | Provides private connectivity to supported AWS services or endpoint services |
| Elastic Network Interface | Provides a virtual network interface for an EC2 instance |
| Route 53 | Provides DNS services |

### Important distinction

A route table determines the next destination for network traffic.

A security group determines which traffic is allowed to reach or leave a resource.

These perform different jobs.

---

# 2. Amazon VPC

## 2.1 What Is a VPC?

VPC stands for **Virtual Private Cloud**.

An Amazon VPC is a logically isolated virtual network in AWS where you can launch resources such as EC2 instances.

When creating a VPC, you choose an IPv4 CIDR block that defines its IPv4 address space.

For example:

```text
VPC CIDR: 10.0.0.0/16
```

This VPC has 65,536 total IPv4 addresses.

Within the VPC, you can create subnets, associate route tables, configure security groups, and connect supported networking services.

## 2.2 Why Do We Need a VPC?

Suppose you are building an application with three components:

1. A web server.
2. An application server.
3. A database server.

You might want the web server to receive public internet requests while keeping the application server and database private.

A VPC lets you design this network architecture.

Example:

```text
                         INTERNET
                            |
                            |
                    Internet Gateway
                            |
                 +---------------------+
                 |     AWS VPC         |
                 |   10.0.0.0/16       |
                 |                     |
                 |  Public Subnet      |
                 |  Web Server         |
                 |                     |
                 |  Private Subnet     |
                 |  Application Server |
                 |                     |
                 |  Private Subnet     |
                 |  Database Server    |
                 +---------------------+
```

The diagram illustrates the logical architecture. Actual connectivity depends on the subnet route tables, IP addressing, security groups, network ACLs, and application configuration.

## 2.3 Creating a VPC

Example configuration:

| Setting | Value |
|---|---|
| Name | `production-vpc` |
| IPv4 CIDR | `10.0.0.0/16` |
| IPv6 CIDR | None, for this example |
| Tenancy | Default |

After creating the VPC, you can create subnets inside its address space.

### AWS Console steps

1. Open the AWS Management Console.
2. Navigate to **VPC**.
3. Select **Your VPCs**.
4. Select **Create VPC**.
5. Choose the VPC-only option if you want to create the VPC without automatically creating additional resources.
6. Enter a name and IPv4 CIDR block.
7. Create the VPC.

The VPC-only option is useful when you want to understand and configure each networking component separately.

## 2.4 VPC CIDR

CIDR stands for **Classless Inter-Domain Routing**.

Example:

```text
10.0.0.0/16
```

Here:

- `10.0.0.0` is the network address.
- `/16` indicates that the first 16 bits represent the network prefix.
- The remaining 16 bits represent the address space available within the network.

Total IPv4 addresses:

```text
2^(32 - 16)
= 2^16
= 65,536
```

## 2.5 Important AWS VPC CIDR Rules

For an IPv4 CIDR block assigned to a VPC, AWS supports sizes from `/16` through `/28`.

Examples:

| VPC CIDR | Total IPv4 addresses |
|---|---:|
| `10.0.0.0/16` | 65,536 |
| `10.0.0.0/20` | 4,096 |
| `10.0.0.0/22` | 1,024 |
| `10.0.0.0/23` | 512 |
| `10.0.0.0/24` | 256 |
| `10.0.0.0/28` | 16 |

**Important:** AWS does not allow you to create a VPC with exactly 500 IPv4 addresses using a single IPv4 CIDR block. CIDR block sizes follow powers of two.

A `/23` VPC provides 512 total IPv4 addresses, not 500.

When planning VPCs, consider future growth and avoid overlapping IP ranges with connected networks.

## 2.6 Can a VPC Have Multiple CIDR Blocks?

Yes. You can associate additional IPv4 CIDR blocks with a VPC, subject to AWS restrictions.

For example:

```text
Primary VPC CIDR:   10.0.0.0/16
Additional CIDR:   172.16.0.0/16
```

This expands the address space available for subnet creation.

However, you must plan the additional CIDR carefully. It must satisfy AWS requirements and must not create prohibited overlaps with existing VPC CIDR blocks or connected networks.

## 2.7 Can Two Subnets in a VPC Have Overlapping CIDRs?

No.

For example, suppose a VPC has this CIDR:

```text
10.0.0.0/16
```

You create:

```text
Subnet A: 10.0.1.0/24
```

You cannot create another subnet:

```text
Subnet B: 10.0.1.128/25
```

Why?

The ranges are:

```text
Subnet A: 10.0.1.0 - 10.0.1.255
Subnet B: 10.0.1.128 - 10.0.1.255
```

Subnet B's entire range is contained within Subnet A's range.

AWS rejects this overlapping subnet CIDR.

### Remember

A subnet's IPv4 CIDR must be contained within an associated VPC IPv4 CIDR block and must not overlap with another subnet in the VPC.

---

# 3. IP Addressing Fundamentals

## 3.1 What Is an IP Address?

An IP address identifies a network interface on an IP network.

IPv4 addresses contain 32 bits.

Example:

```text
192.168.1.10
```

An IPv4 address is divided into four octets.

```text
192       168       1        10
  |         |       |         |
 8 bits   8 bits  8 bits    8 bits
```

Each octet contains 8 bits.

Therefore:

```text
4 × 8 = 32 bits
```

Each octet has a decimal range from 0 to 255.

## 3.2 Why Does an Octet Range from 0 to 255?

An octet contains 8 bits.

The smallest value is:

```text
00000000 = 0
```

The largest value is:

```text
11111111 = 255
```

The decimal place values of an 8-bit octet are:

```text
128 + 64 + 32 + 16 + 8 + 4 + 2 + 1
= 255
```

There are 256 possible combinations, including zero.

Therefore, an octet has 256 possible values: 0 through 255.

## 3.3 Private IPv4 Address Ranges

The following IPv4 ranges are reserved for private networking.

| Private range | CIDR | Typical use |
|---|---|---|
| `10.0.0.0 – 10.255.255.255` | `10.0.0.0/8` | Large private networks |
| `172.16.0.0 – 172.31.255.255` | `172.16.0.0/12` | Private enterprise networks |
| `192.168.0.0 – 192.168.255.255` | `192.168.0.0/16` | Home and smaller private networks |

Examples of private IPv4 addresses:

```text
10.0.1.10
172.16.5.20
192.168.1.100
```

Private IP addresses are not globally routable over the public internet.

They can communicate within private networks and with external destinations through appropriate network connectivity, such as NAT.

## 3.4 Public and Private IP Addresses

**Public IP address**

A globally routable IPv4 address, subject to normal routing and security controls.

**Private IP address**

An address used within a private network that is not globally routable on the public internet.

In AWS, an EC2 instance can have a private IP address, a public IPv4 address, or both, depending on its configuration.

Having a public IP does not automatically make an instance reachable. Routes, security groups, network ACLs, and the service listening on the relevant port must also permit connectivity.

---

# 4. CIDR Notation

## 4.1 What Does `/24` Mean?

Consider:

```text
192.168.1.0/24
```

The `/24` means:

- 24 network-prefix bits.
- 8 remaining host bits.

Calculation:

```text
32 - 24 = 8 host bits
```

Total addresses:

```text
2^8 = 256
```

For this example, the address range is:

```text
192.168.1.0 - 192.168.1.255
```

In a typical IPv4 subnet, the first address identifies the subnet and the last address is its broadcast address.

AWS reserves five IPv4 addresses in each IPv4 subnet, so an AWS `/24` subnet normally has:

```text
256 - 5 = 251 usable IPv4 addresses
```

## 4.2 CIDR and Subnet Masks

The CIDR prefix corresponds to a subnet mask.

| CIDR | Subnet mask | Host bits | Total addresses |
|---|---|---:|---:|
| `/16` | `255.255.0.0` | 16 | 65,536 |
| `/20` | `255.255.240.0` | 12 | 4,096 |
| `/21` | `255.255.248.0` | 11 | 2,048 |
| `/22` | `255.255.252.0` | 10 | 1,024 |
| `/23` | `255.255.254.0` | 9 | 512 |
| `/24` | `255.255.255.0` | 8 | 256 |
| `/25` | `255.255.255.128` | 7 | 128 |
| `/26` | `255.255.255.192` | 6 | 64 |
| `/27` | `255.255.255.224` | 5 | 32 |
| `/28` | `255.255.255.240` | 4 | 16 |

The table shows total addresses. In AWS, subtract five from the subnet's total IPv4 addresses to calculate the number available for resource assignment.

For example, a `/28` subnet has 16 total addresses and 11 AWS-usable IPv4 addresses.

## 4.3 How to Calculate Total IP Addresses

Formula:

```text
Total IPv4 addresses = 2^(32 - CIDR prefix)
```

Example:

```text
CIDR = 10.0.0.0/20

Host bits = 32 - 20
          = 12

Total addresses = 2^12
                = 4,096
```

## 4.4 How to Calculate AWS-Usable IPv4 Addresses

For ordinary AWS IPv4 subnets:

```text
AWS-usable IPv4 addresses = Total IPv4 addresses - 5
```

Example:

```text
Subnet: 10.0.1.0/24

Total addresses = 256
AWS-reserved addresses = 5

AWS-usable addresses = 256 - 5
                     = 251
```

AWS reserves the first four and the last IPv4 address in each subnet.

Do not confuse AWS-usable addresses with the number of addresses an operating system might ordinarily consider usable in a non-AWS network.

## 4.5 Understanding `/24`, `/25`, and `/26`

Consider these subnet sizes:

| CIDR | Total addresses | AWS-usable addresses |
|---|---:|---:|
| `/24` | 256 | 251 |
| `/25` | 128 | 123 |
| `/26` | 64 | 59 |

A smaller numerical prefix, such as `/24`, provides a larger address space than `/25`.

A larger numerical prefix, such as `/26`, provides a smaller address space than `/25`.

Remember:

```text
Smaller prefix number = Larger network
Larger prefix number = Smaller network
```

## 4.6 How Does `/25` Divide a `/24` Network?

Suppose you have:

```text
192.168.1.0/24
```

This range contains 256 addresses.

You can divide it into two `/25` ranges:

```text
First subnet:  192.168.1.0/25
Second subnet: 192.168.1.128/25
```

Ranges:

```text
192.168.1.0/25
192.168.1.0 - 192.168.1.127

192.168.1.128/25
192.168.1.128 - 192.168.1.255
```

Each subnet contains 128 addresses.

In AWS, each `/25` subnet has 123 usable IPv4 addresses.

The two ranges do not overlap.

## 4.7 How to Calculate the Subnet Block Size

For masks that divide a subnet within an octet, use:

```text
Block size = 256 - subnet mask value in the relevant octet
```

Example:

```text
CIDR: 192.168.0.0/20
Mask: 255.255.240.0
```

The relevant octet is the third octet.

```text
Block size = 256 - 240
           = 16
```

The subnet boundaries occur at third-octet increments of 16:

```text
192.168.0.0/20
192.168.16.0/20
192.168.32.0/20
192.168.48.0/20
192.168.64.0/20
```

Each `/20` contains 4,096 addresses.

For example:

```text
192.168.32.0/20
```

Range:

```text
Start: 192.168.32.0
End:   192.168.47.255
```

The last address is `192.168.47.255`, not `192.168.32.255`.

Why? Because `/20` uses the first 20 bits as the network prefix, leaving 12 bits for the remaining address range.

## 4.8 Why Can't We Create `10.0.3.0/23` After `10.0.2.0/24`?

Suppose the existing subnet is:

```text
10.0.2.0/24
```

You want to create:

```text
10.0.3.0/23
```

A `/23` has this mask:

```text
255.255.254.0
```

The block size in the third octet is:

```text
256 - 254 = 2
```

Therefore, valid `/23` network boundaries in the third octet include:

```text
10.0.0.0/23
10.0.2.0/23
10.0.4.0/23
10.0.6.0/23
```

`10.0.3.0/23` is not the canonical network address for that `/23` range. It falls within the network beginning at `10.0.2.0/23`.

That range is:

```text
10.0.2.0 - 10.0.3.255
```

It overlaps with the existing `10.0.2.0/24` subnet.

The next non-overlapping `/23` network is:

```text
10.0.4.0/23
```

However, this is valid only if the VPC has sufficient address space covering that range.

---

# 5. Subnetting

## 5.1 What Is a Subnet?

A subnet is a smaller IP address range within a VPC.

For example:

```text
VPC CIDR: 10.0.0.0/16
```

You can divide it into subnets:

```text
Subnet A: 10.0.1.0/24
Subnet B: 10.0.2.0/24
Subnet C: 10.0.3.0/24
Subnet D: 10.0.4.0/24
```

Each subnet has its own IP address range.

AWS creates a subnet in one Availability Zone.

For high availability, applications often use multiple subnets across multiple Availability Zones.

## 5.2 Why Do We Need Subnets?

Subnets help you:

- Separate public-facing resources from private resources.
- Place resources in different Availability Zones.
- Assign different route tables to different network segments.
- Organize IP address space.
- Apply subnet-level network ACLs.
- Design architectures with appropriate security boundaries.

Example:

```text
VPC: 10.0.0.0/16
|
+-- Public Subnet A
|   +-- Web Server
|
+-- Public Subnet B
|   +-- Web Server
|
+-- Private Subnet A
|   +-- Application Server
|
+-- Private Subnet B
|   +-- Application Server
|
+-- Database Subnet A
|   +-- Database
|
+-- Database Subnet B
    +-- Database
```

The subnets can be placed in different Availability Zones according to the architecture's requirements.

## 5.3 Example: Creating Multiple Subnets

Suppose your VPC is:

```text
10.0.0.0/22
```

Its total address space is:

```text
10.0.0.0 - 10.0.3.255
```

It contains 1,024 total IPv4 addresses.

You could create the following `/25` subnets:

| Subnet | CIDR | Total addresses | AWS-usable addresses |
|---|---|---:|---:|
| Subnet 1 | `10.0.0.0/25` | 128 | 123 |
| Subnet 2 | `10.0.0.128/25` | 128 | 123 |
| Subnet 3 | `10.0.1.0/25` | 128 | 123 |
| Subnet 4 | `10.0.1.128/25` | 128 | 123 |
| Subnet 5 | `10.0.2.0/25` | 128 | 123 |
| Subnet 6 | `10.0.2.128/25` | 128 | 123 |
| Subnet 7 | `10.0.3.0/25` | 128 | 123 |
| Subnet 8 | `10.0.3.128/25` | 128 | 123 |

These eight subnets consume all 1,024 addresses in the VPC's `/22` range.

There are no unallocated IPv4 addresses left in that VPC CIDR for another subnet.

## 5.4 How Does AWS Know How Many IPs a Subnet Can Use?

When creating a subnet, you specify its CIDR block.

For example:

```text
10.0.2.128/25
```

AWS calculates the range represented by that CIDR:

```text
Start: 10.0.2.128
End:   10.0.2.255
```

AWS reserves five IPv4 addresses within the subnet. The remaining addresses are available for assignment to resources.

AWS also checks that:

1. The subnet range is contained within an associated VPC CIDR block.
2. The subnet does not overlap with another subnet.
3. The CIDR block satisfies AWS subnet requirements.

If a proposed subnet falls outside the VPC's address space or overlaps with an existing subnet, AWS rejects it.

## 5.5 What Happens When a Subnet Runs Out of IP Addresses?

Suppose a subnet has CIDR:

```text
10.0.1.0/25
```

It contains:

```text
128 total addresses
123 AWS-usable addresses
```

If all 123 usable addresses are allocated, AWS cannot assign another IPv4 address from that subnet.

Possible solutions include:

- Launching resources in another suitable subnet with available addresses.
- Planning larger subnets when designing the VPC.
- Associating an additional VPC CIDR block when appropriate, then creating new subnets from the additional address space.
- Redesigning the subnet allocation where necessary.

**Important:** Creating another subnet does not create new addresses beyond the VPC's existing CIDR ranges. The subnet must use available address space within an associated VPC CIDR block.

## 5.6 What Is a Public Subnet?

A public subnet is a subnet whose associated route table has a route to an Internet Gateway for internet-bound IPv4 traffic.

Typical route:

```text
Destination: 0.0.0.0/0
Target:      Internet Gateway
```

An EC2 instance generally also needs a public IPv4 address or Elastic IP for direct internet communication over IPv4.

The instance's security group, network ACL, operating system firewall, and application must also permit the required traffic.

## 5.7 What Is a Private Subnet?

A private subnet does not have a direct route to an Internet Gateway for general internet-bound IPv4 traffic.

Its resources might communicate through:

- A NAT Gateway.
- VPC endpoints.
- Other private resources.
- VPN or Direct Connect connectivity.
- Transit Gateway or other routed network connections.

Example private-subnet route table:

```text
Destination: 10.0.0.0/16
Target:      local
```

If the subnet needs outbound internet access through a NAT Gateway, its route table might also contain:

```text
Destination: 0.0.0.0/0
Target:      NAT Gateway
```

## 5.8 How Do Resources Communicate Between Subnets?

Consider two EC2 instances:

```text
Instance A: 10.0.1.10/24
Instance B: 10.0.2.10/24
```

They are in different `/24` subnets.

Instance A recognizes that `10.0.2.10` is outside its local `/24` subnet.

It sends the traffic toward its network gateway. In an AWS VPC, the VPC's local routing provides connectivity between subnets by default, subject to applicable routing and security controls.

The destination instance must allow the required traffic through its security group and network ACL, and its application must be listening on the requested port.

---

# 6. Public and Private Subnets

## 6.1 Public Subnet Example

Suppose you deploy an EC2 web server.

```text
VPC CIDR: 10.0.0.0/16

Public subnet: 10.0.1.0/24
EC2 private IP: 10.0.1.10
EC2 public IP:  Assigned separately
```

Its route table contains:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | `local` |
| `0.0.0.0/0` | Internet Gateway |

The local route allows communication between resources covered by the VPC's local routing.

The Internet Gateway route allows internet-bound IPv4 traffic to use the Internet Gateway, subject to the other network requirements.

## 6.2 Private Subnet Example

Suppose you deploy an application server.

```text
VPC CIDR: 10.0.0.0/16

Private subnet: 10.0.2.0/24
EC2 private IP: 10.0.2.10
```

Its route table could contain:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | `local` |
| `0.0.0.0/0` | NAT Gateway |

The application server can initiate outbound internet connections through the NAT Gateway if the NAT Gateway and routing are configured correctly.

Unsolicited internet connections cannot initiate connections to the private instance through that NAT Gateway.

## 6.3 Can a Public Subnet Contain a Private IP?

Yes!

An EC2 instance in a public subnet still has a private IPv4 address.

Example:

```text
Private IP: 10.0.1.10
Public IP:  198.51.100.25  # Documentation example only
```

The private IP is used for communication within private network paths.

The public IP provides an internet-routable address mapping when configured and supported.

A subnet is not made public simply by assigning public IP addresses to its instances. Its routing configuration must provide a route to an Internet Gateway.

Likewise, adding an IGW route alone does not make an instance directly reachable if it lacks the necessary public addressing or security permissions.

---

# 7. Internet Gateway (IGW)

## 7.1 What Is an Internet Gateway?

An Internet Gateway is an AWS VPC component that enables communication between a VPC and the internet when the necessary routing, addressing, and security configuration is in place.

An IGW is horizontally scaled, redundant, and highly available within its AWS design.

It is not the same as a NAT Gateway.

## 7.2 Why Do We Need an IGW?

Consider a public web server.

Internet users must be able to send requests to the server.

For IPv4 internet connectivity, the typical configuration includes:

1. An Internet Gateway attached to the VPC.
2. A subnet route table with a route to the IGW.
3. A public IPv4 address or Elastic IP associated with the instance.
4. Security group rules allowing the required traffic.
5. Appropriate network ACL rules.
6. An application listening on the required port.

Without the required configuration, the server may not be reachable from the internet.

## 7.3 How to Create an Internet Gateway

AWS Console steps:

1. Open the VPC console.
2. Select **Internet gateways**.
3. Select **Create internet gateway**.
4. Give it a name, such as `production-igw`.
5. Create the Internet Gateway.
6. Select the gateway.
7. Choose **Actions → Attach to a VPC**.
8. Select the intended VPC.

Creating an Internet Gateway does not automatically attach it to your VPC or configure subnet routes.

## 7.4 How Does Internet Traffic Reach an EC2 Instance?

Example:

```text
User's Browser
      |
      | HTTPS request
      |
   Internet
      |
      v
Internet Gateway
      |
      v
VPC Route Table
      |
      v
Public Subnet
      |
      v
EC2 Instance
```

This simplified diagram assumes the required route, public addressing, security rules, and application configuration are present.

## 7.5 What Happens When an EC2 Instance Sends a Request to the Internet?

Suppose an EC2 instance has:

```text
Private IP: 10.0.1.10
Public IP:  Assigned public IPv4 address
```

The instance requests a website.

The subnet route table contains:

```text
0.0.0.0/0 → Internet Gateway
```

The instance sends traffic toward the Internet Gateway.

For an instance with an assigned public IPv4 address, AWS provides the public-to-private address mapping needed for internet communication.

The return traffic can reach the instance when the route and applicable security rules permit it.

## 7.6 Can a Private Instance Use an Internet Gateway Directly?

A typical private-subnet EC2 instance without a public IPv4 address cannot use the IGW route as a direct path to the public IPv4 internet.

Instead, for outbound IPv4 internet access, it can use a NAT Gateway in a public subnet.

Example:

```text
Private EC2
    |
    v
Private Route Table
    |
    | 0.0.0.0/0 → NAT Gateway
    |
    v
NAT Gateway
    |
    v
Public Subnet Route Table
    |
    | 0.0.0.0/0 → Internet Gateway
    |
    v
Internet
```

The NAT Gateway uses its public-side connectivity to enable supported outbound connections from private resources.

## 7.7 IGW vs NAT Gateway

| Feature | Internet Gateway | NAT Gateway |
|---|---|---|
| Primary purpose | Connects a VPC to the internet | Allows private resources to initiate outbound internet connections |
| Attached to | VPC | Created in a subnet |
| Used by public-subnet resources | Yes, with proper routes and public addressing | Not usually required for their direct internet access |
| Used by private-subnet resources | Not usually as a direct IPv4 internet path | Commonly used for outbound IPv4 internet access |
| Direct unsolicited inbound internet connections to private instances | Not provided by an IGW-only route | Not supported through NAT Gateway |
| Public addressing requirement | Public IPv4 addressing is needed for direct IPv4 internet communication | NAT Gateway in a public subnet typically uses an Elastic IP |

### Important interview point

An Internet Gateway does not automatically provide internet access to every resource in a VPC.

The resource's subnet routing, IP addressing, and security configuration must also be correct.

---

# 8. Route Tables

## 8.1 What Is a Route Table?

A route table contains rules called routes.

Each route specifies:

- A destination network.
- A target to which matching traffic should be sent.

A route table helps determine where traffic should go when a resource sends a packet.

Example:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | `local` |
| `0.0.0.0/0` | Internet Gateway |

The first route provides routing within the VPC's IPv4 address space.

The second route sends matching internet-bound IPv4 traffic to the Internet Gateway.

## 8.2 What Is the `local` Route?

When you create a VPC, AWS automatically provides a local route for each associated VPC IPv4 CIDR block.

For example:

```text
VPC CIDR: 10.0.0.0/16
```

The route table includes:

```text
Destination: 10.0.0.0/16
Target:      local
```

This allows resources within the VPC's routed address space to communicate with each other, subject to applicable network controls.

You generally do not need to create this local route manually.

## 8.3 What Is `0.0.0.0/0`?

In IPv4 routing, `0.0.0.0/0` represents all IPv4 destinations.

It is commonly used as a default route.

For example:

```text
Destination: 0.0.0.0/0
Target:      Internet Gateway
```

This means traffic that matches no more-specific route can be sent toward the Internet Gateway.

However, the route alone does not guarantee connectivity. The target, IP addressing, and security configuration must also support the traffic.

## 8.4 How Does AWS Choose a Route?

AWS routing uses the **longest-prefix match** rule.

When multiple routes match a destination, the most specific matching route generally takes precedence.

Example:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | `local` |
| `10.0.1.0/24` | Another valid route target, where configured and applicable |
| `0.0.0.0/0` | Internet Gateway |

Traffic destined for `10.0.1.10` matches both `10.0.0.0/16` and `10.0.1.0/24`.

The `/24` route is more specific than `/16`.

Therefore, the `/24` route takes precedence if it is a valid route for that traffic and AWS supports the target and route configuration.

In normal VPC designs, do not change or replace routing behavior without understanding the implications for the VPC's local route.

## 8.5 What Is the Main Route Table?

When you create a VPC, AWS creates a main route table.

Every subnet is associated with a route table.

If you do not explicitly associate a subnet with another route table, it uses the main route table.

You can create additional route tables and associate them with individual subnets.

This allows different subnets to use different routing configurations.

## 8.6 Public Subnet Route Table

Example:

```text
VPC CIDR: 10.0.0.0/16
Internet Gateway: igw-example
```

Routes:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | `local` |
| `0.0.0.0/0` | `igw-example` |

This is a typical route table for a public subnet.

The subnet's EC2 instances still need appropriate public addressing and security configuration for direct IPv4 internet connectivity.

## 8.7 Private Subnet Route Table

Example:

```text
VPC CIDR: 10.0.0.0/16
NAT Gateway: nat-example
```

Routes:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | `local` |
| `0.0.0.0/0` | `nat-example` |

The default route sends traffic for destinations outside the VPC's local route toward the NAT Gateway.

The NAT Gateway must be configured correctly, normally in a public subnet with a route to the IGW, to provide outbound IPv4 internet access.

## 8.8 Database Subnet Route Table

A database subnet might have only:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | `local` |

This means the subnet has no default route to an Internet Gateway or NAT Gateway in this example.

The database may still communicate with other VPC resources through the local route, provided the applicable network rules permit it.

Private databases often do not need general internet access.

## 8.9 How to Create a Route Table

AWS Console steps:

1. Open the VPC console.
2. Select **Route tables**.
3. Choose **Create route table**.
4. Enter a name.
5. Select the VPC.
6. Create the route table.
7. Select the route table.
8. Open the **Routes** tab.
9. Choose **Edit routes**.
10. Add the required destination and target.
11. Save the changes.
12. Open **Subnet associations**.
13. Edit subnet associations.
14. Associate the intended subnet.

Creating a route table does not automatically associate every subnet with it.

## 8.10 What Happens If You Forget to Associate the Route Table?

Suppose you create:

```text
public-route-table
```

You add:

```text
0.0.0.0/0 → Internet Gateway
```

But your subnet remains associated with a different route table.

The subnet does not automatically use the new route table just because you created it.

You must associate the intended subnet with the route table.

Otherwise, traffic follows the routes in the route table actually associated with that subnet.

## 8.11 What Happens If You Add an IGW Route to a Private Subnet?

Suppose you add:

```text
0.0.0.0/0 → Internet Gateway
```

to a private subnet's route table.

The route table now has a route toward the IGW, but a private EC2 instance without a public IPv4 address still cannot use that route as a direct IPv4 internet connection.

Adding the route does not automatically assign public IP addresses or turn private instances into publicly reachable instances.

**Important:** Route-table configuration is only one part of network connectivity.

---

# 9. How VPC, Subnets, IGW, and Route Tables Work Together

Consider a simple three-tier application.

```text
                         INTERNET
                            |
                            |
                     Internet Gateway
                            |
              +-------------+-------------+
              |                           |
              |                           |
       Public Subnet A             Public Subnet B
       10.0.1.0/24                 10.0.2.0/24
              |                           |
         Web Server                  Web Server
              |                           |
              +-------------+-------------+
                            |
                      Application
                            |
                 +----------+----------+
                 |                     |
          Private Subnet A       Private Subnet B
          10.0.11.0/24           10.0.12.0/24
                 |                     |
           App Server              App Server
                 |                     |
                 +----------+----------+
                            |
                    Database Subnets
                            |
                     Database Servers
```

This is a conceptual architecture. The actual network must be configured with appropriate routes and security rules.

## Traffic Flow 1: Internet User to Web Server

1. A user requests the website.
2. The request reaches the web server's public IPv4 address or a public load balancer.
3. The Internet Gateway provides the VPC's internet connectivity.
4. The route table and network configuration must support the traffic.
5. Security groups and network ACLs must allow the required traffic.
6. The web server processes the request.

## Traffic Flow 2: Web Server to Application Server

1. The web server sends a request to the application server's private IP.
2. The destination is within the VPC's address space.
3. The VPC's local route provides routing between the subnets.
4. Security groups and network ACLs must permit the connection.
5. The application server receives the request.

## Traffic Flow 3: Application Server to Database

1. The application server sends a database request.
2. The request is routed using the VPC's local route, assuming the destination is within the VPC's routed address space.
3. The database security group must permit the application's connection.
4. The database processes the request and returns a response.

## Traffic Flow 4: Private Application Server to the Internet

1. The application server requests an external resource.
2. Its route table sends the traffic to a NAT Gateway.
3. The NAT Gateway uses its public subnet's internet route.
4. The Internet Gateway provides internet connectivity.
5. Return traffic is delivered through the established NAT connection.

This is a common architecture for private application servers that need to download packages, retrieve updates, or connect to external APIs.

---

# 10. Practical AWS Scenario

## Scenario: Deploy a Web Server and an Application Server

### Requirements

You need to create:

- One VPC.
- One public subnet.
- One private subnet.
- One Internet Gateway.
- One public route table.
- One private route table.
- One web server.
- One application server.

### Step 1: Create the VPC

```text
Name: production-vpc
CIDR: 10.0.0.0/16
```

### Step 2: Create the Subnets

Public subnet:

```text
Name: public-subnet
CIDR: 10.0.1.0/24
```

Private subnet:

```text
Name: private-subnet
CIDR: 10.0.2.0/24
```

These CIDR ranges do not overlap.

### Step 3: Create and Attach the Internet Gateway

```text
Name: production-igw
```

Attach it to:

```text
production-vpc
```

### Step 4: Configure the Public Route Table

Routes:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | `local` |
| `0.0.0.0/0` | Internet Gateway |

Associate this route table with:

```text
public-subnet
```

### Step 5: Configure the Private Route Table

For a private application server that does not need general internet access:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | `local` |

Associate this route table with:

```text
private-subnet
```

If the application server needs outbound IPv4 internet access, you can create a NAT Gateway in a public subnet and add:

```text
0.0.0.0/0 → NAT Gateway
```

to the private route table.

A NAT Gateway incurs charges, so consider the architecture and cost before deploying it.

### Step 6: Launch the Web Server

Launch an EC2 instance in:

```text
public-subnet
```

Configure public IPv4 addressing if direct internet access is required.

Security group example:

| Direction | Protocol | Port | Source |
|---|---|---:|---|
| Inbound | TCP | 80 | `0.0.0.0/0` |
| Inbound | TCP | 443 | `0.0.0.0/0` |
| Inbound | TCP | 22 | Your trusted public IP, if SSH is required |

Only open ports required by your application. For production systems, prefer managed access methods such as AWS Systems Manager Session Manager where suitable.

### Step 7: Launch the Application Server

Launch an EC2 instance in:

```text
private-subnet
```

Do not assign a public IPv4 address if it does not need direct public internet access.

Example security group rule:

| Direction | Protocol | Port | Source |
|---|---|---:|---|
| Inbound | TCP | 8080 | Web server security group |

This allows the web server tier to reach the application tier on port 8080, assuming the application is listening on that port.

The application server's outbound security rules and any network ACLs must also permit the necessary traffic.

### Step 8: Test Connectivity

Test the web server from an authorized external client.

Test the application server from the web server using its private IP.

For example:

```bash
curl http://10.0.2.10:8080
```

Replace the example IP with the actual private IP of the application server.

This test assumes the application is listening on port 8080 and the network and security configurations permit the connection.

---

# 11. Troubleshooting

## Problem 1: EC2 Has a Public IP but the Website Is Not Reachable

Check:

1. Is the Internet Gateway attached to the correct VPC?
2. Does the subnet's associated route table contain `0.0.0.0/0 → IGW`?
3. Does the instance have a public IPv4 address or Elastic IP?
4. Does the security group allow inbound traffic on the application port?
5. Are network ACLs permitting the traffic and return traffic?
6. Is the application running?
7. Is the application listening on the correct interface and port?
8. Is the operating system firewall blocking traffic?

Useful commands:

```bash
sudo systemctl status nginx
```

```bash
sudo ss -lntp
```

```bash
curl -I http://localhost
```

These commands help determine whether the web server service is running and listening.

## Problem 2: Private EC2 Cannot Download Software Packages

Check:

1. Does the subnet's route table contain a route to a NAT Gateway?
2. Is the NAT Gateway in a public subnet?
3. Does the public subnet's route table contain `0.0.0.0/0 → IGW`?
4. Does the NAT Gateway have the required public-side configuration, including an Elastic IP?
5. Is the NAT Gateway available?
6. Are outbound security group rules and network ACLs permitting the traffic?
7. Is DNS resolution working?
8. Can the destination service be reached on its required port?

Example:

```bash
curl -I https://example.com
```

This tests connectivity to an example HTTPS destination; it does not prove that all package repositories are reachable.

## Problem 3: Two EC2 Instances Cannot Communicate

Check:

1. Are both instances in the intended VPCs?
2. Is the destination private IP correct?
3. Does the VPC routing provide a path between the subnets?
4. Does the destination security group allow the source?
5. Are network ACLs permitting traffic in both directions?
6. Is the destination service listening on the expected port?
7. Is the operating system firewall blocking traffic?

Example:

```bash
ping 10.0.2.10
```

Note that ping may fail even when the network path is otherwise valid, because ICMP may be blocked.

A TCP connection test is often more useful when troubleshooting a specific application port.

## Problem 4: AWS Rejects a Subnet CIDR

Check:

1. Is the proposed subnet CIDR within an associated VPC CIDR block?
2. Does the proposed subnet overlap with an existing subnet?
3. Is the CIDR aligned to a valid network boundary?
4. Does the subnet meet AWS's minimum and maximum IPv4 subnet size requirements?

Example:

Existing subnet:

```text
10.0.1.0/24
```

Invalid overlapping subnet:

```text
10.0.1.128/25
```

Valid alternative, assuming the VPC includes the range and it is not already allocated:

```text
10.0.2.0/24
```

## Problem 5: A New Route Table Does Not Change Connectivity

Check:

1. Did you add the route to the intended route table?
2. Is the subnet associated with that route table?
3. Is the route target correct and available?
4. Does the route match the destination traffic?
5. Are security groups and network ACLs permitting the traffic?

Creating a route table alone does not make a subnet use it.

---

# 12. Interview Questions and Answers

## Q1. What is a VPC?

A VPC is a logically isolated virtual network in AWS where I can launch resources such as EC2 instances. I define its IPv4 address space using a CIDR block and create subnets, route tables, and network controls to manage connectivity.

## Q2. What is CIDR?

CIDR stands for Classless Inter-Domain Routing. It represents an IP address range using a network prefix.

For example:

```text
10.0.0.0/16
```

The `/16` means that 16 bits form the network prefix.

## Q3. How many IP addresses are in a `/24`?

A `/24` contains:

```text
2^(32 - 24) = 256 total IPv4 addresses
```

In an AWS IPv4 subnet, five addresses are reserved, leaving 251 usable IPv4 addresses.

## Q4. Can a VPC have exactly 500 IPv4 addresses?

Not with a single IPv4 CIDR block, because CIDR block sizes are powers of two.

A `/23` contains 512 total IPv4 addresses, while a `/24` contains 256.

## Q5. What is a subnet?

A subnet is a smaller IP address range within a VPC. Each subnet belongs to one Availability Zone and can be associated with a route table.

## Q6. Can two subnets in the same VPC have overlapping CIDRs?

No. AWS does not allow overlapping IPv4 CIDR blocks between subnets in the same VPC.

## Q7. What is the difference between a public and a private subnet?

A public subnet typically has a route to an Internet Gateway for internet-bound traffic. A private subnet does not have a direct route to the Internet Gateway for general IPv4 internet access.

## Q8. Does an Internet Gateway automatically make an EC2 instance publicly accessible?

No. The IGW must be attached to the VPC, the subnet must use a suitable route table, and the instance must have appropriate public addressing and security configuration.

## Q9. What is an Internet Gateway?

An Internet Gateway is an AWS VPC component that enables communication between a VPC and the internet when the required routing, addressing, and security configuration is present.

## Q10. What is a route table?

A route table contains routes that determine where network traffic is directed. Each route includes a destination and a target.

## Q11. What does `0.0.0.0/0` mean?

It represents all IPv4 destinations and is commonly used as a default route.

For example:

```text
0.0.0.0/0 → Internet Gateway
```

This sends traffic matching the default route toward the Internet Gateway.

## Q12. What is the `local` route?

The local route provides routing within the VPC's associated IPv4 CIDR address space. AWS automatically creates local routes for VPC CIDR blocks.

## Q13. What is the difference between a route table and a security group?

A route table determines where traffic goes. A security group controls which inbound and outbound traffic is permitted for supported resources.

## Q14. Why can't a private EC2 instance directly access the internet through an IGW-only route?

A private instance without a public IPv4 address cannot use that route as a direct IPv4 internet connection. A NAT Gateway is commonly used when private resources need outbound IPv4 internet access.

## Q15. Can a public subnet contain an EC2 instance with only a private IP?

Yes. A subnet's classification depends on its routing, not simply on whether an instance has a public IP.

However, an instance with only a private IPv4 address cannot ordinarily receive direct IPv4 internet connections through an IGW.

## Q16. What happens when a subnet runs out of IP addresses?

AWS cannot allocate additional IPv4 addresses from that subnet when no usable addresses remain. I can use another suitable subnet or plan additional VPC address space and new subnets.

## Q17. What happens if I create a route table but do not associate it with a subnet?

The subnet continues to use its existing associated route table. A newly created route table does not automatically replace the subnet's association.

## Q18. What is the difference between an IGW and a NAT Gateway?

An Internet Gateway enables internet connectivity for a VPC. A NAT Gateway commonly enables private-subnet resources to initiate outbound IPv4 internet connections without giving each resource a public IPv4 address.

## Q19. How does AWS prevent overlapping subnets?

AWS validates subnet CIDRs against the VPC's address space and existing subnets. It rejects a subnet if its range is outside the associated VPC CIDR or overlaps an existing subnet.

## Q20. How would you troubleshoot an EC2 instance that cannot access the internet?

I would check the subnet's route-table association, the default route and its target, the Internet Gateway or NAT Gateway configuration, IP addressing, security groups, network ACLs, DNS resolution, and the destination service.

---

# 13. Key Takeaways

## VPC

- A VPC provides an isolated virtual network in AWS.
- Its CIDR block defines its IP address space.
- A VPC can have additional associated CIDR blocks when AWS requirements are satisfied.
- Plan CIDR ranges carefully to avoid overlap with connected networks.

## Subnetting

- A subnet divides a VPC's address space into smaller ranges.
- Each subnet belongs to one Availability Zone.
- Subnet CIDRs in the same VPC must not overlap.
- AWS reserves five IPv4 addresses in each IPv4 subnet.
- Smaller numerical CIDR prefixes represent larger address ranges.

## Internet Gateway

- An IGW attaches to a VPC.
- A public subnet typically has a default route to an IGW.
- Direct IPv4 internet connectivity for an EC2 instance generally requires public IPv4 addressing.
- An IGW route alone does not guarantee internet access.

## Route Tables

- Route tables determine where matching traffic is directed.
- Every subnet uses an associated route table.
- The `local` route supports routing within the VPC's address space.
- `0.0.0.0/0` is the IPv4 default route.
- The most specific matching route takes precedence.
- Routing and security rules must both permit the required connectivity.

---

## Practical Learning Checklist

After studying these notes, practise the following tasks in AWS:

- [ ] Create a VPC with CIDR `10.0.0.0/16`.
- [ ] Create a public subnet with CIDR `10.0.1.0/24`.
- [ ] Create a private subnet with CIDR `10.0.2.0/24`.
- [ ] Create and attach an Internet Gateway.
- [ ] Create a public route table.
- [ ] Add `0.0.0.0/0 → Internet Gateway`.
- [ ] Associate the public subnet with the public route table.
- [ ] Verify the private subnet's route-table association.
- [ ] Launch an EC2 instance in the public subnet.
- [ ] Verify public addressing and inbound security rules.
- [ ] Test web connectivity.
- [ ] Explain why the private subnet does not automatically have direct internet access.
- [ ] Practise CIDR calculations for `/20`, `/22`, `/24`, `/25`, and `/26`.
- [ ] Explain why AWS rejects overlapping subnet CIDRs.



```text
Add AWS networking fundamentals notes covering VPC, subnetting, IGW, and route tables
```

**Next topic:** AWS networking — Security Groups, Network ACLs, NAT Gateway, and VPC Endpoints.