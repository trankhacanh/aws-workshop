---
title: "Luồng check-in bằng QR & cấp chứng nhận PDF"
date: 2026-06-29
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

---

Phần này demo luồng check-in vé bằng QR và luồng tạo, gửi chứng nhận PDF qua email sau khi check-in thành công.

Toàn bộ nghiệp vụ này do AttendanceCertificateFunction xử lý, đọc/ghi vào bảng Tickets và bảng Attendance, lưu file PDF vào bucket chứng chỉ, và gửi email qua Amazon SES.

Cả 3 route của module này (check-in vé, xem chi tiết vé, lấy chứng nhận) đều không yêu cầu JWT — phù hợp với việc quét QR tại quầy check-in có thể không cần đăng nhập lại trên thiết bị quét.

---

## Luồng check-in

1. Thiết bị quét (hoặc client bất kỳ) gửi request check-in với ticketId lấy từ mã QR trên vé.
2. Lambda tra vé theo TicketId, kiểm tra trạng thái:
   - Nếu trạng thái là Checked_in → trả lỗi 409 Conflict, không cho check-in lại.
   - Nếu trạng thái không thuộc nhóm hợp lệ (Confirmed, Success, Pending_checkin, Pending) → trả lỗi 400 Bad Request.
3. Lambda kiểm tra thêm ở bảng Attendance xem vé đã có bản ghi check-in cho đúng eventId chưa — một lớp kiểm tra thứ hai độc lập với trường trạng thái trên vé.
4. Nếu hợp lệ, ghi bản ghi mới vào bảng Attendance (gồm thời điểm check-in, phương thức check-in) và cập nhật vé sang trạng thái Checked_in.

---

## Luồng tạo và gửi chứng nhận

1. Client gửi request lấy chứng nhận theo ticketId.
2. Lambda kiểm tra vé đã check-in chưa (dựa vào bảng Attendance) — nếu chưa, trả lỗi "Bạn chưa check-in nên chưa thể tải chứng nhận."
3. Nếu đã check-in, Lambda sinh certificateId theo định dạng CERT- kèm 8 ký tự đầu của TicketId viết hoa, tạo file PDF bằng thư viện QuestPDF (Community License) với các trường: họ tên, tên sự kiện, địa điểm, ngày cấp.
4. File PDF được upload lên S3 với đường dẫn certificates/{ticketId}.pdf, sau đó Lambda sinh presigned URL có hiệu lực 15 phút để tải file.
5. Lambda gửi email qua Amazon SES tới địa chỉ email của chủ vé, đính kèm presigned URL vừa tạo.

Một chi tiết đáng chú ý: nếu vé thiếu một số thông tin snapshot (họ tên, tên sự kiện, địa điểm), Lambda tự dùng giá trị mặc định ("Participant", "AWS Event Management Workshop", "HUTECH") thay vì để trống hoặc lỗi — giúp luồng cấp chứng nhận không bị gãy vì thiếu dữ liệu phụ.

---

## Bước 1: Chuẩn bị dữ liệu test

Dùng ticketId của một vé đã đăng ký thành công ở mục 5.7:

```powershell
$baseUrl = "https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev"
$ticketId = "<TicketId lấy từ mục 5.7>"
```

Lưu ý: không cần header Authorization cho các request ở mục này, vì cả 3 route đều không yêu cầu JWT.

---

## Bước 2: Test check-in thành công

```powershell
$body = @{ ticketId = $ticketId; method = "QR" } | ConvertTo-Json
Invoke-RestMethod -Uri "$baseUrl/tickets/checkin" -Method Post -Body $body -ContentType "application/json"
```

Response mong đợi:

```json
{
  "success": true,
  "message": "Check-in thành công.",
  "ticketId": "<TicketId của vé — lấy từ response đăng ký ở mục 5.7>",
  "eventId": "<EventId của sự kiện tương ứng>",
  "eventTitle": "<Title của sự kiện, snapshot tại thời điểm đăng ký>",
  "userId": "<UserId của tài khoản đã đăng ký vé — trùng với sub trong JWT>",
  "userEmail": "<Email của tài khoản đã đăng ký, dùng để nhận email chứng nhận ở Bước 6>",
  "userFullName": "<Họ tên đã đồng bộ từ hồ sơ người dùng lúc đăng ký>",
  "checkInAt": "<Thời điểm check-in thực tế, định dạng UTC ISO 8601, ví dụ 2026-07-05T07:20:11Z>",
  "checkInMethod": "QR",
  "status": "CHECKED_IN"
}
```

