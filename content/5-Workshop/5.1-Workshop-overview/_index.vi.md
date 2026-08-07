---
title: "Tổng quan CampusMeet"
date: 2026-07-27
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Tổng quan CampusMeet

## Giới thiệu

CampusMeet là hệ thống quản lý cuộc họp và công việc nhóm dành cho nhóm học tập, nhóm đồ án và dự án quy mô nhỏ.

Hệ thống được thiết kế để liên kết các hoạt động trước, trong và sau cuộc họp trong cùng một không gian làm việc:

- Tạo nhóm và quản lý thành viên.
- Gửi và xử lý lời mời tham gia nhóm.
- Lập lịch, cập nhật và hủy cuộc họp.
- Quản lý nội dung họp, người tham dự và thông báo.
- Ghi nhận biên bản, quyết định và các đầu việc cần thực hiện sau cuộc họp.
- Giao công việc cho người phụ trách và theo dõi tiến độ.
- Hỗ trợ tạo bản ghi nội dung cuộc họp, hỏi đáp dựa trên nguồn được phép và tạo nội dung nháp có trích dẫn nguồn.

CampusMeet không xây dựng hệ thống gọi video riêng và không sao chép Google Meet. Google Calendar và Google Meet là các dịch vụ tích hợp bên ngoài. CampusMeet chịu trách nhiệm quản lý quy trình, dữ liệu và quyền truy cập liên quan đến cuộc họp.

## Bài toán cần giải quyết

Trong nhiều nhóm học tập và dự án nhỏ, thông tin liên quan đến cuộc họp thường nằm rải rác trên nhiều công cụ:

- Lịch họp được lưu trong ứng dụng lịch.
- Lời nhắc và trao đổi được gửi qua ứng dụng nhắn tin.
- Biên bản được lưu trong tài liệu riêng.
- Công việc sau cuộc họp được quản lý bằng một công cụ khác.
- Tài liệu, bản ghi nội dung và quyết định khó được liên kết với đúng cuộc họp.

Cách tổ chức phân tán này dẫn đến các vấn đề:

- Thành viên khó tìm lại đầy đủ thông tin của một cuộc họp.
- Các đầu việc sau cuộc họp dễ bị bỏ sót.
- Quyền truy cập không được kiểm soát thống nhất theo nhóm.
- Người quản lý khó theo dõi cuộc họp sắp tới, công việc quá hạn và tiến độ chung.
- Việc bổ sung xử lý giọng nói hoặc AI có thể làm lộ dữ liệu nếu không giới hạn đúng phạm vi truy cập.

CampusMeet giải quyết các vấn đề trên bằng một hệ thống tập trung, trong đó nhóm, cuộc họp, biên bản, công việc và nguồn tài liệu được liên kết với nhau. Mọi thao tác trên dữ liệu đều phải tuân theo tư cách thành viên và vai trò của người dùng trong nhóm.

## Mục tiêu của Workshop

Workshop trình bày quá trình xây dựng và triển khai CampusMeet trên kiến trúc AWS Serverless, từ xác thực người dùng đến lưu trữ dữ liệu, xử lý nghiệp vụ, giám sát và dọn dẹp tài nguyên.

Các mục tiêu chính gồm:

- Xây dựng đăng ký, xác nhận tài khoản và đăng nhập bằng Amazon Cognito.
- Bảo vệ HTTP API bằng cơ chế xác thực JWT của Amazon API Gateway.
- Xử lý nghiệp vụ bằng AWS Lambda.
- Kiểm tra tư cách thành viên và vai trò tại dịch vụ phía máy chủ.
- Thiết kế dữ liệu Amazon DynamoDB theo các nhu cầu truy xuất đã xác định.
- Xây dựng chức năng nhóm, thành viên, lời mời, thông báo và cuộc họp.
- Kết nối giao diện React với các dịch vụ AWS phía máy chủ.
- Quản lý hạ tầng bằng AWS SAM và AWS CloudFormation.
- Xử lý tệp, bản ghi âm và bản ghi nội dung bằng các quy trình bất đồng bộ.
- Áp dụng trích dẫn nguồn và bước xác nhận của người dùng cho nội dung do AI hỗ trợ.
- Theo dõi nhật ký, lỗi và trạng thái vận hành bằng Amazon CloudWatch.
- Áp dụng IAM theo nguyên tắc quyền tối thiểu.
- Kiểm thử toàn bộ luồng, kiểm soát chi phí và dọn dẹp tài nguyên.

