---
title : "Build and deploy CloudBrief"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Deployment architecture

![CloudBrief production architecture](/images/5-Workshop/cloudbrief-evidence/production-architecture.png)

CloudFront is the single public entry point. AWS WAF protects the distribution; S3 serves the private frontend through Origin Access Control; `/api/*` is forwarded to an internet-facing Application Load Balancer. The ALB targets two EC2 instances in private subnets. A NAT Gateway handles external RSS, Hacker News, and article fetches, while four VPC endpoints keep SQS, Bedrock, S3, and DynamoDB traffic on AWS networking.

#### Deployment chapters

1. [Build and synthesize locally](5.3.1-create-gwe/)
2. [Deploy the production stack](5.3.2-test-gwe/)

#### Verified production target

| Setting | Verified value |
| --- | --- |
| Stack | `cloudbrief-dev` |
| Region | `us-east-1` |
| CloudFormation status | `UPDATE_COMPLETE` |
| Compute | 2 private EC2 instances across 2 AZs |
| Entry | CloudFront + WAF + ALB |
| Network egress | 1 NAT Gateway |
| VPC endpoints | 2 interface + 2 gateway endpoints |
| Worker queues | 4 queues + 4 DLQs |
| DynamoDB | 6 CloudBrief tables |
| Summary model | `amazon.nova-micro-v1:0`, 256 output tokens |

These values were checked with read-only AWS CLI commands on 16 July 2026.
