---
title: "Bản đề xuất"
date: 2026-07-13
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# LIVE AUCTION PLATFORM ON AWS

## Nền tảng đấu giá trực tuyến triển khai trên điện toán đám mây AWS

### 1. Tóm tắt điều hành

**Live Auction** là nền tảng đấu giá trực tuyến cho phép người dùng đăng sản phẩm, theo dõi phiên đấu giá và đặt giá theo thời gian thực. Hệ thống hướng đến việc tạo ra một môi trường đấu giá minh bạch, thuận tiện và có khả năng phục vụ đồng thời nhiều người dùng.

Giao diện frontend được phát triển bằng **React/Vite**, backend sử dụng **FastAPI và Python**, dữ liệu được quản lý bằng **MySQL**. Trong giai đoạn triển khai ban đầu, nhóm sử dụng Amazon S3 để lưu trữ frontend và hình ảnh, Amazon EC2 để vận hành backend, Amazon RDS for MySQL để quản lý cơ sở dữ liệu và AWS Lambda cho một số tác vụ nền.

Bên cạnh kiến trúc triển khai ban đầu, nhóm xây dựng kiến trúc AWS mở rộng nhằm nghiên cứu khả năng xử lý đấu giá thời gian thực, tăng tính sẵn sàng, bảo mật dữ liệu và hỗ trợ khôi phục khi xảy ra sự cố.

### 2. Tuyên bố vấn đề

#### Vấn đề hiện tại

Các hệ thống đấu giá trực tuyến cần xử lý nhiều thao tác diễn ra gần như đồng thời, đặc biệt khi nhiều người dùng cùng đặt giá cho một sản phẩm. Nếu hệ thống xử lý không chính xác, có thể xảy ra các vấn đề như:

- Dữ liệu giá đấu không được cập nhật kịp thời.
- Nhiều yêu cầu đặt giá được xử lý sai thứ tự.
- Người dùng không nhận được trạng thái mới của phiên đấu giá.
- Hình ảnh sản phẩm và tài nguyên tĩnh tải chậm.
- Hệ thống bị gián đoạn khi máy chủ gặp sự cố.
- Khó theo dõi nhật ký và xác định nguyên nhân khi xảy ra lỗi.
- Dữ liệu người dùng và thông tin phiên đấu giá chưa được bảo vệ phù hợp.

Ngoài ra, việc triển khai tất cả thành phần trên một máy chủ duy nhất sẽ làm giảm khả năng mở rộng và tạo ra một điểm lỗi duy nhất cho toàn hệ thống.

#### Giải pháp đề xuất

Nhóm đề xuất triển khai Live Auction trên AWS theo từng giai đoạn.

Ở giai đoạn đầu, frontend React/Vite được build và triển khai lên **Amazon S3 Static Website Hosting**. Backend FastAPI được đóng gói bằng Docker và vận hành trên **Amazon EC2**. Cơ sở dữ liệu MySQL được chuyển từ môi trường Docker cục bộ sang **Amazon RDS for MySQL**, trong khi hình ảnh sản phẩm được lưu trong một S3 bucket riêng. **AWS Lambda** được sử dụng cho một số tác vụ nền hoặc tác vụ được kích hoạt theo sự kiện.

Ở giai đoạn mở rộng, hệ thống có thể sử dụng Amazon CloudFront, AWS WAF, Amazon API Gateway, Amazon Cognito, Amazon DynamoDB, Amazon SQS, Amazon EventBridge và các dịch vụ giám sát của AWS. Kiến trúc này hướng đến việc xử lý luồng đặt giá theo thứ tự, cải thiện khả năng chịu tải và hỗ trợ dự phòng đa vùng.

#### Lợi ích của giải pháp

Giải pháp mang lại những lợi ích chính như:

- Cho phép người dùng tham gia đấu giá từ xa thông qua trình duyệt.
- Cập nhật thông tin phiên đấu giá nhanh chóng.
- Tách riêng frontend, backend, cơ sở dữ liệu và tài nguyên hình ảnh.
- Hạn chế phụ thuộc vào một máy chủ duy nhất.
- Dễ dàng mở rộng khi số lượng người dùng tăng.
- Cải thiện khả năng bảo mật, giám sát và sao lưu dữ liệu.
- Giúp nhóm tiếp cận quy trình triển khai một ứng dụng thực tế trên AWS.

