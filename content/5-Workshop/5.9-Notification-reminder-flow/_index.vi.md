---
title: "Thông báo, nhắc lịch tự động "
date: 2026-06-29
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

---

Phần này demo hai luồng thông báo độc lập, đều do một Lambda duy nhất (NotificationFunction) xử lý, nhưng được kích hoạt theo hai cơ chế khác nhau và tự phân loại dựa trên cấu trúc JSON của sự kiện đầu vào — không cần route riêng cho từng loại trigger.

Nghiệp vụ này đọc dữ liệu từ các bảng Events, Tickets, Users, ghi log vào bảng Notification Log, và gửi email qua Amazon SES.

---

# PHẦN 1: Thông báo & nhắc lịch tự động

---

Phần này demo hai luồng thông báo độc lập, đều do một Lambda duy nhất (NotificationFunction) xử lý, nhưng được kích hoạt theo hai cơ chế khác nhau và tự phân loại dựa trên cấu trúc JSON của sự kiện đầu vào — không cần route riêng cho từng loại trigger.

Nghiệp vụ này đọc dữ liệu từ các bảng Events, Tickets, Users, ghi log vào bảng Notification Log, và gửi email qua Amazon SES.

---

## Điều kiện tiên quyết

Trước khi test bất kỳ luồng nào, cần đảm bảo email gửi đã được verify trong Amazon SES. Nếu email chưa được verify, toàn bộ luồng gửi mail sẽ thất bại với lỗi MessageRejected từ SES, bất kể code đúng hay sai.

Kiểm tra trạng thái verify:

1. Vào `AWS Console → Amazon SES` → Verified identities
2. Xác nhận địa chỉ khacanh204@gmail.com có trạng thái Verified
3. Nếu chưa verify: chọn Create identity → Email address, nhập email, AWS sẽ gửi link xác nhận vào hộp thư — bấm link đó để hoàn tất

> ⚠️ **Lưu ý:** Nếu tài khoản AWS đang ở chế độ Sandbox (mặc định), SES chỉ cho phép gửi đến các email đã được verify. Cả email gửi (khacanh204@gmail.com) lẫn email nhận đều phải được verify trong Sandbox. Để gửi đến bất kỳ email nào, cần request Production Access trong SES.

---

## Cơ chế phân loại sự kiện đầu vào

Hàm xử lý chính nhận input dạng JSON và tự xác định nguồn gọi dựa trên cấu trúc:

```text
Có trường "httpMethod"        → request từ API Gateway (GET /admin/notifications)
Có trường "detail-type"
  = "Scheduled Event"        → do EventBridge Scheduler gọi (luồng nhắc lịch)
  = "TicketRegistered"       → do EventBridge Rule gọi (luồng xác nhận đăng ký)
```

Đây là một cách thiết kế gộp 3 loại trigger (API, custom event, scheduled event) vào cùng một Lambda handler, thay vì tách thành 3 Lambda riêng.

---

## Luồng 1: Gửi email xác nhận khi đăng ký vé

1. Sau khi hàm đăng ký vé tạo vé thành công (mục 5.7), nó phát một sự kiện lên default event bus với source là eventmanagement.ticket, detail-type là TicketRegistered.
2. Rule EventBridge khớp pattern này sẽ tự động invoke NotificationFunction, truyền cả phần detail chứa thông tin vé (email, họ tên, tiêu đề sự kiện, thời gian, địa điểm).
3. Hàm xử lý luồng này đọc dữ liệu detail, dựng nội dung email xác nhận đăng ký, gửi qua SES từ địa chỉ khacanh204@gmail.com.
4. Kết quả gửi (thành công hay thất bại) được ghi vào bảng Notification Log với loại là RegistrationConfirmed, trạng thái Sent hoặc Failed kèm nội dung lỗi nếu có.

---

## Luồng 2: Gửi email nhắc lịch trước sự kiện

1. EventBridge Scheduler (chạy mỗi giờ một lần) tự động invoke NotificationFunction với một sự kiện dạng "Scheduled Event".
2. Hàm xử lý nhắc lịch quét bảng Events để tìm các sự kiện có thời gian bắt đầu nằm trong khoảng 24 giờ tới kể từ thời điểm hiện tại.
3. Với mỗi sự kiện sắp diễn ra, Lambda lấy danh sách vé đã Confirmed của sự kiện đó.
4. Với từng vé, Lambda dựng nội dung nhắc lịch và gửi email tới địa chỉ email tương ứng của người dùng.
5. Mỗi lần gửi đều được ghi log riêng vào bảng Notification Log với loại là EventReminder.

> ⚠️ **Lưu ý về timeout:** Lambda được cấu hình timeout 30 giây. Nếu số lượng vé Confirmed lớn, luồng nhắc lịch có thể bị timeout trước khi gửi hết email. Trong môi trường test với ít dữ liệu thì không ảnh hưởng.

