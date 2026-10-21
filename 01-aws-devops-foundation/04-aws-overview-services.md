# AWS Overview and Services

**Course:** Sudharsan — AWS Cloud / DevOps  
**Topic:** AWS Cloud Introduction and Overview  
**Suggested repository path:** `AWS/01-Cloud-Computing-Fundamentals/aws-overview-services.md`

> **Study approach:** Review the concepts, repeat the practical steps in your own AWS account, and add your own screenshots in the marked locations. Do not add passwords, access keys, private keys, account-sensitive information, or billing details to a public repository.

---

## 1. What Is Cloud Computing?

Cloud computing is the on-demand delivery of computing resources—such as servers, storage, databases, networking, and software—over a network, usually with usage-based pricing.

Instead of buying and maintaining every physical server yourself, you can provision resources from a cloud provider when needed.

### Traditional infrastructure vs. cloud

| Traditional / on-premises approach | Cloud approach |
|---|---|
| Buy and install physical servers | Provision virtual servers or managed services on demand |
| Plan capacity in advance | Scale resources as demand changes |
| Organization maintains physical hardware | Provider maintains the underlying cloud infrastructure |
| Hardware purchase can require significant upfront cost | Many services use pay-as-you-go pricing |
| Expanding capacity can take days or weeks | New resources can often be provisioned in minutes |
| Disaster recovery requires additional planning and infrastructure | Cloud services offer options for backups, multiple Availability Zones, and disaster recovery |

**Important:** Cloud is not automatically cheaper or more secure. Good architecture, security controls, monitoring, and cost management are still required.

### Why organizations use cloud computing

- **Agility:** Provision resources quickly.
- **Elasticity:** Scale capacity up or down as demand changes.
- **Availability:** Design systems across multiple failure domains.
- **Global reach:** Deploy workloads near users.
- **Managed services:** Reduce some infrastructure maintenance.
- **Measured usage:** Monitor and manage resource consumption.
- **Automation:** Provision and update infrastructure using code and APIs.

---

## 2. Types of Cloud Deployment

### 2.1 Public cloud

Infrastructure is provided by a cloud provider and delivered to customers over a network. The underlying infrastructure is shared by multiple customers with logical isolation.

**Examples:** Amazon Web Services (AWS), Microsoft Azure, and Google Cloud.

**Use cases:** Websites, development environments, application hosting, analytics, and scalable workloads.

### 2.2 Private cloud

Cloud infrastructure is dedicated to one organization. It may be operated in the organization's own data centre or hosted by a provider.

**Examples of technologies used to build private-cloud environments:** OpenStack and VMware-based platforms.

**Use cases:** Organizations that need specific control, customization, or regulatory arrangements.

### 2.3 Hybrid cloud

A hybrid-cloud environment integrates private infrastructure with public-cloud services so workloads or data can be used across both environments.

**Example:** An organization keeps a legacy application on-premises and uses AWS for backup, analytics, or additional application capacity.

**Use cases:** Gradual cloud migration, disaster recovery, and workloads with specific data-location requirements.

### Quick comparison

| Type | Main idea | Example scenario |
|---|---|---|
| Public cloud | Provider-operated cloud services | Host a website on AWS |
| Private cloud | Cloud environment dedicated to one organization | Internal enterprise cloud |
| Hybrid cloud | Integrated private and public environments | On-premises application connected to AWS |

---

## 3. Cloud Service Models: IaaS, PaaS, and SaaS

These models describe how much of the technology stack the provider manages versus how much the customer manages. Exact responsibility varies by service.

### 3.1 IaaS — Infrastructure as a Service

The provider manages the underlying physical infrastructure and virtualization. The customer generally manages the operating system, installed software, applications, and much of the guest-level configuration.

**AWS examples:**
- **Amazon EC2:** Virtual servers.
- **Amazon EBS:** Block storage for EC2.
- **Amazon VPC:** Virtual networking.
- **Elastic Load Balancing:** Managed load-balancing service used with customer-designed architectures.

**Example:** Launch an EC2 Linux instance, install Nginx, deploy an application, configure the OS, and maintain the application.

### 3.2 PaaS — Platform as a Service

The provider manages more of the platform so the customer can focus on application code and configuration. The exact amount of control depends on the service.

**AWS examples:**
- **AWS Elastic Beanstalk:** Helps deploy and manage applications on supported platforms.
- **AWS App Runner:** Builds and runs web applications and APIs without requiring the user to manage the underlying server infrastructure directly.
- **Amazon RDS:** Managed relational database service; commonly described as a managed database platform.

