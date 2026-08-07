---
title: "Tổng quan CampusMeet"
date: 2026-07-27
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Tổng quan CampusMeet

## Giới thiệu

CampusMeet là hệ thống quản lý cuộc họp và công việc nhóm dành cho nhóm học tập, nhóm đồ án và dự án quy mô nhỏ. Mục tiêu của hệ thống là nối các hoạt động trước và sau cuộc họp thành một luồng nhất quán thay vì để lịch, biên bản, công việc và tài liệu nằm rải rác ở nhiều nơi.

Trong phiên bản hiện tại, CampusMeet tập trung vào các chức năng cốt lõi:

- Xác thực người dùng.
- Tạo nhóm và quản lý thành viên.
- Gửi và xử lý lời mời tham gia nhóm.
- Tạo, cập nhật và hủy cuộc họp.
- Lưu biên bản, quyết định và Action Item.
- Chuyển Action Item thành Task và theo dõi trạng thái công việc.
- Quản lý tệp đính kèm.
- Hỗ trợ đồng bộ Google Calendar/Meet theo cơ chế bất đồng bộ.
- Hỗ trợ các chức năng AI dựa trên nguồn được phép và citation.

CampusMeet không xây dựng một hệ thống gọi video riêng. Google Calendar và Google Meet là dịch vụ tích hợp bên ngoài; dữ liệu nghiệp vụ chính vẫn do CampusMeet quản lý.

## Bài toán cần giải quyết

Trong một nhóm nhỏ, thông tin liên quan đến cuộc họp thường bị tách thành nhiều phần:

- Lịch nằm trên ứng dụng Calendar.
- Thành viên và trao đổi nằm ở ứng dụng nhắn tin.
- Biên bản nằm trong tài liệu riêng.
- Công việc sau cuộc họp nằm ở công cụ quản lý task khác.
- Tệp và quyết định khó gắn ngược về đúng cuộc họp.

Khi các nguồn này tách rời, nhóm dễ bỏ sót đầu việc, khó kiểm soát quyền truy cập và mất thời gian tìm lại bối cảnh của một quyết định cũ.

CampusMeet giải quyết bài toán đó bằng cách liên kết Group, Meeting, Minutes, Task và nguồn tài liệu trong cùng một mô hình dữ liệu có phân quyền theo nhóm.

## Mục tiêu của Workshop

Workshop trình bày cách xây dựng và triển khai CampusMeet trên kiến trúc AWS Serverless. Nội dung đi từ xác thực, dữ liệu và API đến frontend, xử lý bất đồng bộ, AI, giám sát, kiểm thử E2E và kiểm soát chi phí.

Các mục tiêu chính:

- Thiết lập Amazon Cognito và JWT authorization.
- Xây dựng API bằng API Gateway và AWS Lambda.
- Thiết kế năm bảng DynamoDB theo nhu cầu truy xuất.
- Xây dựng luồng Group, Invitation, Meeting, Minutes, Task và Dashboard.
- Kết nối frontend React với backend.
- Dùng Amazon S3 cho tệp người dùng.
- Điều phối các công việc dài bằng EventBridge Scheduler và Step Functions.
- Tích hợp Google Calendar/Meet mà không làm phụ thuộc dữ liệu chính vào dịch vụ ngoài.
- Tích hợp Amazon Bedrock với giới hạn phạm vi dữ liệu và citation.
- Theo dõi hệ thống bằng CloudWatch và áp dụng IAM theo nguyên tắc quyền tối thiểu.
- Kiểm thử một luồng E2E thật trên môi trường AWS.
- Theo dõi và kiểm soát chi phí vận hành.

## Kiến trúc tổng quan

![Kiến trúc CampusMeet AWS](images/5-Workshop/5.1-Workshop-overview/architecture-diagram.png?v=2)

Các thành phần chính:

| Thành phần | Vai trò |
| --- | --- |
| CampusMeet Web | Giao diện React/Vite cho người dùng |
| Amazon Cognito | Đăng ký, xác nhận tài khoản và phát JWT |
| Amazon API Gateway | HTTP API và JWT authorizer |
| AWS Lambda | Xử lý nghiệp vụ và kiểm tra quyền |
| Amazon DynamoDB | Lưu dữ liệu nghiệp vụ trong năm bảng |
| Amazon S3 | Lưu tệp người dùng và nội dung lớn |
| EventBridge Scheduler | Reminder và retry bất đồng bộ |
| AWS Step Functions | Điều phối AIJob và các luồng xử lý dài |
| Amazon Bedrock | Generation, retrieval và Knowledge Base |
| Amazon CloudWatch | Log, metric và alarm |
| AWS IAM | Quyền truy cập giữa các thành phần |
| AWS SAM / CloudFormation | Định nghĩa và triển khai hạ tầng |

