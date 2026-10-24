# AWS EC2 Interview Questions and Answers
## Cloud Support Engineer | Junior DevOps Engineer | Intermediate Level

**Learning source:** AWS EC2 topics covered in Sudharsan's AWS classes  
**Preparation goal:** Understand concepts, answer interview questions, and troubleshoot practical scenarios.

---

# LEVEL 1: EC2 Fundamentals

## 1. What is Amazon EC2?

**Answer:**

Amazon EC2 stands for Amazon Elastic Compute Cloud. It is an AWS service that provides virtual servers called instances.

We can use EC2 to:
- Host websites and applications.
- Run backend services.
- Deploy applications using Jenkins.
- Run development and testing environments.
- Host databases where appropriate.
- Execute scripts and scheduled jobs.

**Interview example:**

"If my company wants to host a website without purchasing a physical server, I can launch an EC2 instance, install a web server such as Apache, configure networking and security groups, and access the website through its public IP or domain."

## 2. What is an EC2 instance?

An EC2 instance is a virtual server running in AWS.

An instance has resources such as:
- CPU.
- Memory.
- Operating system.
- Network interfaces.
- Storage.
- Security and access configuration.

Examples of instance types include `t3.micro`, `t3.small`, and `m7i.large`.

The instance type determines the compute resources available to the server.

## 3. What is an AMI?

AMI stands for Amazon Machine Image.

It is a template used to launch EC2 instances.

An AMI typically includes:
- An operating system image.
- Software and configuration stored on the captured volumes.
- Information about the storage volumes used when launching the instance.

**Example:**

Suppose I install Apache and configure a website on an EC2 instance. I can create an AMI from that instance and launch another instance with the captured software and files.

The new instance is separate from the original instance.

## 4. What is the difference between an AMI and an EBS snapshot?

| AMI | EBS Snapshot |
|---|---|
| Used to launch EC2 instances | Used to preserve an EBS volume's data |
| Contains machine-image configuration and references to backing snapshots | Contains point-in-time volume data |
| Can be used to launch a new EC2 instance | Can be used to create an EBS volume |
| Useful for server replication and migration | Useful for volume backup and recovery |

**Remember:**

AMI = Machine image for launching an instance.

Snapshot = Point-in-time backup of an EBS volume.

## 5. What are the steps to launch an EC2 instance?

**Answer:**

1. Open the AWS Management Console.
2. Navigate to EC2.
3. Choose Launch instance.
4. Enter an instance name.
5. Select an AMI.
6. Select an instance type.
7. Configure a key pair or another supported access method.
8. Configure the VPC and subnet.
9. Configure the security group.
10. Configure storage.
11. Attach an IAM role if required.
12. Review and launch the instance.
13. Wait until the instance is running and its status checks pass.
14. Connect using SSH, RDP, or an appropriate Systems Manager capability.

**Interview follow-up:** Does the `Running` state guarantee that the application is working?

No. The instance may be running while the operating system, web server, application, or network configuration has a problem. I would also check the status checks, service status, application logs, and connectivity.

## 6. What is the difference between an instance state and a status check?

**Answer:**

An instance state describes whether an instance is pending, running, stopping, stopped, or terminated.

EC2 status checks help identify underlying infrastructure or instance-level problems.

The main checks include:

- **System status check:** Detects problems affecting the underlying AWS infrastructure.
- **Instance status check:** Detects problems involving the instance itself, such as operating system or network configuration issues.
- **Attached EBS status check:** Helps identify certain problems affecting attached EBS volumes.

If an instance is running but an instance status check fails, I would investigate operating system boot problems, network configuration, resource exhaustion, or other instance-level issues.

## 7. What are EC2 instance types?

EC2 instance types provide different combinations of CPU, memory, storage, and network capabilities.

| Category | Example families | Typical use |
|---|---|---|
| General purpose | T, M | Websites, development, general applications |
| Compute optimized | C | CPU-intensive workloads |
| Memory optimized | R, X | Memory-intensive applications |
| Storage optimized | I, D | High-throughput or high-I/O storage workloads |
| Accelerated computing | G, P | GPU-based workloads and specialized computing |

