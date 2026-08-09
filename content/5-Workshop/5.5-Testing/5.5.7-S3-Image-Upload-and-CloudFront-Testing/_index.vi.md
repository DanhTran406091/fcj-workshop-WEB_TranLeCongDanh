---
title: "Kiểm thử S3, tải ảnh và CloudFront"
date: 2026-08-03
weight: 7
chapter: false
pre: " <b> 5.5.7. </b> "
---


#### Mục tiêu

Phần kiểm thử này xác minh việc lưu trữ và phân phối nội dung tĩnh của hệ thống thông qua Amazon S3 và Amazon CloudFront, bao gồm:

-  User Frontend được phân phối qua CloudFront. 
-  Admin Frontend được phân phối qua CloudFront. 
-  Các bucket S3 không cho phép truy cập công khai trực tiếp. 
-  CloudFront Origin Access Control (OAC) có thể đọc đúng object được cấp quyền. 
-  Người dùng hợp lệ có thể tải ảnh vật phẩm lên Item Media Bucket. 
-  CORS chỉ cho phép đúng origin và HTTP method. 
-  Hệ thống từ chối tệp sai định dạng hoặc vượt giới hạn kích thước. 
-  Ảnh sau khi tải lên có thể hiển thị trên frontend. 
-  S3 Versioning tạo phiên bản mới khi object bị thay thế. 
-  Lambda không thể ghi vào bucket ngoài phạm vi IAM được cấp. 

Ba bucket cần kiểm tra:

| BucketMục đích        |                                        |
| --------------------- | -------------------------------------- |
| User Frontend Bucket  | Lưu React build dành cho người dùng    |
| Admin Frontend Bucket | Lưu React build dành cho quản trị viên |
| Item Media Bucket     | Lưu ảnh của vật phẩm đấu giá           |

---

#### Luồng phân phối frontend

```
```

```
Browser
→ CloudFront
→ Origin Access Control
→ Private S3 Frontend Bucket
→ CloudFront
→ Browser
```

#### Luồng tải ảnh vật phẩm

```
```

```
Authenticated User
→ Backend API hoặc Lambda
→ Tạo presigned URL/presigned POST
→ Browser tải ảnh trực tiếp lên Item Media Bucket
→ S3 lưu object
→ CloudFront hoặc URL phân phối ảnh
→ Frontend hiển thị ảnh
```

---

#### Điều kiện kiểm thử chung

Trước khi kiểm thử, hệ thống cần đáp ứng các điều kiện sau:

-  User Frontend Bucket đã tồn tại. 
-  Admin Frontend Bucket đã tồn tại. 
-  Item Media Bucket đã tồn tại. 
-  Block Public Access được bật cho cả ba bucket. 
-  User Frontend và Admin Frontend đã được triển khai lên S3. 
-  Hai CloudFront distribution đã được tạo hoặc được cấu hình để phân phối đúng hai frontend. 
-  CloudFront sử dụng OAC thay vì để S3 bucket công khai. 
-  Bucket policy chỉ cho phép đúng CloudFront distribution đọc object. 
-  React build chứa `index.html`, JavaScript, CSS và các tài nguyên cần thiết. 
-  React route fallback đã được cấu hình. 
-  Item Media Bucket đã bật CORS. 
-  Item Media Bucket đã bật Versioning nếu đây là yêu cầu của hệ thống. 
-  API hoặc Lambda tạo presigned upload đã được triển khai. 
-  Cơ chế xác thực và phân quyền upload đã được triển khai. 
-  File type và file size được kiểm tra phía server hoặc qua presigned POST policy. 
-  Lambda có IAM role riêng với quyền tối thiểu cần thiết. 
-  CloudFront và S3 access logs hoặc CloudTrail Data Events được bật nếu cần bằng chứng chi tiết. 
-  Môi trường kiểm thử được tách khỏi production. 

Nếu thành phần cần thiết chưa được triển khai, test case tương ứng phải được đánh dấu `BLOCKED`.

---

#### Xác định phương thức tải ảnh trước khi kiểm thử

Trước khi chạy các test case CORS, cần xác định chính xác frontend đang sử dụng phương thức nào.

##### Trường hợp 1: Presigned PUT URL

Frontend gửi trực tiếp nội dung tệp bằng:

```
```

```
PUT /object-key HTTP/1.1
Content-Type: image/jpeg
```

CORS phải cho phép tối thiểu:

```
```

```
PUT
```

##### Trường hợp 2: Presigned POST

Frontend gửi biểu mẫu `multipart/form-data` đến S3 bằng:

```
```

```
POST / HTTP/1.1
Content-Type: multipart/form-data
```

CORS phải cho phép tối thiểu:

```
```

```
POST
```

##### Cách kiểm tra

1.  Mở trang upload ảnh. 
2.  Mở Developer Tools. 
3.  Chọn tab `Network`. 
4.  Thực hiện upload một ảnh. 
5.  Chọn request được gửi đến S3. 
6.  Kiểm tra trường `Request Method`. 
7.  Ghi nhận kết quả là `PUT` hoặc `POST`. 
8.  Kiểm tra request `OPTIONS` preflight nếu trình duyệt tạo request này. 

Không được cấu hình và kiểm thử `POST` nếu ứng dụng thực tế dùng `PUT`, hoặc ngược lại.

---

#### Dữ liệu kiểm thử

