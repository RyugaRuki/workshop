---
title : "Deploy production stack"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

{{% notice warning %}}
Lệnh bootstrap và deploy sẽ tạo hoặc cập nhật tài nguyên AWS. Chỉ chạy khi đã được phê duyệt, có budget active và có rollback plan.
{{% /notice %}}

#### Bước 1: Kiểm tra identity và stack hiện tại

```bash
AWS_PROFILE=cloudbrief-workshop aws sts get-caller-identity
AWS_PROFILE=cloudbrief-workshop aws cloudformation describe-stacks \
  --region us-east-1 --stack-name cloudbrief-dev
```

Che account ID và resource identifier trước khi public output.

#### Bước 2: Bootstrap CDK

```bash
AWS_PROFILE=cloudbrief-workshop AWS_REGION=us-east-1 \
  bun run cdk bootstrap
```

#### Bước 3: Deploy CloudBrief

```bash
AWS_PROFILE=cloudbrief-workshop AWS_REGION=us-east-1 \
DEMO_API_KEY='<đọc từ protected local storage>' \
bunx cdk deploy --require-approval never \
  -c originSecretHeaderValue='<giá trị random mới>'
```

![CDK deploy hoàn tất](/images/5-Workshop/cloudbrief-evidence/cdk-deploy-ok.png)

Deployment tạo CloudFront, WAF, private S3 origin, ALB, VPC, private Auto Scaling compute, NAT, VPC endpoint, SQS queue và DLQ, DynamoDB table, EventBridge schedule, observability, SNS, Backup, Budgets, IAM và Systems Manager configuration.

#### Bước 4: Publish frontend

```bash
bun run --cwd frontend build
AWS_PROFILE=cloudbrief-workshop aws s3 sync \
  frontend/out s3://<frontend-bucket> --delete
AWS_PROFILE=cloudbrief-workshop aws cloudfront create-invalidation \
  --distribution-id <distribution-id> --paths '/*'
```

#### Bước 5: Verify production resources (step by step)

Sau deploy, kiểm tra từng resource bằng AWS CLI read-only. Các ảnh bên dưới được chụp lại ngày **18/07/2026** từ account workshop; account ID, IP private và identifier nhạy cảm đã redacted.

Đặt profile trước khi chạy các lệnh:

```bash
export AWS_PROFILE=cloudbrief-workshop
export AWS_REGION=us-east-1
```

##### 5.1 CloudFormation stack

Xác nhận stack `cloudbrief-dev` ở trạng thái ổn định.

```bash
aws cloudformation describe-stacks \
  --region us-east-1 \
  --stack-name cloudbrief-dev \
  --query 'Stacks[].{Name:StackName,Status:StackStatus}' \
  --output table
```

**Kỳ vọng:** `UPDATE_COMPLETE` (hoặc `CREATE_COMPLETE` nếu lần deploy đầu).

![Step 5.1 CloudFormation](/images/5-Workshop/cloudbrief-evidence/cli-cloudformation.png)

##### 5.2 EC2 workers

Hai worker trong private subnet, không có public IP.

```bash
aws ec2 describe-instances \
  --filters Name=instance-state-name,Values=running \
  --query 'Reservations[].Instances[].{Name:Tags[?Key==`Name`]|[0].Value,State:State.Name,Type:InstanceType,PublicIp:PublicIpAddress}' \
  --output table
```

**Kỳ vọng:** 2 instance `cloudbrief-dev-api-worker`, state `running`, `PublicIp = None`.

![Step 5.2 EC2 workers](/images/5-Workshop/cloudbrief-evidence/cli-ec2-workers.png)

##### 5.3 Auto Scaling group

```bash
aws autoscaling describe-auto-scaling-groups \
  --query 'AutoScalingGroups[].{Name:AutoScalingGroupName,Desired:DesiredCapacity,Min:MinSize,Max:MaxSize,Instances:length(Instances)}' \
  --output table
```

**Kỳ vọng:** Desired / Min / Max / Instances = `2`.

![Step 5.3 Auto Scaling group](/images/5-Workshop/cloudbrief-evidence/cli-asg.png)

##### 5.4 Application Load Balancer

```bash
aws elbv2 describe-load-balancers \
  --query 'LoadBalancers[].{Name:LoadBalancerName,Type:Type,Scheme:Scheme,State:State.Code}' \
  --output table
```

**Kỳ vọng:** type `application`, scheme `internet-facing`, state `active`.

![Step 5.4 ALB](/images/5-Workshop/cloudbrief-evidence/cli-alb.png)

