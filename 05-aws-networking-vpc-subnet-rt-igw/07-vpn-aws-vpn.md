# AWS Networking – VPN (Hardware VPN, Software VPN, OpenVPN and AWS VPN Services)

## 1. What is VPN?

**VPN – Virtual Private Network**

A VPN creates an encrypted connection between two networks or between a user and a network over an untrusted network such as the internet.

It allows users or organizations to communicate securely without exposing private network traffic directly to the public internet.

### Why do we need VPN?

Consider an organization with:

- Head office in Chennai.
- Branch office in Bangalore.
- AWS infrastructure running in the Mumbai Region.
- Employees working remotely.

The organization wants employees and branch offices to access AWS private resources securely.

Instead of exposing private servers directly to the internet, we can use VPN connectivity.

### Example architecture

```text
       CHENNAI OFFICE
       Office Network
       192.168.1.0/24
              |
        VPN Router
              |
          INTERNET
              |
       AWS VPN Gateway
              |
           AWS VPC
         10.0.0.0/16
              |
       Private EC2 Instance
```

The VPN creates an encrypted tunnel between the office network and AWS.

---

## 2. Types of VPN

VPN solutions can be broadly classified into:

1. Hardware VPN.
2. Software VPN.

### A. Hardware VPN

A Hardware VPN uses a dedicated physical networking device to establish a secure VPN connection.

Examples:

- Cisco VPN Router.
- Fortinet Firewall.
- Palo Alto Firewall.
- Sophos Firewall.
- Juniper VPN Router.

These devices are usually deployed in an organization's office or data centre.

### How Hardware VPN works

1. An organization has an office network.
2. A VPN-capable hardware router or firewall is installed.
3. The device establishes an encrypted VPN tunnel over the internet.
4. The AWS VPN endpoint receives the connection.
5. The office network can communicate with AWS private resources through the tunnel.

### Architecture

```text
        OFFICE NETWORK
          192.168.1.0/24
                 |
          Hardware VPN
          Router/Firewall
                 |
              INTERNET
                 |
          AWS VPN Gateway
                 |
               AWS VPC
             10.0.0.0/16
                 |
           Private EC2
```

### Advantages

- Dedicated networking hardware.
- Centralized network management.
- Suitable for organizations with existing enterprise networks.
- Can support advanced firewall and routing features.
- Can provide high availability using redundant devices and tunnels.

### Disadvantages

- Hardware purchasing cost.
- Maintenance and configuration.
- Requires networking expertise.
- Hardware replacement and upgrades.

---

### B. Software VPN

A Software VPN uses software running on a server or virtual machine to establish an encrypted VPN connection.

Instead of purchasing a dedicated VPN hardware device, we install VPN software on a server.

Examples:

- OpenVPN.
- WireGuard.
- StrongSwan.
- SoftEther VPN.

In AWS, we can launch an EC2 instance and install VPN software on it.

### How Software VPN works

1. Launch an EC2 instance in AWS.
2. Install VPN software such as OpenVPN.
3. Configure the VPN server.
4. Generate client configuration files and certificates.
5. Install a VPN client on the employee's laptop.
6. Connect the laptop to the VPN server.
7. The VPN server provides access to authorized private AWS networks.

### Architecture

```text
         EMPLOYEE LAPTOP
                |
         OpenVPN Client
                |
             INTERNET
                |
         Public IP / EIP
                |
          EC2 VPN SERVER
          OpenVPN Server
                |
             AWS VPC
                |
          Private Subnet
                |
          Private EC2
```

### Advantages

- No dedicated physical VPN appliance required.
- Can run on an EC2 instance.
- Flexible configuration.
- Useful for learning and lab environments.
- Can support remote employees.

### Disadvantages

- We manage the operating system.
- We manage VPN software updates.
- We manage certificates and user access.
- We manage server availability and scaling.
- Performance depends on EC2 instance capacity and configuration.

---

## 3. Hardware VPN vs Software VPN

| Feature | Hardware VPN | Software VPN |
|---|---|---|
| Implementation | Physical router/firewall | Software on server |
| Infrastructure | Dedicated hardware | EC2 or another server |
| Cost | Hardware and maintenance | Server and software costs |
| Maintenance | Hardware and firmware | OS and software |
| Configuration | Network device configuration | Software configuration |
| Example | Cisco, Fortinet | OpenVPN, WireGuard |
| Common use | Office-to-cloud connectivity | Remote access and virtual networking |

