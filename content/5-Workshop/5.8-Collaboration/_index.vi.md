---
title: "Nhóm, thành viên và lời mời"
date: 2026-07-27
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

# Nhóm, thành viên và lời mời

## Mục tiêu

Phần này trình bày luồng cộng tác cốt lõi của CampusMeet: tạo nhóm, liệt kê nhóm của người dùng, xem thành viên, cập nhật nhóm, gửi lời mời, chấp nhận hoặc từ chối lời mời và quản lý thông báo liên quan.

Các chức năng này sử dụng:

- Bảng `campusmeet-dev-collaboration` cho nhóm, membership, lời mời và sự kiện kiểm toán.
- Bảng `campusmeet-dev-identity` cho hồ sơ người dùng và thông báo.
- JWT Cognito để xác định người đang thực hiện yêu cầu.
- Vai trò `GROUP_ADMIN` và `MEMBER` từ `@campusmeet/shared`.

## Vai trò trong nhóm

| Vai trò | Quyền chính |
| --- | --- |
| `GROUP_ADMIN` | Xem nhóm, cập nhật thông tin nhóm, quản lý lời mời và thực hiện các thao tác quản trị |
| `MEMBER` | Xem nhóm mình đang tham gia và sử dụng các chức năng được cấp cho thành viên |

Một nhóm phải có ít nhất một Group Admin đang hoạt động. API hiện tại không cho xóa membership có vai trò `GROUP_ADMIN` qua đường dẫn xóa thành viên.

## 1. Tạo nhóm

Đường dẫn:

```http
POST /groups
```

Yêu cầu cần JWT hợp lệ. Mã nguồn tạo nhóm bằng một giao dịch DynamoDB gồm:

1. Tạo mục nhóm với khóa:

   ```text
   PK=GROUP#<groupId>
   SK=META
   ```

2. Tạo membership của người tạo với vai trò `GROUP_ADMIN`:

   ```text
   PK=GROUP#<groupId>
   SK=MEMBER#<userId>
   ```

3. Tạo sự kiện kiểm toán `GROUP_CREATED`.

`groupId` được tạo ổn định từ người dùng và `Idempotency-Key`, giúp yêu cầu gửi lại không tạo nhiều nhóm giống nhau.

Ví dụ nội dung yêu cầu:

```json
{
  "name": "Nhóm đồ án CampusMeet",
  "description": "Nhóm phối hợp phát triển và kiểm thử dự án"
}
```

Không gửi `createdBy`, `role` hoặc `userId` từ giao diện để backend tin tưởng. Các giá trị này được xác định từ JWT và quy tắc nghiệp vụ.

## 2. Liệt kê nhóm của người dùng

Đường dẫn:

```http
GET /groups
```

Membership lưu chỉ mục:

```text
GSI1PK=USER#<userId>
GSI1SK=GROUP#<joinedAt>#<groupId>
```

Repository truy vấn `GSI1` để tìm các nhóm của người dùng, sau đó lấy thông tin nhóm tương ứng. Không quét toàn bộ bảng collaboration.

Mỗi kết quả nhóm bao gồm vai trò và thời điểm tham gia của người dùng hiện tại.

## 3. Xem chi tiết nhóm và thành viên

Đường dẫn:

```http
GET /groups/:groupId
```

Backend thực hiện:

1. Lấy mục nhóm.
2. Kiểm tra membership của người xem bằng `GetItem`.
3. Từ chối nếu membership không tồn tại hoặc không còn hoạt động.
4. Truy vấn các khóa bắt đầu bằng `MEMBER#` trong partition của nhóm.
5. Lấy hồ sơ người dùng từ bảng `identity` để hiển thị tên và email khi có.

Người dùng đã đăng nhập nhưng không thuộc nhóm phải nhận phản hồi từ chối quyền, không được xem danh sách thành viên.

## 4. Cập nhật nhóm

Đường dẫn:

```http
PATCH /groups/:groupId
```

