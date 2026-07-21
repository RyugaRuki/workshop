---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AI-Powered Data Extraction & Summarization Platform
## Dự án nhóm: Giải pháp tự động hoá thu thập và tổng hợp dữ liệu sử dụng AWS & Amazon Bedrock

### 1. Tóm tắt điều hành  
Dự án "AI-Powered Data Extraction & Summarization Platform" được thiết kế và phát triển bởi nhóm chúng em nhằm giải quyết bài toán tự động thu thập, trích xuất và tóm tắt khối lượng lớn dữ liệu từ các nguồn API bên ngoài. Thay vì xử lý thủ công tốn thời gian, nền tảng tận dụng sức mạnh của Amazon Bedrock (Generative AI) kết hợp với kiến trúc AWS có khả năng mở rộng cao (EC2 Auto Scaling, SQS, EventBridge) để xử lý dữ liệu định kỳ một cách hoàn toàn tự động, an toàn và tối ưu chi phí.

### 2. Tuyên bố vấn đề  
*Vấn đề hiện tại*  
Hiện tại, việc thu thập thông tin từ các nguồn dữ liệu bên ngoài (External APIs) và tóm tắt nội dung đang được thực hiện thủ công hoặc thông qua các kịch bản (script) rời rạc. Điều này dẫn đến sự thiếu ổn định, khó mở rộng khi lượng dữ liệu tăng lên, và tiêu tốn nhiều thời gian của nhân sự để đọc và phân tích dữ liệu.

*Giải pháp của nhóm*  
Nhóm chúng tôi đề xuất một hệ thống tự động hóa hoàn toàn trên AWS. Giao diện người dùng được phân phối qua CloudFront. Backend xử lý dữ liệu chạy trên các máy chủ EC2 (nằm trong Private Subnet để bảo mật) được quản lý bởi Auto Scaling Group. Quá trình lấy dữ liệu và xử lý được lập lịch tự động mỗi 8 giờ bởi EventBridge và SQS. Sức mạnh cốt lõi của giải pháp là việc tích hợp Amazon Bedrock để AI tự động đọc hiểu, làm sạch và tóm tắt dữ liệu. Kết quả được lưu trữ bền vững trên S3 và DynamoDB.

*Lợi ích và hoàn vốn đầu tư (ROI)*  
Giải pháp giúp tự động hóa 100% quy trình thu thập và báo cáo dữ liệu định kỳ, tiết kiệm hàng chục giờ làm việc mỗi tuần cho đội ngũ phân tích. Bằng việc sử dụng Auto Scaling và giới hạn tài nguyên, hệ thống tối ưu được chi phí vận hành do chỉ cấp phát tài nguyên khi thực sự có tác vụ xử lý dữ liệu.

### 3. Kiến trúc giải pháp và Luồng chạy (Execution Flow)
Kiến trúc của dự án được nhóm thiết kế tuân thủ các tiêu chuẩn bảo mật (Security) và độ sẵn sàng cao (High Availability) trên AWS. Đảm bảo luồng dữ liệu được xử lý khép kín và an toàn.

![Architecture Diagram](/images/2-Proposal/edge_architecture.jpeg)


*Luồng chạy chi tiết của dự án (Execution Flow):*
1. **Giao tiếp người dùng (Frontend)**: Người dùng truy cập ứng dụng thông qua **CloudFront** (được bảo vệ bởi **AWS WAF**). CloudFront phân phối nội dung tĩnh từ **S3 bucket (frontend, private, versioned)** một cách an toàn thông qua tính năng OAC (Origin Access Control) cùng các behavior đã thiết lập.
2. **Kích hoạt tự động hóa**: **EventBridge Trigger** được cấu hình để kích hoạt mỗi 8 giờ, gửi thông điệp vào hàng đợi **SQS** để bắt đầu quy trình trích xuất dữ liệu.
3. **Compute Network & Load Balancing**: Hệ thống sử dụng **Application Load Balancer** (ALB) để cân bằng các yêu cầu API giữa cụm máy chủ EC2. Các máy chủ này được quản lý bởi **Auto Scaling Group** và được phân bổ an toàn trong các **Private Subnets(A và B)** (không có Public IP) để đảm bảo tính bảo mật. Đồng thời, các EC2 Worker liên tục kéo công việc từ hàng chờ thông qua **SQS VPC Endpoint**.
4. **Thu thập dữ liệu (Outbound Traffic)**: Để lấy dữ liệu từ **External API**, các EC2 Worker kết nối ra internet thông qua **NAT Gateway** (đặt ở Public Subnet) và đi qua **Internet Gateway**.
5. **Tích hợp Generative AI**: Sau khi lấy được dữ liệu, EC2 Worker đẩy nội dung sang **Amazon Bedrock** thông qua **Bedrock VPC Endpoint** để mô hình AI xử lý ngôn ngữ thực hiện việc trích xuất thông tin và tóm tắt.
6. **Lưu trữ kết quả an toàn**: 
   - Dữ liệu văn bản sau khi làm sạch (Cleaned content) được Worker đẩy trực tiếp vào **S3 bucket** thông qua **S3 Gateway Endpoint**.
   - Các siêu dữ liệu (metadata), thông tin có cấu trúc được lưu vào **DynamoDB** thông qua **DynamoDB VPC Endpoint**. Cơ sở dữ liệu này được cấu hình sao lưu (Backup) tự động.
   - Các thông điệp trạng thái hoặc job tóm tắt nâng cao có thể được đẩy vào hàng đợi **SQS Summarize** cũng qua SQS Endpoint.