**Important:** Hardware VPN and Software VPN describe implementation approaches. They are not the same classification as AWS Site-to-Site VPN and AWS Client VPN.

---

# 4. Types of VPN in AWS

AWS provides managed VPN services and supports third-party VPN appliances.

The main AWS VPN services are:

1. AWS Site-to-Site VPN.
2. AWS Client VPN.

We can also use third-party VPN software on EC2 instances.

## A. AWS Site-to-Site VPN

AWS Site-to-Site VPN connects an on-premises network to an AWS VPC using encrypted IPsec tunnels over the internet.

### Why do we need Site-to-Site VPN?

Imagine an organization has an on-premises data centre.

The organization wants to connect its office network to AWS so that office employees can access private EC2 instances.

Instead of assigning public IPs to private EC2 instances, we establish a Site-to-Site VPN connection.

### Architecture

```text
       ON-PREMISES DATA CENTRE
          192.168.1.0/24
                 |
          Customer Gateway
          Hardware VPN Device
                 |
              INTERNET
                 |
          IPsec VPN Tunnels
                 |
          AWS VPN Endpoint
                 |
       Virtual Private Gateway
                 |
              AWS VPC
             10.0.0.0/16
                 |
          Private EC2
```

### Important AWS components

**1. Customer Gateway (CGW)**

Represents the customer-side VPN device or software VPN endpoint in AWS configuration.

It contains information such as:
- Public IP address of the customer VPN device.
- Routing information.
- BGP ASN, when using dynamic routing.

**2. Virtual Private Gateway (VGW)**

A logical VPN gateway attached to an AWS VPC.

It acts as the AWS-side endpoint for Site-to-Site VPN connections when using the VGW architecture.

**3. VPN Connection**

The connection between the Customer Gateway and the AWS VPN gateway.

It uses IPsec tunnels to encrypt communication.

**4. Transit Gateway (TGW)**

An alternative AWS-side VPN endpoint for connecting Site-to-Site VPNs to multiple VPCs and networks through a central hub.

### Site-to-Site VPN configuration steps

1. Create a VPC.
2. Create private subnets.
3. Create a Customer Gateway resource in AWS.
4. Create or select a Virtual Private Gateway, or use Transit Gateway.
5. Attach the VGW to the VPC if using VGW.
6. Create a Site-to-Site VPN connection.
7. Select the Customer Gateway.
8. Select the AWS-side VPN gateway.
9. Configure static or dynamic routing.
10. Download the VPN configuration file.
11. Configure the on-premises VPN router/firewall.
12. Establish the IPsec tunnels.
13. Configure VPC route tables.
14. Test communication between office and AWS private resources.

### Important points

- Site-to-Site VPN uses IPsec encryption.
- AWS provides two VPN tunnels per connection for redundancy.
- It can use static routing or dynamic routing through BGP.
- The customer side requires a compatible VPN device or software appliance.
- The AWS-side endpoint can be a VGW or Transit Gateway.

---

## B. AWS Client VPN

AWS Client VPN allows individual remote users to securely connect to an AWS VPC or connected networks.

For example:

An employee is working from home and needs to access a private EC2 instance in AWS.

Instead of exposing the EC2 instance to the internet, the employee connects through AWS Client VPN.

### Architecture

```text
          EMPLOYEE LAPTOP
                 |
          AWS VPN Client
                 |
              INTERNET
                 |
         AWS CLIENT VPN
              ENDPOINT
                 |
           VPC SUBNETS
                 |
          Private EC2
          10.0.1.10
```

### How AWS Client VPN works

1. Create a Client VPN endpoint.
2. Configure authentication.
3. Associate target subnets in the VPC.
4. Configure authorization rules.
5. Configure routes to destination networks.
6. Download the client configuration file.
7. Install a compatible VPN client.
8. Connect to the Client VPN endpoint.
9. Access authorized private resources.

### Authentication methods

AWS Client VPN supports:

- Mutual authentication using certificates.
- Active Directory authentication.
- Federated authentication using SAML-based identity providers.

### Advantages

- Managed by AWS.
- Suitable for remote employees.
- No need to maintain your own OpenVPN EC2 server.
- Supports user authentication and authorization.
- Can connect users to VPC resources and connected networks.

