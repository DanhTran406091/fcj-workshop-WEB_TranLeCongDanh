---
title: "Kiểm tra kế hoạch triển khai"
date: 2026-07-13
weight: 4
chapter: false
pre: "<b>5.3.4. </b>"
---

## Giới thiệu sơ bộ

Sau khi hoàn tất quá trình khởi tạo môi trường Terraform, nhóm tiến hành kiểm tra cấu hình và lập kế hoạch triển khai bằng lệnh `terraform plan`.

Lệnh này phân tích các tệp cấu hình Terraform, đọc trạng thái hiện tại của hạ tầng và so sánh với cấu hình mong muốn. Kết quả cho biết những tài nguyên nào sẽ được tạo mới, cập nhật hoặc xóa trước khi nhóm thực sự triển khai lên AWS.

Việc kiểm tra kế hoạch giúp nhóm phát hiện sớm lỗi cấu hình, quyền truy cập không hợp lệ hoặc các thay đổi ngoài dự kiến, từ đó hạn chế rủi ro ảnh hưởng đến hạ tầng đang hoạt động.

---

## Kiểm tra định dạng mã Terraform

Trước khi tạo kế hoạch triển khai, nhóm kiểm tra định dạng của các tệp Terraform:

```powershell
terraform fmt -check
```

Nếu các tệp chưa đúng định dạng, sử dụng:

```powershell
terraform fmt
```

Lệnh `terraform fmt` tự động chuẩn hóa cách thụt dòng, khoảng trắng và cách trình bày các khối cấu hình Terraform.

Sau khi định dạng, có thể kiểm tra lại:

```powershell
terraform fmt -check
```

Nếu lệnh không trả về lỗi, các tệp cấu hình đã có định dạng hợp lệ.

---

## Kiểm tra cấu hình Terraform

Tại thư mục module đã được khởi tạo, chạy:

```powershell
terraform validate
```

Lệnh `terraform validate` kiểm tra:

* Cú pháp của các tệp `.tf`.
* Tên và kiểu dữ liệu của biến.
* Các thuộc tính bắt buộc của tài nguyên.
* Cách tham chiếu giữa các tài nguyên.
* Cấu hình Provider và module.
* Tính nhất quán của cấu hình Terraform.

Khi cấu hình hợp lệ, Terraform hiển thị:

```text
Success! The configuration is valid.
```

<!--
HƯỚNG DẪN CHỤP ẢNH:
1. Mở Terminal tại infra/03-identity.
2. Chạy: terraform validate
3. Chụp kết quả có dòng:
   Success! The configuration is valid.
4. Lưu ảnh tại:
static/images/5-Workshop/5.3-Infrastructure/terraform-validate-success.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-validate-success.png" alt="Terraform Validate thành công" width="80%">
    <figcaption style="text-align: center;">
        <b>Hình 5.3.15.</b> Kiểm tra cấu hình Terraform thành công bằng lệnh <code>terraform validate</code>.
    </figcaption>
</figure>

{{% notice warning %}}
Lệnh `terraform validate` chỉ kiểm tra cú pháp và tính nhất quán của cấu hình. Lệnh này không xác nhận rằng tài khoản AWS có đủ quyền để tạo tài nguyên.
{{% /notice %}}

---

## Tạo kế hoạch triển khai

Sau khi cấu hình hợp lệ, chạy lệnh:

```powershell
terraform plan
```

Terraform sẽ:

1. Đọc các tệp cấu hình trong module.
2. Đọc giá trị của các biến đầu vào.
3. Kết nối đến Terraform Backend.
4. Đọc Terraform State hiện tại.
5. Truy vấn trạng thái tài nguyên trên AWS.
6. So sánh trạng thái hiện tại với cấu hình mong muốn.
7. Hiển thị kế hoạch thay đổi hạ tầng.

Trong kết quả của `terraform plan`, Terraform sử dụng các ký hiệu:

| Ký hiệu | Ý nghĩa |
| --- | --- |
| `+` | Tài nguyên sẽ được tạo mới. |
| `~` | Tài nguyên sẽ được cập nhật tại chỗ. |
| `-` | Tài nguyên sẽ bị xóa. |
| `-/+` | Tài nguyên sẽ bị xóa và tạo lại. |
| `<=` | Dữ liệu sẽ được đọc từ Data Source. |

Ví dụ kết quả:

```text
Terraform will perform the following actions:

  # aws_cognito_user_pool.live_auction will be created
  + resource "aws_cognito_user_pool" "live_auction" {
      + id   = (known after apply)
      + name = "live-auction-user-pool"
    }

Plan: 3 to add, 0 to change, 0 to destroy.
```

Dòng tổng kết cho biết:

* `3 to add`: ba tài nguyên sẽ được tạo.
* `0 to change`: không có tài nguyên nào được cập nhật.
* `0 to destroy`: không có tài nguyên nào bị xóa.