7. **Giám sát và Vận hành (Monitoring & Alerting)**:
   - Các EC2 Worker được quản trị thông qua **Systems Manager (Session Manager - SSM)**, không cần mở cổng SSH.
   - Toàn bộ logs và metrics (chỉ số) được thu thập bởi **CloudWatch**.
   - Nếu hệ thống phát hiện bất thường vượt ngưỡng (CloudWatch Alarm), thông báo sẽ được gửi qua **SNS Topic** đến **Email của Admin**.
8. **Bảo mật và Chi phí**:
   - Sử dụng **Security Group** và **IAM Role** cho ASG với nguyên tắc đặc quyền tối thiểu (least scoped).
   - Thiết lập **AWS Budgets** để tự động gửi email cảnh báo khi chi phí hệ thống vượt mức kiểm soát.

### 4. Triển khai kỹ thuật  
*Phân công và các giai đoạn thực hiện của nhóm:*
- **Giai đoạn 1 (Thiết kế & Khởi tạo)**: Nhóm xây dựng mạng VPC nền tảng với đầy đủ Public/Private Subnets, NAT Gateway. Thiết lập S3 Frontend bảo mật qua CloudFront và WAF.
- **Giai đoạn 2 (Tích hợp AI & Lập lịch)**: Cài đặt cấu hình Auto Scaling Group cho EC2 Worker. Xây dựng lịch trình EventBridge và SQS. Phát triển mã nguồn cho EC2 Worker gọi External API và Amazon Bedrock.
- **Giai đoạn 3 (Lưu trữ nội bộ)**: Thiết lập các VPC Endpoints (S3, DynamoDB, SQS, Bedrock) để đảm bảo traffic không đi qua mạng internet công cộng. Tạo các bảng DynamoDB và S3 bucket tương ứng.
- **Giai đoạn 4 (Giám sát & Hoàn thiện)**: Tích hợp CloudWatch, SNS, Systems Manager. Rà soát, kiểm thử các IAM Roles, Security Groups để nghiệm thu dự án.

### 5. Lộ trình & Mốc triển khai  
- **Tuần 10**:
  - *Thiết kế kiến trúc & Nền tảng*: Thống nhất sơ đồ kiến trúc tổng thể, thiết lập mạng VPC (Public/Private Subnets, NAT Gateway, Internet Gateway) và hạ tầng bảo mật cơ bản (Security Groups, IAM Roles).
  - *Phát triển Core Logic*: Viết mã nguồn (Node.js/Python) cho Worker thu thập dữ liệu (Collector Worker) và cấu trúc bảng DynamoDB để lưu trữ metadata. Tích hợp thử nghiệm với Amazon Bedrock thông qua SDK.
- **Tuần 11**: 
  - *Tự động hóa & Tích hợp*: Cấu hình EC2 Auto Scaling Group kết hợp với Launch Templates. Thiết lập EventBridge để tự động kích hoạt tiến trình theo lịch (cron job) và sử dụng SQS để làm hàng đợi (message queue) chịu lỗi giữa các worker.
  - *Bảo mật mạng nội bộ*: Thiết lập và kiểm thử luồng traffic qua các VPC Endpoints (S3, DynamoDB, SQS, Bedrock) để đảm bảo không dữ liệu nào truyền ra ngoài internet công cộng.
- **Tuần 12**:
  - *Frontend & Phân phối nội dung*: Deploy giao diện người dùng tĩnh lên Amazon S3 và cấu hình CloudFront (OAC) cùng AWS WAF để bảo vệ và tăng tốc độ tải trang.
  - *Giám sát & Hoàn thiện*: Cấu hình CloudWatch Dashboard để theo dõi hiệu suất hệ thống, thiết lập SNS để gửi email cảnh báo (alerts) khi có sự cố. Tiến hành kiểm thử bảo mật (penetration testing nội bộ) và chuẩn bị slide trình bày kết quả cuối kỳ của dự án nhóm.

