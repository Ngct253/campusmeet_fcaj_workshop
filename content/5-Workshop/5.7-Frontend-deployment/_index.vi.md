---
title: "Giao diện và triển khai production"
date: 2026-08-08
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# Giao diện và triển khai production

Frontend CampusMeet được xây dựng bằng React và Vite. Bản production được phục vụ qua Amazon S3 và CloudFront để có một đường dẫn HTTPS có thể dùng cho demo và nộp bài.

## Giao diện người dùng

Các màn hình chính gồm:

- đăng ký và đăng nhập;
- Dashboard;
- Group và Invitation;
- Meeting detail;
- Minutes và Task;
- Notifications và Settings.

Frontend gọi API qua `VITE_API_BASE_URL` và dùng Cognito để quản lý phiên đăng nhập.

## Chuẩn bị production

Trước khi deploy cần có:

- data stack;
- user-content/orchestration stack nếu sử dụng upload, reminder hoặc AI;
- full application stack;
- các output như API URL, User Pool ID, User Pool Client ID, frontend bucket và CloudFront domain.

Production nên sử dụng môi trường riêng như `campusmeet-prod` thay vì dùng chung dữ liệu dev.

## Build frontend

Sau khi có output từ CloudFormation, cấu hình các biến public của frontend rồi build:

```powershell
npm run build -w @campusmeet/web
```

Thư mục build được upload lên S3 và CloudFront được dùng để phân phối nội dung qua HTTPS.

## Production link

Kết quả quan trọng nhất của bước này là một URL dạng:

```text
https://<cloudfront-domain>.cloudfront.net
```

URL này phải mở được từ trình duyệt mà không cần chạy `localhost`. Người dùng phải có thể đăng ký, đăng nhập và sử dụng các chức năng core thông qua backend AWS thật.

Nếu API hoặc S3 giới hạn CORS, origin production phải được cập nhật về đúng CloudFront domain.

## Tiêu chí để chuyển sang E2E

Chỉ bắt đầu kiểm thử E2E khi:

- frontend production tải được qua HTTPS;
- `/health` của API hoạt động;
- Cognito production đã có User Pool và Client đúng;
- frontend đang trỏ đến API production;
- các stack CloudFormation không còn ở trạng thái lỗi.

Production URL sẽ được ghi lại ở phần cuối workshop cùng kết quả E2E thực tế.