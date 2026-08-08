---
title: "Worklog Tuần 6"
date: 2026-07-27
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Thời gian:

**Từ ngày 27/07/2026 đến ngày 31/07/2026**

### Mục tiêu tuần 6:

* Chuẩn bị môi trường triển khai hệ thống trên AWS.
* Tìm hiểu và áp dụng Terraform để quản lý hạ tầng dưới dạng mã nguồn.
* Xây dựng cấu trúc thư mục Infrastructure cho đồ án.
* Triển khai các thành phần chính của kiến trúc serverless lên AWS.
* Tích hợp mã nguồn ứng dụng với các dịch vụ AWS đã triển khai.
* Ghi lại các bước thực hiện để xây dựng nội dung Workshop.

### Các công việc đã thực hiện:

| Thứ | Ngày | Công việc | Nguồn tài liệu |
| --- | --- | --- | --- |
| Thứ Hai | 27/07/2026 | Chuẩn bị môi trường triển khai; cài đặt và kiểm tra AWS CLI, Terraform; cấu hình thông tin xác thực và Region làm việc trên AWS. | <https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli> |
| Thứ Ba | 28/07/2026 | Xây dựng cấu trúc thư mục Infrastructure; khai báo AWS Provider, biến đầu vào, tài nguyên và các giá trị đầu ra trong Terraform. | <https://developer.hashicorp.com/terraform/language> |
| Thứ Tư | 29/07/2026 | Thực hiện `terraform init` và `terraform plan`; kiểm tra cấu hình, xử lý lỗi và xem trước các tài nguyên AWS sẽ được khởi tạo. | <https://developer.hashicorp.com/terraform/cli/commands/init> |
| Thứ Năm | 30/07/2026 | Thực hiện `terraform apply` để triển khai các tài nguyên AWS; kiểm tra IAM, Cognito, S3, CloudFront, Lambda, API Gateway, DynamoDB và SQS FIFO. | <https://developer.hashicorp.com/terraform/cli/commands/apply> |
| Thứ Sáu | 31/07/2026 | Tích hợp frontend và các hàm xử lý với tài nguyên AWS đã triển khai; ghi lại quy trình, câu lệnh và kết quả để xây dựng nội dung Workshop. | Mã nguồn và tài liệu triển khai của nhóm |

### Kết quả đạt được tuần 6:

* Cài đặt và kiểm tra thành công các công cụ cần thiết:
