# AWS Aurora RDS – Multi-AZ Deployment, Automatic Failover & Web Application Integration

## 1. Practical Objective

In this practical, we create an AWS VPC with public and private subnets, deploy an Aurora MySQL database with Multi-AZ support, connect two EC2 web servers to the database, deploy a PHP registration application and test automatic database failover.

### Learning Objectives

- Create a custom VPC and subnets.
- Configure Internet Gateway and Route Tables.
- Configure Security Groups.
- Create an RDS DB subnet group.
- Deploy Aurora MySQL with Multi-AZ support.
- Understand Writer and Reader endpoints.
- Launch EC2 instances in different Availability Zones.
- Install Apache, PHP and MariaDB MySQL client.
- Connect EC2 to Aurora using the Writer endpoint.
- Create database tables.
- Deploy a PHP registration application.
- Configure Route 53 and Load Balancer.
- Perform manual Aurora failover.
- Verify automatic Writer endpoint redirection.
- Verify database availability after failover.

---

# 2. Architecture Overview

```text
                       INTERNET
                           |
                     Internet Gateway
                           |
                    Application Load Balancer
                           |
                -------------------------
                |                       |
             AZ - 2a                 AZ - 2b
                |                       |
          Public Subnet            Public Subnet
          10.0.1.0/24              10.0.2.0/24
                |                       |
          EC2 Web Server 1         EC2 Web Server 2
          (Apache + PHP)           (Apache + PHP)
                |                       |
                -----------+------------
                           |
                     Private Network
                           |
                Amazon Aurora MySQL
                      DB Cluster
                           |
                ---------------------
                |                   |
             AZ - 2a              AZ - 2b
                |                   |
           Writer Node          Reader Node
                |                   |
                ---------------------
                           |
                    Cluster Storage

                 Route 53 DNS
                       |
                 ALB DNS Alias
```

**Important:** Aurora uses shared cluster storage across its DB instances. The writer and reader are database compute instances, not separate independent databases.

---

# 3. Step 1 – Create a VPC

Navigate to:

AWS Console → VPC → Your VPCs → Create VPC.

Select:

- Resources to create: VPC only
- Name: `rds-web-vpc`
- IPv4 CIDR: `10.0.0.0/20`
- IPv6 CIDR: No IPv6 CIDR block
- Tenancy: Default

Click Create VPC.

### VPC CIDR

```text
VPC CIDR: 10.0.0.0/20
```

This provides 4,096 total IPv4 addresses before AWS subnet reservations.

---

# 4. Step 2 – Create Subnets

Navigate to:

VPC → Subnets → Create subnet.

Select the newly created VPC.

Create four subnets.

### Public Subnets

| Subnet Name | Availability Zone | IPv4 CIDR | Purpose |
|---|---|---|---|
| rds-web-sub-01 | ap-south-2a* | 10.0.1.0/24 | Web Server 1 |
| rds-web-sub-02 | ap-south-2b* | 10.0.2.0/24 | Web Server 2 |

### Private Subnets

| Subnet Name | Availability Zone | IPv4 CIDR | Purpose |
|---|---|---|---|
| rds-db-private-sub-01 | ap-south-2a* | 10.0.3.0/24 | Aurora Database |
| rds-db-private-sub-02 | ap-south-2b* | 10.0.4.0/24 | Aurora Database |

*Select the two actual Availability Zones available in your chosen AWS region. Use the same region for all resources.*

### Enable Auto-Assign Public IPv4

For the two public web subnets:

- Select subnet.
- Actions → Edit subnet settings.
- Enable Auto-assign public IPv4 address.

Do not enable this for the private database subnets.

---

# 5. Step 3 – Create and Attach Internet Gateway

Navigate to:

VPC → Internet Gateways → Create Internet Gateway.

Configuration:

```text
Name: rds-web-igw
```

Click Create Internet Gateway.

### Attach Internet Gateway to VPC

1. Select `rds-web-igw`.
2. Actions → Attach to VPC.
3. Select `rds-web-vpc`.
4. Click Attach Internet Gateway.

Now the VPC has an Internet Gateway attached.

---

# 6. Step 4 – Create Route Table

Navigate to:

VPC → Route Tables → Create Route Table.

Configuration:

```text
Name: rds-web-public-rt
VPC: rds-web-vpc
```

Click Create Route Table.

### Add Internet Gateway Route

1. Select `rds-web-public-rt`.
2. Go to Routes.
3. Click Edit Routes.
4. Add the following route.