| Dữ liệuMô tả            |                                                      |
| ----------------------- | ---------------------------------------------------- |
| User Frontend URL       | CloudFront URL dành cho người dùng                   |
| Admin Frontend URL      | CloudFront URL dành cho quản trị viên                |
| User Bucket Object URL  | URL trực tiếp đến object trong User Frontend Bucket  |
| Admin Bucket Object URL | URL trực tiếp đến object trong Admin Frontend Bucket |
| Item Media Object URL   | URL trực tiếp đến ảnh trong Item Media Bucket        |
| Trusted User Origin     | Origin frontend người dùng được phép                 |
| Trusted Admin Origin    | Origin frontend quản trị được phép                   |
| Untrusted Origin        | Origin không có trong CORS allowlist                 |
| Valid User              | User đã xác thực và có quyền upload ảnh              |
| Invalid User            | User chưa đăng nhập, token sai hoặc token hết hạn    |
| Valid Image             | Tệp JPEG, PNG hoặc WebP hợp lệ theo quy định         |
| Invalid File            | Tệp `.exe`, `.html`, `.js`, `.pdf` hoặc loại bị cấm  |
| Oversized Image         | Tệp lớn hơn giới hạn upload                          |
| Existing Object Key     | Object key đã tồn tại để kiểm tra Versioning         |
| Allowed Lambda          | Lambda được phép ghi vào Item Media Bucket           |
| Restricted Lambda       | Lambda không được phép ghi vào frontend bucket       |

Không sử dụng dữ liệu production trong quá trình kiểm thử.

---

### STORAGE-01 — Truy cập User Frontend qua CloudFront

| TrườngNội dung           |                                                                                                                                                                                              |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `STORAGE-01`                                                                                                                                                                                 |
| **Tên kiểm thử**         | Truy cập User Frontend thông qua CloudFront                                                                                                                                                  |
| **Mục tiêu**             | Xác minh người dùng có thể mở User Frontend từ CloudFront distribution.                                                                                                                      |
| **Điều kiện tiên quyết** | User Frontend đã được build và deploy; CloudFront distribution ở trạng thái `Deployed`.                                                                                                      |
| **Các bước thực hiện**   | 1. Mở trình duyệt ở chế độ ẩn danh.<br>2. Truy cập User Frontend CloudFront URL.<br>3. Kiểm tra HTTP response.<br>4. Kiểm tra giao diện và browser console.<br>5. Kiểm tra response headers. |
| **Kết quả mong đợi**     | Trang trả về `200 OK`; giao diện hiển thị đúng; không xuất hiện XML Access Denied; không có lỗi tải tài nguyên nghiêm trọng; response được phục vụ qua CloudFront.                           |
| **Kết quả thực tế**      | Điền URL, status code, thời gian phản hồi và kết quả hiển thị.                                                                                                                               |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                               |
| **Bằng chứng**           | Ảnh giao diện, Network tab và response headers.                                                                                                                                              |

---

### STORAGE-02 — Truy cập Admin Frontend qua CloudFront

| TrườngNội dung           |                                                                                                                                                                                                                        |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `STORAGE-02`                                                                                                                                                                                                           |
| **Tên kiểm thử**         | Truy cập Admin Frontend thông qua CloudFront                                                                                                                                                                           |
| **Mục tiêu**             | Xác minh Admin Frontend được phân phối từ đúng CloudFront distribution.                                                                                                                                                |
| **Điều kiện tiên quyết** | Admin Frontend đã được build và deploy; CloudFront distribution đã sẵn sàng.                                                                                                                                           |
| **Các bước thực hiện**   | 1. Mở Admin Frontend CloudFront URL.<br>2. Kiểm tra status code.<br>3. Kiểm tra trang đăng nhập hoặc trang mặc định.<br>4. Kiểm tra Network tab và browser console.<br>5. Đối chiếu distribution đang phục vụ request. |
| **Kết quả mong đợi**     | Trang trả về `200 OK`; giao diện Admin hiển thị đúng; không tải nhầm User Frontend; tài nguyên được phân phối qua đúng CloudFront distribution.                                                                        |
| **Kết quả thực tế**      | Điền URL, distribution, status code và kết quả giao diện.                                                                                                                                                              |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                         |
| **Bằng chứng**           | Ảnh giao diện Admin, Network tab và CloudFront headers.                                                                                                                                                                |

---

### STORAGE-03 — Tải `index.html`, JavaScript và CSS

| TrườngNội dung           |                                                                                                                                                                                                   |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `STORAGE-03`                                                                                                                                                                                      |
| **Tên kiểm thử**         | Tải đầy đủ tài nguyên frontend                                                                                                                                                                    |
| **Mục tiêu**             | Xác minh `index.html` và các bundle JavaScript/CSS được tải thành công với đúng kiểu nội dung.                                                                                                    |
| **Điều kiện tiên quyết** | Frontend build đã được upload đầy đủ lên S3.                                                                                                                                                      |
| **Các bước thực hiện**   | 1. Mở User Frontend và Admin Frontend.<br>2. Mở Developer Tools → Network.<br>3. Tải lại trang.<br>4. Lọc theo `Doc`, `JS` và `CSS`.<br>5. Kiểm tra status code, `Content-Type` và response body. |
| **Kết quả mong đợi**     | `index.html`, JavaScript và CSS đều trả về `200 OK`; JavaScript có content type phù hợp; CSS trả về `text/css`; nội dung JavaScript/CSS không bị thay bằng `index.html`; không có lỗi MIME type.  |
| **Kết quả thực tế**      | Điền tên object, status code, content type và kích thước.                                                                                                                                         |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                    |
| **Bằng chứng**           | Network waterfall, response headers và browser console.                                                                                                                                           |

> Cần kiểm tra kỹ trường hợp một file JavaScript bị thiếu nhưng CloudFront lại trả `index.html` với `200 OK`. Trường hợp này phải được đánh dấu `FAIL`.

---

### STORAGE-04 — Tải lại React route

