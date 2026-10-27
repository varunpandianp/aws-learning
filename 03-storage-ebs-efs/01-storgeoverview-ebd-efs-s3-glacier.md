# Amazon Storage Overview — EBS, EFS, S3 & Glacier

## 1. Introduction to AWS Storage

AWS provides different storage services depending on how applications need to store, access, share, and protect data.

### Types of Storage

| Storage Type | Description | AWS Service | Example |
|---|---|---|---|
| Block Storage | Stores data in blocks and provides disk-like storage | Amazon EBS | EC2 root disk, database disk |
| File Storage | Stores data as files and directories | Amazon EFS | Shared files across EC2 instances |
| Object Storage | Stores data as objects inside buckets | Amazon S3 | Images, videos, backups |
| Archival Storage | Designed for low-cost, long-term data retention | S3 Glacier storage classes | Historical backups, compliance archives |

---

## 2. Amazon EBS — Elastic Block Store

### What is EBS?

Amazon Elastic Block Store (EBS) provides persistent block storage volumes for EC2 instances.

Think of EBS as a virtual hard disk that can be attached to an EC2 instance.

### How EBS Works

1. Create an EBS volume in an Availability Zone.
2. Attach the volume to an EC2 instance in the same Availability Zone.
3. Identify the new disk in Linux.
4. Create a filesystem if required.
5. Create a mount point.
6. Mount the filesystem.
7. Configure `/etc/fstab` for automatic mounting after reboot.

### EBS Volume Types

| Volume Type | Category | Best Use Case |
|---|---|---|
| gp3 | General-purpose SSD | General applications, boot volumes, web servers |
| gp2 | General-purpose SSD | Existing workloads using the previous generation |
| io2 | Provisioned IOPS SSD | I/O-intensive databases and critical applications |
| io1 | Provisioned IOPS SSD | Existing provisioned-IOPS workloads |
| st1 | Throughput-optimized HDD | Throughput-intensive sequential workloads |
| sc1 | Cold HDD | Infrequently accessed, throughput-oriented workloads |

**Note:** `st1` and `sc1` cannot be used as boot volumes.

### Practical: Attach and Permanently Mount EBS

**Step 1: Identify disks**

```bash
lsblk
```

**Step 2: Check existing filesystems**

```bash
lsblk -f
```

**Step 3: Format a new, empty disk**

```bash
sudo mkfs.ext4 /dev/nvme1n1
```

Replace `/dev/nvme1n1` with the actual new disk identified on your instance.

**Warning:** Formatting destroys existing filesystem data. Never format a disk until you have confirmed its identity and that it contains no data you need to preserve.

**Step 4: Create a mount point**

```bash
sudo mkdir -p /data
```

**Step 5: Mount the volume**

```bash
sudo mount /dev/nvme1n1 /data
```

**Step 6: Verify the mount**

```bash
df -hT
```

**Step 7: Find the filesystem UUID**

```bash
sudo blkid /dev/nvme1n1
```

**Step 8: Edit `/etc/fstab`**

```bash
sudo vi /etc/fstab
```

Add the following line, replacing the placeholder with the actual filesystem UUID:

```text
UUID=<your-filesystem-uuid> /data ext4 defaults,nofail 0 2
```

**Step 9: Test the configuration**

```bash
sudo mount -a
```

**Step 10: Verify**

```bash
findmnt /data
```

The filesystem should mount automatically after reboot if the volume is attached and available.

### Why Use UUID Instead of `/dev/sdb`?

Linux device names can change when devices are added or the system reboots.

Using a filesystem UUID in `/etc/fstab` helps identify the correct filesystem independently of its device name.

On Nitro-based EC2 instances, EBS volumes commonly appear as NVMe devices such as `/dev/nvme1n1`.

### Benefits of EBS

- Persistent block storage.
- Suitable for operating systems and databases.
- Supports snapshots for backup and recovery.
- Offers different performance and cost options.
- Supports encryption.
- Can be detached and attached to another compatible instance when permitted.

### When to Choose EBS

Choose EBS when an application needs disk-like storage.

**Use cases:**
- EC2 root volumes.
- MySQL and PostgreSQL data disks.
- Application logs.
- Jenkins server data.
- Persistent application data.

---

## 3. Amazon EFS — Elastic File System

### What is EFS?

Amazon Elastic File System (EFS) is a managed, elastic file storage service that allows multiple clients to access a shared filesystem simultaneously.

Think of EFS as a shared network drive.

### How EFS Works

