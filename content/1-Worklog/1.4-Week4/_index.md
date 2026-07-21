---
title: "Week 4 – Amazon EC2"
weight: 4
date: 2026-05-11
---

**Period:** May 11 – May 17, 2026

---

## Overview

Week 4 was dedicated to **Amazon Elastic Compute Cloud (EC2)** — the core compute service of AWS. This week covered launching and configuring EC2 instances, connecting to them via SSH (Linux) and RDP (Windows), installing applications, and understanding instance types, storage options, and lifecycle management.

---

## Activities & Learnings

### 1. EC2 Instance Fundamentals

Reviewed the EC2 instance model:

| Concept | Description |
|---|---|
| **Instance Type** | CPU, memory, and network specification (e.g., t3.micro, m5.large) |
| **AMI** | Amazon Machine Image — the operating system and pre-installed software |
| **Key Pair** | RSA/ED25519 key pair for SSH authentication on Linux instances |
| **EBS Volume** | Elastic Block Store — persistent disk attached to the instance |
| **Security Group** | Firewall rules controlling inbound/outbound traffic |
| **Elastic IP** | A static public IPv4 address that can be associated with an instance |
| **User Data** | Bootstrap scripts that run when an instance launches for the first time |

### 2. Launching EC2 Instances

- Launched a **Linux (Amazon Linux 2023)** instance in the public subnet of the custom VPC from Week 3
- Launched a **Windows Server 2022** instance in the private subnet
- Selected **t3.micro** instance type (Free Tier eligible)
- Configured EBS root volumes with encryption enabled
- Applied the appropriate Security Groups and IAM Instance Profiles

### 3. Connecting to Instances

**SSH to Linux Instance (from local machine):**
```bash
chmod 400 my-key-pair.pem
ssh -i "my-key-pair.pem" ec2-user@<public-ip-or-dns>
```

**RDP to Windows Instance:**
- Downloaded the Windows instance's RDP file from the AWS Console
- Used the key pair to decrypt the default Administrator password
- Connected via Remote Desktop Protocol (RDP) from local Windows machine
- Configured Windows Firewall to allow necessary inbound connections

### 4. Application Installation

After connecting to the Linux instance, installed a simple web server:

```bash
# Update system packages
sudo dnf update -y

# Install Apache HTTP Server
sudo dnf install -y httpd

# Start and enable the service
sudo systemctl start httpd
sudo systemctl enable httpd

# Verify it's running
sudo systemctl status httpd

# Create a test page
echo "<h1>Hello from EC2!</h1>" | sudo tee /var/www/html/index.html
```

- Configured the Security Group to allow port 80 (HTTP) from the internet
- Verified the web page was accessible via the instance's public IP

### 5. EC2 Instance Types and Storage

Explored the EC2 instance type families:
- **General Purpose (t, m)** — Balanced CPU/memory; best for development and small workloads
- **Compute Optimized (c)** — High CPU for batch processing, gaming servers
- **Memory Optimized (r, x)** — Large memory for in-memory databases (Redis, SAP)
- **Storage Optimized (i, d)** — High IOPS for NoSQL databases, data warehouses

Practiced with **EBS volume management**:
- Added a secondary EBS volume to the instance
- Formatted and mounted the volume on Linux (`mkfs.ext4`, `mount`)
- Created an EBS snapshot for backup

### 6. Instance Lifecycle Management

- Stopped, started, and rebooted instances (noting that public IPs change on stop/start without Elastic IP)
- Terminated test instances and confirmed EBS volumes with `Delete on Termination` = false persisted
- Created a custom **AMI** from the configured instance for reuse

---

## Key Takeaways

EC2 is the most commonly used AWS service and understanding its full lifecycle — launch, configure, connect, operate, snapshot, terminate — is foundational. The key insight this week was that EC2 instances should be treated as **disposable** rather than permanent: automate configuration with User Data or configuration management tools, and rely on AMIs or EBS snapshots for state persistence.

---

## Resources Referenced

- [Amazon EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
- [Connecting to Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-linux-inst-ssh.html)
