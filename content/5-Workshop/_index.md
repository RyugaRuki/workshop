---
title: "Workshop"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5. </b> "
---

# CloudBrief Workshop
## EC2-first Technology News Collection and Summarization System

This workshop documents my personal deployable slice of the CloudBrief group project. The goal is to show a real AWS use case from architecture through validation and cleanup, not to copy the sample workshop.

CloudBrief collects technology news from RSS feeds and Hacker News, deduplicates articles, extracts cleaned text, summarizes content with Amazon Bedrock, processes cover images to WebP, and exposes a public magazine plus authenticated reader and admin workflows.

#### Workshop content

1. [Project overview and architecture](5.1-Workshop-overview/)
2. [Prepare the AWS account and local environment](5.2-Prerequiste/)
3. [Build and deploy CloudBrief](5.3-S3-vpc/)
4. [Verify the production system](5.4-S3-onprem/)
5. [Security, cost, and architecture decisions](5.5-Policy/)
6. [Cleanup and recovery plan](5.6-Cleanup/)

#### Requirement alignment

This workshop covers the HCM project rubric:

* Real AWS use case: technology news summarization dashboard.
* More than 3 AWS services: WAF, CloudFront, S3, ALB, EC2 Auto Scaling, VPC, SQS, DynamoDB, Bedrock, EventBridge, CloudWatch, SNS, SSM, Backup, Budgets, and IAM.
* Architecture diagram and service rationale.
* Step-by-step deployment and validation guide.
* Logs, metrics, alarms, cost controls, security controls, backup, and cleanup.
