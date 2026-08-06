---
title: "Nền tảng dữ liệu DynamoDB"
date: 2026-07-27
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# Nền tảng dữ liệu DynamoDB

## Mục tiêu

Phần này triển khai ngăn xếp dữ liệu độc lập của CampusMeet, xác minh đúng năm bảng DynamoDB và giải thích cách các bảng được thiết kế theo nhu cầu truy xuất thay vì tạo một bảng riêng cho từng thực thể nghiệp vụ.

## Mô hình năm bảng

Tệp `infra/data-foundation.yaml` tạo đúng năm bảng:

```text
campusmeet-dev-identity
campusmeet-dev-collaboration
campusmeet-dev-meeting-data
campusmeet-dev-task-data
campusmeet-dev-ai-work
```

| Bảng | Dữ liệu chính |
| --- | --- |
| `identity` | Hồ sơ người dùng, tùy chọn, liên kết tích hợp, trạng thái OAuth và thông báo |
| `collaboration` | Nhóm, thành viên, lời mời và sự kiện kiểm toán |
| `meeting-data` | Cuộc họp, người tham dự, nội dung họp, biên bản, nhắc lịch, tệp, bản ghi và bản phiên âm |
| `task-data` | Công việc và lịch sử thay đổi công việc |
| `ai-work` | Công việc xử lý AI, nguồn kiến thức, hội thoại, trích dẫn và đề xuất |

Việc dùng năm bảng không làm mất các thực thể nghiệp vụ. Mỗi mục được phân biệt bằng `entityType`, `PK`, `SK` và các tiền tố khóa.

Ví dụ:

```text
PK=GROUP#<groupId>       SK=META
PK=GROUP#<groupId>       SK=MEMBER#<userId>
PK=GROUP#<groupId>       SK=INVITE#<invitationId>

PK=MEETING#<meetingId>   SK=META
PK=MEETING#<meetingId>   SK=ATTENDEE#<userId>
```

## Thuộc tính chung của các bảng

Tất cả năm bảng có:

- Khóa chính `PK` và khóa sắp xếp `SK`, đều là chuỗi.
- Chế độ tính phí `PAY_PER_REQUEST`.
- Mã hóa phía máy chủ được bật.
- TTL trên thuộc tính `expiresAtEpoch`.
- Thẻ `Project=CampusMeet`, `ManagedBy=CloudFormation` và `DataModelVersion=2`.
- `DeletionPolicy: Retain` và `UpdateReplacePolicy: Retain`.
- Tùy chọn bật khôi phục theo thời điểm và bảo vệ xóa cho môi trường phù hợp.

Số chỉ mục phụ:

| Bảng | Số GSI |
| --- | ---: |
| `identity` | 2 |
| `collaboration` | 2 |
| `meeting-data` | 3 |
| `task-data` | 3 |
| `ai-work` | 2 |

## 1. Kiểm tra mẫu dữ liệu

Từ thư mục gốc CampusMeet:

```powershell
sam validate `
  --template-file infra/data-foundation.yaml `
  --lint `
  --region ap-southeast-1
```

Hoặc dùng lệnh của dự án:

```powershell
npm run sam:validate:data -- --region ap-southeast-1
```

Các tài nguyên cần xuất hiện trong mẫu:

```text
IdentityTable
CollaborationTable
MeetingDataTable
TaskDataTable
AIWorkTable
```

## 2. Tạo change set để xem trước

```powershell
sam deploy `
  --template-file infra/data-foundation.yaml `
  --stack-name campusmeet-dev-data `
  --resolve-s3 `
  --parameter-overrides `
    Environment=dev `
    TablePrefix=campusmeet-dev `
    EnablePointInTimeRecovery=false `
    EnableDeletionProtection=false `
  --no-execute-changeset `
  --region ap-southeast-1
```

Trước khi thực thi, kiểm tra:

- Đúng năm tài nguyên `AWS::DynamoDB::Table`.
- Không có thao tác xóa hoặc thay thế bảng ngoài dự kiến.
- Tên bảng bắt đầu bằng `campusmeet-dev`.
- Chế độ tính phí là `PAY_PER_REQUEST`.
- Khóa chính là `PK/SK`.
- Số GSI đúng theo bảng ở trên.
- TTL dùng `expiresAtEpoch`.
- Mã hóa được bật.
- Chính sách giữ lại dữ liệu được áp dụng.

Không thực thi nếu change set có hành động ngoài phạm vi đã kiểm tra.

## 3. Triển khai ngăn xếp dữ liệu

Sau khi review, thực thi change set trong CloudFormation Console hoặc chạy lại lệnh không có `--no-execute-changeset`:

```powershell
sam deploy `
  --template-file infra/data-foundation.yaml `
  --stack-name campusmeet-dev-data `
  --resolve-s3 `
  --parameter-overrides `
    Environment=dev `
    TablePrefix=campusmeet-dev `
    EnablePointInTimeRecovery=false `
    EnableDeletionProtection=false `
  --region ap-southeast-1
```

