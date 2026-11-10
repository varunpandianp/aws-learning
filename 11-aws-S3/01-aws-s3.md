# AWS S3 – Complete DevOps Study Notes

## 1. What is AWS S3?

**Amazon S3 (Simple Storage Service)** is an object storage service provided by AWS. It allows us to store and retrieve any amount of data from anywhere over the internet or through AWS services.

S3 is highly scalable, durable, and designed to store files without managing physical storage servers.

### Why do we use S3?

- Store application files, images, videos and documents.
- Store database backups.
- Store application and server logs.
- Store Terraform state files.
- Store CI/CD build artifacts.
- Host static websites.
- Build data lakes for analytics.
- Archive old data.

### Important S3 concepts

| Concept | Meaning |
|---|---|
| Bucket | Container used to store objects |
| Object | Actual file stored in S3 |
| Key | Unique name/path of an object |
| Metadata | Information about an object |
| Region | AWS Region where the bucket is created |
| Prefix | Logical folder-like grouping of objects |
| Version ID | Unique identifier for an object version |

Example:

```text
my-s3-bucket/
│
├── application/
│   ├── app.jar
│   └── config.json
│
├── logs/
│   ├── application.log
│   └── error.log
│
└── backup/
    ├── database.sql
    └── backup.zip
```

Note: S3 uses object keys and prefixes. These folders are logical groupings, not actual Linux directories.

---

## 2. How to access S3 from a Linux server?

We can access an S3 bucket from a Linux EC2 instance using the **AWS CLI**.

### Architecture

```text
Linux EC2 Instance
       |
       |
     AWS CLI
       |
       |
   IAM Role
       |
       |
   AWS S3 API
       |
       |
    S3 Bucket
```

### Step 1: Create an S3 bucket

1. Open AWS Console.
2. Search for S3.
3. Click Create bucket.
4. Enter a globally unique bucket name.
5. Select AWS Region.
6. Configure required settings.
7. Click Create bucket.

Example bucket:

```text
my-devops-learning-bucket
```

### Step 2: Create an IAM policy

Go to:

```text
IAM → Policies → Create policy → JSON
```

