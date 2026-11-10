# AWS RDS & Amazon Aurora – Database Creation Practical

## 1. Introduction

**Amazon RDS (Relational Database Service)** is a managed database service provided by AWS. It allows us to create, operate and scale relational databases without managing the underlying operating system and database infrastructure.

In this practical, we create an **Amazon Aurora MySQL-compatible database**.

### Supported Database Engines

| Database Engine | Description |
|---|---|
| Aurora MySQL | AWS-managed MySQL-compatible database |
| Aurora PostgreSQL | AWS-managed PostgreSQL-compatible database |
| MySQL | Open-source relational database |
| PostgreSQL | Open-source relational database |
| MariaDB | MySQL-compatible relational database |
| Oracle | Commercial relational database |
| Microsoft SQL Server | Microsoft's relational database |
| IBM Db2 | Enterprise relational database |

**Important:** Select the database engine based on application compatibility and migration requirements.

---

## 2. Database Creation Method

AWS provides two database creation methods.

### Full Configuration
- Manually configure all database settings.
- Configure availability and durability.
- Select instance size.
- Configure networking and security.
- Configure backups and maintenance.

### Easy Create
- AWS automatically selects recommended configurations.
- Limited configuration options.
- Some settings can be modified after creation.

**Practical Selection:** Full Configuration.

---

## 3. Templates

AWS provides two templates:

| Template | Purpose |
|---|---|
| Production | High availability and production workloads |
| Dev/Test | Development and testing workloads |

**Practical Selection:** Dev/Test.

---

## 4. Cluster Scalability Type

Two options are available.

### A. Aurora Serverless
- Automatically adjusts database capacity based on workload.
- Suitable for variable workloads.
- Capacity scales according to configured limits.

### B. Provisioned
- Fixed database compute capacity.
- We select the database instance class.
- Suitable for predictable workloads.

**Practical Selection:** Provisioned.

### Type of Provisioned Configuration

| Configuration | Description |
|---|---|
| Memory Optimized (R classes) | Suitable for workloads requiring large memory |
| Burstable (T classes) | Suitable for variable CPU workloads |

Our selection:

- Instance class: `db.t3.medium`
- vCPU: 2
- RAM: 4 GiB

**Cost Optimization:** Choose a smaller supported instance for learning environments and check regional pricing before deployment.

---

## 5. Settings

### Engine Version

Selected:

`Aurora MySQL 3.10.3 (compatible with MySQL 8.0.42)`

### DB Cluster Identifier

Example:

`database-1`

This is the unique identifier for the Aurora DB cluster in your AWS account and region.

### Credentials Settings

| Field | Selected Value |
|---|---|
| Master Username | root |
| Credentials Management | Self Managed |
| Password | User-defined |

Two credentials management options:

1. **Managed in AWS Secrets Manager:** AWS manages the master password through Secrets Manager.
2. **Self Managed:** We manage the master password ourselves.

**DevOps Best Practice:** Never hardcode database passwords in application code, Terraform files or CI/CD pipelines. Use AWS Secrets Manager or another approved secrets management solution.

---

## 6. Cluster Storage Configuration

Two options are available:

| Configuration | Description |
|---|---|
| Aurora I/O-Optimized | Predictable pricing for I/O-intensive workloads |
| Aurora Standard | Compute and storage pricing with additional I/O charges |

**Practical Selection:** Aurora Standard.

Choose based on expected I/O usage and total workload cost.

---

## 7. Availability and Durability – Multi-AZ

Aurora supports high availability through its cluster architecture.

### Option 1: Create an Aurora Replica or Reader Node in a Different AZ
- Creates an additional reader instance.
- Supports read scaling.
- Provides a failover target.
- Improves availability.

### Option 2: Don't Create an Aurora Replica
- Creates the cluster without an additional reader instance.
- Lower compute cost.
- Less redundancy at the DB instance level.

**Practical Selection:** Create an Aurora Replica or Reader Node in a Different AZ.

**Important Interview Point:**

Aurora replication and traditional RDS Multi-AZ deployments are different architectures. Aurora uses shared cluster storage and reader instances to support failover.

---

## 8. Connectivity Configuration

Connectivity determines how our database communicates with other AWS resources.

