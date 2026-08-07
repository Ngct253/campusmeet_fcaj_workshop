---
title: "Xác thực và API"
date: 2026-08-08
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Xác thực và API

CampusMeet dùng Amazon Cognito để quản lý tài khoản người dùng và API Gateway + Lambda để xử lý các yêu cầu từ frontend.

## Xác thực với Cognito

Luồng cơ bản:

```text
Đăng ký
  ↓
Xác nhận email
  ↓
Đăng nhập
  ↓
Cognito phát JWT
  ↓
Frontend gọi API
```

Frontend chỉ lưu và sử dụng thông tin phiên cần thiết. Mật khẩu, token bí mật hoặc AWS credential không được đưa vào mã nguồn.

## API Gateway và Lambda

Frontend gửi request đến HTTP API. API Gateway kiểm tra JWT trước khi chuyển request nghiệp vụ đến Lambda.

Lambda tiếp tục kiểm tra quyền theo dữ liệu thực tế. Ví dụ, một người đã đăng nhập vẫn không được đọc nhóm hoặc cuộc họp mà mình không tham gia.

Luồng xử lý có thể hiểu đơn giản như sau:

```text
React
  ↓
API Gateway
  ↓
Lambda
  ↓
Kiểm tra quyền
  ↓
DynamoDB / dịch vụ tích hợp
```

## Cấu hình frontend

Sau khi deploy, frontend cần các giá trị public:

```dotenv
VITE_COGNITO_USER_POOL_ID=...
VITE_COGNITO_USER_POOL_CLIENT_ID=...
VITE_API_BASE_URL=...
```

Các giá trị này dùng để kết nối frontend với đúng môi trường AWS. Chúng không phải secret.

## Phân biệt xác thực và phân quyền

- **Authentication** trả lời: người dùng là ai?
- **Authorization** trả lời: người dùng được phép làm gì?

CampusMeet luôn kiểm tra cả hai lớp. Đây là nguyên tắc quan trọng vì dữ liệu của nhiều nhóm được quản lý trong cùng một hệ thống.

Trong production, API chỉ được xem là sẵn sàng khi `/health` hoạt động và các route được bảo vệ thực sự yêu cầu đăng nhập.