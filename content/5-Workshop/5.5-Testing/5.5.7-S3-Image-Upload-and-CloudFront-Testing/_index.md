---
title: "S3, Image Upload, and CloudFront Testing"
date: 2026-08-03
weight: 7
chapter: false
pre: " <b> 5.5.7. </b> "
---

#### Objective

This testing section verifies the storage and delivery of the system's static content through Amazon S3 and Amazon CloudFront, including:

* The User Frontend is distributed through CloudFront.
* The Admin Frontend is distributed through CloudFront.
* S3 buckets do not allow direct public access.
* CloudFront Origin Access Control (OAC) can correctly read authorized objects.
* Valid users can upload item images to the Item Media Bucket.
* CORS allows only the correct origins and HTTP methods.
* The system rejects unsupported file types or files that exceed the size limit.
* Uploaded images can be displayed on the frontend.
* S3 Versioning creates a new version when an object is replaced.
* Lambda cannot write to buckets outside its granted IAM scope.

Three buckets must be tested:

| Bucket                | Purpose                                   |
| --------------------- | ----------------------------------------- |
| User Frontend Bucket  | Stores the React build for users          |
| Admin Frontend Bucket | Stores the React build for administrators |
| Item Media Bucket     | Stores images of auction items            |

---

#### Frontend Delivery Flow

```text id="fsg7tw"
Browser
→ CloudFront
→ Origin Access Control
→ Private S3 Frontend Bucket
→ CloudFront
→ Browser
```

#### Item Image Upload Flow

```text id="2i8zgl"
Authenticated User
→ Backend API or Lambda
→ Generate presigned URL/presigned POST
→ Browser uploads image directly to Item Media Bucket
→ S3 stores object
→ CloudFront or image delivery URL
→ Frontend displays image
```

---

#### General Testing Preconditions

Before testing begins, the system must meet the following conditions:

* User Frontend Bucket already exists.
* Admin Frontend Bucket already exists.
* Item Media Bucket already exists.
* Block Public Access is enabled for all three buckets.
* User Frontend and Admin Frontend have been deployed to S3.
* Two CloudFront distributions have been created or configured to deliver the correct two frontends.
* CloudFront uses OAC instead of making the S3 bucket public.
* The bucket policy allows only the correct CloudFront distribution to read objects.
* The React build contains `index.html`, JavaScript, CSS, and all required assets.
* React route fallback has been configured.
* CORS is enabled on the Item Media Bucket.
* Versioning is enabled on the Item Media Bucket if required by the system.
* The API or Lambda responsible for creating presigned uploads has been implemented.
* Authentication and authorization for uploads have been implemented.
* File type and file size are validated server-side or through a presigned POST policy.
* Lambda has a separate IAM role with the minimum required permissions.
* CloudFront and S3 access logs or CloudTrail Data Events are enabled if detailed evidence is required.
* The testing environment is separated from production.

If a required component has not yet been implemented, the corresponding test case must be marked as `BLOCKED`.

---

#### Determine the Image Upload Method Before Testing

Before running the CORS test cases, the exact upload method used by the frontend must be identified.

##### Case 1: Presigned PUT URL

The frontend sends the file contents directly using:

```http id="y9jp80"
PUT /object-key HTTP/1.1
Content-Type: image/jpeg
```

CORS must allow at minimum:

```text id="brt49f"
PUT
```

##### Case 2: Presigned POST

The frontend sends a `multipart/form-data` form to S3 using:

```http id="hublpe"
POST / HTTP/1.1
Content-Type: multipart/form-data
```

CORS must allow at minimum:

```text id="u0c4ab"
POST
```

##### How to Verify

1. Open the image upload page.
2. Open Developer Tools.
3. Select the `Network` tab.
4. Upload an image.
5. Select the request sent to S3.
6. Check the `Request Method` field.
7. Record whether the result is `PUT` or `POST`.
8. Check the `OPTIONS` preflight request if the browser generates one.

Do not configure and test `POST` if the actual application uses `PUT`, or vice versa.

---

#### Test Data

