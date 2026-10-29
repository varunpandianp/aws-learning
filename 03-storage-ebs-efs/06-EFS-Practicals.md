# AWS EFS Practical: Shared Storage Across Two EC2 Instances

## 1. Objective

In this practical, I will learn how to:

1. Launch two Amazon Linux EC2 instances in different Availability Zones within the same AWS Region.
2. Create and configure security groups for EFS access.
3. Create an Amazon Elastic File System (EFS).
4. Configure EFS mount targets across Availability Zones.
5. Allow NFS traffic through TCP port `2049`.
6. Install EFS utilities on Amazon Linux.
7. Mount EFS on both EC2 instances.
8. Verify that both instances can access the same files.
9. Configure persistent mounting using `/etc/fstab`.
10. Test the mount configuration using `mount -a`.

---

# 2. What Is Amazon EFS?

**Amazon Elastic File System (EFS)** is a managed, elastic file storage service provided by AWS.

It allows multiple Linux-based EC2 instances to access the same file system over a network.

For example, if two EC2 instances mount the same EFS file system, a file created on one instance can be accessed from the other instance.

### EFS architecture

```text
                    AWS REGION
                 (Example: ap-south-1)
                          |
             +------------+------------+
             |                         |
             |                         |
       Availability Zone A       Availability Zone B
             |                         |
             v                         v
        EC2 Instance 1            EC2 Instance 2
        Amazon Linux              Amazon Linux
             |                         |
             | NFS TCP 2049            | NFS TCP 2049
             |                         |
             +------------+------------+
                          |
                          v
                    Amazon EFS
                 Shared File System
                          |
             +------------+------------+
             |                         |
             v                         v
        Mount Target A             Mount Target B
        Availability Zone A       Availability Zone B
```

Both EC2 instances access the same EFS file system through their network connections.

**Important:** EFS is regional storage. To provide access from both Availability Zones, configure an EFS mount target in each Availability Zone used by the instances.

---

# 3. EFS vs EBS vs S3

| Feature | Amazon EFS | Amazon EBS | Amazon S3 |
|---|---|---|---|
| Storage type | File storage | Block storage | Object storage |
| Access method | NFS | Block device attached to EC2 | S3 API/HTTP |
| Typical protocol | NFSv4.1 | Block device I/O | HTTPS/API |
| Default port for the service | TCP 2049 for NFS | No NFS port; block storage attachment | Usually HTTPS TCP 443 |
| Shared access | Multiple compatible clients can access the same file system | A volume is generally attached to one EC2 instance at a time; Multi-Attach is available for certain configurations | Multiple clients can access objects |
| Common use case | Shared application files, web content, shared Linux directories | OS disks, databases, application disks | Images, backups, logs, documents |
| Regional design | Regional EFS supports access across multiple AZs | A volume belongs to one AZ | Regional object storage |
| Capacity management | Elastic | Provisioned volume capacity | Elastic object storage |

### Interview point

- **EBS:** Block storage.
- **EFS:** Shared network file storage.
- **S3:** Object storage.

---

# 4. Why Does EFS Use Port 2049?

Amazon EFS uses the Network File System (NFS) protocol.

Linux clients connect to EFS using **NFS over TCP port `2049`**.

Example:

```text
EC2 Instance 1
      |
      | TCP 2049
      v
EFS Mount Target
```

For this connection to work:

1. The EFS mount target's security group must allow inbound TCP port `2049` from the EC2 clients' security group.
2. The EC2 instance must be able to send outbound traffic to the EFS mount target on TCP port `2049`.
3. Network ACLs and routing must allow the connection.
4. DNS resolution must work if you mount EFS using its DNS name.

The default EC2 security group commonly allows outbound traffic. If your security group has restrictive outbound rules, explicitly allow the required NFS traffic.

**Security best practice:** Do not expose NFS port `2049` to the entire internet. Allow access only from the required EC2 client security group or another appropriately restricted network source.

---

# 5. Practical Architecture and Naming

Use the following example configuration.

