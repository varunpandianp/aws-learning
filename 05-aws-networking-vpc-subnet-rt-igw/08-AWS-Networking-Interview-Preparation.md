# AWS Cloud & Networking Interview Preparation
## Beginner to Intermediate – Practical & Scenario-Based Questions

**Preparation level:** Junior Cloud Engineer / Cloud Support / Junior DevOps Engineer

**Topics:** AWS Fundamentals, Networking, VPC, EC2, NAT, VPC Endpoints, Peering, Transit Gateway, ENI, Flow Logs and Storage.

---

# PART 1 – Networking Fundamentals (Beginner)

### Q1. What is an IP address?

An IP address is a logical address assigned to a device or network interface to identify it and enable communication over a network.

There are two major versions:

- IPv4 – 32-bit address.
- IPv6 – 128-bit address.

Example IPv4:

```text
192.168.1.10
```

### Q2. Why does IPv4 have four octets?

IPv4 contains 32 bits, divided into four groups of 8 bits.

Each 8-bit group is called an octet.

Example:

```text
192.168.1.10

192       168       1        10
8 bits    8 bits    8 bits   8 bits

Total = 32 bits
```

An 8-bit octet can represent values from 0 to 255.

Because:

\[
2^8 = 256
\]

Therefore, the possible values are 0–255.

### Q3. What is the difference between public and private IP addresses?

**Private IP:**
- Used for communication inside private networks.
- Not directly routable over the public internet.
- Commonly used inside AWS VPCs.

**Public IP:**
- Used for internet communication.
- Must be globally unique while allocated.
- Can be used to communicate over the internet when routing and security allow it.

Private IPv4 ranges:

| Classful range | Private CIDR |
|---|---|
| Class A | `10.0.0.0/8` |
| Class B | `172.16.0.0/12` |
| Class C | `192.168.0.0/16` |

### Q4. What is a loopback IP address?

`127.0.0.1` is the IPv4 loopback address.

It refers to the local machine.

Example:

```bash
ping 127.0.0.1
```

This tests the local TCP/IP stack, not connectivity to another computer.

### Q5. What is CIDR notation?

CIDR stands for Classless Inter-Domain Routing.

It represents an IP network using an IP address and a prefix length.

Example:

```text
192.168.1.0/24
```

Here:
- 24 bits are network bits.
- 8 bits are host bits.

### Q6. What does `/16`, `/20`, `/24` mean?

| CIDR | Network bits | Host bits | Total addresses |
|---|---:|---:|---:|
| `/16` | 16 | 16 | 65,536 |
| `/20` | 20 | 12 | 4,096 |
| `/24` | 24 | 8 | 256 |
| `/26` | 26 | 6 | 64 |
| `/27` | 27 | 5 | 32 |

Formula:

\[
\text{Total IP addresses} = 2^{\text{Host bits}}
\]

### Q7. What is the subnet mask for `/20`?

```text
CIDR: /20

Subnet mask: 255.255.240.0
```

The first 20 bits are network bits and the remaining 12 bits are host bits.

### Q8. What is the difference between network IP, usable IP and broadcast IP?

Example:

```text
192.168.1.0/24
```

| Type | Address |
|---|---|
| Network address | `192.168.1.0` |
| First usable | `192.168.1.1` |
| Last usable | `192.168.1.254` |
| Broadcast | `192.168.1.255` |

### Q9. What is a default gateway?

A default gateway is the router that forwards traffic to destinations outside the local subnet when no more specific route exists.

In AWS, the VPC subnet's route table determines the next hop. The instance generally uses the VPC router as its subnet gateway.

### Q10. What is the difference between a switch and a router?

**Switch:**
- Connects devices within a local network.
- Uses MAC addresses to forward frames.

**Router:**
- Connects different IP networks.
- Uses routing tables to forward packets.

---

# PART 2 – Subnetting and Routing (Beginner to Intermediate)

### Q11. How many IP addresses are available in a `/24` subnet?

\[
2^{32-24} = 256
\]

There are 256 total addresses.

In a conventional IPv4 subnet, two are reserved for network and broadcast addresses, leaving 254 usable addresses.

In AWS, five IPv4 addresses are reserved in each subnet.

Therefore:

```text
AWS /24 subnet:

256 total addresses
5 AWS-reserved addresses
251 usable addresses
```

### Q12. What is the difference between `/20` and `/24`?

- `/20` provides 4,096 total addresses.
- `/24` provides 256 total addresses.

A `/20` network is larger than a `/24` network.

### Q13. How do you calculate subnet ranges?

Example:

```text
10.0.0.0/24
```

Range:

```text
Network:   10.0.0.0
First IP:  10.0.0.1
Last IP:   10.0.0.254
Broadcast: 10.0.0.255
```

For AWS, remember its five reserved addresses.

### Q14. What is an overlapping CIDR?

Two networks have overlapping CIDRs when their IP address ranges intersect.

Example:

```text
VPC-A: 10.0.0.0/16
VPC-B: 10.0.0.0/24
```

The second network is inside the first network.

Overlapping VPC CIDRs prevent normal VPC Peering between those VPCs.

### Q15. What is the difference between a route table and a subnet?

**Subnet:**
A smaller IP network created inside a VPC.

**Route table:**
A collection of routing rules that determine where traffic leaving a subnet should go.

