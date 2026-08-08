---
title: "Kiến trúc tổng quan"
date: 2026-08-08
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Cách đọc sơ đồ

![Sơ đồ kiến trúc tổng quan của CampusMeet](images/5-Workshop/5.3-Architecture/architecture-diagram.png?v=2)

Sơ đồ cho biết các phần lớn của CampusMeet và cách thông tin di chuyển giữa chúng. Người đọc không cần hiểu chi tiết kỹ thuật; có thể xem kiến trúc theo sáu lớp sau:

1. **Giao diện người dùng:** nơi người dùng đăng nhập, quản lý nhóm, xem cuộc họp và cập nhật công việc.
2. **Danh tính và bảo vệ truy cập:** xác nhận người dùng là ai trước khi cho phép sử dụng chức năng.
3. **Xử lý trung tâm:** tiếp nhận yêu cầu, kiểm tra quyền và thực hiện quy tắc của CampusMeet.
4. **Lưu trữ:** giữ thông tin nhóm, cuộc họp, nhiệm vụ và các tệp liên quan.
5. **Dịch vụ hỗ trợ:** kết nối lịch, gửi thông báo, phiên âm hoặc đề xuất nội dung bằng AI khi phù hợp.
6. **Theo dõi vận hành:** hỗ trợ quan sát lỗi, tình trạng hoạt động và chi phí.

## Luồng thông tin đơn giản

Khi một thành viên xem cuộc họp, CampusMeet trước hết xác nhận tài khoản và quyền tham gia nhóm. Hệ thống sau đó lấy thông tin phù hợp và hiển thị trên giao diện. Khi người dùng cập nhật biên bản hoặc nhiệm vụ, thay đổi được lưu lại để các thành viên có quyền cùng theo dõi.

Đối với tài liệu, người dùng tải tệp lên khu vực lưu trữ riêng thay vì đưa trực tiếp vào phần dữ liệu cuộc họp. CampusMeet chỉ liên kết tệp đó với đúng nhóm và cuộc họp.

## Phần đã có và phần định hướng

| Phạm vi | Cách hiểu tại mốc workshop |
| --- | --- |
| Tài khoản, nhóm, lời mời, cuộc họp và thông báo | Là nền tảng hiện có của sản phẩm |
| Biểu mẫu cuộc họp, tài liệu, biên bản và nhiệm vụ | Đã có phần triển khai và kiểm thử liên quan, cần tiếp tục kiểm chứng theo từng luồng |
| Google Calendar và Google Meet | Có hướng tích hợp và một số phần đã được kiểm chứng cục bộ |
| Phiên âm và AI | Là phần đang phát triển, chưa được xem là hoàn chỉnh trên môi trường thực tế |
| Giám sát và kiểm soát chi phí | Là yêu cầu cần duy trì khi hệ thống được mở rộng |

Mũi tên trên sơ đồ thể hiện hướng kết nối mong muốn. Nó không có nghĩa rằng tất cả các nhánh đã được hoàn thiện ở cùng một mức độ.