<!--
HƯỚNG DẪN CHỤP ẢNH:
1. Chạy terraform plan tại module 03-identity.
2. Cuộn xuống cuối kết quả.
3. Chụp phần có dòng tổng kết:
   Plan: ... to add, ... to change, ... to destroy.
4. Không chụp Access Key, Secret Key hoặc dữ liệu nhạy cảm.
5. Lưu ảnh tại:
static/images/5-Workshop/5.3-Infrastructure/terraform-plan-summary.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-plan-summary.png" alt="Kết quả Terraform Plan" width="90%">
    <figcaption style="text-align: center;">
        <b>Hình 5.3.16.</b> Kết quả kế hoạch triển khai của module Identity.
    </figcaption>
</figure>

---

## Sử dụng tệp biến đầu vào

Nếu module sử dụng tệp biến riêng, có thể truyền tệp đó vào lệnh `terraform plan`:

```powershell
terraform plan -var-file="terraform.tfvars"
```

Ví dụ nội dung của `terraform.tfvars`:

```hcl
aws_region  = "ap-southeast-1"
environment = "dev"
project_name = "live-auction"
```

Tệp biến giúp tách các giá trị cấu hình khỏi mã nguồn chính và thuận tiện khi triển khai nhiều môi trường khác nhau.

Ví dụ:

```text
terraform.dev.tfvars
terraform.staging.tfvars
terraform.prod.tfvars
```

Khi lập kế hoạch cho môi trường phát triển:

```powershell
terraform plan -var-file="terraform.dev.tfvars"
```

{{% notice warning %}}
Không lưu Access Key, Secret Access Key, mật khẩu hoặc thông tin nhạy cảm trong tệp `.tfvars` được đẩy lên Git. Các giá trị bí mật nên được cung cấp thông qua biến môi trường hoặc dịch vụ quản lý bí mật phù hợp.
{{% /notice %}}

---

## Lưu kế hoạch vào tệp

Sau khi kiểm tra kết quả, nhóm lưu kế hoạch vào tệp `tfplan`:

```powershell
terraform plan -out="tfplan"
```

Tùy chọn `-out` giúp lưu lại chính xác kế hoạch đã được Terraform tạo. Tệp này có thể được sử dụng ở bước `terraform apply`, bảo đảm Terraform triển khai đúng những thay đổi đã được nhóm kiểm tra.

Để xem lại nội dung của kế hoạch đã lưu:

```powershell
terraform show tfplan
```

Để xuất kế hoạch dưới dạng JSON:

```powershell
terraform show -json tfplan
```

Quy trình được sử dụng:

```powershell
terraform validate
terraform plan -out="tfplan"
terraform show tfplan
```

<!--
HƯỚNG DẪN CHỤP ẢNH:
1. Chạy: terraform plan -out="tfplan"
2. Sau khi hoàn tất, mở VS Code Explorer.
3. Chụp thư mục module có tệp tfplan.
4. Lưu ảnh tại:
static/images/5-Workshop/5.3-Infrastructure/terraform-plan-file.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-plan-file.png" alt="Tệp kế hoạch Terraform" width="65%">
    <figcaption style="text-align: center;">
        <b>Hình 5.3.17.</b> Tệp <code>tfplan</code> được tạo sau khi lưu kế hoạch triển khai.
    </figcaption>
</figure>

{{% notice info %}}
Tệp `tfplan` ở dạng nhị phân nên không chỉnh sửa thủ công. Tệp này có thể chứa thông tin liên quan đến cấu hình hạ tầng và không nên được đẩy lên kho mã nguồn công khai.
{{% /notice %}}

---

## Lập kế hoạch theo từng module

Do hạ tầng được chia thành nhiều module, nhóm thực hiện `terraform plan` theo thứ tự phụ thuộc của kiến trúc.

### Module Identity

Di chuyển đến module Identity:

```powershell
cd infra\03-identity
```

Kiểm tra và tạo kế hoạch:

```powershell
terraform validate
terraform plan -out="tfplan"
```

Module này lập kế hoạch cho các tài nguyên liên quan đến:

* AWS IAM.
* Amazon Cognito.
* Role và Policy phục vụ xác thực, phân quyền.

### Module Data

```powershell
cd ..\04-data
terraform validate
terraform plan -out="tfplan"
```

Module Data lập kế hoạch cho các bảng Amazon DynamoDB dùng để lưu:

* Dữ liệu sản phẩm.
* Thông tin phiên đấu giá.
* Lịch sử đặt giá.
* Trạng thái đấu giá.
* Thông tin kết nối WebSocket.

### Module Messaging

```powershell
cd ..\05-messaging
terraform validate
terraform plan -out="tfplan"
```

Module Messaging lập kế hoạch cho:

* Amazon SQS FIFO.
* Dead-letter queue nếu được cấu hình.
* Queue Policy và các quyền truy cập liên quan.

### Module Compute

```powershell
cd ..\06-compute
terraform validate
terraform plan -out="tfplan"
```

Module Compute lập kế hoạch cho:

* AWS Lambda Functions.
* Lambda Layer nếu có.
* IAM Role dành cho Lambda.
* Environment Variables.
* Event Source Mapping giữa Lambda và SQS.

### Module API

```powershell
cd ..\07-api
terraform validate
terraform plan -out="tfplan"
```

Module API lập kế hoạch cho:

* Amazon API Gateway.
* REST API routes.
* API Gateway WebSocket.
* Lambda Integration.
* Authorizer và các Stage triển khai.

### Module Edge

```powershell
cd ..\09-edge
terraform validate
terraform plan -out="tfplan"
```

Module Edge lập kế hoạch cho:

* Amazon S3.
* Amazon CloudFront.
* Origin Access Control.
* Bucket Policy.
* Cấu hình phân phối frontend.

---

## Kiểm tra kế hoạch trước khi triển khai

Trước khi chấp nhận kế hoạch, nhóm kiểm tra các nội dung sau:

### Tên tài nguyên

Tên của Bucket, bảng DynamoDB, Lambda Function, API và Queue phải đúng quy ước của dự án.

### AWS Region

Kiểm tra tất cả tài nguyên được tạo tại Region dự kiến:

```text
ap-southeast-1
```

### Quyền IAM

IAM Policy cần tuân theo nguyên tắc cấp quyền tối thiểu, chỉ cung cấp những quyền cần thiết cho từng dịch vụ.

### Tài nguyên bị xóa hoặc thay thế

Nếu kế hoạch xuất hiện:

```text
Plan: 0 to add, 0 to change, 1 to destroy.
```

hoặc ký hiệu:

```text
-/+
```

nhóm cần kiểm tra kỹ trước khi tiếp tục. Việc thay thế tài nguyên có thể làm gián đoạn hệ thống hoặc mất dữ liệu nếu cấu hình không phù hợp.

### Giá trị đầu ra

Kiểm tra các Output dự kiến như:

* Cognito User Pool ID.
* Cognito Client ID.
* DynamoDB table names.
* SQS Queue URL.
* REST API endpoint.
* WebSocket endpoint.
* S3 Bucket name.
* CloudFront domain name.

---

## Kế hoạch không có thay đổi

Nếu hạ tầng hiện tại đã khớp với cấu hình Terraform, kết quả hiển thị:

```text
No changes. Your infrastructure matches the configuration.
```

Thông báo này cho biết:

* Terraform State khớp với cấu hình.
* Không có tài nguyên cần tạo mới.
* Không có tài nguyên cần cập nhật.
* Không có tài nguyên cần xóa.

Kết quả này thường xuất hiện khi chạy lại `terraform plan` sau khi hạ tầng đã được triển khai thành công và cấu hình không thay đổi.

---

## Một số lỗi thường gặp

### Terraform chưa được khởi tạo

Thông báo lỗi:

```text
Backend initialization required
```

Khởi tạo lại module:

```powershell
terraform init
```

Nếu Backend vừa được thay đổi:

```powershell
terraform init -reconfigure
```

### Thiếu giá trị biến đầu vào

Thông báo lỗi:

```text
No value for required variable
```

Cung cấp giá trị trực tiếp:

```powershell
terraform plan -var="project_name=live-auction"
```

Hoặc sử dụng tệp biến:

```powershell
terraform plan -var-file="terraform.tfvars"
```

### Không đủ quyền AWS

Thông báo:

```text
AccessDenied
```

Kiểm tra danh tính đang sử dụng:

```powershell
aws sts get-caller-identity
```

Sau đó kiểm tra IAM Policy tương ứng.

### Tài nguyên đã tồn tại

Nếu tài nguyên đã được tạo thủ công nhưng chưa nằm trong Terraform State, Terraform có thể báo lỗi khi áp dụng kế hoạch.

Cần kiểm tra tài nguyên hiện có trên AWS và cân nhắc nhập tài nguyên vào State bằng `terraform import` thay vì tạo lại.

### Terraform State đang bị khóa

Khi một tiến trình khác đang cập nhật State, Terraform có thể báo lỗi khóa State.

Không nên tự ý xóa khóa khi chưa xác định tiến trình khác đã kết thúc. Việc nhiều thành viên cùng triển khai một module có thể gây xung đột trạng thái.

---

## Kết quả

Sau khi hoàn tất bước lập kế hoạch:

* Các tệp Terraform đã được chuẩn hóa bằng `terraform fmt`.
* Cấu hình của từng module đã được kiểm tra bằng `terraform validate`.
* Terraform đã so sánh cấu hình mong muốn với trạng thái hạ tầng hiện tại.
* Nhóm đã kiểm tra số lượng tài nguyên được tạo, cập nhật hoặc xóa.
* Kế hoạch triển khai của từng module đã được lưu vào tệp `tfplan`.
* Các thay đổi có nguy cơ ảnh hưởng đến hạ tầng đã được kiểm tra trước.
* Các module đã sẵn sàng cho bước triển khai bằng `terraform apply`.