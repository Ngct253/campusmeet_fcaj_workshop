---
title: "Mục tiêu, chuẩn bị và quyền truy cập"
date: 2026-08-08
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Mục tiêu của CampusMeet

- Giảm tình trạng thông tin bị phân tán.
- Giúp cuộc họp có mục tiêu và nội dung chuẩn bị rõ ràng.
- Lưu lại kết quả thay vì chỉ kết thúc ở phần trao đổi.
- Chuyển đầu việc sau họp thành nhiệm vụ có thể theo dõi.
- Hỗ trợ tìm lại thông tin mà vẫn tôn trọng quyền truy cập.

## Chuẩn bị môi trường kỹ thuật

Kho mã nguồn hiện tại sử dụng Node.js 22 và npm. AWS CLI và AWS SAM CLI chỉ cần khi xác thực hoặc triển khai tài nguyên AWS; PowerShell được dùng cho một số tập lệnh kiểm tra. Có thể kiểm tra công cụ và cài mã nguồn bằng các lệnh sau:

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

Trước khi kết nối với môi trường dùng chung, thay đổi cần vượt qua các bước kiểm tra mã nguồn, kiểu dữ liệu, kiểm thử, tạo bản dựng và cấu hình hạ tầng được trình bày tại phần bàn giao. Các bước này không thay thế kiểm thử trên AWS nhưng giúp loại bỏ lỗi sớm trước khi triển khai.

## Kết nối môi trường AWS

Môi trường phát triển của CampusMeet sử dụng Region `ap-southeast-1`. Trước khi cập nhật stack, người triển khai cần xác nhận đúng tài khoản và Region để tránh tạo tài nguyên nhầm môi trường:

```powershell
aws sts get-caller-identity
aws configure get region
```

Sau khi ngăn xếp xác thực/API được triển khai, giao diện sử dụng ba giá trị đầu ra công khai:

```dotenv
VITE_COGNITO_USER_POOL_ID=...
VITE_COGNITO_USER_POOL_CLIENT_ID=...
VITE_API_BASE_URL=...
```

Các giá trị trên giúp giao diện kết nối đúng Cognito và API; chúng không phải thông tin bí mật. Khóa truy cập AWS, khóa bí mật Google, OAuth token hoặc thông tin phía máy chủ không được đưa vào biến `VITE_*`, mã nguồn hay Git.

## Quyền truy cập

CampusMeet tổ chức quyền theo nhóm. Có tài khoản không có nghĩa là được xem mọi nội dung. Quản trị viên có thể điều phối nhóm và thành viên; thành viên chỉ sử dụng các chức năng phù hợp với vai trò của mình. Người ngoài nhóm không được xem cuộc họp, tài liệu hoặc công việc nội bộ.

Ở cấp hạ tầng cũng cần phân biệt quyền của người triển khai với vai trò thực thi của dịch vụ. Người triển khai chỉ cần quyền cập nhật các ngăn xếp thuộc phạm vi công việc. Lambda hoặc tiến trình xử lý sử dụng vai trò riêng để truy cập đúng bảng, bucket và dịch vụ cần thiết. Việc ứng dụng cần nhiều dịch vụ AWS không có nghĩa mọi thành viên hoặc mọi Lambda đều cần `AdministratorAccess`.

Quyền được xác định từ thông tin thành viên trong hệ thống, không dựa vào vai trò do người dùng tự khai báo. Nội dung do AI gợi ý cũng không tự động trở thành quyết định chính thức mà cần được người có quyền xem lại.

## Phương pháp xác định quyền

Mỗi thao tác được xem xét theo bốn câu hỏi:

1. Tài khoản đã đăng nhập và còn hợp lệ hay chưa?
2. Người đó có phải thành viên đang hoạt động của nhóm hay không?
3. Vai trò hiện tại có cho phép hành động cụ thể hay không?
4. Dữ liệu có đúng nhóm, cuộc họp và phiên bản cần thao tác hay không?

Cách kiểm tra này tách danh tính khỏi quyền sử dụng tài nguyên: đăng nhập hợp lệ không cho phép đọc dữ liệu nhóm khác, còn quyền xem cuộc họp không mặc nhiên cho phép sửa hoặc phê duyệt nội dung. Thành viên chỉ tiếp cận dữ liệu sau khi tham gia nhóm hợp lệ; người tổ chức hoặc quản trị viên xử lý các bước cần thẩm quyền. Tài liệu và bản phiên âm dùng cho AI vẫn giữ phạm vi nhóm, cuộc họp và nguồn ban đầu để câu trả lời không sử dụng nội dung ngoài quyền của người hỏi.

## Nguyên tắc bảo vệ nội dung

Quyền được kiểm tra theo từng nhóm và cuộc họp, không chỉ tại thời điểm đăng nhập. Tài liệu, bản phiên âm, biên bản, nhiệm vụ và nguồn dùng cho AI đều kế thừa phạm vi truy cập của nội dung gốc.

## Tiêu chí sử dụng

CampusMeet có giá trị khi người dùng hoàn thành được hành trình cơ bản: đăng nhập, tham gia nhóm, tạo hoặc xem cuộc họp, tiếp cận tài liệu, ghi nhận kết quả và theo dõi việc được giao. Các chức năng nâng cao chỉ nên hỗ trợ, không làm luồng cốt lõi trở nên khó hiểu.