Example policy to list, upload, download and delete objects in one bucket:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::my-devops-learning-bucket"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::my-devops-learning-bucket/*"
    }
  ]
}
```

Save policy name:

```text
EC2-S3-Access-Policy
```

### Step 3: Create an IAM Role

1. Go to IAM → Roles.
2. Click Create role.
3. Trusted entity: AWS service.
4. Use case: EC2.
5. Attach the `EC2-S3-Access-Policy`.
6. Give the role a name:

```text
EC2-S3-Access-Role
```

7. Create the role.

### Step 4: Attach IAM Role to EC2

Go to:

```text
EC2 → Instances
→ Select your EC2 instance
→ Actions
→ Security
→ Modify IAM role
```

Select:

```text
EC2-S3-Access-Role
```

Click Update IAM role.

### Step 5: Connect to Linux EC2

```bash
ssh -i mykey.pem ec2-user@<EC2-PUBLIC-IP>
```

### Step 6: Install AWS CLI

Amazon Linux 2023:

```bash
sudo dnf install awscli -y
```

Ubuntu:

```bash
sudo apt update
sudo apt install awscli -y
```

Check installation:

```bash
aws --version
```

### Step 7: Verify AWS access

```bash
aws sts get-caller-identity
```

This command displays the AWS account and IAM identity associated with the credentials.

List S3 buckets:

```bash
aws s3 ls
```

List a specific bucket:

```bash
aws s3 ls s3://my-devops-learning-bucket
```

**Important:** When using an attached EC2 IAM role, we don't need to store access keys using `aws configure`. The AWS CLI obtains temporary credentials through EC2 instance metadata.

---

## 3. AWS S3 CLI Commands – Linux Practicals

### 3.1 List buckets

```bash
aws s3 ls
```

### 3.2 List files inside a bucket

```bash
aws s3 ls s3://my-devops-learning-bucket
```

### 3.3 Create a bucket using CLI

```bash
aws s3 mb s3://my-devops-learning-bucket
```

### 3.4 Create a local file

```bash
echo "Welcome to AWS S3" > test.txt
```

### 3.5 Upload a single file – `cp`

```bash
aws s3 cp test.txt s3://my-devops-learning-bucket/
```

Explanation:

- `cp` means copy.
- First argument is the source.
- Second argument is the destination.

### 3.6 Download a file from S3

```bash
aws s3 cp s3://my-devops-learning-bucket/test.txt .
```

The dot (`.`) represents the current Linux directory.

Download to a specific location:

```bash
aws s3 cp s3://my-devops-learning-bucket/test.txt /home/ec2-user/
```

### 3.7 Upload a file with a different name

```bash
aws s3 cp test.txt s3://my-devops-learning-bucket/myfile.txt
```

### 3.8 Bulk upload – Upload multiple files

Bulk upload means uploading multiple files from a local directory to S3.

Create a directory:

```bash
mkdir application
cd application
```

Create files:

```bash
touch app1.txt app2.txt app3.txt
```

Upload all files:

```bash
aws s3 cp . s3://my-devops-learning-bucket/application/ --recursive
```

`--recursive` copies all files and subdirectories.

### 3.9 Bulk download

Download all objects from S3:

```bash
aws s3 cp s3://my-devops-learning-bucket/ ./backup/ --recursive
```

### 3.10 AWS S3 Sync

`sync` synchronizes files between a local directory and an S3 bucket.

Upload local directory to S3:

```bash
aws s3 sync ./application/ s3://my-devops-learning-bucket/application/
```

Download S3 directory to Linux:

```bash
aws s3 sync s3://my-devops-learning-bucket/application/ ./application/
```

Sync one S3 bucket to another:

```bash
aws s3 sync s3://source-bucket/ s3://destination-bucket/
```

Important points:

- Copies new files.
- Copies updated files.
- Skips files that are already synchronized.
- Does not delete destination files by default.
- `--delete` removes destination files that don't exist in the source.

Example:

```bash
aws s3 sync ./application/ s3://my-devops-learning-bucket/application/ --delete
```

**Be careful:** `--delete` can remove objects from the destination. Always check the source and destination before using it.

### 3.11 Remove a file – `aws s3 rm`

Delete a single object:

```bash
aws s3 rm s3://my-devops-learning-bucket/test.txt
```

Delete all objects under a prefix:

```bash
aws s3 rm s3://my-devops-learning-bucket/logs/ --recursive
```

Delete objects from a bucket:

```bash
aws s3 rm s3://my-devops-learning-bucket/ --recursive
```

Note: This removes objects, not the bucket itself. Versioned buckets may retain non-current versions and delete markers.

### 3.12 Move files – `mv`

Move local file to S3:

```bash
aws s3 mv test.txt s3://my-devops-learning-bucket/
```

Move S3 object to another S3 prefix:

```bash
aws s3 mv s3://my-devops-learning-bucket/test.txt s3://my-devops-learning-bucket/backup/
```

### 3.13 Delete an empty bucket

```bash
aws s3 rb s3://my-devops-learning-bucket
```

Delete a bucket and its objects:

```bash
aws s3 rb s3://my-devops-learning-bucket --force
```

Warning: This is destructive. Versioned objects may require additional cleanup.

### 3.14 Generate a pre-signed URL

```bash
aws s3 presign s3://my-devops-learning-bucket/test.txt --expires-in 3600
```

This generates a temporary URL valid for one hour.

Useful when we need to share a private S3 object temporarily without making the bucket public.

---

## 4. AWS S3 Mounting in Linux

### What is mounting?

Mounting means attaching a storage resource to a Linux directory so that we can access it through a filesystem path.

Example:

```text
S3 Bucket
    |
    |
Third-party mounting tool
    |
    |
Linux Mount Point
    |
    |
