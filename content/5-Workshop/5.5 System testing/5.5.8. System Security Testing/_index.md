### 5.5.8. System Security Testing

The objective of security testing is to verify that the system actually rejects invalid or unauthorized behavior, rather than merely confirming that mechanisms such as JWT, IAM, CORS, or S3 Block Public Access have been configured.

The testing focuses on:

* JWT authentication for REST APIs.
* Authorization between User and Admin roles.
* Resource-level authorization.
* S3 and CORS protection.
* Lambda IAM permission restrictions.
* WebSocket connection and message authentication.
* Protection of sensitive data in logs.
* Control of information returned in error responses.

Test cases involving modification of resource IDs must verify authorization against the actual object being accessed, because an API that accepts an ID without checking ownership or access rights may lead to Broken Object Level Authorization. JWT signature and expiration validation are also important requirements for preventing Broken Authentication.

---

#### Testing Scope

The scope includes:

```text id="ub5rr8"
Amazon API Gateway
AWS Lambda
JWT authentication
User/Admin authorization
Auction sessions
Auction items
Bids
Image upload
Amazon S3
CloudFront
CORS
API Gateway WebSocket API
AWS IAM
AWS Secrets Manager
Amazon CloudWatch Logs
AWS CloudTrail
```

Destructive testing must not be performed in production. Test cases that attempt to access resources outside an IAM Policy must be performed using a test function, test role, or development/staging environment.

---

#### Preconditions

Before testing, ensure that:

* The REST API has been deployed through API Gateway and Lambda.
* Protected APIs use an authorizer or an equivalent authentication mechanism.
* JWTs include a signature and expiration time.
* The system has at least two roles: `USER` and `ADMIN`.
* At least two different User accounts are available.
* Each User owns separate test resources.
* S3 Block Public Access has been configured.
* CORS contains a list of trusted origins.
* Lambda execution roles have been created.
* WebSocket `$connect` and message routes have been implemented.
* CloudWatch Logs are enabled.
* The testing environment is separated from production.
* The tester has permission to inspect the required configuration and logs.

For WebSocket APIs, authentication should be performed during `$connect`. Because WebSocket authorization at the connection stage does not automatically validate every later message, the application must continue to verify identity, permissions, and message structure in subsequent routes.

If a required component has not yet been implemented, the corresponding test case must be marked as `BLOCKED`.

---

#### Test Data

| Data                        | Description                                                        |
| --------------------------- | ------------------------------------------------------------------ |
| Protected API               | API requiring authentication, for example `GET /users/me`          |
| Admin API                   | API restricted to Admin users                                      |
| Valid User Token            | Unexpired JWT belonging to a User                                  |
| Valid Admin Token           | Unexpired JWT belonging to an Admin                                |
| Forged Token                | JWT with a modified payload or signed using a different secret/key |
| Expired Token               | JWT whose `exp` claim has expired                                  |
| Unsupported Algorithm Token | Token using an algorithm not allowed by the system                 |
| User A                      | User who owns Resource A                                           |
| User B                      | User who owns Resource B                                           |
| Resource A                  | Session, item, image, or data belonging to User A                  |
| Resource B                  | Session, item, image, or data belonging to User B                  |
| Trusted Origin              | Frontend origin included in the CORS allowlist                     |
| Untrusted Origin            | Origin not included in the CORS allowlist                          |
| Private S3 Object URL       | Direct URL to an object in a private bucket                        |
| Allowed AWS Resource        | Bucket, secret, or prefix Lambda is allowed to access              |
| Restricted AWS Resource     | Resource outside Lambda's IAM Policy                               |
| Valid WebSocket Message     | JSON containing valid action, fields, and data types               |
| Invalid WebSocket Message   | Invalid JSON, missing fields, or unsupported action                |
| Oversized WebSocket Message | Message exceeding the application's configured size limit          |
| Correlation ID              | ID used to locate the corresponding request in logs                |

