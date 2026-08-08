---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Xây dựng và triển khai hệ thống Live Auction trên nền tảng AWS

#### Tổng quan

**Live Auction** là nền tảng đấu giá trực tuyến cho phép người dùng đăng ký tài khoản, theo dõi các phiên đấu giá và thực hiện đặt giá theo thời gian thực.

Trong workshop này, nhóm trình bày quá trình triển khai hệ thống **Live Auction** trên nền tảng **Amazon Web Services (AWS)**. Hạ tầng của hệ thống được triển khai bằng **Terraform**, giúp tự động hóa việc tạo và quản lý các tài nguyên AWS, đồng thời đảm bảo tính nhất quán trong quá trình triển khai.

Frontend của người dùng và trang quản trị được lưu trữ trên **Amazon S3** và phân phối thông qua **Amazon CloudFront**. Chức năng xác thực người dùng sử dụng **Amazon Cognito**. Các API nghiệp vụ được triển khai bằng **AWS Lambda** kết hợp với **Amazon API Gateway**.

Đối với chức năng đấu giá thời gian thực, hệ thống sử dụng **API Gateway WebSocket**, **Amazon SQS FIFO** và **Amazon DynamoDB** nhằm tiếp nhận yêu cầu đặt giá, xử lý theo đúng thứ tự và đồng bộ trạng thái phiên đấu giá đến các người dùng đang kết nối.

Workshop tập trung vào quá trình chuẩn bị môi trường, triển khai hạ tầng bằng Terraform, cấu hình các dịch vụ AWS, kiểm thử hệ thống và đánh giá kết quả sau khi hoàn thành.

#### Nội dung

1. [Giới thiệu đồ án và kiến trúc triển khai](5.1-Workshop-overview/)
2. [Chuẩn bị môi trường](5.2-Preparation/)
3. [Triển khai hạ tầng bằng Terraform](5.3-Infrastructure/)
4. [Các dịch vụ AWS được triển khai](5.4-AWS-Services/)
5. [Kiểm thử hệ thống](5.5-Testing/)
6. [Kết quả đạt được](5.6-Conclusion/)