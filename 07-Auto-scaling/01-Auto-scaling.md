# AWS EC2 Auto Scaling – Complete Notes and Practical

## 1. What is Auto Scaling?

Amazon EC2 Auto Scaling is an AWS service that automatically increases or decreases the number of EC2 instances based on application workload and demand.

When application traffic increases and resource utilization crosses the configured threshold, Auto Scaling launches additional EC2 instances.

When application traffic decreases, Auto Scaling can terminate unnecessary EC2 instances.

### Main Objectives

- Automatically launch EC2 instances.
- Automatically terminate unnecessary EC2 instances.
- Maintain minimum and maximum instance capacity.
- Maintain desired capacity.
- Improve application availability.
- Handle increasing and decreasing workloads.
- Optimize infrastructure costs.

**Important:**

Auto Scaling does not increase the Load Balancer capacity manually. It increases or decreases the number of backend EC2 instances.

AWS Load Balancer and Auto Scaling are two different services that work together.

---

## 2. Why Do We Need Auto Scaling?

Imagine an application running on two EC2 instances.

During normal business hours, two instances are sufficient.

During festival sales or peak traffic, thousands of users access the application simultaneously.

The existing EC2 instances may experience high CPU utilization and become overloaded.

Instead of manually launching additional EC2 instances, we configure Auto Scaling.

Auto Scaling automatically launches additional instances when scaling conditions are met.

### Example

Normal traffic:

```text
             Application
                  |
                  v
             Load Balancer
                  |
             -----------
             |         |
             v         v
            EC2-1    EC2-2
```

High traffic:

```text
             Application
                  |
                  v
             Load Balancer
                  |
        -----------------------
        |       |      |      |
        v       v      v      v
       EC2-1  EC2-2  EC2-3  EC2-4
```

Low traffic:

```text
             Application
                  |
                  v
             Load Balancer
                  |
             -----------
             |         |
             v         v
            EC2-1    EC2-2
```

---

## 3. Auto Scaling Architecture

```text
                    Internet
                       |
                       v
              Application Load Balancer
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
                -----------------
                |               |
                v               v
              EC2-1           EC2-2
                |               |
                -----------------
                       |
                       v
                   CloudWatch
                       |
                 CPU Utilization
                       |
                       v
                 Scaling Policy
                       |
              -------------------
              |                 |
              v                 v
           Scale Out          Scale In
           Add EC2            Remove EC2
```

### Component Responsibilities

| Component | Responsibility |
|---|---|
| ALB | Distributes incoming traffic |
| Listener | Accepts incoming requests |
| Target Group | Registers backend instances |
| Launch Template | Defines EC2 launch configuration |
| Auto Scaling Group | Manages EC2 instance capacity |
| CloudWatch | Monitors resource metrics |
| Scaling Policy | Defines when to scale |

**Important relationship:**

- ALB forwards traffic to Target Groups.
- ASG launches and terminates EC2 instances.
- ASG automatically registers instances with associated Target Groups.
- Target Groups perform health checks.
- ALB forwards traffic to healthy targets.

---

# 4. Important Auto Scaling Terminologies

## A. Launch Template

A Launch Template contains the configuration required to launch EC2 instances automatically.

It defines how every new EC2 instance should be created.

### Launch Template Configuration

| Configuration | Purpose |
|---|---|
| AMI | Operating system image |
| Instance Type | EC2 hardware capacity |
| Key Pair | SSH access |
| Security Group | Network access rules |
| User Data | Startup installation scripts |
| IAM Instance Profile | AWS permissions |
| Storage | EBS volume configuration |

Example:

```text
Launch Template
      |
      |--- AMI: Amazon Linux
      |
      |--- Instance Type: t2.micro
      |
      |--- Key Pair
      |
      |--- Security Group
      |
      |--- User Data: Install Apache
```

### Why Do We Need Launch Templates?

Suppose Auto Scaling needs to launch five EC2 instances.

Instead of manually specifying the AMI, instance type, key pair and security group for every instance, we define these settings once in a Launch Template.

