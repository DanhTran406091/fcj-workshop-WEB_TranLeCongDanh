---
title: "AWS IAM và Amazon Cognito"
date: 2026-07-27
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

## Tổng quan

Hệ thống **Live Auction** sử dụng **AWS Identity and Access Management (IAM)** và **Amazon Cognito** để thực hiện hai nhiệm vụ khác nhau:

* **AWS IAM** quản lý quyền truy cập giữa các dịch vụ AWS.
* **Amazon Cognito** xác thực tài khoản đăng nhập vào ứng dụng và hỗ trợ phân biệt quyền của người dùng với quản trị viên.

IAM không được sử dụng để tạo tài khoản đăng nhập trực tiếp cho người dùng của ứng dụng. Thay vào đó, tài khoản User và Admin được quản lý thông qua Cognito User Pool.

## Vai trò của AWS IAM

AWS IAM cung cấp các Role và Policy cho phép các dịch vụ AWS giao tiếp với nhau theo nguyên tắc **quyền tối thiểu — least privilege**.

Trong hệ thống Live Auction, IAM được sử dụng để:

* Cho phép AWS Lambda ghi log vào Amazon CloudWatch.
* Cho phép Lambda đọc và ghi dữ liệu trong Amazon DynamoDB.
* Cho phép Lambda gửi và nhận thông điệp từ Amazon SQS FIFO.
* Cho phép Lambda quản lý các kết nối của API Gateway WebSocket.
* Cho phép API Gateway gọi các Lambda Function tương ứng.
* Cho phép Amazon S3 và CloudFront phối hợp phân phối nội dung frontend.
* Giới hạn mỗi Lambda Function chỉ được truy cập những tài nguyên cần thiết.
* Kiểm soát quyền truy cập giữa các thành phần trong hệ thống.

## Role và Policy trong hệ thống

Mỗi nhóm Lambda Function được gán một IAM Role phù hợp với nghiệp vụ mà nó xử lý.

| Thành phần | Quyền cần thiết |
| --- | --- |
| **Business Logic Lambda** | Đọc và ghi dữ liệu trong DynamoDB, đồng thời ghi log vào CloudWatch. |
| **Bid Processing Lambda** | Nhận thông điệp từ SQS FIFO, cập nhật dữ liệu đặt giá trong DynamoDB và gửi kết quả đến WebSocket API. |
| **WebSocket Lambda** | Lưu, đọc và xóa Connection ID trong DynamoDB; quản lý kết nối WebSocket. |
| **API Gateway** | Được phép gọi các Lambda Function đã cấu hình. |
| **CloudFront** | Được phép truy cập nội dung frontend trong S3 theo cấu hình phân phối. |

Các IAM Role và IAM Policy được khai báo trong Terraform. Khi chạy `terraform apply`, Terraform tự động tạo Role, gắn Policy và thiết lập quan hệ tin cậy giữa các dịch vụ.

## Kiểm tra IAM Role trên AWS Management Console

Sau khi triển khai Terraform thành công, nhóm kiểm tra các IAM Role đã được tạo bằng các bước sau.

### Bước 1: Truy cập dịch vụ IAM

Đăng nhập vào **AWS Management Console**.

Tại thanh tìm kiếm phía trên, nhập:

```text
IAM
```

Chọn **IAM — Identity and Access Management** từ danh sách kết quả.

### Bước 2: Mở danh sách IAM Role

Tại thanh điều hướng bên trái, chọn:

```text
Access management → Roles
```

Trang Roles hiển thị danh sách các IAM Role hiện có trong tài khoản AWS.

### Bước 3: Tìm IAM Role của dự án

Trong ô tìm kiếm, nhập tiền tố tài nguyên của hệ thống:

```text
la-
```

Kiểm tra các IAM Role được Terraform tạo cho Lambda Function và những thành phần liên quan.

<!--
HƯỚNG DẪN CHỤP HÌNH:

1. Mở IAM → Roles.
2. Nhập "la-" vào ô tìm kiếm.
3. Chụp danh sách IAM Role của dự án.
4. Không để lộ AWS Account ID hoặc thông tin nhạy cảm.
5. Lưu ảnh với tên:
   iam-role-list.png
6. Đặt ảnh tại:
   static/images/5-Workshop/5.4-AWS-services/5.4.1-IAM-Cognito/
-->

{{< figure
    src="/images/5-Workshop/5.4-AWS-services/5.4.1-IAM-Cognito/iam-role-list.png"
    title="Hình 5.4.1.1: Danh sách IAM Role của hệ thống Live Auction"
    width="100%"
>}}

### Bước 4: Kiểm tra quyền của IAM Role

Chọn một IAM Role được gán cho Lambda Function.

Trong tab **Permissions**, kiểm tra:

* Các Policy được gắn với Role.
* Quyền ghi log vào CloudWatch.
* Quyền truy cập DynamoDB.
* Quyền truy cập SQS FIFO nếu Lambda có xử lý hàng đợi.
* Quyền quản lý WebSocket nếu Lambda có gửi dữ liệu thời gian thực.
* Phạm vi Resource mà Policy cho phép truy cập.

Không nên sử dụng quyền toàn phần như `AdministratorAccess` cho Lambda Function. Các quyền cần được giới hạn theo đúng tài nguyên và nghiệp vụ của từng hàm.

<!--
HƯỚNG DẪN CHỤP HÌNH:

1. Chọn một IAM Role của Lambda.
2. Mở tab Permissions.
3. Chụp danh sách Permission policies.
4. Không cần mở và chụp toàn bộ JSON Policy nếu nội dung quá dài.
5. Lưu ảnh với tên:
   iam-role-permissions.png
-->

{{< figure
    src="/images/5-Workshop/5.4-AWS-services/5.4.1-IAM-Cognito/iam-role-permissions.png"
    title="Hình 5.4.1.2: Các Policy được gắn với IAM Role của Lambda"
    width="100%"
>}}

### Bước 5: Kiểm tra Trust Relationship

Trong trang chi tiết của IAM Role, mở tab:

```text
Trust relationships
```

Kiểm tra dịch vụ được phép sử dụng Role.

Đối với Lambda Execution Role, phần **Trusted entities** phải cho phép dịch vụ:

```text
lambda.amazonaws.com
```

Trust Relationship giúp bảo đảm chỉ đúng dịch vụ được chỉ định mới có thể sử dụng IAM Role.

## Vai trò của Amazon Cognito

Amazon Cognito được sử dụng làm dịch vụ xác thực tài khoản cho cả **User Frontend** và **Admin Frontend**.

Cognito chịu trách nhiệm:

* Đăng ký tài khoản người dùng.
* Đăng nhập và đăng xuất.
* Xác minh thông tin tài khoản.
* Quản lý mật khẩu.
* Cấp token sau khi đăng nhập thành công.
* Lưu trữ các thuộc tính cơ bản của tài khoản.
* Hỗ trợ phân biệt quyền User và Admin.
* Cho phép API Gateway hoặc backend kiểm tra danh tính của người gửi yêu cầu.

## Các thành phần Cognito được sử dụng

### Cognito User Pool

Cognito User Pool là thư mục lưu trữ tài khoản của hệ thống.

User Pool quản lý các thông tin như:

* Username hoặc email.
* Mật khẩu.
* Trạng thái xác nhận tài khoản.
* Các thuộc tính của tài khoản.
* Nhóm hoặc vai trò của tài khoản.
* Chính sách mật khẩu.
* Quy trình đăng ký và đăng nhập.

Hệ thống sử dụng chung User Pool cho tài khoản User và Admin. Quyền truy cập được phân biệt dựa trên thông tin vai trò hoặc nhóm của tài khoản.

### Cognito App Client

App Client cho phép frontend kết nối với Cognito User Pool.

Sau khi xác thực thành công, Cognito trả về các token cần thiết, bao gồm:

* **ID Token:** Chứa thông tin nhận dạng của tài khoản.
* **Access Token:** Được sử dụng để xác minh quyền truy cập.
* **Refresh Token:** Được sử dụng để yêu cầu token mới khi token hiện tại hết hạn.

Frontend gửi token kèm theo các yêu cầu đến API Gateway. Backend kiểm tra token trước khi thực hiện nghiệp vụ.

{{% notice warning %}}
Không đưa Client Secret, Access Token, Refresh Token hoặc thông tin xác thực của tài khoản vào ảnh chụp hay nội dung báo cáo.
{{% /notice %}}

## Luồng xác thực tài khoản

Luồng xác thực của hệ thống được thực hiện như sau:

1. User hoặc Admin nhập thông tin đăng nhập trên giao diện tương ứng.
2. Frontend gửi yêu cầu xác thực đến Amazon Cognito.
3. Cognito kiểm tra tài khoản và mật khẩu trong User Pool.
4. Nếu thông tin hợp lệ, Cognito trả về token cho frontend.
5. Frontend lưu thông tin đăng nhập theo cơ chế của ứng dụng.
6. Khi gọi API, frontend đính kèm Access Token hoặc ID Token vào yêu cầu.
7. API Gateway hoặc Lambda kiểm tra token và thông tin vai trò.
8. Yêu cầu chỉ được xử lý nếu tài khoản có quyền phù hợp.
9. Các API quản trị từ chối yêu cầu của tài khoản User thông thường.

## Kiểm tra Cognito User Pool trên AWS Management Console

### Bước 1: Truy cập Amazon Cognito

Tại thanh tìm kiếm của AWS Management Console, nhập:

```text
Cognito
```

Chọn **Amazon Cognito**.

### Bước 2: Mở danh sách User Pool

Trong giao diện Amazon Cognito, chọn:

```text
User pools
```

Kiểm tra User Pool được Terraform tạo cho hệ thống Live Auction.

<!--
HƯỚNG DẪN CHỤP HÌNH:

1. Mở Amazon Cognito → User pools.
2. Chụp danh sách User Pool.
3. Bảo đảm ảnh thể hiện tên và trạng thái User Pool.
4. Che thông tin nhạy cảm nếu có.
5. Lưu ảnh với tên:
   cognito-user-pool.png
-->

{{< figure
    src="/images/5-Workshop/5.4-AWS-services/5.4.1-IAM-Cognito/cognito-user-pool.png"
    title="Hình 5.4.1.3: Cognito User Pool của hệ thống Live Auction"
    width="100%"
>}}

### Bước 3: Kiểm tra cấu hình đăng nhập

Chọn User Pool của hệ thống và kiểm tra các thông tin:

* Tên User Pool.
* User Pool ID.
* AWS Region.
* Phương thức đăng nhập.
* Thuộc tính được sử dụng để xác thực.
* Chính sách mật khẩu.
* Trạng thái tự đăng ký tài khoản.
* Cơ chế xác minh email.
* Các nhóm người dùng nếu hệ thống sử dụng Cognito Group.

Không đưa toàn bộ User Pool ID vào báo cáo nếu muốn hạn chế công khai thông tin định danh tài nguyên.

### Bước 4: Kiểm tra App Client

Trong trang chi tiết User Pool, tìm phần:

```text
Applications → App clients
```

Kiểm tra App Client được frontend sử dụng để đăng ký và đăng nhập.

Các thông tin cần kiểm tra gồm:

* Tên App Client.
* Client ID.
* Authentication flow được bật.
* Thời gian hiệu lực của token.
* Callback URL và Sign-out URL nếu có sử dụng.
* App Client có phù hợp với ứng dụng frontend hay không.

<!--
HƯỚNG DẪN CHỤP HÌNH:

1. Mở User Pool → Applications → App clients.
2. Chụp danh sách App Client.
3. Che Client ID nếu cần.
4. Không chụp Client Secret.
5. Lưu ảnh với tên:
   cognito-app-client.png
-->

