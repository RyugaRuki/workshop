---
title: "Tuần 5 – IAM Role & AWS Cloud9"
weight: 5
date: 2026-05-18
---

**Thời gian:** 18/05/2026 – 24/05/2026

---

## Tổng Quan

Tuần 5 bao gồm hai chủ đề quan trọng: **IAM Roles** để cấp quyền cho các dịch vụ và ứng dụng AWS (thay vì cho user), và **AWS Cloud9** như một môi trường phát triển tích hợp (IDE) dựa trên trình duyệt giúp loại bỏ yêu cầu cài đặt công cụ cục bộ.

---

## Hoạt Động & Học Hỏi

### 1. IAM Roles — Cấp Quyền Cho Ứng Dụng

Trong khi Tuần 2 tập trung vào IAM Users và Groups cho danh tính con người, Tuần 5 khám phá **IAM Roles** — cơ chế cấp quyền AWS cho ứng dụng, dịch vụ và workload.

| Khái niệm | Mô tả |
|---|---|
| **Trust Policy** | Định nghĩa *ai* (principal nào) có thể đảm nhận role |
| **Permission Policy** | Định nghĩa *những gì* role được phép làm |
| **Instance Profile** | Container chứa IAM Role gắn vào EC2 instance |
| **STS (Security Token Service)** | Phát hành thông tin xác thực tạm thời khi role được đảm nhận |

### 2. Tạo và Gắn EC2 Instance Roles

- Tạo IAM Role với trust policy cho principal `ec2.amazonaws.com`
- Gắn managed policy `AmazonS3ReadOnlyAccess` vào role
- Tạo **Instance Profile** và gắn vào một EC2 instance đang chạy
- Xác minh instance có thể truy cập S3 mà không cần cấu hình access keys:

```bash
# Trên EC2 instance — không cần cấu hình credentials
aws s3 ls s3://my-internship-bucket
# Hoạt động nhờ Instance Role cung cấp thông tin xác thực tạm thời
```

**Tại sao dùng roles thay access keys:**
- Không có thông tin xác thực lâu dài lưu trên instance
- Thông tin xác thực được tự động xoay vòng bởi STS
- Dễ dàng cấp/thu hồi quyền tập trung qua IAM Role

### 3. Các Mô Hình Role Phổ Biến

- **Lambda execution role** — Lambda đảm nhận role để truy cập DynamoDB, S3, v.v.
- **ECS task role** — Container đảm nhận role để truy cập API chi tiết
- **Cross-account access** — Role ở Account A có thể được users/dịch vụ ở Account B đảm nhận
- **AWS service roles** — Các dịch vụ như Auto Scaling, CodeDeploy đảm nhận role để thực hiện hành động thay bạn

### 4. AWS Cloud9 — IDE Dựa Trên Trình Duyệt

**AWS Cloud9** là IDE đám mây chạy trên EC2 instance và có thể truy cập qua trình duyệt web, loại bỏ cần cài đặt công cụ phát triển cục bộ.

**Quá trình cài đặt:**
1. Tạo môi trường Cloud9 trong AWS Console
2. Chọn loại EC2 instance (khuyến nghị t3.small)
3. Đặt loại môi trường là **EC2** (Cloud9 quản lý instance)
4. Mở Cloud9 IDE trong trình duyệt

**Tính năng khám phá:**
- **Trình soạn thảo file** với tô sáng cú pháp cho Python, JavaScript, YAML, v.v.
- **Terminal tích hợp** với AWS CLI, Node.js, Python và Git được cài đặt sẵn
- **AWS Resource Explorer** hiển thị EC2 instances, Lambda functions và tài nguyên khác
- Hỗ trợ **cộng tác code** cho lập trình theo cặp thời gian thực

### 5. Cấu Hình Cloud9 với IAM Role Tùy Chỉnh

- Vô hiệu hóa **AWS Managed Temporary Credentials** mặc định của Cloud9
- Gắn IAM Role tùy chỉnh vào EC2 instance bên dưới của Cloud9
- Xác minh chỉ có quyền được định nghĩa trong role tùy chỉnh mới có sẵn (quyền tối thiểu)

---

## Điểm Rút Ra

IAM Roles là cơ chế ưu tiên cho bất kỳ giao tiếp workload-to-AWS-service nào. Sử dụng Instance Profiles loại bỏ rủi ro bảo mật của việc lưu trữ access keys lâu dài trên EC2 instances. AWS Cloud9 là công cụ tuyệt vời khi việc cài đặt môi trường cục bộ là rào cản — nó cung cấp môi trường phát triển nhất quán, được cấu hình sẵn có thể truy cập từ bất kỳ trình duyệt nào.

---

## Tài Nguyên Tham Khảo

- [Tài liệu IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)
- [AWS Cloud9 User Guide](https://docs.aws.amazon.com/cloud9/latest/user-guide/)
