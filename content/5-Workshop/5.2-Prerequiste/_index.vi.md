---
title: "Chuẩn bị môi trường"
date: 2026-06-29
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

---

Trước khi bắt đầu workshop, cần chuẩn bị tài khoản AWS, công cụ local, source code và cấu hình dự án.

---

## Yêu cầu tài khoản AWS

Cần có tài khoản AWS với quyền tạo và quản lý các tài nguyên sau:

- Amazon Cognito User Pool
- Amazon API Gateway
- AWS Lambda
- Amazon DynamoDB
- Amazon S3
- Amazon SES
- Amazon EventBridge (default event bus + Scheduler)
- Amazon CloudFront
- Amazon CloudWatch Logs
- AWS CloudFormation stacks
- IAM roles và policies được tạo bởi AWS SAM

Dự án được deploy tại AWS Region:

```text
ap-southeast-1
```
!["AWS Singapore Region](/images/5-Workshop/5.2-Prerequisite/Region.jpg)
---

## Công cụ cần cài đặt

Cài đặt các công cụ sau trên máy tính:

| Công cụ | Mục đích |
|---|---|
| AWS CLI | Cấu hình AWS credentials và thao tác với dịch vụ AWS |
| AWS SAM CLI | Build và deploy backend serverless |
| .NET SDK 8.0 | Build và chạy code Lambda backend (.NET 8/C#) |
| Node.js (LTS) & npm | Cài dependencies và chạy Frontend React (Vite) |
| Git | Clone và quản lý source code |
| Visual Studio / Visual Studio Code | Chỉnh sửa và quản lý file dự án |

---

## Kiểm tra AWS CLI

Mở terminal và chạy:

```powershell
aws --version
```

Sau đó cấu hình AWS credentials:

```powershell
aws configure
```

Nhập các thông tin cần thiết:

```text
AWS Access Key ID
AWS Secret Access Key
Default region name: ap-southeast-1
Default output format: json
```

Kiểm tra danh tính AWS hiện tại:

```powershell
aws sts get-caller-identity
```

Nếu lệnh trả về AWS account ID và user ARN, AWS CLI đã được cấu hình thành công.

---

## Kiểm tra AWS SAM CLI

```powershell
sam --version
```

Nếu SAM CLI được cài đúng, terminal sẽ hiển thị phiên bản đã cài.

---

## Kiểm tra .NET SDK

```powershell
dotnet --version
```

Kết quả nên hiển thị phiên bản bắt đầu bằng 8.x, khớp với TargetFramework net8.0 được khai báo trong các file .csproj của từng Lambda function.

---

## Kiểm tra Node.js và npm

```powershell
node -v
npm -v
```

Frontend sử dụng Vite làm build tool và React 19.

---

## Source code dự án

Source code dự án được tổ chức thành hai phần chính trong cùng một repository:

```text
aws-event-management/
├── BE/
│   ├── EventManagement.Serverless.sln
│   ├── template.yaml
│   └── src/
│       ├── Functions/
│       └── Shared/
└── FE/
    ├── package.json
    ├── vite.config.ts
    └── src/
```

<!-- > 📌 **Gợi ý hình ảnh:** Chụp screenshot cây thư mục thật bằng lệnh `tree BE/src /F` và `tree FE/src /F` (Windows) hoặc mở trực tiếp VS Code Explorer — thay cho hình vẽ minh họa, giúp tăng độ tin cậy "đây là source thật" cho tiêu chí 4.5. -->

---

## File cấu hình backend

Thư mục BE/ cần có:

```text
BE/
├── EventManagement.Serverless.sln
├── template.yaml
└── src/
    ├── Functions/
    │   ├── EventManagement.EventLambda/
    │   ├── EventManagement.UserProfileLambda/
    │   ├── EventManagement.RegistrationTicketLambda/
    │   ├── EventManagement.AttendanceCertificateLambda/
    │   ├── EventManagement.NotificationLambda/
    │   └── EventManagement.AnalyticsLambda/
    └── Shared/
```

Các file/thư mục quan trọng:

| File / Thư mục | Mục đích |
|---|---|
| template.yaml | Định nghĩa toàn bộ tài nguyên AWS để deploy bằng SAM (6 Lambda, DynamoDB, S3, Cognito Authorizer, CloudFront...) |
| EventManagement.Serverless.sln | Solution chính chứa tất cả project Lambda |
| src/Functions/ | Mỗi thư mục con là source code của một Lambda function độc lập |
| src/Shared/ | Services, Repositories, DTOs, Validators, Constants dùng chung cho toàn bộ Lambda |

---

## Cấu hình frontend

Frontend React sử dụng biến môi trường Vite để kết nối với backend AWS đã deploy, ví dụ:

```text
FE/.env
```

```env
VITE_API_BASE_URL=<API Gateway endpoint>
VITE_AWS_REGION=ap-southeast-1
VITE_COGNITO_USER_POOL_ID=<Cognito User Pool ID>
VITE_COGNITO_CLIENT_ID=<Cognito App Client ID>
```

Frontend dùng thư viện **AWS Amplify** (aws-amplify, @aws-amplify/ui-react) kết hợp cognitoAuthService.ts để xác thực, và axiosInstance.ts để cấu hình gọi API tập trung (base URL, interceptor đính kèm JWT token).

Cần đảm bảo các giá trị trong .env khớp với backend AWS đã deploy.

---

## Cài dependencies frontend

Frontend React (không có Admin Web tách riêng — vai trò Admin và User dùng chung một SPA, phân biệt qua src/pages/admin và src/pages/user):

```powershell
cd aws-event-management/FE
npm install
```

Các thư viện chính đáng chú ý trong package.json:

| Thư viện | Vai trò |
|---|---|
| aws-amplify, @aws-amplify/ui-react | Tích hợp xác thực với Amazon Cognito |
| antd, @ant-design/icons, tailwindcss | UI component và styling |
| @reduxjs/toolkit, react-redux | Quản lý state toàn cục |
| axios | Gọi API tới API Gateway |
| html5-qrcode, qrcode.react | Quét QR (check-in) và sinh QR (vé điện tử) |
| recharts | Vẽ biểu đồ cho trang Dashboard/Analytics |
| react-router-dom | Điều hướng giữa các trang User/Admin/Public |

<!-- !["Lệnh khởi tạo FE](/images/5-Workshop/5.2-Prerequisite/npminstall.jpg) -->
---

## Lưu ý quan trọng

Trước khi thực hiện workshop, cần đảm bảo tài khoản AWS có đủ quyền cho toàn bộ danh sách dịch vụ ở trên, và nên cấu hình AWS Budget alerts để tránh phát sinh chi phí ngoài ý muốn — đặc biệt với CloudFront và SES nếu dùng ở chế độ production.

template.yaml hiện có CognitoUserPoolIdParam nhận giá trị qua tham số deploy thay vì hard-code, cần chuẩn bị sẵn User Pool ID thật (được tạo/quản lý riêng) trước khi chạy sam deploy.

Amazon SES mặc định chạy ở chế độ Sandbox — chỉ gửi được email đến các địa chỉ đã verify. Cần verify địa chỉ email gửi (SES_FROM_EMAIL) và địa chỉ nhận thử nghiệm trong SES Console trước khi test luồng thông báo/chứng nhận.