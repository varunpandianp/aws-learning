# AWS Networking – VPC Peering, Transit Gateway, Elastic Network Interface (ENI) & VPC Flow Logs

## 1. What is VPC Peering?

**VPC Peering** is a networking connection between two VPCs that allows resources in those VPCs to communicate with each other using private IP addresses.

VPC Peering can connect:

- Two VPCs in the same AWS Region.
- Two VPCs in different AWS Regions.
- VPCs belonging to different AWS accounts.

### Why do we need VPC Peering?

Consider this scenario:

- VPC-A contains an application server.
- VPC-B contains a database server.
- Both VPCs are in the same AWS Region.
- The application server needs to communicate with the database server using its private IP address.

Without a connectivity mechanism, the two VPCs are isolated.

**Solution: Create a VPC Peering connection.**

### Architecture

```text
           AWS REGION: US-WEST-2

       VPC-A                     VPC-B
    20.0.0.0/16               10.0.0.0/16
         |                          |
    Public/Private              Public/Private
       Subnet                      Subnet
         |                          |
      EC2-A                       EC2-B
   20.0.1.225                  10.0.1.15
         |                          |
         +------ VPC PEERING -------+
```

### How VPC Peering works

1. Create two VPCs with non-overlapping CIDR ranges.
2. Create a VPC Peering connection.
3. Select the requester VPC.
4. Select the accepter VPC.
5. Accept the peering request.
6. Wait until the connection status becomes Active.
7. Add the peering route to VPC-A's route table.
8. Add the peering route to VPC-B's route table.
9. Configure security groups to allow the required traffic.
10. Test connectivity using private IP addresses.

### Route Table Configuration

VPC-A route table:

| Destination | Target |
|---|---|
| `20.0.0.0/16` | local |
| `10.0.0.0/16` | VPC Peering Connection |

VPC-B route table:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |
| `20.0.0.0/16` | VPC Peering Connection |

### Test Connectivity

From EC2-A:

```bash
ping -c 4 10.0.1.15
```

From EC2-B:

```bash
ping -c 4 20.0.1.225
```

If routing, security groups and network ACLs permit ICMP, both instances can communicate through their private IP addresses.

---

## 2. Why is VPC Peering Non-Transitive?

**Transitive means passing through an intermediate network to reach another network.**

VPC Peering does not support transitive routing.

Consider three VPCs:

```text
       VPC-A
          |
          |
       VPC-B
          |
          |
       VPC-C
```

Assume:

- VPC-A is peered with VPC-B.
- VPC-B is peered with VPC-C.

Can VPC-A communicate with VPC-C through VPC-B?

**No.**

Why?

VPC Peering does not forward traffic from one peering connection to another.

Each peering connection is an independent connection between two VPCs.

### Solution

Create another peering connection directly between VPC-A and VPC-C.

```text
          VPC-A
          /   \
         /     \
        /       \
      VPC-B --- VPC-C
```

Now all three VPCs have direct peering connections.

**Important:** For a large number of VPCs, managing individual peering connections becomes complicated.

This is where AWS Transit Gateway becomes useful.

---

## 3. What is AWS Transit Gateway?

**AWS Transit Gateway** acts as a central networking hub that connects multiple VPCs and on-premises networks.

Instead of creating individual peering connections between every VPC, we connect the VPCs to one central Transit Gateway.

### Why do we need Transit Gateway?

Imagine an organization has:

- VPC-A for application servers.
- VPC-B for database servers.
- VPC-C for monitoring.
- VPC-D for development.
- An on-premises data centre.

Using VPC Peering, we need to manage multiple connections.

Using Transit Gateway, we can connect all these networks through one central networking hub.

### Architecture – Transit Gateway

```text
                  TRANSIT GATEWAY
                         |
          +--------------+--------------+
          |              |              |
        VPC-A          VPC-B          VPC-C
      Application     Database       Monitoring
          |
        VPC-D
       Development
```

### How Transit Gateway works

1. Create a Transit Gateway.
2. Create a Transit Gateway VPC attachment for VPC-A.
3. Create another attachment for VPC-B.
4. Create attachments for additional VPCs.
5. Configure Transit Gateway route tables.
6. Configure VPC subnet route tables to send traffic to the Transit Gateway.
7. Configure security groups and network ACLs.
8. Test private connectivity.

### VPC Route Table Example

In VPC-A:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |
| `172.16.0.0/16` | Transit Gateway |

In VPC-B:

| Destination | Target |
|---|---|
| `172.16.0.0/16` | local |
| `10.0.0.0/16` | Transit Gateway |

Transit Gateway also has its own route tables to control how traffic is forwarded between attachments.

### VPC Peering vs Transit Gateway

