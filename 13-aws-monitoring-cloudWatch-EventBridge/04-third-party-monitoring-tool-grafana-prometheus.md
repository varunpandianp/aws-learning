# Prometheus & Grafana – Third-Party Monitoring Tools

**Target role:** Junior Cloud Engineer / DevOps Engineer / SRE  
**Lab OS:** Amazon Linux 2023  
**Cloud platform:** AWS EC2  
**Monitoring stack:** Prometheus + Grafana + Node Exporter  
**Additional exporter:** Blackbox Exporter  
**Purpose:** Monitor Linux infrastructure, visualize metrics and troubleshoot production systems.

---

# 1. Mental Model & Architecture

## 1.1 Why do we need third-party monitoring?

In a DevOps or SRE environment, monitoring helps engineers understand the health and performance of infrastructure and applications.

For example, imagine that an application is running on three EC2 instances.

An engineer needs to know:

- Is CPU utilization increasing?
- Is memory almost full?
- Is the disk running out of space?
- Is network traffic unusually high?
- Is the application responding?
- Is the website returning HTTP 200?
- Is a server unavailable?
- When should an alert be triggered?

AWS CloudWatch provides monitoring for AWS resources. Prometheus and Grafana provide an open-source monitoring and visualization stack that can also be used across cloud providers, virtual machines and Kubernetes environments.

## 1.2 Mental model: What does each tool do?

| Component | Responsibility | Simple explanation |
|---|---|---|
| Prometheus | Metrics collection and storage | Collects and stores metrics |
| Grafana | Visualization | Displays metrics using dashboards |
| Node Exporter | Linux system metrics | Collects CPU, RAM, disk and network metrics |
| Blackbox Exporter | Endpoint monitoring | Checks websites and APIs |
| PromQL | Query language | Queries metrics in Prometheus |
| Alertmanager | Alert management | Routes Prometheus alerts to notification systems |

**Remember:**

- Prometheus = Collector + Time-series database
- Grafana = Visualization and dashboard tool
- Node Exporter = Linux server metrics
- Blackbox Exporter = Website and endpoint monitoring
- Alertmanager = Prometheus alert notification management

## 1.3 How does Prometheus work?

Prometheus normally uses a pull-based monitoring model.

This means Prometheus periodically connects to a target and collects metrics from its HTTP metrics endpoint.

Example:

1. Install Node Exporter on an EC2 instance.
2. Node Exporter collects Linux system metrics.
3. Node Exporter exposes metrics on port 9100.
4. Prometheus connects to port 9100.
5. Prometheus scrapes the metrics every 15 seconds.
6. Prometheus stores the metrics as time-series data.
7. Grafana queries Prometheus.
8. Grafana displays the metrics in dashboards.

## 1.4 Architecture diagram

```text
                    AWS CLOUD
                       |
             +---------+---------+
             |                   |
             v                   v
        EC2 Server 1        EC2 Server 2
             |                   |
             v                   v
       Node Exporter        Node Exporter
         Port 9100            Port 9100
             |                   |
             +---------+---------+
                       |
                       | Prometheus scrape
                       v
                PROMETHEUS SERVER
                    Port 9090
                       |
               +-------+-------+
               |               |
               v               v
           PromQL         Alerting Rules
               |               |
               v               v
            GRAFANA         ALERTMANAGER
            Port 3000            |
               |                v
               v          Email / Slack
          Dashboards
          Graphs / Charts
```

## 1.5 What is an exporter?

An exporter exposes metrics in a format Prometheus understands.

Prometheus does not automatically understand every operating system, application or external service.

Exporters act as a bridge between the monitored system and Prometheus.

### Node Exporter

Used for Linux system monitoring.

| Metric | Example |
|---|---|
| CPU | CPU utilization |
| Memory | RAM utilization |
| Disk | Filesystem usage |
| Network | Bytes received and transmitted |
| Load | System load average |

### Blackbox Exporter

Used for external endpoint monitoring.

| Metric | Example |
|---|---|
| Website availability | Is the website reachable? |
| HTTP status | Is the response 200, 301, 404 or 500? |
| Response time | How long does the request take? |
| TLS | Is the HTTPS certificate valid? |
| DNS | Can the hostname be resolved? |

### Important difference

Node Exporter answers:

"Is my Linux server healthy?"

Blackbox Exporter answers:

"Can users reach my website or API?"

## 1.6 Important ports

| Service | Default Port |
|---|---:|
| SSH | 22 |
| Prometheus | 9090 |
| Grafana | 3000 |
| Node Exporter | 9100 |
| Blackbox Exporter | 9115 |

---

# 2. Hands-On Lab

## 2.1 Lab architecture

For this beginner lab, install the monitoring stack on one Amazon Linux 2023 EC2 instance.

```text
AWS EC2
   |
   +-- Prometheus :9090
   |
   +-- Grafana :3000
   |
   +-- Node Exporter :9100
   |
   +-- Blackbox Exporter :9115
```