Amazon Transcribe vẫn là hướng mở rộng cho transcription, nhưng live transcription và batch audio transcription chưa được xem là phần bắt buộc của core production E2E hiện tại.

## Mô hình dữ liệu

CampusMeet dùng năm bảng DynamoDB:

```text
campusmeet-dev-identity
campusmeet-dev-collaboration
campusmeet-dev-meeting-data
campusmeet-dev-task-data
campusmeet-dev-ai-work
```

| Bảng | Dữ liệu chính |
| --- | --- |
| `identity` | User profile, integration references, notification |
| `collaboration` | Group, membership, invitation, audit event |
| `meeting-data` | Meeting, attendee, minutes, attachment và dữ liệu liên quan |
| `task-data` | Task và lịch sử trạng thái |
| `ai-work` | AIJob, knowledge source, conversation, citation và proposal |

Tệp nhị phân không được lưu trực tiếp trong DynamoDB. S3 giữ file; DynamoDB giữ metadata và trạng thái nghiệp vụ.

## Xác thực và phân quyền

CampusMeet tách hai khái niệm:

### Xác thực

Cognito xác nhận người dùng là ai và phát JWT. API Gateway kiểm tra token trước khi chuyển request đến Lambda.

### Phân quyền

Lambda vẫn phải kiểm tra:

- Người dùng có phải member của group không.
- Role hiện tại là gì.
- Resource có thuộc đúng group/meeting không.
- Người dùng có quyền thực hiện action đó không.

Vì vậy JWT hợp lệ không đồng nghĩa có quyền đọc mọi dữ liệu.

## Luồng nghiệp vụ chính

### Trước cuộc họp

- Tạo Group.
- Mời thành viên.
- Tạo Meeting.
- Chọn attendee và agenda.
- Tạo reminder.
- Đồng bộ Google Calendar nếu đã kết nối Google.

### Trong cuộc họp

Phiên bản workshop hiện tập trung vào việc hiển thị và quản lý thông tin cuộc họp. Các chức năng live recording/transcription được xem là phần mở rộng và chưa phải yêu cầu của core E2E.

### Sau cuộc họp

- Lưu Minutes.
- Ghi Decision.
- Tạo Action Item.
- Chuyển Action Item thành Task.
- Theo dõi Task trên Dashboard.
- Upload tài liệu và dùng làm nguồn cho AI khi ingestion đã hoàn tất.

## Nguyên tắc đối với AI

AI được dùng như công cụ hỗ trợ, không phải nguồn dữ liệu đáng tin tuyệt đối.

CampusMeet áp dụng các nguyên tắc:

- Kết quả AI quan trọng chỉ là bản nháp cho đến khi người dùng xem lại.
- Retrieval phải giới hạn trong group/meeting được phép.
- Source phải ở trạng thái phù hợp trước khi dùng.
- Citation phải được kiểm tra với dữ liệu đã truy xuất.
- AI không được tự tạo Task chính thức chỉ từ một đề xuất chưa xác nhận.

## Trạng thái triển khai và trạng thái tài liệu

Workshop phân biệt ba mức:

1. **Có trong source**: đã có implementation trong repository.
2. **Đã có automated/local test**: logic đã được kiểm tra trong test suite hoặc build local.
3. **Đã xác minh E2E trên AWS/browser**: chỉ được ghi nhận sau khi thực hiện deploy và test thật.

Cách phân biệt này giúp tài liệu phản ánh đúng trạng thái dự án, tránh biến một template hoặc unit test thành bằng chứng production đã hoạt động.

## Kết quả sau khi hoàn thành Workshop

Sau workshop, người học có thể:

- Giải thích kiến trúc tổng thể của CampusMeet.
- Triển khai các stack AWS bằng SAM/CloudFormation.
- Thiết lập Cognito và API được bảo vệ bằng JWT.
- Hiểu mô hình năm bảng DynamoDB.
- Xây dựng và kiểm thử Group, Meeting, Minutes và Task.
- Kết nối frontend với API thật.
- Hiểu cách CampusMeet xử lý upload và công việc bất đồng bộ.
- Giải thích cơ chế AI/RAG có giới hạn quyền và citation.
- Theo dõi lỗi bằng CloudWatch.
- Thực hiện một đợt E2E test và kiểm soát chi phí của môi trường.
