---
title: "Tổng quan workshop"
date: 2026-06-29
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

---

## Giới thiệu

Workshop này giới thiệu cách triển khai và kiểm thử dự án **Event Management Platform**, một hệ thống hỗ trợ tổ chức quản lý toàn bộ quy trình tổ chức sự kiện, được xây dựng theo kiến trúc serverless trên AWS.

Bài toán thực tế mà dự án giải quyết: các đơn vị tổ chức hội thảo, workshop công nghệ hiện vẫn quản lý đăng ký bằng Google Form/Excel — dẫn đến khó kiểm soát số lượng người tham dự thực tế, phải gửi email xác nhận thủ công, không kiểm soát được tình trạng gian lận vé, và không có dữ liệu để đánh giá hiệu quả sự kiện sau khi kết thúc. Event Management Platform số hóa toàn bộ quy trình này trên nền tảng serverless.

Event Management Platform được thiết kế cho hai vai trò chính:

- **User (Attendee)**: sử dụng ứng dụng web để xem danh sách sự kiện, đăng ký tham gia, nhận vé điện tử, quét QR check-in tại sự kiện và tải chứng nhận tham dự sau khi hoàn thành.
- **Admin**: quản lý sự kiện (tạo/sửa/xóa), quản lý danh mục, upload banner, theo dõi số lượng đăng ký/check-in theo thời gian thực và xem dashboard phân tích hiệu quả sự kiện.

Workshop tập trung vào việc deploy toàn bộ hạ tầng backend serverless (6 Lambda function) bằng AWS SAM, deploy frontend qua CloudFront, và chạy demo ứng dụng thật.


---

## Mục tiêu workshop

Sau khi hoàn thành workshop này, người thực hiện có thể:

- Hiểu kiến trúc serverless của Event Management Platform với 6 Lambda function độc lập.
- Deploy backend bằng AWS SAM và quản lý tài nguyên qua AWS CloudFormation.
- Sử dụng Amazon Cognito để xác thực người dùng và cấp JWT token.
- Sử dụng API Gateway để cung cấp backend API, phân biệt route công khai và route được bảo vệ.
- Sử dụng Lambda function (.NET 8) để xử lý logic nghiệp vụ: quản lý sự kiện, hồ sơ người dùng, đăng ký vé, check-in, cấp chứng nhận, thông báo và phân tích dữ liệu.
- Sử dụng DynamoDB để lưu trữ dữ liệu ứng dụng.
- Sử dụng S3 để lưu banner sự kiện, avatar người dùng, chứng nhận PDF và file build frontend.
- Sử dụng SES để gửi email chứng nhận và thông báo.
- Sử dụng EventBridge (default event bus + EventBridge Scheduler) để xử lý thông báo bất đồng bộ và nhắc lịch tự động — thay vì gọi trực tiếp hoặc dùng SNS/SQS.
- Deploy Frontend React qua Amazon CloudFront với Origin Access Control (OAC).
- Xem log backend bằng CloudWatch Logs.
- Kiểm thử toàn bộ luồng nghiệp vụ chính end-to-end của User và Admin.

---

## Dịch vụ và công cụ AWS sử dụng

Workshop này sử dụng các dịch vụ và công cụ AWS sau:

| Dịch vụ / Công cụ | Mục đích |
|---|---|
| Amazon Cognito | Xác thực người dùng, cấp JWT token, phân quyền User/Admin |
| Amazon API Gateway | Cung cấp backend API; bảo vệ phần lớn route bằng Cognito Authorizer mặc định, một số route đọc công khai được khai báo riêng |
| AWS Lambda (.NET 8) | Chạy logic nghiệp vụ backend qua 6 function: EventFunction, UserProfileFunction, TicketFunction, AttendanceCertificateFunction, NotificationFunction, AnalyticsFunction |
| Amazon DynamoDB | Lưu events, categories, tickets, attendance, users, notification log |
| Amazon S3 | Lưu banner sự kiện, avatar người dùng, chứng nhận PDF và file build frontend |
| Amazon SES | Gửi email chứng nhận tham dự và email thông báo/nhắc lịch |
| Amazon EventBridge (Default Event Bus + Scheduler) | Xử lý thông báo bất đồng bộ khi đăng ký vé và nhắc lịch định kỳ trước sự kiện |
| Amazon CloudFront + OAC | Phân phối Frontend React từ S3 private, không cần bucket public |
| Amazon CloudWatch Logs | Theo dõi log Lambda/API và hỗ trợ debug |
| AWS SAM | Build và deploy backend serverless |
| AWS CloudFormation | Quản lý tài nguyên AWS thông qua stack |

