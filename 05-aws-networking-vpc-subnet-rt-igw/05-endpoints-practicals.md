# AWS Practical – Connect a Private EC2 Instance to Amazon S3 Using VPC Gateway Endpoint

## 1. Practical Objective

**Goal:** Connect a private EC2 instance to Amazon S3 using an S3 Gateway Endpoint without using a NAT Gateway or Internet Gateway for S3 connectivity.

### Scenario

We have:

- One VPC.
- One private subnet.
- One private EC2 instance.
- One S3 bucket.
- One private route table.
- One S3 Gateway Endpoint.

Our private EC2 instance does not have a public IP address.

We want to access Amazon S3 from this private EC2 instance.

### Architecture

```text
                    AWS VPC
                       |
                 Private Subnet
                       |
                  Private EC2
                       |
                  Route Table
                       |
               S3 Gateway Endpoint
                       |
                   Amazon S3
```

## 2. Step 1 – Create an S3 Bucket

1. Open AWS Management Console.
2. Search for S3.
3. Select Buckets.
4. Click Create bucket.
5. Enter a bucket name, for example:

   `varun-private-s3-practical`

6. Select the same AWS Region as your VPC.
7. Keep Block Public Access enabled.
8. Create bucket.
9. Open the bucket.
10. Upload a test file.

Example:

`test.txt`

## 3. Step 2 – Create a VPC

1. Open VPC Dashboard.
2. Select Your VPCs.
3. Click Create VPC.
4. Select VPC only.
5. Enter the VPC name:

   `S3-Endpoint-VPC`

6. IPv4 CIDR:

   `10.0.0.0/16`

7. Create VPC.

## 4. Step 3 – Create a Private Subnet

1. Open VPC → Subnets.
2. Click Create subnet.
3. Select `S3-Endpoint-VPC`.
4. Subnet name:

   `Private-Subnet`

5. Select an Availability Zone.
6. IPv4 CIDR:

   `10.0.1.0/24`

7. Create subnet.
8. Keep auto-assign public IPv4 disabled.

## 5. Step 4 – Create a Private Route Table

1. Open VPC → Route Tables.
2. Click Create route table.
3. Enter the name:

   `Private-Route-Table`

4. Select `S3-Endpoint-VPC`.
5. Create route table.

### Associate the private subnet

1. Select `Private-Route-Table`.
2. Open Subnet associations.
3. Click Edit subnet associations.
4. Select `Private-Subnet`.
5. Save associations.

The route table initially contains the local route:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |

## 6. Step 5 – Launch a Private EC2 Instance

1. Open EC2 Dashboard.
2. Click Launch Instance.
3. Name:

   `Private-S3-EC2`

4. Select an Amazon Linux AMI.
5. Select an instance type.
6. Select VPC:

   `S3-Endpoint-VPC`

7. Select subnet:

   `Private-Subnet`

8. Disable Auto-assign public IP.
9. Configure a security group.
10. Launch instance.

The instance should have a private IP address and no public IPv4 address.

### Important

To connect to this private EC2 instance, use AWS Systems Manager Session Manager or another configured private management method.

For Session Manager, provide the required SSM connectivity through Interface Endpoints or another valid network path.

## 7. Step 6 – Create an S3 Gateway Endpoint

This is the most important step.

1. Open VPC Dashboard.
2. Select Endpoints.
3. Click Create endpoint.
4. Enter the endpoint name:

   `S3-Gateway-Endpoint`

5. Under Service category, select AWS services.
6. Search for:

   `com.amazonaws.REGION.s3`

   Example:

   `com.amazonaws.ap-south-1.s3`

7. Select the S3 service with Gateway endpoint type.
8. Select your VPC:

   `S3-Endpoint-VPC`

9. Under Route tables, select:

   `Private-Route-Table`

10. Review the configuration.
11. Click Create endpoint.

## 8. Step 7 – Verify Automatic Route Creation

After creating the Gateway Endpoint, AWS automatically adds a route to the selected route table.

Open:

VPC → Route Tables → Private-Route-Table → Routes

Before creating the endpoint:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |

After creating the endpoint:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |
| S3 prefix list | `vpce-xxxxxxxx` |

### Important points

- We do not manually add the S3 IP address.
- AWS automatically adds the S3 prefix-list route.
- The route table must be associated with the private subnet.
- The Gateway Endpoint must be associated with that route table.
- S3 traffic matching the prefix list uses the Gateway Endpoint.

## 9. Step 8 – Configure S3 Access Permissions

The private EC2 instance needs permission to access S3.

### Recommended method: IAM Role

1. Open IAM Dashboard.
2. Select Roles.
3. Click Create role.
4. Select AWS service.
5. Select EC2.
6. Attach the required S3 permissions.
7. Create the role.
8. Open EC2.
9. Select your private EC2 instance.
10. Choose Actions → Security → Modify IAM role.
11. Attach the IAM role.

Example IAM policy to list and download objects from one bucket:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME"
    },
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

Replace `YOUR-BUCKET-NAME` with your actual bucket name.

**Remember:** The VPC Endpoint provides network connectivity. IAM permissions authorize the EC2 instance to access S3. Both are required.

## 10. Step 9 – Connect to the Private EC2 Instance

Connect through AWS Systems Manager Session Manager.

Open:

EC2 → Instances → Select Private-S3-EC2 → Connect → Session Manager.

Open the terminal session.

Install AWS CLI if it is not already installed.

For Amazon Linux:

```bash
sudo dnf install awscli -y
```

Check the AWS CLI version:

```bash
aws --version
```

## 11. Step 10 – Test S3 Connectivity Using AWS CLI

### First command: List S3 buckets

```bash
aws s3 ls
```

This command lists the S3 buckets accessible to the configured AWS identity.

### Second command: List a particular bucket

```bash
aws s3 ls s3://YOUR-BUCKET-NAME
```

Example:

```bash
aws s3 ls s3://varun-private-s3-practical
```

### Third command: Download an S3 object

```bash
aws s3 cp s3://YOUR-BUCKET-NAME/test.txt .
```

### Fourth command: Upload an object

For this operation, the IAM role must also have `s3:PutObject` permission.

```bash
echo "Hello from Private EC2" > upload.txt

aws s3 cp upload.txt s3://YOUR-BUCKET-NAME/
```

### Fifth command: Verify the uploaded object

```bash
aws s3 ls s3://YOUR-BUCKET-NAME
```

If permissions and endpoint configuration are correct, the private EC2 instance can communicate with S3.

## 12. What if `aws s3 ls` Does Not Work?

An important point from the practical:

The VPC Endpoint does not automatically authenticate the EC2 instance with AWS.

If you run:

```bash
aws s3 ls
```

and receive an error such as:

`Unable to locate credentials`

It means AWS CLI cannot find credentials.

The endpoint may be working correctly, but authentication is missing.

### Option A – Recommended: Attach an IAM Role

Attach an IAM role to the EC2 instance with the required S3 permissions.

AWS CLI automatically retrieves temporary credentials from the EC2 instance's IAM role.

Then run:

```bash
aws s3 ls
```

No access key configuration is required.

### Option B – AWS Configure (For learning only)

You can configure credentials manually:

```bash
aws configure
```

It asks for four values:

```text
AWS Access Key ID:
AWS Secret Access Key:
Default region name:
Default output format:
```

Example:

```text
AWS Access Key ID: YOUR_ACCESS_KEY
AWS Secret Access Key: YOUR_SECRET_KEY
Default region name: ap-south-1
Default output format: json
```

The configuration is saved locally under:

```text
~/.aws/credentials
~/.aws/config
```

**Security warning:** Avoid storing long-term IAM user access keys on EC2 instances. Use an IAM role instead. Never upload credentials to GitHub or share them publicly.

## 13. Important AWS CLI Commands