### 3. Kiến trúc giải pháp

#### 3.1. Kiến trúc triển khai ban đầu

Kiến trúc ban đầu của hệ thống gồm các thành phần chính:

1. Người dùng truy cập giao diện Live Auction bằng trình duyệt.
2. Frontend React/Vite được build thành các tệp HTML, CSS và JavaScript.
3. Các tệp frontend được triển khai trên Amazon S3.
4. Frontend gửi yêu cầu đến REST API của backend FastAPI.
5. Backend được đóng gói bằng Docker và vận hành trên Amazon EC2.
6. Backend đọc và ghi dữ liệu thông qua Amazon RDS for MySQL.
7. Hình ảnh sản phẩm được lưu trong một Amazon S3 bucket riêng.
8. AWS Lambda xử lý một số tác vụ nền hoặc công việc theo lịch.
9. Amazon CloudWatch hỗ trợ theo dõi nhật ký và tình trạng tài nguyên.

Kiến trúc này phù hợp với phạm vi đồ án hiện tại vì dễ triển khai, dễ kiểm tra và không yêu cầu quá nhiều dịch vụ phức tạp.

#### 3.2. Kiến trúc AWS mở rộng được đề xuất

Sơ đồ dưới đây mô tả kiến trúc AWS mở rộng được nhóm đề xuất cho hệ thống Live Auction. Sơ đồ tập trung vào khả năng xử lý đấu giá thời gian thực, bảo mật, giám sát và dự phòng đa vùng.

Nhấn vào sơ đồ để mở và xem ở kích thước đầy đủ.

[![Sơ đồ kiến trúc AWS đề xuất cho hệ thống Live Auction](/images/2-Proposal/live-auction-proposed-architecture.svg)](/images/2-Proposal/live-auction-proposed-architecture.svg)

> **Lưu ý:** Sơ đồ trên thể hiện kiến trúc mục tiêu được đề xuất. Một số thành phần nâng cao chưa được triển khai đầy đủ trong phiên bản hiện tại của đồ án.

#### 3.3. Luồng hoạt động tổng quát

Luồng hoạt động của kiến trúc đề xuất gồm:

1. Người dùng truy cập tên miền của hệ thống.
2. Amazon Route 53 định tuyến yêu cầu đến Amazon CloudFront.
3. AWS WAF kiểm tra và lọc các yêu cầu không hợp lệ.
4. CloudFront phân phối frontend được lưu trên Amazon S3.
5. Người dùng đăng nhập và được xác thực trước khi sử dụng các chức năng được bảo vệ.
6. REST API xử lý các chức năng như tài khoản, sản phẩm và phiên đấu giá.
7. WebSocket hỗ trợ truyền trạng thái và giá đấu mới đến người dùng.
8. Các yêu cầu đặt giá được đưa vào hàng đợi để xử lý tuần tự.
9. Dữ liệu phiên đấu giá được lưu trong cơ sở dữ liệu phù hợp.
10. Hình ảnh sản phẩm và bản ghi kiểm toán được lưu trên Amazon S3.
11. CloudWatch và CloudTrail hỗ trợ giám sát hoạt động của hệ thống.
12. Khi vùng chính xảy ra sự cố, Route 53 có thể định tuyến sang vùng dự phòng.

### 4. Các dịch vụ AWS đề xuất sử dụng

#### Amazon Route 53

Amazon Route 53 quản lý tên miền và định tuyến người dùng đến hệ thống. Trong kiến trúc mở rộng, Route 53 còn hỗ trợ kiểm tra tình trạng endpoint và chuyển hướng sang vùng dự phòng khi cần thiết.

#### Amazon CloudFront

Amazon CloudFront phân phối các tệp HTML, CSS, JavaScript và hình ảnh thông qua hệ thống CDN. Dịch vụ này giúp giảm thời gian tải trang và giảm số lượng yêu cầu trực tiếp đến S3.

#### AWS WAF

