# AWS EC2 Notes — Instance Types, Purchase Options, AMIs, and Snapshots

**Course:** Sudharsan — AWS Cloud / DevOps  
**Suggested path:** `AWS/03-EC2-Compute-Service/EC2-Instance-Types-Purchase-Options-AMI-vs-Snapshot.md`

> Check current AWS Region availability, pricing, and instance-generation details before choosing an instance. Names and prices change over time.

## 1. What is an EC2 instance type?

An EC2 instance type defines the resources presented to a virtual server:
- **vCPU:** processing capacity
- **Memory (RAM):** working memory for OS and applications
- **Storage options:** local instance-store availability and EBS capabilities
- **Network performance:** bandwidth and packet-processing capability
- **Accelerators:** some types include GPUs or specialized hardware

Example: `m7i.large`
- `m` = general-purpose family
- `7` = generation
- `i` = family variant
- `large` = size

`m4.xlarge` is an older-generation example. For a new workload, compare currently available generations, compatibility, performance, and price rather than selecting an older family by default.

## 2. Major EC2 instance categories

| Category | Best suited for | Example workloads | Illustrative families |
|---|---|---|---|
| General purpose | Balanced CPU, RAM, and networking | Dev/test, web servers, app servers | `t3`, `t4g`, `m6i`, `m7i`, `m7g` |
| Compute optimized | High CPU relative to memory | CPU-heavy apps, batch processing, high-performance web tiers | `c6i`, `c7i`, `c7g` |
| Memory optimized | Large RAM capacity or high memory bandwidth | In-memory databases, caches, real-time analytics | `r6i`, `r7i`, `x2`, `u` families |
| Storage optimized | High local storage throughput/IOPS or low-latency local storage | Distributed databases, data warehousing, log processing | `i4i`, `im4gn`, `is4gen`, `d3` |
| Accelerated computing | GPUs or specialized accelerators | ML training/inference, graphics rendering, scientific workloads | `g`, `p`, selected `inf` / `trn` families |
| HPC optimized | High-performance computing | Simulations, computational fluid dynamics, scientific workloads | `hpc` families and selected compute-optimized families |

Availability varies by Region, OS, architecture, and generation. Burstable `t` instances are useful for variable CPU workloads; sustained CPU-heavy workloads may need another family or careful CPU-credit planning.

### Quick selection guide
- Balanced server → **general purpose**
- CPU-heavy workload → **compute optimized**
- Large RAM requirement → **memory optimized**
- High-speed local storage → **storage optimized**
- GPU/special accelerator → **accelerated computing**
- Scientific simulation/HPC → **HPC optimized**

Compare vCPU, RAM, network performance, EBS bandwidth, architecture, price, and benchmark results. Do not choose solely from the family name.

## 3. Choosing by environment

| Environment | Starting point | Considerations |
|---|---|---|
| Personal learning lab | Small burstable `t` instance or other low-cost eligible type | Check current free-tier eligibility, RAM needs, and Region price |
| Development server | Small general-purpose or burstable type | Enough RAM for tools, builds, and test services |
| Test/staging | Similar family to production for realistic performance testing | Scale down only when test objectives allow |
| Production web/app server | General-purpose or compute-optimized based on metrics | Load test, monitor CPU/RAM/network, plan availability |
| In-memory database | Memory-optimized | Dataset size, cache needs, replication, headroom |
| High-I/O data processing | Storage-optimized or suitable EBS architecture | IOPS, latency, throughput, durability |
| ML training/rendering | Accelerated computing | Accelerator, memory, framework, and scaling requirements |

**Example:** A small development web server may run on a small general-purpose instance. A production application may need a larger instance or several instances across Availability Zones, a load balancer, Auto Scaling, monitoring, backups, and deployment automation. The correct size depends on measured workload and availability targets—not just the word “production.”

## 4. EC2 purchase options

Purchase options describe how capacity is billed; instance families describe resources. They are separate decisions.

