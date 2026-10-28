# AWS Storage Interview Questions and Answers
## Amazon EBS and Amazon EFS — Beginner to Intermediate

**Learning Topics:** AWS EBS, EBS Volume Types, EBS Snapshots, Volume Restoration, Linux Mounting, Persistent Mounting, Volume Resizing, Amazon EFS, NFS, Security Groups, and Shared Storage.

---

# Table of Contents

1. [AWS Storage Fundamentals](#1-aws-storage-fundamentals)
2. [Amazon EBS Fundamentals](#2-amazon-ebs-fundamentals)
3. [EBS Volume Types](#3-ebs-volume-types)
4. [EBS Volume Practicals](#4-ebs-volume-practicals)
5. [EBS Snapshots and Restoration](#5-ebs-snapshots-and-restoration)
6. [EBS Mounting and Linux Commands](#6-ebs-mounting-and-linux-commands)
7. [EBS Resizing and Persistent Mounting](#7-ebs-resizing-and-persistent-mounting)
8. [Amazon EFS Fundamentals](#8-amazon-efs-fundamentals)
9. [EFS Networking and Security Groups](#9-efs-networking-and-security-groups)
10. [EFS Mounting and Persistence](#10-efs-mounting-and-persistence)
11. [EBS vs EFS](#11-ebs-vs-efs)
12. [Scenario-Based Interview Questions](#12-scenario-based-interview-questions)
13. [Troubleshooting Questions](#13-troubleshooting-questions)
14. [Practical Interview Tasks](#14-practical-interview-tasks)
15. [Quick Revision](#15-quick-revision)

---

# 1. AWS Storage Fundamentals

### Q1. What is cloud storage?

Cloud storage is a service that allows users to store and retrieve data using cloud infrastructure instead of managing all the physical storage hardware themselves.

AWS offers several storage services, including EBS, EFS, and S3.

### Q2. What are the main storage services in AWS?

The three commonly discussed services are:

- **Amazon EBS:** Block storage for EC2 instances.
- **Amazon EFS:** Shared, elastic file storage for multiple clients.
- **Amazon S3:** Object storage for files and other objects.

### Q3. What is the difference between block, file, and object storage?

**Block storage:** Data is stored in blocks and presented as a block device. EBS is an example.

**File storage:** Data is organized into files and directories. EFS is an example.

**Object storage:** Data is stored as objects in buckets. S3 is an example.

### Q4. What is persistent storage?

Persistent storage retains data independently of the normal lifecycle of an application or compute process.

EBS volumes and EFS filesystems can persist beyond an EC2 instance reboot. EBS data retention after instance termination also depends on the volume's `DeleteOnTermination` setting.

### Q5. What is the difference between temporary and persistent storage?

Temporary storage is intended for data that can be recreated and may not survive the underlying resource's lifecycle.

Persistent storage is intended to retain data beyond a process or instance reboot.

For example, EBS is persistent block storage, while EC2 instance store provides temporary storage tied to the instance's lifecycle.

---

# 2. Amazon EBS Fundamentals

### Q6. What is Amazon EBS?

Amazon Elastic Block Store (EBS) provides block storage volumes for use with EC2 instances.

EBS volumes are commonly used for:

- Operating system disks.
- Application data.
- Databases.
- Logs and files requiring block storage.
- Persistent storage for applications.

### Q7. Is EBS a file storage service?

No. EBS provides block storage.

A Linux instance can create a filesystem, such as XFS or ext4, on an EBS volume and then mount that filesystem to a directory.

### Q8. What is an EBS volume?

An EBS volume is a virtual block device that can be attached to a compatible EC2 instance.

After attaching a blank volume, you may need to create a filesystem, mount it, and configure permissions before using it.

### Q9. Does an EBS volume survive an EC2 reboot?

Yes. An EBS volume and its data normally persist when the attached EC2 instance is rebooted.

The filesystem may need to be mounted automatically through `/etc/fstab` or another system configuration.

### Q10. Does an EBS volume survive EC2 instance termination?

It depends on the volume's `DeleteOnTermination` setting.

- If enabled, the volume is deleted when the instance terminates.
- If disabled, the volume is retained.

Always verify this setting before terminating an instance containing important data.

### Q11. Can one EBS volume be attached to multiple EC2 instances?

Normally, an EBS volume is attached to one instance at a time.

However, Amazon EBS Multi-Attach supports simultaneous attachment of certain Provisioned IOPS SSD volumes to multiple Nitro-based instances within the same Availability Zone, subject to service limitations and application/filesystem requirements.

A standard filesystem such as ext4 should not simply be mounted read-write from multiple instances concurrently. Shared access requires an appropriate cluster-aware filesystem or application design.

### Q12. What is an Availability Zone in relation to EBS?

An Availability Zone (AZ) is an isolated location within an AWS Region.

An EBS volume belongs to a particular AZ and normally can only be attached to an EC2 instance in that same AZ.

To use its data in another AZ, you can create a snapshot and create a new volume from that snapshot in the target AZ.

### Q13. What happens if an EC2 instance fails but its EBS volume is retained?

You can generally detach the retained volume from the failed or replaced instance, where applicable, and attach it to another compatible instance in the same AZ.

You must still ensure the filesystem and application are recovered safely.

### Q14. What is the difference between an EC2 root volume and a data volume?

The **root volume** contains the operating system and boot files.

A **data volume** provides additional storage for application files, databases, logs, or other data.

Separating data from the root volume can make backups, recovery, and storage management easier.

---

# 3. EBS Volume Types

### Q15. What are the major EBS volume types?

The main modern volume types are:

| Volume type | Category | Common use case |
|---|---|---|
| gp3 | General Purpose SSD | General workloads, application servers, development environments |
| gp2 | General Purpose SSD | Older general-purpose workloads |
| io2 Block Express | Provisioned IOPS SSD | Demanding, I/O-intensive workloads |
| io1 | Provisioned IOPS SSD | I/O-intensive workloads |
| st1 | Throughput Optimized HDD | Large sequential data workloads |
| sc1 | Cold HDD | Infrequently accessed sequential data |

The older `standard` magnetic volume type is a legacy option.

Always verify the current limits and availability of a volume type in the AWS Region you are using.

### Q16. What is gp3?

gp3 is a general-purpose SSD volume type.

It is suitable for many everyday workloads, including application servers, boot volumes, and development environments.

Its IOPS and throughput can be provisioned independently of volume size within the supported limits.

### Q17. What is the difference between gp2 and gp3?

Both are general-purpose SSD volume types.

The main difference is that gp3 allows IOPS and throughput to be provisioned independently of volume size, while gp2 performance is tied more closely to volume size and its burst-credit model.

For many workloads, gp3 is a useful default to evaluate.

### Q18. When would you choose io2?

Choose io2 for workloads requiring high IOPS, consistent storage performance, and high durability, subject to the workload's requirements and supported configuration.

Examples may include demanding transactional databases and other I/O-intensive applications.

### Q19. When would you choose st1?

st1 is a throughput-oriented HDD volume designed for large, sequential data operations.

It may be suitable for workloads such as large data processing jobs or sequential log processing.

It is not intended to be a boot volume.

### Q20. When would you choose sc1?

sc1 is designed for infrequently accessed data that requires lower-cost, throughput-oriented HDD storage.

It is not intended for boot volumes or latency-sensitive workloads.

### Q21. How do you choose an EBS volume type?

Consider:

1. Workload type.
2. Required IOPS.
3. Required throughput.
4. Latency sensitivity.
5. Capacity.
6. Cost.
7. Availability and supported performance in your Region.

For many general-purpose applications, evaluate gp3 first. For demanding I/O-intensive workloads, evaluate io2. For large sequential workloads, consider st1 or sc1.

---

# 4. EBS Volume Practicals

### Q22. How do you create an EBS volume?

1. Open the EC2 console.
2. Navigate to **Elastic Block Store → Volumes**.
3. Select **Create volume**.
4. Select the volume type, size, and Availability Zone.
5. Create the volume.
6. Wait until it becomes `Available`.

### Q23. How do you attach an EBS volume to an EC2 instance?

1. Select the EBS volume.
2. Choose **Actions → Attach volume**.
3. Select the target EC2 instance in the same AZ.
4. Specify a device name if required.
5. Attach the volume.

### Q24. How do you identify an attached disk in Linux?

Run:

```bash
lsblk
```

To inspect filesystems and UUIDs:

```bash
lsblk -f
```

You can also run:

```bash
sudo blkid
```

### Q25. Why might the device name differ from the name specified in AWS?

On Nitro-based instances, EBS volumes commonly appear as NVMe devices.

For example, the requested name `/dev/sdf` may appear inside Linux as `/dev/nvme1n1`.

Always identify the actual device before formatting or mounting it.

### Q26. How do you create an XFS filesystem on a new EBS volume?

First, identify the correct disk:

```bash
lsblk -f
```

If it is a confirmed blank volume, create the filesystem:

```bash
sudo mkfs.xfs /dev/nvme1n1
```

**Warning:** Formatting destroys existing filesystem data. Never run `mkfs` on a volume containing data you need to preserve.

Replace the example device name with the actual device name.

### Q27. How do you mount an EBS volume?

Create a directory:

```bash
sudo mkdir -p /data/ebs
```

Mount the volume:

```bash
sudo mount /dev/nvme1n1 /data/ebs
```

Verify:

```bash
df -hT
```

This example assumes that the filesystem exists directly on `/dev/nvme1n1`. If it exists on a partition, mount the correct partition instead.

### Q28. How do you verify the EBS filesystem is mounted?

Use:

```bash
findmnt
```

Or:

```bash
findmnt /data/ebs
```

You can also use:

```bash
df -hT
```

### Q29. What happens if you detach an EBS volume while an application is actively writing to it?

Detaching a volume without preparing the operating system and application can cause data corruption or application errors.

Before detaching a volume:

1. Stop or quiesce applications using it.
2. Flush pending writes where appropriate.
3. Unmount the filesystem.
4. Detach the volume through AWS.

Do not forcibly detach a volume unless you understand the recovery implications.

---

# 5. EBS Snapshots and Restoration

### Q30. What is an EBS snapshot?

An EBS snapshot is a point-in-time backup of an EBS volume.

It can be used to create new EBS volumes containing the data captured by the snapshot.

### Q31. Why do we create EBS snapshots?

Common reasons include:

- Data backup.
- Disaster recovery.
- Restoring accidentally deleted files.
- Creating test or development copies.
- Migrating storage between Availability Zones.
- Creating recovery points before risky changes.

### Q32. How do you create an EBS snapshot?

1. Open **EC2 → Elastic Block Store → Volumes**.
2. Select the source volume.
3. Choose **Actions → Create snapshot**.
4. Enter a description and tags.
5. Create the snapshot.
6. Monitor its status until it becomes `Completed`.

### Q33. How do you create a new EBS volume from a snapshot?

1. Open **EC2 → Elastic Block Store → Snapshots**.
2. Select the snapshot.
3. Choose **Actions → Create volume from snapshot**.
4. Choose the volume type, size, and target AZ.
5. Create the volume.
6. Attach it to an EC2 instance in the same AZ.
7. Mount the filesystem and verify the restored data.

### Q34. Does creating a volume from a snapshot modify the original volume?

No. The restored volume is a separate volume.

Changes made to the restored volume do not automatically appear on the original volume, and changes made to the original after the snapshot are not automatically reflected in the restored volume.

### Q35. Can you create multiple EBS volumes from one snapshot?

Yes. You can create multiple volumes from the same snapshot, subject to AWS service limits and permissions.

### Q36. Are EBS snapshots full or incremental?

EBS snapshots use incremental storage.

The first snapshot stores the data required for that recovery point. Subsequent snapshots store changed blocks rather than storing every block again.

AWS manages the underlying snapshot data.

### Q37. Can you restore an EBS snapshot in another Availability Zone?

Yes.

Create a new volume from the snapshot and select an Availability Zone where you want to use the volume.

The new volume must be in the same AZ as the EC2 instance to which you intend to attach it.

### Q38. Can you restore an EBS snapshot in another AWS Region?

Yes. Copy the snapshot to the destination Region, then create a volume from the copied snapshot in an AZ in that Region.

Check encryption, KMS permissions, service quotas, and regional availability as needed.

### Q39. What happens if you delete the original EBS volume?

Deleting the original EBS volume does not automatically delete its snapshots.

You can use an appropriate retained snapshot to create a replacement volume, provided the snapshot and its encryption keys remain available.

### Q40. Does a snapshot guarantee application-consistent backups?

No.

A snapshot provides a point-in-time storage backup, but a database or multi-volume application may require additional preparation to ensure application consistency.

Use application-aware backup procedures where necessary.

---

# 6. EBS Mounting and Linux Commands

### Q41. What is a mount point?

A mount point is a directory where a filesystem becomes accessible.

For example:

```text
/data/ebs
```

Once the EBS filesystem is mounted there, its files can be accessed through that directory.

### Q42. What is the difference between a disk and a filesystem?

A disk is a block storage device.

A filesystem organizes data into files and directories on a storage device.

An empty EBS volume may need a filesystem before it can be mounted as a normal Linux filesystem.

### Q43. What is the purpose of `/etc/fstab`?

`/etc/fstab` contains filesystem mount configuration used by Linux during startup and by commands such as `mount -a`.

It can configure an EBS filesystem to mount automatically after a reboot.

### Q44. Why should we use a UUID in `/etc/fstab`?

Device names such as `/dev/nvme1n1` can change between boots or attachment events.

Using the filesystem UUID helps Linux identify the filesystem more reliably.

Find the UUID with:

```bash
sudo blkid
```

### Q45. How do you configure a persistent EBS mount?

First, find the filesystem UUID:

```bash
sudo blkid
```

Back up the configuration:

```bash
sudo cp /etc/fstab /etc/fstab.backup
```

Edit:

```bash
sudo vi /etc/fstab
```

For an XFS filesystem, an example entry is:

```text
UUID=YOUR-FILESYSTEM-UUID /data/ebs xfs defaults,nofail 0 2
```

Replace the example UUID with the actual value.

Validate:

```bash
sudo findmnt --verify --verbose
```

Test:

```bash
sudo mount -a
```

Verify:

```bash
findmnt /data/ebs
```

### Q46. What does `mount -a` do?

`mount -a` attempts to mount filesystems configured in `/etc/fstab`, except those excluded by the relevant options.

It is commonly used to test new persistent mount entries without rebooting.

### Q47. What does `nofail` mean in an EBS fstab entry?

`nofail` allows boot to continue if that filesystem is unavailable.

However, applications that depend on the mount may still fail if the volume is unavailable. Application startup dependencies must be configured appropriately.

### Q48. What is the difference between `resize2fs` and `xfs_growfs`?

- `resize2fs` is used to resize ext2/ext3/ext4 filesystems.
- `xfs_growfs` is used to grow XFS filesystems.

The correct procedure depends on the filesystem and whether it is on a whole disk or a partition.

### Q49. How do you resize an EBS volume?

The general process is:

1. Modify the EBS volume size in AWS.
2. Wait for the modification to reach an appropriate state.
3. Confirm that Linux detects the new block-device size.
4. If necessary, expand the partition.
5. Grow the filesystem.
6. Verify the final size.

For example, for a whole-disk XFS filesystem mounted at `/data/ebs`:

```bash
sudo xfs_growfs /data/ebs
```

For an ext4 filesystem directly on a whole disk:

```bash
sudo resize2fs /dev/nvme1n1
```

These are examples, not universal commands. If the filesystem is on a partition, grow the partition first and use the correct partition device where required.

---

# 7. EBS Resizing and Persistent Mounting

### Q50. Can you reduce an EBS volume's size after increasing it?

Generally, EBS volume modification supports increasing size, not directly shrinking an existing volume.

To use a smaller volume, you normally create a new smaller volume, copy or restore only the required data, and migrate the application carefully.

### Q51. Why might `df -h` still show the old size after increasing an EBS volume?

Increasing the EBS volume size does not always automatically expand the filesystem.

You may need to:

1. Confirm the block device's new size.
2. Expand the partition if present.
3. Grow the filesystem using the appropriate tool.

### Q52. What is the difference between `lsblk` and `df -h`?

- `lsblk` displays block devices, partitions, and their relationships.
- `df -h` displays space usage for mounted filesystems.

Use both to troubleshoot storage problems.

### Q53. Why is an EBS filesystem not mounted after a reboot?

Possible causes include:

- No persistent mount entry exists.
- `/etc/fstab` contains an incorrect UUID.
- The filesystem type is incorrect.
- The volume was detached or deleted.
- The mount configuration contains an error.

Check:

```bash
cat /etc/fstab
```

```bash
lsblk -f
```

```bash
sudo findmnt --verify --verbose
```

### Q54. What should you do before formatting an EBS volume?

1. Identify the device with `lsblk -f`.
2. Confirm that it is the intended volume.
3. Check whether it contains an existing filesystem or data.
4. Confirm that the volume is supposed to be initialized as a new, empty disk.
5. Back up any important data before destructive operations.

Never format a volume just because it fails to mount.

---

# 8. Amazon EFS Fundamentals

### Q55. What is Amazon EFS?

Amazon Elastic File System is a managed, elastic file storage service that allows multiple compatible clients to access a shared filesystem over NFS.

### Q56. Why do we use EFS?

EFS is useful when multiple EC2 instances or other supported clients need access to shared files.

Common use cases include:

- Shared website content.
- Application uploads.
- Shared Linux directories.
- Content management systems.
- Shared development data.
- Applications running across multiple Availability Zones.

### Q57. What is the difference between EFS and EBS?

EBS provides block storage, while EFS provides shared file storage.

An EBS volume is normally attached to one EC2 instance at a time, whereas multiple clients can mount and access the same EFS filesystem concurrently.

### Q58. Can multiple EC2 instances in different Availability Zones use the same EFS filesystem?

Yes. A Regional EFS filesystem can be accessed from multiple AZs when the appropriate mount targets and network access are configured.

### Q59. What is an EFS mount target?

An EFS mount target provides a network endpoint through which clients in a VPC access an EFS filesystem.

Mount targets have network interfaces and security groups.

### Q60. What is the difference between a Regional EFS filesystem and a One Zone EFS filesystem?

**Regional EFS** stores data redundantly across multiple AZs in a Region and is designed for resilient regional access.

**One Zone EFS** stores data in a single AZ and is suitable for workloads that can accept the associated availability and resilience trade-offs.

Choose based on availability, resilience, cost, and application requirements.

### Q61. Does EFS automatically share files between instances?

EFS provides the shared filesystem, but each client must be configured to access it.

The instances must mount the same EFS filesystem and have appropriate network connectivity and permissions.

---

# 9. EFS Networking and Security Groups

### Q62. Which port does EFS use?

EFS uses NFS over **TCP port `2049`**.

### Q63. Why is port 2049 required?

Port 2049 is used for NFS communication between an EFS client and the EFS mount target.

If the connection is blocked by a security group, network ACL, or other network control, the client may be unable to mount EFS.

### Q64. What inbound rule should be configured on the EFS security group?

An example rule is:

| Type | Protocol | Port | Source |
|---|---|---|---|
| NFS | TCP | 2049 | EC2 client security group |

Use the actual client security group from your environment.

Avoid allowing NFS access from unrestricted public sources.

### Q65. Do you need to open port 2049 in the EC2 security group's inbound rules?

Not necessarily.

For a normal EFS mount, the EC2 client initiates the connection to the EFS mount target. The important inbound rule is on the EFS mount target security group.

If the EC2 security group has restrictive outbound rules, allow outbound TCP `2049` to the EFS mount target security group or the appropriate destination.

### Q66. Is ICMP required for EFS?

No. ICMP is not required to mount EFS.

It may be useful for optional diagnostics, but a successful ping does not prove that NFS is accessible. Test TCP port `2049` and check the actual network configuration.

### Q67. Why should the EC2 and EFS resources normally be in the same VPC for this lab?

Using the same VPC simplifies networking, DNS resolution, routing, and security group configuration.

More advanced cross-VPC designs are possible, but require appropriate network connectivity and DNS configuration.

### Q68. Why configure EFS mount targets in multiple AZs?

Mount targets provide network access to EFS. Configuring one in each required AZ allows clients in those AZs to use the filesystem through an appropriate local endpoint and supports resilient architecture.

### Q69. Can you use a security group to restrict which instances access EFS?

Yes. The EFS mount target security group can allow NFS only from an authorized EC2 client security group.

This is generally preferable to allowing all IP addresses.

---

# 10. EFS Mounting and Persistence

### Q70. How do you install EFS utilities on Amazon Linux 2023?

Run:

```bash
sudo dnf install -y amazon-efs-utils
```

### Q71. How do you install EFS utilities on Amazon Linux 2?

Run:

```bash
sudo yum install -y amazon-efs-utils
```

Package availability depends on the image and enabled repositories.

### Q72. How do you mount EFS on Linux?

Create a mount point:

```bash
sudo mkdir -p /mnt/efs
```

Mount using the EFS helper:

```bash
sudo mount -t efs -o tls fs-0123456789abcdef0:/ /mnt/efs
```

Replace the example file system ID with the actual EFS ID.

The `tls` option enables encryption in transit through the EFS mount helper.

### Q73. How do you verify an EFS mount?

Run:

```bash
findmnt /mnt/efs
```

And:

```bash
df -hT
```

You can also run:

```bash
mount | grep efs
```

### Q74. How do you configure a persistent EFS mount?

Add an entry to `/etc/fstab`:

```text
fs-0123456789abcdef0:/ /mnt/efs efs _netdev,tls 0 0
```

Replace the example EFS ID with the real file system ID.

Then validate and test:

```bash
sudo findmnt --verify --verbose
sudo mount -a
findmnt /mnt/efs
```

### Q75. What does `_netdev` mean in an EFS mount entry?

`_netdev` indicates that the filesystem depends on network connectivity. It helps the system treat the mount as a network-dependent filesystem during startup.

### Q76. What does the `tls` mount option do?

When used with the EFS mount helper, `tls` enables encryption in transit for the connection between the client and EFS.

### Q77. Why should both EC2 instances mount the same EFS ID?

Mounting the same EFS filesystem allows both instances to access the same files.

If they mount different EFS filesystems, the files will not automatically be shared.

### Q78. How can you prove that EFS is shared between two EC2 instances?

On Instance 1:

```bash
echo "Shared EFS test" | sudo tee /mnt/efs/shared.txt
```

On Instance 2:

```bash
cat /mnt/efs/shared.txt
```

If both instances mount the same EFS filesystem and permissions allow access, Instance 2 should read the same file.

### Q79. Why might EFS fail to mount even when port 2049 is allowed?

Other possible causes include:

- Missing EFS mount targets.
- Incorrect file system ID or DNS name.
- DNS resolution failure.
- Incorrect VPC or subnet selection.
- Network ACL or routing problems.
- Missing `amazon-efs-utils`.
- Incorrect mount command or filesystem entry.

### Q80. What is the difference between a manual EFS mount and a persistent EFS mount?

A manual mount is configured by running a mount command.

A persistent mount is configured through `/etc/fstab` or another system service so it can be mounted automatically during startup.

---

# 11. EBS vs EFS

### Q81. Which is better for a shared web application directory: EBS or EFS?

EFS is generally a better fit when multiple EC2 instances need to access the same shared filesystem concurrently.

EBS is more appropriate when an application needs block storage for an individual instance or a supported specialized configuration.

### Q82. Which service would you choose for an EC2 database data disk?

EBS is a common choice for a database data disk because it provides block storage with configurable performance characteristics.

The final choice depends on database support, performance requirements, durability, and architecture.

### Q83. Which service would you choose for shared files uploaded by a web application running on two EC2 instances?

EFS can be a good fit because both instances can access the same shared filesystem.

S3 may be a better choice if the application can use object storage rather than a shared filesystem.

### Q84. Can EFS replace the EC2 root volume?

EFS is not a direct replacement for the normal EBS-backed EC2 root volume in a typical Linux instance architecture.

EFS is generally mounted as shared application or data storage.

### Q85. Which is better for disaster recovery: EBS snapshots or EFS backups?

They address different storage types.

- EBS snapshots provide point-in-time backups of EBS volumes.
- EFS backup capabilities can protect EFS filesystems.

The appropriate choice depends on the storage service, recovery objectives, and workload.

---

# 12. Scenario-Based Interview Questions

### Q86. Scenario: An EC2 instance has been terminated, but its EBS data is required. What would you do?

**Answer:**

1. Check whether the original EBS volume still exists.
2. Check whether the volume was configured to be deleted on termination.
3. If the volume exists, attach it to a suitable EC2 instance in the same AZ.
4. If the volume is unavailable, look for an appropriate EBS snapshot.
5. Create a new volume from the snapshot.
6. Attach and mount the volume.
7. Verify the data before using it.

### Q87. Scenario: You increased an EBS volume from 10 GiB to 20 GiB, but Linux still shows the old filesystem size. What would you check?

**Answer:**

Check the block-device size and filesystem size separately:

```bash
lsblk
df -hT
```

The EBS volume may have been enlarged successfully, but the partition or filesystem may still need to be expanded.

### Q88. Scenario: You have two EC2 instances in different AZs and need to share files. Which AWS service would you choose?

**Answer:**

EFS is a common choice for a shared Linux filesystem.

Create a Regional EFS filesystem, configure mount targets in the required AZs, allow NFS TCP `2049` from the client security group, and mount the same filesystem on both instances.

### Q89. Scenario: EFS is mounted on Instance 1, but Instance 2 cannot mount it. What would you troubleshoot?

**Answer:**

1. Confirm both instances use the same EFS file system ID.
2. Confirm both instances are in the intended VPC and Region.
3. Verify a mount target exists in Instance 2's AZ.
4. Check the EFS mount target security group's inbound NFS rule.
5. Check the EC2 security group's outbound rules.
6. Check network ACLs and routing.
7. Verify DNS resolution.
8. Confirm that `amazon-efs-utils` is installed.
9. Inspect the mount error and system logs.

### Q90. Scenario: An EFS mount works until reboot, then disappears. What is the likely issue?

**Answer:**

The mount may have been created manually without a persistent configuration.

Add the correct entry to `/etc/fstab`, validate it, and test it with:

```bash
sudo findmnt --verify --verbose
sudo mount -a
```

### Q91. Scenario: An EFS mount command hangs even though the file system exists. What do you check first?

**Answer:**

Check network connectivity and the security group configuration, especially inbound TCP port `2049` on the EFS mount target security group.

Also verify the mount target, DNS resolution, routing, network ACLs, and the EC2 instance's outbound rules.

### Q92. Scenario: A developer accidentally deletes a file from EBS. How can you recover it?

**Answer:**

If the file cannot be recovered through normal filesystem or application mechanisms, create a new volume from a suitable snapshot, attach and mount it, locate the file, and copy it to the required destination.

Avoid overwriting the original data before verifying the recovery.

### Q93. Scenario: A developer accidentally formats a restored EBS volume. What would you do?

**Answer:**

Stop writing to the affected volume to avoid further damage. Identify a suitable earlier snapshot or backup and create a new recovery volume from it.

If no suitable backup exists, specialized recovery may be required and success is not guaranteed.

### Q94. Scenario: EFS is mounted on both instances, but one instance cannot create files. What would you check?

**Answer:**

Check filesystem ownership, directory permissions, the user ID and group ID, and any applicable EFS access point or IAM authorization configuration.

A successful mount does not guarantee write permission.

### Q95. Scenario: EFS works from one AZ but is inaccessible from another. What is a likely configuration issue?

**Answer:**

A mount target may be missing or unavailable in the second AZ, or the second instance may have a network or security group problem.

Check the mount targets, VPC, subnet configuration, security groups, routes, network ACLs, and DNS resolution.

### Q96. Scenario: Your company needs to restore an EBS backup in another AZ. What is your approach?

**Answer:**

Create a snapshot if a suitable one does not already exist. Then create a new EBS volume from the snapshot and select the target AZ. Attach the new volume to an EC2 instance in that AZ and verify the restored filesystem.

### Q97. Scenario: Your application needs shared files across several Linux instances, but the instances may be replaced or scaled. What would you consider?

**Answer:**

Consider EFS or another suitable shared storage service instead of keeping the only copy of shared data on one instance's local disk.

Ensure that network access, filesystem permissions, availability, backup, and cost requirements are addressed.

### Q98. Scenario: Your company needs to preserve EBS data when terminating EC2 instances. What would you configure?

**Answer:**

Review the `DeleteOnTermination` setting for the relevant volumes. Configure it to retain the data volumes where appropriate and maintain independent backups using EBS snapshots or a suitable backup service.

### Q99. Scenario: You created an EFS security group with port 2049 open to `0.0.0.0/0`. What should you change?

**Answer:**

Restrict the inbound NFS rule to the authorized EC2 client security group or another narrowly defined trusted source. NFS should not be exposed indiscriminately.

### Q100. Scenario: Your application needs both a database disk and shared uploads. Which storage services could you use?

**Answer:**

A common architecture is:

- EBS for database block storage.
- EFS for shared filesystem-based application uploads.
- S3 for object-based assets and backups where suitable.

The final architecture depends on database support, application design, performance, and recovery requirements.

---

# 13. Troubleshooting Questions

### Q101. What command shows disks and partitions?

```bash
lsblk
```

### Q102. What command shows filesystem types and UUIDs?

```bash
lsblk -f
```

### Q103. What command shows filesystem capacity and usage?

```bash
df -hT
```

### Q104. What command shows the filesystem mounted at a specific directory?

```bash
findmnt /data/ebs
```

Or for EFS:

```bash
findmnt /mnt/efs
```

### Q105. What command shows filesystem UUIDs?

```bash
sudo blkid
```

### Q106. What command tests `/etc/fstab` entries?

```bash
sudo findmnt --verify --verbose
```

Then:

```bash
sudo mount -a
```

### Q107. What should you check if an EBS volume cannot be mounted?

Check:

- Whether the volume is attached.
- Whether the correct disk or partition was selected.
- Whether a filesystem exists.
- Whether the filesystem is damaged.
- Whether the mount point exists.
- Whether the filesystem is already mounted elsewhere.

Do not format the volume as a first troubleshooting step.

### Q108. What should you check if EFS cannot be mounted?

Check:

- File system ID and DNS name.
- EFS mount target status.
- Security group inbound NFS rule.
- EC2 security group outbound access.
- Network ACLs and routes.
- DNS resolution.
- EFS utilities installation.
- Mount command and logs.

### Q109. What is the danger of adding an incorrect entry to `/etc/fstab`?

An incorrect entry can cause mounting failures and may affect startup or applications that depend on the filesystem.

Back up the file, validate changes, and test with `mount -a` before rebooting.

### Q110. Why is it important to understand the difference between a mount failure and a permission failure?

A mount failure means the filesystem has not been mounted successfully.

A permission failure may occur after the filesystem is mounted because the user lacks permission to access or modify the files.

Check the mount first, then investigate ownership and permissions.

---

# 14. Practical Interview Tasks

Perform these tasks in your AWS lab and be ready to explain each command.

## Task 1: Create and Mount EBS

- [ ] Create a gp3 EBS volume.
- [ ] Attach it to an EC2 instance.
- [ ] Identify the device using `lsblk -f`.
- [ ] Create a filesystem on the confirmed empty disk.
- [ ] Mount it under `/data/ebs`.
- [ ] Create a test file.
- [ ] Verify the file.

## Task 2: Configure Persistent EBS Mounting

- [ ] Identify the filesystem UUID.
- [ ] Back up `/etc/fstab`.
- [ ] Add a UUID-based entry.
- [ ] Validate using `findmnt --verify`.
- [ ] Test using `mount -a`.
- [ ] Verify with `findmnt` and `df -hT`.

## Task 3: Create and Restore an EBS Snapshot

- [ ] Create a snapshot of the source volume.
- [ ] Wait until it reaches `Completed`.
- [ ] Create a new volume from the snapshot.
- [ ] Select the target Availability Zone.
- [ ] Attach the new volume.
- [ ] Mount it without formatting it.
- [ ] Verify the restored data.

## Task 4: Expand an EBS Volume

- [ ] Modify the volume size in AWS.
- [ ] Confirm that Linux detects the larger device.
- [ ] Expand the partition if necessary.
- [ ] Grow the filesystem using the appropriate tool.
- [ ] Verify the new filesystem capacity.

## Task 5: Create Shared EFS Storage

- [ ] Launch two Amazon Linux EC2 instances in different AZs.
- [ ] Create an EC2 client security group.
- [ ] Create an EFS security group.
- [ ] Allow NFS TCP `2049` from the EC2 client security group.
- [ ] Create a Regional EFS filesystem.
- [ ] Configure mount targets in both AZs.
- [ ] Install `amazon-efs-utils` on both instances.
- [ ] Mount the same EFS filesystem on both.
- [ ] Create a file on Instance 1.
- [ ] Read the file from Instance 2.

## Task 6: Configure Persistent EFS Mounting

- [ ] Create `/mnt/efs` on both instances.
- [ ] Add the correct EFS entry to `/etc/fstab`.
- [ ] Include `_netdev,tls` in the mount options.
- [ ] Validate with `findmnt --verify`.
- [ ] Test with `mount -a`.
- [ ] Verify the mounted filesystem.
- [ ] Optionally reboot and verify automatic mounting.

---

# 15. Quick Revision

## EBS

- EBS = Block storage.
- EBS is commonly used for EC2 root volumes, application disks, and databases.
- EBS volumes belong to a specific AZ.
- EBS volumes normally attach to one EC2 instance at a time.
- Certain Provisioned IOPS volumes support Multi-Attach.
- Snapshots provide point-in-time backups.
- A new volume can be created from a snapshot.
- Use `lsblk -f` to identify devices and filesystems.
- Use `mkfs` only when intentionally creating a filesystem on a confirmed empty device.
- Use UUIDs for reliable persistent mount configuration.
- Use `resize2fs` for ext filesystems and `xfs_growfs` for XFS.

## EFS

- EFS = Shared file storage.
- EFS uses NFS over TCP port `2049`.
- Multiple clients can access the same filesystem.
- A Regional EFS filesystem supports access across AZs.
- Mount targets provide network access to EFS.
- EFS mount target security groups must allow NFS from authorized clients.
- Install `amazon-efs-utils` on Amazon Linux.
- Mount with the EFS helper using the correct file system ID.
- Use `_netdev,tls` for a typical persistent EFS mount entry.
- Use `mount -a` to test `/etc/fstab`.
- Verify shared access by creating a file on one instance and reading it from another.

## Linux Commands to Remember

```bash
lsblk
lsblk -f
sudo blkid
df -hT
findmnt
findmnt /data/ebs
findmnt /mnt/efs
sudo findmnt --verify --verbose
sudo mount -a
```

---

# GitHub Learning Log

**Topic:** AWS Storage — EBS and EFS Interview Preparation

### What I Learned

- Differences between block, file, and object storage.
- EBS volume types and common use cases.
- EBS attachment, formatting, mounting, and persistent mounting.
- EBS snapshots, volume restoration, and resizing.
- Filesystem UUIDs and Linux storage troubleshooting.
- EFS shared storage architecture.
- NFS TCP port `2049`.
- EFS mount targets and security group configuration.
- Mounting EFS on multiple EC2 instances across Availability Zones.
- Persistent EFS mounting through `/etc/fstab`.
- Troubleshooting and scenario-based AWS storage questions.

### Suggested Filename

```text
interview-questions.md
```

### Suggested Git Commit

```bash
git add interview-questions.md
git commit -m "Add AWS EBS and EFS interview questions"
git push
```