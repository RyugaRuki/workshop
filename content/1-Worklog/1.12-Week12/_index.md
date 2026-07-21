---
title: "Week 12 – Final Delivery"
weight: 12
date: 2026-07-06
---

**Period:** July 6 – July 30, 2026

---

## Overview

Week 12 was the culminating week of the FCAJ internship — a demanding final sprint that brought together every skill and concept learned over the previous 11 weeks. All remaining lab work was completed, the group workshop project reached its final production-ready state, and the internship report was completed and submitted.

---

## Activities & Learnings

### 1. Completion of All Remaining Labs

Dedicated the first part of the week to closing out any remaining FCAJ curriculum lab tasks:

- ✅ Verified completion of all 11 previous weekly labs
- ✅ Reviewed and re-ran any labs with unclear outcomes to confirm understanding
- ✅ Documented all lab resources cleanly for inclusion in the final report
- ✅ Checked AWS account for any resources requiring cleanup or final snapshots

### 2. Highly Available Web Application — Final Workshop

The group workshop project was brought to its **production-ready** final state:

**Final Architecture:**

```
                        Internet
                           │
                    ┌──────┴──────┐
                    │    Route 53  │
                    │  (DNS + HCs) │
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    │     ALB      │  ← Public Subnets (AZ-a, AZ-b)
                    │(Multi-AZ LB) │
                    └──────┬──────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
     ┌─────┴───┐     ┌─────┴───┐          Auto
     │  EC2 #1 │     │  EC2 #2 │  ←─────  Scaling
     │  AZ-a   │     │  AZ-b   │          Group
     └─────┬───┘     └─────┬───┘
           │               │
    Private Subnets (AZ-a, AZ-b)
           │
    ┌──────┴──────┐
    │  RDS MySQL   │  ← Multi-AZ
    │  Primary     │     Standby
    └─────────────┘
```

**Final checklist completed:**
- ✅ VPC with 4 subnets across 2 AZs (2 public, 2 private)
- ✅ Internet Gateway and NAT Gateways deployed
- ✅ Application Load Balancer with health checks on `/health`
- ✅ Auto Scaling Group (min 2, desired 2, max 6) with target-tracking on CPU
- ✅ RDS MySQL with Multi-AZ standby
- ✅ Database credentials stored in AWS Secrets Manager
- ✅ CloudWatch alarms for CPU, ALB 5XX errors, and RDS free storage
- ✅ CloudWatch Dashboard for operational visibility
- ✅ S3 bucket for ALB access logs and application assets
- ✅ IAM roles with least-privilege policies for all components
- ✅ Security Groups following the principle of minimal exposure

### 3. Final Group Project Adjustments

- **Performance testing:** Used Apache Bench (`ab`) to simulate load and verify Auto Scaling fired correctly
- **Failover testing:** Rebooted the RDS primary instance and confirmed failover to standby completed within 90 seconds
- **Security review:** Ran AWS Trusted Advisor and addressed all High severity recommendations
- **Cost analysis:** Used AWS Cost Explorer to calculate the estimated monthly cost of the architecture (~$45 USD/month for the lab setup, excluding Free Tier)

```bash
# Load test: 1000 requests, 50 concurrent
ab -n 1000 -c 50 http://my-alb-dns-name.ap-southeast-1.elb.amazonaws.com/

# Verify RDS failover time
aws rds reboot-db-instance \
  --db-instance-identifier myapp-db \
  --force-failover
```

### 4. Internship Report Completion

The final internship report was completed and submitted:

- **Worklog:** 12 weekly entries with detailed activities and learnings
- **Proposal:** Program objectives and achievement review
- **Events:** Documentation of the three AWS events attended
- **Workshop:** Full documentation of the group project
- **Self-Assessment:** Honest evaluation of growth and areas for improvement
- **Sharing & Feedback:** Recommendations for future interns and program feedback

### 5. Resource Cleanup

Following AWS best practices, all lab resources were terminated to prevent ongoing charges:

```bash
# Systematic cleanup order (reverse dependency order)
# 1. Terminate EC2 instances (Auto Scaling Group)
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --min-size 0 --desired-capacity 0

# 2. Delete RDS instance (final snapshot created first)
aws rds delete-db-instance \
  --db-instance-identifier myapp-db \
  --final-db-snapshot-identifier myapp-db-final-snapshot

# 3. Delete NAT Gateways (billing stops immediately)
aws ec2 delete-nat-gateway --nat-gateway-id nat-xxxx

# 4. Release Elastic IPs
aws ec2 release-address --allocation-id eipalloc-xxxx
```

---

## Program Completion Summary

| Metric | Result |
|---|---|
| **Total Labs Completed** | 12 weekly labs + group workshop |
| **AWS Services Mastered** | 12+ (IAM, VPC, EC2, S3, RDS, Route 53, CloudWatch, Auto Scaling, ALB, Lightsail, Cloud9, Secrets Manager) |
| **Events Attended** | 3 AWS events |
| **Group Project** | Highly Available Web Application — fully deployed and tested |
| **Duration** | 12 weeks (April 20 – July 12, 2025) |

---

## Key Takeaways

The final week reinforced that building on AWS is an iterative process — you deploy, test, optimize, and document. The group project served as a synthesis of every concept from the program: networking (VPC), compute (EC2 + Auto Scaling), storage (S3 + EBS), database (RDS), DNS (Route 53), monitoring (CloudWatch), and security (IAM + Security Groups). Completing this internship provides a solid foundation for a career in cloud engineering.

---

> *End of Worklog — Thank you for following along this 12-week journey!*
