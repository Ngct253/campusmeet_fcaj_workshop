---
title: "Phạm vi chức năng hiện tại"
date: 2026-08-08
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Tổng quan chức năng

| Nhóm chức năng | Phạm vi | Trạng thái tổng quát |
| --- | --- | --- |
| Tài khoản và hồ sơ | Đăng ký, đăng nhập và quản lý thông tin cá nhân | Đã có nền tảng chính |
| Nhóm và cộng tác | Nhóm, thành viên, lời mời và thông báo | Đã có các luồng chính |
| Quản lý cuộc họp | Tạo, xem, cập nhật, hủy và chuẩn bị nội dung cuộc họp | Chức năng cốt lõi đã có; một số tình huống phân quyền vẫn cần kiểm chứng |
| Nội dung và công việc sau họp | Tài liệu, biên bản, chỉnh sửa và phê duyệt bản phiên âm, action item, nhiệm vụ và theo dõi tiến độ | Đã có nhiều phần trong giao diện và luồng xử lý; cần tiếp tục kiểm thử xuyên suốt |
| Tích hợp và tự động hóa | Đồng bộ Google Calendar/Meet, upload nội dung, nhắc lịch và email | Đã có các luồng ban đầu; một số phần mới được kiểm chứng cục bộ hoặc vẫn cần xác minh trên môi trường thực tế |
| Kho tri thức và trợ lý AI | Tiếp nhận nguồn đã được phê duyệt, hỏi đáp có trích dẫn, tóm tắt nội dung, tạo bản nháp biên bản/nhiệm vụ và phân tích tiến độ nhóm | Đã có một số luồng và kiểm thử liên quan; chưa được xem là hoàn chỉnh và mọi nội dung đề xuất vẫn cần người dùng xác nhận |

## Yêu cầu xuyên suốt

Bất kể thuộc nhóm chức năng nào, một luồng cần xác định rõ người thao tác, dữ liệu đầu vào, kết quả tạo ra và bước tiếp theo. Quyền phải được kiểm tra trước khi hiển thị hoặc thay đổi nội dung. Trạng thái bản nháp, nội dung đã xác nhận và công việc đã hoàn thành cần được phân biệt để thành viên không hiểu nhầm. Khi một nguồn dữ liệu hoặc tích hợp gặp lỗi, giao diện cần cho biết điều gì chưa sẵn sàng và vẫn giữ được phần còn lại của hành trình nếu có thể.

## Hành trình xuyên suốt giữa các chức năng

Tài khoản xác định người thao tác, tư cách thành viên đặt người đó vào đúng nhóm, còn cuộc họp tạo ngữ cảnh cho tài liệu và nội dung chuẩn bị. Biên bản và đầu việc sau họp tiếp tục thành nhiệm vụ có người phụ trách, thời hạn và trạng thái. Tài liệu hoặc bản phiên âm được phê duyệt theo luồng phù hợp mới đi tiếp vào kho tri thức; kết quả AI vẫn quay lại bước xác nhận của người dùng. Nếu lịch hoặc AI tạm thời chưa sử dụng được, nhóm vẫn phải xem được cuộc họp, ghi nhận kết quả và theo dõi nhiệm vụ.

## Thông tin được quản lý

CampusMeet sử dụng năm bảng DynamoDB vật lý, được thiết kế theo nhóm truy vấn và ranh giới dữ liệu thay vì tạo một bảng riêng cho từng entity:

| Bảng | Dữ liệu chính |
| --- | --- |
| `identity` | Hồ sơ, tùy chọn người dùng, tham chiếu tích hợp và thông báo |
| `collaboration` | Nhóm, thành viên, lời mời và sự kiện kiểm toán |
| `meeting-data` | Cuộc họp, người tham dự, agenda, biên bản, reminder, metadata tệp và transcript |
| `task-data` | Nhiệm vụ, lịch sử, dữ liệu theo người phụ trách và cuộc họp |
| `ai-work` | AI job, nguồn tri thức, hội thoại, citation, proposal và idempotency |

Tệp nhị phân và âm thanh không được lưu trong DynamoDB. Tệp thật nằm trong bucket S3 riêng tư; DynamoDB chỉ giữ siêu dữ liệu và liên kết cần thiết để xác định nhóm, cuộc họp, nguồn và trạng thái.

## Luồng xử lý dữ liệu

```text
Giao diện
  → API client gắn JWT
  → Handler tiếp nhận request
  → Application service kiểm tra quy tắc và quyền
  → Lớp truy cập dữ liệu thực hiện mẫu truy cập
  → DynamoDB hoặc S3
```

Giao diện không truy cập trực tiếp DynamoDB. Bộ xử lý yêu cầu cũng không tự ghép truy vấn dữ liệu cho từng trường hợp mà gọi qua dịch vụ và lớp truy cập dữ liệu; nhờ đó, quy tắc quyền, giao dịch và cách ánh xạ dữ liệu có thể được kiểm thử độc lập.

## Cộng tác và tính nhất quán

Khi tạo nhóm, người tạo trở thành `GROUP_ADMIN`; thành viên khác chỉ được thêm sau một luồng hợp lệ. Lời mời có trạng thái và thời hạn, còn thao tác chấp nhận phải liên kết đúng tài khoản nhận lời mời. Backend luôn kiểm tra tư cách thành viên đang hoạt động trước khi đọc hoặc thay đổi dữ liệu thuộc phạm vi nhóm.

Một số nguyên tắc dữ liệu được áp dụng xuyên suốt:

- request có khả năng gửi lại sử dụng idempotency để tránh tạo kết quả trùng;
- cập nhật cạnh tranh sử dụng version hoặc conditional write;
- thay đổi nhiều item cần tính nguyên tử sử dụng transaction;
- timestamp được lưu theo UTC và hiển thị theo timezone người dùng;
- TTL dành cho dữ liệu tạm, không thay thế quy tắc lưu giữ nội dung chính;
- request nghiệp vụ thông thường sử dụng access pattern/index thay vì quét toàn bộ bảng.

Các nguyên tắc này giải thích phương pháp triển khai mà không cần đưa toàn bộ PK, SK, GSI hoặc transaction expression vào workshop.

## Nguyên tắc tổng kết

Một màn hình xuất hiện không đồng nghĩa toàn bộ chức năng phía sau đã hoàn chỉnh. Một luồng chạy được cục bộ cũng chưa chắc đã được kiểm chứng đầy đủ trên môi trường dùng chung. Báo cáo vì vậy chỉ ghi nhận đúng mức đã quan sát và nêu rõ phần còn lại.