Example:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |
| `0.0.0.0/0` | Internet Gateway |

### Q16. What is the local route in an AWS route table?

Every VPC route table contains a local route for the VPC CIDR.

Example:

```text
VPC CIDR: 10.0.0.0/16

Destination: 10.0.0.0/16
Target: local
```

It allows communication between subnets within the VPC, subject to security and network controls.

### Q17. What is longest-prefix matching?

When multiple routes match a destination IP, the most specific route (the longest prefix) is selected.

Example:

```text
10.0.0.0/16       -> local
10.0.1.0/24       -> another target
```

Traffic to `10.0.1.10` matches both routes, but `/24` is more specific than `/16`.

---

# PART 3 – AWS VPC and Subnets (Beginner)

### Q18. What is an AWS VPC?

Amazon Virtual Private Cloud is a logically isolated virtual network in AWS.

We can define:
- IPv4 and IPv6 CIDR ranges.
- Public and private subnets.
- Route tables.
- Internet Gateways.
- NAT Gateways.
- Security groups.
- Network ACLs.
- VPC Endpoints.

### Q19. What is an Availability Zone?

An Availability Zone is an isolated location within an AWS Region containing one or more data centres.

We use multiple Availability Zones to improve application availability and resilience.

### Q20. Can two subnets in the same VPC communicate across different Availability Zones?

Yes.

Example:

```text
VPC: 10.0.0.0/16

Subnet-A: 10.0.1.0/24 – AZ-A
Subnet-B: 10.0.2.0/24 – AZ-B
```

The VPC local route enables communication between these subnets.

No VPC Peering is required.

### Q21. Can two different VPCs communicate without VPC Peering?

They cannot communicate directly through private IP addresses by default.

However, communication may be established using:
- VPC Peering.
- AWS Transit Gateway.
- VPN.
- AWS Direct Connect.
- Other suitable private connectivity architectures.

They may also communicate through public IP addresses over the internet if internet routing and security rules permit.

### Q22. What is an Internet Gateway?

An Internet Gateway is a VPC component that enables communication between resources in a VPC and the internet.

For normal IPv4 internet access, an EC2 instance generally needs:
- A public IPv4 address.
- A route to the Internet Gateway.
- Appropriate security group and network ACL rules.

### Q23. Does attaching an Internet Gateway automatically make all subnets public?

No.

A subnet is considered public when its associated route table has a route to an Internet Gateway.

For direct IPv4 internet access, the instance also needs an appropriate public IPv4 address.

### Q24. What is the difference between a public subnet and a private subnet?

| Public subnet | Private subnet |
|---|---|
| Has a route to IGW | No direct IGW default route |
| Can host internet-facing servers | Commonly hosts databases and internal applications |
| Instances may have public IPs | Instances generally have no public IPs |
| Direct internet connectivity is possible | NAT can provide outbound internet connectivity |

### Q25. What is the difference between a Security Group and a Network ACL?

| Feature | Security Group | Network ACL |
|---|---|---|
| Level | Network interface | Subnet |
| Type | Stateful | Stateless |
| Rules | Allow rules | Allow and deny rules |
| Return traffic | Automatically allowed for tracked connections | Must be explicitly permitted |
| Evaluation | All applicable rules | Rules evaluated by number |

### Q26. What does stateful mean in Security Groups?

If a Security Group permits an incoming connection, the response traffic for that connection is automatically allowed.

Example:

1. Client sends an HTTP request to EC2 on port 80.
2. Security Group allows inbound port 80.
3. EC2 sends the response.
4. Return traffic is automatically allowed by the stateful Security Group.

You do not need to create a separate outbound rule specifically for that response, although other outbound connections still need suitable permissions.

---

# PART 4 – EC2 and Connectivity (Beginner to Intermediate)

### Q27. What is EC2?

Amazon Elastic Compute Cloud provides virtual servers in AWS.

We can use EC2 to run:
- Web servers.
- Application servers.
- Databases.
- Monitoring tools.
- DevOps applications.

### Q28. What is the difference between public IP and private IP in EC2?

**Private IP:** Used for internal VPC communication.

**Public IP:** Used for internet communication when the required routing and security configuration exists.

An EC2 instance can have both private and public IP addresses.

### Q29. Can an EC2 instance in a private subnet receive a public IP automatically?

A private subnet normally has auto-assign public IPv4 disabled.

Even if an instance has a public IP, it still requires a suitable Internet Gateway route for direct internet access.

### Q30. What happens when we stop and start an EC2 instance?

- Its private IPv4 address is generally retained.
- An automatically assigned public IPv4 address is generally released on stop and start, and a new one is assigned on start.
- An associated Elastic IP remains allocated and associated unless changed.

### Q31. What is an AMI?

AMI stands for Amazon Machine Image.

It is a template used to launch EC2 instances.

It can contain:
- Operating system.
- Installed software.
- Application configuration.
- Storage configuration.

An AMI can be created from an existing EC2 instance to capture its configured state for future launches.

### Q32. What is the difference between a port and an IP address?

- IP address identifies a network interface or destination.
- Port identifies an application service or communication endpoint.

Examples:

| Service | Default port |
|---|---:|
| SSH | 22 |
| HTTP | 80 |
| HTTPS | 443 |
| MySQL | 3306 |
| PostgreSQL | 5432 |
| DNS | 53 |

