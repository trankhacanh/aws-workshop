---
title: "Deploy Frontend qua CloudFront"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

---

Phần này hướng dẫn build ứng dụng Frontend (React + Vite), đưa lên bucket S3 ở chế độ **private** (không bật static website hosting công khai), rồi phân phối qua **Amazon CloudFront** với **Origin Access Control (OAC)** — chỉ CloudFront mới được phép đọc trực tiếp từ bucket, người dùng không thể truy cập S3 trực tiếp.

> **Tình trạng thực tế của hệ thống:** Distribution đang dùng domain mặc định do CloudFront cấp (`dfu4o0ltvpe4o.cloudfront.net`), **chưa cấu hình Custom Domain riêng** (ví dụ `app.tenmien.com`) và **chưa có ACM certificate**. Phần Custom Domain được nêu ở cuối bài (Bước 7) là hướng mở rộng cho sau này, không thuộc phạm vi đã triển khai.

Do đây là phần triển khai hạ tầng (không phải gọi API nghiệp vụ), demo sẽ dùng **AWS CLI** kết hợp **AWS Console** thay vì gọi API Gateway.

---

## Luồng deploy Frontend

1. Build project React ra thư mục tĩnh (`dist/`).
2. Tạo bucket S3 riêng cho Frontend, bật Block Public Access, không bật Static website hosting (vì truy cập sẽ đi qua CloudFront, không đi thẳng vào S3).
3. Tạo CloudFront Distribution, gắn Origin là bucket S3 ở trên, cấu hình **Origin Access Control** để CloudFront có quyền `s3:GetObject` còn người dùng ngoài thì không.
4. Cập nhật Bucket Policy để chỉ cho phép service principal `cloudfront.amazonaws.com` (kèm điều kiện `AWS:SourceArn` là ARN của distribution) được đọc object.
5. Cấu hình **Default Root Object** là `index.html` và **Custom Error Response** để xử lý client-side routing của React Router (trả `index.html` với status `200` khi gặp lỗi `403`/`404`).
6. Upload build lên S3, tạo **Invalidation** trên CloudFront mỗi khi deploy lại để xóa cache CDN.
7. Truy cập ứng dụng qua domain CloudFront mặc định (`dfu4o0ltvpe4o.cloudfront.net`) để kiểm thử toàn bộ luồng, xác nhận Frontend gọi đúng API Gateway/Cognito đã cấu hình ở các mục trước.

---

## Bước 1: Build Frontend

Trong thư mục `FE/`, cấu hình biến môi trường trỏ về API Gateway và Cognito đã tạo (file `.env.production`):

```env
VITE_API_BASE_URL=https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev
VITE_COGNITO_USER_POOL_ID=<user-pool-id>
VITE_COGNITO_CLIENT_ID=<app-client-id>
VITE_COGNITO_REGION=ap-southeast-1
```

Build production:

```powershell
npm install
npm run build
```

Kết quả: thư mục `dist/` chứa `index.html`, thư mục `assets/` (JS/CSS đã bundle) — đây là nội dung sẽ upload lên S3.

---

## Bước 2: Tạo S3 bucket cho Frontend (private)

```powershell
aws s3api create-bucket `
  --bucket event-management-frontend-<suffix> `
  --region ap-southeast-1 `
  --create-bucket-configuration LocationConstraint=ap-southeast-1

aws s3api put-public-access-block `
  --bucket event-management-frontend-<suffix> `
  --public-access-block-configuration BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
```

Lưu ý: bucket này **không** bật Static Website Hosting — CloudFront sẽ truy cập bucket qua REST endpoint (`bucket.s3.amazonaws.com`), không qua website endpoint.

<!-- > 📌 **Gợi ý hình ảnh:** Chụp S3 Console cho thấy bucket Frontend với mục "Block all public access" đang bật (On). -->

---

## Bước 3: Tạo CloudFront Distribution với Origin Access Control

Trên **CloudFront Console**:

1. **Create Distribution** → Origin domain: chọn bucket S3 vừa tạo (chọn dạng REST endpoint, không chọn website endpoint).
2. Ở mục **Origin access**, chọn **Origin access control settings (recommended)** → **Create control setting** (giữ mặc định "Sign requests").
3. **Default root object**: `index.html`.
4. **Viewer protocol policy**: Redirect HTTP to HTTPS.
5. **Alternate domain name (CNAME)**: để trống — hệ thống hiện dùng thẳng domain mặc định do CloudFront cấp, chưa gắn Custom Domain (xem Bước 7 để biết hướng mở rộng sau này).
6. Sau khi tạo xong, CloudFront sẽ hiện cảnh báo yêu cầu cập nhật Bucket Policy — bấm **Copy policy**.

Cập nhật Bucket Policy (dán policy CloudFront cung cấp, có dạng sau) qua S3 Console → Permissions → Bucket Policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipal",
      "Effect": "Allow",
      "Principal": { "Service": "cloudfront.amazonaws.com" },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::event-management-frontend-<suffix>/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::<account-id>:distribution/<distribution-id>"
        }
      }
    }
  ]
}
```

