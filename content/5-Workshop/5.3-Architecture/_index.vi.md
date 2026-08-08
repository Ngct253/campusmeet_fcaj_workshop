---
title: "IAM, CloudFormation và cấu hình AWS"
date: 2026-08-08
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Ba lớp quyền cần phân biệt

CampusMeet sử dụng ba lớp kiểm soát có mục đích khác nhau. Việc trình bày tách biệt giúp không nhầm đăng nhập với quyền sử dụng dữ liệu hoặc quyền của dịch vụ AWS.

| Lớp | Trách nhiệm |
| --- | --- |
| Amazon Cognito | Xác nhận danh tính người dùng và phát JWT cho phiên đăng nhập |
| Phân quyền ứng dụng | Backend kiểm tra tư cách thành viên, vai trò, nhóm, cuộc họp và thao tác được phép |
| AWS IAM | Quy định Lambda hoặc dịch vụ AWS nào được gọi API và truy cập tài nguyên nào |

Người dùng đăng nhập thành công không nhận quyền IAM để truy cập DynamoDB. Giao diện gửi JWT đến API; Lambda sử dụng vai trò thực thi của chính nó để đọc hoặc ghi dữ liệu sau khi backend đã kiểm tra quyền nghiệp vụ.

## IAM cho ngăn xếp xác thực và API

Mẫu `infra/auth-integration.yaml` tạo một vai trò thực thi riêng cho Lambda xác thực/API. Chính sách tin cậy chỉ cho dịch vụ Lambda đảm nhận vai trò này. Chính sách gắn với vai trò giới hạn vào những nhu cầu hiện tại:

- ghi nhật ký vào nhóm nhật ký của Lambda;
- đọc và ghi các thao tác DynamoDB cần thiết trên bảng `identity`, `collaboration`, `meeting-data` và các chỉ mục tương ứng;
- đọc email đã xác minh từ đúng Cognito User Pool bằng `AdminGetUser`.

Vai trò không dùng `AdministratorAccess` và không cấp quyền trực tiếp cho trình duyệt. Các tiến trình xử lý hoặc chức năng điều phối mở rộng sử dụng vai trò riêng để phạm vi S3, Scheduler hoặc dịch vụ AI không bị trộn vào Lambda của luồng xác thực cốt lõi.

## Phân quyền trong ứng dụng

Sau khi API Gateway xác minh JWT, mỗi thao tác vẫn được backend xem xét theo bốn câu hỏi:

1. Tài khoản đã đăng nhập và còn hợp lệ hay chưa?
2. Người đó có phải thành viên đang hoạt động của nhóm hay không?
3. Vai trò hiện tại có cho phép hành động cụ thể hay không?
4. Dữ liệu có đúng nhóm, cuộc họp và phiên bản cần thao tác hay không?

Đăng nhập hợp lệ không cho phép đọc dữ liệu nhóm khác, còn quyền xem cuộc họp không mặc nhiên cho phép sửa hoặc phê duyệt nội dung. Tài liệu, bản phiên âm, biên bản, nhiệm vụ và nguồn dùng cho AI đều kế thừa ranh giới truy cập của nhóm và cuộc họp gốc.

## Ranh giới hạ tầng

CampusMeet tách hạ tầng thành các mẫu có trách nhiệm khác nhau:

- `infra/data-foundation.yaml` quản lý năm bảng DynamoDB và được tách khỏi vòng đời ứng dụng để giảm rủi ro ảnh hưởng dữ liệu.
- `infra/auth-integration.yaml` tạo Cognito, HTTP API, vai trò IAM, Lambda và nhóm nhật ký cho phạm vi xác thực/API đang dùng trên môi trường phát triển.
- `infra/user-content-orchestration.yaml` sở hữu bucket user-content, điều phối, nhắc lịch, vai trò Scheduler và cấu hình email liên quan.
- `infra/template.yaml` là ngăn xếp ứng dụng, nhận tên bảng và đầu ra cần thiết qua tham số thay vì tạo lại tài nguyên thuộc ngăn xếp khác.

