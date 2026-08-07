---
title: "Bản ghi nội dung cuộc họp và xử lý bất đồng bộ"
date: 2026-08-08
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

# Bản ghi nội dung cuộc họp và xử lý bất đồng bộ

## Mục tiêu

Phần này tập trung vào những công việc không nên xử lý hoàn toàn trong một HTTP request ngắn: tải tệp lớn, đồng bộ Google Calendar, nhắc lịch, điều phối AI và ingestion tài liệu.

Kiến trúc bất đồng bộ giúp CampusMeet:

- Không giữ request API chờ dịch vụ ngoài quá lâu.
- Có trạng thái để retry.
- Tránh gọi dịch vụ ngoài lặp lại khi cùng một thay đổi được xử lý nhiều lần.
- Tách lỗi tích hợp khỏi dữ liệu nghiệp vụ chính.

## 1. Tải tệp trực tiếp lên Amazon S3

Tệp nhị phân không đi qua API Gateway/Lambda để lưu vào DynamoDB.

Luồng:

```text
Browser
  ↓
yêu cầu upload URL
  ↓
API kiểm tra quyền + metadata
  ↓
presigned S3 URL
  ↓
Browser upload trực tiếp lên S3
  ↓
client gọi complete
  ↓
backend xác minh và lưu Attachment metadata
```

Lợi ích:

- Không phụ thuộc giới hạn payload của API.
- Lambda không phải nhận toàn bộ file.
- Bucket vẫn private.
- Presigned URL chỉ có hiệu lực trong thời gian giới hạn.

Không ghi presigned URL vào log dùng lâu dài vì URL này chứa thông tin ký tạm thời.

## 2. Attachment metadata

Sau upload, DynamoDB chỉ lưu metadata cần thiết như:

- Attachment ID.
- Meeting/group liên quan.
- S3 object key.
- Loại nội dung.
- Kích thước/trạng thái xác minh theo contract.

Không lưu binary file trong DynamoDB.

## 3. Document ingestion cho AI

Đối với tài liệu được dùng làm nguồn kiến thức, flow tổng quát:

```text
Attachment complete
       ↓
Tạo AIJob INGEST_SOURCE
       ↓
AWS Step Functions
       ↓
AI Worker
       ↓
Chuẩn hóa document
       ↓
S3 prefix dành cho Knowledge Base
       ↓
Bedrock ingestion
       ↓
KnowledgeSource READY khi ingestion hoàn tất
```

Bedrock ingestion là công việc bất đồng bộ. Việc gọi `StartIngestionJob` thành công chưa chứng minh tài liệu đã sẵn sàng để truy xuất.

Orchestration cần theo dõi trạng thái ingestion cho đến khi:

- hoàn thành;
- thất bại;
- hoặc đạt điều kiện dừng được thiết kế.

Trong repo hiện tại AI Worker có logic kiểm tra ingestion. Việc xác nhận vòng polling hoạt động trên AWS thật được thực hiện trong phần E2E, không được suy ra chỉ từ unit test.

## 4. Google Calendar synchronization

Google Calendar không nằm trong transaction DynamoDB chính.

Luồng hiện tại được thiết kế theo hướng:

```text
Meeting thay đổi
      ↓
Ghi dữ liệu Meeting + sync state
      ↓
DynamoDB Stream
      ↓
GoogleSyncWorker
      ↓
Google Calendar API
      ↓
SYNCED / FAILED / ACTION_REQUIRED
```

Worker phải phân biệt:

- Công việc mới.
- Công việc cũ/stale.
- Lỗi retry được như rate limit hoặc dịch vụ tạm thời không sẵn sàng.
- Lỗi cần người dùng kết nối lại Google.

Retry được lên lịch thay vì chặn request tạo Meeting.

## 5. Retry bằng EventBridge Scheduler

Đối với lỗi tạm thời, hệ thống có thể tạo lịch chạy lại tại một thời điểm trong tương lai.

Điều quan trọng:

- Retry phải có giới hạn.
- Mỗi lần retry cần biết revision/công việc mà nó đang xử lý.
- Một retry cũ không được ghi đè trạng thái của thay đổi mới hơn.
- Sau số lần retry tối đa, hệ thống chuyển về trạng thái lỗi cuối cùng để người vận hành hoặc người dùng xử lý.

