# AWS EBS Volume Types and Use Cases

## 1. What Is Amazon EBS?

Amazon Elastic Block Store (EBS) is a block-level storage service designed for use with Amazon EC2 instances.

You can use EBS volumes like virtual hard disks attached to EC2 instances.

Common uses include:
- Operating system storage.
- Application data.
- Database storage.
- Persistent application files.
- Log storage.
- Data processing workloads.

### Example Architecture

```text
                    AWS Cloud
                       |
                       v
                  EC2 Instance
                       |
             +---------+---------+
             |                   |
             v                   v
        Root EBS Volume      Data EBS Volume
             |                   |
             v                   v
        Linux OS Files       Application Data
```

For example:
- Root EBS volume: stores the Linux operating system.
- Data EBS volume: stores application files or database data.

**Important:** EBS volumes are persistent storage devices that can exist independently of an EC2 instance. However, the volume's deletion settings determine whether it is deleted when the instance is terminated.

---

# 2. Types of EBS Volumes

Amazon EBS volume types are divided into two main categories.

```text
Amazon EBS Volume Types
          |
          +-------------------------+
          |                         |
          v                         v
      SSD Volumes               HDD Volumes
          |                         |
          +-- General Purpose       +-- Throughput Optimized HDD
          |   |                     |   `st1`
          |   +-- gp3               |
          |   +-- gp2               +-- Cold HDD
          |                             `sc1`
          |
          +-- Provisioned IOPS SSD
              |
              +-- io2
              +-- io1

Previous Generation:
    standard (Magnetic)
```

## SSD vs. HDD

| Feature | SSD-backed EBS | HDD-backed EBS |
|---|---|---|
| Primary optimization | IOPS and low latency | Throughput |
| Typical access pattern | Random reads and writes | Large, sequential reads and writes |
| Common workloads | Databases, OS disks, application storage | Big data, log processing, large sequential datasets |
| Boot volume support | Yes, for supported SSD types | No for `st1` and `sc1` |
| Examples | `gp3`, `gp2`, `io1`, `io2` | `st1`, `sc1` |

### What Is IOPS?

IOPS means **Input/Output Operations Per Second**.

It measures how many read or write operations a storage device can perform per second.

Example:

A database frequently reads and updates many small records. It may benefit from a volume with high IOPS.

### What Is Throughput?

Throughput measures how much data can be transferred per second.

It is commonly expressed in MiB/s.

Example:

A data processing application reads large log files sequentially. It may benefit from a volume optimized for throughput.

### What Is Latency?

Latency is the time required for an I/O operation to complete.

For latency-sensitive applications, lower storage latency can improve responsiveness.

**Remember:**

- IOPS = number of I/O operations per second.
- Throughput = amount of data transferred per second.
- Latency = time taken to complete an operation.

---

# 3. General Purpose SSD — gp3

## What Is gp3?

`gp3` is a general-purpose SSD EBS volume that provides a balance between price and performance.

It is a strong default choice for many EC2 workloads.

### Typical Use Cases

- EC2 root volumes.
- Linux operating system storage.
- Web servers.
- Application servers.
- Development environments.
- Test environments.
- Small and medium-sized databases.
- General application data.

### Example Scenario

Suppose you create an EC2 instance to host a portfolio website.

Your setup includes:
- Amazon Linux.
- Nginx.
- Application files.
- Logs.
- A small amount of persistent data.

A `gp3` volume is usually a sensible starting choice.

### Why Choose gp3?

- General-purpose performance.
- Predictable baseline performance.
- IOPS can be provisioned independently of volume size.
- Throughput can be provisioned independently of volume size, subject to service limits and configuration.
- Suitable for many common workloads.

### Key Performance Characteristics

| Feature | gp3 |
|---|---|
| Storage range | 1 GiB–64 TiB |
| Baseline IOPS | 3,000 |
| Maximum IOPS | Up to 80,000 |
| Baseline throughput | 125 MiB/s |
| Maximum throughput | Up to 2,000 MiB/s |
| Boot volume | Yes |

The maximum performance depends on volume size, provisioned settings, and other service constraints. The EC2 instance must also be capable of supporting the required performance.

