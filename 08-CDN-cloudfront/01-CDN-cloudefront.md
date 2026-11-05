# AWS CDN – Content Delivery Network | Amazon CloudFront

## 1. What is CDN?

CDN stands for **Content Delivery Network**.

A CDN is a globally distributed network of servers that delivers website content to users from locations geographically closer to them.

Examples of content delivered through CDN:
- Images
- CSS files
- JavaScript files
- HTML pages
- Videos
- Static assets

### Without CDN

Imagine our application is hosted in AWS US West (Oregon), but a user accesses it from India.

Request flow:

1. User in India sends a request to the application.
2. Request travels to the US West region.
3. Load Balancer receives the request.
4. Load Balancer forwards the request to an EC2 instance.
5. EC2 processes the request and returns the response to India.

Problems:
- Higher latency for geographically distant users.
- More requests reach the origin server.
- Increased backend server load.
- Higher origin bandwidth consumption.

### With CDN

1. User sends a request to CloudFront.
2. CloudFront routes the request to a suitable Edge Location.
3. If content is available in the cache, CloudFront returns it directly.
4. If content is not cached, CloudFront fetches it from the origin.
5. Eligible content is cached and served to subsequent users.

Benefits:
- Reduced latency.
- Faster content delivery.
- Reduced origin-server traffic.
- Reduced backend load for cacheable content.
- Improved global application performance.

---

## 2. What is Amazon CloudFront?

Amazon CloudFront is an AWS-managed CDN service.

It delivers content globally using AWS Edge Locations and Regional Edge Caches.

Important CloudFront terminology:

| Term | Explanation |
|---|---|
| CDN | Content Delivery Network |
| CloudFront | AWS CDN service |
| Edge Location | Location where CloudFront serves cached content close to users |
| Regional Edge Cache | Additional caching layer between Edge Locations and origins |
| Origin | Original source of content, such as ALB, CLB, EC2 or S3 |
| Distribution | CloudFront configuration that defines origins and caching behaviors |
| Cache Hit | Requested content is available in cache |
| Cache Miss | Requested content is not available in cache |
| Cache Invalidation | Removes specified objects from CloudFront caches |

### What is an Edge Location?

An Edge Location is a CloudFront point of presence where content can be cached and delivered to nearby users.

Example:

- Application origin: AWS US West.
- User A: India.
- User B: Singapore.
- User C: United States.

CloudFront can serve these users through suitable nearby Edge Locations instead of making every request travel all the way to the origin.

Important:
- Edge Locations are operated by AWS.
- We do not create or manage EC2 instances at Edge Locations.
- CloudFront automatically routes requests to suitable locations.

---

## 3. Why Do We Need CDN?

Without CDN:

- Every request for uncached content reaches the origin.
- Users far away from the origin may experience higher latency.
- Origin servers process more requests.
- Backend infrastructure may need more capacity.

With CDN:

- Cacheable content is served from nearby Edge Locations.
- Repeated requests can avoid reaching the origin.
- Backend traffic and bandwidth consumption can be reduced.
- Users receive content faster.

---

## 4. CloudFront Cache Hit vs Cache Miss

### Cache Hit

A Cache Hit happens when CloudFront already has a valid copy of the requested object.

Request flow:

```text
User
  |
  v
CloudFront Edge Location
  |
  v
Content available in cache
  |
  v
Returns content to user

Origin is not contacted for this request.
```

### Cache Miss

A Cache Miss happens when CloudFront does not have a valid cached copy of the requested object.

Request flow:

```text
User
  |
  v
CloudFront Edge Location
  |
  v
Content NOT available in cache
  |
  v
ALB (Origin)
  |
  v
EC2
  |
  v
Application processes request
  |
  v
Response returned to CloudFront
  |
  v
Eligible content cached
  |
  v
Response delivered to user
```

Important:
A Cache Miss does not guarantee that the object will be cached. Caching depends on the cache policy, TTL and origin response headers.

---

## 5. CDN Architecture – Where Does CloudFront Fit?

Our application architecture:

```text
                    USERS
                      |
                      v
                ROUTE 53 (DNS)
                   Optional
                      |
                      v
              CLOUDFRONT (CDN)
                      |
                Cache Hit / Miss
                      |
               Cache Miss only
                      |
                      v
           APPLICATION LOAD BALANCER
                      |
                      v
                 TARGET GROUP
                      |
                      v
             AUTO SCALING GROUP
                      |
                +-----+-----+
                |           |
                v           v
              EC2-1       EC2-2
                |           |
                +-----+-----+
                      |
                 APPLICATION
```

### Responsibility of Each AWS Service

