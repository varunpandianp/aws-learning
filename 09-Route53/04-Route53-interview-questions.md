# AWS Route 53 – Interview Questions and Answers

## 1. Route 53 Fundamentals

### Q1. What is Amazon Route 53?
Amazon Route 53 is a highly available and scalable AWS DNS web service.

It provides:
- Domain registration
- DNS routing
- Health checking
- Traffic management
- Domain-to-IP resolution

### Q2. What is DNS?
DNS stands for Domain Name System.

It translates human-readable domain names into IP addresses.

Example:

`www.google.com` → IP address

### Q3. What are the main functions of Route 53?
1. Domain registration
2. DNS routing
3. Health checking

### Q4. What is a Hosted Zone?
A Hosted Zone is a container for DNS records belonging to a domain.

Types:
- Public Hosted Zone: Resolves domain names over the public internet.
- Private Hosted Zone: Resolves domain names within associated VPCs.

### Q5. What is the difference between a domain registrar and a DNS provider?
- Domain registrar: Registers and manages domain ownership.
- DNS provider: Hosts DNS records and answers DNS queries.

Example:
- GoDaddy: Domain registrar.
- Route 53: DNS provider.

### Q6. What are NS records?
NS stands for Name Server.

NS records identify the authoritative DNS servers for a domain.

When we create a Route 53 public hosted zone, AWS automatically creates four NS records.

We configure these nameservers at our domain registrar to delegate DNS authority to Route 53.

### Q7. What is an SOA record?
SOA stands for Start of Authority.

It contains administrative information about a DNS zone, such as the primary nameserver, responsible contact, serial number and timing parameters.

### Q8. What is the difference between A and AAAA records?
- A record: Maps a hostname to an IPv4 address.
- AAAA record: Maps a hostname to an IPv6 address.

### Q9. What is a TTL in DNS?
TTL stands for Time To Live.

It specifies how long a DNS resolver can cache a DNS response.

Example:

TTL = 300 seconds means the resolver can cache the response for approximately five minutes.

### Q10. What happens if you delete a DNS record?
The record is removed from the authoritative DNS zone, but recursive resolvers may continue serving previously cached answers until their TTL expires.

---

## 2. Subdomains, CNAME and Alias

### Q11. What is a subdomain?
A subdomain is an additional hostname created under a main domain.

Example:
- vpcloud.online
- dev.vpcloud.online
- test.vpcloud.online
- api.vpcloud.online

### Q12. What is a CNAME record?
CNAME stands for Canonical Name.

It maps one DNS name to another DNS name.

Example:

`www.vpcloud.online` → `vpcloud.online`

### Q13. Can we use CNAME for the root domain?
No. A standard CNAME cannot be created at the zone apex because the apex must contain required DNS records such as NS and SOA.

Use a Route 53 Alias record instead.

### Q14. What is an Alias record?
Alias is a Route 53-specific feature that points a DNS record to supported AWS resources or supported records in the hosted zone.

Examples:
- ALB
- NLB
- CloudFront
- S3 website endpoint

### Q15. What is the difference between CNAME and Alias?

| CNAME | Alias |
|---|---|
| Standard DNS record | Route 53-specific feature |
| Points to another DNS name | Points to supported AWS resources or records |
| Cannot be used at root domain | Can be used at root domain |
| Standard DNS query pricing | Many AWS-resource Alias queries are free |

### Q16. Can we point a subdomain to an ALB?
Yes.

We can use either:
- CNAME record pointing to the ALB DNS name.
- A Alias record pointing to the ALB.

### Q17. How do you connect a root domain to an ALB?
Create an A record in Route 53, enable Alias, and select the ALB as the target.

Example:

`vpcloud.online` → ALB → Target Group → EC2 instances.

### Q18. Can a CNAME point directly to an IP address?
No. A CNAME points to another DNS name. Use an A record for IPv4 or AAAA record for IPv6.

### Q19. Can we create multiple subdomains under one domain?
Yes. We can create multiple DNS records under the same hosted zone.

Example:
- dev.vpcloud.online
- test.vpcloud.online
- stage.vpcloud.online
- prod.vpcloud.online

---

## 3. Connecting Route 53 with Load Balancers

### Q20. How does Route 53 connect to an Application Load Balancer?
1. User requests a domain.
2. Route 53 resolves the DNS record.
3. Route 53 returns the ALB destination.
4. Browser connects to the ALB.
5. ALB forwards the request to a healthy target.
6. EC2 serves the application.

### Q21. Can we use an A record directly with an ALB DNS name?
A normal A record requires an IPv4 address. An ALB DNS name is not an IPv4 address.

Use an Alias A record instead.

