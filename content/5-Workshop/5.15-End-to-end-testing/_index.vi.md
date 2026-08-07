---
title: "Kiểm thử toàn bộ hệ thống"
date: 2026-08-08
weight: 15
chapter: false
pre: " <b> 5.15. </b> "
---

# Kiểm thử toàn bộ hệ thống

## Mục tiêu

Phần này dùng một luồng xuyên suốt để kiểm tra CampusMeet từ trình duyệt đến Cognito, API Gateway, Lambda và DynamoDB. Mục tiêu không phải kiểm thử mọi nhánh nhỏ, mà là chứng minh các phần cốt lõi thực sự nối với nhau trên môi trường AWS.

Các kết quả trong bảng dưới đây phải được cập nhật sau khi chạy thật. Không đánh dấu `PASS` chỉ vì unit test hoặc build đã thành công.

## 1. Chuẩn bị hai người dùng

Dùng hai địa chỉ email có thể nhận mã xác nhận:

- User A: người tạo nhóm, sau đó trở thành `GROUP_ADMIN`.
- User B: người được mời, sau đó trở thành `MEMBER`.

Không dùng cùng một account cho cả hai vai trò vì như vậy không kiểm tra được authorization thực tế.

## 2. Kiểm tra public endpoint và authentication

Trước khi chạy business flow:

```text
GET /health
→ expected: 200

GET /me không có JWT
→ expected: 401
```

Trên browser:

1. User A sign up.
2. Xác nhận email.
3. Sign in.
4. Reload trang `/app`.
5. Xác nhận phiên vẫn được nhận diện.
6. Sign out.
7. Xác nhận route protected yêu cầu đăng nhập lại.

## 3. Core E2E flow

Luồng bắt buộc:

```text
A đăng nhập
   ↓
A tạo Group
   ↓
A mời B
   ↓
B đăng nhập và Accept
   ↓
A tạo Meeting
   ↓
A cập nhật Meeting
   ↓
A tạo Minutes
   ↓
A thêm Action Item gán cho B
   ↓
A Convert Action Item → Task
   ↓
B cập nhật TODO → DOING → DONE
   ↓
Dashboard của B phản ánh trạng thái
```

Nếu chuỗi này chạy ổn sau reload và dữ liệu vẫn tồn tại, core CampusMeet đã có một E2E có ý nghĩa.

## 4. Bảng kết quả core

| Bước | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
| --- | --- | --- | --- |
| Mở CloudFront/frontend URL | Trang tải thành công qua HTTPS | TBD | Chưa xác minh |
| `GET /health` | HTTP 200 | TBD | Chưa xác minh |
| API protected không JWT | HTTP 401 | TBD | Chưa xác minh |
| User A sign up + confirm | Account được xác nhận | TBD | Chưa xác minh |
| User B sign up + confirm | Account được xác nhận | TBD | Chưa xác minh |
| A tạo Group | Group được lưu, A là admin | TBD | Chưa xác minh |
| A gửi lời mời B | Invitation ở trạng thái chờ | TBD | Chưa xác minh |
| B chấp nhận | B trở thành member | TBD | Chưa xác minh |
| A tạo Meeting | Meeting xuất hiện trong list/detail | TBD | Chưa xác minh |
| A cập nhật Meeting | Dữ liệu mới còn sau reload | TBD | Chưa xác minh |
| A lưu Minutes | Minutes được lưu đúng version | TBD | Chưa xác minh |
| A thêm Action Item cho B | Action Item hiển thị đúng assignee | TBD | Chưa xác minh |
| Convert Action Item → Task | Task được tạo một lần | TBD | Chưa xác minh |
| B xem Tasks | Task vừa tạo xuất hiện | TBD | Chưa xác minh |
| B đổi `TODO → DOING → DONE` | Status lưu đúng | TBD | Chưa xác minh |
| B mở Dashboard | Counter phản ánh Task hiện tại | TBD | Chưa xác minh |

## 5. Kiểm tra authorization

Ngoài happy path, cần kiểm tra ít nhất:

- User chưa đăng nhập không vào được `/app`.
- User đã đăng nhập nhưng không thuộc Group không xem được Group detail.
- Member thường không thực hiện được thao tác chỉ dành cho admin.
- User thuộc Group A không đọc được Meeting hoặc Task của Group B.
- Version cũ khi cập nhật dữ liệu có version phải trả conflict thay vì ghi đè.

Ghi lại mã HTTP hoặc thông báo UI thực tế trong kết quả kiểm thử.

