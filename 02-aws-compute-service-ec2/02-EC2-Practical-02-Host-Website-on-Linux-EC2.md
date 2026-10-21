# EC2 Practical 02 — Host a Website on a Linux EC2 Instance

**Course:** Sudharsan — AWS Cloud / DevOps  
**Suggested path:** `AWS/03-EC2-Compute-Service/EC2-Practical-02-Host-Website-on-Linux-EC2.md`

> **Safety:** Use a lab instance, check AWS pricing, and do not upload private keys, credentials, account IDs, or sensitive screenshots to GitHub. Use HTTP only for a simple lab; production websites should use HTTPS.

## 1. Objective

Launch or reuse a Linux EC2 instance, install Apache (`httpd`), download a website ZIP file, extract it, copy the website files into `/var/www/html`, and open the website using the instance's public IPv4 address.

## 2. High-level workflow

1. Confirm the EC2 instance is running and status checks pass.
2. Check the instance's public IPv4 address.
3. Check the attached security group's inbound rules.
4. Connect to Linux using SSH or MobaXterm.
5. Install Apache HTTP Server (`httpd`).
6. Start and enable the Apache service.
7. Install `wget` and `unzip` if needed.
8. Download the website ZIP using `wget`.
9. Extract the ZIP and locate the website's `index.html`.
10. Copy the website files into `/var/www/html`.
11. Check file ownership, permissions, and Apache status.
12. Open `http://PUBLIC_IP` in a browser and verify the website.

## 3. Security group rules

### Inbound

| Type | Protocol | Port | Source | Purpose |
|---|---|---:|---|---|
| SSH | TCP | 22 | My IP (`x.x.x.x/32`) | Connect from your laptop |
| HTTP | TCP | 80 | `0.0.0.0/0` for a deliberately public lab website | Allow browser requests from the internet |

Add IPv6 HTTP source `::/0` only if the instance is intentionally reachable over IPv6 and the networking is configured for it.

- Restrict SSH to your current public IP. Do not open port 22 to everyone for a normal lab.
- Port 80 is needed for a publicly accessible HTTP website.
- An inbound rule for port 443 is not required for this HTTP-only lab. HTTPS requires separate TLS configuration.
- Security groups are stateful, so response traffic for allowed connections is automatically permitted.

### Outbound

The default security group commonly allows all outbound IPv4 traffic to `0.0.0.0/0`. This usually allows the instance to download packages and the website archive. If outbound rules have been restricted, ensure the instance can reach the package repositories and the archive host over the required protocols, typically HTTPS/TCP 443 and sometimes HTTP/TCP 80.

## 4. Connect to the instance

Use MobaXterm/SSH with the instance's public IP, correct username for the AMI, and matching private key.

Common usernames:
- Amazon Linux: `ec2-user`
- Ubuntu: `ubuntu`

## 5. Install Apache, wget, and unzip

First identify the OS:

```bash
cat /etc/os-release
```

### Amazon Linux 2023 / Amazon Linux 2

For Amazon Linux 2023:

```bash
sudo dnf update -y
sudo dnf install -y httpd wget unzip
```

For Amazon Linux 2, `yum` is commonly used:

```bash
sudo yum update -y
sudo yum install -y httpd wget unzip
```

### Ubuntu

On Ubuntu, Apache's package is `apache2`, not `httpd`:

```bash
sudo apt update
sudo apt install -y apache2 wget unzip
```

## 6. Start and enable the web server

### Amazon Linux

```bash
sudo systemctl enable --now httpd
sudo systemctl status httpd
```

### Ubuntu

```bash
sudo systemctl enable --now apache2
sudo systemctl status apache2
```

Confirm the web server is listening on port 80:

```bash
sudo ss -lntp | grep ':80'
```

## 7. Download and extract the website ZIP

Replace `WEBSITE_ZIP_URL` with the actual direct URL of your website ZIP archive. Do not use a guessed URL.

```bash
mkdir -p ~/website-lab
cd ~/website-lab
wget -O website.zip "WEBSITE_ZIP_URL"
unzip -o website.zip -d extracted
find extracted -type f -name 'index.html'
```

If `wget` downloads an HTML error page rather than a ZIP, inspect the URL and file:

```bash
file website.zip
ls -lh website.zip
```

If the ZIP extracts into a nested folder, identify the directory that contains the site's `index.html` and related assets. Copy the contents of that website directory, not the ZIP file itself.

## 8. Copy the website into the web root

The default web root is commonly `/var/www/html`.

**Amazon Linux:**

```bash
sudo find /var/www/html -mindepth 1 -maxdepth 1 -exec rm -rf -- {} +
sudo cp -a ~/website-lab/extracted/WEBSITE_CONTENT_DIRECTORY/. /var/www/html/
sudo restorecon -Rv /var/www/html 2>/dev/null || true
```

