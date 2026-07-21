---
title : "Quyết định security, cost và architecture"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

#### Security controls

| Ranh giới | Control |
| --- | --- |
| Public edge | AWS WAF đứng trước CloudFront distribution |
| Frontend | Private, versioned S3 bucket qua Origin Access Control |
| API origin | CloudFront origin secret và ALB routing; EC2 không có public IP |
| Compute | Hai EC2 trong private subnet, được fixed-capacity ASG quản lý |
| Host access | Systems Manager thay cho inbound SSH |
| Managed services | SQS/Bedrock interface endpoint và S3/DynamoDB gateway endpoint |
| Publisher fetches | SSRF-safe fetcher chặn private, loopback, link-local và reserved destination |
| Article images | Validate, resize, convert sang WebP và lưu trong private S3 bucket |
| Admin API | HttpOnly admin session; static frontend không chứa `x-api-key` |
| Reader data | Đọc public; like/comment cần login; edit/delete kiểm tra owner |
| Alerts | KMS-encrypted SNS topic với quyền publish giới hạn cho EC2 |

#### Bài học IAM

Ảnh 5.2 cho thấy policy `AdministratorAccess` tạm thời dùng trong lần deploy sinh viên ban đầu. Đây là bằng chứng lịch sử hợp lệ nhưng quá rộng để dùng lâu dài trong production. Cải tiến được khuyến nghị là dedicated CDK deployment role chỉ có service action và resource `iam:PassRole` mà stack cần, bật MFA cho console và dùng short-lived credential qua IAM Identity Center khi có thể.

#### Cost model

Kiến trúc hiện tại ưu tiên high availability và private compute hơn phương án demo rẻ nhất:

- Hai EC2 `t4g.small`.
- Một Application Load Balancer.
- Một NAT Gateway.
- Hai interface endpoint có phí và hai gateway endpoint.
- CloudFront, WAF, S3, DynamoDB on-demand, SQS, CloudWatch, SNS, Backup và Bedrock usage.

Cost control gồm Nova Micro, giới hạn 256 output token, article cap, log retention ngắn, S3 lifecycle, DynamoDB on-demand, budget alert và cleanup procedure đã ghi lại.

#### Architecture tradeoff

Thiết kế một EC2 public trước đây rẻ hơn nhưng không khớp kiến trúc cuối. Thiết kế đã deploy thêm ALB, private subnet, ASG redundancy, WAF và service endpoint. Chi phí cao hơn nhưng production story rõ hơn: bảo vệ public edge, instance không có public IP, compute trên hai AZ, outbound access giới hạn, queue isolation và failure handling có observability.
