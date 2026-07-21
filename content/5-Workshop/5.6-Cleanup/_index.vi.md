---
title : "Kế hoạch cleanup và recovery"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

{{% notice warning %}}
Production demo vẫn đang live. Các lệnh dưới đây là cleanup procedure đã review và chưa được chạy trong lúc viết workshop.
{{% /notice %}}

#### 1. Lưu bằng chứng

Trước cleanup, export bằng chứng đã sanitize cho CloudFormation, CloudFront/WAF, ALB target health, ASG instance, VPC endpoint, queue/DLQ, DynamoDB, S3, CloudWatch, SNS, Backup, Budgets, API response và browser test.

#### 2. Xử lý operations còn tồn tại

Audit ngày 16/07 phát hiện ba visible message trong image-processing DLQ. Kiểm tra và phân loại các message này trước khi destroy để báo cáo ghi rõ failure cause và remediation decision. Không public article content từ failed message.

#### 3. Destroy CDK stack

```bash
cd ~/Code/Technology-News-Collection-and-Summarization-System
AWS_PROFILE=cloudbrief-workshop AWS_REGION=us-east-1 \
  bun run cdk destroy
```

Đây là thao tác destructive và cần được phê duyệt tại thời điểm chạy.

#### 4. Xử lý retained data

Nếu CDK cố ý retain versioned S3 object, backup recovery point hoặc durable data khác, cần quyết định từng item sẽ được giữ làm bằng chứng hay xóa. Empty versioned bucket cần xóa cả object version và delete marker.

#### 5. Verify cleanup read-only

Xác nhận các tài nguyên dự kiến không còn tồn tại:

- CloudFormation stack, CloudFront distribution và WAF Web ACL.
- ALB, target group, ASG, EC2 instance, EBS volume và NAT Gateway.
- VPC endpoint, SQS queue/DLQ, DynamoDB table và S3 bucket.
- CloudWatch alarm/log group, SNS topic, Backup resource và project budget.

#### Trạng thái hiện tại

Cleanup đang **pending** vì CloudBrief vẫn là production demo active. Báo cáo không được claim cleanup thành công cho đến khi destroy được phê duyệt, chạy và verify.
