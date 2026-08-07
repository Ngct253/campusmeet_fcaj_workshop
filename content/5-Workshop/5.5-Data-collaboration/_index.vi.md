---
title: "Dữ liệu và cộng tác nhóm"
date: 2026-08-08
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Dữ liệu và cộng tác nhóm

CampusMeet dùng DynamoDB làm kho dữ liệu chính. Thay vì trình bày chi tiết từng khóa và chỉ mục, workshop chỉ tập trung vào cách dữ liệu được chia theo chức năng và cách người dùng cộng tác trong một nhóm.

## Mô hình dữ liệu

Hệ thống sử dụng năm bảng chính:

| Bảng | Nội dung |
| --- | --- |
| `identity` | hồ sơ người dùng và thông báo |
| `collaboration` | nhóm, thành viên và lời mời |
| `meeting-data` | cuộc họp, biên bản và tệp liên quan |
| `task-data` | công việc và trạng thái |
| `ai-work` | AI job, nguồn kiến thức và citation |

Các bảng được quản lý bằng CloudFormation để cấu hình dev và production nhất quán.

## Nhóm và thành viên

Người tạo Group trở thành `GROUP_ADMIN`. Thành viên được mời vào nhóm có vai trò `MEMBER`.

Hai vai trò này đủ cho phiên bản hiện tại:

- `GROUP_ADMIN`: quản lý thông tin nhóm, lời mời và các thao tác quản trị.
- `MEMBER`: tham gia các chức năng trong phạm vi được cấp.

Backend luôn kiểm tra membership hiện tại trước khi trả dữ liệu của nhóm.

## Lời mời tham gia

Luồng người dùng:

```text
Admin tạo Group
   ↓
Nhập email thành viên
   ↓
CampusMeet tạo lời mời
   ↓
Người nhận đăng nhập
   ↓
Accept / Decline
```

Khi người nhận chấp nhận, họ trở thành member của nhóm. Hệ thống đồng thời quản lý trạng thái lời mời và notification để người dùng dễ theo dõi.

## Nguyên tắc dữ liệu

CampusMeet giữ một số nguyên tắc đơn giản:

- dữ liệu của nhóm chỉ được đọc bởi thành viên phù hợp;
- thao tác quản trị chỉ dành cho role được phép;
- tệp lớn nằm trên S3, không lưu trực tiếp trong DynamoDB;
- các thay đổi quan trọng được thực hiện theo cách tránh dữ liệu dở dang hoặc trùng lặp.

Những chi tiết như PK/SK, GSI hay transaction cụ thể thuộc phần triển khai nội bộ và không cần lặp lại trong workshop chính.