### Q22. What is the difference between Route 53 and ALB?
- Route 53: DNS resolution and DNS-level traffic routing.
- ALB: Distributes HTTP/HTTPS application requests among registered targets.

### Q23. Can Route 53 distribute traffic between multiple ALBs?
Yes. Routing policies such as Weighted, Latency, Failover and Geolocation can direct DNS responses to different ALBs.

### Q24. What happens if an ALB target becomes unhealthy?
The ALB health-check mechanism marks that target unhealthy and stops routing normal traffic to it.

If other healthy targets are available, the ALB continues forwarding requests to them.

### Q25. Is Route 53 a load balancer?
No. Route 53 is a DNS service. It can provide DNS-level traffic distribution, but it does not replace an application load balancer.

---

## 4. Route 53 Routing Policies

### Q26. What are the different routing policies in Route 53?
1. Simple
2. Weighted
3. Latency-based
4. Failover
5. Geolocation
6. Geoproximity
7. IP-based
8. Multivalue answer

### Q27. What is Simple routing?
It routes DNS queries to a single destination or returns configured values for a simple record.

Use case: A basic website hosted on one ALB.

### Q28. What is Weighted routing?
It distributes DNS responses based on configured weights.

Example:
- ALB-1: Weight 80
- ALB-2: Weight 20

Use case: Canary deployments and gradual application releases.

### Q29. What is Latency-based routing?
It routes users to the AWS region expected to provide the lowest latency.

Example:
- India users → Mumbai region.
- US users → Virginia region.

The selection depends on AWS latency measurements, not simply geographic distance.

### Q30. What is Failover routing?
It provides primary and secondary DNS destinations.

When the primary resource becomes unhealthy, Route 53 can return the secondary destination.

Use case: Disaster Recovery.

### Q31. What is Geolocation routing?
It routes DNS queries based on the geographic location of the user.

Example:
- India → India website.
- USA → USA website.
- Europe → Europe website.

### Q32. What is Geoproximity routing?
It routes users based on the geographic location of users and resources.

It also supports positive or negative bias to expand or shrink the routing area.

### Q33. What is IP-based routing?
It routes DNS queries based on predefined client IP ranges.

Use case: Routing specific enterprise networks to particular application endpoints.

### Q34. What is Multivalue answer routing?
It returns multiple healthy resource records in response to DNS queries.

It can use health checks to exclude unhealthy endpoints.

It is not a replacement for a load balancer.

### Q35. Difference between Geolocation and Geoproximity?

| Geolocation | Geoproximity |
|---|---|
| Based on user's geographic location | Based on user and resource locations |
| Routes by continent, country or US state | Uses geographic proximity and bias |
| Useful for localization | Useful for adjusting geographic traffic distribution |

### Q36. Difference between Weighted and Latency routing?

| Weighted | Latency |
|---|---|
| Uses configured weights | Uses AWS latency measurements |
| Percentage-based DNS distribution | Lowest expected latency |
| Useful for canary releases | Useful for global applications |

---

## 5. Health Checks and Disaster Recovery

### Q37. What is a Route 53 Health Check?
A health check monitors the health of an endpoint or supported resource.

It can check:
- Endpoint availability
- HTTP/HTTPS responses
- TCP connectivity
- Calculated health based on other health checks
- CloudWatch alarm state through supported configurations

### Q38. What is the purpose of health checks in Failover routing?
Health checks help Route 53 determine whether the primary resource is healthy.

If the primary becomes unhealthy, Route 53 can return the secondary destination.

### Q39. Does Route 53 health checking replace ALB health checks?
No.

- ALB health checks monitor registered targets and determine where the ALB sends application traffic.
- Route 53 health checks monitor configured endpoints or resource health for DNS routing decisions.

### Q40. What happens if both primary and secondary resources are unhealthy?
Failover routing does not guarantee a healthy destination. Depending on record configuration and health evaluation, Route 53 may still return the primary record when all eligible records are unhealthy.

### Q41. Can Route 53 support multi-region disaster recovery?
Yes.

Example:
- Primary infrastructure: Mumbai.
- Secondary infrastructure: Singapore.
- Route 53 Failover routing.
- Health checks to monitor the primary endpoint.

The secondary application, data and infrastructure must also be prepared to handle production traffic.

---

# 6. Scenario-Based Interview Questions

### Q42. You have a domain in GoDaddy and want to manage DNS using Route 53. What steps will you follow?

**Answer:**

1. Purchase/register the domain with GoDaddy.
2. Create a Public Hosted Zone in Route 53.
3. Copy the four Route 53 NS records.
4. Update the custom nameservers in GoDaddy.
5. Wait for DNS delegation to propagate.
6. Create A, CNAME or Alias records as required.
7. Test DNS resolution.