**Interview example:**

For a small test website, I might begin with an appropriate small general-purpose instance. For a CPU-intensive application, I would evaluate a compute-optimized family.

The final choice depends on workload requirements, cost, supported architecture, and availability.

## 8. What is the difference between stopping and terminating an EC2 instance?

| Stop | Terminate |
|---|---|
| Shuts down the instance | Permanently deletes the instance resource |
| Can generally be started again | The terminated instance cannot be restarted |
| EBS volumes are normally retained according to their deletion settings | EBS volumes configured for deletion on termination may be deleted |
| Instance storage data is lost | Instance storage data is lost |

**Important:** Stopping an instance does not automatically delete its EBS volumes. However, attached instance-store data does not survive a stop/start cycle.

## 9. What happens to the public IP when an EC2 instance is stopped and started?

The automatically assigned public IPv4 address is generally released when an instance is stopped and is not guaranteed to remain the same after it starts again.

An Elastic IP address can provide a persistent public IPv4 address while it remains allocated to your account and is managed appropriately.

The private IP address of the primary network interface normally remains associated with the instance across stop/start operations.

## 10. What is User Data in EC2?

EC2 User Data allows us to provide startup instructions when launching an instance.

For Linux instances, User Data commonly contains shell commands.

Example for Amazon Linux 2023:

```bash
#!/bin/bash
dnf install -y httpd
systemctl enable --now httpd
echo "Welcome to my AWS website" > /var/www/html/index.html
```

This script installs Apache, enables and starts the service, and creates a simple website.

**Interview follow-up:** Does User Data always run every time the instance boots?

For common Linux cloud-init configurations, shell-script User Data normally executes on the initial instance boot, not every subsequent boot. Repeated execution requires an appropriate configuration.

## 11. What is the difference between User Data and Launch Templates?

**Answer:**

User Data provides startup commands or configuration.

A Launch Template stores reusable instance launch settings, including the AMI, instance type, networking configuration, storage, IAM instance profile, and optionally User Data.

A Launch Template can therefore use User Data as one part of an automated deployment.

## 12. What is the difference between a public IP and a private IP?

**Answer:**

A private IP is used for communication within a private network, such as a VPC.

A public IP can enable communication between an instance and external networks when the required routing and security configuration is in place.

For example:
- An administrator might connect to a public EC2 instance over SSH.
- An application server might communicate with a database using its private IP.
- A private EC2 instance might access AWS services through VPC endpoints without needing a public IP.

Having a public IP does not automatically make every port accessible.

---

# LEVEL 2: EC2 Networking and Security

## 13. What is a Security Group?

A Security Group is a virtual firewall associated with an EC2 network interface.

It controls allowed inbound and outbound traffic.

Example for a public Linux web server:

| Protocol | Port | Source | Purpose |
|---|---:|---|---|
| SSH | 22 | Your trusted IP | Linux administration |
| HTTP | 80 | `0.0.0.0/0` if a public website is intended | Web traffic |
| HTTPS | 443 | `0.0.0.0/0` if a public website is intended | Encrypted web traffic |

Avoid opening SSH to everyone unless there is a justified and carefully controlled requirement.

## 14. Are Security Groups stateful or stateless?

**Answer:**

Security Groups are stateful.

If inbound traffic is allowed and a connection is established, response traffic for that connection is automatically permitted by the stateful firewall.

You do not need to create a separate outbound rule solely to allow the response to an allowed inbound connection.

Network ACLs, in contrast, are stateless and require rules for both directions of traffic.

## 15. What is the difference between a Security Group and a Network ACL?

| Security Group | Network ACL |
|---|---|
| Associated with network interfaces | Associated with subnets |
| Stateful | Stateless |
| Supports allow rules | Supports allow and deny rules |
| Rules are evaluated collectively | Rules are evaluated in rule-number order |
| Commonly used to control instance-level access | Can provide an additional subnet-level network control |

Both can affect whether traffic reaches an instance.

## 16. My EC2 instance is running, but I cannot connect through SSH. What will you check?

