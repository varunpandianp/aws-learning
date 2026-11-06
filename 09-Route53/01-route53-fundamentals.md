# AWS Route 53 – Complete Notes

## 1. What is Amazon Route 53?

Amazon Route 53 is a highly available, scalable DNS (Domain Name System) web service provided by AWS.

It translates human-readable domain names into IP addresses so users can access websites and applications.

**Example:**
```text
www.vpcloud.online → 16.144.68.70
```

**Interview Definition:**

Amazon Route 53 is an AWS managed DNS service used for domain registration, DNS resolution, traffic routing, and health checking.

## 2. Why Do We Need Route 53?

Imagine deploying an application on an EC2 instance.

**Without DNS:**
```text
User → http://16.144.68.70
```

Problems:
- Users must remember IP addresses.
- Difficult to manage multiple servers.
- Public IP changes require updating the destination.
- Difficult to manage multi-region applications.

**With Route 53:**
```text
User
  |
  v
www.vpcloud.online
  |
  v
Route 53 (DNS)
  |
  v
EC2 / ALB / CloudFront
  |
  v
Application
```

**Benefits:**
1. Easy access using domain names.
2. Centralized DNS management.
3. Global traffic routing.
4. Disaster recovery support.
5. High availability.
6. Health checks and failover.

## 3. What is DNS?

DNS stands for Domain Name System.

DNS converts domain names into IP addresses.

Example:
```text
www.google.com → IP address
```

### DNS Resolution Hierarchy

1. Browser checks DNS cache.
2. Operating system checks DNS cache.
3. Recursive DNS resolver receives the query.
4. Root DNS server directs it to the TLD server.
5. TLD server directs it to the authoritative name server.
6. Authoritative DNS server returns the requested DNS record.
7. Browser connects to the destination IP.

**DNS Resolution Flow:**
```text
User enters www.vpcloud.online
             |
             v
      Recursive Resolver
             |
             v
          Root DNS
             |
             v
        .online TLD
             |
             v
     Authoritative DNS
             |
             v
      Route 53 DNS Record
             |
             v
          EC2 IP
```

## 4. What is a Name Server (NS)?

NS stands for Name Server.

A Name Server identifies which DNS servers are authoritative for a domain.

In simple terms:

**NS tells the internet where to find the DNS records for a particular domain.**

### Example

1. Purchase `vpcloud.online` from GoDaddy.
2. Create a Public Hosted Zone in Route 53.
3. Route 53 automatically generates four name servers.

Example values:
```text
ns-123.awsdns-45.com
ns-456.awsdns-78.net
ns-789.awsdns-12.org
ns-321.awsdns-34.co.uk
```

These are illustrative values, not actual name servers.

### Why Configure Route 53 Name Servers in GoDaddy?

Initially:
```text
Domain purchased from GoDaddy
             |
             v
         GoDaddy DNS
```

After updating nameservers:
```text
Domain purchased from GoDaddy
             |
             v
       Route 53 NS
             |
             v
       Route 53 DNS Records
             |
             v
       EC2 / ALB / CloudFront
```

**Important:**
- Domain Registrar: Where you purchase your domain.
- DNS Provider: Where you manage DNS records.
- Hosting Provider: Where your application runs.

These three services can be different providers.

GoDaddy can remain your domain registrar while Route 53 manages DNS and EC2 hosts your application.

## 5. What are Hosted Zones?

A Hosted Zone is a container that stores DNS records for a domain.

There are two types:

### 1. Public Hosted Zone

Used for domains accessible through the public internet.

Example:
```text
vpcloud.online
```

Use cases:
- Public websites.
- Public APIs.
- Internet-facing applications.

### 2. Private Hosted Zone

Used for internal DNS resolution within associated VPCs.

Example:
```text
db.internal
```

Use cases:
- Internal applications.
- Private databases.
- Microservices communication.

## 6. DNS Record Types in Route 53

| Record | Purpose | Example |
|---|---|---|
| A | Domain → IPv4 | vpcloud.online → 16.144.68.70 |
| AAAA | Domain → IPv6 | Domain → IPv6 address |
| CNAME | Domain → another domain | www → vpcloud.online |
| NS | Authoritative name servers | AWS name servers |
| SOA | DNS zone authority information | Zone metadata |
| MX | Mail server routing | Email servers |
| TXT | Text verification | Domain verification |
| PTR | Reverse DNS lookup | IP → hostname |
| SRV | Service location | Host and port |