AWS WAF giúp lọc các yêu cầu có dấu hiệu bất thường, giới hạn tần suất truy cập và bảo vệ hệ thống trước một số hình thức tấn công phổ biến trên ứng dụng web.

#### Amazon S3

Amazon S3 được sử dụng cho các mục đích:

- Lưu trữ frontend React/Vite sau khi build.
- Lưu trữ hình ảnh sản phẩm.
- Lưu bản ghi hoặc dữ liệu phục vụ kiểm tra.
- Sao chép dữ liệu sang vùng dự phòng khi cần thiết.

#### Amazon EC2

Trong giai đoạn đầu, Amazon EC2 được sử dụng để chạy backend FastAPI. Backend được đóng gói bằng Docker để tạo môi trường chạy thống nhất và thuận tiện cho việc triển khai.

#### Amazon ECS và AWS Fargate

Trong kiến trúc mở rộng, backend có thể được chuyển từ EC2 sang Amazon ECS với AWS Fargate. Cách triển khai này giúp quản lý container thuận tiện hơn và hỗ trợ mở rộng số lượng tác vụ theo nhu cầu.

#### Amazon ECR

Amazon ECR lưu trữ Docker image của backend. Quy trình triển khai có thể build image mới, đẩy lên ECR và cập nhật dịch vụ đang chạy.

#### Elastic Load Balancing

Application Load Balancer phân phối yêu cầu đến các container backend, thực hiện kiểm tra tình trạng và hạn chế việc phụ thuộc vào một máy chủ duy nhất.

#### Amazon API Gateway

Amazon API Gateway cung cấp điểm truy cập cho REST API và WebSocket API. REST API xử lý các yêu cầu nghiệp vụ, còn WebSocket hỗ trợ gửi dữ liệu đấu giá theo thời gian thực.

#### AWS Lambda

AWS Lambda có thể đảm nhiệm:

- Xử lý một số API độc lập.
- Kiểm tra và cập nhật trạng thái phiên đấu giá.
- Xử lý yêu cầu đặt giá.
- Kết thúc phiên đấu giá khi hết thời gian.
- Gửi thông báo cho người dùng.
- Thực hiện công việc nền hoặc công việc theo lịch.

#### Amazon RDS for MySQL

Trong phiên bản triển khai ban đầu, Amazon RDS for MySQL lưu trữ các dữ liệu có quan hệ như:

- Tài khoản người dùng.
- Thông tin sản phẩm.
- Phiên đấu giá.
- Lịch sử đặt giá.
- Thông tin thanh toán và giao dịch.

RDS giúp nhóm giảm công việc quản trị máy chủ cơ sở dữ liệu và hỗ trợ sao lưu dữ liệu.

#### Amazon Aurora

Trong kiến trúc mục tiêu có yêu cầu mở rộng và dự phòng đa vùng, Amazon Aurora có thể được nghiên cứu để thay thế cơ sở dữ liệu quan hệ ban đầu. Đây là thành phần định hướng mở rộng, chưa phải cơ sở dữ liệu đang được sử dụng trong phiên bản hiện tại.

#### Amazon DynamoDB

DynamoDB được đề xuất cho các dữ liệu cần truy cập nhanh và có tần suất cập nhật cao như:

- Trạng thái hiện tại của phiên đấu giá.
- Các sự kiện đặt giá.
- Trạng thái giữ số dư.
- Danh sách kết nối WebSocket.

DynamoDB Streams có thể kích hoạt các tác vụ xử lý tiếp theo khi dữ liệu thay đổi.

#### Amazon SQS FIFO

Amazon SQS FIFO được đề xuất để tiếp nhận yêu cầu đặt giá và xử lý theo thứ tự. Thành phần này giúp hạn chế xung đột khi nhiều người cùng đặt giá trong một khoảng thời gian rất ngắn.

#### Amazon EventBridge

Amazon EventBridge hỗ trợ lập lịch thời điểm bắt đầu hoặc kết thúc phiên đấu giá và chuyển các sự kiện trong hệ thống đến đúng thành phần xử lý.

#### Amazon Kinesis Data Streams

