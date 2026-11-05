# AWS EC2 Auto Scaling Group – Hands-on Practicals

## Objective

Create an EC2 Auto Scaling Group inside a custom VPC and integrate it with:

1. Classic Load Balancer (CLB)
2. Application Load Balancer (ALB)

Understand:
- Launch Template configuration
- Custom VPC and subnet selection
- Availability Zone selection
- Load Balancer integration
- Target Group integration
- Health checks
- Dynamic scaling policies
- Scale-out and scale-in testing
- Integration after ASG creation

---

# PART 1: Prerequisites – Custom VPC Setup

Before creating the Auto Scaling Group, prepare the networking infrastructure.

## Step 1: Create a Custom VPC

Navigate to:

AWS Console → VPC → Your VPCs → Create VPC

Example configuration:

| Configuration | Value |
|---|---|
| Name | devops-vpc |
| IPv4 CIDR | 10.0.0.0/16 |
| Tenancy | Default |

## Step 2: Create Subnets

Create four subnets across two Availability Zones.

| Subnet | CIDR | Availability Zone | Type |
|---|---|---|---|
| public-subnet-1 | 10.0.1.0/24 | us-west-2a | Public |
| public-subnet-2 | 10.0.2.0/24 | us-west-2b | Public |
| private-subnet-1 | 10.0.11.0/24 | us-west-2a | Private |
| private-subnet-2 | 10.0.12.0/24 | us-west-2b | Private |

Note:
- Public subnets need a route to an Internet Gateway.
- Private subnets do not have a direct route to an Internet Gateway.
- For private EC2 instances to download packages from the internet, configure a NAT Gateway or use suitable VPC endpoints.

## Step 3: Create an Internet Gateway

1. VPC → Internet Gateways.
2. Create Internet Gateway.
3. Name: devops-igw.
4. Attach it to devops-vpc.

## Step 4: Configure Public Route Table

Create or select a public route table.

Add:

| Destination | Target |
|---|---|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | Internet Gateway |

Associate:
- public-subnet-1
- public-subnet-2

## Step 5: Configure Private Route Table

Create a private route table.

For private internet access, add a NAT Gateway route if one is configured:

| Destination | Target |
|---|---|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | NAT Gateway |

Associate:
- private-subnet-1
- private-subnet-2

---

# PART 2: Create Security Groups

Navigate to:

EC2 → Security Groups → Create Security Group

## Security Group 1: ALB/CLB Security Group

Name: devops-lb-sg

VPC: devops-vpc

Inbound rules:

| Type | Port | Source |
|---|---|---|
| HTTP | 80 | 0.0.0.0/0 |
| HTTPS | 443 | 0.0.0.0/0 |

Outbound:
- Allow required outbound traffic to backend instances.

## Security Group 2: EC2 Security Group

Name: devops-web-sg

VPC: devops-vpc

Inbound rules:

| Type | Port | Source |
|---|---|---|
| HTTP | 80 | devops-lb-sg |
| SSH | 22 | Your IP address |

Important:
Use the load balancer security group as the source for HTTP traffic instead of allowing HTTP from everyone.

For the CLB practical, use the CLB security group as the source.
For the ALB practical, use the ALB security group as the source.

---

# PART 3: Create a Launch Template

Launch Template defines how Auto Scaling launches new EC2 instances.

Navigate to:

EC2 → Launch Templates → Create Launch Template

## Step 1: Basic configuration

| Configuration | Value |
|---|---|
| Launch Template Name | devops-web-lt |
| AMI | Amazon Linux 2023 |
| Instance Type | t2.micro or eligible free-tier type |
| Key Pair | Your existing key pair |
| Network | Do not select a subnet here |
| Security Group | devops-web-sg |

Important:
- Select the custom VPC security group.
- Do not select a subnet in the Launch Template.
- ASG determines which subnets and Availability Zones to use.

## Step 2: Configure User Data

Under Advanced Details → User Data:

#!/bin/bash
dnf install -y httpd
systemctl enable httpd
systemctl start httpd

TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
-H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

INSTANCE_ID=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" \
http://169.254.169.254/latest/meta-data/instance-id)

