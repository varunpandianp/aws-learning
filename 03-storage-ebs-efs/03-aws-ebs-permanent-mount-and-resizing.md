# AWS EBS Permanent Mounting and Resizing

## Learning Objectives

In this practical, we will learn how to:

1. Mount an EBS volume permanently.
2. Configure `/etc/fstab` using a filesystem UUID.
3. Validate the configuration using `mount -a`.
4. Increase the size of an EBS volume in AWS.
5. Expand the Linux partition when required.
6. Resize the filesystem to use the additional space.
7. Verify the new storage capacity.
8. Troubleshoot common mounting and resizing problems.

---

# Part 1: Permanent Mounting of an EBS Volume

## 1. What Is Permanent Mounting?

When we mount an EBS volume manually, for example:

```bash
sudo mount /dev/nvme1n1 /data/ebs
```

the filesystem is mounted at `/data/ebs`.

However, a manual mount is not automatically configured to return after a reboot.

To configure automatic mounting, we add an entry to:

```text
/etc/fstab
```

Linux uses this file to determine which filesystems should be mounted automatically.

### Manual Mount vs. Permanent Mount

| Manual Mount | Permanent Mount |
|---|---|
| Configured using `mount` | Configured using `/etc/fstab` |
| May not return after reboot | Configured to mount automatically at boot |
| Can use a device name | Usually uses a filesystem UUID |
| Suitable for temporary mounting | Suitable for persistent storage configuration |

---

## 2. Identify the EBS Device

Connect to your EC2 instance and run:

```bash
lsblk
```

Example output:

```text
NAME          SIZE TYPE MOUNTPOINTS
nvme0n1        10G disk
└─nvme0n1p1    10G part /
nvme1n1         5G disk
```

In this example:

- `/dev/nvme0n1` is the root disk.
- `/dev/nvme0n1p1` is the root partition.
- `/dev/nvme1n1` is a new, unpartitioned EBS data disk.

Your actual device names may differ.

Check filesystem information:

```bash
lsblk -f
```

**Important:** Always identify the correct disk before formatting, partitioning, or resizing it. Never assume that a particular device name always refers to the same disk.

---

## 3. Create a Filesystem on a New EBS Volume

Only perform this step if the EBS volume is new, empty, and intended to be formatted.

For an XFS filesystem:

```bash
sudo mkfs.xfs /dev/nvme1n1
```

For an ext4 filesystem instead:

```bash
sudo mkfs.ext4 /dev/nvme1n1
```

**Do not run both commands on the same volume.** Choose the filesystem you intend to use.

Formatting an existing filesystem can destroy data.

Verify the filesystem:

```bash
lsblk -f
```

---

## 4. Create a Mount Directory

Create a directory to access the EBS filesystem:

```bash
sudo mkdir -p /data/ebs
```

Verify the directory:

```bash
ls -ld /data/ebs
```

The directory is called the **mount point**.

---

## 5. Mount the EBS Volume Manually

For an XFS filesystem:

```bash
sudo mount /dev/nvme1n1 /data/ebs
```

Verify the mount:

```bash
findmnt /data/ebs
```

Check storage usage:

```bash
df -hT /data/ebs
```

Check the device and mount point:

```bash
lsblk -f
```

---

## 6. Find the Filesystem UUID

A UUID is a unique identifier assigned to a filesystem.

Run:

```bash
sudo blkid /dev/nvme1n1
```

Example output:

```text
/dev/nvme1n1: UUID="12345678-abcd-1234-abcd-123456789abc" TYPE="xfs"
```

The UUID above is only an example. Use the actual UUID returned by your instance.

You can also run:

```bash
lsblk -f
```

### Why Use UUID Instead of a Device Name?

A device name such as `/dev/nvme1n1` can change depending on how devices are discovered.

Using a filesystem UUID in `/etc/fstab` makes the mount configuration more reliable.

---

## 7. Back Up `/etc/fstab`

Before modifying the file, create a backup:

```bash
sudo cp /etc/fstab /etc/fstab.bak
```

Verify that the backup exists:

```bash
ls -l /etc/fstab.bak
```

---

## 8. Add the Permanent Mount Entry

Open the file:

```bash
sudo vi /etc/fstab
```

Add a new line at the end.

For an XFS filesystem:

```text
UUID=YOUR-ACTUAL-UUID /data/ebs xfs defaults,nofail 0 2
```

For an ext4 filesystem, use:

```text
UUID=YOUR-ACTUAL-UUID /data/ebs ext4 defaults,nofail 0 2
```

Replace `YOUR-ACTUAL-UUID` with the real filesystem UUID.

Do not copy the placeholder literally.

### Understanding `/etc/fstab`

Example:

```text
UUID=12345678-abcd-1234-abcd-123456789abc /data/ebs xfs defaults,nofail 0 2
```

| Field | Meaning |
|---|---|
| `UUID=...` | Identifies the filesystem |
| `/data/ebs` | Mount point |
| `xfs` | Filesystem type |
| `defaults,nofail` | Mount options |
| `0` | Legacy `dump` backup flag |
| `2` | Filesystem check pass value for filesystems that support this check |

The `nofail` option allows the system to continue booting if the volume is unavailable. However, an application that depends on the volume may still fail, so production environments may need additional dependency configuration.

### Save the File in `vi`

1. Press `Esc`.
2. Type `:wq`.
3. Press `Enter`.

---

## 9. Validate `/etc/fstab`

Before rebooting, check the configuration:

```bash
sudo findmnt --verify --verbose
```

Review the output for errors.

Check the file contents:

```bash
cat /etc/fstab
```

Common mistakes include:

- Incorrect UUID.
- Incorrect filesystem type.
- Incorrect mount point.
- Missing spaces between fields.
- Typographical errors.

---

## 10. Use `mount -a`

This is an important command in AWS Linux administration.

```bash
sudo mount -a
```

### What Does `mount -a` Do?

It attempts to mount eligible filesystems configured in `/etc/fstab` that are not already mounted.

If the EBS volume is already mounted at `/data/ebs`, the command generally leaves that existing mount in place.

**Important:** `mount -a` does not resize the EBS volume or filesystem. It only attempts to mount filesystems configured in `/etc/fstab`.

It can also attempt to mount other filesystems listed in the file, so validate the configuration first.

Verify the EBS mount:

```bash
findmnt /data/ebs
```

```bash
df -hT /data/ebs
```

```bash
lsblk -f
```

### Test the Configuration by Remounting

After confirming that the volume is not being used by an application, you can test the entry by unmounting and mounting it again:

```bash
sudo umount /data/ebs
```

Then:

```bash
sudo mount /data/ebs
```

Verify:

```bash
findmnt /data/ebs
```

If the filesystem mounts correctly, the `/etc/fstab` entry is working for this mount.

Avoid unmounting a production filesystem while applications are using it.

---

# Part 2: Resize an EBS Volume

## 11. What Does EBS Volume Resizing Mean?

Suppose you created an EBS volume with a capacity of 5 GiB.

Later, your application needs more storage.

You can increase the EBS volume size in AWS, for example, from 5 GiB to 10 GiB.

However, **increasing the EBS volume size does not necessarily mean that Linux can immediately use all the new capacity**.

There are three possible layers:

```text
AWS EBS Volume
       |
       v
Linux Block Device
       |
       v
Partition, if one exists
       |
       v
Filesystem
```

You must ensure that the operating system, partition, and filesystem recognize the additional space.

### Example

Before resizing:

```text
EBS Volume:       5 GiB
Linux Disk:       5 GiB
Filesystem:       5 GiB
```

After increasing the EBS volume in AWS:

```text
EBS Volume:      10 GiB
Linux Disk:      10 GiB
Filesystem:       5 GiB
```

After expanding the filesystem:

```text
EBS Volume:      10 GiB
Linux Disk:      10 GiB
Filesystem:      Uses the expanded capacity
```

