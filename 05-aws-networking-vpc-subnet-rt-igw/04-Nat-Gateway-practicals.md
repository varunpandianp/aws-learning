# AWS Practical – NAT Gateway Configuration to Provide Internet Access to a Private EC2 Instance

## 1. Practical Objective

**Goal:** Provide internet access to a private EC2 instance (Database Server) using a NAT Gateway without assigning a public IP address to the private EC2 instance.

### Scenario

We have:

- One VPC.
- One public subnet.
- One private subnet.
- One Internet Gateway (IGW).
- One NAT Gateway.
- One public EC2 instance (Web Server).
- One private EC2 instance (Database Server).

The public EC2 instance can access the internet directly through the Internet Gateway.

The private EC2 instance cannot directly access the internet because it does not have a public IP address.

We will configure a NAT Gateway to solve this problem.

## 2. Architecture

```text
                         INTERNET
                            |
                     INTERNET GATEWAY
                            |
                    PUBLIC ROUTE TABLE
                    0.0.0.0/0 -> IGW
                            |
                      PUBLIC SUBNET
                       /          \
                      /            \
                Web Server       NAT Gateway
                Public IP        Elastic IP
                                     |
                                     |
                              PRIVATE ROUTE TABLE
                              0.0.0.0/0 -> NAT
                                     |
                                PRIVATE SUBNET
                                     |
                                Database EC2
                                Private IP only
```

## 3. Step 1 – Create a VPC

1. Open AWS Console.
2. Search for VPC.
3. Select Your VPCs.
4. Click Create VPC.
5. Select VPC only.
6. Enter the name: `NAT-Practical-VPC`.
7. IPv4 CIDR: `10.0.0.0/16`.
8. Create VPC.

## 4. Step 2 – Create Two Subnets

We need one public subnet and one private subnet.

### Public Subnet

1. Open VPC → Subnets.
2. Click Create Subnet.
3. Select `NAT-Practical-VPC`.
4. Subnet name: `Public-Subnet`.
5. Select Availability Zone A.
6. IPv4 CIDR: `10.0.1.0/24`.
7. Create subnet.

Enable auto-assign public IPv4:

1. Select the public subnet.
2. Click Actions.
3. Select Edit subnet settings.
4. Enable Auto-assign public IPv4 address.
5. Save changes.

### Private Subnet

1. Click Create Subnet.
2. Select the same VPC.
3. Subnet name: `Private-Subnet`.
4. Select the same Availability Zone for this practical.
5. IPv4 CIDR: `10.0.2.0/24`.
6. Create subnet.

Keep auto-assign public IPv4 disabled for the private subnet.

## 5. Step 3 – Create an Internet Gateway

1. Open VPC → Internet Gateways.
2. Click Create Internet Gateway.
3. Name: `NAT-Practical-IGW`.
4. Create Internet Gateway.
5. Select the newly created Internet Gateway.
6. Click Actions.
7. Select Attach to a VPC.
8. Select `NAT-Practical-VPC`.
9. Attach Internet Gateway.

Now the VPC has an Internet Gateway attached.

**Important:** Attaching an Internet Gateway alone does not provide internet connectivity. We also need to configure the route table.

## 6. Step 4 – Configure the Public Route Table

A public subnet needs a route to the Internet Gateway.

1. Open VPC → Route Tables.
2. Click Create Route Table.
3. Name: `Public-Route-Table`.
4. Select `NAT-Practical-VPC`.
5. Create route table.

### Add the Internet Gateway route

1. Select `Public-Route-Table`.
2. Open Routes.
3. Click Edit routes.
4. Add route:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |
| `0.0.0.0/0` | Internet Gateway |

5. Select the Internet Gateway as the target.
6. Save changes.

### Associate the public subnet

1. Open Subnet associations.
2. Click Edit subnet associations.
3. Select `Public-Subnet`.
4. Save associations.

Now the public subnet is associated with the public route table.

## 7. Step 5 – Launch a Public EC2 Instance (Web Server)

1. Open EC2.
2. Click Launch Instance.
3. Name: `Public-Web-Server`.
4. Select an Amazon Linux AMI.
5. Select an instance type.
6. Select the VPC: `NAT-Practical-VPC`.
7. Select subnet: `Public-Subnet`.
8. Enable auto-assign public IP if it is not already enabled.
9. Configure a security group.
10. Launch instance.