ASG uses the Launch Template to launch additional instances with the same configuration.

## B. Auto Scaling Group (ASG)

An Auto Scaling Group is a logical group of EC2 instances managed together by Auto Scaling.

It maintains the required number of instances according to minimum, desired and maximum capacity.

### ASG Responsibilities

- Launch EC2 instances.
- Terminate EC2 instances.
- Maintain desired capacity.
- Replace unhealthy instances.
- Integrate with Load Balancers.
- Perform automatic scaling.

## C. Minimum Capacity

Minimum capacity defines the minimum number of EC2 instances that ASG should maintain.

Example:

```text
Minimum Capacity = 2
```

ASG normally maintains at least two instances.

## D. Desired Capacity

Desired capacity defines the target number of EC2 instances that ASG should maintain.

Example:

```text
Desired Capacity = 2
```

When we create the ASG, it initially attempts to launch two instances.

## E. Maximum Capacity

Maximum capacity defines the maximum number of EC2 instances ASG can launch.

Example:

```text
Maximum Capacity = 5
```

ASG cannot scale beyond five instances.

### Minimum, Desired and Maximum Capacity Example

```text
Minimum Capacity  = 2
Desired Capacity  = 2
Maximum Capacity  = 5
```

Initially:

```text
EC2-1
EC2-2
```

When traffic increases:

```text
EC2-1
EC2-2
EC2-3
EC2-4
EC2-5
```

When traffic decreases:

```text
EC2-1
EC2-2
```

| Capacity | Value | Meaning |
|---|---:|---|
| Minimum | 2 | Minimum instances |
| Desired | 2 | Target instance count |
| Maximum | 5 | Maximum instances |

---

# 5. Types of Auto Scaling Policies

Auto Scaling policies determine when to increase or decrease EC2 instances.

## A. Target Tracking Scaling

Target Tracking maintains a target value for a selected metric.

Example:

```text
Target CPU Utilization = 50%
```

ASG automatically adjusts EC2 capacity to maintain average CPU utilization near the target.

When workload increases, ASG scales out.

When workload decreases, ASG can scale in.

### Example

```text
Target CPU = 50%

CPU increases to 80%
        |
        v
    Scale Out
        |
        v
    Add EC2

CPU decreases to 20%
        |
        v
     Scale In
        |
        v
   Remove EC2
```

Common metrics:

- Average CPU Utilization.
- Average Network In.
- Average Network Out.
- ALB Request Count Per Target.

## B. Step Scaling

Step Scaling uses CloudWatch alarms and predefined scaling adjustments.

Example:

| CPU Utilization | Scaling Action |
|---|---|
| Above 70% | Add 1 EC2 |
| Above 85% | Add 2 EC2 |
| Below 30% | Remove 1 EC2 |

We define different scaling adjustments for different metric ranges.

## C. Simple Scaling

Simple Scaling uses a CloudWatch alarm to trigger a single scaling adjustment.

Example:

```text
CPU Utilization > 70%
        |
        v
Launch 1 EC2 Instance
```

Simple Scaling uses cooldown behavior to help prevent repeated scaling actions too quickly.

## D. Scheduled Scaling

Scheduled Scaling increases or decreases capacity at predefined times.

Example:

| Time | Desired Capacity |
|---|---:|
| 8 AM | 5 |
| 9 PM | 2 |

Useful for applications with predictable traffic patterns.

## E. Predictive Scaling

Predictive Scaling uses historical workload patterns to forecast future demand and schedule capacity ahead of expected traffic.

Useful for applications with recurring workload patterns.

### Scaling Policy Comparison

| Policy | Purpose |
|---|---|
| Target Tracking | Maintain a target metric |
| Step Scaling | Scale based on alarm thresholds |
| Simple Scaling | Apply a single scaling adjustment |
| Scheduled Scaling | Scale at predefined times |
| Predictive Scaling | Forecast workload and scale ahead |

---

# 6. Scale Out and Scale In

## Scale Out

Scale Out means increasing the number of EC2 instances when application workload increases.

