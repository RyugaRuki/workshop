---
title: "Tuần 3 – Amazon VPC & Mạng"
weight: 3
date: 2026-05-04
---

**Thời gian:** 04/05/2026 – 10/05/2026

---

## Tổng Quan

Tuần 3 giới thiệu **Amazon Virtual Private Cloud (VPC)** — nền tảng mạng cho toàn bộ hạ tầng AWS. Tuần này bao gồm thiết kế và triển khai VPC tùy chỉnh với các subnet công khai và riêng tư, định tuyến lưu lượng qua Internet Gateway và NAT Gateway. Một sự kiện trực tiếp thú vị tại văn phòng AWS cũng diễn ra trong tuần.

---

## Hoạt Động & Học Hỏi

### 1. Cơ Bản Amazon VPC

Các thành phần cốt lõi của AWS VPC:

| Thành phần | Mục đích |
|---|---|
| **VPC** | Mạng ảo cô lập trong một AWS region |
| **Subnet** | Dải địa chỉ IP trong VPC (công khai hoặc riêng tư) |
| **Route Table** | Các quy tắc xác định lưu lượng mạng được định tuyến đi đâu |
| **Internet Gateway (IGW)** | Cho phép truy cập internet cho tài nguyên trong subnet công khai |
| **NAT Gateway** | Cho phép tài nguyên subnet riêng tư khởi tạo kết nối internet chiều ra |
| **Security Group** | Tường lửa trạng thái ở cấp độ instance |
| **Network ACL** | Tường lửa phi trạng thái ở cấp độ subnet |

### 2. Thiết Kế Kiến Trúc VPC Đa Tầng

Thiết kế và triển khai VPC với kiến trúc sau:

```
VPC: 10.0.0.0/16
├── Public Subnet A  (10.0.1.0/24) — AZ: ap-southeast-1a
├── Public Subnet B  (10.0.2.0/24) — AZ: ap-southeast-1b
├── Private Subnet A (10.0.3.0/24) — AZ: ap-southeast-1a
└── Private Subnet B (10.0.4.0/24) — AZ: ap-southeast-1b
```

- **Internet Gateway** gắn vào VPC để truy cập internet cho public subnet
- **NAT Gateway** triển khai trong Public Subnet A để cho phép các instance riêng tư kết nối internet khi cập nhật
- **Route Tables** được cấu hình:
  - Route table công khai: `0.0.0.0/0 → Internet Gateway`
  - Route table riêng tư: `0.0.0.0/0 → NAT Gateway`

### 3. Security Groups vs Network ACLs

| Tính năng | Security Group | Network ACL |
|---|---|---|
| Cấp độ | Instance | Subnet |
| Trạng thái | Stateful | Stateless |
| Quy tắc | Chỉ Allow | Allow và Deny |
| Đánh giá | Tất cả quy tắc | Theo thứ tự |

### 4. VPC Flow Logs

- Bật **VPC Flow Logs** để ghi lại tất cả metadata lưu lượng IP qua VPC
- Cấu hình logs lưu vào S3 bucket để phân tích
- Phân tích các bản ghi flow log để hiểu mô hình `ACCEPT` và `REJECT`

### 5. Sự Kiện Văn Phòng AWS — Dịch Vụ Đám Mây & AI

Tham dự sự kiện trực tiếp tại **văn phòng AWS Việt Nam** dành cho thực tập sinh FCAJ:
- Tham quan văn phòng AWS và gặp gỡ Solutions Architect, kỹ sư đám mây
- Dự buổi thuyết trình về các dịch vụ AWS (compute, ML, IoT, analytics)
- Tham gia demo **Amazon Bedrock** và **Amazon Q** cho quy trình làm việc hỗ trợ AI
- Giao lưu với các thực tập sinh và nhóm AWS

Sự kiện này cung cấp bối cảnh thực tế thú vị cho công việc kỹ thuật đang được thực hiện trong các lab.

---

## Điểm Rút Ra

VPC được thiết kế tốt là quyết định kiến trúc quan trọng nhất bạn thực hiện trên AWS. Việc phân tách các tầng công khai và riêng tư, kết hợp với định tuyến và security group đúng cách, cung cấp khả năng phòng thủ theo chiều sâu. Thiết kế subnet multi-AZ (hai AZ) là mức tối thiểu cho bất kỳ kiến trúc production nào.

---

## Tài Nguyên Tham Khảo

- [Tài liệu Amazon VPC](https://docs.aws.amazon.com/vpc/)
- [VPC Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html)