| Data                    | Description                                                     |
| ----------------------- | --------------------------------------------------------------- |
| User Frontend URL       | CloudFront URL for users                                        |
| Admin Frontend URL      | CloudFront URL for administrators                               |
| User Bucket Object URL  | Direct URL to an object in the User Frontend Bucket             |
| Admin Bucket Object URL | Direct URL to an object in the Admin Frontend Bucket            |
| Item Media Object URL   | Direct URL to an image in the Item Media Bucket                 |
| Trusted User Origin     | Allowed user frontend origin                                    |
| Trusted Admin Origin    | Allowed administrator frontend origin                           |
| Untrusted Origin        | Origin not included in the CORS allowlist                       |
| Valid User              | Authenticated User with image-upload permission                 |
| Invalid User            | User who is not logged in or uses an invalid or expired token   |
| Valid Image             | Valid JPEG, PNG, or WebP file according to system rules         |
| Invalid File            | `.exe`, `.html`, `.js`, `.pdf`, or another prohibited file type |
| Oversized Image         | File exceeding the upload size limit                            |
| Existing Object Key     | Existing object key used to test Versioning                     |
| Allowed Lambda          | Lambda allowed to write to the Item Media Bucket                |
| Restricted Lambda       | Lambda not allowed to write to frontend buckets                 |

Production data must not be used during testing.

---

### STORAGE-01 — Access User Frontend Through CloudFront

| Field               | Content                                                                                                                                                                                                 |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `STORAGE-01`                                                                                                                                                                                            |
| **Test Name**       | Access User Frontend through CloudFront                                                                                                                                                                 |
| **Objective**       | Verify that users can open the User Frontend from the CloudFront distribution.                                                                                                                          |
| **Preconditions**   | User Frontend has been built and deployed; the CloudFront distribution is in `Deployed` state.                                                                                                          |
| **Test Steps**      | 1. Open a browser in incognito/private mode.<br>2. Access the User Frontend CloudFront URL.<br>3. Check the HTTP response.<br>4. Check the interface and browser console.<br>5. Check response headers. |
| **Expected Result** | The page returns `200 OK`; the interface displays correctly; no XML Access Denied response appears; no critical asset-loading errors occur; the response is served through CloudFront.                  |
| **Actual Result**   | Record URL, status code, response time, and display result.                                                                                                                                             |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                           |
| **Evidence**        | Interface screenshot, Network tab, and response headers.                                                                                                                                                |

---

### STORAGE-02 — Access Admin Frontend Through CloudFront

| Field               | Content                                                                                                                                                                                                                 |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `STORAGE-02`                                                                                                                                                                                                            |
| **Test Name**       | Access Admin Frontend through CloudFront                                                                                                                                                                                |
| **Objective**       | Verify that the Admin Frontend is delivered from the correct CloudFront distribution.                                                                                                                                   |
| **Preconditions**   | Admin Frontend has been built and deployed; the CloudFront distribution is ready.                                                                                                                                       |
| **Test Steps**      | 1. Open the Admin Frontend CloudFront URL.<br>2. Check the status code.<br>3. Check the login page or default page.<br>4. Check the Network tab and browser console.<br>5. Verify the distribution serving the request. |
| **Expected Result** | The page returns `200 OK`; the Admin interface displays correctly; the User Frontend is not loaded by mistake; assets are delivered through the correct CloudFront distribution.                                        |
| **Actual Result**   | Record URL, distribution, status code, and interface result.                                                                                                                                                            |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                           |
| **Evidence**        | Admin interface screenshot, Network tab, and CloudFront headers.                                                                                                                                                        |

---

### STORAGE-03 — Load `index.html`, JavaScript, and CSS

| Field               | Content                                                                                                                                                                                                       |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `STORAGE-03`                                                                                                                                                                                                  |
| **Test Name**       | Load all frontend assets                                                                                                                                                                                      |
| **Objective**       | Verify that `index.html` and JavaScript/CSS bundles load successfully with the correct content types.                                                                                                         |
| **Preconditions**   | The complete frontend build has been uploaded to S3.                                                                                                                                                          |
| **Test Steps**      | 1. Open User Frontend and Admin Frontend.<br>2. Open Developer Tools → Network.<br>3. Reload the page.<br>4. Filter by `Doc`, `JS`, and `CSS`.<br>5. Check status code, `Content-Type`, and response body.    |
| **Expected Result** | `index.html`, JavaScript, and CSS all return `200 OK`; JavaScript has an appropriate content type; CSS returns `text/css`; JavaScript/CSS content is not replaced by `index.html`; no MIME type errors occur. |
| **Actual Result**   | Record object name, status code, content type, and size.                                                                                                                                                      |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                 |
| **Evidence**        | Network waterfall, response headers, and browser console.                                                                                                                                                     |