Production tokens, accounts, and data must not be used.

---

### SEC-01 — API Rejects Requests Without a Token

| Field               | Content                                                                                                                                                                                                                                                                       |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `SEC-01`                                                                                                                                                                                                                                                                      |
| **Test Name**       | Reject requests without an access token                                                                                                                                                                                                                                       |
| **Objective**       | Verify that a protected API does not allow anonymous users to access it.                                                                                                                                                                                                      |
| **Preconditions**   | At least one protected API is available.                                                                                                                                                                                                                                      |
| **Test Steps**      | 1. Send a request to the protected API without an `Authorization` header.<br>2. Send a request with an empty `Authorization` header.<br>3. Send `Authorization: Basic abc`.<br>4. Send `Authorization: Bearer` without a token.<br>5. Check the response and CloudWatch Logs. |
| **Expected Result** | All requests are rejected with `401 Unauthorized`; the business Lambda does not modify data; the response does not return User information; the error uses a stable code such as `AUTH_TOKEN_REQUIRED`.                                                                       |
| **Actual Result**   | Record endpoint, test scenario, status code, and error code.                                                                                                                                                                                                                  |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                 |
| **Evidence**        | Request/response with sensitive data masked and CloudWatch Logs.                                                                                                                                                                                                              |

---

### SEC-02 — Forged Token Is Rejected

| Field               | Content                                                                                                                                                                                                                                                                                                                                                                    |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `SEC-02`                                                                                                                                                                                                                                                                                                                                                                   |
| **Test Name**       | Reject JWTs with invalid signatures or algorithms                                                                                                                                                                                                                                                                                                                          |
| **Objective**       | Verify that the system validates JWT signatures and accepts only configured algorithms.                                                                                                                                                                                                                                                                                    |
| **Preconditions**   | A Valid User Token and a token-generation tool for the test environment are available.                                                                                                                                                                                                                                                                                     |
| **Test Steps**      | 1. Decode the Valid User Token in the testing environment.<br>2. Modify the `sub` or `role` claim while keeping the original signature.<br>3. Send the modified token to the protected API.<br>4. Create a token signed using a different secret/key and send it.<br>5. Try a token using `alg: none` or another unsupported algorithm.<br>6. Check the response and logs. |
| **Expected Result** | All forged tokens are rejected with `401 Unauthorized`; the system does not trust the payload before verifying the signature; only the configured algorithm is accepted; no data is modified.                                                                                                                                                                              |
| **Actual Result**   | Record token type, algorithm, status, and error code without recording the complete token.                                                                                                                                                                                                                                                                                 |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                                                                              |
| **Evidence**        | Request/response with token masked and JWT verification logs.                                                                                                                                                                                                                                                                                                              |

> Do not include complete access tokens in documentation or screenshots.

---

### SEC-03 — Expired Token Is Rejected

| Field               | Content                                                                                                                                                                                                                                       |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `SEC-03`                                                                                                                                                                                                                                      |
| **Test Name**       | Reject an expired JWT                                                                                                                                                                                                                         |
| **Objective**       | Verify that the `exp` claim is checked at request time.                                                                                                                                                                                       |
| **Preconditions**   | An Expired Token is available, or a short-lived token can be generated in the testing environment.                                                                                                                                            |
| **Test Steps**      | 1. Send the token while it is still valid if using a short-lived token.<br>2. Allow the token to expire.<br>3. Send the same request again.<br>4. Try a token without an `exp` claim if `exp` is required.<br>5. Check the response and data. |
| **Expected Result** | A valid token is processed according to the User's permissions; expired tokens or tokens missing required claims are rejected with `401 Unauthorized`; no business operation is performed.                                                    |
| **Actual Result**   | Record masked expiration time, test time, status, and error code.                                                                                                                                                                             |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                 |
| **Evidence**        | Response and CloudWatch Logs without the complete token.                                                                                                                                                                                      |

---

