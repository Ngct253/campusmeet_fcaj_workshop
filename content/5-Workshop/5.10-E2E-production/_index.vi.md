---
title: "Kiểm thử E2E và kết quả production"
date: 2026-08-08
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

# Kiểm thử E2E và kết quả production

Phần cuối workshop dùng để ghi nhận kết quả triển khai thật. Chỉ những gì đã chạy trên AWS và trình duyệt mới được đánh dấu hoàn thành.

## Production link

Sau khi frontend được publish qua CloudFront, ghi URL tại đây:

```text
Production URL: TBD
Region: ap-southeast-1
```

Production link phải mở được qua HTTPS và không phụ thuộc `localhost`.

## Luồng E2E cốt lõi

Dùng hai tài khoản thật để kiểm tra quyền và dữ liệu:

```text
User A đăng ký / đăng nhập
  ↓
A tạo Group
  ↓
A mời User B
  ↓
B chấp nhận lời mời
  ↓
A tạo Meeting
  ↓
A lưu Minutes + Action Item
  ↓
Action Item → Task cho B
  ↓
B đổi TODO → DOING → DONE
  ↓
Dashboard cập nhật
```

Sau các bước quan trọng nên reload trang để xác nhận dữ liệu thực sự được lưu ở backend.

## Kết quả thực tế

| Hạng mục | Trạng thái |
| --- | --- |
| CloudFront production URL | Chưa xác minh |
| Cognito đăng ký / đăng nhập | Chưa xác minh |
| Group và Invitation | Chưa xác minh |
| Meeting | Chưa xác minh |
| Minutes và Task | Chưa xác minh |
| Dashboard | Chưa xác minh |
| Phân quyền cơ bản | Chưa xác minh |

Không đổi trạng thái thành `PASS` chỉ vì unit test hoặc build đã thành công.

## Tích hợp bổ sung

Sau khi core E2E chạy ổn mới kiểm tra thêm:

- Google OAuth và Calendar/Meet sync;
- upload document;
- RAG và citation;
- các chức năng AI khác phù hợp với môi trường production.

Nếu một tích hợp bổ sung chưa hoàn tất, ghi đúng trạng thái đó nhưng không làm mất kết quả của core E2E đã chạy thành công.

## Bằng chứng nộp bài

Nên lưu lại một số ảnh chụp ngắn gọn:

- production homepage hoặc sign-in;
- Group có hai thành viên;
- Meeting detail;
- Minutes/Task;
- Dashboard sau khi Task hoàn thành;
- CloudFormation stack hoặc AWS Console thể hiện tài nguyên production.

Khi các bước trên hoàn tất, workshop có thể kết thúc bằng production URL và trạng thái E2E thực tế thay vì một danh sách kỹ thuật dài.