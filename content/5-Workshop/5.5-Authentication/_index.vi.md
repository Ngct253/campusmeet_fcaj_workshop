---
title: "Frontend, quy trình cuộc họp và tích hợp"
date: 2026-08-08
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Frontend kết nối với xác thực và API

Giao diện React sử dụng cấu hình môi trường đã chuẩn bị ở phần 5.2 để kết nối đúng Cognito và API. Module xác thực quản lý đăng ký, xác nhận tài khoản, đăng nhập và khôi phục phiên; lớp gọi API gắn JWT tập trung thay vì để từng màn hình tự xử lý token.

Tuyến điều hướng được bảo vệ yêu cầu phiên hợp lệ. Lỗi chưa đăng nhập cần được phân biệt với lỗi đã đăng nhập nhưng không có quyền; sau khi tải lại trang, giao diện phải khôi phục phiên và dữ liệu từ nguồn thật thay vì chỉ dựa vào trạng thái tạm trong trình duyệt. Phần này tập trung vào hành vi của giao diện; quá trình xác minh JWT và phân quyền phía máy chủ đã được giải thích tại phần 5.4.

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

![Biên bản, quyết định và việc sau cuộc họp trong CampusMeet](images/5-Workshop/campusmeet-evidence/meeting-minutes.png)

*Màn hình biên bản liên kết phần tóm tắt, quyết định và việc sau cuộc họp. Nội dung trong ảnh là dữ liệu minh họa của môi trường phát triển, không phải mẫu nội dung chuẩn cho mọi cuộc họp.*

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

Trên môi trường phát triển, bucket `campusmeet-uploads-dev` đã bật chặn toàn bộ truy cập công khai và mã hóa mặc định bằng khóa do Amazon S3 quản lý. Đây là lớp bảo vệ ở mức lưu trữ; backend vẫn phải kiểm tra quyền trước khi cấp URL tải lên hoặc tải xuống.

![Bucket upload của CampusMeet đã chặn truy cập công khai](images/5-Workshop/campusmeet-evidence/s3-block-public-access.png)

*Block Public Access đang bật cho bucket chứa nội dung người dùng.*

![Mã hóa mặc định của bucket upload](images/5-Workshop/campusmeet-evidence/s3-default-encryption.png)

*Đối tượng mới được mã hóa phía máy chủ bằng SSE-S3. Ảnh cấu hình không thay thế kiểm tra quyền truy cập ở lớp ứng dụng.*

### Google và reminder

Meeting bên trong CampusMeet là dữ liệu chính. Theo thiết kế, Calendar API đồng bộ sự kiện và liên kết Google Meet sau khi dữ liệu nội bộ đã được lưu; Meet REST API chỉ lấy participant, recording hoặc transcript khi artifact và OAuth scope thực tế cho phép. Nếu Google gặp lỗi, trạng thái đồng bộ được ghi nhận để thử lại nhưng Meeting không bị xóa hoặc rollback.

Thông báo trong ứng dụng cũng được xem là dữ liệu chính của luồng nhắc lịch. Reminder có thể thử gửi email qua SES, nhưng lỗi email không làm mất notification đã tạo. Adapter Google và kiểm chứng bằng tài khoản thật vẫn là phần cần tiếp tục hoàn thiện trên môi trường dùng chung.

![Thông báo nhắc cuộc họp trên CampusMeet](images/5-Workshop/campusmeet-evidence/reminder-notification.png)

*Thông báo “Sắp đến giờ họp” xác nhận reminder đã xuất hiện trong giao diện. Ảnh này không tự chứng minh email SES hoặc mọi lần kích hoạt lịch đều thành công.*

Một lần kiểm tra có kiểm soát đã tạo cuộc họp và ghi yêu cầu đồng bộ Google. Giao diện sau đó chuyển sang trạng thái cần kết nối lại vì tài khoản thử nghiệm chưa nằm trong danh sách OAuth Test users, nên chưa sinh Google Meet URL. Kết quả này cho thấy hệ thống đã nhận biết lỗi tích hợp và giữ cuộc họp nội bộ, nhưng chưa được xem là luồng Google hoàn chỉnh.

![Chi tiết cuộc họp và trạng thái cần kết nối lại Google](images/5-Workshop/campusmeet-evidence/google-reconnect-state.png)

