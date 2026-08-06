---
title: "Bản đề xuất"
date: 2026-06-29
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# CampusMeet

## Nền tảng Serverless hỗ trợ quản lý quy trình cuộc họp cho nhóm học tập và dự án

### 1. Tóm tắt điều hành

CampusMeet là nền tảng hỗ trợ quản lý quy trình trước, trong và sau cuộc họp dành cho các nhóm học tập, đồ án và dự án quy mô nhỏ.

Trong thực tế, thông tin liên quan đến một cuộc họp thường được phân tán trên nhiều công cụ như ứng dụng nhắn tin, lịch cá nhân, tài liệu và bảng quản lý công việc. Điều này gây khó khăn trong việc quản lý thành viên, kiểm soát quyền truy cập, theo dõi lịch họp và tổng hợp kết quả sau cuộc họp.

CampusMeet được đề xuất như một ứng dụng web tập trung, sử dụng kiến trúc Serverless trên AWS. Phạm vi cốt lõi của hệ thống gồm:

- Đăng ký, xác nhận tài khoản và đăng nhập.
- Quản lý nhóm, thành viên và lời mời tham gia nhóm.
- Tạo, xem, cập nhật và hủy cuộc họp.
- Quản lý thông báo trong ứng dụng.
- Phân quyền theo vai trò và phạm vi nhóm.
- Lưu trữ dữ liệu bằng mô hình 5 bảng Amazon DynamoDB.
- Theo dõi log và lỗi hệ thống bằng Amazon CloudWatch.

Kiến trúc chính sử dụng Amazon Cognito, Amazon API Gateway, AWS Lambda, Amazon DynamoDB, AWS Identity and Access Management và Amazon CloudWatch. Hạ tầng được định nghĩa bằng AWS SAM và AWS CloudFormation để có thể kiểm tra, triển khai và quản lý theo quy trình thống nhất.

Mục tiêu của đề xuất là xây dựng một luồng Serverless đầu-cuối có khả năng kiểm thử, đáp ứng các yêu cầu về xác thực, phân quyền và quản lý dữ liệu cuộc họp, đồng thời tạo nền tảng để mở rộng thêm công việc, biên bản, transcript và các chức năng AI trong tương lai.

### 2. Tuyên bố vấn đề

#### Vấn đề hiện tại

Các nhóm học tập và dự án nhỏ thường sử dụng nhiều công cụ riêng biệt để tổ chức công việc liên quan đến cuộc họp:

- Ứng dụng nhắn tin để trao đổi và gửi lời mời.
- Lịch cá nhân để ghi nhớ thời gian họp.
- Tài liệu riêng để lưu nội dung và biên bản.
- Bảng công việc để theo dõi nhiệm vụ sau cuộc họp.

Việc sử dụng nhiều công cụ rời rạc dẫn đến các hạn chế:

- Thông tin cuộc họp bị phân tán và khó tra cứu.
- Không có cơ chế phân quyền thống nhất theo nhóm.
- Khó xác định ai được phép tạo, sửa hoặc hủy cuộc họp.
- Thành viên dễ bỏ sót lời mời và thông báo.
- Dữ liệu nhóm, cuộc họp và công việc không được quản lý tập trung.
- Khó mở rộng transcript, biên bản hoặc trợ lý AI mà vẫn bảo đảm quyền truy cập.

#### Giải pháp đề xuất

CampusMeet cung cấp một nền tảng web tập trung, trong đó người dùng được xác thực bằng Amazon Cognito trước khi truy cập hệ thống.

Frontend gửi JWT đến Amazon API Gateway. API Gateway kiểm tra token trước khi chuyển request đến AWS Lambda. Backend tiếp tục kiểm tra membership và vai trò của người dùng đối với từng nhóm hoặc cuộc họp trước khi thực hiện nghiệp vụ.

Dữ liệu được tổ chức trong 5 bảng DynamoDB:

- `identity`: người dùng, tùy chọn cá nhân và thông báo.
- `collaboration`: nhóm, thành viên, lời mời và lịch sử hoạt động.
- `meeting-data`: cuộc họp và các dữ liệu liên quan.
- `task-data`: công việc và lịch sử thay đổi.
- `ai-work`: trạng thái xử lý AI và dữ liệu điều phối mở rộng.

Việc sử dụng 5 bảng vật lý giúp giảm số lượng tài nguyên cần quản lý nhưng vẫn duy trì ranh giới rõ ràng giữa các nhóm dữ liệu nghiệp vụ.

#### Phạm vi giải pháp

Phạm vi cốt lõi của workshop tập trung vào:

- Xác thực người dùng.
- Quản lý nhóm và thành viên.
- Quản lý lời mời và thông báo.
- Quản lý cuộc họp.
- Phân quyền tại backend.
- Nền tảng dữ liệu DynamoDB.
- Hạ tầng Serverless.
- Kiểm thử, logging và giám sát.
- Quy trình triển khai và dọn dẹp tài nguyên.

CampusMeet không xây dựng hệ thống gọi video riêng và không thay thế Google Meet. Khả năng tích hợp Google Calendar hoặc Google Meet được xem là phần mở rộng của nền tảng.

Live transcription, xử lý tài liệu và trợ lý AI cũng là các hướng phát triển tiếp theo, không phải điều kiện bắt buộc để hoàn thành phạm vi cốt lõi của workshop.

#### Lợi ích dự kiến

- Tập trung hóa dữ liệu nhóm và cuộc họp.
- Giảm thời gian trao đổi và tổng hợp thông tin thủ công.
- Kiểm soát quyền truy cập nhất quán.
- Dễ theo dõi lời mời, thành viên và trạng thái cuộc họp.
- Có thể mở rộng theo nhu cầu nhờ kiến trúc Serverless.
- Tạo nền tảng cho biên bản, công việc, transcript và AI.
- Giảm chi phí vận hành ban đầu nhờ mô hình thanh toán theo mức sử dụng.

### 3. Kiến trúc giải pháp

CampusMeet sử dụng kiến trúc web Serverless với các lớp chính gồm giao diện người dùng, xác thực, API, xử lý nghiệp vụ, lưu trữ dữ liệu và giám sát.

#### Luồng xử lý chính

1. Người dùng truy cập giao diện CampusMeet.
2. Amazon Cognito thực hiện đăng ký, xác nhận tài khoản và đăng nhập.
3. Frontend nhận JWT và gửi token kèm request đến API.
4. Amazon API Gateway kiểm tra JWT trước khi gọi AWS Lambda.
5. Lambda xử lý nghiệp vụ và kiểm tra quyền truy cập của người dùng.
6. Backend đọc hoặc ghi dữ liệu trong Amazon DynamoDB.
7. Amazon CloudWatch thu thập log, lỗi và thông tin vận hành.
8. AWS SAM và CloudFormation quản lý tài nguyên hạ tầng.

#### Các dịch vụ AWS sử dụng

- **Amazon Cognito**: Quản lý tài khoản, đăng nhập và phát JWT cho người dùng.
- **Amazon API Gateway**: Cung cấp HTTP API và xác thực JWT trước khi chuyển request đến backend.
- **AWS Lambda**: Xử lý nghiệp vụ liên quan đến nhóm, lời mời, thông báo và cuộc họp.
- **Amazon DynamoDB**: Lưu dữ liệu ứng dụng theo mô hình 5 bảng vật lý.
- **AWS IAM**: Quản lý quyền truy cập giữa Lambda và các tài nguyên AWS.
- **Amazon CloudWatch**: Lưu log, theo dõi lỗi và hỗ trợ giám sát hệ thống.
- **AWS CloudFormation**: Quản lý tài nguyên dưới dạng Infrastructure as Code.
- **AWS SAM**: Hỗ trợ xây dựng, kiểm tra và triển khai ứng dụng Serverless.
- **Amazon S3 và Amazon CloudFront**: Có thể được sử dụng để lưu trữ và phân phối frontend trong kiến trúc triển khai hoàn chỉnh.

#### Thiết kế thành phần

##### Giao diện người dùng

