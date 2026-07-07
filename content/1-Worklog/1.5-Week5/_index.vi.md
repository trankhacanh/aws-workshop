---
title: "Worklog Tuần 5"
date: 2026-05-18
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---



### Mục tiêu tuần 5:
* Hoàn thành các bài lab của module 04.
* Hiểu quy trình di chuyển máy ảo từ môi trường on-premise lên AWS bằng VM Import/Export.
* Nắm được cách triển khai và quản lý hệ thống lưu trữ Amazon FSx for Windows File Server.
* Tìm hiểu các dịch vụ bảo mật trên AWS như IAM, AWS Organizations, AWS Identity Center (SSO), AWS KMS và AWS Security Hub.
* Hiểu các tiêu chuẩn bảo mật phổ biến (CIS Benchmark, PCI DSS) và cách áp dụng Security Hub để đánh giá mức độ tuân thủ.
* Thực hành sử dụng AWS Lambda để tự động hóa việc quản lý tài nguyên và tối ưu chi phí vận hành EC2.


### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | **Thực hành:** <br> - Thực hiện lab 14 làm việc với máy ảo <br>&emsp; + Export máy ảo từ on-premise <br>&emsp; + Tải máy ảo lên AWS <br>&emsp; + Import máy ảo vào AWS <br>&emsp; + Triển khai Instance từ AMI <br>&emsp; Thiết lập ACL cho S3 Bucket <br>&emsp; + Export máy ảo từ Instance                                                                                          | 18/05/2026   | 18/05/2026      | <https://000014.awsstudygroup.com/>|
| 3   |**Thực hành:**  <br>  - Triển khai FSX trên Windows  <br>&emsp; + Tạo môi trường AWS CloudFormation  <br>&emsp; Tạo SSD và HDD file system <br>&emsp; + Tạo file share <br>&emsp; + Kiểm tra và giám sát hiệu năng <br>&emsp; + Kích hoạt chống dữ liệu trùng lặp, shadow copies <br>&emsp; + Bật hạn mức bộ nhớ cho người dùng <br>&emsp;  + Mở rộng khả năng thông lượng và dung lượng lưu trữ                                      | 19/05/2026   | 19/05/2026      | <https://www.youtube.com/playlist?list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i> |
| 4   | - Tìm hiểu các dịch vụ bảo mật trên AWS <br>&emsp; + Shared Responsibility Model <br>&emsp; + AWS identity and Access Management <br>&emsp; + Amazon Cognito <br>&emsp; + AWS Organization, AWS Identity Center (SSO) <br>&emsp; + AWS KMS <br>&emsp; + AWS Security Hub <br>&emsp; Hands-on and Additional research | 20/05/2026   | 20/05/2026      | <https://cloudjourney.awsstudygroup.com/> <https://www.youtube.com/playlist?list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i>|
| 5   | - Tìm hiểu các tiêu chuẩn bảo mật <br>&emsp; CIS , PCI DSS <br> **Thực hành:**  <br> - Thực hiện lab 18 Làm quen với AWS Security Hub <br>&emsp; + Kích hoạt Security Hub <br>&emsp; + Điểm từng bộ tiêu chuẩn                  | 21/05/2026   | 21/05/2026      | <https://000018.awsstudygroup.com/> |
| 6   |  **Thực hành:** <br> - Lab 22 Tối ưu chi phí EC2 với Lambda <br>&emsp; Tạo VPC, SG, EC2 Instance <br>&emsp; Incoming Web-hooks slack <br>&emsp; Tạo Role cho Lambda, <br>&emsp; Tạo Lambda Function                                                                     | 22/05/2026   | 22/05/2026      | <https://000022.awsstudygroup.com/> |


### Kết quả đạt được tuần 5:


* Hoàn thành các bài lab của module 04.
* Thực hiện thành công quy trình import/export máy ảo giữa môi trường on-premise và AWS, đồng thời triển khai EC2 từ AMI.
* Triển khai và cấu hình Amazon FSx for Windows File Server, thực hành quản lý chia sẻ dữ liệu, giám sát hiệu năng và mở rộng hệ thống lưu trữ.
* Hiểu được mô hình bảo mật của AWS, các dịch vụ IAM, AWS Organizations, AWS Identity Center (SSO), AWS KMS và vai trò của AWS Security Hub trong việc giám sát bảo mật.
* Thực hành đánh giá mức độ tuân thủ theo các tiêu chuẩn CIS Benchmark và PCI DSS thông qua AWS Security Hub.
* Xây dựng Lambda Function để tự động quản lý EC2, kết hợp IAM Role và Slack Webhook nhằm hỗ trợ tối ưu chi phí và tự động hóa vận hành.