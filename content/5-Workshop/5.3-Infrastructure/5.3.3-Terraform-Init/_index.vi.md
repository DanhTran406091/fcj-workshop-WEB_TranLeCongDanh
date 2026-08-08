---
title: "Khởi tạo môi trường Terraform"
date: 2026-07-13
weight: 3
chapter: false
pre: "<b>5.3.3. </b>"
---

## Giới thiệu

Sau khi hoàn thiện cấu trúc thư mục Infrastructure, nhóm tiến hành khởi tạo môi trường làm việc Terraform cho từng module bằng lệnh `terraform init`.

Đây là bước đầu tiên cần thực hiện trước khi sử dụng các lệnh như `terraform validate`, `terraform plan` hoặc `terraform apply`. Lệnh này giúp Terraform chuẩn bị thư mục làm việc, tải Provider cần thiết và thiết lập Backend dùng để lưu trữ trạng thái của hạ tầng.

Trong hệ thống Live Auction, các module Terraform được triển khai riêng biệt theo từng lớp chức năng. Vì vậy, lệnh `terraform init` cần được thực hiện tại thư mục của module tương ứng trước khi lập kế hoạch và triển khai tài nguyên.

---

## Kiểm tra Terraform

Mở PowerShell hoặc Terminal tại thư mục gốc của dự án và kiểm tra phiên bản Terraform:

```powershell
terraform version
```

Nếu Terraform đã được cài đặt thành công, Terminal sẽ hiển thị phiên bản Terraform và nền tảng đang sử dụng.

<!--
HƯỚNG DẪN CHỤP ẢNH:
1. Chạy lệnh: terraform version
2. Chụp cửa sổ Terminal có cả câu lệnh và kết quả.
3. Lưu ảnh tại:
static/images/5-Workshop/5.3-Infrastructure/terraform-version.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-version.png" alt="Kiểm tra phiên bản Terraform" width="75%">
    <figcaption style="text-align: center;">
        <b>Hình 5.3.12.</b> Kiểm tra phiên bản Terraform trên môi trường triển khai.
    </figcaption>
</figure>

{{% notice info %}}
Phiên bản Terraform thực tế có thể khác tùy theo thời điểm cài đặt. Phiên bản đang sử dụng cần đáp ứng yêu cầu được khai báo trong tệp `versions.tf`.
{{% /notice %}}

---

## Kiểm tra kết nối với AWS

Trước khi khởi tạo Terraform, kiểm tra thông tin xác thực và khả năng kết nối đến AWS:

```powershell
aws sts get-caller-identity
```

Nếu AWS CLI đã được cấu hình chính xác, kết quả sẽ trả về các thông tin cơ bản gồm:

* User ID.
* AWS Account ID.
* ARN của IAM User hoặc IAM Role đang sử dụng.

Ví dụ:

```json
{
    "UserId": "EXAMPLEUSERID",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/example-user"
}
```

{{% notice warning %}}
Không đưa Access Key, Secret Access Key hoặc Session Token vào hình ảnh và mã nguồn báo cáo. Nên che một phần Account ID và ARN trước khi sử dụng ảnh chụp.
{{% /notice %}}

---

## Di chuyển đến module cần khởi tạo

Từ thư mục gốc của dự án, di chuyển đến thư mục Infrastructure:

```powershell
cd infra
```

Trong lần khởi tạo đầu tiên, nhóm bắt đầu với module `03-identity`:

```powershell
cd 03-identity
```

Kiểm tra các tệp cấu hình trong module:

```powershell
Get-ChildItem
```

Thư mục module bao gồm các tệp Terraform cơ bản:

```text
backend.tf
main.tf
outputs.tf
providers.tf
variables.tf
versions.tf
```

---

## Khởi tạo Terraform Backend

Dự án sử dụng Terraform Backend để lưu trữ trạng thái hạ tầng từ xa. Cấu hình Backend được khai báo trong tệp `backend.tf` của từng module.

Ví dụ:

```hcl
terraform {
  backend "s3" {
    bucket = "TEN_BUCKET_LUU_TERRAFORM_STATE"
    key    = "identity/terraform.tfstate"
    region = "ap-southeast-1"
  }
}
```

Ý nghĩa của các thuộc tính:

| Thuộc tính | Chức năng |
| --- | --- |
| `bucket` | Tên S3 Bucket dùng để lưu Terraform State. |
| `key` | Đường dẫn của tệp State tương ứng với module. |
| `region` | AWS Region chứa S3 Bucket. |

Việc sử dụng Remote Backend giúp Terraform State không phụ thuộc vào máy tính của một thành viên. Các thành viên trong nhóm có thể sử dụng chung trạng thái hạ tầng trong quá trình phát triển và triển khai.

{{% notice note %}}
Thay `TEN_BUCKET_LUU_TERRAFORM_STATE` bằng tên S3 Bucket thực tế trong tệp cấu hình của nhóm.
{{% /notice %}}

Nếu tài nguyên Remote Backend chưa được tạo, di chuyển đến thư mục `00-bootstrap`:

```powershell
cd ..\00-bootstrap
```

Chạy tập lệnh bootstrap:

```powershell
.\bootstrap-remote-state.ps1
```

Tập lệnh này chuẩn bị các tài nguyên cần thiết để lưu trữ và quản lý Terraform State từ xa.

Sau khi hoàn thành, quay lại module Identity:

```powershell
cd ..\03-identity
```

---

## Thực hiện Terraform Init

Tại thư mục `03-identity`, chạy lệnh:

```powershell
terraform init
```

Khi thực thi, Terraform sẽ:

1. Đọc các tệp cấu hình trong thư mục hiện tại.
2. Khởi tạo Terraform Backend.
3. Kết nối đến nơi lưu trữ Terraform State.
4. Tải AWS Provider theo phiên bản được khai báo.
5. Khởi tạo các module phụ thuộc nếu có.
6. Tạo thư mục `.terraform/`.
7. Tạo hoặc cập nhật tệp `.terraform.lock.hcl`.

Khi quá trình khởi tạo thành công, Terminal hiển thị thông báo:

```text
Terraform has been successfully initialized!
```

<!--
HƯỚNG DẪN CHỤP ẢNH:
1. Mở Terminal tại infra/03-identity.
2. Chạy lệnh: terraform init
3. Chụp phần kết quả có dòng:
   Terraform has been successfully initialized!
4. Lưu ảnh tại:
static/images/5-Workshop/5.3-Infrastructure/terraform-init-success.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-init-success.png" alt="Terraform Init thành công" width="85%">
    <figcaption style="text-align: center;">
        <b>Hình 5.3.13.</b> Khởi tạo module Terraform thành công bằng lệnh <code>terraform init</code>.
    </figcaption>
</figure>

---

## Khởi tạo lại Backend

Khi nội dung trong tệp `backend.tf` thay đổi, Terraform có thể yêu cầu khởi tạo lại Backend.

Sử dụng lệnh:

```powershell
terraform init -reconfigure
```

Tùy chọn `-reconfigure` yêu cầu Terraform bỏ qua cấu hình Backend đã lưu trước đó và đọc lại cấu hình hiện tại.

Nếu cần chuyển Terraform State từ Backend cũ sang Backend mới, sử dụng:

```powershell
terraform init -migrate-state
```

{{% notice warning %}}
Chỉ sử dụng `-migrate-state` khi cần chuyển Terraform State. Nên kiểm tra và sao lưu State trước khi thực hiện để tránh ảnh hưởng đến trạng thái hạ tầng.
{{% /notice %}}

---

## Kiểm tra kết quả khởi tạo

Sau khi `terraform init` hoàn tất, kiểm tra lại nội dung thư mục:

```powershell
Get-ChildItem -Force
```

Terraform sẽ tạo thêm:

```text
.terraform/
.terraform.lock.hcl
```

Trong đó:

* `.terraform/` chứa Provider, module và thông tin Backend được Terraform sử dụng.
* `.terraform.lock.hcl` khóa phiên bản Provider đã được lựa chọn.
* Các tệp `.tf` ban đầu vẫn được giữ nguyên.
* Module đã sẵn sàng cho các bước kiểm tra và triển khai tiếp theo.

<!--
HƯỚNG DẪN CHỤP ẢNH:
1. Trong VS Code Explorer, mở thư mục infra/03-identity.
2. Chụp cấu trúc có:
   .terraform/
   .terraform.lock.hcl
   backend.tf
   main.tf
   outputs.tf
   providers.tf
   variables.tf
   versions.tf
3. Lưu ảnh tại:
static/images/5-Workshop/5.3-Infrastructure/terraform-init-files.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-init-files.png" alt="Các tệp sau Terraform Init" width="65%">
    <figcaption style="text-align: center;">
        <b>Hình 5.3.14.</b> Thư mục <code>.terraform</code> và tệp <code>.terraform.lock.hcl</code> sau khi khởi tạo.
    </figcaption>
</figure>

---

## Khởi tạo các module còn lại

Sau khi khởi tạo module Identity, thực hiện tương tự với các module còn lại.

Khởi tạo module Data:

```powershell
cd ..\04-data
terraform init
```

Khởi tạo module Messaging:

```powershell
cd ..\05-messaging
terraform init
```

Khởi tạo module Compute:

```powershell
cd ..\06-compute
terraform init
```

Khởi tạo module API:

```powershell
cd ..\07-api
terraform init
```

Khởi tạo module Edge:

```powershell
cd ..\09-edge
terraform init
```

Mỗi module có cấu hình Backend và Terraform State riêng. Việc phân chia State theo module giúp nhóm kiểm soát phạm vi thay đổi và hạn chế ảnh hưởng giữa các lớp hạ tầng.

Thứ tự thực hiện của nhóm gồm:

1. Identity.
2. Data.
3. Messaging.
4. Compute.
5. API.
6. Edge.

Lệnh `terraform init` chỉ chuẩn bị môi trường làm việc và chưa tạo tài nguyên AWS. Các tài nguyên chỉ được tạo khi thực hiện `terraform apply`.

---

## Một số lỗi thường gặp

### Terraform không được nhận diện

Thông báo lỗi:

```text
terraform: The term 'terraform' is not recognized
```

Nguyên nhân có thể do Terraform chưa được cài đặt hoặc chưa được thêm vào biến môi trường `PATH`.

Kiểm tra lại bằng lệnh:

```powershell
terraform version
```

### Không tìm thấy thông tin xác thực AWS

Thông báo lỗi:

```text
No valid credential sources found
```

Kiểm tra cấu hình AWS CLI:

```powershell
aws configure list
```

Sau đó kiểm tra kết nối:

```powershell
aws sts get-caller-identity
```

### Không tìm thấy S3 Backend

Thông báo có thể xuất hiện:

```text
Failed to get existing workspaces
```

Cần kiểm tra:

* S3 Bucket lưu Terraform State đã được tạo chưa.
* Tên Bucket trong `backend.tf` có chính xác không.
* AWS Region có đúng không.
* IAM User hoặc IAM Role có quyền truy cập S3 không.

### Không đủ quyền truy cập

Nếu xuất hiện lỗi `AccessDenied`, cần kiểm tra IAM Policy của tài khoản đang sử dụng. Tài khoản cần có quyền truy cập Terraform Backend và các quyền cần thiết để quản lý tài nguyên trong module.

---

## Kết quả

Sau khi hoàn tất quá trình khởi tạo:

* Terraform đã nhận diện các tệp cấu hình trong từng module.
* AWS Provider đã được tải theo phiên bản yêu cầu.
* Terraform Backend đã được kết nối thành công.
* Thư mục `.terraform/` và tệp `.terraform.lock.hcl` đã được tạo.
* Các module đã sẵn sàng cho bước kiểm tra cấu hình.
* Nhóm có thể tiếp tục lập kế hoạch triển khai bằng lệnh `terraform plan`.