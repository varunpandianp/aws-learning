# AWS Elastic Load Balancing (ELB) – Interview Questions and Answers

## Interview Preparation – Beginner to Intermediate + Scenario Based

Topics Covered:
1. Load Balancer Fundamentals
2. Types of AWS Load Balancers
3. Classic Load Balancer (CLB)
4. Application Load Balancer (ALB)
5. Network Load Balancer (NLB)
6. Gateway Load Balancer (GWLB)
7. Target Groups
8. Listeners and Listener Rules
9. Path-Based and Host-Based Routing
10. Health Checks
11. Cross-Zone Load Balancing
12. Connection Draining
13. Auto Scaling and Load Balancer Integration
14. Security Groups and Networking
15. Terraform
16. Real-World Scenario-Based Questions

---

# SECTION 1: Load Balancer Fundamentals – Beginner Level

### Q1. What is a Load Balancer in AWS?

**Answer:**

A Load Balancer is an AWS service that distributes incoming application or network traffic across multiple backend resources such as EC2 instances.

The main purpose is to:
- Distribute incoming traffic.
- Prevent a single EC2 instance from becoming overloaded.
- Improve application availability.
- Perform health checks.
- Route traffic to healthy instances.
- Support high availability across multiple Availability Zones.

Example:

```text
              Users
                |
                v
          Load Balancer
                |
        -----------------
        |       |       |
        v       v       v
       EC2-1   EC2-2   EC2-3
```

### Q2. Why do we need a Load Balancer?

**Answer:**

Imagine an application running on a single EC2 instance.

If 10,000 users access the application simultaneously, the instance may become overloaded.

Instead of handling all requests on one instance, we distribute traffic across multiple EC2 instances using a Load Balancer.

Benefits:
- High availability.
- Scalability.
- Better traffic distribution.
- Health monitoring.
- Improved application reliability.

### Q3. What is ELB in AWS?

**Answer:**

ELB stands for Elastic Load Balancing.

It is the AWS service that automatically distributes incoming traffic across multiple targets.

AWS provides four main types:

1. Classic Load Balancer (CLB).
2. Application Load Balancer (ALB).
3. Network Load Balancer (NLB).
4. Gateway Load Balancer (GWLB).

### Q4. What are the four types of Load Balancers in AWS?

**Answer:**

| Load Balancer | OSI Layer | Purpose |
|---|---|---|
| CLB | Layer 4 and 7 | Legacy load balancing |
| ALB | Layer 7 | HTTP/HTTPS application routing |
| NLB | Layer 4 | TCP, UDP and TLS traffic |
| GWLB | Layer 3 | Network traffic inspection |

### Q5. What is the difference between Layer 4 and Layer 7 Load Balancing?

**Answer:**

Layer 4 works at the Transport Layer.

It makes routing decisions using information such as:
- Source IP.
- Destination IP.
- Port number.
- Transport protocol.

Layer 7 works at the Application Layer.

It can inspect:
- HTTP URL paths.
- Hostnames.
- HTTP headers.
- Query strings.

Example:

Layer 4:
```text
TCP Port 80 → Backend 1
TCP Port 443 → Backend 2
```

Layer 7:
```text
/api/*   → API Target Group
/admin/* → Admin Target Group
```

### Q6. What is High Availability in Load Balancing?

**Answer:**

High availability means keeping an application accessible even when an instance or Availability Zone experiences a failure.

We deploy EC2 instances across multiple Availability Zones and use a Load Balancer to distribute incoming traffic.

If one instance becomes unhealthy, the Load Balancer stops sending new requests to that instance.

### Q7. What is the difference between Scalability and High Availability?

**Answer:**

Scalability means handling increasing or decreasing workloads by adjusting resources.

High Availability means maintaining application availability even when infrastructure components fail.

Load Balancer and Auto Scaling together help achieve both.

---

# SECTION 2: Load Balancer Types – Beginner to Intermediate

### Q8. What is an Application Load Balancer?

