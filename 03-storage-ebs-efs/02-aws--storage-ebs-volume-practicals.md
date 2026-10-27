# AWS EBS Volume Practicals

## Objective

Learn how to create, attach, format, mount, verify, and permanently mount an Amazon EBS volume on a Linux EC2 instance.

## Architecture

```text
AWS Region
   |
   └── Availability Zone (AZ)
          |
          ├── EC2 Instance (Amazon Linux)
          |
          └── EBS Volume (same AZ as EC2)
                    |
                    └── Attach to EC2
                              |
                              └── Linux Block Device
                                        |
                                        ├── Create Filesystem
                                        ├── Create Mount Directory
                                        ├── Mount Volume
                                        ├── Verify Storage
                                        └── Configure /etc/fstab
```

## Prerequisites

- An AWS account.
- A running Linux EC2 instance.
- Permission to create and attach EBS volumes.
- SSH access or AWS Systems Manager Session Manager access.
- `sudo` privileges on the EC2 instance.

> **Important:** The commands below are intended for Amazon Linux and similar Linux distributions. Device names and filesystem tools may differ on other distributions.

---

# Practical 1: Create an EBS Volume

## Step 1: Open the EC2 Console

1. Sign in to the AWS Management Console.
2. Open the EC2 service.
3. Select **Elastic Block Store → Volumes**.
4. Click **Create volume**.

## Step 2: Configure the Volume

Example configuration:

| Setting | Example Value |
|---|---|
| Volume type | gp3 |
| Size | 5 GiB |
| Availability Zone | Same AZ as the EC2 instance |
| Encryption | Enabled, recommended |
| Snapshot | None for this new empty volume |

Click **Create volume**.

## Step 3: Verify the Volume

The new volume should initially show a state such as:

```text
State: Available
```

An `Available` volume is not currently attached to an EC2 instance.

> **Important:** An EBS volume must normally be in the same Availability Zone as the EC2 instance to which you want to attach it.

---

# Practical 2: Attach the EBS Volume to an EC2 Instance

## Step 1: Select the Volume

1. Go to **EC2 → Elastic Block Store → Volumes**.
2. Select the volume you created.
3. Choose **Actions → Attach volume**.

## Step 2: Select the EC2 Instance

1. Select your running EC2 instance.
2. Choose a device name if the console requests one.
3. Click **Attach volume**.

Example:

```text
EC2 Instance: devops-linux-server
EBS Volume:   vol-0123456789abcdef0
Device Name:  /dev/sdf
```

The volume ID and device name above are examples only.

## Step 3: Verify Attachment

Return to the Volumes page and confirm that the volume is attached.

The volume state should show:

```text
In-use
```

This means the volume is attached, but **it does not automatically mean that a filesystem has been created or that the volume is mounted**.

---

# Practical 3: Connect to the EC2 Instance

Connect using SSH or Session Manager.

For SSH:

```bash
ssh -i my-key.pem ec2-user@<EC2-PUBLIC-IP>
```

Replace `my-key.pem` and `<EC2-PUBLIC-IP>` with your actual key file and public IP address.

Check the current user:

```bash
whoami
```

Check the hostname:

```bash
hostname
```

Check the Linux version:

```bash
cat /etc/os-release
```

---

# Practical 4: Identify the Attached EBS Disk

After connecting to the EC2 instance, identify the newly attached disk before making changes.

## Command 1: List Block Devices

```bash
lsblk
```

Example output:

```text
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0  10G  0 disk
├─nvme0n1p1   259:1    0  10G  0 part /
nvme1n1       259:2    0   5G  0 disk
```

In this example:

- `nvme0n1` is the existing root disk.
- `nvme0n1p1` is the root partition.
- `nvme1n1` is the newly attached 5 GiB disk.
- The new disk has no partitions or mount points.

Your actual device names and output may differ.

## Command 2: Display Filesystem Information

```bash
lsblk -f
```

This displays filesystem types, UUIDs, labels, and mount points.

## Command 3: Display Disk Usage

```bash
df -hT
```

This shows mounted filesystems and their available space.

**Important:** `df -hT` normally does not display an unmounted EBS volume. Use `lsblk` to identify unmounted disks.