### What is Alias Record?

Alias is an AWS Route 53 feature that allows you to point DNS records to supported AWS resources.

Examples:
- Application Load Balancer (ALB)
- Network Load Balancer (NLB)
- CloudFront
- S3 website endpoints

| Target | Recommended Record |
|---|---|
| EC2 public IPv4 | A |
| ALB | A – Alias |
| CloudFront | A – Alias |
| Another domain | CNAME |
| IPv6 address | AAAA |

**Advantages of Alias:**
- Works with root domains.
- Supports AWS resources.
- Automatically follows certain AWS resource changes.
- No additional Route 53 query charge for Alias queries to supported AWS resources.

## 7. Route 53 Routing Policies

Routing policies determine how Route 53 responds to DNS queries.

| Routing Policy | Purpose | Real-world Scenario |
|---|---|---|
| Simple | Routes to one resource | Single EC2 |
| Weighted | Splits traffic by percentage | 80% old version, 20% new version |
| Latency | Routes based on lowest latency | Mumbai and Singapore regions |
| Failover | Primary to secondary | Disaster recovery |
| Geolocation | Routes based on user location | India users to Mumbai |
| Geoproximity | Routes based on geographic distance and bias | Regional traffic distribution |
| Multi-value answer | Returns multiple healthy records | Multiple application endpoints |

### 7.1 Simple Routing

Routes traffic to a single resource.

Example:
```text
vpcloud.online → EC2 Public IP
```

Use case: Hosting a simple website on one EC2 instance.

### 7.2 Weighted Routing

Distributes traffic based on configured weights.

Example:
```text
80% → Production EC2
20% → New Version EC2
```

Use cases:
- Canary deployments.
- Blue-Green deployments.
- Application testing.

### 7.3 Latency-Based Routing

Routes users to the AWS region that provides the lowest measured network latency.

Example:
```text
India Users → Mumbai Region
US Users    → Virginia Region
```

Use case: Global applications requiring low latency.

### 7.4 Failover Routing

Routes traffic to a secondary resource when the primary resource becomes unhealthy.

Example:
```text
             Route 53
          Failover Routing
                 |
          ----------------
          |              |
          v              v
       PRIMARY        SECONDARY
       Mumbai          Singapore
          |              |
          v              v
         ALB            ALB
          |              |
          v              v
         EC2            EC2
```

Use cases:
- Disaster recovery.
- Business continuity.
- High availability.

**Important:** Failover is not nearest-region routing. Latency-based routing is used for lowest-latency selection.

### 7.5 Geolocation Routing

Routes traffic based on the geographic location of the DNS query source.

Example:
```text
India Users → Mumbai
US Users    → Virginia
Europe      → Frankfurt
```

### 7.6 Geoproximity Routing

Routes traffic based on the geographic location of users and AWS resources.

You can use a bias value to expand or shrink the geographic area served by a resource.

### 7.7 Multi-Value Answer Routing

Returns multiple healthy records in response to DNS queries.

It can improve availability by returning multiple healthy endpoints.

Note: It is not a replacement for a load balancer.

## 8. What are Route 53 Health Checks?

Health checks monitor the availability of endpoints.

Example:
```text
Route 53 Health Check
        |
        v
https://vpcloud.online/health
        |
   ------------
   |          |
 Healthy    Unhealthy
   |          |
   v          v
  Pass       Fail
```

Health checks can monitor:
- Public IP addresses.
- Domain names.
- HTTP endpoints.
- HTTPS endpoints.
- Other health checks.

Use cases:
- Failover routing.
- Endpoint availability monitoring.
- Disaster recovery.

Note: Route 53 health checks do not directly measure EC2 CPU or memory utilization.

## 9. Alternatives to Route 53

Route 53 is not the only DNS provider.

| DNS Provider | Purpose |
|---|---|
| Cloudflare DNS | DNS management, CDN and security |
| GoDaddy DNS | Domain and DNS management |
| Namecheap DNS | Domain and DNS management |
| Google Cloud DNS | Managed DNS |
| Azure DNS | Managed DNS |
| Oracle Cloud DNS | Managed DNS |

### Can We Use GoDaddy DNS Instead of Route 53?

Yes.

Example:
```text
GoDaddy DNS
    |
    v
A Record
    |
    v
EC2 Public IP
    |
    v
Website
```

You don't need Route 53 just because your application runs on AWS.

