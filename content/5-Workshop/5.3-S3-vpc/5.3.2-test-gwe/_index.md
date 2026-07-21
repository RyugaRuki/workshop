---
title : "Deploy the production stack"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

{{% notice warning %}}
The bootstrap and deploy commands create or update AWS resources. Run them only with explicit approval, an active budget, and a rollback plan.
{{% /notice %}}

#### Step 1: Verify identity and current stack

```bash
AWS_PROFILE=cloudbrief-workshop aws sts get-caller-identity
AWS_PROFILE=cloudbrief-workshop aws cloudformation describe-stacks \
  --region us-east-1 --stack-name cloudbrief-dev
```

Redact the account ID and resource identifiers before publishing output.

#### Step 2: Bootstrap CDK

```bash
AWS_PROFILE=cloudbrief-workshop AWS_REGION=us-east-1 \
  bun run cdk bootstrap
```

#### Step 3: Deploy CloudBrief

```bash
AWS_PROFILE=cloudbrief-workshop AWS_REGION=us-east-1 \
DEMO_API_KEY='<loaded from protected local storage>' \
bunx cdk deploy --require-approval never \
  -c originSecretHeaderValue='<new random value>'
```

![CDK deployment completed](/images/5-Workshop/cloudbrief-evidence/cdk-deploy-ok.png)

The deployment creates CloudFront, WAF, private S3 origins, ALB, VPC, private Auto Scaling compute, NAT, VPC endpoints, SQS queues and DLQs, DynamoDB tables, EventBridge scheduling, observability, SNS, Backup, Budgets, IAM, and Systems Manager configuration.

#### Step 4: Publish the frontend

```bash
bun run --cwd frontend build
AWS_PROFILE=cloudbrief-workshop aws s3 sync \
  frontend/out s3://<frontend-bucket> --delete
AWS_PROFILE=cloudbrief-workshop aws cloudfront create-invalidation \
  --distribution-id <distribution-id> --paths '/*'
```

#### Step 5: Verify production resources (step by step)

After deploy, verify each resource with read-only AWS CLI. The screenshots below were recaptured on **18 July 2026** from the workshop account; account IDs, private IPs, and sensitive identifiers are redacted.

Set the profile before running the commands:

```bash
export AWS_PROFILE=cloudbrief-workshop
export AWS_REGION=us-east-1
```

##### 5.1 CloudFormation stack

Confirm stack `cloudbrief-dev` is in a stable state.

```bash
aws cloudformation describe-stacks \
  --region us-east-1 \
  --stack-name cloudbrief-dev \
  --query 'Stacks[].{Name:StackName,Status:StackStatus}' \
  --output table
```

**Expected:** `UPDATE_COMPLETE` (or `CREATE_COMPLETE` on first deploy).

![Step 5.1 CloudFormation](/images/5-Workshop/cloudbrief-evidence/cli-cloudformation.png)

##### 5.2 EC2 workers

Two workers in private subnets with no public IP.

```bash
aws ec2 describe-instances \
  --filters Name=instance-state-name,Values=running \
  --query 'Reservations[].Instances[].{Name:Tags[?Key==`Name`]|[0].Value,State:State.Name,Type:InstanceType,PublicIp:PublicIpAddress}' \
  --output table
```

**Expected:** 2 `cloudbrief-dev-api-worker` instances, state `running`, `PublicIp = None`.

![Step 5.2 EC2 workers](/images/5-Workshop/cloudbrief-evidence/cli-ec2-workers.png)

##### 5.3 Auto Scaling group

```bash
aws autoscaling describe-auto-scaling-groups \
  --query 'AutoScalingGroups[].{Name:AutoScalingGroupName,Desired:DesiredCapacity,Min:MinSize,Max:MaxSize,Instances:length(Instances)}' \
  --output table
```

**Expected:** Desired / Min / Max / Instances = `2`.

![Step 5.3 Auto Scaling group](/images/5-Workshop/cloudbrief-evidence/cli-asg.png)

##### 5.4 Application Load Balancer

