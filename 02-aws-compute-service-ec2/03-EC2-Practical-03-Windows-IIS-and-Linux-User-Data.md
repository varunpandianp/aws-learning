# EC2 Practical 03 — Windows Server IIS Website and Linux User Data

**Course:** Sudharsan — AWS Cloud / DevOps  
**Suggested path:** `AWS/03-EC2-Compute-Service/EC2-Practical-03-Windows-IIS-and-Linux-User-Data.md`

> **Security:** Restrict RDP (TCP 3389) to your current public IP (`x.x.x.x/32`), not `0.0.0.0/0`. Protect the Windows Administrator password and key pair. Check AWS costs and clean up lab resources.

## Part A — Launch Windows Server and Host a Website

### 1. High-level workflow
1. Open AWS Console → EC2 → select the intended Region → **Launch instance**.
2. Name the instance, e.g. `windows-iis-lab`.
3. Select a supported Windows Server AMI.
4. Select a suitable instance type and review pricing.
5. Create/select a key pair and keep the private key secure.
6. Configure VPC, subnet, public IPv4 if direct public access is intended, and a new security group.
7. Review storage and launch.
8. Wait for **Running** and successful status checks.
9. Retrieve/decrypt the Windows Administrator password using the matching private key, where supported.
10. Connect from your Windows laptop using Remote Desktop Connection (`Win + R` → `mstsc`).
11. Install IIS through Server Manager.
12. Copy website files to `C:\inetpub\wwwroot`.
13. Test locally and from your laptop using the instance's public IP.

### 2. Security group rules

Example name: `windows-iis-lab-sg`.

| Type | Protocol | Port | Source | Purpose |
|---|---|---:|---|---|
| RDP | TCP | 3389 | My IP (`x.x.x.x/32`) | Remote administration |
| HTTP | TCP | 80 | `0.0.0.0/0` only if website should be public | Public HTTP website |
| HTTPS | TCP | 443 | `0.0.0.0/0` only if HTTPS is configured and intended to be public | Public HTTPS website |
| All ICMP - IPv4 (optional) | ICMP | N/A | My IP / restricted trusted range | Ping troubleshooting |

**ICMP note:** Ping uses ICMP, not TCP/UDP. Ping may fail by default because the security group or Windows Firewall blocks Echo Requests. ICMP is not required for RDP, HTTP, or HTTPS. Do not enable all ICMP from the entire internet unless justified.

**Outbound:** The default security group commonly allows all outbound IPv4 traffic to `0.0.0.0/0`. This is usually sufficient for a basic lab. If egress is restricted, allow required destinations for updates and downloads. Security groups are stateful, so response traffic for permitted connections is automatically allowed.

**Screenshot:** `screenshots/windows/02-security-group.png`

### 3. Connect with RDP
1. Launch the instance and wait for status checks.
2. Select the instance → **Connect** → RDP instructions.
3. Choose **Get password / Decrypt password** if available.
4. Use the private key that matches the key pair selected at launch.
5. On the Windows laptop, run `mstsc`.
6. Enter the instance's public IPv4 address.
7. Sign in as `Administrator` using the retrieved password.
8. Accept a certificate prompt only after confirming you are connecting to the intended instance.

The password can take several minutes to become available. If decryption fails, check that you used the correct private key and that Windows initialization has completed.

**Screenshots:** `screenshots/windows/03-instance-running.png`, `screenshots/windows/04-rdp-desktop.png`

### 4. Install IIS using Server Manager
IIS (Internet Information Services) is Microsoft's web server role for Windows Server.

1. Open **Server Manager**.
2. Select **Manage** → **Add Roles and Features**.
3. Choose **Role-based or feature-based installation**.
4. Select the local server.
5. Under **Server Roles**, select **Web Server (IIS)**.
6. Choose **Add Features** if prompted.
7. Review role services; defaults are sufficient for a basic static-site lab.
8. Click **Install** and wait for completion.

**Screenshots:** `screenshots/windows/05-server-manager.png`, `screenshots/windows/06-iis-role-selected.png`, `screenshots/windows/07-iis-installed.png`

Verify inside the server browser:

```text
http://localhost
```

The default IIS welcome page should appear if IIS is installed and the Default Web Site is running.

### 5. Put website files in `C:\inetpub\wwwroot`
The common default IIS website root is:

```text
C:\inetpub\wwwroot
```

1. Open File Explorer and go to the folder.
2. Back up or move the default `iisstart.htm` if replacing the welcome page.
3. Copy the website's HTML, CSS, JavaScript, image, and other asset files into the folder.
4. Make sure the main page is `index.html`, or configure IIS default documents accordingly.
5. Keep relative paths for assets correct.

For a quick test only, save this as `C:\inetpub\wwwroot\index.html` using Notepad (choose **All files** so it is not saved as `.txt`):

```html
<!doctype html>
<html lang="en">
<head><meta charset="utf-8"><title>Windows EC2 IIS Lab</title></head>
<body>
  <h1>My website is running on Windows EC2!</h1>
  <p>Hosted using IIS on Amazon EC2.</p>
</body>
</html>
```

**Screenshot:** `screenshots/windows/08-wwwroot-files.png`

### 6. Test the website
1. Inside Windows Server, open `http://localhost`.
2. On your laptop, open `http://PUBLIC_IPV4_ADDRESS`, replacing the placeholder with the instance's actual public IPv4 address.

If local access works but public access fails, check:
- Inbound security group TCP port 80.
- Public IP and subnet route to an internet gateway.
- Windows Defender Firewall inbound rules for IIS/HTTP.
- IIS service, Default Web Site status, bindings, and default document.
- Network ACLs and other network controls.

For HTTPS, configure a certificate and HTTPS binding in IIS. Opening TCP 443 alone does not configure TLS.

