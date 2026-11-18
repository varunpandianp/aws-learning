
# AWS CLI – Complete Study Notes

## 1. What is AWS CLI?

AWS CLI (Amazon Web Services Command Line Interface) is an open-source command-line tool provided by AWS that allows users to interact with and manage AWS services directly from a terminal.

Instead of accessing AWS through the Management Console, we can execute commands to create, configure, manage, monitor and delete AWS resources.

### Examples

- Launch EC2 instances.
- Create and manage S3 buckets.
- Upload and download files from S3.
- Start, stop and terminate EC2 instances.
- Manage IAM users and roles.
- Monitor AWS resources.
- Automate infrastructure operations.

**Interview Definition:**

AWS CLI is a command-line interface that allows users to interact with AWS services and automate cloud infrastructure management using commands and scripts.

---

## 2. Why Do We Need AWS CLI?

AWS CLI is important for DevOps engineers, Cloud Engineers and SREs because it helps automate AWS infrastructure operations.

### Main reasons

1. Automation – Automate repetitive AWS operations.
2. Time saving – Execute operations quickly using commands.
3. Resource management – Create, update and delete AWS resources.
4. CI/CD integration – Integrate AWS operations with Jenkins and GitHub Actions.
5. Bulk operations – Manage multiple resources simultaneously.
6. Troubleshooting – Retrieve AWS resource information.
7. Scripting – Combine AWS CLI with Bash and Python scripts.

### Real-world DevOps use cases

- Jenkins uploads build artifacts to S3.
- Shell scripts start and stop EC2 instances.
- Deployment scripts retrieve EC2 instance information.
- Backup scripts upload files to S3.
- CI/CD pipelines automate application deployments.

---

## 3. How AWS CLI Works

AWS CLI works as an interface between the user and AWS services.

### Working process

1. User executes an AWS CLI command.
2. AWS CLI reads configuration and credentials.
3. AWS CLI authenticates the request.
4. It sends an API request to the relevant AWS service.
5. AWS processes the request.
6. AWS returns the response to the terminal.

Example:

```bash
aws ec2 describe-instances
```

AWS CLI communicates with the EC2 API and retrieves instance information.

---

## 4. AWS CLI Installation

AWS CLI Version 2 is recommended.

### Windows Installation

Download the official MSI installer:

https://awscli.amazonaws.com/AWSCLIV2.msi

Installation steps:

1. Download AWSCLIV2.msi.
2. Double-click the installer.
3. Accept the license agreement.
4. Click Install.
5. Complete installation.

Verify:

```cmd
aws --version
```

Example output:

```text
aws-cli/2.41.0 Python/3.14.8 Windows/11 exe/AMD64
```

### Ubuntu Installation

Update packages:

```bash
sudo apt update -y
```

Install dependencies:

```bash
sudo apt install unzip curl -y
```

