### 5.5.5. Kiểm thử kết nối WebSocket và cập nhật thời gian thực

#### Mục tiêu kiểm thử

Phần này kiểm tra khả năng kết nối và cập nhật dữ liệu thời gian thực của hệ thống đấu giá thông qua:

* Amazon API Gateway WebSocket API.
* Lambda WebSocket Handler.
* Các route `$connect`, `$disconnect` và `$default`.
* Route tham gia phòng đấu giá và gửi thông điệp.
* Bảng quản lý kết nối trong Amazon DynamoDB.
* Lambda Broadcast.
* API Gateway Management API.
* Amazon CloudWatch Logs.
* Frontend của hệ thống đấu giá.

Các test case được đánh mã từ `WS-01` đến `WS-13`.

Một test case WebSocket chỉ được đánh dấu `PASS` khi kiểm tra được đồng thời:

1. Client nhận đúng trạng thái kết nối hoặc thông điệp.
2. WebSocket Handler thực hiện đúng nghiệp vụ.
3. Connection ID được lưu, cập nhật hoặc xóa đúng trong DynamoDB.
4. Lambda Broadcast gửi đúng người và đúng phòng đấu giá.
5. Kết nối lỗi hoặc hết hạn không làm thất bại toàn bộ quá trình broadcast.
6. Không có dữ liệu riêng của phòng đấu giá bị gửi nhầm sang người dùng khác.
7. Có bằng chứng trực tiếp từ trình duyệt, DynamoDB hoặc CloudWatch Logs.

---

#### Phạm vi kiểm thử

Các thành phần được kiểm tra gồm:

* User Frontend.
* Amazon API Gateway WebSocket API.
* Route `$connect`.
* Route `$disconnect`.
* Route `$default`.
* Route tham gia hoặc rời phòng đấu giá.
* Lambda WebSocket Handler.
* Lambda Broadcast.
* API Gateway Management API.
* DynamoDB Connections Table.
* DynamoDB Auction Room hoặc Subscription Table nếu được tách riêng.
* Amazon CloudWatch Logs.
* Amazon Cognito hoặc cơ chế xác thực WebSocket của hệ thống.

---

#### Điều kiện kiểm thử chung

Trước khi thực hiện kiểm thử, hệ thống cần đáp ứng các điều kiện sau:

* WebSocket API đã được triển khai trên API Gateway.
* WebSocket URL của môi trường kiểm thử đã được cấu hình trên frontend.
* Các route `$connect`, `$disconnect` và `$default` đã được liên kết với đúng Lambda.
* Các route nghiệp vụ như `join_room`, `leave_room` hoặc `send_message` đã được triển khai nếu kiến trúc sử dụng route riêng.
* Lambda có quyền đọc, ghi và xóa dữ liệu trong bảng kết nối DynamoDB.
* Lambda Broadcast có quyền gọi `execute-api:ManageConnections`.
* CloudWatch Logs đã được bật cho các Lambda liên quan.
* Có ít nhất hai tài khoản User hợp lệ.
* Có ít nhất hai vật phẩm hoặc phòng đấu giá khác nhau.
* Có thể mở hai cửa sổ trình duyệt hoặc hai phiên trình duyệt độc lập.
* Có thể kiểm tra trực tiếp bản ghi trong DynamoDB.
* Môi trường kiểm thử được tách khỏi dữ liệu production.
* Đồng hồ trên thiết bị kiểm thử được đồng bộ để đối chiếu thời gian log.

Nếu Lambda, route, bảng DynamoDB hoặc chức năng frontend liên quan chưa được triển khai, test case phải được đánh dấu `BLOCKED`.

---

#### Dữ liệu kiểm thử

| Dữ liệu                 | Mô tả                                                              |
| ----------------------- | ------------------------------------------------------------------ |
| User A                  | Tài khoản hợp lệ tham gia vật phẩm A                               |
| User B                  | Tài khoản hợp lệ tham gia cùng vật phẩm A                          |
| User C                  | Tài khoản hợp lệ nhưng chỉ tham gia vật phẩm B                     |
| Auction Item A          | Vật phẩm có phòng WebSocket hợp lệ                                 |
| Auction Item B          | Vật phẩm khác với Item A                                           |
| Room A                  | Phòng WebSocket của Auction Item A                                 |
| Room B                  | Phòng WebSocket của Auction Item B                                 |
| Connection ID hợp lệ    | Connection ID đang hoạt động do API Gateway cấp                    |
| Connection ID hết hạn   | Connection ID của client đã đóng hoặc mất kết nối                  |
| Thông điệp hợp lệ       | JSON có đúng `action`, `roomId` và các trường bắt buộc             |
| Thông điệp không hợp lệ | JSON lỗi cú pháp, thiếu trường hoặc chứa action không hỗ trợ       |
| Sự kiện hợp lệ          | Cập nhật trạng thái, giá đấu, số người xem hoặc thông báo hệ thống |
| Token hợp lệ            | Token chưa hết hạn và thuộc tài khoản hợp lệ                       |
| Token không hợp lệ      | Token sai chữ ký, hết hạn hoặc không đúng định dạng                |

Không được đưa Access Token, ID Token, Refresh Token hoặc header xác thực vào ảnh chụp và báo cáo.

---

#### Quy ước trạng thái và cấu trúc thông điệp

WebSocket không sử dụng HTTP status cho tất cả thông điệp sau khi kết nối được thiết lập. Vì vậy, kết quả cần được kiểm tra ở hai giai đoạn:

* Giai đoạn bắt tay kết nối: kiểm tra HTTP status của WebSocket handshake.
* Giai đoạn đã kết nối: kiểm tra WebSocket message và trạng thái kết nối.

Các kết quả handshake thông dụng:

|                   Kết quả | Ý nghĩa                                                |
| ------------------------: | ------------------------------------------------------ |
| `101 Switching Protocols` | Kết nối WebSocket được thiết lập thành công            |
|        `401 Unauthorized` | Không có thông tin xác thực hoặc xác thực không hợp lệ |
|           `403 Forbidden` | Người dùng đã xác thực nhưng không được phép kết nối   |
|           Kết nối bị đóng | API Gateway hoặc Lambda từ chối hoặc kết thúc kết nối  |

Thông điệp thành công nên có cấu trúc nhất quán, ví dụ:

```json
{
  "type": "AUCTION_STATUS_UPDATED",
  "roomId": "auction-item-a",
  "data": {
    "status": "ACTIVE"
  },
  "timestamp": "2026-08-09T12:00:00Z"
}
```

Thông điệp lỗi nên có cấu trúc mà frontend có thể xử lý:

```json
{
  "type": "ERROR",
  "error": {
    "code": "INVALID_MESSAGE_FORMAT",
    "message": "The WebSocket message is invalid"
  },
  "timestamp": "2026-08-09T12:00:00Z"
}
```

Thông điệp gửi đến client không được chứa:

* Access Token.
* ID Token.
* Refresh Token.
* AWS credentials.
* Connection ID của người dùng khác.
* Stack trace.
* Tên bảng DynamoDB.
* Chi tiết hạ tầng nội bộ không cần thiết.
* Dữ liệu cá nhân của người dùng khác.

---

#### WS-01 — User kết nối WebSocket thành công

| Trường                   | Nội dung                                                                                                                                                                                                                                                  |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `WS-01`                                                                                                                                                                                                                                                   |
| **Tên kiểm thử**         | User hợp lệ kết nối WebSocket thành công                                                                                                                                                                                                                  |
| **Mục tiêu**             | Xác minh User đã xác thực có thể thiết lập kết nối với API Gateway WebSocket.                                                                                                                                                                             |
| **Điều kiện tiên quyết** | WebSocket API và route `$connect` đã được triển khai; User A có thông tin xác thực hợp lệ.                                                                                                                                                                |
| **Các bước thực hiện**   | 1. Đăng nhập bằng User A.<br>2. Mở trang chi tiết Auction Item A.<br>3. Mở Network tab và chọn bộ lọc WebSocket.<br>4. Quan sát quá trình WebSocket handshake.<br>5. Kiểm tra trạng thái kết nối trên frontend.<br>6. Kiểm tra log của `$connect` Lambda. |
| **Dữ liệu đầu vào**      | WebSocket URL hợp lệ và thông tin xác thực hợp lệ của User A.                                                                                                                                                                                             |
| **Kết quả mong đợi**     | Handshake trả `101 Switching Protocols`; frontend chuyển sang trạng thái Connected hoặc Live; `$connect` Lambda được gọi đúng một lần; không xuất hiện reconnect liên tục; log không chứa token.                                                          |
| **Kết quả thực tế**      | Điền HTTP status, trạng thái kết nối, thời gian kết nối và Request ID thực tế.                                                                                                                                                                            |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                                            |
| **Bằng chứng**           | Network tab, trạng thái Live trên frontend và CloudWatch Logs của `$connect`.                                                                                                                                                                             |

---

#### WS-02 — Kết nối không hợp lệ bị từ chối

| Trường                   | Nội dung                                                                                                                                                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Mã kiểm thử**          | `WS-02`                                                                                                                                                                                                                  |
| **Tên kiểm thử**         | Từ chối kết nối WebSocket không hợp lệ                                                                                                                                                                                   |
| **Mục tiêu**             | Xác minh người dùng không có thông tin xác thực hợp lệ không thể thiết lập kết nối.                                                                                                                                      |
| **Điều kiện tiên quyết** | `$connect` có kiểm tra xác thực hoặc Authorizer đã được cấu hình.                                                                                                                                                        |
| **Các bước thực hiện**   | 1. Thử kết nối không có thông tin xác thực.<br>2. Thử kết nối bằng token sai định dạng.<br>3. Thử kết nối bằng token hết hạn.<br>4. Ghi nhận kết quả handshake.<br>5. Kiểm tra DynamoDB.<br>6. Kiểm tra CloudWatch Logs. |
| **Dữ liệu đầu vào**      | Không có token, token không hợp lệ hoặc token hết hạn.                                                                                                                                                                   |
| **Kết quả mong đợi**     | Kết nối bị từ chối bằng `401`, `403` hoặc bị đóng theo hợp đồng hệ thống; frontend không hiển thị trạng thái Live; không lưu Connection ID hoạt động trong DynamoDB; log ghi error code nhưng không ghi nội dung token.  |
| **Kết quả thực tế**      | Điền từng loại dữ liệu thử nghiệm và kết quả tương ứng.                                                                                                                                                                  |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                           |
| **Bằng chứng**           | Kết quả handshake, DynamoDB không có bản ghi kết nối hợp lệ và CloudWatch Logs đã che dữ liệu nhạy cảm.                                                                                                                  |

> Nếu token được truyền qua query string, nhóm phải xác minh token không bị ghi vào access log, browser history hoặc ảnh chụp bằng chứng. Không được công khai WebSocket URL chứa token.

