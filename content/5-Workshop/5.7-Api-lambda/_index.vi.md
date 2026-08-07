---
title: "API Gateway và AWS Lambda"
date: 2026-07-27
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# API Gateway và AWS Lambda

## Mục tiêu

Phần này giải thích cách CampusMeet nhận request HTTP, xác thực JWT và chuyển request đến các bộ xử lý nghiệp vụ. Workshop sử dụng `infra/auth-integration.yaml` để học core API ban đầu, nhưng cũng đối chiếu với `services/api/src/index.ts` để phản ánh các route của full application hiện tại.

## 1. Luồng xử lý request

```text
Frontend
  ↓
Authorization: Bearer <JWT>
  ↓
API Gateway HTTP API
  ↓
JWT Authorizer
  ↓
Lambda router/handler
  ↓
Business service
  ↓
Repository / integration adapter
```

Frontend không quyết định role. Lambda lấy identity từ JWT rồi kiểm tra membership và quyền bằng dữ liệu phía server.

## 2. Public và protected route

`GET /health` là public endpoint để kiểm tra service.

Các route nghiệp vụ yêu cầu JWT hợp lệ. Nếu không gửi token:

```text
GET /me
→ 401
```

CORS chỉ quyết định browser origin nào được phép gọi API; CORS không thay thế authentication hoặc authorization.

## 3. Core API trong auth stack

`infra/auth-integration.yaml` triển khai Lambda core với các nhóm route:

```text
/health
/me
/groups
/groups/:groupId
/groups/:groupId/invitations
/groups/:groupId/members/:userId
/groups/:groupId/meetings
/meetings
/meetings/:meetingId
/invitations
/notifications
```

Đây là stack phù hợp để học Cognito, Group, Invitation, Notification và Meeting CRUD trước khi chuyển sang full application.

## 4. Full application API

Handler đầy đủ trong `services/api/src/index.ts` mở rộng thêm các domain sau.

### Meeting và Google synchronization

```text
/groups/:groupId/meetings
/meetings
/meetings/:meetingId
/meetings/:meetingId/cancel
/meetings/:meetingId/google-sync/retry
```

### Attachment

```text
/meetings/:meetingId/attachments/*
/attachments/:attachmentId/download-url
```

### Minutes và Task

```text
/meetings/:meetingId/minutes
/meetings/:meetingId/minutes/action-items/:actionItemId/task
/tasks
/tasks/:taskId/status
/dashboard
```

### Google integration

```text
/integrations/google/connect
/integrations/google/callback
/integrations/google/meet-context
```

### AI

```text
/meetings/:meetingId/ai/chat
/groups/:groupId/ai/search
/meetings/:meetingId/ai/minutes-draft
/meetings/:meetingId/ai/task-proposals
/groups/:groupId/ai/progress-analysis
/ai/jobs/:aiJobId
```

Một route xuất hiện trong full application source không có nghĩa route đó đã được deploy trong `campusmeet-dev-auth`. Khi kiểm thử phải xác định chính xác stack và Lambda handler đang chạy.

## 5. Router và handler

Router chịu trách nhiệm:

- Ghép method + path.
- Parse path parameter.
- Chuyển request sang handler đúng domain.
- Trả `404` nếu không có route.

Handler không nên chứa toàn bộ nghiệp vụ và truy vấn DynamoDB trực tiếp nếu service/repository tương ứng đã tồn tại.

Cấu trúc mong muốn:

```text
API event
  ↓
Handler
  ↓
Application/Domain Service
  ↓
Repository / Port
  ↓
AWS adapter
```

## 6. Shared contracts

Frontend và backend dùng các type/DTO từ:

```text
@campusmeet/shared
```

Khi thay đổi API contract:

1. Cập nhật shared type.
2. Cập nhật backend.
3. Cập nhật frontend.
4. Cập nhật tài liệu.
5. Chạy typecheck/test/build.

Không tạo hai interface khác nhau cho cùng một payload nếu có thể dùng type chung.

## 7. Authorization ở Lambda

API Gateway chỉ biết JWT có hợp lệ hay không. Lambda tiếp tục kiểm tra quyền.

Ví dụ trước khi đọc Meeting:

```text
JWT userId
  ↓
load Meeting
  ↓
meeting.groupId
  ↓
load membership
  ↓
allow / deny
```

Các action quản trị có thể yêu cầu `GROUP_ADMIN`, organizer hoặc assignee tùy nghiệp vụ.

## 8. DynamoDB repository

Repository chịu trách nhiệm:

- Xây dựng `PK/SK`.
- Chọn GSI phù hợp.
- `GetItem`/`Query`.
- Conditional write.
- Transaction khi một thay đổi cần tính nguyên tử.

Business service không nên phải biết chi tiết mọi key expression của DynamoDB.

## 9. Idempotency và conflict

Các request tạo dữ liệu quan trọng có thể dùng `Idempotency-Key` để chống double-submit.

Các resource có version dùng conditional write và trả `409` khi client gửi expected version cũ.

Hai cơ chế giải quyết hai vấn đề khác nhau:

- Idempotency: cùng một ý định bị gửi lặp.
- Version conflict: hai người cùng sửa một resource.

## 10. Xử lý lỗi

Các mã phổ biến:

| Mã | Ý nghĩa |
| --- | --- |
| `400` | Request không hợp lệ |
| `401` | Chưa xác thực/JWT không hợp lệ |
| `403` | Đã xác thực nhưng không đủ quyền |
| `404` | Không tìm thấy resource/route |
| `409` | Xung đột trạng thái hoặc version |
| `500` | Lỗi nội bộ/dependency chưa được ánh xạ khác |

Không trả stack trace hoặc secret về frontend.

## 11. Log an toàn

Log nên có:

- `requestId`.
- method/path.
- status/error code.
- resource ID cần thiết.
- latency.

Không log:

- JWT.
- password.
- invitation token gốc.
- Google OAuth code/token.
- presigned URL.
- toàn bộ transcript/document.

## 12. Kiểm tra trước deploy

Từ root repository:

```powershell
npm run infra:validate
npm run lint
npm run typecheck
npm run test
npm run build
```

Với auth/core stack:

```powershell
sam validate --template-file infra/auth-integration.yaml --lint --region ap-southeast-1
npm run sam:build:auth
```

Với full application:

```powershell
npm run sam:validate:app -- --region ap-southeast-1
npm run sam:build:app
```

## 13. Smoke test

Sau deploy:

```text
GET /health
→ 200

GET /me không JWT
→ 401
```

Sau đó test qua frontend để tránh copy JWT thật vào shell history dùng chung.

Đối với full application, tiếp tục kiểm tra Group → Meeting → Minutes → Task → Dashboard và các integration được bật.

## Kết quả cần đạt

- Hiểu luồng request từ browser đến repository.
- Phân biệt core auth stack và full application handler.
- Route được bảo vệ bằng JWT và quyền nghiệp vụ ở backend.
- Frontend/backend dùng shared contract.
- Idempotency và version conflict được xử lý đúng mục đích.
- Log đủ để debug nhưng không làm lộ secret.
