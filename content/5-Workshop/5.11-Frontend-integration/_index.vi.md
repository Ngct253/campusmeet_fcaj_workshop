---
title: "Tích hợp giao diện người dùng"
date: 2026-08-08
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

# Tích hợp giao diện người dùng

## Mục tiêu

Phần này trình bày cách ứng dụng React/Vite của CampusMeet kết nối với Amazon Cognito và HTTP API, quản lý phiên đăng nhập, gửi JWT, tải dữ liệu theo route và đồng bộ trạng thái giao diện với dữ liệu phía máy chủ.

## 1. Cấu trúc frontend

Giao diện nằm trong:

```text
apps/web/
```

Các thành phần chính gồm:

- React và TypeScript.
- Vite cho môi trường phát triển và production build.
- React Router cho điều hướng.
- TanStack Query cho server state, cache và invalidation.
- Amazon Cognito cho đăng nhập và khôi phục phiên.
- API client dùng chung để gọi backend.

Frontend không truy cập DynamoDB trực tiếp.

## 2. Cấu hình môi trường

Các biến public hiện dùng:

```dotenv
VITE_COGNITO_USER_POOL_ID=<UserPoolId>
VITE_COGNITO_USER_POOL_CLIENT_ID=<UserPoolClientId>
VITE_API_BASE_URL=<ApiUrl>
VITE_GOOGLE_CLOUD_PROJECT_NUMBER=<GoogleCloudProjectNumber>
```

Các biến `VITE_*` được đưa vào bundle trình duyệt nên không được chứa:

- password;
- JWT;
- AWS access key;
- Google client secret;
- OAuth refresh token;
- secret của bất kỳ dịch vụ nào.

Tệp `.env` dùng trên máy cá nhân không được commit.

## 3. Đăng nhập và phiên Cognito

Luồng cơ bản:

```text
Sign up
  ↓
Confirm email
  ↓
Sign in
  ↓
Cognito session
  ↓
Protected application routes
```

Ứng dụng phải xử lý:

- Đăng ký.
- Xác nhận tài khoản.
- Đăng nhập.
- Quên mật khẩu.
- Khôi phục phiên khi reload.
- Đăng xuất.

Nếu phiên không hợp lệ hoặc hết hạn, route được bảo vệ phải chuyển người dùng về trang đăng nhập thay vì hiển thị dữ liệu cũ như thể phiên còn hợp lệ.

## 4. Protected routes

Các route chính hiện có trong ứng dụng gồm:

```text
/
/sign-in
/sign-up
/confirm-sign-up
/forgot-password

/app/dashboard
/app/groups
/app/groups/:groupId
/app/groups/:groupId/meetings
/app/meetings/:meetingId
/app/tasks
/app/notifications
/app/invitations
/app/settings

/meet-addon/side-panel
```

Các route dưới `/app` yêu cầu phiên đăng nhập. Frontend có thể ẩn action theo role để UX rõ hơn, nhưng backend vẫn là nơi ra quyết định cuối cùng về quyền.

## 5. API client và JWT

Luồng gọi API:

```text
React component/hook
       ↓
API service
       ↓
API client
       ↓
Authorization: Bearer <token>
       ↓
API Gateway
       ↓
Lambda
```

Không ghi token vào console log hoặc đưa token thật vào tài liệu kiểm thử.

API client cũng cần xử lý các response phổ biến:

- `400`: dữ liệu đầu vào không hợp lệ.
- `401`: thiếu hoặc phiên không hợp lệ.
- `403`: không đủ quyền.
- `404`: tài nguyên không tồn tại hoặc không được tìm thấy.
- `409`: xung đột phiên bản/trạng thái.
- `500`: lỗi phía máy chủ.

## 6. Server state với TanStack Query

TanStack Query được dùng để giữ cache dữ liệu từ backend. Sau mutation thành công, ứng dụng phải invalidate/refetch đúng query thay vì giữ dữ liệu cũ.

Ví dụ:

```text
Create Meeting
    ↓
API success
    ↓
invalidate group meetings
    ↓
Meeting list refetch
```

Tương tự với:

- Group.
- Invitation.
- Minutes.
- Task.
- Dashboard.
- Notification.
- AI job/status khi được sử dụng.

## 7. Loading, error và empty states

Một giao diện production không chỉ có happy path. Với mỗi trang quan trọng cần có:

- Loading state khi request đang chạy.
- Empty state khi chưa có dữ liệu.
- Error state khi API thất bại.
- Disable submit trong lúc request đang xử lý để giảm double submit.
- Thông báo rõ khi người dùng không có quyền.

Không hiển thị stack trace hoặc nội dung lỗi nội bộ nhạy cảm cho người dùng cuối.

## 8. Xử lý conflict

Minutes và một số dữ liệu có version. Khi server trả `409`, frontend không nên tự ghi lại bằng dữ liệu cũ.

Hướng xử lý:

```text
Local draft
   +
409 Conflict
   ↓
Giữ nội dung user đang nhập
   ↓
Thông báo dữ liệu đã thay đổi
   ↓
Cho phép tải bản mới nhất và xử lý lại
```

## 9. Meeting detail như một workspace

Trang Meeting Detail liên kết nhiều domain:

- Meeting information.
- Minutes.
- Action Item.
- Task conversion.
- Attachment.
- Google sync status.
- AI workspace nếu chức năng đã sẵn sàng.

Điều này giúp người dùng không phải chuyển qua nhiều ứng dụng rời rạc để hoàn tất quy trình sau cuộc họp.

## 10. Google Meet Add-on route

CampusMeet có route side panel:

```text
/meet-addon/side-panel
```

Route tồn tại trong frontend không đồng nghĩa Google Meet Add-on đã được publish hoặc xác minh trên Google Cloud. Việc cấu hình Google Cloud project, OAuth và unpublished deployment được kiểm thử riêng ở bước E2E tích hợp.

## 11. Production build

Chạy:

```powershell
npm run build -w @campusmeet/web
```

Hoặc từ root:

```powershell
npm run build
```

Production build phải hoàn tất trước khi đưa thư mục `dist` lên S3/CloudFront.

Build warning về kích thước bundle nên được theo dõi nhưng không tự động đồng nghĩa ứng dụng không thể triển khai. Trước deadline, không thực hiện refactor code splitting lớn nếu không có lỗi runtime thực tế.

## 12. Kiểm thử frontend

Repo hiện có automated tests cho nhiều page/service. Trước deploy chạy:

```powershell
npm run test
npm run typecheck
npm run build
```

Sau deploy phải kiểm thử bằng browser thật:

1. Mở CloudFront URL.
2. Sign up/confirm/sign in.
3. Reload trang `/app` và xác nhận phiên được khôi phục.
4. Điều hướng Group → Meeting → Minutes → Task → Dashboard.
5. Đăng xuất và xác nhận protected route không còn truy cập được.

## Kết quả cần đạt

- Frontend dùng Cognito để xác thực và API Gateway/Lambda cho dữ liệu.
- JWT được gửi qua API client, không lưu vào tài liệu/log.
- Protected routes hoạt động đúng với phiên đăng nhập.
- Query cache được cập nhật sau mutation.
- Loading/error/conflict được xử lý rõ ràng.
- Production build tạo được bundle để publish lên S3/CloudFront.
