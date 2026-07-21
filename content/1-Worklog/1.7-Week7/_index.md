---
title: "Week 7 – Amazon RDS"
weight: 7
date: 2026-06-01
---

**Period:** June 1 – June 7, 2026

---

## Overview

Week 7 introduced **Amazon Relational Database Service (RDS)** — AWS's managed database service that automates backups, patching, failover, and scaling. This week also featured an AWS engineer Q&A session that provided invaluable real-world perspective on production database patterns.

---

## Activities & Learnings

### 1. Amazon RDS Overview

RDS eliminates the heavy lifting of database administration:

| Feature | Self-Managed DB on EC2 | Amazon RDS |
|---|---|---|
| OS Patching | Manual | Automatic |
| DB Software Updates | Manual | Automatic (configurable windows) |
| Automated Backups | Manual setup | Built-in (1–35 day retention) |
| Multi-AZ Failover | Manual clustering | Automated in minutes |
| Read Replicas | Complex setup | One-click creation |
| Monitoring | Custom setup | CloudWatch integration built-in |

### 2. Supported Database Engines

Explored the engines available on RDS:
- **Amazon Aurora** (MySQL & PostgreSQL compatible) — AWS's proprietary cloud-native engine
- **MySQL** — Most widely used open-source relational database
- **PostgreSQL** — Feature-rich open-source database with strong ACID compliance
- **MariaDB** — MySQL fork with additional storage engines
- **Oracle** — Enterprise database (requires license)
- **Microsoft SQL Server** — Windows-native enterprise database

### 3. Launching an RDS Instance

**Lab: Deploy a MySQL RDS instance in a private subnet**

1. Created a **DB Subnet Group** covering two private subnets in different AZs
2. Launched a **MySQL 8.0** instance with:
   - Instance class: `db.t3.micro` (Free Tier eligible)
   - Storage: 20 GB gp3 SSD with encryption enabled
   - Multi-AZ: Enabled (synchronous standby replica in second AZ)
   - Publicly accessible: Disabled (private subnet only)
   - VPC Security Group: Allow port 3306 from app tier Security Group only

3. Set the **Master Username** and **Master Password** (stored in AWS Secrets Manager)

### 4. Connecting Application to RDS

Connected a simple Python application on an EC2 instance to the RDS MySQL instance:

```python
import pymysql
import boto3
import json

# Retrieve credentials from Secrets Manager (best practice)
secrets_client = boto3.client('secretsmanager', region_name='ap-southeast-1')
secret = secrets_client.get_secret_value(SecretId='rds/mysql/myapp')
credentials = json.loads(secret['SecretString'])

connection = pymysql.connect(
    host=credentials['host'],
    user=credentials['username'],
    password=credentials['password'],
    database='myappdb',
    port=3306
)

cursor = connection.cursor()
cursor.execute("SELECT VERSION()")
version = cursor.fetchone()
print(f"MySQL Version: {version[0]}")
```

- Verified connectivity by running test queries from the EC2 application server
- Confirmed that the RDS instance was NOT reachable from the internet (private subnet + SG rules)

### 5. RDS Backup and Recovery

- Reviewed the automatic **daily backup** window configuration
- Created a **manual DB snapshot** before making schema changes
- Practiced **point-in-time recovery** to restore the database to a specific timestamp
- Understood the difference between automated backups (1–35 days) and manual snapshots (retained until deleted)

### 6. Multi-AZ and Read Replicas

- **Multi-AZ:** Synchronous replication to a standby instance in a different AZ; automatic failover in case of primary failure (typically < 2 minutes)
- **Read Replica:** Asynchronous replication for read-scaling; useful for analytics workloads that should not impact the primary DB

### 7. AWS Real-World Experience Session

Attended an AWS engineer Q&A session:
- Discussion on RDS Multi-AZ vs. Aurora Global Database for different availability requirements
- Real-world war stories about database migrations from on-premise to RDS using **AWS DMS**
- Cost optimization: when to use RDS Reserved Instances for significant savings over On-Demand
- Monitoring best practices: key RDS CloudWatch metrics (CPUUtilization, FreeStorageSpace, DatabaseConnections)

---

## Key Takeaways

RDS dramatically reduces the operational burden of running relational databases compared to self-managing MySQL on EC2. The key architectural decision is Multi-AZ for high availability vs. Read Replicas for read scalability — many production systems use both. Always store database credentials in **AWS Secrets Manager** rather than hardcoding them in application configuration.

---

## Resources Referenced

- [Amazon RDS Documentation](https://docs.aws.amazon.com/rds/)
- [RDS Best Practices](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_BestPractices.html)
- [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/)