> Special attention must be given to cases where a JavaScript file is missing but CloudFront incorrectly returns `index.html` with `200 OK`. Such a case must be marked as `FAIL`.

---

### STORAGE-04 — Reload React Route

| Field               | Content                                                                                                                                                                                                                                       |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `STORAGE-04`                                                                                                                                                                                                                                  |
| **Test Name**       | Reload a client-side React route                                                                                                                                                                                                              |
| **Objective**       | Verify that React routes work when accessed directly or when the page is reloaded.                                                                                                                                                            |
| **Preconditions**   | The application uses client-side routing; CloudFront has an appropriate fallback configuration.                                                                                                                                               |
| **Test Steps**      | 1. Access a valid route, for example `/auction-items/item-001`.<br>2. Reload using `Ctrl+R` or `F5`.<br>3. Paste the route URL directly into a new tab.<br>4. Check the response and interface.<br>5. Attempt to access a non-existent asset. |
| **Expected Result** | A valid route displays correctly instead of returning S3 `AccessDenied` or another unexpected error; React initializes successfully; a missing asset is not incorrectly masked as a successful HTML response unless intentionally designed.   |
| **Actual Result**   | Record route, status code, response type, and display result.                                                                                                                                                                                 |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                 |
| **Evidence**        | Route URL, Network tab, CloudFront configuration, and interface screenshot.                                                                                                                                                                   |

---

### STORAGE-05 — Reject Direct Access to Private S3 Objects

| Field               | Content                                                                                                                                                                                             |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `STORAGE-05`                                                                                                                                                                                        |
| **Test Name**       | Block direct access to private S3 bucket                                                                                                                                                            |
| **Objective**       | Verify that Internet users cannot bypass CloudFront and directly read S3 objects.                                                                                                                   |
| **Preconditions**   | Block Public Access is enabled; the bucket is not configured for direct public website hosting.                                                                                                     |
| **Test Steps**      | 1. Copy the direct S3 object URL for `index.html`.<br>2. Open the URL in an incognito browser.<br>3. Repeat for JavaScript, CSS, or an image.<br>4. Check bucket policy and public access settings. |
| **Expected Result** | The direct request is rejected with `403 AccessDenied` or equivalent; the object is not returned; the bucket and object are not public.                                                             |
| **Actual Result**   | Record the object URL with the bucket name masked if necessary, status code, and response code.                                                                                                     |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                       |
| **Evidence**        | `403` response, Block Public Access configuration, and bucket policy.                                                                                                                               |

---

### STORAGE-06 — CloudFront OAC Can Read Objects

| Field               | Content                                                                                                                                                                                                                               |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `STORAGE-06`                                                                                                                                                                                                                          |
| **Test Name**       | CloudFront accesses private S3 using OAC                                                                                                                                                                                              |
| **Objective**       | Verify that OAC allows only the correct CloudFront distribution to read objects from the private bucket.                                                                                                                              |
| **Preconditions**   | OAC is attached to the S3 origin; bucket policy is restricted by the CloudFront service principal and distribution ARN.                                                                                                               |
| **Test Steps**      | 1. Access an object through the CloudFront URL.<br>2. Confirm that the request succeeds.<br>3. Access the same object through the direct S3 URL.<br>4. Check origin configuration.<br>5. Check the bucket policy and `AWS:SourceArn`. |
| **Expected Result** | CloudFront successfully returns the object; direct S3 access is denied; OAC is active; the bucket policy does not grant public read access and allows only the required distribution.                                                 |
| **Actual Result**   | Record CloudFront status, direct S3 status, OAC ID, and masked distribution ARN if needed.                                                                                                                                            |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                         |
| **Evidence**        | Two HTTP responses, CloudFront origin configuration, and bucket policy.                                                                                                                                                               |

---

### STORAGE-07 — Authorized User Uploads an Item Image