---

# 5. OpenVPN – Software VPN Practical Using EC2

This is the practical where we launch an EC2 instance and install OpenVPN software.

**Objective:** Allow a remote laptop to securely connect to a private EC2 instance through an OpenVPN server.

### Architecture

```text
            REMOTE LAPTOP
                  |
             OpenVPN Client
                  |
               INTERNET
                  |
             Elastic IP
                  |
          PUBLIC EC2 INSTANCE
           OpenVPN Server
                  |
               AWS VPC
                  |
             PRIVATE SUBNET
                  |
           PRIVATE EC2 SERVER
```

## Step 1 – Create a VPC

1. Open AWS Console.
2. Search for VPC.
3. Create a VPC.
4. Name: `OpenVPN-VPC`.
5. IPv4 CIDR: `10.0.0.0/16`.
6. Create VPC.

## Step 2 – Create Public and Private Subnets

Create a public subnet:

```text
Name: OpenVPN-Public-Subnet
CIDR: 10.0.1.0/24
```

Create a private subnet:

```text
Name: OpenVPN-Private-Subnet
CIDR: 10.0.2.0/24
```

Enable auto-assign public IPv4 for the public subnet.

Keep it disabled for the private subnet.

## Step 3 – Create an Internet Gateway

1. Open VPC → Internet Gateways.
2. Create Internet Gateway.
3. Name: `OpenVPN-IGW`.
4. Attach it to `OpenVPN-VPC`.

## Step 4 – Configure Public Route Table

Create a public route table.

Add:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |
| `0.0.0.0/0` | Internet Gateway |

Associate the public subnet with this route table.

## Step 5 – Launch OpenVPN EC2 Instance

1. Open EC2 Dashboard.
2. Click Launch Instance.
3. Search AWS Marketplace for OpenVPN Access Server.
4. Select the official OpenVPN Access Server AMI from the appropriate publisher.
5. Review the AMI pricing and licensing details.
6. Select an instance type supported by the AMI.
7. Select `OpenVPN-VPC`.
8. Select `OpenVPN-Public-Subnet`.
9. Enable public IP assignment.
10. Create or select a Security Group.
11. Launch the instance.

**Important:** OpenVPN Access Server is different from the free OpenVPN Community Edition. Access Server has licensing and usage limits that depend on its current offering.

## Step 6 – Configure Security Group

The exact rules depend on the OpenVPN Access Server version and configuration.

Typical ports include:

| Port | Protocol | Purpose |
|---|---|---|
| 22 | TCP | SSH administration |
| 443 | TCP | Web administration and VPN client connection |
| 943 | TCP | Admin/client web interface, depending on configuration |
| 1194 | UDP | OpenVPN tunnel, if configured for UDP |

Restrict SSH and administrative access to trusted IP addresses.

Do not blindly open every port to `0.0.0.0/0`.

## Step 7 – Allocate Elastic IP

1. Open EC2 Dashboard.
2. Select Elastic IPs.
3. Click Allocate Elastic IP address.
4. Allocate an Elastic IP.
5. Select the address.
6. Click Actions → Associate Elastic IP address.
7. Select the OpenVPN EC2 instance.
8. Associate it.

Now the OpenVPN server has a persistent public IPv4 address.

## Step 8 – Configure OpenVPN Access Server

1. Connect to the EC2 instance using SSH.
2. Follow the publisher's setup instructions for the selected AMI.
3. Configure the administrator account.
4. Open the Access Server administration portal.
5. Configure authentication and user accounts.
6. Configure VPN client address allocation.
7. Configure the private VPC network routes that VPN clients are allowed to access.
8. Configure the required access permissions.
9. Download the client profile.

The exact initial administrator credentials and setup procedure depend on the selected Marketplace AMI version.

## Step 9 – Install OpenVPN Client

1. Download the OpenVPN client profile.
2. Install a compatible OpenVPN client on your laptop.
3. Import the downloaded profile.
4. Connect to the VPN server.
5. Authenticate using the configured user credentials.

Now the laptop can establish an encrypted VPN connection to the OpenVPN server.

## Step 10 – Access the Private EC2 Instance

Launch another EC2 instance in the private subnet.

Example:

```text
Private EC2 IP: 10.0.2.10
```

Configure the private EC2 Security Group to allow the required application or SSH traffic from the VPN client CIDR or an appropriate security-group reference.

Test connectivity from your laptop after connecting to the VPN:

```bash
ping 10.0.2.10
```

For SSH, if authorized:

```bash
ssh ec2-user@10.0.2.10
```

**Important networking requirements:**

- Enable IP forwarding on the VPN server where required.
- Configure the appropriate route from VPN clients to the VPC.
- Ensure the VPC route tables have a return path to the VPN client network, often through the VPN server ENI.
- Configure security groups and network ACLs.
- Avoid overlapping VPN client and VPC CIDR ranges.
- Configure source/destination checking appropriately for a VPN appliance that forwards traffic.
- Ensure the VPN server can reach the private subnet.

A VPN connection being established does not automatically mean every private VPC resource is reachable.

---

# 6. AWS Site-to-Site VPN vs AWS Client VPN vs OpenVPN on EC2

| Feature | Site-to-Site VPN | AWS Client VPN | OpenVPN on EC2 |
|---|---|---|---|
| Main purpose | Connect networks | Connect individual users | Connect users or networks |
| Management | AWS manages AWS side | AWS managed | Customer managed |
| Client software | Not normally required on each user device | VPN client required | OpenVPN client required |
| Office VPN router | Required on customer side | Not required | Not required for remote-user setup |
| Encryption | IPsec | TLS-based VPN | TLS-based OpenVPN |
| Authentication | Device/tunnel configuration and optional BGP | Certificates, Active Directory or SAML | Depends on Access Server configuration |
| Infrastructure | AWS gateway plus customer VPN device | AWS-managed endpoint | EC2 instance |
| Maintenance | Customer manages customer gateway | Mostly AWS managed | Customer manages server |

---

# 7. AWS VPN Connectivity Types – Quick Revision

### 1. Site-to-Site VPN with Virtual Private Gateway

Connects an on-premises network to one AWS VPC.

```text
Office Network
      |
Customer Gateway
      |
IPsec VPN
      |
Virtual Private Gateway
      |
AWS VPC
```

### 2. Site-to-Site VPN with Transit Gateway

Connects an on-premises network to a Transit Gateway, which can provide access to multiple attached VPCs according to routing and security rules.

```text
       Office Network
             |
       Site-to-Site VPN
             |
       Transit Gateway
          /    |    \
       VPC-A  VPC-B  VPC-C
```

### 3. AWS Client VPN

Connects individual employees' devices to AWS networks.

```text
Laptop
   |
VPN Client
   |
AWS Client VPN
   |
VPC
   |
Private EC2
```

### 4. Third-party Software VPN on EC2

Uses software such as OpenVPN, WireGuard or StrongSwan installed on an EC2 instance.

```text
Remote Laptop
      |
   Internet
      |
VPN Server on EC2
      |
   AWS VPC
      |
Private Resources
```

---

# 8. VPN Ports and Protocols

| Protocol / Port | Purpose |
|---|---|
| IPsec ESP (IP protocol 50) | Encrypted VPN data |
| UDP 500 | IKE negotiation |
| UDP 4500 | IPsec NAT Traversal |
| UDP 1194 | Common OpenVPN default |
| TCP 443 | HTTPS and commonly configured VPN access |

**Important:** AWS Site-to-Site VPN commonly uses IKE/IPsec. OpenVPN commonly uses UDP 1194, but it can be configured for other ports and protocols.

---

# 9. Scenario-Based Interview Questions

### Q1. Your company has an office network and wants to connect it securely to an AWS VPC. Which VPN would you use?

**Answer:** AWS Site-to-Site VPN.

It creates encrypted IPsec tunnels between the on-premises VPN device and AWS.

### Q2. An employee is working from home and needs to access a private EC2 instance. Which VPN would you use?

**Answer:** AWS Client VPN or a self-managed remote-access VPN such as OpenVPN on EC2.

For an AWS-managed solution, I would consider AWS Client VPN.

### Q3. What is the difference between Site-to-Site VPN and Client VPN?

**Answer:**

- Site-to-Site connects two networks.
- Client VPN connects individual users or devices to a network.

### Q4. Why do we need a Customer Gateway?

