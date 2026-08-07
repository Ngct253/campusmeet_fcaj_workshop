---
title: "Quản lý cuộc họp"
date: 2026-08-08
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

# Quản lý cuộc họp

## Mục tiêu

Phần này trình bày cách CampusMeet quản lý vòng đời một cuộc họp sau khi nhóm đã được tạo và người dùng đã được xác thực. Nội dung bám theo các bộ xử lý, lớp nghiệp vụ, repository và kiểm thử hiện có trong `services/api` và giao diện React.

Sau phần này, người thực hiện cần hiểu được cách CampusMeet:

- Tạo cuộc họp trong phạm vi một nhóm.
- Liệt kê cuộc họp của nhóm và các cuộc họp liên quan đến người dùng hiện tại.
- Xem chi tiết, cập nhật và hủy cuộc họp.
- Kiểm tra quyền dựa trên tư cách thành viên, vai trò và người tổ chức.
- Dùng phiên bản dữ liệu để tránh ghi đè thay đổi cạnh tranh.
- Tách trạng thái cuộc họp nội bộ khỏi trạng thái đồng bộ Google Calendar.

## 1. Mô hình cuộc họp

Dữ liệu cuộc họp nằm trong bảng `meeting-data`. Mục chính của cuộc họp sử dụng khóa dạng:

```text
PK=MEETING#<meetingId>
SK=META
```

Thông tin nghiệp vụ chính gồm:

- `meetingId`.
- `groupId`.
- `organizerId`.
- `title`.
- `startAt` và `endAt`.
- `status`.
- `version`.
- Nội dung agenda.
- Danh sách người tham dự hoặc các mục liên quan đến người tham dự.
- Trạng thái đồng bộ Google khi chức năng tích hợp được sử dụng.

`groupId` là ranh giới quyền quan trọng. Có JWT hợp lệ không có nghĩa người dùng có thể đọc hoặc sửa mọi cuộc họp.

## 2. Tạo cuộc họp

Đường dẫn:

```http
POST /groups/:groupId/meetings
```

Quy trình tổng quát:

```text
JWT hợp lệ
   ↓
Kiểm tra membership của groupId
   ↓
Kiểm tra quyền tạo cuộc họp
   ↓
Kiểm tra thời gian và dữ liệu đầu vào
   ↓
Ghi Meeting + dữ liệu liên quan
   ↓
Trả Meeting đã được lưu
```

Giao diện không gửi một vai trò rồi yêu cầu backend tin giá trị đó. Quyền được xác định từ membership đang lưu.

Khi thao tác tạo hỗ trợ `Idempotency-Key`, client nên gửi khóa ổn định cho cùng một ý định tạo để tránh tạo bản ghi trùng khi người dùng nhấn lại hoặc mạng retry.

## 3. Liệt kê cuộc họp

CampusMeet có hai nhu cầu truy xuất chính:

```http
GET /groups/:groupId/meetings
GET /meetings
```

`GET /groups/:groupId/meetings` dùng khi người dùng đang đứng trong một nhóm cụ thể.

`GET /meetings` dùng cho góc nhìn cá nhân để hiển thị các cuộc họp liên quan đến người dùng hiện tại.

Repository sử dụng khóa và GSI của bảng thay vì quét toàn bộ dữ liệu. Người dùng không thuộc nhóm không được dùng đường dẫn của nhóm để đọc timeline cuộc họp.

## 4. Xem chi tiết cuộc họp

Đường dẫn:

```http
GET /meetings/:meetingId
```

Backend phải xác định `groupId` của cuộc họp rồi kiểm tra membership trước khi trả dữ liệu. Quy tắc này ngăn một người dùng đã đăng nhập đoán `meetingId` của nhóm khác để truy cập dữ liệu.

Trang chi tiết trên frontend là nơi liên kết các chức năng sau:

- Thông tin và trạng thái cuộc họp.
- Agenda và người tham dự.
- Biên bản.
- Action Item và Task liên quan.
- Tệp đính kèm.
- Trạng thái Google sync.
- Không gian AI nếu chức năng AI được bật và đã triển khai đúng môi trường.

## 5. Cập nhật cuộc họp

Đường dẫn:

```http
PATCH /meetings/:meetingId
```

Thay đổi có thể bao gồm tên cuộc họp, thời gian, agenda hoặc các trường được hợp đồng API cho phép.

CampusMeet sử dụng `version` để kiểm soát cập nhật cạnh tranh. Ý tưởng:

```text
Client đọc Meeting version=4
        ↓
Client gửi cập nhật với expected version=4
        ↓
Backend chỉ cập nhật nếu bản lưu vẫn là version=4
        ↓
Thành công → version=5
```

Nếu một người khác đã sửa trước và dữ liệu hiện là version 5, yêu cầu cũ không được âm thầm ghi đè. Backend trả xung đột để client tải lại dữ liệu mới nhất hoặc yêu cầu người dùng quyết định.

## 6. Hủy cuộc họp

Đường dẫn:

```http
POST /meetings/:meetingId/cancel
```

Hủy là thay đổi trạng thái nghiệp vụ, không đồng nghĩa xóa bản ghi khỏi DynamoDB. Giữ lại dữ liệu giúp hệ thống còn lịch sử, biên bản và các liên kết sau cuộc họp.

Sau khi cuộc họp bị hủy, các thao tác không còn hợp lệ phải bị từ chối theo lớp nghiệp vụ.

## 7. Quyền truy cập

Tối thiểu cần kiểm tra các trường hợp:

- Người ngoài nhóm không được đọc cuộc họp.
- Member chỉ được thực hiện những thao tác hợp đồng cho phép.
- Group Admin có quyền quản trị trong nhóm theo quy tắc nghiệp vụ.
- Người tổ chức có thể có thêm quyền trên cuộc họp do mình tổ chức.
- Không tin `groupId`, `organizerId`, `role` do frontend tự khai báo nếu backend có thể suy ra từ dữ liệu hiện có.

Quyền được kiểm tra ở backend ngay cả khi frontend đã ẩn nút thao tác.

## 8. Trạng thái Google Calendar/Meet

Google Calendar là tích hợp bên ngoài và không được làm thay đổi nguồn dữ liệu chính của CampusMeet.

Một cuộc họp có thể có trạng thái đồng bộ như:

```text
PENDING
SYNCED
FAILED
ACTION_REQUIRED
```

Luồng tổng quát:

```text
Meeting được tạo/cập nhật/hủy
          ↓
CampusMeet lưu trạng thái mong muốn
          ↓
Worker đồng bộ Google xử lý bất đồng bộ
          ↓
SYNCED hoặc trạng thái lỗi có thể retry/xử lý lại
```

Nếu Google tạm thời lỗi, cuộc họp nội bộ vẫn phải tồn tại. Người dùng không được mất Meeting chỉ vì API Google trả lỗi.

Việc xác minh thật Google OAuth, Calendar event và Meet URL được thực hiện ở phần kiểm thử E2E. Sự tồn tại của code và unit test không được xem là bằng chứng tích hợp Google trên cloud đã thành công.

## 9. Kiểm thử mã nguồn

Repo hiện có kiểm thử cho các lớp Meeting, repository và frontend pages. Trước khi triển khai chạy:

```powershell
npm run lint
npm run typecheck
npm run test
npm run build
```

Các tình huống nên được giữ trong test suite:

- Tạo Meeting hợp lệ.
- Người ngoài nhóm bị từ chối.
- Đọc Meeting đúng nhóm.
- Cập nhật đúng phiên bản.
- Cập nhật với phiên bản cũ trả xung đột.
- Hủy Meeting.
- Cross-group access bị từ chối.
- Google sync failure không làm rollback dữ liệu Meeting chính.

## 10. Kiểm thử nhanh sau triển khai

Sau khi application stack được triển khai, dùng ít nhất hai tài khoản thật:

1. User A tạo nhóm.
2. User B tham gia nhóm.
3. A tạo Meeting có tiêu đề, thời gian và agenda.
4. B mở được Meeting vì đã là member.
5. User ngoài nhóm không được đọc Meeting.
6. A cập nhật Meeting và tải lại trình duyệt.
7. Dữ liệu mới vẫn còn sau reload.
8. A hủy Meeting và xác nhận trạng thái được lưu.

Nếu Google integration được bật, kiểm thử đồng bộ Google là nhóm kiểm thử bổ sung, không thay thế core Meeting test.

## Kết quả cần đạt

Sau phần này:

- Cuộc họp được quản lý trong ranh giới nhóm.
- Create/list/detail/update/cancel có hợp đồng và implementation thật.
- Cập nhật cạnh tranh được bảo vệ bằng version/conditional write.
- Google sync được tách khỏi giao dịch nghiệp vụ chính.
- Người dùng không thể đọc cuộc họp của nhóm khác chỉ bằng cách biết `meetingId`.
