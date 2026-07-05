---
title: "Deploy backend AWS SAM"
date: 2026-06-29
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

---

Phần này hướng dẫn build và deploy backend Event Management Platform bằng AWS SAM và AWS CloudFormation.

Quá trình deploy backend sẽ tạo toàn bộ tài nguyên serverless: Cognito Authorizer (dùng User Pool có sẵn), API Gateway, 6 Lambda function (.NET 8), 6 bảng DynamoDB, 4 S3 bucket, EventBridge Rule + Scheduler, CloudFront Distribution cho frontend, cùng các IAM Role/Policy tương ứng.

---

## Bước 1: Mở thư mục backend

Mở terminal và chuyển đến thư mục backend:

```powershell
cd aws-event-management\BE
```

Kiểm tra các file/thư mục chính:

```powershell
dir
```

Đảm bảo có:

```text
EventManagement.Serverless.sln
template.yaml
src\
```

---

## Bước 2: Restore dependencies backend

Vì backend viết bằng .NET, không dùng npm install như dự án Node.js — cần restore package .NET cho toàn bộ solution:

```powershell
dotnet restore EventManagement.Serverless.sln
```

Lệnh này khôi phục các package NuGet cần thiết cho cả 6 Lambda function và project Shared.

![restore](/images/5-Workshop/5.3-Deploy-backend/restore.jpg)
---

## Bước 3: Kiểm tra SAM template

Mở file template.yaml trong Visual Studio Code. File này định nghĩa toàn bộ hạ tầng, gồm:

| Nhóm resource | Tên logic trong template.yaml |
|---|---|
| Cognito Authorizer | CognitoAuthorizer (tham chiếu User Pool có sẵn qua tham số CognitoUserPoolIdParam) |
| Lambda function | EventFunction, UserProfileFunction, TicketFunction, AttendanceCertificateFunction, NotificationFunction, AnalyticsFunction |
| DynamoDB Table | EventManagementEventsTable, EventManagementCategoriesTable, EventManagementTicketsTable, EventManagementAttendanceTable, EventManagementUsersTable, EventManagementNotificationLogTable |
| S3 Bucket | EventManagementEventBannersBucket, EventManagementUserAvatarsBucket, EventManagementCertificatesBucket, FrontendBucket |
| EventBridge | Rule OnTicketRegistered (default event bus) + Schedule EventReminderSchedule (rate(1 hour)), khai báo ngay trong Events của NotificationFunction |
| Frontend hosting | FrontendOAC, FrontendDistribution (CloudFront), FrontendBucketPolicy |

Khác với một số mẫu SAM khác, dự án này **không tạo Cognito User Pool mới** — CognitoAuthorizer tham chiếu tới một User Pool đã tồn tại sẵn thông qua tham số CognitoUserPoolIdParam, cần được truyền giá trị thật khi deploy.

Backend được quản lý hoàn toàn dưới dạng Infrastructure as Code thông qua AWS SAM và CloudFormation.

---

## Bước 4: Build backend

```powershell
sam build
```

SAM sẽ tự động build từng Lambda function .NET bằng dotnet build bên dưới. Nếu build cache gây lỗi:

```powershell
sam build --no-cached
```

Kết quả mong đợi:

```text
Build Succeeded
```

---

## Bước 5: Deploy backend (lần đầu — guided)

Vì repo không có sẵn samconfig.toml (thường không commit vì chứa cấu hình riêng từng môi trường), lần deploy đầu tiên nên chạy ở chế độ guided để SAM hỏi và lưu lại cấu hình:

```powershell
sam deploy --guided
```

SAM sẽ hỏi lần lượt:

```text
Stack Name: event-management-backend-dev
AWS Region: ap-southeast-1
Parameter CognitoUserPoolIdParam: <Cognito User Pool ID thật>
Confirm changes before deploy: Y
Allow SAM CLI IAM role creation: Y
Save arguments to configuration file: Y
```

Sau bước này, samconfig.toml sẽ được tạo tự động, và các lần deploy sau chỉ cần:

```powershell
sam deploy
```

hoặc để hạn chế hỏi xác nhận nhiều lần:

```powershell
sam deploy --no-confirm-changeset --no-fail-on-empty-changeset
```

---

## Bước 6: Kiểm tra CloudFormation Stack

Mở AWS Management Console → `CloudFormation > Stacks`, tìm stack vừa tạo (VD: event-management-backend-dev).

Kiểm tra trạng thái stack là CREATE_COMPLETE hoặc UPDATE_COMPLETE.

![User Pool](/images/5-Workshop/5.3-Deploy-backend/CloudFormation.jpg)

Nếu stack bị lỗi, mở tab **Events** để xem tài nguyên nào deploy thất bại.

---

## Bước 7: Lấy Output — API Gateway endpoint & CloudFront domain

Sau khi deploy thành công, xem tab **Outputs** của stack:

```text
FrontendCloudFrontDomain   → domain CloudFront để truy cập Frontend
FrontendBucketName         → tên S3 bucket chứa file build Frontend
FrontendDistributionId     → CloudFront Distribution ID (dùng để invalidate cache)
```