AZ=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" \
http://169.254.169.254/latest/meta-data/placement/availability-zone)

cat <<EOF > /var/www/html/index.html
<h1>AWS Auto Scaling Practical</h1>
<h2>Instance ID: $INSTANCE_ID</h2>
<h2>Availability Zone: $AZ</h2>
EOF

This script:
1. Installs Apache.
2. Starts Apache.
3. Enables Apache after reboot.
4. Creates an HTML page displaying the instance ID and AZ.

Save the Launch Template.

---

# PART 4: Practical 1 – CLB + Auto Scaling Group

## Architecture

Internet
|
v
Classic Load Balancer
|
v
Auto Scaling Group
|
+---- EC2 Instance 1
|
+---- EC2 Instance 2
|
+---- EC2 Instance 3 (during scale-out)

Important:
CLB directly registers EC2 instances.
ASG manages the EC2 instance capacity.

## Step 1: Create Classic Load Balancer

Navigate to:

EC2 → Load Balancers → Create Load Balancer

Select:

Classic Load Balancer

Configure:

| Configuration | Value |
|---|---|
| Name | devops-clb |
| Scheme | Internet-facing |
| VPC | devops-vpc |
| Subnets | public-subnet-1, public-subnet-2 |
| Security Group | devops-lb-sg |

Listener configuration:

| Load Balancer Port | Instance Port | Protocol |
|---|---|---|
| 80 | 80 | HTTP |

Health check configuration:

| Setting | Value |
|---|---|
| Ping Protocol | HTTP |
| Ping Port | 80 |
| Ping Path | / |
| Healthy Threshold | 2 |
| Unhealthy Threshold | 2 |
| Timeout | 5 seconds |
| Interval | 30 seconds |

For this practical, you can create the CLB without manually registering instances. ASG will register its instances after integration.

Create the CLB.

## Step 2: Create Auto Scaling Group

Navigate to:

EC2 → Auto Scaling Groups → Create Auto Scaling Group

### Stage 1: Choose Launch Template

| Configuration | Value |
|---|---|
| Name | devops-clb-asg |
| Launch Template | devops-web-lt |
| Version | Latest |

Click Next.

### Stage 2: Choose Instance Launch Options

Select:

| Configuration | Value |
|---|---|
| VPC | devops-vpc |
| Availability Zones | us-west-2a, us-west-2b |
| Subnets | private-subnet-1, private-subnet-2 |

Note:
The instances are in private subnets.
The CLB is internet-facing and uses public subnets.

Click Next.

### Stage 3: Configure Advanced Options

Under Load Balancing:

Select:

Attach to an existing load balancer.

Choose:

Classic Load Balancers.

Select:

devops-clb

Health check configuration:

| Setting | Value |
|---|---|
| Health Check Type | ELB |
| Health Check Grace Period | 180 seconds |

Important:
Selecting ELB health checks allows ASG to use the load balancer's health status when deciding whether to replace an unhealthy instance.

Click Next.

### Stage 4: Configure Group Size

| Setting | Value |
|---|---|
| Minimum Capacity | 2 |
| Desired Capacity | 2 |
| Maximum Capacity | 5 |

This means:
- Minimum 2 instances must always be maintained.
- ASG initially launches 2 instances.
- ASG can scale up to 5 instances.

### Stage 5: Configure Scaling Policies

Select:

Target Tracking Scaling Policy.

Example:

| Setting | Value |
|---|---|
| Policy Name | cpu-target-tracking |
| Metric | Average CPU Utilization |
| Target Value | 50% |
| Instance Warm-up | 120 seconds |

Note:
AWS manages the CloudWatch alarms associated with target tracking automatically.

### Stage 6: Add Notifications and Tags

Add a tag:

| Key | Value |
|---|---|
| Name | devops-clb-asg-instance |

Review the configuration and create the ASG.

## Step 3: Verify CLB Integration

Navigate to:

EC2 → Load Balancers → devops-clb

Check:
- Instances tab.
- Registered EC2 instances.
- Instance health status.