### When Should You Choose gp3?

Choose `gp3` when:
- You need a general-purpose SSD.
- You are creating a typical EC2 instance.
- Your application needs good performance without specialized storage requirements.
- You want to configure performance without increasing volume size solely to obtain more IOPS.

### Interview Answer

**Question: What is gp3 in AWS?**

Answer:

`gp3` is a general-purpose SSD EBS volume type that provides a balance between cost and performance. It is suitable for boot volumes, application servers, development environments, and many general-purpose database workloads. Its IOPS and throughput can be configured independently of volume size.

---

# 4. General Purpose SSD — gp2

## What Is gp2?

`gp2` is an older generation of general-purpose SSD EBS volume.

It is still important to recognize because existing AWS environments may use it.

### Typical Use Cases

- Existing EC2 root volumes.
- Application servers.
- Development and test systems.
- General-purpose databases.
- Legacy deployments.

### Important Characteristic

With `gp2`, baseline performance generally scales with volume size, and smaller volumes can use burst performance under supported conditions.

This differs from `gp3`, which allows IOPS and throughput to be provisioned independently of volume size.

### gp2 vs. gp3

| Feature | gp2 | gp3 |
|---|---|---|
| Generation | Older | Newer |
| Storage range | 1 GiB–16 TiB | 1 GiB–64 TiB |
| Maximum IOPS | Up to 16,000 | Up to 80,000 |
| Maximum throughput | Up to 250 MiB/s | Up to 2,000 MiB/s |
| Performance scaling | Primarily tied to volume size and burst behavior | Independently configurable IOPS and throughput |
| General recommendation | Recognize for existing environments | Preferred starting point for many new general-purpose workloads |

### When Should You Choose gp2?

For a new general-purpose workload, evaluate `gp3` first.

You may encounter `gp2` when:
- Maintaining existing EC2 instances.
- Supporting older infrastructure.
- Investigating legacy storage configurations.

When appropriate, existing `gp2` volumes can be modified to `gp3`. Evaluate performance requirements and cost before making the change.

### Interview Answer

**Question: What is the difference between gp2 and gp3?**

Answer:

Both are general-purpose SSD EBS volume types. `gp2` performance is primarily tied to volume size and burst behavior, whereas `gp3` provides baseline performance and allows IOPS and throughput to be configured independently of capacity. `gp3` is often the preferred starting point for new general-purpose workloads.

---

# 5. Provisioned IOPS SSD — io2

## What Is io2?

`io2` is a Provisioned IOPS SSD volume type designed for workloads that need high and sustained storage performance.

The `io2 Block Express` offering supports especially demanding I/O-intensive workloads.

### Typical Use Cases

- Mission-critical databases.
- High-performance relational databases.
- Enterprise database applications.
- I/O-intensive transactional systems.
- Applications with strict storage latency requirements.
- Performance-sensitive database workloads.

### Example Scenario

Imagine an online banking application that processes many transactions.

The database may need:
- Consistent I/O performance.
- High IOPS.
- Low latency.
- High storage durability.

An `io2` volume may be appropriate if the workload's measured performance and durability requirements justify its cost.

### Key Characteristics

| Feature | io2 Block Express |
|---|---|
| Storage range | 4 GiB–64 TiB |
| Maximum IOPS | Up to 256,000 |
| Maximum throughput | Up to 4,000 MiB/s |
| Main focus | High-performance, mission-critical workloads |
| Boot volume | Supported |

The maximum performance is subject to supported EC2 instance types, volume configuration, and other AWS service limits.

### When Should You Choose io2?

Choose `io2` when:
- The workload requires sustained, high IOPS.
- Storage latency is a critical requirement.
- The application is performance-sensitive.
- Your measured workload justifies the additional cost.
- You need the durability characteristics associated with `io2`.

### When Should You Avoid It?

Do not automatically choose `io2` for every EC2 instance.

For a basic web server or learning environment, `gp3` is usually a more sensible starting point.

### Interview Answer

**Question: When would you use io2 instead of gp3?**

Answer:

I would consider `io2` for mission-critical, I/O-intensive applications that require sustained high IOPS, low latency, or higher storage durability. For ordinary application servers and general-purpose workloads, I would evaluate `gp3` first and select `io2` only when the workload requirements justify it.

---

# 6. Provisioned IOPS SSD — io1

## What Is io1?

`io1` is an older generation of Provisioned IOPS SSD EBS volume.

It allows you to provision IOPS for demanding storage workloads.

### Typical Use Cases

- Existing high-performance database environments.
- Legacy enterprise applications.
- I/O-intensive workloads already designed around `io1`.

### Key Characteristics

| Feature | io1 |
|---|---|
| Storage range | 4 GiB–16 TiB |
| Maximum IOPS | Up to 64,000 |
| Maximum throughput | Up to 1,000 MiB/s |
| Main focus | Provisioned IOPS |
| Boot volume | Supported |

Performance depends on the volume configuration and the capabilities of the attached EC2 instance.

### When Should You Choose io1?

For new deployments, compare the requirements and costs of `io2` and `gp3` before choosing `io1`.

You may encounter `io1` when:
- Supporting existing systems.
- Maintaining older database environments.
- Working with a deployment that already uses provisioned IOPS storage.

### Interview Answer

**Question: What is the difference between io1 and io2?**

Answer:

Both are Provisioned IOPS SSD EBS volume types. `io2` is the newer option and offers improved durability characteristics, with `io2 Block Express` providing higher performance limits for supported workloads. For new high-performance workloads, I would evaluate `io2` first.

---

# 7. Throughput Optimized HDD — st1

## What Is st1?

`st1` is a hard disk drive-backed EBS volume optimized for throughput-intensive workloads.

It is designed for large, sequential I/O operations rather than workloads dominated by frequent small, random reads and writes.

### Typical Use Cases

- Big data processing.
- Large log processing workloads.
- Data warehouses.
- Sequential data scans.
- Large datasets processed in batches.
- Applications that read or write large amounts of data sequentially.

### Example Scenario

Suppose a company stores several large log files and processes them in batches.

The application reads the files sequentially and performs analytics.

An `st1` volume may be suitable if the workload benefits more from sequential throughput than from high random IOPS.

### Key Characteristics

| Feature | st1 |
|---|---|
| Storage range | 125 GiB–16 TiB |
| Maximum throughput | Up to 500 MiB/s |
| Main focus | Throughput |
| Boot volume | No |

### When Should You Choose st1?

Choose `st1` when:
- You process large amounts of sequential data.
- Throughput is more important than random IOPS.
- You need cost-conscious storage for suitable large datasets.

### When Should You Avoid st1?

Avoid it for workloads that need a boot volume or depend on frequent small, random I/O operations, such as many transactional databases.

### Interview Answer

**Question: When would you use st1?**

Answer:

I would use `st1` for throughput-intensive workloads such as large log processing, big data, and sequential data scans. It is not intended for boot volumes or workloads dominated by small, random I/O operations.

---

# 8. Cold HDD — sc1

## What Is sc1?

`sc1` is a hard disk drive-backed EBS volume designed for throughput-oriented workloads where data is accessed relatively infrequently and storage cost is a major consideration.

### Typical Use Cases

- Infrequently accessed datasets.
- Large, cold datasets.
- Cost-sensitive sequential data processing.
- Data that does not require frequent access.

### Example Scenario

Suppose an application retains large historical datasets that are occasionally scanned for analysis.

If the workload is suitable for HDD storage and performance requirements are modest, `sc1` may be worth evaluating.

### Key Characteristics

| Feature | sc1 |
|---|---|
| Storage range | 125 GiB–16 TiB |
| Maximum throughput | Up to 250 MiB/s |
| Main focus | Low-cost, infrequently accessed data |
| Boot volume | No |

### When Should You Choose sc1?

Choose `sc1` when:
- The dataset is accessed infrequently.
- Large sequential reads are suitable for the workload.
- Low storage cost is a priority.
- You can accept its performance characteristics.

### When Should You Avoid sc1?

Avoid it for:
- Operating system boot volumes.
- Latency-sensitive databases.
- Workloads requiring high random IOPS.
- Applications that need consistently high performance for frequent small reads and writes.

### Interview Answer

**Question: What is the difference between st1 and sc1?**

Answer:

Both are throughput-oriented HDD volume types. `st1` is intended for frequently processed, throughput-intensive data, whereas `sc1` is designed for infrequently accessed data when low storage cost is a priority.

---

# 9. Magnetic — standard

## What Is standard?

`standard` is a previous-generation magnetic EBS volume type.

It is mainly relevant when maintaining or investigating older AWS infrastructure.

### Typical Use Cases

- Legacy environments.
- Existing workloads with modest storage requirements.
- Older deployments where storage performance is not a primary concern.

### When Should You Choose It?

For a new deployment, evaluate current-generation volume types first.

The `standard` volume type is a legacy option, not the default recommendation for new workloads.

### Interview Answer

**Question: Is standard a current-generation EBS volume type?**

Answer:

No. `standard` is a previous-generation magnetic volume type. For new deployments, I would generally evaluate current-generation EBS options such as `gp3`, depending on the workload.

---

# 10. EBS Volume Types Comparison

The following figures are service-level maximums, not guaranteed performance for every volume. Actual performance depends on volume configuration, I/O size, instance capabilities, and other constraints.

| Volume Type | Storage Range | Maximum IOPS | Maximum Throughput | Best Suited For |
|---|---|---:|---:|---|
| gp3 | 1 GiB–64 TiB | 80,000 | 2,000 MiB/s | General-purpose workloads |
| gp2 | 1 GiB–16 TiB | 16,000 | 250 MiB/s | Existing general-purpose workloads |
| io2 Block Express | 4 GiB–64 TiB | 256,000 | 4,000 MiB/s | Mission-critical, I/O-intensive workloads |
| io1 | 4 GiB–16 TiB | 64,000 | 1,000 MiB/s | Existing provisioned-IOPS workloads |
| st1 | 125 GiB–16 TiB | 500* | 500 MiB/s | Large sequential data processing |
| sc1 | 125 GiB–16 TiB | 250* | 250 MiB/s | Infrequently accessed sequential data |
| standard | 1 GiB–1 TiB | Approximately 40–200 | Approximately 40–90 MiB/s | Legacy workloads |

`*` HDD IOPS figures depend on the I/O size used in AWS's performance specifications.

These figures are useful for learning and comparison. Always check the current AWS documentation before making a production design decision.

---

# 11. Real-World Scenarios: Which EBS Volume Should You Choose?

## Scenario 1: EC2 Web Server

**Requirement:**
- Run Linux.
- Host an Nginx website.
- Store application files.
- Use a typical workload.

**Recommended starting point:** `gp3`

**Reason:** It provides a good balance between price and performance for general-purpose workloads.

## Scenario 2: Development and Testing Environment

**Requirement:**
- Create EC2 instances for development.
- Run applications and testing tools.
- Keep storage costs reasonable.

**Recommended starting point:** `gp3`

**Reason:** General-purpose SSD storage is suitable for many development and test workloads.

## Scenario 3: Transactional Database

**Requirement:**
- Process frequent database reads and writes.
- Maintain predictable storage performance.
- Handle application transactions.

**Recommended starting point:** Evaluate `gp3` first; consider `io2` when performance, latency, or durability requirements justify it.

**Reason:** The correct choice depends on actual IOPS, latency, throughput, database size, and business requirements.

## Scenario 4: Mission-Critical Database

**Requirement:**
- High sustained IOPS.
- Low storage latency.
- High durability requirements.
- Performance-sensitive transactions.

**Recommended candidate:** `io2`, including `io2 Block Express` when supported and required.

**Reason:** It is designed for demanding I/O-intensive workloads.

## Scenario 5: Large Log Processing

**Requirement:**
- Read large log files sequentially.
- Process datasets in batches.
- Prioritize throughput.

**Recommended candidate:** `st1`, if the workload fits its limits and access pattern.

**Reason:** It is optimized for throughput-oriented HDD workloads.

## Scenario 6: Infrequently Accessed Data

**Requirement:**
- Store large datasets.
- Access them infrequently.
- Keep storage costs low.