## Command 4: Check Disk Devices

```bash
sudo fdisk -l
```

This lists detected disks and partition information.

## How Do You Identify the Correct Disk?

Compare the output of `lsblk` before and after attaching the volume.

For example:

```text
Before attachment:
nvme0n1

After attachment:
nvme0n1
nvme1n1
```

Here, `nvme1n1` is the newly appearing device.

> **Critical safety warning:** Never format a disk simply because its name looks like a new disk. Confirm its size, attachment, existing filesystem, and intended purpose. Formatting the wrong disk can destroy data.

---

# Practical 5: Check Whether the Disk Already Has a Filesystem

Run:

```bash
lsblk -f
```

You can also inspect the device:

```bash
sudo file -s /dev/nvme1n1
```

Replace `/dev/nvme1n1` with the actual device name identified on your instance.

Possible output for a new, empty disk:

```text
/dev/nvme1n1: data
```

This is consistent with an unformatted disk, but always verify the device and its intended use before proceeding.

If the disk already contains a filesystem or data, **do not format it**. Formatting an existing filesystem can destroy data.

For this practical, assume the volume is brand new and empty.

---

# Practical 6: Create a Filesystem

A filesystem allows Linux to organize and manage files on the volume.

Common Linux filesystems include:

- `ext4`
- `xfs`

For this practical, we will use `xfs`.

## Step 1: Check Whether XFS Tools Are Available

```bash
which mkfs.xfs
```

If the command is not found, install the appropriate XFS utilities for your distribution.

For Amazon Linux 2023, you can use:

```bash
sudo dnf install -y xfsprogs
```

For Amazon Linux 2, you can use:

```bash
sudo yum install -y xfsprogs
```

## Step 2: Create the XFS Filesystem

**Only do this after confirming that the disk is new, empty, and safe to format.**

```bash
sudo mkfs.xfs /dev/nvme1n1
```

Replace the device name with your actual EBS device.

Example output:

```text
meta-data=/dev/nvme1n1
...
data     =                       bsize=4096
...
```

The exact output varies by system.

### What Does `mkfs.xfs` Mean?

- `mkfs` means make filesystem.
- `xfs` specifies the XFS filesystem type.
- `/dev/nvme1n1` identifies the target block device.

This command creates a filesystem directly on the whole disk.

For this simple practical, we are not creating a partition table or a partition. That is a valid setup for a dedicated data volume.

## Step 3: Verify the Filesystem

```bash
lsblk -f
```

Example:

```text
NAME      FSTYPE LABEL UUID                                 MOUNTPOINTS
nvme0n1
└─nvme0n1p1 xfs        existing-root-uuid                   /
nvme1n1   xfs         12345678-abcd-1234-abcd-123456789abc
```

The UUID above is an example.

You should now see `xfs` under the `FSTYPE` column for the new disk.

---

# Practical 7: Create a Mount Directory

Linux uses directories to make filesystems accessible.

For this practical, we will use:

```text
/data/ebs
```

## Step 1: Create the Directory

```bash
sudo mkdir -p /data/ebs
```

### Explanation

- `sudo` runs the command with administrative privileges.
- `mkdir` creates a directory.
- `-p` creates missing parent directories and does not fail if the directory already exists.
- `/data/ebs` is our mount point.

## Step 2: Verify the Directory

```bash
ls -ld /data/ebs
```

Example output:

```text
drwxr-xr-x. 2 root root 6 Oct 27 20:00 /data/ebs
```

The exact output varies.

At this stage, `/data/ebs` is just a directory on the existing root filesystem. The EBS volume is not mounted there yet.

---

# Practical 8: Mount the EBS Volume

## Step 1: Mount the Filesystem

```bash
sudo mount /dev/nvme1n1 /data/ebs
```

Replace `/dev/nvme1n1` with the correct device name.

The command connects the filesystem on the EBS volume to the `/data/ebs` directory.

## Step 2: Verify the Mount

```bash
findmnt /data/ebs
```

Example output:

```text
TARGET     SOURCE         FSTYPE OPTIONS
/data/ebs  /dev/nvme1n1   xfs    rw,relatime,...
```