| Feature | VPC Peering | Transit Gateway |
|---|---|---|
| Architecture | Point-to-point | Hub-and-spoke |
| Connect two VPCs | Yes | Yes |
| Connect multiple VPCs | Multiple peering connections | Central hub |
| Transitive routing | No | Yes, when routes and policies permit |
| Inter-Region connectivity | Yes | Yes, using inter-Region peering |
| On-premises connectivity | Through additional network design | VPN and Direct Connect attachments |
| Management | More complex at scale | Centralized |
| Cost | Connection/data transfer charges may apply | Attachment and data-processing charges may apply |

**Interview point:** VPC Peering is suitable for connecting a small number of VPCs. Transit Gateway is useful for connecting many VPCs and on-premises networks through a centralized network architecture.

---

## 4. What is an Elastic Network Interface (ENI)?

**ENI – Elastic Network Interface**

An ENI is a virtual network interface attached to an EC2 instance.

Think of it as a virtual network card or NIC.

A physical computer has a network interface card to connect to a network.

Similarly, an EC2 instance uses an Elastic Network Interface to communicate with the VPC network.

### Why do we need an additional network interface?

Consider this scenario:

- We have one EC2 instance.
- The instance already has its primary network interface.
- We want to connect another network interface to the same EC2 instance.
- We want the instance to have another private IP address and network identity.

We can attach an additional ENI.

### ENI architecture

```text
             EC2 INSTANCE
                  |
          -------------------
          |                 |
       Primary ENI       Secondary ENI
          |                 |
       Private IP        Private IP
       10.0.1.10         10.0.1.20
          |                 |
          +-------- VPC ----+
```

### What does an ENI contain?

An ENI can have:

1. Primary private IPv4 address.
2. Secondary private IPv4 addresses.
3. MAC address.
4. Security groups.
5. Subnet association.
6. Public IPv4 address association, where configured.
7. Source/destination check setting.

### Primary ENI vs Secondary ENI

| Feature | Primary ENI | Secondary ENI |
|---|---|---|
| Created during instance launch | Yes | Can be created separately |
| Private IP | Yes | Yes |
| Can be detached | No, while attached as primary | Yes |
| Can attach to an existing instance | Already attached | Yes, subject to compatibility |
| Security groups | Yes | Yes |
| MAC address | Yes | Yes |

### Practical – Create an Additional ENI

#### Step 1: Create an EC2 instance

1. Open EC2 Dashboard.
2. Launch an EC2 instance.
3. Select your VPC.
4. Select a subnet.
5. Configure the security group.
6. Launch the instance.

#### Step 2: Create a Network Interface

1. Open EC2 Dashboard.
2. Navigate to Network Interfaces.
3. Click Create network interface.
4. Add a description:

   `Secondary-ENI`

5. Select the same VPC.
6. Select a subnet in the same Availability Zone as the EC2 instance.
7. Select a security group.
8. Create network interface.

#### Step 3: Attach the ENI to the running EC2 instance

1. Select the newly created network interface.
2. Click Actions.
3. Select Attach.
4. Select the running EC2 instance.
5. Select a device index for the secondary interface.
6. Attach the network interface.

The EC2 instance now has an additional network interface.

#### Step 4: Verify inside Linux

Connect to the EC2 instance.

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

You should see the additional network interface and its private IP address.

### Important ENI points

- An ENI belongs to a particular subnet and Availability Zone.
- It can be attached only to a compatible EC2 instance in the same Availability Zone.
- The instance type determines how many network interfaces can be attached.
- A secondary ENI can be detached and attached to another compatible instance in the same Availability Zone.
- The primary network interface cannot be detached from a running instance.
- Each ENI has its own security group associations.
- Attaching an ENI does not automatically configure application routing or make a new network path work.

**Use cases:**

- Network separation.
- Multi-homed EC2 instances.
- Firewall and network appliances.
- High availability designs.
- Moving a secondary network identity between instances.

---

## 5. What is VPC Flow Logs?

**VPC Flow Logs** capture information about IP traffic going to and from network interfaces in a VPC.

They help us understand network traffic and troubleshoot connectivity problems.

### Why do we need VPC Flow Logs?

Consider this scenario:

- EC2-A tries to connect to EC2-B.
- The connection is failing.
- We don't know whether traffic is reaching the destination.
- We need to understand whether traffic is accepted or rejected.

VPC Flow Logs help us investigate this.

### What information do Flow Logs capture?

- Source IP address.
- Destination IP address.
- Source port.
- Destination port.
- Protocol.
- Number of packets.
- Number of bytes.
- Traffic direction.
- ACCEPT or REJECT status.

**Important:** Flow Logs capture network traffic metadata, not the actual packet contents or application payload.

### Architecture – VPC Flow Logs to CloudWatch

