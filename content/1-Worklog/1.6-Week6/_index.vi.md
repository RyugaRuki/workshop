---
title: "Tuần 6 – Amazon S3 & Khởi Động Dự Án Nhóm"
weight: 6
date: 2026-05-25
---

**Thời gian:** 25/05/2026 – 31/05/2026

---

## Tổng Quan

Tuần 6 giới thiệu **Amazon Simple Storage Service (S3)** với trọng tâm là lưu trữ website tĩnh, cùng với việc khởi động chính thức dự án workshop nhóm. Một sự kiện AWS cũng diễn ra tuần này với cơ hội học tập từ đồng nghiệp và gặp gỡ mentor.

---

## Hoạt Động & Học Hỏi

### 1. Các Khái Niệm Cốt Lõi Amazon S3

| Khái niệm | Mô tả |
|---|---|
| **Bucket** | Container chứa objects; yêu cầu tên duy nhất toàn cầu |
| **Object** | File cộng với metadata; được xác định bằng key |
| **Key** | Đường dẫn đầy đủ của object trong bucket (ví dụ: `images/photo.jpg`) |
| **Storage Class** | Xác định đánh đổi chi phí/truy xuất (Standard, IA, Glacier, v.v.) |
| **Versioning** | Lưu giữ các phiên bản trước của objects khi cập nhật hoặc xóa |
| **Bucket Policy** | Policy JSON cấp quyền truy cập vào bucket |

### 2. Lưu Trữ Website Tĩnh trên Amazon S3

Triển khai website tĩnh bằng tính năng hosting tích hợp của S3:

1. Tạo S3 bucket với tên phù hợp
2. Bật **Static Website Hosting** trong thuộc tính bucket
3. Đặt `index.html` là tài liệu chỉ mục và `404.html` là tài liệu lỗi
4. Tắt cài đặt **Block Public Access** để cho phép đọc công khai
5. Gắn **Bucket Policy** cho phép `s3:GetObject` công khai:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-fcaj-site/*"
    }
  ]
}
```

6. Upload các file HTML, CSS và JS
7. Truy cập site qua URL endpoint website S3

### 3. Versioning và Lifecycle Policies

- Bật **Versioning** để lưu giữ tất cả phiên bản lịch sử của objects
- Thực hành khôi phục phiên bản trước của file sau khi vô tình ghi đè
- Tạo **Lifecycle Rules**:
  - Chuyển objects cũ hơn 30 ngày sang S3 Standard-IA (rẻ hơn)
  - Tự động xóa objects cũ hơn 365 ngày

### 4. Bảo Mật S3

- Hiểu sự khác biệt giữa **Bucket ACL**, **Bucket Policy** và **IAM Policy**
- Bật **Server-Side Encryption (SSE-S3)** làm mã hóa mặc định
- Bật **S3 Access Logs** để theo dõi tất cả yêu cầu đến bucket

### 5. Khởi Động Dự Án Nhóm

- Thành lập nhóm với các thực tập sinh FCAJ
- Tổ chức buổi brainstorming để xác định phạm vi dự án
- Thống nhất xây dựng **Ứng dụng Web Khả Dụng Cao** sử dụng EC2, VPC (Multi-AZ), Auto Scaling và RDS
- Phân chia trách nhiệm: mạng, tính toán, cơ sở dữ liệu và tài liệu
- Thiết lập GitHub repository chung

### 6. Sự Kiện Chia Sẻ Kiến Thức AWS

Tham dự sự kiện giữa chương trình với mentor và thực tập sinh FCAJ:
- Chia sẻ kinh nghiệm từ 6 tuần lab đầu tiên
- Thảo luận các thách thức thường gặp với VPC và IAM
- Mentor cung cấp phản hồi về bản phác thảo kiến trúc dự án nhóm

---

## Điểm Rút Ra

S3 là một trong những dịch vụ linh hoạt nhất trên AWS — không chỉ là lưu trữ, mà còn là nền tảng hosting, target ghi log, destination sao lưu và nền tảng data lake. Lưu trữ website tĩnh trên S3 là cách triển khai frontend đơn giản tiết kiệm chi phí mà không cần quản lý server.

---

## Tài Nguyên Tham Khảo

- [Tài liệu Amazon S3](https://docs.aws.amazon.com/s3/)
- [Lưu trữ Website Tĩnh trên S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