---

#### WS-03 — Sự kiện `$connect` lưu Connection ID

| Trường                   | Nội dung                                                                                                                                                                                                                                                                                              |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `WS-03`                                                                                                                                                                                                                                                                                               |
| **Tên kiểm thử**         | Lưu thông tin kết nối khi `$connect` thành công                                                                                                                                                                                                                                                       |
| **Mục tiêu**             | Xác minh `$connect` Lambda lưu Connection ID và thông tin người dùng chính xác trong DynamoDB.                                                                                                                                                                                                        |
| **Điều kiện tiên quyết** | User A có thể kết nối; Lambda có quyền ghi vào Connections Table.                                                                                                                                                                                                                                     |
| **Các bước thực hiện**   | 1. Ghi nhận dữ liệu trong bảng trước khi kết nối.<br>2. User A mở trang Auction Item A.<br>3. Xác nhận kết nối thành công.<br>4. Tìm bản ghi mới trong DynamoDB.<br>5. Đối chiếu thời gian tạo, User ID và Connection ID với CloudWatch Logs.<br>6. Kiểm tra thuộc tính hết hạn nếu bảng sử dụng TTL. |
| **Dữ liệu đầu vào**      | Kết nối hợp lệ của User A.                                                                                                                                                                                                                                                                            |
| **Kết quả mong đợi**     | Một bản ghi kết nối được tạo; Connection ID không rỗng; User ID được lấy từ danh tính đã xác minh; `connectedAt` được ghi đúng; TTL nằm trong tương lai nếu có; không lưu token trong DynamoDB.                                                                                                       |
| **Kết quả thực tế**      | Điền Connection ID đã che một phần, User ID, thời gian tạo và TTL.                                                                                                                                                                                                                                    |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                                                                                        |
| **Bằng chứng**           | Bản ghi DynamoDB và CloudWatch Logs của `$connect`.                                                                                                                                                                                                                                                   |

---

#### WS-04 — User tham gia đúng phòng đấu giá

| Trường                   | Nội dung                                                                                                                                                                                                                                                                       |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Mã kiểm thử**          | `WS-04`                                                                                                                                                                                                                                                                        |
| **Tên kiểm thử**         | Tham gia phòng WebSocket của đúng vật phẩm                                                                                                                                                                                                                                     |
| **Mục tiêu**             | Xác minh User A được liên kết với Room A khi mở Auction Item A.                                                                                                                                                                                                                |
| **Điều kiện tiên quyết** | User A đã kết nối; Auction Item A tồn tại; route tham gia phòng đã được triển khai.                                                                                                                                                                                            |
| **Các bước thực hiện**   | 1. User A mở trang Auction Item A.<br>2. Frontend gửi thông điệp `join_room` nếu kiến trúc yêu cầu.<br>3. Kiểm tra thông điệp phản hồi.<br>4. Kiểm tra bản ghi kết nối trong DynamoDB.<br>5. Kiểm tra log của WebSocket Handler.<br>6. Phát một sự kiện thử nghiệm vào Room A. |
| **Dữ liệu đầu vào**      | Room ID hoặc Auction Item ID của Item A.                                                                                                                                                                                                                                       |
| **Kết quả mong đợi**     | Kết nối của User A được gắn với Room A; Lambda xác minh Room A tồn tại; client nhận xác nhận tham gia; sự kiện của Room A được gửi đến User A; không tạo nhiều subscription trùng cho cùng kết nối và phòng.                                                                   |
| **Kết quả thực tế**      | Điền Room ID, phản hồi nhận được và dữ liệu DynamoDB.                                                                                                                                                                                                                          |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                                                                 |
| **Bằng chứng**           | WebSocket frame, bản ghi DynamoDB và CloudWatch Logs.                                                                                                                                                                                                                          |

---

#### WS-05 — Hai người dùng cùng tham gia một vật phẩm

| Trường                   | Nội dung                                                                                                                                                                                                                                                                                                           |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Mã kiểm thử**          | `WS-05`                                                                                                                                                                                                                                                                                                            |
| **Tên kiểm thử**         | Hai User cùng tham gia Room A                                                                                                                                                                                                                                                                                      |
| **Mục tiêu**             | Xác minh hệ thống quản lý đồng thời nhiều kết nối trong cùng một phòng đấu giá.                                                                                                                                                                                                                                    |
| **Điều kiện tiên quyết** | Có User A và User B; có thể mở hai phiên trình duyệt độc lập.                                                                                                                                                                                                                                                      |
| **Các bước thực hiện**   | 1. Mở Auction Item A bằng User A trong cửa sổ thứ nhất.<br>2. Mở cùng Item A bằng User B trong cửa sổ thứ hai.<br>3. Xác nhận cả hai kết nối đều ở trạng thái Live.<br>4. Kiểm tra số người đang xem nếu frontend hỗ trợ.<br>5. Kiểm tra các bản ghi trong DynamoDB.<br>6. Phát một sự kiện thử nghiệm vào Room A. |
| **Dữ liệu đầu vào**      | Hai User khác nhau và cùng Room A.                                                                                                                                                                                                                                                                                 |
| **Kết quả mong đợi**     | Hai Connection ID khác nhau được liên kết với Room A; số người xem là `2` nếu có chức năng viewer count; cả hai cửa sổ nhận được sự kiện của Room A; không ghi đè kết nối của nhau.                                                                                                                                |
| **Kết quả thực tế**      | Điền số kết nối, số người xem và thông điệp nhận được ở từng cửa sổ.                                                                                                                                                                                                                                               |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                                                                                                     |
| **Bằng chứng**           | Ảnh hai cửa sổ hoạt động đồng thời, WebSocket frames, DynamoDB và log broadcast.                                                                                                                                                                                                                                   |