/mnt/s3
```

Unlike EBS or EFS, S3 is object storage and cannot be mounted as a native Linux block filesystem directly.

We need additional tools to access it through a filesystem-like interface.

### Common tools

1. Mountpoint for Amazon S3 – AWS-supported tool.
2. s3fs-fuse – FUSE-based third-party tool.

### Example: Mount S3 using s3fs-fuse

Step 1: Install s3fs.

Amazon Linux:

```bash
sudo dnf install s3fs-fuse -y
```

Ubuntu:

```bash
sudo apt update
sudo apt install s3fs -y
```

Step 2: Create a mount directory.

```bash
sudo mkdir -p /mnt/s3
```

Step 3: Configure access.

Use an appropriate IAM role with the required S3 permissions. Configure s3fs to use the supported IAM role credential mechanism.

Step 4: Mount the bucket.

```bash
sudo s3fs my-devops-learning-bucket /mnt/s3 -o iam_role=auto -o url=https://s3.us-west-2.amazonaws.com
```

Step 5: Verify mounting.

```bash
df -h
```

```bash
ls -l /mnt/s3
```

Step 6: Create a file.

```bash
echo "Hello S3 Mount" | sudo tee /mnt/s3/test.txt
```

Verify the object in the S3 bucket.

Step 7: Unmount.

```bash
sudo umount /mnt/s3
```

**Important limitations:**

- S3 is not a POSIX filesystem.
- File locking and filesystem semantics are limited.
- Some applications may not work correctly with an S3 mount.
- For shared application storage requiring normal filesystem behavior, prefer EFS.
- For large-scale S3 access through a filesystem interface, evaluate AWS Mountpoint for S3.

---

## 5. AWS S3 Storage Classes

### What is a storage class?

A storage class defines how S3 stores objects based on access frequency, availability, performance, retrieval time and cost.

AWS provides different storage classes for different workloads.

### 5.1 S3 Standard

- Default S3 storage class.
- Frequently accessed data.
- Low-latency access.
- Data stored redundantly across multiple Availability Zones.

Use cases:
- Application files.
- Website images.
- Videos.
- Frequently accessed documents.

### 5.2 S3 Intelligent-Tiering

- Automatically moves eligible objects between access tiers.
- Useful when access patterns are unknown or changing.
- Has a monitoring and automation fee.
- No retrieval fees for its access tiers.

Access tiers:

1. Frequent Access.
2. Infrequent Access.
3. Archive Instant Access.
4. Archive Access (optional).
5. Deep Archive Access (optional).

Note: Archive Access and Deep Archive Access are opt-in tiers. They require restoration before normal access.

Use cases:
- Application data with unpredictable access.
- Data lakes.
- Long-term data with changing access patterns.

### 5.3 S3 Standard-IA

IA means Infrequent Access.

- Data accessed occasionally.
- Millisecond retrieval.
- Stored across multiple Availability Zones.
- Retrieval charges apply.
- Minimum storage duration: 30 days.

Use cases:
- Backups.
- Disaster recovery files.
- Old application data.

### 5.4 S3 One Zone-IA

- Data stored in a single Availability Zone.
- Cheaper than Standard-IA.
- Millisecond retrieval.
- Minimum storage duration: 30 days.
- Not suitable for critical data requiring multi-AZ resilience.

Use cases:
- Secondary backups.
- Re-creatable data.
- Non-critical files.

### 5.5 S3 Glacier Instant Retrieval

- Archive storage.
- Millisecond retrieval.
- Minimum storage duration: 90 days.
- Retrieval charges apply.

Use cases:
- Medical images.
- Historical records.
- Archive data that needs occasional immediate access.

### 5.6 S3 Glacier Flexible Retrieval

- Long-term archive storage.
- Retrieval options: Expedited, Standard and Bulk.
- Retrieval can take minutes to hours.
- Minimum storage duration: 90 days.

Use cases:
- Old database backups.
- Audit records.
- Historical application logs.

### 5.7 S3 Glacier Deep Archive

- Lowest-cost S3 archive storage class.
- Retrieval generally takes hours.
- Standard retrieval is typically within 12 hours.
- Bulk retrieval is typically within 48 hours.
- Minimum storage duration: 180 days.

Use cases:
- Long-term compliance records.
- Legal documents.
- Old backups.
- Data that is rarely accessed.

### 5.8 S3 Express One Zone

- High-performance storage class.
- Single Availability Zone.
- Single-digit millisecond data access.
- Designed for latency-sensitive workloads.
- Uses S3 directory buckets.

Use cases:
- High-performance analytics.
- AI/ML workloads.
- Applications needing very low latency.

### Storage class comparison

| Storage class | Access pattern | Retrieval | Minimum duration |
|---|---|---|---|
| Standard | Frequent | Milliseconds | None |
| Intelligent-Tiering | Changing/unknown | Depends on tier | None |
| Standard-IA | Infrequent | Milliseconds | 30 days |
| One Zone-IA | Infrequent | Milliseconds | 30 days |
| Glacier Instant | Archive | Milliseconds | 90 days |
| Glacier Flexible | Archive | Minutes to hours | 90 days |
| Glacier Deep Archive | Rarely accessed | Hours | 180 days |
| Express One Zone | High performance | Single-digit milliseconds | None |

### When should we choose which storage class?

| Scenario | Recommended class |
|---|---|
| Frequently accessed application files | Standard |
| Unknown access patterns | Intelligent-Tiering |
| Monthly accessed backups | Standard-IA |
| Re-creatable secondary backups | One Zone-IA |
| Quarterly accessed archive files | Glacier Instant Retrieval |
| Yearly accessed backups | Glacier Flexible Retrieval |
| Legal records retained for years | Glacier Deep Archive |
| Very low-latency workloads | Express One Zone |

Important: Storage cost is not the only cost. Consider retrieval fees, request charges, minimum storage duration and minimum billable object size.

---

## 6. Important S3 Bucket Properties

### 6.1 Bucket Versioning

Versioning allows us to maintain multiple versions of the same object.

Example:

```text
index.html
   |
   | Upload version 1
   |
   | Upload version 2
   |
   | Upload version 3
