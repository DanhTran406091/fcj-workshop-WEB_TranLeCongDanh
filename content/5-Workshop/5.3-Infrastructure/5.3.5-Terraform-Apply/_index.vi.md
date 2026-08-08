---
title: "Triển khai hạ tầng"
date: 2026-07-13
weight: 5
chapter: false
pre: "<b>5.3.5. </b>"
---


## Giới thiệu

Sau khi kiểm tra và xác nhận kế hoạch triển khai, nhóm tiến hành tạo các tài nguyên AWS bằng lệnh `terraform apply`.

Lệnh này áp dụng những thay đổi đã được xác định trong kế hoạch Terraform, bao gồm tạo mới, cập nhật hoặc xóa tài nguyên để trạng thái thực tế trên AWS khớp với cấu hình trong mã nguồn.

Trong hệ thống Live Auction, hạ tầng được chia thành nhiều module độc lập. Các module được triển khai theo thứ tự phụ thuộc nhằm bảo đảm tài nguyên nền tảng đã sẵn sàng trước khi các thành phần phía sau được tạo.

Thứ tự triển khai gồm:

1. Identity.
2. Data.
3. Messaging.
4. Compute.
5. API.
6. Edge.

---

## Kiểm tra kế hoạch đã lưu

Trước khi triển khai, kiểm tra tệp `tfplan` trong module:

```powershell
Get-ChildItem
```

Xem lại nội dung kế hoạch:

```powershell
terraform show tfplan
```

Nhóm cần kiểm tra kỹ:

* Số lượng tài nguyên được tạo mới.
* Các tài nguyên được cập nhật.
* Các tài nguyên bị xóa hoặc thay thế.
* Tên và AWS Region của tài nguyên.
* IAM Role và IAM Policy.
* Các giá trị đầu ra dự kiến.

{{% notice warning %}}
Không tiếp tục triển khai nếu kế hoạch xuất hiện tài nguyên bị xóa hoặc thay thế ngoài dự kiến. Cần kiểm tra lại cấu hình và chạy lại `terraform plan`.
{{% /notice %}}

---

## Thực hiện Terraform Apply

Nếu kế hoạch đã được lưu bằng:

```powershell
terraform plan -out="tfplan"
```

áp dụng chính xác kế hoạch đó bằng lệnh:

```powershell
terraform apply "tfplan"
```

Khi sử dụng tệp `tfplan`, Terraform không yêu cầu xác nhận lại vì kế hoạch đã được tạo và lưu từ trước.

Nếu không sử dụng tệp kế hoạch, có thể chạy trực tiếp:

```powershell
terraform apply
```

Terraform sẽ hiển thị kế hoạch và yêu cầu xác nhận:

```text
Do you want to perform these actions?

  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```

Nhập:

```text
yes
```

để bắt đầu triển khai.

{{% notice info %}}
Nên sử dụng `terraform plan -out="tfplan"` và `terraform apply "tfplan"` để bảo đảm Terraform chỉ triển khai những thay đổi đã được nhóm kiểm tra.
{{% /notice %}}

---

## Triển khai module Identity

Module Identity được triển khai đầu tiên vì các module phía sau cần sử dụng IAM Role, IAM Policy và thông tin xác thực người dùng.

Di chuyển đến thư mục module:

```powershell
cd infra\03-identity
```

Kiểm tra lại kế hoạch:

```powershell
terraform show tfplan
```

Triển khai module:

```powershell
terraform apply "tfplan"
```

Module này tạo các tài nguyên chính:

* Amazon Cognito User Pool.
* Amazon Cognito User Pool Client.
* Các IAM Role cần thiết.
* IAM Policy phục vụ Lambda và các dịch vụ AWS.
* Cấu hình xác thực và phân quyền người dùng.

Khi triển khai thành công, Terraform hiển thị:

```text
Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
```

Số lượng tài nguyên thực tế có thể khác tùy theo cấu hình của dự án.

<!--
HƯỚNG DẪN CHỤP ẢNH:
1. Chạy terraform apply "tfplan" trong infra/03-identity.
2. Chờ quá trình triển khai hoàn tất.
3. Chụp phần cuối Terminal có dòng:
   Apply complete! Resources: ... added, ... changed, ... destroyed.
4. Lưu ảnh tại:
static/images/5-Workshop/5.3-Infrastructure/terraform-apply-identity.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-apply-identity.png" alt="Triển khai module Identity" width="90%">
    <figcaption style="text-align: center;">
        <b>Hình 5.3.18.</b> Kết quả triển khai module Identity bằng Terraform.
    </figcaption>
