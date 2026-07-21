---
title : "Verify hệ thống production"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

Phần này verify hệ thống đã deploy từ hạ tầng đến một bài viết thật trên reader UI.

1. [Verify network và compute](5.4.1-prepare/)
2. [Test pipeline RSS và Hacker News thật](5.4.2-create-interface-enpoint/)
3. [Verify magazine và article reader public](5.4.3-test-endpoint/)
4. [Kiểm tra monitoring, queue và operations](5.4.4-dns-simulation/)

#### Kết quả acceptance

| Layer | Bằng chứng |
| --- | --- |
| Edge | Có CloudFront distribution và WAF |
| Compute | ALB active; hai private EC2 được ASG quản lý |
| Network | NAT available; bốn VPC endpoint available |
| Data plane | Tám SQS queue và sáu DynamoDB table |
| API | CloudFront `/api/health` và real brief endpoint phản hồi |
| Content | Bài RSS/Hacker News thật, summary, cleaned text và WebP image |
| Frontend | Magazine và article-reader load qua CloudFront |
| Operations | CloudWatch alarm, SNS, Backup và Budgets đã cấu hình |