```

S3 maintains the previous versions instead of simply replacing the old content.

Benefits:
- Recover accidentally overwritten files.
- Recover deleted files.
- Maintain object history.
- Support replication requirements.

Versioning states:

1. Unversioned – Versioning is disabled.
2. Enabled – New versions are maintained.
3. Suspended – New objects receive the null version ID; existing versions are retained.

Practical:

```bash
aws s3api put-bucket-versioning \
--bucket my-devops-learning-bucket \
--versioning-configuration Status=Enabled
```

List versions:

```bash
aws s3api list-object-versions \
--bucket my-devops-learning-bucket
```

Important: Suspending versioning does not delete existing versions.

### 6.2 Server Access Logging

Server access logging records detailed information about requests made to an S3 bucket.

Information includes:
- Request time.
- Requester.
- Bucket name.
- Requested object.
- HTTP status code.
- Error code.
- Request type.

Example:

```text
User requests index.html
       |
       |
     S3 Bucket
       |
       |
Access request recorded
       |
       |
Destination Logging Bucket
```

Use cases:
- Security auditing.
- Monitoring access.
- Troubleshooting failed requests.
- Understanding bucket usage.

Practical setup:

1. Create a separate destination bucket.
2. Open the source bucket.
3. Go to Properties.
4. Find Server access logging.
5. Enable logging.
6. Select the destination bucket.
7. Save changes.

Important:
- Logs are delivered on a best-effort basis.
- Delivery can be delayed.
- Do not configure the destination bucket to log to itself.
- The destination bucket must meet S3 logging requirements.

### 6.3 AWS CloudTrail

CloudTrail records AWS API activity.

For S3, it can record:
- CreateBucket.
- DeleteBucket.
- PutObject.
- GetObject.
- DeleteObject.
- Other bucket and object API operations.

Example:

```text
IAM User / EC2 Role
       |
       |
   S3 API Request
       |
       |
    CloudTrail
       |
       |
  Event History / S3
