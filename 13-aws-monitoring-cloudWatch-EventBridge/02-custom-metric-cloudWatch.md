# AWS CloudWatch Custom Metrics – Practical

## 1. What are Custom Metrics in CloudWatch?

Amazon CloudWatch provides default metrics for AWS resources such as EC2, RDS, ELB and Lambda.

However, sometimes we need to monitor application-specific information that AWS does not provide as a default metric.

For example:
- Website HTTP status codes
- Application health status
- API response time
- Number of failed login attempts
- Application-specific error counts
- Custom business metrics

These are called **Custom Metrics**.

We can publish custom metrics to CloudWatch using:
- AWS CLI
- AWS SDK
- CloudWatch Agent (for supported custom collection configurations)

For this practical, we will use a Shell Script, AWS CLI and Cron Job.

## 2. Practical Objective

Create a monitoring solution on an EC2 Linux instance that:

1. Uses an IAM role to access CloudWatch.
2. Runs a Shell Script to check website HTTP/HTTPS status.
3. Captures the HTTP response code using curl.
4. Publishes custom metrics to CloudWatch.
5. Uses a Cron Job to execute the script automatically every minute.
6. Creates a CloudWatch alarm.
7. Sends an email notification through SNS when the website becomes unavailable.

## 3. Architecture

```text
                 EC2 Linux Instance
                         |
                         |
                   IAM Role Attached
                         |
                         v
                   Shell Script
                         |
                         v
                    curl Command
                         |
                         v
              Check HTTP / HTTPS Status
                         |
                  +------+------+
                  |             |
                  v             v
              HTTP 200       HTTP 500
                  |             |
                  v             v
             Availability=1 Availability=0
                  |             |
                  +------+------+
                         |
                         v
                  AWS CLI Command
                  PutMetricData
                         |
                         v
                  Amazon CloudWatch
                         |
                         v
                   Custom Metrics
                         |
                         v
                  CloudWatch Alarm
                         |
                         v
                    Amazon SNS
                         |
                         v
                   Email Alert
```

## 4. Step 1 – Create IAM Policy

The EC2 instance needs permission to publish custom metrics to CloudWatch.

Navigate to:

```text
AWS Console → IAM → Policies → Create Policy → JSON
```

Add the following policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudwatch:PutMetricData"
      ],
      "Resource": "*"
    }
  ]
}
```

Policy name:

```text
CloudWatchCustomMetricPolicy
```

### Why do we need this policy?

The `cloudwatch:PutMetricData` permission allows the EC2 instance to publish custom metric values to Amazon CloudWatch.

The policy can be further restricted using the `cloudwatch:namespace` condition key to limit which namespaces can be published.

## 5. Step 2 – Create IAM Role and Attach to EC2

1. Navigate to IAM → Roles.
2. Click Create role.
3. Select AWS service.
4. Select EC2 as the trusted entity.
5. Attach `CloudWatchCustomMetricPolicy`.
6. Name the role:

```text
EC2-CloudWatch-CustomMetric-Role
```

7. Create the role.
8. Navigate to EC2 → Instances.
9. Select your EC2 instance.
10. Actions → Security → Modify IAM role.
11. Attach the newly created role.

Now the EC2 instance can publish custom metrics without storing AWS access keys on the server.

## 6. Step 3 – Install AWS CLI and curl

Connect to your EC2 Linux instance.

For Amazon Linux 2023:

```bash
sudo dnf install awscli curl -y
```

Verify installation:

```bash
aws --version
curl --version
```

Check the IAM identity:

```bash
aws sts get-caller-identity
```

This command verifies that the EC2 instance is using the attached IAM role.

## 7. Step 4 – Create Monitoring Directory

Create a directory for monitoring scripts and logs.

```bash
mkdir -p ~/cloudwatch-monitoring
cd ~/cloudwatch-monitoring
```

Create the Shell Script:

```bash
vi website-monitor.sh
```

## 8. Step 5 – Write the Shell Script

Add the following code:

```bash
#!/bin/bash

# Configuration
URL="https://example.com"
REGION="us-west-2"
NAMESPACE="Custom/WebsiteMonitoring"

# Capture HTTP response code
HTTP_CODE=$(curl -L -sS \
  -o /dev/null \
  -w "%{http_code}" \
  --connect-timeout 5 \
  --max-time 15 \
  "$URL")

# Determine website availability
if [[ "$HTTP_CODE" =~ ^2[0-9][0-9]$ ]]; then
    AVAILABILITY=1
else
    AVAILABILITY=0
fi

# Publish custom metrics to CloudWatch
aws cloudwatch put-metric-data \
  --namespace "$NAMESPACE" \
  --region "$REGION" \
  --metric-data "[
    {
      \"MetricName\": \"WebsiteAvailability\",
      \"Dimensions\": [
        {
          \"Name\": \"Website\",
          \"Value\": \"example.com\"
        }
      ],
      \"Value\": $AVAILABILITY,
      \"Unit\": \"None\"
    },
    {
      \"MetricName\": \"HTTPStatusCode\",
      \"Dimensions\": [
        {
          \"Name\": \"Website\",
          \"Value\": \"example.com\"
        }
      ],
      \"Value\": ${HTTP_CODE:-0},
      \"Unit\": \"None\"
    }
  ]"

