# AWS RDS (Relational Database Service) – Complete Notes

## 1. What is AWS RDS?

**Amazon RDS (Relational Database Service)** is a fully managed relational database service provided by AWS.

It allows users to create, operate and scale relational databases in the AWS Cloud without managing the underlying operating system and database infrastructure.

AWS manages:
- Database provisioning.
- Database engine installation.
- Operating system maintenance.
- Database patching.
- Automated backups.
- Monitoring.
- High availability.
- Failover.
- Storage management.

As a DevOps Engineer, we focus on database provisioning, security, connectivity, monitoring, automation, backup and recovery.

---

## 2. What is a Database?

A database is an organized collection of information that can be stored, accessed, managed and updated.

Example: An e-commerce application stores:

- Customer details.
- Product information.
- Orders.
- Payments.
- Inventory.
- Delivery information.

There are two major categories of databases.

### A. Relational Database (SQL)

Relational databases store information in tables containing rows and columns.

Example:

**Employees Table**

| ID | Name | Role | Salary |
|---|---|---|---|
| 1 | Varun | DevOps Engineer | 50000 |
| 2 | Arun | Developer | 45000 |
| 3 | Kumar | Tester | 40000 |

Characteristics:
- Structured data.
- Tables, rows and columns.
- Primary keys and foreign keys.
- SQL queries.
- Relationships between tables.
- ACID transactions.

Examples:
- MySQL.
- PostgreSQL.
- Oracle.
- Microsoft SQL Server.
- MariaDB.
- IBM Db2.

### B. Non-Relational Database (NoSQL)

NoSQL databases store data in formats other than traditional relational tables.

Common data models:

| Type | Example |
|---|---|
| Key-Value | Redis, DynamoDB |
| Document | MongoDB |
| Wide-Column | Cassandra |
| Graph | Neo4j |

Characteristics:
- Flexible data structures.
- Suitable for certain large-scale distributed workloads.
- Can support high-throughput applications.
- Data relationships are handled differently from relational databases.

**Important:** Amazon RDS is designed for relational databases. Amazon DynamoDB is an AWS-managed NoSQL database service.

---

## 3. What is a Database Engine?

A database engine is the underlying software that manages how data is stored, retrieved, updated and secured.

It is responsible for:
- Query processing.
- Data storage and retrieval.
- Transaction management.
- Concurrency control.
- Indexing.
- Database security.
- Backup and recovery capabilities.

Example:

If we select MySQL while creating an RDS database, AWS provisions a database using the MySQL engine.

The application connects to that engine using its database endpoint and appropriate credentials.

### AWS RDS Database Engines

| Database Engine | Description |
|---|---|
| MySQL | Open-source relational database |
| PostgreSQL | Advanced open-source relational database |
| MariaDB | MySQL-compatible relational database |
| Oracle | Commercial enterprise relational database |
| Microsoft SQL Server | Microsoft relational database |
| IBM Db2 | Enterprise relational database |
| Amazon Aurora MySQL | AWS-managed MySQL-compatible engine |
| Amazon Aurora PostgreSQL | AWS-managed PostgreSQL-compatible engine |

**Interview Point:** Database engine selection depends on application compatibility, licensing, performance, migration requirements and operational requirements.

---

## 4. Why AWS RDS? Why Not Install a Database on EC2?

We can install MySQL or PostgreSQL directly on an EC2 instance. However, we must manage the complete database infrastructure.

### Database on EC2 – Self Managed

```text
AWS EC2
   |
Linux Operating System
   |
Install MySQL
   |
Configure Database
   |
Manage Backup
   |
Manage Patching
   |
Manage Monitoring
   |
Manage Failover
```

Responsibilities:
- Launch EC2.
- Install Linux.
- Install database engine.
- Configure database.
- Manage OS updates.
- Configure backups.
- Manage database security.
- Configure high availability.
- Monitor database performance.

### Amazon RDS – Managed Database

```text
AWS RDS
   |
AWS Managed Infrastructure
   |
Database Engine
   |
Automated Backups
   |
Monitoring
   |
Managed Maintenance
   |
High Availability Options
```

AWS manages the underlying infrastructure and supported database operations.

### RDS vs EC2 Database

