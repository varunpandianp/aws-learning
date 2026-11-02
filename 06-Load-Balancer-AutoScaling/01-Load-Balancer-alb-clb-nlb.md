# AWS Elastic Load Balancing (ELB)

## 1. What is Load Balancing?

A Load Balancer is an AWS service that distributes incoming traffic across multiple backend resources such as EC2 instances.

### Main purposes
- Distribute incoming traffic across multiple EC2 instances.
- Prevent a single server from becoming overloaded.
- Improve application availability.
- Perform health checks on backend instances.
- Route traffic to healthy targets.
- Support high availability across multiple Availability Zones.
- Improve application scalability.

### Basic Architecture

                  Users
                    |
                    v
              Load Balancer
                    |
          ---------------------
          |         |         |
          v         v         v
        EC2-1     EC2-2     EC2-3
        AZ-A      AZ-B      AZ-C


## 2. Types of AWS Load Balancers

AWS Elastic Load Balancing provides four main types:

1. Classic Load Balancer (CLB)
2. Application Load Balancer (ALB)
3. Network Load Balancer (NLB)
4. Gateway Load Balancer (GWLB)

| Load Balancer | OSI Layer | Main Purpose |
|---|---|---|
| CLB | Layer 4 and 7 | Legacy general-purpose load balancing |
| ALB | Layer 7 | HTTP/HTTPS application traffic |
| NLB | Layer 4 | TCP, UDP and TLS traffic |
| GWLB | Layer 3 | Network traffic inspection and security appliances |


# 3. Application Load Balancer (ALB)

OSI Layer: Layer 7 – Application Layer

ALB operates at Layer 7 of the OSI model.

It understands application-level information such as:
- HTTP
- HTTPS
- URL paths
- Hostnames
- HTTP headers
- Query strings

Main advantage: ALB supports intelligent routing based on application requests.

## A. Path-Based Routing

ALB can distribute requests to different Target Groups based on URL paths.

