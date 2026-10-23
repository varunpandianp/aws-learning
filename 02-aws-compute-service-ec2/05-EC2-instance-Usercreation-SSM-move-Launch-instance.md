# AWS EC2 – Final Session Notes
## Sudharsan AWS Cloud Computing Course

## Topics Covered

1. Moving an EC2 machine to another AWS Region
2. Copying an AMI from one Region to another
3. EC2 Launch Templates and their use cases
4. Logging into Linux EC2 without an SSH key pair
5. Linux users, passwords, SSH password authentication, and sudo permissions
6. Updating software on a live EC2 instance using AWS Systems Manager (SSM)
7. Configuring IAM roles and policies for SSM
8. Installing and starting the SSM Agent
9. Managing EC2 instances through Fleet Manager
10. Executing shell scripts through Systems Manager Run Command

---

# 1. Moving an EC2 Instance to Another AWS Region

## What is an AWS Region?

An AWS Region is a separate geographical area where AWS provides cloud services.

Examples:
- Mumbai: `ap-south-1`
- Singapore: `ap-southeast-1`
- N. Virginia: `us-east-1`

Each Region has its own EC2 resources, AMIs, and other regional resources.

## Can We Directly Move an EC2 Instance to Another Region?

An existing EC2 instance cannot simply be transferred directly from one Region to another.

A common approach is:

1. Create an Amazon Machine Image (AMI) from the source EC2 instance.
2. Copy the AMI to the destination Region.
3. Open the destination Region.
4. Locate the copied AMI.
5. Launch a new EC2 instance using that AMI.
6. Configure networking, security groups, IAM roles, and other required settings.
7. Test the new instance before switching production traffic.

## Step-by-Step Procedure

### Step 1: Create an AMI

1. Open the AWS Management Console.
2. Navigate to EC2.
3. Select **Instances**.
4. Select the source instance.
5. Choose **Actions → Image and templates → Create image**.
6. Enter an image name and description.
7. Create the image.
8. Wait until the AMI becomes available.

### Step 2: Copy the AMI

1. Navigate to **EC2 → AMIs**.
2. Select the AMI you created.
3. Choose **Actions → Copy AMI**.
4. Select the destination Region.
5. Enter the required details.
6. Start the copy operation.
7. Switch to the destination Region.
8. Wait until the copied AMI becomes available.

### Step 3: Launch an Instance

1. Open EC2 in the destination Region.
2. Navigate to **AMIs**.
3. Select the copied AMI.
4. Choose **Launch instance from AMI**.
5. Select the instance type.
6. Configure the VPC, subnet, security group, key pair, storage, and IAM role.
7. Launch the instance.
8. Connect to it and verify that the application works.

## Important Points

- AMIs are Region-specific resources.
- Copying an AMI to another Region creates a separate AMI in that Region.
- The new instance receives a different instance ID and generally a different private and public IP address.
- VPCs, subnets, security groups, and key pairs are Region-specific and may need to be recreated or configured in the destination Region.
- Check application configuration, DNS records, certificates, IAM permissions, and external dependencies.
- Data stored outside the AMI, such as in separate EBS volumes, EFS file systems, or external databases, requires its own migration plan.
- Encrypted AMIs may require appropriate AWS KMS permissions and a compatible destination-Region encryption key.
- Creating an AMI can involve reboots depending on the options selected. Plan production migrations carefully.

## Use Cases

- Migrating an application to a different Region.
- Disaster recovery.
- Meeting geographical or regulatory requirements.
- Reducing latency for users in another geographical area.
- Creating a copy of an existing server environment.

---

# 2. Copying an AMI from One Region to Another

## What Is an AMI?

An **Amazon Machine Image (AMI)** is a template used to launch EC2 instances.

It contains the information required to create an instance, including an operating system and configured software, together with the associated image and storage mapping information.

An AMI can include:
- Operating system configuration.
- Installed applications and packages.
- Files contained in the captured root volume.
- EBS volume mappings.
- Other configuration captured when the image is created.

## AMI Copy vs. Snapshot Copy

| Feature | Copy AMI | Copy EBS Snapshot |
|---|---|---|
| Main purpose | Reproduce an EC2 machine image in another Region | Copy an EBS volume backup |
| Contains | AMI configuration and references to its backing snapshots | A point-in-time block-level volume backup |
| Can launch an EC2 instance directly? | Yes, after the copied AMI becomes available | Not directly; create a volume from the snapshot or use it as part of an image workflow |
| Common use case | Region migration and reusable machine images | Volume backup and data migration |

