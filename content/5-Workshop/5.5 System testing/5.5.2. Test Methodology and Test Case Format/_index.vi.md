### 5.5.2. Phương pháp và định dạng test case

#### Phương pháp kiểm thử

Nhóm thực hiện kiểm thử hệ thống theo phương pháp **kiểm thử hộp đen**, kết hợp với việc kiểm tra dữ liệu và nhật ký trên các dịch vụ AWS.

Đối với mỗi test case, nhóm cung cấp dữ liệu đầu vào và thực hiện thao tác thông qua:

- User Frontend.
- Admin Frontend.
- REST API.
- Kết nối WebSocket.
- AWS Management Console.
- Công cụ kiểm thử API như Postman hoặc `curl`.
- Công cụ kiểm thử tải nếu có.

Sau đó, kết quả thực tế được so sánh với kết quả mong đợi để xác định trạng thái của test case.

Quy trình kiểm thử được thực hiện theo trình tự:

```text
Chuẩn bị môi trường và dữ liệu kiểm thử
1. Xác nhận điều kiện tiên quyết
2. Thực hiện các bước kiểm thử
3. Quan sát kết quả trên frontend hoặc API
4. Kiểm tra dữ liệu và log trên AWS
5. So sánh với kết quả mong đợi
6. Ghi nhận trạng thái
7. Lưu bằng chứng kiểm thử
```

Việc đánh giá không chỉ dựa trên nội dung hiển thị ở frontend. Tùy theo test case, nhóm còn kiểm tra:

- HTTP status và response body của REST API.
- Thông điệp gửi và nhận qua WebSocket.
- Bản ghi được tạo hoặc cập nhật trong Amazon DynamoDB.
- Trạng thái message trong Amazon SQS FIFO.
- CloudWatch Logs của AWS Lambda.
- CloudWatch Metrics của Lambda, API Gateway, DynamoDB và SQS.
- Object hoặc phiên bản object trong Amazon S3.
- Nội dung được phân phối qua Amazon CloudFront.
- Trạng thái người dùng và nhóm quyền trong Amazon Cognito.

#### Nguyên tắc thực hiện kiểm thử

Quá trình kiểm thử tuân theo các nguyên tắc sau:

1. Mỗi test case chỉ tập trung xác minh một hành vi hoặc một điều kiện cụ thể.
2. Test case phải có điều kiện tiên quyết và dữ liệu đầu vào rõ ràng.
3. Các bước thực hiện phải đủ chi tiết để thành viên khác có thể kiểm tra lại.
4. Kết quả mong đợi phải xác định được và có thể đo lường hoặc quan sát.
5. Kết quả thực tế phải được ghi nhận dựa trên hành vi thật của hệ thống.
6. Test case chỉ được đánh dấu `PASS` khi kết quả thực tế phù hợp với kết quả mong đợi.
7. Nếu kết quả không đúng, test case phải được đánh dấu `FAIL` và ghi rõ lỗi quan sát được.
8. Nếu chưa thể kiểm thử do thiếu chức năng, tài nguyên hoặc dữ liệu phụ thuộc, test case phải được đánh dấu `BLOCKED`.
9. Một test case `PASS` phải có bằng chứng kiểm thử phù hợp.
10. Không thay đổi kết quả mong đợi sau khi thực hiện kiểm thử chỉ để test case được đánh dấu `PASS`.
11. Dữ liệu do test case trước tạo ra không được làm sai lệch kết quả của test case sau.
12. Không đưa token, mật khẩu, AWS Secret Access Key hoặc thông tin nhạy cảm vào báo cáo.

#### Phân loại test case

Các test case được chia thành các nhóm tương ứng với chức năng và thành phần của hệ thống:


| Tiền tố    | Nhóm kiểm thử                         | Ví dụ         |
| ---------- | ------------------------------------- | ------------- |
| `AUTH`     | Xác thực và phân quyền                | `AUTH-01`     |
| `API`      | REST API và nghiệp vụ quản lý đấu giá | `API-01`      |
| `WS`       | WebSocket và cập nhật thời gian thực  | `WS-01`       |
| `BID`      | Luồng đặt giá đầu cuối                | `BID-01`      |
| `FIFO`     | Thứ tự và xử lý message bằng SQS FIFO | `FIFO-01`     |
| `DB`       | DynamoDB và tính toàn vẹn dữ liệu     | `DB-01`       |
| `STORAGE`  | Amazon S3 và CloudFront               | `STORAGE-01`  |
| `RECOVERY` | Xử lý lỗi và khả năng phục hồi        | `RECOVERY-01` |
| `PERF`     | Hiệu năng và tải đồng thời            | `PERF-01`     |
| `SEC`      | Bảo mật hệ thống                      | `SEC-01`      |


Mã kiểm thử phải là duy nhất trong toàn bộ tài liệu. Cấu trúc mã được quy định như sau:

```text
<TIỀN_TỐ_NHÓM>-<SỐ_THỨ_TỰ>
```

Ví dụ:

```text
AUTH-01
API-03
WS-05
BID-07
FIFO-02
SEC-04
```

#### Định dạng test case

Mỗi test case được ghi nhận theo mẫu sau:


| Trường                   | Nội dung                                                                                   |
| ------------------------ | ------------------------------------------------------------------------------------------ |
| **Mã kiểm thử**          | Mã duy nhất của test case, ví dụ: `AUTH-01`.                                               |
| **Tên kiểm thử**         | Tên ngắn gọn mô tả trường hợp cần kiểm tra.                                                |
| **Mục tiêu**             | Chức năng, hành vi hoặc điều kiện mà test case cần xác minh.                               |
| **Điều kiện tiên quyết** | Tài khoản, dữ liệu, cấu hình và trạng thái hệ thống cần có trước khi kiểm thử.             |
| **Các bước thực hiện**   | Trình tự thao tác cụ thể để thực hiện test case.                                           |
| **Dữ liệu đầu vào**      | Token, tài khoản, mật khẩu kiểm thử, ID phiên, ID vật phẩm, mức giá hoặc nội dung request. |
| **Kết quả mong đợi**     | Hành vi đúng mà hệ thống phải trả về sau khi thực hiện.                                    |
| **Kết quả thực tế**      | Kết quả được quan sát trên frontend, API hoặc các dịch vụ AWS.                             |
| **Trạng thái**           | Kết quả đánh giá: `PASS`, `FAIL` hoặc `BLOCKED`.                                           |
| **Bằng chứng**           | Ảnh giao diện, response API, WebSocket message, CloudWatch Logs, Metrics hoặc dữ liệu AWS. |




#### Ví dụ test case đã ghi nhận

##### AUTH-01 — Đăng nhập bằng tài khoản User hợp lệ


| Trường                   | Nội dung                                                                                                                                       |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `AUTH-01`                                                                                                                                      |
| **Tên kiểm thử**         | Đăng nhập bằng tài khoản User hợp lệ                                                                                                           |
| **Mục tiêu**             | Xác minh người dùng đã được xác nhận có thể đăng nhập vào User Frontend.                                                                       |
| **Điều kiện tiên quyết** | Tài khoản User đã tồn tại và có trạng thái xác nhận trong Amazon Cognito.                                                                      |
| **Các bước thực hiện**   | 1. Truy cập User Frontend. 2. Mở trang đăng nhập. 3. Nhập email và mật khẩu hợp lệ. 4. Nhấn nút đăng nhập. 5. Quan sát kết quả trên giao diện. |
| **Dữ liệu đầu vào**      | Email và mật khẩu của tài khoản User kiểm thử.                                                                                                 |
| **Kết quả mong đợi**     | Amazon Cognito xác thực thành công; người dùng được chuyển đến trang chính; frontend hiển thị trạng thái đã đăng nhập.                         |
| **Kết quả thực tế**      | Điền kết quả quan sát được sau khi thực hiện kiểm thử.                                                                                         |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                 |
| **Bằng chứng**           | Ảnh giao diện sau khi đăng nhập và log liên quan đã được che thông tin nhạy cảm.                                                               |


#### Quy ước trạng thái


