---
title: "Quy trình cuộc họp"
date: 2026-08-08
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Trước cuộc họp

Người phụ trách tạo cuộc họp trong nhóm, cung cấp thời gian, mục tiêu và thông tin cần thiết. Thành viên xem lịch, chuẩn bị nội dung và tiếp cận tài liệu liên quan. Nếu có tích hợp lịch, CampusMeet có thể hỗ trợ đồng bộ sự kiện và liên kết cuộc họp trực tuyến.

## Trong cuộc họp

Nhóm sử dụng nội dung đã chuẩn bị để thảo luận. Các ý kiến quan trọng, quyết định và đầu việc được ghi nhận để có thể xem lại. CampusMeet không thay thế công cụ gọi video; hệ thống là nơi tổ chức và lưu thông tin liên quan đến cuộc họp.

## Sau cuộc họp

Biên bản được rà soát để bảo đảm nội dung chính xác. Action item được chuyển thành nhiệm vụ, gắn người phụ trách, thời hạn và trạng thái. Thành viên tiếp tục cập nhật tiến độ thay vì để kết quả cuộc họp dừng ở phần ghi chú.

| Giai đoạn | Thông tin chính |
| --- | --- |
| Trước họp | Mục tiêu, thời gian, người tham gia, nội dung chuẩn bị và tài liệu |
| Trong họp | Ghi chú, quyết định và đầu việc |
| Sau họp | Biên bản, nhiệm vụ, người phụ trách, thời hạn và tiến độ |

## Tình huống minh họa xuyên suốt

Một nhóm đồ án tổ chức cuộc họp tiến độ hằng tuần. Sau khi quản trị viên tạo nhóm và thành viên chấp nhận lời mời, người phụ trách lập cuộc họp để rà soát phần việc và thống nhất công việc tiếp theo. Thành viên xem nội dung chuẩn bị, tải tài liệu lên đúng cuộc họp; lịch và liên kết Google Meet có thể được đồng bộ nếu tích hợp sẵn sàng.

Trong cuộc họp, quyết định và đầu việc được ghi nhận theo nội dung thảo luận. Bản phiên âm, nếu được sử dụng với sự đồng ý phù hợp, chỉ là nguồn hỗ trợ. Sau cuộc họp, người có quyền rà soát biên bản hoặc bản phiên âm, sửa nội dung chưa chính xác và xác nhận phiên bản phù hợp.

Đầu việc được chuyển thành nhiệm vụ có người phụ trách và thời hạn. Thành viên cập nhật trạng thái, còn nhóm có thể truy lại cuộc họp đã tạo ra nhiệm vụ. Nguồn được phê duyệt có thể hỗ trợ hỏi đáp hoặc tạo bản nháp có trích dẫn, nhưng người dùng vẫn quyết định kết quả chính thức.

## Các điểm kiểm soát trong quy trình

- **Trước họp:** mục tiêu, thời gian, nhóm sở hữu và tài liệu chuẩn bị phải được xác định trước khi gửi thông tin cho thành viên.
- **Trong họp:** ghi chú và bản phiên âm là dữ liệu hỗ trợ; việc thu nhận nội dung cần có sự đồng ý và không tự động trở thành biên bản chính thức.
- **Sau họp:** nội dung phải được rà soát trước khi xác nhận; nhiệm vụ cần có người phụ trách, thời hạn và liên kết về cuộc họp nguồn.
- **Khi dùng AI:** nguồn phải nằm trong phạm vi người dùng được phép xem, câu trả lời cần có căn cứ và mọi thay đổi nghiệp vụ vẫn chờ xác nhận.

## Cách xử lý kỹ thuật

### Upload tài liệu

Tệp không đi xuyên qua payload API hoặc được lưu trực tiếp trong DynamoDB. Luồng upload được tách thành các bước:

```text
Frontend yêu cầu upload
  → Backend kiểm tra tư cách thành viên, loại tệp, kích thước và mã kiểm tra
  → Tạo object key thuộc đúng group/meeting
  → Cấp presigned URL ngắn hạn
  → Trình duyệt tải trực tiếp tệp lên bucket S3 riêng tư
  → Backend đối chiếu object rồi lưu metadata
```

Biết khóa đối tượng không đồng nghĩa có quyền tải tệp. Khi người dùng cần đọc lại, backend tiếp tục kiểm tra quyền trước khi cấp URL có thời hạn. Nếu tệp cần xử lý tiếp, bước hoàn tất tải lên tạo một công việc có cơ chế xử lý lặp an toàn để những lần thử lại không sinh nhiều công việc cho cùng một nguồn.

### Google và reminder

Meeting bên trong CampusMeet là dữ liệu chính. Theo thiết kế, Calendar API đồng bộ sự kiện và liên kết Google Meet sau khi dữ liệu nội bộ đã được lưu; Meet REST API chỉ lấy participant, recording hoặc transcript khi artifact và OAuth scope thực tế cho phép. Nếu Google gặp lỗi, trạng thái đồng bộ được ghi nhận để thử lại nhưng Meeting không bị xóa hoặc rollback.

Thông báo trong ứng dụng cũng được xem là dữ liệu chính của luồng nhắc lịch. Reminder có thể thử gửi email qua SES, nhưng lỗi email không làm mất notification đã tạo. Adapter Google và kiểm chứng bằng tài khoản thật vẫn là phần cần tiếp tục hoàn thiện trên môi trường dùng chung.

### Vòng đời bản phiên âm

Mỗi cuộc họp có một bản phiên âm chính trong phạm vi chỉnh sửa và phê duyệt hiện tại. Partial segment chỉ phục vụ hiển thị tạm; final segment mới được lưu theo thứ tự ổn định. Sau phiên live, trạng thái chuyển qua bước hoàn tất để trở thành `READY` hoặc ghi nhận `FAILED` nếu xử lý không thành công.

Thành viên đang hoạt động có thể đọc nội dung phù hợp. Người tổ chức cuộc họp hoặc quản trị viên nhóm có quyền mới được sửa và phê duyệt. Mỗi lần sửa tăng phiên bản; thao tác phê duyệt gắn với đúng phiên bản mà người dùng đã xem. Nội dung được phê duyệt cần được đóng băng thành nguồn bất biến trước khi chuyển sang xử lý tri thức, tránh trường hợp AI đọc bản mới hơn nhưng gắn nhãn của phiên bản cũ.

### Kho tri thức và AI

Tài liệu, biên bản hoặc bản phiên âm chỉ trở thành nguồn tri thức sau bước phê duyệt phù hợp. Nguồn giữ metadata về nhóm, cuộc họp, loại nội dung, phiên bản và trạng thái. Khi truy xuất, hệ thống lọc quyền và phạm vi nguồn trước khi đưa đoạn nội dung cho mô hình.

AI có thể hỗ trợ hỏi đáp kèm trích dẫn, tóm tắt, tạo bản nháp biên bản hoặc đề xuất nhiệm vụ. Kết quả vẫn là bản nháp. Một đề xuất chỉ trở thành nhiệm vụ sau bước xem trước và xác nhận của người có quyền; cơ chế xử lý lặp an toàn và giao dịch giúp lần thử lại không tạo nhiệm vụ trùng. Các luồng này đã có mã nguồn và kiểm thử ở những phạm vi nhất định, nhưng xử lý âm thanh và kiểm chứng đầu cuối trên AWS vẫn cần tiếp tục hoàn thiện.

## Kết quả mong đợi

Một cuộc họp có kết quả rõ ràng khi thành viên xác định được nội dung đã thống nhất, người chịu trách nhiệm, thời hạn và nơi theo dõi tiến độ.