echo "$(date) HTTP_CODE=$HTTP_CODE AVAILABILITY=$AVAILABILITY"
```

Replace `https://example.com` with your actual website URL.

### Understanding the Script

| Command / Variable | Purpose |
|---|---|
| curl | Sends an HTTP/HTTPS request |
| -L | Follows redirects |
| -sS | Silent mode, but shows errors |
| -o /dev/null | Discards response body |
| -w "%{http_code}" | Captures HTTP response code |
| HTTP_CODE | Stores response code |
| AVAILABILITY=1 | Website returned a 2xx response |
| AVAILABILITY=0 | Website did not return a 2xx response |
| PutMetricData | Publishes custom metrics |

### HTTP Status Examples

| HTTP Status | Meaning | Availability |
|---|---|---:|
| 200 | OK | 1 |
| 201 | Created | 1 |
| 301 | Redirect | 0 |
| 404 | Not Found | 0 |
| 500 | Internal Server Error | 0 |
| 503 | Service Unavailable | 0 |
| 000 | Connection failure / timeout | 0 |

Note: This script treats only HTTP 2xx responses as healthy. Redirects and other responses are considered unavailable. Adjust the success criteria depending on your application.

Make the script executable:

```bash
chmod +x website-monitor.sh
```

## 9. Step 6 – Execute Script Manually

Run the script:

```bash
./website-monitor.sh
```

Expected output when the website is healthy:

```text
HTTP_CODE=200 AVAILABILITY=1
```

Expected output when the website is unavailable:

```text
HTTP_CODE=500 AVAILABILITY=0
```

The script publishes two custom metrics:

1. WebsiteAvailability
2. HTTPStatusCode

Check the command exit status:

```bash
echo $?
```

A successful AWS CLI execution normally returns exit code `0`.

## 10. Step 7 – Configure Cron Job

Now automate the Shell Script using Cron.

Open the crontab editor:

```bash
crontab -e
```

Add:

```bash
* * * * * /home/ec2-user/cloudwatch-monitoring/website-monitor.sh >> /home/ec2-user/cloudwatch-monitoring/monitor.log 2>&1
```

This executes the monitoring script every minute.

Check the Cron Job:

```bash
crontab -l
```

Check execution logs:

```bash
tail -f ~/cloudwatch-monitoring/monitor.log
```

### Understanding Cron Expression

```text
* * * * * command
| | | | |
| | | | +---- Day of week
| | | +------ Month
| | +-------- Day of month
| +---------- Hour
+------------ Minute
```

`* * * * *` means execute every minute.

Note: Use the correct absolute path for the EC2 login user. Cron runs with a limited environment, so ensure AWS CLI is in its PATH and the required Region is configured or supplied explicitly.

## 11. Step 8 – Verify Custom Metrics in CloudWatch

Open AWS Management Console.

Navigate to:

```text
CloudWatch
    |
    v
Metrics
    |
    v
All Metrics
    |
    v
Custom Namespaces
    |
    v
Custom/WebsiteMonitoring
```

You should find:

### Custom Metric 1

```text
Metric Name: WebsiteAvailability
Dimension: Website = example.com
```

Values:

- 1 = Website available
- 0 = Website unavailable

### Custom Metric 2

```text
Metric Name: HTTPStatusCode
Dimension: Website = example.com
```

Values:

- 200
- 404
- 500
- 503
- 0

These metrics are published by our Shell Script through the AWS CLI.

## 12. Step 9 – Create CloudWatch Alarm

Now create an alarm to monitor website availability.

Navigate to:

```text
CloudWatch → Alarms → Create alarm
```

Select the custom metric:

| Configuration | Value |
|---|---|
| Namespace | Custom/WebsiteMonitoring |
| Metric | WebsiteAvailability |
| Dimension | Website = example.com |
| Statistic | Minimum |
| Period | 1 minute |
| Threshold Type | Static |
| Condition | Lower/Equal |
| Threshold | 0 |
| Datapoints to Alarm | 1 out of 1 |

### Why use Minimum?

We publish availability values of either 1 or 0.

If a website becomes unavailable during the evaluation period, the Minimum statistic can detect the zero value.

### Alarm Condition

```text
WebsiteAvailability <= 0
```

When the condition is met, CloudWatch changes the alarm state to ALARM.

## 13. Step 10 – Configure SNS Email Notification

Navigate to:

```text
CloudWatch → Alarms → Configure actions
```

Select:

```text
Alarm State Trigger: In alarm
```

Select an existing SNS topic or create a new topic.

Example:

```text
Topic Name: Website-Monitoring-Alerts
```

Add your email endpoint and confirm the subscription through the confirmation email.

When the website becomes unavailable:

1. The Cron Job executes the script.
2. curl detects the HTTP response.
3. The script publishes availability = 0.
4. CloudWatch evaluates the custom metric.
5. The alarm changes to ALARM.
6. SNS sends an email notification.

## 14. Complete Monitoring Flow

