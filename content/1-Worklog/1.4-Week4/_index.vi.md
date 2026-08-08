---
title: "Worklog Tuần 4"
date: 2026-07-13
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Thời gian:

**Từ ngày 13/07/2026 đến ngày 17/07/2026**

### Mục tiêu tuần 4:

* Lựa chọn và xác định phạm vi đồ án của nhóm.
* Phân tích bài toán đấu giá trực tuyến và các yêu cầu của hệ thống.
* Xác định các chức năng cốt lõi của nền tảng Live Auction.
* Nghiên cứu kiến trúc AWS phù hợp với yêu cầu xử lý theo thời gian thực.
* Lựa chọn các dịch vụ AWS dự kiến sử dụng trong quá trình triển khai.
* Xây dựng bản đề xuất và kế hoạch thực hiện đồ án.

### Các công việc đã thực hiện:

| Thứ | Ngày | Công việc | Nguồn tài liệu |
| --- | --- | --- | --- |
| Thứ Hai | 13/07/2026 | Thảo luận và thống nhất lựa chọn đề tài **Live Auction Platform on AWS**; xác định mục tiêu và phạm vi ban đầu của đồ án. | Tài liệu yêu cầu đồ án |
| Thứ Ba | 14/07/2026 | Phân tích bài toán đấu giá trực tuyến; xác định các đối tượng sử dụng và những chức năng chính như đăng ký, đăng nhập, quản lý sản phẩm, tạo phiên đấu giá và đặt giá. | Tài liệu phân tích yêu cầu của nhóm |
| Thứ Tư | 15/07/2026 | Phân tích các vấn đề cần giải quyết như cập nhật giá theo thời gian thực, xử lý đồng thời nhiều lượt đặt giá, đảm bảo đúng thứ tự và khả năng mở rộng hệ thống. | <https://aws.amazon.com/architecture/> |
| Thứ Năm | 16/07/2026 | Nghiên cứu kiến trúc AWS cho hệ thống; tìm hiểu vai trò của Amazon S3, CloudFront, Cognito, API Gateway, Lambda, DynamoDB và SQS FIFO. | <https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html> |
| Thứ Sáu | 17/07/2026 | Xây dựng sơ đồ kiến trúc đề xuất; thống nhất hướng triển khai bằng Terraform, phân chia nhiệm vụ và hoàn thiện bản đề xuất của đồ án. | <https://developer.hashicorp.com/terraform/tutorials/aws-get-started> |

### Kết quả đạt được tuần 4:

* Thống nhất lựa chọn đề tài **Live Auction Platform on AWS**.
* Xác định mục tiêu xây dựng một nền tảng đấu giá trực tuyến có khả năng:

  * Quản lý tài khoản người dùng.
  * Quản lý sản phẩm đấu giá.
  * Tạo và quản lý phiên đấu giá.
  * Tiếp nhận yêu cầu đặt giá.
  * Cập nhật trạng thái đấu giá theo thời gian thực.
  * Mở rộng theo lượng người dùng truy cập.

* Xác định được các vấn đề chính cần giải quyết:

  * Nhiều người dùng có thể đặt giá trong cùng một thời điểm.
  * Các yêu cầu đặt giá cần được xử lý đúng thứ tự.
  * Trạng thái phiên đấu giá cần được cập nhật kịp thời.
  * Thông tin người dùng và tài nguyên hệ thống cần được bảo vệ.
  * Hệ thống cần có khả năng mở rộng và dễ bảo trì.

* Hoàn thành kiến trúc đề xuất ban đầu cho hệ thống.
* Xác định vai trò dự kiến của các dịch vụ AWS:

  * **Amazon Cognito:** xác thực và quản lý người dùng.
  * **Amazon S3:** lưu trữ frontend và tài nguyên tĩnh.
  * **Amazon CloudFront:** phân phối nội dung đến người dùng.
  * **Amazon API Gateway:** tiếp nhận các yêu cầu API.
  * **API Gateway WebSocket:** truyền dữ liệu đấu giá theo thời gian thực.
  * **AWS Lambda:** xử lý logic nghiệp vụ theo mô hình serverless.
  * **Amazon DynamoDB:** lưu trữ dữ liệu phục vụ xử lý nhanh và theo thời gian thực.
  * **Amazon SQS FIFO:** tiếp nhận và xử lý yêu cầu đặt giá theo đúng thứ tự.
  * **AWS IAM:** quản lý vai trò và quyền truy cập giữa các dịch vụ.

* Thống nhất sử dụng **Terraform** để quản lý và triển khai hạ tầng AWS dưới dạng mã nguồn.
* Hoàn thành bản đề xuất làm cơ sở cho quá trình phát triển và triển khai trong các tuần tiếp theo.