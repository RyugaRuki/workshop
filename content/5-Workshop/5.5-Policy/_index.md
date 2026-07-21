---
title : "Security, cost, and architecture decisions"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

#### Security controls

| Boundary | Control |
| --- | --- |
| Public edge | AWS WAF in front of the CloudFront distribution |
| Frontend | Private, versioned S3 bucket through Origin Access Control |
| API origin | CloudFront origin secret and ALB routing; no direct EC2 public IP |
| Compute | Two EC2 instances in private subnets, managed by a fixed-capacity ASG |
| Host access | Systems Manager instead of inbound SSH |
| Managed services | SQS/Bedrock interface endpoints and S3/DynamoDB gateway endpoints |
| Publisher fetches | SSRF-safe fetcher blocks private, loopback, link-local, and reserved destinations |
| Article images | Validated, resized, converted to WebP, and stored in a private S3 bucket |
| Admin API | HttpOnly admin session; static frontend contains no `x-api-key` |
| Reader data | Public reading; authenticated likes/comments; owner checks for edits/deletes |
| Alerts | KMS-encrypted SNS topic with limited EC2 publish permission |

#### IAM lesson

The 5.2 screenshots show the temporary `AdministratorAccess` policy used for the original student deployment. That is acceptable as historical evidence but too broad as a durable production control. The recommended improvement is a dedicated CDK deployment role with only the service actions and `iam:PassRole` resources required by this stack, MFA for console access, and short-lived credentials through IAM Identity Center where available.

#### Cost model

The current architecture prioritizes high availability and private compute over the cheapest possible demo:

- Two `t4g.small` EC2 instances.
- One Application Load Balancer.
- One NAT Gateway.
- Two paid interface endpoints plus two gateway endpoints.
- CloudFront, WAF, S3, DynamoDB on-demand, SQS, CloudWatch, SNS, Backup, and Bedrock usage.

Cost controls include Nova Micro, a 256-token output cap, article caps, short log retention, S3 lifecycle rules, on-demand DynamoDB, budget alerts, and a documented cleanup procedure.

#### Architecture tradeoff

The earlier demo topology was cheaper but did not match the final architecture. The deployed design adds ALB, private subnets, ASG redundancy, WAF, and service endpoints. It costs more, but gives a clearer production story: public edge protection, no public instance IP, two-AZ compute, bounded outbound access, queue isolation, and observable failure handling.
