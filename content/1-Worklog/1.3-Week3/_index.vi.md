---
title: "Worklog Tuần 3"
date: 2026-05-04
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---



### Mục tiêu tuần 3:
* Hoàn thành được cái bài lab của module 02.
* Hiểu rõ lý thuyết cốt lõi về hạ tầng mạng AWS (VPC, Subnet, Gateways, Security Layers,..)
* Tự tay thiết kế và triển khai một kiến trúc mạng VPC cơ bản gồm 2 vùng Public/Private Subnet, có cấu hình bảo mật (SG, NACL) và thiết lập NAT Gateway cho máy chủ nội bộ kết nối Internet.
* Cấu hình thành công bộ cân bằng tải (Load Balancer) để phân phối lưu lượng truy cập qua Internet Gateway
### Các công việc đã triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Tìm hiểu lý thuyết về hạ tầng mạng AWS trên youtube <br>&emsp; + VPC <br>&emsp; + Subnet <br>&emsp; + ENI <br>&emsp; ... <br> - Tìm hiểu cơ chế hoạt động và trường hợp sử dụng của các giải pháp bảo mật <br>&emsp; + SG <br>&emsp; + NACL <br> - Tìm hiểu các mô hình kết nối liên VPC <br>&emsp; + VPC Peering <br>&emsp; + Transit Gatewway                                                                                             | 04/05/2026   | 04/05/2026      |https://www.youtube.com/watch?v=O9Ac_vGHquM&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=25 <br> https://www.youtube.com/watch?v=BPuD1l2hEQ4&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=26 |
| 3   |  **Thực hành:** <br>-  Cấu hình hạ tầng mạng trên AWS <br>&emsp; + Tạo VPC <br>&emsp; + Tạo Subnet <br>&emsp; + Tạo Internet Gateway <br>&emsp; + Tạo Route Table <br>&emsp; ... <br> - Triển khai Amazon EC2 Instances <br> - Cấu hình Site to Site VPN                                        | 05/05/2026   | 05/05/2026      | <https://000003.awsstudygroup.com/> |
| 4   | **Thực hành:** <br> - Thiết lập Hybrid DNS với Route 53 Resolver <br>&emsp; + Tạo Key Pair  <br>&emsp; + Khởi tạo CloudFormation Template  <br>&emsp; + Cấu hình SG <br> - Kết nối đến RDGW <br> - Thiết lập DNS | 06/05/2026   | 06/05/2026      | <https://000010.awsstudygroup.com/>|
| 5   | **Thực hành:** <br>- Thiết lập VPC Peering  <br>&emsp; + Chuẩn bị CloudFormation, SG, EC2 instance  <br>&emsp; + Cập nhật Network ACL  <br>&emsp; + Tọa kết nối Peering  <br>&emsp; + Kích hoạt Cross-Peer DNS              | 07/05/2026   | 07/05/2026      | <https://000019.awsstudygroup.com/> |
| 6   |  **Thực hành:** <br> - Triển khai kết nối 4 VPC thông qua AWS Transit Gateway <br>&emsp; + Tạo Transit Gateway <br>&emsp; + Tạo Transit Gateway Attachments <br>emsp; Tạo Route Tables cho Transit Gateway                                                                                         | 08/05/2026   | 08/05/2026      | <https://000020.awsstudygroup.com/>|


### Kết quả đạt được tuần 3:

* Hoàn thành các bài lab thuộc Module 02 về hạ tầng mạng AWS.
* Hiểu được kiến trúc và nguyên lý hoạt động của Amazon VPC, Subnet, Route Table, Internet Gateway, NAT Gateway, Security Group và Network ACL.
* Triển khai thành công một kiến trúc VPC cơ bản với Public/Private Subnet và cấu hình các thành phần mạng cần thiết.
* Thiết lập và kiểm tra kết nối AWS Site-to-Site VPN, Hybrid DNS, VPC Peering và AWS Transit Gateway theo hướng dẫn của các bài lab.
* Thực hành triển khai EC2 trong môi trường VPC và cấu hình các thành phần bảo mật, định tuyến và kết nối giữa các mạng.
* Nâng cao kỹ năng sử dụng AWS Management Console và hiểu rõ hơn về cách thiết kế hạ tầng mạng trên AWS.