---
title: "Blog 1 - Triển khai website React/Vite với Amazon S3"
date: 2026-08-03
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# TRIỂN KHAI WEBSITE REACT/VITE VỚI AMAZON S3

Trong quá trình tìm hiểu các dịch vụ AWS, nhóm em đã nghiên cứu và thực hành sử dụng **Amazon S3 Static Website Hosting** để triển khai một website tĩnh được xây dựng bằng React/Vite.

Bài viết này chia sẻ quy trình build ứng dụng, tải các tệp frontend lên Amazon S3 và cho phép người dùng truy cập website thông qua S3 Website Endpoint.

## Tổng quan về Amazon S3 Static Website Hosting

Amazon Simple Storage Service (Amazon S3) là dịch vụ lưu trữ đối tượng của AWS. Ngoài khả năng lưu trữ dữ liệu, Amazon S3 còn hỗ trợ triển khai các website tĩnh gồm:

* HTML
* CSS
* JavaScript
* Hình ảnh và font chữ

Sau khi ứng dụng React/Vite được build, các tệp cần thiết sẽ nằm trong thư mục `dist/`. Những tệp này có thể được tải lên S3 bucket và phân phối trực tiếp đến trình duyệt của người dùng.

## Sơ đồ triển khai

![Sơ đồ triển khai website React/Vite trên Amazon S3](/images/blog1/react-vite-deployment-amazon-s3.drawio.png)

Quy trình trong sơ đồ gồm:

1. Phát triển mã nguồn frontend bằng React/Vite.
2. Build ứng dụng để tạo thư mục `dist/`.
3. Tải các tệp đã build lên Amazon S3.
4. Bật tính năng Static Website Hosting.
5. Người dùng truy cập website bằng S3 Website Endpoint.
6. Amazon S3 trả về các tệp HTML, CSS và JavaScript.

## Các bước thực hiện

### Bước 1: Build ứng dụng React/Vite

Tại thư mục dự án, chạy các lệnh:

```bash
npm install
npm run build
```

Sau khi quá trình build hoàn tất, Vite sẽ tạo thư mục `dist/` chứa phiên bản đã được tối ưu của website.

### Bước 2: Tạo S3 bucket

Đăng nhập vào AWS Management Console và thực hiện:

1. Tìm kiếm dịch vụ **Amazon S3**.
2. Chọn **Create bucket**.
3. Nhập tên bucket.
4. Chọn AWS Region phù hợp.
5. Kiểm tra các thiết lập và chọn **Create bucket**.

Tên S3 bucket phải là duy nhất trong phạm vi AWS. Nhóm em sử dụng tên liên quan đến ứng dụng để thuận tiện cho việc quản lý.

### Bước 3: Tải website lên Amazon S3

Sau khi tạo bucket:

1. Mở bucket vừa tạo.
2. Chọn **Upload**.
3. Chọn **Add files** hoặc **Add folder**.
4. Tải toàn bộ nội dung bên trong thư mục `dist/`.
5. Chọn **Upload** để hoàn tất.

Tệp `index.html` phải nằm ở cấp đầu tiên của bucket. Không nên đặt toàn bộ thư mục `dist` thành một thư mục con vì Amazon S3 có thể không tìm thấy trang mặc định.

Cấu trúc đúng trong bucket:

```text
index.html
assets/
favicon.ico
...
```

### Bước 4: Bật Static Website Hosting

Trong S3 bucket, truy cập:

```text
Properties
→ Static website hosting
→ Edit
```

Sau đó cấu hình:

```text
Static website hosting: Enable
Hosting type: Host a static website
Index document: index.html
```

Nếu ứng dụng có trang xử lý lỗi, có thể thiết lập thêm:

```text
Error document: error.html
```

Sau khi lưu cấu hình, Amazon S3 cung cấp một **Bucket website endpoint** để truy cập website.

### Bước 5: Cấu hình quyền truy cập