| Feature | Amazon RDS | Database on EC2 |
|---|---|---|
| OS Management | AWS | User |
| Database Installation | AWS | User |
| OS Patching | AWS | User |
| Database Backups | Built-in features | User configures |
| High Availability | Managed options | User configures |
| Monitoring | CloudWatch integration | User configures |
| SSH Access | No underlying host access | Yes |
| Infrastructure Control | Limited | Full |
| Operational Effort | Lower | Higher |

**Why choose RDS?**

- Reduced operational overhead.
- Automated backup capabilities.
- Managed maintenance.
- High availability options.
- Easy integration with AWS services.
- Simplified database scaling.

---

## 5. Is AWS RDS a PaaS or IaaS?

AWS RDS is generally classified as **Database as a Service (DBaaS)** and can be understood as a PaaS-like managed database offering.

### Cloud Service Models

| Service Model | Responsibility | AWS Example |
|---|---|---|
| IaaS | User manages OS and applications | EC2 |
| PaaS | Provider manages underlying platform | Elastic Beanstalk |
| SaaS | Complete software delivered as a service | Amazon WorkMail |
| DBaaS | Managed database service | Amazon RDS |

### Why RDS is PaaS-like?

In EC2, we manage the operating system and install the database ourselves.

In RDS, AWS manages the underlying infrastructure, OS and database engine maintenance.

We manage:
- Database users.
- Database schemas.
- Tables and records.
- Queries.
- Database permissions.
- Application connectivity.
- Database-level configuration.

**Important:** RDS is more precisely described as a managed database service (DBaaS), rather than a generic PaaS.

---

## 6. What are the Different RDS Environments?

In real-time DevOps projects, we generally maintain different environments.

### A. Development Environment (Dev)

Purpose:
- Application development.
- Developer testing.
- Database schema changes.
- Feature development.

Typical configuration:
- Smaller database instance.
- Lower storage capacity.
- Minimal backup retention where appropriate.
- No unnecessary replicas.
- Restricted database access.

Example:

```text
Environment: DEV
Database: dev-mysql
Instance: Small supported instance
Storage: 20 GB
Availability: Single-AZ where appropriate
```

### B. Testing Environment (Test/QA)

Purpose:
- Application testing.
- Integration testing.
- Performance testing.
- Bug verification.
- Release validation.

Typical configuration:
- Similar database engine and version to production.
- Test datasets.
- Controlled database access.
- Backup and recovery testing.
- Performance testing resources where needed.

Example:

```text
Environment: TEST
Database: test-mysql
Instance: Appropriate test instance
Storage: Based on test data
Availability: Based on testing requirements
```

### C. Production Environment (PROD)

Purpose:
- Live application.
- Real customer data.
- Business-critical transactions.

Typical configuration:
- Appropriate database instance size.
- Multi-AZ or Aurora high availability.
- Automated backups.
- Encryption.
- Monitoring and alerting.
- Restricted access.
- Disaster recovery strategy.

Example:

```text
Environment: PROD
Database: prod-mysql
Instance: Production-sized instance
Storage: Based on workload
Availability: High availability
Backup: Business recovery requirements
```

### Environment Comparison

| Feature | Dev | Test | Production |
|---|---|---|---|
| Instance Size | Small | Medium or workload-based | Workload-based |
| Data | Development data | Test data | Real business data |
| Backup | Minimal appropriate retention | Testing requirements | Business recovery policy |
| Availability | May be Single-AZ | Depends on testing | HA recommended |
| Security | Restricted | Restricted | Strict |
| Monitoring | Basic | Detailed | Comprehensive |
| Cost | Low | Moderate | Business justified |

**Important:** Dev/Test/Production are logical environment choices. They are not separate RDS database engines. Organizations may use separate AWS accounts, VPCs or database clusters for isolation.

---

## 7. How to Connect an RDS Database to Dev, Test and Production Environments

Consider an application running on EC2.

```text
                  AWS CLOUD
                      |
              Application Environment
                      |
          ---------------------------
          |            |            |
         DEV          TEST         PROD
          |            |            |
       EC2 DEV      EC2 TEST     EC2 PROD
          |            |            |
          |            |            |
       RDS DEV      RDS TEST     RDS PROD
       Database     Database     Database
```

Each environment should use the appropriate database endpoint.

### Step 1: Create Separate Databases

Example:

```text
dev-database
test-database
prod-database
```