### Q33. What does `0.0.0.0/0` mean?

It represents all IPv4 destinations.

In a Security Group inbound rule, it allows traffic from any IPv4 source for the specified port and protocol.

In a route table, it is the default route for destinations not matched by more specific routes.

These are two different uses of the same CIDR notation.

---

# PART 5 – NAT Gateway and NAT Instance (Intermediate)

### Q34. Why do we need a NAT Gateway?

NAT Gateway allows EC2 instances in private subnets to initiate outbound connections to the internet without assigning public IP addresses to those instances.

Example:

A private database server needs to download software updates.

It can use NAT Gateway for outbound internet connectivity.

### Q35. Where should we create a NAT Gateway?

In a public subnet.

A public NAT Gateway requires an Elastic IP address and a route from its public subnet to an Internet Gateway.

### Q36. Explain the NAT Gateway architecture.

```text
                 INTERNET
                    |
              Internet Gateway
                    |
               Public Subnet
                    |
                NAT Gateway
                Elastic IP
                    |
              Private Subnet
                    |
               Private EC2
```

### Q37. What are the steps to configure NAT Gateway?

1. Create a VPC.
2. Create public and private subnets.
3. Attach an Internet Gateway to the VPC.
4. Create a public route table.
5. Add `0.0.0.0/0 → IGW`.
6. Associate the public subnet with the public route table.
7. Create NAT Gateway in the public subnet.
8. Allocate an Elastic IP.
9. Create a private route table.
10. Add `0.0.0.0/0 → NAT Gateway`.
11. Associate the private subnet with the private route table.
12. Test internet access from the private EC2 instance.

### Q38. Does NAT Gateway require an Elastic IP?

Yes, for a public NAT Gateway.

The Elastic IP is used as the public source address for internet-bound traffic.

### Q39. Does the private EC2 instance need a public IP to access the internet through NAT?

No.

The private EC2 instance uses its private IP. NAT Gateway performs address translation for outbound internet communication.

### Q40. Can the internet initiate a new connection to a private EC2 instance through NAT Gateway?

No.

NAT Gateway allows outbound connections initiated by private instances and their response traffic. It does not allow unsolicited inbound connections.

### Q41. What is the difference between NAT Gateway and NAT Instance?

| NAT Gateway | NAT Instance |
|---|---|
| AWS-managed service | EC2 instance |
| AWS manages infrastructure | Customer manages OS and configuration |
| Automatically scales within service limits | Capacity depends on EC2 size |
| High availability design uses NAT per AZ | Customer manages HA |
| No source/destination check configuration | Must disable source/destination check |

### Q42. What is source/destination checking in NAT Instance?

By default, EC2 expects traffic to originate from or be destined for its own IP addresses.

A NAT Instance forwards traffic for other instances.

Therefore, source/destination checking must be disabled so it can forward traffic that is not directly addressed to itself.

---

# PART 6 – VPC Endpoints and S3 (Beginner to Intermediate)

### Q43. What is a VPC Endpoint?

A VPC Endpoint provides private connectivity between a VPC and supported AWS services.

It avoids requiring an Internet Gateway or NAT Gateway for that service connection.

### Q44. Why do we need a VPC Endpoint?

Consider a private EC2 instance that needs to upload backup files to S3.

Without an S3 Gateway Endpoint, traffic may need to go through a NAT Gateway or another valid network path.

With an S3 Gateway Endpoint, the EC2 instance can access S3 privately through AWS networking.

### Q45. What are the types of VPC Endpoints?

Two important types:

1. Gateway Endpoint.
2. Interface Endpoint.

### Q46. What is a Gateway Endpoint?

Gateway Endpoints support:
- Amazon S3.
- Amazon DynamoDB.

Important characteristics:
- Associated with route tables.
- Does not create an ENI.
- Automatically adds a service prefix-list route.
- Provides private connectivity to the supported service.

### Q47. What is an Interface Endpoint?

Interface Endpoint uses AWS PrivateLink.

It creates an ENI in a subnet with private IP addresses.

Examples:
- Systems Manager.
- CloudWatch.
- Secrets Manager.
- ECR.
- KMS.

It uses security groups to control access to the endpoint ENIs.

### Q48. Difference between Gateway and Interface Endpoint?

| Feature | Gateway | Interface |
|---|---|---|
| ENI | No | Yes |
| Route table association | Yes | Not required |
| Security group on endpoint | No | Yes |
| AWS PrivateLink | No | Yes |
| Examples | S3, DynamoDB | SSM, CloudWatch |

### Q49. How do you connect a private EC2 instance to S3 without NAT Gateway?

1. Create an S3 bucket.
2. Create a private EC2 instance.
3. Attach an IAM role with required S3 permissions.
4. Open VPC → Endpoints.
5. Create an endpoint.
6. Select the S3 Gateway service.
7. Select the VPC.
8. Associate the private subnet's route table.
9. Create the endpoint.
10. Verify that AWS automatically adds the S3 prefix-list route.
11. Test with AWS CLI.

### Q50. What happens to the route table after creating an S3 Gateway Endpoint?

AWS automatically adds a route similar to:

| Destination | Target |
|---|---|
| VPC CIDR | local |
| S3 prefix list | `vpce-xxxxxxxx` |