Việc tách ngăn xếp giúp quá trình rà soát thay đổi rõ hơn và tránh để cập nhật giao diện/API kéo theo thay đổi ngoài ý muốn đối với dữ liệu. Tên bảng, bucket hoặc địa chỉ API được truyền qua tham số và đầu ra CloudFormation; thông tin xác thực không được ghi trực tiếp trong mẫu hoặc mã nguồn.

Tại lần kiểm tra ngày 08/08/2026, các ngăn xếp dữ liệu, xác thực và user-content ở trạng thái `UPDATE_COMPLETE`. Ngăn xếp ứng dụng `campusmeet-dev-app` vẫn quản lý frontend, API ứng dụng, worker Google, AI Worker, Knowledge Base và giám sát bằng SAM/CloudFormation, nhưng trạng thái hiện tại là `UPDATE_ROLLBACK_FAILED` do tài nguyên `ApiLambdaRole`. Các tài nguyên AI đã tạo vẫn hoạt động ở control plane; tuy nhiên nhóm cần phục hồi stack và xem lại change set trước lần triển khai tiếp theo. Vì vậy workshop không ghi nhận cả bốn ngăn xếp đều hoàn tất.

![Các ngăn xếp CloudFormation của CampusMeet ở trạng thái hoàn tất](images/5-Workshop/campusmeet-evidence/cloudformation-stacks.png)

*Ba ngăn xếp ổn định của CampusMeet đã được cập nhật thành công trên môi trường phát triển. Ngăn xếp ứng dụng được đánh giá riêng vì đang cần phục hồi trạng thái rollback.*

## Cấu trúc ngăn xếp xác thực/API

Ngăn xếp xác thực/API nhận hai tham số chính: `AllowedOrigin` giới hạn nguồn giao diện được phép gọi API và `DataTablePrefix` xác định đúng bộ bảng của môi trường. Từ đó CloudFormation quản lý các tài nguyên có liên hệ với nhau:

| Nhóm tài nguyên | Nội dung |
| --- | --- |
| Danh tính | Cognito User Pool và User Pool Client cho ứng dụng web |
| Điểm vào API | API Gateway HTTP API với JWT authorizer và CORS |
| Xử lý | Lambda Node.js cho địa chỉ kiểm tra tình trạng và các tuyến được bảo vệ |
| Quyền | Vai trò IAM giới hạn vào nhật ký, các bảng cần thiết và User Pool |
| Quan sát | CloudWatch Log Group với thời gian lưu giữ xác định |

Sau khi triển khai, ngăn xếp xuất `UserPoolId`, `UserPoolClientId`, `ApiUrl` và tên ba bảng được dùng trong phạm vi này. Giao diện nhận các giá trị công khai cần thiết, còn Lambda nhận tên bảng và User Pool ID qua biến môi trường phía máy chủ.

![Tài nguyên của ngăn xếp xác thực và API](images/5-Workshop/campusmeet-evidence/cloudformation-auth-resources.png)

*Ngăn xếp xác thực/API quản lý tập trung Lambda, vai trò IAM, API Gateway, Cognito User Pool, User Pool Client và CloudWatch Log Group. Danh sách này cũng giúp đối chiếu đúng tài nguyên đang được sử dụng thay vì suy luận từ tên hiển thị.*

## Nguyên tắc CloudFormation và kiểm tra thay đổi

- Tài nguyên được khai báo trong mẫu để môi trường có thể tái tạo và rà soát bằng lịch sử Git.
- Tham số mô tả khác biệt giữa môi trường; đầu ra cung cấp định danh cần thiết cho ngăn xếp hoặc giao diện khác.
- Vòng đời dữ liệu được tách khỏi vòng đời ứng dụng và sử dụng chính sách bảo vệ phù hợp với từng môi trường.
- Thay đổi mẫu được kiểm tra trước, xem trước phạm vi cập nhật rồi mới triển khai vào đúng tài khoản và Region.
- Sau triển khai cần đối chiếu trạng thái ngăn xếp, đầu ra, tài nguyên thật và CloudWatch Logs thay vì chỉ dựa vào việc lệnh triển khai kết thúc.

Cách tổ chức này cho thấy IAM và CloudFormation không phải phần cấu hình phụ. Chúng tạo ranh giới bảo mật, kết nối các thành phần và giúp việc triển khai xác thực/API cùng nền tảng dữ liệu có thể kiểm chứng và lặp lại.
