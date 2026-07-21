---
title: "Week 5 – IAM Roles & AWS Cloud9"
weight: 5
date: 2026-05-18
---

**Period:** May 18 – May 24, 2026

---

## Overview

Week 5 covered two important topics: **IAM Roles** for granting permissions to AWS services and applications (rather than users), and **AWS Cloud9** as a cloud-based integrated development environment (IDE) that eliminates local tool setup requirements.

---

## Activities & Learnings

### 1. IAM Roles — Granting Permissions to Applications

While Week 2 focused on IAM Users and Groups for human identities, Week 5 explored **IAM Roles** — the mechanism for granting AWS permissions to applications, services, and workloads.

**Key concepts:**

| Concept | Description |
|---|---|
| **Trust Policy** | Defines *who* (which principal) can assume the role |
| **Permission Policy** | Defines *what* the role is allowed to do |
| **Instance Profile** | Container that holds an IAM Role attached to an EC2 instance |
| **STS (Security Token Service)** | Issues temporary credentials when a role is assumed |

### 2. Creating and Attaching EC2 Instance Roles

- Created an IAM Role with a trust policy for the `ec2.amazonaws.com` principal
- Attached the `AmazonS3ReadOnlyAccess` managed policy to the role
- Created an **Instance Profile** and attached it to a running EC2 instance
- Verified the instance could access S3 without any configured access keys:

```bash
# On the EC2 instance — no credentials configured
aws s3 ls s3://my-internship-bucket
# This works because the Instance Role provides temporary credentials
```

**Why roles over access keys:**
- No long-lived credentials stored on the instance
- Credentials are automatically rotated by STS
- Easy to grant/revoke permissions centrally through the IAM Role

### 3. Cross-Service Role Patterns

Explored common role patterns:
- **Lambda execution role** — Lambda assumes a role to access DynamoDB, S3, etc.
- **ECS task role** — Containers assume a role for fine-grained API access
- **Cross-account access** — Role in Account A can be assumed by users/services in Account B
- **AWS service roles** — Services like Auto Scaling, CodeDeploy assume roles to take actions on your behalf

### 4. AWS Cloud9 — Browser-Based IDE

**AWS Cloud9** is a cloud-based IDE that runs on an EC2 instance and is accessible via a web browser, eliminating the need to install development tools locally.

**Setup process:**
1. Created a Cloud9 environment in the AWS Console
2. Selected an EC2 instance type (t3.small recommended)
3. Set the environment type to **SSH-connected** (to use an existing instance) or **EC2** (Cloud9 manages the instance)
4. Opened the Cloud9 IDE in the browser

**Features explored:**
- **File editor** with syntax highlighting for Python, JavaScript, YAML, etc.
- **Integrated terminal** with pre-installed AWS CLI, Node.js, Python, and Git
- **AWS Resource Explorer** panel showing EC2 instances, Lambda functions, and other resources
- **Code collaboration** support for real-time pair programming

**Practical usage:**
```bash
# AWS CLI is pre-configured in Cloud9 using the Cloud9's IAM credentials
aws sts get-caller-identity

# Deploy a simple Python Flask app from the Cloud9 terminal
pip install flask
python app.py
```

### 5. Configuring Cloud9 with Custom IAM Role

- Disabled Cloud9's default **AWS Managed Temporary Credentials**
- Attached a custom IAM Role to the Cloud9's underlying EC2 instance
- Verified that only the permissions defined in the custom role were available (least privilege)

---

## Key Takeaways

IAM Roles are the preferred mechanism for any workload-to-AWS-service communication. Using Instance Profiles eliminates the security risk of storing long-lived access keys on EC2 instances. AWS Cloud9 is an excellent tool for teams where local environment setup is a bottleneck — it provides a pre-configured, consistent development environment accessible from any browser.

---

## Resources Referenced

- [IAM Roles Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)
- [AWS Cloud9 User Guide](https://docs.aws.amazon.com/cloud9/latest/user-guide/)
- [Using Instance Profiles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html)