```

Use cases:
- Identify who deleted an object.
- Find which IAM role uploaded a file.
- Investigate unauthorized access.
- Audit AWS activity.

Important:
- Management events and S3 data events are different.
- S3 object-level data events are not logged by default in a CloudTrail trail.
- Enable the required data-event selectors.
- Additional charges may apply.

### 6.4 Amazon EventBridge

EventBridge can receive S3 event notifications and trigger automated workflows.

Example:

```text
User uploads file to S3
        |
        |
    S3 Event
        |
        |
  Amazon EventBridge
        |
        |
   Event Rule
        |
        |
   Lambda Function
        |
        |
Process uploaded file
```

Example scenario:

When an image is uploaded to an S3 bucket, EventBridge triggers a Lambda function to resize the image.

Other use cases:
- File processing.
- Backup automation.
- Notifications.
- Triggering workflows.
- Security automation.

### 6.5 Transfer Acceleration

S3 Transfer Acceleration improves file upload and download transfer speeds over long geographical distances.

It uses AWS edge locations and the AWS global network.

Example:

```text
User in India
      |
      |
AWS Edge Location
      |
      |
AWS Global Network
      |
      |
S3 Bucket in Oregon
```

Without Transfer Acceleration:

```text
User → Internet → S3 Bucket
```

With Transfer Acceleration:

```text
User → AWS Edge Location → AWS Network → S3
```

Use cases:
- Global application users.
- Large file uploads.
- Users geographically far from the bucket Region.

Important:
- Must be enabled on the bucket.
- Uses a special accelerated endpoint.
- Additional transfer charges apply.
- Actual speed improvement depends on network conditions.

### 6.6 S3 Object Lock

Object Lock protects objects from being deleted or overwritten for a specified retention period.

It is used for WORM storage.

WORM = Write Once Read Many.

Two retention modes:

**1. Governance Mode**
- Users with specific permissions can bypass retention.
- Useful for controlled administrative access.

**2. Compliance Mode**
- Protected objects cannot be deleted or have their retention shortened during the retention period, even by the AWS account root user.
- Used for strict compliance requirements.

Example:

```text
Upload financial records
       |
       |
Enable Object Lock
       |
       |
Retention: 7 Years
       |
       |
Protected from deletion
during retention period
```

Use cases:
- Financial records.
- Legal documents.
- Regulatory compliance.
- Audit records.

### 6.7 Requester Pays

Normally, the S3 bucket owner pays for storage and request-related charges.

With Requester Pays enabled:

- Bucket owner pays for storage.
- Requester pays for applicable requests and data transfer charges.

Example:

```text
Company A owns S3 bucket
        |
        |
Company B downloads files
        |
        |
Company B pays applicable
request and transfer charges
```

Use cases:
- Public datasets.
- Research data.
- Sharing large datasets with external users.

Important: The requester must explicitly acknowledge Requester Pays, for example by using `--request-payer requester` with supported CLI operations.

### 6.8 Default Encryption

S3 encrypts objects at rest.

Encryption options:

1. SSE-S3 – Encryption using S3-managed keys.
2. SSE-KMS – Encryption using AWS KMS keys.
3. DSSE-KMS – Dual-layer server-side encryption using KMS.
4. SSE-C – Server-side encryption using customer-provided keys.

Use cases:
- Protect sensitive files.
- Compliance requirements.
- Secure backups.
- Enterprise application storage.

### 6.9 Block Public Access

Block Public Access prevents accidental public exposure of S3 data.

Four settings:

1. Block public ACLs.
2. Ignore public ACLs.
3. Block public bucket policies.
4. Restrict public bucket access.

Best practice: Keep public access blocked unless public access is specifically required.

### 6.10 Static Website Hosting

S3 can host static websites.

Supported content:
- HTML.
- CSS.
- JavaScript.
- Images.

Example:

```text
index.html
style.css
script.js
images/
```

Enable:

```text
S3 → Bucket → Properties
→ Static website hosting
```

Important:
- S3 website endpoints support HTTP, not HTTPS.
- HTTPS can be provided using CloudFront.
- Public access needs to be configured carefully.

### 6.11 Lifecycle Configuration

Lifecycle rules automatically manage objects over time.

Two important actions:

**Transition:** Move objects to another storage class.

**Expiration:** Delete objects after a specified period.

Example:

```text
Day 0   → Standard
Day 30  → Standard-IA
Day 90  → Glacier Flexible
Day 365 → Expiration
```

Lifecycle can also clean up old object versions and incomplete multipart uploads.

### 6.12 Tags

Tags are key-value pairs attached to AWS resources.

Example:

```text
Environment = Dev
Project     = DevOps
Owner       = CloudTeam
Department  = Engineering
```

Uses:
- Cost allocation.
- Resource management.
- Automation.
- Attribute-based access control.

### 6.13 Replication

Replication automatically copies objects from a source bucket to a destination bucket.

Two main types:

**CRR – Cross-Region Replication**

Copies objects between different AWS Regions.

Example:

```text
Source Bucket: us-west-2
       |
       |
       | CRR
       |
       v
