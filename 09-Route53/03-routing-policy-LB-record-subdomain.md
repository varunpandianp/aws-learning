# AWS Route 53 – Subdomains, CNAME, Alias Records & Routing Policies

## 1. What is a Subdomain?

A subdomain is an additional domain created under the main domain.

**Example:** Main domain: `vpcloud.online`

| Subdomain | Purpose |
|---|---|
| www.vpcloud.online | Main website |
| dev.vpcloud.online | Development environment |
| test.vpcloud.online | Testing environment |
| api.vpcloud.online | Backend API |
| app.vpcloud.online | Application |

### Practical Example

Suppose we have two EC2 instances:

- EC2-1 hosts the development website.
- EC2-2 hosts the testing website.

We can create these DNS records:

- `dev.vpcloud.online` → EC2-1 public IP
- `test.vpcloud.online` → EC2-2 public IP

**Important:** We don't need to purchase a separate domain for every subdomain. We can manage subdomains through the existing Route 53 hosted zone.

---

## 2. What is a CNAME Record?

**CNAME = Canonical Name Record**

A CNAME record maps one domain name to another domain name.

It does not directly map a hostname to an IP address.

### Example

| Record name | Type | Value |
|---|---|---|
| www.vpcloud.online | CNAME | vpcloud.online |
| dev.vpcloud.online | CNAME | myapp.example.com |

### How CNAME Works

1. User enters `www.vpcloud.online`.
2. DNS checks the CNAME record.
3. CNAME points to `vpcloud.online`.
4. DNS resolves `vpcloud.online` using its A record.
5. The A record returns the EC2 public IPv4 address.
6. The browser connects to the EC2 instance.

### Important Points

- CNAME points to another DNS name, not an IPv4 address.
- A CNAME cannot normally coexist with other record types at the same name.
- A standard CNAME cannot be created at the zone apex (root domain), such as `vpcloud.online`.
- Use Route 53 Alias records for supported AWS resources when appropriate.

---

## 3. What is an Alias Record in Route 53?

An Alias record is an AWS Route 53-specific feature that points a domain to supported AWS resources or another record in the hosted zone.

### Supported AWS resources include:

- Application Load Balancer (ALB)
- Network Load Balancer (NLB)
- CloudFront distribution
- S3 website endpoint
- Other supported Route 53 records

### Example

| Record name | Type | Alias target |
|---|---|---|
| vpcloud.online | A – Alias | Application Load Balancer |
| app.vpcloud.online | A – Alias | Application Load Balancer |
| www.vpcloud.online | A – Alias | Application Load Balancer |

### CNAME vs Alias

| Feature | CNAME | Alias |
|---|---|---|
| Maps to | Another DNS name | Supported AWS resource or record |
| Root domain support | No | Yes |
| AWS-specific | No | Yes |
| Record type | CNAME | A or AAAA |
| Target | DNS hostname | AWS resource or supported record |
| Pricing | Standard DNS query pricing | Alias queries to many AWS resources are free; exceptions apply |

**Remember:** Alias is not a standard DNS record type. It is a Route 53 feature.

---

## 4. How to Connect a Load Balancer Using CNAME and Alias

Suppose we have an Application Load Balancer:

`myapp-alb-123456.ap-south-1.elb.amazonaws.com`

Our domain:

`vpcloud.online`

### Method 1: Using CNAME

Create the following record in Route 53:

| Setting | Value |
|---|---|
| Record name | app |
| Record type | CNAME |
| Value | ALB DNS name |
| Routing policy | Simple |

Result:

`app.vpcloud.online` → ALB DNS name → Target Group → EC2 instances

**Limitation:** A standard CNAME cannot be used for the root domain `vpcloud.online`.

### Method 2: Using Alias

Create the following record in Route 53:

| Setting | Value |
|---|---|
| Record name | Leave blank for root domain |
| Record type | A |
| Alias | Enable |
| Route traffic to | Application and Classic Load Balancer |
| Region | Select ALB region |
| Load Balancer | Select your ALB |
| Routing policy | Simple |

Result:

`vpcloud.online` → ALB → Target Group → EC2 instances

**For IPv6:** Create an AAAA Alias record if your application and load balancer support IPv6.

---

# 5. Route 53 Routing Policies

Routing policies determine how Route 53 responds to DNS queries when multiple destinations or records are configured.

