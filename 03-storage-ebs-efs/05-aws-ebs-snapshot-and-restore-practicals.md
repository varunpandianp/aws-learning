# AWS EBS Snapshot and Restore Practicals

## 1. Objective

Learn how to:

1. Create an EBS volume and attach it to an EC2 instance.
2. Create a filesystem and store sample data on the volume.
3. Take a snapshot of the EBS volume.
4. Create a new EBS volume from the snapshot.
5. Attach the new volume to an EC2 instance.
6. Mount the new volume and verify the restored data.
7. Understand snapshot use cases, limitations, and troubleshooting.

---

## 2. What Is an EBS Snapshot?

An **Amazon EBS snapshot** is a point-in-time backup of an EBS volume.

A snapshot allows us to preserve the data on an EBS volume and create another EBS volume from that backup.

### Example architecture

```text
                    EC2 Instance
                         |
                         |
                    EBS Volume A
                         |
                    Sample Data
                         |
                         v
                    EBS Snapshot
                         |
                         |
               Create Volume from Snapshot
                         |
                         v
                    EBS Volume B
                         |
                         |
                 Attach to EC2 Instance
                         |
                         v
                 Mount and Verify Data
```

### Real-world use cases

- Back up important application data.
- Restore data after accidental deletion or corruption.
- Create a duplicate volume for testing.
- Migrate data to another EC2 instance.
- Create a new volume from a known-good backup.
- Copy snapshots to another AWS Region for disaster recovery.

**Important:** A snapshot is not the same as a live, continuously synchronized copy. It represents the volume's data at a particular point in time.

---

## 3. Important Concepts

| Concept | Explanation |
|---|---|
| EBS Volume | Persistent block storage that can be attached to an EC2 instance. |
| EBS Snapshot | Point-in-time backup of an EBS volume. |
| Source Volume | Original volume from which the snapshot is created. |
| Restored Volume | New volume created from the snapshot. |
| Availability Zone (AZ) | The location within an AWS Region where an EBS volume exists. |
| Incremental Snapshot | A snapshot that stores changes relative to earlier snapshots, avoiding the need to store every block again. |

### Important rules

1. An EBS volume must be created in the same Availability Zone as the EC2 instance to which it will be attached.
2. A snapshot is stored in AWS-managed snapshot infrastructure, not as a normal file inside your EC2 instance.
3. You can create multiple EBS volumes from the same snapshot.
4. The new volume must be in the same AZ as the EC2 instance you want to attach it to.
5. Creating a volume from a snapshot does not delete or modify the original volume.
6. Snapshot creation is asynchronous. Wait until the snapshot status becomes `Completed` before treating the backup as ready for normal use.
7. The new volume's size must be at least as large as the source volume's size.
8. A snapshot does not automatically guarantee application-consistent backups. Databases and other applications may need application-level preparation.

---

# PRACTICAL 1: Create an EBS Volume and Store Sample Data

## Step 1: Create an EBS Volume

1. Sign in to the AWS Management Console.
2. Open the EC2 service.
3. Navigate to **Elastic Block Store → Volumes**.
4. Select **Create volume**.
5. Configure the volume:

| Setting | Example |
|---|---|
| Volume type | gp3 |
| Size | 5 GiB |
| Availability Zone | Same AZ as your EC2 instance |
| Encryption | Enable encryption if appropriate for your environment |
| Name tag | `ebs-source-volume` |

6. Select **Create volume**.

Wait until the volume status becomes `Available`.

## Step 2: Attach the Volume

1. Select the new EBS volume.
2. Choose **Actions → Attach volume**.
3. Select your EC2 instance.
4. Specify a device name, such as `/dev/sdf`, if requested.
5. Select **Attach volume**.

**Note:** The requested device name may appear differently inside a Nitro-based Linux EC2 instance. For example, `/dev/sdf` might appear as `/dev/nvme1n1`.

## Step 3: Identify the Attached Disk

Connect to your EC2 instance and run:

```bash
lsblk
```

Example output:

```text
NAME        MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1     259:0    0  10G  0 disk
├─nvme0n1p1 259:1    0  10G  0 part /
nvme1n1     259:2    0   5G  0 disk
```

Here, `/dev/nvme1n1` is an example of the newly attached disk. **Your actual device name may differ.**

