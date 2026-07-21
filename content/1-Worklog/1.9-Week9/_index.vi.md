---
title: "Tuần 9 – Giám Sát với Amazon CloudWatch"
weight: 9
date: 2026-06-15
---

**Thời gian:** 15/06/2026 – 21/06/2026

---

## Tổng Quan

Tuần 9 tập trung vào **Amazon CloudWatch** — nền tảng giám sát và quan sát toàn diện của AWS. Tuần này bao gồm thiết lập thu thập metrics, tạo alarms để cảnh báo tự động, xây dựng dashboards vận hành và hiểu về tổng hợp log với CloudWatch Logs.

---

## Hoạt Động & Học Hỏi

### 1. Tổng Quan Amazon CloudWatch

| Khả năng | Mô tả |
|---|---|
| **Metrics** | Điểm dữ liệu số được thu thập từ dịch vụ AWS và ứng dụng tùy chỉnh |
| **Alarms** | Thông báo và hành động tự động khi metrics vượt ngưỡng |
| **Logs** | Lưu trữ, tìm kiếm và phân tích log tập trung |
| **Dashboards** | Bảng giám sát trực quan kết hợp nhiều metrics và alarms |
| **Events / EventBridge** | Định tuyến sự kiện dựa trên quy tắc cho quy trình tự động |
| **Insights** | Phát hiện bất thường tự động và công cụ truy vấn log |

### 2. CloudWatch Metrics

**Metrics mặc định được thu thập tự động:**

| Dịch vụ | Metrics Chính |
|---|---|
| EC2 | CPUUtilization, NetworkIn, NetworkOut, StatusCheckFailed |
| RDS | CPUUtilization, FreeStorageSpace, DatabaseConnections |
| S3 | BucketSizeBytes, NumberOfObjects |
| ALB | RequestCount, HTTPCode_ELB_5XX, TargetResponseTime |
| Auto Scaling | GroupInServiceInstances, GroupDesiredCapacity |

**Custom metrics qua CloudWatch Agent:**
- Cài đặt **CloudWatch Agent** trên EC2 instances
- Cấu hình agent thu thập memory utilization và disk usage (không có mặc định)
- Push custom metrics qua CLI:

```bash
aws cloudwatch put-metric-data \
  --namespace "MyApp/Performance" \
  --metric-name "ActiveUsers" \
  --value 42 \
  --unit Count \
  --region ap-southeast-1
```

### 3. CloudWatch Alarms

Tạo alarms để cảnh báo tự động về các điều kiện bất thường:

**Alarm CPU cao EC2:**
- Metric: `CPUUtilization`
- Ngưỡng: Lớn hơn 80% trong 3 chu kỳ 5 phút liên tiếp
- Hành động: Gửi thông báo SNS qua email + kích hoạt Auto Scaling policy

**Alarm lưu trữ RDS thấp:**
- Metric: `FreeStorageSpace`
- Ngưỡng: Ít hơn 2 GB
- Hành động: Thông báo SNS đến nhóm trực

**Alarm ngân sách (billing):**
- Metric: `EstimatedCharges`
- Ngưỡng: $8 USD
- Hành động: Thông báo email

### 4. CloudWatch Logs

- Cấu hình EC2 instances stream application logs đến **CloudWatch Logs**
- Tạo **Log Groups** cho ứng dụng và môi trường khác nhau
- Dùng **Metric Filters** để trích xuất metrics từ dữ liệu log (ví dụ: đếm dòng log `ERROR`)
- Dùng **CloudWatch Logs Insights** để truy vấn logs:

```
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 50
```

### 5. CloudWatch Dashboards

Xây dựng dashboard vận hành kết hợp:
- CPU utilization EC2 trên tất cả instances trong Auto Scaling Group
- Số lượng request ALB và tỷ lệ lỗi 5XX
- CPU và free storage RDS
- Custom metric "Active Users"
- Widget trạng thái alarm

Dashboard được chia sẻ với mentor dự án để review.

---

## Điểm Rút Ra

Giám sát là bắt buộc với bất kỳ hệ thống production nào — đó là cách bạn biết ứng dụng đang hoạt động tốt trước khi khách hàng báo cáo sự cố. CloudWatch cung cấp stack quan sát toàn diện mà không cần công cụ bên thứ ba. Điều quan trọng nhất là xác định ngưỡng cảnh báo *trước* khi đưa vào production.

---

## Tài Nguyên Tham Khảo

- [Tài liệu Amazon CloudWatch](https://docs.aws.amazon.com/cloudwatch/)
- [Cấu hình CloudWatch Agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)
- [Cú pháp truy vấn CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html)
