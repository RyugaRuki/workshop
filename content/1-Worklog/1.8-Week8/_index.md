---
title: "Week 8 – Cost Optimization & Auto Scaling"
weight: 8
date: 2026-06-08
---

**Period:** June 8 – June 14, 2026

---

## Overview

Week 8 focused on two complementary topics: **Amazon Lightsail** as a simplified, cost-predictable alternative to EC2 for smaller workloads, and **EC2 Auto Scaling** to automatically adjust compute capacity in response to demand changes.

---

## Activities & Learnings

### 1. Amazon Lightsail — Simplified Cloud Computing

**Amazon Lightsail** is AWS's simplified cloud platform designed for developers, small businesses, and students who need straightforward virtual servers, databases, storage, and networking at predictable monthly prices.

**Lightsail vs EC2 comparison:**

| Feature | Amazon Lightsail | Amazon EC2 |
|---|---|---|
| Pricing | Fixed monthly bundles | Pay-per-second, complex |
| Configuration | Simplified wizard | Full control, many options |
| Networking | Simplified (built-in firewall) | Full VPC control |
| Use case | Simple apps, blogs, dev/test | Production workloads, enterprise |
| Free Tier | 3 months free on smallest bundle | 750 hours/month t2.micro |

**Lab: Deploy a WordPress site on Lightsail:**
1. Created a Lightsail instance with the WordPress blueprint ($3.50/month plan)
2. Connected via the browser-based SSH terminal
3. Configured the WordPress site with a custom domain using Lightsail's static IP
4. Explored Lightsail's simplified firewall rules (IPv4 + IPv6)
5. Created a Lightsail snapshot for backup

**Cost optimization insight:** For simple workloads like blogs or small APIs, Lightsail can be significantly cheaper than EC2 with its all-inclusive pricing (compute + storage + data transfer bundled).

### 2. EC2 Auto Scaling — Dynamic Capacity Management

**EC2 Auto Scaling** automatically adds or removes EC2 instances based on demand, ensuring availability during traffic spikes while minimizing cost during quiet periods.

**Key components:**

| Component | Purpose |
|---|---|
| **Launch Template** | Defines the instance configuration (AMI, type, SG, IAM role) |
| **Auto Scaling Group (ASG)** | The fleet of instances managed as a group |
| **Scaling Policy** | Rules for when and how to scale in/out |
| **Desired Capacity** | The target number of instances |
| **Minimum/Maximum** | Bounds for the ASG size |

### 3. Creating an Auto Scaling Setup

**Step-by-step lab:**

1. Created a **Launch Template** based on the Amazon Linux AMI with a User Data script:
```bash
#!/bin/bash
sudo dnf install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
INSTANCE_ID=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id)
echo "<h1>Served from Instance: $INSTANCE_ID</h1>" | sudo tee /var/www/html/index.html
```

2. Created an **Auto Scaling Group** with:
   - Min: 1, Desired: 2, Max: 4 instances
   - Spanning two availability zones (Multi-AZ)
   - Attached to an **Application Load Balancer (ALB)**
   - Health check grace period: 300 seconds

3. Configured **Scaling Policies:**
   - **Target Tracking Policy:** Maintain average CPU utilization at 50%
   - **Scheduled Scaling:** Scale out to 3 instances on weekday mornings, scale in on evenings

4. Tested scaling by using the AWS CLI to trigger a CPU stress test on one instance and observing the ASG launch new instances automatically

### 4. Elastic Load Balancing (ALB)

Deployed an **Application Load Balancer** in front of the Auto Scaling Group:
- Created a Target Group with health check on port 80, path `/`
- Registered the Auto Scaling Group instances with the Target Group
- Configured a Listener on port 80 forwarding to the Target Group
- Tested that traffic was evenly distributed across instances (visible from the Instance ID in the page)

### 5. Cost Optimization Strategies Reviewed

| Strategy | Savings Potential |
|---|---|
| Auto Scaling (scale in during off-peak) | 30–70% on compute |
| Reserved Instances (1-year, no upfront) | ~40% vs On-Demand |
| Spot Instances (for fault-tolerant workloads) | Up to 90% vs On-Demand |
| Rightsizing (select correct instance type) | 10–40% |
| Graviton instances (ARM-based) | 20% better price/performance |

---

## Key Takeaways

Auto Scaling is essential for any production workload where traffic is not perfectly flat. The combination of ALB + Auto Scaling Group is the standard pattern for horizontally scalable web applications on AWS. Lightsail is an underused option for cost optimization on simpler workloads that don't need the full flexibility of EC2.

---

## Resources Referenced

- [Amazon Lightsail Documentation](https://docs.aws.amazon.com/lightsail/)
- [EC2 Auto Scaling Documentation](https://docs.aws.amazon.com/autoscaling/ec2/userguide/)
- [AWS Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/)
