# AWS Application Load Balancer (ALB) – Complete Practical Notes

## 1. Practical Overview

In this practical, we create an Application Load Balancer (ALB) and configure path-based routing using multiple Target Groups.

We create six EC2 instances across three different Availability Zones, with two instances in each Availability Zone.

We create three Target Groups:
1. Main Target Group – Main website.
2. Kids Target Group – /kids/index.html
3. Electronics Target Group – /electronics/index.html

The ALB forwards incoming requests to the appropriate Target Group based on the URL path.

### Objectives

- Create six EC2 instances across three Availability Zones.
- Install and configure Apache on all six instances.
- Understand why Target Groups are required.
- Create three Target Groups.
- Register two EC2 instances in each Target Group.
- Create an Internet-facing Application Load Balancer.
- Configure listeners and default routing.
- Configure path-based routing using listener rules.
- Configure rule priorities.
- Test routing using the ALB DNS name.

---

## 2. Architecture

```text
                       Internet
                          |
                          v
                Application Load Balancer
                       (ALB)
                          |
                    Listener HTTP:80
                          |
                  Default Action
                          |
                          v
                    Main Target Group
                          |
                   ----------------
                   |              |
                   v              v
                 EC2-1          EC2-2
                 AZ-A           AZ-A


          Additional Listener Rules
                          |
              ------------------------
              |                      |
              v                      v
        /kids/*               /electronics/*
              |                      |
              v                      v
         Kids Target Group    Electronics Target Group
              |                      |
          ----------              ----------
          |        |              |        |
          v        v              v        v
        EC2-3    EC2-4          EC2-5    EC2-6
        AZ-B     AZ-B           AZ-C     AZ-C
```

---

## 3. Step 1 – Create Six EC2 Instances

Create six EC2 instances across three different Availability Zones.

### EC2 Instance Configuration

| Instance | Availability Zone | Target Group |
|---|---|---|
| Web-Server-1 | us-west-2a | Main TG |
| Web-Server-2 | us-west-2a | Main TG |
| Web-Server-3 | us-west-2b | Kids TG |
| Web-Server-4 | us-west-2b | Kids TG |
| Web-Server-5 | us-west-2c | Electronics TG |
| Web-Server-6 | us-west-2c | Electronics TG |

### Common Configuration

| Configuration | Value |
|---|---|
| Region | us-west-2 |
| Instance Type | t2.micro |
| Operating System | Amazon Linux |
| Web Server | Apache |
| Application Port | 80 |
| Security Group | Allow HTTP from ALB Security Group |

### Install Apache on All Six Instances

```bash
sudo yum install httpd -y

sudo systemctl start httpd

sudo systemctl enable httpd
```

### Configure Main Website

Run on EC2-1 and EC2-2:

```bash
sudo mkdir -p /var/www/html

echo "Welcome to Main Website" | sudo tee /var/www/html/index.html
```

### Configure Kids Website

Run on EC2-3 and EC2-4:

```bash
sudo mkdir -p /var/www/html/kids

echo "Welcome to Kids Section" | sudo tee /var/www/html/kids/index.html
```

### Configure Electronics Website

Run on EC2-5 and EC2-6:

```bash
sudo mkdir -p /var/www/html/electronics

echo "Welcome to Electronics Section" | sudo tee /var/www/html/electronics/index.html
```

### Verify Apache

```bash
sudo systemctl status httpd

curl http://localhost/index.html

curl http://localhost/kids/index.html

curl http://localhost/electronics/index.html
```

Each instance should serve the appropriate application content.

---

## 4. Step 2 – Why Do We Need Target Groups?

Before creating the Application Load Balancer, we create Target Groups.

### What is a Target Group?

A Target Group is a collection of backend resources such as EC2 instances to which the Load Balancer forwards incoming traffic.

Target Groups also perform health checks on registered instances.

### Why Do We Need Target Groups?

Imagine we have three different applications:

- Main website.
- Kids application.
- Electronics application.