## Practical Summary

**Source Region → Create AMI → Copy AMI → Destination Region → Launch EC2**

Remember: copying an AMI does not automatically migrate every AWS resource associated with the original instance.

---

# 3. EC2 Launch Templates

## What Is a Launch Template?

An **EC2 Launch Template** stores the configuration used to launch EC2 instances.

Instead of manually configuring every instance, we create a template and reuse it whenever we need similar instances.

A launch template can define:
- AMI ID.
- Instance type.
- Key pair.
- Security groups.
- Network configuration.
- EBS volume configuration.
- IAM instance profile.
- User data.
- Resource tags.
- Other launch settings.

## Why Do We Need Launch Templates?

Without a launch template, an administrator may need to configure every EC2 instance manually.

This can lead to:
- Repeated configuration work.
- Incorrect security group selection.
- Inconsistent instance types.
- Configuration mistakes.
- Difficulty maintaining standard environments.

A launch template makes instance creation more consistent and repeatable.

## How to Create a Launch Template

1. Open the AWS Management Console.
2. Navigate to **EC2 → Launch Templates**.
3. Choose **Create launch template**.
4. Enter a template name and description.
5. Select an AMI.
6. Select an instance type.
7. Configure the key pair if required.
8. Configure the security groups and network settings.
9. Configure storage.
10. Attach an IAM instance profile if needed.
11. Add user data if you want to automate startup configuration.
12. Create the launch template.

## How to Launch an Instance from a Template

1. Open **EC2 → Launch Templates**.
2. Select your launch template.
3. Choose **Actions → Launch instance from template**.
4. Select the required template version.
5. Review the settings.
6. Launch the instance.

## Launch Template Use Cases

- Creating development, testing, and production instances with consistent configurations.
- Launching multiple similar instances.
- Supporting Auto Scaling groups.
- Standardizing EC2 deployments.
- Automating infrastructure creation.
- Maintaining multiple versions of an instance configuration.

## Launch Template vs. AMI

| AMI | Launch Template |
|---|---|
| Defines the machine image used to launch an instance | Defines the launch configuration |
| Contains the operating system image and captured software/configuration | Can specify the AMI, instance type, networking, storage, user data, and other launch settings |
| Used as the source image for an instance | Used as a reusable configuration for launching instances |
| Can be copied across Regions | Is a separate regional resource and may need to be recreated or managed in the destination Region |

**Easy way to remember:**

- **AMI = What machine image should be used?**
- **Launch Template = How should the instance be launched?**

---

# 4. Logging into Linux EC2 Without an SSH Key Pair

## Default SSH Authentication

Linux EC2 instances commonly use SSH key-based authentication.

For example:

```bash
ssh -i my-key.pem ec2-user@PUBLIC_IP
```

Here:
- `ssh` establishes a secure remote connection.
- `-i my-key.pem` specifies the private key file.
- `ec2-user` is a common default username for Amazon Linux.
- `PUBLIC_IP` is the instance's public IP address.

Ubuntu instances commonly use the default username `ubuntu`.

## Can We Log In Using a Username and Password?

Yes, Linux can support SSH password authentication if the server is configured to allow it and a valid user account has been created.

However, **password authentication is commonly disabled by default or restricted in cloud Linux configurations**.

For production EC2 instances, prefer AWS Systems Manager Session Manager or properly managed SSH key authentication rather than exposing password-based SSH access to the internet.

## Step 1: Connect to the Linux Instance

Connect using an existing authorized method, such as SSH with a key pair or Session Manager.

## Step 2: Create a Linux User

Example:

```bash
sudo useradd -m devuser
```

This creates a user named `devuser` and a home directory.

Set a password:

```bash
sudo passwd devuser
```

Linux prompts you to enter and confirm the password.

**Important:** Use a strong, unique password. Never put passwords directly into shell scripts, source control, or command-line arguments.

## Step 3: Check the SSH Server Configuration

The main SSH server configuration file is:

```bash
/etc/ssh/sshd_config
```

Open it using an editor:

```bash
sudo vi /etc/ssh/sshd_config
```

Look for the password authentication setting:

```text
PasswordAuthentication no
```

If password authentication is required, the setting may need to be changed to:

```text
PasswordAuthentication yes
```

