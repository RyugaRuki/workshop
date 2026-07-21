---
title: "Tuần 1 – Nhập Môn & Cài Đặt Tài Khoản AWS"
weight: 1
date: 2026-04-20
---

**Thời gian:** 20/04/2026 – 26/04/2026

---

## Tổng Quan

Tuần đầu tiên của kỳ thực tập FCAJ đánh dấu sự khởi đầu hành trình 12 tuần vào điện toán đám mây tại **Amazon Web Services Việt Nam**. Tuần này chủ yếu tập trung vào định hướng chương trình, cài đặt hành chính và thiết lập môi trường đám mây AWS an toàn, được cấu hình tốt từ đầu.

---

## Hoạt Động & Học Hỏi

### 1. Định Hướng Chương Trình

- Nhận tài liệu thực tập chính thức bao gồm lịch chương trình, tiêu chí đánh giá và hướng dẫn giao tiếp từ nhóm FCAJ
- Tham dự buổi định hướng ban đầu về triết lý học tập của chương trình: lab thực hành, học tập từ đồng nghiệp và phân phối dự án thực tế
- Kết nối với các thực tập sinh FCAJ và thiết lập kênh giao tiếp nhóm (Zalo, Email)
- Xem xét chương trình 12 tuần để hiểu phạm vi đầy đủ và đặt mục tiêu học tập cá nhân

### 2. Tạo Tài Khoản AWS

- Tạo **tài khoản root AWS** mới bằng địa chỉ email riêng theo best practices AWS
- Bật **Xác thực đa yếu tố (MFA)** cho tài khoản root bằng ứng dụng authenticator
- Tạo **IAM Admin user** riêng cho hoạt động hàng ngày, tránh sử dụng thông tin xác thực root
- Đăng nhập AWS Management Console và khám phá danh mục dịch vụ

### 3. Khám Phá AWS Free Tier

- Xem xét tài liệu **AWS Free Tier** để hiểu phạm vi dịch vụ có sẵn miễn phí trong 12 tháng
- Các dịch vụ Free Tier chính được ghi nhận:
  - **Amazon EC2:** 750 giờ/tháng t2.micro hoặc t3.micro (Linux/Windows)
  - **Amazon S3:** 5 GB lưu trữ tiêu chuẩn
  - **Amazon RDS:** 750 giờ/tháng db.t2.micro
  - **Amazon Lambda:** 1 triệu yêu cầu miễn phí/tháng
- Hiểu sự khác biệt giữa các loại *luôn miễn phí*, *miễn phí 12 tháng* và *dùng thử*

### 4. Cài Đặt AWS Budgets

- Tạo **ngân sách chi phí hàng tháng** $10 USD trong AWS Budgets để ngăn các khoản phí bất ngờ
- Cấu hình **ngưỡng cảnh báo** ở 50%, 80% và 100% ngân sách
- Thiết lập thông báo email khi vượt ngưỡng
- Bật **AWS Cost Explorer** để theo dõi xu hướng chi tiêu theo thời gian

### 5. Tổng Quan AWS Support Plans

- Nghiên cứu bốn cấp độ AWS Support:
  - **Basic** (Miễn phí) — Tài liệu, diễn đàn, AWS Health dashboard
  - **Developer** (~$29/tháng) — Hỗ trợ email trong giờ làm việc
  - **Business** (~$100+/tháng) — Hỗ trợ điện thoại, chat và email 24/7
  - **Enterprise** (Tùy chỉnh) — Technical Account Manager (TAM) + hướng dẫn chủ động
- Hiểu khi nào nên dùng **AWS Trusted Advisor**, **Personal Health Dashboard** và **Support Center**

---

## Điểm Rút Ra

Thiết lập môi trường AWS nền tảng với bảo mật đúng cách (MFA, IAM admin user) và kiểm soát chi phí (Budgets, cảnh báo) trước khi làm bất kỳ công việc kỹ thuật nào là thực hành tốt nhất quan trọng. AWS Free Tier đủ rộng rãi để hỗ trợ toàn bộ chương trình thực tập 12 tuần khi tài nguyên được dọn dẹp đúng cách sau mỗi lab.

---

## Tài Nguyên Tham Khảo

- [Tổng quan AWS Free Tier](https://aws.amazon.com/free/)
- [Thiết lập AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)
- [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