If we send all requests to the same backend instances, we cannot independently route each application to its intended backend.

Target Groups help us organize backend instances based on applications.

### Target Group Architecture

```text
                Application Load Balancer
                            |
                       Listener : 80
                            |
               -------------------------
               |           |           |
               v           v           v
            Main TG      Kids TG    Electronics TG
               |           |           |
            --------    --------    --------
            |      |    |      |    |      |
            v      v    v      v    v      v
           EC2    EC2  EC2    EC2  EC2    EC2
```

### Important Target Group Responsibilities

1. Holds registered backend instances.
2. Performs health checks.
3. Identifies healthy and unhealthy targets.
4. Receives traffic forwarded by ALB listeners.
5. Allows multiple applications to have separate backend groups.

**Important:**

- ALB forwards traffic to Target Groups.
- Target Groups contain registered targets.
- ALB does not directly attach EC2 instances.
- Target Groups do not perform path-based routing. Listener rules perform routing.

---

## 5. Step 3 – Create Main Target Group

Navigate to:

AWS Console → EC2 → Target Groups → Create Target Group.

### A. Basic Configuration

| Configuration | Selection |
|---|---|
| Target Type | Instances |
| Target Group Name | main-tg |
| Protocol | HTTP |
| Port | 80 |
| VPC | Select your application VPC |
| Protocol Version | HTTP1 |

### B. Configure Health Checks

| Parameter | Value |
|---|---|
| Health Check Protocol | HTTP |
| Health Check Path | /index.html |
| Health Check Port | Traffic Port |
| Healthy Threshold | 2 |
| Unhealthy Threshold | 2 |
| Timeout | 5 seconds |
| Interval | 30 seconds |

### C. Register Targets

Select the following instances:

- Web-Server-1
- Web-Server-2

Click Include as pending below and create the Target Group.

Main Target Group:

```text
main-tg
   |
   |--- Web-Server-1 (AZ-A)
   |
   |--- Web-Server-2 (AZ-A)
```

---

## 6. Step 4 – Create Kids Target Group

Repeat the Target Group creation process.

### A. Basic Configuration

| Configuration | Selection |
|---|---|
| Target Type | Instances |
| Target Group Name | kids-tg |
| Protocol | HTTP |
| Port | 80 |
| VPC | Same application VPC |

### B. Health Check Configuration

| Parameter | Value |
|---|---|
| Health Check Protocol | HTTP |
| Health Check Path | /kids/index.html |
| Health Check Port | Traffic Port |
| Healthy Threshold | 2 |
| Unhealthy Threshold | 2 |
| Timeout | 5 seconds |
| Interval | 30 seconds |

### C. Register Targets

Select:

- Web-Server-3
- Web-Server-4

Create the Target Group.

Kids Target Group:

```text
kids-tg
   |
   |--- Web-Server-3 (AZ-B)
   |
   |--- Web-Server-4 (AZ-B)
```

---

## 7. Step 5 – Create Electronics Target Group

Repeat the Target Group creation process.

### A. Basic Configuration

| Configuration | Selection |
|---|---|
| Target Type | Instances |
| Target Group Name | electronics-tg |
| Protocol | HTTP |
| Port | 80 |
| VPC | Same application VPC |

### B. Health Check Configuration

| Parameter | Value |
|---|---|
| Health Check Protocol | HTTP |
| Health Check Path | /electronics/index.html |
| Health Check Port | Traffic Port |
| Healthy Threshold | 2 |
| Unhealthy Threshold | 2 |
| Timeout | 5 seconds |
| Interval | 30 seconds |

### C. Register Targets

Select:

- Web-Server-5
- Web-Server-6

Create the Target Group.

Electronics Target Group:

```text
electronics-tg
   |
   |--- Web-Server-5 (AZ-C)
   |
   |--- Web-Server-6 (AZ-C)
```

### Target Group Summary