##### 5.5 NAT Gateway

Worker egress ra internet (RSS, Hacker News, fetch bài) qua NAT.

```bash
aws ec2 describe-nat-gateways \
  --filter Name=state,Values=available \
  --query 'NatGateways[].{Id:NatGatewayId,State:State,Connectivity:ConnectivityType}' \
  --output table
```

**Kỳ vọng:** 1 NAT `available`, connectivity `public`.

![Step 5.5 NAT Gateway](/images/5-Workshop/cloudbrief-evidence/cli-nat-gateway.png)

##### 5.6 VPC endpoints (interface + gateway)

Traffic nội bộ AWS không đi public internet: SQS, Bedrock (Interface), S3, DynamoDB (Gateway).

```bash
aws ec2 describe-vpc-endpoints \
  --query 'VpcEndpoints[].{Service:ServiceName,Type:VpcEndpointType,State:State}' \
  --output table
```

**Kỳ vọng:** 4 endpoint, tất cả `available`:

| Service | Type |
| --- | --- |
| `sqs` | Interface |
| `bedrock-runtime` | Interface |
| `dynamodb` | Gateway |
| `s3` | Gateway |

![Step 5.6 VPC endpoints](/images/5-Workshop/cloudbrief-evidence/cli-vpc-endpoints.png)

##### 5.7 CloudFront

```bash
aws cloudfront list-distributions \
  --query 'DistributionList.Items[].{Id:Id,Status:Status,Enabled:Enabled,Origins:length(Origins.Items)}' \
  --output table
```

**Kỳ vọng:** status `Deployed`, enabled `True`, có origin (S3 frontend + ALB + image bucket).

![Step 5.7 CloudFront](/images/5-Workshop/cloudbrief-evidence/cli-cloudfront.png)

##### 5.8 AWS WAF

```bash
aws wafv2 list-web-acls \
  --scope CLOUDFRONT \
  --query 'WebACLs[].{Name:Name}' \
  --output table
```

**Kỳ vọng:** có Web ACL gắn entry path (tên có prefix `ApiEntryWebAcl...`).

![Step 5.8 AWS WAF](/images/5-Workshop/cloudbrief-evidence/cli-waf.png)

##### 5.9 S3 buckets

```bash
aws s3api list-buckets \
  --query "Buckets[?contains(Name, 'cloudbrief')].Name" \
  --output table
```

**Kỳ vọng:** frontend bucket, cleaned-content bucket, image bucket.

![Step 5.9 S3 buckets](/images/5-Workshop/cloudbrief-evidence/cli-s3-buckets.png)

##### 5.10 DynamoDB tables

```bash
aws dynamodb list-tables --output table
```

**Kỳ vọng:** 6 bảng CloudBrief (articles, search tokens, daily counters, dedupe, social, source runs).

![Step 5.10 DynamoDB](/images/5-Workshop/cloudbrief-evidence/cli-dynamodb.png)

##### 5.11 SQS queues và DLQs

```bash
aws sqs list-queues --query 'QueueUrls' --output table
```

**Kỳ vọng:** 4 queue chính + 4 DLQ (collection, extraction, image process, summary).

![Step 5.11 SQS](/images/5-Workshop/cloudbrief-evidence/cli-sqs.png)

##### 5.12 Bedrock Nova Micro

```bash
aws bedrock list-foundation-models \
  --query "modelSummaries[?modelId=='amazon.nova-micro-v1:0'].{Id:modelId,Name:modelName}" \
  --output table
```

**Kỳ vọng:** model `amazon.nova-micro-v1:0` xuất hiện (account đã được phép dùng model).

![Step 5.12 Bedrock Nova Micro](/images/5-Workshop/cloudbrief-evidence/cli-bedrock-nova-micro.png)

#### Checklist sau Bước 5

| # | Resource | Pass khi |
| --- | --- | --- |
| 5.1 | CloudFormation | `UPDATE_COMPLETE` |
| 5.2 | EC2 | 2 running, no public IP |
| 5.3 | ASG | capacity = 2 |
| 5.4 | ALB | active, internet-facing |
| 5.5 | NAT | available |
| 5.6 | VPC endpoints | 4 available |
| 5.7 | CloudFront | Deployed |
| 5.8 | WAF | Web ACL tồn tại |
| 5.9 | S3 | 3 cloudbrief buckets |
| 5.10 | DynamoDB | 6 tables |
| 5.11 | SQS | 8 queues (4 + 4 DLQ) |
| 5.12 | Bedrock | Nova Micro listed |