```text
             Amazon EFS
          Shared Filesystem
                 |
          +------+------+
          |             |
        EC2-A         EC2-B
          |             |
       /mnt/efs      /mnt/efs
          |             |
          +------+------+
                 |
           Shared Files
```

Both EC2 instances can access the same files, subject to filesystem permissions and access controls.

### EFS Uses NFS

NFS stands for Network File System.

EFS uses NFS to allow Linux clients to access files over a network.

| Property | Value |
|---|---|
| Protocol | NFS |
| Default Port | TCP 2049 |
| Network | VPC |
| Access Method | Mount a remote filesystem |
| Sharing | Multiple clients can access the same filesystem |
| Capacity | Automatically scales with stored data |

### Why Is Port 2049 Important?

When mounting EFS from an EC2 instance, the instance must be able to communicate with the EFS mount target over TCP port `2049`.

Configure the EFS security group as follows:

| Setting | Value |
|---|---|
| Direction | Inbound |
| Protocol | TCP |
| Port | 2049 |
| Source | EC2 instance security group |

If EC2 outbound rules are restricted, allow the required outbound traffic to the EFS security group.

Security groups are stateful, so return traffic for permitted connections is automatically allowed.

Also verify:

- EFS and EC2 have compatible VPC network configurations.
- An appropriate EFS mount target exists.
- Network ACLs permit the required traffic.
- NFS utilities or the EFS mount helper are installed.
- DNS resolution and filesystem permissions are correct.

### Practical: Mount EFS on EC2

**Step 1: Install the required package**

Amazon Linux:

```bash
sudo yum install -y amazon-efs-utils
```

Ubuntu:

```bash
sudo apt update
sudo apt install -y nfs-common
```

**Step 2: Create a mount point**

```bash
sudo mkdir -p /mnt/efs
```

**Step 3: Mount EFS using the EFS mount helper**

```bash
sudo mount -t efs fs-xxxxxxxx:/ /mnt/efs
```

Replace `fs-xxxxxxxx` with your actual EFS filesystem ID.

**Step 4: Verify**

```bash
df -hT
```

```bash
findmnt /mnt/efs
```

### Permanent EFS Mounting

Edit `/etc/fstab`:

```bash
sudo vi /etc/fstab
```

Add:

```text
fs-xxxxxxxx:/ /mnt/efs efs _netdev,noresvport,tls 0 0
```

Replace the filesystem ID with your actual EFS ID.

This example uses the EFS mount helper and TLS in transit.

Test:

```bash
sudo mount -a
```

Verify:

```bash
findmnt /mnt/efs
```

The `_netdev` option identifies the mount as network-dependent.

### Benefits of EFS

- Multiple EC2 instances can access the same files.
- Automatically scalable storage capacity.
- Managed network filesystem.
- Supports Linux filesystem semantics.
- Suitable for shared application data.
- Integrates with supported container and serverless workloads.

### When to Choose EFS

Choose EFS when multiple servers need access to the same files.

**Use cases:**
- Shared website uploads.
- Shared application files across Auto Scaling groups.
- Common files accessed by multiple Linux servers.
- Container workloads requiring shared persistent files.

---

## 4. Amazon S3 — Simple Storage Service

### What Is S3?

Amazon S3 is an object storage service used to store and retrieve data as objects inside buckets.

Unlike EBS, S3 is not a traditional disk attached to an EC2 instance.

### S3 Structure

```text
Bucket: my-devops-backups

Objects:
    database-backup.sql
    application.log
    website/index.html
    images/profile.png
```

### Important S3 Concepts

| Concept | Description |
|---|---|
| Bucket | Container for objects |
| Object | Stored data and metadata |
| Key | Object identifier within a bucket |
| Versioning | Maintains multiple versions of objects |
| Lifecycle Policy | Transitions or expires objects based on rules |
| Bucket Policy | Controls access through resource-based permissions |
| Encryption | Protects stored data |
| Replication | Copies objects to another bucket based on configuration |

### Benefits of S3

- Highly scalable object storage.
- Designed for high data durability.
- Multiple storage classes.
- Lifecycle management for cost optimization.
- Versioning and replication capabilities.
- Integration with many AWS services.

### When to Choose S3

Choose S3 when you need scalable object storage rather than a traditional disk or shared filesystem.

**Use cases:**
- Images and videos.
- Application backups.
- Terraform state files.
- Static website assets.
- CI/CD build artifacts.
- Log archives.
- Data lakes and analytics datasets.

### DevOps Example

A CI/CD pipeline can upload build artifacts to S3 so deployment systems can retrieve them later.

---

## 5. Amazon S3 Glacier — Archival Storage

### What Is S3 Glacier?