| Trạng thái | Điều kiện ghi nhận                                                                                          |
| ---------- | ----------------------------------------------------------------------------------------------------------- |
| `PASS`     | Kết quả thực tế phù hợp hoàn toàn với kết quả mong đợi và có bằng chứng xác nhận.                           |
| `FAIL`     | Kết quả thực tế khác kết quả mong đợi, hệ thống phát sinh lỗi hoặc dữ liệu được xử lý không chính xác.      |
| `BLOCKED`  | Không thể hoàn thành kiểm thử do thiếu dữ liệu, chức năng, cấu hình, quyền truy cập hoặc dịch vụ phụ thuộc. |


Khi test case có trạng thái `FAIL`, nhóm cần ghi nhận thêm:

- Bước xảy ra lỗi.
- Thời điểm xảy ra lỗi.
- Thông báo lỗi quan sát được.
- HTTP status nếu liên quan đến REST API.
- Mã request hoặc request ID nếu có.
- Lambda Function liên quan.
- CloudWatch Log Group hoặc Log Stream liên quan.
- Ảnh hưởng của lỗi đến hệ thống.
- Hướng xử lý hoặc nhiệm vụ sửa lỗi.

Khi test case có trạng thái `BLOCKED`, nhóm phải ghi rõ:

- Thành phần đang thiếu.
- Chức năng phụ thuộc chưa hoàn thành.
- Cấu hình chưa được triển khai.
- Quyền truy cập chưa được cấp.
- Điều kiện cần hoàn thành trước khi kiểm thử lại.

#### Quy định ghi nhận kết quả thực tế

Trường **Kết quả thực tế** phải mô tả kết quả đã quan sát được, không được sao chép nguyên văn từ trường **Kết quả mong đợi** nếu chưa thực hiện kiểm thử.

Ví dụ ghi nhận phù hợp:

```text
API trả về HTTP 200. Response chứa sessionId, itemId,
currentPrice và status = ACTIVE. Dữ liệu tương ứng tồn tại
trong bảng Auctions của DynamoDB.
```

Ví dụ ghi nhận không phù hợp:

```text
Hoạt động đúng.
```

Nếu test case thất bại, kết quả thực tế cần nêu rõ lỗi:

```text
API trả về HTTP 500 thay vì HTTP 400 khi trường bidAmount bị thiếu.
CloudWatch Logs ghi nhận lỗi KeyError tại Lambda la-ws-handler.
```

#### Quy định thu thập bằng chứng

Bằng chứng kiểm thử phải liên quan trực tiếp đến mục tiêu của test case. Tùy từng trường hợp, nhóm sử dụng một hoặc nhiều loại bằng chứng sau:


| Loại bằng chứng          | Trường hợp sử dụng                                                |
| ------------------------ | ----------------------------------------------------------------- |
| Ảnh User Frontend        | Chứng minh chức năng dành cho người dùng hoạt động.               |
| Ảnh Admin Frontend       | Chứng minh chức năng quản trị và phân quyền.                      |
| HTTP request và response | Chứng minh REST API trả về status và dữ liệu đúng.                |
| WebSocket message        | Chứng minh dữ liệu được gửi và nhận theo thời gian thực.          |
| CloudWatch Logs          | Chứng minh Lambda được kích hoạt và xử lý nghiệp vụ.              |
| CloudWatch Metrics       | Chứng minh số request, lỗi, độ trễ hoặc lượng message được xử lý. |
| DynamoDB item            | Chứng minh dữ liệu được tạo hoặc cập nhật chính xác.              |
| SQS Metrics              | Chứng minh message được gửi, nhận và xóa khỏi Queue.              |
| Nội dung DLQ             | Chứng minh message thất bại được chuyển vào Dead-letter Queue.    |
| S3 object                | Chứng minh tệp được tải lên và lưu đúng bucket.                   |
| CloudFront response      | Chứng minh frontend hoặc nội dung tĩnh được phân phối thành công. |
| Cognito User Pool        | Chứng minh trạng thái tài khoản hoặc nhóm quyền.                  |


Mỗi hình ảnh nên có tiêu đề và mô tả rõ test case tương ứng, ví dụ:

```text
Hình 5.5.2.1: Kết quả thực hiện test case AUTH-01
```

Không nên sử dụng một ảnh chung cho nhiều test case nếu ảnh đó không chứng minh rõ kết quả của từng trường hợp.

