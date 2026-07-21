---
title : "Test the real RSS and Hacker News pipeline"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

#### Step 1: Verify the public health route

```bash
curl -i https://d3u5pkyxnd3uus.cloudfront.net/api/health
```

Verified on 16 July 2026: HTTP `200`, project `cloudbrief`, stage `dev`, status `healthy`.

#### Step 2: Verify protected collection

Calling the admin collection route without credentials must return an unauthorized response. A valid admin request accepts RSS and Hacker News sources and queues work through collection, extraction, summarization, and image processing.

```bash
curl -X POST https://d3u5pkyxnd3uus.cloudfront.net/api/v1/collect \
  -H 'content-type: application/json' \
  -d '{"sources":["rss","hackernews"],"limit":5}'
```

Do not place the real admin key in a screenshot or shell history.

#### Step 3: Read a real brief

```bash
curl 'https://d3u5pkyxnd3uus.cloudfront.net/api/v1/brief?limit=5'
```

Expected article contract:

```json
{
  "id": "article-id",
  "title": "Publisher title",
  "summary": "Bedrock or deterministic fallback summary",
  "canonicalUrl": "https://publisher.example/article",
  "imageUrl": "https://cloudfront.example/article-images/cover.webp"
}
```

The worker accepts publisher PNG, JPEG, or WebP images, validates and converts them to WebP, uploads them to a private S3 bucket, and stores only the CloudFront URL in DynamoDB.

The live response on 16 July returned five Hacker News articles. All five had a canonical publisher URL, `SUMMARIZED` status, and a processed CloudFront WebP cover URL. One verified title was **Bluesky Trademarks ATProto** from the AT Protocol site.

![API smoke test passed](/images/5-Workshop/cloudbrief-evidence/api-smoke-ok.png)

#### Bedrock result

Nova Micro is configured as the low-cost model with a 256-token output cap. During the recorded deployment, the AWS account reached its daily Nova token allowance. CloudBrief therefore stored deterministic fallback summaries instead of dropping articles; CloudWatch exposes this condition.