Chỉ `GROUP_ADMIN` được cập nhật tên và mô tả nhóm. Thao tác cập nhật đồng thời ghi sự kiện kiểm toán `GROUP_UPDATED`.

Ví dụ:

```json
{
  "name": "CampusMeet Project Team",
  "description": "Nhóm phát triển ứng dụng quản lý cuộc họp"
}
```

Backend kiểm tra vai trò từ membership đang lưu, không dùng vai trò do giao diện khai báo.

## 5. Xóa thành viên

Đường dẫn:

```http
DELETE /groups/:groupId/members/:userId
```

Quy tắc hiện tại:

- Yêu cầu phải do Group Admin thực hiện.
- Chỉ membership có vai trò `MEMBER` được xóa qua thao tác này.
- API từ chối xóa mọi membership có vai trò `GROUP_ADMIN`.
- Thay đổi phải được ghi nhận trong dữ liệu kiểm toán.

Quy tắc này tránh việc vô tình làm nhóm không còn quản trị viên.

## 6. Tạo lời mời

Đường dẫn:

```http
POST /groups/:groupId/invitations
```

Chỉ Group Admin được gửi lời mời.

Ví dụ nội dung:

```json
{
  "email": "member@example.com"
}
```

Repository thực hiện:

1. Chuẩn hóa email bằng cách loại khoảng trắng và chuyển về chữ thường.
2. Kiểm tra email chưa có lời mời `PENDING` còn hiệu lực cho cùng nhóm.
3. Tạo token ngẫu nhiên 32 byte.
4. Chỉ lưu mã băm SHA-256 của token trong DynamoDB.
5. Tạo lời mời có thời hạn bảy ngày.
6. Ghi sự kiện kiểm toán `MEMBERSHIP_INVITED`.
7. Nếu email đã có hồ sơ CampusMeet, tạo thông báo trong cùng giao dịch.

Khóa lời mời:

```text
PK=GROUP#<groupId>
SK=INVITE#<invitationId>
```

Các chỉ mục:

```text
GSI1PK=EMAIL#<normalizedEmail>
GSI1SK=INVITE#<expiresAt>#<invitationId>

GSI2PK=TOKEN#<tokenHash>
GSI2SK=INVITE#<invitationId>
```

Token gốc không được lưu trong notification. Thông báo dùng địa chỉ:

```text
/app/invitations?invitationId=<invitationId>
```

## 7. Trạng thái lời mời

Các trạng thái dùng chung:

| Trạng thái | Ý nghĩa |
| --- | --- |
| `PENDING` | Đang chờ phản hồi và chưa hết hạn |
| `ACCEPTED` | Người nhận đã tham gia nhóm |
| `DECLINED` | Người nhận từ chối |
| `EXPIRED` | Lời mời đã hết hạn |
| `REVOKED` | Group Admin đã thu hồi |

TTL trên `expiresAtEpoch` hỗ trợ dọn dữ liệu hết hạn, nhưng nghiệp vụ vẫn phải kiểm tra trạng thái và thời gian hết hạn trước khi xử lý.

## 8. Liệt kê và thu hồi lời mời của nhóm

Đường dẫn:

```http
GET /groups/:groupId/invitations
POST /groups/:groupId/invitations/:invitationId/revoke
```

Cả hai thao tác yêu cầu Group Admin.

Khi thu hồi:

- Trạng thái chuyển thành `REVOKED`.
- Token cũ không còn được sử dụng.
- Sự kiện kiểm toán được ghi lại.

## 9. Hộp thư lời mời của người dùng

Đường dẫn:

```http
GET /invitations
```

Backend lấy email đã xác minh của người dùng và truy vấn lời mời theo email chuẩn hóa. Người dùng chỉ thấy lời mời gửi đến email đăng nhập của mình.

CampusMeet hỗ trợ hai cách phản hồi:

### Phản hồi trong ứng dụng theo ID

