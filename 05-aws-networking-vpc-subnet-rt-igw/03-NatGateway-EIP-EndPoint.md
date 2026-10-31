# AWS Networking – NAT Gateway, NAT Instance, VPC Endpoints, Elastic IP & Route Tables

## 1. What is NAT?

**NAT – Network Address Translation**

NAT allows instances in a private subnet to access external networks, such as the internet, without assigning a public IP address to each private EC2 instance.

### Why do we need NAT?

Consider this scenario:

- We have a public subnet and a private subnet.
- A web server runs in the public subnet.
- A database server runs in the private subnet.
- The private EC2 instance needs to download software updates from the internet.
- We don't want to expose the private EC2 instance directly to the internet.

**Solution: NAT Gateway or NAT Instance.**

### How NAT works

1. A private EC2 instance sends a request to the internet.
2. The request goes to the NAT Gateway.
3. NAT Gateway translates the private source IP address to its public Elastic IP.
4. The request reaches the internet through the Internet Gateway.
5. The response returns to the NAT Gateway.
6. NAT Gateway forwards the response to the private EC2 instance.

```text
                  INTERNET
                     |
              Internet Gateway
                     |
                Public Subnet
                     |
                NAT Gateway
                     |
                Private Subnet
                     |
                  EC2
              (Private IP)
```

**Important:** NAT allows private instances to initiate outbound connections. It does not allow unsolicited inbound internet connections to private instances.

---

## 2. NAT Gateway

**NAT Gateway – AWS-managed networking service (PaaS-like).**

NAT Gateway allows instances in a private subnet to connect to the internet or other external networks without exposing those instances directly.

### Important points

- NAT Gateway is managed by AWS.
- It must be created in a public subnet for public internet access.
- It requires an Elastic IP for a public NAT Gateway.
- The public subnet must have a route to the Internet Gateway.
- The private subnet route table must point internet-bound traffic to the NAT Gateway.
- AWS manages the underlying infrastructure.
- It automatically scales to accommodate traffic within AWS service limits.
- It is highly available within its Availability Zone.
- For Availability Zone resilience, create a NAT Gateway in each AZ and route each private subnet to its same-AZ NAT Gateway.

### NAT Gateway architecture

```text
                 INTERNET
                    |
             Internet Gateway
                    |
              Public Subnet
                    |
               NAT Gateway
               (Elastic IP)
                    |
              Private Subnet
                    |
                Private EC2
```

### Practical: Create NAT Gateway

1. Open AWS VPC Dashboard.
2. Select NAT Gateways.
3. Click Create NAT Gateway.
4. Enter a name.
5. Select your VPC.
6. Select the public subnet.
7. Choose Public connectivity type.
8. Allocate or select an Elastic IP.
9. Create NAT Gateway.
10. Wait until its status becomes Available.

### Configure the private route table

Open VPC → Route Tables.

Select the route table associated with the private subnet.

Add this route:

| Destination | Target |
|---|---|
| VPC CIDR | local |
| `0.0.0.0/0` | NAT Gateway |

Now private EC2 instances can initiate internet connections through the NAT Gateway.

---

## 3. NAT Instance

**NAT Instance – EC2-based NAT solution (IaaS).**

A NAT Instance is an EC2 instance configured to perform Network Address Translation.

Unlike NAT Gateway, we manage the EC2 instance and its operating system.

### Important points

- NAT Instance is an EC2 instance.
- We configure it to perform NAT.
- It is launched in a public subnet.
- It requires a public IP or Elastic IP for internet access.
- Source/destination checking must be disabled.
- We manage operating system updates and security patches.
- We manage instance sizing and performance.
- We must configure high availability and failover ourselves.

### NAT Instance architecture

```text
                 INTERNET
                    |
             Internet Gateway
                    |
              Public Subnet
                    |
               NAT Instance
                  (EC2)
                    |
              Private Subnet
                    |
                Private EC2
```

### NAT Gateway vs NAT Instance

| Feature | NAT Gateway | NAT Instance |
|---|---|---|
| Service type | AWS-managed, PaaS-like | EC2, IaaS |
| Infrastructure | Managed by AWS | Managed by customer |
| Operating system | AWS manages underlying infrastructure | Customer manages |
| Scaling | AWS managed | Depends on EC2 size |
| High availability | Deploy per AZ for resilience | Customer configures |
| Maintenance | Low | Higher |
| Source/destination check | Not applicable | Must be disabled |
| Elastic IP | Required for public NAT Gateway | Can be associated |
| Production recommendation | Generally preferred | Special use cases |