Example:

```text
Initial Instances = 2

CPU Utilization = 80%
Configured Target = 50%

          CloudWatch
               |
               v
         Scaling Policy
               |
               v
          Scale Out
               |
               v
        Launch New EC2
               |
               v
        Target Group
               |
               v
          ALB Traffic
```

### Scale Out Process

1. Users generate application traffic.
2. EC2 CPU utilization increases.
3. CloudWatch monitors CPU utilization.
4. Scaling policy evaluates the configured metric.
5. ASG launches additional EC2 instances.
6. New instances register with the Target Group.
7. Target Group performs health checks.
8. ALB starts sending traffic to healthy instances.

## Scale In

Scale In means decreasing the number of EC2 instances when workload decreases.

Example:

```text
Initial Instances = 5

CPU Utilization = 20%
Configured Target = 50%

          CloudWatch
               |
               v
         Scaling Policy
               |
               v
           Scale In
               |
               v
       Terminate EC2
               |
               v
      Target Group Deregistration
```

### Scale In Process

1. Application traffic decreases.
2. CPU utilization decreases.
3. CloudWatch monitors the metric.
4. Scaling policy evaluates the configured conditions.
5. ASG selects an instance for termination.
6. Instance is deregistered from the Target Group.
7. Remaining instances continue serving traffic.

**Important:**

Scale-in does not happen immediately when CPU decreases. It depends on scaling policy evaluation periods, cooldown or instance warm-up settings and termination policies.

---

# 7. Auto Scaling and Load Balancer Relationship

Auto Scaling and Load Balancing are independent AWS services.

They work together through Target Groups.

### Load Balancer Responsibilities

- Accept incoming user traffic.
- Distribute requests across healthy EC2 instances.
- Perform health checks.
- Support application availability.

### Auto Scaling Responsibilities

- Launch new EC2 instances.
- Terminate unnecessary EC2 instances.
- Maintain desired capacity.
- Replace unhealthy instances.
- Automatically scale based on workload.

### Architecture

```text
                    Internet
                       |
                       v
              Application Load Balancer
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
               ------------------
               |                |
               v                v
             EC2-1            EC2-2
               |                |
               ------------------
                       |
                    CloudWatch
                       |
                 Scaling Policy
                       |
               ------------------
               |                |
               v                v
            Scale Out         Scale In
```

### Important Interview Points

**Does Auto Scaling come under Load Balancing?**

No. Both are separate AWS services.

**Can Auto Scaling work without a Load Balancer?**

Yes. Auto Scaling can launch and terminate EC2 instances independently.

**Can Load Balancer work without Auto Scaling?**

Yes. We can manually create EC2 instances and register them with a Target Group.

**How does ASG connect to ALB?**

Through Target Groups.

---

# 8. Auto Scaling Practical – AWS Console

## Step 1 – Choose Launch Template

Navigate to:

AWS Console → EC2 → Auto Scaling Groups → Create Auto Scaling Group.

The first step is Choose Launch Template.

### Enter Auto Scaling Group Name

Example:

```text
Auto Scaling Group Name = web-asg
```

### Select Launch Template

Select an existing Launch Template.

If you do not have one, create it first.

### Launch Template Configuration

| Configuration | Example |
|---|---|
| Launch Template Name | web-launch-template |
| AMI | Amazon Linux |
| Instance Type | t2.micro |
| Key Pair | Existing key pair |
| Security Group | Web Security Group |
| User Data | Install Apache |

### Example User Data

```bash
#!/bin/bash

yum install httpd -y

systemctl start httpd

systemctl enable httpd

echo "Hello from Auto Scaling EC2" > /var/www/html/index.html
```

### Why User Data?

User Data automatically executes startup scripts when a new EC2 instance launches.

This helps install and configure applications without manually connecting to every instance.

Select the Launch Template and click Next.

## Step 2 – Choose Instance Launch Options

Select the VPC where your application infrastructure is running.

Choose multiple Availability Zones and appropriate subnets.

Example:

| Availability Zone | Subnet |
|---|---|
| us-west-2a | Public Subnet 1 |
| us-west-2b | Public Subnet 2 |
| us-west-2c | Public Subnet 3 |

### Why Multiple Availability Zones?

To improve high availability.

If one Availability Zone becomes unavailable, instances in other Availability Zones can continue serving traffic.

**Important:**

For production applications, backend instances are commonly placed in private subnets, with the Internet-facing ALB in public subnets.

## Step 3 – Integrate with Other Services

This step connects Auto Scaling with the Load Balancer.

Select:

**Attach to an existing Load Balancer.**

Then select:

- Choose from your existing Target Groups.
- Select the appropriate Target Group.
- Configure health-check integration.

Example:

```text
Target Group = main-tg
```

### What Happens?

1. ASG launches EC2 instances.
2. EC2 instances register automatically with the selected Target Group.
3. Target Group performs health checks.
4. ALB sends traffic to healthy instances.

### ELB Health Check

You can enable Elastic Load Balancing health checks for the ASG.

When enabled, ASG can use the Load Balancer's target health information to determine whether an instance is healthy.

This allows ASG to replace instances that fail ELB health checks.

## Step 4 – Configure Group Size and Scaling

Configure:

```text
Minimum Capacity  = 2
Desired Capacity  = 2
Maximum Capacity  = 5
```

### Explanation

| Configuration | Value |
|---|---:|
| Minimum Capacity | 2 |
| Desired Capacity | 2 |
| Maximum Capacity | 5 |

Initially, ASG launches two EC2 instances.

When scaling conditions are met, ASG can increase the number of instances up to five.

### Configure Scaling Policy

Choose the required scaling policy.

Example:

```text
Policy Type = Target Tracking

Metric = Average CPU Utilization

Target Value = 50%
```

ASG attempts to maintain average CPU utilization around 50%.

## Step 5 – Add Notifications

Configure SNS notifications if you want to receive notifications about Auto Scaling events.

Examples:

- EC2 instance launch.
- EC2 instance termination.
- Failed launch.
- Scaling activities.

## Step 6 – Add Tags

Example:

| Key | Value |
|---|---|
| Name | ASG-WebServer |
| Environment | Dev |
| Project | WebApplication |

Tags help identify resources created by Auto Scaling.

## Step 7 – Review and Create

Review:

- Auto Scaling Group name.
- Launch Template.
- VPC.
- Availability Zones.
- Subnets.
- Target Group.
- Minimum capacity.
- Desired capacity.
- Maximum capacity.
- Scaling policy.
- Notifications.
- Tags.

Click Create Auto Scaling Group.

AWS launches the desired number of EC2 instances.

---

# 9. Practical Testing – Scale Out and Scale In

## Test 1 – Verify EC2 Instances

Navigate to:

EC2 → Instances.

Check whether the desired number of instances has launched.

Initially, you should see two EC2 instances.

## Test 2 – Verify Target Group

Navigate to:

EC2 → Target Groups → Select Target Group → Registered Targets.

Verify that newly launched instances are registered.

Wait until their health status becomes healthy.

## Test 3 – Generate CPU Load

On a Linux EC2 instance, install the stress utility.

Amazon Linux 2023:

```bash
sudo dnf install stress -y
```

Amazon Linux 2:

```bash
sudo amazon-linux-extras install epel -y
sudo yum install stress -y
```

Generate CPU load:

```bash
stress --cpu 2 --timeout 300
```

This generates CPU load for 300 seconds.

**Note:** The CPU load must be generated on the instances and the scaling metric must reflect the load. A single instance reaching high CPU does not necessarily mean the ASG's average CPU metric will cross its threshold.

## Test 4 – Monitor CloudWatch

Navigate to:

AWS Console → CloudWatch → Metrics → EC2 → Per-Instance Metrics.

Monitor:

```text
CPUUtilization
```

If you are using ASG average CPU tracking, also monitor the Auto Scaling Group metrics.

## Test 5 – Verify Scale Out

Navigate to:

EC2 → Auto Scaling Groups → Select ASG → Activity.