## 6. Kiểm tra persistence

Sau mỗi nhóm thao tác quan trọng, reload browser hoặc đăng xuất/đăng nhập lại.

Đặc biệt kiểm tra:

- Group còn tồn tại.
- Membership còn đúng.
- Meeting đã cập nhật vẫn đúng.
- Minutes không mất.
- Task status không quay lại trạng thái cũ.

Một giao diện hoạt động nhờ state local nhưng mất dữ liệu sau reload không được coi là E2E thành công.

## 7. Google integration — kiểm thử bổ sung

Chỉ thực hiện sau khi core E2E đã pass.

Quy trình:

1. User A mở Settings và chọn kết nối Google.
2. Hoàn tất OAuth.
3. Tạo hoặc cập nhật Meeting.
4. Quan sát Google sync status.
5. Mở Google Calendar và xác nhận event tương ứng.
6. Kiểm tra Meet URL nếu cấu hình tạo conference thành công.
7. Thử update/cancel để xem reconciliation có chạy đúng không.

Bảng kết quả:

| Bước | Mong đợi | Thực tế | Trạng thái |
| --- | --- | --- | --- |
| OAuth connect | Connection được lưu | TBD | Chưa xác minh |
| Create Meeting | Sync chuyển sang `SYNCED` | TBD | Chưa xác minh |
| Google Calendar | Event tồn tại | TBD | Chưa xác minh |
| Meet URL | URL được lưu khi Google trả về | TBD | Chưa xác minh |
| Update Meeting | Calendar event được cập nhật | TBD | Chưa xác minh |

Nếu Google lỗi nhưng core Meeting vẫn hoạt động, ghi rõ integration chưa đạt thay vì đánh dấu toàn bộ CampusMeet thất bại.

## 8. Document RAG — kiểm thử bổ sung

Dùng một file `.txt` nhỏ có nội dung dễ kiểm tra.

Ví dụ file có một thông tin duy nhất về quyết định hoặc deadline của cuộc họp.

Quy trình:

1. Upload file bằng giao diện.
2. Xác nhận upload hoàn tất.
3. Theo dõi source/job ingestion.
4. Chờ source chuyển `READY`.
5. Hỏi một câu có câu trả lời nằm trong file.
6. Kiểm tra citation trỏ đúng source.
7. Hỏi một câu không được tài liệu hỗ trợ.
8. Kiểm tra hệ thống không tạo câu trả lời khẳng định vô căn cứ.

| Bước | Mong đợi | Thực tế | Trạng thái |
| --- | --- | --- | --- |
| Upload document | Attachment hoàn tất | TBD | Chưa xác minh |
| Ingestion | Source chuyển `READY` | TBD | Chưa xác minh |
| AI question có nguồn | Có câu trả lời + citation | TBD | Chưa xác minh |
| Câu hỏi ngoài nguồn | Trả thiếu ngữ cảnh/an toàn | TBD | Chưa xác minh |

## 9. Kiểm tra log khi có lỗi

Khi một bước fail:

1. Ghi lại thời điểm.
2. Ghi request/resource ID nếu UI hoặc API trả về.
3. Mở CloudWatch log của đúng Lambda/worker.
4. Tìm error code và dependency liên quan.
5. Không copy token hoặc secret vào báo cáo.

Chỉ sửa code sau khi đã xác định lỗi nằm ở frontend, API, IAM, dữ liệu hay dịch vụ ngoài.

## 10. Điều kiện chấp nhận bản E2E cốt lõi

Bản E2E cốt lõi được xem là đạt khi:

- Frontend chạy từ URL đã triển khai, không phụ thuộc localhost.
- Cognito signup/signin thật hoạt động.
- Có ít nhất hai user thật để kiểm tra role.
- Group/invitation/membership chạy xuyên suốt.
- Meeting create/update/read chạy sau reload.
- Minutes lưu được.
- Action Item chuyển thành Task.
- Member cập nhật Task status.
- Dashboard phản ánh Task.
- Authorization cơ bản hoạt động.

Google và RAG là phần mở rộng. Nếu hai phần này chưa pass vào thời điểm nộp, kết quả phải được ghi đúng là `Chưa xác minh` hoặc `Không đưa vào core demo`.

## Kết quả cần đạt

- Có một test flow rõ ràng từ frontend đến dữ liệu thật.
- Kết quả thực tế được ghi riêng với kết quả mong đợi.
- Core và optional integration được tách biệt để dễ xác định lỗi.
- Không dùng automated test thay cho bằng chứng browser/AWS E2E.