> ⚠️ **Lưu ý về tần suất:** Vì chạy mỗi giờ một lần, một sự kiện có thể nhận nhắc lịch nhiều lần nếu vẫn còn nằm trong cửa sổ 24 giờ ở các lần quét liên tiếp — không nên kỳ vọng chỉ có đúng 1 email nhắc lịch cho mỗi vé.

---

## Bước 1: Test luồng xác nhận đăng ký (qua đăng ký vé thật)

Đăng ký một vé mới theo đúng các bước ở mục 5.7:

```powershell
Invoke-RestMethod -Uri "$baseUrl/events/$eventId/register" -Method Post -Headers $headers
```

Sau vài giây (do luồng chạy bất đồng bộ qua EventBridge), kiểm tra log thông báo:

```powershell
Invoke-RestMethod -Uri "$baseUrl/admin/notifications" -Method Get -Headers $headers
```

Response mong đợi (rút gọn, chỉ hiển thị bản ghi mới nhất):

```json
{
  "notificationId": "<GUID tự sinh cho mỗi lần gửi>",
  "eventId": "<EventId của sự kiện vừa đăng ký>",
  "email": "<Email của User vừa đăng ký>",
  "type": "RegistrationConfirmed",
  "status": "Sent",
  "errorMessage": null,
  "sentAt": "<Thời điểm gửi thực tế, UTC ISO 8601>"
}
```

---

## Bước 2: Kiểm tra EventBridge Rule đã invoke NotificationFunction

Mở `Amazon EventBridge → Rules`, chọn rule lắng nghe sự kiện đăng ký vé, tab Monitoring, xác nhận số lượt Invocations tăng thêm đúng 1 sau lần đăng ký ở Bước 1.

---

## Bước 3: Kiểm tra log Lambda cho luồng xác nhận đăng ký

Mở `CloudWatch → Log groups → log group` của NotificationFunction, tìm log stream tại đúng thời điểm test, xác nhận có các dòng log tương ứng với luồng xử lý đăng ký vé và kết quả gọi SES.

---

## Bước 4: Test luồng nhắc lịch bằng cách giả lập Scheduled Event

Vì luồng nhắc lịch chỉ tự chạy mỗi giờ, có thể chủ động test bằng cách invoke trực tiếp Lambda với payload giả lập đúng cấu trúc mà EventBridge Scheduler gửi.

Tạo file test-reminder.json với nội dung:

```json
{
  "version": "0",
  "id": "test-event-id",
  "source": "aws.scheduler",
  "account": "<AccountId>",
  "region": "ap-southeast-2",
  "detail-type": "Scheduled Event",
  "detail": {}
}
```

Sau đó invoke Lambda bằng AWS CLI:

```powershell
aws lambda invoke `
  --function-name NotificationLambda `
  --payload file://test-reminder.json `
  --cli-binary-format raw-in-base64-out `
  response.json

Get-Content response.json
```

> ⚠️ **Lưu ý:** Dùng file JSON thay vì inline string để tránh lỗi encoding trên PowerShell. Để có dữ liệu nhắc lịch thật sự xuất hiện, cần đảm bảo có ít nhất một sự kiện với thời gian bắt đầu nằm trong 24 giờ tới và đã có vé Confirmed.

---

## Bước 5: Kiểm tra kết quả luồng nhắc lịch

```powershell
Invoke-RestMethod -Uri "$baseUrl/admin/notifications" -Method Get -Headers $headers
```

Xác nhận xuất hiện thêm các bản ghi mới với loại là EventReminder, số lượng bản ghi bằng đúng số vé Confirmed của các sự kiện sắp diễn ra trong 24 giờ tới.

---

## Bước 6: Kiểm tra dữ liệu trong DynamoDB

Mở `DynamoDB → Tables → bảng Notification Log → Explore table items`, đối chiếu trực tiếp các item với response ở Bước 1 và Bước 5.

---

## Kết quả mong đợi (Phần 1)

Sau khi hoàn thành phần Notification, người thực hiện có thể:

- Hiểu cách một Lambda duy nhất phân loại và xử lý 3 loại trigger khác nhau (API Gateway, EventBridge custom event, EventBridge Scheduler).
- Xác nhận email khacanh204@gmail.com đã được verify trong SES và có thể gửi mail thành công.
- Test được luồng gửi email xác nhận đăng ký, xác nhận qua log thông báo và email thật nhận được trong hộp thư.
- Test được luồng gửi email nhắc lịch bằng cách giả lập Scheduled Event qua AWS CLI với file JSON.
- Đối chiếu dữ liệu log thông báo giữa API response và item thật trong DynamoDB.
- Xác nhận số lượt invoke của EventBridge Rule khớp với số lần đăng ký vé đã test.

---