**Answer:**

I would troubleshoot in this order:

**Step 1: Verify the instance**
- Confirm the instance is running.
- Check system and instance status checks.
- Verify that I am connecting to the correct instance.

**Step 2: Verify the address**
- Confirm the public IP or DNS name is correct.
- Check whether the instance has a public IPv4 address if I am connecting over the public internet.

**Step 3: Check the Security Group**
- Confirm inbound TCP port 22 is allowed from my current trusted IP address.

**Step 4: Check networking**
- Verify the subnet route table.
- For public IPv4 connectivity, confirm the appropriate internet gateway and routing configuration.
- Check applicable network ACL rules.

**Step 5: Check authentication**
- Use the correct private key.
- Use the correct username, such as `ec2-user` for Amazon Linux or `ubuntu` for Ubuntu.
- Check the private key file permissions where applicable.

**Step 6: Check the server**
- Verify that the SSH service is running.
- Investigate operating system or network problems if the instance is unhealthy.

If SSH remains unavailable, I would use an already configured alternative management method, such as Systems Manager Session Manager, if possible.

## 17. My website is not opening in a browser. How will you troubleshoot it?

**Answer:**

I would separate the problem into application, server, and network layers.

**Step 1: Check the web server**

```bash
sudo systemctl status httpd
```

**Step 2: Check whether the service is listening**

```bash
sudo ss -lntp
```

**Step 3: Test locally**

```bash
curl -I http://localhost
```

**Step 4: Check website files**

```bash
ls -la /var/www/html/
```

**Step 5: Check the Security Group**

Confirm that inbound TCP port 80 is allowed if the site uses HTTP.

**Step 6: Check networking**

Verify public IP assignment, route tables, internet gateway routing, network ACLs, and any relevant DNS configuration.

**Step 7: Check logs**

```bash
sudo journalctl -u httpd --no-pager -n 50
```

On Amazon Linux, Apache error logs are commonly available at:

```bash
sudo tail -n 50 /var/log/httpd/error_log
```

For Ubuntu, Apache is commonly managed by `apache2`, and its log locations may differ.

## 18. What is SSH?

SSH stands for Secure Shell.

It provides encrypted remote access to systems.

For example:

```bash
ssh -i my-key.pem ec2-user@PUBLIC_IP
```

Here:
- `-i` specifies the private key file.
- `ec2-user` is the remote username.
- `PUBLIC_IP` is the target address.

SSH commonly uses TCP port 22.

## 19. What is RDP?

RDP stands for Remote Desktop Protocol.

It is commonly used to connect to Windows servers through a graphical desktop.

The default RDP port is TCP 3389.

For Windows EC2, I would restrict inbound RDP access to a trusted IP address or use a more controlled remote management solution.

## 20. What is an Elastic IP?

An Elastic IP is a static public IPv4 address allocated to an AWS account.

It can be associated with supported resources, including EC2 instances.

It is useful when an application needs a stable public IPv4 address.

Elastic IPs can incur charges depending on the applicable AWS pricing and usage circumstances, so they should be managed carefully.

---

# LEVEL 3: AMIs, Launch Templates, and Migration

## 21. My AMI does not preserve the Apache installation. Why?

**Answer:**

An AMI normally preserves installed software and files stored on volumes included in the captured image.

If Apache is missing, I would investigate:

1. Whether Apache was installed before the AMI was created.
2. Whether I launched an instance from the correct AMI.
3. Whether the installation was stored on the captured root volume or on a separate volume that was not included.
4. Whether startup scripts or automation changed the system after launch.
5. Whether the new instance is running the expected operating system and package configuration.

For Amazon Linux, I would check:

```bash
rpm -q httpd
```

Then:

```bash
sudo systemctl status httpd
```

If Apache is installed but stopped:

```bash
sudo systemctl enable --now httpd
```

## 22. Why is a Linux user missing after launching an instance from an AMI?

**Answer:**

If a Linux user was created before the AMI was captured, the account should normally be preserved when its account information is stored on the captured root filesystem.