### SEC-04 — User Cannot Access Admin API

| Field               | Content                                                                                                                                                                                                                     |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `SEC-04`                                                                                                                                                                                                                    |
| **Test Name**       | Verify Admin API authorization                                                                                                                                                                                              |
| **Objective**       | Verify that an authenticated User without the Admin role is still denied access.                                                                                                                                            |
| **Preconditions**   | Valid User Token, Valid Admin Token, and Admin API are available.                                                                                                                                                           |
| **Test Steps**      | 1. Call the Admin API using Valid User Token.<br>2. Try available `GET`, `POST`, `PUT`, `PATCH`, or `DELETE` methods.<br>3. Call the same function using Valid Admin Token.<br>4. Check data before and after the requests. |
| **Expected Result** | Regular User receives `403 Forbidden`; valid Admin requests are processed according to business rules; the User cannot read or modify Admin data; authorization is enforced server-side.                                    |
| **Actual Result**   | Record endpoint, method, caller role, status, and data changes.                                                                                                                                                             |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                               |
| **Evidence**        | API responses, before/after data, and authorization logs.                                                                                                                                                                   |

---

### SEC-05 — Permissions Cannot Be Changed Through the `role` Field

| Field               | Content                                                                                                                                                                                                                                                                                                                                                                            |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `SEC-05`                                                                                                                                                                                                                                                                                                                                                                           |
| **Test Name**       | Prevent mass assignment and role escalation                                                                                                                                                                                                                                                                                                                                        |
| **Objective**       | Verify that a User cannot elevate privileges through request data.                                                                                                                                                                                                                                                                                                                 |
| **Preconditions**   | A registration, profile update, or User update API exists.                                                                                                                                                                                                                                                                                                                         |
| **Test Steps**      | 1. Send a valid request as a regular User.<br>2. Add `"role": "ADMIN"` to the request body.<br>3. Try adding `"is_admin": true`, `"status": "ACTIVE"`, or similar authorization fields.<br>4. Try supplying a role through query parameters or custom headers.<br>5. Log in again and inspect permissions through the database or profile API.<br>6. Attempt to call an Admin API. |
| **Expected Result** | The server rejects unauthorized fields with `400/422`, or ignores them according to the API contract; the database role remains unchanged; a newly issued JWT does not contain Admin privileges; the User still receives `403` when calling an Admin API.                                                                                                                          |
| **Actual Result**   | Record masked payload, response, role before/after, and Admin API result.                                                                                                                                                                                                                                                                                                          |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                                                                                      |
| **Evidence**        | Request body, API response, and User record before/after.                                                                                                                                                                                                                                                                                                                          |

> Identity and role must come from verified JWT data and trusted server-side data, not from the request body.

---

### SEC-06 — User Cannot Access Another User's Resource by Changing the ID

| Field               | Content                                                                                                                                                                                                                                                                                                                   |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `SEC-06`                                                                                                                                                                                                                                                                                                                  |
| **Test Name**       | Verify Object-Level Authorization                                                                                                                                                                                                                                                                                         |
| **Objective**       | Verify that User A cannot read, modify, or delete User B's private resources.                                                                                                                                                                                                                                             |
| **Preconditions**   | User A and User B own separate resources.                                                                                                                                                                                                                                                                                 |
| **Test Steps**      | 1. Log in as User A.<br>2. Perform a valid operation on Resource A.<br>3. Replace the resource ID with Resource B's ID.<br>4. Try `GET`, `PUT/PATCH`, and `DELETE` if supported.<br>5. Try changing `userId`, `ownerId`, `itemId`, or `sessionId` in the path, query, and body.<br>6. Check Resource B after the request. |
| **Expected Result** | Operations on Resource A are handled according to business rules; access to Resource B is rejected with `403 Forbidden` or `404 Not Found` according to the contract; Resource B is not read, modified, or deleted; hard-to-guess UUIDs are not treated as an authorization mechanism.                                    |
| **Actual Result**   | Record resource type, caller, owner, operation, status, and data state.                                                                                                                                                                                                                                                   |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                             |
| **Evidence**        | Responses and before/after data for both resources.                                                                                                                                                                                                                                                                       |

---

### SEC-07 — S3 Bucket Does Not Allow Public Access

| Field               | Content                                                                                                                                                                                                                                                                                                                                                                             |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `SEC-07`                                                                                                                                                                                                                                                                                                                                                                            |
| **Test Name**       | Block public access to S3                                                                                                                                                                                                                                                                                                                                                           |
| **Objective**       | Verify that Internet users cannot directly read or list the bucket.                                                                                                                                                                                                                                                                                                                 |
| **Preconditions**   | The bucket and at least one test object exist.                                                                                                                                                                                                                                                                                                                                      |
| **Test Steps**      | 1. Open the direct S3 object URL in an incognito browser.<br>2. Send an unsigned request to the object.<br>3. Attempt to access the bucket root or perform unauthenticated `ListBucket`.<br>4. Check all four Block Public Access settings.<br>5. Check bucket policy and object ACL.<br>6. Access the object through CloudFront or a valid presigned URL if allowed by the design. |
| **Expected Result** | Direct S3 access is rejected with `403 AccessDenied`; bucket listing is not allowed; Block Public Access is enabled; policy/ACL does not grant permissions to public principals; CloudFront OAC or a valid presigned URL still functions as designed.                                                                                                                               |
| **Actual Result**   | Record masked bucket, request type, status, and public access settings.                                                                                                                                                                                                                                                                                                             |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                                                                                       |
| **Evidence**        | `403` response, Block Public Access settings, bucket policy, and ACL.                                                                                                                                                                                                                                                                                                               |

---

### SEC-08 — CORS Allows Only Trusted Origins

| Field               | Content                                                                                                                                                                                                                                                                                                                                                        |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `SEC-08`                                                                                                                                                                                                                                                                                                                                                       |
| **Test Name**       | Verify CORS allowlist                                                                                                                                                                                                                                                                                                                                          |
| **Objective**       | Verify that the browser allows access only from configured frontend origins.                                                                                                                                                                                                                                                                                   |
| **Preconditions**   | Trusted Origin, Untrusted Origin, and CORS configuration exist.                                                                                                                                                                                                                                                                                                |
| **Test Steps**      | 1. Send a preflight `OPTIONS` request from Trusted Origin.<br>2. Check allow-origin, allow-methods, allow-headers, and credentials.<br>3. Send the actual request from Trusted Origin.<br>4. Repeat with Untrusted Origin.<br>5. Try similar-looking origins, such as fake subdomains or domains with added suffixes.<br>6. Verify behavior in a real browser. |
| **Expected Result** | Trusted Origin receives the correct CORS headers; Untrusted Origin does not receive `Access-Control-Allow-Origin`; the server does not arbitrarily reflect the `Origin` value; wildcard origin is not used with credentialed requests; only required methods and headers are allowed.                                                                          |
| **Actual Result**   | Record origin, requested method, CORS headers, and browser result.                                                                                                                                                                                                                                                                                             |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                                                                  |
| **Evidence**        | Preflight request/response and browser console.                                                                                                                                                                                                                                                                                                                |

> CORS is not an authentication or authorization mechanism. Requests outside the browser must still be controlled using JWTs, presigned URLs, IAM, or resource policies.

---

### SEC-09 — Lambda Is Denied Access Outside Its IAM Policy

