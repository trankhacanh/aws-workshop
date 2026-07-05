---
title: "Giám sát CloudWatch và dọn dẹp tài nguyên"
date: 2026-06-29
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

---

Phần này hướng dẫn xem log và theo dõi các chỉ số vận hành của hệ thống bằng Amazon CloudWatch (Logs, Metrics), đồng thời liệt kê các bước dọn dẹp toàn bộ tài nguyên đã tạo trong quá trình workshop để tránh phát sinh chi phí ngoài dự kiến sau khi kết thúc.

Nghiệp vụ Notification của hệ thống được xử lý bởi một Lambda C# duy nhất có tên **NotificationFunction** — nhưng được kích hoạt bởi hai nguồn EventBridge độc lập, không qua SNS/SQS:

1. **Luồng nhắc lịch (reminder)**: một schedule tên **EventReminderSchedule** (xây bằng EventBridge Scheduler, chạy mỗi giờ một lần) trigger thẳng NotificationFunction. Lambda tự quét bảng Events và Tickets để tìm các sự kiện sắp diễn ra, gọi SES gửi email, rồi ghi log vào bảng notification log.
2. **Luồng thông báo đăng ký vé**: sau khi hàm đăng ký vé tạo vé thành công, nó sẽ phát ra một sự kiện tên **TicketRegistered** lên một Custom Event Bus. Một rule tên **OnTicketRegistered** lắng nghe đúng sự kiện này và bất đồng bộ trigger NotificationFunction, Lambda gọi SES gửi email rồi ghi log vào cùng bảng notification log.

Cả rule EventBridge, schedule EventBridge và bảng notification log đều được khai báo trong file template hạ tầng (template.yaml), quản lý bằng IaC cùng với các resource còn lại của hệ thống.

Demo phần giám sát sẽ dùng AWS Console là chính — cụ thể là CloudWatch Logs Insights — vì đây là công cụ trực quan nhất để đọc log Lambda. Phần dọn dẹp sẽ dùng AWS CLI kết hợp SAM CLI để đảm bảo xóa đúng thứ tự.

---

## Phần A — Giám sát CloudWatch

### A.1. Log Group của từng Lambda

CloudWatch tự động tạo một Log Group cho mỗi Lambda function, đặt tên theo chính function đó (theo quy tắc /aws/lambda/tên-function). Trong dự án này, sẽ có một log group tương ứng cho từng function sau:

- Quản lý sự kiện
- Đăng ký vé
- Điểm danh & chứng chỉ
- Hồ sơ người dùng
- Notification — log group này đặc biệt vì ghi nhận invocation từ cả hai nguồn: schedule nhắc lịch theo giờ và sự kiện đăng ký vé.

Trên CloudWatch Console, mở mục **Log groups**, rồi chọn log group cần xem. Mỗi lần Lambda "cold start", một Log Stream mới sẽ được tạo bên trong log group đó, nên một log group có thể chứa nhiều stream.

<!-- > 📌 **Gợi ý hình ảnh:** Chụp danh sách Log Groups trên CloudWatch Console, thấy đủ tên các Lambda của dự án. -->

### A.2. Dùng CloudWatch Logs Insights để tìm kiếm nhanh

Thay vì mở từng log stream để đọc thủ công, CloudWatch có sẵn một công cụ tìm kiếm tên là Logs Insights, cho phép lọc trên toàn bộ log của một log group cùng lúc, giống như tìm chữ trong một văn bản — không cần biết bất kỳ ngôn ngữ truy vấn đặc biệt nào để dùng được nó.

Cách dùng: mở CloudWatch, vào mục **Logs → Logs Insights**, chọn (các) log group muốn tìm ở ô dropdown, chọn khoảng thời gian (ví dụ "Last 1 hour"), gõ từ hoặc cụm từ cần tìm, rồi bấm **Run query**. Một vài ví dụ tìm kiếm hữu ích cho dự án này:

- **Tìm các lượt đăng ký bị từ chối vì sự kiện đã hết chỗ**: chọn log group của hàm đăng ký vé và tìm cụm từ xuất hiện trong thông báo lỗi đó (ví dụ "Hết vé"). Sắp xếp theo thời gian mới nhất trước và giới hạn kết quả ở khoảng 20 dòng gần nhất để danh sách dễ đọc.
- **Tìm lỗi trên nhiều function cùng lúc**: có thể chọn nhiều log group cùng lúc trong cùng một ô tìm kiếm, rồi tìm từ "Exception" để thấy mọi lỗi xảy ra trên toàn hệ thống trong khoảng thời gian đã chọn.
- **Phân biệt hai nguồn trigger của Notification**: vì NotificationFunction được kích hoạt bởi hai nguồn khác nhau, có thể tách riêng bằng cách tìm các từ khóa khác nhau — tìm "TicketRegistered" để chỉ xem các email gửi ngay sau khi có người đăng ký vé, hoặc tìm "reminder" (hoặc cụm từ tiếng Việt như "sắp diễn ra") để chỉ xem các email do schedule theo giờ gửi.

