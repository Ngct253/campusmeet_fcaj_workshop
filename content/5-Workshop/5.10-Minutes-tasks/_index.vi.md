---
title: "Biên bản và công việc"
date: 2026-08-08
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

# Biên bản và công việc

## Mục tiêu

Phần này trình bày luồng sau cuộc họp của CampusMeet: ghi biên bản, lưu quyết định, tạo Action Item, chuyển Action Item thành Task chính thức và theo dõi trạng thái công việc trên Dashboard.

Luồng chính:

```text
Meeting
  ↓
Minutes
  ↓
Decision + Action Item
  ↓
Action Item → Task
  ↓
TODO → DOING → DONE
  ↓
Dashboard
```

## 1. Biên bản cuộc họp

Đường dẫn chính:

```http
GET /meetings/:meetingId/minutes
PUT /meetings/:meetingId/minutes
```

Biên bản gắn với một cuộc họp cụ thể. Trước khi đọc hoặc ghi, backend phải xác định cuộc họp thuộc nhóm nào và kiểm tra người dùng có quyền truy cập nhóm đó.

Nội dung biên bản có thể gồm:

- Tóm tắt nội dung đã trao đổi.
- Danh sách quyết định.
- Danh sách Action Item.
- Người phụ trách.
- Thời hạn.
- Phiên bản dữ liệu.

## 2. Phiên bản và xung đột cập nhật

CampusMeet không để một bản biên bản cũ âm thầm ghi đè bản mới.

Ví dụ:

```text
User A đọc Minutes version=3
User B đọc Minutes version=3

B lưu trước → version=4
A tiếp tục lưu expectedVersion=3
        ↓
Backend phát hiện xung đột
        ↓
409 Conflict
```

Frontend cần giữ phần nội dung người dùng đang chỉnh sửa và hiển thị trạng thái xung đột thay vì làm mất dữ liệu cục bộ.

## 3. Decision

Decision là kết quả đã được thống nhất trong cuộc họp. Decision được lưu cùng biên bản và nên thể hiện nội dung ngắn gọn, có thể truy xuất lại sau này.

Không dùng nội dung AI chưa được người dùng xác nhận như một Decision chính thức.

## 4. Action Item

Action Item mô tả một việc phát sinh từ cuộc họp, thường gồm:

- Nội dung việc cần làm.
- Người phụ trách.
- Deadline nếu có.
- Trạng thái chuyển đổi sang Task.

Người được gán phải là thành viên hợp lệ của nhóm tại thời điểm backend kiểm tra.

## 5. Chuyển Action Item thành Task

Đường dẫn:

```http
POST /meetings/:meetingId/minutes/action-items/:actionItemId/task
```

Luồng:

```text
Action Item trong Minutes
      ↓
Kiểm tra Meeting/Group
      ↓
Kiểm tra assignee
      ↓
Tạo Task chính thức
      ↓
Ghi liên kết taskId về Action Item
```

Thao tác cần chống tạo trùng. Nếu người dùng nhấn hai lần hoặc mạng retry, cùng một Action Item không được sinh nhiều Task ngoài ý muốn.

## 6. Quản lý Task

Các đường dẫn chính:

```http
GET /tasks
POST /tasks
PATCH /tasks/:taskId/status
```

Task là dữ liệu chính thức dùng để theo dõi công việc sau cuộc họp. Task có thể được tạo trực tiếp theo hợp đồng hoặc được sinh từ Action Item.

Trạng thái chính:

```text
TODO
  ↓
DOING
  ↓
DONE
```

Một số nghiệp vụ cho phép mở lại công việc theo quy tắc hiện tại. Backend chịu trách nhiệm kiểm tra transition hợp lệ, không chỉ dựa vào dropdown của frontend.

## 7. Phân quyền Task

Các quy tắc cần được kiểm tra ở backend:

- Chỉ người thuộc nhóm liên quan mới được thấy Task.
- Assignee có thể cập nhật trạng thái theo hợp đồng.
- Group Admin có quyền quản trị tương ứng.
- User của nhóm khác không được đọc hoặc sửa Task dù biết `taskId`.

## 8. Dashboard

Dashboard tổng hợp trạng thái công việc của người dùng hiện tại. Các chỉ số đang được sử dụng gồm:

- Tổng số Task.
- TODO.
- DOING.
- DONE.
- Overdue khi thời hạn đã qua nhưng công việc chưa hoàn thành.

Luồng kiểm thử quan trọng:

```text
Task của User B = TODO
        ↓
B đổi DOING
        ↓
Dashboard reload
        ↓
counter thay đổi
        ↓
B đổi DONE
        ↓
Dashboard phản ánh DONE
```

## 9. AI và dữ liệu chính thức

CampusMeet có chức năng AI Task Proposal, nhưng proposal chỉ là đề xuất. Proposal không được coi là Task chính thức chỉ vì model đã sinh kết quả.

Trong bản E2E hiện tại, luồng chắc chắn để tạo công việc chính thức là:

```text
Minutes Action Item
→ người dùng xác nhận
→ Convert to Task
```

Nếu AI proposal confirmation chưa được triển khai E2E đầy đủ, workshop phải mô tả đúng trạng thái đó thay vì tuyên bố AI tự tạo Task.

## 10. Kiểm thử mã nguồn

Trước khi triển khai:

```powershell
npm run lint
npm run typecheck
npm run test
npm run build
```

Các trường hợp quan trọng:

- Tạo và đọc Minutes.
- Edit version N thành N+1.
- Expected version cũ trả 409.
- Thêm/xóa Decision.
- Thêm/xóa Action Item.
- Assignee không thuộc nhóm bị từ chối.
- Convert Action Item thành Task đúng một lần.
- Task status transition hợp lệ.
- User ngoài nhóm bị từ chối.
- Dashboard tổng hợp đúng trạng thái Task.

## 11. E2E nhanh

Sau deploy:

1. A tạo Meeting.
2. A mở Meeting detail.
3. A lưu Minutes có summary, Decision và một Action Item gán cho B.
4. Reload và xác nhận Minutes vẫn tồn tại.
5. A convert Action Item thành Task.
6. B đăng nhập và thấy Task.
7. B đổi `TODO → DOING → DONE`.
8. B mở Dashboard và xác nhận số liệu thay đổi.

Đây là một phần bắt buộc của core E2E production demo.

## Kết quả cần đạt

- Minutes được lưu theo phiên bản và chống ghi đè cạnh tranh.
- Decision và Action Item gắn đúng Meeting.
- Action Item chuyển được thành Task chính thức.
- Task có state transition được backend kiểm soát.
- Dashboard phản ánh dữ liệu Task thực tế.