| Destination | Target |
|---|---|
| 10.0.0.0/20 | local |
| 0.0.0.0/0 | Internet Gateway (rds-web-igw) |

Save changes.

### Associate Public Subnets

Go to Subnet Associations → Edit Subnet Associations.

Select:

- rds-web-sub-01
- rds-web-sub-02

Save associations.

### Private Database Route Table

Create another route table:

```text
Name: rds-db-private-rt
VPC: rds-web-vpc
```

Keep the local VPC route.

Associate:

- rds-db-private-sub-01
- rds-db-private-sub-02

Do not add a direct Internet Gateway route to the database subnets.

---

# 7. Step 5 – Create Security Groups

Navigate to:

VPC → Security Groups → Create Security Group.

Create separate security groups for the web servers, ALB and database.

## A. Web Server Security Group

```text
Name: rds-web-sg
VPC: rds-web-vpc
```

Inbound rules:

| Type | Protocol | Port | Source |
|---|---|---|---|
| HTTP | TCP | 80 | ALB Security Group |
| HTTPS | TCP | 443 | ALB Security Group |
| SSH | TCP | 22 | Your trusted public IP |
| All ICMP - IPv4 | ICMP | All | Your trusted IP (optional) |

If testing web servers directly without an ALB initially, temporarily allow HTTP from your IP.

## B. Load Balancer Security Group

```text
Name: rds-alb-sg
VPC: rds-web-vpc
```

Inbound rules:

| Type | Protocol | Port | Source |
|---|---|---|---|
| HTTP | TCP | 80 | 0.0.0.0/0 |
| HTTPS | TCP | 443 | 0.0.0.0/0 |

## C. RDS Security Group

```text
Name: rds-db-sg
VPC: rds-web-vpc
```

Inbound rules:

| Type | Protocol | Port | Source |
|---|---|---|---|
| MySQL/Aurora | TCP | 3306 | rds-web-sg |

**Important:** Never expose the RDS MySQL port to the entire internet using `0.0.0.0/0`.

The database should accept connections from the web server security group only.

---

# 8. Step 6 – Create RDS DB Subnet Group

Navigate to:

AWS Console → RDS → Subnet Groups → Create DB Subnet Group.

Configuration:

```text
Name: rds-db-subnet-group
Description: Private subnet group for Aurora MySQL
VPC: rds-web-vpc
```

Select Availability Zones:

- AZ 2a
- AZ 2b

Select subnets:

- rds-db-private-sub-01
- rds-db-private-sub-02

Click Create.

### Why DB Subnet Group?

A DB subnet group tells RDS which VPC subnets it can use for database network placement.

For Multi-AZ availability, select subnets across at least two Availability Zones.

---

# 9. Step 7 – Create Aurora MySQL Database

Navigate to:

AWS Console → RDS → Databases → Create Database.

### A. Engine Options

Select:

```text
Engine: Aurora MySQL-Compatible
```

### B. Database Creation Method

Select:

```text
Full Configuration
```

### C. Templates

Select:

```text
Dev/Test
```

### D. Cluster Scalability Type

Select:

```text
Provisioned
```

### E. Provisioned Configuration

Select:

```text
Burstable Classes
Instance: db.t3.medium
vCPU: 2
RAM: 4 GiB
```

Choose a supported instance class based on region availability and cost.

### F. Settings

Example:

```text
Engine Version: Aurora MySQL 3.x
DB Cluster Identifier: rds-multiaz-cluster
Master Username: root
Credentials Management: Self Managed
```

Use a strong password.

### G. Cluster Storage Configuration

Select:

```text
Aurora Standard
```

### H. Availability and Durability

Select:

```text
Create an Aurora Replica or Reader Node
in a different Availability Zone
```

This creates an additional database instance that can act as a failover target.

### I. Connectivity

Select:

```text
Compute Resource: Don't connect to an EC2 compute resource
Network Type: IPv4
VPC: rds-web-vpc
DB Subnet Group: rds-db-subnet-group
Public Access: No
VPC Security Group: rds-db-sg
```

### J. Additional Configuration

Initial Database Name:

```text
varundb
```

DB Cluster Parameter Group:

```text
default.aurora-mysql8.0
```

DB Parameter Group:

```text
default.aurora-mysql8.0
```

Failover Priority:

```text
tier-0
```

### K. Backup

Example:

```text
Backup Retention Period: 1 day
Backup Window: No Preference
```

For production, configure backup retention according to business recovery requirements.

Review all configurations and click **Create Database**.