| Field               | Content                                                                                                                                                                                                                                                                           |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `STORAGE-07`                                                                                                                                                                                                                                                                      |
| **Test Name**       | Image upload by an authorized User                                                                                                                                                                                                                                                |
| **Objective**       | Verify that a valid User can upload an item image to the correct Item Media Bucket.                                                                                                                                                                                               |
| **Preconditions**   | User is authenticated; User has permission to edit the item; presigned-upload API is working.                                                                                                                                                                                     |
| **Test Steps**      | 1. Log in as Valid User.<br>2. Select an item the User is allowed to edit.<br>3. Select Valid Image.<br>4. Request a presigned upload.<br>5. Upload using the correct `POST` or `PUT` method.<br>6. Check the S3 response.<br>7. Check the object in the bucket and its metadata. |
| **Expected Result** | Backend verifies the User and item permission; the presigned upload is generated for the correct bucket/key; upload succeeds; the object appears under the correct prefix; content type and size are correct; the URL expires and does not grant broader access than required.    |
| **Actual Result**   | Record masked User ID, Item ID, method, object key, status code, and size.                                                                                                                                                                                                        |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                     |
| **Evidence**        | API response with signature masked, Network request, S3 object metadata, and CloudWatch Logs.                                                                                                                                                                                     |

> Do not include the complete active presigned URL in testing documentation because the URL contains signature information that could be used to upload an object.

---

### STORAGE-08 — CORS Allows the Correct Origin and HTTP Method

| Field               | Content                                                                                                                                                                                                                                              |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `STORAGE-08`                                                                                                                                                                                                                                         |
| **Test Name**       | Allow upload from trusted origin                                                                                                                                                                                                                     |
| **Objective**       | Verify that the Item Media Bucket allows only the correct frontend origin and the actual upload method.                                                                                                                                              |
| **Preconditions**   | It has been confirmed whether the application uses presigned `POST` or presigned `PUT`.                                                                                                                                                              |
| **Test Steps**      | 1. Open the frontend from the trusted origin.<br>2. Perform an upload.<br>3. Check the `OPTIONS` request if generated.<br>4. Check `Access-Control-Allow-Origin`.<br>5. Check `Access-Control-Allow-Methods`.<br>6. Check the actual upload request. |
| **Expected Result** | Preflight succeeds; the response allows only the trusted origin; the actual `POST` or `PUT` method is allowed; required headers such as `Content-Type` are accepted; the browser completes the upload without CORS errors.                           |
| **Actual Result**   | Record origin, method, request headers, and response CORS headers.                                                                                                                                                                                   |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                        |
| **Evidence**        | Network tab showing the `OPTIONS` request and upload request.                                                                                                                                                                                        |

---

### STORAGE-09 — Reject an Untrusted Origin

| Field               | Content                                                                                                                                                                                                                                                                                |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `STORAGE-09`                                                                                                                                                                                                                                                                           |
| **Test Name**       | Do not allow upload from an untrusted origin                                                                                                                                                                                                                                           |
| **Objective**       | Verify that an origin outside the allowlist is not permitted by CORS.                                                                                                                                                                                                                  |
| **Preconditions**   | A testing origin that is not included in the CORS configuration is available.                                                                                                                                                                                                          |
| **Test Steps**      | 1. Send a preflight request with the `Origin` header set to the untrusted origin.<br>2. Use the application's actual requested method.<br>3. Check response headers.<br>4. Attempt an upload from a browser hosted on the untrusted origin.<br>5. Check whether an object was created. |
| **Expected Result** | The response does not return `Access-Control-Allow-Origin` for the untrusted origin; the browser blocks the request according to CORS; the upload does not complete through the tested browser flow; no unintended object is created.                                                  |
| **Actual Result**   | Record origin, preflight status, CORS headers, and object result.                                                                                                                                                                                                                      |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                          |
| **Evidence**        | Preflight response, browser console, and S3 object verification.                                                                                                                                                                                                                       |

> CORS is not an authentication mechanism. A request outside a browser can still call a valid presigned URL. Therefore, presigned URLs must have short expiration periods and must be restricted to the correct key, method, content type, and required conditions.

---

### STORAGE-10 — Reject Unsupported File Types

| Field               | Content                                                                                                                                                                                                                                                       |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `STORAGE-10`                                                                                                                                                                                                                                                  |
| **Test Name**       | Reject unsupported file types                                                                                                                                                                                                                                 |
| **Objective**       | Verify that the system does not accept files outside the allowed image-format list.                                                                                                                                                                           |
| **Preconditions**   | An allowlist of image formats and file-type validation mechanism have been implemented.                                                                                                                                                                       |
| **Test Steps**      | 1. Select an `.exe`, `.html`, `.js`, or another prohibited file.<br>2. Try renaming its extension to `.jpg`.<br>3. Request a presigned upload.<br>4. If a URL is still issued, attempt the upload.<br>5. Check S3 and logs.                                   |
| **Expected Result** | The file is rejected with a code such as `UNSUPPORTED_FILE_TYPE`; the system does not rely only on the file extension or client-supplied `Content-Type`; the object is not published as a valid image; no executable content is served from the media domain. |
| **Actual Result**   | Record filename, extension, detected MIME type, and error code.                                                                                                                                                                                               |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                 |
| **Evidence**        | API response, file-validation logs, S3 object listing, and frontend.                                                                                                                                                                                          |

