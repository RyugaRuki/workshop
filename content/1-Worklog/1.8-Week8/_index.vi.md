---
title: "Tuần 8 – Tối Ưu Chi Phí & Auto Scaling"
weight: 8
date: 2026-06-08
---

**Thời gian:** 08/06/2026 – 14/06/2026

---

## Tổng Quan

Tuần 8 tập trung vào hai chủ đề bổ sung nhau: **Amazon Lightsail** như một giải pháp thay thế đơn giản, dự đoán được chi phí cho EC2 với workload nhỏ hơn, và **EC2 Auto Scaling** để tự động điều chỉnh dung lượng tính toán theo nhu cầu.

---

## Hoạt Động & Học Hỏi

### 1. Amazon Lightsail — Điện Toán Đám Mây Đơn Giản

**Amazon Lightsail** là nền tảng đám mây đơn giản của AWS dành cho developer, doanh nghiệp nhỏ và sinh viên cần máy chủ ảo, database, lưu trữ và mạng với giá cố định hàng tháng có thể dự đoán được.

**So sánh Lightsail vs EC2:**

| Tính năng | Amazon Lightsail | Amazon EC2 |
|---|---|---|
| Giá | Gói cố định hàng tháng | Trả theo giây, phức tạp |
| Cấu hình | Wizard đơn giản | Kiểm soát đầy đủ |
| Mạng | Đơn giản (tường lửa tích hợp) | Kiểm soát VPC đầy đủ |
| Use case | App đơn giản, blog, dev/test | Workload production, doanh nghiệp |

**Lab: Triển khai WordPress trên Lightsail:**
1. Tạo Lightsail instance với WordPress blueprint (gói $3.50/tháng)
2. Kết nối qua SSH terminal dựa trên trình duyệt
3. Cấu hình WordPress với domain tùy chỉnh bằng static IP của Lightsail
4. Tạo Lightsail snapshot để sao lưu

### 2. EC2 Auto Scaling — Quản Lý Dung Lượng Động

**EC2 Auto Scaling** tự động thêm hoặc xóa EC2 instances dựa trên nhu cầu, đảm bảo khả dụng trong lúc lưu lượng tăng đột biến và giảm chi phí trong thời gian yên tĩnh.

| Thành phần | Mục đích |
|---|---|
| **Launch Template** | Định nghĩa cấu hình instance (AMI, loại, SG, IAM role) |
| **Auto Scaling Group (ASG)** | Fleet của instances được quản lý như một nhóm |
| **Scaling Policy** | Quy tắc khi nào và cách scale in/out |
| **Desired Capacity** | Số instances mong muốn |
| **Minimum/Maximum** | Giới hạn cho kích thước ASG |

### 3. Tạo Thiết Lập Auto Scaling

**Lab từng bước:**

1. Tạo **Launch Template** với User Data script:
```bash
#!/bin/bash
sudo dnf install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
INSTANCE_ID=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id)
echo "<h1>Phục vụ từ Instance: $INSTANCE_ID</h1>" | sudo tee /var/www/html/index.html
```

2. Tạo **Auto Scaling Group** với:
   - Min: 1, Desired: 2, Max: 4 instances
   - Trải rộng hai availability zones (Multi-AZ)
   - Gắn vào **Application Load Balancer (ALB)**

3. Cấu hình **Scaling Policies:**
   - **Target Tracking Policy:** Duy trì CPU trung bình ở 50%
   - **Scheduled Scaling:** Scale out vào buổi sáng ngày làm việc, scale in vào buổi tối

4. Kiểm tra scaling bằng cách kích hoạt stress test CPU và quan sát ASG khởi chạy instances mới tự động

### 4. Elastic Load Balancing (ALB)

Triển khai **Application Load Balancer** trước Auto Scaling Group:
- Tạo Target Group với health check trên port 80
- Đăng ký Auto Scaling Group instances với Target Group
- Cấu hình Listener trên port 80 chuyển đến Target Group
- Xác minh lưu lượng được phân phối đồng đều

### 5. Các Chiến Lược Tối Ưu Chi Phí

| Chiến lược | Tiết kiệm tiềm năng |
|---|---|
| Auto Scaling (scale in giờ thấp điểm) | 30–70% chi phí tính toán |
| Reserved Instances (1 năm, không trả trước) | ~40% so với On-Demand |
| Spot Instances (workload chịu lỗi) | Lên đến 90% so với On-Demand |
| Rightsizing (chọn đúng loại instance) | 10–40% |
| Graviton instances (ARM-based) | 20% giá/hiệu suất tốt hơn |

---

## Điểm Rút Ra

Auto Scaling là thiết yếu cho bất kỳ workload production nào có lưu lượng không phẳng hoàn toàn. Sự kết hợp ALB + Auto Scaling Group là mô hình chuẩn cho ứng dụng web có thể mở rộng ngang trên AWS. Lightsail là tùy chọn tiết kiệm chi phí bị sử dụng ít cho workload đơn giản hơn.

---

## Tài Nguyên Tham Khảo

- [Tài liệu Amazon Lightsail](https://docs.aws.amazon.com/lightsail/)
- [Tài liệu EC2 Auto Scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/)
- [AWS Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/)
