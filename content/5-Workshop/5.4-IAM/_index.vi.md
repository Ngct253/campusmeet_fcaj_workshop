---
title: "Phạm vi chức năng hiện tại"
date: 2026-08-08
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Cách hiểu trạng thái

CampusMeet được phát triển theo từng nhóm chức năng. Workshop phân biệt ba mức để không trình bày cao hơn thực tế:

- **Đã có nền tảng:** luồng chính đã xuất hiện trong sản phẩm.
- **Cần kiểm chứng thêm:** đã có phần triển khai nhưng cần thử nghiệm đầy đủ hơn.
- **Đang phát triển:** chức năng mới hoàn thành một phần hoặc còn phụ thuộc vào thành phần khác.

## Tổng quan theo M1–M4

| Mốc | Phạm vi | Trạng thái tổng quát |
| --- | --- | --- |
| M1 | Tài khoản, hồ sơ, nhóm, thành viên, lời mời và thông báo | Đã có nền tảng chính |
| M2 | Tạo, xem, cập nhật và hủy cuộc họp | Lõi cuộc họp đã được ghi nhận trên môi trường phát triển; một số tình huống quyền vẫn cần kiểm chứng |
| M3 | Nội dung chuẩn bị, tài liệu, biên bản, action item và nhiệm vụ | Đã có nhiều phần trong giao diện và luồng xử lý; cần tiếp tục kiểm thử xuyên suốt |
| M4 | Đồng bộ Google, bản phiên âm, AI và tổng hợp tiến độ | Có các phần ban đầu hoặc thử nghiệm; chưa được xem là hoàn chỉnh |

## Thông tin được quản lý

CampusMeet kết nối hồ sơ người dùng, nhóm, thành viên, cuộc họp, tài liệu, biên bản và nhiệm vụ. Mỗi cuộc họp thuộc một nhóm; tài liệu và biên bản gắn với cuộc họp; đầu việc có người phụ trách, thời hạn và trạng thái.

Tài liệu được đặt trong khu vực lưu trữ riêng và chỉ hiển thị cho người có quyền. Với biên bản, phiên âm và nội dung do AI hỗ trợ, hệ thống cần phân biệt bản nháp với nội dung đã được người dùng xác nhận.

## Nguyên tắc tổng kết

Một màn hình xuất hiện không đồng nghĩa toàn bộ chức năng phía sau đã hoàn chỉnh. Một luồng chạy được cục bộ cũng chưa chắc đã được kiểm chứng đầy đủ trên môi trường dùng chung. Báo cáo vì vậy chỉ ghi nhận đúng mức đã quan sát và nêu rõ phần còn lại.