Frontend được xây dựng bằng React, TypeScript và Vite, cung cấp các màn hình chính:

- Đăng ký, xác nhận tài khoản và đăng nhập.
- Khôi phục mật khẩu.
- Danh sách nhóm và quản lý thành viên.
- Gửi và xử lý lời mời tham gia nhóm.
- Danh sách, chi tiết và biểu mẫu cuộc họp.
- Thông báo trong ứng dụng.

##### Xác thực và phân quyền

Amazon Cognito chịu trách nhiệm xác minh danh tính người dùng và phát JWT. Frontend sử dụng token khi gọi API.

JWT hợp lệ chỉ chứng minh danh tính và không tự động cho phép truy cập mọi dữ liệu. Backend vẫn phải kiểm tra người dùng có phải thành viên hợp lệ của nhóm và có vai trò phù hợp với thao tác đang thực hiện hay không.

##### API và xử lý nghiệp vụ

Amazon API Gateway là điểm truy cập vào backend. AWS Lambda xử lý các use case của hệ thống và thực hiện:

- Kiểm tra dữ liệu đầu vào.
- Kiểm tra membership và role.
- Đọc và ghi dữ liệu thông qua repository.
- Xử lý lỗi theo định dạng thống nhất.
- Ghi log cần thiết cho việc chẩn đoán.

##### Lưu trữ dữ liệu

| Bảng | Dữ liệu chính |
| --- | --- |
| `identity` | Người dùng, tùy chọn và thông báo |
| `collaboration` | Nhóm, thành viên, lời mời và audit event |
| `meeting-data` | Cuộc họp và dữ liệu liên quan |
| `task-data` | Công việc và lịch sử thay đổi |
| `ai-work` | Job xử lý, nguồn kiến thức và dữ liệu AI mở rộng |

Composite key, Global Secondary Index và TTL được sử dụng theo từng nhu cầu truy xuất cụ thể.

##### Giám sát

Amazon CloudWatch được sử dụng để:

- Lưu log của Lambda.
- Theo dõi lỗi khi xử lý request.
- Hỗ trợ kiểm tra lỗi xác thực và phân quyền.
- Theo dõi độ trễ và trạng thái hoạt động.
- Hỗ trợ chẩn đoán trong quá trình triển khai và kiểm thử.

### 4. Triển khai kỹ thuật

#### Các giai đoạn triển khai

##### Giai đoạn 1: Nghiên cứu nền tảng AWS

- Làm quen với AWS Management Console.
- Tìm hiểu IAM, VPC, EC2, Lambda và RDS.
- Thiết lập AWS Budgets.
- Thực hành dọn dẹp tài nguyên.
- Tìm hiểu kiến trúc Serverless.

##### Giai đoạn 2: Xác định yêu cầu và kiến trúc

- Xác định vấn đề mà CampusMeet cần giải quyết.
- Xác định phạm vi chức năng.
- Thiết kế luồng frontend, API và database.
- Lựa chọn Amazon Cognito cho xác thực.
- Lựa chọn API Gateway và Lambda cho backend.
- Xác định DynamoDB là cơ sở dữ liệu chính.

##### Giai đoạn 3: Xây dựng nền tảng xác thực

- Xây dựng giao diện đăng ký và đăng nhập.
- Xác nhận tài khoản qua email.
- Khôi phục mật khẩu.
- Thiết lập protected route.
- Tạo API client gửi access token.
- Cấu hình API Gateway JWT Authorizer.
- Xây dựng auth stack bằng AWS SAM.

##### Giai đoạn 4: Xây dựng nền tảng dữ liệu

- Xác định các nhóm dữ liệu của hệ thống.
- Thiết kế partition key và sort key.
- Xác định các Global Secondary Index cần thiết.
- Thiết lập TTL cho dữ liệu có thời hạn.
- Xây dựng 5 bảng DynamoDB bằng CloudFormation.
- Cấu hình quyền IAM theo nguyên tắc quyền tối thiểu.
- Xây dựng script kiểm tra cấu hình bảng.

##### Giai đoạn 5: Xây dựng chức năng cốt lõi

