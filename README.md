# FCJ Workshop – Live Auction Platform on AWS

Repository báo cáo thực tập song ngữ Việt–Anh của dự án **Live Auction Platform on AWS**, được xây dựng bằng Hugo.

## 1. Yêu cầu

Trước khi chạy website, máy tính cần cài:

* Git
* Hugo Extended
* Visual Studio Code hoặc trình soạn thảo tương đương

## 2. Cài đặt Git

Tải Git tại:

<https://git-scm.com/download/win>

Sau khi cài đặt, mở PowerShell và kiểm tra:

```powershell
git --version
```

Nếu Terminal hiển thị phiên bản Git thì quá trình cài đặt đã thành công.

## 3. Cài đặt Hugo trên Windows

Mở PowerShell hoặc Windows Terminal bằng quyền Administrator và chạy:

```powershell
winget install Hugo.Hugo.Extended
```

Sau khi cài đặt hoàn tất, đóng Terminal, mở lại và kiểm tra:

```powershell
hugo version
```

Kết quả cần hiển thị phiên bản Hugo và thông tin `extended`.

Nếu máy không sử dụng được `winget`, có thể cài bằng Chocolatey:

```powershell
choco install hugo-extended
```

Tài liệu cài đặt Hugo chính thức:

<https://gohugo.io/installation/windows/>

## 4. Tải source code

Mở PowerShell tại thư mục muốn lưu dự án và chạy:

```powershell
git clone --recurse-submodules https://github.com/DanhTran406091/fcj-workshop-WEB_TranLeCongDanh.git
```

Di chuyển vào thư mục dự án:

```powershell
cd fcj-workshop-WEB_TranLeCongDanh
```

Dự án sử dụng Hugo theme dưới dạng Git submodule. Nếu đã clone repository nhưng thư mục `themes/hugo-theme-learn` bị trống, chạy:

```powershell
git submodule update --init --recursive
```

## 5. Chạy website Hugo

Tại thư mục gốc của dự án, chạy:

```powershell
hugo server -D -F
```

Trong đó:

* `hugo server`: khởi động web server trên máy.
* `-D`: hiển thị các nội dung có trạng thái Draft.
* `-F`: hiển thị nội dung có ngày xuất bản trong tương lai.

Sau khi server khởi động, mở trình duyệt và truy cập:

```text
http://localhost:1313
```

Hugo tự động tải lại trang khi file Markdown hoặc hình ảnh được cập nhật.

Để dừng server, nhấn:

```text
Ctrl + C
```

Nếu muốn chạy Hugo và vẫn tiếp tục sử dụng Terminal, hãy mở thêm một Terminal mới trong VS Code:

```text
Terminal → New Terminal
```

Giữ Hugo chạy ở Terminal thứ nhất và thực hiện Git hoặc các lệnh khác ở Terminal thứ hai.

## 6. Cấu trúc thư mục

```text
fcj-workshop-WEB_TranLeCongDanh/
│
├── content/                         # Nội dung báo cáo
│   ├── 1-Worklog/                   # Nhật ký công việc
│   ├── 2-Proposal/                  # Bản đề xuất của dự án
│   ├── 3-BlogsPosted/               # Các bài blog đã đăng
│   ├── 4-EventParticipated/         # Các sự kiện đã tham gia
│   ├── 5-Workshop/                  # Workshop triển khai dự án
│   ├── 6-Self-evaluation/           # Tự đánh giá
│   └── 7-Contribution/              # Chia sẻ và đóng góp
│
├── static/
│   └── images/                      # Hình ảnh sử dụng trong báo cáo
│
├── themes/
│   └── hugo-theme-learn/            # Hugo theme
│
├── layouts/                         # Giao diện tùy chỉnh
├── archetypes/                      # Mẫu tạo nội dung Hugo
├── config.toml                      # Cấu hình website Hugo
├── .gitmodules                      # Cấu hình Git submodule
└── README.md                        # File hướng dẫn này
```

Không chỉnh sửa trực tiếp nội dung trong thư mục `public`. Đây là thư mục kết quả được Hugo tạo ra khi build website.

## 7. Quy tắc viết nội dung song ngữ

Mỗi nội dung cần có hai file Markdown:

```text
_index.vi.md
_index.md
```

Trong đó:

| File | Nội dung |
| --- | --- |
| `_index.vi.md` | Nội dung tiếng Việt |
| `_index.md` | Nội dung tiếng Anh |

Ví dụ:

```text
content/5-Workshop/5.4-AWS-Services/5.4.2-S3/
├── _index.vi.md
└── _index.md
```

Không viết nội dung tiếng Việt và tiếng Anh chung trong một file.

## 8. Cấu trúc phần Workshop

```text
content/5-Workshop/
│
├── 5.1-Workshop-overview/
│   ├── _index.vi.md
│   └── _index.md
│
├── 5.2-Preparation/
│
├── 5.3-Infrastructure/
│   ├── 5.3.1-Terraform-overview/
│   ├── 5.3.2-Infrastructure-structure/
│   ├── 5.3.3-Terraform-init/
│   ├── 5.3.4-Terraform-plan/
│   └── 5.3.5-Terraform-apply/
│
├── 5.4-AWS-Services/
│   ├── 5.4.1-IAM-Cognito/
│   ├── 5.4.2-S3/
│   ├── 5.4.3-CloudFront/
│   ├── 5.4.4-Lambda/
│   ├── 5.4.5-API-Gateway/
│   ├── 5.4.6-WebSocket/
│   ├── 5.4.7-DynamoDB/
│   └── 5.4.8-SQS-FIFO/
│
├── 5.5-System-Testing/
└── 5.6-Results/
```

Tên thư mục thực tế có thể được điều chỉnh nhưng phải đồng nhất giữa `content` và `static/images`.

## 9. Mẫu file nội dung

### File tiếng Việt `_index.vi.md`

```markdown
---
title: "Amazon S3"
date: 2026-07-27
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

# AMAZON S3

## Tổng quan

Viết nội dung tiếng Việt tại đây.

## Các bước triển khai

### Bước 1: Truy cập dịch vụ

Viết hướng dẫn tại đây.

## Kết quả

Viết kết quả đạt được tại đây.
```

### File tiếng Anh `_index.md`

```markdown
---
title: "Amazon S3"
date: 2026-07-27
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

# AMAZON S3

## Overview

Write the English content here.

## Deployment Steps

### Step 1: Open the Service

Write the instructions here.

## Results

Write the results here.
```

Giá trị `weight` quyết định thứ tự hiển thị của nội dung trong menu.

## 10. Quy tắc lưu hình ảnh

Hình ảnh phải được lưu trong:

```text
static/images/
```

Hình của phần nào nên được đặt trong thư mục tương ứng với phần đó.

Ví dụ, hình của mục `5.4.2 Amazon S3` được lưu tại:

```text
static/images/5-Workshop/5.4-AWS-services/5.4.2-S3/
```

Ví dụ tên file:

```text
s3-bucket-list.png
s3-bucket-properties.png
s3-bucket-content.png
```

Nên sử dụng:

* Chữ thường.
* Không dấu.
* Không khoảng trắng.
* Dấu gạch ngang `-` để phân cách từ.
* Tên mô tả đúng nội dung hình.

Không đưa vào hình ảnh các thông tin nhạy cảm như:

* AWS Account ID.
* Email cá nhân.
* Access Key.
* Secret Access Key.
* Token đăng nhập.
* Client Secret.
* Mật khẩu.
* Thông tin cá nhân của người dùng.

## 11. Cách chèn hình vào file Markdown

Ví dụ ảnh được lưu tại:

```text
static/images/5-Workshop/5.4-AWS-services/5.4.2-S3/s3-bucket-list.png
```

Chèn vào file Markdown bằng:

```markdown
{{< figure
    src="/images/5-Workshop/5.4-AWS-services/5.4.2-S3/s3-bucket-list.png"
    title="Hình 5.4.2.1: Danh sách S3 Bucket của hệ thống Live Auction"
    width="100%"
>}}
```

Trong file tiếng Anh:

```markdown
{{< figure
    src="/images/5-Workshop/5.4-AWS-services/5.4.2-S3/s3-bucket-list.png"
    title="Figure 5.4.2.1: S3 Buckets of the Live Auction system"
    width="100%"
>}}
```