The public EC2 instance should receive a public IPv4 address.

## 8. Step 6 – Create NAT Gateway

**This is the most important step in this practical.**

NAT Gateway must be created in a public subnet, not a private subnet.

1. Open VPC Dashboard.
2. Select NAT Gateways.
3. Click Create NAT Gateway.
4. Name: `My-NAT-Gateway`.
5. Select VPC: `NAT-Practical-VPC`.
6. Select subnet: `Public-Subnet`.
7. Connectivity type: Public.
8. Elastic IP allocation: Click Allocate Elastic IP.
9. AWS automatically allocates an Elastic IP for the NAT Gateway.
10. Click Create NAT Gateway.

Wait until the NAT Gateway status becomes **Available**.

### Important points

- NAT Gateway is created in the public subnet.
- NAT Gateway requires an Elastic IP for public internet access.
- NAT Gateway uses the Internet Gateway to reach the internet.
- Private EC2 instances use the NAT Gateway as their outbound route.
- We do not select a private EC2 instance during NAT Gateway creation.

## 9. Step 7 – Create a Private Route Table

Now we need a separate route table for the private subnet.

1. Open VPC → Route Tables.
2. Click Create Route Table.
3. Name: `Private-Route-Table`.
4. Select VPC: `NAT-Practical-VPC`.
5. Create route table.

### Add the NAT Gateway route

1. Select `Private-Route-Table`.
2. Open Routes.
3. Click Edit routes.
4. Add a new route:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |
| `0.0.0.0/0` | NAT Gateway |

5. For the target, select NAT Gateway.
6. Select `My-NAT-Gateway`.
7. Save changes.

**Important:** Do not select the Internet Gateway as the default route for the private subnet. Select the NAT Gateway.

## 10. Step 8 – Associate the Private Route Table with the Private Subnet

This step connects the private subnet to the NAT Gateway route.

1. Select `Private-Route-Table`.
2. Open Subnet associations.
3. Click Edit subnet associations.
4. Select `Private-Subnet`.
5. Save associations.

Now the private subnet uses the route table that sends internet-bound traffic to the NAT Gateway.

Remember:

- Public subnet → Public Route Table → Internet Gateway.
- Private subnet → Private Route Table → NAT Gateway.

We associate route tables with subnets, not directly with EC2 instances.

## 11. Step 9 – Launch a Private EC2 Instance (Database Server)

1. Open EC2.
2. Click Launch Instance.
3. Name: `Private-Database-Server`.
4. Select Amazon Linux AMI.
5. Select an instance type.
6. Select VPC: `NAT-Practical-VPC`.
7. Select subnet: `Private-Subnet`.
8. Disable Auto-assign public IP.
9. Configure a security group.
10. Launch instance.

The private EC2 instance should have:

- Private IPv4 address.
- No public IPv4 address.
- Private subnet association.
- Private route table with the NAT Gateway as the default route.

## 12. Step 10 – Test Internet Connectivity from the Private Instance

Connect to the private instance using AWS Systems Manager Session Manager or another configured management method.

For Session Manager, make sure the required SSM connectivity is available.

### Test 1 – Check private IP

```bash
ip addr
```

### Test 2 – Check default route

```bash
ip route
```

The default route should point to the subnet's VPC router. The subnet route table directs internet-bound traffic to the NAT Gateway.

### Test 3 – Test internet connectivity

```bash
ping -c 4 8.8.8.8
```

Note: Ping may fail because ICMP can be blocked. This alone does not prove NAT is broken.

### Test 4 – Test HTTPS connectivity

```bash
curl -I https://www.google.com
```

### Test 5 – Download a package

For Amazon Linux:

```bash
sudo dnf update -y
```

If the command successfully reaches the package repositories, the private EC2 instance has outbound connectivity.

## 13. How Does the NAT Gateway Work Internally?

Example:

Private EC2:

```text
Private IP: 10.0.2.10
Public IP:  None
```

NAT Gateway:

```text
Private IP: Assigned to NAT Gateway ENI
Elastic IP: Public IPv4 address
```

