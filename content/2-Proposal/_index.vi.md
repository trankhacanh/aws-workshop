---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
# CLOUD-BASED EVENT MANAGEMENT PLATFORM

## Nền tảng Quản lý, Phát hành Vé và Điểm danh Sự kiện Serverless trên AWS

### 1. Tóm tắt dự án

**Cloud-Based Event Management Platform** là giải pháp số hóa toàn diện quy trình tổ chức và quản lý sự kiện dựa trên nền tảng điện toán đám mây với kiến trúc Serverless. Dự án được nghiên cứu và phát triển nhằm tối ưu hóa chi phí vận hành, tự động mở rộng theo lượng request thực tế, hướng tới nhóm đối tượng là các đơn vị tổ chức hội thảo, workshop công nghệ dành cho cộng đồng lập trình viên và doanh nghiệp.

Hệ thống hỗ trợ cấu trúc phân quyền chặt chẽ thông qua phân tách vai trò: **Attendee (Người tham dự)** và **Admin/Organizer (Ban tổ chức)**. Nền tảng bao phủ trọn vẹn vòng đời của một sự kiện thông qua 4 phân hệ cốt lõi liên thông nhau:
- **Quản lý Sự kiện (Event Management):** Cho phép khởi tạo sự kiện công khai (Active) tức thì, cập nhật thông tin và quản lý hình ảnh banner trực quan.
- **Xác thực & Đăng ký Vé (Auth, Registration & Ticketing):** Quản lý định danh người dùng (Email/Google OAuth), xử lý logic kiểm tra và cập nhật chỗ trống (Slot) theo thời gian thực và tự động khóa cổng đăng ký khi sự kiện đạt giới hạn.
- **Điểm danh & Chứng nhận (Attendance & Certificate):** Hỗ trợ ban tổ chức quét mã QR của người tham dự tại địa điểm, tự động tạo chứng nhận điện tử dạng PDF lưu trữ đám mây và gửi link tải qua thư điện tử.
- **Giám sát & Thống kê (Monitoring & Analytics):** Thu thập dữ liệu vận hành thời gian thực và hiển thị các chỉ số đo lường hiệu quả sự kiện trên Dashboard.

Dự án ứng dụng mô hình **Serverless Modular Application** tại khu vực Singapore (**ap-southeast-1**). Toàn bộ hạ tầng đám mây này được quản lý và triển khai tự động dưới dạng Infrastructure as Code (IaC) thông qua công cụ AWS SAM (Serverless Application Model).

---
![Sơ đồ kiến trúc hệ thống](/images/cautruc.jpg)

### 2. Vấn đề đặt ra

#### Vấn đề là gì?
Hiện nay, phần lớn các câu lạc bộ, tổ chức công nghệ quy mô vừa và nhỏ khi tổ chức các buổi workshop vẫn phụ thuộc vào phương pháp thủ công hoặc các công cụ văn phòng rời rạc (Google Forms, Excel, gửi mail thủ công), dẫn đến các bất cập:
- **Khó kiểm soát số lượng thực tế:** Google Forms không có cơ chế chặn đăng ký tự động theo thời gian thực khi hội trường đã hết chỗ (Overbooking), dễ gây vỡ trận về không gian tổ chức.
- **Thiếu cơ chế hàng đợi tự động:** Khi sự kiện quá tải, ban tổ chức không thể tự động xếp người đăng ký muộn vào danh sách chờ (Waiting List) để xử lý cuốn chiếu khi có người hủy vé.
- **Ùn tắc tại điểm check-in:** Việc soát vé bằng cách đối chiếu danh sách Excel thủ công tại bàn đón tiếp gây tốn thời gian, dễ nhầm lẫn và tạo trải nghiệm xếp hàng tiêu cực cho người tham dự.
- **Tốn công sức hậu kỳ:** Quy trình thiết kế, tạo file chứng nhận tham gia (Certificate) và gửi email cho từng lập trình viên sau sự kiện đòi hỏi nhiều nhân sự và thời gian vận hành bằng tay.
- **Hạ tầng truyền thống lãng phí:** Việc duy trì máy chủ ảo (EC2) hoặc cơ sở dữ liệu bật 24/7 gây lãng phí ngân sách lớn trong những khoảng thời gian dài giữa các sự kiện (khi không có traffic).