### A. Compute Resource

Two options:

1. Don't connect to an EC2 compute resource.
2. Connect to an EC2 compute resource.

**Practical Selection:** Don't connect to an EC2 compute resource.

We can manually connect an EC2 application server to RDS later using security groups and database connection settings.

RDS does not require us to launch EC2 just to host the database.

### B. Network Type

| Option | Description |
|---|---|
| IPv4 | Communication through IPv4 |
| Dual-stack | Communication through IPv4 and IPv6 |

**Practical Selection:** IPv4.

### C. Virtual Private Cloud (VPC)

Selected:

- VPC Name: `rds-vpc`
- VPC ID: `vpc-0e3fed9c01a9b82eb`
- Available Subnets: 4
- Availability Zones: 2

VPC provides an isolated networking environment for the database.

### D. DB Subnet Group

Selected:

`skoda-rds-subnet`

- Subnets: 2
- Availability Zones: 2

**Why do we need a DB Subnet Group?**

A DB subnet group is a collection of subnets inside a VPC that RDS can use for database placement.

Benefits:
- Identifies which subnets RDS can use.
- Provides subnet placement across Availability Zones.
- Supports Multi-AZ deployment.
- Provides network placement flexibility.

### E. Public Access

Two options:

| Option | Description |
|---|---|
| Yes | Assigns a publicly accessible IP to the cluster |
| No | Does not assign a public IP to the cluster |

**Practical Selection:** No.

Why use private access?
- Database is not directly exposed to the internet.
- Only permitted resources can access it.
- Better security for production workloads.
- EC2 can communicate with RDS privately inside the VPC.

**Important:** A private subnet is determined by its route table, not merely its name. Public accessibility also depends on the RDS setting and network configuration.

### F. VPC Security Group

Two options:

1. Choose existing security group.
2. Create new security group.

Example RDS inbound security group rule:

| Type | Protocol | Port | Source |
|---|---|---|---|
| MySQL/Aurora | TCP | 3306 | EC2 Security Group |

Do not allow MySQL port 3306 from `0.0.0.0/0` in a normal production setup.

---

## 9. Additional Database Configuration

### Initial Database Name

Selected:

`varundb`

This creates the initial database inside the Aurora cluster.

If we leave this field empty, AWS does not create an initial database automatically.

### DB Cluster Parameter Group

Selected:

`default.aurora-mysql8.0`

Controls cluster-level database configuration parameters.

### DB Parameter Group

Selected:

`default.aurora-mysql8.0`

Controls database instance-level parameters.

### Failover Priority

Selected:

`tier-0`

Controls the failover promotion priority of an Aurora replica.

Tier 0 has the highest promotion priority.

---

## 10. Backup Configuration

### Backup Retention Period

Selected: 1 day.

Backup retention specifies how many days automated backups are retained.

Examples:

- 1 day – Retain backups for one day.
- 7 days – Retain backups for seven days.
- 35 days – Retain backups for 35 days.

Aurora supports automated backups and point-in-time recovery within the configured retention period.

### Backup Window

Selected: No Preference.

AWS automatically chooses the backup window.

**Production Best Practice:** Select an appropriate retention period according to business recovery requirements.

---

## 11. Final Architecture

Our practical architecture:

```text
                   AWS CLOUD
                       |
                      VPC
                       |
             ---------------------
             |                   |
          AZ - 1a              AZ - 1b
             |                   |
       Private Subnet       Private Subnet
             |                   |
       Aurora Writer        Aurora Reader
             |                   |
             ------- Aurora ------
                    Cluster
                       |
                 DB Endpoint
                       |
              EC2 Application Server
```

Note: The application EC2 instance connects to the Aurora cluster endpoint through permitted network paths. The reader endpoint can be used for read traffic.

---

## 12. How EC2 Connects to RDS

Example:

```text
EC2 Private IP: 10.0.2.10
RDS Private IP: 10.0.3.20
Database Port: 3306
```

EC2 connects to RDS using the database endpoint.

Example endpoint:

```text
database-1.cluster-xxxx.ap-south-1.rds.amazonaws.com
```

Example MySQL connection:

```bash
mysql -h <RDS-ENDPOINT> -P 3306 -u root -p
```

