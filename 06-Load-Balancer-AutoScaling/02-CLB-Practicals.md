# AWS Classic Load Balancer (CLB) – Complete Practical Notes

## 1. Practical Overview

In this practical, we create a Classic Load Balancer (CLB) and attach two EC2 instances running in two different Availability Zones.

### Objective

- Create two EC2 instances in different Availability Zones.
- Install Apache web server on both instances.
- Create an Internet-facing Classic Load Balancer.
- Configure VPC and network mapping.
- Configure security groups.
- Configure listeners and routing.
- Configure health checks.
- Register EC2 instances with the Load Balancer.
- Enable Cross-Zone Load Balancing.
- Enable Connection Draining.
- Access the application using the Load Balancer DNS name.

## 2. Architecture

```text
                       Internet
                          |
                          v
                Classic Load Balancer
                   (Internet Facing)
                          |
                    Listener HTTP:80
                          |
                 ---------------------
                 |                   |
                 v                   v
             EC2 Instance 1      EC2 Instance 2
                AZ-A                 AZ-B
                 |                   |
                 v                   v
               Apache              Apache
               HTTP:80             HTTP:80
```

## 3. Step 1 – Create Two EC2 Instances

Create two EC2 instances in two different Availability Zones.

### EC2 Instance Configuration

| Configuration | Instance 1 | Instance 2 |
|---|---|---|
| Instance Name | Web-Server-1 | Web-Server-2 |
| Region | us-west-2 | us-west-2 |
| Availability Zone | us-west-2a | us-west-2b |
| Instance Type | t2.micro | t2.micro |
| Security Group | Allow HTTP | Allow HTTP |
| Application | Apache | Apache |
| Application Port | 80 | 80 |

### Install Apache on Both Instances

Run the following commands on both EC2 instances:

```bash
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

### Create Different Index Pages

**Instance 1:**

```bash
echo "Welcome to Web Server 1 - AZ-A" | sudo tee /var/www/html/index.html
```

**Instance 2:**

```bash
echo "Welcome to Web Server 2 - AZ-B" | sudo tee /var/www/html/index.html
```

### Verify Apache Service

```bash
sudo systemctl status httpd
```

Test the application:

```bash
curl http://localhost
```

Expected output:

Instance 1:
```text
Welcome to Web Server 1 - AZ-A
```

Instance 2:
```text
Welcome to Web Server 2 - AZ-B
```

---

## 4. Step 2 – Create Classic Load Balancer

Navigate to:

AWS Console → EC2 → Load Balancers → Create Load Balancer → Classic Load Balancer.

### A. Basic Configuration

| Configuration | Selection |
|---|---|
| Load Balancer Name | internetfacing |
| Scheme | Internet-facing |
| IP Address Type | IPv4 |

**Internet-facing:**

An Internet-facing Load Balancer receives requests from users over the internet and forwards them to registered backend EC2 instances.

### B. Network Mapping

Select the following:

1. Select the correct VPC where EC2 instances are running.
2. Select Availability Zones.
3. Select the corresponding public subnets.
4. Ensure selected subnets belong to the same VPC as EC2 instances.

Example:

| Availability Zone | Subnet |
|---|---|
| us-west-2a | Public Subnet 1 |
| us-west-2b | Public Subnet 2 |

**Important:**

- CLB and EC2 instances must belong to the same VPC.
- Select the Availability Zones where you want the Load Balancer to receive traffic.
- For an Internet-facing CLB, use appropriate public subnets with internet connectivity.
- The Load Balancer must have sufficient subnet IP addresses.

### C. Security Group Configuration

Select an existing security group or create a new security group for the Load Balancer.

**Load Balancer Security Group:**

| Type | Protocol | Port | Source |
|---|---|---|---|
| HTTP | TCP | 80 | 0.0.0.0/0 |
| HTTPS | TCP | 443 | 0.0.0.0/0 |

**EC2 Instance Security Group:**

| Type | Protocol | Port | Source |
|---|---|---|---|
| HTTP | TCP | 80 | Load Balancer Security Group |

**Important Security Concept:**

- Users send HTTP requests to the Load Balancer.
- The Load Balancer forwards requests to EC2 instances.
- EC2 security group should allow HTTP traffic from the Load Balancer security group.
- This prevents direct public access to backend instances.

---

## 5. Step 3 – Configure Listeners and Routing

Listener configuration determines how the Load Balancer receives incoming requests and forwards them to backend EC2 instances.

### Listener Configuration

| Configuration | Value |
|---|---|
| Load Balancer Protocol | HTTP |
| Load Balancer Port | 80 |
| Instance Protocol | HTTP |
| Instance Port | 80 |

Example Terraform configuration:

```hcl
listener {
  instance_port     = 80
  instance_protocol = "http"
  lb_port           = 80
  lb_protocol       = "http"
}
```

### Understanding Listener Attributes

| Attribute | Meaning |
|---|---|
| lb_port | Port on which Load Balancer receives incoming traffic |
| lb_protocol | Protocol accepted by Load Balancer |
| instance_port | Backend EC2 application port |
| instance_protocol | Protocol used between Load Balancer and EC2 |

### Traffic Flow

```text
                  User
                    |
                    v
            Classic Load Balancer
                  HTTP:80
                    |
                    v
              EC2 Instance
                  HTTP:80