### 7. Troubleshooting

| Problem | Check |
|---|---|
| RDP times out | Public IP, instance status, TCP 3389 source rule, routes, network ACLs |
| Cannot decrypt password | Correct private key, correct key pair, wait for initialization |
| `localhost` fails | IIS installation, Default Web Site status, Windows Firewall, port 80 binding |
| Local site works but public IP fails | Security group, public route, Windows Firewall, network ACL |
| Ping fails | ICMP rule and Windows Firewall Echo Request; ping failure does not prove RDP/HTTP is down |
| Default IIS page remains | Website files, default document settings, old welcome page |

---

## Part B — Linux EC2 User Data

### 8. What is user data?
EC2 user data supplies launch-time configuration instructions. On many Linux AMIs, cloud-init runs a shell script during initial boot. It can automate tasks such as installing Apache and creating a test page. Do not put secrets in user data.

### 9. Launch Linux with user data
1. EC2 → **Launch instance**.
2. Name it `linux-userdata-web-lab`.
3. Select Amazon Linux 2023 and a suitable instance type.
4. Configure a key pair if SSH access is needed.
5. Configure VPC/subnet/public IP as required.
6. Security group: HTTP TCP 80 from intended clients; SSH TCP 22 from My IP if needed.
7. Expand **Advanced details** and locate **User data**.
8. Paste the script below, review settings, and launch.
9. Wait for Running and successful status checks.
10. Open `http://PUBLIC_IPV4_ADDRESS`.

### Amazon Linux 2023 user-data script

```bash
#!/bin/bash
set -euxo pipefail

dnf install -y httpd
systemctl enable --now httpd

cat > /var/www/html/index.html <<'HTML'
<!doctype html>
<html lang="en">
<head><meta charset="utf-8"><title>EC2 User Data Lab</title></head>
<body>
  <h1>Website deployed using EC2 User Data</h1>
  <p>Apache was installed automatically at launch.</p>
</body>
</html>
HTML

chown root:root /var/www/html/index.html
chmod 0644 /var/www/html/index.html
```

This script targets Amazon Linux 2023. Ubuntu typically uses `apt` and `apache2`; do not use this script unchanged on Ubuntu or Windows. User data commonly runs on first boot by default; exact behavior depends on the AMI and initialization configuration.

### 10. Verify Linux user data
After connecting to the Linux instance:

```bash
sudo systemctl status httpd
curl -I http://localhost
curl http://localhost
sudo cloud-init status --long
sudo tail -n 100 /var/log/cloud-init-output.log
```

**Screenshots:** `screenshots/linux/01-user-data-field.png`, `screenshots/linux/02-userdata-website.png`

### Windows IIS vs Linux user data

| Windows practical | Linux user-data practical |
|---|---|
| RDP, TCP 3389 | Usually SSH or Systems Manager |
| Install IIS through Server Manager | Install Apache using startup script |
| `C:\inetpub\wwwroot` | `/var/www/html` |
| Windows Firewall and IIS bindings | Security group, Linux firewall, Apache |
| Often interactive in this lab | Can automate initial setup |

---

## Learning log

**Date:** 21 October 2026  
**Topic:** Windows Server EC2, IIS website hosting, and Linux EC2 user data

### What I learned
- Windows Server EC2 can be administered through RDP (TCP 3389).
- RDP should be restricted to the administrator's current IP.
- IIS is installed through Server Manager → Add Roles and Features → Web Server (IIS).
- The default IIS web root is commonly `C:\inetpub\wwwroot`.
- HTTP uses TCP 80; HTTPS uses TCP 443 and requires TLS configuration.
- Ping uses ICMP and is optional; it is not required for RDP or HTTP.
- Linux user data can automate initial setup, such as installing Apache and creating a page.
- Security group rules, routes, OS firewall rules, and service configuration all affect reachability.

### Practical checklist
- [ ] Launched Windows Server EC2
- [ ] Configured restricted RDP and required website rules
- [ ] Retrieved Administrator password securely
- [ ] Connected using Remote Desktop
- [ ] Installed IIS using Server Manager
- [ ] Added files to `C:\inetpub\wwwroot`
- [ ] Verified `http://localhost`
- [ ] Verified the website through public IPv4
- [ ] Launched Linux EC2 with user data
- [ ] Verified Apache and the generated page
- [ ] Captured sanitized screenshots
- [ ] Reviewed and cleaned up billable resources

Mark items complete only after performing them yourself.

## Interview questions
1. **What is RDP?** Remote Desktop Protocol, commonly using TCP 3389.
2. **What is IIS?** Microsoft's web server role for Windows Server.
3. **Where are default IIS files stored?** Commonly `C:\inetpub\wwwroot`.
4. **What are HTTP and HTTPS ports?** TCP 80 and TCP 443.
5. **Is ICMP required for RDP?** No; RDP uses TCP 3389.
6. **Why restrict RDP to My IP?** To reduce remote-administration exposure.
7. **What is EC2 user data?** Launch-time configuration data, often used to bootstrap an instance.
8. **When does Linux user data usually run?** Many cloud-init scripts run on initial boot; behavior depends on the AMI/configuration.
9. **Where are Apache files commonly stored on Amazon Linux?** `/var/www/html`.
10. **Why might IIS work on localhost but not from the internet?** Security group, route, Windows Firewall, network ACL, binding, or public IP settings.
11. **Does opening port 443 configure HTTPS?** No; a certificate and HTTPS binding/configuration are required.
12. **Should secrets be placed in user data?** No; use an appropriate secrets-management method.

## References
- Windows EC2: https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/Instances.html
- Connect to Windows: https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/connecting_to_windows_instance.html
- EC2 user data: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html
- Security groups: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html