#### Giải pháp
Nền tảng cung cấp một ứng dụng thống nhất, tự động hóa toàn bộ các điểm chạm trong quy trình tổ chức sự kiện dựa trên các dịch vụ đám mây được quản lý hoàn toàn (Fully-Managed Services):
- **Hệ thống Vé & Kiểm soát Slot thời gian thực:** Khách hàng đăng ký vé qua API, hệ thống tự động kiểm tra slot trống bằng biểu thức điều kiện trực tiếp dưới tầng cơ sở dữ liệu. Nếu còn chỗ, hệ thống cập nhật tăng số lượng và vé QR chính thức được tạo; nếu hết chỗ, hệ thống lập tức chặn request ngăn chặn tuyệt đối tình trạng Overbooking.
- **Soát vé QR tốc độ cao:** Ban tổ chức sử dụng camera trên giao diện Web/Mobile để quét mã QR độc bản trên vé của Attendee, hệ thống ghi nhận trạng thái check-in lập tức vào database.
- **Tự động hóa cấp chứng chỉ (PDF):** Sau khi sự kiện kết thúc, hệ thống kích hoạt Lambda biên dịch file PDF cá nhân hóa, đẩy lên Amazon S3 bảo mật và tự động gửi email chứa link tải cho người tham dự thông qua Amazon SES.
- **Thông báo tự động không phụ thuộc thao tác thủ công:** Hệ thống chủ động gửi email xác nhận ngay khi đăng ký vé thành công và tự động nhắc lịch trước giờ sự kiện diễn ra, thông qua cơ chế điều phối sự kiện bất đồng bộ của Amazon EventBridge, không cần ban tổ chức can thiệp gửi thủ công.
- **Bảo mật phân tầng nghiêm ngặt:** Quản lý tài khoản tập trung bằng Amazon Cognito, che chắn backend API bằng cổng API Gateway tích hợp bộ xác thực mã bảo mật Cognito JWT Authorizer.

#### Lợi ích và giá trị mang lại
- **Số hóa và tự động hóa 100%:** Giảm tải đến 90% khối lượng công việc thủ công của ban tổ chức trước, trong và sau sự kiện.
- **Khả năng co giãn vô hạn (Elastic Scalability):** Tự động mở rộng tài nguyên để xử lý hàng ngàn lượt đăng ký vé đồng thời tại thời điểm vừa mở cổng workshop mà không gây nghẽn hệ thống.
- **Tối ưu chi phí tuyệt đối:** Nhờ mô hình Serverless trả phí theo mức dùng (Pay-as-you-go), chi phí vận hành hệ thống trong giai đoạn không diễn ra sự kiện tiệm cận về mức 0 USD.
- **Kinh nghiệm thực tế chuẩn doanh nghiệp:** Dự án mang lại trải nghiệm thực tiễn sâu sắc cho nhóm phát triển trong việc quy hoạch dữ liệu NoSQL, viết mã nguồn hạ tầng (IaC) và kiểm thử liên thông hệ thống phân tán.

---

### 3. Kiến trúc giải pháp

Hệ thống được triển khai tập trung tại AWS Region Singapore (**ap-southeast-1**).

#### Sơ đồ kiến trúc và luồng dữ liệu tổng thể

![Sơ đồ kiến trúc Serverless Event Platform](/images/2-Proposal/system-architecture.png)
<p style="text-align: center;"><i>Hình 3.1: Sơ đồ kiến trúc và luồng tương tác dữ liệu Serverless trên AWS.</i></p>

**Mô tả luồng vận hành:**
Client (React Frontend) tương tác trực tiếp với **Amazon Cognito** để thực hiện luồng đăng nhập và nhận mã bảo mật JWT. Đối với các request nghiệp vụ, Client sẽ đính kèm JWT này vào Header để gửi qua cổng **Amazon API Gateway**. Tại đây, một bộ cấu hình **Cognito JWT Authorizer toàn cục** sẽ giải mã và kiểm tra tính hợp lệ của token trước khi định tuyến request xuống lớp logic xử lý.