**Interview point:** NAT Gateway is an AWS-managed service, whereas NAT Instance is an EC2 instance that we configure and manage ourselves.

---

## 4. What is a VPC Endpoint?

A VPC Endpoint allows resources inside a VPC to privately connect to supported AWS services without requiring an Internet Gateway or NAT Gateway for that service connection.

### Why do we need VPC Endpoints?

Consider this scenario:

1. We have a private EC2 instance.
2. The instance needs to upload backup files to Amazon S3.
3. Without a VPC Endpoint, we might route S3 traffic through a NAT Gateway.
4. This can introduce NAT Gateway processing costs.
5. Instead, we can create an S3 Gateway Endpoint.

Now the private EC2 instance can communicate with S3 using the VPC Endpoint.

### Benefits of VPC Endpoints

- Private connectivity to supported AWS services.
- No public IP required on the private EC2 instance.
- Avoid sending supported service traffic through a NAT Gateway.
- Reduce NAT Gateway data-processing costs.
- Improve network security.
- Keep supported service traffic on AWS networking.

---

## 5. Types of VPC Endpoints

There are two major endpoint types:

1. Gateway Endpoint
2. Interface Endpoint

### A. Gateway Endpoint

Gateway Endpoints support:

- Amazon S3
- Amazon DynamoDB

#### Important points

- Gateway Endpoint does not create an ENI in your subnet.
- It is associated with one or more VPC route tables.
- AWS automatically adds a route using the service prefix list.
- It provides private connectivity to S3 and DynamoDB.

#### Architecture

```text
             PRIVATE EC2
                  |
             Private Subnet
                  |
             Route Table
                  |
           S3 Gateway Endpoint
                  |
              Amazon S3
```

### B. Interface Endpoint

Interface Endpoints use AWS PrivateLink.

Examples of supported services:

- AWS Systems Manager (SSM)
- CloudWatch
- Secrets Manager
- ECR
- KMS

#### Important points

- Creates an Elastic Network Interface (ENI) in your subnet.
- ENI receives private IP addresses.
- Uses security groups to control access.
- Supports private DNS.
- Charges apply for endpoint usage and data processing.

#### Architecture

```text
             PRIVATE EC2
                  |
             Private Subnet
                  |
          Interface Endpoint
                  |
             Private ENI
                  |
             AWS PrivateLink
                  |
             AWS Service
```

### Gateway Endpoint vs Interface Endpoint

| Feature | Gateway Endpoint | Interface Endpoint |
|---|---|---|
| Connectivity | Gateway route | AWS PrivateLink |
| ENI created | No | Yes |
| Route table association | Yes | Not required |
| Security group | No endpoint SG | Yes |
| Supported examples | S3, DynamoDB | SSM, CloudWatch, ECR |
| Private IP | No endpoint ENI | Yes |

---

## 6. Practical: Connect a Private EC2 Instance to S3 Using a Gateway Endpoint

### Objective

Access an S3 bucket from a private EC2 instance without using a NAT Gateway or Internet Gateway for S3 traffic.

### Architecture

```text
             AWS VPC
                |
          Private Subnet
                |
          Private EC2
                |
           Route Table
                |
        S3 Gateway Endpoint
                |
             Amazon S3
```

### Step 1: Create an S3 bucket

1. Open AWS Console.
2. Search for S3.
3. Create a bucket.
4. Keep the bucket in the same AWS Region as your VPC for this practical.
5. Upload a test file.

### Step 2: Create a private EC2 instance

1. Open EC2 Dashboard.
2. Launch an EC2 instance.
3. Select your VPC.
4. Select your private subnet.
5. Disable auto-assign public IP.
6. Attach an IAM role with appropriate S3 permissions.
7. Launch the instance.

### Step 3: Create an S3 Gateway Endpoint

1. Open VPC Dashboard.
2. Select Endpoints.
3. Click Create Endpoint.
4. Enter the name: `s3-gateway-endpoint`.
5. Select AWS Services.
6. Search for `com.amazonaws.REGION.s3`.
7. Select the Gateway endpoint type.
8. Select your VPC.

### Step 4: Associate the route table

This is the most important step.

1. During endpoint creation, select the route table associated with your private subnet.
2. Create the endpoint.
3. AWS automatically adds the S3 prefix-list route to that route table.

Example:

| Destination | Target |
|---|---|
| VPC CIDR | local |
| S3 prefix list | S3 Gateway Endpoint |