Kinesis Data Streams có thể tiếp nhận luồng sự kiện đấu giá nhằm phục vụ phân tích, thống kê và giám sát hoạt động gần thời gian thực.

#### Amazon Cognito

Amazon Cognito được đề xuất để quản lý đăng ký, đăng nhập và xác thực người dùng trong kiến trúc mở rộng. Trong phiên bản hiện tại, hệ thống vẫn sử dụng cơ chế xác thực JWT do backend quản lý.

#### Amazon CloudWatch và AWS CloudTrail

- **Amazon CloudWatch** thu thập log, metric và thiết lập cảnh báo.
- **AWS CloudTrail** ghi nhận các thao tác quản trị được thực hiện trên tài khoản AWS.

#### AWS IAM, AWS KMS và AWS Secrets Manager

- **AWS IAM** phân quyền truy cập theo nguyên tắc đặc quyền tối thiểu.
- **AWS KMS** quản lý khóa mã hóa dữ liệu.
- **AWS Secrets Manager** lưu trữ thông tin nhạy cảm như thông tin kết nối cơ sở dữ liệu và khóa bí mật.

### 5. Thiết kế các thành phần hệ thống

#### Frontend

Frontend được xây dựng bằng React/Vite và cung cấp các giao diện:

- Đăng ký và đăng nhập.
- Xem danh sách sản phẩm.
- Xem chi tiết phiên đấu giá.
- Đặt giá và theo dõi giá hiện tại.
- Đăng sản phẩm.
- Quản lý thông tin cá nhân.
- Quản lý hệ thống dành cho quản trị viên.

Sau khi chạy lệnh build, ứng dụng tạo thư mục `dist/`. Các tệp trong thư mục này được tải lên Amazon S3 để phân phối đến người dùng.

#### Backend

Backend sử dụng FastAPI và cung cấp API cho:

- Xác thực người dùng.
- Quản lý tài khoản.
- Quản lý sản phẩm.
- Quản lý phiên đấu giá.
- Tiếp nhận và kiểm tra yêu cầu đặt giá.
- Quản lý hình ảnh.
- Quản lý thông báo.
- Chức năng quản trị.

Backend được đóng gói bằng Docker để bảo đảm môi trường chạy nhất quán giữa máy phát triển và AWS.

#### Cơ sở dữ liệu

Dữ liệu ban đầu được quản lý bằng MySQL trong Docker. Khi triển khai lên AWS, cơ sở dữ liệu được chuyển sang Amazon RDS for MySQL.

Backend kết nối đến RDS thông qua thông tin cấu hình được lưu trong biến môi trường hoặc AWS Secrets Manager. Security Group chỉ cho phép backend được truy cập cổng cơ sở dữ liệu.

#### Lưu trữ hình ảnh

Hình ảnh sản phẩm không được lưu trực tiếp trong cơ sở dữ liệu. Backend tải hình ảnh lên S3 và lưu đường dẫn hoặc object key trong cơ sở dữ liệu. Cách tổ chức này giúp giảm kích thước cơ sở dữ liệu và thuận tiện khi mở rộng dung lượng lưu trữ.

#### Xử lý đấu giá thời gian thực

Trong phiên bản ban đầu, backend kiểm tra yêu cầu đặt giá và cập nhật dữ liệu trong cơ sở dữ liệu. Trong kiến trúc mở rộng, yêu cầu đặt giá có thể được đưa vào SQS FIFO để bảo đảm thứ tự, sau đó được Lambda hoặc dịch vụ backend xử lý.

WebSocket API gửi giá mới và trạng thái phiên đấu giá đến các trình duyệt đang kết nối, giúp người dùng không phải tải lại toàn bộ trang.

### 6. Kế hoạch triển khai kỹ thuật

#### Giai đoạn 1: Phân tích và thiết kế

- Phân tích yêu cầu của hệ thống đấu giá.
- Xác định vai trò người mua, người bán và quản trị viên.
- Thiết kế cơ sở dữ liệu.
- Thiết kế cây thư mục mã nguồn.
- Xây dựng sơ đồ kiến trúc AWS.

#### Giai đoạn 2: Phát triển ứng dụng