Destination Bucket: ap-south-1
```

Use case: Disaster recovery and geographic separation.

**SRR – Same-Region Replication**

Copies objects between buckets in the same AWS Region.

Use cases:
- Separate production and backup buckets.
- Centralized log storage.
- Account-level data separation.

Requirements include versioning on both source and destination buckets, an appropriate replication IAM role, and suitable replication configuration.

---

## 7. Important S3 Properties – Quick Revision

| Property | Main purpose |
|---|---|
| Versioning | Maintain object versions |
| Server Access Logging | Record bucket access requests |
| CloudTrail | Audit AWS API activity |
| EventBridge | Trigger event-driven automation |
| Transfer Acceleration | Improve long-distance transfers |
| Object Lock | Protect objects from deletion |
| Requester Pays | Charge requester for applicable requests and transfers |
| Encryption | Protect data at rest |
| Block Public Access | Prevent accidental public exposure |
| Static Website Hosting | Host static websites |
| Lifecycle | Automate transitions and deletion |
| Tags | Organize and track resources |
| Replication | Copy objects to another bucket |

---

## 8. Real-Time DevOps Use Cases

### Scenario 1: EC2 application log backup

```text
EC2 Instance
    |
    |
Application Logs
    |
    |
AWS CLI / IAM Role
    |
    |
S3 Logs Bucket
    |
    |
Lifecycle Policy
    |
    |
Glacier Archive
```

### Scenario 2: CI/CD artifact storage

```text
Developer
    |
    |
GitHub / Jenkins
    |
    |
Build Application
    |
    |
Upload Artifact to S3
    |
    |
Deployment Pipeline
```

### Scenario 3: Disaster Recovery

```text
Production Region
     |
     |
Source S3 Bucket
     |
     |
Cross-Region Replication
     |
     |
DR Region
     |
     |
Destination S3 Bucket
```

### Scenario 4: Automated image processing

```text
Image Upload
     |
     |
S3 Bucket
     |
     |
EventBridge
     |
     |
Lambda Function
     |
     |
