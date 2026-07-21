---
title: "Week 3 – Amazon VPC & Networking"
weight: 3
date: 2026-05-04
---

**Period:** May 4 – May 10, 2026

---

## Overview

Week 3 introduced **Amazon Virtual Private Cloud (VPC)** — the networking foundation for all AWS infrastructure. This week covered designing and deploying a custom VPC with public and private subnets, routing traffic through Internet Gateways and NAT Gateways, and understanding how AWS networking differs from traditional on-premise environments. Additionally, an exciting in-person AWS event was attended mid-week.

---

## Activities & Learnings

### 1. Amazon VPC Fundamentals

Understood the core components of an AWS VPC:

| Component | Purpose |
|---|---|
| **VPC** | An isolated virtual network within an AWS region |
| **Subnet** | A range of IP addresses within a VPC (public or private) |
| **Route Table** | Rules that determine where network traffic is directed |
| **Internet Gateway (IGW)** | Enables internet access for resources in public subnets |
| **NAT Gateway** | Allows private subnet resources to initiate outbound internet connections |
| **Security Group** | Stateful firewall at the instance level |
| **Network ACL** | Stateless firewall at the subnet level |

### 2. Designing a Multi-Tier VPC Architecture

Designed and deployed a VPC with the following architecture:

```
VPC: 10.0.0.0/16
├── Public Subnet A  (10.0.1.0/24) — AZ: ap-southeast-1a
├── Public Subnet B  (10.0.2.0/24) — AZ: ap-southeast-1b
├── Private Subnet A (10.0.3.0/24) — AZ: ap-southeast-1a
└── Private Subnet B (10.0.4.0/24) — AZ: ap-southeast-1b
```

- **Internet Gateway** attached to the VPC for public subnet internet access
- **NAT Gateway** deployed in Public Subnet A to allow private instances to reach the internet for updates
- **Route Tables** configured:
  - Public route table: `0.0.0.0/0 → Internet Gateway`
  - Private route table: `0.0.0.0/0 → NAT Gateway`

### 3. Security Groups vs Network ACLs

Understood the key differences:

| Feature | Security Group | Network ACL |
|---|---|---|
| Level | Instance | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow and Deny |
| Evaluation | All rules evaluated | Rules evaluated in order |

- Created Security Groups for web tier (allow port 80/443 from anywhere), app tier (allow port 8080 from web SG only), and DB tier (allow port 3306 from app SG only)

### 4. VPC Flow Logs

- Enabled **VPC Flow Logs** to capture all IP traffic metadata flowing through the VPC
- Configured logs to be stored in an S3 bucket for analysis
- Analyzed flow log records to understand the `ACCEPT` and `REJECT` pattern

### 5. AWS Office Event — Cloud Services & AI in the Workplace

Attended an in-person event at the **AWS Vietnam office** organized for FCAJ interns:
- Toured the AWS office and met with Solutions Architects and cloud engineers
- Attended a presentation on the breadth of AWS services (compute, ML, IoT, analytics)
- Participated in a demo of **Amazon Bedrock** and **Amazon Q** for AI-assisted workflows
- Networked with fellow interns and AWS team members

This event provided motivating real-world context for the technical work being done in the labs.

---

## Key Takeaways

A well-designed VPC is the most important architectural decision you make on AWS. The separation of public and private tiers, combined with proper routing and security groups, provides defense-in-depth that protects application resources from the internet. The multi-AZ subnet design (two AZs) is the minimum for any production architecture.

---

## Resources Referenced

- [Amazon VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [VPC Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html)
- [AWS Networking Fundamentals (re:Invent Talk)](https://www.youtube.com/watch?v=hiKPPy584Mg)
