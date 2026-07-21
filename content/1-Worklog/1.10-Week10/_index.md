---
title: "Week 10 – Hybrid DNS with Amazon Route 53"
weight: 10
date: 2026-06-22
---

**Period:** June 22 – June 28, 2026

---

## Overview

Week 10 covered **Amazon Route 53** with a specific focus on **hybrid DNS** — the configuration that enables seamless name resolution between on-premise infrastructure and AWS VPC environments. This is a critical pattern for enterprise cloud adoption scenarios where not everything moves to AWS at once.

---

## Activities & Learnings

### 1. Amazon Route 53 Overview

Route 53 is AWS's highly available and scalable DNS service offering:

| Feature | Description |
|---|---|
| **Domain Registration** | Register new domains directly through Route 53 |
| **Public Hosted Zones** | DNS for internet-facing domains |
| **Private Hosted Zones** | DNS for internal VPC resources (not publicly accessible) |
| **Health Checks** | Monitor endpoint health and route traffic away from unhealthy targets |
| **Routing Policies** | Simple, Weighted, Latency-based, Failover, Geolocation, Geoproximity |
| **Resolver** | Handles DNS queries between VPCs and on-premise networks |

### 2. Public Hosted Zone Setup

- Created a **Public Hosted Zone** for a test domain
- Configured **A records** and **CNAME records** pointing to the ALB's DNS name
- Understood the distinction between **Alias records** (AWS extension, free, follows ALB/CloudFront DNS changes) vs standard **CNAME records** (standard DNS, cannot be used at zone apex)
- Tested DNS propagation using `nslookup` and `dig` tools

### 3. Private Hosted Zone for VPC

- Created a **Private Hosted Zone** associated with the custom VPC from Week 3
- Added DNS entries for internal services:
  - `app.internal` → EC2 private IP
  - `db.internal` → RDS endpoint
  - `api.internal` → Internal ALB DNS
- Verified that EC2 instances within the VPC could resolve `db.internal` while the same hostname is not resolvable from the internet

### 4. Hybrid DNS Architecture

The hybrid DNS scenario simulates an enterprise environment with both on-premise servers and AWS VPC infrastructure that need to resolve each other's hostnames.

**Architecture:**

```
On-Premise Network (192.168.0.0/16)
    ├── Corporate DNS Server (192.168.1.10)
    └── Workstations, servers using .corp domain

AWS VPC (10.0.0.0/16)
    ├── Route 53 Private Hosted Zone (.internal)
    └── Route 53 Resolver Inbound Endpoint
    └── Route 53 Resolver Outbound Endpoint
```

**Configuration steps:**

**Inbound Endpoint** (on-premise → AWS):
1. Created an **Inbound Resolver Endpoint** in the VPC with two IP addresses (for redundancy across AZs)
2. Configured the on-premise DNS server to forward queries for `.internal` domain to the Inbound Endpoint IPs
3. Tested: an on-premise workstation can now resolve `db.internal` via the Route 53 Private Hosted Zone

**Outbound Endpoint** (AWS → on-premise):
1. Created an **Outbound Resolver Endpoint** in the VPC
2. Created a **Forwarding Rule**: queries for `.corp` domain → forward to on-premise DNS server IP
3. Associated the forwarding rule with the VPC
4. Tested: EC2 instances in AWS VPC can now resolve `server01.corp` from the on-premise DNS

### 5. Route 53 Routing Policies

Explored advanced routing policies for production use:

| Policy | Use Case |
|---|---|
| **Simple** | Single resource for a domain |
| **Weighted** | A/B testing, gradual deployments (e.g., 90% old, 10% new) |
| **Latency** | Route to the AWS region with the lowest latency for the user |
| **Failover** | Active/Passive HA — Route to backup if primary health check fails |
| **Geolocation** | Route based on user's geographic location |
| **Multi-Value Answer** | Return up to 8 healthy IP addresses |

**Lab: Failover routing**
- Created two records for the same domain: one **Primary** (EC2 in ap-southeast-1) and one **Secondary** (S3 static error page)
- Attached Route 53 health checks to the primary endpoint
- Simulated failure by stopping the EC2 instance and verified traffic failed over to the S3 page within ~30 seconds

---

## Key Takeaways

Hybrid DNS is one of the most important networking concepts for enterprise AWS adoption. Companies rarely move 100% of their infrastructure to the cloud at once, so enabling seamless name resolution between on-premise and AWS is critical for any hybrid architecture. Route 53 Resolver Endpoints make this straightforward without requiring custom DNS forwarding solutions.

---

## Resources Referenced

- [Amazon Route 53 Documentation](https://docs.aws.amazon.com/route53/)
- [Route 53 Resolver for Hybrid DNS](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html)
- [Route 53 Routing Policies](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)
