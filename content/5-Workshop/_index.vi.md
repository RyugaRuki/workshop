---
title: "Workshop"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 4. </b> "
---

# CloudBrief Workshop
## Hệ thống thu thập và tóm tắt tin tức công nghệ theo hướng EC2-first

Workshop này ghi lại phần triển khai cá nhân của tôi trong dự án nhóm CloudBrief. Mục tiêu là trình bày một use-case AWS thực tế từ kiến trúc, triển khai, kiểm thử đến dọn dẹp tài nguyên bằng nội dung tự viết.

CloudBrief thu thập tin tức công nghệ từ RSS feeds và Hacker News, loại bỏ trùng lặp, trích xuất nội dung sạch, tóm tắt bằng Amazon Bedrock, xử lý cover image sang WebP và cung cấp public magazine cùng reader/admin workflow có xác thực.

#### Nội dung workshop

1. [Tổng quan dự án và kiến trúc](5.1-Workshop-overview/)
2. [Chuẩn bị tài khoản AWS và môi trường local](5.2-Prerequiste/)
3. [Build và deploy CloudBrief](5.3-S3-vpc/)
4. [Verify hệ thống production](5.4-S3-onprem/)
5. [Quyết định security, cost và architecture](5.5-Policy/)
6. [Kế hoạch cleanup và recovery](5.6-Cleanup/)

#### Đối chiếu yêu cầu

Workshop đáp ứng rubric project:

* Use-case AWS thực tế: dashboard tóm tắt tin tức công nghệ.
* Sử dụng hơn 3 dịch vụ AWS: WAF, CloudFront, S3, ALB, EC2 Auto Scaling, VPC, SQS, DynamoDB, Bedrock, EventBridge, CloudWatch, SNS, SSM, Backup, Budgets và IAM.
* Có sơ đồ kiến trúc và lý do chọn dịch vụ.
* Có hướng dẫn triển khai và kiểm thử từng bước.
* Có logs, metrics, alarms, cost controls, security controls, backup và cleanup.