```

### Example: Different Listener and Instance Ports

```hcl
listener {
  instance_port     = 8080
  instance_protocol = "http"
  lb_port           = 80
  lb_protocol       = "http"
}
```

Traffic flow:

```text
User
  |
  v
Classic LB : 80
  |
  v
EC2 : 8080
```

Here, the user accesses port 80, but the backend application listens on port 8080.

---

## 6. Step 4 – Configure Health Checks

Health checks are used to monitor the health of registered EC2 instances.

The Load Balancer periodically sends health-check requests to backend instances.

If an instance passes the health check, it is marked healthy.

If it fails repeatedly, it is marked unhealthy and the Load Balancer stops routing new requests to it.

### Health Check Configuration

| Parameter | Example Value | Meaning |
|---|---|---|
| Ping Protocol | HTTP | Protocol used for health checks |
| Ping Port | 80 | Backend port to check |
| Ping Path | /index.html | Application endpoint |
| Response Timeout | 3 seconds | Maximum time to wait for response |
| Health Check Interval | 30 seconds | Time between health checks |
| Unhealthy Threshold | 2 | Consecutive failures before unhealthy |
| Healthy Threshold | 2 | Consecutive successes before healthy |

Example:

```text
Ping Protocol          = HTTP
Ping Port              = 80
Ping Path              = /index.html
Response Timeout       = 3 seconds
Health Check Interval  = 30 seconds
Unhealthy Threshold    = 2
Healthy Threshold      = 2
```

### How Health Checks Work

Suppose:

- Health Check Interval = 30 seconds.
- Unhealthy Threshold = 2.
- Healthy Threshold = 2.

If an EC2 instance fails two consecutive health checks, it is marked unhealthy.

If an unhealthy instance passes two consecutive health checks, it is marked healthy again.

### Health Check Architecture

```text
              Classic Load Balancer
                        |
                 Health Check Request
                        |
                        v
                  EC2 Instance
                        |
                 HTTP:80/index.html
                        |
              ---------------------
              |                   |
              v                   v
           Success              Failure
              |                   |
              v                   v
           Healthy             Unhealthy
```

**Important:**

- An unhealthy instance is not automatically terminated by CLB.
- CLB stops routing new requests to unhealthy instances.
- Auto Scaling can replace unhealthy instances when configured and integrated appropriately.

---

## 7. Step 5 – Register EC2 Instances

After configuring the listener and health checks, select the EC2 instances to register with the Classic Load Balancer.

Example:

```text
Web-Server-1 → us-west-2a
Web-Server-2 → us-west-2b
```

Select both instances and add them to the Load Balancer.

### What Happens After Registration?

- CLB registers both EC2 instances.
- CLB starts performing health checks.
- Healthy instances become eligible to receive traffic.
- Incoming requests are distributed across healthy registered instances.

**Important:**

Classic Load Balancer directly registers EC2 instances.

Unlike ALB and NLB, CLB does not use Target Groups.

---

## 8. Step 6 – Configure Additional Attributes

After registering instances, configure additional attributes.

### A. Cross-Zone Load Balancing

Enable Cross-Zone Load Balancing.

**Purpose:**

Cross-Zone Load Balancing distributes incoming traffic across registered backend instances in all enabled Availability Zones instead of restricting distribution to instances in the same Availability Zone as the receiving Load Balancer node.

Example:

```text
                  Classic LB
                      |
             -------------------
             |                 |
            AZ-A              AZ-B
             |                 |
           EC2-1             EC2-2