**Example:** Deploy an application through Elastic Beanstalk rather than manually provisioning and maintaining each web server.

### 3.3 SaaS — Software as a Service

The provider operates a complete application that users access, usually through a web browser or client.

**Examples:** Salesforce, Microsoft 365, and Google Workspace. AWS also offers managed application services, such as Amazon QuickSight for business intelligence.

**Example:** A user creates reports in a hosted business-intelligence application without managing the servers that run it.

### Service model comparison

| Model | Customer typically manages | Provider typically manages |
|---|---|---|
| IaaS | OS, applications, guest configuration, data, access controls | Physical hardware, facilities, virtualization |
| PaaS | Application code, data, app configuration, access controls | Underlying infrastructure and much of the platform |
| SaaS | Usage, user settings, data and access configuration as applicable | Application, platform, and infrastructure |

### Is AWS completely PaaS?

**No. AWS is a cloud provider that offers services across multiple service models.** EC2 is commonly used as an IaaS example; Elastic Beanstalk and App Runner provide more platform-managed experiences; SaaS is a way of consuming a complete application. A provider itself should not be labelled as only IaaS, PaaS, or SaaS.

**Memory tip:**
- IaaS: You manage more of the server.
- PaaS: You focus more on the application.
- SaaS: You use the finished software.

---

## 4. Overview of AWS Services

AWS provides services across compute, storage, networking, databases, security, management, and application delivery.

| Category | Important services | Purpose |
|---|---|---|
| Compute | Amazon EC2, AWS Lambda, AWS Elastic Beanstalk | Run applications and code |
| Storage | Amazon S3, Amazon EBS, Amazon EFS, S3 Glacier storage classes | Object, block, file, and archival storage |
| Networking and content delivery | Amazon VPC, Route 53, Elastic Load Balancing, Amazon CloudFront | Network isolation, DNS, traffic distribution, content delivery |
| Databases | Amazon RDS, Amazon DynamoDB | Relational and NoSQL databases |
| Identity and security | AWS IAM, AWS KMS, AWS Certificate Manager | Access management, key management, TLS certificates |
| Monitoring and auditing | Amazon CloudWatch, AWS CloudTrail | Metrics, logs, alarms, and API activity history |
| Configuration and governance | AWS Config, AWS Trusted Advisor | Resource configuration tracking and recommendations |
| Infrastructure as code | AWS CloudFormation | Provision and manage infrastructure from templates |
| Migration | AWS migration services and tools | Plan and perform workload migration |

### How these services fit into a simple application

1. **Route 53** can resolve a domain name.
2. **CloudFront** can cache and deliver supported content closer to users.
3. **Elastic Load Balancing** can distribute requests across application targets.
4. **EC2 instances** can run the application.
5. **Auto Scaling** can adjust the number of EC2 instances based on configured conditions.
6. **RDS** can store relational application data.
7. **S3** can store images, documents, and static assets.
8. **IAM** controls which identities and workloads can access AWS resources.
9. **CloudWatch** monitors metrics, logs, and alarms.
10. **CloudTrail** records supported account activity and API events.

This is an example architecture, not a requirement that every application use every service.

---

## 5. AWS Regions and Availability Zones

### 5.1 Region

An AWS Region is a separate geographic area where AWS operates infrastructure.

Regions are independent of one another in many important ways. Service availability, pricing, and compliance requirements can vary by Region.

**Example:** A workload may be deployed in the Asia Pacific (Mumbai) Region when that Region fits its latency, availability, and compliance needs.

### 5.2 Availability Zone (AZ)

An Availability Zone is an isolated location within an AWS Region. A Region contains multiple Availability Zones, which are designed to have independent infrastructure and failure boundaries while remaining connected through high-bandwidth, low-latency networking.

A workload can be deployed across multiple AZs to improve resilience against an AZ-level failure.

### Region vs. Availability Zone

| Region | Availability Zone |
|---|---|
| Geographic area | Isolated location within a Region |
| Contains multiple AZs | Belongs to one Region |
| Used to choose geographic placement | Used to design resilience within a Region |
| Can help meet latency or data-location needs | Can help reduce the impact of an AZ failure |

**Interview example:** For a highly available web application, deploy application instances in at least two Availability Zones behind a load balancer, when supported by the application design.

**Remember:** An EC2 instance is launched into a subnet, and the subnet belongs to one Availability Zone.

---

