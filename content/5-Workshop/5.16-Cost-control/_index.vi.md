---
title: "Kiểm soát chi phí"
date: 2026-08-08
weight: 16
chapter: false
pre: " <b> 5.16. </b> "
---

# Kiểm soát chi phí

## Mục tiêu

CampusMeet sử dụng nhiều dịch vụ serverless nên chi phí phần lớn tăng theo mức sử dụng. Điều này phù hợp với đồ án và môi trường thử nghiệm, nhưng không có nghĩa hệ thống tự động miễn phí. Phần này tập trung vào cách nhận biết nguồn phát sinh chi phí và đặt giới hạn vận hành hợp lý.

## 1. AWS Budgets

Trước khi triển khai môi trường dùng chung, nên tạo AWS Budget và cấu hình email nhận cảnh báo.

Ví dụ các ngưỡng có thể dùng:

- 50% ngân sách tháng.
- 80% ngân sách tháng.
- 100% ngân sách tháng.

Budget chỉ cảnh báo, không tự dừng dịch vụ. Vì vậy khi nhận cảnh báo vẫn cần mở Billing/Cost Explorer để tìm dịch vụ đang tăng chi phí.

## 2. DynamoDB

Các bảng CampusMeet dùng `PAY_PER_REQUEST`, phù hợp khi lưu lượng chưa ổn định và nhóm không muốn cấu hình capacity cố định.

Chi phí phụ thuộc chủ yếu vào:

- Read request.
- Write request.
- Dung lượng lưu trữ.
- Backup/PITR khi bật.

Không dùng `Scan` cho nghiệp vụ thông thường vừa vì hiệu năng vừa vì có thể đọc nhiều dữ liệu không cần thiết.

## 3. Lambda và API Gateway

Lambda tính theo số lần gọi và thời gian thực thi. API Gateway tính theo request.

Để tránh chi phí không cần thiết:

- Không retry vô hạn.
- Không để function chờ dịch vụ ngoài nếu có thể chuyển sang async worker.
- Đặt timeout phù hợp với từng function.
- Theo dõi function có duration tăng bất thường.

## 4. Amazon S3

S3 phát sinh chi phí từ:

- Dung lượng lưu trữ.
- Request PUT/GET.
- Data transfer trong một số trường hợp.

Tệp người dùng cần có lifecycle phù hợp với chính sách của dự án. Không giữ nhiều file thử nghiệm lớn không cần thiết trong thời gian dài.

Workshop không hướng dẫn xóa toàn bộ tài nguyên sau khi học xong vì môi trường CampusMeet còn được dùng cho triển khai và kiểm thử tiếp theo.

## 5. CloudFront

CloudFront phục vụ frontend production và có thể phát sinh chi phí theo request/data transfer. Với đồ án lưu lượng nhỏ, mức sử dụng thường không lớn, nhưng vẫn nên theo dõi thay vì giả định bằng 0.

## 6. CloudWatch Logs

CloudWatch Logs có chi phí ingest và lưu trữ.

Do đó:

- Đặt retention thay vì giữ log vô thời hạn nếu không cần.
- Không log payload lớn.
- Không log transcript/document đầy đủ.
- Tránh loop lỗi tạo hàng nghìn log giống nhau.

Log retention 7 ngày ở các môi trường phát triển là một lựa chọn hợp lý nếu vẫn đáp ứng nhu cầu debug.

## 7. EventBridge Scheduler

CampusMeet dùng Scheduler cho reminder và một số retry bất đồng bộ.

Cần kiểm soát:

- Không tạo nhiều schedule trùng cho cùng một công việc.
- Hủy/thay thế schedule cũ khi nghiệp vụ yêu cầu.
- Có giới hạn retry.

Một lỗi tạo schedule lặp lại liên tục vừa gây hành vi sai vừa có thể làm tăng chi phí.

## 8. AWS Step Functions

Step Functions tính phí dựa trên số state transition đối với Standard Workflow.

Do đó vòng polling ingestion cần:

- Khoảng chờ hợp lý.
- Điều kiện dừng rõ ràng.
- Số lần kiểm tra tối đa hoặc timeout tổng thể.

Không thiết kế loop `Wait → Check → Wait` mà không có đường thoát khi dependency gặp lỗi kéo dài.

## 9. Amazon SES

SES tính phí theo số email và dữ liệu liên quan. Reminder email chỉ nên được gửi khi có mục đích rõ ràng.

Không retry email vô hạn nếu địa chỉ không hợp lệ hoặc cấu hình sender sai.

## 10. Amazon Bedrock

Bedrock là nhóm dịch vụ cần theo dõi kỹ vì chi phí phụ thuộc model và lượng token/dữ liệu xử lý.

Các biện pháp:

- Giới hạn kích thước input.
- Không gửi lại toàn bộ lịch sử khi không cần.
- Giới hạn số meeting/source được retrieval cho một request.
- Không tự chạy AI định kỳ nếu không có nhu cầu thật.
- Theo dõi lỗi retry để tránh gọi model lặp nhiều lần.

## 11. Knowledge Base và S3 Vectors

RAG không chỉ có chi phí model generation. Ingestion, lưu embedding/vector và retrieval cũng cần được tính đến.

Không re-ingest cùng một tài liệu liên tục nếu source không thay đổi. Idempotency và trạng thái ingestion giúp tránh việc này.

## 12. Amazon Transcribe

Transcribe là chi phí dự kiến nếu sau này bật live hoặc batch transcription đầy đủ.

Vì runtime transcription chưa thuộc core E2E hiện tại, workshop không yêu cầu bật dịch vụ này chỉ để hoàn thành bản demo.

## 13. Môi trường dev và production

Không nên dùng cùng một cách bảo vệ cho mọi môi trường.

Ví dụ:

- Dev ưu tiên chi phí thấp và thay đổi nhanh.
- Production ưu tiên an toàn dữ liệu hơn, có thể bật PITR/deletion protection.

Việc bật thêm khả năng bảo vệ có thể tăng chi phí nhưng đổi lại giảm rủi ro mất dữ liệu.

## 14. Theo dõi chi phí sau đợt deploy

Sau một đợt test lớn:

1. Mở AWS Billing/Cost Explorer.
2. Lọc theo dịch vụ.
3. Kiểm tra CloudWatch, Bedrock, DynamoDB, S3, Lambda và Step Functions.
4. Kiểm tra có execution/schedule nào chạy bất thường không.
5. Xác nhận Budget alert vẫn hoạt động.

Không cần đợi đến cuối tháng mới kiểm tra.

## 15. Nguyên tắc cho CampusMeet

Ưu tiên theo thứ tự:

```text
Đúng nghiệp vụ
   ↓
Không retry/loop ngoài kiểm soát
   ↓
Không xử lý dữ liệu thừa
   ↓
Theo dõi usage
   ↓
Tối ưu chi phí khi có số liệu thật
```

Không nên làm kiến trúc phức tạp chỉ để tiết kiệm một khoản rất nhỏ trong môi trường đồ án, nhưng cũng không nên bỏ qua các vòng lặp bất đồng bộ hoặc AI có khả năng tăng chi phí nhanh.

## Kết quả cần đạt

- Nhóm biết các dịch vụ chính nào có thể phát sinh chi phí.
- AWS Budget được dùng để cảnh báo sớm.
- Log, retry, Scheduler và Step Functions không chạy ngoài kiểm soát.
- AI/RAG được gọi trong phạm vi cần thiết.
- Workshop kết thúc ở bước kiểm soát chi phí; không có chapter dọn dẹp tài nguyên riêng.
