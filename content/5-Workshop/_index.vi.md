---
title: "Workshop"
date: 2026-07-27
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Xây dựng và triển khai nền tảng quản lý cuộc họp CampusMeet trên AWS

## Tổng quan

CampusMeet là nền tảng quản lý quy trình trước và sau cuộc họp dành cho nhóm học tập, nhóm đồ án và các dự án quy mô nhỏ. Hệ thống tập trung tài khoản, nhóm, thành viên, lời mời, cuộc họp, biên bản, công việc, tài liệu và các chức năng AI trong cùng một luồng làm việc.

Workshop này đi từ nền tảng AWS cơ bản đến một luồng E2E có thể triển khai và kiểm thử thật. Nội dung bám theo mã nguồn CampusMeet hiện tại; những tích hợp chưa được xác minh trên AWS hoặc trình duyệt sẽ được ghi rõ thay vì xem như đã hoàn thành.

Các dịch vụ chính được sử dụng gồm Amazon Cognito, API Gateway, AWS Lambda, DynamoDB, Amazon S3, EventBridge Scheduler, Step Functions, Amazon Bedrock, CloudWatch và IAM. Google Calendar/Meet là tích hợp bên ngoài và được kiểm thử riêng.

## Nội dung Workshop

1. [Tổng quan CampusMeet](5.1-Workshop-overview/)
2. [Điều kiện chuẩn bị](5.2-Prerequiste/)
3. [Kiến trúc hệ thống](5.3-Architecture/)
4. [IAM và cấu hình môi trường](5.4-IAM/)
5. [Xác thực với Amazon Cognito](5.5-Authentication/)
6. [Nền tảng dữ liệu DynamoDB](5.6-Data-foundation/)
7. [API Gateway và AWS Lambda](5.7-Api-lambda/)
8. [Nhóm, thành viên và lời mời](5.8-Collaboration/)
9. [Quản lý cuộc họp](5.9-Meeting-management/)
10. [Biên bản và công việc](5.10-Minutes-tasks/)
11. [Tích hợp giao diện người dùng](5.11-Frontend-integration/)
12. [Bản ghi nội dung cuộc họp và xử lý bất đồng bộ](5.12-Async-content-processing/)
13. [Tích hợp AI](5.13-AI-integration/)
14. [Giám sát và bảo mật](5.14-Monitoring-security/)
15. [Kiểm thử toàn bộ hệ thống](5.15-End-to-end-testing/)
16. [Kiểm soát chi phí](5.16-Cost-control/)

Workshop kết thúc ở phần kiểm soát chi phí. Không có bước dọn dẹp tài nguyên riêng vì môi trường CampusMeet còn được dùng cho các đợt triển khai và kiểm thử tiếp theo.
