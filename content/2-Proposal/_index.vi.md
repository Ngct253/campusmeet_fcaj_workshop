---
title: "Bản đề xuất"
date: 2026-06-29
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## CampusMeet — Nền tảng Serverless quản lý quy trình cuộc họp

### 1. Tóm tắt đề xuất

CampusMeet là ứng dụng web hỗ trợ nhóm học tập, đồ án và dự án nhỏ quản lý thống nhất các hoạt động trước, trong và sau cuộc họp. Hệ thống không thay thế Google Meet và không tự xây dựng chức năng gọi video; CampusMeet quản lý nhóm, lịch họp, nội dung, biên bản, công việc, tệp và quyền truy cập, đồng thời tích hợp Google Calendar/Meet khi người dùng cho phép.

Giải pháp sử dụng React/TypeScript ở phía giao diện, Amazon Cognito cho xác thực, Amazon API Gateway và AWS Lambda cho API, Amazon DynamoDB cho dữ liệu nghiệp vụ, Amazon S3 cho tệp, EventBridge Scheduler và AWS Step Functions cho xử lý bất đồng bộ, cùng Amazon Bedrock cho các chức năng AI có nguồn dẫn. Hạ tầng được quản lý bằng AWS SAM và CloudFormation.

Mục tiêu đầu-cuối của dự án:

- Đăng ký, xác nhận email và đăng nhập.
- Tạo nhóm, mời thành viên và phân quyền theo phạm vi nhóm.
- Lập lịch, cập nhật và theo dõi cuộc họp.
- Quản lý agenda, tệp đính kèm, biên bản, action item và task.
- Đồng bộ Google Calendar/Meet theo trạng thái và cơ chế thử lại.
- Quản lý transcript và nội dung AI theo phiên bản, quyền truy cập, citation và bước xác nhận.
- Có kiểm thử, giám sát, kiểm soát chi phí và quy trình dọn dẹp.

### 2. Vấn đề và phạm vi

Các nhóm nhỏ thường phải ghép ứng dụng nhắn tin, lịch, tài liệu và bảng công việc. Thông tin của một cuộc họp vì thế bị phân tán; action item dễ bị bỏ sót; quyền truy cập tệp và transcript thiếu nhất quán; tích hợp Google hoặc AI có thể tạo dữ liệu trùng hay dùng sai phiên bản.

Phạm vi cốt lõi của CampusMeet gồm:

- Xác thực bằng Cognito và bảo vệ API bằng JWT.
- Quản lý hồ sơ, nhóm, membership, lời mời và thông báo.
- Quản lý cuộc họp, attendee, agenda và vòng đời cuộc họp.
- Quản lý biên bản, quyết định, action item, task và dashboard.
- Lưu dữ liệu theo mô hình năm bảng DynamoDB.
- Upload trực tiếp lên S3 bằng presigned URL và xác minh object sau upload.
- Tích hợp Google OAuth, Calendar/Meet, reminder và Meet Add-on.
- Đọc, phân trang và chỉnh sửa transcript có kiểm soát phiên bản.
- Tạo nội dung AI dạng nháp có citation; người dùng xác nhận trước khi ghi task.

Ngoài phạm vi: xây dựng dịch vụ gọi video/WebRTC riêng; ghi âm khi chưa có consent; để AI tự thay đổi dữ liệu chính thức; đánh giá cá nhân từ dữ liệu cuộc họp; hoặc tuyên bố production-ready khi smoke test và xác minh môi trường thật chưa hoàn tất.

### 3. Trạng thái tại ngày 08/08/2026

| Hạng mục | Trạng thái đã xác minh |
| --- | --- |
| Xác thực và API nền tảng | Cognito, HTTP API, JWT Authorizer và Lambda đã có mã nguồn; stack auth/API đang hoạt động trên AWS dev |
| Dữ liệu | Năm bảng DynamoDB đã deploy và verify tại `ap-southeast-1` |
| Nhóm và lời mời | Frontend, API, repository và kiểm tra quyền đã được xây dựng |
| Chức năng cuộc họp cốt lõi | Luồng tạo, xem, cập nhật và hủy đã được triển khai ngày 06/08/2026; kiểm tra trạng thái hoạt động đạt yêu cầu, một số tình huống phân quyền vẫn cần dữ liệu kiểm thử phù hợp |
| Agenda, biên bản và task | Đã có giao diện, API, kiểm soát phiên bản và kiểm thử liên quan |
| Upload tài liệu | Presigned upload, kiểm tra metadata/checksum và tạo AIJob đã có; audio completion còn chờ worker Amazon Transcribe |
| Google Calendar/Meet | OAuth, side panel và runtime đồng bộ đã local verified; xác minh AWS và trình duyệt đầy đủ vẫn pending |
| Transcript | Đọc, phân trang và chỉnh sửa segment đã có; approval/live transcription chưa phải luồng cloud hoàn chỉnh |
| AI | Chat/draft/proposal và xác nhận task đã có theo từng vertical slice; snapshot tiến độ local verified nhưng chưa deploy đầy đủ |
| Production readiness | Chưa đạt; cần hoàn tất smoke test, browser test, alarm, retention, security/cost review và cleanup rehearsal |

Bảng này phân biệt ba mức: có mã nguồn, đã kiểm thử cục bộ và đã triển khai/xác minh trên AWS. Template hoặc tài nguyên tồn tại không tự động chứng minh toàn bộ chức năng đã hoàn thành.

### 4. Kiến trúc giải pháp

![Kiến trúc AWS của CampusMeet](images/5-Workshop/5.1-Workshop-overview/architecture-diagram.png?v=2)