Để người dùng có thể mở website bằng S3 Website Endpoint, các tệp frontend cần có quyền đọc công khai phù hợp.

Cần kiểm tra:

1. Thiết lập **Block Public Access** của bucket.
2. Bucket policy cho phép đọc các object cần thiết.
3. Tệp `index.html` và thư mục `assets/` đã được tải lên đầy đủ.
4. Không có dữ liệu nhạy cảm trong mã nguồn frontend.

Ví dụ bucket policy cho phép đọc object công khai:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
    }
  ]
}
```

Cần thay `YOUR_BUCKET_NAME` bằng tên bucket thực tế.

Chỉ nên công khai các tệp phục vụ website. Không được lưu mật khẩu, Access Key, Secret Access Key hoặc dữ liệu nhạy cảm trong mã nguồn frontend.

## Cập nhật website

Khi mã nguồn thay đổi, nhóm em thực hiện lại quy trình:

```text
Cập nhật mã nguồn
→ Chạy npm run build
→ Tải các tệp mới trong dist lên S3
→ Kiểm tra lại website
```

Ngoài việc tải tệp thủ công bằng AWS Management Console, có thể sử dụng AWS CLI:

```bash
aws s3 sync dist/ s3://YOUR_BUCKET_NAME --delete
```

Tùy chọn `--delete` sẽ xóa trên S3 những tệp không còn tồn tại trong thư mục `dist`. Vì vậy, cần kiểm tra chính xác tên bucket trước khi chạy.

## Một số lỗi thường gặp

### Lỗi Access Denied

Nguyên nhân thường liên quan đến:

* Block Public Access vẫn đang được bật.
* Bucket policy chưa chính xác.
* ARN của bucket trong policy bị sai.

### Website hiển thị trang trắng

Cần kiểm tra:

* Đường dẫn đến các tệp trong thư mục `assets/`.
* Lỗi trong Developer Tools của trình duyệt.
* Quá trình build ứng dụng có thành công hay không.

### Không tìm thấy index.html

Tệp `index.html` có thể đang nằm trong thư mục `dist` trên bucket thay vì nằm ở cấp đầu tiên.

### Lỗi khi tải lại đường dẫn React Router

Amazon S3 tìm object dựa trên đường dẫn được yêu cầu. Đối với ứng dụng SPA sử dụng React Router, cần có phương án xử lý điều hướng phù hợp hoặc kết hợp thêm Amazon CloudFront.

## Ưu điểm

* Quy trình triển khai tương đối đơn giản.
* Không cần quản lý máy chủ.
* Phù hợp với website tĩnh và frontend SPA.
* Dễ dàng cập nhật các tệp sau khi build.
* Có thể kết hợp với các dịch vụ AWS khác.

## Hạn chế

* Không chạy trực tiếp backend Python, Java hoặc Node.js.
* S3 Website Endpoint chỉ hỗ trợ HTTP.
* Cần cấu hình quyền truy cập cẩn thận.
* Nếu cần HTTPS và CDN, có thể kết hợp Amazon CloudFront.

## Kết quả đạt được

Qua quá trình nghiên cứu và thực hành, nhóm em đã:

* Hiểu cách build ứng dụng React/Vite.
* Biết cách tạo và cấu hình S3 bucket.
* Biết cách bật Static Website Hosting.
* Triển khai frontend lên Amazon S3.
* Hiểu thêm về quyền truy cập và bảo mật dữ liệu.
* Biết cách cập nhật website khi mã nguồn thay đổi.
* Nhận biết một số lỗi thường gặp trong quá trình triển khai.

## Liên kết

* [Tài liệu cấu hình Static Website Hosting của AWS](https://docs.aws.amazon.com/AmazonS3/latest/userguide/HostingWebsiteOnS3Setup.html)
* [Xem bài đăng trên AWS Study Group](DAN_LINK_BAI_DANG_FACEBOOK_VAO_DAY)