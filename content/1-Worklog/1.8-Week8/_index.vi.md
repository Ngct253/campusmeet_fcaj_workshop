---
title: "Worklog Tuần 8"
date: 2026-08-03
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8

- Ghi nhận và xác minh trạng thái triển khai lõi cuộc họp M2 trên môi trường AWS dev.
- Điều chỉnh một số màn hình và luồng thuộc M1–M4 trong phạm vi các commit đã thực hiện.
- Cải thiện biểu mẫu cuộc họp, upload tài liệu, biên bản và quy trình cập nhật công việc trong phạm vi được giao.
- Sửa các lỗi hạ tầng, presigned upload và bố cục giao diện được phát hiện khi kiểm thử.

### Công việc đã thực hiện

| Thứ | Công việc chi tiết | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 5 | Căn chỉnh biến môi trường của bảng `meeting-data` trong hạ tầng để Lambda sử dụng đúng bảng (64f5035).<br>Cập nhật README, tài liệu kiến trúc, runbook triển khai và script kiểm tra để ghi nhận chính xác trạng thái triển khai M2 trên AWS (d3b111d). | 06/08/2026 | 06/08/2026 | <https://github.com/Ngct253/CampusMeet> |
| 7 | Điều chỉnh một số giao diện và cấu hình M1–M4 liên quan đến nhóm, cuộc họp, task, cài đặt, điều hướng và trạng thái chức năng (1c8874a).<br>Cải thiện biểu mẫu tạo/cập nhật cuộc họp và luồng upload tài liệu; bổ sung kiểm thử và cập nhật hạ tầng user-content (1659f86).<br>Làm rõ giao diện biên bản, action item và quy trình cập nhật trạng thái task; bổ sung kiểu dữ liệu, repository, service và kiểm thử liên quan (1c9e99f).<br>Bổ sung quyền đọc hồ sơ theo lô cho reminder Lambda (9413b12).<br>Sửa giao diện để gửi đúng metadata mà presigned S3 URL đã ký (db775ce).<br>Ổn định presigned document upload ở frontend, S3 adapter và kiểm thử tích hợp (5814e4a).<br>Sửa bố cục biểu mẫu cuộc họp và style dùng chung (3b50dee). | 08/08/2026 | 08/08/2026 | <https://github.com/Ngct253/CampusMeet> |

### Kết quả đạt được tuần 8

- Ghi nhận chính xác trạng thái triển khai M2 và đồng bộ tài liệu với môi trường AWS dev.
- Khắc phục cấu hình bảng cuộc họp và quyền đọc dữ liệu cần thiết cho reminder Lambda.
- Điều chỉnh một số màn hình và luồng M1–M4 trong phạm vi phần việc của tôi để cách sử dụng rõ ràng hơn.
- Cải thiện biểu mẫu cuộc họp và luồng upload tài liệu bằng presigned URL trong phần việc của tôi.
- Làm rõ cách trình bày biên bản, action item và quy trình cập nhật trạng thái công việc.
- Sửa lỗi metadata khi upload lên S3, tăng độ ổn định của upload và khắc phục lỗi bố cục biểu mẫu.
- Bổ sung, cập nhật các kiểm thử liên quan đến cuộc họp, task, attachment và tích hợp S3.
