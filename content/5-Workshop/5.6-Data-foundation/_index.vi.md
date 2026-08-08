---
title: "Đánh giá và định hướng"
date: 2026-08-08
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Phương pháp đánh giá

CampusMeet được đánh giá theo hành trình sử dụng và bằng chứng quan sát được, không chỉ dựa vào số lượng màn hình hoặc tệp mã nguồn. Mỗi nhóm chức năng được xem xét theo các tiêu chí sau:

| Tiêu chí | Cách xem xét |
| --- | --- |
| Tính liên tục của quy trình | Người dùng có thể đi từ nhóm, cuộc họp và nội dung chuẩn bị đến biên bản, nhiệm vụ và tiến độ hay không |
| Quyền truy cập | Dữ liệu có giữ đúng ranh giới nhóm, vai trò và cuộc họp trong từng thao tác hay không |
| Độ tin cậy của thông tin | Bản nháp, nội dung đã xác nhận và phiên bản nguồn có được phân biệt rõ hay không |
| Mức độ kiểm chứng | Kết quả mới chỉ có trong mã nguồn, đã qua kiểm thử cục bộ hay đã được xác minh trên môi trường dùng chung và trình duyệt thực tế |
| Khả năng vận hành | Lỗi, trạng thái xử lý, dữ liệu cần lưu giữ và chi phí có thể được theo dõi khi phạm vi sử dụng tăng hay không |

Một chức năng chỉ được xem là hoàn chỉnh khi các phần liên quan phối hợp đúng trong ngữ cảnh thực tế; có giao diện hoặc tài nguyên cloud chưa đủ để chứng minh toàn bộ luồng đã sẵn sàng.

## Bằng chứng dùng để tổng kết

Nhận định trong workshop được đối chiếu ở nhiều lớp thay vì chỉ dựa vào mô tả:

1. Giao diện và hành trình cho biết người dùng có thể thực hiện những bước nào.
2. Mã nguồn và kiểm thử cho biết quy tắc, quyền và tình huống lỗi đã được xử lý trong phạm vi nào.
3. Kết quả trên môi trường dùng chung cho biết các thành phần thực sự kết nối và lưu dữ liệu ra sao.
4. Kiểm tra trên trình duyệt với tài khoản, vai trò và dữ liệu phù hợp xác nhận trải nghiệm đầu-cuối.

Nếu mới có bằng chứng ở một số lớp, workshop chỉ ghi nhận đúng phần đã quan sát và giữ phần còn lại trong danh sách cần kiểm chứng. Cách làm này giúp người đọc phân biệt giữa thiết kế, triển khai cục bộ và kết quả đã hoạt động trong điều kiện sử dụng thực tế.

## Kết quả hiện tại

CampusMeet đã hình thành một quy trình sản phẩm tương đối rõ: người dùng có tài khoản, tham gia nhóm, theo dõi cuộc họp và tiếp cận thông tin liên quan. Chức năng cuộc họp cốt lõi đã được ghi nhận trên môi trường phát triển. Một số phần về giao diện, tài liệu, biên bản, nhiệm vụ và thông báo cũng đã có trong mã nguồn hoặc được kiểm thử ở phạm vi tương ứng.

Google, phiên âm, kho tri thức và trợ lý AI đã có mã nguồn hoặc kiểm thử trong từng phạm vi. Mức độ xác minh vẫn khác nhau giữa môi trường cục bộ, môi trường phát triển và kịch bản đầu-cuối với dữ liệu, tài khoản và quyền thực tế.

## Bài học rút ra

- Bắt đầu từ hành trình người dùng giúp xác định phạm vi rõ hơn so với bắt đầu từ danh sách dịch vụ.
- Xác thực và phân quyền phải là hai bước riêng; tích hợp ngoài không được làm gián đoạn quy trình cốt lõi.
- Phiên âm và AI chỉ tạo giá trị khi nguồn có thể truy vết, kết quả được xác nhận và từng mức kiểm thử được ghi nhận đúng thực tế.

## Những điểm cần tiếp tục kiểm chứng

- Tình huống quyền truy cập giữa nhiều nhóm và vai trò.
- Luồng tải tài liệu trên môi trường sử dụng thực tế.
- Đồng bộ Google Calendar và Google Meet trên trình duyệt với tài khoản thật.
- Xử lý âm thanh và phiên âm xuyên suốt.
- Chất lượng nguồn dẫn và bước xác nhận nội dung do AI gợi ý.
- Khả năng theo dõi lỗi, chi phí và dữ liệu cần lưu giữ khi số lượng người dùng tăng.

## Định hướng phát triển

1. Ưu tiên làm ổn định quy trình từ nhóm đến cuộc họp và nhiệm vụ.
2. Kiểm chứng từng luồng trên môi trường dùng chung trước khi công bố hoàn chỉnh.
3. Hoàn thiện luồng đưa tài liệu và bản phiên âm đã duyệt vào kho tri thức, đồng thời kiểm chứng trợ lý AI đầu-cuối.
4. Giữ AI ở vai trò hỗ trợ, cho phép người dùng xem nguồn và xác nhận.
5. Cải thiện thông báo lỗi và trạng thái chức năng để người dùng biết bước tiếp theo.
6. Duy trì quyền truy cập phù hợp và bảo vệ nội dung của từng nhóm.

## Kết luận

CampusMeet tại mốc tổng kết nên được nhìn nhận là nền tảng đã có phần cốt lõi và hướng mở rộng rõ ràng, nhưng vẫn cần thêm kiểm chứng trước khi xem toàn bộ hệ thống là hoàn chỉnh. Cách đánh giá này phản ánh đúng trạng thái sản phẩm và tránh nhận định cao hơn kết quả đã thực hiện.