| Resource | Example name |
|---|---|
| AWS Region | `ap-south-1` |
| EC2 Instance 1 | `efs-client-az-a` |
| EC2 Instance 2 | `efs-client-az-b` |
| EC2 Security Group | `sg-efs-clients` |
| EFS Security Group | `sg-efs-mount-targets` |
| EFS File System | `efs-shared-storage-lab` |
| First mount directory | `/mnt/efs` |
| Filesystem protocol | NFS |
| NFS port | TCP `2049` |

Choose any AWS Region available to you. The names above are examples.

**Prerequisites:**

- An AWS account with permission to create EC2 instances, security groups, and EFS resources.
- An existing VPC.
- Two suitable subnets in different Availability Zones within the same Region.
- A way to connect to the EC2 instances, such as SSH or EC2 Instance Connect.
- An internet connection or suitable package repository access if installing EFS utilities requires it.

---

# PRACTICAL 1: Create Security Groups

We will use separate security groups for the EC2 clients and EFS mount targets.

This separation makes the network configuration easier to understand and troubleshoot.

## Step 1: Create the EC2 Security Group

1. Open the AWS Management Console.
2. Navigate to **EC2 → Network & Security → Security Groups**.
3. Select **Create security group**.
4. Enter:

| Setting | Value |
|---|---|
| Security group name | `sg-efs-clients` |
| Description | `Security group for EC2 EFS clients` |
| VPC | Select the VPC used by your EC2 instances |

5. Configure inbound rules appropriate for your access method.

For SSH access from your computer:

| Type | Protocol | Port | Source |
|---|---|---|---|
| SSH | TCP | 22 | Your trusted public IP address |

Do not use `0.0.0.0/0` for SSH unless you have a specific, temporary reason and understand the exposure.

6. For this lab, you may optionally add an ICMP rule if you want to test connectivity using `ping`.

Example:

| Type | Protocol | Source |
|---|---|---|
| All ICMP - IPv4 | ICMP | Your trusted lab network or another restricted source |

**ICMP is optional. It is not required for EFS mounting.**

7. Leave outbound access at its default setting for this lab, unless your environment requires restricted outbound rules.
8. Select **Create security group**.

## Step 2: Create the EFS Security Group

1. Navigate to **EC2 → Network & Security → Security Groups**.
2. Select **Create security group**.
3. Enter:

| Setting | Value |
|---|---|
| Security group name | `sg-efs-mount-targets` |
| Description | `Allow NFS from authorized EC2 clients` |
| VPC | The same VPC used by the EC2 clients |

4. Under inbound rules, select **Add rule**.

Configure:

| Setting | Value |
|---|---|
| Type | NFS |
| Protocol | TCP |
| Port range | 2049 |
| Source | `sg-efs-clients` security group |

Select the EC2 client security group as the source instead of allowing traffic from every IP address.

5. Keep the default outbound rule unless you have a specific reason to restrict it.
6. Select **Create security group**.

### Understand the security group relationship

```text
EC2 Instances
     |
     | Outbound TCP 2049
     v
EFS Mount Targets
     |
     | Inbound TCP 2049 allowed
     | from sg-efs-clients
     v
Amazon EFS
```

Security groups are stateful. Return traffic for an allowed connection is automatically permitted by the security group connection tracking behavior.

You do not need to create a separate inbound rule for the response packets.

---

# PRACTICAL 2: Launch Two EC2 Instances in Different AZs

## Step 1: Launch EC2 Instance 1

1. Open **EC2 → Instances**.
2. Select **Launch instances**.
3. Configure:

| Setting | Value |
|---|---|
| Name | `efs-client-az-a` |
| AMI | Amazon Linux 2023 or Amazon Linux 2 |
| Instance type | A suitable free-tier-eligible or low-cost type, if available to your account |
| VPC | Your selected VPC |
| Subnet | A subnet in Availability Zone A |
| Security group | `sg-efs-clients` |
| Key pair | Select an appropriate key pair if using SSH |

4. Review the configuration.
5. Select **Launch instance**.

Wait until the instance is running and its status checks have passed.

## Step 2: Launch EC2 Instance 2

Repeat the process with these changes:

| Setting | Value |
|---|---|
| Name | `efs-client-az-b` |
| AMI | Same Amazon Linux version as Instance 1 |
| VPC | Same VPC |
| Subnet | A subnet in a different Availability Zone |
| Security group | `sg-efs-clients` |

For example:

```text
Instance 1: ap-south-1a
Instance 2: ap-south-1b
```

These AZ names are illustrative. Your account may map AZ names differently, so verify the actual placement in the console.

### Verify the instances

In the EC2 console, confirm:

- Both instances are running.
- Both instances are in the same AWS Region.
- Both instances are in the same VPC.
- They are in different Availability Zones.
- Both instances use `sg-efs-clients`.

---

# PRACTICAL 3: Create the EFS File System

## Step 1: Open EFS

1. Open the AWS Management Console.
2. Search for **EFS**.
3. Open **Amazon Elastic File System**.
4. Select **Create file system**.

## Step 2: Configure the File System

Depending on the console interface, you may see a quick-create option or a customize option.

For this lab, choose the customization path if available.

Example configuration:

| Setting | Value |
|---|---|
| Name | `efs-shared-storage-lab` |
| VPC | The same VPC used by the EC2 instances |
| Availability | Regional |
| Encryption at rest | Enable it where appropriate |
| Performance mode | General Purpose |
| Throughput mode | Elastic, or another suitable option available to you |

Regional EFS is appropriate for this lab because the file system will be accessed from two Availability Zones.

5. Review the settings.
6. Create the file system.

Wait until the EFS file system becomes available.

### Why choose Regional EFS?

Regional EFS is designed to provide shared file storage across multiple Availability Zones within a Region.

It supports multiple mount targets so that clients in different AZs can access the same file system.

---

# PRACTICAL 4: Configure EFS Mount Targets and Security Groups

Creating an EFS file system does not necessarily mean that every subnet or Availability Zone is ready for access.

You must configure mount targets and their security groups.

## Step 1: Open the EFS Network Settings

1. Open **Amazon EFS**.
2. Select `efs-shared-storage-lab`.
3. Open the **Network** or **Network access** section, depending on the console layout.

Review the mount targets and Availability Zones.

## Step 2: Configure the First Mount Target

For the first Availability Zone:

1. Create or edit the mount target for the subnet in AZ A.
2. Select a subnet in the same VPC as Instance 1.
3. Assign the EFS security group:

```text
sg-efs-mount-targets
```

## Step 3: Configure the Second Mount Target

For the second Availability Zone:

1. Create or edit the mount target for the subnet in AZ B.
2. Select a subnet in the same VPC as Instance 2.
3. Assign the same EFS security group:

```text
sg-efs-mount-targets
```

Ensure that each mount target is in a different Availability Zone.

Wait until the mount targets become available.

### Important security group step

If the EFS mount targets currently use the default security group, replace it with `sg-efs-mount-targets` if that is your intended configuration.

Before removing the default security group, ensure that the new security group is attached and correctly configured.

The EFS security group must allow:

```text
Inbound:
NFS
TCP
Port 2049
Source: sg-efs-clients
```

### Checkpoint

You should now have:

```text
EFS File System
     |
     +---- Mount Target in AZ A
     |           |
     |           +---- sg-efs-mount-targets
     |
     +---- Mount Target in AZ B
                 |
                 +---- sg-efs-mount-targets
```

Both EC2 instances should use `sg-efs-clients`.

---

# PRACTICAL 5: Install EFS Utilities on Amazon Linux

Amazon Linux needs the EFS mount helper to use the standard EFS mount commands and options.

The package is called:

```text
amazon-efs-utils
```

Connect to each EC2 instance and install the package.

## Amazon Linux 2023

Run:

```bash
sudo dnf install -y amazon-efs-utils
```

## Amazon Linux 2

Run:

```bash
sudo yum install -y amazon-efs-utils
```

If the package is unavailable, check the enabled repositories and package availability for your AMI.

Verify that the mount helper is installed:

```bash
mount.efs --version
```

If that command is unavailable in your shell environment, check the installed package:

```bash
rpm -q amazon-efs-utils
```

