---
title: "Week 6 – Amazon S3 & Group Project Kickoff"
weight: 6
date: 2026-05-25
---

**Period:** May 25 – May 31, 2026

---

## Overview

Week 6 introduced **Amazon Simple Storage Service (S3)** with a focus on static website hosting, along with the formal kickoff of the group workshop project. A second AWS-organized event also took place this week, providing valuable peer-learning and mentorship opportunities.

---

## Activities & Learnings

### 1. Amazon S3 Core Concepts

S3 is AWS's object storage service — infinitely scalable, highly durable (11 nines), and surprisingly versatile beyond simple file storage.

| Concept | Description |
|---|---|
| **Bucket** | A container for objects; globally unique name required |
| **Object** | A file plus its metadata; identified by a key (path-like name) |
| **Key** | The full path of an object within a bucket (e.g., `images/photo.jpg`) |
| **Storage Class** | Defines cost/retrieval trade-off (Standard, IA, Glacier, etc.) |
| **Versioning** | Preserves previous versions of objects on update or delete |
| **Bucket Policy** | JSON policy granting access to the bucket and its objects |

### 2. Amazon S3 Static Website Hosting

Deployed a static website using S3's built-in hosting feature:

**Step-by-step:**
1. Created an S3 bucket with a name matching the intended domain (e.g., `my-fcaj-site`)
2. Enabled **Static Website Hosting** in the bucket properties
3. Set `index.html` as the index document and `404.html` as the error document
4. Disabled **Block Public Access** settings to allow public reads
5. Attached a **Bucket Policy** to allow public `s3:GetObject`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-fcaj-site/*"
    }
  ]
}
```

6. Uploaded HTML, CSS, and JS files
7. Accessed the site via the S3 website endpoint URL

### 3. S3 Versioning and Lifecycle Policies

- Enabled **Versioning** on the bucket to retain all historical versions of objects
- Practiced restoring a previous version of a file after an accidental overwrite
- Created **Lifecycle Rules** to:
  - Transition objects older than 30 days to S3 Standard-IA (cheaper for infrequently accessed files)
  - Expire (delete) objects older than 365 days automatically

### 4. S3 Security

- Understood the distinction between **Bucket ACLs**, **Bucket Policies**, and **IAM Policies** for controlling S3 access
- Enabled **Server-Side Encryption (SSE-S3)** as the default encryption for all new objects
- Enabled **S3 Access Logs** to track all requests made to the bucket
- Reviewed **S3 Block Public Access** settings and understood when to enable/disable them

### 5. Group Project Kickoff

This week marked the formal beginning of the FCAJ group workshop project:
- Formed a team with fellow interns
- Held initial brainstorming session to define the project scope
- Agreed on building a **Highly Available Web Application** leveraging EC2, VPC (Multi-AZ), Auto Scaling, and RDS
- Divided responsibilities: networking, compute, database, and documentation
- Set up a shared GitHub repository for code and configuration management

### 6. AWS Knowledge Sharing Event

Attended a mid-program event with AWS mentors and FCAJ interns:
- Shared experiences from the first six weeks of labs
- Discussed common challenges with VPC and IAM configurations
- Mentors provided feedback on the group project architecture draft
- Reviewed S3 use cases beyond file storage: data lakes, CloudTrail logs, backup storage

---

## Key Takeaways

S3 is one of the most versatile services on AWS — it's not just storage, but a hosting platform, a logging target, a backup destination, and a data lake foundation. Static website hosting on S3 is a cost-effective way to deploy simple frontends without any server management. The group project kickoff this week shifted the internship into a more collaborative, product-delivery mindset.

---

## Resources Referenced

- [Amazon S3 Documentation](https://docs.aws.amazon.com/s3/)
- [Hosting a Static Website on S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- [S3 Storage Classes](https://aws.amazon.com/s3/storage-classes/)
