---
title: "Week 1 – Onboarding & AWS Account Setup"
weight: 1
date: 2026-04-20
---

**Period:** April 20 – April 26, 2026

---

## Overview

The first week of the FCAJ internship marked the start of a structured 12-week journey into cloud computing at **Amazon Web Services Vietnam**. This week was primarily dedicated to program orientation, administrative setup, and establishing a secure, well-configured AWS cloud environment from scratch.

---

## Activities & Learnings

### 1. Program Orientation

- Received official internship documentation including the program schedule, assessment criteria, and communication guidelines from the FCAJ team
- Attended the initial onboarding session covering the program's learning philosophy: hands-on labs, peer learning, and real-world project delivery
- Connected with fellow FCAJ interns and established group communication channels (Zalo, Email)
- Reviewed the 12-week curriculum to understand the full scope and set personal learning goals

### 2. AWS Account Creation

- Created a new **AWS root account** using a dedicated email address following AWS best practices
- Enabled **Multi-Factor Authentication (MFA)** on the root account using an authenticator app
- Created a dedicated **IAM Admin user** for day-to-day operations, avoiding the use of root credentials
- Logged into the AWS Management Console and explored the service catalog

### 3. AWS Free Tier Exploration

- Reviewed the **AWS Free Tier** documentation to understand the scope of services available at no cost for 12 months
- Key Free Tier services noted:
  - **Amazon EC2:** 750 hours/month of t2.micro or t3.micro (Linux/Windows)
  - **Amazon S3:** 5 GB standard storage
  - **Amazon RDS:** 750 hours/month of db.t2.micro
  - **Amazon Lambda:** 1 million free requests/month
- Understood the difference between *always free*, *12-month free*, and *trial* tier types

### 4. AWS Budgets Setup

- Created a **monthly cost budget** of $10 USD in AWS Budgets to prevent unexpected charges
- Configured **alert thresholds** at 50%, 80%, and 100% of the budget
- Set up email notifications to receive alerts when thresholds are crossed
- Enabled **AWS Cost Explorer** to track spending patterns over time

### 5. AWS Support Plans Overview

- Studied the four AWS Support tiers:
  - **Basic** (Free) — Documentation, forums, AWS Health dashboard
  - **Developer** (~$29/month) — Business hours email support
  - **Business** (~$100+/month) — 24/7 phone, chat, and email support
  - **Enterprise** (Custom) — Technical Account Manager (TAM) + proactive guidance
- Understood when to use **AWS Trusted Advisor**, **Personal Health Dashboard**, and **Support Center** at each tier

---

## Key Takeaways

Setting up the foundational AWS environment with proper security (MFA, IAM admin user) and cost controls (Budgets, alerts) before doing any technical work is a critical best practice. The AWS Free Tier is generous enough to support the full 12-week internship curriculum when resources are properly cleaned up after each lab.

---

## Resources Referenced

- [AWS Free Tier Overview](https://aws.amazon.com/free/)
- [Setting up AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)
- [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
