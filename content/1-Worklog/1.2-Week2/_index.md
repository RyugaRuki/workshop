---
title: "Week 2 – AWS Identity and Access Management (IAM)"
weight: 2
date: 2026-04-27
---

**Period:** April 27 – May 3, 2026

---

## Overview

Week 2 was dedicated to **AWS Identity and Access Management (IAM)** — the backbone of AWS security. This week focused on understanding how AWS controls *who* can do *what* with *which* resources, and implementing least-privilege access patterns.

---

## Activities & Learnings

### 1. IAM Core Concepts

Studied the fundamental IAM entities and their relationships:

| Entity | Description |
|---|---|
| **Root Account** | The master account; should only be used for billing and account management |
| **IAM User** | A named identity with long-term credentials (access key / password) |
| **IAM Group** | A collection of users; policies attached to a group apply to all members |
| **IAM Role** | An identity with temporary credentials; assumed by users, services, or applications |
| **IAM Policy** | A JSON document defining allowed/denied actions on specified resources |

### 2. Creating IAM Users and Groups

- Created multiple IAM users representing different job functions (developer, analyst, read-only auditor)
- Organized users into **IAM Groups** by function:
  - `Developers` — EC2, S3, RDS full access
  - `Analysts` — Read-only access to CloudWatch, S3
  - `Administrators` — AdministratorAccess managed policy
- Applied the **principle of least privilege** — granting only the minimum permissions required for each role

### 3. IAM Policies Deep Dive

- Explored the difference between:
  - **AWS Managed Policies** — Pre-built by AWS (e.g., `AmazonS3ReadOnlyAccess`)
  - **Customer Managed Policies** — Custom JSON policies created by the account owner
  - **Inline Policies** — Embedded directly in a user, group, or role (not reusable)
- Wrote custom JSON policies defining explicit Allow and Deny statements
- Used the **IAM Policy Simulator** to validate policies before attaching them

**Example custom policy (S3 read-only for a specific bucket):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::my-internship-bucket",
        "arn:aws:s3:::my-internship-bucket/*"
      ]
    }
  ]
}
```

### 4. IAM Best Practices Applied

- ✅ Enabled MFA on root account (done Week 1)
- ✅ Created individual IAM users; no shared credentials
- ✅ Applied least-privilege policies
- ✅ Rotated access keys; disabled unused keys
- ✅ Enabled **IAM Access Analyzer** to identify unintended resource access
- ✅ Reviewed the IAM credential report for all users

### 5. Hands-on Lab Completion

Completed the AWS lab: *"Introduction to AWS Identity and Access Management (IAM)"*, including:
- Creating a user with console and programmatic access
- Adding the user to a group with specific permissions
- Testing access via AWS CLI using the new user's credentials
- Verifying that the user could not perform actions outside the policy scope

---

## Key Takeaways

IAM is the most critical service to understand before working with any other AWS service. Every resource interaction in AWS flows through IAM. The most common mistakes — overly permissive policies, using root credentials, no MFA — are also the most dangerous. Building good IAM habits in Week 2 made every subsequent lab significantly easier and safer.

---

## Resources Referenced

- [AWS IAM Documentation](https://docs.aws.amazon.com/iam/)
- [IAM Policy Simulator](https://policysim.aws.amazon.com/)
- [AWS Security Best Practices](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/)