Repeat the installation on both instances.

### Why install EFS utilities?

The package provides the EFS mount helper, which supports mounting EFS using its file system ID or DNS name and options such as TLS encryption in transit.

---

# PRACTICAL 6: Mount EFS on EC2 Instance 1

## Step 1: Open the EFS Attach Instructions

1. Open the EFS console.
2. Select `efs-shared-storage-lab`.
3. Select **Attach**.
4. Review the available mount instructions.

The console may provide DNS-based and mount-helper commands.

Use the file system ID and DNS name shown for your actual EFS file system.

Example file system ID:

```text
fs-0123456789abcdef0
```

This is only an example. Do not use it literally.

## Step 2: Create the Mount Directory

Connect to Instance 1 and run:

```bash
sudo mkdir -p /mnt/efs
```

## Step 3: Mount EFS Using the Mount Helper

Use the following command with your actual file system ID:

```bash
sudo mount -t efs -o tls fs-0123456789abcdef0:/ /mnt/efs
```

Replace the example ID with your real EFS file system ID.

The `tls` option enables encryption in transit using the EFS mount helper.

Alternatively, if you prefer the DNS-based command shown in the AWS console, use the actual DNS name provided there.

## Step 4: Verify the Mount

Run:

```bash
df -hT
```

Then:

```bash
findmnt /mnt/efs
```

You can also run:

```bash
mount | grep efs
```

The mount should show the EFS filesystem associated with `/mnt/efs`.

### Checkpoint

EFS is now mounted on Instance 1.

---

# PRACTICAL 7: Mount the Same EFS on EC2 Instance 2

Connect to Instance 2.

## Step 1: Create the Mount Directory

```bash
sudo mkdir -p /mnt/efs
```

## Step 2: Mount the Same EFS File System

Use the same EFS file system ID:

```bash
sudo mount -t efs -o tls fs-0123456789abcdef0:/ /mnt/efs
```

Replace the example ID with your actual EFS ID.

**Do not create another EFS file system for Instance 2.** Both instances must mount the same file system to share files.

## Step 3: Verify the Mount

```bash
df -hT
```

```bash
findmnt /mnt/efs
```

At this point:

```text
Instance 1 ----+
               |
               +---- Same EFS File System
               |
Instance 2 ----+
```

Both instances can now access the same EFS filesystem, subject to permissions and network configuration.

---

# PRACTICAL 8: Verify Shared Storage

This is the most important test in the lab.

## Step 1: Create a File on Instance 1

Connect to Instance 1 and run:

```bash
echo "Hello from EC2 Instance 1" | sudo tee /mnt/efs/shared-file.txt
```

Verify the file:

```bash
cat /mnt/efs/shared-file.txt
```

Expected output:

```text
Hello from EC2 Instance 1
```

## Step 2: Read the File on Instance 2

Connect to Instance 2 and run:

```bash
cat /mnt/efs/shared-file.txt
```

Expected output:

```text
Hello from EC2 Instance 1
```

Instance 2 can read the file because both instances mount the same EFS file system.

## Step 3: Create a File on Instance 2

On Instance 2, run:

```bash
echo "Hello from EC2 Instance 2" | sudo tee /mnt/efs/instance2-file.txt
```

## Step 4: Read the File on Instance 1

On Instance 1, run:

```bash
cat /mnt/efs/instance2-file.txt
```

Expected output:

```text
Hello from EC2 Instance 2
```

### Practical result

```text
Instance 1:
  Creates shared-file.txt
          |
          v
       Amazon EFS
          |
          v
Instance 2:
  Reads shared-file.txt
```

**Success criteria:** A file created on either instance is visible from the other instance when both are accessing the same EFS filesystem.

---

# PRACTICAL 9: Configure Permanent EFS Mounting

A manual mount may not survive a reboot.

To mount EFS automatically at boot, add an entry to `/etc/fstab`.

We will use the EFS mount helper and enable TLS.

## Step 1: Verify the EFS File System ID

In the EFS console, open your file system and copy the actual file system ID.

Example:

```text
fs-0123456789abcdef0
```

Use your own ID.

