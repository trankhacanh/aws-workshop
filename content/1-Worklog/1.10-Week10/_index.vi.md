---
title: "Worklog Tuần 10"
date: 2026-06-22
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---



### Mục tiêu tuần 10:

- Hoàn thiện kiến trúc giao tiếp giữa các module theo mô hình hướng sự kiện (Event-Driven Architecture) bằng Amazon EventBridge.
- Tìm hiểu sâu hơn về module RegistrationTicketLambda và luồng xử lý đăng ký vé trong hệ thống để hiểu cách các module phối hợp với nhau.
- Hoàn thiện triển khai backend trên AWS, chuẩn bị môi trường cho việc triển khai frontend và hoàn thành dự án.
- Tổng hợp nội dung thực tập, xây dựng khung báo cáo và hệ thống hóa kiến thức đã thực hiện trong suốt quá trình phát triển dự án.


### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Chuyển đổi cơ chế gọi thông báo từ Lambda Invoke trực tiếp sang publish sự kiện qua EventBridge (PutEvents), đồng bộ định dạng payload.                                                                                             | 22/06/2026   | 22/06/2026      |
| 3   | - Tìm hiểu dịch vụ và mã nguồn module RegistrationTicketLambda (Tâm): thiết kế bảng Ticket kèm GSI theo UserId, cơ chế cập nhật nguyên tử chống trùng vé, cách publish sự kiện qua EventBridge khi đăng ký vé thành công.                                          | 23/06/2026   | 23/06/2026      |  |
| 4   | - Nghiên cứu cấu trúc AWS để hiểu rõ luồng hoạt động của toàn dự án | 24/06/2026   | 24/06/2026      |  |
| 5   | - Viết template báo cáo                  | 25/06/2026   | 25/06/2026      |  |
| 6   | - Triển khai backend lên AWS sửa lỗi  để chuẩn bị cho triển khai frontend lên S3 và hoàn thành dự án                                                                                         | 26/06/2026   | 26/06/2026      |  |
| 6   | - Tham dự event ở trên công ty                                                                                         | 27/06/2026   | 27/06/2026      |  |

### Kết quả đạt được tuần 10:

- Đã chuyển đổi thành công cơ chế gửi thông báo từ gọi trực tiếp AWS Lambda sang phát sinh sự kiện thông qua Amazon EventBridge (PutEvents), đồng thời chuẩn hóa cấu trúc payload giữa các module để tăng tính mở rộng và giảm sự phụ thuộc trực tiếp.
- Đã nghiên cứu module **RegistrationTicketLambda**, hiểu được cơ chế lưu trữ thông tin vé bằng Amazon DynamoDB, cách sử dụng Global Secondary Index (GSI) theo **UserId**, cơ chế cập nhật dữ liệu nguyên tử nhằm tránh đăng ký trùng và quy trình phát sinh sự kiện sau khi đăng ký vé thành công.
- Đã phân tích kiến trúc AWS của toàn bộ hệ thống, nắm rõ luồng xử lý dữ liệu giữa các Lambda Function, Amazon API Gateway, Amazon EventBridge, Amazon DynamoDB, Amazon S3, Amazon SES và Amazon Cognito, từ đó hiểu rõ cách các thành phần phối hợp trong mô hình Serverless.
- Đã xây dựng mẫu báo cáo thực tập, tổng hợp các nội dung đã nghiên cứu, các công việc đã thực hiện và kết quả đạt được trong quá trình tham gia dự án.
- Đã triển khai backend lên AWS, kiểm tra và khắc phục các lỗi phát sinh trong quá trình triển khai, bảo đảm các API hoạt động ổn định và sẵn sàng cho bước triển khai giao diện frontend lên Amazon S3.
- Tham gia sự kiện nội bộ của công ty, trao đổi kinh nghiệm phát triển dự án với các thành viên trong nhóm và cập nhật thêm các kiến thức thực tế về AWS và kiến trúc Serverless. 