The filesystem's reported usable space will be slightly less than the nominal volume capacity because of filesystem metadata and other overhead.

---

## 12. Step 1: Check the Current Capacity

Before resizing, run:

```bash
lsblk
```

Check filesystem usage:

```bash
df -hT
```

Check the target mount point:

```bash
df -hT /data/ebs
```

Record the current size so you can compare it with the result after resizing.

---

## 13. Step 2: Increase the EBS Volume Size in AWS

1. Sign in to the AWS Management Console.
2. Open **EC2**.
3. Select **Elastic Block Store → Volumes**.
4. Select the EBS volume attached to your instance.
5. Choose **Actions → Modify volume**.
6. Increase the volume size, for example, from `5 GiB` to `10 GiB`.
7. Review the change and submit the modification.
8. Check the modification status until the requested change has been applied.

The volume can often be enlarged while it is attached and in use, but the exact operational requirements depend on the volume and workload. For production workloads, review the AWS operational guidance and consider the impact on applications.

**Important:** EBS volume expansion is not the same as reducing the volume size. Do not assume that an existing EBS volume can be shrunk in place.

---

## 14. Step 3: Verify the New Disk Size in Linux

After the AWS modification, run:

```bash
lsblk
```

If necessary, check the device:

```bash
sudo fdisk -l
```

Example:

```text
NAME          SIZE TYPE MOUNTPOINTS
nvme0n1        10G disk
└─nvme0n1p1    10G part /
nvme1n1        10G disk
```

The new data disk is now reported as 10 GiB.

Your actual output may differ.

If Linux still shows the old size, check the AWS volume modification status and operating system device information before continuing.

---

# Part 3: Resize the Filesystem

## 15. First Identify the Filesystem Type

Run:

```bash
lsblk -f
```

Or:

```bash
df -hT /data/ebs
```

Check the `FSTYPE` or `Type` column.

The command you use depends on the filesystem:

| Filesystem | Filesystem expansion command |
|---|---|
| ext2 | `resize2fs` |
| ext3 | `resize2fs` |
| ext4 | `resize2fs` |
| XFS | `xfs_growfs` |

**Do not use `resize2fs` on an XFS filesystem.**

---

# Part 4: Resizing an ext4 Filesystem

## 16. Scenario A: ext4 Filesystem Directly on the Whole EBS Disk

Suppose:

```text
Device:     /dev/nvme1n1
Filesystem: ext4
Mount point: /data/ebs
```

In this example, the filesystem is directly on the whole disk, with no partition.

Verify the device and filesystem first:

```bash
lsblk -f
```

```bash
df -hT /data/ebs
```

After increasing the EBS volume size in AWS, run:

```bash
sudo resize2fs /dev/nvme1n1
```

This tells `resize2fs` to expand the ext filesystem to use the available space on that block device.

Verify the new size:

```bash
df -hT /data/ebs
```

Also check:

```bash
lsblk -f
```

### Important

The command above is correct only when the ext filesystem is directly on `/dev/nvme1n1`.

If the filesystem is on a partition such as `/dev/nvme1n1p1`, use the partition device after ensuring that the partition itself has been expanded.

---

# Part 5: Resizing an XFS Filesystem

## 17. Scenario B: XFS Filesystem Directly on the Whole EBS Disk

Suppose:

```text
Device:      /dev/nvme1n1
Filesystem:  xfs
Mount point: /data/ebs
```

After increasing the EBS volume size in AWS, check:

```bash
lsblk -f
```

Confirm that `/dev/nvme1n1` is the device containing the XFS filesystem mounted at `/data/ebs`.

Run:

```bash
sudo xfs_growfs /data/ebs
```

Notice that the command uses the **mounted directory**, not the block device.

Verify:

```bash
df -hT /data/ebs
```

```bash
lsblk -f
```

### Why Do We Use `xfs_growfs`?

`xfs_growfs` expands an XFS filesystem.

An XFS filesystem must be mounted to grow it.

