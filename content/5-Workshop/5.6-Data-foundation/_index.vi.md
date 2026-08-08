---
title: "Kiểm chứng, vận hành và đánh giá"
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

## Kiểm tra kỹ thuật trước khi bàn giao

Các bước kiểm tra chất lượng của kho mã nguồn cần được chạy từ thư mục gốc:

```powershell
npm run lint
npm run typecheck
npm run test
npm run build
npm run infra:validate
```

`lint` và `typecheck` phát hiện lỗi quy tắc hoặc kiểu dữ liệu; `test` kiểm tra thành phần giao diện, dịch vụ, bộ xử lý yêu cầu, lớp truy cập dữ liệu và các bộ kết nối trong phạm vi tương ứng; `build` xác nhận các dự án con có thể tạo bản dựng; `infra:validate` kiểm tra các ràng buộc hạ tầng do kho mã nguồn định nghĩa. Một lần chạy đạt không chứng minh hệ thống đã sẵn sàng cho môi trường thực tế, nhưng là điều kiện cần trước khi triển khai hoặc bàn giao.

Trên môi trường AWS, việc kiểm tra tiếp tục với giá trị đầu ra CloudFormation, trạng thái ngăn xếp, cấu hình chạy, năm bảng dữ liệu và địa chỉ `/health`. Trên trình duyệt, cần xác nhận giao diện đang trỏ đúng User Pool và API, tuyến điều hướng được bảo vệ yêu cầu đăng nhập, lỗi có thông báo phù hợp và dữ liệu vẫn tồn tại sau khi tải lại trang. Chỉ kiểm tra các tích hợp được bật trong môi trường, không đánh dấu hoàn chỉnh dựa trên giao diện hoặc template chưa chạy.

## Vận hành, bảo mật và chi phí

Trước khi dùng môi trường cho demo hoặc đánh giá, cần rà lại:

- Lambda và tiến trình xử lý sử dụng vai trò thực thi theo đúng tài nguyên, không dùng khóa truy cập dài hạn.
- Bucket user-content luôn ở chế độ riêng tư; quyền tải lên và tải xuống đi qua URL có thời hạn sau khi kiểm tra quyền.
- CORS chỉ cho phép nguồn giao diện đã cấu hình; thông tin bí mật và token chỉ nằm phía máy chủ.
- Nhật ký không chứa JWT, OAuth token, URL ký sẵn, toàn bộ bản phiên âm hoặc nội dung tài liệu.
- CloudWatch theo dõi lỗi API, workflow thất bại, job bị kẹt, upload không hoàn tất và lỗi dịch vụ ngoài.
- Số lần thử lại được giới hạn và có cơ chế xử lý lặp an toàn; AWS Budgets hỗ trợ cảnh báo sớm khi chi phí tăng ngoài dự kiến.
- Chính sách lưu giữ, PITR hoặc deletion protection được bật theo môi trường sau khi xem xét chi phí và nhu cầu bảo vệ dữ liệu.

![Số liệu vận hành Lambda của CampusMeet trên CloudWatch](images/5-Workshop/campusmeet-evidence/cloudwatch-lambda-metrics.png)

*CloudWatch đã thu thập số lượt gọi, thời gian xử lý, lỗi và throttling của các Lambda CampusMeet trong khoảng thời gian được chọn. Số liệu vận hành cho biết tài nguyên đã phát sinh hoạt động, nhưng không tự chứng minh toàn bộ quy trình nghiệp vụ đã thành công.*

## Bằng chứng hạ tầng AI hiện tại

Kiểm tra read-only trên môi trường dev ghi nhận các thành phần sau:

| Thành phần | Trạng thái hoặc cấu hình đã xác minh |
| --- | --- |
| Step Functions `campusmeet-dev-ai-jobs` | `Active`, loại `Standard` |
| Lambda `campusmeet-dev-ai-worker` | `Active`, lần cập nhật gần nhất thành công, Node.js 22, 1024 MB, timeout 300 giây |
| Bedrock Knowledge Base | `campusmeet-dev-knowledge-v2`, trạng thái `ACTIVE` |
| Data source | `campusmeet-dev-sources`, trạng thái `AVAILABLE` |
| Vector store | S3 Vectors tại `ap-southeast-1`, được khai báo trong application stack |
| Generation | AI Worker được cấu hình gọi `https://bedrock-mantle.us-east-1.api.aws/v1` với `openai.gpt-oss-20b` |
| Thông tin xác thực model | Tham chiếu Secrets Manager tồn tại; workshop không đọc hoặc hiển thị giá trị secret |