### Step 2: Create Application Servers

Launch EC2 instances in their respective environments.

### Step 3: Configure Security Groups

Allow the application security group to connect to the corresponding database security group.

Example:

| Source | Destination | Port |
|---|---|---|
| DEV EC2-SG | DEV RDS-SG | 3306 |
| TEST EC2-SG | TEST RDS-SG | 3306 |
| PROD EC2-SG | PROD RDS-SG | 3306 |

### Step 4: Configure Database Endpoint

For development:

```text
DEV_DB_HOST=dev-db.xxxx.ap-south-1.rds.amazonaws.com
```

For testing:

```text
TEST_DB_HOST=test-db.xxxx.ap-south-1.rds.amazonaws.com
```

For production:

```text
PROD_DB_HOST=prod-db.xxxx.ap-south-1.rds.amazonaws.com
```

### Step 5: Configure Application Credentials

Use AWS Secrets Manager or another approved secrets management system.

Do not hardcode database credentials in application code or GitHub repositories.

### Step 6: Connect Using MySQL Client

```bash
mysql -h <RDS-ENDPOINT> -P 3306 -u <USERNAME> -p
```

Enter the password when prompted.

**DevOps Best Practice:** Keep Dev, Test and Production database credentials, access policies and data appropriately isolated.

---

## 8. What is a DB Parameter Group?

A DB parameter group is a collection of database engine configuration settings.

It controls how the database engine behaves.

Think of it as the configuration file for database engine settings, managed through AWS.

### Examples of Database Parameters

| Parameter | Purpose |
|---|---|
| max_connections | Maximum permitted database connections |
| slow_query_log | Enables slow query logging where supported |
| long_query_time | Defines the slow-query threshold where supported |
| time_zone | Database session/server time zone behavior |
| character_set_server | Server character set where supported |
| autocommit | Controls default transaction commit behavior |

### Why Do We Need Parameter Groups?

Different applications have different database requirements.

For example:
- An application may need more concurrent connections.
- Another application may require slow-query logging.
- An application may require specific character encoding.
- A production application may require tuning for workload performance.

### Types of Parameter Groups

**1. DB Parameter Group**

Controls database instance-level parameters.

**2. DB Cluster Parameter Group**

For Aurora, controls parameters that apply at the cluster level.

### Default vs Custom Parameter Group

| Default Parameter Group | Custom Parameter Group |
|---|---|
| AWS-provided defaults | User-created configuration |
| Limited customization | Customizable parameters |
| Suitable for default settings | Suitable for application-specific tuning |

Example:

```text
default.mysql8.0
```

Aurora example:

```text
default.aurora-mysql8.0
```

**Important:** Parameter group availability and supported parameters depend on the selected engine and version. Some changes apply immediately, while others require a reboot.

---

## 9. What is a DB Subnet Group?

A DB subnet group is a collection of subnets inside a VPC that RDS can use for database network placement.

It tells AWS where the database can be deployed within the selected VPC.

### Why Do We Need a DB Subnet Group?

- Defines database network placement.
- Provides subnet options across Availability Zones.
- Supports Multi-AZ deployments.
- Helps AWS manage database network interfaces.

### Example VPC Architecture

```text
VPC: 10.0.0.0/16
       |
       |
  ----------------------------
  |                          |
 AZ - 2a                   AZ - 2b
  |                          |
Private Subnet             Private Subnet
10.0.1.0/24                10.0.2.0/24
  |                          |
  ----------------------------
             |
        DB Subnet Group
             |
          Amazon RDS
```

### Example DB Subnet Group

```text
Name: rds-db-subnet-group

VPC: rds-vpc

Subnets:
1. rds-db-private-sub-01
2. rds-db-private-sub-02

Availability Zones:
1. ap-south-2a
2. ap-south-2b
```

### Why Private Subnets?

Databases contain sensitive application information.

We generally deploy production databases in private subnets to prevent direct internet access.

Example:

```text
Internet
   |
Internet Gateway
   |
Public Subnet
   |
Application Load Balancer
   |
Private EC2
   |
Private RDS
```

The application communicates with RDS through private IP connectivity within the VPC.

**Important:** A subnet group itself does not make a subnet private. The subnet's route table and public accessibility settings determine internet reachability.

---

## 10. What are the Specifications of AWS RDS?

