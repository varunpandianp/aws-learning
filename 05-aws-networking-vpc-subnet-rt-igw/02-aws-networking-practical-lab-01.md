# AWS Networking Practical Lab 01
## Create a VPC, 4 Subnets Across 2 AZs, Internet Gateway, Route Tables, and Security Groups

**Level:** Beginner
**Platform:** AWS Management Console
**Services:** VPC, Subnets, Availability Zones, Security Groups, Internet Gateway, Route Tables, EC2
**Architecture:** 2 Web Servers + 2 Database Servers

---

# 1. Objective

Create an AWS network with:

1. One VPC containing approximately 1,000 IPv4 addresses.
2. Four subnets across two Availability Zones.
3. Two public subnets for web servers.
4. Two private subnets for database servers.
5. One Internet Gateway attached to the VPC.
6. Public and private route tables.
7. A web-server security group.
8. A database-server security group.
9. Two web-server EC2 instances.
10. Two database-server EC2 instances.

**Expected result:** Internet users can access the web tier, while the database tier remains private.

# 2. Network Architecture

```text
                         INTERNET
                            |
                            |
                     Internet Gateway
                            |
                            |
                 Public Route Table
                 0.0.0.0/0 -> IGW
                            |
                +-----------+-----------+
                |                       |
                |                       |
          Availability Zone A     Availability Zone B
                |                       |
        +---------------+       +---------------+
        | Public        |       | Public        |
        | Subnet A      |       | Subnet B      |
        | Web Server 1  |       | Web Server 2  |
        +---------------+       +---------------+
                |                       |
                +-----------+-----------+
                            |
                    Application Traffic
                            |
                +-----------+-----------+
                |                       |
        +---------------+       +---------------+
        | Private       |       | Private       |
        | Subnet A      |       | Subnet B      |
        | Database 1    |       | Database 2    |
        +---------------+       +---------------+
                |                       |
                +-----------+-----------+
                            |
                      Private Routing
```

**Important:** The database subnets will not have a direct route to the Internet Gateway. Database access will be restricted using security groups.

For a real application, use a supported database deployment design, such as Amazon RDS Multi-AZ, where appropriate. Two EC2 instances running database software do not automatically provide database replication or high availability.

---

# 3. Plan the VPC CIDR

## Requirement

We want approximately 1,000 IPv4 addresses in the VPC.

A CIDR block cannot provide exactly 1,000 IPv4 addresses because IPv4 CIDR block sizes are powers of two.

We'll use:

```text
VPC CIDR: 10.0.0.0/22
```

### VPC CIDR calculation

```text
IPv4 address bits = 32
Network bits      = 22
Host bits         = 32 - 22 = 10

Total IP addresses = 2^10
                   = 1,024
```

VPC address range:

```text
Start: 10.0.0.0
End:   10.0.3.255
```

The VPC contains 1,024 total IPv4 addresses.

## Important distinction

The VPC's total address count is not the same as the number of addresses available for EC2 instances.

AWS reserves five IPv4 addresses in each IPv4 subnet. The usable address count therefore depends on how the VPC address space is divided into subnets.

---

# 4. Subnet Design

## Requirement

Create four subnets across two Availability Zones.

- Public Subnet A: Web Server 1
- Public Subnet B: Web Server 2
- Private Subnet A: Database Server 1
- Private Subnet B: Database Server 2

Each subnet should provide at least 200 usable IPv4 addresses.

A `/24` subnet contains 256 total IPv4 addresses.

In AWS:

```text
256 total - 5 reserved = 251 usable IPv4 addresses
```

Therefore, a `/24` subnet satisfies our requirement.

## Subnet CIDR plan

| Subnet | Availability Zone | CIDR | Type | Total IPs | AWS-usable IPs |
|---|---|---|---|---:|---:|
| Public Subnet A | AZ A | `10.0.0.0/24` | Public | 256 | 251 |
| Public Subnet B | AZ B | `10.0.1.0/24` | Public | 256 | 251 |
| Private Subnet A | AZ A | `10.0.2.0/24` | Private | 256 | 251 |
| Private Subnet B | AZ B | `10.0.3.0/24` | Private | 256 | 251 |

All four `/24` subnets fit exactly within `10.0.0.0/22`.

```text
VPC: 10.0.0.0/22
|
|-- Public Subnet A:  10.0.0.0/24
|
|-- Public Subnet B:  10.0.1.0/24
|
|-- Private Subnet A: 10.0.2.0/24
|
|-- Private Subnet B: 10.0.3.0/24
```