The exact configuration can vary by operating system. Included configuration files may also affect the effective settings.

Check the effective configuration:

```bash
sudo sshd -T | grep -i passwordauthentication
```

**Security warning:** Enabling password authentication increases the risk of password guessing and unauthorized access. Do not enable it on a publicly accessible server without a carefully reviewed security plan.

## Step 4: Validate the SSH Configuration

Before reloading SSH, check the configuration:

```bash
sudo sshd -t
```

If no error is displayed, the syntax check has passed.

Reload the SSH service using the service name supported by your operating system:

```bash
sudo systemctl reload sshd
```

On some distributions, the service name is `ssh` instead:

```bash
sudo systemctl reload ssh
```

Keep the existing authorized session open while testing a new connection.

## Step 5: Check the Security Group

If connecting over SSH, the EC2 security group must allow inbound TCP port `22` from an authorized source.

For better security:
- Restrict SSH access to a trusted IP address or private network.
- Avoid opening port `22` to `0.0.0.0/0`.
- Use a strong password if password authentication is explicitly required.
- Consider disabling password authentication again after testing.

## Step 6: Connect Using the Username and Password

From a client machine:

```bash
ssh devuser@PUBLIC_IP
```

If password authentication is enabled and network access is allowed, SSH prompts for the user's password.

**Important:** Creating a Linux user and setting a password alone does not guarantee SSH login will work. The SSH configuration, operating system policies, network access, account status, and authentication restrictions must also permit the login.

---

# 5. Linux Users, Root User, and Sudo Permissions

## What Is the Root User?

The `root` account is the Linux superuser.

It has extensive administrative privileges and can modify almost all parts of the operating system.

Examples of administrative tasks include:
- Installing software.
- Creating or deleting users.
- Changing system configuration.
- Managing services.
- Modifying protected system files.

## Why Is Direct Root Login Restricted?

Direct root login can be dangerous because root has unrestricted privileges.

If root access is compromised, an attacker may gain complete control of the system.

For this reason, many Linux cloud images restrict direct root SSH login.

A common SSH configuration is:

```text
PermitRootLogin no
```

The exact setting should be verified on the target system.

## What Is Sudo?

`sudo` allows authorized users to execute commands with elevated privileges.

For example:

```bash
sudo dnf update -y
```

This executes the package update command with administrative privileges on systems that use DNF.

For Ubuntu:

```bash
sudo apt update
```

## Adding a User to the Appropriate Administrative Group

The exact administrative group depends on the Linux distribution.

On Amazon Linux, administrators commonly use the `wheel` group:

```bash
sudo usermod -aG wheel devuser
```

On Ubuntu, the common administrative group is `sudo`:

```bash
sudo usermod -aG sudo devuser
```

Group membership should be granted only when the user genuinely requires administrative access.

## What Is the Sudoers File?

The sudoers configuration controls which users and groups can execute commands with elevated privileges.

The main file is:

```bash
/etc/sudoers
```

**Never edit the sudoers file carelessly.** An incorrect configuration can prevent administrative access.

Use:

```bash
sudo visudo
```

`visudo` validates the syntax before saving changes.

For a user-specific configuration, administrators can create a file under:

```bash
/etc/sudoers.d/
```

Only grant the permissions required for the user's role. Avoid granting unrestricted root-equivalent privileges unless necessary.

## Important Difference

| Concept | Purpose |
|---|---|
| Linux user | Identifies a user account |
| Password | Provides a possible authentication method |
| SSH configuration | Controls how SSH connections are permitted |
| Root account | Provides full administrative privileges |
| Sudo | Allows authorized users to execute elevated commands |
| LDAP | Provides centralized directory-based identity and authentication when integrated with Linux |

---

# 6. What Is LDAP?

**LDAP stands for Lightweight Directory Access Protocol.**

It is a protocol used to access and manage directory information, including user and group information.

Organizations may use LDAP-integrated directory services to centralize user identity management across multiple servers.

## Why Do Organizations Use LDAP?

Imagine an organization with 100 Linux servers and hundreds of employees.

Creating and maintaining separate user accounts on every server would be difficult.

With a centrally managed directory service:
- User identities can be maintained centrally.
- Administrators can manage group membership.
- Authentication can be integrated across multiple systems.
- Access management becomes easier to standardize.

Linux systems can integrate with directory services using tools such as SSSD, depending on the environment.

