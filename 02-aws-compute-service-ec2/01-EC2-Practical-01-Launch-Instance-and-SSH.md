# AWS EC2 Compute Service — Instance Launch and SSH Connection

**Course:** Sudharsan — AWS Cloud / DevOps  
**Suggested path:** `AWS/03-EC2-Compute-Service/EC2-Practical-01-Launch-Instance-and-SSH.md`

> **Lab safety:** Check current AWS pricing and free-tier eligibility before launching. Never upload private keys, passwords, access keys, or sensitive account screenshots to GitHub.

## 1. Objective

Launch a Linux EC2 instance in AWS and connect from a Windows laptop using MobaXterm and SSH.

You will learn to:
- Open EC2 in the AWS Management Console.
- Configure a security group.
- Select an AMI, instance type, key pair, network, and storage.
- Launch and verify an EC2 instance.
- Connect using the public IP address and matching private key.
- Review basic EC2 lifecycle and cost considerations.

## 2. What is Amazon EC2?

Amazon Elastic Compute Cloud (EC2) provides resizable compute capacity in AWS. An EC2 instance is a virtual server used to run operating systems, applications, web servers, automation agents, and other workloads.

## 3. High-level launch workflow

1. Sign in to the AWS Management Console.
2. Select the intended AWS Region and open **EC2**.
3. Choose **Launch instance**.
4. Enter a server name.
5. Select an operating-system image (AMI).
6. Select an instance type.
7. Create or select a key pair.
8. Configure networking and a security group.
9. Review and configure the EBS storage volume.
10. Review the configuration and launch the instance.
11. Wait for **Running** state and successful status checks.
12. Connect from Windows using MobaXterm and SSH.

## 4. EC2 launch settings

| Setting | Meaning |
|---|---|
| Name | Tag used to identify the server, e.g. `devops-ec2-lab` |
| AMI | Template used to launch the OS and initial software |
| Instance type | Defines vCPU, memory, and networking capabilities |
| Key pair | Public/private key pair used for supported secure login |
| VPC | Virtual network for the instance |
| Subnet | Network segment located in one Availability Zone |
| Public IP | Address for direct internet access when routing and rules allow |
| Security group | Stateful virtual firewall controlling allowed traffic |
| EBS volume | Persistent block storage attached to the instance |
| User data (optional) | Script that can run during initial launch for setup automation |

## 5. Create the security group

A security group controls allowed traffic to and from the instance. It is **stateful**, so response traffic for a tracked permitted connection is automatically allowed.

Example security group name: `devops-ec2-lab-sg`

### Inbound rules

Inbound rules control traffic coming **into** the instance.

| Type | Protocol | Port | Source | Purpose |
|---|---|---:|---|---|
| SSH | TCP | 22 | My IP (`x.x.x.x/32`) | Secure remote terminal access from your current public IP |
| HTTPS (optional) | TCP | 443 | `0.0.0.0/0` only when hosting a public HTTPS service | Accept public HTTPS requests |

For a basic SSH lab, SSH port 22 is enough; you do not need HTTPS unless you are deliberately hosting an HTTPS application. Opening port 443 alone does not install a web server or configure TLS.

**Do not expose SSH to `0.0.0.0/0` for a normal lab.** If your public IP changes, update the rule to your current IP.

### Outbound rules

Outbound rules control traffic **leaving** the instance.

- The default security group setup commonly allows all outbound IPv4 traffic to `0.0.0.0/0`.
- This can allow package downloads and connections to external services.
- Learn the default before tightening egress rules. Production environments should apply least-privilege egress where practical.
- Because security groups are stateful, you do not need a separate reverse rule just for response traffic.

### Steps

1. On the launch page, open **Network settings**.
2. Choose **Create security group**.
3. Enter a name and description.
4. Add SSH, TCP port 22, source **My IP**.
5. Add HTTPS, TCP port 443, only if required for your planned service.
6. Review outbound rules.
7. Review all rules before launching.

**Screenshot placeholder:** `screenshots/05-security-group-rules.png`

## 6. Launch the instance

