---
title: "Bài viết 2 - Đơn giản hóa tích hợp AWS AppSync Events với Powertools for AWS Lambda"
date: 2024-01-02
weight: 2
chapter: false
---

### ĐƠN GIẢN HÓA TÍCH HỢP AWS APPSYNC EVENTS VỚI POWERTOOLS FOR AWS LAMBDA

Khả năng thời gian thực đã trở thành một yêu cầu thiết yếu trong các ứng dụng hiện đại, nơi người dùng mong đợi các cập nhật tức thì và trải nghiệm tương tác. Dù bạn đang xây dựng ứng dụng chat, dashboard trực tiếp, bảng xếp hạng trò chơi hay hệ thống IoT, AWS AppSync Events đều hỗ trợ các tính năng thời gian thực này thông qua WebSocket API, cho phép bạn xây dựng ứng dụng thời gian thực có khả năng mở rộng và hiệu năng cao mà không phải lo quản lý kết nối hay mở rộng hạ tầng.

Tuy nhiên, việc tích hợp AWS AppSync Events vào kiến trúc serverless, đặc biệt là xử lý sự kiện trong AWS Lambda, thường phát sinh các thách thức liên quan đến tuần tự hóa payload, định tuyến sự kiện và xử lý lỗi theo lô. Để đơn giản hóa quy trình này, bộ công cụ Powertools for AWS Lambda đã giới thiệu tiện ích AppSyncEventsResolver, loại bỏ nhu cầu viết mã boilerplate và giúp luồng phát triển gọn hơn.

### Vai trò của Powertools for AWS Lambda

Powertools for AWS Lambda là bộ công cụ dành cho lập trình viên nhằm tối ưu hóa việc phát triển ứng dụng serverless. Bộ công cụ này cung cấp các tiện ích quan trọng cho Logging, Tracing, Metrics và Event Handling.

Với việc bổ sung AppSyncEventsResolver, Powertools giờ đây mang lại một giao diện sạch và nhất quán để xử lý AppSync Events trong các hàm Lambda. Công cụ này đảm nhận những phần việc lặp lại nhưng bắt buộc như phân tích sự kiện, định tuyến tới đúng handler theo pattern và định dạng phản hồi phù hợp với yêu cầu của AppSync, giúp lập trình viên tập trung hoàn toàn vào business logic cốt lõi.

### Các tính năng cốt lõi của AppSyncEventsResolver

AppSyncEventsResolver được xây dựng để giải quyết các pattern xử lý sự kiện phổ biến trong ứng dụng thời gian thực. Sơ đồ dưới đây minh họa cách các sự kiện WebSocket từ AWS AppSync được định tuyến tới các handler khác nhau trong hàm Lambda:
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
Blog 3: Amazon S3 Audit Logging – Phần 2: Ghi log và phân tích tập trung
Ghi log và phân tích tập trung các Data Event của S3 trong AWS CloudTrail

Khi xảy ra một sự cố bảo mật, chẳng hạn truy cập trái phép hoặc xóa hàng loạt đối tượng trong một S3 bucket, câu hỏi đầu tiên luôn là: "Ai đã làm điều này?" Các S3 server access log truyền thống cho biết điều gì đã xảy ra, nhưng thiếu ngữ cảnh danh tính IAM đầy đủ để xác định chính xác người dùng hoặc assumed role đã thực hiện hành động đó.

Bài viết này, phần 2 trong chuỗi về S3 audit, trình bày cách triển khai một hệ thống audit dựa trên danh tính người dùng bằng AWS CloudTrail data events và Amazon Athena.
graph TD User[IAM User / Assumed Role] -- S3 API Call --> S3[Amazon S3] S3 -- Data Event Logs --> CloudTrail[AWS CloudTrail] CloudTrail -- Deliver .json.gz --> CentralS3[Central S3 Bucket] CentralS3 -- Query via Partition Projection --> Athena[Amazon Athena] Athena -- Query Results --> Analyst[Security Investigator]
S3 Data Events so với Server Access Logs

