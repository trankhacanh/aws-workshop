---
title: "Workshop"
date: 2026-07-03
weight: 5
chapter: false
pre: " <b> 5. </b> "
---


# Cloud-Based Event Management Platform Workshop

## Triển khai backend serverless và chạy demo ứng dụng

Workshop này giới thiệu cách triển khai và kiểm thử dự án **Event Management Platform**, một hệ thống hỗ trợ tổ chức quản lý toàn bộ quy trình tổ chức sự kiện, được xây dựng theo kiến trúc serverless trên AWS.

Event Management Platform hỗ trợ tạo và quản lý sự kiện, quản lý danh mục, quản lý hồ sơ người dùng, đăng ký tham gia, phát hành vé điện tử, kiểm soát check-in bằng QR, theo dõi tỷ lệ tham dự, cấp chứng nhận tham gia, gửi thông báo và nhắc lịch tự động, cùng với dashboard phân tích hiệu quả sự kiện. Đối tượng khách hàng mục tiêu là các hội thảo, workshop công nghệ dành cho cộng đồng lập trình viên, nhằm thay thế cách làm thủ công bằng Google Form/Excel — vốn khó kiểm soát số lượng tham dự thực tế, phải gửi email thủ công và thiếu dữ liệu đánh giá hiệu quả sự kiện.

Mục tiêu chính của workshop là hướng dẫn triển khai backend bằng các dịch vụ serverless của AWS, deploy frontend qua CloudFront, và kiểm thử toàn bộ các luồng nghiệp vụ chính từ quản lý sự kiện, đăng ký vé, check-in, cấp chứng nhận, thông báo/nhắc lịch cho đến dashboard phân tích.

---

## Phạm vi workshop

Workshop này tập trung vào các dịch vụ và công cụ AWS đang được sử dụng trong dự án:

- **Amazon Cognito** để xác thực người dùng bằng JWT, làm cơ sở extract user claims trong các Lambda nghiệp vụ.
- **Amazon API Gateway** để cung cấp các route nghiệp vụ, phần lớn được bảo vệ bởi Cognito Authorizer mặc định (một số route đọc công khai như `GET /events`, `GET /categories` được khai báo `Authorizer: NONE`).
- **AWS Lambda (.NET 8/C#)** để xử lý logic nghiệp vụ backend, gồm 6 function: EventFunction (EventLambda), UserProfileFunction (UserProfileLambda), TicketFunction (RegistrationTicketLambda), AttendanceCertificateFunction (AttendanceCertificateLambda), NotificationFunction (NotificationLambda) và AnalyticsFunction (AnalyticsLambda).
- **Amazon DynamoDB** để lưu trữ dữ liệu ứng dụng qua các bảng **EventManagementEvents**, **EventManagementCategories**, **EventManagementTickets**, **EventManagementAttendance**, **EventManagementUsers**, **EventManagementNotificationLog**.
- **Amazon S3** để lưu banner sự kiện, avatar người dùng, chứng nhận PDF và file build của Frontend.
- **Amazon SES** để gửi email chứng nhận và thông báo cho người dùng (địa chỉ gửi được cấu hình qua biến môi trường `SES_FROM_EMAIL`, cần verify trước khi dùng).
- **Amazon EventBridge (Default Event Bus)** để xử lý thông báo bất đồng bộ: TicketFunction `PutEvents` lên default event bus của account với `source: eventmanagement.ticket`, `detail-type: TicketRegistered`, NotificationFunction lắng nghe qua EventBridge Rule.
- **Amazon EventBridge Scheduler** (`rate(1 hour)`) để trigger định kỳ luồng quét sự kiện sắp diễn ra và gửi email nhắc lịch.
- **Amazon CloudFront + Origin Access Control (OAC)** để phân phối Frontend React đã build tĩnh từ một S3 bucket private hoàn toàn (không public), hiện dùng domain mặc định `*.cloudfront.net`.
- **Amazon CloudWatch Logs** để theo dõi và debug lỗi Lambda/API Gateway.
- **AWS SAM** để build và deploy toàn bộ hạ tầng backend thông qua **template.yaml**.

Frontend React (TypeScript + Vite) được build tĩnh và deploy qua Amazon CloudFront, giao tiếp với backend qua Axios và xác thực bằng Cognito.

---

## Kiến trúc hệ thống

Event Management Platform sử dụng kiến trúc serverless được triển khai tại AWS Singapore Region: **ap-southeast-1**.

Frontend React giao tiếp với Amazon Cognito để xác thực và Amazon API Gateway để gọi các nghiệp vụ backend. API Gateway kiểm tra JWT token do Cognito cấp trước khi chuyển request đến Lambda tương ứng đối với các route được bảo vệ; riêng một số route đọc dữ liệu công khai (danh sách sự kiện, danh mục) và các route thuộc luồng check-in/chứng nhận được khai báo không yêu cầu JWT. Lambda xử lý logic nghiệp vụ chính (quản lý sự kiện, hồ sơ người dùng, đăng ký vé, check-in, tạo chứng nhận, thông báo, phân tích) và đọc/ghi dữ liệu vào DynamoDB. S3 được dùng để lưu banner, avatar và file chứng nhận PDF.

Luồng thông báo được thiết kế theo hai nhánh riêng biệt, đều đi qua **Amazon EventBridge** thay vì gọi trực tiếp hoặc dùng SNS/SQS:

- **Thông báo khi đăng ký vé:** sau khi tạo vé thành công, TicketFunction thực hiện `PutEvents` lên **default event bus** của EventBridge với `source: eventmanagement.ticket` và `detail-type: TicketRegistered`. Một EventBridge Rule lắng nghe đúng pattern này sẽ bất đồng bộ trigger NotificationFunction, function này gọi Amazon SES để gửi email và ghi log vào `EventManagementNotificationLog`.
- **Nhắc lịch trước sự kiện:** một EventBridge Schedule chạy định kỳ (`rate(1 hour)`) trigger trực tiếp NotificationFunction. Lambda tự quét hai bảng `EventManagementEvents` và `EventManagementTickets` để tìm các sự kiện sắp diễn ra, sau đó gọi thẳng Amazon SES để gửi email nhắc lịch và ghi log vào `EventManagementNotificationLog`.

Ở phần frontend, Frontend React sau khi build được upload lên một S3 bucket chặn public hoàn toàn, và được CloudFront đọc thông qua Origin Access Control (OAC) — không truy cập S3 trực tiếp. CloudFront xử lý lỗi 403/404 bằng cách trả về `index.html` để React Router tự xử lý routing phía client. Hiện CloudFront đang dùng domain mặc định do AWS cấp (`*.cloudfront.net`), chưa cấu hình Custom Domain riêng hay chứng chỉ ACM.

![Sơ đồ kiến trúc Serverless Event Platform](/images/5-Workshop/5.1-Workshop-overview/system-architecture.jpg)
<p style="text-align: center;"><i>Hình 5: Sơ đồ kiến trúc và luồng tương tác dữ liệu Serverless trên AWS.</i></p>

---

## Các phần trong workshop

### [5.1 - Tổng quan workshop](5.1-Workshop-overview/)
Phần này giới thiệu dự án Event Management Platform, mục tiêu workshop, kiến trúc hệ thống và các dịch vụ AWS chính được sử dụng.

### [5.2 - Chuẩn bị môi trường](5.2-Prerequisites/)
Phần này liệt kê các công cụ, tài khoản, quyền truy cập và môi trường local cần chuẩn bị trước khi deploy backend và chạy ứng dụng.

### [5.3 - Deploy backend bằng AWS SAM](5.3-Deploy-backend/)
Phần này hướng dẫn build và deploy backend bằng AWS SAM và CloudFormation.

### [5.4 - Cấu hình xác thực với Amazon Cognito](5.4-Configure-authentication/)
Phần này giải thích cách Amazon Cognito được sử dụng cho đăng ký, đăng nhập, cấp JWT token và phân quyền User/Admin.

### [5.5 - Quản lý sự kiện & danh mục (Event Management)](5.5-Event-management-flow/)
Phần này hướng dẫn demo luồng Admin tạo/sửa/xóa sự kiện, quản lý danh mục, upload banner qua presigned URL và bật/tắt hiển thị sự kiện.

### [5.6 - Quản lý hồ sơ người dùng (User Profile)](5.6-User-profile-flow/)
Phần này hướng dẫn demo luồng khởi tạo, xem và cập nhật hồ sơ người dùng, cùng với upload avatar qua presigned URL.

### [5.7 - Luồng đăng ký vé](5.7-Registration-ticketing-flow/)
Phần này hướng dẫn demo luồng đăng ký vé, cơ chế chống overbooking và lưu vé với trạng thái CONFIRMED.

### [5.8 - Luồng check-in bằng QR & cấp chứng nhận PDF](5.8-Checkin-certificate-flow/)
Phần này hướng dẫn demo luồng check-in bằng QR và luồng tạo, gửi chứng nhận PDF qua email.

### [5.9 - Thông báo & nhắc lịch tự động](5.9-Notification-reminder-flow/)
Phần này demo hai luồng thông báo: gửi email xác nhận qua EventBridge (default event bus) khi đăng ký vé thành công, và gửi email nhắc lịch định kỳ qua EventBridge Scheduler trước thời điểm diễn ra sự kiện.

### [5.10 - Dashboard & phân tích hiệu quả sự kiện (Analytics)](5.10-Analytics-dashboard/)
Phần này hướng dẫn xem dashboard tổng hợp và số liệu phân tích theo từng sự kiện, tổng hợp từ dữ liệu Events, Tickets, Attendance và Notification Log.

### [5.11 - Deploy Frontend qua CloudFront](5.11-Deploy-frontend-cloudfront/)
Phần này hướng dẫn build Frontend React, upload lên S3 private và deploy phân phối qua Amazon CloudFront với Origin Access Control.

### [5.12 - Giám sát CloudWatch và dọn dẹp tài nguyên](5.12-Monitoring-and-cleanup/)
Phần này hướng dẫn xem log Lambda/API Gateway bằng CloudWatch Logs và dọn dẹp các resource DynamoDB, S3, CloudFront, Lambda sau khi hoàn thành workshop.

---

## Kết quả mong đợi

Sau khi hoàn thành workshop này, người thực hiện có thể:

- Hiểu kiến trúc serverless của Event Management Platform với 6 Lambda function độc lập.
- Deploy backend bằng AWS SAM và CloudFormation.
- Sử dụng Amazon Cognito để xác thực và extract user claims trong Lambda.
- Kiểm thử luồng quản lý sự kiện, danh mục và upload banner.
- Kiểm thử luồng quản lý hồ sơ người dùng và upload avatar.
- Kiểm thử luồng đăng ký vé, bao gồm cơ chế chống overbooking bằng điều kiện DynamoDB.
- Kiểm thử luồng check-in bằng QR và luồng tạo/gửi chứng nhận PDF qua SES.
- Kiểm thử luồng thông báo bất đồng bộ qua EventBridge khi đăng ký vé, và luồng nhắc lịch định kỳ qua EventBridge Scheduler.
- Xem dashboard phân tích hiệu quả sự kiện.
- Deploy Frontend React và phân phối qua Amazon CloudFront.
- Xem log backend bằng CloudWatch Logs và dọn dẹp tài nguyên AWS sau khi kiểm thử.

---

## Lưu ý quan trọng

Workshop này được thiết kế cho môi trường phát triển và demo dựa trên hiện trạng dự án.

**Về bảo mật (IAM & Authorization):** ba route thuộc AttendanceCertificateFunction — `POST /tickets/checkin`, `GET /tickets/{ticketId}` và `GET /certificates-v2/{ticketId}` — hiện được khai báo `Auth: Authorizer: NONE` trong `template.yaml`, nghĩa là **không yêu cầu JWT token** khi gọi, khác với các route còn lại vốn mặc định được bảo vệ bởi Cognito Authorizer. Đây là điểm cần cân nhắc bổ sung xác thực hoặc cơ chế token có chữ ký (ví dụ ký QR bằng short-lived token) nếu muốn siết chặt bảo mật hơn trong các phiên bản sau.

Cơ chế validate QR khi check-in hiện chỉ kiểm tra sự tồn tại của **TicketId** trong bảng vé, chưa có chữ ký số, thời hạn hay ràng buộc theo từng sự kiện/session.

**template.yaml** hiện có `CognitoUserPoolIdParam` dùng giá trị placeholder mặc định, cần được truyền đúng User Pool ID thật khi deploy (`sam deploy --parameter-overrides CognitoUserPoolIdParam=<PoolId>`).

CloudFront hiện đang dùng domain mặc định do AWS cấp (`*.cloudfront.net`), chưa cấu hình Custom Domain riêng. Nếu sau này cần domain riêng (ví dụ `app.tenmien.com`), cần tạo thêm ACM certificate ở region `us-east-1` (bắt buộc đối với CloudFront) và cấu hình `Aliases` cùng `ViewerCertificate` trong CloudFrontDistribution.

Địa chỉ gửi email (`SES_FROM_EMAIL`) là thông tin cấu hình riêng của từng môi trường deploy; khi trình bày báo cáo/workshop công khai nên thay bằng placeholder thay vì địa chỉ email thật.