### Step A — Name and OS image
1. Enter a descriptive name, e.g. `devops-ec2-lab`.
2. Select a suitable Linux AMI, such as Amazon Linux or Ubuntu.
3. Verify the publisher, version, and architecture.

**Screenshot:** `screenshots/02-name-and-ami.png`

### Step B — Instance type
1. Select a small instance type appropriate for the lab.
2. Check availability, current pricing, and applicable free-tier terms.

**Screenshot:** `screenshots/03-instance-type.png`

### Step C — Key pair
1. Create or select a key pair you are authorized to use.
2. If creating one, choose a supported private-key format for your SSH client.
3. Download the private key and store it securely on your Windows laptop.
4. Never commit the private key to GitHub. AWS does not provide another download of the same private key.

**Screenshot:** `screenshots/04-key-pair.png`

### Step D — Networking
1. Select the intended VPC and subnet.
2. If connecting directly over the internet, verify that routing supports the connection.
3. Enable auto-assign public IP if appropriate and required for your lab.
4. Select the security group you created.

A public IP alone does not guarantee connectivity. Routing and firewall rules must also allow the traffic.

**Screenshot:** `screenshots/06-network-settings.png`

### Step E — Storage
1. Review the root EBS volume.
2. Select an appropriate volume size and type.
3. Review encryption and delete-on-termination settings.
4. Add extra volumes only if needed.
5. Remember that EBS volumes and snapshots may continue to incur charges when the instance is stopped or terminated.

**Screenshot:** `screenshots/07-storage-volume.png`

### Step F — Launch
1. Review the name, AMI, instance type, key pair, networking, security group, and storage.
2. Click **Launch instance**.
3. Open the instance list and wait for the state to become **Running**.
4. Wait for status checks to pass.
5. Select the instance and review its public and private IPv4 addresses.

**Screenshot:** `screenshots/08-instance-running.png`

## 7. Connect from Windows using MobaXterm

### Before connecting, verify
- The instance is running and status checks pass.
- You have the correct public IP or another valid access path.
- The security group permits TCP port 22 from your current public IP.
- The VPC/subnet routing allows the connection.
- You have the private key matching the instance's key pair.
- You know the correct OS username.

Common default usernames:
- Amazon Linux: `ec2-user`
- Ubuntu: `ubuntu`

Other images may use a different username; check the AMI documentation.

### MobaXterm steps
1. Open MobaXterm on Windows.
2. Click **Session**, then choose **SSH**.
3. Enter the instance's public IPv4 address in **Remote host**.
4. Enable the SSH username option and enter the correct username.
5. In advanced SSH settings, specify the matching private-key file if required by your MobaXterm version.
6. Start the session.
7. On first connection, verify the server host key using an appropriate trusted method before accepting it.
8. After login, run basic commands.

**Screenshot:** `screenshots/09-mobaxterm-ssh-session.png`

### Example OpenSSH command

```bash
ssh -i "C:\path\to\your-private-key.pem" ec2-user@PUBLIC_IP
```

Replace the key path, username, and `PUBLIC_IP` with your own values. For Ubuntu, the username is commonly `ubuntu`. Never share the private key or paste its contents into a chat or repository.

### Verify the connection

```bash
whoami
hostname
pwd
ip addr
df -h
free -h
```

These commands show the logged-in user, hostname, current directory, network interfaces, disk usage, and memory.

**Screenshot:** `screenshots/10-ssh-commands.png`

## 8. Optional: host an HTTPS application

Only do this if you intentionally plan to host a web service:
1. Install and configure the web server/application.
2. Configure TLS with an appropriate certificate.
3. Allow TCP port 443 from intended client sources.
4. Confirm routing and that the service listens on port 443.
5. Test HTTPS using the correct DNS name and certificate.

Opening port 443 alone does not configure HTTPS.

## 9. Troubleshooting

