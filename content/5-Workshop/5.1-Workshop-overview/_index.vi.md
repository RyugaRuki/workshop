---
title : "Tổng quan dự án và kiến trúc"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Tổng quan dự án

CloudBrief là ứng dụng AWS theo hướng EC2-first để thu thập và tóm tắt tin tức công nghệ. Dự án phục vụ sinh viên, người nghiên cứu, người chấm demo và người dùng dashboard cần theo dõi tin tức cloud/phần mềm/bảo mật nhanh hơn.

Ứng dụng thu thập bài viết từ RSS feeds và Hacker News, loại bỏ trùng lặp, trích xuất nội dung sạch, tóm tắt bằng Amazon Bedrock Nova Micro hoặc deterministic fallback, xử lý cover image sang WebP và cung cấp public magazine, article reader, reader community cùng protected operations page.

#### Sơ đồ kiến trúc

![Kiến trúc CloudBrief]({{< asseturl "images\5-Workshop\cloudbrief-evidence/production-architecture.png" >}})

[Tải file draw.io]({{< asseturl "attachments/cloudbrief-ec2-clean-aws.drawio" >}})

#### Luồng dữ liệu chính

1. Reader mở AWS WAF-protected CloudFront distribution.
2. CloudFront phục vụ private S3 frontend hoặc forward `/api/*` đến ALB.
3. ALB phân phối API traffic đến hai EC2 trong private subnet.
4. EventBridge hoặc protected admin request queue collection từ RSS và Hacker News.
5. Worker extract cleaned text, process cover image sang WebP và queue summary qua SQS.
6. Amazon Bedrock Nova Micro hoặc deterministic fallback tạo article summary.
7. DynamoDB lưu article/social state; private S3 bucket lưu cleaned text và processed image.
8. CloudWatch, SNS, Systems Manager, AWS Backup, Budgets và IAM cung cấp operations/recovery controls.

#### Phạm vi cá nhân

Dự án nhóm là CloudBrief. Phạm vi báo cáo của tôi là phần workshop có thể triển khai: giải thích kiến trúc, CDK deployment flow, kiểm thử API/dashboard, kiểm soát chi phí/bảo mật, monitoring evidence, backup evidence và cleanup verification.