Security group configuration:

| Resource | Inbound Rule |
|---|---|
| EC2 Security Group | Application traffic |
| RDS Security Group | MySQL TCP 3306 from EC2 Security Group |

EC2 and RDS can communicate privately within the VPC when routing and security group rules permit it.

---

## 13. Important DevOps Concepts

| Concept | Explanation |
|---|---|
| RDS | Managed relational database service |
| Aurora | AWS-managed MySQL/PostgreSQL-compatible database |
| DB Subnet Group | Collection of subnets used for RDS placement |
| VPC | Isolated networking environment |
| Private Subnet | Subnet without a direct Internet Gateway route |
| Security Group | Virtual firewall |
| Multi-AZ | High availability |
| Reader Instance | Handles read traffic |
| Writer Instance | Handles write traffic |
| DB Endpoint | DNS address used to connect to database |
| Parameter Group | Controls database configuration |
| Secrets Manager | Secure credential management |
| Automated Backups | Backup and recovery |

---

## 14. RDS vs Database on EC2

| Feature | Amazon RDS | Database on EC2 |
|---|---|---|
| OS Management | AWS | User |
| Database Installation | AWS | User |
| Patching | AWS-managed | User |
| Backup | Built-in features | User configures |
| High Availability | Managed options | User configures |
| SSH Access | No underlying host access | Yes |
| Database Control | Limited to supported features | Full control |
| Maintenance | Reduced operational effort | More operational effort |

---

## 15. Important Interview Questions

1. What is Amazon RDS?
2. What is the difference between Amazon RDS and Amazon Aurora?
3. Why do we need a DB subnet group?
4. Why should a production database be deployed in private subnets?
5. What is the difference between Aurora Serverless and Provisioned?
6. What is the difference between a writer and reader instance?
7. What happens when the Aurora writer instance fails?
8. What is an Aurora cluster endpoint?
9. What is a DB parameter group?
10. How does EC2 connect to RDS?
11. How do you securely store database credentials?
12. Can we SSH into an RDS database instance?
13. Does RDS require an EC2 instance to run its database?
14. What is the difference between Multi-AZ and Read Replica?
15. What is point-in-time recovery in RDS?

---

## 16. Practical Checklist

- [ ] Select Aurora MySQL-compatible engine.
- [ ] Select Full Configuration.
- [ ] Choose Dev/Test template.
- [ ] Choose Provisioned capacity.
- [ ] Select burstable instance class.
- [ ] Configure cluster identifier.
- [ ] Configure master username and password.
- [ ] Select Aurora Standard storage.
- [ ] Configure Aurora Replica / Reader.
- [ ] Select VPC.
- [ ] Select DB subnet group.
- [ ] Disable public access.
- [ ] Configure security group.
- [ ] Enter initial database name.
- [ ] Review parameter groups.
- [ ] Configure backup retention.
- [ ] Review estimated monthly cost.
- [ ] Create database.

---

## 17. Next Practical – Connect EC2 to Aurora MySQL

After the database status becomes Available:

1. Launch or select a Linux EC2 instance.
2. Configure the EC2 security group.
3. Allow MySQL TCP port 3306 from EC2 security group in the RDS security group.
4. Install MySQL client on EC2.
5. Connect using the Aurora cluster endpoint.
6. Create a database table.
7. Insert records.
8. Retrieve records using SELECT.

Example commands:

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    role VARCHAR(100)
);

INSERT INTO employees
VALUES (1, 'Varun', 'DevOps Engineer');

SELECT * FROM employees;
```

**Learning Objective:** Understand how an application server communicates with a managed database using private networking and security groups.

---

## 18. Cost Optimization – Important

- Choose the smallest suitable database instance.
- Avoid unnecessary Aurora reader instances in learning environments.
- Monitor storage and I/O costs.
- Review backup retention.
- Check regional pricing before deployment.
- Delete unused database clusters after completing practicals.
- Remember that Dev/Test configuration is not automatically free.

**Final DevOps Takeaway:**

Amazon RDS simplifies database infrastructure management. As DevOps engineers, our responsibilities include database provisioning, networking, security, monitoring, backup strategy, automation, availability and cost optimization.