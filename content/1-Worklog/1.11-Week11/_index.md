---
title: "Week 11 – AWS CLI & Workshop Sprint"
weight: 11
date: 2026-06-29
---

**Period:** June 29 – July 5, 2026

---

## Overview

Week 11 marked the transition from individual skill-building to final delivery preparation. This week covered practical **AWS CLI** usage on both Windows and Ubuntu EC2 instances, intensive work on the group workshop project, and began the process of assembling the final internship report.

---

## Activities & Learnings

### 1. AWS CLI on EC2 — Linux (Ubuntu)

Practiced using the **AWS CLI v2** on an Ubuntu EC2 instance for common infrastructure operations:

**Installation:**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

**Common operations practiced:**

```bash
# List EC2 instances with filtering
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query "Reservations[].Instances[].[InstanceId,InstanceType,PublicIpAddress]" \
  --output table

# Create an S3 bucket and upload files
aws s3 mb s3://my-cli-test-bucket-$(date +%s)
aws s3 cp ./index.html s3://my-cli-test-bucket/
aws s3 sync ./website/ s3://my-cli-test-bucket/

# Manage security groups
aws ec2 describe-security-groups \
  --query "SecurityGroups[?GroupName=='web-sg'].[GroupId,Description]"

# IAM operations
aws iam list-users --query "Users[].UserName" --output table
aws iam get-account-summary
```

### 2. AWS CLI on EC2 — Windows (PowerShell)

Configured and used the AWS CLI on a **Windows Server EC2** instance:

**Installation on Windows:**
```powershell
# Download and install AWS CLI MSI
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi /quiet
aws --version
```

**PowerShell-specific patterns:**

```powershell
# List S3 buckets
aws s3 ls

# Get EC2 instance metadata (IMDSv2)
$token = Invoke-RestMethod -Method PUT `
  -Uri "http://169.254.169.254/latest/api/token" `
  -Headers @{"X-aws-ec2-metadata-token-ttl-seconds"="21600"}

$instanceId = Invoke-RestMethod -Uri "http://169.254.169.254/latest/meta-data/instance-id" `
  -Headers @{"X-aws-ec2-metadata-token"=$token}

Write-Output "Instance ID: $instanceId"

# Create EBS snapshot via CLI
aws ec2 create-snapshot `
  --volume-id vol-0123456789abcdef0 `
  --description "Weekly backup snapshot"
```

### 3. CLI Scripting and Automation

Used the CLI to automate repetitive operations:

```bash
#!/bin/bash
# Automated nightly backup script
INSTANCE_ID=$(curl -sf http://169.254.169.254/latest/meta-data/instance-id)
VOLUMES=$(aws ec2 describe-volumes \
  --filters "Name=attachment.instance-id,Values=$INSTANCE_ID" \
  --query "Volumes[].VolumeId" \
  --output text)

for VOLUME_ID in $VOLUMES; do
  aws ec2 create-snapshot \
    --volume-id "$VOLUME_ID" \
    --description "Auto backup - $(date +%Y-%m-%d)" \
    --tag-specifications "ResourceType=snapshot,Tags=[{Key=Name,Value=auto-backup},{Key=Date,Value=$(date +%Y-%m-%d)}]"
  echo "Snapshot created for $VOLUME_ID"
done
```

### 4. Group Workshop Sprint

The majority of Week 11 was spent on intensive group workshop development:

- **Architecture finalization:** Reviewed and locked the final architecture diagram (VPC + Multi-AZ subnets + ALB + Auto Scaling + RDS Multi-AZ)
- **Infrastructure deployment:** Each team member deployed their assigned component
- **Integration testing:** Connected the application tier to RDS, verified load balancing across AZs, tested Auto Scaling policies
- **Documentation:** Started writing the workshop documentation (this site)
- **Code review:** Reviewed each other's CloudFormation snippets and User Data scripts

**Key decisions made this week:**
- Used RDS with Multi-AZ for the database tier (not Aurora, to stay within Free Tier)
- Stored database credentials in Secrets Manager (not environment variables)
- Deployed the ALB access logs to S3 for audit trail
- Added CloudWatch alarms to the group project for operational visibility

### 5. Internship Report Compilation Begins

Started assembling the final internship report:
- Organized weekly log entries (markdown notes compiled throughout the program)
- Drafted the self-assessment section
- Outlined the workshop project documentation structure
- Requested supervisor review of the draft proposal section

---

## Key Takeaways

The AWS CLI is an indispensable tool for cloud engineers — it enables automation, scripting, and rapid iteration that the GUI console simply cannot match. The ability to write reliable CLI scripts for routine tasks (backups, audits, deployments) is a fundamental production skill. The workshop sprint this week reinforced the value of architecture review and integration testing before the final delivery deadline.

---

## Resources Referenced

- [AWS CLI v2 Documentation](https://docs.aws.amazon.com/cli/latest/userguide/)
- [AWS CLI Command Reference](https://awscli.amazonaws.com/v2/documentation/api/latest/index.html)
- [Querying JSON with JMESPath](https://jmespath.org/tutorial.html)
