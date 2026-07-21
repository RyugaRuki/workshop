---
title: "Tuần 4 – Amazon EC2"
weight: 4
date: 2026-05-11
---

**Thời gian:** 11/05/2026 – 17/05/2026

---

## Tổng Quan

Tuần 4 tập trung vào **Amazon Elastic Compute Cloud (EC2)** — dịch vụ tính toán cốt lõi của AWS. Tuần này bao gồm khởi chạy và cấu hình EC2 instances, kết nối qua SSH (Linux) và RDP (Windows), cài đặt ứng dụng và hiểu về loại instance, tùy chọn lưu trữ và quản lý vòng đời instance.

---

## Hoạt Động & Học Hỏi

### 1. Cơ Bản EC2

| Khái niệm | Mô tả |
|---|---|
| **Instance Type** | Thông số CPU, bộ nhớ và mạng (ví dụ: t3.micro, m5.large) |
| **AMI** | Amazon Machine Image — hệ điều hành và phần mềm được cài đặt sẵn |
| **Key Pair** | Cặp khóa RSA/ED25519 để xác thực SSH trên Linux |
| **EBS Volume** | Elastic Block Store — đĩa cứng bền vững gắn vào instance |
| **Security Group** | Quy tắc tường lửa kiểm soát lưu lượng vào/ra |
| **Elastic IP** | Địa chỉ IPv4 public tĩnh có thể gắn vào instance |
| **User Data** | Script bootstrap chạy khi instance khởi chạy lần đầu |

### 2. Khởi Chạy EC2 Instances

- Khởi chạy một instance **Linux (Amazon Linux 2023)** trong public subnet của VPC tùy chỉnh từ Tuần 3
- Khởi chạy một instance **Windows Server 2022** trong private subnet
- Chọn instance type **t3.micro** (đủ điều kiện Free Tier)
- Cấu hình EBS root volumes với mã hóa được bật
- Áp dụng Security Groups và IAM Instance Profiles phù hợp

### 3. Kết Nối Đến Instances

**SSH đến Linux Instance:**
```bash
chmod 400 my-key-pair.pem
ssh -i "my-key-pair.pem" ec2-user@<public-ip-or-dns>
```

**RDP đến Windows Instance:**
- Tải xuống file RDP của Windows instance từ AWS Console
- Dùng key pair để giải mã mật khẩu Administrator mặc định
- Kết nối qua Remote Desktop Protocol (RDP)
- Cấu hình Windows Firewall cho phép các kết nối vào cần thiết

### 4. Cài Đặt Ứng Dụng

Sau khi kết nối đến Linux instance, cài đặt web server đơn giản:

```bash
# Cập nhật package
sudo dnf update -y

# Cài đặt Apache HTTP Server
sudo dnf install -y httpd

# Khởi động và bật dịch vụ
sudo systemctl start httpd
sudo systemctl enable httpd

# Tạo trang test
echo "<h1>Xin chào từ EC2!</h1>" | sudo tee /var/www/html/index.html
```

- Cấu hình Security Group cho phép port 80 (HTTP) từ internet
- Xác minh trang web có thể truy cập qua IP public của instance

### 5. Quản Lý Vòng Đời Instance

- Dừng, khởi động và khởi động lại instances (lưu ý IP public thay đổi khi stop/start nếu không có Elastic IP)
- Tạo **AMI** tùy chỉnh từ instance đã cấu hình để tái sử dụng
- Tạo **EBS snapshot** để sao lưu
- Thêm secondary EBS volume, format và mount trên Linux

---

## Điểm Rút Ra

EC2 là dịch vụ AWS được sử dụng phổ biến nhất. Điểm quan trọng tuần này: EC2 instances nên được xem là **có thể thay thế** chứ không phải vĩnh viễn — tự động hóa cấu hình với User Data và dựa vào AMI hoặc EBS snapshot để duy trì trạng thái.

---

## Tài Nguyên Tham Khảo

- [Tài liệu Amazon EC2](https://docs.aws.amazon.com/ec2/)
- [Các loại EC2 Instance](https://aws.amazon.com/ec2/instance-types/)
- [Kết nối đến Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-linux-inst-ssh.html)