---

### STORAGE-11 — Reject Files Exceeding the Size Limit

| Field               | Content                                                                                                                                                                                                                                        |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `STORAGE-11`                                                                                                                                                                                                                                   |
| **Test Name**       | Reject an image exceeding the file-size limit                                                                                                                                                                                                  |
| **Objective**       | Verify that images larger than the configured limit are not accepted.                                                                                                                                                                          |
| **Preconditions**   | A size limit has been defined, for example `5 MB`.                                                                                                                                                                                             |
| **Test Steps**      | 1. Prepare an image below the limit.<br>2. Prepare an image exactly at the limit.<br>3. Prepare an image above the limit.<br>4. Attempt to upload each file.<br>5. Check response, S3, and logs.                                               |
| **Expected Result** | Images below or exactly at the limit are processed according to the rules; an image exceeding the limit is rejected with `FILE_TOO_LARGE` or equivalent; the oversized object is not stored or is quarantined/deleted according to the design. |
| **Actual Result**   | Record configured limit, each file size, status, and error code.                                                                                                                                                                               |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                  |
| **Evidence**        | File sizes, API/S3 response, object metadata, and logs.                                                                                                                                                                                        |

> With presigned `POST`, size can be controlled using the `content-length-range` policy condition. With presigned `PUT`, the actual size-control mechanism must be verified because frontend-only validation is not sufficiently secure.

---

### STORAGE-12 — Uploaded Image Displays Correctly on the Frontend

| Field               | Content                                                                                                                                                                                                                                                                            |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `STORAGE-12`                                                                                                                                                                                                                                                                       |
| **Test Name**       | Deliver and display an item image                                                                                                                                                                                                                                                  |
| **Objective**       | Verify that an uploaded image is correctly associated with the item and displayed through an authorized delivery URL.                                                                                                                                                              |
| **Preconditions**   | STORAGE-07 succeeded; item data stores the correct object key or media URL.                                                                                                                                                                                                        |
| **Test Steps**      | 1. Complete the image upload.<br>2. Open the item detail page.<br>3. Do not use an S3 Console URL to display the image.<br>4. Check the image request in the Network tab.<br>5. Check content type, size, and cache headers.<br>6. Reopen the page or check it in another browser. |
| **Expected Result** | The image displays correctly; no broken image appears; the request returns `200 OK`; content matches the uploaded file; the object is retrieved through CloudFront or the designed delivery mechanism; users are not granted permission to browse the entire bucket.               |
| **Actual Result**   | Record Item ID, masked object key, delivery URL, status, and display result.                                                                                                                                                                                                       |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                      |
| **Evidence**        | Interface screenshot, Network response, and S3 object metadata.                                                                                                                                                                                                                    |

---

### STORAGE-13 — S3 Versioning Creates a New Version

| Field               | Content                                                                                                                                                                                                                                     |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `STORAGE-13`                                                                                                                                                                                                                                |
| **Test Name**       | Create a new version when replacing an object                                                                                                                                                                                               |
| **Objective**       | Verify that overwriting the same object key does not immediately destroy the previous version.                                                                                                                                              |
| **Preconditions**   | Versioning is enabled for the bucket under test.                                                                                                                                                                                            |
| **Test Steps**      | 1. Upload image A to a specific object key.<br>2. Record the first `VersionId`.<br>3. Upload image B to the same object key.<br>4. Record the second `VersionId`.<br>5. List the versions.<br>6. Check the current version and old version. |
| **Expected Result** | Two different `VersionId` values exist; image B is the current version; image A remains available as the older version; no `null` Version ID appears if Versioning was correctly enabled before upload.                                     |
| **Actual Result**   | Record object key, Version ID 1, Version ID 2, and current version.                                                                                                                                                                         |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                               |
| **Evidence**        | S3 Versions view, object metadata, and CLI output if used.                                                                                                                                                                                  |