Check the filesystem and mount information:

```bash
lsblk -f
```

## Step 4: Create a Filesystem

**Warning:** Formatting a disk can destroy existing data. Run this command only after confirming that the selected disk is the new, empty EBS volume.

For an XFS filesystem:

```bash
sudo mkfs.xfs /dev/nvme1n1
```

If the disk already contains a filesystem or data, do not format it.

## Step 5: Create a Mount Point

```bash
sudo mkdir -p /data/ebs-source
```

Mount the volume:

```bash
sudo mount /dev/nvme1n1 /data/ebs-source
```

Verify the mount:

```bash
df -hT
```

```bash
lsblk -f
```

## Step 6: Create Sample Data

```bash
echo "This is important application data." | sudo tee /data/ebs-source/app-data.txt
```

Create another file:

```bash
echo "EBS snapshot practical" | sudo tee /data/ebs-source/notes.txt
```

Verify the files:

```bash
ls -l /data/ebs-source
```

Read the sample data:

```bash
cat /data/ebs-source/app-data.txt
```

Expected output:

```text
This is important application data.
```

**Checkpoint:** Your original EBS volume now contains sample data that you can back up using a snapshot.

---

# PRACTICAL 2: Create an EBS Snapshot

## Step 1: Open the Volumes Page

In the AWS Console:

1. Open EC2.
2. Select **Elastic Block Store → Volumes**.
3. Select your source volume, `ebs-source-volume`.

## Step 2: Create the Snapshot

1. Choose **Actions → Create snapshot**.
2. Enter a description:

```text
Backup of EBS source volume for snapshot restore practical
```

3. Add a tag if needed:

| Key | Value |
|---|---|
| Name | `ebs-source-snapshot` |
| Environment | `Practice` |

4. Select **Create snapshot**.

## Step 3: Verify Snapshot Status

Navigate to:

**EC2 → Elastic Block Store → Snapshots**

Locate the snapshot you just created.

Check its status.

Initially, it may show:

```text
Pending
```

Wait until it shows:

```text
Completed
```

The time required depends on the amount of data that needs to be backed up and other factors.

## Step 4: Understand What Was Backed Up

Your snapshot captures the volume's data at the time the snapshot was initiated, subject to AWS's snapshot consistency behavior.

It does not mean that future changes to the original volume automatically appear in the snapshot.

For example:

```text
Before Snapshot:
app-data.txt = Important data

Snapshot Created

After Snapshot:
app-data.txt = Updated data
```

The existing snapshot represents the earlier point in time. Changes made afterward are not automatically included.

### Important consistency consideration

For a basic lab, creating a snapshot of a volume containing sample text files is sufficient.

For production databases and applications, prepare the workload appropriately before creating a snapshot. Depending on the application, this may involve flushing writes, quiescing I/O, using database-native backups, or coordinating snapshots across multiple volumes.

---

# PRACTICAL 3: Create a New EBS Volume from the Snapshot

Now you will restore the backup into a separate EBS volume.

## Step 1: Open the Snapshots Page

In the AWS Console:

1. Open EC2.
2. Select **Elastic Block Store → Snapshots**.
3. Select `ebs-source-snapshot`.

## Step 2: Create a Volume from the Snapshot

1. Choose **Actions → Create volume from snapshot**.
2. Configure the new volume:

| Setting | Example |
|---|---|
| Snapshot ID | Your source snapshot |
| Volume type | gp3 |
| Size | 5 GiB or larger |
| Availability Zone | Same AZ as the target EC2 instance |
| Name tag | `ebs-restored-volume` |

3. Configure encryption according to your requirements.
4. Select **Create volume**.

Wait until the new volume status becomes:

```text
Available
```

### What happened?

AWS created a new EBS volume whose initial data comes from the snapshot.

```text
Original Volume
      |
      v
   Snapshot
      |
      v
New EBS Volume
```

The original volume and the restored volume are independent after creation. Changes to one do not automatically update the other.

---

# PRACTICAL 4: Attach the Restored Volume

## Step 1: Select the Restored Volume

Navigate to:

**EC2 → Elastic Block Store → Volumes**

Select:

```text
ebs-restored-volume
```

## Step 2: Attach It to an EC2 Instance