You should see the EC2 instances launched by ASG.

Navigate to:

EC2 → Auto Scaling Groups → devops-clb-asg → Activity

Check:
- Launching EC2 instances.
- Successful launch activities.

## Step 4: Test CLB

Copy the CLB DNS name.

Open it in your browser:

http://CLB-DNS-NAME

Refresh multiple times.

Observe:
- Requests are distributed across registered EC2 instances.
- The HTML page displays different instance IDs when requests reach different instances.

---

# PART 5: Practical 2 – ALB + Auto Scaling Group

## Architecture

Internet
|
v
Application Load Balancer
|
v
Listener (HTTP:80)
|
v
Target Group
|
v
Auto Scaling Group
|
+---- EC2 Instance 1
|
+---- EC2 Instance 2
|
+---- EC2 Instance 3 (during scale-out)

Important:
ASG does not attach directly to the ALB.
ASG attaches to the ALB Target Group.

## Step 1: Create Target Group

Navigate to:

EC2 → Target Groups → Create Target Group

Configure:

| Configuration | Value |
|---|---|
| Target Type | Instances |
| Target Group Name | devops-alb-tg |
| Protocol | HTTP |
| Port | 80 |
| VPC | devops-vpc |
| Protocol Version | HTTP1 |

Health Check:

| Setting | Value |
|---|---|
| Protocol | HTTP |
| Path | / |
| Healthy Threshold | 2 |
| Unhealthy Threshold | 2 |
| Timeout | 5 seconds |
| Interval | 30 seconds |

Do not manually register instances for this practical.

Create Target Group.

## Step 2: Create Application Load Balancer

Navigate to:

EC2 → Load Balancers → Create Load Balancer

Select:

Application Load Balancer.

Configure:

| Configuration | Value |
|---|---|
| Name | devops-alb |
| Scheme | Internet-facing |
| IP Address Type | IPv4 |
| VPC | devops-vpc |
| Subnets | public-subnet-1, public-subnet-2 |
| Security Group | devops-lb-sg |

Listener:

| Protocol | Port | Default Action |
|---|---|---|
| HTTP | 80 | Forward to devops-alb-tg |

Create ALB.

Important:
- Select public subnets in at least two Availability Zones for this internet-facing ALB.
- Backend EC2 instances can remain in private subnets.
- The Target Group and ALB must belong to the same VPC.

## Step 3: Create ALB Auto Scaling Group

Navigate to:

EC2 → Auto Scaling Groups → Create Auto Scaling Group

### Stage 1: Choose Launch Template

| Setting | Value |
|---|---|
| Name | devops-alb-asg |
| Launch Template | devops-web-lt |
| Version | Latest |

Click Next.

### Stage 2: Network Configuration

Select:

| Setting | Value |
|---|---|
| VPC | devops-vpc |
| Availability Zones | us-west-2a, us-west-2b |
| Subnets | private-subnet-1, private-subnet-2 |

Click Next.

### Stage 3: Load Balancing Integration

Under Load Balancing:

Select:

Attach to an existing load balancer.

Choose:

Choose from your existing load balancer target groups.

Select:

devops-alb-tg

Health Check:

| Setting | Value |
|---|---|
| Health Check Type | ELB |
| Health Check Grace Period | 180 seconds |

Click Next.

### Stage 4: Group Size

| Setting | Value |
|---|---|
| Minimum | 2 |
| Desired | 2 |
| Maximum | 5 |

### Stage 5: Scaling Policy

Select:

Target Tracking Scaling Policy.

Configure:

| Setting | Value |
|---|---|
| Metric | Average CPU Utilization |
| Target | 50% |
| Warm-up | 120 seconds |

Click Next.

Add tags if required and create the ASG.

## Step 4: Verify ALB Integration

Navigate to:

EC2 → Auto Scaling Groups → devops-alb-asg

Check:
- Details → Load balancing.
- Target Group attachment.
- Activity history.
- Instance management.

Now navigate to:

EC2 → Target Groups → devops-alb-tg → Targets

You should see:
- Two registered EC2 instances.
- Health status initially in the initial state.
- Later, healthy status after successful health checks.