Check whether additional instances have launched.

Example:

```text
Before Scale Out:

EC2-1
EC2-2

After Scale Out:

EC2-1
EC2-2
EC2-3
EC2-4
```

## Test 6 – Verify Scale In

Stop generating CPU load.

Wait until the scaling policy evaluation conditions are satisfied.

ASG can terminate unnecessary instances.

Example:

```text
Before Scale In:

EC2-1
EC2-2
EC2-3
EC2-4

After Scale In:

EC2-1
EC2-2
```

**Important:**

The minimum capacity prevents ASG from scaling below the configured minimum.

---

# 10. Auto Scaling Health Checks

Auto Scaling can use different health-check sources.

### EC2 Health Checks

Monitor the underlying EC2 instance status.

### ELB Health Checks

Monitor the health status reported by the associated Load Balancer or Target Group.

### Difference

| EC2 Health Check | ELB Health Check |
|---|---|
| Monitors EC2 infrastructure health | Monitors registered target health |
| Checks instance/system status | Checks application endpoint availability |
| Detects infrastructure failures | Detects application-level failures |

### Example

An EC2 instance can be running and pass EC2 status checks, but Apache may have stopped.

In this situation:

- EC2 health checks can pass.
- Target Group health checks can fail.
- If ASG uses ELB health checks, it can replace the unhealthy instance.

---

# 11. Auto Scaling Important Interview Questions

### Q1. What is Auto Scaling?

Auto Scaling automatically increases or decreases EC2 instances based on workload demand and configured scaling policies.

### Q2. What is an Auto Scaling Group?

An Auto Scaling Group is a logical group of EC2 instances managed together by Auto Scaling.

### Q3. What is a Launch Template?

A Launch Template defines the EC2 configuration used when Auto Scaling launches new instances.

### Q4. What is the difference between Minimum, Desired and Maximum Capacity?

- Minimum: Lowest instance count.
- Desired: Target instance count.
- Maximum: Highest permitted instance count.

### Q5. What is Scale Out?

Increasing the number of EC2 instances when workload increases.

### Q6. What is Scale In?

Reducing the number of EC2 instances when workload decreases.

### Q7. What is Target Tracking Scaling?

A scaling policy that automatically adjusts instance capacity to maintain a selected metric near a target value.

### Q8. What is Step Scaling?

A scaling policy that applies different capacity adjustments based on CloudWatch alarm thresholds.

### Q9. What is Scheduled Scaling?

Scaling based on predefined schedules.

### Q10. Can ASG work without ALB?

Yes. ASG can operate independently without a Load Balancer.

### Q11. Can ALB work without ASG?

Yes. EC2 instances can be manually registered with Target Groups.

### Q12. How does ASG integrate with ALB?

ASG registers EC2 instances with the associated Target Group. ALB forwards traffic to that Target Group.

### Q13. What happens if an EC2 instance becomes unhealthy?

If ASG is configured with the appropriate health-check integration, it can replace the unhealthy instance.

### Q14. What is the purpose of CloudWatch in Auto Scaling?

CloudWatch monitors metrics and generates alarms that can trigger scaling policies.

### Q15. What happens when CPU utilization decreases?

If the configured scale-in policy conditions are satisfied, ASG can terminate unnecessary instances.

---

# 12. Scenario-Based Interview Questions

### Scenario 1 – CPU utilization increases to 90%.

**Question:**

Your application is running on two EC2 instances. Suddenly CPU utilization reaches 90%. How will you handle it?

**Answer:**

I would configure an Auto Scaling Group with a suitable scaling policy.

CloudWatch monitors CPU utilization. When the scaling conditions are met, ASG launches additional EC2 instances using the Launch Template.

The new instances register with the Target Group and receive traffic after becoming healthy.

### Scenario 2 – Traffic decreases during night time.

**Question:**

Your application receives high traffic during the day and very low traffic at night. How would you optimize costs?

**Answer:**

I would configure scheduled scaling or target tracking scaling.

During high traffic, ASG increases capacity.