The endpoint route directs S3 traffic to the Gateway Endpoint.

### Q51. Why does `aws s3 ls` fail even after creating an endpoint?

Possible reasons:

- EC2 has no IAM credentials.
- IAM role lacks S3 permissions.
- Endpoint policy denies the request.
- Bucket policy denies the request.
- Route table is not associated with the endpoint.
- Incorrect bucket name or Region.
- AWS CLI is not configured to use the intended identity.

### Q52. What is the purpose of `aws configure`?

It configures AWS CLI credentials and default settings.

Command:

```bash
aws configure
```

It asks for:

```text
AWS Access Key ID
AWS Secret Access Key
Default region name
Default output format
```

Example:

```text
AWS Access Key ID: YOUR_ACCESS_KEY
AWS Secret Access Key: YOUR_SECRET_KEY
Default region name: ap-south-1
Default output format: json
```

For EC2, an IAM role is the recommended approach instead of storing long-term access keys.

### Q53. What are the important AWS S3 CLI commands?

List accessible buckets:

```bash
aws s3 ls
```

List objects in a bucket:

```bash
aws s3 ls s3://YOUR-BUCKET-NAME
```

Upload:

```bash
aws s3 cp file.txt s3://YOUR-BUCKET-NAME/
```

Download:

```bash
aws s3 cp s3://YOUR-BUCKET-NAME/file.txt .
```

Check current identity:

```bash
aws sts get-caller-identity
```

---

# PART 7 – VPC Peering (Intermediate)

### Q54. What is VPC Peering?

VPC Peering connects two VPCs privately, allowing resources in those VPCs to communicate using private IP addresses.

It supports:
- Same-Region VPC Peering.
- Inter-Region VPC Peering.
- Cross-account VPC Peering.

### Q55. Can two VPCs in different Availability Zones communicate through VPC Peering?

Availability Zones are locations within a Region, while VPC Peering connects VPCs.

Yes. VPC Peering supports VPCs containing subnets across different AZs.

### Q56. Can two different VPCs communicate using private IP addresses without peering?

Not directly by default.

They need a valid connectivity mechanism such as:
- VPC Peering.
- Transit Gateway.
- VPN.
- Another private networking solution.

### Q57. What are the steps to configure VPC Peering?

1. Create two VPCs.
2. Ensure CIDR ranges do not overlap.
3. Open VPC → Peering Connections.
4. Click Create Peering Connection.
5. Select requester VPC.
6. Select accepter VPC.
7. Create the connection.
8. Accept the request.
9. Add a route to the peering connection in VPC-A's route table.
10. Add a route to the peering connection in VPC-B's route table.
11. Configure Security Groups.
12. Test private IP connectivity.

### Q58. Why do we need routes in both VPCs?

Communication requires a forward path and a return path.

Example:

VPC-A:

```text
Destination: 10.0.0.0/16
Target: pcx-xxxxxxxx
```

VPC-B:

```text
Destination: 20.0.0.0/16
Target: pcx-xxxxxxxx
```

Both VPCs must know how to reach the other VPC's CIDR.

### Q59. Why is VPC Peering non-transitive?

VPC Peering does not forward traffic from one peering connection to another.

Example:

```text
VPC-A ---- VPC-B ---- VPC-C
```

A cannot automatically communicate with C through B.

A separate connection or another networking architecture is required.

### Q60. Can VPCs with overlapping CIDRs be peered?

No.

Example:

```text
VPC-A: 10.0.0.0/16
VPC-B: 10.0.1.0/24
```

These CIDRs overlap, so normal VPC Peering cannot be established.

### Q61. Can VPC Peering work across Regions?

Yes.

Inter-Region VPC Peering allows private connectivity between VPCs in different AWS Regions.

### Q62. What is the difference between VPC Peering and public internet communication?

VPC Peering provides private connectivity using private IP addresses.

Public internet communication uses public IP addresses and internet routing.

---

# PART 8 – Transit Gateway (Intermediate)

### Q63. What is AWS Transit Gateway?

AWS Transit Gateway is a centralized networking hub that connects multiple VPCs and on-premises networks.

### Q64. Why do we need Transit Gateway?

Imagine an organization has 20 VPCs.

Creating and managing individual peering connections between every VPC becomes complicated.

Transit Gateway allows VPCs to connect through a central networking hub.

### Q65. What is the difference between VPC Peering and Transit Gateway?

| Feature | VPC Peering | Transit Gateway |
|---|---|---|
| Architecture | Point-to-point | Hub-and-spoke |
| Transitive routing | No | Yes, when configured |
| Large network | More complex | Centralized management |
| VPC connections | Direct connection | Attachments |
| On-premises | Additional architecture required | VPN/Direct Connect attachments supported |

### Q66. Does Transit Gateway automatically connect every VPC?

No.

We need to:
- Create Transit Gateway.
- Create VPC attachments.
- Configure Transit Gateway route tables.
- Configure VPC route tables.
- Set up route association and propagation as required.
- Configure security controls.

---

# PART 9 – Elastic Network Interface (ENI)

### Q67. What is an ENI?

An Elastic Network Interface is a virtual network interface attached to an EC2 instance.

Think of it as a virtual network card.

### Q68. What information does an ENI contain?

