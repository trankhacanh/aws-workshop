---
title: "Luồng đăng ký vé"
date: 2026-06-29
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

---

Phần này demo luồng đăng ký vé, cơ chế chống overbooking bằng điều kiện DynamoDB, và việc publish sự kiện TicketRegistered lên EventBridge để trigger thông báo bất đồng bộ (sẽ xử lý chi tiết ở mục 5.9).

Nghiệp vụ này do TicketFunction xử lý, đọc/ghi vào bảng Tickets, đọc/tăng biến đếm ở bảng Events, và đọc thông tin hồ sơ từ bảng Users để đồng bộ vào vé.

---

## Luồng đăng ký vé

1. User gửi request đăng ký vé (kèm eventId) với JWT hợp lệ.
2. TicketService kiểm tra User đã đăng ký sự kiện này chưa (query theo UserId qua GSI, so khớp EventId) — nếu đã đăng ký, trả lỗi nghiệp vụ.
3. Hệ thống thực hiện thao tác tăng RegisteredCount của sự kiện bằng một update có điều kiện:

```csharp
UpdateExpression = "SET #regCount = #regCount + :inc",
ConditionExpression = "#regCount < #maxSlots"
```

4. Nếu điều kiện không thỏa (đã đủ MaxSlots), DynamoDB tự ném lỗi ConditionalCheckFailedException — đây là cơ chế atomic chống race condition khi nhiều user đăng ký cùng lúc, không cần lock thủ công ở tầng ứng dụng.
5. Nếu qua được điều kiện, hệ thống tạo vé mới với trạng thái Confirmed, snapshot lại các thông tin sự kiện (tên sự kiện, thời gian bắt đầu, địa điểm, danh mục) và thông tin User (email, họ tên) ngay tại thời điểm đăng ký — để vé vẫn hiển thị đúng thông tin dù sau này sự kiện hoặc hồ sơ user có thay đổi.
6. Sau khi tạo vé thành công, Lambda phát một sự kiện lên EventBridge (source là eventmanagement.ticket, detail-type là TicketRegistered) để trigger NotificationFunction gửi email xác nhận bất đồng bộ. Thao tác này được bọc trong try/catch riêng — nếu việc phát sự kiện thất bại, chỉ ghi log cảnh báo, không làm rollback hay fail request đăng ký vé.
7. User xem lại vé của mình qua route lấy danh sách vé cá nhân.

---

## Hành vi giao diện theo trạng thái đăng ký

Ở trang chi tiết sự kiện, nút hành động thay đổi tùy theo hai field isFull và trạng thái đã đăng ký (isAlreadyRegistered) trả về từ API — không có trạng thái "chờ" trung gian nào khác:

| Trạng thái | Hiển thị trên giao diện |
|---|---|
| Chưa đăng ký, còn chỗ (isFull: false) | Nút "Đăng ký sự kiện" |
| Đã đăng ký thành công (isAlreadyRegistered: true) | Nút chuyển thành "Xem vé của tôi", không cho đăng ký lại |
| Sự kiện đã đủ chỗ (isFull: true) | Hiển thị thông báo "Đã hết slot đăng ký", ẩn nút đăng ký |
| Sự kiện đã kết thúc / đã hủy | Hiển thị thông báo trạng thái tương ứng, ẩn nút đăng ký |

Sau khi gọi thành công request đăng ký vé, Frontend load lại dữ liệu sự kiện để cập nhật ngay số lượng đã đăng ký/tình trạng đầy chỗ mới nhất, đảm bảo nút chuyển trạng thái tức thời mà không cần tải lại trang.

---

## Bước 1: Chuẩn bị token và biến môi trường test

```powershell
$token = "<dán ID Token>"
$baseUrl = "https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev"
$headers = @{ Authorization = "Bearer $token" }
$eventId = "<EventId lấy từ mục 5.5>"
```

---

## Bước 2: Test đăng ký vé thành công

```powershell
Invoke-RestMethod -Uri "$baseUrl/events/$eventId/register" -Method Post -Headers $headers
```

Response thực tế trả về:

```json
{
  "ticketId": "8f2c1a4e-7b3d-4e91-9c5a-1d6f0b8a2e3c",
  "eventId": "4573b866-0a75-4453-9346-655c416fd97e",
  "userId": "a1b2c3d4-5678-90ab-cdef-1234567890ab",
  "eventTitle": "AWS Serverless Event Management Workshop 2026",
  "eventStartTime": "2026-07-05T06:03:00.000Z",
  "eventLocation": "Phòng Lab Cloud – Tầng 4, Khu A, Trường Đại học Công nghệ Thông tin",
  "eventCategory": "AI PROMT",
  "userEmail": "user.test@example.com",
  "userFullName": "Nguyen Van Test",
  "status": "CONFIRMED",
  "createdAt": "2026-07-05T06:10:22.000Z"
}
```