**Important:** LDAP is not automatically enabled on an EC2 instance. It requires an appropriate directory service and client-side configuration. AWS EC2 does not require LDAP for ordinary SSH access.

---

# 7. Updating Software on a Live EC2 Instance Using AWS Systems Manager

## What Is AWS Systems Manager?

**AWS Systems Manager (SSM)** is an AWS service used to manage and operate supported resources, including EC2 instances.

It can help administrators:
- Execute commands remotely.
- Install software updates.
- Inspect managed instances.
- Run automation tasks.
- Manage instances without opening inbound SSH or RDP ports when using appropriate features.

## Why Use SSM?

Traditionally, an administrator might connect to a Linux instance through SSH and manually execute commands.

With Systems Manager, authorized administrators can execute commands remotely through the AWS Management Console or AWS APIs.

For example, an administrator can use Run Command to install updates on a supported Linux instance.

### Benefits

- Remote command execution.
- Centralized management.
- Reduced need for inbound SSH access.
- Easier administration of multiple instances.
- Better automation and operational visibility.
- Integration with IAM permissions and AWS auditing services.

**Important:** Installing updates on a live production server can restart services, change dependencies, or affect application availability. Review updates and test them before production deployment.

---

# 8. Step 1 — Create or Configure the IAM Role for SSM

## Why Do We Need an IAM Role?

An EC2 instance needs permission to communicate with AWS Systems Manager.

Without the appropriate permissions and identity configuration, the instance may not register as a Systems Manager managed node.

The recommended approach is to attach an IAM role to the EC2 instance through an instance profile.

## Steps to Configure the IAM Role

1. Open the AWS Management Console.
2. Navigate to **IAM → Roles**.
3. Choose **Create role**.
4. Select **AWS service** as the trusted entity type.
5. Choose **EC2** as the use case.
6. Attach the required Systems Manager permissions.
7. For the standard managed-node setup, use the AWS-managed policy:

```text
AmazonSSMManagedInstanceCore
```

8. Enter a role name, such as:

```text
EC2-SSM-Management-Role
```

9. Create the role.
10. Open **EC2 → Instances**.
11. Select the target instance.
12. Choose **Actions → Security → Modify IAM role**.
13. Attach the newly created role.

## What Does AmazonSSMManagedInstanceCore Do?

It provides commonly required permissions for an EC2 instance to communicate with Systems Manager and register as a managed node.

Use the AWS-managed policy rather than creating an overly broad policy with administrator permissions.

## Important Points

- The IAM role must be attached to the EC2 instance.
- Creating a role alone does not attach it to the instance.
- Permissions must be appropriate for the operations being performed.
- SSM management and patching may require additional permissions or configuration depending on the task.

---

# 9. Step 2 — Install and Start the SSM Agent

## What Is the SSM Agent?

The **AWS Systems Manager Agent (SSM Agent)** is software that runs on a managed machine.

It enables the instance to communicate with Systems Manager and perform supported management operations.

## Why Do We Need the Agent?

The agent:
- Communicates with Systems Manager.
- Registers the machine as a managed node.
- Receives and executes authorized management commands.
- Reports information needed for management operations.

Many supported AWS-provided AMIs already include SSM Agent. If it is absent or outdated, you may need to install or update it.

## Check Whether the Agent Is Installed

On a Linux instance:

```bash
systemctl status amazon-ssm-agent
```

Check whether the service is enabled:

```bash
systemctl is-enabled amazon-ssm-agent
```

Check whether the service is running:

```bash
systemctl is-active amazon-ssm-agent
```

If the service exists but is stopped, start it:

```bash
sudo systemctl start amazon-ssm-agent
```

Enable it to start automatically at boot:

```bash
sudo systemctl enable amazon-ssm-agent
```

Start and enable it in one command:

```bash
sudo systemctl enable --now amazon-ssm-agent
```

## What If the Agent Is Not Installed?

Installation steps depend on the operating system and installation source.

For supported Amazon Linux images, you can check package availability:

```bash
sudo dnf info amazon-ssm-agent
```

On systems that use `yum`:

```bash
sudo yum info amazon-ssm-agent
```

Follow the official AWS Systems Manager Agent installation instructions for your operating system if the package is unavailable.

Do not assume that every Linux distribution uses the same package manager or installation procedure.

## Check Agent Logs

On common Linux installations:

```bash
sudo journalctl -u amazon-ssm-agent
```