You can also run:

```bash
lsblk -f
```

The new disk should now show `/data/ebs` as its mount point.

## Step 3: Check Disk Space

```bash
df -hT /data/ebs
```

Example output:

```text
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/nvme1n1   xfs   5.0G   50M  5.0G   1% /data/ebs
```

Values are illustrative.

### What Does This Prove?

- The filesystem exists.
- The volume is mounted.
- Linux can report the available capacity.

**Important:** The `mount` command usually creates a temporary mount configuration. If the instance reboots, this mount may not return automatically. We will configure a persistent mount later.

---

# Practical 9: Create Files and Test the Volume

Now verify that the EBS volume can store data.

## Step 1: Create a Test File

```bash
sudo touch /data/ebs/test-file.txt
```

## Step 2: Write Data into the File

```bash
echo "This file is stored on an EBS volume" | sudo tee /data/ebs/test-file.txt
```

## Step 3: Read the File

```bash
cat /data/ebs/test-file.txt
```

Expected output:

```text
This file is stored on an EBS volume
```

## Step 4: List the Files

```bash
ls -lh /data/ebs
```

## Step 5: Verify the Filesystem and Mount Point

```bash
df -hT /data/ebs
```

```bash
findmnt /data/ebs
```

```bash
lsblk -f
```

These commands help verify that the data directory is backed by the attached EBS filesystem.

---

# Practical 10: Configure Permanent Mounting

## Why Is Permanent Mounting Required?

Suppose you mount the EBS volume with:

```bash
sudo mount /dev/nvme1n1 /data/ebs
```

After a reboot, the system might not mount that filesystem automatically.

To make the mount persistent, configure `/etc/fstab`.

However, using a device name such as `/dev/nvme1n1` is less reliable than using the filesystem UUID because device names can change.

**Recommended approach: Use the filesystem UUID in `/etc/fstab`.**

## Step 1: Find the Filesystem UUID

Run:

```bash
sudo blkid /dev/nvme1n1
```

Example output:

```text
/dev/nvme1n1: UUID="12345678-abcd-1234-abcd-123456789abc" TYPE="xfs"
```

The UUID above is only an example.

You can also use:

```bash
lsblk -f
```

Copy the actual UUID from your system.

## Step 2: Back Up `/etc/fstab`

Before editing the file, create a backup:

```bash
sudo cp /etc/fstab /etc/fstab.bak
```

Verify the backup:

```bash
ls -l /etc/fstab.bak
```

## Step 3: Edit `/etc/fstab`

Open the file:

```bash
sudo vi /etc/fstab
```

Add a new line at the end using your actual UUID:

```text
UUID=12345678-abcd-1234-abcd-123456789abc /data/ebs xfs defaults,nofail 0 2
```

**Replace the example UUID with the actual UUID from your volume.**

### Explanation of the Fields

| Field | Meaning |
|---|---|
| `UUID=...` | Identifies the filesystem |
| `/data/ebs` | Mount point |
| `xfs` | Filesystem type |
| `defaults,nofail` | Standard mount options; `nofail` allows boot to continue if the volume is unavailable |
| `0` | Disables the legacy `dump` backup flag |
| `2` | Filesystem check pass value for filesystems that use `fsck` |

For XFS, the filesystem check pass value does not cause the usual boot-time `fsck` check performed for some other filesystems. The important part of this configuration is the correct UUID, mount point, filesystem type, and options.

### Why Use `nofail`?

The `nofail` option helps prevent a missing or unavailable data volume from blocking the boot process.

However, applications that depend on `/data/ebs` might still fail if the volume is unavailable. Production systems may require additional mount and application dependency configuration.

## Step 4: Save the File

If you are using `vi`:

1. Press `Esc`.
2. Type `:wq`.
3. Press `Enter`.

## Step 5: Verify the `/etc/fstab` Entry

```bash
tail -n 5 /etc/fstab
```

Confirm that your new entry has the correct UUID, mount point, and filesystem type.

---

# Practical 11: Test the Permanent Mount Configuration

**Do not reboot before testing `/etc/fstab`.** A bad entry can cause boot or mount problems.

## Step 1: Test the Configuration Syntax