| Problem | Checks |
|---|---|
| SSH connection times out | Public IP, route table, internet gateway, security group source, network ACLs, local network restrictions |
| Permission denied | Correct username, private key, key-pair match, key permissions |
| No public IP | Subnet and auto-assign settings; consider SSM, VPN, or a bastion |
| Running but unreachable | Status checks, routes, security group, network ACL, OS firewall, SSH daemon |
| HTTPS fails | Service status, port 443 rule, routing, TLS configuration, DNS |
| Package downloads fail | Outbound access, DNS, internet/NAT route as appropriate |
| Public IP changes | Some public IPs are reassigned after stop/start; review Elastic IP needs and costs |

## 10. EC2 lifecycle and cleanup

- **Reboot:** Restarts the operating system.
- **Stop:** Shuts down an EBS-backed instance; EBS generally remains and can continue to incur charges.
- **Start:** Starts a stopped instance; its public IPv4 address may change.
- **Terminate:** Permanently deletes the instance. Volumes configured for deletion on termination may be deleted; other resources may remain.

After the lab:
- [ ] Save sanitized screenshots and notes.
- [ ] Record errors and resolutions.
- [ ] Stop or terminate the instance when appropriate.
- [ ] Check for leftover EBS volumes, snapshots, Elastic IPs, and other billable resources.
- [ ] Do not delete volumes or snapshots whose data is still needed.

## 11. Learning log

**Date:** 21 October 2026  
**Topic:** AWS EC2 — Launch an instance and connect through SSH

### What I learned
- EC2 provides virtual compute capacity in AWS.
- An AMI is the OS image/template used to launch an instance.
- Instance types determine compute resources.
- A key pair supports secure authentication.
- Security groups control allowed inbound/outbound traffic and are stateful.
- SSH commonly uses TCP port 22; HTTPS commonly uses TCP port 443.
- A public IP needs suitable routing and security rules to be reachable.
- EBS storage can continue to incur charges after an instance stops.
- MobaXterm can connect to Linux EC2 using SSH, the instance address, correct username, and matching private key.

### Practical checklist
- [ ] Opened EC2 in the intended Region
- [ ] Selected AMI and instance type
- [ ] Created/selected a key pair
- [ ] Created security group and reviewed inbound/outbound rules
- [ ] Configured networking and storage
- [ ] Launched instance and verified status checks
- [ ] Connected from Windows with MobaXterm
- [ ] Ran Linux verification commands
- [ ] Captured and sanitized screenshots
- [ ] Checked costs and cleaned up resources

**Status:** Mark each item complete only after doing the task yourself.

## 12. Interview questions and answers

1. **What is EC2?** A service that provides resizable virtual compute capacity in AWS.
2. **What is an AMI?** A template used to launch an instance, including an OS image and launch configuration information.
3. **What does an instance type define?** Compute characteristics such as vCPU, memory, and networking.
4. **What is a key pair?** A public/private key pair used for supported secure access, such as SSH.
5. **What is a security group?** A stateful virtual firewall that allows traffic matching its rules.
6. **What is an inbound rule?** A rule controlling traffic entering the protected resource.
7. **What is an outbound rule?** A rule controlling traffic leaving the protected resource.
8. **Why restrict SSH to My IP?** It reduces exposure by allowing SSH from the intended public IP rather than the entire internet.
9. **What is the difference between port 22 and 443?** TCP 22 is commonly SSH; TCP 443 is commonly HTTPS.
10. **Why can a public-IP instance be unreachable?** Routing, security groups, network ACLs, host firewall rules, or the service may block access.
11. **Why does MobaXterm need a private key?** It uses the key to authenticate when SSH public-key authentication is configured.
12. **Stop vs. terminate?** Stop shuts down an instance that can generally be restarted; terminate permanently deletes it.
13. **Does opening port 443 configure HTTPS?** No; the service, TLS certificate, and application configuration are also required.
14. **What would you check if SSH times out?** Address, port 22 rule, routing, public connectivity, network ACLs, and instance status.
15. **What should you check after the lab?** Running resources, EBS volumes, snapshots, Elastic IPs, and other possible charges.

## References
- EC2 concepts: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html
- Launch instances: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/LaunchingAndUsingInstances.html
- Security groups: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html
- Connect to Linux instances: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-linux-inst-ssh.html
- Amazon EBS: https://docs.aws.amazon.com/ebs/
