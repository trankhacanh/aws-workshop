---
title: "Worklog Tuần 7"
date: 2026-06-01
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---



### Mục tiêu tuần 7:
- Tìm hiểu các dịch vụ AWS cốt lõi được sử dụng trong dự án Serverless gồm AWS Lambda, AWS SAM, Amazon API Gateway, Amazon Cognito, Amazon DynamoDB và Amazon SES.
- Hiểu kiến trúc tổng thể của hệ thống, cách các dịch vụ AWS phối hợp với nhau trong quá trình xử lý nghiệp vụ.
- Nắm được cấu trúc source code của dự án và chức năng của các module Lambda để chuẩn bị cho giai đoạn phát triển và chỉnh sửa tính năng ở các tuần tiếp theo.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | **Tìm hiểu dịch vụ để làm project** <br> - Tổng quan AWS Lambda: mô hình thực thi, cold start, giới hạn tài nguyên (memory, timeout), runtime .NET 8.<br> -	Nắm được vòng đời 1 Lambda function và cách viết Handler cơ bản.<br> - Tìm hiểu AWS SAM (Serverless Application Model): cấu trúc template.yaml, cú pháp Resources/Globals/Events.                                                                                             | 01/06/2026   | 01/06/2026      |
| 3   | - Tìm hiểu Amazon API Gateway: REST API, tích hợp Lambda Proxy Integration, cấu hình CORS. <br> - Đọc tài liệu Amazon Cognito: User Pool, App Client, Authorizer cho API Gateway,                                        | 02/06/2026   | 02/06/2026      |  |
| 4   | - Tìm hiểu Amazon DynamoDB <br> - Tìm hiểu các thao tác DynamoDB SDK cho .NET: PutItem, GetItem, Query| 03/06/2026   | 03/06/2026      |  |
| 5   | - Tìm hiểu dịch vụ và mã nguồn module EventLambda <br>- Tìm hiểu Amazon SES (Simple Email Service)                 | 04/06/2026   | 04/06/2026      |  |
| 6   | - Xem lại kiến trúc hệ thống và hiểu rõ các dịch vụ để hiểu cách lưu trữ diệu, hướng sự kiện event <br> - Khảo sát tổng quan 4 module Lambda của nhóm: EventLambda + UserProfileLambda (Thịnh), RegistrationTicketLambda (Tâm), AttendanceCertificateLambda (Thư Kỳ) để nắm quy ước đặt tên, cấu trúc project chung và các bảng DynamoDB/Bucket S3 dùng chung.                                                                                 | 05/06/2026   | 05/06/2026      | |


### Kết quả đạt được tuần 7:

- Đã tìm hiểu nguyên lý hoạt động của AWS Lambda, vòng đời thực thi, Handler, cold start, giới hạn tài nguyên và runtime .NET 8.
- Đã nghiên cứu AWS SAM, hiểu cấu trúc `template.yaml` và cách khai báo tài nguyên, sự kiện trong ứng dụng Serverless.
- Đã tìm hiểu Amazon API Gateway, cơ chế Lambda Proxy Integration, cấu hình CORS và cách tích hợp với Lambda.
- Đã tìm hiểu Amazon Cognito, bao gồm User Pool, App Client và cơ chế xác thực API thông qua Authorizer.
- Đã nghiên cứu Amazon DynamoDB và các thao tác cơ bản bằng AWS SDK for .NET như PutItem, GetItem và Query.
- Đã tìm hiểu Amazon SES và quy trình gửi email trong hệ thống.
- Đã phân tích kiến trúc tổng thể của dự án, hiểu luồng xử lý dữ liệu, cơ chế event-driven và mối liên kết giữa các dịch vụ AWS.
- Đã khảo sát cấu trúc của các module Lambda trong dự án, nắm được quy ước tổ chức mã nguồn, chức năng của từng module và các tài nguyên dùng chung như DynamoDB và Amazon S3, tạo nền tảng cho việc tham gia phát triển dự án ở các tuần tiếp theo.


