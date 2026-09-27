# ☁️ Cloud Computing Fundamentals

> AWS Learning Notes — Sudharsan AWS Class

---

# 1. Why Cloud Computing?

Before cloud computing, most organizations used **traditional on-premises infrastructure**.

In the traditional approach, a company had to purchase and maintain its own:

- Physical Servers
- Storage
- Networking equipment
- Firewalls
- Data Center
- Power supply
- Cooling systems
- Backup systems

The company was responsible for managing the complete infrastructure.

### Traditional Infrastructure

Users  
↓  
Network  
↓  
Firewall  
↓  
Physical Server  
↓  
Application  
↓  
Database

---

# 2. What Was the Problem with the Traditional Approach?

## 2.1 High Initial Cost

The company needs to purchase physical infrastructure before running the application.

For example:

- Servers
- Storage
- Switches
- Routers
- Firewalls
- Racks
- UPS
- Cooling systems
- Data center

This requires a large **upfront investment**.

This type of spending is called:

**CAPEX — Capital Expenditure**

---

## 2.2 Hardware Procurement Takes Time

If the company needs a new server, they cannot immediately create one.

Typical process:

Requirement  
↓  
Purchase Request  
↓  
Vendor  
↓  
Hardware Delivery  
↓  
Data Center Installation  
↓  
Network Configuration  
↓  
Operating System Installation  
↓  
Application Deployment

This can take days or weeks depending on the organization.

---

## 2.3 Resource Underutilization

Suppose a physical server has:

- CPU: 16 Cores
- RAM: 64 GB

But the application only uses:

- CPU: 4 Cores
- RAM: 16 GB

A large amount of the server's capacity remains unused.

Example:

Server Capacity: 100%  
Application Usage: 25%  
Unused Capacity: 75%

This is inefficient resource utilization.

---

## 2.4 Scaling Problem

Suppose an application normally requires one server.

During high traffic:

Current capacity: 1 server  
Required capacity: 5 servers

In a traditional environment, the company needs to purchase additional physical servers.

The process takes time.

---

## 2.5 Hardware Maintenance

The organization is responsible for:

- Hardware maintenance
- Disk failures
- Server failures
- Power
- Cooling
- Networking
- Hardware replacement
- Physical security

---

## 2.6 Data Center Management

The company needs to manage:

- Data center space
- Electricity
- Cooling
- Physical security
- Network infrastructure
- Backup power
- Hardware
- Hardware replacement

---

## 2.7 Disaster Recovery

If the physical data center fails because of:

- Fire
- Flood
- Power failure
- Hardware failure
- Natural disaster

The application may become unavailable.

A separate disaster recovery infrastructure may be required.

---

# 3. What is a Database?

A **database is an organized collection of data** that allows applications to store, retrieve, update, and manage information.

### Simple Example

A company application may store customer information:

Customer ID | Name | Email  
101 | Arun | arun@example.com  
102 | Ravi | ravi@example.com  
103 | Kumar | kumar@example.com

This information is stored in a database.

### Examples of Databases

- MySQL
- PostgreSQL
- Oracle Database
- Microsoft SQL Server
- MongoDB

### AWS Database Services

- Amazon RDS
- Amazon Aurora
- Amazon DynamoDB

---

# 4. What is a Physical Server?

A **physical server** is a real hardware machine.

A physical server contains resources such as:

- CPU
- RAM
- Storage
- Network Interface

Basic structure:

Physical Server  
├── CPU  
├── RAM  
├── Storage  
└── Network Interface

If we directly install one operating system on one physical server:

Physical Server  
↓  
Operating System  
↓  
Application

The problem is that the server may be underutilized.

---

# 5. What is Virtualization?

**Virtualization** is the process of creating virtual versions of computing resources using physical hardware.

Instead of using one physical server for only one operating system, we can divide the physical server into multiple virtual machines.

Example:

Physical Server  
↓  
Virtualization  
↓  
VM 1  
VM 2  
VM 3

Each VM can run its own operating system.

---

# 6. What is a Hypervisor?

