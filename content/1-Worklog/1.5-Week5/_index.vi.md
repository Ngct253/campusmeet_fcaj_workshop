---
title: "Worklog Tuần 5"
date: 2026-07-13
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5

- Tích hợp chức năng xác thực với tài nguyên AWS.
- Bảo vệ API bằng JWT Authorizer.
- Triển khai auth stack bằng AWS SAM.
- Kiểm tra luồng xác thực và dọn dẹp tài nguyên sau khi thực hành.

### Công việc đã thực hiện

| Thứ | Công việc chi tiết | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| Thứ 2 | Hoàn thiện kết nối frontend với Amazon Cognito.<br>Cấu hình API Gateway JWT Authorizer và endpoint `GET /me` được bảo vệ.<br>Xây dựng AWS SAM stack gồm Cognito, API Gateway, Lambda, IAM và CloudWatch Log Group.<br>Triển khai và kiểm tra đăng ký, xác nhận email, đăng nhập và gọi API bằng access token.<br>Chạy test, typecheck, build, SAM validate và dọn dẹp stack sau khi kiểm tra. | 13/07/2026 | 13/07/2026 | <https://github.com/Ngct253/CampusMeet> |

### Kết quả đạt được tuần 5

- Hoàn thiện luồng xác thực bằng Amazon Cognito.
- Bảo vệ được API bằng JWT Authorizer.
- Xây dựng và triển khai được auth integration stack bằng AWS SAM.
- Hiểu luồng xử lý giữa frontend, Cognito, API Gateway và Lambda.
- Thực hiện được quy trình deploy, kiểm thử và cleanup tài nguyên xác thực.