| Field               | Content                                                                                                                                                                                                                                                                                                                                     |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `SEC-09`                                                                                                                                                                                                                                                                                                                                    |
| **Test Name**       | Verify least-privilege IAM for Lambda                                                                                                                                                                                                                                                                                                       |
| **Objective**       | Verify that Lambda can access only authorized resources and actions.                                                                                                                                                                                                                                                                        |
| **Preconditions**   | A test Lambda role, Allowed AWS Resource, and Restricted AWS Resource are available.                                                                                                                                                                                                                                                        |
| **Test Steps**      | 1. Have the test Lambda perform an allowed action on Allowed Resource.<br>2. Confirm success.<br>3. Have Lambda perform the same action on Restricted Resource.<br>4. Try an unauthorized action on Allowed Resource.<br>5. Check the execution role, permission boundary, resource policy, and CloudTrail.<br>6. Check data after testing. |
| **Expected Result** | Authorized actions on the correct resource succeed; actions or resources outside the policy are rejected with `AccessDenied`; broad permissions such as `s3:*`, `secretsmanager:*`, or resource `*` are not used unless necessary; Lambda handles the error safely.                                                                         |
| **Actual Result**   | Record role, action, masked resource, authorization decision, and AWS error code.                                                                                                                                                                                                                                                           |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                                               |
| **Evidence**        | IAM policy, Lambda logs, CloudTrail event, and resource state.                                                                                                                                                                                                                                                                              |

> Do not modify a production Lambda to create unauthorized behavior. Use a controlled test function or test role.

---

### SEC-10 — WebSocket Messages Are Authenticated and Validated

| Field               | Content                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `SEC-10`                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| **Test Name**       | Verify WebSocket connection and message security                                                                                                                                                                                                                                                                                                                                                                                          |
| **Objective**       | Verify that only valid clients can connect and invalid messages are not processed.                                                                                                                                                                                                                                                                                                                                                        |
| **Preconditions**   | WebSocket API, `$connect` authorization, and message handlers have been implemented.                                                                                                                                                                                                                                                                                                                                                      |
| **Test Steps**      | 1. Connect without credentials.<br>2. Connect using a forged or expired token.<br>3. Connect using Valid User Token.<br>4. Send valid JSON.<br>5. Send a non-JSON string.<br>6. Send JSON without `action` or required fields.<br>7. Send an unsupported action.<br>8. Send incorrect data types, extra fields, or an oversized message.<br>9. Try sending unauthorized `userId`, `role`, or `itemId` values.<br>10. Check data and logs. |
| **Expected Result** | Unauthenticated connections are rejected; valid connections succeed; identity is derived from verified authentication; invalid messages return controlled errors or cause the connection to close according to the contract; no unintended data changes occur; clients cannot impersonate another User through message fields.                                                                                                            |
| **Actual Result**   | Record connection/message type, response event, close code, and processing result.                                                                                                                                                                                                                                                                                                                                                        |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                                                                                                                                             |
| **Evidence**        | WebSocket client output, API Gateway access logs, and Lambda logs.                                                                                                                                                                                                                                                                                                                                                                        |

---

### SEC-11 — Logs Do Not Contain Sensitive Data

| Field               | Content                                                                                                                                                                                                                                                                                                                                                                                  |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `SEC-11`                                                                                                                                                                                                                                                                                                                                                                                 |
| **Test Name**       | Check for secret leakage in logs                                                                                                                                                                                                                                                                                                                                                         |
| **Objective**       | Verify that logs do not contain passwords, tokens, secret keys, or presigned signatures.                                                                                                                                                                                                                                                                                                 |
| **Preconditions**   | CloudWatch Logs are enabled and the tester can read them.                                                                                                                                                                                                                                                                                                                                |
| **Test Steps**      | 1. Perform registration, login, and token refresh operations.<br>2. Call a protected API.<br>3. Generate a presigned upload if available.<br>4. Trigger controlled authentication, S3, and database errors.<br>5. Search logs for sensitive strings.<br>6. Check API Gateway access logs, Lambda logs, and application logs.<br>7. Check log retention and log-group access permissions. |
| **Expected Result** | Logs do not contain passwords, access tokens, refresh tokens, cookies, AWS access keys, secret keys, database passwords, JWT secrets, or complete presigned URLs; only required identifiers are logged; sensitive values are masked; log-reading permissions are restricted.                                                                                                             |
| **Actual Result**   | Record log group, time range, search keywords, and number of matches without copying secrets into the report.                                                                                                                                                                                                                                                                            |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                                                                                            |
| **Evidence**        | Redacted log queries and log-group configuration.                                                                                                                                                                                                                                                                                                                                        |