## 6. Reminder

CampusMeet có luồng nhắc lịch sử dụng EventBridge Scheduler và Lambda.

```text
Meeting có reminder
      ↓
EventBridge Scheduler
      ↓
Reminder Lambda
      ↓
Tạo notification trong ứng dụng
      ↓
Gửi email bằng SES nếu cấu hình cho phép
```

Notification trong ứng dụng là dữ liệu chính. Nếu SES gửi email lỗi, notification đã được tạo không nên bị rollback.

Để gửi email thật cần:

- SES sender được xác minh.
- Nếu tài khoản SES còn sandbox, recipient thử nghiệm cũng phải đáp ứng hạn chế của sandbox.

## 7. Step Functions cho AIJob

Các job AI chạy bất đồng bộ thay vì giữ HTTP request chờ model.

```text
API request
  ↓
AIJob QUEUED
  ↓
StartExecution
  ↓
AI Worker
  ↓
COMPLETED / FAILED
```

Client dùng `aiJobId` để theo dõi trạng thái và lấy kết quả khi hoàn tất.

Cách này giúp tách timeout của HTTP API khỏi thời gian xử lý AI.

## 8. Idempotency và stale work

Hệ thống bất đồng bộ dễ gặp tình huống cùng một công việc được phát nhiều lần. Do đó cần:

- Idempotency key.
- Revision/version.
- Conditional write.
- So sánh trạng thái hiện tại trước khi gọi dịch vụ ngoài.

Ví dụ, một Google sync event cũ phải trở thành no-op nếu Meeting đã có revision mới hơn.

## 9. Quan sát lỗi

Các worker nên ghi log theo cấu trúc với những thông tin an toàn:

- `requestId` hoặc correlation ID.
- resource ID.
- attempt/revision.
- failure class.
- error code.
- latency.

Không log:

- access token/refresh token;
- full OAuth code;
- presigned URL;
- toàn bộ document;
- toàn bộ transcript;
- nội dung nhạy cảm không cần cho chẩn đoán.

## 10. Phạm vi chưa hoàn thiện trong E2E hiện tại

Workshop phân biệt rõ kiến trúc dự kiến và runtime đã hoàn thiện.

### Live transcription

Luồng realtime bằng Amazon Transcribe, heartbeat/reconnect và lưu final transcript chưa được xem là một phần của core production E2E hiện tại.

Trạng thái:

```text
Thiết kế/contract: có tài liệu
Production E2E hiện tại: chưa xác minh hoàn chỉnh
```

### Recording lifecycle

Thu âm đầy đủ với consent, recording state và recovery chưa phải luồng bắt buộc của bản E2E hiện tại.

### Batch audio transcription

UI/contract có thể nhận biết audio nhưng worker `BATCH_TRANSCRIPTION` hoàn chỉnh không được giả định tồn tại nếu chưa có implementation runtime tương ứng.

Vì vậy bản workshop hiện tại dùng **document upload** làm luồng ingestion chính để kiểm thử AI/RAG.

## 11. E2E nên kiểm tra

Core async:

1. Tạo Meeting.
2. Upload một file text nhỏ qua presigned URL.
3. Gọi complete.
4. Kiểm tra Attachment được lưu.
5. Nếu AI integration bật, kiểm tra AIJob được tạo.

Optional Google:

1. Connect Google.
2. Tạo Meeting.
3. Quan sát trạng thái từ `PENDING` sang `SYNCED`.
4. Xác nhận Calendar event thật.

Optional RAG:

1. Upload document.
2. Đợi source `READY`.
3. Hỏi câu có đáp án trong tài liệu.
4. Kiểm tra citation.

Actual result chỉ được ghi `PASS` sau khi thực hiện trên AWS/browser thật.

## Kết quả cần đạt

- File được upload trực tiếp vào S3 private bằng presigned URL.
- Metadata được quản lý riêng trong DynamoDB.
- Google, reminder và AI dùng async worker/orchestration thay vì kéo dài HTTP request.
- Retry có giới hạn và stale work được kiểm soát.
- Workshop không tuyên bố live STT/audio recording đã hoàn chỉnh khi runtime chưa được E2E verify.