| Target Group | Registered Instances | Health Check Path |
|---|---|---|
| main-tg | EC2-1, EC2-2 | /index.html |
| kids-tg | EC2-3, EC2-4 | /kids/index.html |
| electronics-tg | EC2-5, EC2-6 | /electronics/index.html |

**Important:**

Each Target Group has its own health-check configuration.

This allows ALB to monitor the health of each application independently.

---

## 8. Step 6 – Create Application Load Balancer

After creating all three Target Groups, create the ALB.

Navigate to:

AWS Console → EC2 → Load Balancers → Create Load Balancer → Application Load Balancer.

### A. Basic Configuration

| Configuration | Selection |
|---|---|
| Load Balancer Name | application-alb |
| Scheme | Internet-facing |
| IP Address Type | IPv4 |

**Internet-facing:**

An Internet-facing ALB receives incoming traffic from the internet.

### B. Network Mapping

Select:

1. Your application VPC.
2. Three Availability Zones.
3. Corresponding public subnets.

Example:

| Availability Zone | Subnet |
|---|---|
| us-west-2a | Public Subnet 1 |
| us-west-2b | Public Subnet 2 |
| us-west-2c | Public Subnet 3 |

**Important:**

- ALB and Target Groups must belong to the same VPC.
- Select at least two Availability Zones for ALB.
- Public subnets should have a route to the Internet Gateway for an Internet-facing ALB.
- Backend EC2 instances can be in private or public subnets.

### C. Security Group

Select an existing security group or create a new one.

Example ALB Security Group:

| Type | Protocol | Port | Source |
|---|---|---|---|
| HTTP | TCP | 80 | 0.0.0.0/0 |
| HTTPS | TCP | 443 | 0.0.0.0/0 |

EC2 Security Group:

| Type | Protocol | Port | Source |
|---|---|---|---|
| HTTP | TCP | 80 | ALB Security Group |

**Important Security Concept:**

Users communicate with the ALB.

The ALB communicates with backend EC2 instances.

Therefore, EC2 security groups should allow application traffic from the ALB security group.

### D. Listeners and Routing During ALB Creation

Configure:

| Configuration | Value |
|---|---|
| Listener Protocol | HTTP |
| Listener Port | 80 |
| Default Action | Forward to main-tg |

**Important Practical Point:**

During ALB creation, configure the default listener action to forward traffic to the Main Target Group.

Do not try to configure all three Target Groups as the default action.

Why?

Because we want:

- Main website to be the default application.
- Kids application to receive traffic through a path-based rule.
- Electronics application to receive traffic through another path-based rule.

The additional Target Groups will be connected through listener rules after the ALB is created.

### Create the ALB

Review:

- ALB name.
- Internet-facing scheme.
- VPC.
- Availability Zones.
- Subnets.
- Security Group.
- HTTP listener.
- Default action pointing to main-tg.

Click Create Load Balancer.

Wait until the ALB becomes Active.

---

## 9. Step 7 – Configure Path-Based Routing

Now the ALB and all three Target Groups are created.

We will configure path-based routing using ALB listener rules.

Navigate to:

AWS Console → EC2 → Load Balancers → Select application-alb → Listeners and Rules.

Select the HTTP:80 listener.

Click View/Edit Rules or Manage Rules, then Add Rule.

### A. Create Kids Path-Based Rule

Configure:

| Configuration | Value |
|---|---|
| Condition | Path |
| Path Pattern | /kids/* |
| Action | Forward to kids-tg |
| Priority | 10 |

Traffic flow:

```text
User Request
    |
    v
http://ALB-DNS/kids/index.html
    |
    v
ALB Listener : 80
    |
    v
Path Rule: /kids/*
    |
    v
Kids Target Group
    |
    ----------------
    |              |
    v              v
  EC2-3          EC2-4
  AZ-B           AZ-B
```

### B. Create Electronics Path-Based Rule

Click Add Rule again.

Configure:

| Configuration | Value |
|---|---|
| Condition | Path |
| Path Pattern | /electronics/* |
| Action | Forward to electronics-tg |
| Priority | 20 |

Traffic flow:

```text
User Request
    |
    v
http://ALB-DNS/electronics/index.html
    |
    v
ALB Listener : 80
    |
    v
Path Rule: /electronics/*
    |
    v
Electronics Target Group
    |
    ----------------
    |              |
    v              v
  EC2-5          EC2-6
  AZ-C           AZ-C
```

### C. Main Website – Default Rule

The Main Target Group is configured as the default action during ALB creation.

Any request that does not match a higher-priority listener rule is forwarded to main-tg.

Example:

```text
http://ALB-DNS/
http://ALB-DNS/index.html
```

Traffic flow:

```text
User Request
    |
    v
ALB Listener : 80
    |
    v
No Matching Path Rule
    |
    v
Default Action
    |
    v
Main Target Group
    |
    ----------------
    |              |
    v              v
  EC2-1          EC2-2
  AZ-A           AZ-A
```

---

## 10. Step 8 – Understand Listener Rule Priority

ALB listener rules are evaluated based on priority.

Lower numerical priority values are evaluated first.

Example:

| Priority | Condition | Forward Action |
|---|---|---|
| 10 | /kids/* | kids-tg |
| 20 | /electronics/* | electronics-tg |
| Default | All unmatched requests | main-tg |

Important:

- Each non-default listener rule must have a unique priority.
- Lower priority numbers are evaluated first.
- The default rule has no configurable numerical priority.
- The default action handles requests that do not match other rules.

---

## 11. Step 9 – Test Path-Based Routing Using ALB DNS

After configuring the listener rules, copy the ALB DNS name.

Example:

```text
application-alb-123456789.us-west-2.elb.amazonaws.com
```

### Test 1: Main Website

Open:

```text
http://application-alb-123456789.us-west-2.elb.amazonaws.com/
```

Expected response:

```text
Welcome to Main Website
```

Traffic goes to:

```text
Main TG → EC2-1 or EC2-2
```

### Test 2: Kids Application

Open:

```text
http://application-alb-123456789.us-west-2.elb.amazonaws.com/kids/index.html
```

Expected response:

```text
Welcome to Kids Section
```

Traffic goes to:

```text
Kids TG → EC2-3 or EC2-4
```

### Test 3: Electronics Application

Open:

```text
http://application-alb-123456789.us-west-2.elb.amazonaws.com/electronics/index.html
```

Expected response:

```text
Welcome to Electronics Section
```

Traffic goes to:

```text
Electronics TG → EC2-5 or EC2-6
```

### Test Using curl

```bash
# Main website
curl http://ALB-DNS/

# Kids application
curl http://ALB-DNS/kids/index.html

# Electronics application
curl http://ALB-DNS/electronics/index.html
```

Replace ALB-DNS with your actual Application Load Balancer DNS name.

---

## 12. Complete Architecture – Final Result

```text
                         Internet
                            |
                            v
                 Application Load Balancer
                       (ALB - HTTP:80)
                            |
                     Listener Rules
                            |
             --------------------------------
             |              |               |
             v              v               v
          Default         /kids/*      /electronics/*
             |              |               |
             v              v               v
          Main TG         Kids TG       Electronics TG
             |              |               |
          --------       --------        --------
          |      |       |      |        |      |
          v      v       v      v        v      v
         EC2-1  EC2-2   EC2-3  EC2-4    EC2-5  EC2-6
         AZ-A   AZ-A    AZ-B   AZ-B     AZ-C   AZ-C
```

### Final Configuration Summary

| Component | Configuration |
|---|---|
| Load Balancer | Application Load Balancer |
| Scheme | Internet-facing |
| Listener | HTTP:80 |
| Availability Zones | 3 |
| EC2 Instances | 6 |
| Target Groups | 3 |
| Main Target Group | main-tg |
| Kids Target Group | kids-tg |
| Electronics Target Group | electronics-tg |
| Default Action | Forward to main-tg |
| Path Rule 1 | /kids/* → kids-tg |
| Path Rule 2 | /electronics/* → electronics-tg |

---

## 13. Important Concepts to Remember

### Why create Target Groups before ALB?

Target Groups define the backend destinations to which ALB forwards traffic.

Creating them first allows us to select the Main Target Group while configuring the ALB listener's default action.

### Why use multiple Target Groups?

Different applications can have different backend EC2 instances and health checks.

### Can one ALB have multiple Target Groups?

Yes. One ALB can have multiple Target Groups.

### Can one ALB have multiple listener rules?

Yes. An ALB listener can have multiple rules with different conditions and priorities.

### Can Target Groups perform path-based routing?

No. Path-based routing is configured on ALB listener rules.

### Can we attach Auto Scaling Groups to Target Groups?

Yes. Each Auto Scaling Group can register its EC2 instances with the appropriate Target Group.

### Can the same EC2 instance belong to multiple Target Groups?

Yes, an EC2 instance can be registered with multiple Target Groups, provided the application and port configurations support it.

---

## 14. Important Interview Questions

### Q1. What is an Application Load Balancer?

ALB is a Layer 7 load balancer that distributes HTTP/HTTPS traffic based on application-level information such as URL paths, hostnames and HTTP headers.

### Q2. Why do we need Target Groups?

Target Groups define the backend destinations where ALB forwards requests and perform health checks on registered targets.

### Q3. What is path-based routing?

Path-based routing forwards requests to different Target Groups based on the URL path.

Example:

- /kids/* → Kids TG
- /electronics/* → Electronics TG

### Q4. What is host-based routing?

Host-based routing forwards requests based on the requested domain name.

Example:

- api.example.com → API TG
- admin.example.com → Admin TG

### Q5. Can one ALB support multiple applications?

Yes. One ALB can support multiple applications using multiple Target Groups and listener rules.

### Q6. What is the purpose of the default action?

The default action handles requests that do not match any configured listener rule.

### Q7. What is listener rule priority?

Priority determines the order in which ALB evaluates listener rules. Lower numerical values are evaluated first.

### Q8. What happens if a Target Group has no healthy instances?

The ALB cannot forward traffic successfully to healthy targets in that Target Group. Requests may return HTTP 503 if no usable targets are available.

### Q9. Can ALB perform TCP or UDP load balancing?

ALB is designed for HTTP/HTTPS application traffic. NLB is used for TCP, UDP and TLS network traffic.

### Q10. Can ALB work with Auto Scaling Groups?

Yes. An Auto Scaling Group can automatically register newly launched EC2 instances with the associated Target Group and deregister terminated instances.

### Q11. What is the difference between a Listener and a Listener Rule?

- Listener accepts incoming traffic on a specific protocol and port.
- Listener Rule evaluates request conditions and determines which Target Group receives the request.

### Q12. Why do we configure different health-check paths for different Target Groups?

Each Target Group may host a different application. Different health-check paths help verify that each application is functioning correctly.

---

## 15. Practical Summary

Created six EC2 instances across three Availability Zones, with two instances in each Availability Zone.

Installed Apache on all six instances and configured different website paths for Main, Kids and Electronics applications.

Created three Target Groups:

1. Main TG – /index.html
2. Kids TG – /kids/index.html
3. Electronics TG – /electronics/index.html

Created an Internet-facing Application Load Balancer and configured the HTTP:80 listener with Main TG as the default action.

After creating the ALB, configured listener rules for:

- /kids/* → Kids TG
- /electronics/* → Electronics TG

Finally, tested all three applications using the ALB DNS name and verified that path-based routing forwarded requests to the correct backend Target Groups.

### Key Learning

**ALB = Traffic Router**

**Listener = Entry Point**

**Listener Rules = Routing Conditions**

**Target Group = Backend EC2 Collection**

**Health Check = Backend Application Health Monitoring**

**Path-Based Routing = Route requests based on URL paths.**