1. Choose **Actions → Attach volume**.
2. Select your target EC2 instance.
3. Specify a device name if required, such as `/dev/sdg`.
4. Select **Attach volume**.

The restored volume must be in the same Availability Zone as the target instance.

You can attach both the source and restored volumes to the same EC2 instance if the instance and attachment limits permit it.

## Step 3: Identify the New Disk

Connect to the EC2 instance:

```bash
lsblk
```

Example:

```text
NAME        SIZE TYPE MOUNTPOINTS
nvme0n1      10G disk
└─nvme0n1p1  10G part /
nvme1n1       5G disk
nvme2n1       5G disk
```

In this example:

- `/dev/nvme1n1` could be the original EBS volume.
- `/dev/nvme2n1` could be the restored EBS volume.

**Do not assume the device names are fixed.** Identify the correct volume before performing any operation.

Useful commands:

```bash
lsblk -f
```

```bash
sudo blkid
```

For Nitro-based EC2 instances, you can also inspect NVMe device information using the appropriate NVMe tools available on the instance.

---

# PRACTICAL 5: Mount the Restored Volume and Verify Data

## Step 1: Create a Separate Mount Point

```bash
sudo mkdir -p /data/ebs-restored
```

## Step 2: Mount the Restored Volume

Assuming the restored volume is `/dev/nvme2n1` and contains an XFS filesystem directly on the disk:

```bash
sudo mount /dev/nvme2n1 /data/ebs-restored
```

**Important:** This is an example only. If your original volume uses a partition, such as `/dev/nvme2n1p1`, mount the correct partition instead. Check `lsblk -f` first.

If the filesystem is already present, do not run `mkfs` on the restored volume. Formatting it would destroy the data you are trying to recover.

## Step 3: Verify the Restored Files

```bash
ls -l /data/ebs-restored
```

Read the files:

```bash
cat /data/ebs-restored/app-data.txt
```

```bash
cat /data/ebs-restored/notes.txt
```

Expected output includes:

```text
This is important application data.
```

and:

```text
EBS snapshot practical
```

## Step 4: Verify the Mounted Filesystem

```bash
df -hT
```

```bash
findmnt /data/ebs-restored
```

```bash
lsblk -f
```

**Success criteria:** The restored volume is mounted, and the sample files from the snapshot are accessible.

---

# PRACTICAL 6: Prove That the Original and Restored Volumes Are Independent

This is an important part of understanding snapshots.

## Step 1: Add a New File to the Original Volume

```bash
echo "This file was created after the snapshot." | sudo tee /data/ebs-source/after-snapshot.txt
```

## Step 2: Check the Restored Volume

```bash
ls -l /data/ebs-restored
```

The file `after-snapshot.txt` should not appear in the restored volume if it was created after the snapshot was taken.

Why?

Because the snapshot represents the original volume at the earlier point in time.

## Step 3: Modify a File in the Restored Volume

```bash
echo "This change exists only on the restored volume." | sudo tee /data/ebs-restored/restored-only.txt
```

Check the original volume:

```bash
ls -l /data/ebs-source
```

The file `restored-only.txt` should not appear on the original volume.

### Key lesson

```text
Original Volume  ──────> Snapshot ──────> Restored Volume

Changes made to the original after the snapshot
do not automatically appear on the restored volume.

Changes made to the restored volume
do not automatically appear on the original volume.
```

---

# PRACTICAL 7: Optional — Make the Restored Volume Mount Persistent

If you want the restored volume to mount automatically after a reboot, configure `/etc/fstab` carefully.

First, identify its filesystem UUID:

```bash
sudo blkid
```

Example output:

```text
/dev/nvme2n1: UUID="abcd-1234-example" TYPE="xfs"
```

Your actual UUID will be different.

Create the mount point if it does not exist:

```bash
sudo mkdir -p /data/ebs-restored
```

Back up the existing `fstab` file:

```bash
sudo cp /etc/fstab /etc/fstab.backup
```

Edit the file:

```bash
sudo vi /etc/fstab
```

For an XFS filesystem, add an entry using your actual UUID:

```text
UUID=abcd-1234-example  /data/ebs-restored  xfs  defaults,nofail  0  2
```

Do not copy the example UUID literally.