## Step 5: Test ALB

Copy the ALB DNS name.

Open:

http://ALB-DNS-NAME

Refresh the page.

You should see the instance ID and AZ of the EC2 instance handling the request.

---

# PART 6: Attach Load Balancer After Creating ASG

This is an important additional practical.

You can create an ASG first without attaching any load balancer and integrate it later.

## Scenario A: Attach CLB After ASG Creation

1. Navigate to EC2.
2. Select Auto Scaling Groups.
3. Select your ASG.
4. Open Details.
5. Find Load balancing.
6. Select Edit.
7. Find Classic Load Balancers.
8. Select the existing CLB.
9. Save changes.

Now ASG will manage registration and deregistration of its instances with the CLB.

Verify:

EC2 → Load Balancers → CLB → Instances.

## Scenario B: Attach ALB Target Group After ASG Creation

1. Navigate to EC2.
2. Select Auto Scaling Groups.
3. Select your ASG.
4. Open Details.
5. Find Load balancing.
6. Select Edit.
7. Under Existing load balancer target groups, select the ALB Target Group.
8. Save changes.

Verify:

EC2 → Target Groups → Your Target Group → Targets.

The ASG will register its instances with the Target Group.

Important:
You attach the Target Group, not the ALB itself.

---

# PART 7: Modify ASG Integration After Creation

You can modify these settings without deleting the ASG.

Navigate to:

EC2 → Auto Scaling Groups → Select ASG.

## Available modifications

| Setting | Can Modify? |
|---|---|
| Minimum Capacity | Yes |
| Desired Capacity | Yes |
| Maximum Capacity | Yes |
| Scaling Policy | Yes |
| Load Balancer Integration | Yes |
| Target Group Attachment | Yes |
| Health Check Type | Yes |
| Health Check Grace Period | Yes |
| Subnets | Yes |
| Launch Template Version | Yes |

Important:
Changing the Launch Template version does not automatically replace existing EC2 instances. You may need an Instance Refresh to roll out the new configuration.

---

# PART 8: Test Scale-Out

Objective: Increase CPU utilization and observe ASG launching additional instances.

## Step 1: Check Current ASG Capacity

Example:

Minimum = 2
Desired = 2
Maximum = 5

Initially, two EC2 instances should be running.

## Step 2: Connect to EC2

Connect to an ASG-managed EC2 instance.

Install stress:

Amazon Linux 2023:

sudo dnf install stress -y

If the package is unavailable, use an appropriate stress-ng package or another approved load-testing method.

Check available CPUs:

nproc

## Step 3: Generate CPU Load

Run:

stress --cpu 2 --timeout 600

This generates CPU load for 10 minutes using two CPU workers.

## Step 4: Monitor CloudWatch

Navigate to:

CloudWatch → Metrics → EC2 → By Auto Scaling Group.

Select CPUUtilization.

Observe the average CPU utilization.

## Step 5: Monitor ASG

Navigate to:

EC2 → Auto Scaling Groups → Your ASG → Activity.

Observe:
- Launching EC2 instance.
- Successful launch.
- Desired capacity increasing.

Check Target Group:
- New EC2 instance gets registered.
- Health check runs.
- Target eventually becomes healthy.

---

# PART 9: Test Scale-In

Objective: Stop CPU stress and observe ASG reducing capacity.

## Step 1: Stop Stress

If running in the foreground:

Ctrl + C

If running in the background:

sudo pkill stress

Verify:

pgrep -a stress

## Step 2: Monitor CPU

Use:

top

CPU utilization should decrease after stress stops.

## Step 3: Monitor CloudWatch

Check the ASG average CPUUtilization metric.

## Step 4: Monitor ASG Activity

Navigate to:

EC2 → Auto Scaling Groups → Your ASG → Activity.

Observe:
- Terminating EC2 instance.
- Successful termination.

## Step 5: Verify Target Group

Navigate to:

EC2 → Target Groups → Your Target Group → Targets.

The terminating instance is deregistered, and remaining healthy instances continue serving requests.

