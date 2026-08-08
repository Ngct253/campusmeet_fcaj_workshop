---
title: "Chuẩn bị môi trường và kiến trúc triển khai"
date: 2026-08-08
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Môi trường cần chuẩn bị

Kho mã nguồn CampusMeet sử dụng Node.js 22 và npm. Git được dùng để quản lý phiên bản; AWS CLI và AWS SAM CLI phục vụ việc xác nhận tài khoản, kiểm tra template và triển khai tài nguyên AWS. PowerShell hỗ trợ một số tập lệnh kiểm tra trong dự án.

Có thể xác nhận các công cụ và cài mã nguồn bằng những lệnh sau:

```powershell
node --version
npm --version
git --version
aws --version
sam --version

git clone https://github.com/Ngct253/CampusMeet.git
cd CampusMeet
npm ci
```

Các phiên bản công cụ cần phù hợp với cấu hình của kho mã nguồn. Việc cài thành công các gói phụ thuộc mới chỉ xác nhận môi trường cục bộ sẵn sàng; các bước kiểm tra mã nguồn, hạ tầng và môi trường AWS vẫn được thực hiện riêng trước khi bàn giao.

## Cấu trúc kho mã nguồn

Cấu trúc kho mã nguồn được tách theo trách nhiệm:

| Khu vực | Vai trò |
| --- | --- |
| `apps/web` | Giao diện React, tuyến điều hướng và các chức năng phía người dùng |
| `services/api` | API, xử lý nghiệp vụ, phân quyền và lớp truy cập dữ liệu |
| `services/ai-worker` | Chuẩn hóa nguồn, xử lý công việc AI và kết nối dịch vụ liên quan |
| `packages/shared` | Kiểu dữ liệu, enum và hợp đồng dùng chung |
| `infra` | Các template AWS SAM/CloudFormation |
| `scripts` | Kiểm tra cấu hình, hạ tầng và luồng xử lý |
| `docs` | Đặc tả, kiến trúc, mô hình dữ liệu và runbook |

Cách tổ chức này giúp thay đổi hợp đồng dữ liệu hoặc quy tắc quyền được dùng nhất quán giữa frontend, API và kiểm thử. Tài nguyên AWS được quản lý bằng template thay vì tạo thủ công rồi phụ thuộc vào cấu hình riêng của từng máy. Trước khi kết nối với môi trường dùng chung, thay đổi vẫn cần vượt qua các bước kiểm tra được trình bày tại phần bàn giao.

## Kết nối đúng môi trường AWS

Môi trường phát triển của CampusMeet sử dụng Region `ap-southeast-1`. Trước khi xem hoặc cập nhật một ngăn xếp, người triển khai cần xác nhận đúng tài khoản và Region:

```powershell
aws sts get-caller-identity
aws configure get region
```

Phần ứng dụng, dữ liệu, lưu trữ nguồn và truy xuất tri thức đặt tại `ap-southeast-1`. Riêng bước sinh nội dung được AI Worker cấu hình gọi Bedrock Mantle tại `us-east-1`. Việc tách Region này cần được xem xét về quyền truy cập, dữ liệu được phép gửi đi, độ trễ, quota và chi phí; nó không làm thay đổi Region chính của dữ liệu nghiệp vụ.

Sau khi ngăn xếp xác thực/API được triển khai, giao diện sử dụng ba giá trị đầu ra công khai:

```dotenv
VITE_COGNITO_USER_POOL_ID=...
VITE_COGNITO_USER_POOL_CLIENT_ID=...
VITE_API_BASE_URL=...
```

Các giá trị trên giúp giao diện kết nối đúng Cognito và API; chúng không phải thông tin bí mật. Khóa truy cập AWS, khóa bí mật Google, OAuth token hoặc thông tin phía máy chủ không được đưa vào biến `VITE_*`, mã nguồn hay Git.

## Kiến trúc triển khai

![Sơ đồ kiến trúc tổng quan của CampusMeet](images/5-Workshop/5.3-Architecture/architecture-diagram.png?v=2)

Sơ đồ thể hiện các lớp từ giao diện người dùng, danh tính, API, xử lý nghiệp vụ và lưu trữ đến các dịch vụ hỗ trợ và theo dõi vận hành. Luồng cuộc họp cốt lõi sử dụng Cognito, API Gateway, Lambda và DynamoDB; S3, lịch, phiên âm và AI được kết nối theo nhu cầu của từng chức năng.

Kiến trúc mục tiêu không đồng nghĩa mọi nhánh đã hoàn chỉnh ở cùng một mức. Workshop phân biệt phần đã được triển khai hoặc kiểm chứng với phần vẫn cần tiếp tục thử nghiệm trên môi trường dùng chung.