Download AWS CLI:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
```

Extract:

```bash
unzip awscliv2.zip
```

Install:

```bash
sudo ./aws/install
```

Verify:

```bash
aws --version
```

Official documentation:

https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

---

## 5. AWS CLI Configuration

After installation, AWS CLI needs authentication credentials and default configuration.

Run:

```bash
aws configure
```

It asks for four details:

| Configuration | Example |
|---|---|
| AWS Access Key ID | IAM access key |
| AWS Secret Access Key | IAM secret key |
| Default region | us-west-2 |
| Default output format | json |

The values above are examples. Use your own authorized credentials.

### Configuration Files

AWS CLI creates two important files.

**1. Credentials file**

Windows:

```text
C:\Users\YourUsername\.aws\credentials
```

Linux:

```text
~/.aws/credentials
```

Stores credentials.

**2. Config file**

Windows:

```text
C:\Users\YourUsername\.aws\config
```

Linux:

```text
~/.aws/config
```

Stores region, output format and profile configuration.

### Verify configuration

```bash
aws configure list
```

Check region:

```bash
aws configure get region
```

Check output format:

```bash
aws configure get output
```

### Change default region

Example: Mumbai region.

```bash
aws configure set region ap-south-1
```

Verify:

```bash
aws configure get region
```

### Important Security Best Practices

- Never upload access keys to GitHub.
- Never share secret keys in screenshots.
- Use IAM roles for EC2 workloads.
- Prefer temporary credentials and IAM Identity Center where appropriate.
- Follow least-privilege IAM permissions.
- Rotate exposed access keys immediately.

---

## 6. AWS CLI Command Structure

General syntax:

```bash
aws <service> <operation> [options]
```

Example:

```bash
aws ec2 describe-instances
```

Explanation:

- aws – AWS CLI executable.
- ec2 – AWS service.
- describe-instances – Operation.

Another example:

```bash
aws s3 ls
```

- aws – AWS CLI executable.
- s3 – S3 service.
- ls – List operation.

---

## 7. Basic AWS CLI Commands

### General Commands

Check AWS CLI version:

```bash
aws --version
```

Check AWS identity:

```bash
aws sts get-caller-identity
```

Check configuration:

```bash
aws configure list
```

Display help:

```bash
aws help
```

Service-specific help:

```bash
aws ec2 help
```

### IAM Commands

List IAM users:

```bash
aws iam list-users
```

List IAM roles:

```bash
aws iam list-roles
```

List locally created IAM policies:

```bash
aws iam list-policies --scope Local
```

### S3 Commands

List buckets:

```bash
aws s3 ls
```

Create bucket:

```bash
aws s3 mb s3://my-unique-bucket-12345 --region us-west-2
```

Upload file:

```bash
aws s3 cp test.txt s3://my-unique-bucket-12345/
```

Download file:

```bash
aws s3 cp s3://my-unique-bucket-12345/test.txt .
```

List bucket contents:

```bash
aws s3 ls s3://my-unique-bucket-12345/
```

Synchronize directory:

```bash
aws s3 sync ./backup s3://my-unique-bucket-12345/backup/
```

Delete object:

```bash
aws s3 rm s3://my-unique-bucket-12345/test.txt
```

Delete empty bucket:

```bash
aws s3 rb s3://my-unique-bucket-12345
```

### EC2 Commands

List EC2 instances:

```bash
aws ec2 describe-instances
```

Launch EC2 instance:

```bash
aws ec2 run-instances
```

Start instance:

```bash
aws ec2 start-instances --instance-ids i-1234567890abcdef0
```

Stop instance:

```bash
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
```

Reboot instance:

```bash
aws ec2 reboot-instances --instance-ids i-1234567890abcdef0
```

Terminate instance:

```bash
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0
```

### CloudWatch Commands

List available metrics:

```bash
aws cloudwatch list-metrics
```

List CloudWatch alarms:

```bash
aws cloudwatch describe-alarms
```

Retrieve metric statistics:

```bash
aws cloudwatch get-metric-statistics
```

Note: get-metric-statistics requires namespace, metric name, dimensions, time range and period parameters.

---

## 8. Practical 1 – Launch EC2 Using AWS CLI

**Objective:** Launch an Ubuntu EC2 instance without using AWS Management Console.

### Step 1: Get Ubuntu AMI ID

For Ubuntu 24.04 in the current region:

```cmd
aws ssm get-parameter --name "/aws/service/canonical/ubuntu/server/24.04/stable/current/amd64/hvm/ebs-gp3/ami-id" --query "Parameter.Value" --output text
```

Copy the returned AMI ID.

### Step 2: List available key pairs

```cmd
aws ec2 describe-key-pairs --output table
```

### Step 3: List security groups

```cmd
aws ec2 describe-security-groups --query "SecurityGroups[*].[GroupId,GroupName]" --output table
```

Select a security group that allows SSH access from your IP address.

### Step 4: Launch EC2 instance

```cmd
aws ec2 run-instances --image-id ami-0abcdef1234567890 --instance-type t3.micro --key-name my-ubuntu-key --security-group-ids sg-0123456789abcdef0 --count 1 --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=Ubuntu-CLI-Instance}]"
```

Replace the dummy AMI ID, key pair name and security group ID with actual resources in your AWS account.

### Step 5: Verify EC2 instance

```cmd
aws ec2 describe-instances --output table
```

Retrieve instance ID, state and public IP:

```cmd
aws ec2 describe-instances --query "Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress,PrivateIpAddress]" --output table
```

### Step 6: Stop EC2

```cmd
aws ec2 stop-instances --instance-ids i-xxxxxxxxxxxxxxxxx
```

### Step 7: Start EC2

```cmd
aws ec2 start-instances --instance-ids i-xxxxxxxxxxxxxxxxx
```

### Step 8: Reboot EC2

```cmd
aws ec2 reboot-instances --instance-ids i-xxxxxxxxxxxxxxxxx
```

### Step 9: Terminate EC2

```cmd
aws ec2 terminate-instances --instance-ids i-xxxxxxxxxxxxxxxxx
```

### Step 10: Verify termination

```cmd
aws ec2 describe-instances --instance-ids i-xxxxxxxxxxxxxxxxx
```

Instance termination is permanent. The root EBS volume is deleted by default unless configured otherwise.

---

## 9. Practical 2 – Create S3 Bucket Using AWS CLI

**Objective:** Create an S3 bucket and upload/download files using AWS CLI.

### Step 1: List buckets

```bash
aws s3 ls
```

### Step 2: Create bucket

```bash
aws s3 mb s3://my-cli-practical-unique-12345 --region us-west-2
```

Bucket names must be globally unique.

### Step 3: Create a sample file

Windows CMD:

```cmd
echo Hello AWS CLI > test.txt
```

### Step 4: Upload file

```bash
aws s3 cp test.txt s3://my-cli-practical-unique-12345/
```

### Step 5: Verify uploaded file

```bash
aws s3 ls s3://my-cli-practical-unique-12345/
```

### Step 6: Download file

```bash
aws s3 cp s3://my-cli-practical-unique-12345/test.txt downloaded.txt
```

### Step 7: Delete bucket and objects

```bash
aws s3 rb s3://my-cli-practical-unique-12345 --force
```

Warning: --force deletes objects inside the bucket before removing the bucket.

---

## 10. Practical 3 – AWS CLI Filters and Queries

Filters and queries help retrieve only the required AWS resource information.

### List running instances

```bash
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"
```

### Get only instance IDs

```bash
aws ec2 describe-instances --query "Reservations[*].Instances[*].InstanceId" --output text
```

### Get instance ID and public IP

```bash
aws ec2 describe-instances --query "Reservations[*].Instances[*].[InstanceId,PublicIpAddress]" --output table
```

### Filter instances by name

```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=Ubuntu-CLI-Instance"
```

### Output formats

AWS CLI supports three common formats:

| Format | Description |
|---|---|
| json | Structured JSON output |
| text | Plain text output |
| table | Human-readable table |

Examples:

```bash
aws ec2 describe-instances --output json
aws ec2 describe-instances --output text
aws ec2 describe-instances --output table
```

---

## 11. AWS CLI Profiles

Profiles allow us to manage multiple AWS accounts or environments.

Example:

- Development account.
- Testing account.
- Production account.

Create development profile:

```bash
aws configure --profile dev
```

Create production profile:

```bash
aws configure --profile prod
```

Use development profile:

```bash
aws s3 ls --profile dev
```

Use production profile:

```bash
aws s3 ls --profile prod
```

List profiles:

```bash
aws configure list-profiles
```

Profiles maintain separate credentials and configurations.

---

## 12. AWS CLI vs AWS Console vs AWS SDK

| Feature | AWS Console | AWS CLI | AWS SDK |
|---|---|---|---|
| Interface | Web browser | Terminal | Programming code |
| Usage | Manual operations | Commands and scripts | Application integration |
| Automation | Limited | Excellent | Excellent |
| Example | Launch EC2 through UI | aws ec2 run-instances | Python Boto3 |
| Best for | Visual management | DevOps and SRE | Developers |

### What is AWS SDK?

AWS SDK (Software Development Kit) allows developers to interact with AWS services using programming languages such as Python, Java, JavaScript and Go.

Example using Python Boto3:

```python
import boto3