RDS specifications depend on the selected database engine, engine version, instance class, storage type, AWS region and deployment configuration.

### A. Compute

Database compute resources provide CPU and memory.

Example:

| Instance Class | vCPU | RAM |
|---|---:|---:|
| db.t3.micro | 2 | 1 GiB |
| db.t3.small | 2 | 2 GiB |
| db.t3.medium | 2 | 4 GiB |
| db.m6g.large | 2 | 8 GiB |
| db.r6g.large | 2 | 16 GiB |

Availability depends on the selected engine and AWS region.

### B. Storage

Storage is used to store:
- Database tables.
- Indexes.
- Transaction logs.
- Other database files.

Common RDS storage types include:

| Storage Type | Description |
|---|---|
| General Purpose SSD gp2 | General-purpose SSD storage |
| General Purpose SSD gp3 | General-purpose SSD with configurable performance |
| Provisioned IOPS SSD io1/io2 | High-performance storage for demanding workloads |
| Magnetic | Legacy storage option where still supported |

### C. Storage Autoscaling

RDS can automatically increase allocated storage when storage autoscaling is enabled and configured.

It helps prevent the database from running out of storage.

### D. Database Size

Storage limits depend on the database engine, storage type and deployment configuration.

Do not assume every RDS engine has the same maximum storage capacity.

### E. Availability

RDS supports:
- Single-AZ deployments.
- Multi-AZ deployments.
- Read replicas for supported engines.
- Automated backups.
- Manual snapshots.

### F. Security

RDS security features include:
- VPC isolation.
- Security Groups.
- Encryption at rest using AWS KMS.
- Encryption in transit using TLS.
- IAM database authentication for supported engines.
- Secrets Manager integration.

### G. Monitoring

RDS integrates with Amazon CloudWatch.

Common monitoring metrics:

| Metric | Purpose |
|---|---|
| CPUUtilization | CPU usage |
| FreeableMemory | Available memory |
| FreeStorageSpace | Remaining storage |
| DatabaseConnections | Active database connections |
| ReadIOPS | Read operations per second |
| WriteIOPS | Write operations per second |
| ReadLatency | Read latency |
| WriteLatency | Write latency |

---

## 11. What is an RDS Endpoint?

An endpoint is a DNS address used by an application or client to connect to an RDS database.

Instead of connecting directly using a database server IP address, we use the endpoint.

Example:

```text
database-1.abcdefghij.ap-south-1.rds.amazonaws.com
```

Connection example:

```bash
mysql -h database-1.abcdefghij.ap-south-1.rds.amazonaws.com \
-P 3306 -u admin -p
```

### Types of Aurora Endpoints

**1. Cluster Writer Endpoint**

Used for read and write operations.

Example:

```text
mydb.cluster-xxxx.ap-south-1.rds.amazonaws.com
```

Routes connections to the current writer instance.

**2. Reader Endpoint**

Used for read-only workloads.

Example:

```text
mydb.cluster-ro-xxxx.ap-south-1.rds.amazonaws.com
```

Distributes new connections across available Aurora reader instances.

**3. Instance Endpoint**

Each database instance has its own endpoint.

It connects directly to a specific database instance.

**4. Custom Endpoint**

Allows us to define a group of Aurora instances for specific workloads.

### Standard RDS Endpoint

For a standard RDS DB instance, the endpoint normally identifies that DB instance. A Multi-AZ failover updates the DNS resolution to point to the new primary.

---

## 12. What Happens During RDS Multi-AZ Failover?

Multi-AZ provides high availability.

In a standard Multi-AZ DB instance deployment:

```text
             RDS Multi-AZ
                  |
          ------------------
          |                |
        AZ 2a            AZ 2b
          |                |
      Primary DB       Standby DB
          |                |
          ------------------
                  |
             DB Endpoint
```

When the primary database becomes unavailable:

1. AWS detects the failure.
2. AWS initiates failover.
3. The standby database becomes primary.
4. The DB endpoint DNS resolution is updated.
5. The application reconnects using the same endpoint.

### Aurora Multi-AZ Architecture

```text
          Aurora DB Cluster
                 |
        ---------------------
        |                   |
      AZ 2a               AZ 2b
        |                   |
     Writer              Reader
        |                   |
        ---------------------
                 |
           Shared Storage
```

