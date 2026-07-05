---
title: "Dashboard & phân tích hiệu quả sự kiện (Analytics)"
date: 2026-06-29
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

---

Phần này hướng dẫn xem dashboard tổng hợp và số liệu phân tích theo từng sự kiện, tổng hợp dữ liệu từ 4 bảng: Events, Tickets, Attendance, Notification Log, cùng với bucket S3 chứa chứng chỉ.

Nghiệp vụ này do AnalyticsFunction xử lý, chỉ có quyền đọc trên toàn bộ tài nguyên liên quan — không có quyền ghi, đúng với vai trò một module thuần tổng hợp/báo cáo.

---

## Luồng dashboard tổng quan

Route lấy dashboard tổng quan trả về các chỉ số tổng hợp trên toàn hệ thống:

| Chỉ số | Nguồn tính toán |
|---|---|
| Tổng số sự kiện | Đếm toàn bộ item trong bảng Events |
| Vé đã phát hành | Đếm toàn bộ item trong bảng Tickets |
| Đã xác nhận | Số vé có trạng thái khác Cancelled |
| Người tham gia | Đếm toàn bộ item trong bảng Attendance |
| Tỷ lệ tham dự trung bình | Tính từ tỷ lệ check-in trên số vé đã xác nhận |
| Chứng chỉ đã cấp | Đếm số file thật trong S3 với tiền tố certificates/ — đếm file PDF thực tế, không dựa vào cờ trạng thái trong DynamoDB |
| Email đã gửi / lỗi | Đếm theo trạng thái Sent / Failed trong bảng Notification Log |

Điểm đáng chú ý: **"Chứng chỉ đã cấp" được đếm bằng cách liệt kê trực tiếp object trong S3**, thay vì đếm qua một cờ trạng thái trong DynamoDB — đảm bảo con số phản ánh đúng số file PDF thực sự tồn tại, tránh sai lệch nếu có lỗi ghi log ở bước tạo chứng nhận.

---

## Luồng phân tích theo từng sự kiện

Route phân tích theo sự kiện trả về số liệu chi tiết cho một sự kiện cụ thể (truyền vào eventId):

| Chỉ số | Nguồn tính toán |
|---|---|
| Tổng đăng ký | Đếm vé theo EventId |
| Đã xác nhận | Số vé của sự kiện có trạng thái khác Cancelled |
| Đã tham dự | Query trực tiếp bảng Attendance theo EventId (Partition Key — dùng Query thay vì Scan để tối ưu hiệu năng) |
| Tỷ lệ tham dự | (Số check-in / Số đã xác nhận) × 100, làm tròn 1 chữ số thập phân |

---

## Bước 1: Test lấy dashboard tổng quan

```powershell
Invoke-RestMethod -Uri "$baseUrl/admin/analytics/dashboard" -Method Get -Headers $headers
```

Response mong đợi:

```json
{
  "totalEvents": "<Tổng số sự kiện đã tạo>",
  "totalRegistrations": "<Tổng số vé đã phát hành>",
  "confirmedRegistrations": "<Số vé đã xác nhận>",
  "totalCheckIns": "<Tổng số lượt check-in>",
  "averageAttendanceRate": "<Tỷ lệ tham dự trung bình, %>",
  "emailSentCount": "<Số email gửi thành công>",
  "emailFailedCount": "<Số email gửi thất bại>",
  "certificatesIssuedCount": "<Số file PDF chứng chỉ thật trong S3>"
}
```

![Dashboard](/images/5-Workshop/5.10-Analytics-dashboard/Dashboard.jpg)

---

## Bước 2: Test phân tích theo từng sự kiện

```powershell
$eventId = "<EventId muốn xem chi tiết>"
Invoke-RestMethod -Uri "$baseUrl/admin/analytics/events/$eventId" -Method Get -Headers $headers
```

Response mong đợi:

```json
{
  "eventId": "<EventId đã truyền vào>",
  "eventTitle": "<Tiêu đề sự kiện tương ứng>",
  "totalRegistrations": "<Tổng số vé của sự kiện>",
  "confirmedCount": "<Số vé đã xác nhận>",
  "checkInCount": "<Số lượt check-in thực tế>",
  "attendanceRate": "<Tỷ lệ tham dự, %>"
}
```

![DashboardEvent](/images/5-Workshop/5.10-Analytics-dashboard/DashboardEvent.jpg)

---

## Bước 3: Đối chiếu số liệu với dữ liệu thô

Để xác nhận số liệu tổng hợp là chính xác, đối chiếu chéo với dữ liệu đã kiểm tra ở các mục trước:

1. Số "Vé đã phát hành" ở Bước 1 nên bằng đúng số item trong bảng Tickets trên DynamoDB (đã xem ở mục 5.7).
2. Số "Chứng chỉ đã cấp" nên bằng đúng số file trong bucket S3 chứa chứng chỉ, thư mục certificates/ (đã xem ở mục 5.8).
3. Số "Email đã gửi / lỗi" nên khớp với số bản ghi trạng thái Sent / Failed trong bảng Notification Log trên DynamoDB (đã xem ở mục 5.9).

---

## Kết quả mong đợi

Sau khi hoàn thành phần này, người thực hiện có thể:

- Xem được dashboard tổng quan với các chỉ số tổng hợp toàn hệ thống.
- Xem được thống kê chi tiết theo từng sự kiện cụ thể.
- Hiểu cách "Chứng chỉ đã cấp" được đếm trực tiếp từ S3 thay vì dựa vào cờ trạng thái DynamoDB.
- Hiểu cách tính tỷ lệ tham dự dựa trên số check-in thực tế so với số vé đã xác nhận.
- Đối chiếu và xác nhận số liệu dashboard khớp đúng với dữ liệu thô trong DynamoDB và S3 đã kiểm tra ở các mục trước.
- Đây là mục tổng kết dữ liệu của toàn bộ hệ thống — hoàn thành mục này đồng nghĩa đã kiểm thử xong toàn bộ vòng đời nghiệp vụ chính từ tạo sự kiện đến phân tích hiệu quả.