Dashboard Mantle tại thời điểm kiểm tra có ghi nhận request và token của model `openai.gpt-oss-20b`. Dữ liệu sử dụng này cho thấy model endpoint đã được gọi trong tài khoản AWS, nhưng không đủ để quy mọi request cho CampusMeet hoặc xác nhận luồng retrieval–citation–authorization đã đạt đầu-cuối.

Các tài nguyên AI Worker, Knowledge Base, S3 Vectors, IAM role và alarm có tag CloudFormation/SAM tương ứng với `campusmeet-dev-app`. Tuy nhiên, application stack hiện ở trạng thái `UPDATE_ROLLBACK_FAILED` do `ApiLambdaRole`. Nhóm cần phục hồi stack trước khi tiếp tục cập nhật hạ tầng; không nên dựa vào việc từng tài nguyên con đang `Active` để kết luận toàn bộ stack khỏe mạnh.

## Kết quả hiện tại

Giao diện cho xác thực, nhóm, lời mời, thông báo và các thao tác tạo, xem, cập nhật, xóa cuộc họp đã kết nối API. Năm bảng DynamoDB đã được triển khai và xác minh trong `ap-southeast-1`. Các stack dữ liệu, xác thực và user-content đang `UPDATE_COMPLETE`; riêng application stack cần phục hồi trạng thái rollback. Phần xác thực/API và lõi cuộc họp đã có trên môi trường phát triển, trong khi một số kiểm thử nhanh về thao tác dữ liệu và phân quyền trên môi trường dùng chung vẫn còn thiếu điều kiện phù hợp.

Tải tệp, bản phiên âm, kho tri thức và trợ lý AI đã có thêm mã nguồn, hợp đồng dữ liệu, giao diện và tài nguyên AWS cho nhiều phạm vi như đọc, chỉnh sửa, phê duyệt bản phiên âm, giữ nguồn bất biến, hỏi đáp kèm trích dẫn, tạo bản nháp và phân tích tiến độ. Kiểm tra Google đã đi đến trạng thái cần kết nối lại nhưng chưa tạo được Meet URL; AI đã có model usage và pipeline control-plane nhưng chưa có bằng chứng đầy đủ cho retrieval, citation và xác nhận xuyên suốt. Vì vậy toàn hệ thống chưa được xem là sẵn sàng cho môi trường thực tế.

## Bài học rút ra

- Bắt đầu từ hành trình người dùng giúp xác định phạm vi rõ hơn so với bắt đầu từ danh sách dịch vụ.
- Xác thực và phân quyền phải là hai bước riêng; tích hợp ngoài không được làm gián đoạn quy trình cốt lõi.
- Phiên âm và AI chỉ tạo giá trị khi nguồn có thể truy vết, kết quả được xác nhận và từng mức kiểm thử được ghi nhận đúng thực tế.

## Những điểm cần tiếp tục kiểm chứng

- Tình huống quyền truy cập giữa nhiều nhóm và vai trò.
- Luồng tải tài liệu trên môi trường sử dụng thực tế.
- Phục hồi `campusmeet-dev-app` khỏi `UPDATE_ROLLBACK_FAILED` và rà lại change set trước lần triển khai tiếp theo.
- Đồng bộ Google Calendar và Google Meet sau khi tài khoản thử nghiệm được cấp đúng quyền OAuth.
- Xử lý âm thanh và phiên âm xuyên suốt.
- Luồng AI đầu-cuối từ nguồn đã duyệt đến retrieval, nguồn dẫn, phản hồi Mantle và bước xác nhận.
- Tác động cross-Region của Mantle đối với dữ liệu được gửi đi, độ trễ, quota và chi phí.
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