I would check whether:
- The user was created before the AMI was created.
- I launched the correct AMI.
- The user information was stored on an included volume.
- A startup script, configuration management tool, or identity-management service changed the account.

I would verify the user with:

```bash
id devuser
```

And:

```bash
getent passwd devuser
```

## 23. How do you move an EC2 instance from Mumbai to Singapore?

**Answer:**

I would use an AMI-based migration.

1. Create an AMI from the source instance in Mumbai.
2. Wait until the AMI is available.
3. Copy the AMI to Singapore.
4. Open EC2 in the Singapore Region.
5. Locate the copied AMI.
6. Launch a new instance.
7. Configure the destination VPC, subnet, security groups, IAM role, storage, and access method.
8. Test the application.
9. Update DNS or application routing when ready.

I would also plan for data migration, encryption keys, application dependencies, and downtime.

**Important:** Copying an AMI does not automatically copy every related resource or guarantee application consistency for a live database.

## 24. What is an EC2 Launch Template?

**Answer:**

A Launch Template stores reusable configuration for creating EC2 instances.

It can include:
- AMI ID.
- Instance type.
- Key pair.
- Security groups.
- Network configuration.
- Storage.
- IAM instance profile.
- User Data.
- Tags.

It helps maintain consistency and is commonly used with Auto Scaling groups.

## 25. What is the difference between a Launch Template and a Launch Configuration?

**Answer:**

Launch Templates support versioning and a broader range of launch settings.

Launch Configurations are the older mechanism and have fewer capabilities. For new deployments, Launch Templates are generally preferred.

## 26. What happens if I change the Launch Template after launching an EC2 instance?

**Answer:**

Changing a Launch Template does not automatically change the configuration of existing instances.

The updated template can be used when launching new instances or when an Auto Scaling group is configured to use a new version.

Existing instances require an appropriate update or replacement process if their configuration must change.

## 27. What is the difference between an AMI and a Launch Template when creating multiple EC2 instances?

**Answer:**

The AMI provides the machine image.

The Launch Template specifies how to launch the instance using that image and other settings.

For example, I can use the same AMI with two Launch Templates that specify different instance types, security groups, or startup configurations.

---

# LEVEL 4: Linux Administration on EC2

## 28. How do you create a Linux user?

**Answer:**

On a Linux instance:

```bash
sudo useradd -m devuser
```

Set the password if needed:

```bash
sudo passwd devuser
```

Check the account:

```bash
id devuser
```

Creating the account does not automatically grant administrative permissions or guarantee SSH access.

## 29. What is the difference between root and a normal Linux user?

**Answer:**

Root is the superuser with extensive administrative privileges.

A normal user has permissions based on the account, group membership, and file permissions.

Administrators should avoid using root directly for routine work and should grant elevated permissions only when required.

## 30. What is sudo?

**Answer:**

`sudo` allows authorized users to execute commands with elevated privileges.

For example:

```bash
sudo dnf install -y httpd
```

The user must have the appropriate sudo permissions.

## 31. What is the sudoers file?

**Answer:**

The sudoers configuration controls which users and groups can execute commands with elevated privileges.

The main file is:

```text
/etc/sudoers
```

It can be edited safely using:

```bash
sudo visudo
```

`visudo` validates the configuration syntax before saving.

## 32. Can we log in to Linux EC2 without a key pair?

**Answer:**

Yes, through supported alternative access methods such as AWS Systems Manager Session Manager, or by configuring another valid authentication mechanism.

SSH password authentication is possible when properly configured, but it increases security risk if exposed to the internet.

For Cloud Support and DevOps environments, I would prefer Session Manager or appropriately managed SSH key authentication where practical.

## 33. What changes are required to enable SSH password authentication?

**Answer:**

The SSH server configuration may include:

```text
PasswordAuthentication yes
```

The configuration file is commonly:

```text
/etc/ssh/sshd_config
```

Before reloading the SSH service, I would validate the configuration:

```bash
sudo sshd -t
```

The effective configuration can be checked with:

```bash
sudo sshd -T | grep -i passwordauthentication
```

The SSH service name varies by distribution. On systems using `sshd`:

```bash
sudo systemctl reload sshd
```

Password authentication should only be enabled when there is a justified requirement and appropriate access controls.

## 34. Why is direct root SSH login commonly restricted?

**Answer:**

Root has unrestricted administrative privileges. If an attacker compromises root access, they can potentially take complete control of the server.

A common SSH setting is:

```text
PermitRootLogin no
```

Administrators should use authorized accounts and sudo rather than enabling direct root login without a strong justification.

## 35. What is LDAP?

**Answer:**

LDAP stands for Lightweight Directory Access Protocol.

It is used to access directory information and can be part of a centralized identity-management solution.

Organizations can integrate Linux systems with directory services to manage user accounts and authentication across multiple servers.

LDAP is not automatically configured on every EC2 instance.

## 36. How do you check disk usage on Linux EC2?

**Answer:**

Use:

```bash
df -h
```

To identify block devices:

```bash
lsblk
```

To find large directories, use:

```bash
sudo du -xhd1 /var
```

This helps identify whether a full filesystem or large directory is contributing to an application problem.

## 37. How do you check memory usage?

**Answer:**

Use:

```bash
free -m
```

For process-level usage:

```bash
top
```

If the instance is experiencing memory pressure, I would identify resource-intensive processes and inspect system logs before deciding whether to optimize the application or resize the instance.

## 38. Apache is installed but not starting. What will you do?

**Answer:**

First, check the service status:

```bash
sudo systemctl status httpd
```

Check recent service logs:

```bash
sudo journalctl -u httpd --no-pager -n 100
```

Validate Apache configuration:

```bash
sudo httpd -t
```

Check whether another process is using the required port:

```bash
sudo ss -lntp
```

Then investigate configuration errors, missing files, permissions, port conflicts, and resource problems.

On Ubuntu, the service and validation commands commonly use `apache2` instead of `httpd`.

---

# LEVEL 5: AWS Systems Manager (SSM)

## 39. What is AWS Systems Manager?

**Answer:**

AWS Systems Manager is an AWS service that helps administrators manage, automate, and operate supported resources, including EC2 instances.

Relevant capabilities include:
- Session Manager.
- Run Command.
- Fleet Manager.
- Patch Manager.
- Automation.

The exact capabilities available depend on the feature, operating system, configuration, and permissions.

## 40. What is SSM Agent?

**Answer:**

SSM Agent is software that runs on a managed machine and communicates with Systems Manager.

It enables supported remote management operations.

Many AWS-provided AMIs already include the agent. If it is absent or outdated, I would follow the official installation procedure for the operating system.

## 41. Why does an EC2 instance need an IAM role for SSM?

**Answer:**

The EC2 instance needs AWS permissions to communicate with Systems Manager.

An instance role provides temporary AWS credentials to the instance without requiring hard-coded access keys.

A commonly used AWS-managed policy for standard EC2 Systems Manager management is:

```text
AmazonSSMManagedInstanceCore
```

The role must be attached to the EC2 instance through an instance profile.

## 42. I attached the IAM role, but my EC2 instance does not appear in Systems Manager. Why?

**Answer:**

I would check:

1. Whether the correct IAM role is attached.
2. Whether the role has the required permissions.
3. Whether SSM Agent is installed and running.
4. Whether the instance can resolve DNS.
5. Whether it can reach the required Systems Manager endpoints over HTTPS.
6. Whether the AWS Region is correct.
7. Whether the operating system and configuration are supported.

I would inspect the agent status:

```bash
sudo systemctl status amazon-ssm-agent
```

And review recent logs:

```bash
sudo journalctl -u amazon-ssm-agent -n 100
```

## 43. Does SSM require SSH port 22 to be open?

**Answer:**

No. Systems Manager capabilities such as Run Command and Session Manager can operate without inbound SSH access when the instance, IAM permissions, agent, and network connectivity are configured correctly.

This can reduce the need to expose management ports.

## 44. What is Fleet Manager?

**Answer:**

Fleet Manager is a Systems Manager capability that helps administrators view and manage registered managed nodes.

