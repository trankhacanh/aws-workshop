---
title: "Worklog Tuần 9"
date: 2026-06-15
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---



### Mục tiêu tuần 9:

- Hoàn thiện và kiểm thử chức năng gửi email nhắc lịch sự kiện thông qua Amazon EventBridge Scheduler.
- Phát triển module Analytics để thống kê dữ liệu của hệ thống từ Amazon DynamoDB và Amazon S3.
- Xây dựng giao diện Dashboard trên Frontend, tích hợp với API thống kê nhằm hỗ trợ quản trị viên theo dõi tình trạng hoạt động của hệ thống.



### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Kiểm thử luồng email nhắc lịch sự kiện (EventBridge Schedule) bằng cách giả lập sự kiện Scheduled Event, xác nhận quét đúng dữ liệu và gửi email thành công.                                                                                   | 15/06/2026   | 15/06/2026      |
| 3   | - Khảo sát cấu trúc dữ liệu thực tế các bảng Ticket/Attendance từ mã nguồn của các thành viên khác, thiết kế AnalyticsRepository tổng hợp số liệu (tổng sự kiện, vé đăng ký, check-in).                                            | 16/06/2026   | 16/06/2026      |  |
| 4   | - Viết AnalyticsRepository: đếm số liệu qua DynamoDB Scan/Query, đếm số chứng chỉ đã cấp qua liệt kê object trong S3 Bucket. | 17/06/2026   | 17/06/2026      |  |
| 5   | - Viết AnalyticsLambda Function.cs, cập nhật template.yaml (biến môi trường, quyền DynamoDBReadPolicy/S3ReadPolicy), deploy và kiểm thử API Dashboard.                | 18/06/2026   | 18/06/2026      |  |
| 6   | - Xây dựng trang AnalyticsPage phía Frontend (React), gọi API Dashboard và API thống kê theo từng sự kiện, cập nhật route và menu điều hướng Admin.                                                                                         | 19/06/2026   | 19/06/2026      |  |
| 7   | - Tham dự event ở trên công ty                                                                                       | 20/06/2026   | 20/06/2026      |  |


### Kết quả đạt được tuần 9:

 Đã kiểm thử thành công luồng gửi email nhắc lịch sự kiện bằng cách giả lập Scheduled Event của Amazon EventBridge, xác nhận hệ thống truy xuất đúng dữ liệu và gửi email đến người dùng.
- Đã khảo sát cấu trúc dữ liệu của các bảng Ticket và Attendance từ các module liên quan, phân tích mối quan hệ dữ liệu và thiết kế **AnalyticsRepository** để phục vụ chức năng thống kê.
- Đã xây dựng **AnalyticsRepository**, thực hiện thống kê số lượng sự kiện, lượt đăng ký, lượt check-in từ Amazon DynamoDB và số lượng chứng chỉ đã cấp từ Amazon S3.
- Đã hoàn thiện **AnalyticsLambda**, cập nhật **template.yaml** với các biến môi trường và quyền truy cập cần thiết (DynamoDB Read Policy và S3 Read Policy), triển khai thành công lên AWS và kiểm thử API Dashboard.
- Đã phát triển giao diện **AnalyticsPage** bằng React, tích hợp API Dashboard và API thống kê theo từng sự kiện, đồng thời cập nhật hệ thống định tuyến và menu quản trị để hỗ trợ truy cập chức năng thống kê.
- Tham gia sự kiện nội bộ của công ty, trao đổi với các thành viên trong dự án về quá trình phát triển hệ thống và tiếp thu thêm kinh nghiệm thực tế trong việc xây dựng ứng dụng Serverless trên AWS.