```http
POST /invitations/by-id/:invitationId/accept
POST /invitations/by-id/:invitationId/decline
```

### Liên kết dự phòng dùng token

```http
GET  /invitations/:token
POST /invitations/:token/accept
POST /invitations/:token/decline
```

Backend vẫn phải đối chiếu email của người đăng nhập với email nhận lời mời. Có token không đồng nghĩa được phép chấp nhận thay người khác.

## 10. Chấp nhận lời mời

Thao tác chấp nhận cần bảo đảm lời mời:

- Đang ở trạng thái `PENDING`.
- Chưa hết hạn.
- Thuộc email của người đang đăng nhập.
- Chưa có membership hoạt động trùng lặp.

Một thao tác thành công cập nhật lời mời thành `ACCEPTED`, tạo membership vai trò `MEMBER` và ghi sự kiện kiểm toán. Các thay đổi liên quan được thực hiện bằng giao dịch để tránh trạng thái chỉ cập nhật một phần.

## 11. Từ chối lời mời

Khi từ chối:

- Lời mời chuyển từ `PENDING` sang `DECLINED`.
- Không tạo membership.
- Không thể dùng lại lời mời đã từ chối như một lời mời mới.

Group Admin cần tạo lời mời mới nếu người dùng được mời lại sau đó.

## 12. Thông báo lời mời

Khi email nhận lời mời đã có hồ sơ CampusMeet, hệ thống tạo notification loại `INVITATION` trong bảng `identity`:

```text
PK=USER#<userId>
SK=NOTIFICATION#<createdAt>#invitation-<invitationId>
```

Thông báo chưa đọc có chỉ mục:

```text
GSI2PK=USER#<userId>#UNREAD
```

Khi người nhận chấp nhận hoặc từ chối, notification `invitation-<invitationId>` tương ứng được đánh dấu đã đọc. Việc phản hồi lời mời không thất bại chỉ vì một notification cũ không còn tồn tại.

Đường dẫn quản lý thông báo:

```http
GET  /notifications
POST /notifications/:notificationId/read
```

## 13. Kiểm thử luồng cộng tác

Chuẩn bị hai tài khoản đã xác nhận:

- Tài khoản A: người tạo nhóm.
- Tài khoản B: người được mời.

Thực hiện:

1. A đăng nhập và tạo nhóm với `Idempotency-Key`.
2. A xem chi tiết nhóm và xác nhận mình là `GROUP_ADMIN`.
3. A gửi lời mời đến email của B.
4. A kiểm tra danh sách lời mời của nhóm.
5. B đăng nhập và mở hộp thư lời mời.
6. B chấp nhận lời mời.
7. B xuất hiện trong danh sách thành viên với vai trò `MEMBER`.
8. Thông báo lời mời của B chuyển sang đã đọc.
9. B thử cập nhật nhóm và phải bị từ chối.
10. A thử gửi lại một yêu cầu tạo nhóm hoặc lời mời giống hệt theo điều kiện kiểm thử để xác minh cơ chế chống trùng.

Các trường hợp lỗi cần kiểm tra:

- Người ngoài nhóm xem chi tiết nhóm.
- Member gửi lời mời.
- Mời cùng email khi còn một lời mời đang chờ.
- Chấp nhận lời mời đã hết hạn hoặc đã thu hồi.
- Chấp nhận lời mời bằng tài khoản có email khác.
- Xóa một Group Admin bằng đường dẫn xóa thành viên.

## Kết quả cần đạt

- Tạo nhóm đồng thời tạo membership Group Admin và sự kiện kiểm toán.
- Người dùng chỉ liệt kê và xem các nhóm mình đang tham gia.
- Chỉ Group Admin cập nhật nhóm, quản lý lời mời và xóa Member.
- Lời mời dùng email chuẩn hóa, token băm, thời hạn và trạng thái rõ ràng.
- Chấp nhận lời mời tạo membership bằng thao tác nguyên tử.
- Thông báo mở đúng lời mời và không chứa token gốc.