Resize / Process Image
```

---

## 9. Important Interview Questions

**Q1. What is Amazon S3?**

Amazon S3 is an AWS object storage service used to store and retrieve any amount of data.

**Q2. What is the difference between EBS, EFS and S3?**

- EBS – Block storage, commonly used with EC2.
- EFS – Managed shared file storage using NFS.
- S3 – Object storage for files, backups and application data.

**Q3. How do you connect EC2 to S3 securely?**

Attach an IAM role with the required S3 permissions to the EC2 instance. The instance receives temporary credentials.

**Q4. What is the difference between `aws s3 cp` and `aws s3 sync`?**

- `cp` copies files or objects.
- `sync` compares source and destination and copies new or changed files.

**Q5. What is the difference between S3 and EFS mounting?**

S3 is object storage and requires a mounting tool for filesystem-like access. EFS provides a managed shared filesystem through NFS.

**Q6. What is S3 Versioning?**

It maintains multiple versions of an object and helps recover from accidental deletion or overwriting.

**Q7. What is S3 Lifecycle Policy?**

It automatically transitions objects between storage classes or expires objects based on configured rules.

**Q8. What is Cross-Region Replication?**

It automatically replicates objects from a source S3 bucket to a destination bucket in another AWS Region.

**Q9. What is Transfer Acceleration?**

It uses AWS edge locations and the AWS global network to improve long-distance data transfers.

**Q10. What is S3 Object Lock?**

It prevents objects from being deleted or overwritten during a specified retention period.

**Q11. Difference between CloudTrail and Server Access Logging?**

CloudTrail records AWS API activity for auditing. Server Access Logging records detailed requests made to an S3 bucket.

**Q12. What is Requester Pays?**

It makes the requester responsible for applicable request and data transfer costs instead of the bucket owner.

**Q13. What happens when we delete an object in a versioned bucket?**

S3 typically adds a delete marker as the current version. Previous versions remain available until permanently deleted.

**Q14. Can we mount S3 directly in Linux?**

Not as a native filesystem. Tools such as s3fs-fuse and Mountpoint for Amazon S3 provide filesystem-like access.

**Q15. What is the difference between S3 Intelligent-Tiering and Lifecycle?**

Intelligent-Tiering automatically changes access tiers based on observed access patterns. Lifecycle follows rules configured by the administrator.

---

## 10. Practical Commands – Quick Revision

```bash
# Check AWS identity
aws sts get-caller-identity

# List all buckets
aws s3 ls

# List objects
aws s3 ls s3://my-devops-learning-bucket

# Create bucket
aws s3 mb s3://my-devops-learning-bucket

# Upload file
aws s3 cp test.txt s3://my-devops-learning-bucket/

# Download file
aws s3 cp s3://my-devops-learning-bucket/test.txt .

# Bulk upload
aws s3 cp ./application/ s3://my-devops-learning-bucket/application/ --recursive

# Bulk download
aws s3 cp s3://my-devops-learning-bucket/application/ ./application/ --recursive

# Sync local to S3
aws s3 sync ./application/ s3://my-devops-learning-bucket/application/

# Sync S3 to local
aws s3 sync s3://my-devops-learning-bucket/application/ ./application/

# Delete one object
aws s3 rm s3://my-devops-learning-bucket/test.txt

# Delete objects recursively
aws s3 rm s3://my-devops-learning-bucket/logs/ --recursive

# Move file
aws s3 mv test.txt s3://my-devops-learning-bucket/

# Generate temporary URL
aws s3 presign s3://my-devops-learning-bucket/test.txt --expires-in 3600

# Delete empty bucket
aws s3 rb s3://my-devops-learning-bucket
```

---

## 11. Final DevOps Learning Checklist

- [ ] Understand S3 bucket and object concepts.
- [ ] Create an S3 bucket using AWS Console.
- [ ] Create an S3 bucket using Terraform.
- [ ] Enable bucket versioning.
- [ ] Configure lifecycle policies.
- [ ] Configure default encryption.
- [ ] Understand bucket policies and IAM policies.
- [ ] Connect EC2 to S3 using an IAM role.
- [ ] Install AWS CLI on Linux.
- [ ] Practise upload and download commands.
- [ ] Practise bulk upload and download.
- [ ] Practise AWS S3 sync.
- [ ] Practise AWS S3 rm and mv.
- [ ] Mount S3 using s3fs-fuse or Mountpoint.
- [ ] Understand all S3 storage classes.
- [ ] Configure server access logging.
- [ ] Understand CloudTrail and EventBridge integration.
- [ ] Understand Transfer Acceleration.
- [ ] Understand Object Lock and Requester Pays.
- [ ] Practise Cross-Region Replication.

**Learning objective:** Be able to create, secure, manage, automate and troubleshoot S3 storage in a real AWS DevOps environment.