When the private EC2 instance requests an internet resource:

1. Private EC2 sends a request.
2. The private subnet route table matches `0.0.0.0/0`.
3. Traffic is forwarded to the NAT Gateway.
4. NAT Gateway translates the source address for internet communication.
5. NAT Gateway forwards traffic through the Internet Gateway.
6. The destination server responds.
7. The NAT Gateway maps the response back to the original private EC2 instance.

The private instance does not need its own public IP.

## 14. Route Table Comparison

### Public Route Table

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |
| `0.0.0.0/0` | Internet Gateway |

### Private Route Table

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |
| `0.0.0.0/0` | NAT Gateway |

## 15. Important Practical Points

1. NAT Gateway must be created in a public subnet.
2. Public subnet must have a route to the Internet Gateway.
3. NAT Gateway needs an Elastic IP for public internet connectivity.
4. Private subnet must use a route table whose default route targets the NAT Gateway.
5. Private EC2 instances do not need public IP addresses.
6. NAT Gateway does not need to be associated directly with the private EC2 instance.
7. The private route table is associated with the private subnet.
8. NAT Gateway supports outbound connections initiated by private instances.
9. NAT Gateway does not allow unsolicited inbound connections.
10. NAT Gateway is AZ-specific. For high availability, deploy one NAT Gateway per AZ and configure private subnet routes accordingly.

## 16. Troubleshooting Checklist

If the private EC2 instance cannot access the internet, check:

- [ ] NAT Gateway status is Available.
- [ ] NAT Gateway is in the public subnet.
- [ ] NAT Gateway has an Elastic IP.
- [ ] Internet Gateway is attached to the VPC.
- [ ] Public route table has `0.0.0.0/0 → IGW`.
- [ ] Public subnet is associated with the public route table.
- [ ] Private route table has `0.0.0.0/0 → NAT Gateway`.
- [ ] Private subnet is associated with the private route table.
- [ ] Security group outbound rules permit the required traffic.
- [ ] Network ACL rules permit outbound traffic and return traffic.
- [ ] DNS resolution is working if using domain names.
- [ ] NAT Gateway and private instance are in the same VPC.
- [ ] The NAT Gateway is in an available state and its AZ is healthy.

## 17. Interview Questions

**Q1. Why do we need NAT Gateway?**

To provide outbound internet connectivity to instances in private subnets without assigning public IP addresses to those instances.

**Q2. Where do we create NAT Gateway?**

Inside a public subnet.

**Q3. Does NAT Gateway require an Elastic IP?**

Yes, a public NAT Gateway requires an Elastic IP.

**Q4. Do we assign a public IP to the private EC2 instance?**

No. The private EC2 instance uses its private IP and reaches the internet through the NAT Gateway.

**Q5. Which route table should point to NAT Gateway?**

The route table associated with the private subnet.

**Q6. Which route table should point to Internet Gateway?**

The route table associated with the public subnet.

**Q7. Can we directly add an Internet Gateway to a private subnet's route table?**

Technically, a route can target an IGW, but a private instance without a public IPv4 address cannot use that route for normal direct IPv4 internet access. NAT Gateway is the appropriate solution.

**Q8. Do we select an EC2 instance when creating a NAT Gateway?**

No. We select the VPC, public subnet and Elastic IP. We then configure the private subnet's route table to target the NAT Gateway.

**Q9. Can a private EC2 instance receive inbound internet connections through NAT Gateway?**

No. NAT Gateway allows outbound connections initiated by private instances and their response traffic, not unsolicited inbound connections.

**Q10. Why do we create a separate route table for the private subnet?**

To send private subnet internet-bound traffic to the NAT Gateway instead of directly to the Internet Gateway.

## 18. Cleanup – Avoid Unnecessary AWS Charges

After completing the practical:

1. Terminate the EC2 instances.
2. Delete the NAT Gateway.
3. Release its Elastic IP after the NAT Gateway deletion completes, if no longer needed.
4. Delete custom route tables after removing their subnet associations.
5. Detach and delete the Internet Gateway.
6. Delete the subnets.
7. Delete the VPC.

**Cost reminder:** NAT Gateway has hourly and data-processing charges. Delete it after the practical if you no longer need it.