At minimum, search for strings such as:

```text id="hz7ztc"
password
passwd
Authorization
Bearer
access_token
refresh_token
id_token
secret
secret_key
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
X-Amz-Signature
X-Amz-Credential
Cookie
Set-Cookie
```

Do not include real secret values in queries or testing documentation.

---

### SEC-12 — Error Messages Do Not Reveal Internal System Structure

| Field               | Content                                                                                                                                                                                                                                                                                                         |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `SEC-12`                                                                                                                                                                                                                                                                                                        |
| **Test Name**       | Check information disclosure in error responses                                                                                                                                                                                                                                                                 |
| **Objective**       | Verify that client-facing errors do not expose internal information.                                                                                                                                                                                                                                            |
| **Preconditions**   | Validation, authentication, authorization, not-found, and controlled internal errors can be generated in the testing environment.                                                                                                                                                                               |
| **Test Steps**      | 1. Send malformed JSON.<br>2. Omit required fields.<br>3. Send a non-existent ID.<br>4. Send an invalid token.<br>5. Trigger a controlled database/S3 error.<br>6. Call a non-existent route.<br>7. Check response body, headers, and logs.                                                                     |
| **Expected Result** | The client receives only status, safe error code/message, and correlation ID; the response does not contain stack traces, file paths, table names, SQL, database hosts, internal bucket names, Lambda ARNs, secret names, source code, or dependency versions; technical details remain only in protected logs. |
| **Actual Result**   | Record error type, status, error code, and returned fields.                                                                                                                                                                                                                                                     |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                   |
| **Evidence**        | Redacted error responses and corresponding logs identified by correlation ID.                                                                                                                                                                                                                                   |

A safe error response may look like:

```json id="azc662"
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred.",
    "requestId": "req-123456"
  }
}
```

The following must not be returned:

```json id="zk0gsl"
{
  "error": "OperationalError",
  "sql": "SELECT * FROM users...",
  "databaseHost": "auction-db.xxxx.ap-southeast-1.rds.amazonaws.com",
  "file": "/var/task/app/infrastructure/database.py",
  "stackTrace": "..."
}
```

---

### Security Testing Matrix

| ID       | Test Content                   | Main Component           | Expected Result                 |
| -------- | ------------------------------ | ------------------------ | ------------------------------- |
| `SEC-01` | Request without token          | API Gateway/Lambda       | `401 Unauthorized`              |
| `SEC-02` | Forged token                   | JWT verifier             | `401 Unauthorized`              |
| `SEC-03` | Expired token                  | JWT verifier             | `401 Unauthorized`              |
| `SEC-04` | User calls Admin API           | Authorization guard      | `403 Forbidden`                 |
| `SEC-05` | Modify role in request         | Validation/authorization | Role remains unchanged          |
| `SEC-06` | Change resource ID             | Object authorization     | `403` or `404`                  |
| `SEC-07` | Public S3 access               | S3 policy/BPA            | `403 AccessDenied`              |
| `SEC-08` | Untrusted origin               | CORS                     | No CORS permission              |
| `SEC-09` | Lambda exceeds IAM Policy      | IAM                      | `AccessDenied`                  |
| `SEC-10` | Invalid WebSocket              | WebSocket API/Lambda     | Connection/message rejected     |
| `SEC-11` | Sensitive data in logs         | CloudWatch/API Gateway   | No secrets found                |
| `SEC-12` | Internal information in errors | Error handler            | No internal structure disclosed |

---

### Result Summary Table