You do not manually enter S3 IP addresses. AWS manages the S3 prefix list.

### Step 5: Configure IAM permissions

Attach an IAM role to your EC2 instance with the required S3 permissions.

Example policy for listing and downloading objects:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

### Step 6: Connect to private EC2

Use AWS Systems Manager Session Manager or another configured private management method.

Install AWS CLI if necessary.

List all S3 buckets:

```bash
aws s3 ls
```

List a particular bucket:

```bash
aws s3 ls s3://YOUR-BUCKET-NAME
```

Download a file:

```bash
aws s3 cp s3://YOUR-BUCKET-NAME/test.txt .
```

If IAM permissions, endpoint policy, bucket policy and networking are configured correctly, the private EC2 instance can access S3 without a NAT Gateway.

**Note:** For Session Manager access, configure the required SSM connectivity separately, such as the necessary Interface Endpoints or another valid network path.

---

## 7. What is an Elastic IP?

**Elastic IP (EIP)** is a static public IPv4 address allocated to your AWS account.

Normally, an automatically assigned public IPv4 address can change after an EC2 instance is stopped and started.

An Elastic IP provides a persistent public IPv4 address that can be associated with supported AWS resources.

### Why do we need an Elastic IP?

- To maintain a static public IPv4 address.
- To provide a consistent public address for applications.
- To associate a stable public IP with an EC2 instance.
- To provide a public IP for a NAT Gateway.

### Practical: Associate Elastic IP with EC2

1. Open EC2 Dashboard.
2. Select Elastic IPs.
3. Click Allocate Elastic IP address.
4. Allocate the address.
5. Select the allocated Elastic IP.
6. Click Actions.
7. Select Associate Elastic IP address.
8. Select your EC2 instance.
9. Associate it.

Now your EC2 instance can use the Elastic IP as its public IPv4 address.

**Important:** An Elastic IP alone does not provide internet connectivity. You also need:

- Internet Gateway attached to the VPC.
- Route table with `0.0.0.0/0` pointing to the Internet Gateway.
- Suitable security group rules.
- Appropriate network ACL rules.
- Elastic IP associated with the instance's network interface.

---

## 8. What is a Route Table?

A route table contains rules called routes that determine where network traffic from a subnet is directed.

### Important concepts

- VPC contains subnets.
- EC2 instances are launched inside subnets.
- Route tables control traffic leaving a subnet.
- Route table associations connect route tables to subnets.
- Gateway Endpoints are associated with route tables.

### Public Subnet Route Table

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |
| `0.0.0.0/0` | Internet Gateway |

This allows public IPv4 internet routing for instances with suitable public IPv4 addresses.

### Private Subnet Route Table – NAT Gateway

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |
| `0.0.0.0/0` | NAT Gateway |
| S3 prefix list | S3 Gateway Endpoint |

Here:

- Internal VPC traffic uses the local route.
- General internet traffic goes through NAT Gateway.
- S3 traffic goes through the S3 Gateway Endpoint.

### Private Subnet Route Table – S3 Endpoint Only

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |
| S3 prefix list | S3 Gateway Endpoint |

This allows private EC2 instances to access S3 but does not provide general internet access.

---

## 9. Route Table Association vs VPC Endpoint Association

These are two different concepts.

### Route Table Association

A route table association connects a route table to a subnet.

Example:

- Public subnet A → Public route table.
- Public subnet B → Public route table.
- Private subnet A → Private route table.
- Private subnet B → Private route table.

### VPC Endpoint Route Table Association

For a Gateway Endpoint, we select the route tables that should receive the endpoint route.

Example:

```text
Private Subnet A
       |
Private Route Table
       |
S3 Gateway Endpoint
       |
Amazon S3
```

The route table is associated with the private subnet, and the S3 Gateway Endpoint is associated with that route table.

**Remember:** We do not directly associate an EC2 instance with a route table. The instance inherits subnet routing through the subnet's route table association.

---

## 10. Complete AWS Networking Architecture

```text
                         INTERNET
                            |
                     Internet Gateway
                            |
                   -------------------
                   |                 |
              Public Subnet     Public Subnet
                   |                 |
              NAT Gateway       Web Server
                   |
             Private Subnet
                   |
              Private EC2
                   |
              Route Table
               /       \
              /         \
         NAT Gateway   S3 Gateway
                           |
                        Amazon S3
```

### Which AWS service should we use?