However, Route 53 offers convenient AWS integration, health checks and advanced routing policies.

## 10. Why is Route 53 Important for DevOps Engineers?

In real-world DevOps and SRE environments, applications are distributed across multiple servers and regions.

Route 53 helps with:

1. Application deployment: Connect domain names to ALB endpoints.
2. Blue-Green deployment: Use weighted routing to distribute traffic between environments.
3. Canary deployment: Gradually send traffic to a new version using weighted records.
4. Disaster recovery: Redirect traffic to a secondary region.
5. High availability: Use health checks and failover routing.
6. Multi-region architecture: Route users based on latency or geographic location.
7. Microservices: Manage DNS names for internal services using private hosted zones.

## 11. Practical – VP Cloud Lab

Domain:
```text
vpcloud.online
```

Architecture:
```text
                 Internet
                    |
                    v
              vpcloud.online
                    |
                    v
                Route 53
                    |
                    v
              EC2 Public IP
                    |
                    v
             Apache / Nginx
                    |
                    v
              VP Cloud Website
```

### Hands-on Tasks

- [ ] Purchase a domain from GoDaddy.
- [ ] Create a Public Hosted Zone in Route 53.
- [ ] Copy Route 53 NS records.
- [ ] Update custom nameservers in GoDaddy.
- [ ] Create an A record pointing to EC2 public IPv4.
- [ ] Create a CNAME record for www.
- [ ] Verify DNS using nslookup and dig.
- [ ] Connect domain to an ALB.
- [ ] Configure HTTPS using ACM.
- [ ] Practice weighted routing.
- [ ] Practice latency-based routing.
- [ ] Practice failover routing.

## 12. Interview Questions – Beginner to Intermediate

**Q1. What is Route 53?**

Amazon Route 53 is an AWS managed DNS service used for domain registration, DNS resolution, traffic routing and health checking.

**Q2. What is the difference between DNS and Route 53?**

DNS is the naming and resolution system. Route 53 is AWS's managed implementation of DNS hosting and related capabilities.

**Q3. What is a Name Server?**

It identifies the authoritative DNS servers responsible for a domain.

**Q4. Why do we update nameservers in GoDaddy?**

To delegate DNS authority to Route 53 so that Route 53 can manage the domain's DNS records.

**Q5. Can we use GoDaddy DNS instead of Route 53?**

Yes. DNS hosting and application hosting are independent.

**Q6. What is the difference between Public and Private Hosted Zones?**

Public hosted zones serve public DNS queries; private hosted zones resolve DNS names within associated VPCs.

**Q7. How do you implement disaster recovery using Route 53?**

Configure primary and secondary endpoints with failover routing and appropriate health checks.

**Q8. Can Route 53 route traffic to multiple AWS regions?**

Yes, using latency, weighted, geolocation and failover routing policies.

**Q9. What happens when the EC2 public IP changes?**

A standard A record pointing to the old IP becomes stale. Update the record or use a more stable endpoint such as an ALB.

**Q10. Does Route 53 distribute application traffic like an ALB?**

No. Route 53 makes DNS-based routing decisions. ALB distributes application traffic among registered targets.

## 13. Important Commands for Practical Verification

```bash
# Check domain resolution
nslookup vpcloud.online

# Detailed DNS lookup
dig vpcloud.online

# Check authoritative nameservers
dig NS vpcloud.online

# Check A record
dig A vpcloud.online

# Query a particular DNS server
dig @8.8.8.8 vpcloud.online

# Check HTTP connectivity
curl -I http://vpcloud.online

# Check HTTPS connectivity
curl -I https://vpcloud.online
```

## 14. Important Concepts to Remember

| Component | Responsibility |
|---|---|
| Domain Registrar | Where you purchase your domain |
| DNS Provider | Where you manage DNS records |
| Name Server | Identifies authoritative DNS servers |
| Hosted Zone | Container for DNS records |
| A Record | Maps hostname to IPv4 |
| CNAME | Maps hostname to another hostname |
| Alias | Points to supported AWS resources |
| Routing Policy | Determines DNS response based on routing rules |
| Health Check | Monitors endpoint availability |
| ALB | Distributes application traffic |

### Final DevOps Takeaway

Don't just learn Route 53 record creation. Understand how DNS delegation, DNS resolution, ALB integration, health checks and multi-region disaster recovery work together.

These are important real-world DevOps and SRE skills.