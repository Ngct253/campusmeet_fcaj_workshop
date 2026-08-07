---
title: "IAM và cấu hình môi trường"
date: 2026-07-27
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# IAM và cấu hình môi trường

## Mục tiêu

Phần này thiết lập quyền truy cập AWS cho nhóm phát triển CampusMeet, kiểm tra đúng tài khoản và khu vực trước khi triển khai, đồng thời phân biệt quyền của người dùng với quyền của Lambda khi hệ thống vận hành.

## Mô hình trách nhiệm

CampusMeet sử dụng một tài khoản AWS phát triển dùng chung. Quyền được phân chia theo vai trò:

| Vai trò | Trách nhiệm |
| --- | --- |
| Tài khoản root | Chỉ dùng trong trường hợp bắt buộc ở cấp tài khoản; không dùng cho công việc hằng ngày |
| Người phụ trách triển khai | Quản lý các thay đổi cần quyền IAM, xem trước thay đổi CloudFormation và triển khai ngăn xếp dùng chung |
| Thành viên phát triển | Viết mã, chạy kiểm thử, đọc nhật ký và sử dụng tài nguyên phát triển trong phạm vi được cấp |
| Vai trò thực thi Lambda | Cho phép Lambda ghi nhật ký và truy cập đúng bảng hoặc dịch vụ cần thiết khi vận hành |

Không chia sẻ tài khoản IAM, mật khẩu, phiên đăng nhập hoặc khóa truy cập giữa các thành viên.

## 1. Xác nhận tài khoản phát triển

Cả nhóm cần thống nhất và ghi lại:

- Mã tài khoản AWS.
- Khu vực `ap-southeast-1`.
- Tên người phụ trách triển khai.
- Tên các ngăn xếp dùng chung.
- Tiền tố tài nguyên `campusmeet-dev`.

Các tên được dùng trong workshop:

| Thành phần | Tên |
| --- | --- |
| Ngăn xếp dữ liệu | `campusmeet-dev-data` |
| Ngăn xếp xác thực và API cốt lõi | `campusmeet-dev-auth` |
| Tiền tố năm bảng | `campusmeet-dev` |
| Nguồn được phép của giao diện cục bộ | `http://localhost:5173` |

## 2. Tạo người phụ trách triển khai

Chỉ thực hiện bước này nếu tài khoản chưa có người quản trị phù hợp:

1. Đăng nhập root một lần.
2. Tạo IAM user `campusmeet-admin` có quyền truy cập AWS Console.
3. Gắn `AdministratorAccess` cho người phụ trách triển khai.
4. Bật xác thực nhiều lớp cho tài khoản root và tài khoản quản trị.
5. Đăng xuất root và không dùng root cho các bước tiếp theo.

Không tạo thêm người quản trị nếu tài khoản đã có người giữ vai trò tương đương.

## 3. Cấp quyền cho thành viên

Người phụ trách triển khai thực hiện trong AWS Console:

1. Mở **IAM → Policies** và chọn **Create policy**.
2. Chuyển sang tab **JSON** và dán cấu hình chính sách bên dưới:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListCampusMeetTables",
      "Effect": "Allow",
      "Action": [
        "dynamodb:ListTables",
        "dynamodb:DescribeLimits"
      ],
      "Resource": "*"
    },
    {
      "Sid": "ReadWriteCampusMeetDevTables",
      "Effect": "Allow",
      "Action": [
        "dynamodb:DescribeTable",
        "dynamodb:DescribeTimeToLive",
        "dynamodb:DescribeContinuousBackups",
        "dynamodb:ListTagsOfResource",
        "dynamodb:GetItem",
        "dynamodb:BatchGetItem",
        "dynamodb:Query",
        "dynamodb:Scan",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem",
        "dynamodb:BatchWriteItem",
        "dynamodb:ConditionCheckItem"
      ],
      "Resource": [
        "arn:aws:dynamodb:ap-southeast-1:<ACCOUNT_ID>:table/campusmeet-dev-*",
        "arn:aws:dynamodb:ap-southeast-1:<ACCOUNT_ID>:table/campusmeet-dev-*/index/*"
      ]
    }
  ]
}
```

3. Đặt thông tin cho chính sách:
   - **Policy name**: `CampusMeetDevDatabaseAccess`
   - **Description**: `Read and write item access to CampusMeet dev DynamoDB tables`
4. Chọn **Create policy** để hoàn tất.
5. Mở **IAM → User groups**.
6. Tạo nhóm `CampusMeetDevelopers` nếu nhóm chưa tồn tại.
7. Gắn chính sách `CampusMeetDevDatabaseAccess` vừa tạo và `SignInLocalDevelopmentAccess` cho nhóm để thành viên có thể truy cập DynamoDB và dùng `aws login`.
8. Trong tài khoản phát triển chỉ dành cho CampusMeet, gắn `PowerUserAccess` nếu thành viên cần thao tác các dịch vụ AWS khác.
9. Tạo một IAM user riêng cho từng thành viên.
10. Bật quyền đăng nhập Console, yêu cầu đổi mật khẩu ở lần đăng nhập đầu và thêm user vào `CampusMeetDevelopers`.
11. Gửi đường dẫn đăng nhập, tên IAM user và mật khẩu tạm qua kênh riêng.

`PowerUserAccess` không cho phép quản lý IAM user, tạo mọi vai trò IAM hoặc tự do dùng `iam:PassRole`. Vì `infra/auth-integration.yaml` tạo vai trò thực thi cho Lambda, việc triển khai ngăn xếp này thuộc trách nhiệm của người phụ trách triển khai.

Không tạo access key dài hạn cho người dùng là con người.

## 4. Đăng nhập bằng AWS CLI

Mỗi thành viên sử dụng thông tin đăng nhập Console của chính mình:

```powershell
aws --version
aws login
aws sts get-caller-identity
```

Kết quả `aws sts get-caller-identity` phải hiển thị đúng phiên người dùng và mã tài khoản đã được nhóm xác nhận.

Lưu mã tài khoản để dùng trong các bước xác minh:

```powershell
$AccountId = aws sts get-caller-identity --query Account --output text
$AccountId
```

Kiểm tra khu vực đang cấu hình:

```powershell
aws configure get region
```

Các lệnh triển khai trong workshop đều chỉ rõ:

```text
ap-southeast-1
```

Không tiếp tục nếu mã tài khoản hoặc khu vực không đúng môi trường của nhóm.

## 5. Kiểm tra nhiều hồ sơ AWS

Khi máy có nhiều tài khoản AWS, sử dụng một hồ sơ riêng và dùng cùng hồ sơ trong toàn bộ lệnh:

```powershell
aws login --profile campusmeet-dev
aws sts get-caller-identity --profile campusmeet-dev
```

Khi dùng hồ sơ riêng, thêm `--profile campusmeet-dev` vào các lệnh `sam deploy`, `aws cloudformation` và tập lệnh xác minh liên quan. Không trộn hai hồ sơ trong cùng một lần triển khai.

## 6. Quyền của Lambda khi vận hành

Người dùng AWS và Lambda không dùng chung quyền.

Trong `infra/auth-integration.yaml`, Lambda nhận một vai trò thực thi riêng với các quyền chính:

- Ghi `CreateLogStream` và `PutLogEvents` vào nhóm nhật ký của chính hàm.
- Đọc và ghi dữ liệu cần thiết trong các bảng `identity`, `collaboration` và `meeting-data` cùng các chỉ mục của chúng.
- Dùng `cognito-idp:AdminGetUser` trên đúng User Pool để đọc email đã được xác minh.

Các quyền DynamoDB được giới hạn vào những hành động cần cho chức năng hiện tại như:

```text
dynamodb:GetItem
dynamodb:BatchGetItem
dynamodb:Query
dynamodb:PutItem
dynamodb:UpdateItem
dynamodb:DeleteItem
dynamodb:ConditionCheckItem
```

Không cấp `dynamodb:*` hoặc quyền trên toàn bộ bảng trong tài khoản chỉ để khắc phục lỗi nhanh.

## 7. Biến môi trường và dữ liệu nhạy cảm

Các giá trị đầu ra công khai được dùng cho giao diện:

```dotenv
VITE_COGNITO_USER_POOL_ID=<UserPoolId>
VITE_COGNITO_USER_POOL_CLIENT_ID=<UserPoolClientId>
VITE_API_BASE_URL=<ApiUrl>
```

Các giá trị `VITE_*` được đóng vào ứng dụng web nên không được chứa:

- Mật khẩu.
- JWT.
- Khóa truy cập AWS.
- Mã bí mật của ứng dụng khách.
- Mã OAuth hoặc mã làm mới.

Lambda nhận tên bảng thông qua biến môi trường do CloudFormation truyền vào. Lambda dùng vai trò IAM để truy cập AWS, không đặt khóa truy cập trong biến môi trường.

## 8. Cảnh báo ngân sách

Trước khi triển khai tài nguyên, tạo AWS Budget cho tài khoản phát triển và thêm địa chỉ email nhận cảnh báo. Cảnh báo ngân sách không chặn chi phí, nhưng giúp nhóm phát hiện tài nguyên bị giữ lại hoặc mức sử dụng tăng bất thường.

Khi kiểm tra chi phí, cần chú ý:

- CloudWatch Logs còn giữ dữ liệu sau các lần kiểm thử.
- Bảng DynamoDB có chính sách `Retain` có thể còn tồn tại sau khi xóa ngăn xếp.
- Các dịch vụ xử lý âm thanh hoặc AI có thể phát sinh chi phí theo mức sử dụng.
- Tài nguyên được tạo ở khu vực khác không xuất hiện khi chỉ xem `ap-southeast-1`.

## Kết quả cần đạt

Sau phần này, mỗi thành viên có thể đăng nhập AWS bằng phiên của riêng mình, xác minh đúng tài khoản và khu vực, đồng thời hiểu rõ tài nguyên nào chỉ người phụ trách triển khai được phép cập nhật.