**Remember:**

- ext4 → `resize2fs`
- XFS → `xfs_growfs`

---

# Part 6: If the EBS Volume Uses a Partition

## 18. How to Identify a Partitioned Disk

Run:

```bash
lsblk
```

Example:

```text
NAME          SIZE TYPE MOUNTPOINTS
nvme1n1        10G disk
└─nvme1n1p1    5G part /data/ebs
```

In this example:

- `/dev/nvme1n1` is the disk.
- `/dev/nvme1n1p1` is a partition.
- The disk has been expanded to 10 GiB.
- The partition still occupies only 5 GiB.

The partition must be expanded before the filesystem can use the remaining space.

## 19. Install `growpart` If Necessary

The `growpart` utility can expand a partition to use additional space on its disk.

For Amazon Linux 2023:

```bash
sudo dnf install -y cloud-utils-growpart
```

For Amazon Linux 2:

```bash
sudo yum install -y cloud-utils-growpart
```

The utility is commonly provided by the `cloud-utils-growpart` package.

Check whether it is available:

```bash
which growpart
```

## 20. Expand the Partition

For this example:

```text
Disk:      /dev/nvme1n1
Partition: /dev/nvme1n1p1
```

Run:

```bash
sudo growpart /dev/nvme1n1 1
```

### Understand the Command

```bash
sudo growpart DISK PARTITION_NUMBER
```

For example:

```bash
sudo growpart /dev/nvme1n1 1
```

- `/dev/nvme1n1` is the disk.
- `1` is the partition number.
- The command attempts to extend partition 1 into available free space.

**Do not use this example blindly.** Confirm the disk, partition number, and layout with `lsblk` first.

Verify:

```bash
lsblk
```

The partition should now be larger if the operation succeeded.

---

## 21. Expand an ext4 Filesystem on the Partition

Suppose the filesystem is ext4 on `/dev/nvme1n1p1`.

After expanding the partition, run:

```bash
sudo resize2fs /dev/nvme1n1p1
```

Verify:

```bash
df -hT /data/ebs
```

```bash
lsblk -f
```

The filesystem should now recognize the additional capacity.

### Complete ext4 Partitioned-Disk Workflow

```bash
# Inspect the disk and filesystem
lsblk -f
df -hT /data/ebs

# Expand partition 1
sudo growpart /dev/nvme1n1 1

# Expand the ext4 filesystem
sudo resize2fs /dev/nvme1n1p1

# Verify the result
lsblk
df -hT /data/ebs
```

This sequence assumes that `/dev/nvme1n1p1` is the correct partition, that it contains ext4, and that the EBS disk has already been enlarged.

---

## 22. Expand an XFS Filesystem on the Partition

Suppose the filesystem is XFS on `/dev/nvme1n1p1`, mounted at `/data/ebs`.

After increasing the EBS volume size, run:

```bash
sudo growpart /dev/nvme1n1 1
```

Then grow the mounted XFS filesystem:

```bash
sudo xfs_growfs /data/ebs
```

Verify:

```bash
df -hT /data/ebs
```

```bash
lsblk -f
```

### Complete XFS Partitioned-Disk Workflow

```bash
# Inspect the disk and filesystem
lsblk -f
df -hT /data/ebs

# Expand partition 1
sudo growpart /dev/nvme1n1 1

# Expand the mounted XFS filesystem
sudo xfs_growfs /data/ebs

# Verify the result
lsblk
df -hT /data/ebs
```

This example assumes that the filesystem is XFS, the partition number is 1, and the filesystem is mounted at `/data/ebs`.

---

# Part 7: Quick Decision Guide

Use the following guide to choose the correct resizing commands.

```text
Increase EBS Volume Size in AWS
              |
              v
       Check with lsblk
              |
              v
   Is the filesystem on a partition?
          /             \
        Yes              No
         |                |
         v                v
  Expand partition    Use whole disk
  with growpart       as filesystem device
         |                |
         v                |
  Check filesystem type  |
         |                |
      +--+--+             |
      |     |             |
    ext4   XFS             |
      |     |              |
      v     v              v
 resize2fs xfs_growfs   Check filesystem
 on partition on mount point type first
```