**Recommended candidate:** `sc1`, if the workload is compatible with HDD performance.

**Reason:** It is designed for cold, throughput-oriented data.

## Scenario 7: Existing EC2 Instance Uses gp2

**Requirement:**
- Maintain an existing instance.
- Evaluate whether storage cost or performance can be improved.

**Recommended action:** Compare the current configuration with `gp3`.

**Reason:** `gp3` allows performance to be provisioned independently of storage capacity, which may improve cost efficiency for suitable workloads.

---

# 12. EBS Volume Selection Decision Flowchart

```text
                 Choose an EBS Volume
                          |
                          v
          Is the workload I/O intensive?
                   /             \
                 Yes              No
                  |                |
                  v                v
        Does it require       Is it mostly
        high sustained IOPS   sequential data?
        and low latency?         /      \
             /    \            Yes       No
           Yes     No            |         |
            |       |            v         v
            v       v           Is data   General-
          Evaluate gp3          accessed purpose
          io2                  frequently? workload
                                  /  \
                                Yes   No
                                 |     |
                                 v     v
                                st1   sc1
```

### Simplified Decision Guide

- General-purpose EC2 workload → `gp3`
- Existing general-purpose workload → evaluate `gp2` versus `gp3`
- High-performance, I/O-intensive database → evaluate `io2`
- Existing provisioned-IOPS system → assess `io1` and migration options
- Large sequential data processing → `st1`
- Infrequently accessed sequential data → `sc1`
- Legacy magnetic volume → `standard`

This flowchart is a starting point. For production systems, benchmark the workload and check performance requirements before selecting a volume type.

---

# 13. EBS Volume Type vs. EBS Volume Size

A common mistake is assuming that increasing storage capacity always improves performance.

The volume type and the volume size are separate considerations.

For example:

```text
Volume A:
Type: gp3
Size: 20 GiB

Volume B:
Type: gp3
Size: 100 GiB
```

Both volumes use the same general-purpose SSD volume type, but their storage capacities differ.

With `gp3`, IOPS and throughput can be configured independently of size, subject to the service limits and required configuration.

With `gp2`, baseline performance generally scales with volume size, and burst behavior may also affect performance.

**Remember:** Select a volume type based on the workload, then select the capacity and performance settings the workload requires.

---

# 14. Important EBS Concepts for DevOps Engineers

## EBS Volume Persistence

An EBS volume can persist independently of the EC2 instance.

However, check the `DeleteOnTermination` setting before terminating an instance.

## Availability Zone

An EBS volume must ordinarily be attached to an EC2 instance in the same Availability Zone.

To use the data in another Availability Zone, you can use a snapshot to create a new volume in the target Availability Zone.

## EBS Snapshots

EBS snapshots provide point-in-time backups of volume data.

Use snapshots as part of a backup and recovery strategy, and test that the recovery process works.

## EBS Encryption

EBS supports encryption to protect data at rest and data moving between the instance and the volume, subject to the service's encryption behavior and supported instance configurations.

## EBS Resizing

You can generally increase the size of supported EBS volumes using volume modification.

After increasing the volume size, Linux may also need the partition and filesystem to be expanded.

## EBS Monitoring

Use Amazon CloudWatch metrics to monitor relevant volume performance and workload behavior.

Examples include:
- Read and write operations.
- Read and write throughput.
- I/O latency-related metrics where available.
- Queue length and I/O performance indicators.

Use monitoring data to determine whether a volume is meeting application requirements.

---

# 15. Common Interview Questions

## Q1. What are the main EBS volume types?

The main types are:
- General Purpose SSD: `gp3`, `gp2`
- Provisioned IOPS SSD: `io2`, `io1`
- Throughput Optimized HDD: `st1`
- Cold HDD: `sc1`
- Previous-generation Magnetic: `standard`

## Q2. Which EBS volume type would you choose for a normal EC2 web server?

I would generally start with `gp3` because it provides a balance between price and performance for common workloads.

## Q3. Which EBS volume is suitable for a high-performance database?