There are eight commonly discussed routing policies.

## 5.1 Simple Routing

**Purpose:** Routes DNS queries to a single destination.

**Use case:** Hosting a basic website.

Example:

User → Route 53 → ALB → EC2

### Explanation

- Used for a single resource or destination.
- Can also return multiple configured values in a simple record.
- Does not provide weighted or geographic traffic distribution.
- Does not perform health-based destination selection.

## 5.2 Weighted Routing

**Purpose:** Distributes DNS responses according to configured weights.

**Use case:** Canary deployment or testing a new application version.

### Example

| Destination | Weight |
|---|---:|
| ALB-1 – Version 1 | 80 |
| ALB-2 – Version 2 | 20 |

Expected DNS response distribution:

- 80% to ALB-1
- 20% to ALB-2

### Explanation

- Assign weights to multiple records.
- Route 53 uses the configured weight ratios to select DNS responses.
- Useful for application testing and gradual rollouts.
- Actual application request distribution may vary because of DNS caching.

## 5.3 Latency-Based Routing

**Purpose:** Routes users toward the AWS region expected to provide the lowest latency.

**Use case:** Applications deployed in multiple AWS regions.

### Example

| User location | AWS region |
|---|---|
| India | Mumbai – ap-south-1 |
| USA | N. Virginia – us-east-1 |
| Singapore | Singapore – ap-southeast-1 |

### Explanation

- Route 53 uses AWS latency measurements.
- Selects the region expected to provide the lowest latency.
- Requires records for the relevant regional resources.
- The lowest-latency region is not necessarily the geographically nearest region.

## 5.4 Failover Routing

**Purpose:** Provides primary and secondary DNS destinations.

**Use case:** Disaster Recovery (DR).

### Example

| Record | Resource | Role |
|---|---|---|
| Primary | Mumbai ALB | Main application |
| Secondary | Singapore ALB | Backup application |

### How it works

1. User requests the application.
2. Route 53 checks the primary resource's health.
3. If the primary is healthy, Route 53 returns the primary destination.
4. If the primary becomes unhealthy, Route 53 can return the secondary destination.

### Explanation

- Configure primary and secondary records.
- Configure Route 53 health checks or supported resource health evaluation.
- Useful for disaster recovery.
- The secondary infrastructure should be ready to serve application traffic.

## 5.5 Geolocation Routing

**Purpose:** Routes DNS queries based on the geographic location of the user.

**Use case:** Delivering different content to different countries.

### Example

| User location | Destination |
|---|---|
| India | India website |
| USA | USA website |
| Europe | Europe website |

### Explanation

- Routes based on the user's geographic location.
- Supports continent, country and US state-level locations.
- Useful for localization, compliance and regional content.
- Configure a default record for unmatched locations where appropriate.

## 5.6 Geoproximity Routing

**Purpose:** Routes users based on the geographic location of users and resources, with adjustable bias.

**Use case:** Controlling the geographic area served by different AWS regions.

### Explanation

- Considers the location of users and resources.
- Uses positive or negative bias to expand or shrink a resource's routing area.
- Uses Route 53 Traffic Flow.
- Useful when you need more control over geographic traffic distribution.

**Difference from Geolocation:**

- Geolocation: Routes based on the user's geographic location.
- Geoproximity: Considers user and resource locations and supports bias adjustments.

## 5.7 IP-Based Routing

**Purpose:** Routes DNS queries based on defined client IP address ranges.

**Use case:** Enterprise networks and specific ISP or network routing requirements.

### Example

| Client IP range | Destination |
|---|---|
| 192.168.1.0/24 | Destination A |
| 10.0.0.0/8 | Destination B |

### Explanation

- Define CIDR blocks and associate them with locations.
- Route 53 uses the source IP address of the DNS query to select a destination.
- Useful for specific networks and enterprise users.

**Note:** These ranges must be configured as IP-based routing CIDR collections; private IP examples above are for illustration.

## 5.8 Multivalue Answer Routing

**Purpose:** Returns multiple healthy resource IP addresses in response to DNS queries.

**Use case:** Returning multiple healthy endpoints.

### Example

| Resource | IP address |
|---|---|
| EC2-1 | 10.0.1.10 |
| EC2-2 | 10.0.2.10 |
| EC2-3 | 10.0.3.10 |

