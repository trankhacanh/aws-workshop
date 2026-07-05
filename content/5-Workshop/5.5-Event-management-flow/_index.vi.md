---
title: "Quản lý sự kiện & danh mục (Event Management)"
date: 2026-06-29
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

---

Phần này hướng dẫn demo luồng Admin quản lý sự kiện và danh mục — bước tạo dữ liệu nền bắt buộc phải có trước khi User có thể đăng ký vé ở các mục sau.

Toàn bộ nghiệp vụ này do EventFunction (EventLambda) xử lý, đọc/ghi vào hai bảng EventManagementEvents và EventManagementCategories, cùng bucket S3 EventManagementEventBannersBucket để lưu banner.

---

## Luồng quản lý sự kiện

1. Admin đăng nhập (đã có JWT hợp lệ, thuộc group Admins — xem lại mục 5.4).
2. Admin gọi GET /admin/events/banner-upload-url để lấy presigned URL, sau đó upload trực tiếp file banner lên S3 từ trình duyệt (không qua Lambda).
3. Admin gọi POST /admin/events để tạo sự kiện mới, kèm BannerUrl vừa upload, tiêu đề, mô tả, địa điểm, thời gian bắt đầu/kết thúc, danh mục, diễn giả, yêu cầu tiên quyết, công cụ cần chuẩn bị và số lượng chỗ tối đa (MaxSlots).
4. Admin có thể xem danh sách sự kiện quản trị (GET /admin/events), xem chi tiết (GET /admin/events/{eventId}) hoặc cập nhật (PUT /admin/events/{eventId}).
5. Admin bật/tắt hiển thị sự kiện qua PATCH /admin/events/{eventId}/visibility — chuyển cờ IsVisible mà không cần xóa sự kiện.
6. User (không cần đăng nhập) xem danh sách sự kiện công khai qua GET /events và chi tiết qua GET /events/{eventId} — cả hai route này được khai báo Authorizer: NONE.

Lưu ý: hệ thống **không có route xóa sự kiện** (DeleteEvent) — chỉ có thể ẩn sự kiện bằng cờ IsVisible, giữ lại toàn bộ lịch sử vé/attendance liên quan thay vì xóa cứng dữ liệu.

---

## Cơ chế upload banner qua Presigned URL

Thay vì cho phép Frontend upload file trực tiếp qua Lambda (tốn thời gian xử lý và giới hạn payload API Gateway), hệ thống dùng cơ chế **presigned URL**:

1. Frontend gọi GET /admin/events/banner-upload-url?contentType=<mime-type>.
2. EventService.GenerateBannerUploadUrlAsync sinh một fileKey duy nhất, xác định phần mở rộng file (jpg, webp, hoặc mặc định png) dựa theo contentType, rồi tạo presigned URL có hiệu lực **15 phút** để PUT trực tiếp lên S3.
3. Frontend dùng URL này để upload file thẳng lên S3, không đi qua backend.
4. Sau khi upload xong, Frontend gửi BannerUrl (tính từ fileKey) kèm trong request POST /admin/events hoặc PUT /admin/events/{eventId}.

Khi trả banner về cho client hiển thị, hệ thống tiếp tục sinh một presigned URL khác có hiệu lực **24 giờ** để đọc ảnh (ResolveBannerAccessUrl), thay vì để bucket public — nhất quán với cách CloudFront/OAC xử lý ở phần Frontend.

---

## Cơ chế tính trạng thái sự kiện tự động

Trạng thái sự kiện (Status) không chỉ do Admin đặt thủ công mà còn được tính lại tự động qua EventStatusHelper.ApplyEffectiveStatus, dựa trên so sánh EndTime với thời gian hiện tại — một sự kiện đã quá EndTime sẽ tự động được xem là đã kết thúc (IsEnded) khi trả về cho client, dù giá trị Status lưu trong DynamoDB chưa được cập nhật.

Vì cơ chế này, SetEventVisibilityAsync sẽ **từ chối** yêu cầu bật hiển thị (IsVisible = true) cho một sự kiện đã kết thúc, tránh trường hợp Admin vô tình hiển thị lại sự kiện đã qua.

---

## Luồng quản lý danh mục

Song song với sự kiện, Admin quản lý danh mục qua các route riêng:

- GET /categories — công khai, không cần đăng nhập (Authorizer: NONE).
- GET /admin/categories, POST /admin/categories — xem danh sách và tạo danh mục mới.
- PUT /admin/categories/{categoryId} — cập nhật danh mục.
- DELETE /admin/categories/{categoryId} — xóa danh mục (khác với sự kiện, danh mục **có** route xóa thật).

---

## Bước 1: Test luồng tạo sự kiện qua Admin UI

1. Đăng nhập bằng tài khoản thuộc group Admins.
2. Vào trang quản trị sự kiện, tạo một danh mục mới trước (nếu chưa có).
3. Tạo sự kiện mới: nhập tiêu đề, mô tả, địa điểm, thời gian, chọn danh mục, upload banner, nhập số lượng chỗ tối đa.
4. Lưu sự kiện, kiểm tra sự kiện xuất hiện trong danh sách quản trị.

![Event](/images/5-Workshop/5.5-Event-management-flow/Event.jpg)


---

## Bước 2: Kiểm tra dữ liệu trong DynamoDB

`Mở DynamoDB > Tables > EventManagementEvents > Explore table items`, kiểm tra item vừa tạo có đầy đủ các field: EventId, Title, Status, BannerUrl, MaxSlots, RegisteredCount (khởi tạo bằng 0), IsVisible.

![Dynamo](/images/5-Workshop/5.5-Event-management-flow/Dynamo.jpg)
---

## Bước 3: Test bật/tắt hiển thị sự kiện

1. Từ trang quản trị, bấm "Ẩn sự kiện" cho sự kiện vừa tạo.
2. Kiểm tra sự kiện không còn xuất hiện ở trang danh sách công khai (GET /events) nhưng vẫn còn trong danh sách quản trị (GET /admin/events).
3. Bật lại hiển thị, xác nhận sự kiện xuất hiện trở lại ở trang công khai.

| ![Ảnh 1](/images/5-Workshop/5.5-Event-management-flow/EventUnHiden.jpg) | ![Ảnh 2](/images/5-Workshop/5.5-Event-management-flow/EventHiden.jpg) |
|---|---|

---

## Bước 4: Test route công khai không cần đăng nhập

1. Mở trình duyệt ở chế độ ẩn danh (chưa đăng nhập).
2. Truy cập trang danh sách sự kiện, xác nhận vẫn xem được danh sách sự kiện đang IsVisible = true.
3. Mở DevTools Network tab, xác nhận request GET /events **không có** header Authorization nhưng vẫn trả về 200 OK.

---

## Kết quả mong đợi

Sau khi hoàn thành phần này, người thực hiện có thể:

- Tạo, xem danh sách và cập nhật sự kiện với đầy đủ thông tin qua Admin API.
- Hiểu và test được cơ chế upload banner qua presigned URL, không đi qua Lambda.
- Hiểu cơ chế tự động tính trạng thái sự kiện (ApplyEffectiveStatus) dựa trên EndTime.
- Test được luồng bật/tắt hiển thị sự kiện (IsVisible) và xác nhận đúng hành vi từ chối hiển thị lại sự kiện đã kết thúc.
- Quản lý danh mục sự kiện (tạo, sửa, xóa) qua các route admin riêng.
- Xác nhận các route đọc công khai (GET /events, GET /categories) hoạt động đúng mà không cần JWT.
- Dữ liệu sự kiện đã sẵn sàng để dùng cho luồng đăng ký vé ở mục 5.7.