This is a learning setup, not a recommended production architecture.

### EC2 requirements

| Setting | Example |
|---|---|
| Instance name | monitoring-server |
| OS | Amazon Linux 2023 |
| Architecture | x86_64 |
| Instance type | t3.medium |
| Root volume | 20 GB or more for this lab |
| Access | SSH |

Actual production sizing depends on the number of monitored targets, scrape frequency, metric cardinality, retention and dashboard workload.

### Security group

Use the following access model:

| Port | Source | Purpose |
|---|---|---|
| 22 | Your public IP | SSH |
| 9090 | Trusted admin IP or monitoring network | Prometheus UI |
| 3000 | Trusted admin IP or VPN | Grafana UI |
| 9100 | Prometheus server security group | Node Exporter |
| 9115 | Prometheus server security group | Blackbox Exporter |

**Important:** Do not expose exporter ports or Prometheus directly to the public internet. Prefer private networking and restrict inbound access.

## 2.2 Connect to EC2

```bash
ssh -i monitoring-key.pem ec2-user@YOUR_PUBLIC_IP
```

Update packages:

```bash
sudo dnf update -y
```

Install basic tools:

```bash
sudo dnf install wget tar curl vim -y
```

Check OS:

```bash
cat /etc/os-release
```

Check architecture:

```bash
uname -m
```

For this lab, the expected architecture is:

```text
x86_64
```

## 2.3 Install Prometheus

### Step 1: Create Prometheus user

```bash
sudo useradd \
  --system \
  --no-create-home \
  --shell /sbin/nologin \
  prometheus
```

### Step 2: Download Prometheus

Check the latest stable version and the correct Linux architecture at:

https://prometheus.io/download/

Set the version to the release you selected:

```bash
VERSION="REPLACE_WITH_SELECTED_VERSION"
ARCH="linux-amd64"
```

Download:

```bash
cd /tmp

wget "https://github.com/prometheus/prometheus/releases/download/v${VERSION}/prometheus-${VERSION}.${ARCH}.tar.gz"
```

Extract:

```bash
tar -xzf "prometheus-${VERSION}.${ARCH}.tar.gz"

cd "prometheus-${VERSION}.${ARCH}"
```

### Step 3: Install Prometheus binaries

```bash
sudo install -m 0755 prometheus /usr/local/bin/prometheus

sudo install -m 0755 promtool /usr/local/bin/promtool
```

Verify:

```bash
prometheus --version

promtool --version
```

### Step 4: Create directories

```bash
sudo mkdir -p /etc/prometheus

sudo mkdir -p /var/lib/prometheus
```

Set ownership:

```bash
sudo chown prometheus:prometheus /etc/prometheus

sudo chown prometheus:prometheus /var/lib/prometheus
```

## 2.4 Configure Prometheus

Create the configuration file:

```bash
sudo vi /etc/prometheus/prometheus.yml
```

Add this valid YAML configuration:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:

  - job_name: prometheus
    static_configs:
      - targets:
          - localhost:9090

  - job_name: node_exporter
    static_configs:
      - targets:
          - localhost:9100
```

### Understand the configuration

**global**

Defines settings that apply across scrape jobs.

**scrape_interval**

```yaml
scrape_interval: 15s
```

Prometheus collects metrics every 15 seconds unless a job overrides the interval.

**evaluation_interval**

```yaml
evaluation_interval: 15s
```

Prometheus evaluates alerting and recording rules at this interval.

**job_name**

Identifies the monitoring job.

**targets**

Defines the endpoints Prometheus should scrape.

In this lab, Node Exporter and Prometheus are running on the same EC2 instance, so `localhost` works.

If Node Exporter runs on another EC2 instance, use its private IP:

```yaml
- job_name: node_exporter
  static_configs:
    - targets:
        - 10.0.1.20:9100
        - 10.0.2.20:9100
```

Replace these example IPs with the actual private IP addresses.

Set configuration ownership:

```bash
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

### Step 5: Validate configuration

```bash
sudo promtool check config /etc/prometheus/prometheus.yml
```

Expected result:

```text
SUCCESS: /etc/prometheus/prometheus.yml is valid prometheus config file syntax
```

## 2.5 Create Prometheus systemd service

Create:

```bash
sudo vi /etc/systemd/system/prometheus.service
```

Add:

```ini
[Unit]
Description=Prometheus Monitoring Server
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple

ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --storage.tsdb.retention.time=15d \
  --storage.tsdb.retention.size=5GB

Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

### Understand systemd configuration

| Parameter | Purpose |
|---|---|
| User | Runs Prometheus as a dedicated user |
| Group | Runs with the Prometheus group |
| ExecStart | Starts the Prometheus binary |
| config.file | Specifies the YAML configuration |
| storage.tsdb.path | Defines the metrics database directory |
| retention.time | Keeps metrics for 15 days |
| retention.size | Limits retained storage to 5 GB |
| Restart | Restarts Prometheus after a failure |

The 15-day and 5-GB values are lab examples, not universal production recommendations.

### Step 6: Start Prometheus

```bash
sudo systemctl daemon-reload

