---
title: "Week 9 – Amazon CloudWatch Monitoring"
weight: 9
date: 2026-06-15
---

**Period:** June 15 – June 21, 2026

---

## Overview

Week 9 was dedicated to **Amazon CloudWatch** — AWS's comprehensive monitoring and observability service. This week covered setting up metrics collection, creating alarms for automated alerting, building operational dashboards, and understanding log aggregation with CloudWatch Logs.

---

## Activities & Learnings

### 1. Amazon CloudWatch Overview

CloudWatch is the central observability platform for AWS, providing:

| Capability | Description |
|---|---|
| **Metrics** | Numeric data points collected from AWS services and custom applications |
| **Alarms** | Automated notifications and actions triggered when metrics breach thresholds |
| **Logs** | Centralized log storage, search, and analysis |
| **Dashboards** | Visual monitoring panels combining multiple metrics and alarms |
| **Events / EventBridge** | Rule-based event routing for automated workflows |
| **Insights** | Automated anomaly detection and query tools for logs |

### 2. CloudWatch Metrics

**Default metrics collected automatically:**

| Service | Key Metrics |
|---|---|
| EC2 | CPUUtilization, NetworkIn, NetworkOut, StatusCheckFailed |
| RDS | CPUUtilization, FreeStorageSpace, DatabaseConnections, ReadLatency |
| S3 | BucketSizeBytes, NumberOfObjects, AllRequests |
| ALB | RequestCount, HTTPCode_ELB_5XX, TargetResponseTime |
| Auto Scaling | GroupInServiceInstances, GroupDesiredCapacity |

**Custom metrics via CloudWatch Agent:**
- Installed the **CloudWatch Agent** on EC2 instances
- Configured the agent to collect memory utilization (not available by default) and disk usage
- Pushed custom application metrics to CloudWatch using the `put-metric-data` CLI command:

```bash
aws cloudwatch put-metric-data \
  --namespace "MyApp/Performance" \
  --metric-name "ActiveUsers" \
  --value 42 \
  --unit Count \
  --region ap-southeast-1
```

### 3. CloudWatch Alarms

Created alarms to automatically alert on abnormal conditions:

**EC2 CPU High alarm:**
- Metric: `CPUUtilization`
- Threshold: Greater than 80% for 3 consecutive 5-minute periods
- Action: Send SNS notification to email + trigger Auto Scaling policy

**RDS Free Storage Low alarm:**
- Metric: `FreeStorageSpace`
- Threshold: Less than 2 GB
- Action: SNS notification to on-call team

**Budget Alarm (billing):**
- Metric: `EstimatedCharges`
- Threshold: $8 USD
- Action: Email notification

**Alarm States:**
- `OK` — Metric is within the acceptable range
- `ALARM` — Metric has breached the threshold
- `INSUFFICIENT_DATA` — Not enough data to evaluate (e.g., instance just launched)

### 4. CloudWatch Logs

- Configured EC2 instances to stream application logs to **CloudWatch Logs** using the CloudWatch Agent
- Created **Log Groups** for different applications and environments
- Used **Metric Filters** to extract numeric metrics from log data (e.g., count of `ERROR` log lines)
- Used **CloudWatch Logs Insights** to query logs with a SQL-like syntax:

```
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 50
```

### 5. CloudWatch Dashboards

Built an operational dashboard combining:
- EC2 CPU utilization across all instances in the Auto Scaling Group
- ALB request count and 5XX error rate
- RDS CPU and free storage
- Custom "Active Users" metric
- Alarm status widgets

The dashboard was shared with the team mentor for project review.

### 6. Practical Monitoring Strategy

Applied the **USE method** (Utilization, Saturation, Errors) to define metrics for each resource:

| Resource | Utilization | Saturation | Errors |
|---|---|---|---|
| EC2 | CPUUtilization | NetworkPacketsDropped | StatusCheckFailed |
| RDS | CPUUtilization | DatabaseConnections | FailedLoginAttempts |
| ALB | ActiveConnectionCount | RejectedConnectionCount | HTTPCode_ELB_5XX |

---

## Key Takeaways

Monitoring is not optional for any production system — it's how you know your application is healthy before your customers tell you it's broken. CloudWatch provides a comprehensive observability stack without requiring any third-party tools. The most important thing is to define alerting thresholds *before* going to production, not after the first incident.

---

## Resources Referenced

- [Amazon CloudWatch Documentation](https://docs.aws.amazon.com/cloudwatch/)
- [CloudWatch Agent Configuration](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)
- [CloudWatch Logs Insights Query Syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html)
