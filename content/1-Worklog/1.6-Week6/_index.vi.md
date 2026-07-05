---
title: "Worklog Tuần 6"
date: 2026-05-25
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---



### Mục tiêu tuần 6:
* Học cách tổ chức và quản lý tài nguyên AWS bằng cách sử dụng **Thẻ** và **Nhóm tài nguyên**.
* Hiểu cách kiểm soát quyền truy cập vào tài nguyên AWS bằng cách sử dụng **chính sách IAM**, **thẻ tài nguyên** và **Giới hạn quyền**.
* Có được kinh nghiệm thực hành với **Dịch vụ quản lý khóa AWS (KMS)** để mã hóa dữ liệu và các dịch vụ kiểm toán như **AWS CloudTrail** và **Amazon Athena**.
* Nghiên cứu và thực hành triển khai các ứng dụng có tính khả dụng cao và khả năng mở rộng bằng cách sử dụng **Amazon EC2 Auto Scaling** và **Application Load Balancer (ALB)**.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | **Thực hành:** - Thực hiện lab 27 quản lý tài nguyên bằng Tag và Resouce Groups <br>&emsp; + Sử dụng tags với CLI <br>&emsp; + Tạo một Resource Group                                                                                             | 25/05/2026   | 25/05/2026      | <https://000027.awsstudygroup.com/>|
| 3   |  **Thực hành:**<br> - Thực hiện lab 28 quản lý truy cập vào dịch vụ EC2 resource tag với AWS IAM <br>&emsp; + Tạo IAM User, IAM Policy, IAM Role <br>&emsp;+ Chuyển Role <br>&emsp; + Truy cập EC2 console ở AWS Region <br>&emsp;Tạo EC2 có Tags và không Tag <br>&emsp; + Chỉnh sửa Resouce Tag                                            | 26/05/2026   | 26/05/2026      | <https://000028.awsstudygroup.com> |
| 4   |  **Thực hành:**<br> - Thực hiện lab 30 giới hạn quyền của user với IAM Permission Boundary <br>&emsp; + Tạo Policy <br>&emsp; + Tạo IAM User giới hạn <br>&emsp; + Kiểm tra IAM User giới hạn| 27/05/2026   | 27/05/2026      | <https://000030.awsstudygroup.com> |
| 5   |  **Thực hành:**<br> - Thực hiện lab 33 mã hóa ở trạng thái lưu trữ với AWS KMS <br>&emsp; + Tạo Policy, Role, Group và User <br>&emsp; + Tạo Key Management Service <br>&emsp; + Tạo Amazon S3 <br>&emsp; + Tạo AWS CloudTrail và AWS Athena                   | 28/05/2026   | 28/05/2026      | <https://000033.awsstudygroup.com>> |
| 6   | - Nghiên cứu và thực hành Amazon EC2 Auto Scaling <br>&emsp; + Tìm hiểu Auto Scaling Group <br>&emsp;+ Tìm hiểu Launch Template <br>&emsp; + Tìm hiểu Elastic Load Balancer (ALB) + <br>&emsp; Thực hành tạo Auto Scaling Group và Load Balancer                                                                                | 29/05/2026   | 29/05/2026      | <https://www.youtube.com/playlist?list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i> <https://cloudjourney.awsstudygroup.com/> |
| 7   | - Tham dự event ở trên công ty                                                                                         | 30/05/2026   | 30/05/2026      | |


### Kết quả đạt được tuần 6:

* Hoàn thành thành công các bài thực hành AWS về **Thẻ**, **Nhóm tài nguyên**, **Kiểm soát truy cập dựa trên thẻ tài nguyên IAM**, **Ranh giới quyền** và **AWS KMS**.
* Học cách tổ chức và quản lý tài nguyên AWS hiệu quả bằng cách sử dụng Thẻ và Nhóm tài nguyên.
* Có được kinh nghiệm thực tế trong việc triển khai kiểm soát truy cập chi tiết với Chính sách IAM, Vai trò IAM, Thẻ tài nguyên và Ranh giới quyền.
* Cấu hình thành công AWS KMS để mã hóa dữ liệu khi lưu trữ và sử dụng AWS CloudTrail cùng với Amazon Athena để kiểm tra các hoạt động của AWS.
* Xây dựng và kiểm thử môi trường Amazon EC2 Auto Scaling tích hợp với Application Load Balancer, cải thiện khả năng mở rộng và tính khả dụng của ứng dụng.
* Nâng cao kiến ​​thức thực tiễn về bảo mật, quản trị, mã hóa, kiểm toán và thiết kế cơ sở hạ tầng có khả năng mở rộng của AWS.