```text
              AWS VPC
                 |
          ----------------
          |              |
        EC2-A          EC2-B
          |              |
          +------ VPC ---+
                 |
             FLOW LOGS
                 |
           CloudWatch Logs
                 |
              Log Group
                 |
              Log Streams
                 |
          Network Traffic Logs
```

### Flow Log capture options

When creating a Flow Log, we can choose:

| Traffic type | Meaning |
|---|---|
| ACCEPT | Capture accepted traffic |
| REJECT | Capture rejected traffic |
| ALL | Capture both accepted and rejected traffic |

For troubleshooting, selecting `ALL` is useful.

### Where can we send Flow Logs?

VPC Flow Logs support these destinations:

1. Amazon CloudWatch Logs.
2. Amazon S3.
3. Amazon Data Firehose.

In this practical, we use CloudWatch Logs.

---

## 6. Practical – Create a CloudWatch Log Group

Before creating the Flow Log, we will create a destination log group in CloudWatch.

### Step 1: Open CloudWatch

1. Open AWS Console.
2. Search for CloudWatch.
3. Open CloudWatch.
4. Select Logs.
5. Select Log groups.
6. Click Create log group.

### Step 2: Create the Log Group

Enter:

```text
Log Group Name: /aws/vpc/flowlogs
```

Choose the retention period as required.

Click Create.

Now the CloudWatch Log Group is ready to receive Flow Logs.

---

## 7. Practical – Create VPC Flow Logs

### Step 1: Open VPC Dashboard

1. Open VPC Dashboard.
2. Select Your VPCs.
3. Select the VPC for which you want to capture traffic.
4. Open the Flow Logs tab.
5. Click Create flow log.

### Step 2: Configure Flow Log

Example configuration:

| Setting | Value |
|---|---|
| Name | `VPC-Traffic-Flow-Log` |
| Filter | All |
| Maximum aggregation interval | 1 minute or 10 minutes |
| Destination | Send to CloudWatch Logs |
| Log group | `/aws/vpc/flowlogs` |
| IAM role | Select/create a role with required permissions |

Choose the appropriate traffic type and destination.

### Step 3: Configure IAM Role

Flow Logs need permission to publish log records to CloudWatch Logs.

Create or select an IAM role that trusts the VPC Flow Logs service and has the required CloudWatch Logs permissions.

Typical permissions include:

- `logs:CreateLogStream`
- `logs:PutLogEvents`
- `logs:DescribeLogGroups`
- `logs:DescribeLogStreams`

The AWS-managed policy `AmazonVPCFlowLogsDeliveryRolePolicy` can be used where appropriate.

### Step 4: Create Flow Log

Review the configuration and click Create flow log.

The Flow Log will start collecting traffic metadata.

---

## 8. Practical – View Flow Logs in CloudWatch

1. Open CloudWatch.
2. Select Logs.
3. Select Log groups.
4. Open `/aws/vpc/flowlogs`.
5. Open the log stream.
6. View the network traffic records.

Example Flow Log record:

```text
2 123456789012 eni-123456789abcdef01
10.0.1.10 10.0.2.20
49152 22 6 10 840 ACCEPT
```

Example fields:

| Field | Example | Meaning |
|---|---|---|
| Version | 2 | Flow Log format version |
| Account ID | 123456789012 | AWS account |
| Interface ID | eni-123456789abcdef01 | Network interface |
| Source IP | 10.0.1.10 | Sender IP |
| Destination IP | 10.0.2.20 | Receiver IP |
| Source port | 49152 | Sender port |
| Destination port | 22 | SSH port |
| Protocol | 6 | TCP |
| Packets | 10 | Number of packets |
| Bytes | 840 | Number of bytes |
| Action | ACCEPT | Traffic accepted |

Common protocol numbers:

| Protocol Number | Protocol |
|---|---|
| 1 | ICMP |
| 6 | TCP |
| 17 | UDP |

### ACCEPT vs REJECT

`ACCEPT` means the traffic was permitted by the relevant network controls.

`REJECT` means the traffic was rejected by a network control, such as a security group or network ACL.

Flow Logs help identify rejected traffic, but they do not always identify the exact rule responsible.

---

## 9. VPC Flow Logs – Important Notes

1. Flow Logs capture IP traffic metadata.
2. They do not capture actual application data or packet contents.
3. We can create Flow Logs at VPC, subnet or network interface level.
4. We can capture ACCEPT, REJECT or ALL traffic.
5. We can send Flow Logs to CloudWatch Logs, S3 or Data Firehose.
6. CloudWatch Log Groups store the collected records.
7. Flow Logs are not real-time packet captures; records are published periodically.
8. Flow Logs do not affect network traffic performance.
9. Flow Logs are useful for security auditing and troubleshooting.
10. Flow Logs do not replace packet capture tools such as tcpdump.

### Useful troubleshooting commands

Check network interfaces:

```bash
ip addr show
```

Check routing:

```bash
ip route
```

Test connectivity:

```bash
ping -c 4 10.0.2.20
```

Test a specific TCP port:

```bash
nc -vz 10.0.2.20 22
```

Check listening ports:

```bash
sudo ss -tulnp
```

---

## 10. VPC Peering vs Transit Gateway vs ENI vs Flow Logs

| AWS Service | Main Purpose |
|---|---|
| VPC Peering | Connect two VPCs privately |
| Transit Gateway | Connect multiple VPCs and networks through a central hub |
| ENI | Provide virtual network interfaces to EC2 instances |
| VPC Flow Logs | Capture IP traffic metadata for network monitoring |

---

## 11. Interview Questions

### VPC Peering

**Q1. What is VPC Peering?**

A private networking connection between two VPCs.

**Q2. Can we connect VPCs across different Regions?**

Yes. Inter-Region VPC Peering is supported.

**Q3. Why is VPC Peering non-transitive?**

Because traffic cannot pass through one peering connection to another peering connection.

**Q4. What is the alternative to multiple VPC Peering connections?**

AWS Transit Gateway.

### Transit Gateway

**Q5. What is AWS Transit Gateway?**

A centralized networking hub that connects multiple VPCs and on-premises networks.

**Q6. What is the difference between VPC Peering and Transit Gateway?**

VPC Peering provides point-to-point connectivity. Transit Gateway provides centralized hub-and-spoke connectivity and supports routing between attachments when configured.

### ENI

**Q7. What is an ENI?**

A virtual network interface attached to an EC2 instance.

**Q8. Can we attach an additional ENI to a running EC2 instance?**

Yes, provided the instance type and Availability Zone support it.

**Q9. Can we detach the primary ENI?**

No. The primary network interface cannot be detached while attached as the instance's primary interface.

**Q10. Why do we use multiple ENIs?**

For network separation, multi-homing, firewall appliances and network architecture requirements.

### VPC Flow Logs

**Q11. What is VPC Flow Logs?**

A feature that captures metadata about IP traffic going to and from VPC network interfaces.

**Q12. Where can we store Flow Logs?**

CloudWatch Logs, Amazon S3 or Amazon Data Firehose.

**Q13. What is the difference between ACCEPT and REJECT?**

ACCEPT means traffic was permitted; REJECT means traffic was blocked by a network control.

**Q14. Do Flow Logs capture packet contents?**

No. They capture traffic metadata, not packet payload.

**Q15. Can we use Flow Logs to troubleshoot security group issues?**

Yes. They can help identify accepted and rejected traffic, although they do not always identify the exact rule responsible.

---

## 12. Practical Learning Checklist

### VPC Peering

- [ ] Create two VPCs.
- [ ] Use non-overlapping CIDR blocks.
- [ ] Launch one EC2 instance in each VPC.
- [ ] Create a VPC Peering connection.
- [ ] Accept the peering request.
- [ ] Configure routes in both VPC route tables.
- [ ] Configure security groups.
- [ ] Test private IP connectivity.

### Transit Gateway

- [ ] Create a Transit Gateway.
- [ ] Create VPC attachments.
- [ ] Associate attachments with the appropriate Transit Gateway route table.
- [ ] Configure propagation or static routes.
- [ ] Add routes in VPC route tables.
- [ ] Test communication between VPCs.

### Additional ENI

- [ ] Launch an EC2 instance.
- [ ] Create a secondary ENI.
- [ ] Select the same Availability Zone.
- [ ] Attach the ENI to the running EC2 instance.
- [ ] Verify using `ip addr`.
- [ ] Verify using `ip route`.
- [ ] Test connectivity.

### VPC Flow Logs

- [ ] Create a CloudWatch Log Group.
- [ ] Create an IAM role for Flow Logs.
- [ ] Open VPC Dashboard.
- [ ] Select the VPC.
- [ ] Create a Flow Log.
- [ ] Select ALL traffic.
- [ ] Select CloudWatch Logs as the destination.
- [ ] Select the Log Group.
- [ ] Configure the IAM role.
- [ ] Generate network traffic.
- [ ] View traffic records in CloudWatch.

## 13. Final Key Takeaways

1. VPC Peering connects two VPCs privately.
2. VPC Peering is non-transitive.
3. Transit Gateway connects multiple VPCs through a centralized hub.
4. ENI is a virtual network interface for EC2.
5. We can attach an additional ENI to a running EC2 instance when supported.
6. VPC Flow Logs capture IP traffic metadata.
7. Flow Logs can be sent to CloudWatch Logs.
8. CloudWatch Log Groups organize and store log records.
9. Flow Logs help troubleshoot accepted and rejected network traffic.
10. VPC Flow Logs are not packet capture tools.

**Learning sequence:** VPC Peering → Transit Gateway → ENI → VPC Flow Logs → CloudWatch Log Analysis.