sudo systemctl enable prometheus

sudo systemctl start prometheus
```

Check:

```bash
sudo systemctl status prometheus
```

Check port:

```bash
sudo ss -lntp | grep 9090
```

Test API:

```bash
curl http://localhost:9090/-/healthy
```

Expected:

```text
Prometheus is Healthy.
```

Access UI:

```text
http://YOUR_PUBLIC_IP:9090
```

For production, access through a secured administrative network rather than opening the UI to the internet.

## 2.6 Install Node Exporter

Node Exporter collects Linux operating system metrics.

### Step 1: Create user

```bash
sudo useradd \
  --system \
  --no-create-home \
  --shell /sbin/nologin \
  node_exporter
```

### Step 2: Download Node Exporter

Check the selected release at:

https://prometheus.io/download/

```bash
VERSION="REPLACE_WITH_SELECTED_VERSION"

cd /tmp

wget "https://github.com/prometheus/node_exporter/releases/download/v${VERSION}/node_exporter-${VERSION}.linux-amd64.tar.gz"

tar -xzf "node_exporter-${VERSION}.linux-amd64.tar.gz"
```

Install binary:

```bash
sudo install -m 0755 \
  "node_exporter-${VERSION}.linux-amd64/node_exporter" \
  /usr/local/bin/node_exporter
```

Verify:

```bash
node_exporter --version
```

### Step 3: Create systemd service

```bash
sudo vi /etc/systemd/system/node_exporter.service
```

Add:

```ini
[Unit]
Description=Prometheus Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple

ExecStart=/usr/local/bin/node_exporter

Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

### Step 4: Start service

```bash
sudo systemctl daemon-reload

sudo systemctl enable node_exporter

sudo systemctl start node_exporter
```

Check:

```bash
sudo systemctl status node_exporter
```

Check port:

```bash
sudo ss -lntp | grep 9100
```

Test endpoint:

```bash
curl http://localhost:9100/metrics
```

You should see metrics such as:

```text
node_cpu_seconds_total
node_memory_MemTotal_bytes
node_memory_MemAvailable_bytes
node_filesystem_size_bytes
node_network_receive_bytes_total
node_network_transmit_bytes_total
```

### Step 5: Connect Node Exporter to Prometheus

The Prometheus configuration already contains:

```yaml
  - job_name: node_exporter
    static_configs:
      - targets:
          - localhost:9100
```

Restart Prometheus:

```bash
sudo systemctl restart prometheus
```

Open:

```text
http://YOUR_PUBLIC_IP:9090/targets
```

Expected:

| Job | State |
|---|---|
| prometheus | UP |
| node_exporter | UP |

**UP means Prometheus successfully scraped the target. DOWN means the scrape failed.**

## 2.7 Install Grafana

Grafana provides the graphical dashboard for the metrics stored in Prometheus.

Official installation guide:

https://grafana.com/docs/grafana/latest/setup-grafana/installation/redhat-rhel-fedora/

### Step 1: Import repository signing key

```bash
wget -q -O /tmp/grafana.gpg.key https://rpm.grafana.com/gpg.key

sudo rpm --import /tmp/grafana.gpg.key
```

### Step 2: Create repository

```bash
sudo tee /etc/yum.repos.d/grafana.repo > /dev/null <<'EOF'
[grafana]
name=grafana
baseurl=https://rpm.grafana.com
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
sslverify=1
EOF
```

### Step 3: Install Grafana OSS

```bash
sudo dnf install grafana -y
```

### Step 4: Start Grafana

```bash
sudo systemctl daemon-reload

sudo systemctl enable grafana-server

sudo systemctl start grafana-server
```

Check:

```bash
sudo systemctl status grafana-server
```

Check port:

```bash
sudo ss -lntp | grep 3000
```

Access:

```text
http://YOUR_PUBLIC_IP:3000
```

On a fresh installation, the default credentials are generally:

```text
Username: admin
Password: admin
```

Change the default password immediately.

## 2.8 Connect Grafana to Prometheus

### Step 1: Open Grafana

Navigate to:

```text
Connections
    |
    v
Data Sources
    |
    v
Add data source
    |
    v
Prometheus
```

### Step 2: Configure URL

If Grafana and Prometheus are on the same server:

```text
http://localhost:9090
```

If Prometheus runs on a separate server:

```text
http://PROMETHEUS_PRIVATE_IP:9090
```

### Step 3: Save and test

Click:

```text
Save & Test
```

Expected result:

```text
Successfully queried the Prometheus API
```

Grafana can now query Prometheus.

## 2.9 Create a Grafana dashboard

