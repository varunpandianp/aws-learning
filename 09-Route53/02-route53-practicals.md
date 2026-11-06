# AWS Route 53 Practical – Connect GoDaddy Domain to EC2 Instance

## Objective

Purchase a domain from GoDaddy, configure AWS Route 53 as the DNS provider, and connect the domain name to a single EC2 instance using its public IPv4 address.

**Architecture:**

```text
          User / Browser
                |
                v
         vpcloud.online
                |
                v
          GoDaddy Registrar
                |
          Name Server (NS)
                |
                v
           AWS Route 53
          Public Hosted Zone
                |
             A Record
                |
                v
          EC2 Public IPv4
                |
                v
         Apache / Nginx
                |
                v
           Website
```

## Prerequisites

- AWS account.
- Domain purchased from GoDaddy.
- One EC2 Linux instance.
- EC2 instance with a public IPv4 address.
- Apache or Nginx web server installed.
- Security Group allowing HTTP traffic on port 80.

## Step 1: Purchase a Domain from GoDaddy

1. Open GoDaddy.
2. Search for an available domain.
3. Purchase the domain.

Example:

```text
Domain Name: vpcloud.online
Registrar: GoDaddy
```

## Step 2: Create a Hosted Zone in AWS Route 53

1. Login to AWS Management Console.
2. Search for Route 53.
3. Open Hosted Zones.
4. Click Create Hosted Zone.
5. Enter your purchased domain name.

Configuration:

```text
Domain Name: vpcloud.online
Type: Public Hosted Zone
```

6. Click Create Hosted Zone.

## Step 3: Copy Name Servers from Route 53

After creating the Public Hosted Zone, AWS automatically creates two records:

| Record Type | Purpose |
|---|---|
| NS | Identifies authoritative name servers |
| SOA | Contains DNS zone authority information |

Example NS records:

```text
ns-123.awsdns-45.com
ns-456.awsdns-78.net
ns-789.awsdns-12.org
ns-321.awsdns-34.co.uk
```

**Important:** These are example values. Copy the actual four NS values generated in your hosted zone.

Copy all four name servers.

## Step 4: Update Name Servers in GoDaddy

1. Login to GoDaddy.
2. Open My Products.
3. Select your purchased domain.
4. Navigate to Domain Settings / Nameservers.
5. Click Change Nameservers.
6. Select Custom Nameservers.
7. Paste all four Route 53 name servers.
8. Save the changes.

Now GoDaddy delegates DNS authority to AWS Route 53.

```text
GoDaddy Domain Registrar
          |
          v
    Route 53 Name Servers
          |
          v
    Route 53 DNS Management
```

**Note:** GoDaddy remains your domain registrar. Route 53 becomes your DNS provider.

## Step 5: Get the EC2 Public IPv4 Address

1. Open AWS EC2 Console.
2. Select your running EC2 instance.
3. Copy its Public IPv4 address.

Example:

```text
EC2 Public IPv4: 16.144.68.70
```

Use your actual instance public IPv4 address.

## Step 6: Create an A Record in Route 53

1. Open Route 53.
2. Select Hosted Zones.
3. Open `vpcloud.online`.
4. Click Create Record.

Configure:

```text
Record Name: Leave empty
Record Type: A
Value: EC2 Public IPv4
TTL: 60 seconds
Routing Policy: Simple Routing
```

Example:

```text
vpcloud.online → 16.144.68.70
```

Click Create Records.

**Why A Record?**

An A record maps a domain name to an IPv4 address.

## Step 7: Configure www Subdomain (Optional)

To access the website using `www.vpcloud.online`, create another record.

```text
Record Name: www
Record Type: CNAME
Value: vpcloud.online
TTL: 60 seconds
```

Now both domain names can resolve to the same EC2 instance:

```text
vpcloud.online
www.vpcloud.online
```

## Step 8: Configure EC2 Security Group

Open EC2 → Security Groups → Inbound Rules.

Allow:

| Type | Protocol | Port | Source |
|---|---|---|---|
| HTTP | TCP | 80 | 0.0.0.0/0 |
| HTTPS | TCP | 443 | 0.0.0.0/0 (if HTTPS configured) |
| SSH | TCP | 22 | Your public IP |

For this practical, HTTP port 80 is sufficient.

## Step 9: Install and Start Apache Web Server

For Amazon Linux:

```bash
sudo dnf install httpd -y
```

Start Apache:

```bash
sudo systemctl start httpd
```

Enable Apache on system boot:

```bash
sudo systemctl enable httpd
```

Check service status:

```bash
sudo systemctl status httpd
```

Create a sample website:

```bash
echo "<h1>Welcome to VP Cloud</h1>" | sudo tee /var/www/html/index.html
```

Test Apache locally:

```bash
curl http://localhost
```

Test using the EC2 public IP:

```text
http://16.144.68.70
```

## Step 10: Wait for DNS Propagation and Test Domain

After updating nameservers and creating DNS records, allow time for DNS delegation and caching to update.

Verify using:

```bash
# Check domain resolution
nslookup vpcloud.online

# Check DNS A record
dig A vpcloud.online

# Check authoritative nameservers
dig NS vpcloud.online

# Query Google's public DNS resolver
dig @8.8.8.8 vpcloud.online

# Test website response
curl -I http://vpcloud.online
```

Finally, open the browser:

```text
http://vpcloud.online
```

Expected result:

```text
Welcome to VP Cloud
```

## Final Architecture

```text
                 INTERNET
                    |
                    v
             vpcloud.online
                    |
                    v
              GODADDY DOMAIN
                 REGISTRAR
                    |
                    v
              ROUTE 53 NS
                    |
                    v
            PUBLIC HOSTED ZONE
                    |
                    v
                 A RECORD
                    |
                    v
             EC2 PUBLIC IPv4
                    |
                    v
               SECURITY GROUP
                    |
                 HTTP 80
                    |
                    v
              APACHE / NGINX
                    |
                    v
              VP CLOUD WEBSITE
```

## Important Learning Points

1. Domain registration and DNS hosting are two different services.
2. GoDaddy acts as the domain registrar.
3. Route 53 acts as the DNS provider.
4. NS records identify authoritative name servers.
5. SOA records contain DNS zone authority information.
6. A records map domain names to IPv4 addresses.
7. CNAME records map one hostname to another hostname.
8. DNS resolution and website hosting are separate processes.
9. EC2 Security Groups must allow inbound HTTP traffic.
10. A public EC2 IP can change after stopping and starting an instance. An Elastic IP can provide a stable IPv4 address.

## Practical Completed

**Objective achieved:** Connected a GoDaddy-purchased domain to a single AWS EC2 Linux instance using Route 53 Public Hosted Zone and an A record.

```text
GoDaddy → Route 53 → A Record → EC2 → Apache Website
```

**Note:** This practical uses one EC2 instance directly. No Application Load Balancer, Target Group or Auto Scaling Group is involved. HTTPS is not configured in this exercise.