| Service | Responsibility |
|---|---|
| Route 53 | DNS resolution |
| CloudFront | Global content delivery and caching |
| ALB | Distributes application requests |
| Target Group | Identifies backend targets |
| ASG | Manages EC2 instance capacity |
| EC2 | Runs the application |

Remember:

- CloudFront comes before ALB.
- ALB comes before backend EC2 instances.
- ASG manages EC2 instances.
- Route 53 is optional for the basic CloudFront practical.
- CloudFront does not replace ALB.

---

# 6. Hands-on Practical – Create CloudFront Distribution with ALB

## Objective

Integrate an existing Application Load Balancer with Amazon CloudFront.

### Prerequisites

We already have:

- Custom VPC.
- EC2 instances.
- Auto Scaling Group.
- Application Load Balancer.
- ALB Target Group.
- Working website accessible through ALB DNS.

Architecture:

```text
USER
  |
  v
CLOUDFRONT
  |
  v
ALB (ORIGIN)
  |
  v
TARGET GROUP
  |
  v
AUTO SCALING GROUP
  |
  v
EC2 INSTANCES
  |
  v
APPLICATION
```

## Step 1: Open CloudFront

Navigate to:

AWS Console → CloudFront → Distributions → Create Distribution.

Select the distribution type:

Single website or app (if shown in the console).

Enter a distribution name:

```text
devops-cloudfront-cdn
```

## Step 2: Select Origin Type

Choose:

Elastic Load Balancing.

Select your existing Application Load Balancer.

Example:

```text
devops-alb
```

CloudFront uses the ALB DNS name as its origin.

Remember:
- Origin means the original source from which CloudFront retrieves content.
- Here, our origin is the ALB.
- ALB forwards requests to its Target Group.

## Step 3: Configure Origin Settings

For a custom origin such as an ALB, select the origin connection protocol.

Available options:

| Setting | Explanation |
|---|---|
| HTTP only | CloudFront connects to ALB using HTTP |
| HTTPS only | CloudFront connects to ALB using HTTPS |
| Match viewer | CloudFront uses the viewer's protocol |

For a basic lab where the ALB accepts HTTP on port 80:

```text
Origin protocol policy: HTTP only
```

For production, prefer HTTPS between CloudFront and ALB.

Important:
For HTTPS to the origin, the ALB must present a valid certificate matching the origin hostname. If using the ALB's default AWS DNS name, a certificate for your own domain will not automatically match it. Use a suitable custom origin hostname and certificate.

## Step 4: Configure Cache Settings

Select the cache policy:

```text
CachingOptimized
```

CachingOptimized is an AWS-managed CloudFront cache policy suitable for common static content.

It:
- Allows caching of eligible content.
- Uses a cache key that excludes cookies and query strings.
- Helps improve cache efficiency.

## Step 5: Configure Origin Request Policy

Select:

```text
AllViewer
```

This forwards viewer request headers, cookies and query strings to the origin.

Important difference:

| Policy | Responsibility |
|---|---|
| Cache Policy | Controls cache key and TTL behavior |
| Origin Request Policy | Controls information forwarded to the origin |

Important production consideration:

CachingOptimized + AllViewer is not automatically suitable for every application.

If the application returns personalized or dynamic responses, caching with an incomplete cache key can expose one user's response to another user.

For dynamic applications, consider:

```text
Cache Policy: CachingDisabled
Origin Request Policy: Appropriate viewer values
```

For static assets, CachingOptimized is usually a good starting point.

## Step 6: Configure Viewer Protocol Policy

Select:

```text
Redirect HTTP to HTTPS
```

This means:
- Users can initially send HTTP requests.
- CloudFront redirects HTTP requests to HTTPS.
- Users access the CloudFront distribution securely.

Remember:
Viewer-to-CloudFront HTTPS and CloudFront-to-origin HTTPS are separate connections.

## Step 7: Create Distribution

Review the configuration.

Select:

```text
Create Distribution
```

CloudFront starts deploying the distribution configuration globally.

Wait until:

```text
Distribution Status: Enabled
Deployment Status: Deployed
```

## Step 8: Test CloudFront

Navigate to:

CloudFront → Distributions → Your Distribution.

Copy the distribution domain name.

Example:

```text
https://d123456789.cloudfront.net
```

Open it in your browser.

Your request reaches CloudFront. When the cache misses, CloudFront contacts the ALB origin, which forwards the request to a healthy backend instance.

---

# 7. Verify CloudFront Caching

Open your browser:

1. Press F12.
2. Select Network.
3. Refresh the website.
4. Open the requested resource.
5. Check Response Headers.
6. Find `X-Cache`.

### First request

```text
X-Cache: Miss from cloudfront
```

CloudFront fetches the object from the origin.

### Subsequent request

```text
X-Cache: Hit from cloudfront
```

