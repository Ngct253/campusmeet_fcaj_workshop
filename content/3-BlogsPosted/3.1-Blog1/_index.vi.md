---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---
# CAMPUSMEET – HỆ THỐNG QUẢN LÝ CUỘC HỌP VÀ KHAI THÁC TRI THỨC VỚI AWS SERVERLESS & GENERATIVE AI

Bắt đầu từ một vấn đề khá quen thuộc: sau mỗi cuộc họp, thông tin thường bị phân tán ở rất nhiều nơi — lịch nằm trên Calendar, cuộc họp diễn ra trên Google Meet, tài liệu ở Drive, biên bản được ghi riêng và các công việc sau họp lại tiếp tục được quản lý ở một công cụ khác.

Từ bài toán đó, tụi mình xây dựng **CampusMeet** với mục tiêu quản lý xuyên suốt vòng đời trước – trong – sau cuộc họp, đồng thời biến dữ liệu từ các cuộc họp thành một nguồn tri thức có thể tiếp tục tìm kiếm và khai thác bằng AI. CampusMeet hướng đến các nhóm học tập, nhóm đồ án và nhóm dự án nhỏ.

Thay vì tự xây dựng một nền tảng video call mới, CampusMeet tập trung vào phần giá trị phía sau cuộc họp: agenda, participant, reminder, transcript, biên bản, action item, task và knowledge retrieval, đồng thời tích hợp Google Calendar/Google Meet cho phần conference.

Toàn bộ giải pháp được thiết kế theo hướng managed serverless trên AWS, với một số điểm nổi bật:

- **Quản lý cuộc họp end-to-end**: Amazon Cognito đảm nhiệm xác thực, API Gateway + AWS Lambda xử lý các API nghiệp vụ, còn DynamoDB lưu trữ dữ liệu về nhóm, cuộc họp, biên bản, task và AI job. Data Model được thiết kế theo access pattern và gom về 5 physical DynamoDB tables thay vì tạo một bảng riêng cho từng entity.
- **Tích hợp Google Calendar/Meet nhưng không phụ thuộc hoàn toàn vào Google**: CampusMeet quản lý dữ liệu cuộc họp nội bộ và tích hợp Google để tạo event/Meet link. Khi artifact từ Google không khả dụng hoặc không đủ quyền, hệ thống vẫn có thể tiếp tục bằng các cơ chế fallback thay vì làm mất dữ liệu nội bộ.
- **Upload và xử lý dữ liệu bất đồng bộ**: tài liệu hoặc audio lớn được upload trực tiếp từ Browser lên Amazon S3 bằng Presigned URL, không truyền toàn bộ file qua API Gateway/Lambda. Sau đó AIJob và Step Functions đảm nhiệm pipeline xử lý tiếp theo.
- **Generative AI với Amazon Bedrock và RAG nhiều cuộc họp**: thay vì chỉ tóm tắt một transcript, CampusMeet hướng đến khả năng hỏi đáp trên nhiều meeting và trả lời kèm citation về meeting/tài liệu nguồn. Việc retrieval phải lọc trước theo groupId, phạm vi meeting và ACL để tránh truy xuất dữ liệu ngoài quyền của người dùng.
- **AI hỗ trợ nhưng không tự quyết định**: AI có thể tạo biên bản hoặc đề xuất action item/task, nhưng không tự ý ghi dữ liệu nghiệp vụ. Người dùng phải review và xác nhận trước khi backend kiểm tra quyền và thực hiện thao tác thật. Đây là một trong những nguyên tắc tụi mình muốn giữ để AI đóng vai trò trợ lý thay vì trở thành một agent có quyền thay đổi dữ liệu không kiểm soát.
- **Tối ưu vận hành và chi phí**: kiến trúc MVP chủ động không đưa EC2, RDS, EKS hay NAT Gateway vào nếu chưa có nhu cầu thực tế. Hệ thống ưu tiên các dịch vụ managed/serverless, đồng thời sử dụng CloudWatch, SNS và Infrastructure as Code với AWS SAM/CloudFormation để phục vụ monitoring và vận hành.

Điều tụi mình thấy thú vị nhất ở CampusMeet không phải là việc sử dụng nhiều AWS service, mà là cách kết nối chúng thành một luồng dữ liệu xuyên suốt:  
**Meeting → Transcript → Minutes → Action Items → Tasks → Knowledge Base → RAG → Progress Analysis.**

Từ đó, dữ liệu của một cuộc họp không bị “đóng lại” sau khi mọi người rời Google Meet mà có thể tiếp tục được sử dụng để hỗ trợ những cuộc họp và quyết định sau này.

### Bài viết trên AWS Study Group
🔗 **Link bài viết**: [https://www.facebook.com/groups/awsstudygroupfcj/permalink/2237833836981576](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2237833836981576)

### Minh chứng bài đăng
![Minh chứng đăng bài trên AWS Study Group](images/3-BlogsPosted/3.1-Blog1/blog-evidence.png?v=2)