I would evaluate `io2` for workloads requiring high sustained IOPS, low latency, or high durability. I would compare those requirements with the workload's actual performance measurements and cost constraints.

## Q4. What is the difference between IOPS and throughput?

IOPS measures the number of I/O operations per second. Throughput measures the amount of data transferred per second.

## Q5. Why is gp3 often preferred over gp2?

`gp3` provides baseline performance and lets users configure IOPS and throughput independently of storage capacity. `gp2` performance is primarily tied to volume size and burst behavior.

## Q6. Can I use st1 as an EC2 boot volume?

No. `st1` is not supported as an EBS boot volume.

## Q7. Can I use sc1 for a transactional database?

It is generally not a good choice for transactional databases that require frequent small, random I/O operations. Evaluate SSD-backed storage instead.

## Q8. What is the difference between gp3 and io2?

`gp3` is general-purpose SSD storage. `io2` is Provisioned IOPS SSD storage designed for more demanding I/O-intensive workloads and higher durability.

## Q9. Does increasing the EBS volume size automatically resize the Linux filesystem?

Not necessarily. The operating system may need to recognize the new capacity, expand the partition if one exists, and resize the filesystem.

## Q10. Can one EBS volume be attached to multiple EC2 instances?

Ordinarily, an EBS volume is attached to one instance at a time. EBS Multi-Attach is supported for eligible `io1` and `io2` volumes and compatible Nitro-based instances, subject to AWS limitations. Multi-Attach does not by itself provide a shared filesystem or coordinate concurrent writes; the application and filesystem must support the configuration.

---

# 16. Hands-On Practice Tasks

## Task 1: Create a gp3 Volume

1. Open the EC2 console.
2. Navigate to Elastic Block Store → Volumes.
3. Create a 5 GiB `gp3` volume.
4. Select the same Availability Zone as your EC2 instance.
5. Attach the volume to the instance.
6. Identify the device using `lsblk`.
7. Create a filesystem only if the volume is new and empty.
8. Mount it at `/data/ebs`.
9. Configure persistent mounting using `/etc/fstab`.
10. Verify the mount with `findmnt` and `df -hT`.

## Task 2: Compare gp2 and gp3

Research the differences in:
- Performance scaling.
- Baseline IOPS.
- Throughput.
- Cost.
- Volume capacity.

Write a short explanation of why `gp3` may be a better option for a new general-purpose workload.

## Task 3: Choose a Volume for Each Scenario

| Scenario | Your Choice | Reason |
|---|---|---|
| Linux EC2 root disk | | |
| Typical web server | | |
| High-performance database | | |
| Large sequential log processing | | |
| Infrequently accessed sequential data | | |
| Existing gp2 workload | | |

Fill in the table and explain your reasoning.

## Task 4: Resize an EBS Volume

1. Record the original volume size.
2. Increase the volume size in AWS.
3. Verify the new capacity with `lsblk`.
4. Identify the filesystem type.
5. Expand the partition if necessary.
6. Use `resize2fs` for ext filesystems or `xfs_growfs` for XFS.
7. Verify the result with `df -hT`.

Perform this task only on a volume you are authorized to modify, and protect any data that must be retained.

---

# 17. Key Takeaways

1. `gp3` is a sensible starting point for many general-purpose workloads.
2. `gp2` is an older general-purpose SSD type that you may encounter in existing environments.
3. `io2` is designed for demanding I/O-intensive and mission-critical workloads.
4. `io1` is an older Provisioned IOPS SSD type.
5. `st1` is designed for throughput-intensive, sequential HDD workloads.
6. `sc1` is designed for infrequently accessed, cost-sensitive sequential data.
7. `standard` is a previous-generation magnetic volume type.
8. IOPS, throughput, and latency describe different aspects of storage performance.
9. Increasing volume capacity does not always mean that the filesystem can immediately use the additional space.
10. Choose EBS volume types according to workload requirements, measured performance, availability needs, and cost.

**Final rule for AWS interviews:**

Choose `gp3` for general-purpose workloads, evaluate `io2` for demanding I/O-intensive workloads, consider `st1` for large sequential data processing, and consider `sc1` for infrequently accessed sequential data when its performance characteristics are appropriate.