Navigate to:

```text
Dashboards
    |
    v
New
    |
    v
New Dashboard
    |
    v
Add Visualization
```

Select Prometheus as the data source.

### CPU utilization

```promql
100 - (
  avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  ) * 100
)
```

### Memory utilization

```promql
100 * (
  1 -
  node_memory_MemAvailable_bytes /
  node_memory_MemTotal_bytes
)
```

### Disk utilization

```promql
100 * (
  1 -
  node_filesystem_avail_bytes{
    mountpoint="/",
    fstype!=""
  } /
  node_filesystem_size_bytes{
    mountpoint="/",
    fstype!=""
  }
)
```

### Network receive rate

```promql
rate(node_network_receive_bytes_total[5m])
```

### Network transmit rate

```promql
rate(node_network_transmit_bytes_total[5m])
```

### System load

```promql
node_load1
```

Choose a visualization:

- Time series for CPU, memory and network trends.
- Gauge for a current percentage.
- Stat for a single current value.
- Table for detailed metric information.

Save the dashboard with a meaningful name:

```text
Linux Server Monitoring Dashboard
```

## 2.10 Install Blackbox Exporter for website monitoring

Blackbox Exporter checks external services such as HTTP, HTTPS, TCP, DNS and ICMP endpoints.

### Step 1: Create user

```bash
sudo useradd \
  --system \
  --no-create-home \
  --shell /sbin/nologin \
  blackbox_exporter
```

### Step 2: Download binary

Check the release at:

https://prometheus.io/download/

```bash
VERSION="REPLACE_WITH_SELECTED_VERSION"

cd /tmp

wget "https://github.com/prometheus/blackbox_exporter/releases/download/v${VERSION}/blackbox_exporter-${VERSION}.linux-amd64.tar.gz"

tar -xzf "blackbox_exporter-${VERSION}.linux-amd64.tar.gz"
```

Install:

```bash
sudo install -m 0755 \
  "blackbox_exporter-${VERSION}.linux-amd64/blackbox_exporter" \
  /usr/local/bin/blackbox_exporter
```

### Step 3: Create configuration

```bash
sudo mkdir -p /etc/blackbox_exporter

sudo vi /etc/blackbox_exporter/blackbox.yml
```

Add:

```yaml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      method: GET
      preferred_ip_protocol: ip4
```

Set ownership:

```bash
sudo chown -R blackbox_exporter:blackbox_exporter \
  /etc/blackbox_exporter
```

### Step 4: Create systemd service

```bash
sudo vi /etc/systemd/system/blackbox_exporter.service
```

Add:

```ini
[Unit]
Description=Prometheus Blackbox Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=blackbox_exporter
Group=blackbox_exporter
Type=simple

ExecStart=/usr/local/bin/blackbox_exporter \
  --config.file=/etc/blackbox_exporter/blackbox.yml \
  --web.listen-address=:9115

Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

Start:

```bash
sudo systemctl daemon-reload

sudo systemctl enable blackbox_exporter

sudo systemctl start blackbox_exporter
```

Verify:

```bash
sudo systemctl status blackbox_exporter

curl http://localhost:9115/metrics
```

### Step 5: Test a website

```bash
curl "http://localhost:9115/probe?target=https://example.com&module=http_2xx"
```

Check:

```text
probe_success 1
```

`1` means the probe succeeded. `0` means it failed.

### Step 6: Add Blackbox to Prometheus

Add this job to `/etc/prometheus/prometheus.yml`:

```yaml
  - job_name: blackbox_http
    metrics_path: /probe

    params:
      module:
        - http_2xx

    static_configs:
      - targets:
          - https://example.com
          - https://google.com

    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target

      - source_labels: [__param_target]
        target_label: instance

      - target_label: __address__
        replacement: localhost:9115