---

#### WS-06 — User gửi thông điệp hợp lệ

| Trường                   | Nội dung                                                                                                                                                                                                                                                                                     |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `WS-06`                                                                                                                                                                                                                                                                                      |
| **Tên kiểm thử**         | Xử lý WebSocket message hợp lệ                                                                                                                                                                                                                                                               |
| **Mục tiêu**             | Xác minh WebSocket Handler nhận, xác thực và xử lý đúng thông điệp hợp lệ.                                                                                                                                                                                                                   |
| **Điều kiện tiên quyết** | User A đã kết nối và tham gia Room A.                                                                                                                                                                                                                                                        |
| **Các bước thực hiện**   | 1. User A gửi một thông điệp có action được hỗ trợ.<br>2. Kiểm tra frame đã gửi.<br>3. Kiểm tra phản hồi của server.<br>4. Kiểm tra CloudWatch Logs.<br>5. Nếu thông điệp gây broadcast, kiểm tra kết quả ở User B.<br>6. Nếu thông điệp thay đổi dữ liệu, kiểm tra dữ liệu nguồn tương ứng. |
| **Dữ liệu đầu vào**      | JSON hợp lệ với đúng `action`, `roomId` và các trường bắt buộc.                                                                                                                                                                                                                              |
| **Kết quả mong đợi**     | Handler đọc đúng action; xác minh User và Room; xử lý nghiệp vụ một lần; client nhận ACK hoặc kết quả phù hợp; các User liên quan nhận đúng sự kiện; không để client tự xác định danh tính hoặc quyền hạn bằng dữ liệu trong message.                                                        |
| **Kết quả thực tế**      | Điền message type, phản hồi và kết quả broadcast thực tế.                                                                                                                                                                                                                                    |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                                                                               |
| **Bằng chứng**           | WebSocket frames đã che dữ liệu nhạy cảm, CloudWatch Logs và dữ liệu liên quan.                                                                                                                                                                                                              |

---

#### WS-07 — Thông điệp sai định dạng bị từ chối

| Trường                   | Nội dung                                                                                                                                                                                                                                           |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `WS-07`                                                                                                                                                                                                                                            |
| **Tên kiểm thử**         | Từ chối WebSocket message không hợp lệ                                                                                                                                                                                                             |
| **Mục tiêu**             | Xác minh Lambda xử lý an toàn khi message không phải JSON hợp lệ, thiếu trường hoặc chứa action không hỗ trợ.                                                                                                                                      |
| **Điều kiện tiên quyết** | User A đã kết nối WebSocket.                                                                                                                                                                                                                       |
| **Các bước thực hiện**   | 1. Gửi chuỗi không phải JSON.<br>2. Gửi JSON thiếu `action`.<br>3. Gửi action không được hỗ trợ.<br>4. Gửi message thiếu `roomId` khi trường này bắt buộc.<br>5. Gửi trường sai kiểu dữ liệu.<br>6. Kiểm tra phản hồi, log và dữ liệu sau mỗi lần. |
| **Dữ liệu đầu vào**      | Message sai cú pháp hoặc vi phạm schema.                                                                                                                                                                                                           |
| **Kết quả mong đợi**     | Server trả message lỗi có cấu trúc nhất quán; không thực hiện nghiệp vụ; không broadcast message không hợp lệ; không thay đổi dữ liệu ngoài ý muốn; Lambda không bị lỗi chưa xử lý; kết nối có thể được giữ hoặc đóng theo hợp đồng đã định nghĩa. |
| **Kết quả thực tế**      | Điền từng message kiểm thử, error code và trạng thái kết nối sau lỗi.                                                                                                                                                                              |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                                     |
| **Bằng chứng**           | Frame gửi và nhận, DynamoDB hoặc dữ liệu nghiệp vụ không thay đổi, CloudWatch Logs.                                                                                                                                                                |

---

#### WS-08 — Trạng thái đấu giá được gửi đến tất cả người tham gia

| Trường                   | Nội dung                                                                                                                                                                                                                                                                  |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `WS-08`                                                                                                                                                                                                                                                                   |
| **Tên kiểm thử**         | Broadcast cập nhật trạng thái đến toàn bộ Room A                                                                                                                                                                                                                          |
| **Mục tiêu**             | Xác minh Lambda Broadcast gửi cập nhật trạng thái đấu giá đến mọi kết nối đang hoạt động trong đúng phòng.                                                                                                                                                                |
| **Điều kiện tiên quyết** | User A và User B đang tham gia Room A; Lambda Broadcast và Management API đã sẵn sàng.                                                                                                                                                                                    |
| **Các bước thực hiện**   | 1. Mở hai cửa sổ tại Auction Item A.<br>2. Thực hiện nghiệp vụ thay đổi trạng thái hoặc tạo sự kiện hợp lệ.<br>3. Ghi nhận thời điểm phát sự kiện.<br>4. Quan sát message tại cả hai cửa sổ.<br>5. Kiểm tra giao diện được cập nhật.<br>6. Kiểm tra log Lambda Broadcast. |
| **Dữ liệu đầu vào**      | Sự kiện như `AUCTION_STATUS_UPDATED`, `BID_UPDATED` hoặc `VIEWER_COUNT_UPDATED`.                                                                                                                                                                                          |
| **Kết quả mong đợi**     | Cả User A và User B nhận cùng loại sự kiện, Room ID và dữ liệu mới; frontend cập nhật mà không cần tải lại trang; không gửi trùng ngoài số lần được thiết kế; log cho biết tổng số kết nối mục tiêu, số lần thành công và thất bại.                                       |
| **Kết quả thực tế**      | Điền message nhận được ở từng cửa sổ và độ trễ quan sát được.                                                                                                                                                                                                             |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                                                            |
| **Bằng chứng**           | Ảnh hai cửa sổ, WebSocket messages và CloudWatch Logs của Lambda Broadcast.                                                                                                                                                                                               |