**Ubuntu:**

```bash
sudo find /var/www/html -mindepth 1 -maxdepth 1 -exec rm -rf -- {} +
sudo cp -a ~/website-lab/extracted/WEBSITE_CONTENT_DIRECTORY/. /var/www/html/
```

Replace `WEBSITE_CONTENT_DIRECTORY` with the real extracted directory that contains `index.html`. If `index.html` is directly inside `extracted`, use `~/website-lab/extracted/.` as the source.

> The `find ... rm -rf` command clears the current web root before copying. Use it only on a disposable lab instance and only after confirming the path. Do not run it on a production server or a directory containing files you need.

Check the deployed files:

```bash
ls -la /var/www/html
test -f /var/www/html/index.html && echo "index.html found"
```

If the ZIP has no `index.html`, inspect its contents and identify the correct web entry point before copying.

## 9. Test the website

On the instance, test Apache locally:

```bash
curl -I http://localhost
curl http://localhost
```

Then, on your Windows laptop, open a browser and enter:

```text
http://PUBLIC_IPV4_ADDRESS
```

Replace `PUBLIC_IPV4_ADDRESS` with the instance's actual public IPv4 address. Do not include `https://` for this HTTP-only lab.

If it works locally but not from your laptop, re-check:
- Security group inbound TCP port 80 source.
- Public IPv4 address and subnet route to an internet gateway.
- Network ACLs and any host firewall.
- Apache service status and whether it listens on port 80.
- Whether the browser is using `http://` rather than `https://`.

## 10. Troubleshooting commands

```bash
# Service status — use the service matching your OS
sudo systemctl status httpd
sudo systemctl status apache2

# Web server logs (Amazon Linux)
sudo journalctl -u httpd --no-pager -n 50

# Web server logs (Ubuntu)
sudo journalctl -u apache2 --no-pager -n 50

# Confirm port 80 is listening
sudo ss -lntp | grep ':80'

# Test local response
curl -I http://localhost

# Check deployed files
ls -la /var/www/html
```

One of the service names may not exist on your OS; use `httpd` on Amazon Linux and `apache2` on Ubuntu.

## 11. Learning log

**Date:** 21 October 2026  
**Topic:** Deploy a static website on Linux EC2 using Apache

### What I learned
- Apache HTTP Server can serve website files from `/var/www/html`.
- Linux package commands vary by distribution: `dnf`/`yum` on Amazon Linux and `apt` on Ubuntu.
- `wget` downloads an archive and `unzip` extracts it.
- The website's `index.html` and supporting assets must be copied into the correct document root.
- Security group inbound TCP port 80 permits HTTP requests; SSH port 22 should be restricted to My IP.
- Outbound connectivity may be needed for package installation and archive download.
- Testing with `curl` locally helps distinguish web-server problems from network access problems.

### Practical checklist
- [ ] Instance running and status checks passed
- [ ] Public IPv4 address identified
- [ ] SSH inbound rule restricted to My IP
- [ ] HTTP inbound TCP port 80 configured
- [ ] Outbound access checked
- [ ] Connected using MobaXterm
- [ ] Installed Apache, wget, and unzip
- [ ] Started and enabled Apache
- [ ] Downloaded and extracted the actual website ZIP
- [ ] Copied site files into `/var/www/html`
- [ ] Verified `index.html` and local `curl` response
- [ ] Opened the public IP in a browser
- [ ] Captured and sanitized screenshots
- [ ] Reviewed AWS costs and cleaned up resources

**Status:** Mark items complete only after doing the steps yourself.

## 12. Interview questions

1. **What is Apache `httpd`?** The Apache HTTP Server service used to serve web content.
2. **What is `/var/www/html`?** A common default document root for Apache on Amazon Linux and Ubuntu installations.
3. **What is the purpose of port 80?** It is the standard TCP port for HTTP.
4. **Why does the security group need inbound port 80?** It allows browser requests to reach the web server.
5. **Why can SSH work while the website fails?** Port 22 may be allowed while port 80 is blocked, Apache is stopped, or files are misplaced.
6. **What does `wget` do?** Downloads content from a URL.
7. **What does `unzip` do?** Extracts ZIP archives.
8. **How can you test Apache locally?** Run `curl -I http://localhost`.
9. **Why might a website show the Apache test page?** The custom files may not be in the correct document root or the default page may still exist.
10. **Does opening port 443 make the website HTTPS?** No. A web server and valid TLS configuration are also required.

## References
- EC2 security groups: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html
- Amazon Linux documentation: https://docs.aws.amazon.com/linux/
- Apache HTTP Server: https://httpd.apache.org/docs/
