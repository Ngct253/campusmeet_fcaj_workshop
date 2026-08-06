---
title: "Điều kiện chuẩn bị"
date: 2026-07-27
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Điều kiện chuẩn bị

Trước khi bắt đầu triển khai CampusMeet, cần chuẩn bị môi trường phát triển cục bộ, tài khoản AWS và quyền truy cập phù hợp. Các bước trong phần này bám theo cấu hình và quy trình đang được sử dụng trong mã nguồn CampusMeet.

## 1. Tài khoản và môi trường AWS

Cần chuẩn bị:

- Một tài khoản AWS dùng cho môi trường phát triển của CampusMeet.
- Mã tài khoản AWS đã được nhóm xác nhận.
- Khu vực AWS `ap-southeast-1`.
- Một người phụ trách triển khai các thành phần cần quyền tạo vai trò IAM.
- Mỗi thành viên sử dụng một IAM user riêng.
- Cảnh báo ngân sách đã được cấu hình trước khi triển khai tài nguyên.

Không sử dụng tài khoản root cho công việc hằng ngày. Không dùng chung IAM user và không tạo hoặc chia sẻ khóa truy cập dài hạn cho người dùng.

Môi trường AWS dùng chung nên được sử dụng cho kiểm thử tích hợp và xác minh triển khai. Việc phát triển chức năng hằng ngày được thực hiện trên máy cá nhân bằng kho dữ liệu trong bộ nhớ hoặc DynamoDB Local khi phù hợp.

## 2. Công cụ cần cài đặt

| Công cụ | Yêu cầu | Mục đích |
| --- | --- | --- |
| Node.js | Phiên bản 22 LTS | Chạy frontend, backend và các công cụ JavaScript/TypeScript |
| npm | Phiên bản 10 trở lên | Cài đặt thư viện và chạy các lệnh của dự án |
| Git | Phiên bản đang được hỗ trợ | Tải và quản lý mã nguồn |
| AWS CLI | Phiên bản 2.32.0 trở lên | Đăng nhập, kiểm tra tài khoản và thao tác với AWS |
| AWS SAM CLI | Đã cài đặt | Kiểm tra, xây dựng và triển khai các mẫu SAM |
| PowerShell | Đã cài đặt | Chạy các lệnh và tập lệnh kiểm tra AWS hiện có |

Kiểm tra các công cụ sau khi cài đặt:

```powershell
node --version
npm --version
git --version
aws --version
sam --version
```

Không tiếp tục nếu Node.js, npm, AWS CLI hoặc AWS SAM CLI chưa được nhận diện đúng trên dòng lệnh.

## 3. Tải mã nguồn CampusMeet

Tải mã nguồn và chuyển vào thư mục dự án:

```powershell
git clone https://github.com/Ngct253/CampusMeet.git
cd CampusMeet
```

Cài đặt các thư viện của toàn bộ workspace:

```powershell
npm install
```

CampusMeet được tổ chức dưới dạng monorepo với các thư mục chính:

```text
apps/web/          Giao diện React và Vite
services/api/      Lambda API và các lớp nghiệp vụ
packages/shared/   Kiểu dữ liệu, enum và DTO dùng chung
infra/             Các mẫu AWS SAM và CloudFormation
scripts/           Tập lệnh kiểm tra tài nguyên AWS
docs/              Tài liệu yêu cầu, kiến trúc và triển khai
```

## 4. Kiểm tra mã nguồn trước khi triển khai

Chạy các lệnh kiểm tra cơ bản từ thư mục gốc của CampusMeet:

```powershell
npm run lint
npm run typecheck
npm run test
npm run build
npm run format:check
```

Các lệnh tương ứng kiểm tra quy tắc mã nguồn, kiểu dữ liệu TypeScript, kiểm thử tự động, khả năng xây dựng dự án và định dạng tệp.

Kiểm tra mẫu nền tảng dữ liệu:

```powershell
npm run sam:validate:data -- --region ap-southeast-1
```

Việc mẫu hạ tầng được kiểm tra thành công chỉ xác nhận cấu trúc mẫu hợp lệ; chưa chứng minh tài nguyên đã được triển khai trên AWS.

## 5. Khởi động giao diện trên máy cá nhân

Khởi động ứng dụng web:

```powershell
npm run dev
```

Theo cấu hình hiện tại, Vite phục vụ ứng dụng tại:

```text
http://localhost:5173
```

Ở bước chuẩn bị, giao diện có thể khởi động nhưng các chức năng cần Cognito và API sẽ chỉ hoạt động sau khi các stack AWS được triển khai và tệp `apps/web/.env` được điền bằng đúng output của CloudFormation.

Không tự đặt giá trị giả cho User Pool, User Pool Client hoặc địa chỉ API. Không đưa tệp `.env`, token, mật khẩu hoặc thông tin xác thực AWS vào Git.

## 6. Đăng nhập AWS bằng AWS CLI

Mỗi thành viên đăng nhập bằng thông tin của IAM user được cấp riêng:

```powershell
aws login
aws sts get-caller-identity
```

Lệnh `get-caller-identity` phải trả về đúng tài khoản và danh tính đang sử dụng. Ghi lại mã tài khoản để dùng trong các bước xác minh sau:

```powershell
$AccountId = aws sts get-caller-identity --query Account --output text
$AccountId
```

Kiểm tra khu vực được cấu hình:

```powershell
aws configure get region
```

Các bước triển khai của workshop sử dụng:

```text
ap-southeast-1
```

Nếu máy đang đăng nhập nhiều tài khoản AWS, phải sử dụng đúng hồ sơ cấu hình trong toàn bộ các lệnh liên quan. Không triển khai khi chưa xác nhận chính xác mã tài khoản và khu vực AWS.

## 7. Quy ước tài nguyên của CampusMeet

Các phần tiếp theo của workshop sử dụng các giá trị thống nhất:

| Thành phần | Giá trị |
| --- | --- |
| Môi trường | `dev` |
| Khu vực AWS | `ap-southeast-1` |
| Tiền tố bảng | `campusmeet-dev` |
| Data stack | `campusmeet-dev-data` |
| Auth stack | `campusmeet-dev-auth` |
| Địa chỉ frontend cục bộ | `http://localhost:5173` |

Năm bảng DynamoDB dự kiến được quản lý bởi data stack:

```text
campusmeet-dev-identity
campusmeet-dev-collaboration
campusmeet-dev-meeting-data
campusmeet-dev-task-data
campusmeet-dev-ai-work
```

Không tự tạo bảng, chỉ mục hoặc thay đổi quyền trực tiếp trên AWS Console khi các tài nguyên đó đã được quản lý bằng CloudFormation.

## 8. Danh sách kiểm tra trước khi tiếp tục

Chỉ chuyển sang phần kiến trúc và triển khai khi đã xác nhận:

- Đã có quyền truy cập mã nguồn CampusMeet.
- Node.js 22 LTS và npm 10 trở lên hoạt động.
- Git, AWS CLI, AWS SAM CLI và PowerShell hoạt động.
- Các thư viện của dự án đã được cài đặt.
- Các lệnh kiểm tra mã nguồn cơ bản chạy thành công.
- Đã đăng nhập đúng IAM user bằng AWS CLI.
- Mã tài khoản AWS đúng với môi trường của nhóm.
- Khu vực triển khai là `ap-southeast-1`.
- Không sử dụng root hoặc khóa truy cập dùng chung.
- Cảnh báo ngân sách đã được thiết lập.

Sau khi hoàn tất các điều kiện trên, có thể tiếp tục tìm hiểu kiến trúc hệ thống và ranh giới giữa các stack của CampusMeet.
