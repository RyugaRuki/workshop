---
title : "Test pipeline RSS và Hacker News thật"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

#### Bước 1: Verify public health route

```bash
curl -i https://d3u5pkyxnd3uus.cloudfront.net/api/health
```

Kết quả verify ngày 16/07/2026: HTTP `200`, project `cloudbrief`, stage `dev`, status `healthy`.

#### Bước 2: Verify protected collection

Khi gọi admin collection route không có credential, API phải trả unauthorized. Request admin hợp lệ nhận source RSS và Hacker News, sau đó queue công việc qua collection, extraction, summarization và image processing.

```bash
curl -X POST https://d3u5pkyxnd3uus.cloudfront.net/api/v1/collect \
  -H 'content-type: application/json' \
  -d '{"sources":["rss","hackernews"],"limit":5}'
```

Không đưa admin key thật vào screenshot hoặc shell history.

#### Bước 3: Đọc brief thật

```bash
curl 'https://d3u5pkyxnd3uus.cloudfront.net/api/v1/brief?limit=5'
```

Article contract mong đợi:

```json
{
  "id": "article-id",
  "title": "Publisher title",
  "summary": "Bedrock hoặc deterministic fallback summary",
  "canonicalUrl": "https://publisher.example/article",
  "imageUrl": "https://cloudfront.example/article-images/cover.webp"
}
```

Worker nhận ảnh publisher PNG, JPEG hoặc WebP, validate và convert sang WebP, upload vào private S3 bucket rồi chỉ lưu CloudFront URL trong DynamoDB.

Live response ngày 16/07 trả về năm article Hacker News. Cả năm đều có canonical publisher URL, status `SUMMARIZED` và processed CloudFront WebP cover URL. Một title đã verify là **Bluesky Trademarks ATProto** từ trang AT Protocol.

![API smoke test đã pass](/images/5-workshop/cloudbrief-evidence/api-smoke-ok.png)

#### Kết quả Bedrock

Nova Micro được cấu hình là model chi phí thấp với giới hạn 256 output token. Trong deployment đã ghi nhận, tài khoản AWS chạm daily Nova token allowance. CloudBrief vì vậy lưu deterministic fallback summary thay vì làm mất article; CloudWatch hiển thị trạng thái này.