CloudTrail data events cung cấp khả năng theo dõi chi tiết ở cấp API đối với các thao tác trên đối tượng S3 cùng thông tin danh tính, bổ sung cho các access log ở cấp HTTP.

		Nội dung được ghi nhận: Danh tính chi tiết của IAM user/role, các API operation như GetObject, PutObject, DeleteObject, ngữ cảnh xác thực như MFA và chuỗi assumed role, cùng thông tin truy cập liên tài khoản.
		Nội dung không được ghi nhận: Các chỉ số hiệu năng ở cấp HTTP, status code hoặc thời gian phản hồi chính xác, những thông tin này cần S3 server access logs như đã đề cập ở phần 1.
		Cách phân phối và chi phí: Log được phân phối dưới dạng JSON nén trong khoảng 5 phút. Chi phí là 0,10 USD cho mỗi 100.000 data event được ghi. Lưu ý rằng data event không có free tier.

### Cấu hình từng bước

Để thiết lập ghi log audit tập trung cho S3 trên phạm vi toàn tổ chức:

		Tạo CloudTrail Trail: Trong CloudTrail console, tạo một trail mới, ví dụ `s3-data-events-trail`, và trỏ tới một S3 bucket tập trung.
		Bật ghi log trên toàn tổ chức: Nếu sử dụng AWS Organizations, hãy chọn Enable for all accounts in my organization tại management account để tự động triển khai trail tới toàn bộ member account.
		Cấu hình Advanced Event Selectors: Bỏ chọn management events để tránh phát sinh chi phí ghi log trùng lặp nếu đã có trail khác, rồi chọn Data Events với S3 là loại tài nguyên. Bạn có thể lọc theo bucket hoặc prefix.
		Cấu hình S3 Lifecycle Policy: Trên bucket trung tâm, thiết lập lifecycle rule để chuyển log sang Standard-IA sau 90 ngày, Glacier sau 180 ngày và xóa sau 7 năm để tối ưu chi phí lưu trữ.

### Tạo bảng Athena với Partition Projection

Partition projection tính toán giá trị partition một cách động từ đường dẫn S3, giúp tăng tốc truy vấn và giảm chi phí quét metadata. Sử dụng truy vấn dưới đây để tạo bảng:

```sql
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
```

Lưu ý: Với trail trên toàn tổ chức, hãy thêm `organization_id STRING` làm cột partition đầu tiên và đưa nó vào `storage.location.template`.

### Các mẫu truy vấn bảo mật quan trọng

Athena partition projection yêu cầu chỉ định `account`, `region` và `timestamp` trong mệnh đề `WHERE` của mọi truy vấn để cắt tỉa partition và tránh quét toàn bộ bảng với chi phí cao.
1. Theo dõi hoạt động của người dùng cụ thể

Xác định các hành động trên data plane của S3 được thực hiện bởi một IAM user hoặc role trong một ngày nhất định:

```sql
SELECT eventtime, useridentity.arn as user_arn, eventname, sourceipaddress,
			 JSON_EXTRACT_SCALAR(requestparameters, '$.bucketName') as bucket,
			 JSON_EXTRACT_SCALAR(requestparameters, '$.key') as object_key
FROM cloudtrail_s3_events
WHERE account = '123456789012' AND region = 'us-east-1' AND timestamp = '2026/05/15'
	AND (useridentity.username = 'specific-user' OR useridentity.arn LIKE '%specific-user%')
	AND eventname IN ('GetObject', 'PutObject', 'DeleteObject')
ORDER BY eventtime DESC;
```
2. Giám sát truy cập liên tài khoản

Đánh dấu các thao tác S3 phát sinh từ các AWS account bên ngoài:

```sql
SELECT eventtime, useridentity.accountid as source_account, recipientaccountid as target_account,
			 useridentity.arn as user_arn, eventname, sourceipaddress
FROM cloudtrail_s3_events
WHERE account = '123456789012' AND region = 'us-east-1' AND timestamp = '2026/05/15'
	AND useridentity.accountid != recipientaccountid
ORDER BY eventtime DESC;
```
3. Hiển thị các lần truy cập thất bại