- Private IPv4 addresses.
- Secondary private IPv4 addresses.
- MAC address.
- Security groups.
- Subnet association.
- Public IPv4 association, where configured.

### Q69. Why do we need an additional ENI?

Use cases include:
- Network separation.
- Multiple network interfaces.
- Firewall appliances.
- Multi-homed EC2 instances.
- Moving a secondary network identity between compatible instances.

### Q70. Can we attach an additional ENI to a running EC2 instance?

Yes, if the instance type supports additional network interfaces.

The ENI must belong to the same Availability Zone as the EC2 instance.

### Q71. Can we detach the primary ENI?

No. The primary network interface cannot be detached while it is attached as the primary interface.

### Q72. How do you verify an additional ENI in Linux?

List network interfaces:

```bash
ip link show
```

Check IP addresses:

```bash
ip addr show
```

Check routing:

```bash
ip route
```

---

# PART 10 – VPC Flow Logs and CloudWatch

### Q73. What are VPC Flow Logs?

VPC Flow Logs capture metadata about IP traffic going to and from network interfaces in a VPC.

They are useful for:
- Troubleshooting connectivity.
- Security analysis.
- Network traffic monitoring.
- Investigating accepted and rejected traffic.

### Q74. What information does VPC Flow Logs capture?

- Source IP.
- Destination IP.
- Source port.
- Destination port.
- Protocol.
- Number of packets.
- Number of bytes.
- ACCEPT or REJECT status.

### Q75. What are the Flow Log traffic types?

| Type | Meaning |
|---|---|
| ACCEPT | Capture accepted traffic |
| REJECT | Capture rejected traffic |
| ALL | Capture accepted and rejected traffic |

### Q76. Where can we store VPC Flow Logs?

Three supported destinations:

1. Amazon CloudWatch Logs.
2. Amazon S3.
3. Amazon Data Firehose.

### Q77. What is CloudWatch Log Group?

A Log Group is a logical container in CloudWatch Logs that stores log streams.

We can create a log group such as:

```text
/aws/vpc/flowlogs
```

### Q78. How do you configure VPC Flow Logs to CloudWatch?

1. Open CloudWatch.
2. Create a Log Group.
3. Open VPC Dashboard.
4. Select the VPC.
5. Open Flow Logs.
6. Click Create Flow Log.
7. Select ALL traffic.
8. Select CloudWatch Logs as destination.
9. Select the Log Group.
10. Configure the required IAM delivery role.
11. Create Flow Log.
12. Generate traffic.
13. View records in CloudWatch.

### Q79. What is the difference between ACCEPT and REJECT?

- ACCEPT: Traffic was permitted by the relevant network controls.
- REJECT: Traffic was rejected by a network control.

Flow Logs can help identify rejected traffic but may not tell you exactly which rule caused the rejection.

### Q80. Do Flow Logs capture actual packet contents?

No.

They capture network traffic metadata, not the actual application payload.

---

# PART 11 – AWS Storage Fundamentals

### Q81. What is the difference between EBS, EFS and S3?

| Feature | EBS | EFS | S3 |
|---|---|---|---|
| Storage type | Block storage | File storage | Object storage |
| Access | Attached to EC2 | Shared file system | API/HTTP |
| Typical use | OS and database disks | Shared application files | Backups, media, objects |
| Multi-instance sharing | Generally limited; special Multi-Attach support exists | Yes | Through APIs |
| Access protocol | Block device | NFS | S3 API |

### Q82. What is Amazon EBS?

Elastic Block Store provides persistent block storage volumes for EC2 instances.

Example use cases:
- Operating system disks.
- Database storage.
- Application data.
- Persistent file systems.

### Q83. What is Amazon EFS?

Elastic File System is a managed, scalable file system that multiple EC2 instances can mount and access concurrently.

It uses NFS.

Default NFS port:

```text
2049
```

### Q84. Can we mount EFS on EC2 instances in different Availability Zones?

Yes.

An EFS file system can be accessed by EC2 instances across multiple Availability Zones in the same Region through EFS mount targets and suitable network connectivity.

### Q85. What is Amazon S3?

Amazon Simple Storage Service is an object storage service.

Common uses:
- Backup.
- Static website assets.
- Media storage.
- Application logs.
- Data lakes.

### Q86. What is an EBS Snapshot?

An EBS Snapshot is a point-in-time backup of an EBS volume stored by AWS in its managed snapshot infrastructure.

We can create a new EBS volume from a snapshot.

### Q87. What is the difference between gp2 and gp3?

Both are General Purpose SSD EBS volume types.

- gp2 performance depends on volume size and uses burst credits.
- gp3 allows independent configuration of IOPS and throughput within its limits, without relying on burst credits.

---

# PART 12 – Scenario-Based Interview Questions

These questions are especially important for junior DevOps, Cloud Support and Cloud Engineer interviews.

## Scenario 1 – Private EC2 Cannot Access Internet

**Question:**

You created an EC2 instance in a private subnet. It has a private IP but no public IP. You try to install software using:

```bash
sudo dnf update -y
```

The command fails because it cannot reach the package repositories.

How do you solve this?

**Answer:**

I would configure a NAT Gateway.

Steps:

1. Create a NAT Gateway in a public subnet.
2. Allocate an Elastic IP.
3. Ensure the public subnet route table has `0.0.0.0/0 → IGW`.
4. Create or update the private route table.
5. Add `0.0.0.0/0 → NAT Gateway`.
6. Associate the private route table with the private subnet.
7. Check security groups, NACLs and DNS.
8. Test internet connectivity.

The private EC2 instance does not need a public IP.

## Scenario 2 – Public EC2 Cannot Access Internet

**Question:**

Your EC2 instance is in a public subnet and has a public IP. However, it cannot access the internet.

What will you check?

**Answer:**

I would check:

1. Internet Gateway is attached to the VPC.
2. Public route table contains `0.0.0.0/0 → IGW`.
3. Public subnet is associated with that route table.
4. EC2 has a public IP.
5. Security Group outbound rules permit the traffic.
6. Network ACL permits outbound and return traffic.
7. DNS configuration is correct.
8. Application or operating system firewall is not blocking the request.

## Scenario 3 – Two EC2 Instances in Different Subnets Cannot Ping

**Question:**

Two EC2 instances are in the same VPC but different Availability Zones. They cannot ping each other.

What will you check?

**Answer:**

They can communicate through the VPC local route without Peering.

I would check:

1. Both instances belong to the same VPC.
2. Their CIDR ranges are valid and non-overlapping.
3. Security Groups allow ICMP.
4. Network ACLs allow ICMP and return traffic.
5. OS firewall permits ping.
6. Instance network configuration is correct.

I would not create VPC Peering because both instances are already in the same VPC.

## Scenario 4 – Two VPCs Cannot Communicate Using Private IPs

**Question:**

You have two VPCs in the same AWS Region. Each VPC has one EC2 instance. They cannot communicate using private IP addresses.

How do you solve this?

**Answer:**

I would create a VPC Peering connection.

1. Verify CIDRs do not overlap.
2. Create the peering connection.
3. Accept the request.
4. Add a route to the peer CIDR in VPC-A.
5. Add a route to the peer CIDR in VPC-B.
6. Update Security Groups.
7. Verify NACLs.
8. Test using private IP addresses.

## Scenario 5 – Ping Works Using Public IP but Not Private IP

**Question:**

Two EC2 instances are in different VPCs. Ping works using public IP addresses, but fails using private IP addresses.

Why?

**Answer:**

The public IP traffic is travelling through the internet.

Private IP traffic requires private connectivity between the VPCs.

I would configure VPC Peering or Transit Gateway, add appropriate routes in both directions and permit ICMP in the Security Groups.

## Scenario 6 – Three VPCs Need Communication

**Question:**

An organization has three VPCs:

- VPC-A – Application.
- VPC-B – Database.
- VPC-C – Monitoring.

They need private communication with each other.

What would you recommend?

**Answer:**

For a small environment, individual VPC Peering connections can work.

For a larger environment, I would recommend Transit Gateway.

Transit Gateway provides centralized connectivity through a hub-and-spoke architecture.

I would configure attachments, route tables, route propagation or static routes, and security controls.

## Scenario 7 – Private EC2 Needs S3 Access Without NAT

**Question:**

A private EC2 instance needs to upload backups to S3. You don't want to pay for NAT Gateway processing for this S3 traffic.

What would you do?

**Answer:**

I would create an S3 Gateway Endpoint.

1. Create the S3 bucket.
2. Create the S3 Gateway Endpoint.
3. Select the correct VPC.
4. Associate the private route table.
5. Attach an IAM role to EC2.
6. Grant required S3 permissions.
7. Verify the S3 prefix-list route.
8. Test using `aws s3 ls`.

The EC2 instance can access S3 without NAT Gateway for this S3 traffic.

## Scenario 8 – S3 Endpoint Created but AWS CLI Fails

**Question:**

You successfully created an S3 Gateway Endpoint. But when you run:

```bash
aws s3 ls
```

It returns:

```text
Unable to locate credentials
```

What is the issue?

**Answer:**

The issue is likely authentication, not endpoint connectivity.

I would attach an IAM role to the EC2 instance with the required S3 permissions.

AWS CLI can then retrieve temporary credentials automatically.

## Scenario 9 – S3 Access Denied

**Question:**

The EC2 instance has an IAM role, the S3 Gateway Endpoint exists, and the route is correct. But S3 returns AccessDenied.

What will you check?

**Answer:**

I would check:

1. IAM role permissions.
2. Bucket policy.
3. Endpoint policy.
4. AWS account and bucket ownership.
5. Object-level permissions.
6. Whether the requested action is allowed.

Network connectivity and authorization are two separate things.

## Scenario 10 – NAT Gateway Created but Private EC2 Cannot Access Internet

**Question:**

You created a NAT Gateway and allocated an Elastic IP. But the private EC2 instance cannot access the internet.

What is your troubleshooting process?

**Answer:**

I would check:

1. NAT Gateway status is Available.
2. NAT Gateway is in a public subnet.
3. NAT Gateway has an Elastic IP.
4. Public route table has a default route to IGW.
5. Public subnet is associated with the public route table.
6. Private route table has a default route to NAT Gateway.
7. Private subnet is associated with the correct route table.
8. Security Groups and NACLs allow traffic.
9. DNS is functioning.

## Scenario 11 – Database Must Not Be Publicly Accessible

**Question:**

