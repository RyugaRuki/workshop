---
title : "Build và deploy CloudBrief"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Kiến trúc triển khai

![Kiến trúc production CloudBrief](/images/5-Workshop/cloudbrief-evidence/production-architecture.png)

CloudFront là public entry point duy nhất. AWS WAF bảo vệ distribution; S3 cung cấp private frontend qua Origin Access Control; `/api/*` được forward đến Application Load Balancer internet-facing. ALB gửi request đến hai EC2 trong private subnet. NAT Gateway dùng để lấy RSS, Hacker News và bài viết bên ngoài, còn bốn VPC endpoint giữ traffic đến SQS, Bedrock, S3 và DynamoDB trong mạng AWS.

#### Các chương triển khai

1. [Build và synth local](5.3.1-create-gwe/)
2. [Deploy production stack](5.3.2-test-gwe/)

#### Production target đã verify

| Cấu hình | Giá trị đã verify |
| --- | --- |
| Stack | `cloudbrief-dev` |
| Region | `us-east-1` |
| CloudFormation status | `UPDATE_COMPLETE` |
| Compute | 2 private EC2 trên 2 AZ |
| Entry | CloudFront + WAF + ALB |
| Network egress | 1 NAT Gateway |
| VPC endpoints | 2 interface + 2 gateway endpoints |
| Worker queues | 4 queue + 4 DLQ |
| DynamoDB | 6 bảng CloudBrief |
| Summary model | `amazon.nova-micro-v1:0`, 256 output token |

Các giá trị trên được kiểm tra bằng AWS CLI read-only ngày 16/07/2026.