---

#### WS-09 — Một User rời trang hoặc ngắt kết nối

| Trường                   | Nội dung                                                                                                                                                                                                                                                        |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `WS-09`                                                                                                                                                                                                                                                         |
| **Tên kiểm thử**         | User rời phòng đấu giá                                                                                                                                                                                                                                          |
| **Mục tiêu**             | Xác minh hệ thống xử lý đúng khi một User đóng trang, chuyển trang hoặc mất kết nối.                                                                                                                                                                            |
| **Điều kiện tiên quyết** | User A và User B đang cùng tham gia Room A.                                                                                                                                                                                                                     |
| **Các bước thực hiện**   | 1. Xác nhận hai User đang kết nối.<br>2. Đóng tab của User B hoặc chuyển khỏi trang Item A.<br>3. Quan sát trạng thái trên cửa sổ User A.<br>4. Kiểm tra viewer count hoặc thông báo rời phòng nếu có.<br>5. Kiểm tra CloudWatch Logs.<br>6. Kiểm tra DynamoDB. |
| **Dữ liệu đầu vào**      | Hành động đóng tab, chuyển trang hoặc ngắt mạng của User B.                                                                                                                                                                                                     |
| **Kết quả mong đợi**     | User B không tiếp tục được xem là thành viên hoạt động của Room A; viewer count giảm từ `2` xuống `1` nếu có; User A nhận sự kiện cập nhật tương ứng; hệ thống không ảnh hưởng đến kết nối của User A.                                                          |
| **Kết quả thực tế**      | Điền trạng thái hai client, viewer count và thời điểm phát hiện ngắt kết nối.                                                                                                                                                                                   |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                                                  |
| **Bằng chứng**           | Ảnh trước và sau khi rời trang, frame của User A, DynamoDB và CloudWatch Logs.                                                                                                                                                                                  |

> Khi thiết bị mất mạng đột ngột, `$disconnect` có thể không được xử lý ngay. Nếu hệ thống sử dụng heartbeat hoặc TTL, cần ghi rõ khoảng thời gian tối đa để phát hiện và dọn kết nối.

---

#### WS-10 — Sự kiện `$disconnect` xóa hoặc vô hiệu hóa kết nối

| Trường                   | Nội dung                                                                                                                                                                                                                                                                            |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `WS-10`                                                                                                                                                                                                                                                                             |
| **Tên kiểm thử**         | Dọn bản ghi kết nối khi `$disconnect` xảy ra                                                                                                                                                                                                                                        |
| **Mục tiêu**             | Xác minh `$disconnect` Lambda xóa hoặc đánh dấu không hoạt động đối với Connection ID đã ngắt.                                                                                                                                                                                      |
| **Điều kiện tiên quyết** | User B có Connection ID đang được lưu trong DynamoDB.                                                                                                                                                                                                                               |
| **Các bước thực hiện**   | 1. Ghi nhận bản ghi User B trước khi ngắt kết nối.<br>2. Đóng kết nối WebSocket của User B.<br>3. Kiểm tra log `$disconnect`.<br>4. Đọc lại bản ghi trong DynamoDB.<br>5. Thực hiện broadcast tiếp theo tới Room A.<br>6. Kiểm tra danh sách kết nối được Lambda Broadcast sử dụng. |
| **Dữ liệu đầu vào**      | Connection ID đang hoạt động của User B.                                                                                                                                                                                                                                            |
| **Kết quả mong đợi**     | `$disconnect` nhận đúng Connection ID; bản ghi bị xóa hoặc chuyển sang trạng thái inactive theo thiết kế; User B không còn trong danh sách broadcast; thao tác dọn dẹp có tính idempotent và không lỗi khi chạy lại.                                                                |
| **Kết quả thực tế**      | Điền trạng thái bản ghi trước và sau `$disconnect`.                                                                                                                                                                                                                                 |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                                                                      |
| **Bằng chứng**           | CloudWatch Logs của `$disconnect`, bản ghi DynamoDB trước và sau và log lần broadcast kế tiếp.                                                                                                                                                                                      |

---

#### WS-11 — Kết nối hết hạn không làm broadcast thất bại toàn bộ