You are deploying a web application and database in AWS.

How would you design the network?

**Answer:**

I would create:

- Public subnets for the internet-facing load balancer or web tier.
- Private subnets for application and database tiers.
- An Internet Gateway for public internet connectivity.
- NAT Gateway if private application instances need outbound internet access.
- Security Groups that allow database traffic only from the application tier.
- No public IP on database instances.

For production database availability, I would consider Amazon RDS Multi-AZ rather than simply running two independent database EC2 instances.

## Scenario 12 – Additional Network Interface Required

**Question:**

An EC2 instance already has one network interface. Your application requires another private IP address and separate network configuration.

What would you do?

**Answer:**

I would create a secondary ENI in the same Availability Zone and VPC, select the required subnet and security group, and attach it to the EC2 instance.

Then I would verify the interface using:

```bash
ip addr show
```

I would also configure operating-system routing if the application requires a separate traffic path.

## Scenario 13 – Network Traffic Is Being Rejected

**Question:**

An application cannot connect to another EC2 instance. You want to investigate whether network traffic is accepted or rejected.

What AWS service would you use?

**Answer:**

I would enable VPC Flow Logs.

I would:

1. Create a CloudWatch Log Group.
2. Configure the required IAM delivery role.
3. Create Flow Logs at the VPC, subnet or ENI level.
4. Select ALL traffic.
5. Send records to CloudWatch Logs.
6. Investigate source IP, destination IP, ports, protocol and action.

This helps identify rejected traffic.

## Scenario 14 – VPC Peering Works in One Direction Only

**Question:**

EC2-A can communicate with EC2-B, but EC2-B cannot communicate with EC2-A.

What would you check?

**Answer:**

I would check:

1. Peering connection status is Active.
2. Route table in VPC-A has the route to VPC-B.
3. Route table in VPC-B has the route to VPC-A.
4. Security Group inbound and outbound rules.
5. Network ACL rules in both subnets.
6. Operating-system firewalls.

Both VPCs need valid forward and return routing.

## Scenario 15 – Flow Logs Show REJECT

**Question:**

You see REJECT records in VPC Flow Logs for traffic between two EC2 instances.

What could be the reason?

**Answer:**

Possible reasons include:

- Security Group blocking traffic.
- Network ACL blocking traffic.
- Incorrect destination or port.
- Network path or routing configuration problems.

I would compare the Flow Log record with the relevant Security Group and NACL rules.

---

# PART 13 – Linux Commands for AWS Networking Troubleshooting

These commands are important during Cloud Support and DevOps interviews.

| Command | Purpose |
|---|---|
| `ip addr` | Display IP addresses |
| `ip link` | Display network interfaces |
| `ip route` | Display routing table |
| `ping IP` | Test ICMP connectivity |
| `curl -I URL` | Test HTTP/HTTPS response headers |
| `nc -vz IP PORT` | Test TCP port connectivity |
| `ss -tulnp` | Display listening sockets |
| `nslookup DOMAIN` | DNS lookup |
| `dig DOMAIN` | Detailed DNS lookup |
| `traceroute IP` | Trace network path |
| `hostname -I` | Display instance IP addresses |

### Practical troubleshooting examples

Check IP address:

```bash
ip addr
```

Check default route:

```bash
ip route
```

Test connectivity:

```bash
ping -c 4 8.8.8.8
```

Test HTTPS:

```bash
curl -I https://www.google.com
```

Check whether port 3306 is reachable:

```bash
nc -vz 10.0.2.10 3306
```

Check listening services:

```bash
sudo ss -tulnp
```

Check DNS:

```bash
nslookup amazon.com
```

---

# PART 14 – Mock Interview Test

Try answering these questions without looking at the previous answers.