### 6. Ước tính ngân sách (Budget Estimation)
Bạn có thể xem chi tiết bảng tính trên [AWS Pricing Calculator](#).

*Chi phí hạ tầng (AWS Services)*
- **Amazon EC2 (t4g.small)**: ~6,00 USD/tháng (chạy Auto Scaling, ước tính thời gian hoạt động bán thời gian).
- **NAT Gateway & Truyền tải dữ liệu**: ~15,00 USD/tháng (dành cho việc Worker gọi External API).
- **Amazon Bedrock (Nova Lite)**: ~2,50 USD/tháng (ước lượng số lượng token để tóm tắt văn bản).
- **Amazon S3 Standard**: ~0,15 USD/tháng (lưu trữ frontend tĩnh và file nội dung).
- **Amazon DynamoDB**: 0,00 USD/tháng (nằm trong giới hạn Free Tier).
- **Amazon SQS & EventBridge**: 0,00 USD/tháng (nằm trong giới hạn Free Tier).
- **Amazon CloudFront & WAF**: ~5,00 USD/tháng (phí duy trì WAF WebACL, CloudFront nằm trong Free Tier).

**Tổng chi phí ước tính**: ~28,65 USD/tháng

### 7. Đánh giá rủi ro và Kế hoạch dự phòng 
- **Lỗi kết nối hoặc thay đổi cấu trúc từ External API**: Các API bên ngoài có thể bị lỗi giới hạn tỷ lệ (Rate Limit) hoặc thay đổi cấu trúc JSON đột ngột. 
  - *Dự phòng*: Xây dựng cơ chế *Exponential Backoff & Retry* trong code của Worker. Sử dụng *Dead-Letter Queue (DLQ)* trong SQS để lưu trữ lại các request thất bại, cho phép xử lý lại sau khi kỹ sư khắc phục lỗi cấu trúc API.
- **Chi phí phát sinh từ Amazon Bedrock & NAT Gateway**: Do Bedrock tính phí theo số lượng token và NAT Gateway tính phí theo giờ/lưu lượng, việc Worker rơi vào vòng lặp vô tận (infinite loop) có thể gây thiệt hại lớn.
  - *Dự phòng*: Đặt các mức trần nghiêm ngặt trong *AWS Budgets* để cảnh báo nhóm qua email/SMS ngay lập tức nếu chi phí đạt 50%, 80%, và 100% giới hạn ngân sách (ví dụ: $30/tháng).
- **Rủi ro bảo mật máy chủ & rò rỉ dữ liệu**: Hacker có thể dò quét và tấn công các máy chủ EC2 nếu mở port ra internet.
  - *Dự phòng*: Tuyệt đối không cấp Public IP cho EC2 (đặt trong Private Subnet). Chỉ truy cập và quản trị máy chủ thông qua *AWS Systems Manager (Session Manager)*. Sử dụng IAM Roles với quyền hạn tối thiểu (Least Privilege).
- **Mất mát dữ liệu trong quá trình xử lý**: Lỗi phần cứng hoặc phần mềm khiến EC2 Worker bị sập khi đang phân tích bài viết.
  - *Dự phòng*: SQS có cơ chế khóa thông điệp (Visibility Timeout). Nếu EC2 bị sập trước khi xóa thông điệp, thông điệp sẽ hiển thị lại trên hàng đợi để một EC2 Worker khác (do Auto Scaling tạo ra) tiếp tục xử lý, đảm bảo tính toàn vẹn dữ liệu.

### 8. Kết quả kỳ vọng  
Dự án kiến trúc AWS của nhóm kỳ vọng sẽ đạt được các mục tiêu mang tính toàn diện về cả mặt công nghệ và hiệu quả vận hành:
- **Tự động hóa hoàn toàn luồng công việc**: Chuyển đổi quy trình thu thập và tóm tắt thông tin thủ công thành một hệ thống tự động hóa 100%. Tiết kiệm tới 80% thời gian xử lý dữ liệu hàng ngày cho các đội ngũ phân tích nghiệp vụ.
- **Khả năng mở rộng không giới hạn (High Scalability)**: Thông qua việc tích hợp EC2 Auto Scaling và SQS, hệ thống có khả năng tự động tăng số lượng worker khi lượng tin tức cần xử lý tăng đột biến, và tự động thu hẹp khi không có công việc để tiết kiệm tối đa chi phí.
- **Áp dụng AI tạo sinh (GenAI) vào thực tế**: Khai thác thành công sức mạnh của Amazon Bedrock (Nova Lite/Claude) để đọc hiểu và cô đọng hàng ngàn trang tài liệu thành những tóm tắt ngắn gọn, chính xác với độ trễ thấp.
- **Đạt tiêu chuẩn bảo mật doanh nghiệp (Enterprise-Grade Security)**: Cấu trúc mạng VPC được thiết kế chuẩn mực với Private Subnets, VPC Endpoints, mã hóa dữ liệu trên S3/DynamoDB và bảo vệ Frontend bằng CloudFront + WAF. Giải pháp này không chỉ phục vụ cho một bài tập nhóm mà hoàn toàn có thể được ứng dụng như một sản phẩm thương mại (SaaS) có độ tin cậy cao trên môi trường thực tế.