### Scenario A – Website Healthy

```text
Cron Job Executes
       |
       v
curl Checks Website
       |
       v
HTTP Response = 200
       |
       v
Availability = 1
       |
       v
CloudWatch Custom Metric
       |
       v
Alarm State = OK
```

### Scenario B – Website Failure

```text
Cron Job Executes
       |
       v
curl Checks Website
       |
       v
HTTP Response = 500
       |
       v
Availability = 0
       |
       v
CloudWatch Custom Metric
       |
       v
Alarm State = ALARM
       |
       v
SNS Email Notification
```

## 15. Practical Troubleshooting

| Problem | Possible Solution |
|---|---|
| AccessDeniedException | Check IAM role and PutMetricData permission |
| AWS CLI command fails | Verify AWS CLI installation and Region |
| curl returns 000 | Check network connectivity, DNS and website availability |
| Custom metric not visible | Check namespace, Region and metric dimensions |
| Cron job not executing | Check crontab, script permissions and Cron service |
| No SNS email | Confirm SNS email subscription |
| Alarm shows insufficient data | Check whether metric datapoints are being published |
| Script works manually but not in Cron | Check PATH, environment variables, absolute paths and permissions |

## 16. Production Considerations

1. Use an EC2 IAM role instead of hardcoding AWS credentials.
2. Keep IAM permissions restricted to the required CloudWatch namespace where possible.
3. Use a suitable Cron interval depending on application requirements.
4. Use meaningful metric names and dimensions.
5. Monitor script execution failures as well as website failures.
6. Avoid generating unnecessary custom metrics because custom metrics can incur additional charges.
7. Avoid excessive metric dimensions, which can create additional metric combinations.
8. Configure CloudWatch alarms with suitable evaluation periods to avoid alert fatigue.
9. Consider an external monitoring service to avoid depending entirely on the EC2 instance being monitored.
10. Use HTTPS certificate validation; do not disable TLS verification in production.

**Important limitation:** If the monitored website and monitoring script are both hosted on the same EC2 instance, a complete EC2 failure can stop the monitoring script. An external uptime monitor or independent monitoring instance provides better reliability.

---

# 17. Interview Questions

### Q1. What are CloudWatch Custom Metrics?

Custom metrics are user-defined measurements published to CloudWatch to monitor application-specific information that is not available through default AWS metrics.

### Q2. How do you create custom metrics?

We can use AWS CLI or AWS SDK and the `PutMetricData` API to publish custom metric values to CloudWatch.

### Q3. Why do we need an IAM role?

The EC2 instance needs permission to publish custom metrics. We attach an IAM role with `cloudwatch:PutMetricData` permission.

### Q4. Why do we use Cron Jobs?

Cron Jobs automate the execution of scripts at predefined intervals. In this practical, Cron executes the website monitoring script every minute.

### Q5. How do you monitor website availability?

We can use curl to capture HTTP response codes, determine whether the website is healthy, and publish the result as a custom metric to CloudWatch.

### Q6. What is PutMetricData?

PutMetricData is an Amazon CloudWatch API operation used to publish custom metric data.

### Q7. What is the difference between default and custom metrics?

Default metrics are provided by AWS services, such as EC2 CPUUtilization. Custom metrics are published by users to monitor application-specific information, such as website availability.

### Q8. Can we trigger an email when a website goes down?

Yes. We can create a CloudWatch alarm for the WebsiteAvailability custom metric and configure an SNS email notification.

---

# 18. Hands-on Assignment Checklist

- [ ] Create IAM policy for cloudwatch:PutMetricData.
- [ ] Create IAM role and attach it to EC2.
- [ ] Install AWS CLI and curl.
- [ ] Verify IAM identity using AWS STS.
- [ ] Create website-monitor.sh.
- [ ] Capture HTTP response code.
- [ ] Publish custom metrics using PutMetricData.
- [ ] Execute the script manually.
- [ ] Configure Cron Job to execute every minute.
- [ ] Verify execution through log files.
- [ ] Verify custom metrics in CloudWatch.
- [ ] Create WebsiteAvailability alarm.
- [ ] Configure SNS email notification.
- [ ] Test with an intentionally unavailable test URL.

---

# 19. Important Takeaways

- CloudWatch provides default AWS metrics and supports custom metrics.
- IAM roles provide secure AWS API access to EC2.
- curl can capture HTTP/HTTPS response codes.
- Shell scripts can collect application-specific monitoring information.
- AWS CLI PutMetricData publishes custom metrics to CloudWatch.
- Cron Jobs automate script execution.
- CloudWatch Alarms detect metric threshold breaches.
- SNS sends email notifications.
- Custom metrics help DevOps engineers monitor application health beyond infrastructure-level monitoring.

## Official Documentation

- [Amazon CloudWatch Custom Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/publishingMetrics.html)
- [PutMetricData API](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_PutMetricData.html)
- [IAM Roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
- [CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)

**Learning Goal:** Understand how to create application-level monitoring using Shell Scripting, Cron Jobs, IAM Roles, AWS CLI, CloudWatch Custom Metrics and SNS notifications.