### Command Summary

| Layout | Filesystem | Commands after EBS expansion |
|---|---|---|
| Whole disk, no partition | ext4 | `sudo resize2fs /dev/nvme1n1` |
| Whole disk, no partition | XFS | `sudo xfs_growfs /data/ebs` |
| Partitioned disk | ext4 | `growpart`, then `resize2fs` on the partition |
| Partitioned disk | XFS | `growpart`, then `xfs_growfs` on the mount point |

The device names and mount point must match your system.

---

# Part 8: Verify the Final Capacity

After resizing, run:

```bash
lsblk
```

```bash
lsblk -f
```

```bash
df -hT /data/ebs
```

```bash
findmnt /data/ebs
```

### What Does Each Command Verify?

| Command | Purpose |
|---|---|
| `lsblk` | Shows disk and partition sizes |
| `lsblk -f` | Shows filesystems, UUIDs, and mount points |
| `df -hT` | Shows mounted filesystem capacity and type |
| `findmnt` | Shows the active mount source and options |

A successful resize should be reflected in the filesystem capacity shown by `df -hT`.

---

# Part 9: Troubleshooting

## Problem 1: AWS Shows 10 GiB, but Linux Shows 5 GiB

Check:

```bash
lsblk
```

Verify that the EBS modification has been applied in AWS.

If the Linux device still shows the old capacity, investigate the device and volume modification status before attempting to grow a partition or filesystem.

## Problem 2: The Disk Is 10 GiB, but the Partition Is 5 GiB

Example:

```text
nvme1n1        10G disk
└─nvme1n1p1    5G part /data/ebs
```

If the layout supports expansion and the free space is contiguous after the partition, expand it with the appropriate `growpart` command.

For partition 1:

```bash
sudo growpart /dev/nvme1n1 1
```

Then resize the filesystem using the correct tool.

## Problem 3: `resize2fs` Says the Device Does Not Contain a Recognized ext Filesystem

Check:

```bash
lsblk -f
```

```bash
df -hT /data/ebs
```

Possible reasons:

- The filesystem is XFS, not ext4.
- The command targets the wrong device.
- The filesystem is on a partition rather than the whole disk.

For XFS, use:

```bash
sudo xfs_growfs /data/ebs
```

For ext4 directly on the whole disk:

```bash
sudo resize2fs /dev/nvme1n1
```

For ext4 on a partition:

```bash
sudo resize2fs /dev/nvme1n1p1
```

Use only the command that matches your actual filesystem layout.

## Problem 4: `xfs_growfs` Fails

Check the mount point:

```bash
findmnt /data/ebs
```

Check the filesystem type:

```bash
df -hT /data/ebs
```

`xfs_growfs` requires an XFS filesystem mounted at the specified mount point.

## Problem 5: The Mount Disappears After Reboot

Check:

```bash
cat /etc/fstab
```

Check the UUID:

```bash
sudo blkid
```

Validate the configuration:

```bash
sudo findmnt --verify --verbose
```

Test eligible entries:

```bash
sudo mount -a
```

Use the actual filesystem UUID and correct filesystem type in `/etc/fstab`.

---

# Part 10: Important Production Considerations

1. **Back up critical data.** Take and verify suitable EBS snapshots before risky storage operations.
2. **Confirm the correct disk.** A wrong device name can result in data loss.
3. **Do not format an existing filesystem.** `mkfs` is for creating a filesystem, not for resizing an existing one.
4. **Increase the EBS volume first.** Then ensure Linux sees the new capacity.
5. **Expand the partition if necessary.** A partitioned volume may need `growpart`.
6. **Use the correct filesystem tool.** `resize2fs` is for ext filesystems; `xfs_growfs` is for XFS.
7. **Do not assume shrinking is supported.** Increasing a volume is different from reducing its size.
8. **Test `/etc/fstab` before rebooting.** A bad mount configuration can create boot or application problems.
9. **Use UUIDs.** They are generally more reliable than device names in persistent mount configuration.
10. **Check application dependencies.** A mounted filesystem does not automatically guarantee that an application is configured to use it correctly.

