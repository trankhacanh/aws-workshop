---
title: "Blog 3"
date: 2026-07-03
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---


# PHÂN TÍCH SÂU VỀ AWS LAMBDA CONCURRENCY – ĐIỀU GÌ THỰC SỰ XẢY RA KHI HÀNG NGHÌN REQUEST CÙNG LÚC GỌI MỘT LAMBDA?

Khi làm việc với các ứng dụng Serverless, khả năng tự động mở rộng của AWS Lambda phụ thuộc vào một cơ chế cốt lõi được gọi là Concurrency (Số lượng thực thi đồng thời). Việc hiểu rõ cách quản lý Môi trường thực thi (Execution Environment) cùng các trạng thái Cold Start/Warm Start sẽ giúp lập trình viên thiết kế hệ thống chịu tải cao một cách hiệu quả, tránh các lỗi nghẽn hoặc từ chối dịch vụ (Throttling).

Các điểm chính cần nắm:

* **Bản chất của Concurrency:** Là số lượng request mà AWS Lambda có thể xử lý tại cùng một thời điểm bằng cách tạo ra nhiều môi trường thực thi song song thay vì xếp hàng chạy lần lượt.
* **Cơ chế Execution Environment:** Lambda không tạo Thread mới trên cùng một môi trường cho mỗi request. Mỗi môi trường thực thi (gồm Runtime, tài nguyên CPU/RAM, mã nguồn) chỉ xử lý duy nhất 1 request tại 1 thời điểm để đảm bảo tính độc lập dữ liệu.
* **Hiện tượng Cold Start:** Xảy ra khi Lambda nhận request đầu tiên hoặc khi môi trường cũ đã bị hủy. AWS phải chuẩn bị tài nguyên, khởi tạo Runtime, tải code và chạy phần mã khởi tạo ngoài Handler, làm tăng thời gian phản hồi của request.
* **Tối ưu hóa với Warm Start:** Sau khi xử lý xong, AWS giữ lại môi trường trong một khoảng thời gian. Request tiếp theo đến cấu hình này sẽ bỏ qua bước thiết lập ban đầu, thực thi trực tiếp hàm xử lý (Handler) với tốc độ phản hồi cực nhanh.
* **Reserved Concurrency:** Tính năng dành riêng một hạn mức tài nguyên cố định cho một hàm Lambda cụ thể, đảm bảo nó luôn có tài nguyên chạy và không bị các Lambda khác trong cùng tài khoản chiếm dụng hết Concurrency.
* **Provisioned Concurrency:** Giữ sẵn một số lượng môi trường thực thi luôn ở trạng thái khởi tạo hoàn chỉnh (Pre-warmed), giúp triệt tiêu hoàn toàn hiện tượng Cold Start cho các API yêu cầu độ trễ cực thấp nhưng sẽ phát sinh thêm chi phí duy trì.

![Sơ đồ tổng quan cơ chế Concurrent Execution của AWS Lambda](/images/3.3-Blog3/Picture.png)
<p style="text-align: center;"><i>Hình 3.1: Sơ đồ tổng quan cơ chế Concurrent Execution của AWS Lambda.</i></p>

### LINK THAM KHẢO DỊCH VỤ CỐT LÕI (AWS OFFICIAL DOCUMENTATION)
* [AWS Lambda Developer Guide - Managing Lambda Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html)
* [AWS Lambda Operator Guide - Understanding Execution Environment Lifecycle](https://docs.aws.amazon.com/lambda/latest/operatorguide/execution-environments.html)

### HƯỚNG DẪN TRIỂN KHAI VÀ PHÂN TÍCH CHI TIẾT

![Sơ đồ chi tiết thuật toán phân nhánh Cold Start và Warm Start của AWS Lambda](/images/3.3-Blog3/PictureDetails.png)
<p style="text-align: center;"><i>Hình 3.2: Sơ đồ chi tiết thuật toán kiểm tra môi trường và phân nhánh Cold Start / Warm Start.</i></p>

Dựa trên các sơ đồ minh họa quy trình vận hành, luồng xử lý đồng thời và tối ưu tài nguyên của AWS Lambda được phân tích cụ thể như sau:

1. **Tiếp nhận Request đồng thời (Incoming Requests Execution):**
   * Theo **Sơ đồ tổng quan về cơ chế hoạt động (Hình 3.1)**, khi người dùng hoặc ứng dụng client (Web/Mobile) gửi hàng loạt yêu cầu đồng thời (Req_1, Req_2, ..., Req_N) qua **Amazon CloudFront** và **Amazon API Gateway**, AWS Lambda Service sẽ phân tích tải lượng ngay lập tức.
   * Hệ thống sẽ tự động scale-out theo chiều ngang bằng cách phân phối mỗi request độc lập vào một **Môi trường thực thi riêng biệt 1 đến N (Execution Environment 1 đến N)**. Quy trình này loại bỏ hoàn toàn việc thiết lập Load Balancer thủ công.

2. **Kiểm tra và Phân nhánh Trạng thái Môi trường (Execution Flow):**
   * Theo **Sơ đồ chi tiết thuật toán phân nhánh (Hình 3.2)**, khi request đổ vào, Lambda Service sẽ đánh giá trạng thái hiện tại: *Execution Environment Available?*
   * **Nhánh YES (Warm Start):** Hệ thống điều hướng trực tiếp để tái sử dụng môi trường sẵn có (*Reuse Existing Execution Environment*) và gọi thẳng hàm xử lý của người dùng (*Execute Handler*). Để tận dụng tối đa cơ chế này, lập trình viên nên khởi tạo các kết nối cơ sở dữ liệu hoặc thư viện nặng ở **bên ngoài hàm Handler**.
   * **Nhánh NO (Cold Start):** Hệ thống bắt buộc phải tạm dừng luồng để kích hoạt quy trình khởi động tuần tự bao gồm: Cấp phát tài nguyên (Provision Resources) -> Khởi tạo môi trường chạy (Init Runtime) -> Tải mã nguồn ứng dụng (Load Code).

3. **Tương tác Dịch vụ Backend và Trả kết quả (Execution & Access):**
   * Sau khi môi trường thực thi đã sẵn sàng (thông qua một trong hai nhánh được biểu diễn trên **Hình 3.2**), hàm xử lý chính được kích hoạt (**Execute Function Handler**) để xử lý các logic nghiệp vụ và tương tác với các dịch vụ AWS Backend khác trên **Hình 3.1** như **Amazon DynamoDB**, **Amazon S3**, hoặc đẩy gói tin vào hàng đợi **Amazon SQS**.
   * Kết thúc quá trình tính toán, dữ liệu phản hồi (**Return Response**) được tổng hợp và trả ngược lại cho client thông qua proxy của API Gateway.

4. **Giám sát chống nghẽn (Observability & Monitoring Layer):**
   * Toàn bộ các chỉ số vận hành cốt lõi của hệ thống như số lượng thực thi đồng thời (*Concurrent Executions*), thời gian chạy (*Duration*), và lỗi nghẽn (*Throttles*) liên tục được đẩy trực tiếp và tập trung về **Amazon CloudWatch**.
   * Kỹ sư DevOps cần thiết lập cụ thể các chỉ số **CloudWatch Alarms** kết hợp với **Amazon SNS** hoặc cấu hình kiểm vết phân tán **AWS X-Ray (Tracing)** nhằm phát hiện sớm hiện tượng tài khoản chạm ngưỡng giới hạn Concurrency. Từ đó có phương án nâng quota kịp thời hoặc áp dụng hàng đợi SQS điều tiết tải khi lưu lượng tăng đột biến.