---
title: "Các bài blogs đã đăng"
date: 2026-0-03
weight: 3
chapter: false
pre: " <b> 3. </b> "
---


###  [Blog 1 - Xây dựng Kiến trúc SaaS Đa thuê bao dạng Lai cho các Dịch vụ Lưu trạng thái trên AWS](3.1-blog1/)
Bài viết này giới thiệu giải pháp Kiến trúc Lai (Hybrid) trên AWS, kết hợp giữa mô hình Cô lập (Silo) và Chia sẻ (Pool) cho các dịch vụ lưu trạng thái (Stateful Services như Server Game, Chat thời gian thực). Hệ thống tối ưu hóa chi phí nhờ mô hình Pool cho khách hàng nhỏ và đảm bảo hiệu năng cho khách hàng VIP/Enterprise bằng phân vùng Silo độc lập, sử dụng DynamoDB làm "Routing Table" động cùng cụm Amazon EKS và ElastiCache (Redis) để đồng bộ trạng thái phiên (Session State) theo thời gian thực.

###  [Blog 2 - Xây dựng hệ thống Notification, Analytics và Deployment hoàn toàn Serverless trên AWS](3.2-blog2/)
Bài viết này chia sẻ kinh nghiệm triển khai dự án quản lý Workshop sử dụng hoàn toàn kiến trúc Serverless và tư duy Hướng sự kiện (Event-Driven) trên AWS. Hệ thống sử dụng Amazon EventBridge làm trung tâm tiếp nhận và phân phối sự kiện giúp giảm sự phụ thuộc giữa các thành phần, kết hợp với AWS Lambda và Amazon SES phục vụ gửi email tự động; DynamoDB hỗ trợ tổng hợp số liệu cho Dashboard; cùng CloudWatch đảm nhiệm vai trò giám sát và vận hành toàn diện.

###  [Blog 3 - Phân Tích Sâu Về AWS Lambda Concurrency – Điều gì thực sự xảy ra khi hàng nghìn request cùng lúc gọi một Lambda?](3.3-blog3/)
Bài viết đi sâu tìm hiểu cơ chế vận hành của AWS Lambda Concurrency và cách hệ thống tự động mở rộng (Auto Scaling). Nội dung làm rõ lầm tưởng về việc tạo Thread bằng cách phân tích vòng đời của Môi trường thực thi (Execution Environment), phân biệt hiện tượng Khởi động lạnh (Cold Start) và Khởi động ấm (Warm Start), đồng thời hướng dẫn cách cấu hình Reserved Concurrency và Provisioned Concurrency để tránh lỗi nghẽn (Throttling) và tối ưu hóa hiệu năng hệ thống.