Đồng thời lấy API Gateway endpoint từ tab **Resources** hoặc từ output của lệnh sam deploy:

```text
https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev
```

Endpoint này sẽ được cấu hình vào file .env của Frontend React (VITE_API_BASE_URL).

*Giá trị thật (API endpoint, CloudFront domain) sẽ được cung cấp riêng khi demo trực tiếp.*

---

## Bước 8: Kiểm tra DynamoDB Tables

Mở `DynamoDB > Tables`, kiểm tra 6 bảng đã được tạo:

```text
EventManagementEvents
EventManagementCategories
EventManagementTickets
EventManagementAttendance
EventManagementUsers
EventManagementNotificationLog
```

Tất cả đều dùng BillingMode: PAY_PER_REQUEST (on-demand), phù hợp cho môi trường workshop/demo vì không cần ước lượng capacity trước và tránh phát sinh chi phí khi không có traffic.

![User Pool](/images/5-Workshop/5.3-Deploy-backend/DynamoDB.jpg)

---

## Bước 9: Kiểm tra S3 Buckets

Mở `Amazon S3 > Buckets`, kiểm tra 4 bucket được tạo bởi stack:

| Bucket | Mục đích |
|---|---|
| event-management-user-avatars-<region>-<account-id> | Lưu avatar người dùng |
| Banner bucket (tên tự sinh) | Lưu banner sự kiện |
| event-management-certificates-<account-id>-<region> | Lưu file chứng nhận PDF |
| event-management-frontend-<account-id>-<region> | Lưu file build Frontend React, phân phối qua CloudFront |

Riêng FrontendBucket được cấu hình **chặn public hoàn toàn** (BlockPublicAcls/BlockPublicPolicy/IgnorePublicAcls/RestrictPublicBuckets: true) — chỉ CloudFront mới đọc được thông qua Origin Access Control, không truy cập trực tiếp bằng URL S3.

![Buckets](/images/5-Workshop/5.3-Deploy-backend/Buckets.jpg)
---

## Bước 10: Kiểm tra Cognito User Pool (tham chiếu, không tạo mới)

Mở `Amazon Cognito > User pools`, kiểm tra đúng User Pool đã dùng làm giá trị cho CognitoUserPoolIdParam lúc deploy.

Lưu ý: User Pool này **không do stack này tạo ra** — backend chỉ tham chiếu tới ARN của User Pool có sẵn để cấu hình CognitoAuthorizer cho API Gateway.

![User Pool](/images/5-Workshop/5.3-Deploy-backend/UserPool.jpg)


---

## Bước 11: Kiểm tra Lambda và CloudWatch Logs

Mở `AWS Lambda > Functions`, kiểm tra 6 function đã được tạo: EventLambda, UserProfileLambda, TicketLambda, AttendanceCertificateLambda, NotificationLambda, AnalyticsLambda.

!["Lệnh khởi tạo FE](/images/5-Workshop/5.3-Deploy-backend/Functions.jpg)

Sau đó mở `CloudWatch > Log groups`, tìm các log group dạng /aws/lambda/<function-name>. Các log này hữu ích khi debug lỗi API, lỗi xác thực và lỗi nghiệp vụ.

---

## Bước 12: Kiểm tra CloudFront Distribution

Mở `Amazon CloudFront > Distributions`, kiểm tra distribution vừa tạo cho Frontend.

Truy cập thử domain CloudFront (dạng <id>.cloudfront.net) — nếu Frontend chưa được build & upload lên FrontendBucket, bước này sẽ chưa hiển thị giao diện (sẽ thực hiện ở mục 5.11).

!["Lệnh khởi tạo FE](/images/5-Workshop/5.3-Deploy-backend/CloudFront.jpg)

---

## Các lỗi deploy thường gặp

### AWS credentials chưa được cấu hình

```powershell
aws configure
aws sts get-caller-identity
```

### SAM build bị lỗi

```powershell
sam build --no-cached
```

Kiểm tra phiên bản .NET SDK:

```powershell
dotnet --version
```

### Deploy báo thiếu tham số CognitoUserPoolIdParam

Nếu không dùng --guided, cần truyền tham số trực tiếp:

```powershell
sam deploy --parameter-overrides CognitoUserPoolIdParam=<Cognito User Pool ID thật>
```

### CloudFormation stack bị rollback

Mở `CloudFormation > Stacks > <stack-name> > Events`, đọc event bị lỗi đầu tiên (thường là nguyên nhân gốc) và sửa tài nguyên hoặc quyền liên quan.

---

## Kết quả mong đợi

Sau khi hoàn thành phần này:

- Backend stack với 6 Lambda function, 6 bảng DynamoDB, 4 S3 bucket, CloudFront Distribution được deploy thành công.
- API Gateway endpoint và CloudFront domain đã sẵn sàng để cấu hình cho Frontend.
- Cognito Authorizer đã được liên kết đúng với User Pool có sẵn.
- Dự án đã sẵn sàng cho phần cấu hình xác thực (5.4) và các luồng nghiệp vụ tiếp theo.