---

# 10. Step 8 – Verify Aurora Cluster and Database Instances

Navigate to:

RDS → Databases.

You should see your Aurora DB cluster.

Open the cluster and check its database instances.

Expected configuration:

| Database Instance | AZ | Role |
|---|---|---|
| rds-multiaz-cluster-instance-1 | AZ 2a | Writer |
| rds-multiaz-cluster-instance-2 | AZ 2b | Reader |

Check the status of both instances.

Wait until the database cluster and instances are available.

### Understand Aurora Endpoints

Aurora provides different endpoints.

**1. Cluster Writer Endpoint**

Used for read and write operations.

Example:

```text
rds-multiaz-cluster.cluster-xxxx.ap-south-2.rds.amazonaws.com
```

**2. Reader Endpoint**

Used to distribute read-only connections across available Aurora replicas.

Example:

```text
rds-multiaz-cluster.cluster-ro-xxxx.ap-south-2.rds.amazonaws.com
```

**3. Instance Endpoint**

Each individual database instance also has its own endpoint.

### Important Interview Point

The Aurora cluster writer endpoint is a stable DNS name.

When failover occurs, AWS redirects the writer endpoint to the newly promoted writer instance.

You do not need to manually change the endpoint in your application.

Existing database connections may disconnect during failover. Applications should reconnect using retry logic.

---

# 11. Step 9 – Launch EC2 Web Servers

Navigate to:

EC2 → Instances → Launch Instances.

Create two EC2 instances.

### EC2 Instance 1

```text
Name: web-sub-rds-01
AMI: Amazon Linux
Instance Type: t2.micro or supported free/low-cost option
VPC: rds-web-vpc
Subnet: rds-web-sub-01
Security Group: rds-web-sg
```

### EC2 Instance 2

```text
Name: web-sub-rds-02
AMI: Amazon Linux
Instance Type: t2.micro or supported free/low-cost option
VPC: rds-web-vpc
Subnet: rds-web-sub-02
Security Group: rds-web-sg
```

For this public-subnet lab, enable public IP assignment if needed.

Both instances must be in different Availability Zones.

**Production note:** Prefer private web instances with an ALB in public subnets and use Systems Manager Session Manager or a bastion for administration. The public EC2 arrangement here is a simplified lab setup.

---

# 12. Step 10 – Install Apache, PHP and MySQL Client

Connect to both EC2 instances using SSH or Session Manager.

For Amazon Linux 2023:

```bash
sudo dnf update -y

sudo dnf install httpd php php-mysqli mariadb105 -y
```

For Amazon Linux 2, use the appropriate packages available in its repositories, for example:

```bash
sudo yum update -y

sudo yum install httpd php php-mysqlnd mariadb -y
```

Start Apache:

```bash
sudo systemctl start httpd

sudo systemctl enable httpd
```

Check Apache status:

```bash
sudo systemctl status httpd
```

Check PHP:

```bash
php -v
```

Check MySQL client:

```bash
mysql --version
```

Repeat on both web servers.

---

# 13. Step 11 – Connect EC2 to Aurora MySQL

Login to one EC2 instance.

Copy the Aurora Writer Cluster Endpoint from:

RDS → Databases → Your Aurora Cluster → Connectivity & Security.

Connect using:

```bash
mysql -h <WRITER-CLUSTER-ENDPOINT> -P 3306 -u root -p
```

Enter the master password.

If the connection succeeds, you are connected to Aurora MySQL.

### Verify Database

```sql
SHOW DATABASES;
```

Select the database:

```sql
USE varundb;
```

### Create Sample Table

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Verify table:

```sql
SHOW TABLES;
```

Describe table:

```sql
DESCRIBE users;
```

Insert sample record:

```sql
INSERT INTO users (name, email, password)
VALUES ('Varun', 'varun@example.com', 'test-password');
```

Check entries:

```sql
SELECT * FROM users;
```

Exit:

```sql
EXIT;
```

**Important:** This sample password is only for database testing. Real registration systems must hash passwords using PHP `password_hash()` and verify them using `password_verify()`.

---

# 14. Step 12 – Deploy PHP Registration Application

Reference tutorial:

https://www.tutorialrepublic.com/php-tutorial/php-mysql-login-system.php

Use the registration/login tutorial as the application reference.

Create the application files on both EC2 web servers.

Navigate:

```bash
cd /var/www/html/
```

Create:

```text
/var/www/html/
    |
    |-- register.php
    |
    |-- config.php
```