> If the object is delivered through CloudFront using the same URL, cache invalidation or a versioned/hashed object-key strategy must also be verified. S3 Versioning does not automatically remove an old CloudFront cached response.

---

### STORAGE-14 — Lambda Cannot Write Outside Its IAM Scope

| Field               | Content                                                                                                                                                                                                                                                                                                                                               |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `STORAGE-14`                                                                                                                                                                                                                                                                                                                                          |
| **Test Name**       | Restrict Lambda S3 write permissions                                                                                                                                                                                                                                                                                                                  |
| **Objective**       | Verify that Lambda can operate only on buckets and prefixes explicitly allowed by IAM.                                                                                                                                                                                                                                                                |
| **Preconditions**   | The Lambda execution role follows least privilege; one valid bucket/prefix and one out-of-scope bucket/prefix are available.                                                                                                                                                                                                                          |
| **Test Steps**      | 1. Trigger Lambda to write to the authorized Item Media Bucket and prefix.<br>2. Confirm that the operation succeeds.<br>3. Attempt to write to the User Frontend Bucket.<br>4. Attempt to write to the Admin Frontend Bucket.<br>5. Attempt to write to another prefix in the Item Media Bucket.<br>6. Check results and CloudTrail/CloudWatch Logs. |
| **Expected Result** | Lambda successfully writes only to the authorized bucket/prefix; out-of-scope operations are denied with `AccessDenied`; no overly broad wildcard write permission such as `arn:aws:s3:::*/*` is granted; errors are logged without exposing sensitive information.                                                                                   |
| **Actual Result**   | Record role, action, masked resource, result, and error code.                                                                                                                                                                                                                                                                                         |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                                                         |
| **Evidence**        | IAM policy, Lambda logs, CloudTrail event, and S3 object listing.                                                                                                                                                                                                                                                                                     |

> Negative testing should be performed using a dedicated test function or controlled testing environment. Do not modify a production Lambda merely to intentionally write to an out-of-scope bucket.

---

### Expected Access Control Matrix

| Component                     | User Frontend Bucket | Admin Frontend Bucket | Item Media Bucket                                    |
| ----------------------------- | -------------------- | --------------------- | ---------------------------------------------------- |
| User accesses S3 directly     | Denied               | Denied                | Denied, except with a valid signed URL               |
| User CloudFront Distribution  | Read allowed         | Denied                | According to design                                  |
| Admin CloudFront Distribution | Denied               | Read allowed          | According to design                                  |
| Upload-generation Lambda      | No write             | No write              | Generate URL only or write only to authorized prefix |
| Image-processing Lambda       | No write             | No write              | Read/write only to authorized prefix                 |
| Unauthenticated User          | No upload            | No upload             | Must not receive a presigned upload                  |
| Authorized User               | No direct write      | No direct write       | Upload through restricted presigned request          |

---

### Upload Verification Matrix

| Scenario                                   | Presigned Upload Issued       | S3 Stores Object                | Frontend Displays |
| ------------------------------------------ | ----------------------------- | ------------------------------- | ----------------- |
| Authorized User, valid image type and size | Yes                           | Yes                             | Yes               |
| User not logged in                         | No                            | No                              | No                |
| User without permission on the item        | No                            | No                              | No                |
| Trusted origin                             | According to User permissions | Can upload                      | Yes               |
| Untrusted origin                           | Not allowed by browser CORS   | Not through normal browser flow | No                |
| Invalid file type                          | No or quarantined             | Not stored as valid image       | No                |
| Oversized file                             | No or rejected by S3          | No                              | No                |
| Expired presigned URL                      | Not applicable                | Rejected                        | No                |
| Object key outside signed URL scope        | Not applicable                | Rejected                        | No                |

---

### CORS Verification Requirements

The Item Media Bucket CORS configuration must:

* Contain only trusted origins.
* Distinguish User Frontend and Admin Frontend if their upload permissions differ.
* Allow only the method actually used: `POST` or `PUT`.
* Allow only required headers.
* Avoid `AllowedOrigins: ["*"]` when the design requires origin restrictions.
* Avoid unnecessary methods such as `DELETE`.
* Return correct CORS headers for `OPTIONS` preflight.
* Not confuse CORS with authentication or authorization.
* Be retested in a real browser whenever CORS configuration changes.

Example for presigned `PUT`:

```json id="otvp2y"
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

Example for presigned `POST`:

```json id="7on7ln"
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