| TrườngNội dung           |                                                                                                                                                                                                                                          |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `STORAGE-04`                                                                                                                                                                                                                             |
| **Tên kiểm thử**         | Tải lại route phía client của React                                                                                                                                                                                                      |
| **Mục tiêu**             | Xác minh React route hoạt động khi truy cập trực tiếp hoặc tải lại trang.                                                                                                                                                                |
| **Điều kiện tiên quyết** | Ứng dụng dùng client-side routing; CloudFront có cấu hình fallback phù hợp.                                                                                                                                                              |
| **Các bước thực hiện**   | 1. Truy cập một route hợp lệ, ví dụ `/auction-items/item-001`.<br>2. Tải lại trang bằng `Ctrl+R` hoặc `F5`.<br>3. Dán trực tiếp URL route vào tab mới.<br>4. Kiểm tra response và giao diện.<br>5. Thử truy cập một asset không tồn tại. |
| **Kết quả mong đợi**     | Route hợp lệ hiển thị đúng thay vì trả S3 `AccessDenied` hoặc lỗi không mong muốn; React được khởi tạo thành công; asset không tồn tại không được che thành một response HTML thành công ngoài thiết kế.                                 |
| **Kết quả thực tế**      | Điền route, status code, response type và kết quả hiển thị.                                                                                                                                                                              |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                           |
| **Bằng chứng**           | URL route, Network tab, CloudFront configuration và ảnh giao diện.                                                                                                                                                                       |

---

### STORAGE-05 — Từ chối truy cập trực tiếp private S3 object

| TrườngNội dung           |                                                                                                                                                                                                              |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Mã kiểm thử**          | `STORAGE-05`                                                                                                                                                                                                 |
| **Tên kiểm thử**         | Chặn truy cập trực tiếp vào private S3 bucket                                                                                                                                                                |
| **Mục tiêu**             | Xác minh người dùng Internet không thể bỏ qua CloudFront để đọc object trực tiếp từ S3.                                                                                                                      |
| **Điều kiện tiên quyết** | Block Public Access đã bật; bucket không cấu hình public website hosting phục vụ trực tiếp.                                                                                                                  |
| **Các bước thực hiện**   | 1. Sao chép S3 object URL trực tiếp của `index.html`.<br>2. Mở URL trong trình duyệt ẩn danh.<br>3. Thực hiện tương tự với JavaScript, CSS hoặc ảnh.<br>4. Kiểm tra bucket policy và public access settings. |
| **Kết quả mong đợi**     | Request trực tiếp bị từ chối với `403 AccessDenied` hoặc kết quả tương đương; object không bị trả về; bucket và object không public.                                                                         |
| **Kết quả thực tế**      | Điền object URL đã che bucket nếu cần, status code và response code.                                                                                                                                         |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                               |
| **Bằng chứng**           | Response `403`, Block Public Access và bucket policy.                                                                                                                                                        |

---

### STORAGE-06 — CloudFront OAC đọc được object

| TrườngNội dung           |                                                                                                                                                                                                                             |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `STORAGE-06`                                                                                                                                                                                                                |
| **Tên kiểm thử**         | CloudFront truy cập private S3 bằng OAC                                                                                                                                                                                     |
| **Mục tiêu**             | Xác minh OAC cho phép đúng CloudFront distribution đọc object trong private bucket.                                                                                                                                         |
| **Điều kiện tiên quyết** | OAC đã gắn với S3 origin; bucket policy giới hạn theo CloudFront service principal và distribution ARN.                                                                                                                     |
| **Các bước thực hiện**   | 1. Truy cập một object thông qua CloudFront URL.<br>2. Xác nhận request thành công.<br>3. Truy cập cùng object bằng S3 URL trực tiếp.<br>4. Kiểm tra origin configuration.<br>5. Kiểm tra bucket policy và `AWS:SourceArn`. |
| **Kết quả mong đợi**     | CloudFront trả object thành công; truy cập S3 trực tiếp bị từ chối; OAC ở trạng thái hoạt động; bucket policy không cấp quyền đọc công khai và chỉ cho phép đúng distribution cần thiết.                                    |
| **Kết quả thực tế**      | Điền CloudFront status, S3 direct status, OAC ID và distribution ARN đã che nếu cần.                                                                                                                                        |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                              |
| **Bằng chứng**           | Hai HTTP response, CloudFront origin configuration và bucket policy.                                                                                                                                                        |

---

### STORAGE-07 — User hợp lệ tải ảnh vật phẩm lên

| TrườngNội dung           |                                                                                                                                                                                                                                                                     |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `STORAGE-07`                                                                                                                                                                                                                                                        |
| **Tên kiểm thử**         | Upload ảnh bởi User có quyền                                                                                                                                                                                                                                        |
| **Mục tiêu**             | Xác minh một User hợp lệ có thể tải ảnh vật phẩm lên đúng Item Media Bucket.                                                                                                                                                                                        |
| **Điều kiện tiên quyết** | User đã xác thực; có quyền chỉnh sửa vật phẩm; API tạo presigned upload đã hoạt động.                                                                                                                                                                               |
| **Các bước thực hiện**   | 1. Đăng nhập bằng Valid User.<br>2. Chọn vật phẩm được phép chỉnh sửa.<br>3. Chọn Valid Image.<br>4. Gửi yêu cầu lấy presigned upload.<br>5. Upload bằng đúng method `POST` hoặc `PUT`.<br>6. Kiểm tra response S3.<br>7. Kiểm tra object trong bucket và metadata. |
| **Kết quả mong đợi**     | Backend xác thực User và quyền trên vật phẩm; presigned upload được tạo cho đúng bucket/key; upload thành công; object xuất hiện ở đúng prefix; content type và kích thước đúng; URL hết hạn và không cấp quyền rộng hơn yêu cầu.                                   |
| **Kết quả thực tế**      | Điền User ID đã che, Item ID, method, object key, status code và kích thước.                                                                                                                                                                                        |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                                                      |
| **Bằng chứng**           | API response đã che chữ ký, Network request, S3 object metadata và CloudWatch Logs.                                                                                                                                                                                 |