**Important:** The subnets consume the entire VPC IPv4 address space. There is no unallocated IPv4 space left in this VPC CIDR for additional subnets.

---

# 5. Step 1 — Create the VPC

### AWS Console

1. Sign in to the AWS Management Console.
2. Search for **VPC**.
3. Open the VPC dashboard.
4. Select **Your VPCs**.
5. Click **Create VPC**.
6. Select the VPC-only option.
7. Enter the following details.

| Setting | Value |
|---|---|
| Name tag | `devops-practical-vpc` |
| IPv4 CIDR | `10.0.0.0/22` |
| IPv6 CIDR | No IPv6 CIDR for this lab |
| Tenancy | Default |

8. Click **Create VPC**.

### Verification

Open **Your VPCs** and verify:

```text
Name: devops-practical-vpc
CIDR: 10.0.0.0/22
State: Available
```

### Learning point

The VPC defines the overall IPv4 address space. Subnets divide that space into smaller ranges.

---

# 6. Step 2 — Create Four Subnets

Navigate to:

**VPC → Subnets → Create subnet**

Select:

```text
VPC: devops-practical-vpc
```

Create the following subnets.

## Subnet 1 — Public Subnet A

```text
Name: public-subnet-a
Availability Zone: Choose AZ A
IPv4 CIDR: 10.0.0.0/24
```

## Subnet 2 — Public Subnet B

```text
Name: public-subnet-b
Availability Zone: Choose AZ B
IPv4 CIDR: 10.0.1.0/24
```

## Subnet 3 — Private Subnet A

```text
Name: private-subnet-a
Availability Zone: Same AZ A
IPv4 CIDR: 10.0.2.0/24
```

## Subnet 4 — Private Subnet B

```text
Name: private-subnet-b
Availability Zone: Same AZ B
IPv4 CIDR: 10.0.3.0/24
```

Choose two different Availability Zones available in your selected AWS Region. The AZ names shown in the console may differ between AWS accounts.

## Enable automatic public IPv4 assignment

For each public subnet:

1. Navigate to **VPC → Subnets**.
2. Select `public-subnet-a`.
3. Choose **Actions**.
4. Select **Edit subnet settings**.
5. Enable **Auto-assign public IPv4 address**.
6. Save the changes.
7. Repeat for `public-subnet-b`.

Leave automatic public IPv4 assignment disabled for the private subnets.

### Important

Enabling automatic public IPv4 assignment does not, by itself, make a subnet public.

The subnet also needs an appropriate route to an Internet Gateway for direct IPv4 internet connectivity.

---

# 7. Step 3 — Create the Internet Gateway

An Internet Gateway allows communication between the VPC and the internet when the required routing, addressing, and security configuration is in place.

### AWS Console

1. Navigate to **VPC → Internet gateways**.
2. Click **Create internet gateway**.
3. Enter:

```text
Name: devops-practical-igw
```

4. Click **Create internet gateway**.
5. Select the newly created Internet Gateway.
6. Choose **Actions → Attach to a VPC**.
7. Select:

```text
devops-practical-vpc
```

8. Attach the Internet Gateway.

### Verification

Confirm that the Internet Gateway shows:

```text
Name: devops-practical-igw
State: Attached
VPC: devops-practical-vpc
```

### Learning point

Creating an Internet Gateway and attaching it to a VPC are separate operations.

An attached IGW does not automatically give every subnet internet access. The route table and instance configuration must also be correct.

---

# 8. Step 4 — Configure the Public Route Table

When you create a VPC, AWS creates a main route table automatically. It normally contains a local route for the VPC's CIDR.

We will create a separate route table for the public subnets.

### Create the public route table

1. Navigate to **VPC → Route tables**.
2. Click **Create route table**.
3. Enter:

```text
Name: public-route-table
VPC: devops-practical-vpc
```

4. Click **Create route table**.

## Add the Internet Gateway route

1. Select `public-route-table`.
2. Open the **Routes** tab.
3. Click **Edit routes**.
4. Add the following route.

| Destination | Target |
|---|---|
| `0.0.0.0/0` | `devops-practical-igw` |

5. Save the changes.

The public route table should contain:

| Destination | Target | Purpose |
|---|---|---|
| `10.0.0.0/22` | `local` | Routes traffic within the VPC |
| `0.0.0.0/0` | Internet Gateway | Routes other IPv4 destinations toward the internet |

