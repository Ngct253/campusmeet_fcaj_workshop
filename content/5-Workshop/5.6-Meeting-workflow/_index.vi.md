---
title: "Cuộc họp, biên bản và công việc"
date: 2026-08-08
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# Cuộc họp, biên bản và công việc

Đây là luồng nghiệp vụ chính của CampusMeet sau khi nhóm đã được tạo.

## Quản lý cuộc họp

Group Admin hoặc người có quyền phù hợp có thể tạo cuộc họp, chọn thời gian, agenda và người tham dự. Thành viên trong nhóm có thể xem các cuộc họp liên quan đến mình.

Cuộc họp được lưu trong CampusMeet trước; Google Calendar chỉ là phần đồng bộ bổ sung nếu người tổ chức đã kết nối Google.

## Biên bản

Sau cuộc họp, người dùng có thể lưu:

- nội dung tóm tắt;
- các quyết định;
- Action Item;
- người phụ trách và thời hạn.

Biên bản được gắn trực tiếp với Meeting để người dùng có thể quay lại xem bối cảnh của công việc sau này.

## Action Item và Task

Luồng chính:

```text
Meeting
  ↓
Minutes
  ↓
Action Item
  ↓
Task
  ↓
TODO → DOING → DONE
```

Action Item là đầu việc phát sinh từ cuộc họp. Khi được xác nhận, Action Item có thể chuyển thành Task chính thức và giao cho một thành viên trong nhóm.

## Dashboard

Dashboard giúp người dùng nhìn nhanh công việc đang ở trạng thái nào, công việc nào đã hoàn thành và công việc nào còn cần xử lý.

Đây cũng là một phần quan trọng trong E2E cuối vì nó chứng minh dữ liệu từ Meeting đã đi xuyên suốt đến Task và được tổng hợp lại trên giao diện.

## Phạm vi hiện tại

Core production ưu tiên luồng Meeting → Minutes → Task. Các chức năng live recording hoặc transcription không phải điều kiện bắt buộc để hoàn thành luồng này.