### A. Create config.php

```bash
sudo vi /var/www/html/config.php
```

Example database connection:

```php
<?php

$host = "YOUR-AURORA-WRITER-ENDPOINT";
$username = "root";
$password = "YOUR-MASTER-PASSWORD";
$database = "varundb";

$conn = new mysqli(
    $host,
    $username,
    $password,
    $database
);

if ($conn->connect_error) {
    die("Database connection failed: " . $conn->connect_error);
}

?>
```

For this lab, we are hardcoding credentials for demonstration purposes.

**Production Best Practice:**
- Store credentials in AWS Secrets Manager.
- Retrieve credentials securely from the application.
- Never commit credentials to GitHub.
- Use a dedicated application database user instead of root.

### B. Create register.php

```bash
sudo vi /var/www/html/register.php
```

Use the registration application code from the TutorialRepublic reference.

Ensure that:
- The PHP file includes `config.php`.
- The registration form accepts name, email and password.
- The application inserts registration data into the `users` table.
- Passwords are hashed before insertion.
- SQL queries use prepared statements.

### C. Configure File Permissions

```bash
sudo chown -R apache:apache /var/www/html/
```

### D. Restart Apache

```bash
sudo systemctl restart httpd

sudo systemctl enable httpd
```

Repeat the application setup on the second EC2 instance.

### E. Verify Application

Open the web server public IP:

```text
http://<EC2-PUBLIC-IP>/register.php
```

Check whether the registration page is loading.

---

# 15. Step 13 – Create Application Load Balancer

Navigate to:

EC2 → Load Balancers → Create Load Balancer.

Select:

```text
Application Load Balancer
```

Configuration:

```text
Name: rds-web-alb
Scheme: Internet-facing
IP Address Type: IPv4
VPC: rds-web-vpc
```

Select public subnets:

- rds-web-sub-01 (AZ 2a)
- rds-web-sub-02 (AZ 2b)

Select Security Group:

```text
rds-alb-sg
```

### Create Target Group

Navigate to Target Groups → Create Target Group.

Select:

```text
Target Type: Instances
Name: rds-web-target-group
Protocol: HTTP
Port: 80
VPC: rds-web-vpc
```

Health check:

```text
Protocol: HTTP
Path: /register.php
```

Register targets:

- web-sub-rds-01
- web-sub-rds-02

Register both instances and create the target group.

### Configure Listener

Configure:

```text
Protocol: HTTP
Port: 80
Default Action: Forward to rds-web-target-group
```

Create the ALB.

Verify both targets become healthy.

---

# 16. Step 14 – Configure Route 53

Navigate to:

AWS Console → Route 53 → Hosted Zones.

### A. Create Hosted Zone

Enter your domain name.

Example:

```text
example.com
```

Select:

```text
Public Hosted Zone
```

Create the hosted zone.

### B. Update Nameservers

Route 53 generates NS records.

Copy the four nameservers and update them at your domain registrar.

This delegates DNS management to Route 53.

### C. Create DNS Record

Navigate to:

Route 53 → Hosted Zone → Create Record.

Configuration:

```text
Record Name: www
Record Type: A
Alias: Yes
Route Traffic To: Application Load Balancer
Region: Your ALB Region
Load Balancer: rds-web-alb
```

Create record.

Now access the application using:

```text
http://www.example.com/register.php
```

**Important:** Route 53 nameservers are updated at the domain registrar. The ALB is connected through a Route 53 Alias record, not by changing the nameservers to the load balancer.

For HTTPS, configure an ACM certificate and an HTTPS listener on port 443.

---

# 17. Step 15 – Test Database Connectivity and Registration

Open:

```text
http://www.example.com/register.php
```

Enter sample registration details:

```text
Name: Varun
Email: varun@example.com
Password: ********
```

Submit the registration form.

Now connect to Aurora from EC2:

```bash
mysql -h <WRITER-CLUSTER-ENDPOINT> -P 3306 -u root -p
```

Select database:

```sql
USE varundb;
```

Check records:

```sql
SELECT * FROM users;
```

Verify whether the registration information is stored successfully.

### What happens?

```text
User
 |
 | Register
 |
Route 53
 |
 |
Application Load Balancer
 |
 |----------------------|
 |                      |
EC2 Web Server 1     EC2 Web Server 2
 |                      |
 |------- PHP ----------|
          |
          |
    Aurora Writer
          |
     Shared Storage
```

Both EC2 web servers communicate with the same Aurora database cluster.