Các field success, message, checkInMethod, status luôn cố định như trên với mọi tài khoản test. Các field còn lại (ticketId, eventId, eventTitle, userId, userEmail, userFullName, checkInAt) sẽ khác nhau tùy vé và tài khoản đang test — khi chụp ảnh minh họa thật, đây chính là những giá trị cần đối chiếu khớp với vé đã tạo ở mục 5.7.

![Checkin](/images/5-Workshop/5.8-Checkin-certificate-flow/Checkin.jpg)

---

## Bước 3: Test chặn check-in trùng

Gọi lại đúng request ở Bước 2 lần thứ hai:

```powershell
try {
    Invoke-RestMethod -Uri "$baseUrl/tickets/checkin" -Method Post -Body $body -ContentType "application/json"
} catch {
    $_.Exception.Response.StatusCode.value__
    $_.ErrorDetails.Message
}
```

Kết quả mong đợi: lỗi 409 Conflict với message "Vé này đã được check-in trước đó. Không cần check-in lại."

---

## Bước 4: Test check-in với ticketId không hợp lệ

```powershell
$badBody = @{ ticketId = "invalid-ticket-id"; method = "QR" } | ConvertTo-Json
try {
    Invoke-RestMethod -Uri "$baseUrl/tickets/checkin" -Method Post -Body $badBody -ContentType "application/json"
} catch {
    $_.Exception.Response.StatusCode.value__
}
```

Kết quả mong đợi: lỗi 404 Not Found với message "Vé không tồn tại."

---

## Bước 5: Kiểm tra dữ liệu Attendance trong DynamoDB

Mở DynamoDB → Tables → bảng Attendance → Explore table items, kiểm tra bản ghi vừa tạo có đầy đủ thời điểm check-in, và phương thức check-in là QR.

Mở tiếp DynamoDB → Tables → bảng Tickets, kiểm tra vé tương ứng đã chuyển sang trạng thái Checked_in.

---

## Bước 6: Test tạo và gửi chứng nhận

```powershell
Invoke-RestMethod -Uri "$baseUrl/certificates-v2/$ticketId" -Method Get
```

Response mong đợi:

```json
{
  "success": true,
  "message": "Tạo certificate thành công.",
  "ticketId": "8f2c1a4e-7b3d-4e91-9c5a-1d6f0b8a2e3c",
  "certificateId": "CERT-8F2C1A4E",
  "downloadUrl": "https://<bucket>.s3.ap-southeast-1.amazonaws.com/certificates/8f2c1a4e-....pdf?X-Amz-..."
}
```

![certificates](/images/5-Workshop/5.8-Checkin-certificate-flow/certificates.jpg)

---

## Bước 7: Test chặn tạo chứng nhận khi chưa check-in

Dùng một ticketId khác (đã đăng ký nhưng chưa check-in):

```powershell
Invoke-RestMethod -Uri "$baseUrl/certificates-v2/<ticketId-chua-checkin>" -Method Get
```

Kết quả mong đợi: lỗi "Bạn chưa check-in nên chưa thể tải chứng nhận."

---

## Bước 8: Kiểm tra file PDF trong S3 và email đã gửi

Mở Amazon S3 → Buckets → bucket chứng chỉ của dự án, kiểm tra file certificates/ticketId.pdf đã tồn tại.

Mở CloudWatch → Log groups → log group của AttendanceCertificateFunction, tìm log stream tại thời điểm test, kiểm tra các dòng log ghi nhận email gửi đi và trạng thái gọi SES thành công.

![S3](/images/5-Workshop/5.8-Checkin-certificate-flow/S3bucketcertificates.jpg)
![ClouldWatch](/images/5-Workshop/5.8-Checkin-certificate-flow/CloudWatch.jpg)
![Mail](/images/5-Workshop/5.8-Checkin-certificate-flow/Mail.jpg)

---

## Kết quả mong đợi

Sau khi hoàn thành phần này, người thực hiện có thể:

- Check-in vé thành công bằng TicketId và xác nhận dữ liệu Attendance được ghi đúng.
- Xác nhận cơ chế chặn check-in trùng (409 Conflict) và check-in vé không tồn tại (404 Not Found).
- Tạo chứng nhận PDF thành công cho vé đã check-in, tải được file qua presigned URL.
- Xác nhận cơ chế chặn tạo chứng nhận cho vé chưa check-in.
- Kiểm tra được file PDF thật trong S3 và log xác nhận email đã gửi qua SES.
- Hiểu cơ chế fallback dữ liệu mặc định khi vé thiếu thông tin snapshot phụ.
- Dữ liệu Attendance đã sẵn sàng cho dashboard phân tích ở mục 5.10.