<!-- > 📌 **Gợi ý hình ảnh:** Chụp song song 2 ảnh giao diện — trước khi đăng ký (nút "Đăng ký sự kiện") và sau khi đăng ký (nút đã chuyển thành "Xem vé của tôi") — minh họa trực tiếp hành vi UI vừa mô tả ở trên. -->

---

## Bước 3: Test chặn đăng ký trùng

Gọi lại đúng request ở Bước 2 lần thứ hai:

```powershell
try {
    Invoke-RestMethod -Uri "$baseUrl/events/$eventId/register" -Method Post -Headers $headers
} catch {
    $_.Exception.Response.StatusCode.value__
    $_.ErrorDetails.Message
}
```

Kết quả mong đợi: lỗi nghiệp vụ dạng "Bạn đã đăng ký vé cho sự kiện này rồi!". Trên giao diện, tình huống này không xảy ra được vì nút đã chuyển thành "Xem vé của tôi" ngay từ lần đăng ký đầu — request thử nghiệm lần 2 này chỉ có thể tái hiện được qua gọi API trực tiếp.

---

## Bước 4: Test cơ chế chống overbooking

Nếu sự kiện test có số chỗ tối đa nhỏ (ví dụ = 1), dùng nhiều tài khoản/token khác nhau để đăng ký liên tiếp cho tới khi vượt quá số chỗ:

```powershell
$headers2 = @{ Authorization = "Bearer <token của user khác>" }
try {
    Invoke-RestMethod -Uri "$baseUrl/events/$eventId/register" -Method Post -Headers $headers2
} catch {
    $_.ErrorDetails.Message
}
```

Kết quả mong đợi khi đã hết chỗ: lỗi "Sự kiện này đã đạt giới hạn số lượng chỗ ngồi (Hết vé)!" — xuất phát từ lỗi ConditionalCheckFailedException của DynamoDB. Trên giao diện, tại thời điểm này tình trạng đầy chỗ chuyển thành true, nút đăng ký được thay bằng thông báo "Đã hết slot đăng ký" cho các user chưa đăng ký còn lại.

<!-- > 📌 **Gợi ý hình ảnh:** Chụp giao diện sự kiện lúc này hiển thị "Đã hết slot đăng ký", kèm ảnh CloudWatch Logs thể hiện dòng log lỗi ConditionalCheckFailedException tương ứng. -->

---

## Bước 5: Kiểm tra dữ liệu trong DynamoDB

Mở DynamoDB → Tables → bảng Tickets → Explore table items, kiểm tra vé vừa tạo có đầy đủ snapshot dữ liệu (tên sự kiện, họ tên người dùng...) và trạng thái là Confirmed.

Tiếp tục mở DynamoDB → Tables → bảng Events, kiểm tra số lượng đã đăng ký của sự kiện đã tăng đúng số lượng vé đã đăng ký thành công, và đã chạm số chỗ tối đa khi giao diện hiển thị "Đã hết slot đăng ký".

![FullFe](/images/5-Workshop/5.7-Registration-ticketing-flow/FullSlotFe.jpg)

![FullAWS](/images/5-Workshop/5.7-Registration-ticketing-flow/FullSlot.jpg)

---

## Bước 6: Kiểm tra sự kiện đã publish lên EventBridge

Mở Amazon EventBridge → Rules, tìm rule lắng nghe sự kiện đăng ký vé (nằm trên default event bus). Vào tab Monitoring của rule, kiểm tra biểu đồ Invocations tăng lên đúng số lần đăng ký vé vừa test.

![Event](/images/5-Workshop/5.7-Registration-ticketing-flow/Rule.jpg)

---

## Bước 7: Test xem vé cá nhân

```powershell
Invoke-RestMethod -Uri "$baseUrl/my-tickets" -Method Get -Headers $headers
```

Xác nhận response trả về danh sách vé, bao gồm vé vừa đăng ký ở Bước 2 — đây cũng chính là dữ liệu hiển thị khi bấm nút "Xem vé của tôi" trên giao diện.

![Tikcet](/images/5-Workshop/5.7-Registration-ticketing-flow/MyTicket.jpg)

---

## Kết quả mong đợi

Sau khi hoàn thành phần này, người thực hiện có thể:

- Đăng ký vé thành công và nhận đúng dữ liệu snapshot trong response.
- Xác nhận nút giao diện chuyển đúng trạng thái theo tình trạng đầy chỗ và trạng thái đã đăng ký (Đăng ký → Xem vé, hoặc → Đã hết slot đăng ký).
- Xác nhận cơ chế chặn đăng ký trùng cho cùng một sự kiện.
- Xác nhận cơ chế chống overbooking hoạt động đúng bằng lỗi ConditionalCheckFailedException.
- Đối chiếu dữ liệu giữa API response và item thật trong hai bảng DynamoDB (Tickets, Events).
- Xác nhận sự kiện đăng ký vé được publish lên EventBridge thành công qua biểu đồ Monitoring của Rule.
- Xem lại danh sách vé cá nhân qua route lấy vé cá nhân.
- Dữ liệu vé đã sẵn sàng cho luồng check-in và cấp chứng nhận ở mục 5.8.