Nếu triển khai thất bại, đọc tab **Events** của CloudFormation trước khi thử lại. Không sửa bảng thủ công trên DynamoDB Console để né lỗi của mẫu.

## 4. Xác minh năm bảng

Lấy mã tài khoản hiện tại:

```powershell
$AccountId = aws sts get-caller-identity --query Account --output text
```

Chạy tập lệnh kiểm tra:

```powershell
powershell -NoProfile -File scripts/verify-data-foundation.ps1 `
  -Region ap-southeast-1 `
  -TablePrefix campusmeet-dev `
  -ExpectedAccountId $AccountId
```

Tập lệnh kiểm tra:

- Bảng tồn tại và ở trạng thái `ACTIVE`.
- Chế độ tính phí là `PAY_PER_REQUEST`.
- Khóa chính là `PK/SK`.
- Số lượng và tên GSI đúng.
- TTL được bật trên `expiresAtEpoch`.
- Thẻ `DataModelVersion=2` tồn tại.

Có thể dùng lệnh của dự án:

```powershell
npm run aws:verify:data -- `
  -Region ap-southeast-1 `
  -TablePrefix campusmeet-dev `
  -ExpectedAccountId $AccountId
```

## 5. Đọc đầu ra CloudFormation

```powershell
aws cloudformation describe-stacks `
  --stack-name campusmeet-dev-data `
  --query "Stacks[0].Outputs" `
  --output table `
  --region ap-southeast-1
```

Ngăn xếp xuất tên và ARN của cả năm bảng. Không chép cứng ARN hoặc mã tài khoản vào mã nguồn ứng dụng.

## 6. Quy tắc thiết kế dữ liệu

- Thời gian lưu ở UTC theo ISO 8601.
- Mục thuộc nhóm phải có `groupId` để kiểm tra quyền và kiểm toán.
- Không tin `userId`, `groupId` hoặc vai trò do giao diện gửi lên.
- Không sử dụng `Scan` trong yêu cầu nghiệp vụ thông thường.
- Dùng `Query` hoặc `GetItem` dựa trên khóa đã thiết kế.
- Dùng ghi có điều kiện để kiểm soát trạng thái, phiên bản và tính duy nhất.
- Dùng giao dịch DynamoDB khi một thao tác phải cập nhật nhiều mục theo cách nguyên tử.
- TTL chỉ dành cho dữ liệu tạm như trạng thái OAuth, lời mời hết hạn, dữ liệu chống xử lý lặp hoặc thông báo theo chính sách lưu giữ.
- Tệp, âm thanh và nội dung lớn nằm trong S3; DynamoDB chỉ lưu thông tin mô tả và tham chiếu.

## 7. Ví dụ nhu cầu truy xuất

### Nhóm và thành viên

```text
GetItem: PK=GROUP#<groupId>, SK=MEMBER#<userId>
Query:   PK=GROUP#<groupId>, begins_with(SK, "MEMBER#")
```

### Các nhóm của người dùng

Membership đặt:

```text
GSI1PK=USER#<userId>
GSI1SK=GROUP#<joinedAt>#<groupId>
```

Dịch vụ truy vấn `GSI1` thay vì quét toàn bộ bảng.

### Lời mời

Lời mời có thể được tra theo email hoặc mã băm của token:

```text
GSI1PK=EMAIL#<normalizedEmail>
GSI2PK=TOKEN#<tokenHash>
```

Token gốc không được lưu trong chỉ mục hoặc thông báo.

## 8. Ranh giới ngăn xếp và an toàn dữ liệu

Ngăn xếp dữ liệu được triển khai độc lập với Cognito và API. `infra/auth-integration.yaml` và `infra/template.yaml` chỉ nhận tiền tố hoặc tên bảng; chúng không tạo lại năm bảng.

Do bảng dùng chính sách `Retain`:

- Xóa ngăn xếp không bảo đảm bảng được xóa.
- Bảng còn lại vẫn có thể phát sinh chi phí.
- Cần kiểm tra dữ liệu và sự phụ thuộc trước khi xóa thủ công.
- Không xóa ngăn xếp dữ liệu khi thành viên khác vẫn dùng môi trường chung.

## Kết quả cần đạt

- Ngăn xếp `campusmeet-dev-data` triển khai thành công.
- Đúng năm bảng ở trạng thái `ACTIVE` trong `ap-southeast-1`.
- Khóa, GSI, TTL, thẻ và chế độ tính phí vượt qua tập lệnh xác minh.
- Không có bảng hoặc chỉ mục được tạo thủ công ngoài CloudFormation.
