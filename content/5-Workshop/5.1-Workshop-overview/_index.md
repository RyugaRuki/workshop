---
title : "Project overview and architecture"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Project overview

CloudBrief is an EC2-first AWS application for technology news collection and summarization. It is designed for students, researchers, demo evaluators, and future dashboard users who need a faster way to follow cloud/software/security news.

The application collects candidate articles from RSS feeds and Hacker News, deduplicates them, extracts cleaned article text, summarizes articles with Amazon Bedrock Nova Micro or a deterministic fallback, processes cover images to WebP, and serves a public magazine, article reader, reader community, and protected operations page.

#### Architecture diagram

![CloudBrief architecture]({{< asseturl "images\5-Workshop\cloudbrief-evidence/production-architecture.png" >}})

[Download draw.io source]({{< asseturl "attachments/cloudbrief-ec2-clean-aws.drawio" >}})

#### Main data flow

1. A reader opens the AWS WAF-protected CloudFront distribution.
2. CloudFront serves the private S3 frontend or forwards `/api/*` to the ALB.
3. The ALB distributes API traffic to two EC2 instances in private subnets.
4. EventBridge or a protected admin request queues RSS and Hacker News collection.
5. Workers extract cleaned text, process cover images to WebP, and queue summaries through SQS.
6. Amazon Bedrock Nova Micro or the deterministic fallback produces the article summary.
7. DynamoDB stores article/social state while private S3 buckets store cleaned text and processed images.
8. CloudWatch, SNS, Systems Manager, AWS Backup, Budgets, and IAM provide operations and recovery controls.

#### Personal scope

The group project is CloudBrief. My report scope is the deployable AWS workshop slice: architecture explanation, CDK deployment flow, API/dashboard validation, cost and security controls, monitoring evidence, backup evidence, and cleanup verification.