```

**Without Cross-Zone Load Balancing:**

- Traffic received by a Load Balancer node in AZ-A is distributed among instances in AZ-A.
- Traffic received by a Load Balancer node in AZ-B is distributed among instances in AZ-B.

**With Cross-Zone Load Balancing:**

- Traffic can be distributed across instances in both AZ-A and AZ-B.

### Advantages

1. Better traffic distribution.
2. Helps avoid uneven instance distribution across AZs.
3. Improves backend instance utilization.
4. Supports better traffic distribution across Availability Zones.

### B. Connection Draining

Enable Connection Draining.

Connection draining allows existing connections to complete before an instance is deregistered from the Load Balancer.

Example:

```text
Connection Draining = Enabled
Timeout             = 300 seconds
```

### How Connection Draining Works

Suppose an EC2 instance is going to be removed from the Load Balancer.

**Without Connection Draining:**

- Existing connections may be interrupted.
- Users may experience failed requests.
- Application sessions may be affected.

**With Connection Draining:**

1. Load Balancer stops sending new requests to the deregistering instance.
2. Existing connections get time to complete.
3. After the configured timeout, remaining connections may be closed.
4. Instance deregistration completes.

Example:

```text
Connection Draining Timeout = 300 seconds

Existing Connections
         |
         v
Allow up to 300 seconds
         |
         v
Instance Deregistered
```

**Important:**

- Connection draining does not wait indefinitely.
- It respects the configured timeout.
- It is useful during instance replacement and maintenance.
- It helps avoid interrupting active user requests.

---

## 9. Step 7 – Create the Load Balancer

Review all configurations before creating the Load Balancer.

### Configuration Checklist

- [ ] Load Balancer name.
- [ ] Internet-facing scheme.
- [ ] Correct VPC.
- [ ] Availability Zones.
- [ ] Public subnets.
- [ ] Security groups.
- [ ] Listener configuration.
- [ ] Health check configuration.
- [ ] Registered EC2 instances.
- [ ] Cross-Zone Load Balancing.
- [ ] Connection Draining.

Click **Create Load Balancer**.

AWS creates the Classic Load Balancer and provides a DNS name.

Wait until the Load Balancer becomes active.

---

## 10. Step 8 – Access the Application

After the Load Balancer becomes active, copy its DNS name.

Example:

```text
internetfacing-123456789.us-west-2.elb.amazonaws.com
```

Open the DNS name in a browser:

```text
http://internetfacing-123456789.us-west-2.elb.amazonaws.com
```

The Load Balancer forwards the incoming request to one of the healthy EC2 instances.

### Expected Responses

Instance 1:

```text
Welcome to Web Server 1 - AZ-A
```

Instance 2:

```text
Welcome to Web Server 2 - AZ-B
```

Refresh the browser multiple times to observe responses from different backend instances.

**Important:**

- The Load Balancer DNS name is the endpoint used by users.
- It resolves to the Load Balancer's network endpoints.
- CLB does not provide a guaranteed static IP address.
- Do not assume every refresh must return a different instance because connection reuse, DNS resolution and traffic distribution can affect the result.

---

## 11. Step 9 – Practical Testing and Troubleshooting

### Test 1: Verify Load Balancer DNS

```bash
nslookup internetfacing-123456789.us-west-2.elb.amazonaws.com
```

### Test 2: Access the Application

```bash
curl http://internetfacing-123456789.us-west-2.elb.amazonaws.com
```

### Test 3: Check Apache on EC2

```bash
sudo systemctl status httpd
```

### Test 4: Check Apache Listening Port

```bash
sudo ss -tulnp | grep :80
```

### Test 5: Test Backend Application

```bash
curl http://localhost/index.html
```

### Common Problems

| Problem | Possible Reason |
|---|---|
| Website not loading | Security group issue |
| Connection timeout | Incorrect network configuration |
| Instance unhealthy | Apache not running |
| Health check failing | Incorrect ping path or port |
| HTTP 403 | Apache permissions or configuration |
| HTTP 503 | No healthy registered instances |
| One instance not receiving traffic | Instance unhealthy or configuration issue |

### Troubleshooting Checklist

1. Check whether the Load Balancer is active.
2. Check whether both EC2 instances are registered.
3. Check the health status of both instances.
4. Verify Apache is running.
5. Verify application is listening on port 80.
6. Check Load Balancer security group inbound rules.
7. Check EC2 security group inbound rules.
8. Verify public subnet routing through Internet Gateway.
9. Verify health-check path.
10. Check Load Balancer Availability Zone configuration.

---

## 12. Complete Practical Flow

```text
1. Create VPC
       |