| ID       | Test Content                                   | Actual Result | Status     |
| -------- | ---------------------------------------------- | ------------- | ---------- |
| `SEC-01` | API rejects request without token              | Not tested    | Not tested |
| `SEC-02` | Forged token is rejected                       | Not tested    | Not tested |
| `SEC-03` | Expired token is rejected                      | Not tested    | Not tested |
| `SEC-04` | User cannot access Admin API                   | Not tested    | Not tested |
| `SEC-05` | Role cannot be changed through request         | Not tested    | Not tested |
| `SEC-06` | Cannot manipulate another User's resource      | Not tested    | Not tested |
| `SEC-07` | S3 does not allow public access                | Not tested    | Not tested |
| `SEC-08` | CORS allows only trusted origins               | Not tested    | Not tested |
| `SEC-09` | Lambda is restricted by IAM Policy             | Not tested    | Not tested |
| `SEC-10` | WebSocket authenticates and validates messages | Not tested    | Not tested |
| `SEC-11` | Logs do not contain sensitive data             | Not tested    | Not tested |
| `SEC-12` | Errors do not disclose internal information    | Not tested    | Not tested |

---

### HTTP Status Code Requirements

| Scenario                                  | Expected Status                                           |
| ----------------------------------------- | --------------------------------------------------------- |
| Missing access token                      | `401 Unauthorized`                                        |
| Token has invalid signature               | `401 Unauthorized`                                        |
| Token expired                             | `401 Unauthorized`                                        |
| Token missing required claim              | `401 Unauthorized`                                        |
| Authenticated but insufficient permission | `403 Forbidden`                                           |
| User does not own resource                | `403 Forbidden` or `404 Not Found` according to contract  |
| Invalid request body                      | `400 Bad Request` or `422 Unprocessable Entity`           |
| Resource does not exist                   | `404 Not Found`                                           |
| Request too large                         | `413 Payload Too Large`                                   |
| Unsupported WebSocket message/action      | Error event or close code according to WebSocket contract |
| Internal error                            | `500 Internal Server Error` with a safe message           |

Do not use `500 Internal Server Error` for all authentication or validation errors.

---

### JWT Verification Requirements

The JWT verifier must verify at minimum:

```text id="5a52fs"
signature
allowed algorithm
expiration
required claims
subject/user ID
issuer, if used by the system
audience, if used by the system
token type, if access and refresh tokens are distinguished
```

The following values from the request body must not be trusted:

```text id="9hpzlv"
userId
ownerId
email
role
isAdmin
status
```

Access tokens and refresh tokens must not be used interchangeably.

---

### Authorization Verification Requirements

Every API that accesses a resource by ID must verify:

* The User is authenticated.
* The User remains in `ACTIVE` status.
* The User has the appropriate role.
* The User has permission to perform the requested action.
* The User owns the resource or has been granted access to it.
* The resource belongs to the correct related session/item.
* Authorization is rechecked server-side on every request.
* Authorization does not depend on the frontend hiding buttons.
* Hard-to-guess UUIDs are not used as a substitute for authorization.

---

### WebSocket Verification Requirements

WebSocket security must verify:

* Authentication at `$connect`.
* Invalid or expired tokens are rejected.
* Connections are associated with verified identity.
* Users cannot self-declare `userId` or `role`.
* Users can join only authorized rooms.
* Every message must be valid JSON.
* `action` must belong to an allowlist.
* Required fields must exist and use the correct data types.
* Message size limits are enforced.
* Message contents are not executed as code or commands.
* Rate limiting or throttling is configured where necessary.
* Connection ID is not treated as the sole proof of identity.
* Invalid messages do not cause Lambda to crash or return a stack trace.

---

### Log Verification Requirements

Logs may contain:

```text id="nbj7aw"
requestId
correlationId
verified userId
action
resourceId
route
HTTP method
status code
authorization decision
AWS error code
Lambda request ID
processing duration
```

