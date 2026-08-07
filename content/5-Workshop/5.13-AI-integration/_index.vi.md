---
title: "Tích hợp AI"
date: 2026-08-08
weight: 13
chapter: false
pre: " <b> 5.13. </b> "
---

# Tích hợp AI

## Mục tiêu

Phần này trình bày cách CampusMeet sử dụng Amazon Bedrock để hỗ trợ người dùng tra cứu lại nội dung cuộc họp, tạo bản nháp biên bản và đề xuất công việc. AI trong CampusMeet không thay người dùng ra quyết định và cũng không được phép đọc dữ liệu ngoài phạm vi nhóm mà người dùng có quyền truy cập.

Điểm quan trọng nhất của phần này không phải chỉ là gọi được model, mà là bảo đảm dữ liệu đưa vào AI có nguồn rõ ràng, đúng phạm vi và kết quả có thể kiểm tra lại.

## 1. Kiến trúc xử lý AI

Luồng tổng quát:

```text
CampusMeet API
     ↓
Tạo AIJob
     ↓
AWS Step Functions
     ↓
AI Worker
     ↓
Amazon Bedrock
     ↓
Knowledge Base / S3 Vectors
     ↓
Kết quả + citation
```

API không giữ kết nối HTTP chờ toàn bộ quá trình xử lý. Khi một yêu cầu cần thời gian dài hơn, backend tạo `AIJob` và trả về `aiJobId` để frontend theo dõi trạng thái.

## 2. AIJob

AIJob là bản ghi theo dõi một yêu cầu AI. Các trạng thái chính gồm:

```text
QUEUED
PROCESSING
COMPLETED
FAILED
```

Mỗi job giữ các thông tin cần thiết như:

- `aiJobId`.
- `groupId`.
- `meetingId` nếu tác vụ gắn với một cuộc họp.
- loại tác vụ.
- trạng thái.
- số lần thử.
- `requestId`.
- thời điểm tạo và cập nhật.

Việc lưu job giúp người dùng có thể rời trang rồi quay lại mà không cần giữ request HTTP ban đầu.

## 3. Các chức năng AI hiện có trong mã nguồn

CampusMeet hiện có các luồng AI chính:

### Meeting Chat

Người dùng đặt câu hỏi trong phạm vi một cuộc họp. Hệ thống truy xuất các nguồn được phép rồi mới gọi model để tạo câu trả lời.

### Group Search

Cho phép tìm kiếm hoặc hỏi đáp trong phạm vi một nhóm và các cuộc họp được chọn. Backend phải kiểm tra mọi `meetingId` đều thuộc đúng `groupId` trước khi xử lý.

### Minutes Draft

AI có thể tạo bản nháp biên bản dựa trên các nguồn đã được chấp thuận. Kết quả này vẫn là bản nháp và cần người dùng đọc lại trước khi lưu thành biên bản chính thức.

### Task Proposals

AI có thể đề xuất các công việc có thể phát sinh từ nội dung cuộc họp. Đề xuất không tự động trở thành Task chính thức.

### Progress Analysis

Mã nguồn có luồng tạo phân tích tiến độ dựa trên dữ liệu tổng hợp của nhóm. Tuy nhiên, phần này chỉ nên được đưa vào demo khi nguồn snapshot đầu vào đã được xác minh đầy đủ trên môi trường triển khai.

## 4. Ingestion tài liệu

Để một tài liệu có thể được dùng làm nguồn cho AI, file cần đi qua quá trình ingestion.

```text
Attachment
    ↓
INGEST_SOURCE
    ↓
AI Worker
    ↓
Chuẩn hóa nội dung
    ↓
Knowledge Base ingestion
    ↓
READY
```

Một source ở trạng thái `PROCESSING` chưa được dùng như nguồn hoàn chỉnh. Chỉ khi ingestion hoàn tất và source chuyển sang trạng thái phù hợp thì retrieval mới được phép sử dụng.

Trong bản hiện tại, việc polling trạng thái ingestion cần được kiểm tra trên AWS thật trong phần E2E. Không xem việc worker có code `CHECK_INGESTION` là bằng chứng quy trình đã chạy hoàn chỉnh trên cloud.

## 5. Giới hạn phạm vi dữ liệu

Trước khi truy xuất dữ liệu cho AI, backend phải kiểm tra:

- Người dùng là thành viên của nhóm.
- Group ID của source đúng với group đang truy vấn.
- Meeting ID nằm trong tập cuộc họp người dùng được phép đọc.
- Source ở trạng thái được phép sử dụng.
- Source đã hoàn tất ingestion.

