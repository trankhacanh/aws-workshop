---
title: "Cấu hình xác thực với Amazon Cognito"
date: 2026-06-29
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

---

Phần này giải thích cách Event Management Platform sử dụng Amazon Cognito để xác thực người dùng và phân quyền User/Admin.

Hệ thống hỗ trợ **hai phương thức đăng nhập**: đăng ký/đăng nhập trực tiếp bằng **email và mật khẩu**, và đăng nhập bằng **Google** thông qua Cognito Hosted UI (Federated Identity Provider). Cognito User Pool được backend tham chiếu tới thông qua tham số CognitoUserPoolIdParam trong template.yaml, dùng để cấu hình Cognito Authorizer bảo vệ các route API cần đăng nhập.

---

## Luồng xác thực bằng email/mật khẩu

1. User đăng ký bằng email, mật khẩu, họ tên (số điện thoại là tùy chọn) thông qua Amazon Cognito (qua thư viện aws-amplify, hàm signUp).
2. Cognito gửi mã xác nhận (confirmation code) đến email đã đăng ký.
3. User xác nhận mã bằng confirmSignUp.
4. User đăng nhập bằng signIn — Cognito trả về Access Token và ID Token.
5. Frontend gọi userProfileService.initProfile() để backend tạo/đồng bộ hồ sơ người dùng trong DynamoDB.
6. Mọi request gọi API sau đó, axiosInstance.ts tự động lấy **ID Token** hiện tại (fetchAuthSession) và đính vào header Authorization: Bearer <idToken>.
7. API Gateway dùng **Cognito Authorizer (COGNITO_USER_POOLS)** kiểm tra ID Token hợp lệ trước khi chuyển request vào Lambda tương ứng.
8. Lambda đọc claims đã được API Gateway giải mã sẵn từ request.RequestContext.Authorizer.Claims — không tự verify JWT trong code Lambda.

---

## Luồng đăng nhập bằng Google (Federated Sign-In)

Bên cạnh email/mật khẩu, hệ thống tích hợp đăng nhập bằng Google thông qua **Cognito Hosted UI**, sử dụng OAuth 2.0 Authorization Code Grant:

1. User bấm "Tiếp tục với Google" trên trang đăng nhập — Frontend gọi signInWithRedirect({ provider: 'Google' }) của Amplify.
2. Trình duyệt được chuyển hướng sang Cognito Hosted UI, rồi tiếp tục chuyển sang trang đăng nhập Google.
3. Sau khi xác thực thành công ở Google, Cognito nhận authorization code, đổi lấy token, rồi redirect ngược về Frontend tại route callback (/auth/callback — do component AuthCallback xử lý màn hình chờ).
4. Amplify Hub phát ra sự kiện signInWithRedirect — AuthContext.tsx lắng nghe sự kiện này qua Hub.listen("auth", ...), sau khi nhận thành công sẽ gọi initProfile() để đồng bộ hồ sơ cho user Google mới (trường hợp lần đầu đăng nhập bằng Google).
5. Từ thời điểm này, user Google được xử lý hoàn toàn giống user đăng ký bằng email: có ID Token, gọi API qua axiosInstance.ts như bình thường.

Cấu hình OAuth được khai báo tập trung tại main.tsx khi khởi tạo Amplify:

```ts
Amplify.configure({
  Auth: {
    Cognito: {
      userPoolId: import.meta.env.VITE_COGNITO_USER_POOL_ID,
      userPoolClientId: import.meta.env.VITE_COGNITO_CLIENT_ID,
      loginWith: {
        oauth: {
          domain: import.meta.env.VITE_COGNITO_OAUTH_DOMAIN,
          scopes: ['openid', 'email', 'profile', 'aws.cognito.signin.user.admin'],
          redirectSignIn: [import.meta.env.VITE_COGNITO_REDIRECT_SIGN_IN],
          redirectSignOut: [import.meta.env.VITE_COGNITO_REDIRECT_SIGN_OUT],
          responseType: 'code',
        }
      }
    },
  },
});
```

Cần bổ sung các biến môi trường sau vào `.env` của Frontend (ngoài các biến đã nêu ở mục 5.2):

```env
VITE_COGNITO_CLIENT_ID=<Cognito App Client ID>
VITE_COGNITO_OAUTH_DOMAIN=<Cognito Hosted UI domain>
VITE_COGNITO_REDIRECT_SIGN_IN=<URL callback sau khi đăng nhập, VD http://localhost:5173/auth/callback>
VITE_COGNITO_REDIRECT_SIGN_OUT=<URL sau khi đăng xuất>
```

| ![Ảnh 1](/images/5-Workshop/5.4-Configure-authentication/Login.jpg) | ![Ảnh 2](/images/5-Workshop/5.4-Configure-authentication/Logingg.jpg) |
|---|---|

---

## Một điểm tùy biến đáng chú ý: "Remember Me" động

Thay vì luôn lưu token vào localStorage (tồn tại vĩnh viễn) hoặc luôn dùng sessionStorage (mất khi đóng tab), dự án tự viết một KeyValueStorageInterface tùy chỉnh cho Amplify (dynamicAuthStorage trong main.tsx), quyết định nơi lưu token **theo thời gian thực** dựa trên lựa chọn "Remember Me" của user lúc đăng nhập:

- Nếu user chọn "Remember Me" → token lưu ở localStorage (giữ đăng nhập qua nhiều phiên).
- Nếu không chọn → token lưu ở sessionStorage (mất khi đóng trình duyệt).

---

## Bước 1: Mở Amazon Cognito User Pool

Mở AWS Management Console → `Amazon Cognito > User pools`, chọn đúng User Pool đang được dùng cho hệ thống.

---

## Bước 2: Kiểm tra cấu hình User Pool

| Nội dung | Cấu hình đang dùng |
|---|---|
| Sign-in method | Đăng nhập bằng **email** |
| Federated Identity Provider | **Google** (qua Cognito Hosted UI) |
| App Client | Loại **Public client** (không có Client Secret) |
| Hosted UI Domain | Domain dạng `<prefix>.auth.ap-southeast-1.amazoncognito.com |
| Token | Access Token + ID Token; Frontend sử dụng **ID Token** để gọi API |



---
## Bước 3: Kiểm tra cấu hình Google làm Identity Provider

Mở Amazon Cognito > User pools > Sign-in experience > Federated identity provider sign-in, kiểm tra provider **Google** đã được cấu hình với Client ID/Client Secret lấy từ Google Cloud Console (OAuth Consent Screen + Credentials).

![Ảnh sign-inGoogle](/images/5-Workshop/5.4-Configure-authentication/sign-inGoogle.jpg)

Kiểm tra tiếp `App integration > App client > Hosted UI` để đảm bảo:
- Provider **Google** đã được bật cho App Client đang dùng.
- **Allowed callback URLs** và **Allowed sign-out URLs** khớp với VITE_COGNITO_REDIRECT_SIGN_IN / VITE_COGNITO_REDIRECT_SIGN_OUT ở Frontend.
![Ảnh sign-inGoogle](/images/5-Workshop/5.4-Configure-authentication/sign-inGoogle1.jpg)
---

## Bước 4: Kiểm tra Cognito User Groups

Event Management Platform dùng **Cognito Group Admins** để phân biệt vai trò Admin.

Mở `Amazon Cognito > User pools > Groups`, kiểm tra group Admins. Người dùng thuộc group này khi đăng nhập sẽ có claim cognito:groups chứa "Admins" — áp dụng cho cả user đăng nhập bằng email lẫn Google.

![UserPoolAdmin](/images/5-Workshop/5.4-Configure-authentication/UserPoolAdmin.jpg)

---

## Bước 5: Cách backend đọc claims và xác định quyền Admin

Sau khi API Gateway xác thực JWT thành công, claims được đưa vào request.RequestContext.Authorizer.Claims:

```text
sub            → UserId
email          → Email
name           → FullName
cognito:groups → chuỗi dạng "[Admins]" hoặc "[Admins, Organizers]"
```

```csharp
public bool IsAdmin => Groups.Contains("Admins");
```

Vai trò (IsAdmin) được đồng bộ vào DynamoDB (bảng EventManagementUsers) mỗi khi gọi POST /profile/init — áp dụng chung cho cả user đăng ký thường và user đăng nhập bằng Google.

---

## Bước 6: Test luồng xác thực

1. Đăng ký một tài khoản mới bằng email, xác nhận mã, đăng nhập.
2. Test đăng nhập bằng nút "Tiếp tục với Google" — xác nhận redirect qua Google và quay lại Frontend thành công.
3. Mở DevTools (Network tab), kiểm tra các request gọi API có header Authorization: Bearer <idToken> với cả hai loại tài khoản.
4. Vào Cognito Console, thêm một trong hai user vào group Admins, đăng xuất/đăng nhập lại, gọi GET /profile/me để xác nhận vai trò trả về là Admin.

![Profileme](/images/5-Workshop/5.4-Configure-authentication/Profileme.jpg)

---

## Kết quả mong đợi

Sau khi hoàn thành phần này, người thực hiện có thể:

- Đăng ký và đăng nhập thành công bằng email/mật khẩu thông qua Amazon Cognito.
- Đăng nhập thành công bằng tài khoản Google qua Cognito Hosted UI (Federated Identity Provider).
- Xác nhận được ID Token được đính kèm đúng vào header Authorization của mọi request gọi API, với cả hai phương thức đăng nhập.
- Thêm một user vào Cognito Group Admins và xác nhận claim cognito:groups phản ánh đúng vai trò khi gọi GET /profile/me.
- Hiểu cách backend trích xuất claims từ API Gateway (sub, email, name, cognito:groups) mà không cần tự verify JWT trong Lambda.
- Hiểu cơ chế "Remember Me" động, quyết định lưu token ở localStorage hay sessionStorage tùy theo lựa chọn của người dùng.
- Dự án đã sẵn sàng cho các phần nghiệp vụ tiếp theo (Event Management, Ticket, Check-in...) vốn đều phụ thuộc vào JWT hợp lệ và claim vai trò được xác lập ở bước này.