| Command | Purpose |
|---|---|
| `aws --version` | Check AWS CLI version |
| `aws configure` | Configure AWS CLI credentials and defaults |
| `aws s3 ls` | List accessible S3 buckets |
| `aws s3 ls s3://bucket-name` | List objects in a bucket |
| `aws s3 cp file.txt s3://bucket-name/` | Upload a file |
| `aws s3 cp s3://bucket-name/file.txt .` | Download a file |
| `aws s3 mb s3://bucket-name` | Create a bucket |
| `aws sts get-caller-identity` | Check the current AWS identity |

## 14. Troubleshooting Checklist

If S3 access does not work, check:

- [ ] S3 Gateway Endpoint is in Available state.
- [ ] Correct VPC is selected.
- [ ] Correct AWS Region is selected.
- [ ] S3 Gateway Endpoint is associated with the private route table.
- [ ] Private subnet is associated with that route table.
- [ ] S3 prefix-list route appears in the route table.
- [ ] EC2 instance has an IAM role.
- [ ] IAM role has required S3 permissions.
- [ ] Endpoint policy allows the required S3 actions.
- [ ] Bucket policy does not deny the request.
- [ ] Bucket name and object name are correct.
- [ ] AWS CLI is installed.
- [ ] AWS CLI is using the expected identity and Region.

## 15. Important Interview Questions

**Q1. Why do we need an S3 Gateway Endpoint?**

To access Amazon S3 privately from a VPC without routing S3 traffic through a NAT Gateway or Internet Gateway.

**Q2. Does an S3 Gateway Endpoint create an ENI?**

No. Gateway Endpoints do not create ENIs.

**Q3. What happens to the route table after creating an S3 Gateway Endpoint?**

AWS automatically adds a route using the S3 managed prefix list and the Gateway Endpoint as its target.

**Q4. Do we need a NAT Gateway to access S3 from a private subnet?**

No. An S3 Gateway Endpoint can provide private connectivity to S3.

**Q5. Why does `aws s3 ls` fail even after creating an endpoint?**

Possible reasons include missing IAM credentials, insufficient IAM permissions, restrictive endpoint policies, bucket policies, or incorrect route table association.

**Q6. What is the difference between network connectivity and IAM permission?**

- VPC Endpoint provides network connectivity.
- IAM permissions authorize access to AWS resources.

**Q7. Can we access S3 without assigning a public IP to EC2?**

Yes. We can use an S3 Gateway Endpoint from a private subnet.

**Q8. Do we need to configure `aws configure` if EC2 has an IAM role?**

No. AWS CLI can automatically use temporary credentials from the attached IAM role.

## 16. Practical Learning Checklist

- [ ] Create an S3 bucket.
- [ ] Create a VPC.
- [ ] Create a private subnet.
- [ ] Create a private route table.
- [ ] Associate the private subnet with the route table.
- [ ] Launch a private EC2 instance without a public IP.
- [ ] Create an S3 Gateway Endpoint.
- [ ] Select the correct VPC.
- [ ] Select the S3 Gateway service.
- [ ] Associate the private route table with the endpoint.
- [ ] Verify automatic S3 prefix-list route creation.
- [ ] Attach an IAM role to the EC2 instance.
- [ ] Connect to the EC2 instance.
- [ ] Run `aws s3 ls`.
- [ ] Run `aws s3 ls s3://YOUR-BUCKET-NAME`.
- [ ] Upload and download an S3 object.
- [ ] Verify that S3 access works without NAT Gateway.

## 17. Final Key Takeaways

1. S3 Gateway Endpoint provides private connectivity between a VPC and S3.
2. It does not require a NAT Gateway for S3 connectivity.
3. It does not require an Internet Gateway for S3 connectivity.
4. Gateway Endpoint does not create an ENI.
5. Gateway Endpoint is associated with route tables.
6. AWS automatically creates the S3 prefix-list route.
7. Private EC2 instances do not need public IP addresses.
8. IAM roles provide secure AWS authentication for EC2.
9. `aws s3 ls` lists accessible S3 buckets.
10. `aws s3 ls s3://bucket-name` lists objects inside a specific bucket.

**Remember the difference:**

- NAT Gateway = General outbound internet connectivity.
- S3 Gateway Endpoint = Private connectivity to Amazon S3.
- IAM Role = Authentication and authorization for AWS API access.