**Answer:** It represents the customer-side VPN device or software endpoint in AWS configuration. It provides the information needed to establish the VPN connection.

### Q5. What is a Virtual Private Gateway?

**Answer:** It is an AWS-side logical VPN gateway attached to a VPC for Site-to-Site VPN connectivity.

### Q6. What is the purpose of Transit Gateway in VPN architecture?

**Answer:** Transit Gateway acts as a central hub connecting VPNs, multiple VPCs and other supported network attachments.

### Q7. Why do we need two VPN tunnels in AWS Site-to-Site VPN?

**Answer:** AWS provides two tunnels to support redundancy and improve availability. The customer gateway should be configured to support both tunnels.

### Q8. What is OpenVPN?

**Answer:** OpenVPN is software used to establish encrypted VPN connections. We can install it on an EC2 instance to create a self-managed VPN server.

### Q9. Does installing OpenVPN on EC2 automatically provide access to all private subnets?

**Answer:** No.

We must configure VPN client routes, VPC routes, IP forwarding, security groups, NACLs and return traffic paths.

### Q10. What happens if the VPN connects successfully but the private EC2 instance cannot be reached?

**Answer:** I would check:

1. VPN client routing.
2. VPN server forwarding configuration.
3. VPC route table.
4. Security Group.
5. Network ACL.
6. Return route to the VPN client CIDR.
7. Overlapping IP address ranges.

### Q11. What is IPsec?

**Answer:** IPsec is a collection of protocols used to secure IP communications through encryption, authentication and integrity protection. AWS Site-to-Site VPN uses IPsec tunnels.

### Q12. Can we use a software VPN instead of a hardware VPN?

**Answer:** Yes. We can use compatible software VPN solutions on a server or virtual appliance instead of a physical VPN router, provided the solution supports the required VPN configuration.

---

# 10. Practical Learning Checklist

## Hardware / Site-to-Site VPN Concepts

- [ ] Understand hardware VPN devices.
- [ ] Understand Customer Gateway.
- [ ] Understand Virtual Private Gateway.
- [ ] Understand IPsec tunnels.
- [ ] Understand static routing and BGP.
- [ ] Understand Transit Gateway VPN attachment.

## OpenVPN Software VPN Practical

- [ ] Create a VPC.
- [ ] Create a public subnet.
- [ ] Create a private subnet.
- [ ] Attach an Internet Gateway.
- [ ] Configure the public route table.
- [ ] Launch an OpenVPN Access Server AMI from AWS Marketplace.
- [ ] Configure the security group.
- [ ] Allocate an Elastic IP.
- [ ] Associate Elastic IP with OpenVPN EC2.
- [ ] Configure VPN users.
- [ ] Download VPN client configuration.
- [ ] Install OpenVPN client on laptop.
- [ ] Connect to the VPN.
- [ ] Launch a private EC2 instance.
- [ ] Configure private routing and return routing.
- [ ] Test private EC2 connectivity.

## AWS Managed VPN Practical

- [ ] Understand AWS Site-to-Site VPN.
- [ ] Understand AWS Client VPN.
- [ ] Understand Customer Gateway and Virtual Private Gateway.
- [ ] Understand Transit Gateway.
- [ ] Understand authentication and routing.
- [ ] Understand VPN tunnel redundancy.

---

# 11. Final Interview Takeaways

1. Hardware VPN uses dedicated networking hardware.
2. Software VPN uses VPN software running on a server or virtual machine.
3. AWS Site-to-Site VPN connects on-premises networks to AWS.
4. AWS Client VPN connects individual remote users to AWS networks.
5. OpenVPN can be installed on an EC2 instance to create a self-managed VPN server.
6. Customer Gateway represents the customer-side VPN endpoint in AWS configuration.
7. Virtual Private Gateway is an AWS-side VPN gateway attached to a VPC.
8. Transit Gateway can connect multiple VPCs and VPNs through a central hub.
9. IPsec provides encryption for AWS Site-to-Site VPN.
10. A VPN connection alone does not guarantee access; routes and security rules must also be configured.

**Remember:**

- Office network to AWS VPC → Site-to-Site VPN.
- Remote employee laptop to AWS VPC → Client VPN.
- Self-managed VPN server on EC2 → OpenVPN / WireGuard.
- Multiple VPCs and networks → Transit Gateway with suitable VPN attachments.