S3 Glacier refers to S3 storage classes designed for low-cost archival storage and long-term data retention.

These storage classes are suitable for data that must be retained but does not need to be accessed frequently.

Glacier storage classes are managed through Amazon S3; they are not volumes that you attach directly to EC2.

### Glacier Storage Classes

| Storage Class | Use Case | Retrieval Characteristics |
|---|---|---|
| S3 Glacier Instant Retrieval | Rarely accessed archives that require immediate access | Milliseconds |
| S3 Glacier Flexible Retrieval | Archives where retrieval can take minutes to hours | Minutes to hours |
| S3 Glacier Deep Archive | Long-term archives where retrieval can take many hours | Typically hours |

Exact retrieval times, costs, and minimum storage durations depend on the selected storage class and retrieval configuration.

### Benefits of Glacier Storage Classes

- Lower storage costs for suitable archival workloads.
- Supports long-term retention.
- Integrates with S3 lifecycle policies.
- Helps manage backup and compliance archives.
- Provides different retrieval options.

### When to Choose Glacier

Choose Glacier storage classes when data must be retained for a long time and immediate access is unnecessary or infrequent.

**Use cases:**
- Old database backups.
- Historical audit logs.
- Compliance archives.
- Long-term application backups.
- Records that must be retained for several years.

### Example: S3 Lifecycle Policy

```text
New Application Backup
          |
          v
      S3 Standard
          |
          | After 30 days
          v
 S3 Glacier Flexible Retrieval
          |
          | After 180 days
          v
   S3 Glacier Deep Archive
```

This is an illustrative design. Actual lifecycle rules should consider access patterns, minimum storage durations, retrieval costs, and retention requirements.

---

## 6. EBS vs EFS vs S3 vs Glacier

| Feature | EBS | EFS | S3 | S3 Glacier Classes |
|---|---|---|---|---|
| Storage Type | Block | File | Object | Object |
| Access Method | Block device | Network filesystem | API and supported integrations | API and supported integrations |
| Directly attached as an EC2 disk? | Yes | No | No | No |
| Multiple clients | Multi-Attach for eligible configurations; otherwise typically one instance | Yes | Yes, subject to permissions | Yes, subject to permissions |
| Filesystem | Usually required for normal file access | Managed filesystem | Not required | Not required |
| NFS | No | Yes | No | No |
| Network Port | Depends on access method | TCP 2049 | TCP 443 for normal HTTPS API access | TCP 443 for normal HTTPS API access |
| Capacity | Provisioned volume | Elastic | Elastic | Elastic |
| Main Use Case | EC2 disks and databases | Shared files | Application objects and backups | Long-term archives |

**Important:** EBS Multi-Attach is available only for eligible volume types and compatible instances within the same Availability Zone. It does not make EBS a general-purpose shared filesystem.

---

## 7. Which Storage Should You Choose?

| Requirement | Recommended Service | Reason |
|---|---|---|
| Install Linux on EC2 | EBS | Provides block storage for the root filesystem |
| Store database files on EC2 | EBS | Suitable for disk-oriented database workloads |
| Share uploaded files across EC2 instances | EFS | Multiple instances can access a shared filesystem |
| Store images and videos | S3 | Designed for scalable object storage |
| Store Terraform state centrally | S3 | Provides centralized object storage with access controls and versioning |
| Store daily backups | S3 | Scalable object storage |
| Retain backups for years | S3 Glacier Deep Archive | Designed for low-cost long-term retention |
| Share common Linux files | EFS | Provides network filesystem access |
| Archive historical audit logs | S3 Glacier classes | Supports long-term retention |

---

## 8. Real-World DevOps Architecture

Imagine deploying a website using an EC2 Auto Scaling group.

```text
                   Users
                     |
                     v
              Application Load
                  Balancer
                     |
             +-------+-------+
             |               |
             v               v
           EC2-A           EC2-B
             |               |
             +-------+-------+
                     |
                     v
                Amazon EFS
             Shared application files

                Amazon S3
          Static assets and backups

                Amazon EBS
          EC2 operating system disks

           S3 Glacier Classes
             Long-term archives
```

Each service solves a different storage problem.

In practice, static assets can also be served directly from S3. EFS is useful when the application requires shared filesystem access.

---

## 9. Common Troubleshooting

### EBS Troubleshooting

**Problem:** The volume is attached, but it is not mounted.

Check the disks:

```bash
lsblk
```

Check filesystems:

```bash
lsblk -f
```

Check mounted filesystems:

```bash
df -hT
```

Remember:

- Attached does not mean formatted.
- Formatted does not mean mounted.
- Mounted does not mean configured for automatic mounting.
- Never format a disk until you have identified it and confirmed that existing data can be destroyed.

### EFS Troubleshooting

**Problem:** EFS mounting fails or times out.

Check:

1. Is an appropriate EFS mount target available?
2. Does the EFS security group allow inbound TCP 2049 from the EC2 security group?
3. Can EC2 reach the mount target?
4. Does the EFS DNS name resolve?
5. Are NFS utilities or the EFS mount helper installed?
6. Are filesystem permissions and access-point settings correct?

Check DNS resolution:

```bash
getent hosts fs-xxxxxxxx.efs.<region>.amazonaws.com
```

Check TCP port 2049 connectivity:

```bash
nc -zv fs-xxxxxxxx.efs.<region>.amazonaws.com 2049
```

Replace the example filesystem ID and Region with your actual values.

### S3 Troubleshooting

**Problem:** An application receives `AccessDenied`.

Check:

- IAM permissions.
- Bucket policies.
- S3 Block Public Access settings.
- KMS key permissions, if applicable.
- Bucket name and object key.

A successful network connection does not guarantee authorization.

---

## 10. AWS Storage Interview Questions

### Q1. What is the difference between EBS and EFS?

EBS provides block storage for EC2 workloads, whereas EFS provides a shared network filesystem that multiple clients can mount simultaneously.

### Q2. What is the difference between EFS and S3?

EFS provides filesystem access using NFS, whereas S3 stores data as objects accessed through APIs and supported integrations.

### Q3. Which protocol and port does EFS use?

EFS uses NFS. The default port is TCP 2049.

### Q4. Why must we configure a security group for EFS?

The EFS mount target must permit incoming NFS connections from authorized clients, commonly through an inbound TCP 2049 rule referencing the EC2 security group.

### Q5. What happens to EBS when an EC2 instance is terminated?

An EBS volume configured for deletion on termination may be deleted along with the instance. A volume configured to be retained remains available, subject to its lifecycle and attachment state.

EFS and S3 data are independent of the EC2 instance's lifecycle.

### Q6. Why use UUID in `/etc/fstab`?

A UUID identifies a filesystem more reliably than a Linux device name, which can change.

### Q7. What is the difference between S3 Standard and S3 Glacier Deep Archive?

S3 Standard is designed for frequently accessed data with immediate retrieval. S3 Glacier Deep Archive is designed for low-cost long-term archival, with retrieval typically taking hours.

### Q8. Can we mount S3 directly like EBS?

S3 is object storage, not a native block device. Specialized software can expose S3 objects through filesystem-like interfaces, but their semantics and performance differ from conventional disks or NFS.

### Q9. Which service should you use to share files between two EC2 instances?

EFS is a natural choice when both instances need simultaneous filesystem access. S3 may be more suitable when the application can work with objects instead.

### Q10. How do you make an EBS filesystem mount automatically after reboot?

Create a filesystem when needed, mount it, obtain its UUID, add the appropriate entry to `/etc/fstab`, and test the configuration using `sudo mount -a`.

---

## 11. Quick Revision

- **EBS = Block Storage:** Disk-like storage for EC2, including root volumes and database disks.
- **EFS = File Storage:** Shared network filesystem using NFS over TCP port 2049.
- **S3 = Object Storage:** Buckets and objects for backups, static assets, artifacts, and data lakes.
- **S3 Glacier = Archival Storage Classes:** Low-cost long-term storage with retrieval characteristics determined by the selected class.

## 12. Practical Learning Tasks

### Task 1: EBS

- Create an EBS volume.
- Attach it to EC2.
- Identify the disk using `lsblk`.
- Create a filesystem if it is a new, empty volume.
- Mount it at `/data`.
- Configure `/etc/fstab`.
- Verify the mount after reboot.
- Safely unmount and detach the volume before deleting it.

### Task 2: EFS

- Create an EFS filesystem.
- Create or verify the required mount targets.
- Configure security groups for TCP 2049.
- Install NFS utilities or the EFS mount helper.
- Mount EFS on one EC2 instance.
- Mount the same EFS filesystem on a second EC2 instance.
- Create a test file on one instance and verify that it is accessible from the other.
- Configure persistent mounting.

### Task 3: S3 and Glacier

- Create an S3 bucket.
- Upload a test file.
- Enable versioning.
- Create a lifecycle policy.
- Configure an appropriate archival transition.
- Review storage costs and retrieval requirements.

**Learning objective:** Understand when to use block, file, object, and archival storage in real AWS infrastructure.