Logs must not contain:

```text id="sxufxx"
password
password hash
access token
refresh token
ID token
session cookie
JWT secret
AWS access key
AWS secret key
database password
Secrets Manager secret value
complete presigned URL
X-Amz-Signature
complete Authorization header
Base64 image data
```

If token identification is required for investigation, only a non-reversible fingerprint/hash or a masked portion should be logged, never the complete token.

---

### Testing Evidence

Evidence should include:

* `401` response when a token is missing.
* `401` response for a forged token.
* `401` response for an expired token.
* `403` response when a User calls an Admin API.
* User role before and after a request containing `"role": "ADMIN"`.
* User B's data before and after User A changes the resource ID.
* `403` response when accessing S3 directly.
* S3 Block Public Access configuration.
* Preflight responses for trusted and untrusted origins.
* Lambda IAM Policy.
* CloudTrail or CloudWatch `AccessDenied` record.
* Result of a WebSocket connection without a token.
* Result of sending an invalidly formatted WebSocket message.
* CloudWatch Logs query for sensitive data.
* Error response showing no stack trace or infrastructure details.

Suggested figure names:

```text id="22j97z"
Figure 5.5.8.1: API rejects a request without a token
Figure 5.5.8.2: Forged JWT is rejected
Figure 5.5.8.3: Expired JWT is rejected
Figure 5.5.8.4: User is rejected when calling an Admin API
Figure 5.5.8.5: Role field in the request does not change permissions
Figure 5.5.8.6: User A cannot modify User B's resource
Figure 5.5.8.7: S3 object cannot be accessed publicly
Figure 5.5.8.8: Untrusted origin is not allowed by CORS
Figure 5.5.8.9: Lambda receives AccessDenied outside IAM scope
Figure 5.5.8.10: WebSocket rejects an invalid message
Figure 5.5.8.11: Logs do not contain sensitive data
Figure 5.5.8.12: Error response does not disclose internal structure
```

---

### Evaluation Criteria

A test case may only be marked as `PASS` when:

* The API rejects requests without a token.
* Forged, unsupported-algorithm, and expired JWTs are rejected.
* A regular User cannot perform Admin functions.
* `role`, `isAdmin`, or `userId` fields in requests do not change identity or permissions.
* A User cannot read, modify, or delete another User's resources.
* S3 buckets and objects cannot be accessed publicly.
* CORS responds only to trusted origins.
* Lambda can access only actions and resources allowed by its IAM Policy.
* WebSocket authenticates connections and validates every message.
* Logs contain no passwords, tokens, credentials, or secrets.
* Error responses do not expose internal system structure.
* Rejected requests do not cause unintended data changes.

A test case must be marked as `FAIL` when:

* A protected API returns data without a token.
* A token with a modified payload is still accepted.
* An expired token can still call an API.
* A regular User can perform Admin functions.
* A User can elevate privileges using the request body.
* Changing a resource ID grants access to another User's data.
* An S3 object can be read publicly outside the intended design.
* CORS allows arbitrary origins or reflects uncontrolled origins.
* Lambda can access resources outside its IAM Policy.
* WebSocket permits unauthenticated connections or processes invalid messages.
* Logs contain passwords, tokens, AWS credentials, or secret values.
* Error responses expose stack traces, SQL, file paths, hosts, or infrastructure details.
* A rejected request still modifies data.

A test case is marked as `BLOCKED` when:

* The API or Lambda has not been deployed.
* JWT verifier or authorizer has not been implemented.
* User/Admin test accounts are unavailable.
* Separate data for two different Users is unavailable.
* Admin API does not exist.
* S3 bucket has not been created.
* CORS has not been configured.
* Lambda execution role does not exist.
* WebSocket API or `$connect` authorization has not been implemented.
* CloudWatch Logs are disabled or cannot be accessed.
* No safe environment exists for negative IAM testing.
* The security contract or HTTP error contract has not been defined.