### On-Demand
- No long-term commitment.
- Useful for short labs, unpredictable workloads, and initial tests.
- EBS, public IPv4, data transfer, and other resources may have separate charges.

### Savings Plans
- Commit to eligible compute usage at a rate measured in dollars per hour, usually for a 1- or 3-year term depending on the plan.
- May reduce costs for steady usage.
- A commitment can still be charged even if usage is lower than expected.

### Reserved Instances
- Billing discount commitment for eligible usage, commonly over one or three years.
- Benefits and flexibility vary by reservation type and attributes.
- Compare with Savings Plans before committing.

### Spot Instances
- Use spare AWS capacity at a discount.
- Can be interrupted.
- Suitable for fault-tolerant batch jobs, retryable CI workers, stateless processing, and checkpointed workloads.
- Avoid for workloads that cannot tolerate interruption unless designed to handle it.

### Dedicated Hosts / Dedicated Instances
- Provide dedicated physical-server or tenancy characteristics for particular licensing, compliance, or isolation requirements.
- Usually cost more and are not the default for ordinary learning labs.

| Situation | Option to evaluate | Why |
|---|---|---|
| A few hours of learning | On-Demand | No long-term commitment |
| Variable testing | On-Demand | Flexible |
| Stable 24/7 usage | Savings Plans / Reserved Instances | Potential savings with commitment |
| Interruptible batch processing | Spot | Lower cost if interruptions are acceptable |
| Host-level licensing need | Dedicated Host | Host-level control/visibility for supported cases |

## 5. Check CPU and RAM on Linux

Connect to your Linux EC2 instance through SSH or MobaXterm.

### Memory / RAM

```bash
free -m
```

Displays memory and swap in MiB. The `available` column is often more useful than `free` for estimating memory available to applications.

```bash
free -h
cat /proc/meminfo
```

### CPU information

```bash
lscpu
```

Shows architecture, visible CPU count, threads, and other CPU details.

```bash
nproc
cat /proc/cpuinfo
grep -c '^processor' /proc/cpuinfo
```

**Correction:** The correct path is `/proc/cpuinfo`, not `/proc/cpinfo`.

### Disk and OS information

```bash
df -h
lsblk
cat /etc/os-release
```

- `df -h`: filesystem space usage
- `lsblk`: block devices and attached disks
- `/etc/os-release`: Linux distribution information

These commands show resources visible inside the OS. Compare them with EC2 console details; virtualization and system reservations can cause small differences.

## 6. What is an AMI?

An **Amazon Machine Image (AMI)** is a template used to launch EC2 instances. It includes an operating-system image and launch information such as block-device mappings, and may include preinstalled software and configuration.

AMI types can include AWS-provided images, Marketplace images, custom images, and shared images subject to permissions and restrictions.

### When should you create an AMI?
- After preparing a reusable, patched “golden image”
- To launch multiple instances with a consistent OS and software baseline
- In an image-based deployment pipeline
- Before a major change as one part of a recovery plan
- To standardize development, test, and production environments

Build images through a documented, repeatable process. Remove secrets and machine-specific data, patch images regularly, and control who can launch or share them.

### High-level AMI creation
1. Select the source EC2 instance.
2. Prepare the OS and application configuration.
3. Choose **Actions → Image and templates → Create image** (console wording may vary).
4. Enter an image name and description.
5. Review volumes included in the image.
6. Create the image and monitor its status.
7. Launch a test instance from the AMI.

Creating an AMI can create EBS snapshots and incur storage charges. Review reboot/consistency options and application impact before proceeding.

## 7. What is an EBS snapshot?

An **EBS snapshot** is a point-in-time backup of an EBS volume. It can be used to create a new EBS volume or as part of the storage image behind an AMI.

Use snapshots to back up data volumes, create recovery points before risky changes, restore volumes, or support backup and disaster recovery.

A snapshot is not a complete machine launch template: it primarily captures the selected EBS volume's data.

## 8. AMI vs EBS snapshot