Validate the configuration:

```bash
sudo findmnt --verify --verbose
```

Test the mount configuration:

```bash
sudo mount -a
```

Verify:

```bash
findmnt /data/ebs-restored
```

### Important

- Use the filesystem UUID, not a guessed device name.
- Ensure that the filesystem type matches the actual filesystem.
- `nofail` helps the system continue booting if the volume is unavailable, but it does not guarantee that applications depending on that volume will work.
- Be careful when configuring `/etc/fstab`; incorrect entries can cause boot or mount problems.
- Do not reboot until the configuration has been validated.

---

# PRACTICAL 8: Snapshot and Restore Workflow Summary

| Step | Action | Expected Result |
|---|---|---|
| 1 | Create an EBS volume | Source volume becomes available |
| 2 | Attach the volume | Linux detects the disk |
| 3 | Create a filesystem | Volume is ready to store data |
| 4 | Mount the volume | Filesystem becomes accessible |
| 5 | Create sample files | Source volume contains test data |
| 6 | Create a snapshot | Backup appears in the Snapshots page |
| 7 | Wait for completion | Snapshot status becomes `Completed` |
| 8 | Create a volume from the snapshot | Restored volume becomes available |
| 9 | Attach the restored volume | Linux detects another disk |
| 10 | Mount the restored volume | Restored filesystem becomes accessible |
| 11 | Read the sample files | Snapshot data is successfully recovered |
| 12 | Modify each volume independently | Changes remain separate |

---

# Troubleshooting

## Problem 1: The Restored Volume Is Not Visible

Check:

```bash
lsblk
```

```bash
lsblk -f
```

Possible causes:

- The volume was not attached successfully.
- The wrong EC2 instance was selected.
- The instance and volume are in different Availability Zones.
- You are looking for the requested device name instead of the actual Linux device name.

Check the volume attachment status in the AWS Console.

## Problem 2: Mount Fails with "Wrong FS Type" or "Bad Superblock"

Check the filesystem:

```bash
sudo blkid
```

```bash
lsblk -f
```

Possible causes:

- You selected the wrong disk.
- You tried to mount the whole disk when the filesystem is on a partition.
- The filesystem type or filesystem metadata needs investigation.

Do not immediately format the disk. Formatting can destroy the data you are attempting to restore.

## Problem 3: The Restored Volume Appears Empty

Check:

```bash
lsblk -f
```

```bash
findmnt
```

Possible causes:

- You mounted the wrong device.
- You mounted the original volume instead of the restored volume.
- The snapshot was taken before the sample files were created.
- The source filesystem was on a partition, and you mounted the wrong device.
- The snapshot was created while application data was not in a consistent state.

## Problem 4: Cannot Attach the Volume to the Instance

Check:

- Whether the volume and instance are in the same Availability Zone.
- Whether the instance supports additional EBS volume attachments.
- Whether attachment limits have been reached.
- Whether the volume is already attached to another instance.

## Problem 5: Snapshot Is Still Pending

Snapshot creation can take time. Check its status in the AWS Console.

Do not assume that the snapshot is ready simply because the creation request succeeded.

---

# Important Production Considerations

## 1. Snapshot Consistency

A snapshot is a point-in-time backup, but application consistency depends on the workload and how the backup is prepared.

For databases, consider database-native backup mechanisms or application-consistent snapshot procedures.

## 2. Encryption

Encrypted EBS volumes produce encrypted snapshots. When restoring a volume, configure encryption and KMS permissions appropriately.

## 3. Cross-Region Disaster Recovery

To recover in another Region, copy the snapshot to the destination Region and create a volume there. You must select an Availability Zone in the destination Region for the new volume.

## 4. Snapshot Lifecycle

For production environments, consider automating backups and retention using AWS Backup or Amazon Data Lifecycle Manager, where appropriate.

## 5. Cost Awareness

EBS snapshots incur storage charges based on the applicable snapshot storage model and Region. EBS volumes also incur charges while provisioned, even when they are not actively being used.

For this practical, delete unneeded test resources after verifying that you no longer need them.

---

# Interview Questions and Answers

### 1. What is an EBS snapshot?

An EBS snapshot is a point-in-time backup of an EBS volume that can be used to restore data by creating a new volume.