Lưu ý: đường dẫn trong `src` bắt đầu từ `/images/`, không ghi thêm `static`.

## 12. Cách viết khối lệnh Terminal

Lệnh PowerShell:

```markdown
```powershell
hugo server -D -F
```
```

Lệnh Terraform:

```markdown
```powershell
terraform init
terraform plan
terraform apply
```
```

Kết quả hoặc giá trị không cần thực thi:

```markdown
```text
http://localhost:1313
```
```

## 13. Kiểm tra trước khi gửi nội dung

Trước khi gửi phần đã viết, cần kiểm tra:

* Có đủ file `_index.vi.md` và `_index.md`.
* Nội dung tiếng Việt và tiếng Anh tương ứng với nhau.
* Front matter ở đầu file có đầy đủ `title`, `date`, `weight`, `chapter` và `pre`.
* Số thứ tự trong `pre` chính xác.
* Các lệnh Terminal được đặt trong code block.
* Hình ảnh được lưu đúng thư mục.
* Đường dẫn chèn hình hoạt động.
* Chú thích hình có số thứ tự.
* Không có thông tin nhạy cảm trong nội dung hoặc hình ảnh.
* Website chạy được bằng `hugo server -D -F`.
* Không có trang hoặc hình ảnh bị lỗi.

## 14. Cập nhật code mới nhất

Trước khi bắt đầu viết, lấy phiên bản mới nhất:

```powershell
git pull origin main
```

Nếu repository của nhóm trưởng được đặt là `upstream`, sử dụng:

```powershell
git pull upstream main
```

## 15. Lưu và đẩy thay đổi lên GitHub

Kiểm tra các file đã thay đổi:

```powershell
git status
```

Thêm thay đổi:

```powershell
git add .
```

Tạo commit:

```powershell
git commit -m "Add assigned workshop sections"
```

Đẩy lên GitHub:

```powershell
git push origin main
```

Sau khi push xong, gửi:

* Link repository.
* Mã commit mới nhất.
* Danh sách các mục đã hoàn thành.

Xem mã commit:

```powershell
git log -1 --oneline
```

## 16. Gửi bằng file ZIP

Nếu không sử dụng Git, có thể gửi bằng file ZIP.

File ZIP phải bao gồm:

```text
content/5-Workshop/PHAN_DA_VIET/
static/images/5-Workshop/HINH_ANH_CUA_PHAN_DA_VIET/
```

Không chỉ gửi riêng file Markdown vì khi ghép báo cáo sẽ bị thiếu hình ảnh.

## 17. Build website

Để build toàn bộ website:

```powershell
hugo -D -F
```

Kết quả build được tạo trong thư mục:

```text
public/
```

Không chỉnh sửa file trong `public` vì Hugo sẽ ghi đè thư mục này ở lần build tiếp theo.

## 18. Xử lý lỗi thường gặp

### Không nhận lệnh Hugo

Nếu xuất hiện:

```text
hugo: The term 'hugo' is not recognized
```

Đóng Terminal, mở lại và chạy:

```powershell
hugo version
```

Nếu vẫn lỗi, cài lại Hugo Extended:

```powershell
winget install Hugo.Hugo.Extended
```

### Website thiếu theme hoặc không chạy

Chạy:

```powershell
git submodule update --init --recursive
```

Sau đó chạy lại:

```powershell
hugo server -D -F
```

### Không thấy nội dung mới

Kiểm tra:

* File có đúng đuôi `.md`.
* File có đúng tên `_index.vi.md` hoặc `_index.md`.
* Front matter có đủ hai dấu `---`.
* Giá trị `date` có nằm trong tương lai không.
* Server có được chạy bằng `-D -F` hay không.

Khởi động lại:

```powershell
hugo server -D -F
```

### Hình ảnh không hiển thị

Kiểm tra:

* Ảnh đã nằm trong `static/images`.
* Tên file trong `src` trùng hoàn toàn với tên ảnh.
* Đường dẫn không chứa `static`.
* Chữ hoa và chữ thường trong tên thư mục có đồng nhất hay không.

## 19. Repository

Repository chính:

<https://github.com/DanhTran406091/fcj-workshop-WEB_TranLeCongDanh>