## Đối tượng phù hợp

Workshop phù hợp với:

- Sinh viên muốn tìm hiểu cách xây dựng ứng dụng AWS Serverless đầu-cuối.
- Người đã có kiến thức cơ bản về giao diện web, dịch vụ phía máy chủ và cơ sở dữ liệu.
- Người muốn thực hành Amazon Cognito, API Gateway, Lambda và DynamoDB trong cùng một dự án.
- Nhóm phát triển muốn tìm hiểu xác thực, phân quyền, hạ tầng dưới dạng mã nguồn và xử lý bất đồng bộ.

## Kiến trúc tổng quan

![Kiến trúc CampusMeet AWS](images/5-Workshop/5.1-Workshop-overview/architecture-diagram.png?v=2)



## Vai trò của các thành phần chính

| Thành phần | Vai trò trong CampusMeet |
| --- | --- |
| CampusMeet Web | Cung cấp giao diện quản lý nhóm, cuộc họp, biên bản, công việc, bản ghi nội dung và các chức năng AI |
| Amazon Cognito | Xác thực người dùng và phát JWT |
| Amazon API Gateway | Cung cấp HTTP API và kiểm tra JWT trước khi gọi Lambda |
| AWS Lambda | Thực hiện nghiệp vụ và kiểm tra quyền truy cập tài nguyên |
| Amazon DynamoDB | Lưu dữ liệu nghiệp vụ trong mô hình năm bảng vật lý |
| Amazon S3 | Lưu tệp, âm thanh, bản ghi và nội dung có kích thước lớn |
| EventBridge Scheduler | Thực hiện lịch nhắc một lần theo thời điểm đã định |
| AWS Step Functions | Điều phối các quy trình xử lý dài hoặc bất đồng bộ |
| Amazon Transcribe | Chuyển đổi âm thanh thành văn bản |
| Amazon Bedrock | Hỗ trợ hỏi đáp, tóm tắt và tạo nội dung nháp dựa trên nguồn |
| Amazon CloudWatch | Thu thập nhật ký, chỉ số và thông tin vận hành |
| AWS IAM | Giới hạn quyền của người dùng và tài nguyên AWS |
| AWS SAM và CloudFormation | Định nghĩa, kiểm tra và triển khai hạ tầng |

## Mô hình dữ liệu

CampusMeet sử dụng năm bảng DynamoDB vật lý, được thiết kế theo các nhu cầu truy xuất của hệ thống:

```text
campusmeet-dev-identity
campusmeet-dev-collaboration
campusmeet-dev-meeting-data
campusmeet-dev-task-data
campusmeet-dev-ai-work
```

| Bảng | Nhóm dữ liệu chính |
| --- | --- |
| `identity` | Người dùng, tùy chọn cá nhân, tham chiếu tích hợp và thông báo |
| `collaboration` | Nhóm, thành viên, lời mời và lịch sử hoạt động |
| `meeting-data` | Cuộc họp, người tham dự, nội dung họp, biên bản, bản ghi nội dung và dữ liệu liên quan |
| `task-data` | Công việc và lịch sử thay đổi |
| `ai-work` | Tác vụ xử lý AI, nguồn kiến thức, hội thoại, trích dẫn và nội dung đề xuất |

Tệp nhị phân và âm thanh không được lưu trực tiếp trong DynamoDB. Nội dung có kích thước lớn được lưu trong Amazon S3; DynamoDB lưu thông tin mô tả và trạng thái nghiệp vụ liên quan.

## Xác thực và phân quyền

CampusMeet tách xác thực và phân quyền thành hai lớp độc lập:

### Xác thực danh tính

Amazon Cognito xác thực người dùng và phát JWT. Amazon API Gateway kiểm tra token trước khi chuyển yêu cầu đến Lambda.