I can use the AWS Management Console to locate a managed node and inspect its available management information.

If an instance is missing, I would check the IAM role, agent, Region, and network connectivity.

## 45. What is Run Command?

**Answer:**

Run Command allows administrators to execute commands on managed nodes without requiring an interactive SSH connection.

For a Linux instance, an appropriate Systems Manager document is:

```text
AWS-RunShellScript
```

It can be used for supported operations such as inspecting disk usage, checking services, installing packages, or running maintenance scripts.

## 46. How would you update software on 20 EC2 instances using SSM?

**Answer:**

I would:

1. Verify that all target instances are registered managed nodes.
2. Confirm the appropriate IAM permissions.
3. Group the instances by environment or application.
4. Test the update on a development or staging instance.
5. Use Run Command or an appropriate patch-management workflow.
6. Select the intended instances using controlled targeting.
7. Execute the approved update commands.
8. Monitor execution status and output.
9. Verify application health.
10. Investigate failures and roll back or recover where necessary.

For production systems, I would use staged deployment, maintenance windows where appropriate, and a recovery plan.

---

# LEVEL 6: Intermediate Scenario-Based Questions

## 47. An EC2 instance is running, but CPU utilization is consistently at 95%. What will you do?

**Answer:**

I would investigate before resizing the instance.

1. Check CPU utilization in Amazon CloudWatch.
2. Review the time range and determine whether the spike is temporary or sustained.
3. Connect using an authorized management method.
4. Inspect processes using `top` or another process-monitoring tool.
5. Check application logs for unusual activity.
6. Determine whether the load is caused by traffic, a background job, or a faulty process.
7. Optimize the application or workload where possible.
8. Consider scaling or changing the instance type if the workload requires more compute capacity.

I would also review memory, disk, network activity, and application response times.

## 48. An EC2 instance is running, but the application is down. What will you do?

**Answer:**

I would distinguish infrastructure health from application health.

First, check EC2 status checks.

Then inspect:
- Application service status.
- Web server status.
- Application logs.
- CPU, memory, and disk usage.
- Listening ports.
- Security groups and network routing.
- Database or external-service connectivity.

For example:

```bash
sudo systemctl status httpd
```

```bash
sudo ss -lntp
```

```bash
df -h
```

The objective is to identify the failing layer rather than restarting services blindly.

## 49. The root EBS volume is almost full. What will you do?

**Answer:**

I would first identify what is consuming disk space.

```bash
df -h
```

```bash
lsblk
```

```bash
sudo du -xhd1 /var
```

Then I would inspect application logs, temporary files, package caches, and other large directories.

If the volume needs more capacity, I would evaluate increasing the EBS volume size. Depending on the filesystem and operating system, I might also need to extend the partition and filesystem.

I would take appropriate precautions before deleting files or modifying the storage layout.

## 50. An EC2 instance is unreachable after a configuration change. How will you recover it?

**Answer:**

I would first determine whether the issue is related to networking, SSH configuration, the operating system, or the underlying infrastructure.

I would check:
- EC2 status checks.
- Security groups and network ACLs.
- Route tables and IP addresses.
- SSH service and authentication configuration.
- Recent changes and logs.

If SSH is unavailable, I would use an already configured recovery method such as Session Manager, where available.

Depending on the situation, I might use supported recovery procedures involving the root EBS volume, a helper instance, or AWS recovery capabilities.

I would avoid terminating the instance until I had evaluated data preservation and recovery requirements.

## 51. You need to deploy the same website to development and production EC2 instances. What approach would you use?

**Answer:**

I would separate application code from environment-specific configuration.

A possible approach is:

1. Store the application code in GitHub.
2. Create a repeatable deployment process.
3. Use separate development and production configurations.
4. Use an AMI or Launch Template to standardize the base infrastructure.
5. Use Jenkins or another deployment tool to automate the deployment.
6. Store secrets securely rather than in source control.
7. Test the deployment in development.
8. Promote the approved release to production.
9. Verify application health and monitor logs.

