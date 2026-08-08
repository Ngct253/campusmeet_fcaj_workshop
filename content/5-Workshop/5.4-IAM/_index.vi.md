---
title: "Cognito, API và nền tảng dữ liệu"
date: 2026-08-08
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Cấu hình Amazon Cognito

Ngăn xếp xác thực tạo một Cognito User Pool dùng email làm tên đăng nhập và tự động yêu cầu xác minh email. Chính sách mật khẩu hiện tại yêu cầu tối thiểu tám ký tự, có chữ thường, chữ hoa, số và ký hiệu. MFA đang tắt trong cấu hình phát triển hiện tại; đây là trạng thái được ghi nhận, không phải khuyến nghị bỏ MFA cho môi trường thực tế.

User Pool Client được dành cho ứng dụng web nên không sinh khóa bí mật phía trình duyệt. Client cho phép luồng SRP để đăng nhập và refresh token để duy trì phiên; tùy chọn ngăn tiết lộ sự tồn tại của người dùng cũng được bật. Sau khi xác nhận tài khoản và đăng nhập thành công, Cognito phát token để giao diện gọi API.

![Cognito User Pool của ngăn xếp xác thực CampusMeet](images/5-Workshop/campusmeet-evidence/cognito-auth-user-pool.png)

*User Pool `campusmeet-dev-auth-users` được đối chiếu từ tài nguyên vật lý của ngăn xếp `campusmeet-dev-auth`. Việc đối chiếu ID giúp tránh nhầm với User Pool khác còn tồn tại trong cùng tài khoản phát triển.*

## Cognito kết nối với API Gateway

HTTP API sử dụng JWT authorizer làm cơ chế mặc định. Authorizer kiểm tra hai thành phần chính của token:

- `issuer` phải thuộc đúng Cognito User Pool của ngăn xếp;
- `audience` phải khớp User Pool Client của ứng dụng web.

Địa chỉ `/health` không yêu cầu đăng nhập để phục vụ kiểm tra tình trạng. Các tuyến nghiệp vụ qua `/{proxy+}` sử dụng authorizer; yêu cầu `OPTIONS` được tách riêng để trình duyệt xử lý CORS. Nguồn giao diện được truyền qua tham số `AllowedOrigin`, còn phương thức và tiêu đề được giới hạn trong template.

![JWT authorizer được gắn với tuyến nghiệp vụ](images/5-Workshop/campusmeet-evidence/api-jwt-routes.png)

*Cấu hình API Gateway cho thấy tuyến `ANY /{proxy+}` được bảo vệ bằng `JWT Auth`, trong khi `/health`, callback Google và `OPTIONS` được xử lý theo mục đích riêng.*

## Từ JWT đến quyền nghiệp vụ

API Gateway loại bỏ yêu cầu có token không hợp lệ trước khi gọi Lambda. Với yêu cầu hợp lệ, Lambda lấy định danh người dùng từ token, ánh xạ với hồ sơ nội bộ rồi kiểm tra thành viên và vai trò trên tài nguyên cụ thể. Vì vậy, Cognito chịu trách nhiệm xác thực, còn phần xử lý phía máy chủ chịu trách nhiệm phân quyền.

```text
Giao diện đăng nhập với Cognito
  → nhận JWT
  → gửi tiêu đề Authorization đến HTTP API
  → API Gateway xác minh issuer và audience
  → Lambda xác định người dùng và kiểm tra quyền
  → lớp nghiệp vụ đọc hoặc cập nhật dữ liệu
```

Các lỗi xác thực, không có quyền, dữ liệu không hợp lệ và lỗi hệ thống cần được trả về theo trạng thái khác nhau để giao diện hướng dẫn người dùng đúng bước tiếp theo.

## Nền tảng DynamoDB

CampusMeet sử dụng năm bảng DynamoDB vật lý, được thiết kế theo nhóm truy vấn và ranh giới dữ liệu thay vì tạo một bảng riêng cho từng loại thực thể:

| Bảng | Dữ liệu chính |
| --- | --- |
| `identity` | Hồ sơ, tùy chọn người dùng, tham chiếu tích hợp và thông báo |
| `collaboration` | Nhóm, thành viên, lời mời và sự kiện kiểm toán |
| `meeting-data` | Cuộc họp, người tham dự, nội dung chuẩn bị, biên bản, nhắc lịch, siêu dữ liệu tệp và bản phiên âm |
| `task-data` | Nhiệm vụ, lịch sử, dữ liệu theo người phụ trách và cuộc họp |
| `ai-work` | Công việc AI, nguồn tri thức, hội thoại, trích dẫn, đề xuất và dữ liệu chống xử lý trùng |

Tệp nhị phân và âm thanh không được lưu trong DynamoDB. Tệp thật nằm trong bucket S3 riêng tư; DynamoDB chỉ giữ siêu dữ liệu và liên kết cần thiết để xác định nhóm, cuộc họp, nguồn và trạng thái.

CloudFormation tạo các bảng ở chế độ `PAY_PER_REQUEST`, bật mã hóa phía máy chủ và TTL cho dữ liệu tạm. `DeletionPolicy` cùng `UpdateReplacePolicy` được đặt là `Retain`; PITR và bảo vệ xóa có thể bật theo tham số sau khi cân nhắc môi trường và chi phí. Bảng `meeting-data` có luồng thay đổi để hỗ trợ các chức năng cần phản ứng với dữ liệu mới.

![Năm bảng DynamoDB của CampusMeet](images/5-Workshop/campusmeet-evidence/dynamodb-tables.png)

*Năm bảng dữ liệu của CampusMeet đang ở trạng thái `Active`. Ảnh đồng thời cho thấy số lượng chỉ mục và trạng thái bảo vệ xóa hiện tại để phần đánh giá không trình bày cao hơn cấu hình thực tế.*

## Luồng từ API đến dữ liệu

```text
Giao diện
  → Lớp gọi API gắn JWT
  → Bộ xử lý tiếp nhận yêu cầu
  → Dịch vụ ứng dụng kiểm tra quy tắc và quyền
  → Lớp truy cập dữ liệu thực hiện mẫu truy cập
  → DynamoDB hoặc S3
```

Giao diện không truy cập trực tiếp DynamoDB. Bộ xử lý yêu cầu cũng không tự ghép truy vấn dữ liệu cho từng trường hợp mà gọi qua dịch vụ và lớp truy cập dữ liệu; nhờ đó, quy tắc quyền, giao dịch và cách ánh xạ dữ liệu có thể được kiểm thử độc lập.

## Cộng tác và tính nhất quán

Khi tạo nhóm, người tạo trở thành `GROUP_ADMIN`; thành viên khác chỉ được thêm sau một luồng hợp lệ. Lời mời có trạng thái và thời hạn, còn thao tác chấp nhận phải liên kết đúng tài khoản nhận lời mời. Backend luôn kiểm tra tư cách thành viên đang hoạt động trước khi đọc hoặc thay đổi dữ liệu thuộc phạm vi nhóm.

Một số nguyên tắc dữ liệu được áp dụng xuyên suốt:

- yêu cầu có khả năng gửi lại sử dụng cơ chế xử lý lặp an toàn để tránh tạo kết quả trùng;
- cập nhật cạnh tranh sử dụng phiên bản hoặc thao tác ghi có điều kiện;
- thay đổi nhiều bản ghi cần tính nguyên tử sử dụng giao dịch;
- thời gian được lưu theo UTC và hiển thị theo múi giờ người dùng;
- TTL dành cho dữ liệu tạm, không thay thế quy tắc lưu giữ nội dung chính;
- yêu cầu nghiệp vụ thông thường sử dụng mẫu truy cập hoặc chỉ mục thay vì quét toàn bộ bảng.

Các nguyên tắc này giải thích phương pháp triển khai mà không cần đưa toàn bộ PK, SK, GSI hoặc biểu thức giao dịch vào workshop.

## Ý nghĩa của phần triển khai này

Cognito, API Gateway, Lambda và DynamoDB tạo thành nền tảng để các chức năng CampusMeet sử dụng chung danh tính, ranh giới quyền và cách lưu dữ liệu. Phạm vi thực tập tập trung đáng kể vào phần nền tảng này, từ mẫu hạ tầng đến kết nối xác thực và mô hình bảng. Workshop trình bày phương pháp cùng các quyết định chính, nhưng không thay thế danh mục địa chỉ API hoặc đặc tả PK, SK và GSI đầy đủ của dự án.