Run:

```bash
sudo findmnt --verify --verbose
```

Review the output for errors.

## Step 2: Unmount the Volume

```bash
sudo umount /data/ebs
```

Make sure no process is actively using the mount point. If the filesystem is busy, investigate the process rather than forcing an unmount.

## Step 3: Confirm It Is Unmounted

```bash
findmnt /data/ebs
```

If the command returns no matching mount, the volume is no longer mounted there.

You can also check:

```bash
lsblk -f
```

The filesystem should no longer show `/data/ebs` as its mount point.

## Step 4: Mount Using `/etc/fstab`

```bash
sudo mount /data/ebs
```

Because `/etc/fstab` contains the entry, Linux can use that configuration to mount the filesystem.

Alternatively, test all filesystems configured in `/etc/fstab` that are not already mounted:

```bash
sudo mount -a
```

**Important:** `mount -a` can attempt to mount other configured filesystems, not just this EBS volume. Review the configuration first.

## Step 5: Verify the Mount Again

```bash
findmnt /data/ebs
```

```bash
df -hT /data/ebs
```

```bash
lsblk -f
```

The EBS volume should be mounted at `/data/ebs`.

## Step 6: Confirm Your Test File Still Exists

```bash
cat /data/ebs/test-file.txt
```

Expected output:

```text
This file is stored on an EBS volume
```

If the file is present, you have verified that the data is accessible after unmounting and remounting the filesystem.

## Step 7: Reboot Test (Optional)

Once the configuration has passed validation, you can test automatic mounting after a reboot:

```bash
sudo reboot
```

Reconnect to the instance and run:

```bash
findmnt /data/ebs
```

```bash
df -hT /data/ebs
```

```bash
cat /data/ebs/test-file.txt
```

If the mount point is present and the test file is readable, the persistent mount is working.

---

# Practical 12: Useful EBS and Linux Verification Commands

| Purpose | Command |
|---|---|
| List block devices | `lsblk` |
| List filesystems and UUIDs | `lsblk -f` |
| Inspect disk partitions | `sudo fdisk -l` |
| Identify filesystem type | `sudo blkid` |
| Inspect one device | `sudo file -s /dev/nvme1n1` |
| Show mounted filesystem usage | `df -hT` |
| Check a particular mount | `findmnt /data/ebs` |
| Check `/etc/fstab` | `cat /etc/fstab` |
| Validate mount configuration | `sudo findmnt --verify --verbose` |
| Mount all eligible `/etc/fstab` entries | `sudo mount -a` |
| Unmount a filesystem | `sudo umount /data/ebs` |
| Check disk and filesystem usage | `df -hT /data/ebs` |
| List directory contents | `ls -lh /data/ebs` |
| Read the test file | `cat /data/ebs/test-file.txt` |

> Replace `/dev/nvme1n1` with your actual device name when necessary.

---

# Practical 13: Troubleshooting Common Issues

## Issue 1: EBS Volume Is Not Visible

Check:

```bash
lsblk
```

Possible causes:

- The volume is not attached to the instance.
- The attachment is still being processed.
- You are checking the wrong device name.

Solution:

1. Check the volume in the AWS EC2 console.
2. Confirm the volume is attached to the intended instance.
3. Confirm the Availability Zone is correct.
4. Run `lsblk` again.

## Issue 2: `mkfs.xfs` Command Not Found

Check:

```bash
which mkfs.xfs
```

Install the XFS utilities if needed:

```bash
sudo dnf install -y xfsprogs
```

For Amazon Linux 2:

```bash
sudo yum install -y xfsprogs
```

## Issue 3: Mount Fails

Check the filesystem:

```bash
lsblk -f
```

Check the mount point:

```bash
ls -ld /data/ebs
```

Check recent kernel messages:

```bash
sudo dmesg | tail -n 50
```

Possible causes:

- Incorrect device name.
- Filesystem has not been created.
- Wrong filesystem type.
- Mount directory does not exist.
- Filesystem or device errors.

## Issue 4: `mount -a` Returns an Error

Check the `/etc/fstab` entry:

```bash
cat /etc/fstab
```

Verify the filesystem UUID:

```bash
sudo blkid
```

Validate the configuration:

```bash
sudo findmnt --verify --verbose
```

Common mistakes include:

- Incorrect UUID.
- Incorrect mount point.
- Incorrect filesystem type.
- Missing spaces between fields.
- Typographical errors.

Correct the entry and run the validation commands again.

## Issue 5: The Volume Is Mounted, but the Application Cannot Write

Check permissions:

```bash
ls -ld /data/ebs
```

Check the current user:

```bash
whoami
```

Test writing with administrative privileges:

```bash
sudo touch /data/ebs/admin-test.txt
```

If the application runs as a non-root user, configure the directory ownership and permissions for that application user. Avoid using `chmod 777` as a general fix.

## Issue 6: The Volume Appears to Have No Data After Mounting

When you mount a filesystem on a directory, the mounted filesystem hides the directory's previous contents until you unmount it.

For example, files that existed in `/data/ebs` on the root filesystem before the mount will not be visible while the EBS volume is mounted there.

This does not mean those files were automatically deleted.

## Issue 7: The EBS Device Name Is Different from the Console Name

On Nitro-based EC2 instances, a requested device name such as `/dev/sdf` can appear inside Linux as an NVMe device such as `/dev/nvme1n1`.

Check:

```bash
lsblk
```

Use the actual Linux device shown by the operating system. Do not assume the console device name and Linux device name will always be identical.

---

# Practical 14: Important Production Considerations

1. **Backups:** Use EBS snapshots to protect important volume data.
2. **Encryption:** Enable EBS encryption to protect data at rest.
3. **Filesystem:** Choose a suitable filesystem for the workload.
4. **Persistent mounting:** Use filesystem UUIDs in `/etc/fstab`.
5. **Permissions:** Give applications only the filesystem permissions they need.
6. **Monitoring:** Monitor free disk space and relevant EC2/EBS metrics.
7. **Availability Zone:** An EBS volume is tied to a particular Availability Zone.
8. **Deletion protection:** Understand the volume's lifecycle and deletion settings before terminating an EC2 instance.
9. **Detach safely:** Stop applications that are writing to the volume and unmount it before detaching it from the instance.
10. **Data safety:** Never format a disk or delete a volume until you have verified that it contains no required data.

---

# Practical 15: Complete Command Workflow

This is the command sequence for the lab after you have created and attached a **new, empty EBS volume**.

> **Before running the formatting command:** Verify the device name with `lsblk` and confirm that it is the intended empty volume. Formatting the wrong device can destroy data.

## Step 1: Identify the Device

```bash
lsblk
```

```bash
lsblk -f
```

For this example, assume the new device is `/dev/nvme1n1`.

## Step 2: Create the Filesystem

```bash
sudo mkfs.xfs /dev/nvme1n1
```

## Step 3: Create the Mount Directory

```bash
sudo mkdir -p /data/ebs
```

## Step 4: Mount the Volume

```bash
sudo mount /dev/nvme1n1 /data/ebs
```

## Step 5: Verify the Mount

```bash
findmnt /data/ebs
```

```bash
df -hT /data/ebs
```

```bash
lsblk -f
```

## Step 6: Write and Read Data

```bash
echo "AWS EBS practical completed" | sudo tee /data/ebs/test-file.txt
```

```bash
cat /data/ebs/test-file.txt
```

## Step 7: Get the UUID

```bash
sudo blkid /dev/nvme1n1
```

Copy the actual UUID from the output.

## Step 8: Back Up the Configuration

```bash
sudo cp /etc/fstab /etc/fstab.bak
```

## Step 9: Add the Persistent Mount Entry

```bash
sudo vi /etc/fstab
```

Add the following line, replacing the example UUID with your actual UUID:

```text
UUID=YOUR-ACTUAL-UUID /data/ebs xfs defaults,nofail 0 2
```

## Step 10: Validate the Configuration

```bash
sudo findmnt --verify --verbose
```

## Step 11: Test the Persistent Mount

```bash
sudo umount /data/ebs
```

```bash
sudo mount /data/ebs
```

## Step 12: Verify the Data

```bash
findmnt /data/ebs
```

