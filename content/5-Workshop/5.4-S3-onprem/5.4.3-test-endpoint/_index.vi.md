---
title : "Verify magazine và article reader public"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.4.3 </b> "
---

Mở production site:

[https://d3u5pkyxnd3uus.cloudfront.net/](https://d3u5pkyxnd3uus.cloudfront.net/)

#### Hành vi public

- Có thể đọc magazine mà không cần account.
- Article card dùng processed CloudFront WebP URL do API trả về.
- Reader có thể mở full cleaned article text, generated summary, source attribution và link publisher gốc.
- Chỉ like và comment mới cần login.
- Operations page dùng admin session riêng và không embed admin API key.

![CloudBrief article reader đã deploy](/images/5-workshop/cloudbrief-evidence/article-reader.png)

#### Bằng chứng article thật

Browser test đã mở một bài Hacker News thật, hiển thị cleaned full text, summary, source URL, processed cover image và reader discussion controls. Kiểm tra desktop và mobile không có horizontal overflow.

#### Ranh giới frontend/API

Frontend không download hoặc convert ảnh publisher. Frontend chỉ nhận `imageUrl` cuối từ API. Việc file validation, resize, conversion, S3 upload và CDN URL thuộc trách nhiệm backend worker.