---

# Part 11: Interview Questions and Answers

## 1. What is permanent mounting?

Permanent mounting means configuring a filesystem to mount automatically, typically at boot, by defining it in `/etc/fstab`.

## 2. What does `mount -a` do?

It attempts to mount eligible filesystems listed in `/etc/fstab` that are not already mounted.

## 3. Why do we use UUID in `/etc/fstab`?

A UUID provides a stable way to identify a filesystem when device names can change.

## 4. Does increasing an EBS volume automatically resize the filesystem?

Not necessarily. Linux may need to recognize the new device capacity, expand a partition if one exists, and grow the filesystem.

## 5. What is the purpose of `resize2fs`?

`resize2fs` grows or resizes ext2, ext3, and ext4 filesystems, depending on the filesystem and supported operation.

## 6. What is the purpose of `xfs_growfs`?

`xfs_growfs` expands a mounted XFS filesystem to use available space on its underlying device.

## 7. What is the purpose of `growpart`?

`growpart` expands a partition to use additional available space on its disk when the partition layout permits it.

## 8. What is the difference between a disk, a partition, and a filesystem?

- **Disk:** The block storage device provided by EBS.
- **Partition:** A logical region of a disk.
- **Filesystem:** The structure Linux uses to store and organize files.

A filesystem can be created directly on a whole disk or on a partition.

## 9. Can I use `resize2fs` for XFS?

No. Use `xfs_growfs` for XFS.

## 10. How do you verify that an EBS filesystem has been resized successfully?

Use:

```bash
lsblk
```

```bash
df -hT /data/ebs
```

`lsblk` shows disk and partition sizes, while `df -hT` shows the capacity recognized by the mounted filesystem.

---

# Lab Completion Checklist

- [ ] Identified the EBS disk and filesystem.
- [ ] Retrieved the filesystem UUID.
- [ ] Backed up `/etc/fstab`.
- [ ] Added a permanent mount entry.
- [ ] Validated the entry with `findmnt --verify --verbose`.
- [ ] Tested the mount configuration with `mount -a`.
- [ ] Verified the mount using `findmnt`.
- [ ] Recorded the original EBS volume capacity.
- [ ] Increased the EBS volume size in AWS.
- [ ] Verified the new disk capacity using `lsblk`.
- [ ] Expanded the partition if required.
- [ ] Identified whether the filesystem is ext4 or XFS.
- [ ] Used `resize2fs` for ext4 or `xfs_growfs` for XFS.
- [ ] Verified the final filesystem capacity using `df -hT`.

---

# Final Summary

```text
PERMANENT MOUNTING
------------------
Find UUID:
sudo blkid

Edit configuration:
sudo vi /etc/fstab

Validate:
sudo findmnt --verify --verbose

Mount eligible entries:
sudo mount -a

Verify:
findmnt /data/ebs
df -hT /data/ebs


EBS RESIZING
------------
1. Increase the EBS volume size in AWS.
2. Verify the new device capacity:
   lsblk

3. If the filesystem is on a partition, expand it:
   sudo growpart /dev/nvme1n1 1

4. Resize the filesystem using the correct tool:

   ext4 on whole disk:
   sudo resize2fs /dev/nvme1n1

   ext4 on partition:
   sudo resize2fs /dev/nvme1n1p1

   XFS mounted at /data/ebs:
   sudo xfs_growfs /data/ebs

5. Verify:
   lsblk
   df -hT /data/ebs
```

**Remember these three commands:**

- `mount -a` — attempts to mount eligible filesystems configured in `/etc/fstab`.
- `resize2fs` — grows ext2/ext3/ext4 filesystems.
- `xfs_growfs` — grows a mounted XFS filesystem.