ec2 = boto3.client('ec2')

response = ec2.describe_instances()

print(response)
```

---

## 13. AWS CLI in DevOps CI/CD Pipelines

AWS CLI is widely used in CI/CD pipelines for automating deployments.

### Example Jenkins workflow

1. Developer pushes code to GitHub.
2. Jenkins starts a pipeline.
3. Application is built.
4. Build artifact is uploaded to S3.
5. Deployment process retrieves the artifact.
6. Application is deployed to EC2.

Example Jenkins shell commands:

```bash
aws s3 cp application.zip s3://my-deployment-bucket/

aws s3 ls s3://my-deployment-bucket/
```

AWS CLI can integrate with:

- Jenkins
- GitHub Actions
- Terraform workflows
- Ansible
- Shell scripts
- Python automation

Production pipelines should use IAM roles or OIDC federation and temporary credentials instead of long-lived access keys.

---

## 14. AWS CLI Troubleshooting

| Error | Possible reason | Solution |
|---|---|---|
| Unable to locate credentials | Credentials not configured | Run aws configure |
| InvalidClientTokenId | Invalid credentials | Verify credentials |
| AccessDenied | Insufficient IAM permissions | Check IAM policy |
| InvalidAMIID.NotFound | Incorrect AMI or region | Verify AMI and region |
| InvalidKeyPair.NotFound | Key pair unavailable | Check key pair in region |
| UnauthorizedOperation | Missing IAM permission | Review IAM policy |
| BucketAlreadyExists | S3 name already taken | Choose another name |

---

## 15. AWS CLI Interview Questions

### Q1. What is AWS CLI?

AWS CLI is a command-line tool used to interact with AWS services and automate cloud infrastructure management.

### Q2. Why do we use AWS CLI?

To automate AWS resource creation, management, monitoring and deployment without manually accessing the AWS Console.

### Q3. Difference between AWS CLI and AWS SDK?

AWS CLI manages AWS services through terminal commands, whereas AWS SDK integrates AWS services into applications using programming languages.

### Q4. What is the purpose of aws configure?

It configures AWS credentials, default region and output format.

### Q5. What is an AWS profile?

An AWS profile stores a separate set of AWS CLI credentials and configuration.

### Q6. Difference between stop and terminate EC2?

Stop temporarily shuts down an EC2 instance, while terminate permanently deletes the instance.

### Q7. How do you check the current AWS account?

```bash
aws sts get-caller-identity
```

### Q8. How does an EC2 instance authenticate with AWS CLI?

Using an attached IAM instance profile and temporary security credentials.

### Q9. What is the default output format in AWS CLI?

JSON is the default output format unless another format is configured.

### Q10. Can AWS CLI be integrated with Jenkins?

Yes. Jenkins can execute AWS CLI commands in pipeline stages to automate resource management and application deployment.

---

## 16. Hands-on AWS CLI Assignment

### Installation and Configuration

- [ ] Install AWS CLI v2.
- [ ] Verify AWS CLI version.
- [ ] Configure AWS CLI.
- [ ] Check AWS identity.

### EC2 Management

- [ ] List EC2 instances.
- [ ] Launch Ubuntu EC2 instance.
- [ ] Retrieve instance ID and public IP.
- [ ] Stop EC2 instance.
- [ ] Start EC2 instance.
- [ ] Reboot EC2 instance.
- [ ] Terminate EC2 instance.

### S3 Management

- [ ] Create S3 bucket.
- [ ] Upload a file.
- [ ] Download a file.
- [ ] List bucket objects.
- [ ] Synchronize a local directory with S3.
- [ ] Delete S3 bucket.

### Advanced Practice

- [ ] Create multiple AWS CLI profiles.
- [ ] Use EC2 filters.
- [ ] Retrieve specific information using JMESPath queries.
- [ ] Execute AWS CLI commands through a shell script.
- [ ] Integrate AWS CLI into a Jenkins pipeline.

---

## 17. AWS CLI Quick Revision Cheat Sheet

| Service / Operation | Command |
|---|---|
| Check version | aws --version |
| Check identity | aws sts get-caller-identity |
| Configuration | aws configure list |
| EC2 list | aws ec2 describe-instances |
| EC2 launch | aws ec2 run-instances |
| EC2 start | aws ec2 start-instances |
| EC2 stop | aws ec2 stop-instances |
| EC2 reboot | aws ec2 reboot-instances |
| EC2 terminate | aws ec2 terminate-instances |
| S3 list | aws s3 ls |
| S3 create | aws s3 mb |
| S3 upload | aws s3 cp |
| S3 download | aws s3 cp |
| S3 sync | aws s3 sync |
| IAM users | aws iam list-users |
| IAM roles | aws iam list-roles |
| CloudWatch metrics | aws cloudwatch list-metrics |

---

## 18. Important Learning Resources

1. AWS CLI User Guide:
   https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html

2. AWS CLI Command Reference:
   https://docs.aws.amazon.com/cli/latest/reference/

3. AWS CLI Configuration:
   https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html

4. IAM Security Best Practices:
   https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html

---

## 19. Real-world DevOps Interview Scenario

**Question:**

You have 50 EC2 instances running in AWS. Your manager asks you to stop all instances having the tag Environment=dev. How would you achieve this using AWS CLI?

**Hint:**

Use:

```bash
aws ec2 describe-instances
```

Filter instances using:

```text
Name=tag:Environment,Values=dev
```

Retrieve their instance IDs and pass those IDs to:

```bash
aws ec2 stop-instances
```

Remember to exclude instances that are already stopped or terminated. Verify the target list before executing the stop operation.

---

## 20. Final Learning Roadmap

Learn AWS CLI in this order:

1. Installation and configuration.
2. IAM authentication and permissions.
3. EC2 lifecycle management.
4. S3 bucket and object management.
5. VPC and networking commands.
6. CloudWatch monitoring commands.
7. Shell scripting with AWS CLI.
8. Jenkins and GitHub Actions integration.

**Key takeaway:**

AWS CLI is an essential tool for DevOps and SRE engineers because it enables automation, faster resource management, troubleshooting and integration with CI/CD pipelines.
