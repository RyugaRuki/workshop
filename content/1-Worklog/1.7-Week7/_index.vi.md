---
title: "Tuần 7 – Amazon RDS"
weight: 7
date: 2026-06-01
---

**Thời gian:** 01/06/2026 – 07/06/2026

---

## Tổng Quan

Tuần 7 giới thiệu **Amazon Relational Database Service (RDS)** — dịch vụ cơ sở dữ liệu được quản lý của AWS tự động hóa sao lưu, vá lỗi, chuyển đổi dự phòng và mở rộng. Tuần này cũng có buổi Q&A với kỹ sư AWS cung cấp góc nhìn thực tế vô giá về các mô hình database production.

---

## Hoạt Động & Học Hỏi

### 1. Tổng Quan Amazon RDS

| Tính năng | DB tự quản trên EC2 | Amazon RDS |
|---|---|---|
| Vá lỗi OS | Thủ công | Tự động |
| Cập nhật phần mềm DB | Thủ công | Tự động |
| Sao lưu tự động | Phải tự cài | Tích hợp sẵn |
| Chuyển đổi dự phòng Multi-AZ | Phải tự cấu hình | Tự động trong vài phút |
| Read Replicas | Cài đặt phức tạp | Tạo bằng một cú nhấp |
| Giám sát | Phải tự cài | Tích hợp CloudWatch |

### 2. Các Database Engine Được Hỗ Trợ

- **Amazon Aurora** (MySQL & PostgreSQL tương thích) — Engine cloud-native độc quyền của AWS
- **MySQL** — Database quan hệ mã nguồn mở phổ biến nhất
- **PostgreSQL** — Database mã nguồn mở với tuân thủ ACID mạnh
- **MariaDB** — Nhánh MySQL với thêm storage engines
- **Oracle** — Database doanh nghiệp
- **Microsoft SQL Server** — Database doanh nghiệp Windows

### 3. Khởi Chạy RDS Instance

**Lab: Triển khai MySQL RDS trong private subnet**

1. Tạo **DB Subnet Group** bao gồm hai private subnets ở các AZ khác nhau
2. Khởi chạy instance **MySQL 8.0** với:
   - Instance class: `db.t3.micro` (đủ điều kiện Free Tier)
   - Storage: 20 GB gp3 SSD với mã hóa được bật
   - Multi-AZ: Bật (bản sao standby đồng bộ ở AZ thứ hai)
   - Publicly accessible: Tắt (chỉ private subnet)
   - Security Group: Cho phép port 3306 từ Security Group app tier

3. Lưu **Master Username** và **Master Password** vào AWS Secrets Manager

### 4. Kết Nối Ứng Dụng Với RDS

Kết nối ứng dụng Python đơn giản trên EC2 instance đến RDS MySQL:

```python
import pymysql
import boto3
import json

# Lấy thông tin xác thực từ Secrets Manager
secrets_client = boto3.client('secretsmanager', region_name='ap-southeast-1')
secret = secrets_client.get_secret_value(SecretId='rds/mysql/myapp')
credentials = json.loads(secret['SecretString'])

connection = pymysql.connect(
    host=credentials['host'],
    user=credentials['username'],
    password=credentials['password'],
    database='myappdb',
    port=3306
)

cursor = connection.cursor()
cursor.execute("SELECT VERSION()")
version = cursor.fetchone()
print(f"MySQL Version: {version[0]}")
```

### 5. Sao Lưu và Khôi Phục RDS

- Xem lại cấu hình cửa sổ **sao lưu tự động hàng ngày**
- Tạo **DB snapshot thủ công** trước khi thay đổi schema
- Thực hành **khôi phục theo thời điểm** để khôi phục DB về timestamp cụ thể

### 6. Multi-AZ và Read Replicas

- **Multi-AZ:** Nhân bản đồng bộ đến instance standby ở AZ khác; tự động chuyển đổi khi primary bị lỗi (thường < 2 phút)
- **Read Replica:** Nhân bản bất đồng bộ để mở rộng đọc; hữu ích cho workload analytics

### 7. Buổi Chia Sẻ Kinh Nghiệm Thực Tế AWS

Tham dự buổi Q&A với kỹ sư AWS:
- Thảo luận RDS Multi-AZ vs. Aurora Global Database cho các yêu cầu khả dụng khác nhau
- Kinh nghiệm thực tế về di chuyển database từ on-premise lên RDS bằng **AWS DMS**
- Tối ưu chi phí: khi nào dùng RDS Reserved Instances để tiết kiệm đáng kể
- Các metric RDS CloudWatch quan trọng cần theo dõi trong production

---

## Điểm Rút Ra

RDS giảm đáng kể gánh nặng vận hành chạy database quan hệ so với tự quản MySQL trên EC2. Quyết định kiến trúc quan trọng: Multi-AZ cho tính khả dụng cao vs. Read Replicas cho khả năng mở rộng đọc. Luôn lưu thông tin xác thực database trong **AWS Secrets Manager** thay vì hardcode.

---

## Tài Nguyên Tham Khảo

- [Tài liệu Amazon RDS](https://docs.aws.amazon.com/rds/)
- [RDS Best Practices](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_BestPractices.html)
- [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/)