Phát hiện các lỗi quyền như AccessDenied hoặc NoSuchKey, qua đó chỉ ra khả năng dò quét hoặc brute-force:

```sql
SELECT eventtime, useridentity.arn as user_arn, eventname, sourceipaddress, errorcode, errormessage
FROM cloudtrail_s3_events
WHERE account = '123456789012' AND region = 'us-east-1' AND timestamp = '2026/05/15'
	AND errorcode IS NOT NULL
ORDER BY eventtime DESC LIMIT 100;
```
4. Audit các thao tác xóa

Kiểm tra các thao tác xóa đối tượng trong một khoảng thời gian, bao gồm thông tin assumed role và trạng thái MFA:

```sql
SELECT eventtime, useridentity.arn as user_arn, sourceipaddress,
			 JSON_EXTRACT_SCALAR(requestparameters, '$.bucketName') as bucket,
			 JSON_EXTRACT_SCALAR(requestparameters, '$.key') as deleted_object,
			 useridentity.sessioncontext.attributes.mfaauthenticated as mfa_used
FROM cloudtrail_s3_events
WHERE account = '123456789012' AND region = 'us-east-1'
	AND timestamp BETWEEN '2026/05/15' AND '2026/05/17'
	AND eventname IN ('DeleteObject', 'DeleteObjects')
ORDER BY eventtime DESC;
```

### Best practices và xử lý sự cố

		Partition pruning là yếu tố then chốt: Luôn lọc trước theo các cột partition là `account`, `region`, `timestamp`. Nếu chỉ dùng `eventtime`, truy vấn có thể quét toàn bộ bảng và làm tăng chi phí.
		Loại trừ HeadObject: Nếu bạn thấy chi phí tăng cao do các thao tác metadata, hãy cấu hình CloudTrail event selectors để loại trừ API `HeadObject`.
		Tính toàn vẹn và bảo mật: Bật log file validation để đảm bảo log không bị chỉnh sửa. Để đạt mức an toàn cao nhất, nên tách log vào một AWS log-archive account chuyên dụng.
		HIVE_PARTITION_SCHEMA_MISMATCH: Nếu gặp lỗi này, hãy kiểm tra xem `storage.location.template` có khớp với cấu trúc đường dẫn S3 thực tế hay không.

### Kết luận

Việc tập trung hóa S3 data events thông qua CloudTrail mang lại ngữ cảnh danh tính rất quan trọng cho công tác tuân thủ và điều tra bảo mật. Khi kết hợp với Athena partition projection, bạn có thể thực hiện các cuộc điều tra bảo mật hiệu quả về chi phí trên hàng triệu hành động.

---

### Cài đặt và thiết lập

AppSyncEventsResolver được hỗ trợ trên các ngôn ngữ chính của Powertools for AWS Lambda: Python, TypeScript và .NET.

### Cài đặt package:

- TypeScript / Node.js:
```TypeScript
npm install @aws-lambda-powertools/event-handler
```
- Python:
```Python
npm install @aws-lambda-powertools/event-handler

```

---

### Ví dụ mã triển khai

### 1. Cài đặt bằng TypeScript
Dưới đây là cách thiết lập các handler publish event trong TypeScript bằng resolver:

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

### 2. Cài đặt bằng Python

Trong Python, bạn có thể sử dụng decorator để đăng ký handler một cách gọn gàng cho các channel và action khác nhau:
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

### Kết luận

Việc xây dựng các hệ thống serverless thời gian thực quy mô lớn chưa bao giờ dễ dàng hơn. Bằng cách kết hợp khả năng mở rộng cao của AWS AppSync Events với năng lực định tuyến và chuẩn hóa của Powertools for AWS Lambda, lập trình viên có thể triển khai các WebSocket API sạch, an toàn và dễ bảo trì với rất ít công sức.

---
**Reference Link:** [https://www.facebook.com/share/p/1BS1nG91UA/](https://www.facebook.com/share/p/1BS1nG91UA/)