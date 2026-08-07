---
title: "Giám sát, bảo mật và chi phí"
date: 2026-08-08
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

# Giám sát, bảo mật và chi phí

Sau khi hệ thống chạy được, ba việc cần quan tâm là theo dõi lỗi, bảo vệ dữ liệu và tránh tài nguyên chạy ngoài kiểm soát.

## Giám sát

CloudWatch được dùng để theo dõi log của API và các worker. Log nên giúp xác định request hoặc tài nguyên nào lỗi nhưng không chứa JWT, OAuth token, presigned URL hay toàn bộ nội dung tài liệu của người dùng.

Các lỗi đáng chú ý gồm API failure, worker thất bại liên tục, AI job lỗi hoặc Google sync không hoàn tất sau retry.

## Bảo mật

CampusMeet áp dụng một số nguyên tắc chính:

- Cognito xác thực người dùng, backend kiểm tra quyền nghiệp vụ.
- Mỗi Lambda/worker sử dụng execution role phù hợp với chức năng.
- Google client secret và token nằm ở phía server.
- Bucket chứa tài liệu người dùng không public.
- Production CORS giới hạn về đúng frontend origin.
- Dữ liệu production có thể bật PITR và deletion protection.

## Kiểm soát chi phí

Các dịch vụ cần theo dõi chủ yếu là DynamoDB, Lambda, API Gateway, S3, CloudFront, CloudWatch, Step Functions và Bedrock.

Với đồ án có lưu lượng nhỏ, kiến trúc serverless giúp chi phí bám theo mức sử dụng. Tuy nhiên retry lỗi, log quá nhiều hoặc AI chạy lặp có thể làm chi phí tăng nhanh.

AWS Budgets nên được cấu hình để gửi cảnh báo sớm. Với Bedrock và các workflow bất đồng bộ, cần có giới hạn retry và điều kiện dừng rõ ràng.

## Trước khi release

Trước đợt production nên kiểm tra lại:

- không có secret trong Git;
- frontend production không chứa credential phía server;
- user-content S3 không public;
- CORS dùng đúng CloudFront origin;
- log không lộ token;
- Budget alert vẫn hoạt động.

Workshop không có phần dọn dẹp tài nguyên riêng vì môi trường production còn được dùng cho demo và đánh giá.