## 6. AWS Account, Root User, and Account Alias

### 6.1 AWS account

An AWS account is a resource and security boundary used to access and manage AWS services, permissions, and billing.

### 6.2 Root user

The root user is the identity associated with the email address used to create the AWS account. It has extensive privileges and should be protected carefully.

**Best practices:**
- Enable multi-factor authentication (MFA) for the root user.
- Use the root user only for tasks that specifically require it.
- Do not create or use root access keys for routine work.
- Create named IAM identities or use IAM Identity Center for everyday access, according to your organization's setup.
- Never share root credentials or store them in GitHub.

### 6.3 IAM users, groups, roles, and policies

- **IAM user:** An identity for a person or workload in an IAM-based setup.
- **IAM group:** A way to manage permissions for a collection of IAM users.
- **IAM role:** An identity with permissions that can be assumed by an authorized principal or service.
- **IAM policy:** A document defining allowed or denied actions on resources.

Prefer temporary credentials and roles for workloads when possible.

### 6.4 Account alias

An account alias is an optional, human-readable name for an AWS account's sign-in URL. It does **not** replace the account ID and does not grant permissions.

### Practical: review account and identity settings

1. Sign in to the AWS Management Console using your authorized account access.
2. Open the account menu and review account information.
3. Open IAM and review the identity and access configuration permitted for your account.
4. Review the account alias if configured.
5. Confirm that root MFA is enabled without sharing screenshots that expose sensitive account information.

**Screenshot placeholder:** `screenshots/aws-account-identity-overview.png`

---

## 7. AWS Billing and Cost Management

Cloud resources can create charges even when you are not actively using them. Check costs regularly, especially while learning.

### 7.1 Review your bill

1. Open the AWS Management Console.
2. Open **Billing and Cost Management**.
3. Review the available billing summary and current charges.
4. Open the cost breakdown to understand which services and Regions contribute to usage.
5. Review invoices and payment status when applicable.
6. Investigate unfamiliar usage promptly.

Billing data may be delayed and is not always a real-time record of all usage.

**Screenshot placeholder:** `screenshots/aws-billing-overview.png`

### 7.2 AWS Budgets

AWS Budgets lets you set cost, usage, or other supported budget thresholds and configure notifications.

Example learning budget:
- Monthly cost threshold: choose an amount you can afford.
- Notification thresholds: for example, 50%, 80%, and 100% of the planned amount.
- Notification destination: configure an appropriate email or supported notification mechanism.

**Important:** A budget alert does not automatically stop or delete resources. Automated actions, where available, must be configured separately and carefully.

**Screenshot placeholder:** `screenshots/aws-budget-alerts.png`

### 7.3 AWS Pricing Calculator

The AWS Pricing Calculator helps estimate the cost of a proposed architecture before deployment.

Typical workflow:
1. Open the AWS Pricing Calculator.
2. Create an estimate.
3. Add the services you plan to use, such as EC2, EBS, and data transfer.
4. Select the intended Region and service configuration.
5. Enter expected usage, storage, and traffic assumptions.
6. Review the estimate and save or share it as appropriate.

The estimate depends on the inputs and pricing assumptions. It is not a guarantee of the final bill.

**Screenshot placeholder:** `screenshots/aws-pricing-calculator-estimate.png`

### Cost-control checklist for labs

- [ ] Choose an appropriate Region and instance size.
- [ ] Check current service pricing and any applicable free-tier terms.
- [ ] Create a budget and configure notifications.
- [ ] Stop EC2 instances when not needed, understanding that some attached resources may still incur charges.
- [ ] Check EBS volumes, snapshots, Elastic IP addresses, public IPv4 addresses, load balancers, NAT Gateways, and other billable resources.
- [ ] Delete only resources that are no longer required and whose data you no longer need.
- [ ] Review Billing and Cost Management after practical sessions.

---

## 8. Hands-on Revision Checklist

Use this checklist during revision. Mark an item complete only after you have performed it yourself.

### Fundamentals
- [ ] Explain cloud computing in your own words.
- [ ] Compare traditional infrastructure with cloud computing.
- [ ] Explain public, private, and hybrid cloud.
- [ ] Explain IaaS, PaaS, and SaaS with examples.
- [ ] Name common AWS services and their purposes.
- [ ] Explain Region vs. Availability Zone.

### Account and cost
- [ ] Identify the difference between the root user and IAM identities.
- [ ] Explain the purpose of an account alias.
- [ ] Review Billing and Cost Management.
- [ ] Create or review an AWS Budget and its alerts.
- [ ] Build a Pricing Calculator estimate.