- Quản lý nhóm và thành viên.
- Gửi và xử lý lời mời.
- Quản lý thông báo.
- Tạo, xem, cập nhật và hủy cuộc họp.
- Kiểm tra quyền truy cập theo nhóm.
- Kết nối frontend với API.

##### Giai đoạn 6: Kiểm thử và hoàn thiện tài liệu

- Chạy lint và typecheck.
- Chạy unit test.
- Kiểm tra frontend build.
- Validate AWS SAM template.
- Kiểm tra cấu hình DynamoDB.
- Kiểm tra luồng xác thực đầu-cuối.
- Viết hướng dẫn triển khai và cleanup.
- Chuẩn hóa tài liệu workshop.

#### Yêu cầu kỹ thuật

- Node.js 22 LTS và npm 10 trở lên.
- React, TypeScript và Vite.
- AWS CLI và AWS SAM CLI.
- Tài khoản AWS có quyền triển khai các tài nguyên cần thiết.
- Git và GitHub để quản lý mã nguồn.

#### Kiểm tra chất lượng

Các bước kiểm tra chính gồm:

```bash
npm run lint
npm run typecheck
npm run test
npm run build
npm run format:check
npm run sam:validate:data
```

AWS SAM template cần được validate trước khi tạo hoặc cập nhật CloudFormation stack. Các thay đổi hạ tầng cần được xem trước bằng change set để giảm nguy cơ thay đổi hoặc xóa nhầm tài nguyên.

#### Nguyên tắc bảo mật

- Không sử dụng root account cho công việc hằng ngày.
- Bật MFA cho tài khoản quản trị.
- Không commit access key, secret key, token hoặc password.
- Lambda sử dụng IAM execution role.
- Giới hạn quyền IAM theo tài nguyên và hành động cần thiết.
- Backend kiểm tra membership và role trên từng thao tác.
- Không ghi JWT, OAuth token hoặc dữ liệu nhạy cảm vào log.
- Không đưa dữ liệu người dùng thật vào source code hoặc tài liệu công khai.

### 5. Lộ trình và mốc triển khai

| Giai đoạn | Nội dung |
| --- | --- |
| Nghiên cứu AWS | AWS Console, Budgets, IAM, VPC, EC2 và Serverless |
| Phân tích dự án | Xác định vấn đề, phạm vi và kiến trúc CampusMeet |
| Xác thực | Landing page, Cognito, protected route và JWT Authorizer |
| Hạ tầng xác thực | AWS SAM, API Gateway, Lambda, IAM và CloudWatch |
| Nền tảng dữ liệu | Thiết kế và xây dựng 5 bảng DynamoDB |
| Chức năng cốt lõi | Nhóm, thành viên, lời mời, thông báo và cuộc họp |
| Kiểm thử | Lint, typecheck, test, build và infrastructure validation |
| Hoàn thiện workshop | Tài liệu triển khai, bảo mật, giám sát và cleanup |

Sau khi hoàn thành phạm vi cốt lõi, hệ thống có thể được mở rộng theo các hướng:

- Đồng bộ lịch với Google Calendar.
- Tạo liên kết Google Meet.
- Quản lý biên bản và action item.
- Upload file và recording.
- Live transcription.
- Tìm kiếm nội dung bằng Amazon Bedrock.
- Trợ lý AI có citation.

### 6. Ước tính ngân sách

CampusMeet sử dụng kiến trúc Serverless và ưu tiên các dịch vụ có mô hình thanh toán theo mức sử dụng. Vì hệ thống vẫn đang trong quá trình phát triển, chi phí trong giai đoạn này được xem là ước tính sơ bộ và sẽ được cập nhật sau khi xác định rõ số lượng người dùng, lưu lượng API, dung lượng dữ liệu và phạm vi triển khai thực tế.