For a DevOps workflow, I would aim for repeatable deployments rather than manually configuring each server.

## 52. How would you reduce EC2 costs?

**Answer:**

I would:
- Identify idle or underutilized instances.
- Right-size instance types.
- Stop eligible non-production instances when not required.
- Review EBS volumes and snapshots.
- Remove unused Elastic IP allocations where appropriate.
- Consider Savings Plans or Reserved Instances for suitable predictable workloads.
- Consider Spot Instances for interruption-tolerant workloads.
- Monitor costs and set appropriate budgets and alerts.

I would not terminate or resize production resources without understanding their dependencies and availability requirements.

## 53. What is EC2 Auto Scaling?

**Answer:**

EC2 Auto Scaling helps automatically maintain and adjust the number of EC2 instances according to configured requirements and scaling policies.

It can:
- Replace unhealthy instances.
- Maintain a desired capacity.
- Increase capacity when demand rises.
- Reduce capacity when demand falls.

Launch Templates are commonly used to define the configuration of instances launched by an Auto Scaling group.

## 54. What is the difference between horizontal and vertical scaling?

**Answer:**

**Vertical scaling** means increasing or decreasing the resources of a single machine, such as changing to a larger EC2 instance type.

**Horizontal scaling** means adding or removing machines.

For example:
- Vertical: Change an instance to a larger type with more CPU and memory.
- Horizontal: Add more EC2 instances behind a load balancer.

Horizontal scaling can improve availability and capacity, but applications must be designed to handle multiple instances correctly.

## 55. How would you monitor an EC2 instance?

**Answer:**

I would use Amazon CloudWatch for infrastructure metrics and alarms.

Important metrics include:
- CPU utilization.
- Network activity.
- EBS performance metrics.
- Status-check failures.

Memory and filesystem utilization generally require a CloudWatch Agent or another monitoring mechanism because they are not available as standard EC2 host metrics in the same way as CPU utilization.

I would also review:
- Application logs.
- System logs.
- CloudWatch alarms.
- Systems Manager status.
- Application health checks.

The goal is to detect problems early and respond before they cause significant impact.

---

# LEVEL 7: Additional Interview Questions to Practise

Prepare short answers for these questions as well.

1. What is an EBS volume?
2. What is the difference between EBS and instance store?
3. What happens to EBS data when an EC2 instance is terminated?
4. What is an Elastic Network Interface?
5. What is a VPC?
6. What is a subnet?
7. What is an internet gateway?
8. What is a route table?
9. What is the difference between a public subnet and a private subnet?
10. What is the difference between IAM users and IAM roles?
11. What is the difference between an IAM role and a Security Group?
12. What is the difference between an EC2 key pair and an IAM role?
13. What is the difference between User Data and cloud-init?
14. What is the difference between On-Demand and Spot Instances?
15. What is the purpose of an Elastic IP?
16. What are EC2 system status checks?
17. How do you resize an EC2 instance?
18. What precautions should you take before resizing a production instance?
19. What happens when you change an instance type?
20. How would you troubleshoot an EC2 instance that fails to boot?
21. How would you recover a deleted website file?
22. How do you find which process is using port 80?
23. How do you check whether Apache is listening on port 80?
24. How do you find errors in Linux system logs?
25. How do you automate package installation during instance startup?
26. How do you avoid storing secrets in User Data?
27. How would you rotate SSH keys?
28. How do you give an EC2 instance permission to access an S3 bucket?
29. How would you deploy the same AMI in multiple Regions?
30. How would you verify that an AMI-based deployment is working correctly?

---

# LEVEL 8: Practical Tasks Before Your Interview

Do not just read these questions. Complete the practical tasks on your own test environment.

## Lab 1: Launch and Connect to EC2

- [ ] Launch an Amazon Linux instance.
- [ ] Select an appropriate instance type.
- [ ] Configure a key pair.
- [ ] Configure SSH access securely.
- [ ] Connect using SSH or MobaXterm.
- [ ] Check the operating system and system resources.

Commands to practise:

```bash
cat /etc/os-release
```

```bash
lscpu
```

