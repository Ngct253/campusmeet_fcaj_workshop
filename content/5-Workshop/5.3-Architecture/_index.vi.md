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

## Lý do lựa chọn cách tổ chức kiến trúc

Kiến trúc được chia theo trách nhiệm để luồng họp cốt lõi không bị trộn với các khả năng hỗ trợ. Giao diện tập trung vào trải nghiệm người dùng; lớp danh tính và xử lý trung tâm giữ quy tắc truy cập; dữ liệu nghiệp vụ và tệp được lưu theo đặc tính riêng; các tích hợp bên ngoài được kết nối qua ranh giới rõ ràng. Nhờ đó, thay đổi ở lịch, phiên âm hoặc AI không làm thay đổi ý nghĩa của nhóm, cuộc họp, biên bản và nhiệm vụ.

Tệp và âm thanh có kích thước lớn được lưu trong khu vực riêng tư, còn hệ thống chỉ quản lý thông tin liên kết cần thiết. Những công việc cần nhiều thời gian như xử lý âm thanh, chuẩn hóa tài liệu hoặc tạo nội dung được theo dõi theo trạng thái thay vì buộc người dùng chờ tại một màn hình. Cách làm này cần quản lý trạng thái và lỗi cẩn thận hơn, nhưng giúp luồng chính phản hồi rõ ràng và cho phép thử lại khi một dịch vụ hỗ trợ gặp sự cố.

## Các nguyên tắc kiến trúc chính

- Đăng nhập xác nhận danh tính, còn quyền trên từng nhóm và cuộc họp vẫn được kiểm tra riêng.
- Google Meet là dịch vụ họp bên ngoài; CampusMeet quản lý quy trình và kết quả, không tự xây công cụ gọi video.
- Dữ liệu cuộc họp, tệp và nội dung AI giữ liên kết với nhóm và nguồn ban đầu để có thể truy vết.
- Chỉ tài liệu hoặc bản phiên âm đã được phê duyệt theo luồng phù hợp mới được dùng làm nguồn tri thức chính thức.
- Kết quả AI là bản nháp có nguồn dẫn; thay đổi biên bản hoặc nhiệm vụ cần người dùng có quyền xác nhận.
- Lỗi, tình trạng xử lý và chi phí cần được theo dõi để chức năng nâng cao không che khuất vấn đề vận hành.
