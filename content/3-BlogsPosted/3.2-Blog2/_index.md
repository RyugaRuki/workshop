---
title: "Blog 2 - Simplify AWS AppSync Events Integration with Powertools for AWS Lambda"
date: 2024-01-02
weight: 2
chapter: false
---

### SIMPLIFY AWS APPSYNC EVENTS INTEGRATION WITH POWERTOOLS FOR AWS LAMBDA

Real-time capabilities have become essential in modern applications, where users expect immediate updates and interactive experiences. Whether you're building chat applications, live dashboards, gaming leaderboards, or IoT systems, AWS AppSync Events enables these real-time features through WebSocket APIs, allowing you to build scalable and performant real-time applications without worrying about connection management or scaling infrastructure.

However, integrating AWS AppSync Events into serverless architectures, specifically handling events in AWS Lambda, often introduces challenges regarding payload serialization, event routing, and batch error handling. To streamline this process, the Powertools for AWS Lambda toolkit introduced the AppSyncEventsResolver utility, removing the need for boilerplate code and simplifying the development workflow.

### The Role of Powertools for AWS Lambda

Powertools for AWS Lambda is a developer toolkit designed to optimize serverless development. It provides essential utilities for Logging, Tracing, Metrics, and Event Handling.

With the addition of AppSyncEventsResolver, Powertools now offers a clean, consistent interface for processing AppSync Events in Lambda functions. It handles the undifferentiated heavy lifting—parsing events, routing them to the correct handler based on patterns, and formatting the response to conform to AppSync expectations—enabling developers to focus entirely on core business logic.

### Core Features of AppSyncEventsResolver

AppSyncEventsResolver is built to resolve common event processing patterns in real-time applications. The diagram below illustrates how WebSocket events from AWS AppSync are routed to different handlers inside the Lambda function:
Workblog chuyển đổi
Nhận xét kiến trúc AWS
Giải thích workshop FCAJ
Kiểm tra file Markdown
Nội dung workshop FCAJ
Đệ quy mảng C++
Mẹo ngữ pháp B1
Đề ôn lập trình
Sửa lỗi PlayTranslate
Cách tải ứng dụng CH Play bị chặn quốc gia
Cách tải game Dot Abyss
Công nghệ chống copy paste ITCoder HUTECH
Sửa lỗi chuyển đổi kiểu SinhVien
Tổng chẵn mảng số
Hướng dẫn dùng thử DeepSeek API miễn phí
Viết game RTS đầy đủ code chạy ngay trong VSCode
Quản lý sách CRUD C
Hướng dẫn sử dụng token DeepSeek cho lập trình
Nghệ thuật ưu tiên trong lịch trình bận rộn
Lỗi spawn php ENOENT trong Node.js
Giải bài tập lý thuyết cơ sở dữ liệu
Implement Admin-Only Employer Addition Feature
Cách lấy layout trang web có sẵn
Implementing Delete All Functionality in ASP.NET
Bài tập quản lý sản phẩm và danh mục
Hướng dẫn tích hợp AI vào ASP.NET MVC
ASP.NET Core View Not Found Error
Package Compatibility Error in .NET Projects
Cách viết Chính sách Bảo mật website
Bắt đầu tích hợp AI vào trang web
Sửa lỗi và định dạng code C#
Skip freeze debuff, wait for buff video
Công cụ tạo giao diện web phổ biến
Workblog chuyển đổi
Blog 3: Amazon S3 Audit Logging – Part 2: Centralized Logging and Analysis
Centralized Logging and Analysis of S3 Data Events in AWS CloudTrail

When a security incident occurs—such as unauthorized access or a bulk deletion in an S3 bucket—the first question is always: “Who did this?” Traditional S3 server access logs show what request happened, but lack the full IAM identity context to identify the specific user or assumed role.

This post (Part 2 of our S3 audit series) explores how to implement an identity-driven audit system using AWS CloudTrail data events and Amazon Athena.
graph TD User[IAM User / Assumed Role] -- S3 API Call --> S3[Amazon S3] S3 -- Data Event Logs --> CloudTrail[AWS CloudTrail] CloudTrail -- Deliver .json.gz --> CentralS3[Central S3 Bucket] CentralS3 -- Query via Partition Projection --> Athena[Amazon Athena] Athena -- Query Results --> Analyst[Security Investigator]
S3 Data Events vs. Server Access Logs

CloudTrail data events capture granular API-level tracking for S3 object operations with identity information, complementing HTTP-level access logs.

    What is Captured: Detailed IAM user/role identities, API operations (GetObject, PutObject, DeleteObject), authentication context (MFA, role assumption chains), and cross-account access details.
    What is Not Captured: HTTP-level performance metrics, status codes, or precise response times (these require S3 server access logs as discussed in Part 1).
    Delivery & Cost: Logs are delivered as compressed JSON within ~5 minutes. Cost is $0.10 per 100,000 recorded data events. (Note: Data events do not have a free tier).

Step-by-Step Configuration

To set up centralized, organization-wide S3 audit logging:

    Create a CloudTrail Trail: Under the CloudTrail console, create a new trail (e.g., s3-data-events-trail) and target a centralized S3 bucket.
    Enable Organization-Wide Logging: If using AWS Organizations, check Enable for all accounts in my organization in the management account to automatically deploy the trail to all member accounts.
    Configure Advanced Event Selectors: Deselect management events (to avoid duplicate logging charges if another trail exists) and select Data Events with S3 as the resource type. You can filter by bucket or prefix.
    Configure S3 Lifecycle Policy: On the central bucket, set a lifecycle rule to transition logs to Standard-IA after 90 days, Glacier after 180 days, and expire them after 7 years to minimize storage costs.

Creating the Athena Table with Partition Projection

Partition projection calculates partition values dynamically from the S3 path, speeding up queries and reducing metadata scanning costs. Use the query below to create the table:

CREATE EXTERNAL TABLE cloudtrail_s3_events (
  eventversion STRING,
  useridentity STRUCT<
    type: STRING, principalid: STRING, arn: STRING, accountid: STRING,
    username: STRING, sessioncontext: STRUCT<
      attributes: STRUCT<mfaauthenticated: STRING, creationdate: STRING>,
      sessionissuer: STRUCT<arn: STRING, accountid: STRING, username: STRING>
    >
  >,
  eventtime STRING, eventsource STRING, eventname STRING, awsregion STRING,
  sourceipaddress STRING, useragent STRING, errorcode STRING, errormessage STRING,
  requestparameters STRING, responseelements STRING, additionaleventdata STRING,
  recipientaccountid STRING, sharedEventId STRING
)
PARTITIONED BY (
  account STRING,
  region STRING,
  `timestamp` STRING 
)
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
STORED AS INPUTFORMAT 'com.amazon.emr.cloudtrail.CloudTrailInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION 's3://centralized-s3-cloudtrail-logs/AWSLogs/'
TBLPROPERTIES (
  'projection.enabled' = 'true',
  'projection.account.type' = 'injected',
  'projection.region.type' = 'injected',
  'projection.timestamp.type' = 'date',
  'projection.timestamp.format' = 'yyyy/MM/dd',
  'projection.timestamp.range' = '2024/01/01,NOW',
  'projection.timestamp.interval' = '1',
  'projection.timestamp.interval.unit' = 'DAYS', 
  'storage.location.template' = 's3://centralized-s3-cloudtrail-logs/AWSLogs/${account}/CloudTrail/${region}/${timestamp}/'
);

Note: For Organization-wide trails, add organization_id STRING as the first partition column and inject it into the storage.location.template.
Key Security Query Patterns

Athena partition projection requires specifying account, region, and timestamp in the WHERE clause of every query to prune partitions and avoid expensive full-table scans.
1. Trace Specific User Activity

Identifies S3 data-plane actions performed by an IAM user or role on a given day:

SELECT eventtime, useridentity.arn as user_arn, eventname, sourceipaddress,
       JSON_EXTRACT_SCALAR(requestparameters, '$.bucketName') as bucket,
       JSON_EXTRACT_SCALAR(requestparameters, '$.key') as object_key
FROM cloudtrail_s3_events
WHERE account = '123456789012' AND region = 'us-east-1' AND timestamp = '2026/05/15'
  AND (useridentity.username = 'specific-user' OR useridentity.arn LIKE '%specific-user%')
  AND eventname IN ('GetObject', 'PutObject', 'DeleteObject')
ORDER BY eventtime DESC;

2. Cross-Account Access Monitoring

Flags S3 operations originating from external AWS accounts:

SELECT eventtime, useridentity.accountid as source_account, recipientaccountid as target_account,
       useridentity.arn as user_arn, eventname, sourceipaddress
FROM cloudtrail_s3_events
WHERE account = '123456789012' AND region = 'us-east-1' AND timestamp = '2026/05/15'
  AND useridentity.accountid != recipientaccountid
ORDER BY eventtime DESC;

3. Surface Failed Access Attempts

Detects permission errors (AccessDenied, NoSuchKey), highlighting potential scanning or brute-force attempts:

SELECT eventtime, useridentity.arn as user_arn, eventname, sourceipaddress, errorcode, errormessage
FROM cloudtrail_s3_events
WHERE account = '123456789012' AND region = 'us-east-1' AND timestamp = '2026/05/15'
  AND errorcode IS NOT NULL
ORDER BY eventtime DESC LIMIT 100;

4. Delete Operations Audit

Audits object deletions over a date range, tracking assumed roles and MFA status:

SELECT eventtime, useridentity.arn as user_arn, sourceipaddress,
       JSON_EXTRACT_SCALAR(requestparameters, '$.bucketName') as bucket,
       JSON_EXTRACT_SCALAR(requestparameters, '$.key') as deleted_object,
       useridentity.sessioncontext.attributes.mfaauthenticated as mfa_used
FROM cloudtrail_s3_events
WHERE account = '123456789012' AND region = 'us-east-1'
  AND timestamp BETWEEN '2026/05/15' AND '2026/05/17'
  AND eventname IN ('DeleteObject', 'DeleteObjects')