### Explanation

- Route 53 can return multiple healthy records.
- Health checks can be associated with records.
- Helps improve DNS-level availability.
- It is not a replacement for an Application Load Balancer.

---

# 6. Quick Comparison – Routing Policies

| Routing policy | Main purpose | Example |
|---|---|---|
| Simple | Single destination | One website |
| Weighted | Weight-based distribution | 80% / 20% |
| Latency | Lowest measured latency | Mumbai / Virginia |
| Failover | Primary and secondary | Disaster recovery |
| Geolocation | User's geographic location | India / USA |
| Geoproximity | Proximity with bias | Expand Mumbai routing area |
| IP-based | Client IP range | Enterprise network |
| Multivalue answer | Multiple healthy answers | Multiple EC2 IPs |

---

# 7. Interview Questions and Answers

### Q1. Can we connect an ALB to Route 53 using CNAME?

Yes. A CNAME record can point a subdomain to the ALB DNS name.

Example:

`app.vpcloud.online` → `myapp-alb-123456.ap-south-1.elb.amazonaws.com`

### Q2. Can we use CNAME for the root domain?

No. A standard CNAME cannot be used at the zone apex. Use a Route 53 Alias record instead.

### Q3. What is the difference between Weighted and Latency routing?

- Weighted: Distributes DNS responses according to configured weights.
- Latency: Selects a region expected to provide the lowest latency.

### Q4. Which routing policy is useful for Disaster Recovery?

Failover routing with health checks and primary/secondary resources.

### Q5. Is Multivalue Answer routing a load balancer?

No. It returns multiple healthy DNS answers. An ALB actively distributes application traffic to healthy targets.

### Q6. What is the difference between Geolocation and Geoproximity?

- Geolocation: Routes based on the user's geographic location.
- Geoproximity: Considers the location of users and resources, with adjustable bias.

### Q7. Does Weighted routing guarantee exactly 80% of application requests go to one ALB?

No. DNS caching and resolver behavior can make the actual application request distribution different from the configured DNS weights.

### Q8. What is the difference between CNAME and Alias?

- CNAME points one DNS name to another DNS name.
- Alias points to supported AWS resources or supported Route 53 records.
- Alias supports the root domain, whereas a standard CNAME does not.

---

# 8. Practical Lab – My vpcloud.online Domain

### Objective

Connect a Route 53 domain to an Application Load Balancer.

### Prerequisites

- Domain: `vpcloud.online`
- Route 53 public hosted zone
- Two EC2 instances
- Application Load Balancer
- Target Group

### Practical Steps

1. Launch two EC2 instances.
2. Install Apache or Nginx on both instances.
3. Configure different sample web pages on each instance.
4. Create a Target Group.
5. Register both EC2 instances.
6. Verify that both targets are healthy.
7. Create an Application Load Balancer.
8. Configure the ALB listener and target group.
9. Copy or identify the ALB DNS name.
10. Open Route 53 → Hosted Zones → `vpcloud.online`.
11. Create an A record with Alias enabled.
12. Select the Application Load Balancer as the Alias target.
13. Save the record.
14. Open `vpcloud.online` in the browser.
15. Verify that the ALB forwards requests to healthy EC2 instances.

### Additional Practical

Create a CNAME record:

- Record name: `dev`
- Type: CNAME
- Value: Your ALB DNS name

Test:

`dev.vpcloud.online`

### Cleanup

After completing the lab:

- Delete unused ALBs.
- Delete unused Target Groups.
- Terminate unnecessary EC2 instances.
- Remove unused Route 53 records.

**Important:** ALBs and running EC2 instances can incur charges.

---

# 9. Important Interview Takeaways

- Route 53 is an AWS DNS service.
- CNAME maps one DNS name to another DNS name.
- Alias is a Route 53 feature used to point to supported AWS resources.
- Use Alias records for root-domain ALB integration.
- Weighted routing is useful for canary deployments.
- Latency routing selects a region based on expected latency.
- Failover routing supports disaster recovery.
- Geolocation routing uses the user's geographic location.
- Geoproximity routing uses user and resource locations with bias.
- IP-based routing uses configured client IP ranges.
- Multivalue answer routing returns multiple healthy DNS answers.
- Route 53 handles DNS resolution; ALB handles application traffic distribution.