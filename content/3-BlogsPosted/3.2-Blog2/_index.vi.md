---
title: "Blog 2"
date: 2026-07-03
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---


# XÂY DỰNG HỆ THỐNG NOTIFICATION, ANALYTICS VÀ DEPLOYMENT HOÀN TOÀN SERVERLESS TRÊN AWS

Khi bắt tay vào thiết kế hệ thống quản lý Workshop, các module Notification, Analytics và Deployment/Monitoring đóng vai trò quyết định đến khả năng vận hành, giám sát và mở rộng của toàn bộ hệ thống. Thay vì triển khai theo mô hình truyền thống với máy chủ ứng dụng chạy liên tục, việc lựa chọn kiến trúc Serverless trên AWS giúp giảm đáng kể chi phí hạ tầng, hạn chế quản lý máy chủ và tận dụng khả năng tự động mở rộng linh hoạt.

Các điểm chính cần nắm:

* Thiết kế kiến trúc theo hướng Event-Driven: Sử dụng Amazon EventBridge làm trung tâm tiếp nhận và phân phối các sự kiện phát sinh (như tạo vé thành công, đăng ký workshop), giúp giảm sự phụ thuộc (decouple) hoàn toàn giữa các module.
* Notification – Tự động hóa quá trình gửi Email: Kết hợp AWS Lambda để xử lý nội dung và Amazon SES để đảm nhiệm gửi email xác nhận hoặc nhắc lịch thông qua Amazon EventBridge Scheduler mà không cần duy trì Cron Server liên tục.
* Analytics – Dashboard dành cho Admin: Analytics Lambda tổng hợp trực tiếp các số liệu thời gian thực từ DynamoDB (EventTable, RegistrationTable, TicketTable, AttendanceTable) để hiển thị tỷ lệ tham dự và trạng thái vé lên Dashboard.
* Monitoring – Giám sát toàn diện hệ thống: Cấu hình ghi log tập trung từ các Lambda vào Amazon CloudWatch, thiết lập CloudWatch Alarms và Amazon SNS để tự động gửi cảnh báo khi tỷ lệ lỗi vượt ngưỡng mong muốn.
* Triển khai Frontend tối ưu: Lưu trữ mã nguồn tĩnh trên Amazon S3 và phân phối qua CDN toàn cầu bằng Amazon CloudFront giúp cải thiện tốc độ truy cập, giảm độ trễ và tối ưu chi phí vận hành.

![Kiến trúc Serverless Notification, Analytics và Deployment](/images/3.2-Blog2/Picture.png)

### LINK THAM KHẢO DỊCH VỤ CỐT LÕI (AWS OFFICIAL DOCUMENTATION)
* [AWS Serverless Multi-Tier Architectures with Amazon API Gateway and AWS Lambda](https://docs.aws.amazon.com/whitepapers/latest/serverless-multi-tier-architectures-api-gateway-lambda/serverless-multi-tier-architectures-api-gateway-lambda.html)
* [Amazon EventBridge - Building Event-Driven Architectures](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html)
* [Amazon SES - Automated Email Sending Integration Guide](https://docs.aws.amazon.com/ses/latest/dg/welcome.html)

### HƯỚNG DẪN TRIỂN KHAI VÀ PHÂN TÍCH CHI TIẾT

Dựa trên sơ đồ kiến trúc Serverless, hệ thống được module hóa chặt chẽ theo 4 phân tầng độc lập:

1. **Tầng Tích hợp Sự kiện (Event Integration Layer):**
   * Khi các nguồn sự kiện (Web/Mobile, Hệ thống nội bộ, Scheduler) kích hoạt, **Amazon EventBridge** đóng vai trò Event Bus trung tâm nhận tín hiệu.
   * EventBridge kích hoạt **AWS Lambda (Processor)** để xử lý nghiệp vụ phân loại sự kiện, sau đó đẩy sang **Amazon SNS** thực hiện cơ chế Fan-out để đồng thời kích hoạt hoặc gửi thông báo đến các bên liên quan.

2. **Tầng Ứng dụng trong VPC (Application Layer):**
   * Người dùng tương tác qua **Amazon API Gateway** (REST API), chuyển tiếp yêu cầu đến **AWS Lambda (Business Logic)** để thực thi nghiệp vụ và tương tác với cơ sở dữ liệu NoSQL **Amazon DynamoDB** (gồm hệ thống bảng chính: EventTable, RegistrationTable, TicketTable, AttendanceTable).
   * Khi có log gửi email, một tiến trình **AWS Lambda (Notification Service)** độc lập sẽ được kích hoạt để đẩy dữ liệu qua **Amazon SES** gửi mail đến người dùng, đồng thời lưu vết vào bảng *NotificationLogTable*.

3. **Tầng Phân tích Dữ liệu (Analytics Layer):**
   * Để phục vụ Dashboard cho Admin, **AWS Lambda (Analytics Processor)** sẽ thực hiện truy vấn đọc dữ liệu (**Read Data**) từ Amazon DynamoDB.
   * Dữ liệu sau khi tổng hợp sẽ được đẩy trực tiếp lên **Amazon QuickSight** giúp hiển thị các biểu đồ trực quan sinh động trên Dashboard quản trị.

4. **Tầng Giám sát & Ghi log (Monitoring & Logging Layer):**
   * Toàn bộ trạng thái hoạt động và log của AWS Lambda hay Amazon SES đều được đồng bộ về **Amazon CloudWatch**.
   * Hệ thống cấu hình sẵn **CloudWatch Alarms** để phát hiện bất thường (lỗi runtime, quá tải). Khi có cảnh báo, CloudWatch Alarm sẽ kích hoạt **Amazon SNS (Alert Notifications)** để gửi cảnh báo tức thời tới Admin/DevOps qua Email/SMS hoặc Chatbot.