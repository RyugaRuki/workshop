---
title : "Build và synth local"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

#### Bước 1: Cài dependencies

```bash
cd ~/Code/Technology-News-Collection-and-Summarization-System
bun install
```

#### Bước 2: Chạy production build

```bash
bun run build
```

![CloudBrief build thành công](/images/5-Workshop/cloudbrief-evidence/build-ok.png)

#### Bước 3: Chạy regression tests

```bash
bun run test
```

![CloudBrief tests đã pass](/images/5-Workshop/cloudbrief-evidence/tests-ok.png)

Test suite bao phủ route validation, API-key protection, SSRF-safe fetching, collection, extraction, image processing, social actions và infrastructure assertions.

#### Bước 4: Synth CloudFormation

```bash
bun run cdk synth
```

![CDK synth thành công](/images/5-Workshop/cloudbrief-evidence/cdk-synth-ok.png)

Synth là local gate cuối trước khi deploy. Bước này xác nhận CDK app tạo được CloudFormation template mà chưa thay đổi tài nguyên AWS.

#### Bước 5: Kiểm tra giá trị nhạy cảm

Trước khi deploy, xác nhận các giá trị sau đến từ local configuration, Keychain hoặc AWS store được phê duyệt và không nằm trong Git:

- Admin API key và hash.
- CloudFront origin header secret.
- Google OAuth client configuration.
- Notification email.
- AWS access key.