{@body const questions = [
{q:"What is the purpose of a VPC local route?",a:["Connect the VPC to the internet","Allow communication between subnets in the VPC","Connect two AWS Regions","Assign public IPs"],correct:1,why:"The local route provides routing between networks inside the same VPC."},
{q:"Where should a public NAT Gateway be created?",a:["Private subnet","Public subnet","S3 bucket","Transit Gateway"],correct:1,why:"A public NAT Gateway is placed in a public subnet with a route to an Internet Gateway."},
{q:"Which endpoint type is used for Amazon S3?",a:["Interface only","Gateway Endpoint","Transit Gateway","NAT Endpoint"],correct:1,why:"S3 supports Gateway Endpoints, which are associated with route tables."},
{q:"What is required for private IP communication between two separate VPCs?",a:["Only a Security Group","Only an Elastic IP","A private connectivity mechanism and routes","Only an Internet Gateway"],correct:2,why:"Peering, Transit Gateway or another private connectivity solution is needed, along with appropriate routes and security."},
{q:"Why is VPC Peering non-transitive?",a:["It does not support private IPs","It cannot forward traffic through another peering connection","It works only across Regions","It requires a NAT Gateway"],correct:1,why:"A peering connection connects two VPCs; it cannot act as a transit router to another peered VPC."},
{q:"Which service captures source and destination IP traffic metadata?",a:["CloudTrail","VPC Flow Logs","Route 53","EBS"],correct:1,why:"VPC Flow Logs record network traffic metadata and ACCEPT/REJECT status."},
{q:"What should you attach to EC2 to provide secure AWS CLI credentials?",a:["Elastic IP","IAM Role","Internet Gateway","NACL"],correct:1,why:"An EC2 instance profile associated with an IAM role provides temporary credentials."},
{q:"Can a secondary ENI be attached to a running EC2 instance?",a:["Never","Yes, if instance limits and AZ requirements are met","Only after deleting the primary ENI","Only through S3"],correct:1,why:"A compatible secondary ENI can be attached to a running instance in the same AZ."}
]}
{@body const [answers,setAnswers] = DIL.useState({})}
{@body const [show,setShow] = DIL.useState(false)}
<box border radius="lg" padding={3} gap={3}>
<row align="center" justify="between">
**AWS Networking Quiz**
<badge color="info">{Object.keys(answers).length}/{questions.length} answered</badge>
</row>
{#each questions as q,i}
<box key={i} gap={2}>
**{i+1}. {q.q}**
<radio-group direction="col" value={answers[i]===undefined?"":String(answers[i])} onChange={v=>{setAnswers(p=>({...p,[i]:Number(v)}));setShow(false);}}>
{#each q.a as a,j}
<radio value={String(j)}>{a}</radio>
{/each}
</radio-group>
{#if show}
<box background="surface-secondary" padding={2} radius="md" gap={1}>
<row align="center" gap={2}>
{#if answers[i]===q.correct}
<icon name="check-circle-filled" color="success"/>
**Correct**
{:else}
<icon name="x-circle" color="danger"/>
**Correct answer: {q.a[q.correct]}**
{/if}
</row>
<text size="sm">{q.why}</text>
</box>
{/if}
</box>
{#if i!==questions.length-1}
<divider color="subtle"/>
{/if}
{/each}
<button block disabled={Object.keys(answers).length!==questions.length} onClick={()=>setShow(true)}>Submit and check answers</button>
{#if show}
<title size="xl" color="success" tabularNums>{questions.filter((q,i)=>answers[i]===q.correct).length} / {questions.length}</title>
<text color="secondary" size="sm">Your quiz score</text>
<button block color="secondary" variant="outline" onClick={()=>{setAnswers({});setShow(false);}}>Retake quiz</button>
{/if}
</box>

---

# PART 15 – How to Answer in a Real Interview

For practical and scenario-based questions, use this four-step structure:

1. **Identify the problem:** Explain what is failing.
2. **Explain the AWS concept:** Identify the service or networking component involved.
3. **Explain the solution:** Describe the exact configuration steps.
4. **Verify:** Mention the commands or AWS console checks used to prove the solution.

### Example interview answer

**Interviewer:** A private EC2 instance cannot access the internet. How will you troubleshoot?

**Your answer:**

"First, I will check the subnet and confirm that the EC2 instance does not have a public IP address.

Then I will verify whether the private subnet's route table has a default route pointing to a NAT Gateway.

Next, I will verify that the NAT Gateway is available, is located in a public subnet, and has an Elastic IP.

I will also check that the public subnet's route table points to the Internet Gateway.

Finally, I will verify Security Group outbound rules, Network ACL rules and DNS resolution.

I will test connectivity using `curl -I https://www.google.com` and check the routing table using `ip route`."

---

# PART 16 – Recommended Preparation Order

| Day | Topic | Preparation |
|---|---|---|
| Day 1 | Networking fundamentals | IP, CIDR, subnetting, routing |
| Day 2 | AWS VPC | VPC, subnets, AZ, IGW, route tables |
| Day 3 | EC2 networking | Public/private IP, SG, NACL |
| Day 4 | NAT | NAT Gateway, NAT Instance, practical |
| Day 5 | VPC Endpoints | S3 Gateway and Interface Endpoint |
| Day 6 | VPC Peering | Same Region, cross-Region, routes |
| Day 7 | Transit Gateway and ENI | Hub-and-spoke, secondary NIC |
| Day 8 | Flow Logs and CloudWatch | Traffic monitoring and troubleshooting |
| Day 9 | AWS Storage | EBS, EFS, S3, snapshots |
| Day 10 | Mock interview | Scenario-based revision |

## Final Interview Preparation Checklist

- [ ] Explain IPv4 and CIDR without memorising.
- [ ] Calculate subnet ranges manually.
- [ ] Explain public and private subnet differences.
- [ ] Explain VPC local routing.
- [ ] Configure an Internet Gateway.
- [ ] Configure public and private route tables.
- [ ] Configure a NAT Gateway.
- [ ] Explain NAT Gateway vs NAT Instance.
- [ ] Create an S3 Gateway Endpoint.
- [ ] Explain IAM role vs AWS CLI credentials.
- [ ] Create VPC Peering and configure both route tables.
- [ ] Explain non-transitive peering.
- [ ] Explain Transit Gateway.
- [ ] Attach a secondary ENI.
- [ ] Create VPC Flow Logs and view them in CloudWatch.
- [ ] Troubleshoot connectivity using Linux commands.
- [ ] Explain EBS vs EFS vs S3.
- [ ] Answer at least 10 real-world scenarios.

**Final advice:** Don't just memorise these answers. Reproduce the practicals in your AWS account, explain why each route or security rule is required, and practise troubleshooting when you deliberately break a configuration. That is what will help you move from theoretical knowledge to a confident junior Cloud/DevOps engineer.