| Requirement | AWS Service |
|---|---|
| Private EC2 needs general internet access | NAT Gateway |
| Private EC2 needs S3 access | S3 Gateway Endpoint |
| Private EC2 needs DynamoDB access | DynamoDB Gateway Endpoint |
| Private EC2 needs private SSM access | SSM Interface Endpoint |
| EC2 needs a static public IPv4 address | Elastic IP |
| Public subnet needs internet routing | Internet Gateway |
| Connect a route table to a subnet | Route Table Association |
| Private EC2 needs private access to AWS services | VPC Endpoint |

---

## 11. Interview Questions

**Q1. Why do we need NAT Gateway?**

To allow private subnet instances to initiate outbound connections to the internet without assigning public IP addresses to those instances.

**Q2. What is the difference between NAT Gateway and NAT Instance?**

NAT Gateway is an AWS-managed service. NAT Instance is an EC2 instance that we configure and manage.

**Q3. Why do we need VPC Endpoints?**

To privately connect resources in a VPC to supported AWS services without routing that service traffic through the public internet.

**Q4. What is the difference between Gateway Endpoint and Interface Endpoint?**

Gateway Endpoint supports S3 and DynamoDB and uses route tables. Interface Endpoint creates ENIs with private IP addresses and uses AWS PrivateLink.

**Q5. Can a private EC2 instance access S3 without NAT Gateway?**

Yes. We can create an S3 Gateway Endpoint and associate it with the private subnet's route table.

**Q6. Does S3 Gateway Endpoint create an ENI?**

No. Gateway Endpoints do not create ENIs.

**Q7. What is the purpose of associating a Gateway Endpoint with a route table?**

It allows traffic destined for the supported AWS service to use the endpoint route.

**Q8. What is an Elastic IP?**

A static public IPv4 address allocated to an AWS account.

**Q9. Can a private subnet access the internet through an Internet Gateway directly without a public IP?**

No. For normal direct IPv4 internet access through an Internet Gateway, the instance needs a public IPv4 address and an appropriate route.

**Q10. Can we use both NAT Gateway and S3 Gateway Endpoint in the same private subnet?**

Yes. General internet traffic can use NAT Gateway, while S3 traffic uses the S3 Gateway Endpoint.

**Q11. Do we associate an EC2 instance directly with a route table?**

No. We associate the route table with the subnet. EC2 instances inherit the subnet's routing.

---

## 12. Practical Learning Tasks

- [ ] Create a VPC.
- [ ] Create one public subnet.
- [ ] Create one private subnet.
- [ ] Create an Internet Gateway.
- [ ] Attach the Internet Gateway to the VPC.
- [ ] Create a public route table.
- [ ] Add `0.0.0.0/0` pointing to the Internet Gateway.
- [ ] Associate the public route table with the public subnet.
- [ ] Create an EC2 instance in the public subnet.
- [ ] Create an Elastic IP and associate it with the EC2 instance.
- [ ] Create a NAT Gateway in the public subnet.
- [ ] Allocate and associate an Elastic IP with the NAT Gateway.
- [ ] Create a private route table.
- [ ] Add `0.0.0.0/0` pointing to the NAT Gateway.
- [ ] Associate the private route table with the private subnet.
- [ ] Launch a private EC2 instance without a public IP.
- [ ] Test internet access from the private EC2 instance through NAT.
- [ ] Create an S3 bucket.
- [ ] Create an S3 Gateway Endpoint.
- [ ] Associate the endpoint with the private route table.
- [ ] Attach an IAM role to the private EC2 instance.
- [ ] Test S3 access using AWS CLI.
- [ ] Compare NAT access and VPC Endpoint access.

## 13. Key Takeaways

1. NAT Gateway provides outbound internet connectivity to private subnet instances.
2. NAT Instance is an EC2-based alternative that requires customer management.
3. VPC Endpoints provide private connectivity to supported AWS services.
4. S3 and DynamoDB support Gateway Endpoints.
5. Interface Endpoints use ENIs and AWS PrivateLink.
6. Elastic IP provides a persistent public IPv4 address.
7. Route tables determine where subnet traffic goes.
8. Route table association connects a route table to a subnet.
9. Gateway Endpoint association adds service routes to selected route tables.
10. We do not directly associate EC2 instances with route tables.

**My practical learning order:** First test S3 access through a Gateway Endpoint without NAT. Then configure NAT Gateway and test general internet access from the same private EC2 instance. This demonstrates the difference between NAT and VPC Endpoints.