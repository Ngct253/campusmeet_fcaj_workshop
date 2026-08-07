---
title: "Xử lý bất đồng bộ, Google và AI"
date: 2026-08-08
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

# Xử lý bất đồng bộ, Google và AI

Một số công việc của CampusMeet không nên xử lý hoàn toàn trong một request ngắn, đặc biệt khi phải gọi dịch vụ ngoài hoặc xử lý tài liệu.

## Tệp và Amazon S3

Tài liệu người dùng được upload trực tiếp lên S3 bằng URL có thời hạn. Backend kiểm tra quyền và lưu metadata, còn file thật nằm trong bucket private.

Cách này giúp Lambda không phải nhận toàn bộ file và giữ ranh giới dữ liệu rõ ràng.

## Google Calendar và Google Meet

Khi người tổ chức kết nối Google, CampusMeet có thể đồng bộ cuộc họp sang Calendar và nhận lại thông tin Google Meet khi Google hỗ trợ.

Luồng được xử lý bất đồng bộ:

```text
Meeting thay đổi
  ↓
CampusMeet lưu dữ liệu
  ↓
Worker đồng bộ Google
  ↓
Cập nhật trạng thái sync
```

Nếu Google lỗi, Meeting bên trong CampusMeet vẫn được giữ lại.

## AI với Amazon Bedrock

CampusMeet dùng Bedrock để hỗ trợ một số chức năng như:

- hỏi đáp theo nội dung cuộc họp;
- tìm kiếm trong phạm vi nhóm;
- tạo bản nháp Minutes;
- đề xuất Action Item hoặc Task.

Tài liệu cần được đưa vào Knowledge Base trước khi dùng cho retrieval. Kết quả AI quan trọng vẫn cần người dùng xem lại trước khi trở thành dữ liệu chính thức.

## Quyền truy cập và citation

AI chỉ được dùng dữ liệu thuộc group/meeting mà người dùng có quyền đọc. Câu trả lời dựa trên tài liệu nên đi kèm citation để người dùng kiểm tra nguồn.

Nếu không có nguồn phù hợp, hệ thống nên thể hiện thiếu ngữ cảnh thay vì tạo một câu trả lời khẳng định không có căn cứ.

## Những phần chưa thuộc core production

Live transcription, recording lifecycle và batch audio transcription chưa phải điều kiện của bản E2E hiện tại. Workshop không bật các phần này chỉ để tăng số lượng tính năng trong bản nộp.