A **hypervisor** is software or firmware that allows multiple virtual machines to run on a single physical server.

Without a hypervisor:

Physical Server  
↓  
Operating System  
↓  
Application

With a hypervisor:

Physical Server  
↓  
Hypervisor  
├── VM 1  
├── VM 2  
└── VM 3

The hypervisor manages the physical resources and allocates them to virtual machines.

---

# 7. How Does a Hypervisor Solve the Problem?

Suppose a physical server has:

CPU: 16 Cores  
RAM: 64 GB

The hypervisor can allocate resources to multiple VMs.

Example:

VM 1 → 4 CPU + 16 GB RAM  
VM 2 → 4 CPU + 16 GB RAM  
VM 3 → 4 CPU + 16 GB RAM  
VM 4 → 4 CPU + 16 GB RAM

Now multiple workloads can run on the same physical server.

This improves resource utilization.

---

# 8. Benefits of a Hypervisor

## 8.1 Better Resource Utilization

Instead of one application using the entire physical server:

Physical Server  
↓  
Hypervisor  
↓  
Multiple VMs

The physical resources can be shared between multiple workloads.

---

## 8.2 Isolation

Each virtual machine behaves like an independent machine.

Example:

VM 1 → Application A  
VM 2 → Application B  
VM 3 → Database

Each VM has its own operating system and allocated resources.

---

## 8.3 Flexibility

Virtual machines can be:

- Created
- Deleted
- Started
- Stopped
- Resized
- Migrated

more easily than physical servers.

---

## 8.4 Cost Reduction

Multiple workloads can run on the same physical hardware.

Therefore, fewer physical servers may be required.

---

# 9. Physical Server vs Hypervisor

## Without Hypervisor

Physical Server  
↓  
Operating System  
↓  
Application

One physical server is primarily being used by one operating system.

---

## With Hypervisor

Physical Server  
↓  
Hypervisor  
├── VM 1 → Linux  
├── VM 2 → Linux  
└── VM 3 → Windows

Multiple operating systems can run on the same physical server.

---

# 10. What is Cloud Computing?

**Cloud computing is the delivery of computing resources over a network, usually the internet, on demand.**

Instead of purchasing and maintaining all physical infrastructure ourselves, we can consume computing resources from a cloud provider.

Cloud resources can include:

- Compute
- Storage
- Database
- Networking
- Security
- Monitoring
- Application services

---

# 11. What is AWS?

**AWS — Amazon Web Services** is a cloud computing platform that provides a large number of cloud services.

Examples:

Compute → EC2  
Storage → S3  
Database → RDS  
Networking → VPC  
DNS → Route 53  
Monitoring → CloudWatch  
Serverless → Lambda

---

# 12. Traditional Infrastructure vs AWS

## Traditional Approach

In the traditional approach, the company owns and manages the infrastructure.

Company  
↓  
Data Center  
↓  
Physical Servers  
↓  
Operating Systems  
↓  
Applications

The company is responsible for purchasing and maintaining the infrastructure.

---

## AWS Cloud

With AWS, the company consumes cloud infrastructure and services from AWS.

Company  
↓  
Internet  
↓  
AWS  
↓  
EC2 / S3 / RDS / VPC / etc.  
↓  
Application

AWS operates the underlying cloud infrastructure.

---

# 13. How AWS Overcomes Traditional Technology Problems

## Problem 1 — Large Upfront Investment

### Traditional Approach

The company must purchase:

- Physical servers
- Storage
- Network devices
- Firewalls
- Data center infrastructure

This requires a large upfront investment.

### AWS Approach

The organization can provision cloud resources when required.

Example:

AWS  
↓  
Create EC2  
↓  
Use the instance  
↓  
Pay according to the applicable pricing model

This can reduce the need for large upfront hardware investment.

---

# 14. Problem — Slow Server Provisioning

### Traditional Approach

Order Hardware  
↓  
Hardware Delivery  
↓  
Rack Installation  
↓  
Network Configuration  
↓  
Operating System Installation  
↓  
Application Deployment

This can take considerable time.

### AWS Approach

A cloud resource can be provisioned through:

- AWS Management Console
- AWS CLI
- AWS API
- Infrastructure as Code

Example:

Requirement  
↓  
AWS  
↓  
EC2  
↓  
Instance Available

Cloud provisioning can therefore be much faster than traditional hardware procurement.

---

# 15. Problem — Scaling

### Traditional Approach

Low Traffic  
↓  
1 Physical Server

High Traffic  
↓  
Need additional physical servers  
↓  
Purchase hardware  
↓  
Install hardware  
↓  
Configure hardware

### AWS Approach

Cloud resources can be scaled according to workload requirements.

Example:

Low Traffic  
↓  
2 EC2 Instances

High Traffic  
↓  
10 EC2 Instances

AWS Auto Scaling can automate this type of scaling.

---

# 16. Problem — Resource Underutilization

### Traditional Approach

A physical server may have:

64 GB RAM  
16 CPU Cores

But the application may only use:

16 GB RAM  
4 CPU Cores

The remaining capacity may remain unused.

### Cloud Approach

Resources can be selected based on workload requirements.

Small workload  
↓  
Smaller EC2 Instance

Large workload  
↓  
Larger EC2 Instance

This provides greater flexibility in resource allocation.

---

# 17. Problem — Global Availability

### Traditional Approach

A company may have only one physical data center.

If that data center fails, the application may become unavailable.

### AWS Approach

AWS provides infrastructure across:

- Regions
- Availability Zones
- Edge Locations

Example:

AWS Region  
├── Availability Zone A  
├── Availability Zone B  
└── Availability Zone C

Applications can be designed across multiple Availability Zones to improve availability and fault tolerance.

---

# 18. Problem — Disaster Recovery

### Traditional Approach

The company may need to build and maintain a separate disaster recovery data center.

This can be expensive and complex.

### AWS Approach

AWS services can be used to build disaster recovery solutions using:

- Backups
- Snapshots
- Replication
- Multi-AZ architectures
- Cross-Region architectures
- Disaster recovery strategies

---

# 19. Problem — Hardware Maintenance

### Traditional Approach

The company manages:

- Physical servers
- Storage
- Network hardware
- Power
- Cooling
- Hardware replacement

### AWS Approach

AWS manages the underlying infrastructure for AWS services.

The customer can focus more on:

- Applications
- Configuration
- Data
- Security configuration
- Deployment
- Monitoring

The exact responsibilities depend on the AWS service and follow the **AWS Shared Responsibility Model**.

---

# 20. Problem — Hardware Failure

### Traditional Approach

If a physical server fails:

Physical Server Failure  
↓  
Application Downtime  
↓  
Manual Hardware Replacement

### AWS Approach

AWS architectures can use services and features such as:

- Auto Scaling
- Load Balancers
- Multi-AZ
- Health Checks
- Automated replacement

This allows applications to be designed for higher availability and fault tolerance.

---

# 21. Advantages of Cloud Computing Over Traditional Infrastructure

| Traditional Approach | Cloud Computing |
|---|---|
| Large upfront investment | Pay-as-you-go options |
| Physical hardware required | Resources available on demand |
| Slow hardware provisioning | Faster resource provisioning |
| Difficult to scale quickly | Flexible scaling |
| Hardware maintenance required | Provider manages underlying infrastructure |
| Limited physical location | Global infrastructure |
| Capacity planning required | Flexible resource selection |
| Data center required | Cloud provider operates data centers |
| Separate DR infrastructure may be required | Cloud services can support DR architectures |
| Manual infrastructure management | APIs, CLI and Infrastructure as Code |
| Hardware replacement required | Cloud architectures can automate recovery |
| Large physical infrastructure | Virtual and managed services |

---

# 22. CAPEX vs OPEX

## CAPEX — Capital Expenditure

CAPEX means spending money upfront to purchase physical infrastructure.

Examples:

- Servers
- Storage
- Network equipment
- Firewalls
- Data center

Example:

Company  
↓  
Buy Server  
↓  
Own Server  
↓  
Maintain Server

---

## OPEX — Operational Expenditure

OPEX refers to ongoing operational spending.