```

Restart:

```bash
sudo systemctl restart prometheus
```

Check targets:

```text
http://YOUR_PUBLIC_IP:9090/targets
```

Useful queries:

```promql
probe_success
```

```promql
probe_http_status_code
```

```promql
probe_duration_seconds
```

**Important:** Blackbox Exporter checks the endpoint from the network location of the exporter. If you want to monitor user experience from multiple geographic locations, deploy probes in multiple locations.

---

# 3. Real-World Production Practices

A basic lab teaches installation and configuration. Production monitoring requires more than installing Prometheus and Grafana.

## 3.1 Lab setup vs production setup

| Area | Basic lab | Production practice |
|---|---|---|
| Infrastructure | One EC2 | Multiple monitoring and target servers |
| Target configuration | Static IPs | Service discovery or file-based discovery |
| Dashboard | Manually created | Shared, version-controlled dashboards |
| Security | Basic security group | Private networking, authentication and TLS |
| Retention | Small fixed period | Capacity-based retention planning |
| Alerts | Simple threshold | Actionable alerts with severity and routing |
| Availability | One Prometheus server | HA or managed monitoring architecture |
| Configuration | Manual edits | Git and configuration management |
| Backup | Often ignored | Backup and recovery strategy |

## 3.2 Importing Grafana dashboard IDs

In real projects, engineers do not always create every dashboard panel manually.

They often import existing community dashboards.

Example workflow:

1. Open Grafana.
2. Navigate to Dashboards.
3. Select New.
4. Select Import.
5. Enter a dashboard ID.
6. Select the Prometheus data source.
7. Import the dashboard.

For example, the Node Exporter Full dashboard is commonly available under dashboard ID:

```text
1860
```

Verify the dashboard and its compatibility before using it:

https://grafana.com/grafana/dashboards/1860/

**Production practice:**

- Review the dashboard before importing.
- Check whether it expects a particular exporter version.
- Confirm the Prometheus data source.
- Check the dashboard's variables and labels.
- Avoid importing dashboards from untrusted sources.
- Export and store important dashboard JSON in Git.

A dashboard is only as useful as the metrics and labels available in the data source.

## 3.3 Service discovery

In a basic lab, we configure target IP addresses manually.

Example:

```yaml
static_configs:
  - targets:
      - 10.0.1.20:9100
      - 10.0.2.20:9100
```

Problem:

Suppose an Auto Scaling Group has 10 EC2 instances.

Tomorrow, the Auto Scaling Group terminates two instances and launches three new ones.

Their IP addresses may change.

Manually updating Prometheus configuration every time is not a good production approach.

This is why service discovery is useful.

### Types of service discovery

- AWS EC2 service discovery
- Kubernetes service discovery
- DNS service discovery
- File-based service discovery
- Consul service discovery

### File-based service discovery example

Create:

```bash
sudo mkdir -p /etc/prometheus/file_sd

sudo vi /etc/prometheus/file_sd/node_targets.yml
```

Add:

```yaml
- targets:
    - 10.0.1.20:9100
    - 10.0.2.20:9100
  labels:
    environment: production
    team: platform
