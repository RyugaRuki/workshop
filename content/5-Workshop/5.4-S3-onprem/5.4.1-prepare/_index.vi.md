---
title : "Verify network và compute"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

Dùng AWS CLI read-only để quá trình verify không thay đổi deployment.

#### Stack và compute

```bash
aws cloudformation describe-stacks --profile cloudbrief-workshop \
  --region us-east-1 --stack-name cloudbrief-dev

aws autoscaling describe-auto-scaling-groups \
  --profile cloudbrief-workshop --region us-east-1

aws ec2 describe-instances --profile cloudbrief-workshop \
  --region us-east-1
```

Kết quả verify ngày 16/07/2026:

- CloudFormation status: `UPDATE_COMPLETE`.
- Auto Scaling desired/min/max capacity: `2/2/2` trên hai Availability Zone.
- Hai EC2 đang chạy; cả hai chỉ dùng private IP.
- Một Application Load Balancer internet-facing đang active.

#### Network path

```bash
aws ec2 describe-nat-gateways --profile cloudbrief-workshop \
  --region us-east-1
aws ec2 describe-vpc-endpoints --profile cloudbrief-workshop \
  --region us-east-1
```

Kết quả verify:

- Một NAT Gateway available cho traffic đến publisher bên ngoài.
- Hai interface endpoint available cho SQS và Bedrock Runtime.
- Hai gateway endpoint available cho S3 và DynamoDB.

Cách tách này giúp worker truy cập RSS/article public trong khi traffic đến AWS managed service vẫn đi trong mạng AWS.