CloudFront serves the cached object.

Important:
This is most useful for cacheable objects. Dynamic responses or objects with caching disabled may continue to show cache misses.

You can also monitor:

CloudFront → Distribution → Monitoring.

Check:
- Requests.
- Cache hit rate.
- Bytes downloaded.
- Bytes uploaded.
- Origin latency.

---

# 8. Route 53 – Is It Mandatory?

No. Route 53 is optional for the basic CloudFront practical.

CloudFront provides a default distribution domain:

```text
https://d123456789.cloudfront.net
```

If you want your own domain:

```text
https://www.example.com
```

Configure:

1. Route 53 hosted zone.
2. Custom domain in CloudFront.
3. ACM certificate.
4. Route 53 Alias record pointing to CloudFront.

Important:
For CloudFront viewer HTTPS with a custom domain, the ACM certificate must be in US East (N. Virginia), region `us-east-1`.

---

# 9. Important Interview Questions

### Q1. What is CDN?

CDN is a globally distributed network of servers that delivers content to users from locations closer to them.

### Q2. What is Amazon CloudFront?

CloudFront is AWS's CDN service that uses Edge Locations and Regional Edge Caches to deliver content globally.

### Q3. What is an Edge Location?

An Edge Location is a CloudFront point of presence that serves cached content to nearby users.

### Q4. What is an Origin in CloudFront?

An Origin is the source from which CloudFront retrieves content, such as S3, ALB, CLB or a custom HTTP server.

### Q5. What is a Cache Hit?

When CloudFront finds the requested content in its cache and serves it without contacting the origin.

### Q6. What is a Cache Miss?

When CloudFront cannot find a valid cached object and needs to retrieve or revalidate it with the origin.

### Q7. Does CloudFront replace ALB?

No. CloudFront delivers and caches content globally. ALB distributes application traffic across backend targets.

### Q8. Can CloudFront integrate with ALB?

Yes. Configure the ALB as an origin for the CloudFront distribution.

### Q9. Is Route 53 mandatory for CloudFront?

No. CloudFront provides a default distribution domain name.

### Q10. Does CloudFront cache every request?

No. Caching depends on cache policies, TTL settings, request methods and origin response headers.

### Q11. Difference between Cache Policy and Origin Request Policy?

Cache Policy determines the cache key and TTL behavior. Origin Request Policy determines which headers, cookies and query strings are forwarded to the origin.

### Q12. What happens when CloudFront cache expires?

CloudFront may need to retrieve or revalidate the object with the origin when a subsequent request arrives, depending on its caching configuration.

### Q13. What is CloudFront invalidation?

Invalidation removes specified objects from CloudFront caches before their normal expiration.

Example:

```text
/images/logo.png
/*
```

### Q14. Can CloudFront work directly with ASG?

No. CloudFront uses an origin such as ALB. ALB forwards requests to the Target Group, and ASG manages EC2 capacity.

### Q15. What is CloudFront cache hit ratio?

It is the percentage of viewer requests served from CloudFront cache rather than requiring an origin fetch.

---

# 10. Final Architecture to Remember

```text
USER
  |
  v
ROUTE 53 (OPTIONAL DNS)
  |
  v
CLOUDFRONT CDN
  |
  +---- CACHE HIT ----> RETURNS CACHED CONTENT
  |
  +---- CACHE MISS
          |
          v
       ALB / CLB
          |
          v
       TARGET GROUP (ALB)
          |
          v
       AUTO SCALING GROUP
          |
          v
       EC2 INSTANCES
          |
          v
       APPLICATION
```

## One-line Summary

Route 53 resolves the domain, CloudFront delivers and caches content globally, ALB distributes application requests, ASG manages EC2 capacity, and EC2 runs the application.

## Hands-on Completion Checklist

- [ ] Understand CDN.
- [ ] Understand CloudFront.
- [ ] Understand Edge Locations.
- [ ] Understand Cache Hit and Cache Miss.
- [ ] Understand CloudFront Origin.
- [ ] Understand CloudFront Distribution.
- [ ] Create CloudFront distribution.
- [ ] Select existing ALB as origin.
- [ ] Configure origin protocol.
- [ ] Configure cache policy.
- [ ] Configure origin request policy.
- [ ] Configure viewer protocol policy.
- [ ] Test CloudFront domain.
- [ ] Verify X-Cache response header.
- [ ] Compare ALB DNS and CloudFront domain.
- [ ] Understand optional Route 53 integration.
- [ ] Understand CloudFront + ALB + ASG architecture.

**Final takeaway:** CloudFront reduces latency and origin traffic by caching eligible content close to users. ALB distributes requests, ASG manages compute capacity, and EC2 runs the application.