Important:
Scale-in is not immediate. It depends on metric evaluation, scaling policy behavior, instance warm-up and other scaling controls.

ASG should eventually return to the desired/minimum capacity of 2 when the policy decides to scale in.

---

# PART 10: CLB vs ALB Integration with ASG

| Feature | CLB | ALB |
|---|---|---|
| Load Balancer Type | Classic | Application |
| Layer | L4 and L7 | L7 |
| Integration with ASG | Attach CLB | Attach Target Group |
| Target Group Required | No | Yes |
| Backend Registration | Direct instance registration | Target Group registration |
| Path-based Routing | No | Yes |
| Host-based Routing | No | Yes |
| Health Checks | CLB health checks | Target Group health checks |
| Scale-out | ASG launches and registers instances | ASG launches and registers instances |
| Scale-in | ASG deregisters and terminates instances | ASG deregisters and terminates instances |

---

# PART 11: Important Interview Questions

Q1. Can we create an ASG without a Load Balancer?

Answer:
Yes. An ASG can independently manage EC2 capacity without any load balancer.

Q2. Can we attach a Load Balancer after creating an ASG?

Answer:
Yes. We can edit the ASG configuration and attach a CLB or an ALB Target Group.

Q3. Does ASG attach directly to ALB?

Answer:
No. ASG attaches to the ALB Target Group. The ALB listener forwards requests to that Target Group.

Q4. How does ASG know when an EC2 instance is unhealthy?

Answer:
ASG can use EC2 health checks or ELB health checks. With ELB health checks enabled, an unhealthy registered target can trigger instance replacement.

Q5. What happens when ASG scales out?

Answer:
ASG launches a new EC2 instance using the Launch Template. It registers the instance with the associated load balancer or Target Group after launch.

Q6. What happens when ASG scales in?

Answer:
ASG selects an instance according to its termination policy, deregisters it from the load balancer, and terminates it. Existing healthy instances continue serving traffic.

Q7. Can we attach multiple Target Groups to one ASG?

Answer:
Yes. An ASG can be associated with multiple Target Groups, provided the configuration is supported and appropriate for the application.

Q8. Can one ASG launch EC2 instances in multiple Availability Zones?

Answer:
Yes. Select subnets in multiple Availability Zones while configuring the ASG.

Q9. What is the difference between Launch Template and ASG?

Answer:
Launch Template defines how an EC2 instance is launched.
ASG defines how many instances should run and where they should run, and manages scaling and replacement.

Q10. Does ALB launch EC2 instances?

Answer:
No. ALB distributes incoming traffic. ASG manages EC2 instance capacity.

---

# Final Practical Verification Checklist

## CLB Practical

- [ ] Custom VPC created.
- [ ] Public and private subnets created.
- [ ] Internet Gateway and route tables configured.
- [ ] Security Groups configured.
- [ ] Launch Template created.
- [ ] User Data configured.
- [ ] Classic Load Balancer created.
- [ ] ASG created.
- [ ] CLB attached to ASG.
- [ ] Two EC2 instances launched.
- [ ] Instances registered with CLB.
- [ ] Health checks passed.
- [ ] Browser access verified.
- [ ] Scale-out tested.
- [ ] Scale-in tested.

## ALB Practical

- [ ] Custom VPC created.
- [ ] Public and private subnets created.
- [ ] Internet Gateway and route tables configured.
- [ ] Security Groups configured.
- [ ] Launch Template created.
- [ ] Target Group created.
- [ ] ALB created.
- [ ] Listener configured.
- [ ] Listener forwards to Target Group.
- [ ] ASG created.
- [ ] Target Group attached to ASG.
- [ ] Two EC2 instances launched.
- [ ] Instances registered with Target Group.
- [ ] Health checks passed.
- [ ] Browser access verified.
- [ ] Scale-out tested.
- [ ] Scale-in tested.

## Important Architecture Difference

CLB:

ASG → Classic Load Balancer

ALB:

ASG → Target Group ← ALB Listener ← ALB

Remember:

**ASG manages compute capacity. Load Balancer manages incoming traffic. CloudWatch provides metrics used by scaling policies.**