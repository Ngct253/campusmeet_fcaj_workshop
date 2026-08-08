---
title: "Tổng quan và phạm vi CampusMeet"
date: 2026-08-08
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## CampusMeet là gì?

CampusMeet là nền tảng quản lý cuộc họp dành cho nhóm học tập và nhóm dự án nhỏ. Hệ thống tập trung các thông tin thường bị phân tán như thành viên, lịch họp, tài liệu, nội dung thảo luận, biên bản và công việc sau cuộc họp.

CampusMeet không thay thế công cụ gọi video. Cuộc họp vẫn có thể diễn ra trên Google Meet hoặc trực tiếp. Vai trò của CampusMeet là giúp nhóm chuẩn bị tốt hơn, lưu lại kết quả và tiếp tục theo dõi công việc sau khi cuộc họp kết thúc.

## Vấn đề cần giải quyết

Trong một nhóm, thông tin thường nằm ở nhiều nơi: lịch trong ứng dụng Calendar, tài liệu trong ổ đĩa, ghi chú trong tin nhắn và công việc trong một danh sách khác. Cách làm này khiến thành viên khó biết đâu là thông tin mới nhất, ai chịu trách nhiệm và quyết định nào đã được thống nhất.

CampusMeet kết nối các thông tin đó theo một mạch thống nhất:

1. Tạo nhóm và mời thành viên.
2. Lập lịch cuộc họp và chuẩn bị nội dung.
3. Chia sẻ tài liệu liên quan.
4. Ghi nhận biên bản, quyết định và đầu việc.
5. Theo dõi tiến độ sau cuộc họp.
6. Tra cứu lại thông tin khi cần.

## Phương pháp xác định phạm vi

Phạm vi CampusMeet được xác định từ hành trình của một cuộc họp thay vì danh sách công nghệ. Luồng cơ bản phải giúp nhóm tập hợp thành viên, tổ chức cuộc họp, ghi nhận kết quả và theo dõi đầu việc. Đồng bộ lịch, phiên âm và AI được bổ sung sau để giảm thao tác hoặc hỗ trợ tra cứu. Nếu một tích hợp chưa sẵn sàng, nhóm vẫn có thể dùng các chức năng nền tảng; nhờ đó quy trình chính không phụ thuộc hoàn toàn vào Google, xử lý âm thanh hoặc AI.

## Phương hướng triển khai

CampusMeet được phát triển theo các luồng chức năng hoàn chỉnh thay vì xây toàn bộ giao diện trước rồi mới bổ sung backend. Với mỗi phạm vi, nhóm xác định hành trình người dùng, hợp đồng dữ liệu, quy tắc quyền, phần xử lý, cách lưu trữ và kiểm thử liên quan. Cách làm này giúp một chức năng được đánh giá theo toàn bộ đường đi của dữ liệu chứ không chỉ theo màn hình đã xuất hiện.

Trình tự chung gồm bốn lớp:

1. Chuẩn hóa kho mã nguồn, kiểu dữ liệu dùng chung và quy trình kiểm tra chất lượng.
2. Thiết lập danh tính, API và nền tảng dữ liệu để các chức năng có cùng ranh giới truy cập.
3. Hoàn thiện hành trình cốt lõi từ nhóm, cuộc họp và biên bản đến nhiệm vụ.
4. Bổ sung upload, đồng bộ Google, phiên âm và AI như các nhánh mở rộng có trạng thái và khả năng phục hồi riêng.

Nhờ trình tự này, các chức năng nâng cao có thể tiếp tục hoàn thiện mà không làm thay đổi ý nghĩa của dữ liệu cuộc họp đã được lưu trong CampusMeet.

## Trọng tâm triển khai trong phạm vi thực tập

Phần công việc được trình bày sâu hơn trong workshop tập trung vào xác thực và nền tảng dữ liệu: tham gia cấu hình Amazon Cognito, kết nối API Gateway với Lambda và JWT authorizer, tổ chức tài nguyên bằng AWS SAM/CloudFormation, đồng thời xây dựng và kiểm chứng mô hình năm bảng DynamoDB. Đây là nền tảng để các luồng tài khoản, nhóm, lời mời và cuộc họp dùng chung cách xác định danh tính, phân quyền và lưu trữ.