| Feature | AMI | EBS snapshot |
|---|---|---|
| Main purpose | Template to launch EC2 instances | Point-in-time backup of an EBS volume |
| Contents | OS image and launch-related metadata; may reference EBS snapshots for volumes | Volume data at a point in time |
| Launch EC2 directly? | Yes, using the AMI | Not directly as a normal instance launch; create a volume from it or use an AMI |
| Preserves server setup? | Can preserve OS/software configuration on included volumes | Only the selected volume's data |
| Typical use | Golden images, repeatable deployments, image-based recovery | Data backup, restore, volume cloning |
| Relationship | An EBS-backed AMI commonly uses snapshots for its volume data | Snapshots may be created as part of AMI creation |

**Example — snapshot:** Take a snapshot of an application data volume before changing its contents, then create a volume from that snapshot if restoration is needed.

**Example — AMI:** Configure a server with the OS and application dependencies, create an AMI, and use it to launch more instances with the same base setup.

**Remember:** A snapshot backs up a volume; an AMI is a launch template for an EC2 instance.

## 9. Learning log

**Date:** 21 October 2026  
**Topic:** EC2 instance types, purchase options, Linux resource commands, AMIs, and snapshots

### What I learned
- Instance types define vCPU, RAM, network, and storage capabilities.
- Categories include general purpose, compute optimized, memory optimized, storage optimized, accelerated computing, and HPC optimized.
- On-Demand, Savings Plans, Reserved Instances, Spot, and dedicated options suit different billing and isolation needs.
- `free -m` checks memory; `lscpu` and `/proc/cpuinfo` show CPU information.
- An AMI is a reusable template for launching an EC2 instance.
- An EBS snapshot is a point-in-time backup of an EBS volume.
- AMIs and snapshots may incur storage charges.

### Practical checklist
- [ ] Inspected an instance type in the AWS Console
- [ ] Compared CPU/RAM needs for dev, test, and production
- [ ] Reviewed purchase options and current pricing
- [ ] Ran `free -m`
- [ ] Ran `lscpu`
- [ ] Inspected `/proc/cpuinfo`
- [ ] Reviewed AMI details and creation workflow
- [ ] Compared AMIs and EBS snapshots
- [ ] Reviewed storage charges and cleanup requirements

Mark items complete only after doing them yourself.

## 10. Interview questions

1. **What is an EC2 instance type?** A specification of compute, memory, network, and storage capabilities.
2. **Name the major instance categories.** General purpose, compute optimized, memory optimized, storage optimized, accelerated computing, and HPC optimized.
3. **When would you choose compute optimized?** For workloads needing high CPU performance relative to memory.
4. **When would you choose memory optimized?** For in-memory datasets, caches, and memory-intensive databases or analytics.
5. **When is Spot appropriate?** When interruption is acceptable and the workload can retry or checkpoint.
6. **Instance family vs purchase option?** Family determines resources; purchase option determines billing/commitment.
7. **What does `free -m` show?** RAM and swap statistics in MiB.
8. **What does `lscpu` show?** CPU architecture and visible processor configuration.
9. **What is `/proc/cpuinfo`?** A Linux procfs file exposing CPU information.
10. **What is an AMI?** A reusable image and launch template used to create EC2 instances.
11. **What is an EBS snapshot?** A point-in-time backup of an EBS volume.
12. **Can an instance be launched directly from an EBS snapshot?** Not directly as a normal instance launch; create a volume or use an AMI for launching.
13. **When would you create an AMI?** For consistent repeatable launches, scaling, or image-based recovery.
14. **Can AMIs incur storage charges?** Their backing EBS snapshots can incur storage charges.
15. **What should guide production instance sizing?** Load tests, real metrics, application needs, availability targets, and cost.

## References
- EC2 instance types: https://aws.amazon.com/ec2/instance-types/
- EC2 purchase options: https://aws.amazon.com/ec2/pricing/
- AMIs: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html
- EBS snapshots: https://docs.aws.amazon.com/ebs/latest/userguide/ebs-snapshots.html
