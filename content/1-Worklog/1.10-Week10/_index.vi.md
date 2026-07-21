---
title: "Tuần 10 – Hybrid DNS với Amazon Route 53"
weight: 10
date: 2026-06-22
---

**Thời gian:** 22/06/2026 – 28/06/2026

---

## Tổng Quan

Tuần 10 nghiên cứu **Amazon Route 53** với trọng tâm là **hybrid DNS** — cấu hình cho phép phân giải tên liền mạch giữa hạ tầng on-premise và môi trường AWS VPC. Đây là mô hình quan trọng cho các tình huống áp dụng đám mây doanh nghiệp khi không phải mọi thứ đều chuyển sang AWS cùng một lúc.

---

## Hoạt Động & Học Hỏi

### 1. Tổng Quan Amazon Route 53

| Tính năng | Mô tả |
|---|---|
| **Đăng ký Domain** | Đăng ký domain mới trực tiếp qua Route 53 |
| **Public Hosted Zones** | DNS cho domain internet-facing |
| **Private Hosted Zones** | DNS cho tài nguyên VPC nội bộ (không truy cập công khai) |
| **Health Checks** | Giám sát sức khỏe endpoint và định tuyến lưu lượng |
| **Routing Policies** | Simple, Weighted, Latency, Failover, Geolocation |
| **Resolver** | Xử lý DNS queries giữa VPCs và on-premise |

### 2. Cài Đặt Public Hosted Zone

- Tạo **Public Hosted Zone** cho test domain
- Cấu hình **A records** và **CNAME records** trỏ đến DNS của ALB
- Hiểu sự khác biệt giữa **Alias records** (miễn phí, theo dõi thay đổi DNS ALB/CloudFront) và **CNAME records** tiêu chuẩn

### 3. Private Hosted Zone cho VPC

- Tạo **Private Hosted Zone** liên kết với VPC tùy chỉnh từ Tuần 3
- Thêm DNS entries cho dịch vụ nội bộ:
  - `app.internal` → Private IP của EC2
  - `db.internal` → Endpoint của RDS
  - `api.internal` → DNS của Internal ALB
- Xác minh EC2 instances trong VPC có thể phân giải `db.internal` trong khi hostname đó không thể phân giải từ internet

### 4. Kiến Trúc Hybrid DNS

Kịch bản hybrid DNS mô phỏng môi trường doanh nghiệp với cả server on-premise và hạ tầng AWS VPC cần phân giải hostname của nhau.

**Kiến trúc:**
```
Mạng On-Premise (192.168.0.0/16)
    ├── DNS Server công ty (192.168.1.10)
    └── Domain .corp

AWS VPC (10.0.0.0/16)
    ├── Route 53 Private Hosted Zone (.internal)
    ├── Route 53 Resolver Inbound Endpoint
    └── Route 53 Resolver Outbound Endpoint
```

**Inbound Endpoint** (on-premise → AWS):
1. Tạo **Inbound Resolver Endpoint** trong VPC với hai IP (dự phòng trên AZs)
2. Cấu hình DNS server on-premise chuyển tiếp queries cho domain `.internal` đến Inbound Endpoint IPs
3. Kết quả: workstation on-premise có thể phân giải `db.internal` qua Private Hosted Zone

**Outbound Endpoint** (AWS → on-premise):
1. Tạo **Outbound Resolver Endpoint** trong VPC
2. Tạo **Forwarding Rule**: queries cho domain `.corp` → chuyển đến DNS server on-premise
3. Liên kết forwarding rule với VPC
4. Kết quả: EC2 instances trong AWS VPC có thể phân giải `server01.corp` từ DNS on-premise

### 5. Routing Policies Route 53

| Policy | Use Case |
|---|---|
| **Simple** | Một tài nguyên cho một domain |
| **Weighted** | A/B testing, triển khai dần dần (ví dụ: 90% cũ, 10% mới) |
| **Latency** | Định tuyến đến AWS region có độ trễ thấp nhất cho user |
| **Failover** | Active/Passive HA — Định tuyến đến backup nếu health check primary thất bại |
| **Geolocation** | Định tuyến dựa trên vị trí địa lý của user |
| **Multi-Value Answer** | Trả về tối đa 8 IP healthy |

**Lab: Failover routing**
- Tạo hai records cho cùng domain: một **Primary** (EC2) và một **Secondary** (S3 static error page)
- Gắn Route 53 health checks vào primary endpoint
- Mô phỏng failure bằng cách dừng EC2 và xác minh traffic chuyển sang S3 page trong ~30 giây

---

## Điểm Rút Ra

Hybrid DNS là một trong những khái niệm mạng quan trọng nhất cho áp dụng AWS doanh nghiệp. Các công ty hiếm khi chuyển 100% hạ tầng lên cloud cùng một lúc, vì vậy việc cho phép phân giải tên liền mạch giữa on-premise và AWS là rất quan trọng. Route 53 Resolver Endpoints làm điều này đơn giản mà không cần giải pháp DNS tùy chỉnh.

---

## Tài Nguyên Tham Khảo

- [Tài liệu Amazon Route 53](https://docs.aws.amazon.com/route53/)
- [Route 53 Resolver cho Hybrid DNS](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html)