Có thể sử dụng [AWS Pricing Calculator](https://calculator.aws/) để xây dựng bảng chi phí chi tiết khi các thông số vận hành được hoàn thiện.

#### Các dịch vụ chính cần tính chi phí

| Dịch vụ | Mục đích sử dụng | Yếu tố ảnh hưởng đến chi phí |
| --- | --- | --- |
| Amazon Cognito | Đăng ký, đăng nhập và quản lý danh tính | Số người dùng hoạt động hằng tháng |
| Amazon API Gateway | Cung cấp HTTP API cho frontend | Tổng số API request |
| AWS Lambda | Xử lý nghiệp vụ | Số lần gọi, thời gian thực thi và bộ nhớ |
| Amazon DynamoDB | Lưu dữ liệu ứng dụng | Số request đọc/ghi, dung lượng và chỉ mục |
| Amazon CloudWatch | Log và giám sát | Dung lượng log và thời gian lưu trữ |
| Amazon S3 | Lưu frontend hoặc tệp tải lên | Dung lượng và số request |
| Amazon CloudFront | Phân phối frontend và nội dung tĩnh | Lưu lượng truyền dữ liệu |
| Amazon SES | Gửi email thông báo trong giai đoạn mở rộng | Số email được gửi |

Các dịch vụ như Amazon Transcribe, Amazon Bedrock, AWS Step Functions và Bedrock Knowledge Bases chỉ được đưa vào dự toán khi các chức năng transcript và AI được triển khai.

#### Giả định cho môi trường phát triển

- Số lượng người dùng thử nghiệm nhỏ.
- Lưu lượng API thấp.
- DynamoDB sử dụng chế độ `PAY_PER_REQUEST`.
- Lambda chỉ chạy khi có request.
- CloudWatch Log Group có thời gian lưu phù hợp.
- Tệp thử nghiệm được xóa sau khi hoàn thành.
- Chỉ duy trì một môi trường phát triển dùng chung.
- Không chạy các chức năng AI hoặc transcription liên tục.

Trong điều kiện phục vụ học tập và kiểm thử, nhiều dịch vụ có thể nằm trong giới hạn AWS Free Tier hoặc phát sinh chi phí thấp. Tuy nhiên, Free Tier không được xem là cam kết rằng hệ thống luôn miễn phí.

#### Biện pháp kiểm soát chi phí

- Thiết lập AWS Budgets và cảnh báo chi phí.
- Kiểm tra đúng AWS Account và Region trước khi triển khai.
- Gắn tag cho tài nguyên CampusMeet.
- Thiết lập thời gian lưu log phù hợp.
- Sử dụng DynamoDB Query theo key thay vì `Scan`.
- Xem trước CloudFormation change set trước khi deploy.
- Dọn dẹp stack và tài nguyên thử nghiệm không còn sử dụng.
- Không lưu file hoặc recording thử nghiệm lâu hơn cần thiết.
- Tách tài nguyên dữ liệu khỏi tài nguyên ứng dụng để hạn chế xóa nhầm dữ liệu.

Bảng chi phí chính thức sẽ được cập nhật khi kiến trúc triển khai, số lượng người dùng và các chức năng mở rộng của CampusMeet được hoàn thiện.

### 7. Đánh giá rủi ro

#### Ma trận rủi ro

| Rủi ro | Mức ảnh hưởng | Khả năng xảy ra |
| --- | --- | --- |
| Cấu hình IAM cấp quyền rộng hơn cần thiết | Cao | Trung bình |
| Backend kiểm tra membership hoặc role chưa đầy đủ | Cao | Trung bình |
| Thiết kế DynamoDB không đáp ứng đúng access pattern | Cao | Trung bình |
| Thay đổi CloudFormation ảnh hưởng đến dữ liệu hiện có | Cao | Thấp |
| Frontend và backend sử dụng API contract không đồng bộ | Trung bình | Trung bình |
| Log hoặc dữ liệu cuộc họp chứa thông tin nhạy cảm | Cao | Trung bình |
| Phát sinh chi phí do log hoặc tài nguyên không được dọn dẹp | Trung bình | Trung bình |
| Chức năng Google Workspace, transcript hoặc AI chưa ổn định | Trung bình | Trung bình |
| Phạm vi dự án mở rộng nhanh hơn tiến độ triển khai | Trung bình | Cao |

#### Chiến lược giảm thiểu

**Xác thực và phân quyền**

- Sử dụng Amazon Cognito và API Gateway JWT Authorizer để xác thực người dùng.
- Kiểm tra membership và role tại backend trên từng thao tác.
- Không xem JWT hợp lệ là quyền truy cập toàn bộ dữ liệu.
- Áp dụng IAM theo nguyên tắc quyền tối thiểu.

**Dữ liệu DynamoDB**

- Xác định access pattern trước khi thay đổi bảng hoặc GSI.
- Truy cập dữ liệu thông qua repository thay vì gọi DynamoDB trực tiếp từ handler.
- Sử dụng conditional write hoặc transaction cho các thao tác cần tính nhất quán.
- Kiểm tra schema, TTL và GSI trước và sau khi triển khai.

**Triển khai hạ tầng**

- Validate AWS SAM và CloudFormation template trước khi deploy.
- Xem change set trước khi cập nhật stack.
- Tách data stack khỏi application stack.
- Hạn chế thay đổi hoặc xóa trực tiếp tài nguyên trên AWS Console.
- Duy trì runbook triển khai, kiểm tra và rollback.

**Bảo mật dữ liệu**

- Không ghi JWT, OAuth token, secret hoặc nội dung cuộc họp nhạy cảm vào log.
- Không commit file `.env`, credential hoặc dữ liệu người dùng.
- Áp dụng consent và retention cho recording, transcript và tệp tải lên.
- Đặt S3 ở chế độ private và chỉ cấp quyền truy cập có thời hạn.

**Kiểm soát chất lượng và phạm vi**

- Chạy lint, typecheck, unit test và build trước khi tích hợp.
- Duy trì shared type và API contract giữa frontend và backend.
- Ưu tiên hoàn thiện xác thực, nhóm, thành viên và quản lý cuộc họp.
- Tách Google Workspace, transcription và AI thành các module mở rộng.
- Không để chức năng cốt lõi phụ thuộc hoàn toàn vào dịch vụ bên ngoài.

#### Kế hoạch dự phòng

- Rollback application stack về phiên bản ổn định khi triển khai lỗi.
- Giữ data stack độc lập để hạn chế nguy cơ mất dữ liệu.
- Khôi phục mã nguồn từ commit đã được kiểm thử.
- Sử dụng in-memory repository hoặc DynamoDB Local trong quá trình phát triển.
- Tạm vô hiệu hóa Google, transcription hoặc AI nếu dịch vụ ngoài gặp lỗi.
- Tiếp tục cho phép quản lý cuộc họp cốt lõi khi các chức năng mở rộng chưa khả dụng.
- Xóa và triển khai lại stack kiểm thử theo runbook khi cấu hình môi trường bị sai.

### 8. Kết quả kỳ vọng

Sau khi hoàn thành phạm vi workshop, CampusMeet dự kiến đạt được các kết quả:

- Xây dựng được ứng dụng Serverless có luồng frontend và backend rõ ràng.
- Hoàn thiện đăng ký, xác nhận tài khoản và đăng nhập bằng Amazon Cognito.
- Bảo vệ API bằng JWT Authorizer.
- Kiểm tra quyền truy cập theo membership và role.
- Quản lý được nhóm, thành viên, lời mời và thông báo.
- Thực hiện được các thao tác tạo, xem, cập nhật và hủy cuộc họp.
- Xây dựng được mô hình 5 bảng DynamoDB.
- Quản lý hạ tầng bằng AWS SAM và CloudFormation.
- Có quy trình kiểm thử, triển khai và dọn dẹp tài nguyên.
- Sử dụng CloudWatch để theo dõi và chẩn đoán lỗi.
- Có tài liệu workshop mô tả kiến trúc và quá trình triển khai đầu-cuối.

Về lâu dài, nền tảng có thể được mở rộng thêm quản lý công việc, biên bản cuộc họp, upload tài liệu, live transcription và trợ lý AI có nguồn dẫn, trong khi tiếp tục sử dụng chung cơ chế xác thực, dữ liệu và phân quyền của CampusMeet.