</figure>

---

## Triển khai module Data

Sau khi module Identity hoàn tất, chuyển đến module Data:

```powershell
cd ..\04-data
```

Nếu kế hoạch chưa được tạo hoặc cấu hình vừa thay đổi, chạy lại:

```powershell
terraform init
terraform validate
terraform plan -out="tfplan"
```

Triển khai module:

```powershell
terraform apply "tfplan"
```

Module Data tạo các bảng Amazon DynamoDB dùng để lưu trữ dữ liệu của hệ thống, bao gồm:

* Thông tin sản phẩm.
* Thông tin phiên đấu giá.
* Trạng thái phiên đấu giá.
* Lịch sử đặt giá.
* Thông tin kết nối WebSocket.
* Các dữ liệu nghiệp vụ liên quan.

Sau khi triển khai, kiểm tra các giá trị đầu ra:

```powershell
terraform output
```

<!--
HƯỚNG DẪN CHỤP ẢNH:
1. Triển khai module 04-data.
2. Chụp kết quả Apply complete hoặc terraform output.
3. Không để lộ dữ liệu nhạy cảm.
4. Lưu ảnh tại:
static/images/5-Workshop/5.3-Infrastructure/terraform-apply-data.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-apply-data.png" alt="Triển khai module Data" width="90%">
    <figcaption style="text-align: center;">
        <b>Hình 5.3.19.</b> Kết quả triển khai module Data.
    </figcaption>
</figure>

---

## Triển khai module Messaging

Chuyển đến module Messaging:

```powershell
cd ..\05-messaging
```

Kiểm tra và triển khai:

```powershell
terraform init
terraform validate
terraform plan -out="tfplan"
terraform apply "tfplan"
```

Module này tạo các tài nguyên phục vụ hàng đợi thông điệp:

* Amazon SQS FIFO Queue.
* Dead-letter queue nếu được cấu hình.
* Queue Policy.
* Quyền gửi và nhận thông điệp.
* Cấu hình xử lý yêu cầu đặt giá theo thứ tự.

Kiểm tra các giá trị đầu ra:

```powershell
terraform output
```

Các Output có thể bao gồm:

* SQS Queue URL.
* SQS Queue ARN.
* Dead-letter Queue ARN.

---

## Triển khai module Compute

Chuyển đến module Compute:

```powershell
cd ..\06-compute
```

Khởi tạo và kiểm tra cấu hình nếu cần:

```powershell
terraform init
terraform validate
terraform plan -out="tfplan"
```

Triển khai module:

```powershell
terraform apply "tfplan"
```

Module Compute triển khai các thành phần xử lý nghiệp vụ:

* AWS Lambda Functions.
* IAM Role dành cho Lambda.
* Environment Variables.
* Lambda Permission.
* Event Source Mapping giữa Amazon SQS và AWS Lambda.
* Các thành phần xử lý REST API và WebSocket.

Trong quá trình triển khai, mã nguồn Lambda được đóng gói và tải lên AWS. Terraform sau đó cấu hình Runtime, Handler, bộ nhớ, thời gian thực thi và các biến môi trường tương ứng.

<!--
HƯỚNG DẪN CHỤP ẢNH:
1. Triển khai module 06-compute.
2. Chụp phần kết quả có các tài nguyên aws_lambda_function.
3. Chụp thêm dòng Apply complete ở cuối.
4. Lưu ảnh tại:
static/images/5-Workshop/5.3-Infrastructure/terraform-apply-compute.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-apply-compute.png" alt="Triển khai module Compute" width="90%">
    <figcaption style="text-align: center;">
        <b>Hình 5.3.20.</b> Triển khai các AWS Lambda Function của hệ thống.
    </figcaption>
</figure>

---

## Triển khai module API

Sau khi các Lambda Function đã sẵn sàng, chuyển đến module API:

```powershell
cd ..\07-api
```

Thực hiện:

```powershell
terraform init
terraform validate
terraform plan -out="tfplan"
terraform apply "tfplan"
```

Module API triển khai:

* Amazon API Gateway.
* REST API.
* Các Route và Method.
* Lambda Integration.
* Cognito Authorizer.
* API Gateway WebSocket.
* Các WebSocket Route như `$connect`, `$disconnect` và `$default`.
* Stage triển khai.
* Quyền cho phép API Gateway gọi Lambda.

Sau khi triển khai, xem các Endpoint:

```powershell
terraform output
```

Kết quả có thể bao gồm:

```text
rest_api_endpoint      = "https://example.execute-api.ap-southeast-1.amazonaws.com/dev"
websocket_api_endpoint = "wss://example.execute-api.ap-southeast-1.amazonaws.com/dev"
```

{{% notice warning %}}
Endpoint trong ví dụ chỉ dùng để minh họa. Sử dụng Endpoint được Terraform trả về trong môi trường triển khai thực tế.
{{% /notice %}}

<!--
HƯỚNG DẪN CHỤP ẢNH:
1. Chạy terraform output sau khi triển khai module 07-api.
2. Chụp REST API endpoint và WebSocket endpoint.
3. Có thể che một phần API ID nếu cần.
4. Lưu ảnh tại:
static/images/5-Workshop/5.3-Infrastructure/terraform-api-outputs.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-api-outputs.png" alt="Terraform API Outputs" width="85%">
    <figcaption style="text-align: center;">
        <b>Hình 5.3.21.</b> REST API Endpoint và WebSocket Endpoint sau khi triển khai.
    </figcaption>
</figure>

---

## Triển khai module Edge

Module Edge được triển khai sau khi API và các thành phần backend đã sẵn sàng.

Chuyển đến module:

```powershell
cd ..\09-edge
```

Kiểm tra và triển khai:

```powershell
terraform init
terraform validate
terraform plan -out="tfplan"
terraform apply "tfplan"
```

Module Edge triển khai:

* S3 Bucket lưu frontend.
* S3 Bucket Policy.
* Amazon CloudFront Distribution.
* Origin Access Control.
* Cấu hình Default Root Object.
* Cấu hình phân phối nội dung tĩnh.
* Các Output liên quan đến Bucket và CloudFront.

Sau khi triển khai, kiểm tra Output:

```powershell
terraform output
```

Ví dụ:

```text
frontend_bucket_name   = "live-auction-frontend-example"
cloudfront_domain_name = "example.cloudfront.net"
```

<!--
HƯỚNG DẪN CHỤP ẢNH:
1. Chạy terraform output trong module 09-edge.
2. Chụp tên S3 Bucket và CloudFront Domain.
3. Lưu ảnh tại:
static/images/5-Workshop/5.3-Infrastructure/terraform-edge-outputs.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-edge-outputs.png" alt="Terraform Edge Outputs" width="85%">
    <figcaption style="text-align: center;">
        <b>Hình 5.3.22.</b> Kết quả đầu ra của S3 và CloudFront sau khi triển khai.
    </figcaption>
</figure>

---

## Kiểm tra Terraform Output

Sau khi triển khai từng module, sử dụng:

```powershell
terraform output
```

Để xem một Output cụ thể:

```powershell
terraform output rest_api_endpoint
```

Để xuất kết quả dưới dạng JSON:

```powershell
terraform output -json
```

Các Output được sử dụng để:

* Cấu hình frontend.
* Cấu hình biến môi trường cho Lambda.
* Kết nối API Gateway với ứng dụng.
* Kết nối WebSocket.
* Kiểm tra tên và ARN của tài nguyên.
* Truyền dữ liệu giữa các module.

Nếu Output được đánh dấu là nhạy cảm, Terraform sẽ không hiển thị trực tiếp giá trị trong kết quả thông thường.

{{% notice warning %}}
Không đăng công khai các Output có chứa token, mật khẩu, thông tin xác thực hoặc dữ liệu nhạy cảm.
{{% /notice %}}

---

## Kiểm tra trạng thái Terraform

Sau khi triển khai, kiểm tra danh sách tài nguyên được Terraform quản lý:

```powershell
terraform state list
```

Ví dụ:

```text
aws_cognito_user_pool.live_auction
aws_cognito_user_pool_client.web_client
aws_iam_role.lambda_execution_role
```

Xem chi tiết một tài nguyên:

```powershell
terraform state show aws_cognito_user_pool.live_auction
```

Lệnh `terraform state list` giúp xác nhận rằng các tài nguyên đã được ghi nhận trong Terraform State.

{{% notice warning %}}
Không chỉnh sửa trực tiếp tệp Terraform State. Thao tác sai với State có thể khiến Terraform mất khả năng quản lý chính xác hạ tầng.
{{% /notice %}}

---

## Kiểm tra tính đồng bộ sau khi triển khai

Sau khi `terraform apply` hoàn tất, chạy lại:

```powershell
terraform plan
```