### 2. Can we create a new EBS volume from a snapshot?

Yes. Select the snapshot in the EC2 Console and choose the option to create a volume from it.

### 3. Can we create multiple volumes from the same snapshot?

Yes. You can create multiple volumes from the same snapshot, subject to AWS service limits and permissions.

### 4. Does creating a new volume from a snapshot affect the original volume?

No. The original volume remains separate and is not modified by creating another volume from its snapshot.

### 5. Can an EBS volume be attached to an EC2 instance in another Availability Zone?

No. An EBS volume must be in the same Availability Zone as the EC2 instance to which it is attached. You can create a new volume from a snapshot in the target AZ.

### 6. What is the difference between an EBS volume and an EBS snapshot?

An EBS volume provides block storage that can be attached to an EC2 instance. An EBS snapshot is a point-in-time backup used to preserve and restore volume data.

### 7. Are EBS snapshots full or incremental?

EBS snapshots use incremental storage: after the first snapshot, subsequent snapshots store changed blocks rather than storing every block again. AWS manages the underlying snapshot data.

### 8. If I delete the original volume, will the snapshot also be deleted?

No. Deleting an EBS volume does not automatically delete its snapshots. Manage snapshots separately, keeping in mind any backup and retention requirements.

### 9. Can we restore a snapshot to a larger EBS volume?

Yes. You can create a volume larger than the source volume, subject to service limits. You may need to expand the partition and filesystem to use the additional capacity.

### 10. Does creating a volume from a snapshot automatically attach it to an EC2 instance?

No. Creating the volume and attaching it are separate steps.

### 11. Can we use an EBS snapshot for disaster recovery in another Region?

Yes. Copy the snapshot to the destination Region, then create a new EBS volume in an Availability Zone in that Region.

### 12. Why should we avoid formatting a restored volume?

The restored volume already contains the filesystem and data captured in the snapshot. Formatting it can erase the restored data.

### 13. Is a snapshot always application-consistent?

No. Filesystem or crash consistency does not necessarily guarantee application consistency. Databases and other applications may require additional backup preparation.

### 14. What happens when we modify data after creating a snapshot?

Those changes are not automatically added to the existing snapshot. A later snapshot is needed to capture subsequent changes.

### 15. How would you restore a deleted file using an EBS snapshot?

Create a volume from a suitable snapshot, attach it to an instance, mount the filesystem safely, locate the file, and copy the required data to the appropriate destination.

---

# Practice Tasks

Complete these tasks yourself:

- [ ] Create a 5 GiB gp3 EBS volume.
- [ ] Attach it to a Linux EC2 instance.
- [ ] Create an XFS filesystem.
- [ ] Mount it under `/data/ebs-source`.
- [ ] Create two sample files.
- [ ] Create a snapshot of the source volume.
- [ ] Wait until the snapshot status is `Completed`.
- [ ] Create a new volume from that snapshot.
- [ ] Attach the restored volume to an EC2 instance in the same AZ.
- [ ] Identify the correct Linux device using `lsblk -f`.
- [ ] Mount it under `/data/ebs-restored`.
- [ ] Verify the original sample files.
- [ ] Create a file on the original volume after the snapshot and confirm it is absent from the restored volume.
- [ ] Create a different file on the restored volume and confirm it is absent from the original volume.
- [ ] Configure persistent mounting only if needed.
- [ ] Clean up test resources when finished.

---

# GitHub Learning Log

## Topic: AWS EBS Snapshot and Restore

**What I learned:**

- How to create an EBS snapshot from a source volume.
- How to create a new EBS volume from a snapshot.
- How to attach and mount the restored volume.
- How to verify restored files using Linux commands.
- How to prove that the original and restored volumes are independent.
- Why Availability Zones matter when attaching EBS volumes.
- How snapshots support backups, recovery, and disaster recovery.

**Commands practiced:**

```bash
lsblk
lsblk -f
sudo blkid
sudo mkdir -p /data/ebs-source
sudo mkdir -p /data/ebs-restored
sudo mount /dev/nvme1n1 /data/ebs-source
sudo mount /dev/nvme2n1 /data/ebs-restored
df -hT
findmnt
```

**Note:** Replace the example device names with the actual device names identified on your EC2 instance.

