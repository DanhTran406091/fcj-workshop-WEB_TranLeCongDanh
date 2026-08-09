---
title : "Các bước chuẩn bị"
date : 2026-07-13 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

# Chuẩn bị môi trường

## Giới thiệu

Trước khi triển khai hệ thống **Live Auction** trên nền tảng **Amazon Web Services (AWS)**, cần chuẩn bị đầy đủ môi trường phát triển và các công cụ hỗ trợ. Việc chuẩn bị trước giúp đảm bảo quá trình triển khai hạ tầng bằng **Terraform** diễn ra thuận lợi, đồng thời giúp các thành viên trong nhóm sử dụng cùng một môi trường làm việc.

Trong workshop này, nhóm sử dụng nhiều công cụ để quản lý mã nguồn, phát triển ứng dụng, triển khai hạ tầng và tương tác với các dịch vụ AWS.

---

## Chuẩn bị phần mềm và công cụ

Trước khi bắt đầu, cần chuẩn bị các thành phần sau:

- Tài khoản AWS đã được cấp quyền truy cập.
- Git.
- AWS CLI.
- Terraform.
- Docker Desktop.
- Node.js.
- Python 3.
- Visual Studio Code (hoặc IDE tương đương).

---

## Chuẩn bị mã nguồn

Clone mã nguồn của dự án từ GitHub:

```bash
git clone <repository-url>
cd Live-Auction
```

Sau khi clone thành công, cấu trúc chính của dự án bao gồm:

```text
backend/
frontend/
admin-frontend/
infra/
docker-compose.yml
```

Trong đó:

- **backend/**: Mã nguồn Backend sử dụng FastAPI.
- **frontend/**: Giao diện dành cho người dùng.
- **admin-frontend/**: Giao diện dành cho quản trị viên.
- **infra/**: Mã nguồn Terraform dùng để triển khai hạ tầng AWS.

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.2-Prerequisite/project-structure.png" alt="Project Structure" width="35%">
    <figcaption style="text-align: center;">
        <b>Hình 5.2.1.</b> Cấu trúc thư mục chính của dự án.
    </figcaption>
</figure>

---

## Cài đặt và cấu hình AWS CLI

Sau khi cài đặt AWS CLI, kiểm tra phiên bản bằng lệnh:

```bash
aws --version
```

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.2-Prerequisite/aws-version.png" alt="AWS CLI Version" width="80%">
    <figcaption style="text-align: center;">
        <b>Hình 5.2.2.</b> Kiểm tra phiên bản AWS CLI.
    </figcaption>
</figure>

Tiếp theo, cấu hình thông tin tài khoản AWS:

```bash
aws configure
```

Nhập lần lượt các thông tin sau:

- AWS Access Key ID
- AWS Secret Access Key
- Default Region
- Default Output Format

Sau khi hoàn tất, AWS CLI sẽ lưu thông tin xác thực để Terraform có thể sử dụng trong quá trình triển khai.

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.2-Prerequisite/aws-configure.png" alt="AWS Configure" width="80%">
    <figcaption style="text-align: center;">
        <b>Hình 5.2.3.</b> Cấu hình AWS CLI bằng lệnh <code>aws configure</code>.
    </figcaption>
</figure>

---

## Cài đặt Terraform

Sau khi cài đặt Terraform, kiểm tra phiên bản bằng lệnh:

```bash
terraform version
```

Nếu cài đặt thành công, hệ thống sẽ hiển thị phiên bản Terraform hiện tại.

Terraform sẽ được sử dụng trong các bước tiếp theo để triển khai và quản lý toàn bộ hạ tầng AWS của hệ thống Live Auction.

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.2-Prerequisite/terraform-version.png" alt="Terraform Version" width="60%">
    <figcaption style="text-align: center;">
        <b>Hình 5.2.4.</b> Kiểm tra phiên bản Terraform.
    </figcaption>
</figure>

---

## Kết quả

Sau khi hoàn thành các bước chuẩn bị trên, môi trường phát triển đã sẵn sàng để triển khai hạ tầng AWS bằng Terraform.

Trong phần tiếp theo, nhóm sẽ tiến hành khởi tạo dự án Terraform, kiểm tra kế hoạch triển khai và tạo toàn bộ tài nguyên AWS phục vụ hệ thống Live Auction.