```bash
free -m
```

```bash
df -h
```

## Lab 2: Host a Website

- [ ] Install Apache.
- [ ] Start and enable the service.
- [ ] Create an HTML page.
- [ ] Test the page locally.
- [ ] Configure the Security Group.
- [ ] Access the website using the instance's public IP.

Commands:

```bash
sudo dnf install -y httpd
```

```bash
sudo systemctl enable --now httpd
```

```bash
echo "My EC2 Website" | sudo tee /var/www/html/index.html
```

```bash
curl -I http://localhost
```

## Lab 3: Create an AMI and Launch a New Instance

- [ ] Install Apache on the original instance.
- [ ] Create a website file.
- [ ] Create a Linux user.
- [ ] Verify the configuration.
- [ ] Create an AMI.
- [ ] Wait until the AMI is available.
- [ ] Launch a new instance from the AMI.
- [ ] Verify Apache, website files, and the user.
- [ ] Explain which resources are not automatically copied.

## Lab 4: Launch Template

- [ ] Create a Launch Template.
- [ ] Select an AMI and instance type.
- [ ] Configure the security group.
- [ ] Add appropriate User Data.
- [ ] Launch an instance from the template.
- [ ] Verify the result.

## Lab 5: Systems Manager

- [ ] Check whether SSM Agent is installed.
- [ ] Attach an IAM role with the required permissions.
- [ ] Verify agent status.
- [ ] Check network connectivity.
- [ ] Confirm the instance is registered as a managed node.
- [ ] Run a test command using `AWS-RunShellScript`.
- [ ] Review execution output.
- [ ] Troubleshoot a failed command.

## Lab 6: Troubleshooting

Practise diagnosing these problems:

- [ ] SSH connection refused.
- [ ] SSH connection timed out.
- [ ] Apache is stopped.
- [ ] Website is inaccessible externally.
- [ ] Linux user cannot log in.
- [ ] EC2 instance is missing from Systems Manager.
- [ ] Disk space is full.
- [ ] CPU utilization is high.
- [ ] AMI does not contain expected changes.
- [ ] Launch Template starts an instance with unexpected configuration.

---

# How to Answer Interview Questions Confidently

Use this four-step structure for troubleshooting questions:

**1. Identify the problem**

Explain what is failing and what the expected behavior should be.

**2. Check the evidence**

Mention relevant AWS console details, status checks, Linux commands, logs, and metrics.

**3. Identify the root cause**

Explain how you would narrow down the problem rather than guessing.

**4. Fix and verify**

Describe the corrective action and how you would confirm the service works again.

### Example: SSH is not working

"I would first check the EC2 instance state and status checks. Then I would verify the target IP address, security group rules, subnet routing, and network ACLs. Next, I would check the private key and Linux username. If network access appears correct, I would investigate the SSH service and operating system. After applying the fix, I would test the connection again and document the root cause."

This style demonstrates practical troubleshooting ability rather than memorized definitions.

---

# Your Interview Preparation Priority

For your current Cloud Support and Junior DevOps preparation, master these topics in this order:

1. **EC2 fundamentals:** AMIs, instance types, launch process, start/stop/terminate.
2. **Networking:** IP addresses, ports, Security Groups, VPCs, subnets, routing, SSH, and HTTP/HTTPS.
3. **Linux administration:** Users, permissions, sudo, services, disk, memory, and logs.
4. **Web hosting:** Apache installation, service troubleshooting, website files, and connectivity.
5. **AMI and Launch Templates:** Creating images, copying across Regions, launching instances, and troubleshooting missing configuration.
6. **Systems Manager:** IAM roles, SSM Agent, managed nodes, Fleet Manager, and Run Command.
7. **Troubleshooting:** Diagnose real problems methodically.
8. **Intermediate operations:** Monitoring, backups, cost optimization, scaling, and deployment automation.

**Final advice:** You have already studied a useful set of EC2 topics. Your next step is to practise each workflow until you can perform it without following a tutorial. In an interview, being able to explain how you diagnosed and fixed a problem is often more valuable than simply knowing a service definition.