Cloud computing can shift much infrastructure spending toward an operational/pay-as-you-use model.

Example:

AWS  
↓  
Use Resources  
↓  
Pay According to Applicable Usage/Pricing

---

# 23. Evolution of Infrastructure

The evolution can be understood as:

Traditional Infrastructure  
↓  
Physical Servers  
↓  
Virtualization  
↓  
Hypervisor  
↓  
Virtual Machines  
↓  
Cloud Computing  
↓  
On-Demand Infrastructure  
↓  
Automation  
↓  
Infrastructure as Code  
↓  
DevOps

---

# 24. Hypervisor vs Cloud Computing

Hypervisor and cloud computing are **not the same thing**.

## Hypervisor

A hypervisor is a technology used to create and manage virtual machines on physical hardware.

Physical Server  
↓  
Hypervisor  
↓  
Virtual Machines

## Cloud Computing

Cloud computing is a broader model for consuming computing resources as services over a network.

Cloud Provider  
├── Compute  
├── Storage  
├── Database  
├── Networking  
├── Security  
└── Monitoring

Cloud providers may use virtualization and many other technologies internally, but cloud computing is much broader than virtualization.

---

# 25. Why Do Companies Use AWS?

Companies may use AWS because it provides a broad range of infrastructure and managed services.

Common reasons include:

- Faster infrastructure provisioning
- Flexible scaling
- Global infrastructure
- Automation
- Managed services
- High availability options
- Disaster recovery options
- Security services
- Monitoring
- Infrastructure as Code
- Pay-as-you-go pricing models

---

# 26. Job Opportunities After Learning AWS Cloud Computing

Learning AWS can be useful for several technology roles.

## 26.1 Cloud Support Engineer

Typical responsibilities:

- AWS resource management
- EC2
- S3
- IAM
- Networking
- Monitoring
- Troubleshooting
- Incident support

---

## 26.2 Cloud Engineer

Typical responsibilities:

- AWS infrastructure
- VPC
- EC2
- Load Balancers
- Auto Scaling
- RDS
- S3
- IAM
- Cloud monitoring
- Automation

---

## 26.3 DevOps Engineer

AWS can be combined with DevOps technologies such as:

- Linux
- Git
- GitHub
- CI/CD
- Jenkins
- GitHub Actions
- Docker
- Kubernetes
- Terraform
- Ansible
- Monitoring

Typical DevOps flow:

Developer  
↓  
GitHub  
↓  
CI/CD  
↓  
Build  
↓  
Docker  
↓  
AWS  
↓  
Deployment  
↓  
Monitoring

---

## 26.4 Site Reliability Engineer — SRE

SRE roles commonly involve:

- AWS
- Linux
- Networking
- Automation
- Monitoring
- Observability
- Incident response
- Reliability
- Kubernetes
- Infrastructure as Code
- CI/CD

---

## 26.5 Cloud / Infrastructure Engineer

Responsibilities can include:

- Cloud infrastructure
- Networking
- Compute
- Storage
- Security
- Automation
- Infrastructure as Code
- Monitoring
- Troubleshooting

---

## 26.6 Cloud Architect

With additional experience, AWS knowledge can be applied to architecture-focused responsibilities.

Typical areas:

- AWS architecture
- High availability
- Scalability
- Security
- Networking
- Cost optimization
- Disaster recovery
- Infrastructure design

---

# 27. Important Learning Path

AWS is one part of becoming a strong Cloud/DevOps Engineer.

A practical learning path is:

Linux  
↓  
Networking  
↓  
AWS Fundamentals  
↓  
EC2 / S3 / IAM / VPC  
↓  
Load Balancer / Auto Scaling  
↓  
RDS / Route 53 / CloudFront  
↓  
CloudWatch / CloudTrail  
↓  
AWS CLI  
↓  
Terraform  
↓  
Git / GitHub  
↓  
CI/CD  
↓  
Docker  
↓  
Kubernetes  
↓  
Monitoring  
↓  
Real-World Projects

---

# 28. Important Concepts to Remember

## Cloud Computing

