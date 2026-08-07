---
title: "Chuẩn bị và quyền truy cập AWS"
date: 2026-07-27
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Chuẩn bị và quyền truy cập AWS

Trước khi triển khai CampusMeet, cần chuẩn bị môi trường phát triển và xác nhận đúng tài khoản AWS. Phần này chỉ giữ những yêu cầu cần thiết để bắt đầu workshop.

## Công cụ cần có

- Node.js 22 và npm.
- Git.
- AWS CLI.
- AWS SAM CLI.
- PowerShell trên Windows.

Kiểm tra nhanh:

```powershell
node --version
npm --version
git --version
aws --version
sam --version
```

## Mã nguồn CampusMeet

```powershell
git clone https://github.com/Ngct253/CampusMeet.git
cd CampusMeet
npm ci
```

Repo được tổ chức thành các phần chính:

```text
apps/web/       giao diện React
services/api/   API và nghiệp vụ
services/ai-worker/  xử lý AI
infra/          hạ tầng AWS
scripts/        script hỗ trợ triển khai
```

Trước khi deploy nên chạy:

```powershell
npm run lint
npm run typecheck
npm run test
npm run build
```

## Tài khoản và khu vực AWS

Workshop sử dụng khu vực:

```text
ap-southeast-1 (Singapore)
```

Sau khi đăng nhập AWS CLI, luôn xác nhận lại danh tính và region trước khi deploy:

```powershell
aws sts get-caller-identity
aws configure get region
```

Không sử dụng tài khoản root cho công việc hằng ngày và không chia sẻ access key giữa các thành viên.

## Quyền truy cập

Nhóm phát triển cần phân biệt hai loại quyền:

- **Quyền của người triển khai**: dùng để tạo hoặc cập nhật CloudFormation stack và các IAM role cần thiết.
- **Execution role của dịch vụ**: quyền mà Lambda hoặc worker sử dụng khi truy cập DynamoDB, S3, Bedrock hay các dịch vụ AWS khác.

Không cần cấp `AdministratorAccess` cho mọi thành viên chỉ để ứng dụng hoạt động. Các dịch vụ nên nhận đúng quyền mà chúng cần.

Thông tin nhạy cảm như Google client secret, token hoặc AWS credential không được đưa vào Git hoặc biến `VITE_*` của frontend.

## Môi trường sử dụng

Trong quá trình phát triển có thể dùng tài nguyên `campusmeet-dev-*`. Bản nộp production nên sử dụng môi trường riêng như `campusmeet-prod-*` để tránh thay đổi nhầm dữ liệu dev.

Sau khi các công cụ, AWS account và quyền truy cập đã sẵn sàng, có thể chuyển sang kiến trúc và triển khai hệ thống.