---
title: "Kiến trúc tổng quan"
date: 2026-08-08
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Kiến trúc CampusMeet

![Sơ đồ kiến trúc tổng quan của CampusMeet](images/5-Workshop/5.3-Architecture/architecture-diagram.png?v=2)

Sơ đồ thể hiện các lớp từ giao diện người dùng, danh tính, xử lý nghiệp vụ và lưu trữ đến các dịch vụ hỗ trợ và theo dõi vận hành. Mỗi lớp có trách nhiệm riêng nhưng cùng phục vụ hành trình họp và theo dõi công việc.

## Dịch vụ và trách nhiệm

| Thành phần | Vai trò trong CampusMeet |
| --- | --- |
| React, TypeScript và Vite | Xây dựng giao diện web và quản lý hành trình phía người dùng |
| Amazon Cognito | Đăng ký, xác nhận tài khoản, đăng nhập và phát JWT |
| API Gateway HTTP API | Kiểm tra token trước khi chuyển yêu cầu đến backend |
| AWS Lambda | Thực thi tình huống nghiệp vụ, kiểm tra quyền và điều phối lớp truy cập dữ liệu hoặc dịch vụ tích hợp |
| Amazon DynamoDB | Lưu dữ liệu nghiệp vụ trong năm bảng vật lý |
| Amazon S3 | Lưu frontend theo kiến trúc mục tiêu và lưu tệp người dùng trong bucket riêng tư |
| EventBridge Scheduler | Kích hoạt lịch nhắc một lần tại thời điểm phù hợp |
| Step Functions và AI Worker | Điều phối các công việc dài, thử lại khi phù hợp và cập nhật trạng thái xử lý |
| Amazon Transcribe và Bedrock | Hỗ trợ phiên âm, xử lý nguồn, truy xuất và tạo nội dung có căn cứ |
| CloudWatch, SNS và SES | Theo dõi vận hành, gửi cảnh báo và thử gửi email thông báo |

## Luồng thông tin đơn giản

Khi một thành viên xem cuộc họp, CampusMeet trước hết xác nhận tài khoản và quyền tham gia nhóm. Hệ thống sau đó lấy thông tin phù hợp và hiển thị trên giao diện. Khi người dùng cập nhật biên bản hoặc nhiệm vụ, thay đổi được lưu lại để các thành viên có quyền cùng theo dõi.

Đối với tài liệu, người dùng tải tệp lên khu vực lưu trữ riêng thay vì đưa trực tiếp vào phần dữ liệu cuộc họp. CampusMeet chỉ liên kết tệp đó với đúng nhóm và cuộc họp.

Luồng của một yêu cầu nghiệp vụ có thể tóm tắt như sau:

```text
React
  → Cognito cung cấp JWT
  → API Gateway xác minh token
  → Lambda xác định người dùng và kiểm tra tư cách thành viên/vai trò
  → Application service thực hiện quy tắc
  → Lớp truy cập dữ liệu đọc hoặc ghi DynamoDB/S3
  → API trả kết quả phù hợp về giao diện
```

API Gateway xác nhận token hợp lệ nhưng không quyết định người dùng được đọc nhóm nào. Quyền trên `groupId`, `meetingId` và tài nguyên liên quan vẫn được backend kiểm tra ở từng thao tác.

## Xử lý nội dung mở rộng

Tài liệu và bản phiên âm đã được kiểm tra hoặc phê duyệt có thể trở thành nguồn cho kho tri thức, đồng thời giữ liên kết với nhóm, cuộc họp và phiên bản nguồn. Khi trợ lý AI trả lời hoặc tạo bản nháp, hệ thống giới hạn dữ liệu theo quyền người dùng, cung cấp trích dẫn và chờ xác nhận trước khi áp dụng vào biên bản hoặc nhiệm vụ chính thức.

## Ranh giới hạ tầng

CampusMeet tách hạ tầng thành các template có trách nhiệm khác nhau:

- `infra/data-foundation.yaml` quản lý đúng năm bảng DynamoDB và được tách khỏi vòng đời application để giảm rủi ro ảnh hưởng dữ liệu.
- `infra/auth-integration.yaml` là stack tích hợp Cognito, HTTP API và phạm vi core đang dùng trên môi trường phát triển; đây không phải toàn bộ kiến trúc mục tiêu.
- `infra/user-content-orchestration.yaml` sở hữu bucket user-content, orchestration, reminder, Scheduler role và cấu hình email liên quan.
- `infra/template.yaml` là ngăn xếp ứng dụng, nhận tên bảng và giá trị đầu ra cần thiết qua tham số thay vì tạo lại dữ liệu thuộc ngăn xếp khác.

Việc tách ngăn xếp giúp quá trình rà soát thay đổi rõ hơn và tránh để cập nhật giao diện/API kéo theo thay đổi ngoài ý muốn đối với dữ liệu. Tên bảng, bucket hoặc địa chỉ API được truyền qua cấu hình môi trường; thông tin xác thực không được ghi trực tiếp trong template hoặc mã nguồn.

## Trình tự thiết lập môi trường AWS

1. Xác nhận tài khoản, Region và quyền triển khai.
2. Kiểm tra template dữ liệu, xem trước thay đổi rồi triển khai và xác minh năm bảng.
3. Thiết lập user-content/orchestration nếu môi trường sử dụng upload, reminder, transcript hoặc AI.
4. Triển khai hoặc cập nhật các ngăn xếp xác thực, API và ứng dụng với đúng tên bảng cùng các giá trị đầu ra liên quan.
5. Đưa User Pool ID, User Pool Client ID và API URL vào cấu hình frontend.
6. Kiểm tra `/health`, đăng nhập, tuyến điều hướng được bảo vệ, nhật ký và các luồng được bật trong môi trường đó.

Đây là thứ tự triển khai và kiểm chứng, không phải khẳng định toàn bộ kiến trúc mục tiêu đã sẵn sàng cho môi trường thực tế. Ngăn xếp `auth-integration` phù hợp với phạm vi phát triển và chức năng lõi hiện tại; ngăn xếp ứng dụng đầy đủ vẫn cần được đánh giá bằng giá trị đầu ra, kiểm thử nhanh và nhật ký thực tế.

## Lý do lựa chọn cách tổ chức kiến trúc

Kiến trúc được chia theo trách nhiệm để luồng họp cốt lõi không bị trộn với các khả năng hỗ trợ. Giao diện tập trung vào trải nghiệm người dùng; lớp danh tính và xử lý trung tâm giữ quy tắc truy cập; dữ liệu nghiệp vụ và tệp được lưu theo đặc tính riêng; các tích hợp bên ngoài được kết nối qua ranh giới rõ ràng. Nhờ đó, thay đổi ở lịch, phiên âm hoặc AI không làm thay đổi ý nghĩa của nhóm, cuộc họp, biên bản và nhiệm vụ.

Tệp và âm thanh có kích thước lớn được lưu trong khu vực riêng tư, còn hệ thống chỉ quản lý thông tin liên kết cần thiết. Những công việc cần nhiều thời gian như xử lý âm thanh, chuẩn hóa tài liệu hoặc tạo nội dung được theo dõi theo trạng thái thay vì buộc người dùng chờ tại một màn hình. Cách làm này cần quản lý trạng thái và lỗi cẩn thận hơn, nhưng giúp luồng chính phản hồi rõ ràng và cho phép thử lại khi một dịch vụ hỗ trợ gặp sự cố.

## Các nguyên tắc kiến trúc chính

- Đăng nhập xác nhận danh tính, còn quyền trên từng nhóm và cuộc họp vẫn được kiểm tra riêng.
- Google Meet là dịch vụ họp bên ngoài; CampusMeet quản lý quy trình và kết quả, không tự xây công cụ gọi video.
- Dữ liệu cuộc họp, tệp và nội dung AI giữ liên kết với nhóm và nguồn ban đầu để có thể truy vết.
- Chỉ tài liệu hoặc bản phiên âm đã được phê duyệt theo luồng phù hợp mới được dùng làm nguồn tri thức chính thức.
- Kết quả AI là bản nháp có nguồn dẫn; thay đổi biên bản hoặc nhiệm vụ cần người dùng có quyền xác nhận.
- Lỗi, tình trạng xử lý và chi phí cần được theo dõi để chức năng nâng cao không che khuất vấn đề vận hành.