2. Create two public subnets
       |
3. Select two Availability Zones
       |
4. Create Internet Gateway
       |
5. Configure public route table
       |
6. Launch two EC2 instances
       |
7. Install Apache on both EC2s
       |
8. Create Classic Load Balancer
       |
9. Select Internet-facing
       |
10. Configure VPC and Availability Zones
       |
11. Select public subnets
       |
12. Configure Security Group
       |
13. Configure Listener HTTP:80
       |
14. Configure Health Checks
       |
15. Register both EC2 instances
       |
16. Enable Cross-Zone Load Balancing
       |
17. Enable Connection Draining
       |
18. Create Load Balancer
       |
19. Copy Load Balancer DNS name
       |
20. Access website through browser
       |
21. Verify traffic distribution
```

---

## 13. Important Interview Questions

### Q1. Why do we create EC2 instances in different Availability Zones?

To improve high availability. If one Availability Zone experiences a failure, the application can continue serving traffic through instances in another Availability Zone.

### Q2. What is the difference between Listener Port and Instance Port?

Listener Port is where the Load Balancer receives incoming traffic.

Instance Port is where the backend EC2 application receives traffic from the Load Balancer.

### Q3. What is Cross-Zone Load Balancing?

It allows a Load Balancer node in one Availability Zone to distribute traffic across registered instances in other enabled Availability Zones.

### Q4. What happens when an EC2 instance becomes unhealthy?

The Load Balancer stops routing new requests to that unhealthy instance. It does not automatically terminate the instance.

### Q5. What is Connection Draining?

It allows existing connections to complete before an instance is deregistered from the Load Balancer.

### Q6. What happens if Connection Draining is disabled?

Existing connections may be interrupted when an instance is deregistered.

### Q7. Does Classic Load Balancer support Target Groups?

No. CLB directly registers EC2 instances. ALB and NLB use Target Groups.

### Q8. Does Classic Load Balancer support path-based routing?

No. Path-based routing is supported by Application Load Balancer.

### Q9. What is the DNS name provided by a Load Balancer?

It is the DNS endpoint used by clients to access the Load Balancer. It resolves to the Load Balancer's network endpoints.

### Q10. Does a Load Balancer automatically replace an unhealthy EC2 instance?

No. Auto Scaling Group can replace unhealthy EC2 instances when health-check integration and replacement policies are configured.

### Q11. What is the purpose of a health check interval?

It defines how frequently the Load Balancer checks the health of registered EC2 instances.

### Q12. What is the difference between Healthy Threshold and Unhealthy Threshold?

- Healthy Threshold: Number of consecutive successful checks required to mark an instance healthy.
- Unhealthy Threshold: Number of consecutive failed checks required to mark an instance unhealthy.

### Q13. Can Classic Load Balancer work with Auto Scaling?

Yes. An Auto Scaling Group can register and deregister EC2 instances with a Classic Load Balancer automatically.

---

## 14. Key Takeaways

| Concept | Purpose |
|---|---|
| Internet-facing | Accepts traffic from the internet |
| Network Mapping | Select VPC, Availability Zones and subnets |
| Listener | Accepts incoming traffic |
| Listener Port | Port exposed by Load Balancer |
| Instance Port | Backend application port |
| Health Check | Monitors backend instance health |
| Cross-Zone Load Balancing | Distributes traffic across enabled AZs |
| Connection Draining | Allows existing connections to complete |
| DNS Name | Endpoint used to access the Load Balancer |

## 15. Practical Summary

Created two EC2 instances in different Availability Zones, installed Apache on both instances, configured an Internet-facing Classic Load Balancer, registered both EC2 instances, configured HTTP listener and health checks, enabled Cross-Zone Load Balancing and Connection Draining, and accessed the application using the Load Balancer DNS name.

### Important Learning

Classic Load Balancer directly attaches EC2 instances, whereas Application Load Balancer and Network Load Balancer use Target Groups.

For modern AWS DevOps projects, practise ALB + Target Group + Auto Scaling Group.