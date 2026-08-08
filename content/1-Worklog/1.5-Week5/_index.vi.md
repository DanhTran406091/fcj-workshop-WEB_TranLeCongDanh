---
title: "Worklog Tuần 5"
date: 2026-07-20
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Thời gian:

**Từ ngày 20/07/2026 đến ngày 24/07/2026**

### Mục tiêu tuần 5:

* Bắt đầu phát triển các chức năng cốt lõi của hệ thống Live Auction.
* Xây dựng giao diện người dùng và các chức năng quản lý cơ bản.
* Phát triển API và xử lý nghiệp vụ cho hệ thống.
* Thiết kế cấu trúc dữ liệu phục vụ sản phẩm, phiên đấu giá và lượt đặt giá.
* Nghiên cứu phương án tích hợp mã nguồn với các dịch vụ AWS.
* Theo dõi tiến độ và tích hợp mã nguồn giữa các thành viên trong nhóm.

### Các công việc đã thực hiện:

| Thứ | Ngày | Công việc | Nguồn tài liệu |
| --- | --- | --- | --- |
| Thứ Hai | 20/07/2026 | Thống nhất cấu trúc mã nguồn, quy ước phát triển và phân chia chức năng cho các thành viên; chuẩn bị môi trường phát triển frontend và backend. | Tài liệu kỹ thuật của nhóm |
| Thứ Ba | 21/07/2026 | Phát triển giao diện người dùng bằng React/Vite; xây dựng các trang đăng nhập, danh sách sản phẩm, thông tin sản phẩm và danh sách phiên đấu giá. | <https://react.dev/learn> |
| Thứ Tư | 22/07/2026 | Phát triển API và logic nghiệp vụ cho các chức năng quản lý người dùng, sản phẩm và phiên đấu giá; kiểm tra dữ liệu đầu vào và phản hồi từ API. | <https://fastapi.tiangolo.com/tutorial/> |
| Thứ Năm | 23/07/2026 | Thiết kế cấu trúc dữ liệu cho người dùng, sản phẩm, phiên đấu giá và lịch sử đặt giá; nghiên cứu cách chuyển đổi dữ liệu sang Amazon DynamoDB. | <https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html> |
| Thứ Sáu | 24/07/2026 | Tích hợp các chức năng đã hoàn thành; kiểm tra luồng hoạt động từ giao diện đến API; tổng hợp lỗi và xác định các chức năng cần tiếp tục hoàn thiện. | Mã nguồn và tài liệu kiểm thử của nhóm |

### Kết quả đạt được tuần 5:

* Xây dựng được cấu trúc mã nguồn ban đầu cho hệ thống Live Auction.
* Thống nhất quy trình phát triển và tích hợp mã nguồn giữa các thành viên.
* Hoàn thành bước đầu giao diện của một số chức năng chính:

  * Đăng ký và đăng nhập.
  * Hiển thị danh sách sản phẩm.
  * Xem thông tin chi tiết sản phẩm.
  * Hiển thị danh sách phiên đấu giá.
  * Tạo và quản lý sản phẩm đấu giá.
  * Theo dõi trạng thái của phiên đấu giá.

* Xây dựng bước đầu các API phục vụ:

  * Quản lý người dùng.
  * Quản lý sản phẩm.
  * Quản lý phiên đấu giá.
  * Tiếp nhận yêu cầu đặt giá.
  * Truy xuất lịch sử đặt giá.

* Thiết kế được các nhóm dữ liệu chính của hệ thống:

  * Thông tin người dùng.
  * Thông tin sản phẩm.
  * Thông tin phiên đấu giá.
  * Trạng thái phiên đấu giá.
  * Lịch sử và mức giá đặt của người tham gia.

* Kết nối bước đầu giữa frontend và backend thông qua API.
* Kiểm tra được luồng thao tác cơ bản từ giao diện người dùng đến thành phần xử lý nghiệp vụ.
* Nghiên cứu phương án chuyển đổi các chức năng backend sang AWS Lambda.
* Nghiên cứu cách lưu trữ dữ liệu thời gian thực trên Amazon DynamoDB.
* Xác định nhu cầu sử dụng API Gateway WebSocket để cập nhật trạng thái đấu giá theo thời gian thực.
* Tổng hợp được các lỗi và chức năng cần hoàn thiện trong tuần tiếp theo.