During low traffic, ASG reduces capacity while respecting the minimum capacity.

### Scenario 3 – ASG launches a new EC2 instance, but it does not receive traffic.

**Question:**

What would you check?

**Answer:**

I would check:

1. Target Group registration.
2. Target Group health status.
3. Application service status.
4. Application port.
5. Security Group rules.
6. Listener forwarding configuration.
7. Health-check path.

### Scenario 4 – EC2 is running, but Apache is stopped.

**Question:**

Will Auto Scaling replace this instance?

**Answer:**

Not necessarily.

If ASG only uses EC2 health checks, the instance may still be considered healthy because the EC2 infrastructure is running.

If ELB health checks are enabled and the Target Group reports the instance unhealthy, ASG can replace it.

### Scenario 5 – Maximum capacity is 5 and CPU reaches 95%.

**Question:**

What happens when the ASG has already reached maximum capacity?

**Answer:**

ASG cannot launch additional instances beyond the configured maximum capacity.

The existing instances continue handling traffic, but performance may degrade.

I would review the maximum capacity, application optimization and scaling configuration.

### Scenario 6 – One Availability Zone becomes unavailable.

**Question:**

Your ASG has instances in three Availability Zones. One AZ becomes unavailable. What happens?

**Answer:**

ASG can maintain capacity using the remaining Availability Zones, provided they have sufficient subnet capacity and available EC2 capacity.

The Load Balancer can continue directing traffic to healthy instances in the remaining zones.

---

# 13. Important AWS Console Locations

| Component | AWS Console Location |
|---|---|
| Launch Template | EC2 → Instances → Launch Templates |
| Auto Scaling Group | EC2 → Auto Scaling Groups |
| Target Group | EC2 → Load Balancing → Target Groups |
| ALB | EC2 → Load Balancing → Load Balancers |
| Scaling Policies | EC2 → Auto Scaling Groups → Automatic Scaling |
| Scaling Activities | EC2 → Auto Scaling Groups → Activity |
| CloudWatch Metrics | CloudWatch → Metrics |
| CloudWatch Alarms | CloudWatch → Alarms |
| SNS Notifications | Amazon SNS |

---

# 14. Final Architecture and Revision

```text
                       Internet
                          |
                          v
                 Application Load Balancer
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
                  -----------------
                  |               |
                  v               v
                EC2-1           EC2-2
                  |               |
                  -----------------
                          |
                       CloudWatch
                          |
                    Scaling Policy
                          |
                  -----------------
                  |               |
                  v               v
               Scale Out        Scale In
               Add EC2          Remove EC2
```

### Key Takeaways

| AWS Component | Responsibility |
|---|---|
| Launch Template | Defines how EC2 instances are created |
| Auto Scaling Group | Manages EC2 instance capacity |
| CloudWatch | Monitors workload metrics |
| Scaling Policy | Determines when to scale |
| Target Group | Registers backend instances |
| ALB | Distributes incoming traffic |

### Complete Practical Learning Flow

```text
1. Create Launch Template
          |
2. Create Auto Scaling Group
          |
3. Select Launch Template
          |
4. Select VPC and Subnets
          |
5. Select Availability Zones
          |
6. Attach Existing Target Group
          |
7. Configure Health Checks
          |
8. Configure Minimum Capacity
          |
9. Configure Desired Capacity
          |
10. Configure Maximum Capacity
          |
11. Configure Scaling Policy
          |
12. Add Notifications
          |
13. Add Tags
          |
14. Review and Create ASG
          |
15. Verify EC2 Instances
          |
16. Verify Target Group Registration
          |
17. Monitor CloudWatch Metrics
          |
18. Test Scale Out
          |
19. Test Scale In
```

### Final Learning

**Load Balancer = Manages incoming traffic.**

**Auto Scaling = Manages EC2 instance capacity.**

**Launch Template = Defines how EC2 instances are created.**

**CloudWatch = Monitors workload metrics.**

**Scaling Policy = Determines when to scale.**

**Target Group = Connects Auto Scaling with ALB.**