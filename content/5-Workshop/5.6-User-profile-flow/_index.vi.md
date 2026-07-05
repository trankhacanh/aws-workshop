---
title: "Quản lý hồ sơ người dùng (User Profile)"
date: 2026-06-29
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

---

Phần này hướng dẫn demo luồng khởi tạo, xem, cập nhật hồ sơ người dùng và upload avatar. Nghiệp vụ này do UserProfileFunction (UserProfileLambda) xử lý, đọc/ghi vào bảng EventManagementUsers và bucket S3 EventManagementUserAvatarsBucket.

Vì đây là module có luồng dữ liệu ngắn và rõ ràng, phần này sẽ demo trực tiếp bằng **PowerShell (Invoke-RestMethod)** gọi thẳng vào API Gateway thay vì thao tác qua giao diện — cách này cho thấy rõ ràng hơn dữ liệu request/response thật ở từng bước, đồng thời không phụ thuộc vào giao diện Frontend.

---

## Luồng quản lý hồ sơ người dùng

1. Sau khi đăng nhập lần đầu (email hoặc Google), Frontend gọi POST /profile/init — Lambda kiểm tra user đã tồn tại trong EventManagementUsers chưa; nếu chưa, tạo mới hồ sơ với Role và Status mặc định; nếu đã có, chỉ cập nhật LastLoginAt và đồng bộ lại Role theo claim cognito:groups mới nhất.
2. User xem hồ sơ hiện tại qua GET /profile/me.
3. User cập nhật họ tên và avatar qua PUT /profile/me.
4. Trước khi cập nhật avatar, User gọi GET /profile/avatar-upload-url để lấy presigned URL, upload ảnh trực tiếp lên S3 (cùng cơ chế presigned URL 15 phút như banner sự kiện ở mục 5.5), sau đó gửi AvatarUrl vào PUT /profile/me.

Schema hồ sơ người dùng (UserProfileDto) gồm:

```text
UserId, Email, FullName, AvatarUrl, Role (USER | ADMIN), Status (ACTIVE | INACTIVE | BLOCKED), CreatedAt, UpdatedAt, LastLoginAt
```

Tất cả 4 route trên đều yêu cầu JWT hợp lệ qua Cognito Authorizer mặc định — không có route nào công khai trong module này, vì hồ sơ luôn gắn với danh tính người dùng đang đăng nhập.

---

## Bước 1: Chuẩn bị token để test

Lấy ID Token từ phiên đăng nhập (có thể lấy từ DevTools Network tab sau khi đăng nhập ở mục 5.4, hoặc từ fetchAuthSession()), sau đó lưu vào biến PowerShell:

```powershell
$token = "<dán ID Token vào đây>"
$baseUrl = "https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev"
$headers = @{ Authorization = "Bearer $token" }
```

---

## Bước 2: Test khởi tạo hồ sơ (InitProfile)

```powershell
Invoke-RestMethod -Uri "$baseUrl/profile/init" -Method Post -Headers $headers
```

Kết quả mong đợi (lần đầu gọi):

```json
{
  "isNewUser": true,
  "profile": {
    "userId": "...",
    "email": "...",
    "fullName": "...",
    "role": "USER",
    "status": "ACTIVE",
    "createdAt": "...",
    "lastLoginAt": "..."
  }
}
```

Gọi lại lần thứ hai, xác nhận isNewUser trả về false và lastLoginAt được cập nhật thời gian mới.

<!-- > 📌 **Gợi ý hình ảnh:** Chụp terminal PowerShell hiển thị 2 lần gọi liên tiếp, thấy rõ isNewUser: true rồi `isNewUser: false` — bằng chứng logic phân biệt user mới/cũ hoạt động đúng. -->

---

## Bước 3: Test lấy thông tin hồ sơ (GetProfile)

```powershell
Invoke-RestMethod -Uri "$baseUrl/profile/me" -Method Get -Headers $headers
```

Kiểm tra response trả về đầy đủ các field như schema đã nêu ở trên.

---

## Bước 4: Test lấy presigned URL và upload avatar

```powershell
$uploadInfo = Invoke-RestMethod -Uri "$baseUrl/profile/avatar-upload-url?contentType=image/jpeg" -Method Get -Headers $headers
$uploadInfo
```

Response mong đợi:

```json
{
  "uploadUrl": "https://<bucket>.s3.ap-southeast-1.amazonaws.com/...",
  "avatarUrl": "https://<bucket>.s3.ap-southeast-1.amazonaws.com/..."
}
```

Upload file ảnh thật trực tiếp lên uploadUrl bằng PUT (không cần token, vì presigned URL đã tự chứa quyền truy cập tạm thời):

```powershell
Invoke-WebRequest -Uri $uploadInfo.uploadUrl -Method Put -InFile ".\avatar-test.jpg" -ContentType "image/jpeg"
```

> 📌 **Gợi ý hình ảnh:** Chụp terminal hiển thị response `uploadUrl`/`avatarUrl`, và chụp S3 Console cho thấy file ảnh vừa upload xuất hiện trong bucket avatar — không cần chụp giao diện Frontend.

---

## Bước 5: Test cập nhật hồ sơ (UpdateProfile)

```powershell
$body = @{
    fullName  = "Nguyen Van Test"
    avatarUrl = $uploadInfo.avatarUrl
} | ConvertTo-Json

Invoke-RestMethod -Uri "$baseUrl/profile/me" -Method Put -Headers $headers -Body $body -ContentType "application/json"
```

Gọi lại GET /profile/me để xác nhận fullName và avatarUrl đã được cập nhật đúng.

---

## Bước 6: Kiểm tra dữ liệu trong DynamoDB

Mở DynamoDB > Tables > EventManagementUsers > Explore table items, kiểm tra item của user vừa test có đầy đủ field đã cập nhật, và Role phản ánh đúng nếu user thuộc group Admins (Role = ADMIN).

<!-- > 📌 **Gợi ý hình ảnh:** Chụp item trong DynamoDB Console — đối chiếu trực tiếp với response JSON ở Bước 5 để chứng minh dữ liệu được ghi đúng. -->

---

## Bước 7: Test trường hợp không có token (kiểm tra bảo vệ route)

```powershell
try {
    Invoke-RestMethod -Uri "$baseUrl/profile/me" -Method Get
} catch {
    $_.Exception.Response.StatusCode.value__
}
```

Kết quả mong đợi: 401 Unauthorized, xác nhận toàn bộ route của module User Profile được Cognito Authorizer bảo vệ đúng như thiết kế — khác với các route công khai ở module Event Management (mục 5.5).

---

## Kết quả mong đợi

Sau khi hoàn thành phần này, người thực hiện có thể:

- Test được luồng khởi tạo hồ sơ và phân biệt đúng user mới/cũ (isNewUser).
- Test được luồng xem và cập nhật hồ sơ người dùng qua request API trực tiếp.
- Test được cơ chế upload avatar qua presigned URL, xác nhận file xuất hiện đúng trong S3.
- Xác nhận toàn bộ route của module này đều yêu cầu JWT hợp lệ (trả về 401 khi thiếu token).
- Đối chiếu được dữ liệu trả về từ API với dữ liệu thực tế lưu trong DynamoDB.
- Dữ liệu hồ sơ (đặc biệt FullName, Email) đã sẵn sàng để đồng bộ vào vé điện tử ở luồng đăng ký vé (mục 5.7).