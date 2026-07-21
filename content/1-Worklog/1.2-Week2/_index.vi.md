---
title: "Tuần 2 – AWS Identity and Access Management (IAM)"
weight: 2
date: 2026-04-27
---

**Thời gian:** 27/04/2026 – 03/05/2026

---

## Tổng Quan

Tuần 2 tập trung vào **AWS Identity and Access Management (IAM)** — nền tảng bảo mật của AWS. Tuần này tìm hiểu cách AWS kiểm soát *ai* có thể làm *gì* với *tài nguyên nào*, và triển khai các mô hình quyền truy cập tối thiểu.

---

## Hoạt Động & Học Hỏi

### 1. Các Khái Niệm Cốt Lõi IAM

Nghiên cứu các thực thể IAM cơ bản và mối quan hệ của chúng:

| Thực thể | Mô tả |
|---|---|
| **Root Account** | Tài khoản chính; chỉ nên dùng cho billing và quản lý tài khoản |
| **IAM User** | Danh tính được đặt tên với thông tin xác thực lâu dài (access key / mật khẩu) |
| **IAM Group** | Tập hợp user; policies gắn vào group áp dụng cho tất cả thành viên |
| **IAM Role** | Danh tính với thông tin xác thực tạm thời; được user, dịch vụ hoặc ứng dụng đảm nhận |
| **IAM Policy** | Tài liệu JSON định nghĩa các hành động được phép/từ chối trên tài nguyên cụ thể |

### 2. Tạo IAM Users và Groups

- Tạo nhiều IAM user đại diện cho các chức năng công việc khác nhau (developer, analyst, auditor chỉ đọc)
- Tổ chức user vào **IAM Groups** theo chức năng:
  - `Developers` — EC2, S3, RDS full access
  - `Analysts` — Quyền đọc CloudWatch, S3
  - `Administrators` — AdministratorAccess managed policy
- Áp dụng **nguyên tắc quyền tối thiểu** — chỉ cấp quyền tối thiểu cần thiết cho mỗi vai trò

### 3. Tìm Hiểu Sâu IAM Policies

- Khám phá sự khác biệt giữa:
  - **AWS Managed Policies** — Được tạo sẵn bởi AWS (ví dụ: `AmazonS3ReadOnlyAccess`)
  - **Customer Managed Policies** — Policies JSON tùy chỉnh do chủ tài khoản tạo
  - **Inline Policies** — Nhúng trực tiếp vào user, group hoặc role (không tái sử dụng được)
- Viết các policy JSON tùy chỉnh định nghĩa câu lệnh Allow và Deny rõ ràng
- Sử dụng **IAM Policy Simulator** để xác thực policies trước khi gắn vào tài nguyên

**Ví dụ policy tùy chỉnh (S3 chỉ đọc cho một bucket cụ thể):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::my-internship-bucket",
        "arn:aws:s3:::my-internship-bucket/*"
      ]
    }
  ]
}
```

### 4. Thực Hành Tốt Nhất IAM

- ✅ Bật MFA cho tài khoản root (đã làm Tuần 1)
- ✅ Tạo IAM user riêng lẻ; không chia sẻ thông tin xác thực
- ✅ Áp dụng policies quyền tối thiểu
- ✅ Xoay vòng access keys; vô hiệu hóa các khóa không sử dụng
- ✅ Bật **IAM Access Analyzer** để phát hiện quyền truy cập tài nguyên không có chủ đích
- ✅ Xem xét báo cáo thông tin xác thực IAM cho tất cả users

---

## Điểm Rút Ra

IAM là dịch vụ quan trọng nhất cần hiểu trước khi làm việc với bất kỳ dịch vụ AWS nào khác. Mọi tương tác tài nguyên trong AWS đều đi qua IAM. Các lỗi phổ biến nhất — policy quá rộng quyền, sử dụng thông tin xác thực root, không có MFA — cũng là những lỗi nguy hiểm nhất.

---

## Tài Nguyên Tham Khảo

- [Tài liệu AWS IAM](https://docs.aws.amazon.com/iam/)
- [IAM Policy Simulator](https://policysim.aws.amazon.com/)
- [AWS Security Best Practices](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/)
