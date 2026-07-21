---
title : "Chuẩn bị tài khoản AWS và môi trường local"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

Phần này giữ cấu trúc hướng dẫn và bằng chứng như workshop mẫu, nhưng toàn bộ thao tác, công cụ, quyền và ảnh đều thuộc dự án CloudBrief.

#### 1. Quyền IAM

Người chuẩn bị tài khoản workshop cần cấp cho deployment user quyền tạo và xóa CloudBrief stack. [Tải policy JSON](/files/cloudbrief-workshop-deployer-policy.json) với tên `cloudbrief-workshop-deployer-policy.json`, hoặc lưu nội dung bên dưới bằng đúng tên file đó.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CloudBriefInfrastructure",
      "Effect": "Allow",
      "Action": [
        "autoscaling:*",
        "backup:*",
        "bedrock:*",
        "budgets:*",
        "cloudformation:*",
        "cloudfront:*",
        "cloudwatch:*",
        "dynamodb:*",
        "ec2:*",
        "elasticloadbalancing:*",
        "events:*",
        "kms:*",
        "logs:*",
        "s3:*",
        "sns:*",
        "sqs:*",
        "ssm:*",
        "wafv2:*"
      ],
      "Resource": "*"
    },
    {
      "Sid": "CloudBriefDeploymentRoles",
      "Effect": "Allow",
      "Action": [
        "iam:AttachRolePolicy",
        "iam:CreateInstanceProfile",
        "iam:CreatePolicy",
        "iam:CreateRole",
        "iam:DeleteInstanceProfile",
        "iam:DeletePolicy",
        "iam:DeleteRole",
        "iam:DeleteRolePolicy",
        "iam:DetachRolePolicy",
        "iam:GetInstanceProfile",
        "iam:GetPolicy",
        "iam:GetRole",
        "iam:PassRole",
        "iam:PutRolePolicy",
        "iam:RemoveRoleFromInstanceProfile",
        "iam:AddRoleToInstanceProfile",
        "iam:TagInstanceProfile",
        "iam:TagPolicy",
        "iam:TagRole",
        "iam:UntagInstanceProfile",
        "iam:UntagPolicy",
        "iam:UntagRole",
        "iam:UpdateAssumeRolePolicy",
        "sts:AssumeRole"
      ],
      "Resource": [
        "arn:aws:iam::*:role/cloudbrief-*",
        "arn:aws:iam::*:role/cdk-hnb659fds-*",
        "arn:aws:iam::*:instance-profile/cloudbrief-*",
        "arn:aws:iam::*:policy/cloudbrief-*"
      ]
    }
  ]
}
```

Chạy các lệnh sau bằng administrator profile hoặc account-bootstrap profile. Chuỗi lệnh sẽ tạo user nếu chưa có, tạo hoặc dùng lại customer-managed policy, attach policy và hiển thị kết quả cuối mà không in credential.

```bash
ADMIN_PROFILE=cloudbrief-bootstrap-admin
DEPLOY_USER=cloudbrief-workshop
POLICY_NAME=cloudbrief-workshop-deployer
POLICY_FILE=cloudbrief-workshop-deployer-policy.json

aws iam get-user \
  --user-name "$DEPLOY_USER" \
  --profile "$ADMIN_PROFILE" >/dev/null 2>&1 || \
aws iam create-user \
  --user-name "$DEPLOY_USER" \
  --profile "$ADMIN_PROFILE"

POLICY_ARN=$(aws iam list-policies \
  --scope Local \
  --query "Policies[?PolicyName=='${POLICY_NAME}'].Arn | [0]" \
  --output text \
  --profile "$ADMIN_PROFILE")

if [ "$POLICY_ARN" = "None" ]; then
  POLICY_ARN=$(aws iam create-policy \
    --policy-name "$POLICY_NAME" \
    --policy-document "file://${POLICY_FILE}" \
    --query 'Policy.Arn' \
    --output text \
    --profile "$ADMIN_PROFILE")
fi

aws iam attach-user-policy \
  --user-name "$DEPLOY_USER" \
  --policy-arn "$POLICY_ARN" \
  --profile "$ADMIN_PROFILE"

aws iam list-attached-user-policies \
  --user-name "$DEPLOY_USER" \
  --profile "$ADMIN_PROFILE"
