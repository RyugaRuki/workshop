---
title : "Cleanup and recovery plan"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

{{% notice warning %}}
The production demo is still live. The commands below are a reviewed cleanup procedure and were not executed while writing this workshop.
{{% /notice %}}

#### 1. Preserve evidence

Before cleanup, export sanitized proof for CloudFormation, CloudFront/WAF, ALB target health, ASG instances, VPC endpoints, queues/DLQs, DynamoDB, S3, CloudWatch, SNS, Backup, Budgets, API responses, and browser tests.

#### 2. Resolve outstanding operations

The 16 July audit found three visible messages in the image-processing DLQ. Inspect and classify those messages before destroy so the report records the failure cause and remediation decision. Do not publish article content from a failed message.

#### 3. Destroy the CDK stack

```bash
cd ~/Code/Technology-News-Collection-and-Summarization-System
AWS_PROFILE=cloudbrief-workshop AWS_REGION=us-east-1 \
  bun run cdk destroy
```

This is destructive and requires explicit approval at execution time.

#### 4. Handle retained data

If CDK intentionally retains versioned S3 objects, backup recovery points, or other durable data, decide whether each item must be preserved for evidence or deleted. Emptying a versioned bucket requires removing both object versions and delete markers.

#### 5. Verify cleanup read-only

Confirm that expected resources no longer exist:

- CloudFormation stack, CloudFront distribution, and WAF Web ACL.
- ALB, target group, ASG, EC2 instances, EBS volumes, and NAT Gateway.
- VPC endpoints, SQS queues/DLQs, DynamoDB tables, and S3 buckets.
- CloudWatch alarms/log groups, SNS topic, Backup resources, and project budget.

#### Current status

Cleanup is **pending** because CloudBrief remains the active production demo. The report must not claim cleanup success until destroy is approved, executed, and verified.
