---
title: "Kiến trúc hệ thống"
date: 2026-07-27
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Kiến trúc hệ thống CampusMeet

## Mục tiêu

Phần này mô tả cách các thành phần của CampusMeet phối hợp với nhau và quan trọng hơn là ranh giới giữa dữ liệu, API, xử lý bất đồng bộ và các tích hợp bên ngoài. Việc tách đúng ranh giới giúp giảm nguy cơ một thay đổi ở application stack làm ảnh hưởng trực tiếp đến dữ liệu lâu dài.

## 1. Nguyên tắc thiết kế

CampusMeet đi theo một số nguyên tắc xuyên suốt:

- Frontend không truy cập DynamoDB trực tiếp.
- Cognito xác thực danh tính, còn Lambda kiểm tra quyền nghiệp vụ.
- Dữ liệu nghiệp vụ nằm trong năm bảng DynamoDB dùng chung.
- File và nội dung lớn nằm trong S3.
- Các công việc dài hoặc phụ thuộc dịch vụ ngoài được xử lý bất đồng bộ.
- Google Calendar/Meet không phải nguồn dữ liệu chính của Meeting.
- AI chỉ được truy xuất nguồn mà người dùng có quyền đọc.
- Hạ tầng được quản lý bằng AWS SAM/CloudFormation thay vì sửa thủ công rồi bỏ qua source.

## 2. Luồng request chính

```text
Người dùng
   ↓
CampusMeet Web
   ↓
Amazon Cognito
   ↓ JWT
Amazon API Gateway
   ↓
AWS Lambda API
   ↓
Business service
   ↓
Repository / integration adapter
   ↓
DynamoDB / S3 / Google / Bedrock
```

Một request nghiệp vụ thường đi qua các bước:

1. Người dùng đăng nhập bằng Cognito.
2. Frontend gửi JWT qua `Authorization` header.
3. API Gateway kiểm tra token.
4. Lambda đọc danh tính từ JWT.
5. Business layer kiểm tra membership, role và resource boundary.
6. Repository truy cập DynamoDB hoặc adapter gọi dịch vụ ngoài.
7. Kết quả được trả về frontend theo contract dùng chung.

## 3. Bốn ranh giới hạ tầng hiện tại

CampusMeet hiện có bốn template chính, mỗi template có mục đích khác nhau.

| Template | Trách nhiệm |
| --- | --- |
| `infra/data-foundation.yaml` | Sở hữu năm bảng DynamoDB và stream của `meeting-data` |
| `infra/auth-integration.yaml` | Stack dev/core dùng để triển khai Cognito, HTTP API và các chức năng M1/M2 cốt lõi |
| `infra/user-content-orchestration.yaml` | S3 user content, Step Functions, reminder, Scheduler và các tài nguyên orchestration M4 |
| `infra/template.yaml` | Full application stack: frontend hosting, API, Cognito, Google sync, AI Worker, Bedrock, monitoring và các integration còn lại |

Hai template `auth-integration.yaml` và `template.yaml` không nên được hiểu là cùng một stack.

`auth-integration.yaml` hữu ích cho workshop các phần đầu vì nhỏ, dễ quan sát và đã được dùng cho môi trường dev core. Khi chuyển sang E2E đầy đủ với Minutes, Tasks, Upload, Google và AI, cần dùng application stack phù hợp thay vì giả định auth stack đã chứa toàn bộ chức năng.

## 4. Data foundation

`infra/data-foundation.yaml` tạo năm bảng:

```text
campusmeet-<env>-identity
campusmeet-<env>-collaboration
campusmeet-<env>-meeting-data
campusmeet-<env>-task-data
campusmeet-<env>-ai-work
```

Data stack được tách khỏi application stack để tránh việc cập nhật Lambda hoặc frontend vô tình thay thế bảng dữ liệu.

Bảng `meeting-data` có DynamoDB Stream để phát hiện thay đổi cần xử lý bất đồng bộ, điển hình là Google synchronization.

## 5. Auth/Core stack

`infra/auth-integration.yaml` cung cấp một stack nhỏ gồm:

- Cognito User Pool.
- User Pool Client.
- HTTP API.
- JWT authorizer.
- Lambda xử lý core route.
- IAM execution role.
- CloudWatch Log Group.

Stack này phù hợp để học và kiểm thử các phần Auth, Group, Invitation và Meeting CRUD ban đầu.

