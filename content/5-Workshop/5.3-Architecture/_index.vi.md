---
title: "Kiến trúc tổng quan"
date: 2026-08-08
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Kiến trúc CampusMeet

![Sơ đồ kiến trúc tổng quan của CampusMeet](images/5-Workshop/5.3-Architecture/architecture-diagram.png?v=2)

Kiến trúc CampusMeet gồm sáu thành phần chính:

1. **Giao diện người dùng:** nơi người dùng đăng nhập, quản lý nhóm, xem cuộc họp và cập nhật công việc.
2. **Danh tính và bảo vệ truy cập:** xác nhận người dùng là ai trước khi cho phép sử dụng chức năng.
3. **Xử lý trung tâm:** tiếp nhận yêu cầu, kiểm tra quyền và thực hiện quy tắc của CampusMeet.
4. **Lưu trữ:** giữ thông tin nhóm, cuộc họp, nhiệm vụ và các tệp liên quan.
5. **Dịch vụ hỗ trợ:** kết nối lịch, gửi thông báo, phiên âm hoặc đề xuất nội dung bằng AI khi phù hợp.
6. **Theo dõi vận hành:** hỗ trợ quan sát lỗi, tình trạng hoạt động và chi phí.

## Luồng thông tin đơn giản

Khi một thành viên xem cuộc họp, CampusMeet trước hết xác nhận tài khoản và quyền tham gia nhóm. Hệ thống sau đó lấy thông tin phù hợp và hiển thị trên giao diện. Khi người dùng cập nhật biên bản hoặc nhiệm vụ, thay đổi được lưu lại để các thành viên có quyền cùng theo dõi.

Đối với tài liệu, người dùng tải tệp lên khu vực lưu trữ riêng thay vì đưa trực tiếp vào phần dữ liệu cuộc họp. CampusMeet chỉ liên kết tệp đó với đúng nhóm và cuộc họp.

## Xử lý nội dung mở rộng

Tài liệu và bản phiên âm đã được kiểm tra hoặc phê duyệt có thể trở thành nguồn cho kho tri thức, đồng thời giữ liên kết với nhóm, cuộc họp và phiên bản nguồn. Khi trợ lý AI trả lời hoặc tạo bản nháp, hệ thống giới hạn dữ liệu theo quyền người dùng, cung cấp trích dẫn và chờ xác nhận trước khi áp dụng vào biên bản hoặc nhiệm vụ chính thức.

## Trạng thái triển khai hiện tại

| Phạm vi | Trạng thái tại mốc workshop |
| --- | --- |
| Tài khoản, nhóm, lời mời, cuộc họp và thông báo | Đã có các chức năng nền tảng và luồng sử dụng chính |
| Biểu mẫu cuộc họp, tài liệu, biên bản và nhiệm vụ | Đã có nhiều phần trong giao diện, luồng xử lý và kiểm thử liên quan; cần tiếp tục kiểm chứng xuyên suốt trên môi trường dùng chung |
| Google Calendar và Google Meet | Đã có luồng kết nối, đồng bộ và kiểm chứng cục bộ; còn cần xác minh đầy đủ trên AWS và trình duyệt với tài khoản thực tế |
| Bản phiên âm | Đã có các phần đọc, phân trang, chỉnh sửa, phê duyệt và chuyển nguồn đã duyệt sang bước xử lý tiếp theo; xử lý âm thanh và kiểm chứng cloud đầu-cuối vẫn cần hoàn thiện |
| Kho tri thức và trợ lý AI | Đã có luồng tiếp nhận nguồn được phê duyệt, hỏi đáp có trích dẫn, tóm tắt nội dung, tạo bản nháp biên bản/nhiệm vụ và phân tích tiến độ nhóm, cùng các kiểm thử liên quan; còn cần kiểm chứng đầu-cuối trên cloud trước khi xem là sẵn sàng vận hành |
| Giám sát và kiểm soát chi phí | Đã được đưa vào kiến trúc và hạ tầng; cần tiếp tục theo dõi khi phạm vi sử dụng tăng |

Mũi tên trên sơ đồ thể hiện cách các thành phần phối hợp; bảng trên ghi rõ mức độ đã triển khai và phần còn cần kiểm chứng.