> Không đưa đầy đủ presigned URL còn hiệu lực vào tài liệu kiểm thử vì URL chứa thông tin chữ ký có thể được sử dụng để upload object.

---

### STORAGE-08 — CORS cho phép đúng origin và HTTP method

| TrườngNội dung           |                                                                                                                                                                                                                                        |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `STORAGE-08`                                                                                                                                                                                                                           |
| **Tên kiểm thử**         | Cho phép upload từ trusted origin                                                                                                                                                                                                      |
| **Mục tiêu**             | Xác minh Item Media Bucket chỉ cho phép đúng frontend origin và upload method thực tế.                                                                                                                                                 |
| **Điều kiện tiên quyết** | Đã xác định ứng dụng sử dụng presigned `POST` hoặc presigned `PUT`.                                                                                                                                                                    |
| **Các bước thực hiện**   | 1. Mở frontend từ trusted origin.<br>2. Thực hiện upload.<br>3. Kiểm tra request `OPTIONS` nếu có.<br>4. Kiểm tra `Access-Control-Allow-Origin`.<br>5. Kiểm tra `Access-Control-Allow-Methods`.<br>6. Kiểm tra upload request thực tế. |
| **Kết quả mong đợi**     | Preflight thành công; response chỉ cho phép trusted origin; method thực tế là `POST` hoặc `PUT` được cho phép; các header cần thiết như `Content-Type` được chấp nhận; trình duyệt hoàn tất upload mà không có lỗi CORS.               |
| **Kết quả thực tế**      | Điền origin, method, request headers và response CORS headers.                                                                                                                                                                         |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                         |
| **Bằng chứng**           | Network tab của request `OPTIONS` và request upload.                                                                                                                                                                                   |

---

### STORAGE-09 — Từ chối origin không được tin cậy

| TrườngNội dung           |                                                                                                                                                                                                                                              |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `STORAGE-09`                                                                                                                                                                                                                                 |
| **Tên kiểm thử**         | Không cho phép upload từ untrusted origin                                                                                                                                                                                                    |
| **Mục tiêu**             | Xác minh origin ngoài allowlist không được CORS cho phép.                                                                                                                                                                                    |
| **Điều kiện tiên quyết** | Có một origin kiểm thử không nằm trong cấu hình CORS.                                                                                                                                                                                        |
| **Các bước thực hiện**   | 1. Gửi preflight với header `Origin` là untrusted origin.<br>2. Dùng đúng requested method của ứng dụng.<br>3. Kiểm tra response headers.<br>4. Thử upload từ trình duyệt tại untrusted origin.<br>5. Kiểm tra object có được tạo hay không. |
| **Kết quả mong đợi**     | Response không trả `Access-Control-Allow-Origin` cho untrusted origin; trình duyệt chặn request theo CORS; upload không hoàn tất qua luồng trình duyệt kiểm thử; không tạo object ngoài ý muốn.                                              |
| **Kết quả thực tế**      | Điền origin, preflight status, CORS headers và kết quả object.                                                                                                                                                                               |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                               |
| **Bằng chứng**           | Preflight response, browser console và kiểm tra S3 object.                                                                                                                                                                                   |

> CORS không phải cơ chế xác thực. Một request không chạy trong trình duyệt vẫn có thể gọi presigned URL hợp lệ. Vì vậy presigned URL phải có thời hạn ngắn, giới hạn đúng key, method, content type và các điều kiện cần thiết.

---

### STORAGE-10 — Từ chối tệp sai loại

| TrườngNội dung           |                                                                                                                                                                                                                                   |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `STORAGE-10`                                                                                                                                                                                                                      |
| **Tên kiểm thử**         | Từ chối file type không được hỗ trợ                                                                                                                                                                                               |
| **Mục tiêu**             | Xác minh hệ thống không chấp nhận tệp ngoài danh sách định dạng ảnh được phép.                                                                                                                                                    |
| **Điều kiện tiên quyết** | Allowlist định dạng ảnh và cơ chế kiểm tra file type đã được triển khai.                                                                                                                                                          |
| **Các bước thực hiện**   | 1. Chọn một tệp `.exe`, `.html`, `.js` hoặc loại bị cấm.<br>2. Thử đổi phần mở rộng thành `.jpg`.<br>3. Gửi yêu cầu lấy presigned upload.<br>4. Nếu URL vẫn được cấp, thử upload.<br>5. Kiểm tra S3 và logs.                      |
| **Kết quả mong đợi**     | Tệp bị từ chối với mã như `UNSUPPORTED_FILE_TYPE`; hệ thống không chỉ tin phần mở rộng hoặc `Content-Type` do client khai báo; object không được công bố như ảnh hợp lệ; không có nội dung thực thi được phục vụ từ media domain. |
| **Kết quả thực tế**      | Điền tên tệp, extension, MIME type nhận diện và error code.                                                                                                                                                                       |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                    |
| **Bằng chứng**           | API response, logs kiểm tra file, S3 object listing và frontend.                                                                                                                                                                  |

---

### STORAGE-11 — Từ chối tệp vượt kích thước cho phép

| TrườngNội dung           |                                                                                                                                                                                                          |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `STORAGE-11`                                                                                                                                                                                             |
| **Tên kiểm thử**         | Từ chối ảnh vượt giới hạn dung lượng                                                                                                                                                                     |
| **Mục tiêu**             | Xác minh ảnh lớn hơn giới hạn không được chấp nhận.                                                                                                                                                      |
| **Điều kiện tiên quyết** | Giới hạn kích thước đã được định nghĩa, ví dụ `5 MB`.                                                                                                                                                    |
| **Các bước thực hiện**   | 1. Chuẩn bị một ảnh nhỏ hơn giới hạn.<br>2. Chuẩn bị một ảnh đúng tại giới hạn.<br>3. Chuẩn bị một ảnh lớn hơn giới hạn.<br>4. Thử upload từng tệp.<br>5. Kiểm tra response, S3 và logs.                 |
| **Kết quả mong đợi**     | Ảnh nhỏ hơn hoặc đúng giới hạn được xử lý theo quy định; ảnh vượt giới hạn bị từ chối với `FILE_TOO_LARGE` hoặc mã tương đương; object vượt giới hạn không được lưu hoặc được cách ly/xóa theo thiết kế. |
| **Kết quả thực tế**      | Điền giới hạn, kích thước từng tệp, status và error code.                                                                                                                                                |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                           |
| **Bằng chứng**           | Kích thước tệp, API/S3 response, object metadata và logs.                                                                                                                                                |

> Với presigned `POST`, có thể kiểm soát kích thước bằng policy condition `content-length-range`. Với presigned `PUT`, cần xác minh cơ chế kiểm soát kích thước thực tế vì việc kiểm tra chỉ ở frontend không đủ an toàn.

---

### STORAGE-12 — Ảnh tải lên hiển thị đúng trên frontend

| TrườngNội dung           |                                                                                                                                                                                                                                                                                |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Mã kiểm thử**          | `STORAGE-12`                                                                                                                                                                                                                                                                   |
| **Tên kiểm thử**         | Phân phối và hiển thị ảnh vật phẩm                                                                                                                                                                                                                                             |
| **Mục tiêu**             | Xác minh ảnh sau khi upload được liên kết đúng với vật phẩm và hiển thị qua URL phân phối được phép.                                                                                                                                                                           |
| **Điều kiện tiên quyết** | STORAGE-07 đã thành công; dữ liệu vật phẩm lưu đúng object key hoặc media URL.                                                                                                                                                                                                 |
| **Các bước thực hiện**   | 1. Hoàn tất upload ảnh.<br>2. Mở trang chi tiết vật phẩm.<br>3. Không sử dụng S3 Console URL để hiển thị ảnh.<br>4. Kiểm tra request ảnh trong Network tab.<br>5. Kiểm tra content type, kích thước và cache headers.<br>6. Mở lại trang hoặc kiểm tra ở một trình duyệt khác. |
| **Kết quả mong đợi**     | Ảnh hiển thị đúng; không xuất hiện broken image; request trả `200 OK`; nội dung khớp tệp đã upload; object được lấy qua CloudFront hoặc cơ chế URL được thiết kế; người dùng không được cấp quyền duyệt toàn bộ bucket.                                                        |
| **Kết quả thực tế**      | Điền Item ID, object key đã che, URL phân phối, status và kết quả hiển thị.                                                                                                                                                                                                    |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                                                                 |
| **Bằng chứng**           | Ảnh giao diện, Network response và S3 object metadata.                                                                                                                                                                                                                         |

---

### STORAGE-13 — S3 Versioning tạo phiên bản mới

| TrườngNội dung           |                                                                                                                                                                                                                                              |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mã kiểm thử**          | `STORAGE-13`                                                                                                                                                                                                                                 |
| **Tên kiểm thử**         | Tạo version mới khi thay thế object                                                                                                                                                                                                          |
| **Mục tiêu**             | Xác minh việc ghi đè cùng object key không phá hủy ngay phiên bản trước.                                                                                                                                                                     |
| **Điều kiện tiên quyết** | Versioning đã được bật cho bucket cần kiểm tra.                                                                                                                                                                                              |
| **Các bước thực hiện**   | 1. Upload ảnh A vào một object key xác định.<br>2. Ghi nhận `VersionId` thứ nhất.<br>3. Upload ảnh B vào đúng object key đó.<br>4. Ghi nhận `VersionId` thứ hai.<br>5. Liệt kê các version.<br>6. Kiểm tra version hiện tại và phiên bản cũ. |
| **Kết quả mong đợi**     | Hai `VersionId` khác nhau tồn tại; ảnh B là current version; ảnh A vẫn tồn tại dưới version cũ; không xuất hiện `null` Version ID nếu Versioning đã được bật đúng trước khi upload.                                                          |
| **Kết quả thực tế**      | Điền object key, Version ID 1, Version ID 2 và current version.                                                                                                                                                                              |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                               |
| **Bằng chứng**           | S3 Versions view, object metadata và CLI output nếu được sử dụng.                                                                                                                                                                            |

> Nếu object được phân phối qua CloudFront với cùng URL, cần kiểm tra cache invalidation hoặc chiến lược object key có version/hash. S3 Versioning không tự làm CloudFront bỏ bản cache cũ.

---

### STORAGE-14 — Lambda không thể ghi ngoài phạm vi IAM