## Step 2: Back Up the Existing fstab File

On each EC2 instance, run:

```bash
sudo cp /etc/fstab /etc/fstab.backup
```

## Step 3: Edit fstab

Open the file:

```bash
sudo vi /etc/fstab
```

Add this line, replacing the example ID with your actual EFS ID:

```text
fs-0123456789abcdef0:/ /mnt/efs efs _netdev,tls 0 0
```

### Explanation

| Field | Meaning |
|---|---|
| `fs-0123456789abcdef0:/` | EFS file system ID and root path |
| `/mnt/efs` | Local mount point |
| `efs` | Filesystem type handled by the EFS mount helper |
| `_netdev,tls` | Indicates network storage and enables encryption in transit |
| `0` | Do not use the traditional dump utility for this filesystem |
| `0` | Do not run the traditional boot-time filesystem check for this network filesystem |

The example ID is not a real file system ID. Replace it before saving.

Save and exit the editor.

## Step 4: Test the fstab Configuration

First, validate the mount configuration:

```bash
sudo findmnt --verify --verbose
```

Then test mounting entries in `/etc/fstab`:

```bash
sudo mount -a
```

If the command returns without an error, verify the mount:

```bash
findmnt /mnt/efs
```

Also run:

```bash
df -hT
```

Repeat this configuration on Instance 2.

### Important

- Create `/mnt/efs` before testing the mount.
- Use the correct EFS file system ID.
- Ensure the mount helper is installed.
- Confirm the EFS security group allows TCP port `2049` from the EC2 clients.
- Confirm the EFS mount target exists in the required AZ.
- Do not add duplicate entries for the same mount to `/etc/fstab`.

The `_netdev` option identifies the mount as network-dependent. It helps the operating system handle the network mount appropriately during startup.

---

# PRACTICAL 10: Reboot and Verify Persistent Mounting

This optional test confirms that EFS is configured for automatic mounting.

Before rebooting, verify:

```bash
sudo mount -a
```

```bash
findmnt /mnt/efs
```

```bash
df -hT
```

After verifying that there are no mount errors, reboot one instance at a time:

```bash
sudo reboot
```

Reconnect after the instance becomes available.

Check the mount:

```bash
findmnt /mnt/efs
```

```bash
df -hT
```

Verify the shared file:

```bash
cat /mnt/efs/shared-file.txt
```

Repeat for the second instance if required.

**Note:** Automatic mounting can fail if network connectivity, DNS resolution, security group rules, mount targets, or EFS utilities are unavailable. Validate these dependencies when troubleshooting startup issues.

---

# PRACTICAL 11: Troubleshooting EFS

## Problem 1: Mount Command Hangs or Times Out

Check the EFS security group.

Required inbound rule:

```text
Type: NFS
Protocol: TCP
Port: 2049
Source: sg-efs-clients
```

Also check:

- Both instances and EFS use the intended VPC.
- The EFS mount target exists and is available.
- The mount target's security group allows NFS.
- The EC2 security group permits the required outbound traffic.
- Network ACLs and routes allow the connection.
- The EFS DNS name resolves correctly.

## Problem 2: Connection Timed Out

A timeout commonly indicates a network connectivity issue.

Check DNS resolution:

```bash
getent hosts fs-0123456789abcdef0.efs.ap-south-1.amazonaws.com
```

Replace the example file system ID and Region with your actual values.

Test TCP port 2049 if `nc` is installed:

```bash
nc -zv fs-0123456789abcdef0.efs.ap-south-1.amazonaws.com 2049
```

If `nc` is not installed, use available network diagnostic tools or inspect the security group and network configuration in the AWS console.

## Problem 3: DNS Resolution Fails

Check:

- The EFS file system ID and DNS name.
- The Region in the DNS name.
- VPC DNS resolution and DNS hostnames settings.
- Whether the EFS mount target has become available.
- Whether the correct VPC and mount target configuration are being used.

## Problem 4: Permission Denied When Creating Files

Check the directory permissions:

```bash
ls -ld /mnt/efs
```

Check the current user:

```bash
whoami
```