To view recent messages:

```bash
sudo journalctl -u amazon-ssm-agent -n 50
```

## Network Requirements

The instance must be able to communicate with the Systems Manager service endpoints.

This generally requires:
- Working DNS resolution.
- Outbound HTTPS connectivity on TCP port `443`.
- Internet access through an appropriate route, or configured VPC endpoints.
- Correct IAM permissions.
- A running, compatible SSM Agent.

A public IP address is not always required. Instances in private subnets can use properly configured VPC endpoints or another supported network path.

---

# 10. Step 3 — Verify the Instance in Fleet Manager

## What Is Fleet Manager?

**Fleet Manager** is a capability within AWS Systems Manager that helps administrators view and manage registered managed nodes.

It can provide information about managed instances and support certain remote management tasks.

## Steps to Check the EC2 Instance

1. Open the AWS Management Console.
2. Navigate to **Systems Manager**.
3. Open **Fleet Manager** or **Managed nodes**, depending on the console layout.
4. Select the correct AWS Region.
5. Find your EC2 instance.
6. Check its managed-node status and available information.

You can also check **Systems Manager → Managed nodes** to confirm that the instance is registered.

## If the Instance Does Not Appear

Check the following:

1. Is the correct IAM role attached?
2. Does the role provide the required Systems Manager permissions?
3. Is SSM Agent installed?
4. Is SSM Agent running?
5. Can the instance reach the Systems Manager endpoints over HTTPS?
6. Is the AWS Region correct?
7. Is the operating system supported?
8. Are there relevant agent errors in the system logs?

**Important:** An EC2 instance running successfully does not automatically mean it is registered as an SSM managed node.

---

# 11. Step 4 — Execute a Shell Script Using Systems Manager Run Command

## What Is Run Command?

Run Command is an AWS Systems Manager capability that executes commands on managed nodes without requiring an interactive SSH connection.

For Linux instances, administrators can use it to run shell commands and supported scripts.

## Example Use Case: Update Linux Software

Suppose a Linux EC2 instance is running and registered as a managed node.

Instead of logging in manually, an administrator can use Run Command to execute a package update.

## Console Steps

1. Open **AWS Systems Manager**.
2. Navigate to **Run Command**.
3. Choose **Run command**.
4. Select an appropriate document for the operating system.
5. For a Linux shell script, choose the appropriate `AWS-RunShellScript` document.
6. Select the target EC2 managed node.
7. Enter the commands to execute.
8. Review the targets and commands.
9. Choose **Run**.
10. Open the command execution details.
11. Review the status and output.

The exact console labels may vary slightly as AWS updates its interface.

## Example A: Check the Linux Operating System

Use this script:

```bash
#!/bin/bash
set -euxo pipefail

echo "Checking operating system"
cat /etc/os-release

echo "Checking disk usage"
df -h

echo "Checking memory usage"
free -m

echo "Checking system uptime"
uptime
```

This script:
- Displays the operating system information.
- Checks disk usage.
- Checks memory usage.
- Displays system uptime.

## Example B: Update Packages on Amazon Linux

For Amazon Linux 2023 or another supported DNF-based system:

```bash
#!/bin/bash
set -euxo pipefail

echo "Starting package update"
sudo dnf check-update || rc=$?

if [ "${rc:-0}" -gt 100 ]; then
    echo "Package manager returned an unexpected error"
    exit "$rc"
fi

sudo dnf upgrade -y

echo "Package update completed"
```

**Simpler alternative for a controlled lab:**

```bash
sudo dnf upgrade -y
```

For Ubuntu, the package update commands are different:

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

Choose the correct package manager for the operating system.

**Production warning:** Package upgrades may change important dependencies or restart services. Test updates first, use maintenance windows when appropriate, and make sure you have a recovery plan.

## Example C: Install and Start Apache on Amazon Linux

```bash
#!/bin/bash
set -euxo pipefail

sudo dnf install -y httpd
sudo systemctl enable --now httpd

echo "Apache installation completed"
sudo systemctl status httpd --no-pager
```

This script:
1. Installs Apache.
2. Enables Apache at boot.
3. Starts the service.
4. Displays the service status.

## Example D: Create a Test File

```bash
#!/bin/bash
set -euxo pipefail

echo "SSM Run Command executed successfully" | sudo tee /tmp/ssm-test.txt

cat /tmp/ssm-test.txt
```

