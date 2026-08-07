---
title: "Workshop"
date: 2026-07-27
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Xây dựng và triển khai CampusMeet trên AWS

Workshop trình bày cách CampusMeet được xây dựng và đưa lên AWS từ kiến trúc, xác thực và dữ liệu đến các chức năng chính, giao diện production, tích hợp AI và kiểm thử E2E.

Nội dung tập trung vào **cách hệ thống hoạt động và cách triển khai**, không đi sâu vào từng khóa DynamoDB, từng endpoint hay từng test case nội bộ. Những phần chưa được xác minh trên AWS thật sẽ được ghi rõ ở phần cuối.

## Nội dung Workshop

1. [Tổng quan CampusMeet](5.1-Workshop-overview/)
2. [Chuẩn bị và quyền truy cập AWS](5.2-Prerequiste/)
3. [Kiến trúc hệ thống](5.3-Architecture/)
4. [Xác thực và API](5.4-Authentication-api/)
5. [Dữ liệu và cộng tác nhóm](5.5-Data-collaboration/)
6. [Cuộc họp, biên bản và công việc](5.6-Meeting-workflow/)
7. [Giao diện và triển khai production](5.7-Frontend-deployment/)
8. [Xử lý bất đồng bộ, Google và AI](5.8-Integrations-ai/)
9. [Giám sát, bảo mật và chi phí](5.9-Operations/)
10. [Kiểm thử E2E và kết quả production](5.10-E2E-production/)

Workshop kết thúc bằng bản triển khai thực tế: frontend chạy qua HTTPS, backend hoạt động trên AWS và một luồng CampusMeet cốt lõi được kiểm thử từ đầu đến cuối.