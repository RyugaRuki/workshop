---
title : "Verify the public magazine and article reader"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.4.3 </b> "
---

Open the production site:

[https://d3u5pkyxnd3uus.cloudfront.net/](https://d3u5pkyxnd3uus.cloudfront.net/)

#### Public behavior

- The magazine is readable without an account.
- Article cards use the processed CloudFront WebP URL returned by the API.
- A reader can open full cleaned article text, the generated summary, source attribution, and the original publisher link.
- Login is required only for likes and comments.
- The operations page uses a separate admin session and never embeds the admin API key.

![Deployed CloudBrief article reader](/images/5-workshop/cloudbrief-evidence/article-reader.png)

#### Real article proof

The browser test opened a real Hacker News article, displayed its cleaned full text, summary, source URL, processed cover image, and reader discussion controls. Desktop and mobile checks showed no horizontal overflow.

#### Frontend/API boundary

The frontend never downloads or converts the publisher image. It receives the final `imageUrl` from the API. This keeps file validation, resizing, conversion, S3 upload, and CDN URL ownership in the backend worker.