After running the command, review the execution output in Systems Manager.

## Understanding Command Output

A command execution can report states such as:
- Pending
- In Progress
- Success
- Failed
- Timed Out
- Cancelled

Review both the overall command status and the output for each target instance.

A successful command submission does not necessarily mean the script completed successfully.

---

# 12. Troubleshooting SSM Run Command

If the instance does not appear in Systems Manager or a command fails, use the following checklist.

| Problem | What to Check |
|---|---|
| Instance is not listed | Correct Region, IAM role, SSM Agent, and network connectivity |
| Access denied | IAM permissions of the person running the command |
| Instance cannot register | EC2 instance role and SSM Agent logs |
| Command remains pending | Managed-node connectivity, service health, and command status |
| Package update fails | Package manager, repositories, network connectivity, and available disk space |
| Script fails | Command output, exit code, shell syntax, and operating-system compatibility |
| No output is visible | Run Command execution details and output configuration |

## Check Agent Logs

```bash
sudo journalctl -u amazon-ssm-agent -n 100
```

## Check Agent Status

```bash
sudo systemctl status amazon-ssm-agent
```

## Check Disk Space

```bash
df -h
```

## Check Network Connectivity

The instance must have a valid route and access to the required Systems Manager endpoints.

Do not open inbound SSH to the internet just to fix an SSM connectivity issue. Diagnose the IAM role, agent, DNS, routing, and outbound HTTPS connectivity first.

---

# 13. Complete SSM Workflow

Remember the complete workflow:

```text
EC2 Instance
     |
     v
Attach IAM Role
     |
     v
Grant Required SSM Permissions
     |
     v
Install SSM Agent if Missing
     |
     v
Start and Enable SSM Agent
     |
     v
Ensure Network Connectivity
     |
     v
Instance Registers with Systems Manager
     |
     v
Verify Managed Node in Fleet Manager
     |
     v
Open Run Command
     |
     v
Choose AWS-RunShellScript
     |
     v
Select EC2 Instance
     |
     v
Enter Shell Script
     |
     v
Run Command
     |
     v
Review Execution Status and Output
```

---

# 14. Important AWS EC2 Commands

## Linux User Management

Create a user:

```bash
sudo useradd -m devuser
```

Set a password:

```bash
sudo passwd devuser
```

Check the user:

```bash
id devuser
```

## SSH Configuration

Open SSH configuration:

```bash
sudo vi /etc/ssh/sshd_config
```

Validate configuration:

```bash
sudo sshd -t
```

Check effective password authentication:

```bash
sudo sshd -T | grep -i passwordauthentication
```

## Sudo Permissions

Edit sudoers safely:

```bash
sudo visudo
```

Add a user to the Amazon Linux administrative group:

```bash
sudo usermod -aG wheel devuser
```

Add a user to the Ubuntu administrative group:

```bash
sudo usermod -aG sudo devuser
```

## SSM Agent

Check status:

```bash
sudo systemctl status amazon-ssm-agent
```

Start and enable:

```bash
sudo systemctl enable --now amazon-ssm-agent
```

View logs:

```bash
sudo journalctl -u amazon-ssm-agent -n 50
```

## Linux System Information

Check operating system:

```bash
cat /etc/os-release
```

Check disk space:

```bash
df -h
```

Check memory:

```bash
free -m
```

Check uptime:

```bash
uptime
```

---

# 15. Interview Questions and Answers

## Q1. Can we directly move an EC2 instance from one Region to another?

**Answer:** No. A common migration approach is to create an AMI from the source instance, copy the AMI to the destination Region, and launch a new instance from the copied AMI.

## Q2. What is an AMI?

**Answer:** An Amazon Machine Image is a template used to launch EC2 instances. It contains the machine image and configuration information needed to create an instance.

## Q3. What is the purpose of copying an AMI to another Region?

**Answer:** It enables us to launch an equivalent machine image in another Region for migration, disaster recovery, or geographical deployment.

## Q4. What is an EC2 Launch Template?

**Answer:** A Launch Template stores reusable EC2 launch settings, such as the AMI, instance type, security groups, storage, IAM instance profile, and user data.

## Q5. What is the difference between an AMI and a Launch Template?

**Answer:** An AMI provides the machine image, while a Launch Template defines how to launch an instance using an AMI and other settings.

## Q6. How can we create a Linux user?

