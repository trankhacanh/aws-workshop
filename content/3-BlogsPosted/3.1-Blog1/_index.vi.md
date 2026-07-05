---
title: "Blog 1"
date: 2026-07-03
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---


# XÂY DỰNG KIẾN TRÚC SAAS ĐA THUÊ BAO DẠNG LAI CHO CÁC DỊCH VỤ LƯU TRẠNG THÁI TRÊN AWS

Thiết kế kiến trúc đa thuê bao (Multi-tenancy) cho các dịch vụ lưu trạng thái (Stateful Services như Server Game, Chat thời gian thực) rất phức tạp. Doanh nghiệp thường bị kẹt giữa mô hình Cô lập (Silo – an toàn nhưng đắt) và Chia sẻ (Pool – rẻ nhưng dễ nghẽn hiệu năng). Giải pháp Kiến trúc Lai (Hybrid) kết hợp cả hai mô hình trên cùng một hệ thống AWS nhằm tối ưu hóa cả chi phí lẫn hiệu năng.

Các điểm chính cần nắm:

* Phân tầng khách hàng linh hoạt: Gom khách hàng nhỏ vào nhóm dùng chung tài nguyên (Pool) để tiết kiệm chi phí, tách riêng khách hàng VIP/Enterprise vào phân vùng độc lập (Silo) để đảm bảo hiệu năng.
* Quản lý trạng thái phân tán: Trạng thái phiên (Session State) được đồng bộ liên tục, giúp người dùng không bị mất dữ liệu hay ngắt quãng trải nghiệm nếu có một máy chủ gặp sự có.
* Lớp Tiếp nhận & Định tuyến Thông minh: Sử dụng Amazon CloudFront, AWS WAF, kết hợp API Gateway hoặc ALB kiểm tra Tenant ID, tra cứu cấu hình động trong Amazon DynamoDB để điều hướng chính xác đến phân vùng tài nguyên.
* Lớp Tính toán & Lưu trữ Trạng thái: Tận dụng cụm Amazon EKS (Namespaces và Node Affinities) để phân chia ranh giới giữa các Tenant, kết hợp Amazon ElastiCache (Redis) làm bộ nhớ đệm tốc độ cao để đồng bộ trạng thái thời gian thực giữa các Pods với độ trễ mili-giây.
* Tối ưu chi phí và vận hành tập trung: Triệt tiêu tối đa chi phí tài nguyên nhàn rỗi nhờ mô hình Pool, bảo vệ khách hàng VIP khỏi hiện tượng "Noisy Neighbor", và quản lý mọi Tenant linh hoạt trên một cụm EKS duy nhất.
* Thách thức co giãn (Auto-scaling): Do kết nối luôn giữ trạng thái (Sticky Sessions), việc giảm số lượng Pods (Scale-down) rất dễ làm ngắt kết nối người dùng. Dev phải tự viết thêm logic "di cư" dữ liệu trạng thái (State Migration) an toàn trước khi hủy Pod.

Cơ chế dùng DynamoDB làm "Routing Table" động giúp nâng cấp khách hàng từ gói thường lên gói VIP chỉ bằng cách thay đổi cấu hình dữ liệu, không cần sửa code hay sửa hạ tầng. Hãy thiết kế lớp Định tuyến (Routing Layer) thật lỏng lẻo (decoupled) ngay từ đầu để thoải mái tùy biến hạ tầng bên dưới mà không làm ảnh hưởng tới khách hàng.

![Sơ đồ kiến trúc Hybrid Multi-Tenant cho Stateful Services](/images/3.1-Blog1/Picture1.png)

### LINK THAM KHẢO
* [AWS Architecture Blog - Building a Hybrid Multi-Tenant Architecture for Stateful Services on AWS](https://aws.amazon.com/blogs/architecture/building-hybrid-multi-tenant-architecture-for-stateful-services-on-aws/)

### HƯỚNG DẪN TRIỂN KHAI VÀ PHÂN TÍCH CHI TIẾT

Dựa trên sơ đồ kiến trúc hệ thống, quy trình thiết kế và vận hành lớp định tuyến cùng lớp lưu trữ trạng thái cần tập trung vào các thành phần cốt lõi sau:

1. **Xây dựng Lớp Định tuyến Thống nhất (Unified Endpoint):**
   * Sử dụng **Amazon Route 53** kết hợp cấu hình định tuyến linh hoạt (**Headers-based routing** hoặc **Weighted Routing**) để tiếp nhận traffic từ các Tenant (Tenant 1, Tenant 2, Tenant 3) đổ về hệ thống.
   * Traffic sau khi đi qua DNS Management sẽ được định tuyến trực tiếp đến các nhóm hạ tầng tương ứng (**Infra group 1**, **Infra group 2**) nằm trong các phân vùng VPC (Tier 1-Cell-1).

2. **Cấu hình Định tuyến tại Application Load Balancer (ALB):**
   * Tại mỗi Infra group, cấu hình ALB phân tích HTTP Headers hoặc các tham số định danh để nhận diện Tenant ID.
   * Dựa trên kết quả định danh, ALB sẽ điều hướng các kết nối của khách hàng thuộc nhóm dùng chung (Pool) vào các Container dùng chung, hoặc hướng các kết nối của khách hàng VIP/Enterprise vào các Pods biệt lập (Silo) đã được cấu hình sẵn.

3. **Quản lý Tập trung và Giám sát với CloudWatch:**
   * Toàn bộ dữ liệu logs, metrics vận hành của các Infra group phải được cấu hình xuất ra (**Export Metrics**) và tập trung tại một AWS Account quản trị (Tier 1) thông qua **Amazon CloudWatch Dashboards**.
   * Việc này giúp DevOps theo dõi được chi tiết hiệu năng và phát hiện sớm các hiện tượng nghẽn tài nguyên do hiệu ứng "Noisy Neighbor" gây ra ở nhóm Pool.

4. **Kết nối Bảo mật và Lưu trữ Trạng thái Phiên (Session State):**
   * Giữa cụm tính toán tính toán (EKS Pods) và lớp bộ nhớ đệm (Cache), thiết lập một đường truyền mạng an toàn, biệt lập bằng **AWS PrivateLink**.
   * Kết nối này dẫn trực tiếp sang tài khoản Cache chuyên biệt (**AWS Account Tier 1 Cache**) - nơi vận hành cụm **Amazon ElastiCache (Redis)**. Tại đây, trạng thái phiên làm việc (Sticky Sessions) của các người dùng trong game hoặc ứng dụng chat chat thời gian thực sẽ được đồng bộ và duy trì liên tục với độ trễ cực thấp dưới 10ms.