For this lab, you can create test files with `sudo`:

```bash
sudo touch /mnt/efs/test-file.txt
```

In production, configure appropriate Linux ownership, permissions, and access controls rather than relying on `sudo` for all application writes.

For example, use a dedicated application user and group where appropriate.

## Problem 5: EFS Does Not Mount After Reboot

Check the `/etc/fstab` entry:

```bash
cat /etc/fstab
```

Validate the configuration:

```bash
sudo findmnt --verify --verbose
```

Test:

```bash
sudo mount -a
```

Inspect logs if the mount fails:

```bash
sudo journalctl -b --no-pager | grep -iE 'efs|mount.nfs|mount'
```

Also check that `amazon-efs-utils` is installed and the network is available.

---

# PRACTICAL 12: EFS Security Group Flow

Understand the difference between the EC2 client security group and the EFS mount target security group.

```text
+-------------------------------+
| EC2 Instance 1                |
| Security Group: sg-efs-clients|
+-------------------------------+
               |
               | NFS TCP 2049
               v
+-------------------------------+
| EFS Mount Target in AZ A      |
| SG: sg-efs-mount-targets      |
| Inbound TCP 2049              |
| Source: sg-efs-clients        |
+-------------------------------+

+-------------------------------+
| EC2 Instance 2                |
| Security Group: sg-efs-clients|
+-------------------------------+
               |
               | NFS TCP 2049
               v
+-------------------------------+
| EFS Mount Target in AZ B      |
| SG: sg-efs-mount-targets      |
| Inbound TCP 2049              |
| Source: sg-efs-clients        |
+-------------------------------+
```

### Important distinction

**Port 2049 is required for NFS. ICMP is not required to mount EFS.**

ICMP may be enabled temporarily for troubleshooting using `ping`, but a successful ping does not prove that TCP port 2049 is open. A failed ping does not necessarily mean EFS is unreachable.

---

# Interview Questions and Answers

## 1. What is Amazon EFS?

Amazon EFS is a managed, elastic file storage service that allows multiple compatible clients to access a shared filesystem over NFS.

## 2. Which port does EFS use?

EFS uses NFS over TCP port `2049`.

## 3. Why do we use EFS instead of EBS?

EFS is suitable when multiple instances need access to the same shared files. EBS is block storage typically used for instance disks, databases, and application storage.

## 4. Can two EC2 instances in different Availability Zones access the same EFS filesystem?

Yes. A Regional EFS filesystem can be accessed from multiple Availability Zones when appropriate mount targets and network connectivity are configured.

## 5. What is an EFS mount target?

A mount target provides network access to an EFS filesystem from a VPC. It has network interfaces and an associated security group.

## 6. Why configure a mount target in each Availability Zone?

A mount target in each required AZ provides a local connection endpoint for clients in that AZ and supports resilient access across AZs.

## 7. Which security group rule is required for EFS?

The EFS mount target's security group must allow inbound NFS traffic on TCP port `2049` from authorized clients, commonly by referencing the EC2 client security group.

## 8. Do we need ICMP to mount EFS?

No. ICMP is optional and is mainly useful for certain connectivity diagnostics. NFS TCP port `2049` is the important EFS network rule.

## 9. Why install amazon-efs-utils?

It provides the EFS mount helper and supports EFS-specific mounting options, including TLS encryption in transit.

## 10. How do you mount EFS on Linux?

For example:

```bash
sudo mkdir -p /mnt/efs
sudo mount -t efs -o tls fs-0123456789abcdef0:/ /mnt/efs
```

Replace the example file system ID with the actual ID.

## 11. How do you mount EFS permanently?

Add an appropriate entry to `/etc/fstab`, such as:

```text
fs-0123456789abcdef0:/ /mnt/efs efs _netdev,tls 0 0
```

Then validate and test:

```bash
sudo findmnt --verify --verbose
sudo mount -a
```

## 12. How can you prove that EFS is shared?

Create a file on Instance 1 and read it from Instance 2 using the same mounted EFS directory.

## 13. What does `_netdev` mean?

It indicates that the filesystem depends on network connectivity and should be treated as a network filesystem during system startup.