Example:

                  User
                    |
                    v
             Application LB
                    |
               Listener : 80
                    |
          ---------------------
          |         |         |
          v         v         v
        /web/*    /api/*   /admin/*
          |         |         |
          v         v         v
        Web TG    API TG   Admin TG
          |         |         |
          v         v         v
         EC2       EC2       EC2

Example routing:

| Incoming URL | Target Group |
|---|---|
| example.com/web/ | Web TG |
| example.com/api/users | API TG |
| example.com/admin/login | Admin TG |

The ALB listener evaluates the incoming request and forwards it to the matching Target Group.

## B. Host-Based Routing

ALB can route traffic based on domain names.

Example:

| Domain | Target Group |
|---|---|
| www.example.com | Frontend TG |
| api.example.com | API TG |
| admin.example.com | Admin TG |

## C. Other ALB Capabilities

- Supports HTTP and HTTPS.
- Path-based routing.
- Host-based routing.
- Header-based routing.
- Query-string routing.
- WebSocket support.
- Authentication integration.
- Integration with Auto Scaling Groups.
- Health checks through Target Groups.
- Multiple Target Groups behind one ALB.

## D. ALB Architecture

                 Internet
                    |
                    v
             Application LB
                    |
               Listener 443
                    |
              Listener Rules
                    |
          ---------------------
          |                   |
        /api/*              /web/*
          |                   |
          v                   v
        API TG              Web TG
          |                   |
          v                   v
        API ASG             Web ASG
          |                   |
         EC2s                EC2s

## E. ALB Advantages

1. Intelligent traffic distribution.
2. Supports microservices architecture.
3. One ALB can handle multiple applications.
4. Supports path-based and host-based routing.
5. Works with Auto Scaling Groups.
6. Automatically registers and deregisters EC2 instances through Target Groups.
7. Supports HTTPS termination using SSL/TLS certificates through ACM.
8. Supports WebSocket connections.
9. Supports multiple Availability Zones.

## F. ALB Limitations

- Does not support UDP load balancing.
- Does not support arbitrary TCP application load balancing.
- Does not provide static IP addresses for the load balancer itself.
- Requires listener and Target Group configuration.

## G. When to Use ALB?

- Web applications.
- E-commerce websites.
- Microservices.
- REST APIs.
- HTTP/HTTPS applications.
- Container-based applications.

Real-world example:

An e-commerce application has frontend, backend API and admin services.

One ALB can route traffic to all three services using listener rules.


# 4. Network Load Balancer (NLB)

OSI Layer: Layer 4 – Transport Layer

NLB operates at Layer 4 of the OSI model.

It distributes traffic based on:
- Source IP address.
- Destination IP address.
- Port number.
- Network protocol (TCP/UDP).

Unlike ALB, NLB does not inspect HTTP URL paths or hostnames to make routing decisions.

## A. How NLB Works

Example:

                 Users
                   |
                   v
              Network LB
                   |
           ----------------
           |              |
        Port 80         Port 443
           |              |
           v              v
         EC2-1          EC2-2

NLB uses listeners to accept connections on specific ports and forwards them to registered targets.

## B. NLB Supported Protocols

| Protocol | Purpose |
|---|---|
| TCP | Reliable connection-based communication |
| UDP | Connectionless communication |
| TLS | Encrypted transport-layer connections |
| TCP_UDP | Supports TCP and UDP on the same port |

## C. Important NLB Characteristics

1. Operates at Layer 4.
2. Supports TCP, UDP and TLS traffic.
3. Supports static IP addresses.
4. Can associate Elastic IP addresses with enabled Availability Zones.
5. Preserves client source IP in common target configurations.
6. Handles high volumes of connections with low latency.
7. Supports EC2 instances, IP addresses and other supported target types.
8. Supports Auto Scaling Group integration through Target Groups.
9. Supports cross-zone load balancing.
10. Supports health checks.

## D. Important Difference: ALB vs NLB Routing

Suppose you have:

https://example.com/api/users
https://example.com/admin/login

ALB:
- Reads the HTTP request.
- Identifies /api/*.
- Routes to API Target Group.
- Identifies /admin/*.
- Routes to Admin Target Group.

NLB:
- Does not inspect these URL paths for routing.
- Routes based on listener protocol and port.
- Example: TCP port 80 → Target Group 1.
- Example: TCP port 443 → Target Group 2.

## E. NLB Health Check Example

Imagine:
- EC2 instance is running.
- Port 80 is accepting TCP connections.
- Apache application has an internal error.

If NLB uses a TCP health check, the instance can still be considered healthy because the TCP connection succeeds.

If you configure an HTTP health check for a specific endpoint, NLB can check the HTTP response as well.

This is an important distinction for troubleshooting.

## F. NLB Advantages

- Very high performance.
- Low latency.
- Static IP support.
- Supports TCP and UDP.
- Suitable for gaming applications.
- Suitable for real-time communication.
- Supports high-throughput network applications.
- Can handle traffic without requiring application-level routing.

## G. NLB Limitations

- No path-based routing.
- No host-based routing.
- No HTTP header-based routing.
- No application-level routing decisions.
- Not designed for URL-based microservice routing.

## H. When to Use NLB?

- Gaming applications.
- TCP/UDP applications.
- Real-time communication.
- High-performance networking.
- Applications requiring static IP addresses.
- Applications requiring low latency.


# 5. Classic Load Balancer (CLB)

Legacy Load Balancer

Classic Load Balancer is the older generation of AWS Load Balancing.

It supports both:
- Layer 4 (TCP/SSL)
- Layer 7 (HTTP/HTTPS)

CLB is less flexible than ALB and NLB.

## A. How Classic Load Balancer Works

The important concept in CLB is:

Load Balancer Listener Port vs Instance Port.

Example:

                 User
                   |
                   v
              Classic LB
                   |
             Listener Port 80
                   |
             Instance Port 80
                   |
           ------------------
           |                |
           v                v
         EC2-1            EC2-2
         Port 80          Port 80

## B. Listener Configuration

Terraform example:

listener {
instance_port     = 80
instance_protocol = "http"
lb_port           = 80
lb_protocol       = "http"
}

Understanding each field:

| Attribute | Meaning |
|---|---|
| lb_port | Port on which the load balancer receives traffic |
| lb_protocol | Protocol accepted by the load balancer |
| instance_port | Backend EC2 port |
| instance_protocol | Protocol used between LB and EC2 |

## C. Different Listener and Instance Ports

Example:

listener {
instance_port     = 8080
instance_protocol = "http"
lb_port           = 80
lb_protocol       = "http"
}

Flow:

User
|
v
Classic LB : 80
|
v
EC2 : 8080

Here, the user accesses port 80, but the backend application listens on port 8080.

## D. Classic Load Balancer Features

- Supports HTTP and HTTPS.
- Supports TCP and SSL.
- Health checks.
- Connection draining.
- Cross-zone load balancing.
- Idle timeout configuration.
- Direct EC2 instance registration.
- Basic traffic distribution.
- Can integrate with Auto Scaling Groups.

## E. CLB Limitations

- No path-based routing.
- No host-based routing.
- No modern listener rule functionality.
- No Target Group architecture.
- Limited routing capabilities compared with ALB.
- Legacy architecture; AWS recommends newer load balancer types for new applications.

## F. CLB Advantages

- Simple configuration.
- Useful for legacy applications.
- Supports Layer 4 and basic Layer 7 load balancing.
- Can directly attach EC2 instances.
- Supports health checks and connection draining.

## G. When to Use CLB?

Primarily when maintaining existing legacy applications that already depend on Classic Load Balancer.


# 6. Gateway Load Balancer (GWLB)

OSI Layer: Layer 3 – Network Layer

Gateway Load Balancer operates at Layer 3 of the OSI model.

Its primary purpose is to deploy, scale and manage third-party virtual network appliances.

Examples:
- Firewalls.
- Intrusion Detection Systems (IDS).
- Intrusion Prevention Systems (IPS).
- Deep Packet Inspection (DPI).
- Network security appliances.

## A. How GWLB Works

Imagine your organization has EC2 instances in a private subnet.

All incoming and outgoing network traffic must pass through a firewall for inspection.

Instead of manually managing multiple firewall instances, GWLB distributes traffic across firewall appliances.

## B. Architecture

              Internet
                  |
                  v
             Internet Gateway
                  |
                  v
          Gateway Load Balancer
                  |
           -------------------
           |                 |
           v                 v
        Firewall 1        Firewall 2
           |                 |
           -------------------
                  |
                  v
              Application
                  EC2

## C. Important GWLB Concepts

- Operates at Layer 3.
- Supports transparent network traffic inspection.
- Uses GENEVE encapsulation on UDP port 6081.
- Distributes traffic across virtual appliances.
- Supports health checks.
- Supports appliance scaling.
- Uses Gateway Load Balancer Endpoints (GWLBE) for traffic insertion into VPC routing.

## D. GWLB Advantages

1. Centralized security inspection.
2. Automatic scaling of security appliances.
3. High availability.
4. Simplifies firewall management.
5. Supports third-party security solutions.
6. Reduces the need to manage individual appliance traffic distribution.

## E. When to Use GWLB?

When you need to inspect network traffic using firewalls or third-party security appliances.


# 7. Complete Comparison – All Four Load Balancers

| Feature | CLB | ALB | NLB | GWLB |
|---|---|---|---|---|
| Full name | Classic LB | Application LB | Network LB | Gateway LB |
| OSI layer | 4 and 7 | 7 | 4 | 3 |
| Main purpose | Legacy applications | Web applications | Network traffic | Security inspection |
| HTTP | Yes | Yes | Through TCP listener | Not an HTTP router |
| HTTPS | Yes | Yes | TLS listener | Not an HTTPS router |
| TCP | Yes | No native TCP listener | Yes | IP traffic |
| UDP | No | No | Yes | IP traffic |
| Path routing | No | Yes | No | No |
| Host routing | No | Yes | No | No |
| Static IP | No fixed static IP | No fixed static IP | Yes | Via endpoint architecture |
| Target Groups | No | Yes | Yes | Yes |
| Auto Scaling integration | Yes | Yes | Yes | Appliance targets |
| Health checks | Yes | Yes | Yes | Yes |
| Primary use | Legacy systems | Microservices | High-performance apps | Firewalls and IDS/IPS |


# 8. Routing Methods – Important Differences

| Load Balancer | Routing Method | Example |
|---|---|---|
| ALB | Path-based | /api/*, /admin/* |
| ALB | Host-based | api.example.com |
| ALB | Header-based | X-App-Version: v2 |
| NLB | Port-based | TCP 80, TCP 443, UDP 53 |
| CLB | Listener-based | LB port 80 → EC2 port 8080 |
| GWLB | Network traffic steering | Route traffic through firewall appliances |

Important:

NLB does not support multipath routing in the URL-based sense. It can have multiple listeners on different ports and forward traffic to their respective Target Groups.


# 9. Terraform Resource Mapping

| AWS Service | Terraform Resource |
|---|---|
| Classic Load Balancer | aws_elb |
| Application Load Balancer | aws_lb |
| Network Load Balancer | aws_lb |
| Gateway Load Balancer | aws_lb |
| Target Group | aws_lb_target_group |
| Listener | aws_lb_listener |
| Listener Rule | aws_lb_listener_rule |
| Auto Scaling Group | aws_autoscaling_group |
| Launch Template | aws_launch_template |

Example: Application Load Balancer

resource "aws_lb" "app_alb" {
name               = "my-app-alb"
load_balancer_type = "application"
}

Example: Network Load Balancer

resource "aws_lb" "network_lb" {
name               = "my-network-lb"
load_balancer_type = "network"
}

Example: Gateway Load Balancer

resource "aws_lb" "gateway_lb" {
name               = "my-gateway-lb"
load_balancer_type = "gateway"
}

Notice:

ALB, NLB and GWLB all use the Terraform resource aws_lb.

The load_balancer_type determines which type AWS creates.


# 10. How Load Balancer Connects with Auto Scaling

This is an important concept in AWS DevOps.

Architecture:

                Users
                  |
                  v
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
          -------------------
          |                 |
          v                 v
        EC2-1             EC2-2
                            |
                            v
                          EC2-3
                      (Scale Out)

## How They Work Together

1. Launch Template defines the EC2 configuration.
2. Auto Scaling Group launches EC2 instances.
3. ASG automatically registers instances with the Target Group.
4. ALB listener forwards traffic to the Target Group.
5. Target Group performs health checks.
6. ASG can replace unhealthy instances based on its health-check configuration.

## Remember These Three Relationships

- ALB → Target Group: Routes application traffic.
- ASG → Target Group: Registers and deregisters instances.
- Launch Template → ASG: Defines how EC2 instances are launched.

Important Terraform configuration:

target_group_arns = [aws_lb_target_group.web_tg.arn]

This connects the Auto Scaling Group to the Target Group.

When ASG launches new EC2 instances, they are automatically registered with the Target Group.

When ASG terminates instances, they are deregistered.


# 11. Interview Questions – Beginner to Intermediate

### Q1. What is the difference between ALB and NLB?

ALB operates at Layer 7 and supports application-level routing, such as path-based and host-based routing.

NLB operates at Layer 4 and distributes network connections based on protocol and port.

### Q2. Can NLB perform path-based routing?

No. NLB does not inspect HTTP URL paths. It routes connections based on listener protocols and ports.

### Q3. Why do we use Target Groups in ALB?

Target Groups define the backend destinations where ALB forwards requests. They also manage target health checks.

### Q4. Can one ALB have multiple Target Groups?

Yes. One ALB can have multiple Target Groups, and listener rules can route requests to different Target Groups.

### Q5. What is the difference between CLB and ALB?

CLB directly registers EC2 instances and supports basic load balancing.

ALB uses Target Groups and supports advanced Layer 7 routing.

### Q6. Which Load Balancer is suitable for gaming applications?

NLB is generally suitable because it supports TCP and UDP traffic with high performance and low latency.

### Q7. Which Load Balancer is suitable for firewall inspection?

GWLB, because it distributes network traffic across virtual security appliances.

### Q8. Can NLB work if the application is not functioning properly?

Yes, depending on the health check.

For example, a TCP health check may pass if the port is accepting connections, even when the application itself has an internal error.

### Q9. Does Auto Scaling Group attach directly to ALB?

No. ASG attaches to the Target Group, and the ALB listener forwards traffic to that Target Group.

### Q10. What does instance_port mean in Classic Load Balancer?

It is the backend port on which the EC2 application receives traffic from the Classic Load Balancer.


# 12. Quick Revision

| Load Balancer | Layer | Remember |
|---|---|---|
| ALB | Layer 7 | HTTP/HTTPS, path-based routing, host-based routing, microservices |
| NLB | Layer 4 | TCP, UDP, TLS, static IP, low latency |
| CLB | Layer 4 and 7 | Legacy, listener port, instance port, direct EC2 registration |
| GWLB | Layer 3 | Firewall, IDS/IPS, security appliances, GENEVE |

## My Practical Learning Recommendation

1. Practise ALB + Target Group + ASG first.
2. Understand path-based routing.
3. Understand host-based routing.
4. Practise NLB with TCP and UDP listeners.
5. Understand CLB because you may encounter it in legacy infrastructure.
6. Understand GWLB architecture and security appliance integration.