ORDER BY eventtime DESC;

Best Practices & Troubleshooting

    Partition Pruning is Crucial: Always filter on partition columns (account, region, timestamp) first. Using eventtime alone leads to scanning the entire table, increasing cost.
    Exclude HeadObject: If you experience high costs from metadata operations, configure CloudTrail event selectors to exclude HeadObject API calls.
    Integrity and Security: Enable log file validation to ensure logs are tamper-proof. For maximum security, isolate logs in a dedicated AWS log-archive account.
    HIVE_PARTITION_SCHEMA_MISMATCH: If this error occurs, verify that the storage.location.template matches your actual S3 path structure.

Conclusion

Centralizing S3 data events via CloudTrail provides the vital identity context needed for security compliance. When combined with Athena partition projection, you can cost-effectively run security investigations across millions of actions.

Reference Link: https://www.facebook.com/share/p/1UDZGdgAx6/?
markdown

---
title: "Blog 3 - Amazon S3 Audit Logging – Part 2: Centralized Logging and Analysis"
date: 2024-01-03
weight: 3
chapter: false
---

### CENTRALIZED LOGGING AND ANALYSIS OF S3 DATA EVENTS IN AWS CLOUDTRAIL

When a security incident occurs—such as unauthorized access or a bulk deletion in an S3 bucket—the first question is always: "Who did this?" Traditional S3 server access logs show what request happened, but lack the full IAM identity context to identify the specific user or assumed role.

This post (Part 2 of our S3 audit series) explores how to implement an identity-driven audit system using AWS CloudTrail data events and Amazon Athena.

```mermaid
graph LR subgraph Lambda Function AppSyncEventsResolver -->|"Match /default/chat"| ChatHandler["Chat Handler"] AppSyncEventsResolver -->|"Match /notifications/*"| NotificationHandler["Notification Handler"] end ChatHandler --> ReturnPayload["Return Chat Payload"] NotificationHandler --> ReturnPayload ReturnPayload --> AppSync["AWS AppSync Events"] AppSync -->|"WebSocket Event"| Lambda["AWS Lambda"] 
```
### 1. Pattern-based Routing
Instead of writing complex conditional blocks (such as if-else or switch-case) to manually route events depending on their destination channel, the resolver routes events automatically based on the namespace and channel path pattern.
### 2. Wildcard Support

You can register catch-all handlers using wildcards (e.g., /default/*). The resolver automatically matches child channels and forwards incoming events to the registered handler.
### 3. Automatic Response Formatting

AWS AppSync Events expects responses from Lambda functions to adhere to specific structures for connection authorization and event acknowledgement. AppSyncEventsResolver automatically generates this response envelope behind the scenes, allowing you to simply return your payload.
### 4. Batch Processing & Error Handling
When processing multiple messages in a single batch request, the resolver supports sequential or parallel execution. It also provides granular, built-in error handling at the individual message level, ensuring successful messages are acknowledged even if one message in the batch fails.

---

### Installation & Setup

AppSyncEventsResolver is supported across the major Powertools for AWS Lambda languages: Python, TypeScript, and .NET.

### Package Installation:

- TypeScript / Node.js:
```TypeScript
npm install @aws-lambda-powertools/event-handler
```
- Python:
```Python
npm install @aws-lambda-powertools/event-handler

```

---

### Implementation Code Examples

### 1. TypeScript Implementation
Here is how to set up publish event handlers in TypeScript using the resolver:

```TypeScript
graph TD
import { AppSyncEventsResolver } from '@aws-lambda-powertools/event-handler/appsync-events';

const app = new AppSyncEventsResolver();


app.onPublish('/default/chat', async (payload) => {
    console.log('Received chat payload:', payload);
    return payload; 
});


app.onPublish('/notifications/*', async (payload) => {
    console.log('Received system notification:', payload);
    return { status: 'notification_sent' };
});

export const handler = async (event: any, context: any) => {
    return app.resolve(event, context);
};

```

### 2. Python Implementation

In Python, you can utilize decorators to cleanly register handlers for different channels and actions:
```python
from aws_lambda_powertools.event_handler import AppSyncEventsResolver

app = AppSyncEventsResolver()


@app.on_publish(path="/default/chat")
def handle_chat(payload):
    print(f"Received chat payload: {payload}")
    return {"status": "success", "message": payload}


@app.on_subscribe(path="/default/chat")
def handle_subscribe(payload):
    print(f"User subscribed to chat channel: {payload}")
    return {"authorized": True}

def lambda_handler(event, context):
    return app.resolve(event, context)

```

---

### Conclusion

Building large-scale, real-time serverless systems has never been easier. By combining the high scalability of AWS AppSync Events with the routing and standardizing capabilities of Powertools for AWS Lambda, developers can deploy clean, secure, and highly maintainable WebSocket APIs with minimal effort.

---
**Reference Link:** [https://www.facebook.com/share/p/1BS1nG91UA/](https://www.facebook.com/share/p/1BS1nG91UA/)