#### Luồng yêu cầu đồng bộ

1. Trình duyệt tải giao diện React; kiến trúc hoàn chỉnh phân phối tài nguyên S3 private qua CloudFront.
2. Cognito xác thực và phát JWT.
3. API Gateway kiểm tra JWT trước khi chuyển request đến Lambda.
4. Lambda lấy danh tính từ token, kiểm tra membership/role và thực thi nghiệp vụ.
5. Repository truy vấn DynamoDB bằng đúng PK/SK/GSI; không dùng `Scan` cho luồng thông thường.
6. Google Calendar/Meet và SES được gọi qua adapter có trạng thái, idempotency và retry.

#### Luồng tệp và AI bất đồng bộ

1. Backend kiểm tra quyền, loại tệp, kích thước và checksum trước khi cấp URL.
2. Client upload trực tiếp lên S3 private bằng presigned URL.
3. Backend dùng `HeadObject` để xác minh object.
4. Hệ thống tạo đúng một AIJob; Step Functions và worker xử lý tác vụ dài.
5. Nguồn được phép được chuẩn hóa và ingest vào Bedrock Knowledge Base/S3 Vectors.
6. Câu trả lời hoặc bản nháp phải có citation và đi qua bước xem trước/xác nhận.

#### Mô hình năm bảng

| Bảng | Dữ liệu chính |
| --- | --- |
| `identity` | Hồ sơ, tùy chọn, trạng thái tích hợp và thông báo |
| `collaboration` | Nhóm, membership, lời mời và audit event |
| `meeting-data` | Cuộc họp, agenda, attendee, reminder, attachment, minutes và transcript |
| `task-data` | Task, lịch sử task và snapshot tiến độ |
| `ai-work` | AIJob, nguồn kiến thức, hội thoại, citation và proposal |

Tệp nhị phân nằm trong S3; vector nằm trong Bedrock Knowledge Bases/S3 Vectors. DynamoDB lưu metadata và trạng thái điều phối.

### 5. Kế hoạch triển khai

| Giai đoạn | Kết quả |
| --- | --- |
| 1. Nền tảng | Repository, shared contract, CI và cấu hình môi trường |
| 2. Danh tính | Cognito, JWT Authorizer, hồ sơ và protected route |
| 3. Cộng tác | Nhóm, membership, lời mời, thông báo và authorization |
| 4. Cuộc họp | CRUD, attendee, agenda, lifecycle và dashboard |
| 5. Sau cuộc họp | Minutes, decision, action item và task |
| 6. Tích hợp | Google OAuth/Calendar/Meet, reminder, SES và Meet Add-on |
| 7. Tệp và transcript | S3 presigned upload, AIJob và transcript version/edit/approval |
| 8. AI có nguồn dẫn | Ingestion, RAG, citation, draft và proposal confirmation |
| 9. Hoàn thiện | Smoke test, giám sát, bảo mật, chi phí, retention và cleanup |

Một chức năng chỉ hoàn thành khi có contract dùng chung, kiểm tra quyền phía server, trạng thái loading/empty/error trên giao diện, happy-path test và ít nhất một negative/security test.

### 6. Bảo mật và chất lượng

- Xác thực JWT và phân quyền nghiệp vụ là hai lớp độc lập.
- Backend không tin `userId`, `groupId`, role hoặc metadata phê duyệt do client khai báo.
- Lambda dùng IAM execution role theo nguyên tắc quyền tối thiểu.
- Không commit `.env`, credential, token, secret hoặc dữ liệu người dùng.
- S3 user-content luôn private; URL upload/download có thời hạn.
- Cập nhật cạnh tranh dùng expected version/conditional write; thay đổi nhiều item dùng transaction khi cần nguyên tử.
- Log chỉ chứa request ID, resource ID, trạng thái và mã lỗi an toàn.
- Quality gate gồm lint, typecheck, test, build, format check và SAM validation.

### 7. Chi phí, rủi ro và kết quả kỳ vọng

Chi phí phụ thuộc request API, thời gian Lambda/Step Functions, DynamoDB/S3, CloudWatch Logs, SES, Transcribe và Bedrock. Dự án sử dụng AWS Budgets, tag tài nguyên, DynamoDB `PAY_PER_REQUEST`, retention có giới hạn, change set và cleanup rehearsal để kiểm soát chi phí.

Rủi ro chính gồm thiếu kiểm tra quyền, tạo trùng sự kiện Google, AI truy xuất chéo nhóm, dùng transcript chưa duyệt, chi phí AI tăng và giữ lại tài nguyên thử nghiệm. Biện pháp giảm thiểu là authorization phía server, idempotency, filter theo group/meeting/source version, human-in-the-loop, alarm và runbook dọn dẹp.

Kết quả kỳ vọng là một luồng có thể trình diễn và kiểm chứng:

- Đăng nhập → tạo nhóm → mời thành viên.
- Lập lịch → quản lý agenda → đồng bộ Google và nhắc lịch.
- Upload tài liệu → xác minh S3 → xử lý bất đồng bộ.
- Transcript/biên bản → action item → task → dashboard.
- AI tạo câu trả lời hoặc bản nháp có citation → người dùng xem lại và xác nhận.
- Hạ tầng được quản lý bằng mã nguồn, có log, kiểm thử, cảnh báo chi phí và hướng dẫn cleanup.

Sản phẩm bàn giao không chỉ là sơ đồ kiến trúc, mà là các vertical slice có bằng chứng rõ ràng về mã nguồn, kiểm thử và trạng thái triển khai.
