---
title: "Workshop"
date: 2026-07-27
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Xây dựng và triển khai nền tảng quản lý cuộc họp CampusMeet trên AWS

## Tổng quan

CampusMeet là nền tảng hỗ trợ quản lý toàn bộ quy trình trước, trong và sau cuộc họp cho các nhóm học tập, đồ án và dự án quy mô nhỏ.

Hệ thống tập trung các chức năng quản lý người dùng, nhóm, thành viên, lời mời, cuộc họp, thông báo, biên bản và công việc trên một nền tảng thống nhất. CampusMeet sử dụng các dịch vụ được AWS quản lý để giảm khối lượng quản trị máy chủ, hỗ trợ mở rộng theo nhu cầu và kiểm soát chi phí theo mức sử dụng.

Trong workshop này, chúng ta sẽ xây dựng và triển khai các thành phần chính của CampusMeet, bao gồm:

- Xác thực người dùng bằng Amazon Cognito.
- Bảo vệ API bằng cơ chế xác thực JWT của Amazon API Gateway.
- Xử lý nghiệp vụ bằng AWS Lambda.
- Lưu trữ dữ liệu bằng mô hình năm bảng Amazon DynamoDB.
- Quản lý nhóm, thành viên, lời mời và thông báo.
- Quản lý cuộc họp và dữ liệu liên quan.
- Kết nối giao diện người dùng với các dịch vụ phía máy chủ.
- Quản lý hạ tầng bằng AWS SAM và AWS CloudFormation.
- Theo dõi hệ thống bằng Amazon CloudWatch.
- Áp dụng IAM, kiểm soát quyền và các nguyên tắc bảo mật.
- Kiểm thử toàn bộ luồng hoạt động.
- Kiểm soát chi phí và dọn dẹp tài nguyên.

## Bài toán

Thông tin liên quan đến cuộc họp thường bị phân tán giữa ứng dụng nhắn tin, lịch, tài liệu và công cụ quản lý công việc. Điều này khiến nhóm khó quản lý thành viên, kiểm soát quyền truy cập, theo dõi nội dung cuộc họp và duy trì các công việc sau cuộc họp.

CampusMeet giải quyết vấn đề này bằng một hệ thống tập trung, trong đó người dùng được xác thực, dữ liệu được phân quyền theo nhóm và các hoạt động trước, trong và sau cuộc họp được quản lý trong cùng một quy trình.

## Kiến trúc tổng quan

![CampusMeet AWS Architecture](images/5-Workshop/5.1-Workshop-overview/architecture-diagram.png)

CampusMeet sử dụng các dịch vụ AWS chính:

| Dịch vụ | Vai trò |
| --- | --- |
| Amazon Cognito | Xác thực và quản lý danh tính người dùng |
| Amazon API Gateway | Cung cấp HTTP API và kiểm tra JWT |
| AWS Lambda | Xử lý nghiệp vụ của hệ thống |
| Amazon DynamoDB | Lưu dữ liệu trong mô hình năm bảng |
| Amazon S3 | Lưu tệp, bản ghi âm và nội dung có kích thước lớn |
| AWS Step Functions | Điều phối các quy trình xử lý bất đồng bộ |
| Amazon Transcribe | Chuyển đổi âm thanh thành văn bản |
| Amazon Bedrock | Hỗ trợ xử lý và truy xuất nội dung bằng AI |
| Amazon CloudWatch | Ghi nhật ký, giám sát và cảnh báo |
| AWS IAM | Quản lý quyền truy cập tài nguyên |
| AWS SAM và CloudFormation | Quản lý và triển khai hạ tầng |

## Nội dung Workshop

1. [Tổng quan CampusMeet](5.1-Workshop-overview/)
2. [Điều kiện chuẩn bị](5.2-Prerequiste/)
3. [Kiến trúc hệ thống](5.3-Architecture/)
4. [IAM và cấu hình môi trường](5.4-IAM/)
5. [Xác thực với Amazon Cognito](5.5-Authentication/)
6. [Nền tảng dữ liệu DynamoDB](5.6-Data-foundation/)
7. [API Gateway và AWS Lambda](5.7-Api-lambda/)
8. [Nhóm, thành viên và lời mời](5.8-Collaboration/)
9. Quản lý cuộc họp
10. Biên bản và công việc
11. Tích hợp giao diện người dùng
12. Bản ghi nội dung cuộc họp và xử lý bất đồng bộ
13. Tích hợp AI
14. Giám sát và bảo mật
15. Kiểm thử toàn bộ hệ thống
16. Kiểm soát chi phí
17. Dọn dẹp tài nguyên