*Cuộc họp vẫn tồn tại trong CampusMeet khi đồng bộ Google chưa hoàn tất, cho phép người dùng biết nguyên nhân và thực hiện bước kết nối lại.*

### Vòng đời bản phiên âm

Mỗi cuộc họp có một bản phiên âm chính trong phạm vi chỉnh sửa và phê duyệt hiện tại. Partial segment chỉ phục vụ hiển thị tạm; final segment mới được lưu theo thứ tự ổn định. Sau phiên live, trạng thái chuyển qua bước hoàn tất để trở thành `READY` hoặc ghi nhận `FAILED` nếu xử lý không thành công.

Thành viên đang hoạt động có thể đọc nội dung phù hợp. Người tổ chức cuộc họp hoặc quản trị viên nhóm có quyền mới được sửa và phê duyệt. Mỗi lần sửa tăng phiên bản; thao tác phê duyệt gắn với đúng phiên bản mà người dùng đã xem. Nội dung được phê duyệt cần được đóng băng thành nguồn bất biến trước khi chuyển sang xử lý tri thức, tránh trường hợp AI đọc bản mới hơn nhưng gắn nhãn của phiên bản cũ.

### Kho tri thức và AI

Tài liệu, biên bản hoặc bản phiên âm chỉ trở thành nguồn tri thức sau bước phê duyệt phù hợp. Nguồn giữ metadata về nhóm, cuộc họp, loại nội dung, phiên bản và trạng thái. Khi truy xuất, hệ thống lọc quyền và phạm vi nguồn trước khi đưa đoạn nội dung cho mô hình.

Trong môi trường dev, phần truy xuất và sinh nội dung được tách thành hai Region. Nguồn đã duyệt được chuẩn hóa dưới prefix `kb/` trong S3 tại Singapore; Bedrock Knowledge Base sử dụng Cohere Embed Multilingual v3 và S3 Vectors để lập chỉ mục, còn AI Worker được cấu hình gọi Bedrock Mantle tại N. Virginia bằng model `openai.gpt-oss-20b`. Chỉ context đã qua kiểm tra quyền mới được gửi sang bước sinh nội dung.

```text
AIJob trong DynamoDB
  → Step Functions quản lý trạng thái và lần thử lại
  → AI Worker chuẩn hóa hoặc truy xuất nguồn đã duyệt
  → Knowledge Base và S3 Vectors tìm đoạn nội dung phù hợp
  → kiểm tra group, meeting, source và version
  → Bedrock Mantle sinh câu trả lời hoặc bản nháp
  → giao diện hiển thị nguồn để người dùng xem và xác nhận
```

![State machine điều phối công việc AI của CampusMeet](images/5-Workshop/campusmeet-evidence/ai-state-machine.png)

*State machine `campusmeet-dev-ai-jobs` đang ở trạng thái `Active`. Trạng thái này chứng minh tài nguyên điều phối tồn tại, không tự chứng minh mọi execution hoặc kết quả AI đều đúng.*

AI có thể hỗ trợ hỏi đáp kèm trích dẫn, tóm tắt, tạo bản nháp biên bản hoặc đề xuất nhiệm vụ. Kết quả vẫn là bản nháp. Một đề xuất chỉ trở thành nhiệm vụ sau bước xem trước và xác nhận của người có quyền; cơ chế xử lý lặp an toàn và giao dịch giúp lần thử lại không tạo nhiệm vụ trùng. Các luồng này đã có mã nguồn và kiểm thử ở những phạm vi nhất định, nhưng xử lý âm thanh và kiểm chứng đầu cuối trên AWS vẫn cần tiếp tục hoàn thiện.

![Giao diện trợ lý AI theo phạm vi cuộc họp](images/5-Workshop/campusmeet-evidence/ai-assistant-interface.png)

*Giao diện trợ lý cho phép người dùng đặt câu hỏi từ nguồn trong nhóm hoặc yêu cầu tóm tắt phần đã chọn. Sự xuất hiện của giao diện chỉ xác nhận trải nghiệm đã được xây dựng; chất lượng retrieval, citation và phản hồi vẫn được đánh giá riêng.*

## Kết quả mong đợi

Một cuộc họp có kết quả rõ ràng khi thành viên xác định được nội dung đã thống nhất, người chịu trách nhiệm, thời hạn và nơi theo dõi tiến độ.