When the Aurora writer fails:

1. Aurora identifies an eligible reader.
2. The reader is promoted to writer.
3. The cluster Writer Endpoint redirects to the new writer.
4. Applications reconnect using the same cluster endpoint.

**Important:** A standard RDS Multi-AZ standby and an Aurora reader instance are not the same architecture. Aurora readers can serve read traffic and can be promoted during failover.

---

## 13. Difference Between Parameter Group and Subnet Group

| Parameter Group | Subnet Group |
|---|---|
| Controls database configuration | Controls network placement |
| Defines engine parameters | Defines available VPC subnets |
| Used for database tuning | Used for networking |
| Example: max_connections | Example: private subnets in two AZs |
| Cluster and instance parameter groups available | DB subnet group |

Easy way to remember:

**Parameter Group = How the database behaves.**

**Subnet Group = Where the database can be placed.**

---

## 14. RDS vs Aurora

| Feature | Amazon RDS | Amazon Aurora |
|---|---|---|
| Database Service | Managed relational database | AWS-managed relational database engine |
| Engines | MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Db2 | MySQL-compatible and PostgreSQL-compatible |
| Storage | Engine-dependent storage architecture | Distributed cluster storage |
| High Availability | Multi-AZ options | Aurora cluster architecture |
| Read Scaling | Read replicas where supported | Aurora reader instances |
| Failover | Engine/deployment dependent | Aurora cluster failover |
| Endpoints | DB instance endpoint and other supported endpoints | Cluster, reader, instance and custom endpoints |

---

## 15. DevOps Responsibilities in AWS RDS

As a DevOps Engineer, our responsibilities include:

1. Provisioning RDS using Terraform or CloudFormation.
2. Creating VPC and DB subnet groups.
3. Configuring database security groups.
4. Managing database credentials.
5. Configuring backup and recovery.
6. Monitoring database performance.
7. Configuring Multi-AZ availability.
8. Testing database failover.
9. Managing database parameter groups.
10. Managing database migrations.
11. Connecting applications to databases.
12. Optimizing database costs.
13. Configuring disaster recovery.
14. Managing Dev, Test and Production environments.

---

## 16. Important DevOps Interview Questions

1. What is Amazon RDS?
2. What is a relational database?
3. What is the difference between SQL and NoSQL?
4. What is a database engine?
5. Which database engines does RDS support?
6. Why do we use AWS RDS instead of installing MySQL on EC2?
7. Is RDS IaaS or PaaS?
8. What is the difference between RDS and Aurora?
9. What is a DB subnet group?
10. Why do we need private subnets for RDS?
11. What is a DB parameter group?
12. What is the difference between a parameter group and subnet group?
13. What is an RDS endpoint?
14. What is the difference between a Writer Endpoint and Reader Endpoint?
15. What happens during Multi-AZ failover?
16. What is the difference between a standby database and a read replica?
17. How do you connect EC2 to a private RDS database?
18. How do you manage database credentials securely?
19. How do you monitor RDS using CloudWatch?
20. How do you take a manual RDS snapshot?
21. What is point-in-time recovery?
22. How do you reduce RDS costs?
23. How do you separate Dev, Test and Production databases?
24. What happens if an RDS database runs out of storage?
25. How do you troubleshoot a database connection timeout?

---

## 17. Final Summary

| Topic | Key Learning |
|---|---|
| RDS | Managed relational database service |
| Database Engine | Software that manages database operations |
| SQL | Relational database query language |
| NoSQL | Non-relational database models |
| PaaS / DBaaS | Managed database service |
| Dev Environment | Development and feature testing |
| Test Environment | Integration and QA testing |
| Production | Live application database |
| Parameter Group | Database engine configuration |
| Subnet Group | Database network placement |
| Instance Class | CPU and RAM specification |
| Storage | Database data storage |
| Endpoint | DNS address to connect to database |
| Multi-AZ | High availability |
| Reader | Read-only database workload |
| Writer | Read and write database workload |
| CloudWatch | Database monitoring |

**Final DevOps Takeaway:**

AWS RDS reduces the operational effort of managing databases. A DevOps Engineer must understand database engines, instance sizing, storage, networking, subnet groups, parameter groups, endpoints, security, backup, monitoring, automation and high availability to manage RDS effectively in production.