Nếu hạ tầng đã khớp với cấu hình, Terraform hiển thị:

```text
No changes. Your infrastructure matches the configuration.
```

Kết quả này xác nhận:

* Tài nguyên trên AWS đã được tạo theo cấu hình.
* Terraform State đã được cập nhật.
* Không còn thay đổi nào chưa được triển khai.
* Mã Terraform và hạ tầng thực tế đang đồng bộ.

<!--
HƯỚNG DẪN CHỤP ẢNH:
1. Sau khi apply thành công, chạy lại terraform plan.
2. Chụp dòng:
   No changes. Your infrastructure matches the configuration.
3. Lưu ảnh tại:
static/images/5-Workshop/5.3-Infrastructure/terraform-no-changes.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-no-changes.png" alt="Terraform không còn thay đổi" width="90%">
    <figcaption style="text-align: center;">
        <b>Hình 5.3.23.</b> Terraform xác nhận hạ tầng đã đồng bộ với cấu hình.
    </figcaption>
</figure>

---

## Một số lỗi thường gặp

### Kế hoạch đã hết hiệu lực

Nếu cấu hình hoặc State thay đổi sau khi tệp `tfplan` được tạo, Terraform có thể báo:

```text
Saved plan is stale
```

Tạo lại kế hoạch:

```powershell
terraform plan -out="tfplan"
```

Sau đó triển khai:

```powershell
terraform apply "tfplan"
```

### Không đủ quyền tạo tài nguyên

Thông báo lỗi:

```text
AccessDenied
```

Kiểm tra danh tính AWS:

```powershell
aws sts get-caller-identity
```

Sau đó kiểm tra IAM Policy của tài khoản đang triển khai.

### Tài nguyên đã tồn tại

Nếu tài nguyên đã được tạo thủ công, Terraform có thể báo lỗi trùng tên.

Cần:

* Kiểm tra tài nguyên hiện có trên AWS.
* Đổi tên tài nguyên nếu phù hợp.
* Hoặc nhập tài nguyên vào State bằng `terraform import`.

### Lỗi phụ thuộc giữa các module

Nếu một module yêu cầu Output của module trước nhưng tài nguyên chưa được tạo, quá trình triển khai có thể thất bại.

Cần triển khai theo đúng thứ tự:

```text
Identity → Data → Messaging → Compute → API → Edge
```

### Lambda không được đóng gói đúng

Kiểm tra:

* Đường dẫn tệp mã nguồn.
* Tên Handler.
* Runtime.
* Các thư viện phụ thuộc.
* Cấu trúc tệp ZIP.
* Giá trị `source_code_hash`.

### CloudFront chưa cập nhật ngay

Sau khi triển khai, CloudFront có thể cần một khoảng thời gian để chuyển sang trạng thái `Deployed`. Chờ quá trình phân phối hoàn tất trước khi kiểm tra frontend.

---

## Hủy tài nguyên trong môi trường thử nghiệm

Nếu cần xóa tài nguyên thử nghiệm, trước tiên xem kế hoạch hủy:

```powershell
terraform plan -destroy
```

Sau khi kiểm tra cẩn thận, có thể thực hiện:

```powershell
terraform destroy
```

Terraform yêu cầu nhập:

```text
yes
```

để xác nhận.

{{% notice danger %}}
`terraform destroy` sẽ xóa các tài nguyên do module quản lý. Không thực hiện lệnh này trên môi trường đang sử dụng nếu chưa sao lưu dữ liệu và xác nhận chính xác phạm vi ảnh hưởng.
{{% /notice %}}

---

## Kết quả

Sau khi hoàn tất quá trình triển khai:

* Các module Terraform đã được áp dụng theo đúng thứ tự phụ thuộc.
* IAM Role, IAM Policy và Amazon Cognito đã được tạo.
* Các bảng Amazon DynamoDB đã được triển khai.
* Amazon SQS FIFO đã được cấu hình cho luồng xử lý yêu cầu đặt giá.
* Các AWS Lambda Function đã được triển khai.
* REST API và WebSocket API đã được tạo trên Amazon API Gateway.
* Amazon S3 và Amazon CloudFront đã được cấu hình cho frontend.
* Các giá trị Output cần thiết đã được lấy từ Terraform.
* Terraform State đã được cập nhật với các tài nguyên mới.
* Kết quả `terraform plan` sau triển khai xác nhận hạ tầng không còn thay đổi.
* Hạ tầng đã sẵn sàng cho bước kiểm tra kết quả triển khai.