The origins and headers in these examples must be replaced with the actual system values. Do not copy sample configuration directly into production without comparing it against real browser requests.

---

### CloudFront Cache Verification Requirements

Verify the following:

* `index.html` is not cached for too long after each deployment.
* Files with hashes in their names, such as JavaScript/CSS bundles, may use long-term caching.
* Object `Content-Type` values are correct.
* User Frontend and Admin Frontend do not accidentally use the wrong origins.
* CloudFront does not expose private S3 URLs.
* The cache key does not contain unnecessary data.
* Images are cached according to an appropriate policy.
* When the same object key is replaced, CloudFront is invalidated or a new object name is used.
* CloudFront custom error responses do not unintentionally hide asset failures by returning `index.html`.
* HTTPS is enforced, or HTTP is redirected to HTTPS.

Recommended cache policy:

| Object Type                                      | Suggested Cache-Control                    |
| ------------------------------------------------ | ------------------------------------------ |
| `index.html`                                     | `no-cache` or short cache duration         |
| JavaScript/CSS with content hash                 | `public, max-age=31536000, immutable`      |
| Image with immutable object key                  | `public, max-age=31536000, immutable`      |
| Image that may be overwritten using the same key | Short cache duration or cache invalidation |

---

### Upload Security Verification Requirements

At minimum, the upload flow must verify:

* The User is authenticated.
* The User has permission to edit the correct item.
* Object keys are controlled by the server.
* A User cannot change the key to overwrite another User's image.
* Input filenames are not used directly as keys if path-manipulation risk exists.
* Presigned URLs have a short expiration time.
* Presigned URLs are restricted to the correct bucket, key, and HTTP method.
* Tokens and complete presigned URLs are not written to logs.
* File extension and `Content-Type` are not the only evidence used to determine file type.
* File size is controlled server-side or by an S3 policy.
* Images should be inspected or processed before publication when the system accepts untrusted content.
* Objects must not receive a `public-read` ACL.
* Bucket owner enforced should be used where appropriate.
* Server-side encryption is enabled according to requirements.
* Lifecycle rules are configured for old versions or incomplete uploads if required.

---

### IAM Verification Requirements

Lambda IAM roles must follow the principle of least privilege:

* Grant only required actions.
* Grant access only to the correct bucket.
* Restrict permissions to the correct prefix when possible.
* Separate read and write permissions where appropriate.
* Do not use `s3:*`.
* Do not use resource `*` for object operations unless required.
* Do not grant media-processing Lambda write access to User Frontend or Admin Frontend buckets.
* Permission to generate a presigned URL does not mean users receive AWS credentials.
* Explicit Deny from a bucket policy, permission boundary, or SCP must still take effect.

Example of the expected resource scope:

```json id="9dl3wn"
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject",
    "s3:PutObject"
  ],
  "Resource": "arn:aws:s3:::item-media-bucket/items/*"
}
```

The bucket name must be replaced with the actual system resource.

---

### Logging and Evidence Verification Requirements

Logs should contain:

* Correlation ID or Request ID.
* Verified User ID, masked where appropriate.
* Item ID.
* Object key.
* Upload HTTP method.
* Content type.
* File size.
* Authentication result.
* Authorization result.
* Presigned-upload generation result.
* Image-processing result.
* S3 error code.
* Lambda Request ID.
* Processing time.

Logs must not contain:

* Access Token.
* ID Token.
* Refresh Token.
* AWS access key or secret key.
* Complete active presigned URL.
* Signature contained in query strings.
* Login cookies.
* Base64 image data.
* Unnecessary personal information.

---

### Result Summary Table

| ID           | Test Content                      | Main Expected Result                                      | Status     |
| ------------ | --------------------------------- | --------------------------------------------------------- | ---------- |
| `STORAGE-01` | User Frontend through CloudFront  | Returns `200`, interface displays correctly               | Not tested |
| `STORAGE-02` | Admin Frontend through CloudFront | Returns `200`, correct Admin UI                           | Not tested |
| `STORAGE-03` | Load HTML, JavaScript, and CSS    | Assets load with correct content type                     | Not tested |
| `STORAGE-04` | Reload React route                | Route displays without unexpected errors                  | Not tested |
| `STORAGE-05` | Direct S3 access                  | Rejected                                                  | Not tested |
| `STORAGE-06` | CloudFront OAC reads S3           | CloudFront can read, direct access is blocked             | Not tested |
| `STORAGE-07` | Authorized User uploads image     | Image stored in correct bucket and prefix                 | Not tested |
| `STORAGE-08` | CORS trusted origin/method        | Upload succeeds                                           | Not tested |
| `STORAGE-09` | CORS untrusted origin             | Browser rejects                                           | Not tested |
| `STORAGE-10` | Invalid file type                 | Rejected                                                  | Not tested |
| `STORAGE-11` | File exceeds size limit           | Rejected                                                  | Not tested |
| `STORAGE-12` | Display image on frontend         | Image displays correctly                                  | Not tested |
| `STORAGE-13` | S3 Versioning                     | New Version ID is created                                 | Not tested |
| `STORAGE-14` | Lambda IAM restrictions           | Authorized writes succeed; out-of-scope writes are denied | Not tested |