#### Quản lý dữ liệu kiểm thử

Để kết quả có thể được kiểm tra lại, nhóm cần chuẩn bị dữ liệu kiểm thử trước khi thực hiện:

- Tài khoản User đã được xác nhận.
- Tài khoản Admin thuộc đúng nhóm quyền.
- Tài khoản User không có quyền Admin.
- Phiên đấu giá ở trạng thái `SCHEDULED`.
- Phiên đấu giá ở trạng thái `ACTIVE`.
- Phiên đấu giá ở trạng thái `ENDED`.
- Vật phẩm có giá khởi điểm và bước giá tối thiểu.
- ID phiên và ID vật phẩm hợp lệ.
- ID tài nguyên không tồn tại.
- Mức giá hợp lệ và không hợp lệ.
- Hình ảnh đúng và sai định dạng.
- Tệp có kích thước nằm trong và vượt quá giới hạn.
- Kết nối WebSocket đang hoạt động.
- Message trùng lặp để kiểm tra idempotency nếu chức năng này đã được triển khai.

Các dữ liệu kiểm thử phải được phân biệt với dữ liệu thật. Sau khi hoàn thành kiểm thử, nhóm cần xóa hoặc đánh dấu dữ liệu kiểm thử nếu dữ liệu đó không còn cần thiết.

#### Trình tự thực hiện các nhóm kiểm thử

Các nhóm test case nên được thực hiện theo thứ tự phụ thuộc sau:

1. Kiểm tra môi trường và các endpoint.
2. Kiểm thử xác thực và phân quyền.
3. Kiểm thử REST API.
4. Kiểm thử dữ liệu trong DynamoDB.
5. Kiểm thử kết nối WebSocket.
6. Kiểm thử luồng đặt giá đầu cuối.
7. Kiểm thử SQS FIFO và xử lý đồng thời.
8. Kiểm thử S3 và CloudFront.
9. Kiểm thử xử lý lỗi và khả năng phục hồi.
10. Kiểm thử bảo mật.
11. Kiểm thử hiệu năng.
12. Tổng hợp kết quả.

Nếu nhóm kiểm thử xác thực chưa đạt, các test case phụ thuộc vào token có thể được đánh dấu `BLOCKED`. Tương tự, nếu WebSocket hoặc SQS FIFO chưa hoạt động, nhóm chưa thể kết luận luồng đặt giá đầu cuối đạt yêu cầu.

#### Kiểm thử lại sau khi sửa lỗi

Khi một test case có trạng thái `FAIL`, nhóm thực hiện quy trình sau:

```text
Ghi nhận lỗi
→ Xác định nguyên nhân
→ Sửa lỗi
→ Triển khai phiên bản mới
→ Thực hiện lại test case thất bại
→ Kiểm tra các chức năng liên quan
→ Cập nhật kết quả và bằng chứng
```

Sau khi sửa lỗi, test case phải được thực hiện lại từ đầu. Nhóm không được chuyển trạng thái từ `FAIL` sang `PASS` chỉ dựa trên việc mã nguồn đã được chỉnh sửa.

Ngoài việc kiểm thử lại test case thất bại, nhóm cần thực hiện **kiểm thử hồi quy** đối với các chức năng liên quan để xác nhận thay đổi mới không làm ảnh hưởng đến những chức năng đã hoạt động trước đó.

#### Bảo mật thông tin kiểm thử

Trong tài liệu và bằng chứng kiểm thử, nhóm không được hiển thị:

- Mật khẩu tài khoản kiểm thử.
- Access Token, ID Token hoặc Refresh Token.
- Header `Authorization`.
- AWS Access Key ID.
- AWS Secret Access Key.
- AWS Session Token.
- Cognito Client Secret.
- Cookie hoặc thông tin phiên đăng nhập.
- Nội dung tệp `.env`.
- Presigned URL còn hiệu lực.
- Dữ liệu cá nhân không cần thiết.

Nếu request hoặc log chứa thông tin nhạy cảm, nhóm phải che hoặc loại bỏ phần dữ liệu đó trước khi đưa vào báo cáo. Chỉ giữ lại những thông tin cần thiết để chứng minh test case đã được thực hiện và cho kết quả tương ứng.