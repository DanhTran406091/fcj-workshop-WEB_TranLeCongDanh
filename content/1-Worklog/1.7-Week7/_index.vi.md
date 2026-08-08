---
title: "Worklog Tuần 7"
date: 2026-08-03
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Thời gian:

**Từ ngày 03/08/2026 đến ngày 07/08/2026**

### Mục tiêu tuần 7:

* Kiểm thử toàn bộ hệ thống Live Auction sau khi triển khai trên AWS.
* Kiểm tra khả năng kết nối giữa frontend và các dịch vụ backend.
* Kiểm thử luồng đăng ký, đăng nhập và phân quyền người dùng.
* Kiểm tra chức năng đấu giá và cập nhật dữ liệu theo thời gian thực.
* Phát hiện và khắc phục các lỗi còn tồn tại.
* Kiểm tra lại cấu hình Terraform, quyền IAM và tài nguyên AWS.
* Tổng duyệt hệ thống trước khi hoàn thiện đồ án.

### Các công việc đã thực hiện:

| Thứ | Ngày | Công việc | Nguồn tài liệu |
| --- | --- | --- | --- |
| Thứ Hai | 03/08/2026 | Xây dựng danh sách trường hợp kiểm thử; kiểm tra giao diện, điều hướng và khả năng truy cập frontend thông qua Amazon CloudFront. | Tài liệu kiểm thử của nhóm |
| Thứ Ba | 04/08/2026 | Kiểm thử luồng đăng ký, xác thực, đăng nhập và phân quyền người dùng thông qua Amazon Cognito; kiểm tra quyền truy cập của các IAM Role. | <https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools.html> |
| Thứ Tư | 05/08/2026 | Kiểm thử REST API và các hàm AWS Lambda; kiểm tra chức năng quản lý sản phẩm, phiên đấu giá, dữ liệu đầu vào và kết quả lưu trữ trên DynamoDB. | <https://docs.aws.amazon.com/lambda/latest/dg/testing-guide.html> |
| Thứ Năm | 06/08/2026 | Kiểm thử kết nối WebSocket và luồng đặt giá theo thời gian thực; kiểm tra khả năng tiếp nhận và xử lý tuần tự yêu cầu thông qua Amazon SQS FIFO. | <https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-websocket-api.html> |
| Thứ Sáu | 07/08/2026 | Khắc phục các lỗi phát hiện trong quá trình kiểm thử; kiểm tra lại Terraform, tổng duyệt luồng hoạt động và đánh giá mức độ hoàn thành của đồ án. | Mã nguồn và tài liệu triển khai của nhóm |

### Kết quả đạt được tuần 7:

* Xây dựng được danh sách các trường hợp kiểm thử cho hệ thống.
* Kiểm tra được khả năng truy cập frontend thông qua Amazon CloudFront.
* Kiểm thử các chức năng xác thực người dùng:

  * Đăng ký tài khoản.
  * Xác nhận tài khoản.
  * Đăng nhập.
  * Đăng xuất.
  * Khôi phục mật khẩu.
  * Kiểm tra quyền truy cập vào các chức năng yêu cầu xác thực.

* Kiểm thử các chức năng chính của hệ thống:

  * Hiển thị danh sách sản phẩm.
  * Xem thông tin chi tiết sản phẩm.
  * Tạo và cập nhật sản phẩm.
  * Tạo và quản lý phiên đấu giá.
  * Tham gia phiên đấu giá.
  * Gửi yêu cầu đặt giá.
  * Xem trạng thái và lịch sử đặt giá.

* Kiểm tra khả năng kết nối giữa các thành phần:

  * Frontend với Amazon Cognito.
  * Frontend với Amazon API Gateway.
  * API Gateway với AWS Lambda.
  * AWS Lambda với Amazon DynamoDB.
  * AWS Lambda với Amazon SQS FIFO.
  * API Gateway WebSocket với các kết nối người dùng.

* Kiểm tra được khả năng cập nhật trạng thái phiên đấu giá theo thời gian thực.
* Kiểm tra luồng tiếp nhận và xử lý yêu cầu đặt giá thông qua hàng đợi SQS FIFO.
* Phát hiện và khắc phục một số nhóm lỗi:

  * Lỗi cấu hình biến môi trường.
  * Lỗi CORS khi frontend gọi API.
  * Lỗi quyền truy cập giữa các dịch vụ AWS.
  * Lỗi xử lý dữ liệu đầu vào của Lambda.
  * Lỗi kết nối hoặc mất kết nối WebSocket.
  * Lỗi hiển thị trạng thái trên giao diện.
  * Lỗi cấu hình tài nguyên trong Terraform.

* Kiểm tra lại các IAM Policy theo nguyên tắc cấp quyền tối thiểu.
* Kiểm tra trạng thái tài nguyên và nhật ký hoạt động trên AWS.
* Tổng duyệt được luồng hoạt động chính của hệ thống từ lúc người dùng đăng nhập đến khi tham gia và đặt giá.
* Xác định được các nội dung cuối cùng cần hoàn thiện trong tuần 8.