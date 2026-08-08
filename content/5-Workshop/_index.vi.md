---
title: "Workshop"
date: 2026-08-08
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

## Giới thiệu CampusMeet

CampusMeet là nền tảng hỗ trợ nhóm học tập và nhóm dự án quản lý thông tin trước, trong và sau cuộc họp tại một nơi. Thay vì tách lịch họp, tài liệu, biên bản và công việc sang nhiều công cụ, CampusMeet liên kết chúng theo từng nhóm và từng cuộc họp.

Workshop trình bày CampusMeet theo cả góc nhìn sản phẩm và triển khai. Nội dung tập trung vào vấn đề cần giải quyết, kiến trúc, quyền truy cập, dữ liệu, quy trình cuộc họp và cách kiểm chứng. Các phần kỹ thuật được chọn lọc để người đọc hiểu phương hướng xây dựng hệ thống nhưng không thay thế tài liệu API hoặc runbook đầy đủ.

### Mục tiêu

Sau khi đọc workshop, người xem có thể:

- Hiểu vấn đề và mục tiêu mà CampusMeet hướng đến.
- Nắm được quy trình từ khi tạo nhóm đến khi kết thúc cuộc họp và theo dõi công việc.
- Hiểu vai trò của các thành phần AWS trong sơ đồ ở mức tổng quan.
- Nắm được cách chuẩn bị môi trường, kết nối frontend với AWS và kiểm tra chất lượng trước khi triển khai.
- Phân biệt phần đã có, phần cần kiểm chứng thêm và định hướng phát triển.
- Nhận biết nguyên tắc về quyền truy cập, bảo vệ thông tin và kiểm soát nội dung do AI đề xuất.

### Kiến trúc tổng quan

![Kiến trúc tổng quan của CampusMeet](images/5-Workshop/5.1-Workshop-overview/architecture-diagram.png?v=2)

Sơ đồ mô tả một luồng đơn giản: người dùng truy cập CampusMeet, đăng nhập, thực hiện thao tác qua hệ thống trung tâm, sau đó thông tin được lưu trữ và có thể được hỗ trợ bởi lịch, thông báo, phiên âm và AI. Một số nhánh là định hướng kiến trúc, chưa được xem là chức năng hoàn chỉnh.

### Nội dung workshop

1. [Tổng quan CampusMeet](5.1-Workshop-overview/)
2. [Mục tiêu, chuẩn bị và quyền truy cập](5.2-Prerequiste/)
3. [Kiến trúc tổng quan](5.3-Architecture/)
4. [Phạm vi chức năng hiện tại](5.4-IAM/)
5. [Quy trình cuộc họp](5.5-Authentication/)
6. [Đánh giá và định hướng](5.6-Data-foundation/)

Sáu phần trên tạo thành một câu chuyện đầy đủ về CampusMeet, kèm các bước kỹ thuật cần thiết nhưng không lặp lại toàn bộ tài liệu triển khai nội bộ.