| Trường                   | Nội dung                                                                                                                                                                                                                                                          |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `WS-11`                                                                                                                                                                                                                                                           |
| **Tên kiểm thử**         | Cô lập lỗi của một kết nối hết hạn                                                                                                                                                                                                                                |
| **Mục tiêu**             | Xác minh một Connection ID không còn hợp lệ không ngăn Lambda gửi dữ liệu đến các kết nối còn hoạt động.                                                                                                                                                          |
| **Điều kiện tiên quyết** | Room A có ít nhất một kết nối hợp lệ và một Connection ID đã hết hạn hoặc không còn tồn tại.                                                                                                                                                                      |
| **Các bước thực hiện**   | 1. Chuẩn bị User A đang kết nối.<br>2. Tạo điều kiện để bản ghi cũ của User B vẫn còn trong DynamoDB.<br>3. Phát một sự kiện đến Room A.<br>4. Quan sát message tại User A.<br>5. Kiểm tra kết quả Lambda Broadcast.<br>6. Kiểm tra log lỗi cho Connection ID cũ. |
| **Dữ liệu đầu vào**      | Một Connection ID hợp lệ và một Connection ID hết hạn trong cùng Room A.                                                                                                                                                                                          |
| **Kết quả mong đợi**     | User A vẫn nhận được sự kiện; Lambda xử lý lỗi theo từng Connection ID; một lần gửi thất bại không kết thúc toàn bộ vòng broadcast; kết quả log thể hiện ít nhất một lần thành công và một lần thất bại; Lambda không trả lỗi toàn bộ chỉ vì một kết nối cũ.      |
| **Kết quả thực tế**      | Điền số kết nối mục tiêu, số lần gửi thành công, thất bại và kết quả tại User A.                                                                                                                                                                                  |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                                                    |
| **Bằng chứng**           | Message của User A, CloudWatch Logs và bản ghi Connection ID hết hạn.                                                                                                                                                                                             |

---

#### WS-12 — `GoneException` được xử lý và dọn khỏi DynamoDB

| Trường                   | Nội dung                                                                                                                                                                                                                                                                                |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `WS-12`                                                                                                                                                                                                                                                                                 |
| **Tên kiểm thử**         | Dọn kết nối cũ khi Management API trả `GoneException`                                                                                                                                                                                                                                   |
| **Mục tiêu**             | Xác minh Lambda Broadcast nhận biết lỗi HTTP `410 Gone` và xóa Connection ID không còn hợp lệ.                                                                                                                                                                                          |
| **Điều kiện tiên quyết** | DynamoDB còn một Connection ID của kết nối đã đóng; Lambda Broadcast có quyền xóa hoặc cập nhật bản ghi.                                                                                                                                                                                |
| **Các bước thực hiện**   | 1. Xác định một Connection ID đã hết hạn.<br>2. Xác nhận bản ghi vẫn tồn tại trước broadcast.<br>3. Kích hoạt Lambda Broadcast.<br>4. Kiểm tra log của lệnh `postToConnection`.<br>5. Kiểm tra việc bắt `GoneException`.<br>6. Đọc lại DynamoDB.<br>7. Kích hoạt broadcast lần thứ hai. |
| **Dữ liệu đầu vào**      | Connection ID không còn tồn tại trong API Gateway nhưng vẫn còn trong DynamoDB.                                                                                                                                                                                                         |
| **Kết quả mong đợi**     | Management API trả lỗi tương đương `410 Gone`; Lambda bắt lỗi mà không dừng toàn bộ broadcast; bản ghi kết nối cũ bị xóa hoặc vô hiệu hóa; lần broadcast tiếp theo không tiếp tục gửi đến Connection ID đó.                                                                             |
| **Kết quả thực tế**      | Điền mã lỗi, hành động dọn dẹp và trạng thái bản ghi sau xử lý.                                                                                                                                                                                                                         |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                                                                          |
| **Bằng chứng**           | CloudWatch Logs có `GoneException` hoặc `410`, DynamoDB trước và sau và log lần broadcast tiếp theo.                                                                                                                                                                                    |

> Không được coi mọi lỗi từ Management API là kết nối hết hạn. Chỉ những lỗi xác định kết nối không còn tồn tại, chẳng hạn `GoneException`, mới được dùng làm căn cứ xóa bản ghi.

---

#### WS-13 — Người ngoài phòng không nhận dữ liệu riêng của phòng

| Trường                   | Nội dung                                                                                                                                                                                                                                                                                  |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `WS-13`                                                                                                                                                                                                                                                                                   |
| **Tên kiểm thử**         | Cô lập dữ liệu giữa các phòng đấu giá                                                                                                                                                                                                                                                     |
| **Mục tiêu**             | Xác minh sự kiện của Room A chỉ được gửi đến các kết nối thuộc Room A.                                                                                                                                                                                                                    |
| **Điều kiện tiên quyết** | User A và User B tham gia Room A; User C tham gia Room B.                                                                                                                                                                                                                                 |
| **Các bước thực hiện**   | 1. Mở Room A bằng User A và User B.<br>2. Mở Room B bằng User C.<br>3. Xác nhận ba kết nối hoạt động.<br>4. Phát một sự kiện chỉ thuộc Room A.<br>5. Kiểm tra message tại cả ba cửa sổ.<br>6. Kiểm tra truy vấn DynamoDB của Lambda Broadcast.<br>7. Lặp lại bằng một sự kiện của Room B. |
| **Dữ liệu đầu vào**      | Sự kiện riêng của Auction Item A và sự kiện riêng của Auction Item B.                                                                                                                                                                                                                     |
| **Kết quả mong đợi**     | User A và User B nhận sự kiện của Room A; User C không nhận sự kiện đó; User C chỉ nhận sự kiện của Room B; Lambda truy vấn kết nối theo đúng Room ID; không có dữ liệu giá đấu, trạng thái hoặc thông tin riêng của phiên bị gửi chéo.                                                   |
| **Kết quả thực tế**      | Điền message nhận được hoặc không nhận được ở từng cửa sổ.                                                                                                                                                                                                                                |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                                                                            |
| **Bằng chứng**           | Ảnh ba cửa sổ hoặc các WebSocket frames, truy vấn/log broadcast và dữ liệu subscription trong DynamoDB.                                                                                                                                                                                   |

