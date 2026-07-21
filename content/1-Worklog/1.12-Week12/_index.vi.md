---
title: "Tuần 12 – Hoàn Thành & Bàn Giao"
weight: 12
date: 2026-07-06
---

**Thời gian:** 06/07/2025 – 30/07/2026

---

## Tổng Quan

Tuần 12 là tuần tổng kết của kỳ thực tập FCAJ — sprint cuối đầy thử thách kết hợp tất cả kỹ năng và kiến thức tích lũy trong 11 tuần trước. Tất cả công việc lab còn lại được hoàn thành, dự án workshop nhóm đạt trạng thái sẵn sàng production, và báo cáo thực tập được hoàn thiện và nộp.

---

## Hoạt Động & Học Hỏi

### 1. Hoàn Thành Tất Cả Lab

- ✅ Xác minh hoàn thành tất cả 11 lab của các tuần trước
- ✅ Review và chạy lại các lab chưa rõ kết quả để xác nhận hiểu biết
- ✅ Ghi lại tất cả tài nguyên lab sạch sẽ để đưa vào báo cáo cuối kỳ
- ✅ Kiểm tra tài khoản AWS để dọn dẹp tài nguyên hoặc tạo snapshots cuối

### 2. Ứng Dụng Web Khả Dụng Cao — Workshop Cuối Kỳ

Dự án workshop nhóm đạt trạng thái **sẵn sàng production** cuối cùng:

**Kiến Trúc Cuối:**
```
                    Internet
                       │
                ┌──────┴──────┐
                │   Route 53   │
                └──────┬──────┘
                       │
                ┌──────┴──────┐
                │     ALB      │  ← Public Subnets (AZ-a, AZ-b)
                └──────┬──────┘
                       │
       ┌───────────────┼───────────────┐
       │                               │
 ┌─────┴───┐                    ┌─────┴───┐
 │  EC2 #1 │     Auto Scaling   │  EC2 #2 │
 │  AZ-a   │ ←─────────────── │  AZ-b   │
 └─────┬───┘                    └─────┬───┘
       │     Private Subnets           │
       └───────────────┬───────────────┘
                       │
                ┌──────┴──────┐
                │  RDS MySQL   │  ← Multi-AZ
                └─────────────┘
```

**Checklist hoàn thành:**
- ✅ VPC với 4 subnets trên 2 AZ (2 public, 2 private)
- ✅ Internet Gateway và NAT Gateways
- ✅ Application Load Balancer với health checks trên `/health`
- ✅ Auto Scaling Group (min 2, desired 2, max 6) với target-tracking CPU
- ✅ RDS MySQL với Multi-AZ standby
- ✅ Thông tin xác thực database trong AWS Secrets Manager
- ✅ CloudWatch alarms cho CPU, ALB 5XX errors và RDS free storage
- ✅ CloudWatch Dashboard cho khả năng quan sát vận hành
- ✅ S3 bucket cho ALB access logs
- ✅ IAM roles với policies quyền tối thiểu
- ✅ Security Groups theo nguyên tắc tiếp xúc tối thiểu

### 3. Điều Chỉnh Dự Án Nhóm Lần Cuối

- **Kiểm thử hiệu năng:** Dùng Apache Bench mô phỏng tải và xác minh Auto Scaling hoạt động
- **Kiểm thử chuyển đổi dự phòng:** Khởi động lại RDS primary và xác nhận failover hoàn thành trong 90 giây
- **Review bảo mật:** Chạy AWS Trusted Advisor và giải quyết tất cả khuyến nghị mức độ High
- **Phân tích chi phí:** Dùng AWS Cost Explorer tính chi phí hàng tháng ước tính (~$45 USD/tháng)

```bash
# Kiểm thử tải: 1000 requests, 50 concurrent
ab -n 1000 -c 50 http://my-alb-dns.ap-southeast-1.elb.amazonaws.com/

# Xác minh thời gian failover RDS
aws rds reboot-db-instance \
  --db-instance-identifier myapp-db \
  --force-failover
```

### 4. Hoàn Thành Báo Cáo Thực Tập

Báo cáo thực tập cuối kỳ được hoàn thiện và nộp:
- **Nhật ký công việc:** 12 mục hàng tuần với hoạt động và kiến thức chi tiết
- **Đề xuất thực tập:** Mục tiêu chương trình và review kết quả đạt được
- **Sự kiện:** Tài liệu 3 sự kiện AWS đã tham dự
- **Workshop:** Tài liệu đầy đủ dự án nhóm
- **Tự đánh giá:** Đánh giá trung thực về sự phát triển
- **Chia sẻ & Phản hồi:** Khuyến nghị cho thực tập sinh tương lai

### 5. Dọn Dẹp Tài Nguyên

Theo thực hành AWS tốt nhất, tất cả tài nguyên lab được xóa để ngăn phí phát sinh:

```bash
# Dọn dẹp theo thứ tự (thứ tự phụ thuộc ngược)
# 1. Xóa Auto Scaling Group
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --min-size 0 --desired-capacity 0

# 2. Xóa RDS instance (tạo snapshot cuối trước)
aws rds delete-db-instance \
  --db-instance-identifier myapp-db \
  --final-db-snapshot-identifier myapp-db-final-snapshot

# 3. Xóa NAT Gateways (billing dừng ngay lập tức)
aws ec2 delete-nat-gateway --nat-gateway-id nat-xxxx

# 4. Giải phóng Elastic IPs
aws ec2 release-address --allocation-id eipalloc-xxxx
```

---

## Tóm Tắt Hoàn Thành Chương Trình

| Chỉ số | Kết quả |
|---|---|
| **Tổng số Lab hoàn thành** | 12 lab hàng tuần + workshop nhóm |
| **Dịch vụ AWS đã thành thạo** | 12+ (IAM, VPC, EC2, S3, RDS, Route 53, CloudWatch, Auto Scaling, ALB, Lightsail, Cloud9, Secrets Manager) |
| **Sự kiện đã tham dự** | 3 sự kiện AWS |
| **Dự án nhóm** | Ứng dụng Web Khả Dụng Cao — triển khai và kiểm thử hoàn chỉnh |
| **Thời gian** | 12 tuần (20/04 – 12/07/2025) |

---

## Điểm Rút Ra

Tuần cuối này củng cố rằng xây dựng trên AWS là một quy trình lặp — triển khai, kiểm thử, tối ưu và tài liệu hóa. Dự án nhóm là sự tổng hợp mọi khái niệm từ chương trình. Hoàn thành kỳ thực tập này cung cấp nền tảng vững chắc cho sự nghiệp kỹ thuật đám mây.

---

> *Kết thúc Nhật ký — Cảm ơn bạn đã theo dõi hành trình 12 tuần này!*