AWS normally creates the local route automatically. Do not remove it.

## Associate the public subnets

1. Select `public-route-table`.
2. Open **Subnet associations**.
3. Click **Edit subnet associations**.
4. Select:
   - `public-subnet-a`
   - `public-subnet-b`
5. Save the associations.

### Learning point

A route table must be associated with a subnet for that subnet to use its routes.

The route:

```text
0.0.0.0/0 → Internet Gateway
```

is a routing rule. It is not an inbound firewall rule.

---

# 9. Step 5 — Configure the Private Route Table

The database subnets should not have direct internet routing in this lab.

### Create the private route table

1. Navigate to **VPC → Route tables**.
2. Click **Create route table**.
3. Enter:

```text
Name: private-route-table
VPC: devops-practical-vpc
```

4. Create the route table.

The private route table should contain:

| Destination | Target |
|---|---|
| `10.0.0.0/22` | `local` |

Do not add this route:

```text
0.0.0.0/0 → Internet Gateway
```

to the database route table.

### Associate the private subnets

1. Select `private-route-table`.
2. Open **Subnet associations**.
3. Click **Edit subnet associations**.
4. Select:
   - `private-subnet-a`
   - `private-subnet-b`
5. Save the associations.

### Expected result

The public subnets use the public route table.

The private subnets use the private route table.

This separation allows the public and private tiers to have different routing configurations.

**Note:** The private subnets can still communicate with resources in the VPC through the local route, subject to security rules. They simply do not have a default route to the internet in this design.

---

# 10. Step 6 — Create the Web Server Security Group

A security group is a virtual firewall associated with supported network resources, including EC2 network interfaces.

It controls allowed inbound and outbound traffic.

### Create the security group

1. Navigate to **VPC → Security groups**.
2. Click **Create security group**.
3. Enter:

```text
Security group name: web-server-sg
Description: Security group for web servers
VPC: devops-practical-vpc
```

## Inbound rules

Configure the following rules for a basic web-server lab.

| Type | Protocol | Port | Source |
|---|---|---:|---|
| HTTP | TCP | 80 | `0.0.0.0/0` |
| HTTPS | TCP | 443 | `0.0.0.0/0` |
| SSH, if required | TCP | 22 | Your trusted public IP `/32` |

Do not expose SSH to the entire internet unless you have a specific, justified lab requirement. Prefer AWS Systems Manager Session Manager where suitable.

## Outbound rules

For a simple lab, the default outbound rule allows all outbound IPv4 traffic.

For production environments, consider restricting outbound traffic to the destinations and ports the application actually needs.

### Create the security group

Click **Create security group**.

### Learning point

A security group does not make a subnet public. Public connectivity requires suitable routing and IP addressing as well.

---

# 11. Step 7 — Create the Database Security Group

The database tier must accept connections only from the required application or web tier, depending on the architecture.

For this lab, we will allow a database connection from the web-server security group to demonstrate security-group referencing.

### Create the security group

Navigate to:

**VPC → Security groups → Create security group**

Enter:

```text
Security group name: database-server-sg
Description: Security group for database servers
VPC: devops-practical-vpc
```

## Inbound rule

For a MySQL database, use:

| Type | Protocol | Port | Source |
|---|---|---:|---|
| MySQL/Aurora | TCP | 3306 | `web-server-sg` |

For PostgreSQL, use port `5432` instead.

Select the web-server security group as the source rather than allowing `0.0.0.0/0`.

**Production recommendation:** In a three-tier architecture, the database security group should normally reference the application-server security group, not the public web-server security group. The rule above is a simplified lab example.

## Outbound rules

The default outbound rule is sufficient for this basic lab. Review and restrict it as needed for production.

Click **Create security group**.

### Important

Do not create a database security group rule that allows database access from:

```text
0.0.0.0/0
```

That would expose the database port to IPv4 sources permitted by the surrounding network configuration.

---

# 12. Step 8 — Launch Web Server 1

Navigate to:

**EC2 → Instances → Launch instances**

Configure:

| Setting | Value |
|---|---|
| Name | `web-server-a` |
| AMI | A suitable Linux AMI |
| Instance type | Appropriate free-tier-eligible or lab instance type, if available |
| VPC | `devops-practical-vpc` |
| Subnet | `public-subnet-a` |
| Auto-assign public IP | Enable, if needed |
| Security group | `web-server-sg` |