### Phân quyền nghiệp vụ

JWT hợp lệ không đồng nghĩa người dùng được truy cập mọi dữ liệu. Dịch vụ phía máy chủ phải tiếp tục kiểm tra:

- Người dùng có phải thành viên đang hoạt động của nhóm hay không.
- Người dùng có vai trò phù hợp với thao tác hay không.
- Tài nguyên được yêu cầu có thuộc đúng nhóm hoặc cuộc họp hay không.
- Người dùng có quyền đọc, tạo, cập nhật hoặc hủy tài nguyên hay không.

Cách phân tách này giúp ngăn người dùng đã đăng nhập truy cập dữ liệu ngoài phạm vi được cấp quyền.

## Quy trình cuộc họp

CampusMeet được thiết kế để quản lý một chu trình hoàn chỉnh.

### Trước cuộc họp

- Tạo nhóm và quản lý thành viên.
- Lập lịch cuộc họp.
- Chọn người tham dự.
- Chuẩn bị nội dung họp và tài liệu.
- Tạo thông báo nhắc lịch.
- Đồng bộ Google Calendar và yêu cầu tạo liên kết Google Meet khi người tổ chức đã kết nối tài khoản Google.

### Trong cuộc họp

- Hiển thị thông tin cuộc họp và nội dung cần thảo luận.
- Yêu cầu sự đồng ý trước khi thu âm hoặc xử lý giọng nói.
- Tạo bản ghi nội dung cuộc họp theo thời gian thực.
- Lưu các đoạn nội dung đã được xác nhận là kết quả cuối cùng.
- Hỗ trợ hỏi đáp hoặc tóm tắt cho người vào trễ dựa trên các nguồn mà họ được phép truy cập.

### Sau cuộc họp

- Hoàn thiện biên bản.
- Ghi nhận quyết định.
- Tạo danh sách các đầu việc cần thực hiện.
- Yêu cầu người dùng xem lại và xác nhận trước khi tạo công việc chính thức.
- Theo dõi người phụ trách, thời hạn và trạng thái công việc.
- Hỗ trợ hỏi đáp trên tài liệu, bản ghi nội dung và biên bản trong phạm vi quyền của nhóm.

## Nguyên tắc đối với AI

Các chức năng AI của CampusMeet tuân theo các nguyên tắc:

- Nội dung do AI tạo ra chỉ là bản nháp cho đến khi được người dùng xác nhận.
- Câu trả lời và bản tóm tắt phải kèm trích dẫn đến nguồn hỗ trợ.
- Việc truy xuất chỉ được thực hiện trong nhóm và tập cuộc họp mà người dùng có quyền truy cập.
- Công việc hoặc thao tác do AI đề xuất phải được xem trước và xác nhận.
- Hệ thống không tự suy đoán danh tính thật của người nói.
- AI không được tự ghi thay đổi quan trọng nếu chưa kiểm tra quyền và chưa có sự chấp thuận của người dùng.

## Kết quả sau khi hoàn thành Workshop

Sau khi hoàn thành workshop, người học có thể:

- Giải thích kiến trúc tổng thể của CampusMeet.
- Triển khai các thành phần Serverless bằng AWS SAM và CloudFormation.
- Thiết lập xác thực bằng Amazon Cognito.
- Bảo vệ API bằng cơ chế xác thực JWT.
- Xây dựng Lambda theo ranh giới nghiệp vụ và lớp truy cập dữ liệu.
- Thiết kế dữ liệu DynamoDB theo nhu cầu truy xuất.
- Xây dựng các chức năng nhóm, cuộc họp và công việc.
- Kết nối giao diện người dùng với API và các dịch vụ AWS.
- Xử lý tệp và bản ghi nội dung bằng quy trình bất đồng bộ.
- Áp dụng trích dẫn nguồn và bước xác nhận của người dùng cho chức năng AI.
- Kiểm tra nhật ký, chỉ số và lỗi bằng CloudWatch.
- Áp dụng các nguyên tắc IAM, bảo vệ dữ liệu và kiểm soát chi phí.
- Thực hiện kiểm thử toàn bộ hệ thống và dọn dẹp tài nguyên.