---

#### Ma trận phân phối sự kiện cần kiểm tra

| Người dùng         | Phòng đã tham gia      | Sự kiện Room A           | Sự kiện Room B                  |
| ------------------ | ---------------------- | ------------------------ | ------------------------------- |
| User A             | Room A                 | Phải nhận                | Không được nhận                 |
| User B             | Room A                 | Phải nhận                | Không được nhận                 |
| User C             | Room B                 | Không được nhận          | Phải nhận                       |
| Kết nối đã hết hạn | Room A                 | Gửi thất bại và được dọn | Không áp dụng                   |
| User đã rời Room A | Không còn subscription | Không được nhận          | Chỉ nhận nếu đã tham gia Room B |

---

#### Quy định kiểm tra DynamoDB Connections Table

Đối với bảng quản lý kết nối, cần kiểm tra:

* Connection ID được lấy từ `requestContext.connectionId`.
* User ID được lấy từ danh tính đã xác minh.
* Không tin tưởng `userId`, `role` hoặc `connectionId` do client gửi trong message.
* Mỗi kết nối được liên kết với đúng User.
* Mỗi subscription được liên kết với đúng Room ID hoặc Auction Item ID.
* Hai tab của cùng một User có thể có hai Connection ID khác nhau.
* Kết nối bị đóng được xóa hoặc vô hiệu hóa.
* TTL được thiết lập đúng nếu hệ thống sử dụng cơ chế tự động dọn dữ liệu.
* Không lưu Access Token, ID Token hoặc Refresh Token.
* Không ghi đè kết nối của người dùng khác.
* Không tồn tại subscription trùng ngoài thiết kế.
* `GoneException` dẫn đến việc dọn Connection ID tương ứng.
* Broadcast chỉ truy vấn kết nối thuộc phòng mục tiêu.
* Lỗi của một kết nối không làm mất cập nhật của các kết nối còn lại.

Nếu bảng sử dụng thiết kế single-table, nhóm phải kiểm tra đúng partition key, sort key và các index dùng để truy vấn kết nối theo Room ID.

---

#### Quy định kiểm tra Lambda Broadcast

Lambda Broadcast phải được kiểm tra theo các tiêu chí sau:

* Nhận đúng Room ID và loại sự kiện.
* Truy vấn đúng danh sách kết nối của phòng.
* Không broadcast dựa hoàn toàn vào danh sách Connection ID do client cung cấp.
* Gửi dữ liệu bằng đúng WebSocket API endpoint và stage.
* Message có cấu trúc JSON hợp lệ.
* Không gửi dữ liệu nhạy cảm.
* Tiếp tục xử lý khi một kết nối gửi thất bại.
* Bắt và xử lý `GoneException`.
* Dọn Connection ID hết hạn.
* Ghi nhận số kết nối mục tiêu.
* Ghi nhận số lần gửi thành công và thất bại.
* Không ghi toàn bộ token hoặc dữ liệu cá nhân vào log.
* Có cơ chế giới hạn kích thước message.
* Không gửi trùng sự kiện ngoài thiết kế.
* Không để lỗi của một phòng ảnh hưởng đến phòng khác.

---

#### Quy định kiểm tra CloudWatch Logs

CloudWatch Logs nên chứa các thông tin cần thiết để truy vết:

* Request ID.
* Route key như `$connect`, `$disconnect`, `$default` hoặc `join_room`.
* Connection ID đã che một phần khi đưa vào báo cáo.
* User ID hoặc Cognito `sub` đã xác minh.
* Room ID hoặc Auction Item ID.
* Loại message hoặc loại sự kiện.
* Số kết nối mục tiêu.
* Số lần gửi thành công.
* Số lần gửi thất bại.
* Error code như `INVALID_MESSAGE_FORMAT` hoặc `GONE_CONNECTION`.
* Thời gian xử lý.
* Kết quả dọn bản ghi kết nối cũ.

Không được ghi:

* Access Token.
* ID Token.
* Refresh Token.
* Header hoặc query parameter chứa thông tin xác thực.
* Mật khẩu.
* AWS Access Key ID.
* AWS Secret Access Key.
* AWS Session Token.
* Toàn bộ WebSocket message nếu message chứa dữ liệu nhạy cảm.
* Stack trace trong response gửi về client.

---

#### Bảng tổng hợp kết quả