---

### Testing Evidence

Evidence should include:

* User Frontend and Admin Frontend displayed through CloudFront.
* Network requests for `index.html`, JavaScript, and CSS.
* Response headers from CloudFront.
* React route reload result.
* Response when directly accessing a private S3 object.
* CloudFront origin and OAC configuration.
* Bucket policy and Block Public Access settings.
* `OPTIONS` request and `POST` or `PUT` upload request.
* Successful valid-image upload result.
* Invalid-file rejection result.
* Oversized-file rejection result.
* Image displayed on the item page.
* Two S3 Version IDs for the same object key.
* Lambda IAM policy.
* CloudWatch Logs and CloudTrail `AccessDenied` event.

Example figure naming:

```text id="rvt7di"
Figure 5.5.7.1: User Frontend accessed through CloudFront
Figure 5.5.7.2: Admin Frontend accessed through CloudFront
Figure 5.5.7.3: JavaScript and CSS loaded successfully
Figure 5.5.7.4: React route works after page reload
Figure 5.5.7.5: Direct access to private S3 object is denied
Figure 5.5.7.6: CloudFront OAC reads an object from the private bucket
Figure 5.5.7.7: Image uploaded using the correct POST or PUT method
Figure 5.5.7.8: Untrusted origin rejected by CORS
Figure 5.5.7.9: Item image displayed correctly on the frontend
Figure 5.5.7.10: S3 creates a new version when an object is replaced
Figure 5.5.7.11: Lambda receives AccessDenied when writing outside IAM scope
```

---

### Evaluation Criteria

A test case may only be marked as `PASS` when:

* The frontend can be accessed successfully through CloudFront.
* HTML, JavaScript, and CSS load correctly.
* A valid React route works after page reload.
* The S3 bucket remains private.
* CloudFront reads S3 through OAC.
* Only valid and authorized Users receive upload permission.
* CORS allows the correct origin and actual HTTP method.
* An untrusted origin does not receive CORS permission.
* Unsupported and oversized files are rejected.
* Valid images are stored and displayed correctly.
* Versioning creates a new version when an object is replaced.
* Lambda writes only to the authorized bucket or prefix.
* Logs contain no credentials, tokens, or complete presigned URLs.

A test case must be marked as `FAIL` when:

* The frontend works only when the S3 bucket is made public.
* CloudFront returns `AccessDenied` for a valid object.
* User Frontend displays Admin Frontend or vice versa.
* JavaScript or CSS is returned as `index.html`.
* A valid React route fails when the page is reloaded.
* An S3 object can be accessed directly without a signature.
* Any origin can upload outside the intended design.
* An invalid or oversized file is still published.
* An unauthorized User still receives a presigned upload.
* One User can overwrite another User's object.
* An uploaded image does not display or unintentionally displays an outdated cached version.
* An overwritten object does not create a new version even though Versioning is required.
* Lambda can write to a frontend bucket or an out-of-scope prefix.
* Logs expose tokens, AWS credentials, or active presigned URLs.

A test case is marked as `BLOCKED` when:

* One or more required buckets do not exist.
* The frontend has not been built or deployed.
* The CloudFront distribution has not been created or is not yet `Deployed`.
* OAC has not been configured.
* React route fallback has not been finalized.
* The API for generating presigned uploads has not been implemented.
* It has not yet been determined whether the actual upload uses `POST` or `PUT`.
* CORS has not been configured.
* File type or file-size validation has not been implemented.
* The Item Media Bucket has not enabled Versioning.
* The Lambda execution role does not exist.
* Permission is unavailable to inspect S3, CloudFront, IAM, or CloudWatch Logs.