Ví dụ, dữ liệu của Group A không được xuất hiện trong kết quả tìm kiếm của Group B chỉ vì cả hai cùng nằm trong một Knowledge Base.

## 6. Citation

Một câu trả lời AI có giá trị hơn khi người dùng biết thông tin đó đến từ đâu.

CampusMeet gắn citation vào kết quả để người dùng có thể kiểm tra lại tài liệu hoặc đoạn nội dung liên quan.

Backend không tin citation do model tự bịa. Citation trả về phải được đối chiếu với tập source/chunk mà hệ thống thực sự đã truy xuất cho yêu cầu đó.

Nếu model tạo câu trả lời nhưng không có nguồn hỗ trợ phù hợp, hệ thống nên trả trạng thái thiếu ngữ cảnh thay vì cố tạo một câu trả lời nghe có vẻ hợp lý.

## 7. Human-in-the-loop

AI trong CampusMeet chỉ hỗ trợ người dùng.

Các quy tắc chính:

- Minutes do AI tạo là bản nháp.
- Task Proposal chỉ là đề xuất.
- AI không được tự tạo quyết định chính thức cho nhóm.
- Nội dung quan trọng cần người dùng xem lại trước khi ghi vào dữ liệu nghiệp vụ.

Luồng an toàn:

```text
AI sinh kết quả
      ↓
Người dùng xem lại
      ↓
Chỉnh sửa nếu cần
      ↓
Người dùng xác nhận
      ↓
Dữ liệu chính thức
```

Trong phiên bản E2E hiện tại, luồng chắc chắn để tạo Task chính thức vẫn là `Minutes Action Item → Convert to Task`. AI Task Proposal có thể được trình bày như chức năng hỗ trợ nếu bước xác nhận chưa được nối hoàn chỉnh.

## 8. Xử lý lỗi model và dịch vụ

Các lỗi AI không được làm hỏng dữ liệu chính của Meeting hoặc Task.

Các trường hợp cần xử lý:

- Model tạm thời không khả dụng.
- Job timeout.
- Retrieval không có nguồn phù hợp.
- Citation không hợp lệ.
- Source chưa READY.
- Người dùng không còn quyền truy cập nhóm.

AIJob chuyển sang `FAILED` với mã lỗi phù hợp để frontend có thể hiển thị trạng thái rõ ràng.

## 9. Không ghi dữ liệu nhạy cảm vào log

Không log toàn bộ:

- prompt chứa dữ liệu riêng tư;
- nội dung tài liệu;
- transcript;
- OAuth token;
- presigned URL;
- model response nếu chứa dữ liệu nhạy cảm và không cần cho chẩn đoán.

Log nên tập trung vào metadata an toàn như job ID, loại tác vụ, trạng thái, latency và error code.

## 10. Kiểm thử trước khi triển khai

Chạy:

```powershell
npm run lint
npm run typecheck
npm run test
npm run build
```

Các test quan trọng:

- User ngoài nhóm không tạo được AI request cho group đó.
- Meeting được chọn phải thuộc đúng group.
- Source chưa approved/READY không được retrieval sử dụng.
- Citation phải thuộc tập chunk đã truy xuất.
- AI result không citation phù hợp được xử lý an toàn.
- Job lỗi chuyển trạng thái đúng.
- Không tạo nhiều job khi cùng idempotency key được gửi lại.

## 11. Kiểm thử E2E trên AWS

Phần AI chỉ được đánh dấu hoàn tất sau khi kiểm thử thật.

Quy trình tối thiểu:

1. Upload một file `.txt` hoặc `.pdf` nhỏ.
2. Hoàn tất Attachment.
3. Theo dõi `AIJob` ingestion.
4. Xác nhận Knowledge Source chuyển sang `READY`.
5. Hỏi một câu có đáp án nằm rõ trong tài liệu.
6. Kiểm tra câu trả lời và citation.
7. Hỏi một câu không có thông tin trong source.
8. Xác nhận hệ thống không bịa dữ liệu.

Trạng thái tại thời điểm viết workshop:

```text
Source implementation: đã có
Automated/local tests: đã có nhiều lớp kiểm tra
AWS production E2E: cần xác minh trong đợt triển khai cuối
```

## Kết quả cần đạt

- AI được chạy qua AIJob và worker bất đồng bộ.
- Retrieval bị giới hạn theo quyền của người dùng.
- Source phải ở trạng thái hợp lệ trước khi được dùng.
- Citation được kiểm tra thay vì tin trực tiếp model.
- Kết quả AI quan trọng cần người dùng xem lại.
- Workshop phân biệt rõ giữa code đã triển khai và tích hợp AWS đã được xác minh thật.