Cloud computing is the delivery of computing resources over a network on demand instead of owning all physical infrastructure ourselves.

## Physical Server

A physical server is a real hardware machine that provides CPU, memory, storage and networking resources.

## Virtualization

Virtualization is the process of creating virtual computing resources using physical hardware.

## Hypervisor

A hypervisor is software or firmware that enables multiple virtual machines to run on a physical server.

## Virtual Machine

A virtual machine is a software-defined computer running on physical infrastructure through virtualization.

## Database

A database is an organized collection of data that applications can store, retrieve, update and manage.

## AWS

AWS is a cloud computing platform that provides computing, storage, networking, database, security, monitoring and many other services.

---

# 29. Interview Questions

## Cloud Computing

1. What is cloud computing?
2. Why did companies move from traditional infrastructure to cloud?
3. What problems existed in traditional infrastructure?
4. What are the advantages of cloud computing?
5. What is CAPEX?
6. What is OPEX?
7. What is scalability?
8. What is elasticity?
9. What is high availability?
10. What is disaster recovery?

## Physical Server and Virtualization

11. What is a physical server?
12. What is virtualization?
13. What is a hypervisor?
14. How does a hypervisor work?
15. Why do we need a hypervisor?
16. What is a virtual machine?
17. How does virtualization improve resource utilization?
18. What is the difference between a physical server and a virtual machine?
19. What are the advantages of virtualization?

## Database

20. What is a database?
21. Why do applications need databases?
22. What is the difference between a database and a file?
23. What are some common databases?
24. What database services are available in AWS?

## AWS

25. What is AWS?
26. How is AWS different from traditional infrastructure?
27. How does AWS solve the problems of traditional infrastructure?
28. What are AWS Regions?
29. What are Availability Zones?
30. What is the AWS Shared Responsibility Model?
31. Why do companies use AWS?
32. What are the major AWS services?

## Career

33. What roles can AWS knowledge help prepare for?
34. What skills are required for a Cloud Engineer?
35. What additional skills are useful for a DevOps Engineer?
36. What additional skills are useful for an SRE?
37. Why are Linux and networking important for AWS?
38. Why are Terraform and Infrastructure as Code important?
39. Why are Docker and Kubernetes useful with AWS?
40. Why are CI/CD and AWS commonly used together?

---

# 30. Simple Overall Understanding

## Traditional IT

Company  
↓  
Buy Physical Servers  
↓  
Install Hardware  
↓  
Install Operating System  
↓  
Deploy Application  
↓  
Maintain Hardware  
↓  
Difficult / Slow Scaling

---

## Virtualization

Physical Server  
↓  
Hypervisor  
↓  
Multiple Virtual Machines  
↓  
Better Resource Utilization  
↓  
Better Hardware Utilization

---

## Cloud Computing

Cloud Provider  
↓  
On-Demand Resources  
↓  
Compute + Storage + Network + Database  
↓  
Automation + Scaling + Global Infrastructure

---

## AWS

AWS  
↓  
EC2 + S3 + VPC + RDS + IAM  
↓  
Load Balancer + Auto Scaling  
↓  
CloudWatch + CloudTrail  
↓  
Terraform + CI/CD + DevOps  
↓  
Production Cloud Infrastructure

---

# 31. Final Understanding

The main reason cloud computing became important is that organizations wanted to overcome many limitations of owning and operating all infrastructure themselves.

The progression is:

**Physical Infrastructure → Virtualization → Cloud Computing → Automation → DevOps**

Traditional infrastructure required organizations to purchase, install and maintain physical infrastructure.

Virtualization improved resource utilization by allowing multiple virtual machines to run on the same physical server.

Cloud computing went further by allowing organizations to consume computing resources on demand from cloud providers.

AWS provides services for:

**Compute → Storage → Networking → Database → Security → Monitoring → Automation**

For a Cloud or DevOps Engineer, the important skill is not simply memorizing AWS service names.

The goal is to understand:

**Why the service exists → What problem it solves → How it works → How to configure it → How services connect → How to automate it → How to troubleshoot it → How it is used in production**