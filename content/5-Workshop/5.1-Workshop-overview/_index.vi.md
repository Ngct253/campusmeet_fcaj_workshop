---
title: "Tổng quan CampusMeet"
date: 2026-07-27
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Tổng quan CampusMeet

CampusMeet là nền tảng hỗ trợ nhóm học tập và nhóm dự án quản lý thông tin trước và sau cuộc họp trong cùng một nơi. Thay vì dùng lịch cho một phần, tài liệu cho một phần và công cụ quản lý việc cho một phần khác, CampusMeet liên kết các dữ liệu này theo từng nhóm và từng cuộc họp.

## Bài toán

Một nhóm nhỏ thường gặp ba vấn đề chính:

- lịch họp, thành viên và tài liệu nằm ở nhiều công cụ khác nhau;
- biên bản và công việc sau cuộc họp dễ bị bỏ sót;
- dữ liệu khó kiểm soát quyền khi nhiều thành viên cùng sử dụng.

CampusMeet giải quyết bài toán đó bằng một luồng thống nhất:

```text
Tạo nhóm
  ↓
Mời thành viên
  ↓
Tạo cuộc họp
  ↓
Lưu biên bản và Action Item
  ↓
Chuyển thành Task
  ↓
Theo dõi tiến độ
```

Ngoài luồng cốt lõi, hệ thống còn có hướng tích hợp Google Calendar/Meet, tài liệu trên Amazon S3 và các chức năng AI dựa trên Amazon Bedrock.

## Kiến trúc tổng quan

![Kiến trúc CampusMeet AWS](images/5-Workshop/5.1-Workshop-overview/architecture-diagram.png?v=2)

CampusMeet sử dụng kiến trúc serverless trên AWS:

| Thành phần | Vai trò |
| --- | --- |
| React/Vite | Giao diện người dùng |
| Amazon Cognito | Đăng ký, đăng nhập và quản lý phiên |
| Amazon API Gateway | Cung cấp HTTP API |
| AWS Lambda | Xử lý nghiệp vụ và phân quyền |
| Amazon DynamoDB | Lưu dữ liệu chính của hệ thống |
| Amazon S3 | Lưu tệp và nội dung lớn |
| EventBridge Scheduler / Step Functions | Xử lý các công việc bất đồng bộ |
| Amazon Bedrock | Hỗ trợ tìm kiếm và tạo nội dung bằng AI |
| Amazon CloudWatch | Log và giám sát hệ thống |

Google Calendar và Google Meet là dịch vụ tích hợp bên ngoài; CampusMeet không xây dựng hệ thống gọi video riêng.

## Phạm vi của workshop

Workshop tập trung vào những phần có giá trị trực tiếp với bản triển khai cuối:

- xác thực và API;
- dữ liệu và phân quyền theo nhóm;
- Group, Invitation, Meeting, Minutes, Task và Dashboard;
- frontend production chạy qua CloudFront;
- upload tài liệu và xử lý bất đồng bộ;
- Google và AI ở mức tích hợp phù hợp với trạng thái dự án;
- giám sát, bảo mật, kiểm thử E2E và chi phí.

Live transcription, recording và batch audio transcription không được xem là yêu cầu bắt buộc của bản production E2E hiện tại.

## Cách đọc trạng thái dự án

Trong workshop, ba mức sau được phân biệt rõ:

1. **Có trong mã nguồn**: chức năng đã được triển khai trong repository.
2. **Đã kiểm tra bằng test/build**: logic đã vượt qua kiểm tra tự động hoặc local build.
3. **Đã xác minh trên AWS**: chức năng đã được deploy và chạy thật từ trình duyệt đến dịch vụ AWS.

Bản nộp cuối cùng ưu tiên mức thứ ba: có production link và một luồng E2E hoạt động thật.