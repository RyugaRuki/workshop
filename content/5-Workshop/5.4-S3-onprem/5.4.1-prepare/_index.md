---
title : "Verify network and compute"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

Use read-only AWS CLI commands so verification cannot change the deployment.

#### Stack and compute

```bash
aws cloudformation describe-stacks --profile cloudbrief-workshop \
  --region us-east-1 --stack-name cloudbrief-dev

aws autoscaling describe-auto-scaling-groups \
  --profile cloudbrief-workshop --region us-east-1

aws ec2 describe-instances --profile cloudbrief-workshop \
  --region us-east-1
```

Verified result on 16 July 2026:

- CloudFormation status: `UPDATE_COMPLETE`.
- Auto Scaling desired/min/max capacity: `2/2/2` across two Availability Zones.
- Two EC2 instances running; both use private IP addresses only.
- One internet-facing Application Load Balancer is active.

#### Network path

```bash
aws ec2 describe-nat-gateways --profile cloudbrief-workshop \
  --region us-east-1
aws ec2 describe-vpc-endpoints --profile cloudbrief-workshop \
  --region us-east-1
```

Verified result:

- One NAT Gateway is available for external publisher traffic.
- Two interface endpoints are available for SQS and Bedrock Runtime.
- Two gateway endpoints are available for S3 and DynamoDB.

This separation lets the worker reach public RSS/article sources while AWS managed-service traffic remains on AWS networking.