Workshop không xem toàn bộ CampusMeet là kết quả của một cá nhân. Các phần tích hợp, giao diện và AI được trình bày trong bối cảnh chung của dự án; riêng xác thực và DynamoDB được giải thích rõ hơn vì gắn trực tiếp với phạm vi công việc đã tham gia.

## Tiêu chí thành công

CampusMeet không được đánh giá chỉ bằng số lượng màn hình. Một hành trình có giá trị khi thông tin trước họp đủ để thành viên chuẩn bị; kết quả sau họp xác định được quyết định, người phụ trách và thời hạn; dữ liệu có thể truy lại đúng nhóm và cuộc họp; người không có quyền không tiếp cận được nội dung. Các khả năng nâng cao được xem là hiệu quả khi chúng giảm thao tác hoặc giúp tìm lại thông tin mà không làm thay đổi trách nhiệm xác nhận của người dùng.

## Cách các phần liên kết với nhau

Nhóm là không gian chung và ranh giới truy cập của CampusMeet. Mỗi cuộc họp liên kết với tài liệu, bản phiên âm, biên bản và nhiệm vụ. Nhờ đó, thành viên có thể lần theo một công việc hoặc quyết định về đúng cuộc họp đã tạo ra nó thay vì tìm kiếm trong nhiều công cụ rời rạc.

## CampusMeet trong quá trình sử dụng

Sau khi đăng nhập, trang tổng quan tập hợp nhóm đang tham gia, cuộc họp sắp tới, thông báo, trạng thái tài khoản và công việc cá nhân. Cách bố trí này phản ánh mục tiêu của CampusMeet: người dùng bắt đầu từ một không gian chung rồi đi đến nhóm, cuộc họp và phần việc liên quan, thay vì phải tự ghép thông tin từ nhiều công cụ.

![Trang tổng quan CampusMeet sau khi đăng nhập](images/5-Workshop/campusmeet-evidence/product-dashboard.png)

*Trang tổng quan cho thấy các nhóm chức năng được đặt trong cùng hành trình sản phẩm. Ảnh giao diện xác nhận phạm vi người dùng có thể quan sát, còn mức độ hoàn chỉnh của từng luồng vẫn được đối chiếu thêm bằng API, dữ liệu và trạng thái AWS.*

## Giá trị chính

- **Tập trung:** thông tin của nhóm và cuộc họp được đặt trong cùng một không gian.
- **Rõ trách nhiệm:** mỗi đầu việc có người phụ trách và trạng thái theo dõi.
- **Có lịch sử:** quyết định và nội dung cuộc họp có thể được xem lại.
- **Hỗ trợ cộng tác:** thành viên cùng theo dõi một nguồn thông tin chung.
- **Hỗ trợ thông minh có kiểm soát:** nội dung do AI gợi ý chỉ là bản nháp và cần người dùng xác nhận.

## Phạm vi hiện tại

CampusMeet đã có nền tảng cho tài khoản, nhóm, lời mời, cuộc họp và thông báo. Một số luồng về biểu mẫu cuộc họp, tài liệu, biên bản và công việc cũng đã được xây dựng hoặc cải thiện. Tích hợp Google, phiên âm và AI đã có các luồng cùng kiểm thử ở những phạm vi nhất định, nhưng mức độ kiểm chứng trên cloud chưa đồng đều; workshop vì vậy ghi rõ phần đã có và phần cần tiếp tục xác minh.

| Nhóm chức năng | Phạm vi | Trạng thái tổng quát |
| --- | --- | --- |
| Tài khoản và cộng tác | Đăng ký, đăng nhập, hồ sơ, nhóm, thành viên, lời mời và thông báo | Đã có các luồng nền tảng |
| Quản lý cuộc họp | Tạo, xem, cập nhật, hủy và chuẩn bị nội dung cuộc họp | Chức năng cốt lõi đã có; một số tình huống phân quyền cần tiếp tục kiểm chứng |
| Nội dung sau họp | Tài liệu, biên bản, bản phiên âm, nhiệm vụ và theo dõi tiến độ | Đã có nhiều phần trong giao diện và xử lý; cần tiếp tục kiểm thử xuyên suốt |
| Tích hợp và AI | Google Calendar/Meet, nhắc lịch, kho tri thức và nội dung gợi ý | Có các phạm vi đã triển khai hoặc kiểm thử; chưa được xem là hoàn chỉnh trên môi trường thực tế |