Therefore, registration data submitted through either server is stored in the same database.

---

# 18. Step 16 – Perform Manual Aurora Failover

This is the main objective of our practical.

Navigate to:

AWS Console → RDS → Databases → Select Aurora DB Cluster.

Check the current database instances.

Example:

| Instance | AZ | Role |
|---|---|---|
| Instance 1 | AZ 2a | Writer |
| Instance 2 | AZ 2b | Reader |

### Before Failover

Copy and save the current Writer and Reader instance endpoints.

Check the cluster Writer Endpoint:

```text
rds-multiaz-cluster.cluster-xxxx.rds.amazonaws.com
```

Check the Reader Endpoint:

```text
rds-multiaz-cluster.cluster-ro-xxxx.rds.amazonaws.com
```

### Initiate Manual Failover

1. Open RDS → Databases.
2. Select the Aurora DB cluster.
3. Choose Actions.
4. Select Failover.
5. Confirm the failover operation.

AWS promotes an eligible Aurora replica to become the new writer.

### After Failover

Check the database instance roles again.

Example:

| Instance | Before Failover | After Failover |
|---|---|---|
| Instance 1 (AZ 2a) | Writer | Reader |
| Instance 2 (AZ 2b) | Reader | Writer |

The actual promotion depends on the eligible replicas and failover priority.

### Important Observation

The Aurora cluster Writer Endpoint remains the same DNS name.

However, it automatically redirects to the newly promoted writer instance.

We do not need to manually update the Writer Endpoint inside the PHP application.

The instance-specific endpoints remain associated with their respective DB instances.

---

# 19. Step 17 – Verify Application After Failover

Open the application:

```text
http://www.example.com/register.php
```

Submit another registration:

```text
Name: Varun Test 2
Email: varuntest2@example.com
Password: ********
```

Check the database:

```bash
mysql -h <WRITER-CLUSTER-ENDPOINT> -P 3306 -u root -p
```

Run:

```sql
USE varundb;

SELECT * FROM users;
```

Verify that the newly submitted registration record is present.

### What happened?

1. Initially, Instance 1 was the Writer.
2. Instance 2 was the Reader.
3. We manually initiated failover.
4. AWS promoted Instance 2 to Writer.
5. The cluster Writer Endpoint redirected to the new Writer.
6. The application continued using the same cluster endpoint.
7. New database writes were accepted after connectivity recovered.

**Important:** During failover, existing database connections may be interrupted. Applications must reconnect and retry failed transactions safely. Failover is not necessarily instantaneous.

---

# 20. Important Concept – Writer Endpoint vs Reader Endpoint

| Feature | Writer Endpoint | Reader Endpoint |
|---|---|---|
| Purpose | Read and write | Read-only traffic |
| Connects to | Current writer | Available reader instances |
| Failover | Redirects to promoted writer | Routes to available readers |
| Application use | Insert, Update, Delete, Select | Read-heavy operations |

Example:

```text
Writer Endpoint:
INSERT INTO users ...
UPDATE users ...
DELETE FROM users ...

Reader Endpoint:
SELECT * FROM users ...
```

**Note:** Reader endpoint connections are for read-only workloads. They do not guarantee immediate read-after-write consistency because Aurora replication to readers is asynchronous.

---

# 21. Important Clarification – Backup vs Failover

An automated backup does not normally cause the Writer and Reader roles or endpoints to change.

| Operation | Expected Behaviour |
|---|---|
| Automated Backup | Database continues operating |
| Manual Snapshot | Database continues operating |
| Manual Failover | Eligible reader is promoted to writer |
| Writer Instance Failure | Aurora attempts automatic failover |
| Reader Instance Failure | Writer normally continues serving traffic |

The Writer Endpoint changes its DNS resolution to the newly promoted writer after failover. The endpoint hostname itself remains stable.

---

# 22. Troubleshooting

### Problem 1: EC2 Cannot Connect to Aurora

Check:

- EC2 and RDS are in the correct VPC.
- DB subnet group contains private subnets.
- RDS public access is disabled as intended.
- RDS security group allows TCP 3306 from EC2 security group.
- Network ACLs and routes permit communication.
- Correct Writer Endpoint is used.

Test connectivity:

```bash
nc -zv <RDS-ENDPOINT> 3306
```

### Problem 2: PHP Database Connection Failed

Check:

```bash
php -m | grep mysqli
```

Check Apache:

```bash
sudo systemctl status httpd
```

Check Apache error log:

```bash
sudo tail -f /var/log/httpd/error_log
```