Trình soạn Logs Insights trên Console vốn đã có sẵn các mẫu tìm kiếm cho phép chọn từ dropdown và chỉnh sửa lại bằng cách đổi từ khóa cần tìm — không cần phải nhớ bất kỳ cú pháp đặc biệt nào vẫn có thể ra được kết quả hữu ích.

<!-- > 📌 **Gợi ý hình ảnh:** Chụp kết quả 2 lượt tìm kiếm ở trên, cho thấy tách biệt rõ giữa log của luồng nhắc lịch và log của luồng đăng ký vé. -->

### A.3. Theo dõi Metrics của Lambda, API Gateway, DynamoDB và EventBridge

Trên CloudWatch, ở mục **Metrics → All metrics**, các chỉ số cần theo dõi gồm:

- **Lambda**: số lần invoke, số lỗi, thời gian chạy, và số lần bị throttle — theo từng function, đặc biệt lưu ý NotificationFunction vì đây là điểm hội tụ của cả hai luồng EventBridge.
- **API Gateway**: số lượng request, lỗi 4XX, lỗi 5XX, và độ trễ (latency) — theo từng API/stage.
- **DynamoDB**: dung lượng ghi đã dùng và số request bị throttle — theo dõi trên các bảng Tickets/Events (tranh chấp ghi lúc mở đăng ký vé) và trên bảng notification log (được ghi liên tục bởi cả hai luồng notification).
- **Rule EventBridge** lắng nghe sự kiện đăng ký vé: số lần invoke, số lần invoke thất bại, và số lần bị throttle. Số lần invoke thất bại tăng bất thường nghĩa là sự kiện đăng ký vé không tới được NotificationFunction.
- **Schedule EventBridge** gửi nhắc lịch: số lần cố gắng invoke và số lỗi ở đích đến (target error). Số lỗi đích đến lớn hơn 0 nghĩa là schedule đã chạy đúng giờ nhưng gọi NotificationFunction thất bại (có thể do lỗi quyền hoặc lỗi SES).

Có thể dựng một Dashboard gộp tất cả các chỉ số trên vào một màn hình để theo dõi trong lúc demo end-to-end (đăng ký vé → nhận mail xác nhận → chờ tới giờ → nhận mail nhắc lịch), thay vì mở nhiều tab riêng lẻ.

Hệ thống hiện chưa cấu hình CloudWatch Alarms (ví dụ alarm khi Lambda có lỗi, khi lỗi 5XX tăng vọt, hoặc khi số lỗi đích đến của schedule nhắc lịch lớn hơn 0). Đây là hạng mục còn thiếu, cần ghi vào phần khuyến nghị của báo cáo.

<!-- > 📌 **Gợi ý hình ảnh:** Chụp một Dashboard CloudWatch tự tạo, gộp số lần invoke/lỗi của Lambda, số request/4XX/5XX của API Gateway, và số lần invoke/thất bại của rule + schedule EventBridge trong lúc test luồng đăng ký + nhắc lịch. -->

---

## Phần B — Dọn dẹp tài nguyên (Clean-up)

Vì rule EventBridge, schedule EventBridge và bảng notification log đều được khai báo trong file template hạ tầng, các resource này sẽ tự động bị xóa cùng lúc ở Bước B.3 bên dưới — không cần thao tác xóa tay riêng cho chúng. Cần thực hiện đúng thứ tự các bước dưới đây để tránh lỗi xóa thiếu phụ thuộc (ví dụ CloudFormation không xóa được bucket S3 nếu bucket còn file).

### B.1. Làm rỗng các bucket S3 trước khi xóa stack

CloudFormation và SAM không tự xóa bucket S3 còn chứa file. Cần làm rỗng thủ công từng bucket trước:

```powershell
aws s3 rm s3://event-management-frontend-<suffix> --recursive
aws s3 rm s3://<EventManagementEventBannersBucket-name> --recursive
aws s3 rm s3://<EventManagementCertificatesBucket-name> --recursive
aws s3 rm s3://<EventManagementUserAvatarsBucket-name> --recursive
```

### B.2. Xóa CloudFront Distribution

CloudFront không thể xóa ngay khi đang bật hoặc đang ở trạng thái "Deploying". Cần disable trước, rồi đợi trạng thái chuyển sang "Deployed" thì mới xóa được:

```powershell
aws cloudfront get-distribution-config --id <distribution-id> > dist-config.json
# Trong dist-config.json, sửa "Enabled": true thành "Enabled": false, giữ nguyên ETag

aws cloudfront update-distribution `
  --id <distribution-id> `
  --distribution-config file://dist-config.json `
  --if-match <ETag-from-get-distribution-config>