Select or create an appropriate key pair if using SSH.

Launch the instance.

## Launch Web Server 2

Repeat the process with:

```text
Name: web-server-b
Subnet: public-subnet-b
Security group: web-server-sg
```

Ensure that the instance receives public IPv4 addressing if you intend to access it directly from the internet.

### Verification

Open **EC2 → Instances**.

Check:

- Instance state is `Running`.
- Both web servers are in the correct VPC.
- Each server is in the intended Availability Zone.
- Each server has the expected private IP.
- Each server has a public IP if direct public access is required.
- The correct security group is attached.

---

# 13. Step 9 — Launch Database Server 1

Navigate to:

**EC2 → Instances → Launch instances**

Configure:

| Setting | Value |
|---|---|
| Name | `database-server-a` |
| AMI | A suitable Linux AMI |
| Instance type | Appropriate lab instance type |
| VPC | `devops-practical-vpc` |
| Subnet | `private-subnet-a` |
| Auto-assign public IP | Disabled |
| Security group | `database-server-sg` |

Launch the instance.

## Launch Database Server 2

Repeat with:

```text
Name: database-server-b
Subnet: private-subnet-b
Security group: database-server-sg
```

### Verification

Confirm:

- Both instances are running.
- Both instances have private IP addresses.
- Neither instance has a public IPv4 address.
- Both instances use the intended private subnets.
- The database security group is attached.

### Important operational note

Because these instances have no public IP addresses and their subnets have no internet default route, you cannot ordinarily connect to them directly from the public internet.

Use a suitable private access method, such as Systems Manager Session Manager when its prerequisites are configured, or a properly secured administrative path.

---

# 14. Step 10 — Verify the Network Configuration

## VPC

```text
Name: devops-practical-vpc
CIDR: 10.0.0.0/22
```

## Subnets

| Subnet | CIDR | Type | AZ |
|---|---|---|---|
| `public-subnet-a` | `10.0.0.0/24` | Public | AZ A |
| `public-subnet-b` | `10.0.1.0/24` | Public | AZ B |
| `private-subnet-a` | `10.0.2.0/24` | Private | AZ A |
| `private-subnet-b` | `10.0.3.0/24` | Private | AZ B |

## Route tables

| Route table | Subnet association | Default route |
|---|---|---|
| `public-route-table` | Both public subnets | `0.0.0.0/0 → IGW` |
| `private-route-table` | Both private subnets | No internet default route |

## Security groups

| Security group | Attached resources | Key inbound access |
|---|---|---|
| `web-server-sg` | Web servers | HTTP/HTTPS from the internet; SSH only from a trusted source if needed |
| `database-server-sg` | Database servers | Database port from the permitted source security group |

## EC2 instances

| Instance | Subnet | Public IP expected? |
|---|---|---|
| `web-server-a` | `public-subnet-a` | Yes, if direct public access is intended |
| `web-server-b` | `public-subnet-b` | Yes, if direct public access is intended |
| `database-server-a` | `private-subnet-a` | No |
| `database-server-b` | `private-subnet-b` | No |

---

# 15. Step 11 — Test Internet Connectivity

## Test 1: Web Server

From an external machine, try:

```bash
curl -I http://WEB_SERVER_PUBLIC_IP
```

Replace `WEB_SERVER_PUBLIC_IP` with the actual public IPv4 address.

This test will succeed only if a web server is installed, running, listening on port 80, and permitted by the network and host firewall configuration.

For example, on a suitable Amazon Linux instance, you can install and start Nginx using the commands appropriate for that AMI.

## Test 2: Database Server

The database instances should not be directly reachable from the public internet.

Verify that:

- The instances have no public IPv4 addresses.
- The private route table has no default route to the IGW.
- The database security group permits only the intended source and database port.

## Test 3: Private Connectivity

From an authorized web or application server, test the database's private IP on the required database port.

For MySQL:

```bash
nc -vz DATABASE_PRIVATE_IP 3306
```

For PostgreSQL:

```bash
nc -vz DATABASE_PRIVATE_IP 5432
```

These commands test TCP connectivity. They do not verify database credentials, replication, or application-level functionality.

---

# 16. Troubleshooting Checklist

## Web Server Cannot Be Reached

Check:

- Is the Internet Gateway attached to the correct VPC?
- Does the public route table contain `0.0.0.0/0 → IGW`?
- Are both public subnets associated with that route table?
- Does the EC2 instance have a public IPv4 address?
- Does the web security group allow inbound port 80 or 443?
- Is the application running?
- Are the network ACL and host firewall permitting the traffic?

