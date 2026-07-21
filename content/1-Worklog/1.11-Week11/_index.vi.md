---
title: "Tuần 11 – AWS CLI & Sprint Workshop"
weight: 11
date: 2026-06-29
---

**Thời gian:** 29/06/2026 – 05/07/2026

---

## Tổng Quan

Tuần 11 đánh dấu sự chuyển đổi từ xây dựng kỹ năng cá nhân sang chuẩn bị bàn giao cuối kỳ. Tuần này bao gồm thực hành **AWS CLI** trên cả Windows và Ubuntu EC2 instances, làm việc chuyên sâu cho dự án workshop nhóm và bắt đầu hoàn thiện báo cáo thực tập.

---

## Hoạt Động & Học Hỏi

### 1. AWS CLI trên EC2 — Linux (Ubuntu)

Thực hành sử dụng **AWS CLI v2** trên EC2 Ubuntu cho các thao tác hạ tầng phổ biến:

**Cài đặt:**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

**Các thao tác thường dùng:**
```bash
# Liệt kê EC2 instances đang chạy
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query "Reservations[].Instances[].[InstanceId,InstanceType,PublicIpAddress]" \
  --output table

# Tạo S3 bucket và upload file
aws s3 mb s3://my-cli-test-bucket-$(date +%s)
aws s3 cp ./index.html s3://my-cli-test-bucket/
aws s3 sync ./website/ s3://my-cli-test-bucket/

# Thao tác IAM
aws iam list-users --query "Users[].UserName" --output table
```

### 2. AWS CLI trên EC2 — Windows (PowerShell)

Cấu hình và sử dụng AWS CLI trên **Windows Server EC2** instance:

```powershell
# Cài đặt AWS CLI
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi /quiet
aws --version

# Lấy instance metadata (IMDSv2)
$token = Invoke-RestMethod -Method PUT `
  -Uri "http://169.254.169.254/latest/api/token" `
  -Headers @{"X-aws-ec2-metadata-token-ttl-seconds"="21600"}

$instanceId = Invoke-RestMethod `
  -Uri "http://169.254.169.254/latest/meta-data/instance-id" `
  -Headers @{"X-aws-ec2-metadata-token"=$token}

Write-Output "Instance ID: $instanceId"
```

### 3. Script Tự Động Hóa CLI

Sử dụng CLI để tự động hóa các thao tác lặp lại:

```bash
#!/bin/bash
# Script sao lưu tự động hàng đêm
INSTANCE_ID=$(curl -sf http://169.254.169.254/latest/meta-data/instance-id)
VOLUMES=$(aws ec2 describe-volumes \
  --filters "Name=attachment.instance-id,Values=$INSTANCE_ID" \
  --query "Volumes[].VolumeId" \
  --output text)

for VOLUME_ID in $VOLUMES; do
  aws ec2 create-snapshot \
    --volume-id "$VOLUME_ID" \
    --description "Auto backup - $(date +%Y-%m-%d)" \
    --tag-specifications "ResourceType=snapshot,Tags=[{Key=Name,Value=auto-backup}]"
  echo "Đã tạo snapshot cho $VOLUME_ID"
done
```

### 4. Sprint Workshop Nhóm

Phần lớn Tuần 11 dành cho phát triển workshop nhóm chuyên sâu:

- **Hoàn thiện kiến trúc:** Review và khóa sơ đồ kiến trúc cuối (VPC + Multi-AZ subnets + ALB + Auto Scaling + RDS Multi-AZ)
- **Triển khai hạ tầng:** Mỗi thành viên triển khai thành phần được phân công
- **Kiểm thử tích hợp:** Kết nối app tier với RDS, xác minh load balancing qua AZs, kiểm tra Auto Scaling policies
- **Tài liệu hóa:** Bắt đầu viết tài liệu workshop (trang web này)
- **Code review:** Review CloudFormation snippets và User Data scripts của nhau

**Quyết định kỹ thuật quan trọng tuần này:**
- Dùng RDS với Multi-AZ cho database tier (không dùng Aurora, để ở trong Free Tier)
- Lưu thông tin xác thực database trong Secrets Manager (không dùng environment variables)
- Triển khai ALB access logs vào S3
- Thêm CloudWatch alarms vào dự án nhóm

### 5. Bắt Đầu Hoàn Thiện Báo Cáo Thực Tập

Bắt đầu tổng hợp báo cáo thực tập cuối kỳ:
- Tổ chức các mục nhật ký hàng tuần
- Soạn thảo phần tự đánh giá
- Phác thảo cấu trúc tài liệu dự án workshop
- Yêu cầu giám sát viên review bản thảo phần đề xuất

---

## Điểm Rút Ra

AWS CLI là công cụ không thể thiếu cho kỹ sư đám mây — nó cho phép tự động hóa, scripting và iteration nhanh mà GUI console không thể. Khả năng viết CLI scripts đáng tin cậy cho các tác vụ thường xuyên là kỹ năng production cơ bản. Sprint workshop tuần này củng cố giá trị của review kiến trúc và kiểm thử tích hợp.

---

## Tài Nguyên Tham Khảo

- [Tài liệu AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/)
- [AWS CLI Command Reference](https://awscli.amazonaws.com/v2/documentation/api/latest/index.html)
- [Truy vấn JSON với JMESPath](https://jmespath.org/tutorial.html)