## Trách nhiệm của các dịch vụ

| Thành phần | Vai trò trong CampusMeet |
| --- | --- |
| React, TypeScript và Vite | Xây dựng giao diện web và quản lý hành trình phía người dùng |
| Amazon Cognito | Đăng ký, xác nhận tài khoản, đăng nhập và phát JWT |
| API Gateway HTTP API | Xác minh JWT trước khi chuyển yêu cầu đến backend |
| AWS Lambda | Thực thi tình huống nghiệp vụ, kiểm tra quyền và điều phối truy cập dữ liệu |
| Amazon DynamoDB | Lưu dữ liệu nghiệp vụ trong năm bảng vật lý |
| Amazon S3 | Lưu tệp người dùng trong bucket riêng tư và phục vụ tài nguyên web theo kiến trúc mục tiêu |
| EventBridge Scheduler | Kích hoạt lịch nhắc tại thời điểm phù hợp |
| Step Functions và AI Worker | Điều phối AIJob, thử lại có giới hạn, chuẩn hóa nguồn và cập nhật trạng thái xử lý |
| Bedrock Knowledge Base, Cohere embedding và S3 Vectors | Truy xuất nguồn đã duyệt tại `ap-southeast-1` |
| Bedrock Mantle | Sinh câu trả lời hoặc bản nháp tại `us-east-1` bằng model `openai.gpt-oss-20b` |
| Amazon Transcribe | Hỗ trợ xử lý âm thanh và phiên âm khi luồng tương ứng được bật |
| CloudWatch, SNS và SES | Theo dõi vận hành, gửi cảnh báo và hỗ trợ thông báo |

Bảng trên chỉ xác định vai trò kiến trúc của từng dịch vụ. Luồng Cognito–API–Lambda và cách dữ liệu được kiểm tra quyền trước khi lưu được trình bày chi tiết tại phần 5.4 để tránh lặp lại cùng một quy trình ở nhiều nơi.

## Luồng đồng bộ và bất đồng bộ

Trong luồng đồng bộ, frontend gọi HTTP API kèm JWT; API Gateway xác minh token, Lambda kiểm tra tư cách thành viên và vai trò rồi mới truy vấn DynamoDB. Kết quả được trả về cho giao diện trong cùng yêu cầu. Trình duyệt không tự khai báo quyền và không truy cập trực tiếp các bảng dữ liệu.

Các công việc phụ thuộc dịch vụ ngoài hoặc cần nhiều thời gian được tách khỏi yêu cầu chính. DynamoDB Stream có thể kích hoạt worker đồng bộ Google, EventBridge Scheduler gọi reminder theo lịch, còn upload sử dụng URL có thời hạn để trình duyệt gửi tệp trực tiếp tới S3 riêng tư.

Luồng AI được tổ chức theo hướng sau:

```text
Nguồn đã được kiểm tra và phê duyệt
  → chuẩn hóa dưới vùng lưu trữ kb/ tại Singapore
  → Bedrock Knowledge Base tạo embedding và truy xuất qua S3 Vectors
  → backend lọc theo nhóm, cuộc họp, nguồn và phiên bản
  → AI Worker gọi Bedrock Mantle tại N. Virginia
  → trả câu trả lời hoặc bản nháp có nguồn dẫn
  → người dùng xem lại và xác nhận trước khi ghi dữ liệu chính thức
```

Step Functions quản lý trạng thái và lần thử lại của công việc AI. Sự tồn tại của state machine, Knowledge Base hoặc endpoint model chỉ chứng minh hạ tầng đã được cấu hình; chất lượng truy xuất, nguồn dẫn và quyền truy cập vẫn cần kiểm tra đầu-cuối.

## Trình tự thiết lập môi trường

1. Xác nhận tài khoản AWS, Region và quyền triển khai.
2. Kiểm tra template dữ liệu, xem trước thay đổi rồi triển khai và xác minh năm bảng DynamoDB.
3. Triển khai hoặc cập nhật ngăn xếp xác thực/API với đúng tiền tố bảng và nguồn frontend được cho phép.
4. Lấy User Pool ID, User Pool Client ID và API URL từ đầu ra CloudFormation để cấu hình giao diện.
5. Thiết lập tài nguyên nội dung và điều phối nếu môi trường sử dụng upload, nhắc lịch, phiên âm hoặc AI.
6. Kiểm tra `/health`, đăng nhập, tuyến điều hướng được bảo vệ, nhật ký và các chức năng được bật trong môi trường đó.

Trình tự trên giúp các phần phụ thuộc được thiết lập theo thứ tự rõ ràng. Nó là phương pháp triển khai và kiểm chứng, không phải khẳng định toàn bộ kiến trúc đã sẵn sàng cho môi trường thực tế.
