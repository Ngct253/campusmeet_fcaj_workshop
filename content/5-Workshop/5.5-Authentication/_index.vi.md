---
title: "Xác thực với Amazon Cognito"
date: 2026-07-27
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Xác thực với Amazon Cognito

## Mục tiêu

Phần này triển khai ngăn xếp xác thực dùng chung của CampusMeet, lấy các giá trị đầu ra CloudFormation và cấu hình giao diện React để người dùng có thể đăng ký, xác nhận tài khoản, đăng nhập và truy cập khu vực được bảo vệ.

## Thành phần được tạo

Tệp `infra/auth-integration.yaml` định nghĩa:

| Tài nguyên | Vai trò |
| --- | --- |
| Amazon Cognito User Pool | Quản lý tài khoản và xác minh email |
| Cognito User Pool Client | Cho phép ứng dụng web dùng luồng SRP và làm mới phiên, không tạo mã bí mật ứng dụng khách |
| Amazon API Gateway HTTP API | Cung cấp API với stage `$default` và bộ kiểm tra JWT |
| AWS Lambda | Xử lý `/health` và các đường dẫn API cốt lõi |
| Vai trò IAM của Lambda | Ghi nhật ký và truy cập đúng các bảng cần thiết |
| CloudWatch Log Group | Lưu nhật ký của Lambda trong 7 ngày |

Cấu hình User Pool hiện tại:

- Đăng nhập bằng email.
- Tự động xác minh email.
- Cho phép người dùng tự đăng ký.
- Mật khẩu tối thiểu 8 ký tự, có chữ thường, chữ hoa, số và ký hiệu.
- Không tạo client secret cho ứng dụng chạy trên trình duyệt.
- MFA đang để `OFF` trong mẫu hiện tại.

## 1. Kiểm tra tài khoản và mã nguồn

Từ thư mục gốc CampusMeet:

```powershell
aws login
aws sts get-caller-identity
npm install
```

Mã tài khoản phải đúng tài khoản phát triển đã thống nhất. Ngăn xếp dữ liệu và năm bảng DynamoDB cần tồn tại trước khi triển khai API dùng các bảng này.

## 2. Kiểm tra và xây dựng mẫu SAM

```powershell
sam validate `
  --template-file infra/auth-integration.yaml `
  --lint `
  --region ap-southeast-1

npm run sam:build:auth
```

Không tiếp tục nếu `sam validate` hoặc quá trình xây dựng báo lỗi.

## 3. Xem trước thay đổi CloudFormation

Người phụ trách triển khai tạo change set nhưng chưa thực thi:

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

Kiểm tra change set phải phù hợp với phạm vi của ngăn xếp:

- Cognito User Pool và User Pool Client.
- HTTP API.
- Lambda và vai trò thực thi của Lambda.
- CloudWatch Log Group.
- Không tạo thêm bảng DynamoDB.
- Không thay thế tài nguyên ngoài dự kiến.

## 4. Thực hiện triển khai

Sau khi kiểm tra change set, thực thi trong CloudFormation Console hoặc chạy lại lệnh và bỏ `--no-execute-changeset`:

```powershell
sam deploy `
  --template-file infra/auth-integration.yaml `
  --stack-name campusmeet-dev-auth `
  --resolve-s3 `
  --capabilities CAPABILITY_IAM `
  --parameter-overrides `
    AllowedOrigin=http://localhost:5173 `
    DataTablePrefix=campusmeet-dev `
  --region ap-southeast-1
```

Kiểm tra trạng thái:

```powershell
aws cloudformation describe-stacks `
  --stack-name campusmeet-dev-auth `
  --region ap-southeast-1 `
  --query "Stacks[0].StackStatus" `
  --output text
```

Kết quả cần là `CREATE_COMPLETE` hoặc `UPDATE_COMPLETE`.

## 5. Lấy các giá trị đầu ra

```powershell
aws cloudformation describe-stacks `
  --stack-name campusmeet-dev-auth `
  --query "Stacks[0].Outputs" `
  --output table `
  --region ap-southeast-1
```

Ba giá trị dùng cho giao diện:

| CloudFormation output | Biến giao diện |
| --- | --- |
| `UserPoolId` | `VITE_COGNITO_USER_POOL_ID` |
| `UserPoolClientId` | `VITE_COGNITO_USER_POOL_CLIENT_ID` |
| `ApiUrl` | `VITE_API_BASE_URL` |

`ApiUrl` của ngăn xếp này dùng stage `$default`, vì vậy không thêm `/dev` vào cuối địa chỉ.

## 6. Cấu hình giao diện

Trong `apps/web`, sao chép `.env.example` thành `.env` và điền các giá trị vừa lấy:

```dotenv
VITE_COGNITO_USER_POOL_ID=<UserPoolId>
VITE_COGNITO_USER_POOL_CLIENT_ID=<UserPoolClientId>
VITE_API_BASE_URL=<ApiUrl>
```

Quy tắc:

- Không thêm dấu `/` cuối `VITE_API_BASE_URL`.
- Không đặt mật khẩu, token hoặc khóa AWS trong `.env`.
- Không commit `.env`.
- Khởi động lại Vite sau khi thay đổi cấu hình.

## 7. Chạy và kiểm tra luồng đăng nhập

```powershell
npm run dev
```

Kiểm tra theo thứ tự:

1. Mở `http://localhost:5173/sign-up`.
2. Đăng ký bằng email có thể nhận mã.
3. Mở trang xác nhận và nhập mã Cognito gửi qua email.
4. Đăng nhập tại `/sign-in`.
5. Xác nhận có thể truy cập `/app`.
6. Đăng xuất.
7. Xác nhận truy cập lại `/app` được chuyển về `/sign-in`.

Kiểm tra API công khai và API được bảo vệ:

```powershell
curl.exe -i "<ApiUrl>/health"
curl.exe -i "<ApiUrl>/me"
```

Kỳ vọng:

- `/health` không yêu cầu JWT và trả `200` khi API hoạt động.
- `/me` không có JWT trả `401`.

Giao diện quản lý phiên Cognito; không sao chép JWT vào tài liệu, ảnh chụp hoặc issue.

## 8. Cấu hình API Gateway liên quan đến xác thực

HTTP API sử dụng:

- Stage `$default`.
- JWT lấy từ tiêu đề `Authorization`.
- `issuer` trỏ đến đúng Cognito User Pool.
- `audience` là User Pool Client vừa được tạo.
- CORS cho phép nguồn được truyền qua tham số `AllowedOrigin`.
- Các phương thức `GET`, `POST`, `PATCH`, `DELETE` và `OPTIONS`.
- Các tiêu đề `authorization`, `content-type`, `x-request-id` và `idempotency-key`.

`GET /health` và yêu cầu `OPTIONS` không dùng bộ kiểm tra JWT. Các đường dẫn còn lại đi qua `/{proxy+}` và yêu cầu JWT hợp lệ.

## 9. Lỗi thường gặp

| Hiện tượng | Kiểm tra |
| --- | --- |
| Giao diện báo thiếu cấu hình AWS | Tên file phải là `apps/web/.env`, đủ ba biến và đã khởi động lại Vite |
| User Pool hoặc Client không tồn tại | Các ID phải thuộc cùng khu vực và cùng ngăn xếp |
| API trả `404` khi địa chỉ có `/dev` | Xóa `/dev`; ngăn xếp dùng stage `$default` |
| Trình duyệt báo CORS | `AllowedOrigin` phải đúng `http://localhost:5173`, không có đường dẫn phía sau |
| `/me` trả `401` khi gọi bằng curl | Đúng hành vi nếu không gửi JWT |
| Không nhận được email xác nhận | Kiểm tra địa chỉ email, thư rác và trạng thái người dùng trong Cognito |

## Kết quả cần đạt

Sau phần này:

- Ngăn xếp `campusmeet-dev-auth` ở trạng thái thành công.
- Giao diện có đúng `UserPoolId`, `UserPoolClientId` và `ApiUrl`.
- Người dùng đăng ký, xác nhận tài khoản, đăng nhập và đăng xuất được.
- `/health` hoạt động công khai và `/me` được bảo vệ bởi JWT.