| Mã      | Nội dung kiểm thử            | Kết quả chính mong đợi                  | Kiểm tra DynamoDB   | Trạng thái    |
| ------- | ---------------------------- | --------------------------------------- | ------------------- | ------------- |
| `WS-01` | User kết nối thành công      | Handshake `101`, frontend hiển thị Live | Có                  | Chưa kiểm thử |
| `WS-02` | Kết nối không hợp lệ         | Bị từ chối, không có kết nối hoạt động  | Có                  | Chưa kiểm thử |
| `WS-03` | `$connect` lưu Connection ID | Bản ghi kết nối được tạo chính xác      | Bắt buộc            | Chưa kiểm thử |
| `WS-04` | User tham gia đúng phòng     | Connection được gắn với đúng Room ID    | Bắt buộc            | Chưa kiểm thử |
| `WS-05` | Hai User cùng một vật phẩm   | Hai kết nối cùng nhận sự kiện           | Bắt buộc            | Chưa kiểm thử |
| `WS-06` | Gửi message hợp lệ           | Handler xử lý và phản hồi đúng          | Khi có thay đổi     | Chưa kiểm thử |
| `WS-07` | Message sai định dạng        | Bị từ chối, không broadcast             | Phải không thay đổi | Chưa kiểm thử |
| `WS-08` | Broadcast trạng thái         | Tất cả thành viên phòng nhận cập nhật   | Có                  | Chưa kiểm thử |
| `WS-09` | User rời trang               | Không còn được xem là kết nối hoạt động | Có                  | Chưa kiểm thử |
| `WS-10` | `$disconnect` dọn kết nối    | Xóa hoặc vô hiệu hóa Connection ID      | Bắt buộc            | Chưa kiểm thử |
| `WS-11` | Có kết nối hết hạn           | Các kết nối hợp lệ vẫn nhận dữ liệu     | Có                  | Chưa kiểm thử |
| `WS-12` | Xử lý `GoneException`        | Kết nối cũ được dọn khỏi bảng           | Bắt buộc            | Chưa kiểm thử |
| `WS-13` | Cô lập giữa các phòng        | Người ngoài phòng không nhận dữ liệu    | Có                  | Chưa kiểm thử |

---

#### Bằng chứng kiểm thử

Bằng chứng cho phần kiểm thử WebSocket bao gồm:

* Hai hoặc ba cửa sổ trình duyệt hoạt động đồng thời.
* Network tab hiển thị WebSocket handshake.
* WebSocket frames được gửi và nhận.
* Trạng thái Live hoặc Connected trên frontend.
* Số người đang xem trước và sau khi User tham gia hoặc rời trang.
* CloudWatch Logs của `$connect`.
* CloudWatch Logs của `$disconnect`.
* CloudWatch Logs của WebSocket Handler.
* CloudWatch Logs của Lambda Broadcast.
* Bản ghi Connection ID trong DynamoDB.
* Bản ghi subscription giữa Connection ID và Room ID.
* Dữ liệu DynamoDB trước và sau khi ngắt kết nối.
* Log thể hiện `GoneException` hoặc HTTP `410`.
* Message nhận được ở từng cửa sổ.
* Request ID dùng để đối chiếu giữa API Gateway và Lambda.

Mỗi bằng chứng cần ghi rõ mã test case, ví dụ:

```text
Hình 5.5.5.1: WebSocket handshake thành công của test case WS-01
Hình 5.5.5.2: Connection ID được lưu trong DynamoDB sau WS-03
Hình 5.5.5.3: Hai cửa sổ cùng nhận sự kiện của Room A trong WS-08
Hình 5.5.5.4: Log GoneException và kết quả dọn kết nối trong WS-12
Hình 5.5.5.5: User C không nhận sự kiện của Room A trong WS-13
```

Trong ảnh chụp bằng chứng phải che:

* Token.
* Thông tin xác thực trong WebSocket URL.
* Header xác thực.
* AWS account ID nếu không cần công khai.
* Connection ID đầy đủ nếu tài liệu được chia sẻ công khai.
* Dữ liệu cá nhân không liên quan đến kiểm thử.

---

#### Tiêu chí đánh giá

Test case chỉ được đánh dấu `PASS` khi:

* Kết quả thực tế phù hợp với kết quả mong đợi.
* Client nhận đúng message hoặc bị từ chối đúng trường hợp.
* Connection ID được lưu và dọn chính xác.
* Subscription thuộc đúng phòng đấu giá.
* Broadcast gửi đúng người, đúng phòng và đúng nội dung.
* Người ngoài phòng không nhận dữ liệu.
* Kết nối hết hạn không làm thất bại toàn bộ broadcast.
* `GoneException` được xử lý và Connection ID cũ được dọn.
* Không có dữ liệu ngoài ý muốn bị thay đổi.
* Có bằng chứng trực tiếp.

Test case phải được đánh dấu `FAIL` khi:

* Frontend hiển thị Live nhưng Connection ID không được lưu.
* `$connect` thành công với token không hợp lệ.
* Connection ID của User này ghi đè kết nối của User khác.
* User tham gia Room A nhưng được lưu vào Room B.
* Một User trong phòng không nhận được sự kiện hợp lệ.
* Người không thuộc phòng vẫn nhận dữ liệu riêng của phòng.
* Message sai định dạng vẫn được xử lý hoặc broadcast.
* Một kết nối hết hạn làm Lambda dừng toàn bộ broadcast.
* `GoneException` xảy ra nhưng bản ghi cũ không được dọn.
* `$disconnect` xóa nhầm kết nối khác.
* Frontend phải tải lại trang mới nhận được dữ liệu được thiết kế là thời gian thực.
* Log hoặc bằng chứng làm lộ token hay thông tin xác thực.

Test case được đánh dấu `BLOCKED` khi:

* WebSocket API chưa được triển khai.
* Route cần kiểm thử chưa được cấu hình.
* Lambda Handler hoặc Lambda Broadcast chưa tồn tại.
* Bảng kết nối hoặc index theo Room ID chưa được tạo.
* Frontend chưa tích hợp WebSocket.
* Không có quyền kiểm tra DynamoDB hoặc CloudWatch Logs.
* Chức năng tương ứng chưa được triển khai.