### Q43. Your domain is working, but the website displays the GoDaddy parking page. How will you troubleshoot?

**Answer:**

1. Check the domain's authoritative NS records.
2. Verify that GoDaddy delegates to Route 53.
3. Check the A or Alias record in Route 53.
4. Verify the record points to the correct EC2 or ALB destination.
5. Check DNS resolution using public resolvers.
6. Check browser and DNS caching.
7. Verify EC2 security group, web server and application health.

Useful commands:

```bash
dig NS vpcloud.online
dig A vpcloud.online
dig @8.8.8.8 vpcloud.online
dig +trace vpcloud.online
```

### Q44. You have two application versions and want to send 90% traffic to the old version and 10% to the new version. Which routing policy will you use?

**Answer:** Weighted routing.

Configure:
- Old version: Weight 90.
- New version: Weight 10.

### Q45. Your company has applications deployed in Mumbai and Virginia. Users should reach the region with the lowest latency. Which policy will you use?

**Answer:** Latency-based routing.

### Q46. Your production application runs in Mumbai. If the Mumbai application becomes unavailable, traffic should go to Singapore. Which policy will you use?

**Answer:** Failover routing with primary and secondary records and suitable health checks.

### Q47. Your company wants Indian users to see Indian content and US users to see US content. Which policy will you use?

**Answer:** Geolocation routing.

### Q48. You have three EC2 instances with public IPs. You want Route 53 to return multiple healthy IP addresses. Which policy will you use?

**Answer:** Multivalue answer routing.

### Q49. You want users from a particular corporate IP range to reach a dedicated application. Which policy will you use?

**Answer:** IP-based routing.

### Q50. Your company wants to adjust the geographic traffic area served by an AWS region using positive or negative bias. Which policy will you use?

**Answer:** Geoproximity routing.

### Q51. You deleted an A record, but the website is still accessible. Why?

**Answer:**

Possible reasons:
- DNS resolver caching.
- Browser or operating-system DNS cache.
- Another A or AAAA record exists.
- Wildcard record exists.
- Existing network connections are still active.
- DNS queries are resolving from another authoritative zone.

Check the authoritative DNS server directly.

### Q52. You configured an Alias record to an ALB, but the website is not opening. What will you check?

**Answer:**

1. Verify Alias target and AWS region.
2. Check ALB state.
3. Check listener configuration (HTTP/HTTPS).
4. Check target group health.
5. Check EC2 security group allows traffic from the ALB security group.
6. Verify application is listening on the correct port.
7. Check SSL certificate if using HTTPS.
8. Verify DNS resolution.

### Q53. Can Route 53 automatically move application traffic to another AWS region when an EC2 instance fails?

**Answer:**

Not automatically just because an EC2 instance fails.

A suitable architecture needs:
- A secondary region.
- Healthy application infrastructure in that region.
- Route 53 health checks or supported resource health evaluation.
- Failover records.
- Replicated or recoverable application data.

---

# 7. Important Commands for Practical Troubleshooting

```bash
# Check NS records
dig NS vpcloud.online

# Check A record
dig A vpcloud.online

# Check CNAME
dig CNAME www.vpcloud.online

# Query Google DNS
dig @8.8.8.8 vpcloud.online

# Query Cloudflare DNS
dig @1.1.1.1 vpcloud.online

# Trace DNS resolution
dig +trace vpcloud.online

# Check using nslookup
nslookup vpcloud.online

# Check HTTP response
curl -I http://vpcloud.online

# Check HTTPS response
curl -I https://vpcloud.online
```

---

# 8. Final Interview Revision

| Topic | Key point to remember |
|---|---|
| Route 53 | AWS DNS service |
| Hosted Zone | Container for DNS records |
| Public Hosted Zone | Public DNS resolution |
| Private Hosted Zone | DNS resolution within associated VPCs |
| NS | Authoritative nameservers |
| SOA | DNS zone administrative information |
| A | IPv4 mapping |
| AAAA | IPv6 mapping |
| CNAME | DNS name to DNS name |
| Alias | Route 53 feature for supported AWS resources and records |
| TTL | DNS caching duration |
| Simple | Single destination / configured values |
| Weighted | Weight-based DNS response distribution |
| Latency | Lowest expected latency |
| Failover | Primary and secondary |
| Geolocation | User geographic location |
| Geoproximity | Geographic proximity and bias |
| IP-based | Client IP range |
| Multivalue | Multiple healthy DNS answers |

## Interview Golden Rule

**Route 53 decides where DNS queries should resolve. ALB decides which healthy application target receives the request. EC2 runs the application.**

Example architecture:

`User → Route 53 → ALB → Target Group → EC2 → Application`