## Private Database Server Cannot Be Reached

Check:

- Is the database in the intended private subnet?
- Is the correct security group attached?
- Does its security group permit the database port from the intended source?
- Is the database service running and listening?
- Are network ACLs permitting the traffic and return traffic?
- Is the source using the correct private IP address?

## EC2 Instance Cannot Download Software Packages

The private subnets in this lab do not have a route to the internet.

If a private instance requires outbound internet access, a typical solution is:

1. Create a NAT Gateway in a public subnet.
2. Ensure that subnet's route table sends `0.0.0.0/0` to the IGW.
3. Add `0.0.0.0/0 → NAT Gateway` to the appropriate private route table.
4. Check the NAT Gateway state, security configuration, DNS resolution, and required outbound connectivity.

NAT Gateways incur charges. For access only to supported AWS services, consider whether VPC endpoints can meet the requirement instead.

---

# 17. Interview Questions

### Q1. Why do we use `/22` for the VPC?

Because a `/22` contains 1,024 total IPv4 addresses, which is the smallest power-of-two CIDR block that accommodates at least 1,000 total addresses.

### Q2. Why does each `/24` subnet have 251 usable IP addresses in AWS?

A `/24` contains 256 total IPv4 addresses. AWS reserves five addresses in each IPv4 subnet:

```text
256 - 5 = 251
```

### Q3. Does attaching an Internet Gateway automatically make a subnet public?

No. The subnet needs an associated route table with a suitable route to the IGW. Direct IPv4 internet connectivity for an EC2 instance also generally requires public IPv4 addressing and appropriate security rules.

### Q4. Why do we create separate public and private route tables?

They allow different routing behavior for the two tiers. Public subnets can have a default route to an IGW, while private database subnets can remain without a direct internet default route.

### Q5. Why should we not allow database port 3306 from `0.0.0.0/0`?

That would allow traffic from any IPv4 source to attempt a connection whenever the surrounding routing and network controls permit it. The safer approach is to allow the required database port from a trusted application-tier security group.

### Q6. Can two subnets in the same VPC use overlapping CIDR ranges?

No. AWS rejects overlapping IPv4 subnet CIDR blocks in the same VPC.

### Q7. Why do we deploy resources across two Availability Zones?

To improve resilience against an Availability Zone-level failure, provided the application, data layer, and traffic distribution are also designed for high availability.

### Q8. What is the difference between a route table and a security group?

A route table determines where traffic is sent. A security group controls which inbound and outbound traffic is allowed for a resource.

### Q9. Can private instances access the internet without a public IP?

Yes, for outbound IPv4 internet connectivity, they can use a properly configured NAT Gateway. They can also access supported AWS services privately through appropriate VPC endpoints.

### Q10. Why do database servers not receive public IP addresses in this lab?

The goal is to keep the database tier private and reduce direct exposure to the internet.

---

# 18. Cleanup — Avoid Unnecessary AWS Charges

After finishing the lab, review your resources and delete those you no longer need.

Suggested cleanup order:

1. Terminate the EC2 instances.
2. Delete the NAT Gateway if you created one; wait for deletion to complete.
3. Release any Elastic IP addresses that are no longer needed.
4. Delete the route tables you created, after removing subnet associations where required.
5. Detach and delete the Internet Gateway.
6. Delete the custom security groups after they are no longer in use.
7. Delete the subnets.
8. Delete the VPC.

AWS may prevent deletion while dependencies remain. Follow the console's dependency errors and resolve them before continuing.

**Cost reminder:** EC2 instances, public IPv4 addresses, NAT Gateways, and other networking resources may incur charges. Check the current AWS pricing for your Region and clean up promptly.

---

# Final Learning Summary

In this practical, I learned how to:

- Create a VPC using CIDR `10.0.0.0/22`.
- Calculate the total number of IPv4 addresses.
- Divide a VPC into four non-overlapping `/24` subnets.
- Place subnets across two Availability Zones.
- Enable automatic public IPv4 assignment for public subnets.
- Create and attach an Internet Gateway.
- Configure public and private route tables.
- Associate route tables with the correct subnets.
- Create web-server and database-server security groups.
- Launch two web servers and two private database servers.
- Verify routing and connectivity.
- Troubleshoot common networking problems.

