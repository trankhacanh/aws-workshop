---
title: "Worklog Tuần 4"
date: 2026-05-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---


### Mục tiêu tuần 4:

* Hoàn thành và hiểu cơ bản các dịch vụ ở labs 03
* Nắm vững kiến trúc, cách vận hành và quản lý các dịch vụ Máy ảo (EC2, Lightsail, Auto Scaling) cùng hệ thống Lưu trữ/Dịch chuyển đi kèm trên AWS
* Nắm vững kiến trúc lưu trữ đối tượng (Object Storage) của Amazon S3, bao gồm cách quản lý Bucket, phân quyền bảo mật và tối ưu hóa chi phí qua các Storage Class.
### Các công việc đã triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Tìm hiểu dịch vụ Compute VM trên AWS được đăng tải trên youtube AWS Study Group <br>&emsp; + Nghiên cứu & Cấu hình Amazon EC2 Core: Instance types, Key Pair, User data, Meta data <br>&emsp; + Quản lý Sao lưu & Đóng gói hạ tầng: AMI, Backup/Snapshot <br>&emsp; + Cấu hình Hệ thống Lưu trữ AWS (Storage): EBS, Instance store, EFS, FSx. <br>&emsp;                          + Hạ tầng Tự động co giãn: EC2 Auto Scaling <br>&emsp; + Dịch vụ Compute thay thế & Dịch chuyển: Amazon Lightsail, AWS MGN.                                                          | 11/05/2026   | 11/05/2026      | <https://www.youtube.com/watch?v=-t5h4N6vfBs&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=72>|
| 3   | **Thực hành:** <br> - Triển khai AWS Backup <br>&emsp; + Triển khai hạ tầng <br>&emsp; + Tạo Backup plan <br>&emsp; + Thiết lập thông báo và kiểm tra hoạt động <br> - Triển khai File Storage Gateway <br>&emsp; + Tạo S3 Bucket và EC2 cho Storage Gateway <br>&emsp; + Tạo Storage Gateway <br>&emsp; Tạo File Shares                                | 12/05/2026   | 12/05/2026      | <https://000013.awsstudygroup.com/> <https://000024.awsstudygroup.com/>|
| 4   | - Lên công ty thực tập <br> **Thực hành:** <br> - Làm quen với Amazon S3 <br>&emsp; + Tạo S3 bucket và tải dữ liệu <br>&emsp; + Bật tính năng static website <br>&emsp; + Cấu hình Block Public Access <br>&emsp; Cấu hình public object <br>&emsp; Tăng tốc website với Cloudfront <br> - Tìm hiểu lý thuyết dịch vụ và cách ứng dụng Cloundfront <br> - Tạo một Static Website deploy lên S3  | 13/05/2026   | 13/05/2026      | <https://000057.awsstudygroup.com/> |
| 5   | - Tìm hiểu dịch vụ lưu trữ trên AWS ngoài S3 <br>&emsp; + Amazon Storage Gateway, Snow Family, Disaster Recovery, AWS Backup                  | 14/05/2026   | 15/05/2026      | <https://www.youtube.com/watch?v=_yunukwcAwc&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=104>  <https://www.youtube.com/watch?v=mPBjB6Ltl_Q&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=105> <https://www.youtube.com/watch?v=YXn8Q_Hpsu4&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=106> <https://aws.amazon.com/backup>|
| 6   |  - **Thực hành (Compute & Storage):** <br>&emsp; + Triển khai máy chủ EC2, gắn thêm ổ đĩa EBS và mount ổ đĩa vào hệ thống.<br>&emsp; + Cấu hình tự động đồng bộ/đẩy file log từ máy chủ EC2 về Amazon S3 Bucket.<br>- **Bảo mật & Tối ưu:**<br> &emsp; + Gán IAM Role cho EC2 để phân quyền truy cập S3 an toàn (không dùng Access Key).<br>&emsp; + Cấu hình S3 Bucket Policy để bảo mật và chặn truy cập công khai.                                                                                        | 15/05/2026   | 15/05/2026      | <https://cloudjourney.awsstudygroup.com/> |


### Kết quả đạt được tuần 4:
* Hoàn thành các bài lab thuộc Module 03 về dịch vụ Compute và Storage trên AWS.
* Hiểu được cách triển khai, quản lý và sử dụng các dịch vụ Amazon EC2, Amazon Lightsail, Auto Scaling, EBS, EFS và AWS Backup.
* Thực hành thành công triển khai AWS Backup, File Storage Gateway và cấu hình sao lưu dữ liệu trên AWS.
* Nắm được cách sử dụng Amazon S3, bao gồm tạo Bucket, quản lý quyền truy cập, cấu hình Static Website Hosting và tích hợp Amazon CloudFront để phân phối nội dung.
* Triển khai thành công máy chủ EC2, gắn và sử dụng ổ đĩa EBS, đồng thời cấu hình đồng bộ dữ liệu từ EC2 lên Amazon S3.
* Áp dụng các cơ chế bảo mật cho hệ thống bằng IAM Role và S3 Bucket Policy nhằm kiểm soát quyền truy cập và tăng cường bảo mật dữ liệu.
* Nâng cao kỹ năng triển khai, quản lý và tối ưu các dịch vụ Compute và Storage trong môi trường AWS.


