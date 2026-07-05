---
title: "Worklog Tuần 11"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---



### Mục tiêu tuần 11:

- Hoàn thiện triển khai giao diện Frontend trên AWS thông qua Amazon S3 và Amazon CloudFront.
- Cấu hình đồng bộ các dịch vụ liên quan như Amazon Cognito, CloudFront và các biến môi trường để hệ thống hoạt động ổn định trên môi trường thực tế.
- Thiết lập cơ chế giám sát hệ thống bằng Amazon CloudWatch, kiểm thử toàn bộ các chức năng và hoàn thiện mã nguồn, tài liệu trước khi kết thúc dự án.


### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Thiết kế và bổ sung tài nguyên S3 Bucket + CloudFront Distribution (kèm Origin Access Control) vào template.yaml phục vụ lưu trữ và phân phối Frontend.                                                                                             | 29/06/2026   | 29/06/2026      |
| 3   | - Cập nhật cấu hình Cognito App Client (Allowed callback/sign-out URL) và biến môi trường Frontend theo domain CloudFront thật                                            | 30/06/2026   | 30/06/2026      |  |
| 4   | - Rà soát và sửa các lỗi biên dịch TypeScript còn tồn đọng chặn quá trình build Frontend, build và triển khai (upload S3 + invalidate cache CloudFront). | 01/07/2026   | 01/07/2026      |  |
| 5   | - Cấu hình CloudWatch Logs/Alarms cơ bản để giám sát lỗi các Lambda, kiểm tra log runtime của toàn bộ hệ thống sau khi triển khai.                  | 02/07/2026   | 02/07/2026      |  |
| 6   | - Kiểm thử tổng thể toàn bộ luồng nghiệp vụ đã triển khai, dọn dẹp mã nguồn thừa, tổng hợp kiến trúc và kết quả để đưa vào báo cáo đồ án.                                                                                    | 03/07/2026   | 03/07/2026      |  |


### Kết quả đạt được tuần 11:

- Đã thiết kế và bổ sung tài nguyên **Amazon S3 Bucket** và **Amazon CloudFront Distribution** (sử dụng Origin Access Control) vào **template.yaml**, hoàn thiện hạ tầng phục vụ lưu trữ và phân phối giao diện Frontend.
- Đã cập nhật cấu hình **Amazon Cognito App Client**, bao gồm Callback URL, Sign-out URL và các biến môi trường của Frontend theo tên miền CloudFront, bảo đảm chức năng xác thực người dùng hoạt động chính xác sau khi triển khai.
- Đã rà soát và khắc phục các lỗi biên dịch TypeScript còn tồn đọng, build thành công ứng dụng Frontend, triển khai lên Amazon S3 và làm mới bộ nhớ đệm (Cache Invalidation) của Amazon CloudFront để cập nhật phiên bản mới nhất.
- Đã cấu hình **Amazon CloudWatch Logs** và **CloudWatch Alarms** để theo dõi hoạt động của các Lambda Function, kiểm tra log runtime và hỗ trợ phát hiện, xử lý lỗi sau khi hệ thống được triển khai.
- Đã thực hiện kiểm thử tổng thể toàn bộ các luồng nghiệp vụ của hệ thống, bao gồm xác thực người dùng, quản lý sự kiện, đăng ký vé, gửi email thông báo, điểm danh, cấp chứng chỉ và thống kê dữ liệu, bảo đảm các chức năng hoạt động ổn định.
- Đã dọn dẹp và tối ưu mã nguồn, tổng hợp kiến trúc hệ thống, các công nghệ sử dụng và kết quả triển khai để hoàn thiện báo cáo đồ án và kết thúc quá trình phát triển dự án.