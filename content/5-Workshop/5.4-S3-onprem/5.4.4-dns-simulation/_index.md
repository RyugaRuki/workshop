---
title : "Inspect monitoring, queues, and operations"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4.4 </b> "
---

#### CloudWatch and SNS

The deployed stack includes alarms for Auto Scaling capacity, ALB target health and 5xx responses, EC2 CPU, all four DLQs, and Bedrock fallback. Alarm notifications publish to a KMS-encrypted SNS topic.

![CloudWatch alarms](/images/5-workshop/cloudbrief-evidence/cloudwatch-alarms.png)

![SNS notification topic](/images/5-workshop/cloudbrief-evidence/sns-topic.png)

#### Read-only audit on 16 July 2026

| Check | Result |
| --- | --- |
| CloudWatch alarms | 9 total: 5 OK, 3 insufficient data, 1 ALARM |
| SQS queues | 8 total |
| Non-empty queue | Image-processing DLQ, 3 visible messages |
| DynamoDB | 6 CloudBrief tables |
| WAF | 1 CloudFront Web ACL |
| Budget records in account | 3 |

The image-processing DLQ alarm is honest residual evidence, not a successful-green claim. It proves the alarm path is working and identifies the next operational task: inspect the failed image messages, correct retryability if needed, redrive only valid messages, and verify the DLQ returns to zero. No queue mutation was performed while writing this workshop.

#### Backup and account events

AWS Backup protects durable DynamoDB tables. Account-registration and admin role/status notifications publish only a user identifier and state; they do not include email or display-name PII.

#### Budget evidence

![AWS Budgets evidence](/images/5-workshop/cloudbrief-evidence/budget.png)

Budget notifications are part of the deployment gate because ALB, NAT Gateway, interface endpoints, WAF, and two EC2 instances are recurring cost drivers.