```bash
aws elbv2 describe-load-balancers \
  --query 'LoadBalancers[].{Name:LoadBalancerName,Type:Type,Scheme:Scheme,State:State.Code}' \
  --output table
```

**Expected:** type `application`, scheme `internet-facing`, state `active`.

![Step 5.4 ALB](/images/5-Workshop/cloudbrief-evidence/cli-alb.png)

##### 5.5 NAT Gateway

Worker egress for RSS, Hacker News, and article fetches goes through NAT.

```bash
aws ec2 describe-nat-gateways \
  --filter Name=state,Values=available \
  --query 'NatGateways[].{Id:NatGatewayId,State:State,Connectivity:ConnectivityType}' \
  --output table
```

**Expected:** 1 NAT `available`, connectivity `public`.

![Step 5.5 NAT Gateway](/images/5-Workshop/cloudbrief-evidence/cli-nat-gateway.png)

##### 5.6 VPC endpoints (interface + gateway)

Internal AWS traffic stays off the public internet: SQS and Bedrock (Interface), S3 and DynamoDB (Gateway).

```bash
aws ec2 describe-vpc-endpoints \
  --query 'VpcEndpoints[].{Service:ServiceName,Type:VpcEndpointType,State:State}' \
  --output table
```

**Expected:** 4 endpoints, all `available`:

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

**Expected:** status `Deployed`, enabled `True`, origins present (S3 frontend + ALB + image bucket).

![Step 5.7 CloudFront](/images/5-Workshop/cloudbrief-evidence/cli-cloudfront.png)

##### 5.8 AWS WAF

```bash
aws wafv2 list-web-acls \
  --scope CLOUDFRONT \
  --query 'WebACLs[].{Name:Name}' \
  --output table
```

**Expected:** a Web ACL on the entry path (name prefix `ApiEntryWebAcl...`).

![Step 5.8 AWS WAF](/images/5-Workshop/cloudbrief-evidence/cli-waf.png)

##### 5.9 S3 buckets

```bash
aws s3api list-buckets \
  --query "Buckets[?contains(Name, 'cloudbrief')].Name" \
  --output table
```

**Expected:** frontend, cleaned-content, and image buckets.

![Step 5.9 S3 buckets](/images/5-Workshop/cloudbrief-evidence/cli-s3-buckets.png)

##### 5.10 DynamoDB tables

```bash
aws dynamodb list-tables --output table
```

**Expected:** 6 CloudBrief tables (articles, search tokens, daily counters, dedupe, social, source runs).

![Step 5.10 DynamoDB](/images/5-Workshop/cloudbrief-evidence/cli-dynamodb.png)

##### 5.11 SQS queues and DLQs

```bash
aws sqs list-queues --query 'QueueUrls' --output table
```

**Expected:** 4 main queues + 4 DLQs (collection, extraction, image process, summary).

![Step 5.11 SQS](/images/5-Workshop/cloudbrief-evidence/cli-sqs.png)

##### 5.12 Bedrock Nova Micro

```bash
aws bedrock list-foundation-models \
  --query "modelSummaries[?modelId=='amazon.nova-micro-v1:0'].{Id:modelId,Name:modelName}" \
  --output table
```

**Expected:** model `amazon.nova-micro-v1:0` is listed (account has model access).

![Step 5.12 Bedrock Nova Micro](/images/5-Workshop/cloudbrief-evidence/cli-bedrock-nova-micro.png)

#### Checklist after Step 5

| # | Resource | Pass when |
| --- | --- | --- |
| 5.1 | CloudFormation | `UPDATE_COMPLETE` |
| 5.2 | EC2 | 2 running, no public IP |
| 5.3 | ASG | capacity = 2 |
| 5.4 | ALB | active, internet-facing |
| 5.5 | NAT | available |
| 5.6 | VPC endpoints | 4 available |
| 5.7 | CloudFront | Deployed |
| 5.8 | WAF | Web ACL exists |
| 5.9 | S3 | 3 cloudbrief buckets |
| 5.10 | DynamoDB | 6 tables |
| 5.11 | SQS | 8 queues (4 + 4 DLQ) |
| 5.12 | Bedrock | Nova Micro listed |