| TrườngNội dung           |                                                                                                                                                                                                                                                                                                                                      |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Mã kiểm thử**          | `STORAGE-14`                                                                                                                                                                                                                                                                                                                         |
| **Tên kiểm thử**         | Giới hạn quyền ghi S3 của Lambda                                                                                                                                                                                                                                                                                                     |
| **Mục tiêu**             | Xác minh Lambda chỉ có thể thao tác với bucket và prefix được IAM cho phép.                                                                                                                                                                                                                                                          |
| **Điều kiện tiên quyết** | Lambda execution role đã được cấu hình least privilege; có một bucket/prefix hợp lệ và một bucket/prefix ngoài phạm vi.                                                                                                                                                                                                              |
| **Các bước thực hiện**   | 1. Kích hoạt Lambda ghi vào Item Media Bucket và prefix được cấp.<br>2. Xác nhận thao tác thành công.<br>3. Cho Lambda thử ghi vào User Frontend Bucket.<br>4. Cho Lambda thử ghi vào Admin Frontend Bucket.<br>5. Cho Lambda thử ghi vào prefix khác trong Item Media Bucket.<br>6. Kiểm tra kết quả và CloudTrail/CloudWatch Logs. |
| **Kết quả mong đợi**     | Lambda ghi thành công vào đúng bucket/prefix được cấp; các thao tác ngoài phạm vi bị từ chối với `AccessDenied`; không có wildcard quyền ghi rộng như `arn:aws:s3:::*/*`; lỗi được ghi log nhưng không làm lộ thông tin nhạy cảm.                                                                                                    |
| **Kết quả thực tế**      | Điền role, action, resource đã che, kết quả và error code.                                                                                                                                                                                                                                                                           |
| **Trạng thái**           | `PASS`, `FAIL` hoặc `BLOCKED`.                                                                                                                                                                                                                                                                                                       |
| **Bằng chứng**           | IAM policy, Lambda logs, CloudTrail event và S3 object listing.                                                                                                                                                                                                                                                                      |

> Nên thực hiện test phủ định bằng một test function hoặc môi trường kiểm thử có kiểm soát. Không sửa Lambda production để cố tình ghi vào bucket ngoài phạm vi.

---

### Ma trận quyền truy cập mong đợi

| Thành phầnUser Frontend BucketAdmin Frontend BucketItem Media Bucket |                     |                     |                                           |
| -------------------------------------------------------------------- | ------------------- | ------------------- | ----------------------------------------- |
| Người dùng truy cập S3 trực tiếp                                     | Từ chối             | Từ chối             | Từ chối, trừ URL có chữ ký hợp lệ         |
| User CloudFront Distribution                                         | Được đọc            | Từ chối             | Theo thiết kế                             |
| Admin CloudFront Distribution                                        | Từ chối             | Được đọc            | Theo thiết kế                             |
| Lambda tạo upload                                                    | Không ghi           | Không ghi           | Chỉ tạo URL hoặc ghi đúng prefix được cấp |
| Lambda xử lý ảnh                                                     | Không ghi           | Không ghi           | Đọc/ghi đúng prefix được cấp              |
| User chưa xác thực                                                   | Không upload        | Không upload        | Không được cấp presigned upload           |
| User hợp lệ có quyền                                                 | Không ghi trực tiếp | Không ghi trực tiếp | Upload qua presigned request giới hạn     |

---

### Ma trận kiểm tra upload

| Trường hợpCấp presigned uploadS3 lưu objectFrontend hiển thị |                                  |                               |       |
| ------------------------------------------------------------ | -------------------------------- | ----------------------------- | ----- |
| User hợp lệ, ảnh đúng loại và kích thước                     | Có                               | Có                            | Có    |
| User chưa đăng nhập                                          | Không                            | Không                         | Không |
| User không có quyền trên vật phẩm                            | Không                            | Không                         | Không |
| Origin được tin cậy                                          | Theo quyền User                  | Có thể upload                 | Có    |
| Origin không được tin cậy                                    | Không được browser CORS cho phép | Không qua luồng browser chuẩn | Không |
| File sai loại                                                | Không hoặc bị cách ly            | Không lưu như ảnh hợp lệ      | Không |
| File vượt kích thước                                         | Không hoặc S3 từ chối            | Không                         | Không |
| Presigned URL hết hạn                                        | Không áp dụng                    | Từ chối                       | Không |
| Object key ngoài phạm vi URL ký                              | Không áp dụng                    | Từ chối                       | Không |

---

### Quy định kiểm tra CORS

CORS của Item Media Bucket cần đáp ứng:

-  Chỉ chứa các origin được tin cậy. 
-  Phân biệt User Frontend và Admin Frontend nếu quyền upload khác nhau. 
-  Chỉ cho phép method thực tế được sử dụng: `POST` hoặc `PUT`. 
-  Chỉ cho phép các header cần thiết. 
-  Không dùng `AllowedOrigins: ["*"]` nếu thiết kế yêu cầu giới hạn origin. 
-  Không cho phép thêm method không cần thiết như `DELETE`. 
- `OPTIONS` preflight trả đúng CORS headers. 
-  Không nhầm lẫn CORS với authentication hoặc authorization. 
-  Thay đổi CORS phải được kiểm thử lại trên trình duyệt thật. 

Ví dụ cho presigned `PUT`:

```
```

```
[
  {
    "AllowedOrigins": [
      "https://user.example.com"
    ],
    "AllowedMethods": [
      "PUT"
    ],
    "AllowedHeaders": [
      "Content-Type"
    ],
    "ExposeHeaders": [
      "ETag"
    ],
    "MaxAgeSeconds": 3000
  }
]
```

Ví dụ cho presigned `POST`:

```
```

```
[
  {
    "AllowedOrigins": [
      "https://user.example.com"
    ],
    "AllowedMethods": [
      "POST"
    ],
    "AllowedHeaders": [
      "*"
    ],
    "ExposeHeaders": [
      "ETag"
    ],
    "MaxAgeSeconds": 3000
  }
]
```

Các origin và header trong ví dụ phải được thay bằng giá trị thực tế của hệ thống. Không sao chép cấu hình mẫu vào production khi chưa đối chiếu request thật trong trình duyệt.

---

### Quy định kiểm tra cache CloudFront

Cần xác minh:

- `index.html` không bị cache quá lâu sau mỗi lần deploy. 
-  Các file có hash trong tên như JavaScript/CSS có thể dùng cache dài hạn. 
- `Content-Type` của object chính xác. 
-  User Frontend và Admin Frontend không dùng nhầm origin. 
-  CloudFront không làm lộ private S3 URL. 
-  Cache key không chứa dữ liệu không cần thiết. 
-  Ảnh được cache theo chính sách phù hợp. 
-  Khi thay cùng object key, CloudFront được invalidated hoặc sử dụng tên object mới. 
-  CloudFront custom error response không che lỗi asset thành `index.html` ngoài ý muốn. 
-  HTTPS được bắt buộc hoặc HTTP được redirect sang HTTPS. 

Cache policy khuyến nghị:

| Loại objectCache-Control gợi ý |                                        |
| ------------------------------ | -------------------------------------- |
| `index.html`                   | `no-cache` hoặc thời gian cache ngắn   |
| JavaScript/CSS có content hash | `public, max-age=31536000, immutable`  |
| Ảnh có object key bất biến     | `public, max-age=31536000, immutable`  |
| Ảnh có thể bị ghi đè cùng key  | Cache ngắn hoặc thực hiện invalidation |

---

### Quy định kiểm tra bảo mật upload

Luồng upload cần kiểm tra tối thiểu:

-  User đã được xác thực. 
-  User có quyền chỉnh sửa đúng vật phẩm. 
-  Object key do server kiểm soát. 
-  User không thể thay key để ghi đè ảnh của User khác. 
-  Tên file đầu vào không được dùng trực tiếp làm key nếu có rủi ro path manipulation. 
-  Presigned URL có thời hạn ngắn. 
-  Presigned URL chỉ dùng cho đúng bucket, key và HTTP method. 
-  Không ghi token hoặc toàn bộ presigned URL vào logs. 
-  File extension và `Content-Type` không phải bằng chứng duy nhất về loại tệp. 
-  Kích thước tệp được kiểm soát phía server hoặc bằng policy của S3. 
-  Ảnh nên được kiểm tra hoặc xử lý trước khi công khai nếu hệ thống tiếp nhận nội dung không tin cậy. 
-  Object không được cấp ACL public-read. 
-  Bucket owner enforced nên được sử dụng nếu phù hợp. 
-  Server-side encryption được bật theo yêu cầu. 
-  Lifecycle rule được cấu hình cho version cũ hoặc upload chưa hoàn tất nếu cần. 

---

### Quy định kiểm tra IAM

IAM role của Lambda cần đáp ứng nguyên tắc quyền tối thiểu:

-  Chỉ cấp các action cần thiết. 
-  Chỉ cấp quyền trên đúng bucket. 
-  Giới hạn xuống đúng prefix nếu có thể. 
-  Tách quyền đọc và ghi khi phù hợp. 
-  Không sử dụng `s3:*`. 
-  Không sử dụng resource `*` cho thao tác object nếu không cần. 
-  Không cấp quyền ghi vào User Frontend hoặc Admin Frontend cho Lambda xử lý media. 
-  Quyền tạo presigned URL không có nghĩa người dùng được nhận AWS credentials. 
-  Deny từ bucket policy, permission boundary hoặc SCP vẫn phải có hiệu lực. 

Ví dụ phạm vi tài nguyên mong đợi:

```
```

```
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject",
    "s3:PutObject"
  ],
  "Resource": "arn:aws:s3:::item-media-bucket/items/*"
}
```

Tên bucket phải được thay bằng resource thực tế của hệ thống.

---

### Quy định kiểm tra logs và bằng chứng

Logs nên có:

-  Correlation ID hoặc Request ID. 
-  User ID đã xác minh và được che khi cần. 
-  Item ID. 
-  Object key. 
-  HTTP method upload. 
-  Content type. 
-  Kích thước tệp. 
-  Kết quả xác thực. 
-  Kết quả phân quyền. 
-  Kết quả tạo presigned upload. 
-  Kết quả xử lý ảnh. 
-  S3 error code. 
-  Lambda Request ID. 
-  Thời gian xử lý. 

Logs không được chứa:

-  Access Token. 
-  ID Token. 
-  Refresh Token. 
-  AWS access key hoặc secret key. 
-  Toàn bộ presigned URL còn hiệu lực. 
-  Chữ ký trong query string. 
-  Cookie đăng nhập. 
-  Dữ liệu ảnh dạng Base64. 
-  Thông tin cá nhân không cần thiết. 

---

### Bảng tổng hợp kết quả

| MãNội dung kiểm thửKết quả chính mong đợiTrạng thái |                               |                                                 |               |
| --------------------------------------------------- | ----------------------------- | ----------------------------------------------- | ------------- |
| `STORAGE-01`                                        | User Frontend qua CloudFront  | Trả `200`, giao diện hiển thị đúng              | Chưa kiểm thử |
| `STORAGE-02`                                        | Admin Frontend qua CloudFront | Trả `200`, đúng Admin UI                        | Chưa kiểm thử |
| `STORAGE-03`                                        | Tải HTML, JavaScript và CSS   | Tài nguyên tải đúng content type                | Chưa kiểm thử |
| `STORAGE-04`                                        | Tải lại React route           | Route hiển thị không có lỗi ngoài ý muốn        | Chưa kiểm thử |
| `STORAGE-05`                                        | Truy cập S3 trực tiếp         | Bị từ chối                                      | Chưa kiểm thử |
| `STORAGE-06`                                        | CloudFront OAC đọc S3         | CloudFront đọc được, truy cập trực tiếp bị chặn | Chưa kiểm thử |
| `STORAGE-07`                                        | User hợp lệ upload ảnh        | Ảnh được lưu đúng bucket và prefix              | Chưa kiểm thử |
| `STORAGE-08`                                        | CORS trusted origin/method    | Upload thành công                               | Chưa kiểm thử |
| `STORAGE-09`                                        | CORS untrusted origin         | Trình duyệt từ chối                             | Chưa kiểm thử |
| `STORAGE-10`                                        | File sai loại                 | Bị từ chối                                      | Chưa kiểm thử |
| `STORAGE-11`                                        | File vượt kích thước          | Bị từ chối                                      | Chưa kiểm thử |
| `STORAGE-12`                                        | Hiển thị ảnh trên frontend    | Ảnh hiển thị đúng                               | Chưa kiểm thử |
| `STORAGE-13`                                        | S3 Versioning                 | Tạo Version ID mới                              | Chưa kiểm thử |
| `STORAGE-14`                                        | Giới hạn IAM của Lambda       | Ghi đúng phạm vi, ngoài phạm vi bị từ chối      | Chưa kiểm thử |

