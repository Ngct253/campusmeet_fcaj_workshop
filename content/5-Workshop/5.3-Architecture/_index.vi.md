---
title: "Kiến trúc hệ thống"
date: 2026-07-27
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Kiến trúc hệ thống CampusMeet

## Mục tiêu

Phần này trình bày cách các thành phần của CampusMeet phối hợp với nhau, ranh giới trách nhiệm giữa giao diện, API, dữ liệu và các dịch vụ tích hợp. Việc hiểu đúng kiến trúc giúp triển khai tài nguyên theo thứ tự, tránh cấp quyền quá rộng và không nhầm lẫn giữa xác thực người dùng với quyền truy cập dữ liệu.

## Nguyên tắc thiết kế

CampusMeet tuân theo các nguyên tắc sau:

- CampusMeet quản lý quy trình trước, trong và sau cuộc họp; hệ thống không tự xây dựng chức năng gọi video và không sao chép Google Meet.
- Giao diện người dùng không truy cập trực tiếp Amazon DynamoDB.
- Amazon Cognito xác thực danh tính, còn dịch vụ phía máy chủ tiếp tục kiểm tra tư cách thành viên và vai trò.
- Dữ liệu nghiệp vụ được lưu trong năm bảng DynamoDB theo nhu cầu truy xuất.
- Tệp, âm thanh và nội dung có kích thước lớn được lưu trong Amazon S3; DynamoDB chỉ giữ thông tin mô tả và trạng thái.
- Các quy trình dài như xử lý tệp, chuyển giọng nói thành văn bản và tạo nội dung bằng AI được thực hiện bất đồng bộ.
- Mọi kết quả do AI tạo ra là bản nháp có nguồn dẫn và cần người dùng xác nhận trước khi trở thành dữ liệu chính thức.

## Luồng yêu cầu chính

![Sơ đồ kiến trúc CampusMeet AWS](images/5-Workshop/5.3-Architecture/architecture-diagram.png?v=2)

```text
Người dùng
    |
    v
CampusMeet Web
React + TypeScript + Vite
    |
    | 1. Đăng ký và đăng nhập
    v
Amazon Cognito
    |
    | 2. Phát JWT
    v
Amazon API Gateway
    |
    | 3. Kiểm tra JWT
    v
AWS Lambda
    |
    | 4. Kiểm tra thành viên và vai trò
    v
Lớp nghiệp vụ và lớp truy cập dữ liệu
    |
    +----------------------+----------------------+
    |                      |                      |
    v                      v                      v
DynamoDB               Amazon S3          Dịch vụ tích hợp
Dữ liệu nghiệp vụ      Tệp và âm thanh    Google, SES và AI
```

Một yêu cầu đọc hoặc thay đổi dữ liệu đi qua các bước:

1. Người dùng đăng nhập bằng Amazon Cognito.
2. Giao diện gửi JWT trong tiêu đề `Authorization` khi gọi API.
3. Amazon API Gateway kiểm tra chữ ký, nơi phát hành, đối tượng nhận và thời hạn của JWT.
4. AWS Lambda lấy danh tính từ JWT, không tin `userId` hoặc vai trò do giao diện tự gửi.
5. Dịch vụ phía máy chủ kiểm tra người dùng có phải thành viên đang hoạt động của nhóm và có vai trò phù hợp hay không.
6. Lớp truy cập dữ liệu thực hiện `GetItem`, `Query`, ghi có điều kiện hoặc giao dịch trên đúng bảng.
7. Kết quả được trả về theo kiểu dữ liệu dùng chung trong `@campusmeet/shared`.

## Trách nhiệm của các thành phần

| Thành phần | Trách nhiệm trong CampusMeet |
| --- | --- |
| CampusMeet Web | Hiển thị và gửi yêu cầu quản lý hồ sơ, nhóm, lời mời, cuộc họp, thông báo và các chức năng tiếp theo |
| Amazon Cognito | Đăng ký, xác nhận tài khoản, đăng nhập và phát JWT |
| Amazon API Gateway | Cung cấp HTTP API, xử lý CORS và kiểm tra JWT trước khi gọi Lambda |
| AWS Lambda | Thực thi nghiệp vụ, kiểm tra quyền và điều phối lớp truy cập dữ liệu |
| Amazon DynamoDB | Lưu dữ liệu nghiệp vụ trong năm bảng vật lý |
| Amazon S3 | Lưu tệp, âm thanh, bản ghi và nội dung lớn bằng quyền truy cập riêng tư |
| EventBridge Scheduler | Chạy lịch nhắc một lần theo thời điểm của cuộc họp |
| AWS Step Functions | Điều phối các bước xử lý dài hoặc cần thử lại có kiểm soát |
| Amazon Transcribe | Chuyển âm thanh thành văn bản khi quy trình phiên âm được thực hiện |
| Amazon Bedrock | Hỗ trợ hỏi đáp, tóm tắt và đề xuất nội dung dựa trên nguồn được phép |
| Amazon CloudWatch | Thu thập nhật ký, số liệu và thông tin lỗi |
| AWS IAM | Giới hạn quyền của người dùng và tài nguyên AWS |
| AWS SAM và CloudFormation | Định nghĩa, kiểm tra và triển khai hạ tầng bằng mã nguồn |

## Ranh giới các ngăn xếp CloudFormation

CampusMeet tách dữ liệu khỏi tài nguyên ứng dụng để giảm nguy cơ ảnh hưởng đến dữ liệu khi cập nhật mã nguồn.