## 14. What happens if you mount different EFS filesystems on the two instances?

The instances will not automatically share files. Both must mount the same EFS filesystem to access the same data.

## 15. Is EFS the same as S3?

No. EFS is shared file storage accessed through NFS. S3 is object storage accessed through APIs and HTTP-based interfaces.

---

# Practical Checklist

- [ ] Create `sg-efs-clients`.
- [ ] Add appropriate SSH access to the EC2 client security group.
- [ ] Optionally add restricted ICMP rules for troubleshooting.
- [ ] Create `sg-efs-mount-targets`.
- [ ] Add inbound NFS TCP port `2049` from `sg-efs-clients`.
- [ ] Launch Instance 1 in AZ A.
- [ ] Launch Instance 2 in AZ B.
- [ ] Confirm both instances are in the same VPC and Region.
- [ ] Create a Regional EFS filesystem.
- [ ] Configure mount targets in both AZs.
- [ ] Attach `sg-efs-mount-targets` to the EFS mount targets.
- [ ] Install `amazon-efs-utils` on both instances.
- [ ] Create `/mnt/efs` on both instances.
- [ ] Mount the same EFS filesystem on both instances.
- [ ] Verify mounts with `findmnt` and `df -hT`.
- [ ] Create a file on Instance 1.
- [ ] Read the file from Instance 2.
- [ ] Create an `/etc/fstab` entry on both instances.
- [ ] Run `sudo findmnt --verify --verbose`.
- [ ] Run `sudo mount -a`.
- [ ] Verify the mount and shared data.
- [ ] Optionally reboot each instance and verify automatic mounting.
- [ ] Clean up lab resources when finished.

---

# GitHub Learning Log

## Topic: Amazon EFS — Shared Storage Across Multiple EC2 Instances

### What I Learned

- EFS is a managed, elastic file storage service.
- EFS uses NFS over TCP port `2049`.
- Multiple EC2 instances can access the same EFS filesystem.
- EFS mount targets provide network access to the filesystem.
- Mount targets and security groups must be configured correctly.
- A Regional EFS filesystem can support clients in different AZs.
- The EFS mount helper simplifies mounting and supports TLS.
- `/etc/fstab` can configure persistent EFS mounts.
- `mount -a` tests mounting entries configured in `/etc/fstab`.
- Shared storage can be verified by creating a file on one EC2 instance and reading it from another.

### Commands Practiced

```bash
# Install EFS utilities on Amazon Linux 2023
sudo dnf install -y amazon-efs-utils

# Install EFS utilities on Amazon Linux 2
sudo yum install -y amazon-efs-utils

# Create a mount directory
sudo mkdir -p /mnt/efs

# Mount EFS using the EFS mount helper
sudo mount -t efs -o tls fs-0123456789abcdef0:/ /mnt/efs

# Verify mounted filesystems
df -hT
findmnt /mnt/efs

# Create a shared test file
echo "Hello from EC2 Instance 1" | sudo tee /mnt/efs/shared-file.txt

# Read the shared file
cat /mnt/efs/shared-file.txt

# Validate fstab
sudo findmnt --verify --verbose

# Test persistent mount configuration
sudo mount -a

# Check EFS package
rpm -q amazon-efs-utils
```

**Note:** Replace the example EFS file system ID with your real ID. Run the appropriate package installation command for your Amazon Linux version.

```

---

# Cleanup: Avoid Unnecessary AWS Charges

After completing the lab:

1. Unmount EFS from each EC2 instance if you no longer need it:

   ```bash
   sudo umount /mnt/efs
   ```

2. If you added an EFS entry to `/etc/fstab`, remove it or comment it out before deleting the EFS filesystem.
3. Terminate the two test EC2 instances if they are no longer required.
4. Delete the EFS filesystem if you no longer need its data.
5. Verify that the EFS filesystem and mount targets have been removed.
6. Delete unused security groups after checking that no other resources depend on them.

**Remember:** EFS storage and related resources can incur charges even when your EC2 instances are stopped. Review the AWS pricing for your Region and clean up resources you no longer need.