---
title: "Worklog Tuần 8"
date: 2026-06-08
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---



### Mục tiêu tuần 8:

- Tìm hiểu các dịch vụ AWS phục vụ xử lý sự kiện và triển khai hệ thống gồm Amazon EventBridge, Amazon S3, Amazon CloudFront, AWS IAM và Amazon CloudWatch.
- Bắt đầu phát triển module NotificationLambda, tích hợp Amazon SES và DynamoDB để gửi email thông báo và lưu lịch sử gửi email.
- Hoàn thiện luồng xử lý thông báo của hệ thống, triển khai thử nghiệm trên AWS và kiểm thử toàn bộ quy trình gửi email.



### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Tìm hiểu Amazon EventBridge: Event Bus, Rule, Pattern matching, EventBridge Scheduler (rate/cron) <br> - Sử dụng dữ liệu giả làm luồng đăng ký vé đầu-cuối.                                                                                             | 08/06/2026   | 08/06/2026      |
| 3   | - Xem lại dịch vụ Amazon S3: Static Website Hosting, Bucket Policy, Public Access Block; Amazon CloudFront phục vụ việc triển khai giao diện sau này ,Tìm hiểu AWS IAM (Role, Policy, nguyên tắc least-privilege) và Amazon CloudWatch (Log Group, Alarm, Metric Filter);                                            | 09/06/2026   | 09/06/2026       |  |
| 4   | - Thiết kế bảng EventManagementNotificationLog, cập nhật template.yaml thêm NotificationFunction (Lambda, IAM Role, Policies). <br> - Viết NotificationRepository, NotificationLogDynamoMapper, SesEmailService để gửi email và ghi log qua SES + DynamoDB | 10/06/2026   | 10/06/2026       |  |
| 5   | - Viết Function.cs xử lý 3 loại trigger cho NotificationLambda: API Gateway (xem lịch sử), EventBridge Rule (email xác nhận đăng ký), Schedule (email nhắc lịch).                | 11/06/2026   | 11/06/2026       |  |
| 6   | - Deploy thử nghiệm lên AWS, debug và sửa lỗi runtime (sai tên thuộc tính khóa chính DynamoDB), kiểm thử end-to-end gửi email xác nhận đăng ký.                                                                                         | 12/06/2026   | 12/06/2026       |  |
| 7   | - Tham dự event ở trên công ty                                                                                         | 13/06/2026   | 13/06/2026       |  |


### Kết quả đạt được tuần 8:

 Đã tìm hiểu Amazon EventBridge, bao gồm Event Bus, Rule, Pattern Matching và EventBridge Scheduler (Rate/Cron), đồng thời mô phỏng luồng đăng ký vé bằng dữ liệu giả để hiểu cơ chế phát sinh sự kiện.
- Đã nghiên cứu Amazon S3, Amazon CloudFront, AWS IAM và Amazon CloudWatch, nắm được vai trò của từng dịch vụ trong việc lưu trữ, phân phối nội dung, phân quyền truy cập và giám sát hệ thống.
- Đã thiết kế bảng **EventManagementNotificationLog** trên Amazon DynamoDB và cập nhật **template.yaml** để bổ sung tài nguyên cho **NotificationLambda**.
- Đã xây dựng các thành phần **NotificationRepository**, **NotificationLogDynamoMapper** và **SesEmailService**, thực hiện chức năng gửi email thông qua Amazon SES và lưu lịch sử gửi email vào DynamoDB.
- Đã hoàn thiện **Function.cs** cho NotificationLambda, xử lý thành công ba loại trigger gồm API Gateway, EventBridge Rule và EventBridge Scheduler.
- Đã triển khai thử nghiệm NotificationLambda lên AWS, phát hiện và khắc phục lỗi runtime liên quan đến khóa chính của DynamoDB, đồng thời kiểm thử thành công luồng gửi email xác nhận đăng ký từ đầu đến cuối.
- Tham gia sự kiện nội bộ của công ty, trao đổi kinh nghiệm thực tế về phát triển ứng dụng trên nền tảng AWS và cập nhật thêm kiến thức phục vụ quá trình thực tập.