{{< figure
    src="/images/5-Workshop/5.4-AWS-services/5.4.1-IAM-Cognito/cognito-app-client.png"
    title="Hình 5.4.1.4: App Client được cấu hình cho hệ thống Live Auction"
    width="100%"
>}}

### Bước 5: Kiểm tra tài khoản User và Admin

Trong User Pool, mở phần:

```text
User management → Users
```

Kiểm tra các tài khoản đã được tạo trong hệ thống.

Các thông tin cần kiểm tra gồm:

* Username.
* Email.
* Trạng thái xác nhận tài khoản.
* Trạng thái kích hoạt tài khoản.
* Ngày tạo tài khoản.
* Vai trò hoặc nhóm của tài khoản.

Nếu hệ thống sử dụng Cognito Group, tiếp tục mở phần:

```text
User management → Groups
```

Kiểm tra các nhóm tương ứng, chẳng hạn:

```text
User
Admin
```

Tài khoản quản trị viên phải được gán đúng quyền Admin trước khi truy cập các chức năng quản lý.

<!--
HƯỚNG DẪN CHỤP HÌNH:

1. Mở User management → Users.
2. Chụp một phần danh sách tài khoản.
3. Che email, username hoặc thông tin cá nhân nếu cần.
4. Không chụp token hoặc thông tin đăng nhập.
5. Lưu ảnh với tên:
   cognito-users.png
-->

{{< figure
    src="/images/5-Workshop/5.4-AWS-services/5.4.1-IAM-Cognito/cognito-users.png"
    title="Hình 5.4.1.5: Danh sách tài khoản được quản lý trong Amazon Cognito"
    width="100%"
>}}

## Kiểm tra chức năng xác thực

Sau khi kiểm tra cấu hình trên AWS Management Console, nhóm kiểm tra hoạt động xác thực trực tiếp trên hai giao diện.

### Kiểm tra tài khoản User

1. Truy cập User Frontend.
2. Tạo tài khoản mới.
3. Thực hiện bước xác minh tài khoản nếu được yêu cầu.
4. Đăng nhập bằng tài khoản vừa tạo.
5. Kiểm tra khả năng truy cập thông tin cá nhân.
6. Kiểm tra khả năng xem và tạo phiên đấu giá.
7. Xác nhận tài khoản User không thể truy cập các chức năng quản trị.

### Kiểm tra tài khoản Admin

1. Truy cập Admin Frontend.
2. Đăng nhập bằng tài khoản Admin.
3. Kiểm tra chức năng quản lý tài khoản người dùng.
4. Kiểm tra chức năng quản lý danh mục sản phẩm.
5. Kiểm tra chức năng duyệt phiên đấu giá.
6. Kiểm tra chức năng tạo thêm tài khoản Admin.
7. Xác nhận API quản trị chỉ chấp nhận tài khoản có quyền Admin.

## Kết quả

Sau khi hoàn tất quá trình triển khai và kiểm tra:

* Các IAM Role và IAM Policy đã được Terraform tạo thành công.
* Các Lambda Function được gán quyền phù hợp với nghiệp vụ.
* Quyền truy cập giữa các dịch vụ được kiểm soát theo nguyên tắc quyền tối thiểu.
* Lambda có thể ghi log vào Amazon CloudWatch.
* Các Lambda cần thiết có thể truy cập DynamoDB, SQS FIFO và WebSocket API.
* Cognito User Pool đã được tạo và hoạt động ổn định.
* App Client đã được cấu hình để frontend thực hiện đăng ký và đăng nhập.
* Người dùng có thể tạo tài khoản và đăng nhập vào User Frontend.
* Quản trị viên có thể đăng nhập vào Admin Frontend.
* Quyền User và Admin được phân biệt khi truy cập các chức năng của hệ thống.
* Các API quản trị được bảo vệ khỏi tài khoản không có quyền phù hợp.
* Thông tin xác thực và quyền truy cập đã sẵn sàng để tích hợp với các dịch vụ tiếp theo.