**Answer:**

ALB operates at Layer 7 of the OSI model.

It supports HTTP and HTTPS traffic and provides advanced routing capabilities.

Features:
- Path-based routing.
- Host-based routing.
- Header-based routing.
- Query-string routing.
- WebSocket support.
- Target Group integration.
- Auto Scaling integration.

Use cases:
- Web applications.
- REST APIs.
- Microservices.
- E-commerce applications.

### Q9. What is a Network Load Balancer?

**Answer:**

NLB operates at Layer 4.

It distributes connections based on network protocols and ports.

Features:
- TCP support.
- UDP support.
- TLS listeners.
- Static IP addresses.
- High performance.
- Low latency.

Use cases:
- Gaming applications.
- Real-time communication.
- TCP/UDP applications.
- Applications requiring static IP addresses.

### Q10. What is Classic Load Balancer?

**Answer:**

Classic Load Balancer is the legacy generation of AWS Load Balancing.

It supports basic Layer 4 and Layer 7 load balancing.

Unlike ALB and NLB, CLB directly registers EC2 instances instead of using Target Groups.

It supports:
- HTTP/HTTPS.
- TCP/SSL.
- Health checks.
- Cross-Zone Load Balancing.
- Connection Draining.

It is primarily used for maintaining legacy applications.

### Q11. What is Gateway Load Balancer?

**Answer:**

GWLB operates at Layer 3.

It distributes network traffic across third-party virtual security appliances.

Examples:
- Firewalls.
- Intrusion Detection Systems.
- Intrusion Prevention Systems.
- Deep Packet Inspection appliances.

GWLB uses GENEVE encapsulation over UDP port 6081.

### Q12. Difference between ALB and NLB?

**Answer:**

| Feature | ALB | NLB |
|---|---|---|
| OSI Layer | Layer 7 | Layer 4 |
| Protocol | HTTP/HTTPS | TCP/UDP/TLS |
| Path Routing | Yes | No |
| Host Routing | Yes | No |
| Static IP | No fixed static IP | Yes |
| Target Groups | Yes | Yes |
| Main Use | Web applications | Network applications |

### Q13. Which Load Balancer would you choose for a gaming application?

**Answer:**

I would choose Network Load Balancer because gaming applications often use TCP or UDP protocols and require low latency and high-performance network connections.

### Q14. Which Load Balancer would you choose for microservices?

**Answer:**

I would choose Application Load Balancer because it supports path-based and host-based routing.

For example:

```text
/api/*    → API Service
/admin/*  → Admin Service
/payment/* → Payment Service
```

One ALB can route requests to multiple Target Groups.

### Q15. Does NLB support path-based routing?

**Answer:**

No. NLB operates at Layer 4 and does not inspect HTTP URL paths for routing decisions.

It routes connections using listener protocols and ports.

---

# SECTION 3: Target Groups – Beginner to Intermediate

### Q16. What is a Target Group?

**Answer:**

A Target Group is a collection of backend resources to which a Load Balancer forwards incoming traffic.

It can contain:
- EC2 instances.
- IP addresses.
- Lambda functions (ALB).
- Other supported target types depending on the Load Balancer.

Target Groups also perform health checks on registered targets.

### Q17. Why do we need Target Groups?

**Answer:**

Target Groups act as the backend destination for ALB and NLB.

For example, an e-commerce application may have three services:

```text
             Application LB
                    |
          ---------------------
          |         |         |
          v         v         v
        Main TG   API TG   Admin TG
          |         |         |
          v         v         v
        EC2s      EC2s      EC2s
```

Each Target Group contains the backend instances for a particular application.

### Q18. Can one ALB have multiple Target Groups?

**Answer:**

Yes.

One ALB can have multiple Target Groups.

We can use listener rules to forward incoming requests to different Target Groups.

### Q19. Can one Target Group contain multiple EC2 instances?

**Answer:**

Yes.

A Target Group can contain multiple EC2 instances.