# Sau khi trạng thái chuyển thành "Deployed" (kiểm tra qua get-distribution), mới xóa được:
aws cloudfront delete-distribution --id <distribution-id> --if-match <ETag-moi-nhat>
```

<!-- > 📌 **Gợi ý hình ảnh:** Chụp CloudFront Console lúc Distribution chuyển trạng thái từ "Deploying" sang "Disabled/Deployed", đủ điều kiện xóa. -->

### B.3. Xóa SAM Stack (Lambda, API Gateway, DynamoDB, EventBridge Rule/Schedule, IAM Role liên quan)

```powershell
sam delete --stack-name event-management-stack --region ap-southeast-1
```

Lệnh này xóa CloudFormation stack cùng toàn bộ resource định nghĩa trong file template hạ tầng, bao gồm:

- Toàn bộ Lambda function (quản lý sự kiện, đăng ký vé, điểm danh & chứng chỉ, hồ sơ người dùng, notification...).
- API Gateway.
- Toàn bộ bảng DynamoDB: events, categories, tickets, attendance, users, và notification log.
- Rule EventBridge và schedule EventBridge đã nêu ở trên.
- IAM Role/Policy đi kèm (bao gồm các quyền mà các resource trên cần để phát sự kiện, chạy theo lịch, và gọi Lambda function).
- Các bucket S3 (chỉ xóa được nếu đã rỗng ở bước B.1).

Nếu lệnh xóa báo lỗi do resource bị khai báo lặp ID trong file template, cần vào CloudFormation Console, mở stack và xóa thủ công tại đó, xử lý từng resource báo lỗi xóa thất bại một lượt. Riêng schedule nhắc lịch, nếu bị kẹt không xóa được, kiểm tra xem nó có đang chạy hoặc còn lượt thực thi dang dở hay không trước khi xóa lại.

### B.4. Xóa Cognito User Pool

```powershell
aws cognito-idp delete-user-pool --user-pool-id <user-pool-id>
```

User Pool cần được xóa bằng lệnh riêng này, vì nó được tạo thủ công ở mục 5.4 chứ không phải qua file template hạ tầng — nên sẽ không tự bị xóa cùng với phần còn lại của stack.

### B.5. Xóa bucket Frontend (sau khi đã rỗng ở B.1)

```powershell
aws s3api delete-bucket --bucket event-management-frontend-<suffix> --region ap-southeast-1
```

### B.6. Kiểm tra lại toàn bộ tài nguyên còn sót

- **CloudFormation Console**: xác nhận stack đã ở trạng thái xóa hoàn tất.
- **S3 Console**: xác nhận không còn bucket nào của dự án.
- **CloudFront Console**: xác nhận Distribution đã biến mất khỏi danh sách.
- **DynamoDB Console**: xác nhận không còn bảng nào của dự án, bao gồm bảng notification log.
- **EventBridge Console**: xác nhận mục Schedules không còn schedule nhắc lịch, và mục Rules không còn rule đăng ký vé.
- **Cognito Console**: xác nhận User Pool đã bị xóa.
- **Billing Console → Cost Explorer**: kiểm tra không còn chi phí phát sinh mới sau thời điểm dọn dẹp — đặc biệt lưu ý schedule nhắc lịch, vì nếu quên xóa, nó sẽ tiếp tục kích hoạt Notification function mỗi giờ vô thời hạn.

<!-- > 📌 **Gợi ý hình ảnh:** Chụp CloudFormation Console, S3 Console và EventBridge Console (mục Schedules + Rules) ở trạng thái trống, không còn resource nào của dự án — làm bằng chứng đã dọn dẹp hoàn tất. -->

---

## Kết quả mong đợi

Sau khi hoàn thành phần này, người thực hiện có thể:

- Đọc và tìm kiếm log Lambda bằng CloudWatch Logs Insights để phục vụ debug, phân biệt được log của luồng nhắc lịch và luồng đăng ký vé trên cùng một Notification function.
- Nắm được các nhóm Metrics quan trọng cần theo dõi cho hệ thống Serverless, bao gồm Metrics riêng của rule EventBridge và schedule EventBridge.
- Nhận diện đúng hạng mục còn thiếu (CloudWatch Alarms) để ghi vào phần khuyến nghị của báo cáo.
- Thực hiện dọn dẹp toàn bộ tài nguyên AWS đã tạo theo đúng thứ tự phụ thuộc, xác nhận rule EventBridge, schedule và bảng notification log được xử lý tự động khi xóa stack.
- Xác minh được bằng chứng dọn dẹp hoàn tất trên từng Console dịch vụ liên quan, kể cả EventBridge.