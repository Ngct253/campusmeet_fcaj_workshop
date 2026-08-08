---
title: "Workshop"
date: 2026-08-08
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

## Giới thiệu CampusMeet

CampusMeet là nền tảng hỗ trợ nhóm học tập và nhóm dự án quản lý thông tin trước, trong và sau cuộc họp tại một nơi. Thay vì tách lịch họp, tài liệu, biên bản và công việc sang nhiều công cụ, CampusMeet liên kết chúng theo từng nhóm và từng cuộc họp.

Workshop trình bày CampusMeet theo cả góc nhìn sản phẩm và triển khai. Bên cạnh hành trình cuộc họp, nội dung làm rõ phần xác thực và nền tảng dữ liệu đã được thực hiện trong phạm vi thực tập: IAM, CloudFormation, Amazon Cognito, API Gateway, Lambda và DynamoDB. Chi tiết được chọn lọc để người đọc hiểu cách hệ thống được xây dựng nhưng không thay thế tài liệu API hoặc runbook đầy đủ.

### Mục tiêu

Sau khi đọc workshop, người xem có thể:

- Hiểu vấn đề và mục tiêu mà CampusMeet hướng đến.
- Nắm được quy trình từ khi tạo nhóm đến khi kết thúc cuộc họp và theo dõi công việc.
- Hiểu vai trò của các thành phần AWS và ranh giới giữa các ngăn xếp triển khai.
- Nắm được cách IAM, CloudFormation, Cognito, API và DynamoDB phối hợp trong luồng xác thực và lưu trữ.
- Nắm được cách chuẩn bị môi trường, kết nối frontend với AWS và kiểm tra chất lượng trước khi triển khai.
- Phân biệt phần đã có, phần cần kiểm chứng thêm và định hướng phát triển.
- Nhận biết nguyên tắc về quyền truy cập, bảo vệ thông tin và kiểm soát nội dung do AI đề xuất.

### Kiến trúc tổng quan

![Kiến trúc tổng quan của CampusMeet](images/5-Workshop/5.1-Workshop-overview/architecture-diagram.png?v=2)

Sơ đồ mô tả một luồng đơn giản: người dùng truy cập CampusMeet, đăng nhập, thực hiện thao tác qua hệ thống trung tâm, sau đó thông tin được lưu trữ và có thể được hỗ trợ bởi lịch, thông báo, phiên âm và AI. Một số nhánh là định hướng kiến trúc, chưa được xem là chức năng hoàn chỉnh.

### Nội dung workshop

1. [Tổng quan và phạm vi CampusMeet](5.1-Workshop-overview/)
2. [Chuẩn bị môi trường và kiến trúc triển khai](5.2-Prerequiste/)
3. [IAM, CloudFormation và cấu hình AWS](5.3-Architecture/)
4. [Cognito, API và nền tảng dữ liệu](5.4-IAM/)
5. [Frontend, quy trình cuộc họp và tích hợp](5.5-Authentication/)
6. [Kiểm chứng, vận hành và đánh giá](5.6-Data-foundation/)

Sáu phần trên giữ mạch trình bày gọn, đồng thời làm nổi bật phần xác thực và DynamoDB đã được tham gia triển khai. Các tích hợp mở rộng vẫn được trình bày đúng trạng thái, không nhận định cao hơn bằng chứng hiện có.
