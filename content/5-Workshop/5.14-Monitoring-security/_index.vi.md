---
title: "Giám sát và bảo mật"
date: 2026-08-08
weight: 14
chapter: false
pre: " <b> 5.14. </b> "
---

# Giám sát và bảo mật

## Mục tiêu

Một hệ thống chạy được chưa có nghĩa là đã sẵn sàng để vận hành. Phần này trình bày những lớp bảo vệ và khả năng quan sát cần thiết để CampusMeet có thể phát hiện lỗi, giới hạn quyền truy cập và giảm rủi ro khi làm việc với dữ liệu cuộc họp, OAuth và AI.

## 1. CloudWatch Logs

Các Lambda chính cần có log group riêng để dễ tìm lỗi theo thành phần, ví dụ API, AI Worker, Google Sync Worker và Reminder Worker.

Log nên đủ để trả lời các câu hỏi:

- Request nào thất bại?
- Thành phần nào xử lý request đó?
- Resource ID nào liên quan?
- Lỗi thuộc nhóm validation, authorization, dependency hay internal error?
- Mất bao lâu để xử lý?

Log không nên chứa toàn bộ dữ liệu người dùng chỉ để tiện debug.

## 2. Dữ liệu không được ghi vào log

Không ghi trực tiếp:

- JWT.
- Password hoặc confirmation code.
- OAuth authorization code.
- Google access token/refresh token.
- Presigned S3 URL.
- Invitation token gốc.
- Toàn bộ tài liệu, transcript hoặc nội dung AI nhạy cảm.

Thay vào đó dùng metadata an toàn như:

```text
requestId
meetingId
groupId
aiJobId
syncRevision
status
errorCode
latency
```

## 3. CloudWatch metrics và alarms

Application infrastructure hiện có các thành phần quan sát lỗi cho API và worker. Mục tiêu của alarm không phải tạo thật nhiều cảnh báo, mà là bắt những lỗi người dùng không thể tự nhìn thấy ngay.

Các nhóm cần theo dõi:

- Lỗi Lambda API tăng bất thường.
- AI Worker lỗi hoặc chạy gần timeout.
- Google synchronization thất bại sau retry.
- Các job bất đồng bộ thất bại liên tục.

Alarm có thể gửi đến SNS topic để người vận hành nhận thông báo.

## 4. IAM theo nguyên tắc quyền tối thiểu

Mỗi Lambda dùng execution role riêng. Không dùng chung một role có `AdministratorAccess` cho toàn hệ thống.

Một role chỉ nên có:

- Quyền đọc/ghi đúng bảng cần thiết.
- Quyền gọi đúng AWS service cần thiết.
- Quyền trên đúng ARN hoặc prefix nếu có thể giới hạn.
- Quyền ghi log của chính function.

Ví dụ AI Worker cần Bedrock và bảng AI liên quan, trong khi Reminder Worker không cần quyền gọi Bedrock.

## 5. Secrets Manager

Google OAuth client secret là dữ liệu phía máy chủ và phải được lưu trong AWS Secrets Manager hoặc cơ chế bí mật tương đương.

Không đưa client secret vào:

```text
apps/web/.env
VITE_*
Git repository
CloudWatch logs
issue/PR
```

Frontend chỉ cần các cấu hình public như Cognito IDs, API URL và Google Cloud project number.

## 6. S3 private

Bucket chứa tài liệu người dùng không public.

Browser upload/download thông qua URL có chữ ký tạm thời sau khi backend đã kiểm tra quyền. CORS của bucket production cần giới hạn về đúng frontend origin thay vì để wildcard lâu dài.

CloudFront/frontend bucket và user-content bucket có mục đích khác nhau, không nên dùng chung chính sách public access.

## 7. Xác thực và phân quyền là hai lớp khác nhau

Amazon Cognito và API Gateway xác nhận người gọi là ai. Lambda tiếp tục quyết định người đó được làm gì.

Ví dụ:

```text
JWT hợp lệ
   ≠
được đọc mọi group
```

Backend vẫn kiểm tra:

- membership hiện tại;
- role;
- group/resource boundary;
- organizer/assignee khi nghiệp vụ yêu cầu.

## 8. Idempotency

Các thao tác có nguy cơ retry phải dùng idempotency khi hợp đồng yêu cầu, đặc biệt:

- tạo Group;
- tạo Meeting;
- tạo AIJob;
- các thao tác chuyển đổi có thể bị double-submit.

Idempotency giúp một request gửi lại do mạng chậm không tạo dữ liệu trùng ngoài ý muốn.

## 9. Optimistic concurrency

Minutes, Meeting và các entity có version cần conditional write để tránh lost update.

Thay vì "last write wins" một cách âm thầm, backend trả conflict khi client dùng version cũ. Frontend sau đó có thể hiển thị thông báo và giữ draft của người dùng.

## 10. Retry và stale work

Các worker bất đồng bộ cần phân biệt retry hợp lệ và công việc đã lỗi thời.

Ví dụ Google sync:

```text
revision 7 đang retry
meeting đã lên revision 8
        ↓
revision 7 không được ghi đè kết quả revision 8
```

Tương tự với ingestion hoặc các job khác, trạng thái hiện tại phải được kiểm tra trước khi ghi kết quả cuối.

## 11. CORS

Trong môi trường local có thể cho phép:

```text
http://localhost:5173
```

Trong production, API và S3 user-content nên giới hạn về CloudFront origin thực tế.

Không coi `AllowOrigins: ['*']` là cấu hình production cuối cùng cho luồng có dữ liệu người dùng chỉ vì nó làm trình duyệt hết lỗi CORS.

## 12. Bảo vệ dữ liệu production

Data stack hỗ trợ các tùy chọn bảo vệ như:

- Point-in-time recovery.
- Deletion protection.
- `DeletionPolicy: Retain`.
- `UpdateReplacePolicy: Retain`.

Trong production nên bật các tùy chọn phù hợp trước khi nạp dữ liệu quan trọng.

## 13. Kiểm tra trước release

Chạy:

```powershell
npm run infra:validate
npm run lint
npm run typecheck
npm run test
npm run build
npm run format:check
```

Ngoài ra kiểm tra dependency runtime:

```powershell
npm audit --omit=dev
```

Không chạy `npm audit fix --force` ngay trước release mà không review vì có thể nâng major version và làm thay đổi hành vi ứng dụng.

Nếu có HIGH/CRITICAL, cần xác định nó nằm ở runtime dependency hay chỉ trong tooling, sau đó quyết định cách xử lý dựa trên package cụ thể.

## 14. Kiểm tra sau deploy

Sau khi deploy:

1. `/health` trả 200.
2. API protected không token trả 401.
3. User ngoài group bị từ chối.
4. Log không làm lộ token hoặc secret.
5. Worker lỗi xuất hiện trong CloudWatch.
6. Alarm/metric quan trọng tồn tại theo template.
7. S3 user content không public.
8. Secret Google không xuất hiện trong frontend bundle.

## Kết quả cần đạt

- Mỗi thành phần có log đủ để chẩn đoán nhưng không làm lộ dữ liệu nhạy cảm.
- IAM được tách theo execution role và chức năng.
- Secret ở phía server, không nằm trong bundle web.
- Authentication và domain authorization được kiểm tra độc lập.
- Idempotency, version và stale-work protection được dùng ở các luồng có nguy cơ race/retry.
- Production CORS và data protection được cấu hình chặt hơn môi trường local.