- Xây dựng giao diện bằng React/Vite.
- Xây dựng API bằng FastAPI.
- Thiết lập MySQL bằng Docker.
- Phát triển chức năng xác thực bằng JWT.
- Phát triển chức năng sản phẩm và đấu giá.
- Tích hợp lưu trữ hình ảnh.

#### Giai đoạn 3: Triển khai phiên bản ban đầu

- Build frontend bằng `npm run build`.
- Tải thư mục `dist/` lên Amazon S3.
- Bật Static Website Hosting cho frontend.
- Đóng gói backend bằng Docker.
- Triển khai backend lên Amazon EC2.
- Khởi tạo Amazon RDS for MySQL.
- Cấu hình kết nối từ EC2 đến RDS.
- Tạo S3 bucket lưu trữ hình ảnh.
- Cấu hình IAM role và Security Group.
- Kiểm tra kết nối giữa các thành phần.

#### Giai đoạn 4: Kiểm thử

- Kiểm thử đăng ký và đăng nhập.
- Kiểm thử đăng sản phẩm.
- Kiểm thử đặt giá.
- Kiểm thử quyền truy cập của từng vai trò.
- Kiểm thử tải lên và hiển thị hình ảnh.
- Kiểm thử khi nhiều người dùng cùng đặt giá.
- Kiểm tra log và chi phí tài nguyên AWS.

#### Giai đoạn 5: Nghiên cứu mở rộng

- Tích hợp CloudFront và AWS WAF.
- Nghiên cứu WebSocket API.
- Đưa yêu cầu đặt giá vào SQS FIFO.
- Nghiên cứu DynamoDB cho trạng thái thời gian thực.
- Chuyển backend container sang ECS Fargate.
- Xây dựng quy trình CI/CD.
- Nghiên cứu dự phòng đa vùng.

### 7. Yêu cầu kỹ thuật

#### Công nghệ phát triển

- Frontend: React, Vite, TypeScript hoặc JavaScript.
- Backend: Python và FastAPI.
- Cơ sở dữ liệu: MySQL.
- Container: Docker và Docker Compose.
- Quản lý mã nguồn: Git và GitHub.
- Công cụ thiết kế kiến trúc: diagrams.net.

#### Yêu cầu bảo mật

- Không sử dụng tài khoản AWS root cho công việc hằng ngày.
- Mỗi thành viên chỉ được cấp các quyền cần thiết.
- Không đưa Access Key hoặc mật khẩu vào GitHub.
- Thông tin kết nối cơ sở dữ liệu được lưu bằng biến môi trường hoặc Secrets Manager.
- S3 bucket hình ảnh cần có chính sách truy cập phù hợp.
- Security Group của RDS không được mở công khai.
- Backend phải kiểm tra token và vai trò người dùng.
- Dữ liệu cần được truyền qua HTTPS khi đưa hệ thống vào vận hành chính thức.

### 8. Lộ trình và mốc triển khai

- **Tuần 1:** Làm quen với môi trường thực tập và tìm hiểu tổng quan về AWS.
- **Tuần 2:** Nghiên cứu các dịch vụ AWS phổ biến.
- **Tuần 3:** Thực hành trên AWS Management Console và tìm hiểu chi phí.
- **Tuần 4:** Phân tích yêu cầu và thiết kế kiến trúc hệ thống.
- **Tuần 5:** Phát triển và tích hợp các chức năng chính.
- **Tuần 6:** Triển khai các thành phần của hệ thống lên AWS.
- **Tuần 7:** Kiểm thử, sửa lỗi và tổng duyệt đồ án.
- **Tuần 8:** Hoàn thiện đồ án, workshop và báo cáo thực tập.

### 9. Ước tính ngân sách

Chi phí thực tế phụ thuộc vào khu vực triển khai, lưu lượng truy cập, dung lượng dữ liệu và thời gian vận hành. Các thành phần có khả năng phát sinh chi phí gồm:

| Dịch vụ | Mục đích | Yếu tố ảnh hưởng chi phí |
| --- | --- | --- |
| Amazon S3 | Frontend và hình ảnh | Dung lượng, request và truyền dữ liệu |
| Amazon EC2 | Backend FastAPI | Loại instance và thời gian vận hành |
| Amazon RDS | Cơ sở dữ liệu MySQL | Loại instance, dung lượng và backup |
| AWS Lambda | Tác vụ nền | Số lần gọi và thời gian xử lý |
| Amazon CloudFront | Phân phối nội dung | Dung lượng truyền ra Internet |
| API Gateway | REST API và WebSocket | Số lượng request và thời gian kết nối |
| CloudWatch | Log và giám sát | Dung lượng log và metric |
| Route 53 | Tên miền và định tuyến | Hosted zone và DNS query |

Trong phạm vi đồ án sinh viên, nhóm ưu tiên cấu hình tài nguyên có quy mô nhỏ, tắt hoặc xóa tài nguyên không sử dụng và thiết lập AWS Budgets để kiểm soát chi phí.

Số liệu chi phí chính xác sẽ được tính lại bằng [AWS Pricing Calculator](https://calculator.aws/) sau khi xác định cấu hình và thời gian vận hành của từng tài nguyên.

### 10. Đánh giá rủi ro

| Rủi ro | Mức ảnh hưởng | Biện pháp giảm thiểu |
| --- | --- | --- |
| Nhiều người đặt giá đồng thời | Cao | Transaction, khóa dữ liệu hoặc SQS FIFO |
| Backend EC2 gặp sự cố | Cao | Health check, backup và kiến trúc nhiều instance |
| Cơ sở dữ liệu mất kết nối | Cao | RDS backup, kiểm tra kết nối và Multi-AZ khi cần |
| Hình ảnh không truy cập được | Trung bình | Kiểm tra S3 policy, CORS và object key |
| Lộ thông tin đăng nhập AWS | Cao | IAM role, Secrets Manager và không commit khóa |
| Chi phí vượt dự kiến | Trung bình | AWS Budgets, cảnh báo và xóa tài nguyên thừa |
| WebSocket bị ngắt | Trung bình | Cơ chế kết nối lại và lưu trạng thái kết nối |
| Vùng chính gặp sự cố | Cao | Route 53 failover và vùng dự phòng |
| Dữ liệu đặt giá sai thứ tự | Cao | Hàng đợi FIFO và kiểm tra trạng thái phiên |

### 11. Kế hoạch dự phòng

- Sao lưu định kỳ dữ liệu RDS.
- Bật versioning cho S3 bucket quan trọng.
- Lưu Docker image ổn định trong Amazon ECR.
- Theo dõi lỗi thông qua CloudWatch.
- Chuẩn bị cấu hình khôi phục backend từ Docker image.
- Hạn chế thay đổi trực tiếp trên môi trường đang vận hành.
- Nghiên cứu Infrastructure as Code bằng Terraform hoặc AWS CloudFormation.
- Xây dựng vùng dự phòng khi hệ thống cần độ sẵn sàng cao.

### 12. Kết quả kỳ vọng

Sau khi hoàn thành, hệ thống dự kiến đạt được các kết quả:

- Cung cấp website đấu giá có thể truy cập qua Internet.
- Cho phép người dùng đăng ký, đăng nhập và quản lý tài khoản.
- Cho phép người bán đăng sản phẩm và tạo phiên đấu giá.
- Cho phép người mua xem và đặt giá cho sản phẩm.
- Lưu trữ dữ liệu có cấu trúc trên Amazon RDS.
- Lưu trữ hình ảnh sản phẩm trên Amazon S3.
- Triển khai frontend và backend trên nền tảng AWS.
- Theo dõi hoạt động và lỗi của hệ thống.
- Hình thành nền tảng để phát triển khả năng đấu giá thời gian thực.
- Giúp các thành viên nâng cao kỹ năng phát triển phần mềm, làm việc nhóm, triển khai hệ thống và sử dụng các dịch vụ AWS.

Kiến trúc mở rộng trong sơ đồ là định hướng phát triển dài hạn. Phiên bản đầu tiên ưu tiên hoàn thiện các chức năng cốt lõi và triển khai ổn định với Amazon S3, Amazon EC2, Amazon RDS và AWS Lambda trước khi tích hợp thêm các dịch vụ nâng cao.