Lớp nghiệp vụ bao gồm các cụm **AWS Lambda** độc lập chạy trên môi trường .NET 8. Dữ liệu của hệ thống được tổ chức theo mô hình **Multi-Table Design** trên **Amazon DynamoDB**. Đối với tệp tin media lớn (Banner sự kiện, ảnh đại diện, file chứng nhận PDF), hệ thống sử dụng **Amazon S3** kết hợp cơ chế **Presigned URL** bảo mật để Client tải dữ liệu lên/xuống trực tiếp.

Về tầng phân phối Frontend, hệ thống sử dụng **Amazon CloudFront** làm lớp CDN đặt trước bucket S3 Frontend (được cấu hình private, bật Block Public Access), liên kết qua cơ chế **Origin Access Control (OAC)** để đảm bảo chỉ CloudFront được phép đọc object từ bucket, người dùng không thể truy cập thẳng vào S3. Ở giai đoạn hiện tại, dự án sử dụng domain mặc định do CloudFront cấp phát (dạng `*.cloudfront.net`), chưa gắn Custom Domain riêng và chưa phát hành chứng chỉ ACM; đây là hạng mục mở rộng của giai đoạn phát triển tiếp theo khi dự án triển khai một tên miền thương hiệu riêng.

Về phần thông báo và tác vụ tự động, hệ thống áp dụng mô hình hướng sự kiện (event-driven), sử dụng trực tiếp **Amazon EventBridge** làm trung gian điều phối, không thông qua Amazon SNS hay Amazon SQS. Hai luồng thông báo được tách biệt rõ ràng:

- **Luồng nhắc lịch sự kiện (reminder):** Một EventBridge Schedule chạy định kỳ mỗi giờ (`rate(1 hour)`) kích hoạt trực tiếp Lambda Notification. Hàm này tự quét hai bảng `EventManagementEvents` và `EventManagementTickets` để xác định các sự kiện sắp diễn ra, gọi trực tiếp Amazon SES để gửi email nhắc lịch, và ghi nhận kết quả gửi vào bảng `EventManagementNotificationLogTable`.
- **Luồng thông báo đăng ký vé:** Ngay sau khi Lambda Registration & Ticketing ghi nhận một vé thành công, hàm này phát hành (`PutEvents`) một sự kiện tùy chỉnh (source: `eventmanagement.ticket`, detail-type: `TicketRegistered`) lên một Custom Event Bus của EventBridge. Một EventBridge Rule được cấu hình để lắng nghe đúng mẫu sự kiện này sẽ bất đồng bộ kích hoạt cùng Lambda Notification nêu trên, từ đó gọi Amazon SES gửi email xác nhận đăng ký và ghi log tương ứng.