The Load Balancer distributes traffic across healthy registered targets.

### Q20. Can one EC2 instance belong to multiple Target Groups?

**Answer:**

Yes.

The same EC2 instance can be registered with multiple Target Groups.

For example, the same backend instance can serve different applications or ports, provided the application configuration supports it.

### Q21. What happens when all instances in a Target Group become unhealthy?

**Answer:**

The Target Group has no healthy targets.

The Load Balancer cannot successfully forward traffic to healthy instances in that group. Requests may receive HTTP 503 responses.

### Q22. Can we attach an Auto Scaling Group to a Target Group?

**Answer:**

Yes.

We configure the Target Group ARN inside the Auto Scaling Group.

Terraform example:

```hcl
resource "aws_autoscaling_group" "web_asg" {
  target_group_arns = [
    aws_lb_target_group.web_tg.arn
  ]
}
```

When ASG launches new EC2 instances, they are automatically registered with the Target Group.

When ASG terminates instances, they are deregistered.

---

# SECTION 4: Listeners and Routing – Intermediate

### Q23. What is a Listener?

**Answer:**

A Listener checks for incoming connection requests on a configured protocol and port.

Example:

```text
HTTP:80
HTTPS:443
```

For ALB, the listener evaluates requests and forwards them to the appropriate Target Group based on configured rules.

### Q24. What is the difference between Listener and Listener Rule?

**Answer:**

Listener:
- Accepts incoming traffic.
- Listens on a particular protocol and port.

Listener Rule:
- Evaluates request conditions.
- Determines which Target Group receives the request.

### Q25. What is Path-Based Routing?

**Answer:**

Path-based routing forwards requests to different Target Groups based on URL paths.

Example:

```text
example.com/api/*     → API TG
example.com/admin/*   → Admin TG
example.com/kids/*    → Kids TG
```

### Q26. What is Host-Based Routing?

**Answer:**

Host-based routing forwards requests based on the domain name.

Example:

```text
api.example.com   → API TG
admin.example.com → Admin TG
shop.example.com  → Shop TG
```

### Q27. What is the Default Action in ALB?

**Answer:**

The default action handles requests that do not match any configured listener rules.

For example:

```text
/kids/*        → Kids TG
/electronics/* → Electronics TG

Everything else → Main TG
```

### Q28. What is Listener Rule Priority?

**Answer:**

Listener rules are evaluated according to their priority.

Lower numerical values are evaluated first.

Example:

| Priority | Path | Target Group |
|---|---|---|
| 10 | /api/* | API TG |
| 20 | /admin/* | Admin TG |
| Default | All unmatched requests | Main TG |

Each non-default rule must have a unique priority.

### Q29. Can we configure multiple conditions in one ALB listener rule?

**Answer:**

Yes.

We can combine conditions such as:
- Path pattern.
- Host header.
- HTTP header.
- Query string.
- Source IP.

For example, a rule can match a particular hostname and URL path together.

### Q30. What is HTTPS termination in ALB?

**Answer:**

HTTPS termination means the ALB handles the TLS encryption and decryption using an SSL/TLS certificate.

We can configure an HTTPS listener on port 443 and associate an ACM certificate.

Traffic can then be forwarded to backend instances using HTTP or HTTPS, depending on the architecture.

---

# SECTION 5: Health Checks – Intermediate

### Q31. What is a Health Check?

**Answer:**

A health check is a mechanism used by the Load Balancer to monitor the availability of registered backend targets.

It periodically checks whether the target is healthy.

### Q32. What are Healthy Threshold and Unhealthy Threshold?

**Answer:**

Healthy Threshold:
Number of consecutive successful health checks required to mark a target healthy.

Unhealthy Threshold:
Number of consecutive failed health checks required to mark a target unhealthy.

Example:

```text
Healthy Threshold   = 2
Unhealthy Threshold = 2
```

### Q33. What is Health Check Interval?

**Answer:**

It specifies how frequently the Load Balancer sends health-check requests to backend instances.

Example:

```text
Health Check Interval = 30 seconds
```

### Q34. What is Health Check Timeout?

**Answer:**

It defines how long the Load Balancer waits for a health-check response before considering that check unsuccessful.

### Q35. What is the difference between EC2 Status Check and Load Balancer Health Check?

**Answer:**

EC2 Status Checks:
- Monitor EC2 infrastructure health.
- Check system and instance reachability.

Load Balancer Health Checks:
- Monitor whether registered application targets are responding according to the configured health-check protocol and path.

An EC2 instance can pass EC2 status checks but fail the application health check.

### Q36. Does an unhealthy instance get terminated automatically by ALB?

**Answer:**

No.

ALB stops routing traffic to unhealthy targets.

If an Auto Scaling Group is configured with ELB health checks, it can detect unhealthy instances and replace them according to its configuration.

---

# SECTION 6: Cross-Zone Load Balancing and Connection Draining

### Q37. What is Cross-Zone Load Balancing?

**Answer:**

Cross-Zone Load Balancing distributes traffic received by a Load Balancer node across registered targets in multiple enabled Availability Zones.

It helps distribute traffic more evenly when target counts differ between Availability Zones.

### Q38. What happens if Cross-Zone Load Balancing is disabled?

**Answer:**

Traffic received by a Load Balancer node is distributed among targets in its own Availability Zone.

This can result in uneven traffic distribution if different AZs have different numbers of targets.

### Q39. What is Connection Draining?

**Answer:**

Connection Draining allows existing connections to complete before an instance is deregistered from the Load Balancer.

It prevents unnecessary interruption during instance replacement or maintenance.

For ALB and NLB, the corresponding concept is called Deregistration Delay.

### Q40. What is the difference between Connection Draining and Deregistration Delay?

**Answer:**

- CLB uses Connection Draining.
- ALB and NLB use Target Group Deregistration Delay.

Both help allow existing connections to finish before target deregistration completes.

---

# SECTION 7: Auto Scaling and Load Balancer Integration

### Q41. How are Auto Scaling Groups and Load Balancers connected?

**Answer:**

Auto Scaling Groups connect to ALB through Target Groups.

Architecture:

```text
              Application LB
                    |
                 Listener
                    |
                    v
                Target Group
                    |
                    v
              Auto Scaling Group
                    |
                Launch Template
                    |
              ----------------
              |              |
              v              v
            EC2-1          EC2-2
```

ASG launches EC2 instances and automatically registers them with the associated Target Group.

### Q42. Does Auto Scaling Group attach directly to ALB?

**Answer:**

No.

ASG attaches to the Target Group, and the ALB listener forwards traffic to that Target Group.

### Q43. Can one ALB support multiple Auto Scaling Groups?

**Answer:**

Yes.

Example:

```text
                  ALB
                   |
          -------------------
          |        |        |
          v        v        v
        Main TG  API TG  Admin TG
          |        |        |
          v        v        v
        Main ASG API ASG Admin ASG
```

Each Target Group can be associated with its own Auto Scaling Group.

### Q44. Can one ASG attach to multiple Target Groups?

**Answer:**

Yes.

We can configure multiple Target Group ARNs in the Auto Scaling Group.

Example:

```hcl
target_group_arns = [
  aws_lb_target_group.web_tg.arn,
  aws_lb_target_group.web_tg2.arn
]
```

### Q45. What happens when ASG launches a new EC2 instance?

**Answer:**

1. ASG launches an EC2 instance using the Launch Template.
2. The instance is registered with the associated Target Group.
3. Target Group performs health checks.
4. Once healthy, it becomes eligible to receive traffic.

### Q46. What happens when ASG terminates an EC2 instance?

**Answer:**

The instance is deregistered from the Target Group.

The Load Balancer stops sending new requests to it, and the configured deregistration delay allows existing connections to complete where applicable.

---

# SECTION 8: Security and Networking

### Q47. What is the difference between ALB Security Group and EC2 Security Group?

**Answer:**

ALB Security Group:
Controls which clients can send traffic to the Load Balancer.

EC2 Security Group:
Controls which sources can communicate with backend EC2 instances.

Recommended configuration:

```text
Internet
   |
   v
ALB Security Group
   |
   | Allow HTTP:80
   v
EC2 Security Group
   |
   v
Application
```

EC2 should allow application traffic from the ALB security group.

### Q48. Why do we allow HTTP from the ALB Security Group in EC2 inbound rules?

**Answer:**

Because the ALB needs to communicate with the backend EC2 instances.

Instead of allowing traffic from everyone, we allow traffic specifically from the ALB security group.

This improves security.

### Q49. Does ALB need to be in a public subnet?

**Answer:**

An Internet-facing ALB requires subnets with internet connectivity, typically public subnets with a route to an Internet Gateway.

An internal ALB is deployed in subnets for private access.

Backend EC2 instances do not need to be public.

### Q50. Can an Internet-facing ALB send traffic to EC2 instances in private subnets?

**Answer:**

Yes.

The ALB can be Internet-facing while backend EC2 instances remain in private subnets.

The EC2 security group should allow application traffic from the ALB security group.

### Q51. Does an ALB have a static IP address?

**Answer:**

ALB does not provide a guaranteed static IP address.

For applications requiring static IP addresses, NLB is generally a better choice.

---

# SECTION 9: Terraform Interview Questions

### Q52. Which Terraform resource creates an Application Load Balancer?

**Answer:**

```hcl
resource "aws_lb" "app_alb" {
  name               = "app-alb"
  load_balancer_type = "application"
}
```

### Q53. Which Terraform resource creates a Target Group?

**Answer:**

```hcl
resource "aws_lb_target_group" "web_tg" {
  name     = "web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
}
```

### Q54. Which Terraform resource creates a Listener?

**Answer:**

```hcl
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.app_alb.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web_tg.arn
  }
}
```

### Q55. Which Terraform resource creates a Listener Rule?

**Answer:**

```hcl
resource "aws_lb_listener_rule" "api_rule" {
  listener_arn = aws_lb_listener.http.arn
  priority     = 10

  condition {
    path_pattern {
      values = ["/api/*"]
    }
  }

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.api_tg.arn
  }
}
```

### Q56. Which Terraform resource creates a Classic Load Balancer?

**Answer:**

```hcl
resource "aws_elb" "classic_lb" {
  name = "classic-lb"
}
```

### Q57. What is the difference between aws_elb and aws_lb?

**Answer:**

- aws_elb creates a Classic Load Balancer.
- aws_lb creates an Application, Network or Gateway Load Balancer depending on load_balancer_type.

### Q58. How do you connect ASG to a Target Group in Terraform?

**Answer:**

```hcl
resource "aws_autoscaling_group" "web_asg" {
  min_size         = 2
  max_size         = 5
  desired_capacity = 2

  target_group_arns = [
    aws_lb_target_group.web_tg.arn
  ]
}
```

### Q59. Does Terraform depend on the order of resources in the file?

**Answer:**

No.

Terraform builds a dependency graph based on resource references.

For example:

```hcl
target_group_arn = aws_lb_target_group.web_tg.arn
```

Terraform understands that the Target Group must be created before the Listener that references it.

---

# SECTION 10: Scenario-Based Interview Questions

## Scenario 1: Website is not accessible through ALB DNS.

**Question:**

You created an ALB and registered two EC2 instances. Both EC2 instances are running, but when you access the ALB DNS name, the website is not opening. How will you troubleshoot?

**Answer:**

I would troubleshoot layer by layer.

1. Check whether the ALB state is Active.
2. Check the listener configuration.
3. Check whether the default action points to the correct Target Group.
4. Check Target Group health status.
5. Verify EC2 security group inbound rules.
6. Verify ALB security group inbound rules.
7. Check Apache service status.
8. Check whether the application is listening on the configured port.
9. Verify the health-check path.
10. Check VPC routing and subnet configuration.

Commands:

```bash
sudo systemctl status httpd

sudo ss -tulnp | grep :80

curl http://localhost/index.html
```

## Scenario 2: EC2 is running but Target Group shows unhealthy.

**Question:**

Your EC2 instance status is running and both EC2 status checks are passed, but the Target Group shows the instance as unhealthy. Why?

**Answer:**

EC2 status checks and Target Group health checks monitor different things.

Possible reasons:

- Apache is not running.
- Application is listening on a different port.
- Health-check path is incorrect.
- Security group blocks traffic from ALB.
- Application returns an unsuccessful health-check response.
- Network ACL or routing issue.

I would check the health-check reason in the Target Group and verify the application locally.

## Scenario 3: One ALB needs to serve three applications.

**Question:**

You have three applications: frontend, backend API and admin application. Each application runs on separate EC2 instances. How would you design this?

**Answer:**

I would create:

- One Application Load Balancer.
- Three Target Groups.
- One Listener.
- Three routing destinations, including the default action.

Listener rules:

```text
/                 → Frontend TG
/api/*            → Backend API TG
/admin/*          → Admin TG
```

Each Target Group contains the respective application's EC2 instances.

This architecture reduces the need for separate Load Balancers for every application.

## Scenario 4: One Availability Zone goes down.

**Question:**

Your application runs in two Availability Zones. AZ-A experiences a failure. What happens?

**Answer:**

If the ALB and backend instances are deployed across multiple Availability Zones, traffic can continue through healthy targets in the remaining Availability Zone.

The ALB health checks stop routing traffic to unhealthy targets.

However, the application must have sufficient healthy capacity in the surviving AZ.

## Scenario 5: Traffic suddenly increases from 1,000 users to 50,000 users.

**Question:**

How would you handle sudden traffic growth?

**Answer:**

I would use ALB with an Auto Scaling Group.

1. ALB receives incoming traffic.
2. ALB distributes traffic across healthy instances.
3. CloudWatch monitors metrics such as CPU utilization or request count per target.
4. Auto Scaling policies trigger scale-out.
5. ASG launches additional EC2 instances.
6. New instances register automatically with the Target Group.
7. ALB starts distributing traffic to healthy new instances.

When traffic decreases, ASG can scale in.

## Scenario 6: Kids application works, but Electronics application returns 404.

**Question:**

You configured path-based routing for two applications. Kids works correctly, but Electronics returns 404. What will you check?

**Answer:**

I would check:

1. Listener rule path pattern.
2. Listener rule priority.
3. Target Group selected in the forward action.
4. Target Group health.
5. Registered EC2 instances.
6. Whether the application directory exists.
7. Whether the requested URL path matches the application structure.

Example:

```text
Listener Rule:

/electronics/* → electronics-tg
```

On the backend EC2:

```bash
ls -l /var/www/html/electronics/

curl http://localhost/electronics/index.html
```

A 404 can indicate that the backend application does not contain the requested resource.

## Scenario 7: ALB returns HTTP 503.

**Question:**

Users are receiving HTTP 503 when accessing your application through ALB. What could be the reason?

**Answer:**

A common reason is that the Target Group has no healthy registered targets.

I would check:

- Target Group health.
- Registered instance status.
- Health-check path.
- Health-check port.
- EC2 security group.
- Application availability.

I would also verify the listener's forwarding configuration.

## Scenario 8: One EC2 instance becomes unhealthy.

**Question:**

Your Target Group has four EC2 instances. One instance becomes unhealthy. What happens?

**Answer:**

ALB stops routing new requests to the unhealthy instance.

The other three healthy instances continue serving traffic.

If an Auto Scaling Group is integrated with ELB health checks, it can replace the unhealthy instance.

## Scenario 9: Application works directly through EC2 public IP but not through ALB.

**Question:**

You can access the application through the EC2 public IP, but the same application is not working through ALB. What would you investigate?

**Answer:**

I would investigate:

1. ALB listener configuration.
2. Target Group registration.
3. Target Group health.
4. ALB security group.
5. EC2 security group inbound source.
6. Health-check configuration.
7. ALB subnet and Availability Zone configuration.

A common mistake is allowing HTTP from the internet in the EC2 security group but not allowing traffic from the ALB security group.

## Scenario 10: You want to expose an application using HTTPS.

**Question:**

Your company wants to move from HTTP to HTTPS. How would you configure ALB?

**Answer:**

I would:

1. Request or import an SSL/TLS certificate in AWS Certificate Manager.
2. Create an HTTPS listener on port 443.
3. Associate the ACM certificate with the listener.
4. Configure the listener's default action or rules to forward traffic to the appropriate Target Group.
5. Allow HTTPS traffic in the ALB security group.
6. Optionally configure an HTTP listener on port 80 to redirect requests to HTTPS.

Example:

```text
User
  |
  v
ALB HTTPS:443
  |
  v
ACM Certificate
  |
  v
Target Group
  |
  v
EC2 Application
```

## Scenario 11: Backend EC2 instances are in private subnets.

**Question:**

Can an Internet-facing ALB communicate with EC2 instances in private subnets?

**Answer:**

Yes.

The ALB can be deployed in public subnets while backend EC2 instances remain in private subnets.

The ALB security group allows public HTTP/HTTPS traffic, and the EC2 security group allows application traffic from the ALB security group.

Backend instances do not require public IP addresses.

## Scenario 12: Two applications need different scaling policies.

**Question:**

Frontend requires 2–5 EC2 instances, while backend API requires 4–10 EC2 instances. How would you design the architecture?

**Answer:**

I would create:

- One ALB.
- Frontend Target Group.
- Backend API Target Group.
- Frontend Auto Scaling Group.
- Backend API Auto Scaling Group.
- Separate scaling policies.

Architecture:

```text
                 ALB
                  |
          ----------------
          |              |
          v              v
      Frontend TG     API TG
          |              |
          v              v
      Frontend ASG     API ASG
       2 to 5           4 to 10
```

Each application can scale independently according to its workload.

## Scenario 13: A new EC2 instance is launched by ASG but receives no traffic.

**Question:**

ASG successfully launches a new EC2 instance, but the instance is not receiving application traffic. What would you check?

**Answer:**

I would verify:

1. Whether the instance is registered in the correct Target Group.
2. Whether the instance is healthy.
3. Whether the application is running.
4. Whether the application listens on the Target Group port.
5. Whether security groups allow traffic.
6. Whether the listener forwards traffic to the correct Target Group.

A newly registered instance must become healthy before it receives normal traffic.

## Scenario 14: Users are experiencing failed requests during deployment.

**Question:**

During deployment, old EC2 instances are terminated before new instances become healthy. How would you avoid this?

**Answer:**

I would use a suitable Auto Scaling deployment strategy, such as a rolling update or instance refresh with healthy capacity maintained.

I would also configure:

- Appropriate health checks.
- Instance warm-up.
- Target Group deregistration delay.
- Minimum healthy capacity.
- Connection draining behavior.

This allows new instances to become healthy before old capacity is removed.

## Scenario 15: One application requires static IP addresses.

**Question:**

Your application must be accessed through static IP addresses. Which Load Balancer would you select?

**Answer:**

I would generally choose Network Load Balancer because it supports static IP addresses and Elastic IP associations.

ALB does not provide guaranteed static IP addresses.

## Scenario 16: You need to inspect all network traffic using third-party firewalls.

**Question:**

Your organization wants all network traffic inspected by firewall appliances before reaching applications. Which Load Balancer would you use?

**Answer:**

I would use Gateway Load Balancer.

It distributes traffic across virtual security appliances such as firewalls and IDS/IPS solutions.

It uses GENEVE encapsulation and Gateway Load Balancer Endpoints for traffic insertion.

---

# SECTION 11: Troubleshooting Quick Reference

| Issue | Possible Cause | What to Check |
|---|---|---|
| ALB DNS not working | Listener or SG issue | Listener, ALB SG |
| Target unhealthy | Health check failing | Path, port, application |
| HTTP 503 | No healthy targets | Target Group health |
| HTTP 404 | Missing application resource | URL path, backend files |
| EC2 not receiving traffic | Wrong Target Group | Listener rules |
| HTTPS not working | Certificate/listener issue | ACM, port 443 |
| New ASG instance not receiving traffic | Registration or health issue | ASG and Target Group |
| Traffic uneven across AZs | Cross-zone configuration or target distribution | AZ and target counts |
| Application timeout | Network or application issue | SG, NACL, routes, logs |

---

# SECTION 12: Interview Quick Revision

| Question | Key Answer |
|---|---|
| What is ELB? | Elastic Load Balancing |
| ALB OSI Layer? | Layer 7 |
| NLB OSI Layer? | Layer 4 |
| GWLB OSI Layer? | Layer 3 |
| CLB OSI Layers? | Layer 4 and 7 |
| Which LB supports path routing? | ALB |
| Which LB supports static IP? | NLB |
| Which LB supports UDP? | NLB |
| Which LB is used for firewalls? | GWLB |
| What is a Target Group? | Backend target collection |
| What is a Listener? | Incoming connection entry point |
| What is a Listener Rule? | Routing condition and action |
| What is Health Check? | Backend availability monitoring |
| What is Cross-Zone LB? | Distributes traffic across AZ targets |
| What is Connection Draining? | Allows existing connections to finish |
| How does ASG connect to ALB? | Through Target Groups |
| What is Path-Based Routing? | Routing based on URL paths |
| What is Host-Based Routing? | Routing based on domain names |
| What is HTTPS Termination? | ALB handles TLS termination |
| Terraform ALB resource? | aws_lb |
| Terraform CLB resource? | aws_elb |
| Terraform Target Group? | aws_lb_target_group |
| Terraform Listener? | aws_lb_listener |
| Terraform Listener Rule? | aws_lb_listener_rule |

---

# SECTION 13: Practical Tasks for Interview Preparation

Practise these tasks independently without referring to notes.

- [ ] Create two EC2 instances in different Availability Zones.
- [ ] Create a Classic Load Balancer.
- [ ] Configure CLB listener HTTP:80.
- [ ] Configure CLB health checks.
- [ ] Enable Cross-Zone Load Balancing.
- [ ] Enable Connection Draining.
- [ ] Access the application through CLB DNS.
- [ ] Create an Application Load Balancer.
- [ ] Create three Target Groups.
- [ ] Register two EC2 instances in each Target Group.
- [ ] Configure ALB default listener action.
- [ ] Configure path-based routing.
- [ ] Configure host-based routing.
- [ ] Configure listener rule priorities.
- [ ] Test all applications through ALB DNS.
- [ ] Create an NLB with a TCP listener.
- [ ] Understand UDP listener configuration.
- [ ] Integrate ALB with an Auto Scaling Group.
- [ ] Configure CPU-based scaling.
- [ ] Configure HTTPS using ACM.
- [ ] Troubleshoot unhealthy Target Groups.

---

# Final Interview Advice

When answering Load Balancer questions, explain using this order:

1. What the service is.
2. Which OSI layer it operates on.
3. How it works.
4. Where it is used in real-world applications.
5. Give an architecture example.
6. Explain how you would troubleshoot failures.

Do not just memorize definitions.

Practise explaining the traffic flow:

User → Load Balancer → Listener → Listener Rule → Target Group → EC2.

For Auto Scaling:

Launch Template → Auto Scaling Group → Target Group → Load Balancer.

These two architectures are especially important for junior AWS DevOps and Cloud Engineer interviews.