![Sơ đồ kiến trúc Serverless Event Platform](/images/5-Workshop/5.1-Workshop-overview/system-architecture.jpg)

---

## Dịch vụ không nằm trong phạm vi workshop

Các dịch vụ sau không nằm trong phạm vi workshop hiện tại:

| Dịch vụ | Trạng thái |
|---|---|
| AWS Amplify Hosting | Không sử dụng (dùng CloudFront + S3 trực tiếp thay thế) |
| Amazon EC2 | Không sử dụng |
| Amazon VPC | Không sử dụng |
| Amazon RDS | Không sử dụng |
| Amazon SNS / SQS | Không sử dụng cho luồng thông báo (thay bằng EventBridge) |

---

## Công nghệ và cấu trúc dự án hiện tại

Source code dự án được chia thành hai phần chính:

- **Backend (BE):** kiến trúc Serverless trên AWS Lambda (.NET 8/C#), dùng AWS SAM (template.yaml) làm Infrastructure as Code, tổ chức theo src/Functions (6 Lambda function) và src/Shared (Services, Repositories, DTOs, Validators, Constants dùng chung).
- **Frontend (FE):** React 18+ với TypeScript, build bằng Vite, tổ chức theo feature (components/events, components/tickets, components/certificates, components/notifications...), gọi API qua Axios và xác thực qua Cognito.


| ![Ảnh 1](/images/5-Workshop/5.1-Workshop-overview/BE.jpg) | ![Ảnh 2](/images/5-Workshop/5.1-Workshop-overview/FE.jpg) |
|---|---|

---

## Thông tin deploy

Backend hiện tại sử dụng cấu hình deploy như sau:

| Nội dung | Giá trị |
|---|---|
| AWS Region | ap-southeast-1 |
| CloudFormation Stack | aws-event-management |
| API Gateway Endpoint | https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev |
| Cognito User Pool ID | ap-southeast-1_XXXXXXXXX |
| CloudFront Domain | <distribution-id>.cloudfront.net |

*Các giá trị định danh thật (Pool ID, endpoint, domain) được quản lý riêng qua tham số deploy và biến môi trường, không commit vào source code; sẽ cung cấp trực tiếp khi demo.*

---

## Flow demo chính

Workshop demo theo flow sau:

1. Admin đăng nhập, tạo danh mục và tạo sự kiện mới kèm banner.
2. User đăng ký hoặc đăng nhập thông qua Cognito.
3. User xem danh sách sự kiện (route công khai) và chọn sự kiện muốn tham gia.
4. User gửi yêu cầu đăng ký vé; hệ thống kiểm tra số chỗ còn trống và tạo vé CONFIRMED.
5. Hệ thống publish sự kiện TicketRegistered lên EventBridge; NotificationFunction gửi email xác nhận qua SES.
6. User xem vé cá nhân qua GET /my-tickets.
7. Gần đến giờ sự kiện, EventBridge Scheduler trigger NotificationFunction quét và gửi email nhắc lịch.
8. Tại sự kiện, User đưa mã QR (chứa TicketId) để check-in.
9. Sau khi check-in, User yêu cầu chứng nhận; hệ thống tạo PDF, upload S3 và gửi email qua SES.
10. Admin xem dashboard phân tích: số lượng đăng ký, tỷ lệ check-in, log thông báo theo từng sự kiện.

---

## Kết quả mong đợi

Sau khi hoàn thành workshop, backend gồm 6 Lambda function sẽ được deploy thành công lên AWS, đồng thời Frontend React được build và phân phối qua CloudFront có thể kết nối với backend đã deploy.

Người thực hiện có thể kiểm thử toàn bộ các luồng chính của Event Management Platform: quản lý sự kiện, đăng ký vé, check-in bằng QR, cấp chứng nhận, thông báo/nhắc lịch tự động qua EventBridge, và xem dashboard phân tích hiệu quả sự kiện.