```

{{% notice warning %}}
Policy này chủ đích phục vụ deployment và dùng wildcard theo service vì CDK phải tạo các resource chưa có identifier trước khi deploy. Chỉ dùng policy trong tài khoản workshop. Với production, hãy deploy qua AWS IAM Identity Center hoặc CI role, giới hạn resource và condition, đồng thời tách deployment permission khỏi runtime permission.
{{% /notice %}}

#### 2. Công cụ cần có

| Công cụ | Mục đích |
| --- | --- |
| AWS account và IAM user | Deploy và kiểm tra tài nguyên workshop |
| AWS CLI v2 | Kiểm tra identity và thu thập bằng chứng hạ tầng read-only |
| Bun `1.3.x` | Cài monorepo, build, test và chạy scripts |
| AWS CDK v2 | Synth và deploy CloudFormation stack |
| Git | Quản lý source và bằng chứng triển khai |

Dự án deploy tại `us-east-1`. Amazon Bedrock cần cho phép model `amazon.nova-micro-v1:0`.

#### 3. Tạo IAM user riêng cho workshop

Mở **IAM > Users**, sau đó chọn **Create user**.

![Mở danh sách IAM user]({{< asseturl "images/5-Workshop/5.2-Prerequisite/iam-01-existing-user.png" >}})

Nhập tên user dành riêng cho workshop. Console access là tùy chọn nếu AWS CLI là công cụ deploy chính.

![Nhập thông tin IAM user]({{< asseturl "images/5-Workshop/5.2-Prerequisite/iam-02-user-details.png" >}})

Trong lần deploy demo ban đầu, nhóm tạm gắn `AdministratorAccess` để CDK tạo đầy đủ stack. Đây là bằng chứng lịch sử của workshop, không phải khuyến nghị production. Tài khoản production nên dùng deployment role giới hạn quyền và gỡ policy tạm sau khi deploy.

![Gắn quyền deploy]({{< asseturl "images/5-Workshop/5.2-Prerequisite/iam-03-permissions.png" >}})

Kiểm tra lại user và policy trước khi chọn **Create user**.

![Kiểm tra IAM user]({{< asseturl "images/5-Workshop/5.2-Prerequisite/iam-04-review-create.png" >}})

Xác nhận user mới xuất hiện trong danh sách IAM.

![IAM user đã được tạo]({{< asseturl "images/5-Workshop/5.2-Prerequisite/iam-05-user-created.png" >}})

#### 4. Tạo credential cho AWS CLI

Mở tab **Security credentials** của user và chọn **Create access key**.

![Mở security credentials]({{< asseturl "images/5-Workshop/5.2-Prerequisite/iam-06-security-credentials.png" >}})

Chọn **Command Line Interface (CLI)**, xác nhận khuyến nghị và tiếp tục.

![Chọn use case CLI]({{< asseturl "images/5-Workshop/5.2-Prerequisite/iam-07-cli-use-case.png" >}})

Thêm description tag để key có owner và mục đích rõ ràng.

![Thêm mô tả cho access key]({{< asseturl "images/5-Workshop/5.2-Prerequisite/iam-08-key-description.png" >}})

Chỉ tải CSV một lần và lưu ngoài repository. Ảnh public đã che access-key ID và secret vì các giá trị này không được công khai.

![Access key đã tạo và được che giá trị]({{< asseturl "images/5-Workshop/5.2-Prerequisite/iam-09-key-created.png" >}})

{{% notice warning %}}
Không commit hai file credentials CSV, `backend/.env`, API key, origin secret hoặc AWS account identifier. Hãy rotate hoặc deactivate long-lived key sau workshop.
{{% /notice %}}

#### 5. Cấu hình và kiểm tra AWS CLI

```bash
aws configure --profile cloudbrief-workshop
aws sts get-caller-identity --profile cloudbrief-workshop
aws configure get region --profile cloudbrief-workshop
```

Đặt region mặc định là `us-east-1` và output format là `json`. Ngày 16/07/2026, access key được cung cấp đã xác thực thành công với dedicated CloudBrief teammate user. Account identifier đã được loại khỏi báo cáo.

#### 6. Chuẩn bị repository

```bash
cd ~/Code/Technology-News-Collection-and-Summarization-System
bun install
cp backend/.env.example backend/.env
```

Cấu hình local cho `DEMO_API_KEY`, `BUDGET_EMAIL`, `AWS_REGION`, `BEDROCK_REGION` và `SUMMARY_MODEL_ID`. Chỉ giữ giá trị thật trong local environment hoặc AWS configuration store được phê duyệt.

#### Checklist sẵn sàng

- Đã tạo IAM user và CLI key.
- AWS CLI identity check thành công.
- Region là `us-east-1`.
- Bun và CDK đã sẵn sàng.
- Có quyền dùng Bedrock Nova Micro.
- Credential và secret được loại khỏi Git.