### Problem 3: ALB Target Unhealthy

Check:

- Apache is running.
- Port 80 is open from the ALB security group.
- Target group health check path is correct.
- PHP page is loading correctly.

### Problem 4: Database Connection During Failover

- Wait for failover to complete.
- Retry the connection using the cluster Writer Endpoint.
- Do not hardcode an individual writer instance endpoint.
- Implement application connection retry logic.

---

# 23. Final Practical Checklist

## VPC and Networking

- [ ] Create VPC – 10.0.0.0/20
- [ ] Create public subnet – 10.0.1.0/24 (AZ 2a)
- [ ] Create public subnet – 10.0.2.0/24 (AZ 2b)
- [ ] Create private subnet – 10.0.3.0/24 (AZ 2a)
- [ ] Create private subnet – 10.0.4.0/24 (AZ 2b)
- [ ] Create Internet Gateway
- [ ] Attach Internet Gateway to VPC
- [ ] Create public route table
- [ ] Add Internet Gateway route
- [ ] Associate public subnets
- [ ] Create private route table
- [ ] Associate private subnets
- [ ] Create Security Groups

## RDS and Aurora

- [ ] Create DB subnet group
- [ ] Add two private subnets
- [ ] Create Aurora MySQL cluster
- [ ] Configure provisioned instance
- [ ] Enable Aurora Reader in another AZ
- [ ] Disable public access
- [ ] Attach RDS security group
- [ ] Create initial database
- [ ] Verify Writer and Reader instances
- [ ] Copy Writer and Reader endpoints

## EC2 and Application

- [ ] Launch EC2 in AZ 2a
- [ ] Launch EC2 in AZ 2b
- [ ] Install Apache
- [ ] Install PHP
- [ ] Install MySQL client
- [ ] Connect EC2 to Aurora
- [ ] Create users table
- [ ] Create config.php
- [ ] Create register.php
- [ ] Restart Apache
- [ ] Verify PHP application

## Load Balancer and Route 53

- [ ] Create Application Load Balancer
- [ ] Create Target Group
- [ ] Register both EC2 instances
- [ ] Verify healthy targets
- [ ] Create Route 53 Hosted Zone
- [ ] Update nameservers at domain registrar
- [ ] Create Alias A record
- [ ] Connect Route 53 to ALB
- [ ] Verify application through domain name

## Failover Testing

- [ ] Verify initial Writer
- [ ] Verify initial Reader
- [ ] Perform manual Aurora failover
- [ ] Verify new Writer
- [ ] Verify new Reader
- [ ] Confirm cluster Writer Endpoint remains the same
- [ ] Register another user
- [ ] Verify new entry in database
- [ ] Confirm application works after failover

---

# 24. DevOps Interview Questions

1. What is Multi-AZ deployment in AWS RDS?
2. What is the difference between Aurora and standard RDS Multi-AZ?
3. What is an Aurora Writer Endpoint?
4. What is an Aurora Reader Endpoint?
5. What happens when the primary writer instance fails?
6. How does Aurora automatically perform failover?
7. Does the application need to change its database endpoint after failover?
8. What is the difference between a cluster endpoint and an instance endpoint?
9. Why do we use private subnets for RDS?
10. Why do we need two Availability Zones in a DB subnet group?
11. How does EC2 connect to a private RDS database?
12. How can we distribute database read traffic?
13. Does an automated backup trigger failover?
14. What happens to existing connections during failover?
15. How do you monitor Aurora failover using CloudWatch?
16. What is the role of Route 53 in this architecture?
17. Why do we use an ALB in front of multiple EC2 instances?
18. How do you securely manage RDS credentials in production?
19. How would you troubleshoot an EC2-to-RDS connection failure?
20. How do you implement database connection retries in a production application?

---

## 25. Practical Conclusion

In this practical, we deployed an end-to-end highly available web application architecture using AWS services.

We created a custom VPC with public and private subnets across two Availability Zones, deployed Aurora MySQL with a Writer and Reader instance, connected two EC2 web servers to the database, configured PHP application connectivity, and exposed the application using an Application Load Balancer and Route 53.

Finally, we performed manual Aurora failover and verified that the Writer Endpoint automatically redirected to the newly promoted writer without changing the endpoint configured in the application.

**Key DevOps Learning:** Aurora Multi-AZ failover improves database availability by promoting an eligible reader instance to writer while maintaining the same cluster Writer Endpoint. Applications must still handle connection interruptions and retry safely.