**Answer:** Use `sudo useradd -m username`, then set a password with `sudo passwd username` if password-based authentication is required.

## Q7. Why might SSH password authentication fail even when a Linux user has a password?

**Answer:** Password authentication may be disabled in SSH configuration, the account may be restricted, or network access and other authentication policies may prevent the connection.

## Q8. What is the root user?

**Answer:** Root is the Linux superuser with extensive administrative privileges.

## Q9. What is sudo?

**Answer:** Sudo allows authorized users to execute commands with elevated privileges without requiring direct root login.

## Q10. What is LDAP?

**Answer:** LDAP is a protocol used to access and manage directory information. It can be used as part of centralized user and group management.

## Q11. What is AWS Systems Manager?

**Answer:** AWS Systems Manager is an AWS service that provides capabilities for managing, automating, and operating supported resources such as EC2 instances.

## Q12. What is SSM Agent?

**Answer:** SSM Agent is software that runs on a managed machine and enables communication with Systems Manager for supported management operations.

## Q13. Why do we attach an IAM role to an EC2 instance for SSM?

**Answer:** The role supplies the AWS permissions the instance needs to communicate with Systems Manager and perform authorized operations.

## Q14. What is `AmazonSSMManagedInstanceCore`?

**Answer:** It is an AWS-managed IAM policy that provides commonly required permissions for an EC2 instance to use Systems Manager as a managed node.

## Q15. What is Fleet Manager?

**Answer:** Fleet Manager is a Systems Manager capability used to view and manage registered managed nodes.

## Q16. What is Run Command?

**Answer:** Run Command executes commands on managed nodes without requiring a direct interactive SSH connection.

## Q17. What is `AWS-RunShellScript`?

**Answer:** It is a Systems Manager command document used to run shell commands or scripts on supported Linux managed nodes.

## Q18. Does SSM require a public IP address?

**Answer:** Not necessarily. A private instance can use appropriate VPC endpoints or another supported network path to reach Systems Manager endpoints.

## Q19. Why might an EC2 instance not appear in Fleet Manager?

**Answer:** Possible causes include an incorrect Region, missing IAM permissions, an absent or stopped SSM Agent, unsupported configuration, or network connectivity problems.

## Q20. Why should we be careful when updating a live EC2 instance?

**Answer:** Updates can change dependencies, restart services, or cause application downtime. Production updates should be tested and performed with an appropriate recovery plan.

---

# 16. Learning Log

## AWS EC2 – Final Session

### What I Learned

- Learned how to migrate an EC2 machine to another AWS Region using an AMI.
- Learned how to copy an AMI from one Region to another.
- Understood the difference between an AMI and an EC2 Launch Template.
- Learned how Launch Templates simplify repeated instance creation.
- Understood Linux users, passwords, SSH password authentication, and root access restrictions.
- Learned how sudo grants controlled administrative privileges.
- Understood the role of LDAP in centralized identity management.
- Learned why an EC2 instance needs the correct IAM role for Systems Manager.
- Understood the purpose of SSM Agent.
- Learned how to check SSM Agent status and logs.
- Learned how to verify managed nodes in Fleet Manager.
- Learned how to execute shell scripts using Systems Manager Run Command.
- Practiced the concepts of Linux software updates and remote administration.

### Practical Tasks to Complete

- [ ] Create an AMI from an EC2 instance.
- [ ] Copy the AMI to another Region.
- [ ] Launch a new instance from the copied AMI.
- [ ] Create a Launch Template.
- [ ] Launch an instance using the Launch Template.
- [ ] Create a Linux user and understand SSH authentication.
- [ ] Review root login restrictions and sudo permissions.
- [ ] Attach an IAM role with the required SSM permissions.
- [ ] Check whether SSM Agent is installed and running.
- [ ] Verify the instance under Systems Manager managed nodes.
- [ ] Execute a test script using Run Command.
- [ ] Review the command output and troubleshoot a failure.

---

# Final Summary

In this final EC2 session, I learned how to migrate EC2 instances between Regions, reuse machine configurations through AMIs and Launch Templates, understand Linux authentication and administrative permissions, and manage EC2 instances using AWS Systems Manager.

The most important practical workflow is:

**EC2 → IAM Role → SSM Agent → Managed Node → Fleet Manager → Run Command → Execute Shell Script → Verify Output**

These concepts are useful for AWS Cloud Support, Cloud Engineer, and DevOps roles.