```bash
df -hT /data/ebs
```

```bash
cat /data/ebs/test-file.txt
```

If these commands succeed, the volume is formatted, mounted, and configured for persistent mounting.

---

# Interview Questions and Answers

## 1. What is Amazon EBS?

Amazon Elastic Block Store (EBS) is a block storage service that provides persistent storage volumes for use with EC2 instances.

## 2. What is the difference between attaching and mounting an EBS volume?

**Attaching** connects the EBS volume to the EC2 instance at the infrastructure level.

**Mounting** makes a filesystem on the device accessible through a Linux directory.

Attaching a volume does not automatically format or mount it.

## 3. Why do we use `mkfs`?

`mkfs` creates a filesystem on a storage device. For example, `mkfs.xfs` creates an XFS filesystem.

Formatting should only be performed when appropriate for the device's existing data and filesystem.

## 4. What is a mount point?

A mount point is a directory through which a mounted filesystem is accessed.

Example:

```text
/data/ebs
```

## 5. What is the purpose of `/etc/fstab`?

`/etc/fstab` defines filesystems that Linux can mount automatically, including during boot.

## 6. Why is a UUID preferred over `/dev/nvme1n1` in `/etc/fstab`?

Device names can change. A filesystem UUID provides a more stable way to identify the filesystem.

## 7. What does `mount -a` do?

It attempts to mount eligible filesystems defined in `/etc/fstab` that are not already mounted.

## 8. What is the purpose of `lsblk`?

`lsblk` displays block devices, their relationships, sizes, partitions, and mount points.

## 9. What is the purpose of `df -hT`?

It displays mounted filesystem usage in human-readable units and includes filesystem types.

## 10. What happens if you delete an EC2 instance that has an attached EBS volume?

The result depends on the volume's configuration and the instance termination behavior.

- A root volume may be deleted if `DeleteOnTermination` is enabled.
- A separate data volume can be retained if configured appropriately.

Always verify the volume's deletion settings before terminating the instance.

## 11. Can an EBS volume be attached to an EC2 instance in a different Availability Zone?

An ordinary EBS volume must be attached to an EC2 instance in the same Availability Zone. To use the data in another Availability Zone, you can create a snapshot and create a new volume from it in the target Availability Zone.

## 12. What is the difference between EBS and EFS?

- **EBS:** Block storage, typically attached to an EC2 instance as a disk.
- **EFS:** Managed shared file storage that multiple clients can access through a network filesystem.

---

# Lab Completion Checklist

- [ ] Created an EBS volume.
- [ ] Selected the same Availability Zone as the EC2 instance.
- [ ] Attached the volume to the instance.
- [ ] Identified the new Linux block device.
- [ ] Checked whether a filesystem already existed.
- [ ] Created an XFS filesystem on the new empty volume.
- [ ] Created `/data/ebs`.
- [ ] Mounted the EBS volume.
- [ ] Verified the mount using `findmnt`.
- [ ] Verified storage capacity using `df -hT`.
- [ ] Created and read a test file.
- [ ] Retrieved the filesystem UUID.
- [ ] Backed up `/etc/fstab`.
- [ ] Added the persistent mount entry.
- [ ] Validated the configuration.
- [ ] Unmounted and remounted the filesystem using `/etc/fstab`.
- [ ] Verified that the test file remained accessible.

---

# Key Learning Summary

```text
Create EBS Volume
       |
       v
Attach to EC2
       |
       v
Identify Disk with lsblk
       |
       v
Check Existing Filesystem
       |
       v
Create Filesystem with mkfs.xfs
       |
       v
Create Mount Directory with mkdir
       |
       v
Mount the Volume
       |
       v
Verify with findmnt, lsblk and df
       |
       v
Create and Read a Test File
       |
       v
Get Filesystem UUID
       |
       v
Add Entry to /etc/fstab
       |
       v
Validate Configuration
       |
       v
Unmount and Remount
       |
       v
Verify Persistent Mount
```

**Main takeaway:** Creating an EBS volume in AWS is only the first step. To use it as Linux storage, you must identify the correct disk, prepare its filesystem if needed, mount it at a directory, verify access, and configure persistent mounting when required.