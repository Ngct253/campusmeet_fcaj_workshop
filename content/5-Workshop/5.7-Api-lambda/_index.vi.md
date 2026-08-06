---
title: "API Gateway và AWS Lambda"
date: 2026-07-27
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# API Gateway và AWS Lambda

## Mục tiêu

Phần này giải thích cách CampusMeet tiếp nhận yêu cầu HTTP, xác thực JWT, chuyển yêu cầu đến Lambda và truy cập dữ liệu thông qua lớp nghiệp vụ cùng lớp truy cập dữ liệu. Nội dung bám theo `infra/auth-integration.yaml`, `services/api/src/auth-integration.ts` và `docs/api-contract.md`.

## Cấu trúc API hiện tại

Ngăn xếp `campusmeet-dev-auth` tạo một HTTP API và một Lambda cốt lõi:

```text
Amazon API Gateway HTTP API
        |
        +--- GET /health          Không yêu cầu JWT
        |
        +--- OPTIONS /{proxy+}    Không yêu cầu JWT
        |
        +--- ANY /{proxy+}        Yêu cầu JWT Cognito
                                  |
                                  v
                     AuthIntegrationFunction
```

HTTP API dùng stage `$default`. Địa chỉ cơ sở có dạng:

```text
https://<api-id>.execute-api.ap-southeast-1.amazonaws.com
```

Không thêm `/dev` vào địa chỉ này.

## 1. Luồng xử lý yêu cầu

Một yêu cầu được bảo vệ đi qua các bước:

1. Giao diện lấy phiên đăng nhập từ Amazon Cognito.
2. JWT được gửi trong tiêu đề `Authorization`.
3. API Gateway kiểm tra JWT theo User Pool và User Pool Client đã triển khai.
4. Lambda đọc phương thức, đường dẫn, tham số và nội dung yêu cầu.
5. Lambda lấy danh tính người dùng từ thông tin JWT.
6. Lớp nghiệp vụ kiểm tra thành viên và vai trò trước khi gọi lớp truy cập dữ liệu.
7. Lớp truy cập dữ liệu thực hiện truy vấn hoặc ghi vào đúng bảng DynamoDB.
8. Lambda trả phản hồi theo hợp đồng dùng chung trong `@campusmeet/shared`.

Giao diện không được truyền một vai trò rồi yêu cầu backend tin vào giá trị đó. Quyền luôn được xác minh bằng dữ liệu membership đang lưu.

## 2. Cấu hình HTTP API

`infra/auth-integration.yaml` cấu hình:

| Thuộc tính | Giá trị |
| --- | --- |
| Stage | `$default` |
| Nguồn CORS mặc định | `http://localhost:5173` |
| Phương thức CORS | `GET`, `POST`, `PATCH`, `DELETE`, `OPTIONS` |
| Tiêu đề được phép | `authorization`, `content-type`, `x-request-id`, `idempotency-key` |
| Nguồn JWT | `$request.header.Authorization` |
| Bộ kiểm tra mặc định | `CognitoAuthorizer` |

`GET /health` được khai báo riêng và không dùng bộ kiểm tra JWT. Tất cả đường dẫn nghiệp vụ còn lại đi qua `/{proxy+}`.

## 3. Các nhóm đường dẫn cốt lõi

| Nhóm chức năng | Đường dẫn chính | Trạng thái trong mã nguồn |
| --- | --- | --- |
| Kiểm tra dịch vụ | `GET /health` | Có bộ xử lý thật |
| Hồ sơ | `GET /me`, `PATCH /me` | Đã triển khai |
| Nhóm | `GET /groups`, `POST /groups`, `GET/PATCH /groups/:groupId` | Đã triển khai |
| Thành viên | `DELETE /groups/:groupId/members/:userId` | Đã triển khai; không cho xóa Group Admin |
| Lời mời | Các đường dẫn dưới `/groups/:groupId/invitations` và `/invitations` | Đã triển khai |
| Cuộc họp | `GET /meetings`, các đường dẫn dưới `/groups/:groupId/meetings` và `/meetings/:meetingId` | Lõi tạo, xem, sửa và hủy đã có trong mã nguồn |
| Thông báo | `GET /notifications`, `POST /notifications/:notificationId/read` | Đã triển khai |

Các mô-đun chưa có bộ xử lý đầy đủ có thể trả `501 Not Implemented` theo hợp đồng API. Không xem việc đường dẫn xuất hiện trong tài liệu là bằng chứng chức năng đã được triển khai hoàn chỉnh.

## 4. Kiểu dữ liệu dùng chung

Frontend và backend dùng các kiểu dữ liệu từ:

```text
@campusmeet/shared
```

Không sao chép lại giao diện TypeScript cho cùng một request hoặc response ở nhiều nơi. Khi thay đổi hợp đồng:

1. Cập nhật kiểu dữ liệu dùng chung.
2. Cập nhật backend.
3. Cập nhật giao diện.
4. Cập nhật `docs/api-contract.md`.
5. Chạy kiểm tra kiểu, kiểm thử và build.

Các lệnh kiểm tra:

```powershell
npm run lint
npm run typecheck
npm run test
npm run build
npm run format:check
```

## 5. Ranh giới mã nguồn Lambda

Các lớp chính:

```text
API Gateway event
      ↓
Bộ xử lý yêu cầu
      ↓
Lớp nghiệp vụ
      ↓
Giao diện repository
      ↓
DynamoDB repository hoặc kho dữ liệu trong bộ nhớ
```

Nguyên tắc:

- Bộ xử lý không truy vấn DynamoDB trực tiếp khi đã có lớp repository.
- Lớp nghiệp vụ chịu trách nhiệm kiểm tra quyền và quy tắc nghiệp vụ.
- Repository chịu trách nhiệm ánh xạ khóa, chỉ mục và thao tác DynamoDB.
- Kiểm thử đơn vị có thể dùng repository trong bộ nhớ.
- Môi trường AWS dùng chung dành cho kiểm thử tích hợp và kiểm thử nhanh sau triển khai.

## 6. Quyền truy cập dữ liệu của Lambda

Vai trò Lambda trong ngăn xếp hiện tại truy cập:

- Bảng `campusmeet-dev-identity` và chỉ mục.
- Bảng `campusmeet-dev-collaboration` và chỉ mục.
- Bảng `campusmeet-dev-meeting-data` và chỉ mục.

Các hành động được giới hạn vào `GetItem`, `BatchGetItem`, `Query`, `PutItem`, `UpdateItem`, `DeleteItem` và `ConditionCheckItem`.

Lambda cũng có `cognito-idp:AdminGetUser` trên đúng User Pool để lấy email đã xác minh khi nghiệp vụ cần đối chiếu lời mời.

## 7. Triển khai cập nhật API

Khi mã Lambda hoặc mẫu hạ tầng thay đổi:

```powershell
sam validate `
  --template-file infra/auth-integration.yaml `
  --lint `
  --region ap-southeast-1

npm run sam:build:auth
```

Tạo change set trước khi thực thi:

```powershell
sam deploy `
  --template-file infra/auth-integration.yaml `
  --stack-name campusmeet-dev-auth `
  --resolve-s3 `
  --capabilities CAPABILITY_IAM `
  --parameter-overrides `
    AllowedOrigin=http://localhost:5173 `
    DataTablePrefix=campusmeet-dev `
  --no-execute-changeset `
  --region ap-southeast-1
```

Kiểm tra thay đổi chỉ cập nhật tài nguyên thuộc ngăn xếp. Không sửa vai trò IAM hoặc đường dẫn API thủ công trên Console rồi bỏ qua mã nguồn.

## 8. Kiểm thử API

### Đường dẫn công khai

```powershell
curl.exe -i "<ApiUrl>/health"
```

Kỳ vọng trả `200` và thông tin trạng thái dịch vụ.

### Đường dẫn được bảo vệ

```powershell
curl.exe -i "<ApiUrl>/me"
```

Không có JWT phải trả `401`.

Đối với luồng có đăng nhập, kiểm thử qua giao diện CampusMeet để ứng dụng tự lấy và gửi JWT. Không đưa token thật vào tài liệu hoặc lịch sử lệnh dùng chung.

### Các trường hợp quyền cần kiểm tra

- Người đã đăng nhập nhưng không thuộc nhóm không được xem chi tiết nhóm.
- Thành viên thường không được sửa nhóm hoặc gửi lời mời.
- Group Admin được thực hiện thao tác quản trị trong nhóm của mình.
- Người dùng không được truy cập cuộc họp thuộc nhóm khác.
- Yêu cầu tạo có khả năng gửi lặp phải dùng `Idempotency-Key` khi hợp đồng yêu cầu.

## 9. Nhật ký và xử lý lỗi

Nhóm nhật ký của Lambda:

```text
/aws/lambda/campusmeet-dev-auth-api
```

Mẫu hiện tại giữ nhật ký trong 7 ngày.

Nhật ký nên chứa:

- Mã yêu cầu.
- Đường dẫn và phương thức.
- Mã trạng thái.
- Mã lỗi và ID tài nguyên cần cho chẩn đoán.

Không ghi:

- JWT hoặc mã OAuth.
- Mật khẩu hoặc mã xác nhận.
- Token lời mời gốc.
- Toàn bộ nội dung tệp, bản phiên âm hoặc hội thoại.

Ý nghĩa các phản hồi thường gặp:

| Mã | Ý nghĩa |
| --- | --- |
| `401` | Thiếu hoặc JWT không hợp lệ |
| `403` | Đã xác thực nhưng không đủ quyền hoặc không thuộc nhóm |
| `404` | Không tìm thấy tài nguyên hoặc đường dẫn |
| `409` | Xung đột trạng thái hoặc dữ liệu đã tồn tại |
| `501` | Mô-đun có hợp đồng nhưng chưa có bộ xử lý hoàn chỉnh |

## Kết quả cần đạt

- Hiểu rõ đường đi của một yêu cầu từ trình duyệt đến DynamoDB.
- `/health` hoạt động công khai và API nghiệp vụ yêu cầu JWT.
- Frontend và backend dùng cùng kiểu dữ liệu từ `@campusmeet/shared`.
- Mọi thao tác theo nhóm đều kiểm tra membership và vai trò ở backend.
- Nhật ký hỗ trợ chẩn đoán mà không làm lộ dữ liệu nhạy cảm.