| Tệp mẫu | Trách nhiệm |
| --- | --- |
| `infra/data-foundation.yaml` | Sở hữu năm bảng DynamoDB dùng chung |
| `infra/auth-integration.yaml` | Sở hữu Cognito User Pool, ứng dụng khách Cognito, HTTP API, Lambda, vai trò IAM và nhóm nhật ký của phần xác thực cùng chức năng cốt lõi |
| `infra/template.yaml` | Mô tả kiến trúc ứng dụng mở rộng và tham chiếu năm bảng qua tiền tố tên bảng; không tạo lại các bảng dữ liệu |

Thứ tự triển khai nền tảng:

```text
Kiểm tra tài khoản và quyền IAM
        ↓
Triển khai ngăn xếp dữ liệu
        ↓
Xác minh năm bảng DynamoDB
        ↓
Triển khai ngăn xếp Cognito và API
        ↓
Lấy các giá trị đầu ra CloudFormation
        ↓
Cấu hình giao diện và kiểm thử
```

## Xác thực và phân quyền

Hai lớp kiểm tra được thực hiện độc lập:

| Lớp | Nơi thực hiện | Nội dung |
| --- | --- | --- |
| Xác thực | Amazon Cognito và API Gateway | JWT hợp lệ, đúng User Pool và ứng dụng khách, chưa hết hạn |
| Phân quyền | Lambda và lớp nghiệp vụ | Người dùng là thành viên đang hoạt động, có vai trò phù hợp và được truy cập đúng nhóm hoặc cuộc họp |

JWT hợp lệ không cho phép người dùng đọc mọi nhóm. Ví dụ, trước khi trả chi tiết một nhóm, dịch vụ phía máy chủ vẫn phải đọc membership tương ứng. Các thao tác quản trị như sửa nhóm, gửi lời mời hoặc hủy cuộc họp yêu cầu vai trò `GROUP_ADMIN`.

## Ranh giới dữ liệu

CampusMeet sử dụng năm bảng:

```text
campusmeet-dev-identity
campusmeet-dev-collaboration
campusmeet-dev-meeting-data
campusmeet-dev-task-data
campusmeet-dev-ai-work
```

Các bảng sử dụng khóa `PK` và `SK`, kết hợp các chỉ mục phụ để đáp ứng nhu cầu truy xuất. Không sử dụng `Scan` cho yêu cầu nghiệp vụ thông thường.

Tệp nhị phân không được gửi xuyên qua Lambda để lưu vào DynamoDB. Quy trình dành cho tệp lớn là:

1. Giao diện yêu cầu quyền tải lên.
2. Dịch vụ phía máy chủ kiểm tra quyền, loại tệp, kích thước và thông tin liên quan.
3. Hệ thống cấp địa chỉ tải lên có thời hạn.
4. Trình duyệt tải trực tiếp lên S3 riêng tư.
5. Dịch vụ phía máy chủ xác minh tệp trước khi lưu thông tin mô tả và khởi chạy bước xử lý tiếp theo.

## Ranh giới tích hợp bên ngoài

- Google Calendar là hướng tích hợp để tạo hoặc cập nhật sự kiện và yêu cầu liên kết Google Meet.
- Google Meet REST API chỉ được dùng khi bản ghi hoặc bản phiên âm thực sự tồn tại và tài khoản có quyền phù hợp.
- Thông báo trong ứng dụng là dữ liệu chính; lỗi gửi email không được làm mất thông báo đã tạo.
- Các lệnh gọi Google, Amazon Transcribe, Amazon Bedrock hoặc dịch vụ khác cần có trạng thái, cơ chế chống xử lý lặp và chính sách thử lại.
- Dữ liệu truy xuất cho AI phải được lọc theo nhóm, cuộc họp, trạng thái phê duyệt và quyền người dùng trước khi gửi cho mô hình.

## Bảo mật và vận hành

- Lambda sử dụng vai trò thực thi IAM, không sử dụng khóa truy cập được ghi trong biến môi trường.
- Không ghi JWT, mã OAuth, mật khẩu, địa chỉ tải tệp có chữ ký hoặc toàn bộ nội dung nhạy cảm vào nhật ký.
- S3 dùng cho giao diện và nội dung người dùng phải ở chế độ riêng tư.
- Các thao tác cập nhật cạnh tranh sử dụng điều kiện phiên bản hoặc ghi có điều kiện.
- Thao tác thay đổi nhiều mục cần tính nguyên tử sử dụng giao dịch DynamoDB.
- CloudWatch được dùng để kiểm tra lỗi API, lỗi Lambda và các quy trình bất đồng bộ.
- AWS Budgets được dùng để cảnh báo chi phí của môi trường dùng chung.

## Các tệp cần đối chiếu

| Tệp | Nội dung |
| --- | --- |
| `docs/architecture.md` | Kiến trúc tổng thể và ranh giới thành phần |
| `docs/CampusMeet-SRS.md` | Yêu cầu nghiệp vụ và phạm vi hệ thống |
| `docs/dynamodb-data-model.md` | Mô hình vật lý của năm bảng DynamoDB |
| `docs/api-contract.md` | Đường dẫn API, kiểu dữ liệu và trạng thái triển khai |
| `infra/data-foundation.yaml` | Ngăn xếp dữ liệu |
| `infra/auth-integration.yaml` | Cognito, HTTP API và Lambda cốt lõi |

## Kết quả cần đạt

Sau phần này, người thực hiện cần:

- Giải thích được luồng từ giao diện đến API, Lambda và DynamoDB.
- Phân biệt xác thực JWT với phân quyền theo nhóm.
- Xác định đúng tài nguyên thuộc ngăn xếp dữ liệu và ngăn xếp ứng dụng.
- Hiểu vì sao tệp lớn nằm trong S3 thay vì DynamoDB.
- Biết các thành phần nào là tích hợp ngoài và không được giả định luôn thành công.