```

Update Prometheus configuration:

```yaml
  - job_name: node_exporter
    file_sd_configs:
      - files:
          - /etc/prometheus/file_sd/*.yml
        refresh_interval: 30s
```

Validate:

```bash
sudo promtool check config /etc/prometheus/prometheus.yml
```

Restart Prometheus:

```bash
sudo systemctl restart prometheus
```

Prometheus periodically checks the file and updates the target list.

### AWS EC2 service discovery concept

Prometheus can discover EC2 instances using EC2 service discovery.

Example:

```yaml
  - job_name: aws_ec2_node_exporter
    ec2_sd_configs:
      - region: ap-south-1
        port: 9100
        filters:
          - name: tag:Monitoring
            values:
              - enabled
```

This is an example configuration. In production, configure the correct AWS region, filters, permissions and relabeling for your environment.

The Prometheus EC2 discovery configuration needs AWS API permissions to discover instances. Use an attached IAM role rather than hardcoded AWS access keys.

An EC2 discovery role typically needs read-only permissions for the required EC2 discovery APIs, such as describing instances, regions and availability zones.

The monitoring server should only scrape the discovered instances that are reachable through the private network and security groups.

## 3.4 Security hardening

Prometheus and exporters expose system information.

Do not expose their endpoints publicly without appropriate security controls.

### Recommended practices

1. Keep Prometheus and exporters in private subnets where possible.
2. Allow port 9100 only from Prometheus.
3. Allow port 9115 only from Prometheus.
4. Restrict port 9090 to trusted administrators.
5. Restrict Grafana access to VPN or approved networks.
6. Change default Grafana credentials.
7. Enable HTTPS and authentication.
8. Use IAM roles for AWS API access.
9. Keep packages and binaries updated.
10. Protect configuration files and credentials.

### Example AWS security group design

| Server | Inbound port | Allowed source |
|---|---:|---|
| Prometheus | 9090 | Admin network |
| Grafana | 3000 | Admin network |
| Prometheus | 9100 | Exporter servers |
| Prometheus | 9115 | Blackbox Exporter |
| Exporter server | 9100 | Prometheus security group |
| Blackbox server | 9115 | Prometheus security group |

For a single-server lab, use localhost wherever possible and do not open exporter ports externally.

## 3.5 Retention and disk management

Prometheus stores time-series metrics on disk.

If retention is not planned, the metrics database can consume too much storage.

Example:

```text
--storage.tsdb.retention.time=15d
--storage.tsdb.retention.size=5GB
```

This means:

- Retain metrics for up to 15 days.
- Limit retained TSDB storage to 5 GB.

The effective retention is constrained by whichever configured limit is reached first.

### Production considerations

- Monitor Prometheus disk usage.
- Estimate ingestion rate and retention needs.
- Choose storage capacity based on actual metric volume.
- Use persistent storage.
- Monitor compaction and WAL behavior.
- Avoid excessive metric cardinality.
- Consider remote storage for long-term retention.

Prometheus is not intended to be an unlimited historical data warehouse.

## 3.6 Metric cardinality

Cardinality means the number of unique time series.

For example:

```text
http_requests_total{method="GET",status="200"}
```

This is usually reasonable.

However, using a unique user ID or request ID as a label can create huge numbers of time series.

Bad example:

```text
http_requests_total{user_id="unique-user-12345"}
```

Why is this dangerous?

- More time series.
- More memory consumption.
- More disk consumption.
- Higher query cost.
- Increased operational complexity.

Avoid labels containing unbounded values such as request IDs, user IDs and arbitrary URLs.

## 3.7 Dashboard and configuration as code

In production, engineers should not depend only on manually created dashboards.

Use:

- Git repositories.
- Dashboard JSON.
- Grafana provisioning.
- Ansible.
- Terraform.
- CI/CD pipelines.

Example repository:

```text
monitoring/
|
+-- prometheus/
|   |
|   +-- prometheus.yml
|   +-- alert-rules.yml
|
+-- grafana/
|   |
|   +-- dashboards/
|   |   +-- linux-dashboard.json
|   |
|   +-- provisioning/
|       +-- datasources/
|
+-- exporters/
    |
    +-- node-exporter/
    +-- blackbox-exporter/
```

Grafana provisioning can define data sources and dashboards through files. This makes configuration repeatable and easier to review through Git.

## 3.8 High availability

A single Prometheus server can become a single point of failure.

In production, depending on scale and requirements, teams may use:

- Two independent Prometheus replicas.
- Alertmanager clustering.
- Remote write to long-term metrics storage.
- Managed monitoring services.
- Multiple Grafana instances behind a load balancer.

Prometheus replicas scrape targets independently. They do not automatically form one shared, distributed database.

HA architecture must also consider alert deduplication, data storage and Grafana data source configuration.

---

# 4. Day-2 Troubleshooting

Day-2 operations means diagnosing and resolving problems after the monitoring stack has been installed.

The following are five common production-style failure scenarios.

## Scenario 1: Prometheus service is not starting

### Symptoms

- Prometheus UI is unavailable.
- Port 9090 is not listening.
- systemd reports a failed service.

### Step 1: Check service status

```bash
sudo systemctl status prometheus
```

### Step 2: Check logs

```bash
sudo journalctl -u prometheus -n 100 --no-pager
```

Follow logs:

```bash
sudo journalctl -u prometheus -f
```

### Step 3: Validate configuration

```bash
sudo promtool check config /etc/prometheus/prometheus.yml
```

### Step 4: Check file permissions

```bash
sudo ls -l /etc/prometheus/prometheus.yml

sudo ls -ld /var/lib/prometheus
```

### Step 5: Fix and restart

```bash
sudo systemctl restart prometheus

sudo systemctl status prometheus
```

### Common causes

- Invalid YAML.
- Incorrect configuration path.
- Incorrect binary path.
- Wrong file ownership.
- Port 9090 already in use.
- Incorrect systemd service configuration.

## Scenario 2: Node Exporter target shows DOWN

### Symptoms

Prometheus UI:

```text
job_name: node_exporter
state: DOWN
```

### Step 1: Check Node Exporter

```bash
sudo systemctl status node_exporter
```

### Step 2: Check logs

```bash
sudo journalctl -u node_exporter -n 100 --no-pager
```

### Step 3: Check listening port

```bash
sudo ss -lntp | grep 9100
```

### Step 4: Test locally

On the exporter server:

```bash
curl http://localhost:9100/metrics
```

### Step 5: Test remotely

From the Prometheus server:

```bash
curl http://NODE_PRIVATE_IP:9100/metrics
```

### Step 6: Check AWS security group

Verify:

- Node Exporter server allows inbound TCP 9100.
- Source is the Prometheus security group.
- Network ACLs and routes allow communication.

### Fix

```bash
sudo systemctl enable node_exporter

sudo systemctl restart node_exporter
```

If the service is healthy but remote access fails, troubleshoot networking and security groups.

## Scenario 3: Grafana dashboard shows No Data

### Symptoms

- Grafana is accessible.
- Dashboard loads.
- CPU and memory graphs are empty.

### Step 1: Check Prometheus

```bash
curl http://localhost:9090/-/healthy
```

### Step 2: Check Node Exporter

```bash
curl http://localhost:9100/metrics
```

### Step 3: Check Prometheus targets

Open:

```text
http://PROMETHEUS_PRIVATE_IP:9090/targets
```

Ensure Node Exporter is UP.

### Step 4: Test PromQL directly in Prometheus

```promql
up
```

Then:

```promql
node_memory_MemTotal_bytes
```

Then:

```promql
node_cpu_seconds_total
```

### Step 5: Check Grafana data source

Navigate:

```text
Connections
    |
    v
Data Sources
    |
    v
Prometheus
```

Check the URL.

If Grafana and Prometheus are on the same server:

```text
http://localhost:9090
```

If they are on different servers, use the reachable Prometheus private address.

### Common causes

- Wrong Prometheus URL.
- Prometheus target DOWN.
- Incorrect PromQL query.
- Wrong dashboard data source.
- Dashboard expects different metric labels.
- Time range does not include collected data.

## Scenario 4: Prometheus disk usage is increasing rapidly

### Symptoms

- Disk usage increases.
- Prometheus storage directory becomes large.
- EC2 root filesystem approaches 100%.
- Prometheus may eventually fail to write data.

### Step 1: Check filesystem usage

```bash
df -h
```

### Step 2: Check Prometheus data directory

```bash
sudo du -sh /var/lib/prometheus
```

### Step 3: Check largest directories

```bash
sudo du -h --max-depth=2 /var/lib/prometheus | sort -h
```

### Step 4: Check retention configuration

```bash
sudo systemctl cat prometheus
```

Look for:

```text
--storage.tsdb.retention.time
--storage.tsdb.retention.size
```

### Step 5: Review metric cardinality

Use Prometheus queries to investigate the number of active series:

```promql
prometheus_tsdb_head_series
```

Check scrape volume and unexpected high-cardinality labels.

### Fix

Adjust retention based on capacity:

```text
--storage.tsdb.retention.time=15d
--storage.tsdb.retention.size=5GB
```

Restart Prometheus after updating the service.

Do not manually delete active Prometheus TSDB files while Prometheus is running. Plan storage changes and backups carefully.

## Scenario 5: Prometheus is UP, but website monitoring is failing

### Symptoms

- Prometheus service is healthy.
- Blackbox Exporter service is running.
- Website probe returns `probe_success 0`.

### Step 1: Check Blackbox Exporter

```bash
sudo systemctl status blackbox_exporter
```

### Step 2: Check logs

```bash
sudo journalctl -u blackbox_exporter -n 100 --no-pager
```

### Step 3: Check port

```bash
sudo ss -lntp | grep 9115
```

### Step 4: Test the website from the exporter host

```bash
curl -Iv https://example.com
```

### Step 5: Test the Blackbox probe

```bash
curl "http://localhost:9115/probe?target=https://example.com&module=http_2xx"
```

### Step 6: Check metrics

```text
probe_success
probe_http_status_code
probe_duration_seconds
```

### Common causes

- DNS resolution failure.
- Website is unavailable.
- Network route is blocked.
- Outbound access is restricted.
- TLS certificate problem.
- HTTP endpoint returns an unexpected status.
- Blackbox configuration is incorrect.

### Fix

Check DNS:

```bash
getent hosts example.com
```

Check TLS and HTTP response:

```bash
curl -Iv https://example.com
```

Check outbound routing, NAT Gateway configuration if applicable, security controls and exporter configuration.

Remember: a successful Prometheus scrape of Blackbox Exporter does not necessarily mean the website probe succeeded. Check `probe_success` separately.

---

# 5. Scenario-Based Interview Q&A

These questions test practical monitoring knowledge rather than simple definitions.

## Q1. Your Grafana dashboard shows no CPU metrics, but Prometheus is running. How will you troubleshoot?

**Expected answer:**

First, I will verify whether Prometheus itself is healthy.

```bash
curl http://localhost:9090/-/healthy
```

Then I will check the Node Exporter service:

```bash
sudo systemctl status node_exporter
```

I will verify port 9100:

```bash
sudo ss -lntp | grep 9100
```

Then I will check whether Node Exporter exposes CPU metrics:

```bash
curl http://localhost:9100/metrics | grep node_cpu_seconds_total
```

Next, I will open the Prometheus Targets page and verify whether Node Exporter is UP.

If the target is UP, I will execute the CPU PromQL query directly in Prometheus.

Finally, I will check Grafana's Prometheus data source, dashboard query, labels and time range.

**What the interviewer expects:** A structured approach that checks the exporter, scrape target, query and visualization separately.

## Q2. Your EC2 Auto Scaling Group launches new instances, but Prometheus does not monitor them. What will you do?

**Expected answer:**

If Prometheus uses static IP addresses, newly launched EC2 instances will not automatically become monitoring targets.

I will implement service discovery.

For AWS, I can use Prometheus EC2 service discovery and filter instances using tags.

For example:

```yaml
ec2_sd_configs:
  - region: ap-south-1
    port: 9100
    filters:
      - name: tag:Monitoring
        values:
          - enabled
```

I will also ensure:

- Node Exporter is installed on new instances.
- Prometheus has the required EC2 discovery IAM permissions.
- Instances are reachable over private networking.
- Security groups permit TCP 9100 from Prometheus.
- Appropriate relabeling is configured.

**What the interviewer expects:** Understanding of dynamic infrastructure and service discovery rather than manually maintaining IP addresses.

## Q3. Prometheus is collecting metrics, but the EC2 disk is almost full. How will you troubleshoot and prevent this?

**Expected answer:**

First, I will check filesystem usage:

```bash
df -h
```

Then I will check Prometheus storage:

```bash
sudo du -sh /var/lib/prometheus
```

I will inspect the retention configuration and identify whether the problem is excessive retention, high ingestion or high metric cardinality.

I will review:

- Retention time.
- Retention size.
- Scrape interval.
- Number of active time series.
- Labels generating excessive cardinality.

I will configure appropriate retention and storage capacity.

For long-term metrics retention, I will evaluate remote storage solutions.

I will not manually delete active TSDB files while Prometheus is running.

**What the interviewer expects:** Understanding of TSDB storage, retention and cardinality.

## Q4. Prometheus shows the Blackbox Exporter target as UP, but the website is down. Why?

**Expected answer:**

The Prometheus target being UP only confirms that Prometheus can scrape the Blackbox Exporter.

It does not guarantee that the website probe succeeded.

I will check:

```promql
probe_success
```

Then:

```promql
probe_http_status_code
```

And:

```promql
probe_duration_seconds
```

I will also test the website directly from the exporter server:

```bash
curl -Iv https://example.com
```

I will troubleshoot DNS, routing, outbound connectivity, TLS and HTTP response behavior.

**What the interviewer expects:** Understanding the difference between exporter scrape health and the health of the endpoint being probed.

## Q5. Your team needs a production monitoring solution for 100 EC2 instances across multiple Auto Scaling Groups. How would you design it?

**Expected answer:**

I would design a monitoring architecture with:

1. Prometheus running on dedicated monitoring infrastructure.
2. Node Exporter installed on all required EC2 instances.
3. EC2 service discovery to identify instances dynamically.
4. Consistent EC2 tags for environment, application and ownership.
5. Security groups that allow scraping only from Prometheus.
6. Grafana connected to Prometheus for visualization.
7. Standard dashboards for CPU, memory, disk and network.
8. Alerting rules for important service and infrastructure conditions.
9. Alertmanager for routing notifications.
10. Retention and disk capacity planning.
11. Configuration and dashboards maintained in Git.
12. Monitoring of Prometheus and exporters themselves.

For higher availability, I would evaluate Prometheus replicas and remote storage according to availability and retention requirements.

I would also monitor the monitoring system so that failures in the observability stack do not go unnoticed.

**What the interviewer expects:** A complete design considering scalability, discovery, security, alerting, retention and reliability.

---

# Final Revision Checklist

## Core concepts

- [ ] Understand Prometheus pull-based architecture.
- [ ] Understand Grafana visualization.
- [ ] Understand exporters.
- [ ] Understand Node Exporter.
- [ ] Understand Blackbox Exporter.
- [ ] Understand PromQL.
- [ ] Understand Alertmanager.

## Hands-on skills

- [ ] Install Prometheus on Amazon Linux 2023.
- [ ] Create a Prometheus systemd service.
- [ ] Install Node Exporter.
- [ ] Configure Prometheus scrape targets.
- [ ] Connect Grafana to Prometheus.
- [ ] Import a Grafana dashboard.
- [ ] Create a CPU and memory dashboard.
- [ ] Configure Blackbox Exporter.
- [ ] Monitor an HTTP/HTTPS endpoint.
- [ ] Configure CPU alerting.
- [ ] Troubleshoot DOWN targets.
- [ ] Troubleshoot Grafana No Data.

## Production skills

- [ ] Understand service discovery.
- [ ] Understand AWS EC2 discovery.
- [ ] Understand IAM roles for discovery.
- [ ] Understand security groups and private networking.
- [ ] Understand retention limits.
- [ ] Understand metric cardinality.
- [ ] Understand dashboard provisioning.
- [ ] Understand high availability.
- [ ] Understand Alertmanager routing.

---

# Official Documentation

1. Prometheus documentation: https://prometheus.io/docs/introduction/overview/
2. Prometheus installation: https://prometheus.io/docs/installation/
3. Prometheus configuration: https://prometheus.io/docs/prometheus/latest/configuration/configuration/
4. Prometheus security: https://prometheus.io/docs/operating/security/
5. Grafana installation: https://grafana.com/docs/grafana/latest/setup-grafana/installation/
6. Grafana provisioning: https://grafana.com/docs/grafana/latest/administration/provisioning/
7. Node Exporter: https://github.com/prometheus/node_exporter
8. Blackbox Exporter: https://github.com/prometheus/blackbox_exporter

**Learning objective:** Be able to install and operate a Prometheus and Grafana monitoring stack, collect Linux infrastructure metrics, monitor websites, create dashboards, troubleshoot failures and explain production monitoring architecture in a Junior DevOps or Cloud Engineer interview.