---

### Bằng chứng kiểm thử

Bằng chứng nên bao gồm:

-  User Frontend và Admin Frontend hiển thị qua CloudFront. 
-  Network request của `index.html`, JavaScript và CSS. 
-  Response headers từ CloudFront. 
-  Kết quả tải lại React route. 
-  Response khi truy cập trực tiếp private S3 object. 
-  CloudFront origin và OAC configuration. 
-  Bucket policy và Block Public Access. 
-  Request `OPTIONS` và request upload `POST` hoặc `PUT`. 
-  Kết quả upload ảnh hợp lệ. 
-  Kết quả từ chối file sai loại. 
-  Kết quả từ chối file vượt kích thước. 
-  Ảnh hiển thị trên trang vật phẩm. 
-  Hai S3 Version ID của cùng một object key. 
-  IAM policy của Lambda. 
-  CloudWatch Logs và CloudTrail event `AccessDenied`. 

Ví dụ cách đặt tên hình:

```
```

```
Hình 5.5.7.1: User Frontend được truy cập qua CloudFront
Hình 5.5.7.2: Admin Frontend được truy cập qua CloudFront
Hình 5.5.7.3: JavaScript và CSS được tải thành công
Hình 5.5.7.4: React route hoạt động sau khi tải lại
Hình 5.5.7.5: Truy cập trực tiếp private S3 object bị từ chối
Hình 5.5.7.6: CloudFront OAC đọc được object trong private bucket
Hình 5.5.7.7: Upload ảnh bằng đúng phương thức POST hoặc PUT
Hình 5.5.7.8: Untrusted origin bị CORS từ chối
Hình 5.5.7.9: Ảnh vật phẩm hiển thị đúng trên frontend
Hình 5.5.7.10: S3 tạo version mới khi object được thay thế
Hình 5.5.7.11: Lambda bị AccessDenied khi ghi ngoài phạm vi IAM
```

---

### Tiêu chí đánh giá

Test case chỉ được đánh dấu `PASS` khi:

-  Frontend được truy cập thành công qua CloudFront. 
-  HTML, JavaScript và CSS được tải đúng. 
-  React route hợp lệ hoạt động khi tải lại. 
-  S3 bucket vẫn ở trạng thái private. 
-  CloudFront đọc S3 thông qua OAC. 
-  Chỉ User hợp lệ và có quyền mới được cấp quyền upload. 
-  CORS cho phép đúng origin và đúng method thực tế. 
-  Origin không được tin cậy không nhận CORS permission. 
-  Tệp sai loại và vượt kích thước bị từ chối. 
-  Ảnh hợp lệ được lưu và hiển thị chính xác. 
-  Versioning tạo version mới khi object bị thay thế. 
-  Lambda chỉ ghi vào đúng bucket hoặc prefix được cấp. 
-  Không có credential, token hoặc presigned URL đầy đủ trong logs. 

Test case phải được đánh dấu `FAIL` khi:

-  Frontend chỉ hoạt động khi S3 bucket được đặt public. 
-  CloudFront trả `AccessDenied` cho object hợp lệ. 
-  User Frontend hiển thị nhầm Admin Frontend hoặc ngược lại. 
-  JavaScript hoặc CSS bị trả về dưới dạng `index.html`. 
-  React route hợp lệ trả lỗi khi tải lại. 
-  S3 object có thể được truy cập trực tiếp mà không cần chữ ký. 
-  Origin bất kỳ được phép upload ngoài thiết kế. 
-  File sai loại hoặc vượt giới hạn vẫn được công bố. 
-  User không có quyền vẫn nhận được presigned upload. 
-  Một User có thể ghi đè object của User khác. 
-  Ảnh đã upload không hiển thị hoặc hiển thị phiên bản cache cũ ngoài ý muốn. 
-  Object bị ghi đè nhưng không tạo version mới dù Versioning được yêu cầu. 
-  Lambda ghi được vào frontend bucket hoặc prefix ngoài phạm vi. 
-  Logs làm lộ token, AWS credentials hoặc presigned URL còn hiệu lực. 

Test case được đánh dấu `BLOCKED` khi:

-  Một hoặc nhiều bucket chưa tồn tại. 
-  Frontend chưa được build hoặc deploy. 
-  CloudFront distribution chưa được tạo hoặc chưa `Deployed`. 
-  OAC chưa được cấu hình. 
-  React route fallback chưa được thống nhất. 
-  API tạo presigned upload chưa được triển khai. 
-  Chưa xác định upload thực tế dùng `POST` hay `PUT`. 
-  CORS chưa được cấu hình. 
-  Chưa có cơ chế kiểm tra loại hoặc kích thước tệp. 
-  Item Media Bucket chưa bật Versioning. 
-  Lambda execution role chưa tồn tại. 
-  Không có quyền kiểm tra S3, CloudFront, IAM hoặc CloudWatch Logs