<!-- > 📌 **Gợi ý hình ảnh:** Chụp màn hình cấu hình Origin access control lúc tạo Distribution, và Bucket Policy sau khi dán. -->

---

## Bước 4: Cấu hình Custom Error Response cho React Router

Vì React Router xử lý routing phía client, khi user F5 ở một route con (ví dụ `/events/123`), S3 sẽ trả `403`/`404` do không tồn tại object đó. Cần cấu hình CloudFront trả về `index.html` cho các mã lỗi này:

Trong Distribution → tab **Error pages** → **Create custom error response**:

| HTTP Error Code | Customize Error Response | Response Page Path | HTTP Response Code |
|---|---|---|---|
| 403 | Yes | `/index.html` | 200 |
| 404 | Yes | `/index.html` | 200 |

<!-- > 📌 **Gợi ý hình ảnh:** Chụp bảng cấu hình Custom Error Response trong CloudFront Console. -->

---

## Bước 5: Upload build và tạo Invalidation

```powershell
aws s3 sync ./dist s3://event-management-frontend-<suffix> --delete

aws cloudfront create-invalidation `
  --distribution-id <distribution-id> `
  --paths "/*"
```

- `--delete` đảm bảo object cũ không còn dùng bị xóa khỏi bucket.
- `create-invalidation` bắt buộc mỗi lần deploy lại, nếu không CloudFront vẫn phục vụ bản cache cũ do TTL của cache behavior.

<!-- > 📌 **Gợi ý hình ảnh:** Chụp terminal sau khi chạy `aws s3 sync`, và tab Invalidations trên CloudFront Console hiện trạng thái "Completed". -->

---

## Bước 6: Kiểm thử truy cập qua CloudFront

Mở domain CloudFront mặc định trên trình duyệt: https://dfu4o0ltvpe4o.cloudfront.net
1. Xác nhận trang chủ tải được, gọi đúng `GET /events` (route public đã cấu hình ở mục 5.5).
2. Đăng nhập thử qua Cognito (mục 5.4), xác nhận điều hướng và gọi các route cần JWT (Profile, Registration...) hoạt động đúng.
3. Truy cập trực tiếp một route con (ví dụ `/my-tickets`) rồi F5 lại — xác nhận không bị lỗi 403/404 trắng trang, nhờ Custom Error Response ở Bước 4.
4. Thử truy cập trực tiếp URL S3 (`https://event-management-frontend-<suffix>.s3.amazonaws.com/index.html`) để xác nhận bị từ chối (`AccessDenied`) — chứng minh bucket không public, chỉ CloudFront mới đọc được.

<!-- > 📌 **Gợi ý hình ảnh:** Chụp ứng dụng chạy trên domain `dfu4o0ltvpe4o.cloudfront.net`, và chụp lỗi `AccessDenied` khi truy cập thẳng URL S3. -->

---

## Bước 7 (Hướng mở rộng — chưa triển khai): Gắn Custom Domain riêng

Hiện tại hệ thống **chưa cấu hình domain riêng**, chỉ dùng domain mặc định của CloudFront. Nếu sau này cần dùng domain riêng (ví dụ `app.tenmien.com`), cần bổ sung:

1. **Tạo ACM Certificate ở region `us-east-1`** — bắt buộc, vì CloudFront chỉ chấp nhận chứng chỉ ACM nằm ở `us-east-1` bất kể Distribution/origin đặt ở region nào khác. Xác thực bằng DNS validation (thêm CNAME record vào domain registrar/Route 53).
2. Trong CloudFront Distribution, cập nhật:
   - **Alternate domain names (CNAMEs)**: thêm `app.tenmien.com`.
   - **Custom SSL certificate**: chọn ACM certificate vừa tạo ở bước 1 (mục `ViewerCertificate` trong CloudFormation nếu quản lý bằng IaC).
3. Tạo bản ghi DNS trỏ `app.tenmien.com` về domain CloudFront (dùng **Alias record** nếu domain quản lý trong Route 53, hoặc **CNAME** nếu dùng DNS provider khác).
4. Kiểm thử lại toàn bộ luồng ở Bước 6 nhưng thay domain mặc định bằng domain riêng.

> Đây là hạng mục khuyến nghị bổ sung, chưa nằm trong phạm vi đã triển khai của workshop hiện tại.

---

## Kết quả mong đợi

Sau khi hoàn thành phần này, người thực hiện có thể:

- Build và deploy được bản production của Frontend lên S3.
- Xác nhận bucket Frontend hoàn toàn private, không thể truy cập trực tiếp từ bên ngoài.
- Xác nhận CloudFront là điểm truy cập duy nhất, có OAC để lấy dữ liệu từ S3.
- Xác nhận cơ chế xử lý lỗi 403/404 trả về đúng `index.html`, phù hợp với client-side routing của React.
- Nắm được quy trình cập nhật deploy: build lại → `s3 sync` → `create-invalidation`.
- Nắm được hệ thống hiện đang dùng domain mặc định CloudFront, và biết rõ các bước cần làm nếu sau này muốn gắn domain riêng (ACM ở `us-east-1` + Aliases + ViewerCertificate).