---
title: "Worklog Tuần 7"
date: 2026-07-27
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7

- Chuẩn hóa cấu hình xác thực cho các thành viên.
- Hoàn thiện nền tảng dữ liệu DynamoDB.
- Đồng bộ hạ tầng, quyền truy cập và tài liệu dự án.
- Xây dựng các chức năng cốt lõi của CampusMeet.

### Công việc đã thực hiện

| Thứ | Công việc chi tiết | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| Thứ 2 | Cập nhật hướng dẫn lấy Cognito User Pool ID, Client ID và API URL.<br>Chuẩn hóa cấu hình frontend và phân chia phạm vi chức năng.<br>Thay kế hoạch 17 bảng DynamoDB bằng mô hình 5 bảng vật lý theo nhu cầu truy xuất.<br>Cập nhật CloudFormation, IAM, PK/SK, GSI, TTL, script validation và tài liệu triển khai.<br>Chạy kiểm tra hạ tầng, lint, typecheck, test và build. | 27/07/2026 | 27/07/2026 | <https://github.com/Ngct253/CampusMeet> |
| Thứ 6 | Đồng bộ tài liệu kiến trúc, kế hoạch nhóm và cấu hình môi trường frontend.<br>Hoàn thiện các chức năng liên quan đến group, membership, invitation, authorization, notification và dashboard.<br>Xây dựng chức năng tạo, xem, cập nhật và hủy cuộc họp.<br>Bổ sung kiểm tra phân quyền nhóm và lưu dữ liệu trong bảng `meeting-data`.<br>Hoàn thiện giao diện lập lịch và chi tiết cuộc họp.<br>Chạy test, lint, typecheck và build. | 31/07/2026 | 02/08/2026 | <https://github.com/Ngct253/CampusMeet> |

### Kết quả đạt được tuần 7

- Hoàn thiện tài liệu cấu hình Amazon Cognito dùng chung cho nhóm.
- Xây dựng định nghĩa và validation local cho mô hình 5 bảng DynamoDB.
- Đồng bộ CloudFormation, IAM và tài liệu triển khai.
- Hoàn thiện các chức năng nền tảng của CampusMeet.
- Xây dựng lõi quản lý cuộc họp gồm tạo, xem, cập nhật và hủy.
- Hoàn thiện giao diện lập lịch và chi tiết cuộc họp.
- Hoàn thành các bước kiểm tra lint, typecheck, test và build.