### Evidence
- [ ] Add screenshots that you captured yourself.
- [ ] Remove or redact email addresses, account IDs, balances, and other sensitive information before publishing.
- [ ] Never publish passwords, access keys, secret keys, session tokens, or private keys.
- [ ] Record questions and errors encountered during the practical.

---

## 9. Learning Log

**Date:** 21 October 2026  
**Topic:** AWS Cloud Introduction and Service Overview  
**Learning source:** Sudharsan's recorded class and revision notes

### What I revised
- Cloud computing and the difference between traditional infrastructure and cloud.
- Public, private, and hybrid cloud deployment models.
- IaaS, PaaS, and SaaS service models.
- AWS service categories and common use cases.
- AWS Regions and Availability Zones.
- Root user, account alias, IAM identities, and access security.
- Billing review, AWS Budgets, and AWS Pricing Calculator.

### What I learned
AWS provides compute, storage, networking, database, security, monitoring, and migration services. Regions define geographic placement, while Availability Zones support resilience within a Region. AWS offers services that fit different cloud service models; AWS as a provider is not exclusively PaaS.

I also learned that cloud cost management is part of responsible cloud usage. Budgets and notifications help identify potential overspending, while reviewing resources and billing details helps find usage that may continue to incur charges.

### Practical status
- [ ] AWS account and identity settings reviewed
- [ ] Billing dashboard reviewed
- [ ] Budget and notifications reviewed or configured
- [ ] Pricing Calculator estimate created
- [ ] Screenshots captured and sanitized
- [ ] Interview questions revised

**Next revision:** EC2 compute service — AMIs, instance types, launch configuration, security groups, key pairs, storage, connectivity, lifecycle, and cost control.

---

## 10. Interview Questions and Answers

### Q1. What is cloud computing?
Cloud computing is the on-demand delivery of computing resources over a network, often with usage-based pricing.

### Q2. What are the three common cloud deployment types covered here?
Public cloud, private cloud, and hybrid cloud.

### Q3. What is the difference between IaaS, PaaS, and SaaS?
IaaS provides infrastructure, PaaS provides a more managed application platform, and SaaS provides a complete application for users.

### Q4. Is AWS entirely a PaaS provider?
No. AWS offers a broad range of services across infrastructure, managed platforms, and application services. EC2 is commonly used as an IaaS example.

### Q5. What is an AWS Region?
A geographic area where AWS operates infrastructure and offers services.

### Q6. What is an Availability Zone?
An isolated location within an AWS Region, designed to support fault isolation and resilient architectures.

### Q7. How can a web application improve availability within a Region?
Where appropriate, deploy application capacity across multiple Availability Zones and use a load balancer and suitable health checks.

### Q8. What is the AWS root user?
The original account identity associated with the email address used to create the AWS account. It has extensive privileges and should be protected with MFA.

### Q9. What is an AWS account alias?
An optional human-readable name used in the account's sign-in URL. It does not replace the account ID or change permissions.

### Q10. What is IAM?
AWS Identity and Access Management controls authentication and authorization for access to AWS resources.

### Q11. What is the purpose of AWS Budgets?
To define cost or usage thresholds and notify appropriate recipients when configured thresholds are reached or forecast to be reached.

### Q12. Does creating a budget automatically stop spending?
No. Budget notifications help alert you. Stopping or deleting resources requires separate action or carefully configured automation.

### Q13. What is the AWS Pricing Calculator?
A tool used to estimate expected AWS service costs based on a proposed architecture and usage assumptions.

### Q14. Why might the final AWS bill differ from an estimate?
Actual usage, data transfer, storage, service configuration, Region, pricing changes, and other billable resources can differ from the assumptions.

### Q15. How are these fundamentals connected to DevOps?
DevOps engineers use AWS compute, networking, storage, IAM, automation, monitoring, and cost controls to build, deploy, operate, secure, and troubleshoot applications.

---

## References

- AWS overview: https://aws.amazon.com/what-is-aws/
- AWS global infrastructure: https://aws.amazon.com/about-aws/global-infrastructure/
- AWS IAM: https://aws.amazon.com/iam/
- AWS Billing and Cost Management: https://aws.amazon.com/aws-cost-management/
- AWS Pricing Calculator: https://calculator.aws/
- AWS Budgets: https://aws.amazon.com/aws-cost-management/aws-budgets/