Nhờ thiết kế này, một Lambda Notification duy nhất (viết bằng .NET 8/C#) đảm nhiệm cả hai nghiệp vụ thông báo, được kích hoạt từ hai nguồn sự kiện độc lập của EventBridge mà không cần bổ sung thêm tầng hàng đợi trung gian.

#### Các dịch vụ AWS sử dụng trong dự án
- **Amazon Cognito:** Quản lý đăng ký tài khoản qua Email (xác thực bằng mã Verification Code) và liên thông tài khoản Google Login (OAuth2), phát hành JWT token phân quyền nhóm (Attendee/Admin).
- **Amazon API Gateway:** Cổng HTTP/REST API tiếp nhận request, định tuyến luồng dữ liệu và thực thi Authorizer bảo mật ở tầng biên.
- **AWS Lambda (.NET 8):** Thực thi toàn bộ logic nghiệp vụ (CRUD sự kiện, đăng ký giữ chỗ, xử lý hàng đợi Waiting List, kiểm tra mã QR check-in, khởi tạo tệp chứng nhận PDF, và xử lý thông báo/nhắc lịch qua EventBridge).
- **Amazon DynamoDB (Multi-Table):** Hệ thống cơ sở dữ liệu NoSQL lưu trữ thông tin phân tán qua các bảng độc lập: **EventManagementEvents**, **EventManagementCategories**, **EventManagementTickets**, **EventManagementUsers**, **EventManagementAttendance**, và **EventManagementNotificationLogTable**.
- **Amazon S3:** Lưu trữ tĩnh bảo mật cho tài nguyên hệ thống (Event Banners, User Avatars, Certificate PDFs, Frontend build) với chính sách chặn truy cập công khai (Block Public Access) nghiêm ngặt.
- **Amazon SES (Simple Email Service):** Dịch vụ gửi thư điện tử tin cậy, dùng để gửi mã xác thực tài khoản, gửi email xác nhận đăng ký vé, gửi email nhắc lịch sự kiện, và gửi liên kết tải chứng nhận PDF cho người dùng.
- **Amazon CloudWatch:** Hệ thống giám sát tập trung, thu thập log thực thi từ Lambda và các chỉ số của API Gateway, DynamoDB, EventBridge để phục vụ công tác debug và theo dõi sức khỏe hệ thống.
- **AWS SAM / AWS CloudFormation:** Công cụ định nghĩa hạ tầng bằng mã nguồn, quản lý và deploy toàn bộ stack tài nguyên đám mây thông qua file **template.yaml**.
- **Amazon CloudFront:** Đóng vai trò lớp phân phối nội dung tĩnh (CDN) cho Frontend React, đặt trước bucket S3 Frontend private thông qua cơ chế Origin Access Control (OAC). Hiện sử dụng domain mặc định do AWS cấp phát, chưa tích hợp Custom Domain/ACM Certificate.
- **Amazon EventBridge (Schedule & Custom Event Bus):** Đóng vai trò trung tâm điều phối sự kiện bất đồng bộ cho phân hệ Notification, gồm một Schedule chạy định kỳ hàng giờ để gửi email nhắc lịch sự kiện, và một Rule lắng nghe sự kiện `TicketRegistered` phát ra từ Lambda đăng ký vé để gửi email xác nhận.
- **Amazon SNS / SQS:** Không sử dụng trong kiến trúc hiện tại. Toàn bộ luồng thông báo được xử lý trực tiếp qua Amazon EventBridge kết hợp gọi thẳng Amazon SES từ Lambda Notification, không qua tầng hàng đợi hay pub/sub trung gian bổ sung.

---

### 4. Triển khai kỹ thuật

#### Các giai đoạn triển khai (Từ Tuần 7 đến Tuần 12)
- **Giai đoạn 1 – Nghiên cứu & Đánh giá (Tuần 7):** Nghiên cứu tổng quan yêu cầu dự án quản lý sự kiện, chọn lọc và đánh giá tính khả thi kỹ thuật của cụm dịch vụ Serverless AWS.
- **Giai đoạn 2 – Phân tích & Thiết kế luồng (Tuần 8):** Định vị phạm vi MVP, thiết kế sơ đồ User Flow cho luồng xác thực (Email OTP, Google OAuth) và luồng nghiệp vụ đăng ký giữ chỗ / hàng đợi danh sách chờ.
- **Giai đoạn 3 – Khởi tạo hạ tầng IaC (Tuần 9):** Cấu hình AWS SAM CLI, viết file cấu hình tài nguyên triển khai cụm bảng DynamoDB dạng Multi-Table, thiết lập S3 Bucket lưu trữ assets và phân quyền IAM Roles bảo mật.
- **Giai đoạn 4 – Tích hợp Xác thực & Bảo mật API (Tuần 10):** Triển khai Cognito User Pool, thiết lập luồng gửi mã Verification Code qua Email và cấu hình bộ lọc Cognito JWT Authorizer trên API Gateway.
- **Giai đoạn 5 – Phát triển Code Logic Nghiệp vụ (Tuần 11):** Tích hợp Google OAuth vào Cognito. Viết mã nguồn .NET 8 cho Lambda xử lý tạo/sửa/xóa sự kiện, sinh S3 Presigned URL cho banner/avatar, xử lý trừ slot đăng ký vé, logic check-in bằng mã QR, cấu hình EventBridge Schedule gửi nhắc lịch sự kiện định kỳ hàng giờ, thiết lập Custom Event Bus và EventBridge Rule lắng nghe sự kiện `TicketRegistered` để kích hoạt Lambda Notification bất đồng bộ, deploy Frontend qua S3 + CloudFront với Origin Access Control, và tích hợp SES gửi email cho cả hai luồng thông báo.
- **Giai đoạn 6 – Kiểm thử End-to-End & Tối ưu hóa (Tuần 12):** Thực hiện kiểm thử liên thông toàn hệ thống, tối ưu hóa cấu hình tài nguyên để tiết kiệm chi phí, hoàn thiện tài liệu Workshop Report và chuẩn bị kịch bản demo tốt nghiệp.

---

### 5. Timeline & Milestones

| Mốc thời gian | Tên giai đoạn / Cột mốc quan trọng | Kết quả bàn giao thực tế (Deliverables) |
| --- | --- | --- |
| **Tuần 7** | Nghiên cứu yêu cầu & Đánh giá khả thi | Tài liệu phân tích yêu cầu, danh sách dịch vụ AWS lựa chọn. |
| **Tuần 8** | Thiết kế Kiến trúc & Sơ đồ User Flow | Sơ đồ kiến trúc Serverless, Sơ đồ User Flow cho Auth và Vé. |
| **Tuần 9** | Khởi tạo hạ tầng bằng AWS SAM | Mã nguồn **template.yaml** tạo thành công Multi-Table DynamoDB, S3, IAM. |
| **Tuần 10** | Triển khai Xác thực & Cổng Bảo mật | Cụm Cognito hoạt động ổn định, API Gateway kích hoạt JWT Authorizer. |
| **Tuần 11** | Phát triển Logic Nghiệp vụ Backend | Hoàn thiện mã nguồn các hàm Lambda xử lý Sự kiện, Đăng ký vé, QR Check-in, EventBridge Schedule/Rule cho Notification, deploy Frontend qua CloudFront, và SES. |
| **Tuần 12** | Kiểm thử End-to-End & Đóng gói | Hệ thống chạy thông suốt, hoàn thiện tài liệu kỹ thuật, chuẩn bị kịch bản demo tốt nghiệp. |

---

### 6. Ước tính chi phí vận hành (Bản đề xuất kinh tế cho quy mô Demo)

| Tên dịch vụ AWS | Mức sử dụng giả định | Chi phí ước tính / Tháng | Ghi chú chính sách AWS |
| --- | --- | --- | --- |
| **Amazon Cognito** | 200 MAU (Email & Google Login) | **$0.00** | Miễn phí hoàn toàn cho 50.000 MAU đầu tiên hàng tháng. |
| **Amazon API Gateway** | 20.000 REST API requests | **$0.07** | Đơn giá vùng ap-southeast-1 là $3.50 / 1 triệu requests. |
| **AWS Lambda** | 40.000 lượt chạy (Cấu hình 512MB RAM, trung bình 400ms/lượt) | **$0.00** | Nằm hoàn toàn trong hạn mức Free Tier miễn phí của AWS. |
| **Amazon DynamoDB** | 6 bảng độc lập (bao gồm `EventManagementNotificationLogTable`), chế độ On-Demand, dung lượng 500MB | **$0.05** | Chi phí lưu trữ $0.25/GB. |
| **Amazon S3** | Lưu trữ 2 GB dữ liệu Standard (bao gồm build Frontend), 5.000 requests PUT/GET | **$0.08** | Chi phí lưu trữ Standard: $0.025/GB. |
| **Amazon SES** | Gửi 500 emails (Mã OTP, xác nhận đăng ký vé, nhắc lịch, link tải Chứng nhận) | **$0.05** | Đơn giá: $0.10 cho mỗi 1.000 emails gửi đi. |
| **Amazon CloudWatch** | Thu thập và nạp 1 GB dữ liệu Log hệ thống | **$0.50** | Đơn giá nạp log: $0.50/GB tại Singapore. |
| **Amazon CloudFront** | ~2.000 requests, dùng domain mặc định `*.cloudfront.net`, chưa Custom Domain | **$0.02** | Nằm phần lớn trong hạn mức Free Tier 1TB data transfer/12 tháng đầu; ước tính cho phần vượt nhẹ. |
| **Amazon EventBridge (Schedule + Custom Event Bus)** | ~720 lượt Schedule/tháng (mỗi giờ) + số lượt `PutEvents` bằng số vé đăng ký | **$0.01** | Custom event tính phí theo $1/triệu sự kiện phát hành; ở quy mô demo gần như không đáng kể. |
| **Data Transfer Out** | ~2 GB dữ liệu truyền ra Internet | **$0.18** | Đơn giá truyền dữ liệu ra Internet công cộng: $0.09/GB. |

#### Tổng kết chi phí dự kiến: ~ $0.96 USD / tháng.

---

### 7. Đánh giá rủi ro và Biện pháp giảm thiểu

#### Ma trận rủi ro kỹ thuật
1. **Trùng lặp request đăng ký vé khi sự kiện sắp hết chỗ (Race Condition):**
   - *Biện pháp:* Sử dụng cơ chế khóa điều kiện **DynamoDB Conditional Expressions** tại tầng database để kiểm tra biến đếm số chỗ trống một cách cô lập và tuần tự trước khi ghi nhận trạng thái **CONFIRMED**.
2. **Kẻ xấu spam tải tệp độc hại hoặc chiếm quyền ghi tệp trên Cloud:**
   - *Biện pháp:* Kích hoạt **S3 Block Public Access** trên diện rộng. Toàn bộ hoạt động upload ảnh banner hoặc avatar bắt buộc phải xin cấp quyền qua endpoint sinh **S3 Presigned URL** với thời gian sống giới hạn dưới 15 phút. Riêng bucket Frontend chỉ cho phép đọc qua CloudFront bằng Origin Access Control, không public trực tiếp.
3. **Chi phí tăng ngoài kiểm soát do mã nguồn dính vòng lặp vô hạn (Infinite Loop):**
   - *Biện pháp:* Đặt giới hạn **Hard Timeout** nghiêm ngặt cho mỗi hàm Lambda không quá 10 giây. Đồng thời, cấu hình **AWS Budgets Alerts** để gửi cảnh báo nếu chi phí tài khoản chạm ngưỡng 5 USD.
4. **Quá tải/throttle khi gửi email hàng loạt hoặc sai lệch múi giờ trong EventBridge Schedule:** EventBridge Schedule chạy `rate(1 hour)` theo giờ UTC có thể dẫn đến gửi email nhắc lịch lệch múi giờ nếu không quy đổi đúng sang giờ Việt Nam (UTC+7) khi so sánh với thời gian sự kiện; đồng thời nếu số lượng sự kiện/vé lớn, việc gọi Amazon SES đồng loạt từ Lambda Notification có thể chạm giới hạn gửi (sending rate) của tài khoản SES.
   - *Biện pháp:* Chuẩn hóa toàn bộ timestamp lưu trong DynamoDB về UTC và quy đổi tường minh sang giờ Việt Nam ở tầng logic khi so sánh "sự kiện sắp diễn ra". Theo dõi chỉ số `TargetErrorCount` của EventBridge Schedule và `Throttle`/`Bounce` của SES qua CloudWatch. Giới hạn số email gửi mỗi lượt thực thi và thực hiện thủ tục nâng hạn mức gửi (Sending Limits) của tài khoản SES tương ứng với quy mô sự kiện thực tế.

---

### 8. Kết quả mong đợi và Sản phẩm bàn giao

#### Sản phẩm bàn giao của dự án
1. **Mã nguồn ứng dụng & Hạ tầng đám mây:** Thư mục mã nguồn Backend .NET 8 chạy ổn định và file cấu hình kiến trúc **template.yaml** (AWS SAM CLI) hoàn chỉnh, bao gồm cấu hình EventBridge Schedule, Custom Event Bus/Rule và CloudFront Distribution.
2. **Cơ sở dữ liệu đám mây:** Cấu trúc 6 bảng DynamoDB Multi-Table (bao gồm `EventManagementNotificationLogTable`) hoạt động thông suốt, đồng bộ dữ liệu chuẩn xác.
3. **Tài liệu Kỹ thuật (Workshop Report):** Tài liệu phân tích hệ thống, sơ đồ User Flow, thiết kế chi tiết bảng NoSQL, mô tả luồng Notification/Reminder qua EventBridge, và các kịch bản kiểm thử API liên thông End-to-End thành công.