Nó không phải bằng chứng rằng Minutes, Tasks, Upload và AI đã được deploy nếu các route đó chỉ tồn tại trong full application handler.

## 6. User-content và orchestration stack

`infra/user-content-orchestration.yaml` xử lý những thành phần không nên nằm trực tiếp trong request API:

```text
S3 user-content
Step Functions AI orchestration
Reminder Lambda
EventBridge Scheduler role
SES configuration
```

Stack này nhận các ARN/tên tài nguyên liên quan từ stack khác qua parameter thay vì tự sở hữu mọi thứ.

Ví dụ upload document:

```text
API
 ↓
presigned URL
 ↓
S3 private bucket
 ↓
Attachment complete
 ↓
AIJob
 ↓
Step Functions
```

## 7. Full application stack

`infra/template.yaml` là stack tích hợp rộng hơn, chứa các thành phần như:

- Frontend S3 bucket.
- CloudFront distribution.
- Cognito cho application environment.
- HTTP API và API Lambda đầy đủ.
- Google Sync Worker.
- AI Worker.
- Bedrock Knowledge Base / vector integration.
- CloudWatch/SNS monitoring.

Các phần sau của workshop dùng stack này khi cần một E2E đầy đủ hơn.

## 8. Ranh giới xác thực và phân quyền

Hai lớp được tách riêng:

| Lớp | Nơi thực hiện | Ví dụ |
| --- | --- | --- |
| Authentication | Cognito + API Gateway | JWT hợp lệ, đúng issuer/audience |
| Authorization | Lambda/business service | member của group, role, resource scope |

Một người dùng có JWT hợp lệ nhưng không thuộc Group A vẫn phải bị từ chối khi truy cập dữ liệu Group A.

## 9. Ranh giới dữ liệu

Frontend không gửi `role` hoặc `userId` rồi yêu cầu backend tin giá trị đó.

Backend suy ra danh tính từ JWT và kiểm tra dữ liệu thật trong DynamoDB.

Các thao tác truy xuất ưu tiên `GetItem` và `Query` theo khóa đã thiết kế. `Scan` không phải lựa chọn mặc định cho request nghiệp vụ.

## 10. Tích hợp Google

Google synchronization chạy ngoài giao dịch chính của Meeting.

```text
Meeting mutation
      ↓
DynamoDB
      ↓
Stream
      ↓
GoogleSyncWorker
      ↓
Google Calendar API
```

Nếu Google lỗi, Meeting vẫn còn trong CampusMeet. Worker cập nhật trạng thái để retry hoặc yêu cầu người dùng kết nối lại khi cần.

## 11. Tích hợp AI

AI chạy theo hướng job bất đồng bộ:

```text
API
 ↓
AIJob
 ↓
Step Functions
 ↓
AI Worker
 ↓
Bedrock / Knowledge Base
```

Retrieval phải lọc nguồn theo group, meeting, trạng thái source và quyền của người dùng trước khi đưa dữ liệu vào model.

## 12. Những phần chưa thuộc core E2E hiện tại

Kiến trúc tổng thể có hướng mở rộng cho recording và Amazon Transcribe, nhưng workshop không xem live transcription, recording lifecycle hoặc batch audio transcription là phần đã hoàn tất chỉ vì tài liệu thiết kế hoặc contract đã tồn tại.

Những phần này được ghi rõ là mở rộng/chưa xác minh khi chưa có runtime E2E tương ứng.

## 13. Thứ tự triển khai hợp lý

Với môi trường đầy đủ:

```text
Xác nhận AWS account/region
        ↓
Data foundation
        ↓
User-content/orchestration
        ↓
Full application stack
        ↓
Lấy CloudFormation outputs
        ↓
Cấu hình frontend/Google
        ↓
Deploy frontend
        ↓
E2E test
```

Một số giá trị như CloudFront domain hoặc API URL chỉ có sau lần deploy đầu, vì vậy production run có thể cần update stack lần hai với origin/redirect URI chính xác.

## Kết quả cần đạt

Sau phần này, người học cần:

- Hiểu đường đi của request từ browser đến dữ liệu.
- Phân biệt authentication với authorization.
- Biết bốn template hiện tại sở hữu những tài nguyên nào.
- Không nhầm auth/core stack với full application stack.
- Hiểu vì sao Google và AI được tách khỏi request nghiệp vụ chính.
- Phân biệt kiến trúc dự kiến với chức năng đã được E2E verify.
