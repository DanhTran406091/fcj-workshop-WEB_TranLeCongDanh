---
title: "Các bài blog đã đăng"
date: 2026-08-03
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Trong quá trình thực tập, em cùng các thành viên trong nhóm nghiên cứu và chia sẻ những kiến thức đã học được về các dịch vụ AWS trên cộng đồng [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj).

Hiện tại, nhóm em đã thực hiện bài blog sau:

### [Blog 1 - Triển khai website React/Vite với Amazon S3](3.1-Blog1/)

Bài viết giới thiệu cách sử dụng **Amazon S3 Static Website Hosting** để triển khai một website tĩnh được xây dựng bằng React/Vite. Nội dung trình bày quy trình build ứng dụng, tải các tệp trong thư mục `dist` lên S3, cấu hình quyền truy cập và sử dụng S3 Website Endpoint để truy cập website.

Bài viết cũng chia sẻ một số lỗi thường gặp, lưu ý về bảo mật và những hạn chế cần biết khi triển khai website trực tiếp bằng Amazon S3.

### [Blog 2 - Lambda scale nhanh, nhưng Database không scale theo cùng cách](3.2-Blog2/)

Bài viết phân tích vấn đề quản lý kết nối cơ sở dữ liệu khi chuyển backend từ mô hình chạy lâu dài trên **EC2 hoặc container** sang kiến trúc **AWS Lambda**, trong khi dữ liệu vẫn được lưu trữ trên **Amazon RDS for MySQL**.

Nội dung làm rõ nguy cơ số lượng kết nối tăng đột ngột khi Lambda mở rộng theo lượng yêu cầu đồng thời, đặc biệt trong thời điểm cuối phiên đấu giá. Bài viết đồng thời giới thiệu **Reserved Concurrency** và **Amazon RDS Proxy** như những giải pháp giúp bảo vệ cơ sở dữ liệu, tái sử dụng kết nối và giảm áp lực kết nối trực tiếp lên Amazon RDS.

### [Blog 3 - Tối ưu chi phí lưu trữ trên Amazon S3: Tìm hiểu về Storage Classes và Lifecycle Policy](3.3-Blog3/)

Bài viết giới thiệu các lớp lưu trữ phổ biến của **Amazon S3**, bao gồm S3 Standard, Standard-IA, One Zone-IA và các lớp S3 Glacier. Nội dung phân tích cách lựa chọn lớp lưu trữ phù hợp dựa trên tần suất truy cập, khả năng phục hồi và thời gian cần thiết để truy xuất dữ liệu.

Bài viết đồng thời trình bày cách sử dụng **S3 Lifecycle Policy** để tự động chuyển object sang lớp lưu trữ có chi phí thấp hơn hoặc xóa dữ liệu khi hết thời hạn lưu giữ. Qua đó, hệ thống có thể tối ưu chi phí lưu trữ mà không cần xây dựng cron job hoặc Lambda Function riêng.