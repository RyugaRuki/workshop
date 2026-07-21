---
title : "Kiểm tra monitoring, queue và operations"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4.4 </b> "
---

#### CloudWatch và SNS

Stack đã deploy có alarm cho Auto Scaling capacity, ALB target health và 5xx response, EC2 CPU, cả bốn DLQ và Bedrock fallback. Alarm notification publish đến KMS-encrypted SNS topic.

![CloudWatch alarms](/images/5-workshop/cloudbrief-evidence/cloudwatch-alarms.png)

![SNS notification topic](/images/5-workshop/cloudbrief-evidence/sns-topic.png)

#### Read-only audit ngày 16/07/2026

| Kiểm tra | Kết quả |
| --- | --- |
| CloudWatch alarms | 9 tổng: 5 OK, 3 insufficient data, 1 ALARM |
| SQS queues | 8 tổng |
| Queue không rỗng | Image-processing DLQ, 3 visible message |
| DynamoDB | 6 bảng CloudBrief |
| WAF | 1 CloudFront Web ACL |
| Budget record trong account | 3 |

Image-processing DLQ alarm là residual evidence trung thực, không phải claim mọi thứ đều xanh. Nó chứng minh alarm path hoạt động và xác định công việc operations tiếp theo: kiểm tra failed image message, sửa retryability nếu cần, chỉ redrive message hợp lệ và verify DLQ trở về zero. Không có queue mutation nào được chạy trong lúc viết workshop.

#### Backup và account event

AWS Backup bảo vệ các DynamoDB table durable. Notification cho account registration và thay đổi admin role/status chỉ publish user identifier và state; không chứa email hoặc display-name PII.

#### Bằng chứng budget

![AWS Budgets evidence](/images/5-workshop/cloudbrief-evidence/budget.png)

Budget notification là deployment gate vì ALB, NAT Gateway, interface endpoint, WAF và hai EC2 đều tạo chi phí định kỳ.
