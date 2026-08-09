---
title: "Authentication and Authorization Testing"
date: 2026-08-03
weight: 3
chapter: false
pre: " <b> 5.5.3. </b> "
---

### Authentication and Authorization Testing

#### Test Objectives

This section verifies authentication and authorization of the Live Auction system at the system level, including:

- User registration, confirmation and login using Amazon Cognito.
- Synchronization of user information from Cognito to Amazon DynamoDB.
- Operation of the Post Confirmation Lambda trigger.
- JWT validation through API Gateway Authorizer.
- Authorization separation between `User` and `Admin` accounts.
- Use of verified Cognito claims such as `sub` and `cognito:groups`.
- Preventing identity or role spoofing via request bodies.
- Ensuring Lambdas access AWS resources only within their IAM policy scope.

Test cases are numbered `AUTH-01` to `AUTH-13`. Post Confirmation and idempotency tests are grouped in `AUTH-03`. Verified claims are tested in `AUTH-10`. Rejecting `userId` and `role` from request bodies is tested in `AUTH-13`.

#### Common Prerequisites

Before executing tests, ensure the following:

- An Amazon Cognito User Pool is deployed.
- A User Pool App Client is configured for the frontends.
- Post Confirmation Lambda is linked to the Cognito User Pool.
- REST APIs are protected by API Gateway Authorizer.
- Lambda functions have appropriate IAM execution roles.
- DynamoDB has a table for user information.
- User Frontend and Admin Frontend are operational.
- At least one regular API for Users exists.
- At least one Admin-only API exists.
- CloudWatch Logs enabled for relevant Lambdas.
- Test User and Admin accounts available.

#### Test Data

| Data               | Description                                                         |
|--------------------|---------------------------------------------------------------------|
| New User           | Email not present in Cognito User Pool                              |
| Confirmed User     | Account with `CONFIRMED` status                                     |
| Unconfirmed User   | Account not yet confirmed                                           |
| Admin              | Account in Cognito Group `Admin`                                    |
| Correct password   | Valid password for test account                                     |
| Wrong password     | Incorrect password                                                  |
| Valid token        | JWT issued by the Cognito User Pool and not expired                 |
| Invalid token      | Token tampered, bad signature or malformed                          |
| Expired token      | JWT whose `exp` is before current time                              |
| Forged user ID     | Another user's ID supplied in request body                          |
| Forged role        | `Admin` value submitted by a regular User in request body           |

Do not include real emails, passwords, Access Tokens, ID Tokens or Refresh Tokens in test documentation.

---

#### AUTH-01 — Successful User Registration

| Field                  | Content                                                                                                                           |
|------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Test ID**            | `AUTH-01`                                                                                                                         |
| **Test name**          | Successful User registration                                                                                                      |
| **Objective**          | Verify a new user can register via the User Frontend and Amazon Cognito.                                                         |
| **Prerequisites**      | Test email does not exist in Cognito; User Frontend can reach Cognito.                                                            |
| **Steps**              | 1. Open registration page.<br>2. Enter valid email and password.<br>3. Fill required profile fields.<br>4. Submit registration.<br>5. Check account state in Cognito User Pool. |
| **Input data**         | New email, password that meets policy, valid profile fields.                                                                     |
| **Expected result**    | Registration succeeds; account created in Cognito; frontend prompts for confirmation code; account is not allowed to sign in before confirmation. |
| **Actual result**      | Fill after execution.                                                                                                             |
| **Status**             | `PASS`, `FAIL` or `BLOCKED`.                                                                                                      |
| **Evidence**           | Registration screenshot and sanitized Cognito User Pool entry.                                                                   |

---

#### AUTH-02 — Successful Account Confirmation

| Field                  | Content                                                                                                  |
|------------------------|----------------------------------------------------------------------------------------------------------|
| **Test ID**            | `AUTH-02`                                                                                                 |
| **Test name**          | Confirm account with a valid code                                                                         |
| **Objective**          | Verify a newly registered account can be confirmed using a valid code.                                     |
| **Prerequisites**      | Account registered but unconfirmed; confirmation code received.                                            |
| **Steps**              | 1. Open confirmation page.<br>2. Enter test email and valid code.<br>3. Submit confirmation.<br>4. Check Cognito status. |
| **Input data**         | Test email and valid confirmation code.                                                                   |
| **Expected result**    | Cognito confirms the account; account status becomes `CONFIRMED`; user can proceed to sign in.           |
| **Actual result**      | Fill after execution.                                                                                      |
| **Status**             | `PASS`, `FAIL` or `BLOCKED`.                                                                              |
| **Evidence**           | Confirmation UI screenshot and `CONFIRMED` status in Cognito.                                              |

---

#### AUTH-03 — Post Confirmation Lambda Trigger and Idempotency

| Field                  | Content                                                                                                                                              |
|------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Test ID**            | `AUTH-03`                                                                                                                                           |
| **Test name**          | Trigger Post Confirmation and verify idempotency                                                                                                    |
| **Objective**          | Verify Cognito triggers the Post Confirmation Lambda and repeated processing does not create duplicate user records.                                |
| **Prerequisites**      | Post Confirmation Lambda configured and has permission to write to the Users table in DynamoDB.                                                    |
| **Steps**              | 1. Register and confirm a new user.<br>2. Inspect CloudWatch Logs of Post Confirmation Lambda.<br>3. Check the user record in DynamoDB.<br>4. Replay the same event or re-run processing with the same `sub`.<br>5. Check record count after second run. |
| **Input data**         | Post Confirmation event containing the same Cognito `sub`.                                                                                         |
| **Expected result**    | Lambda runs on confirmation; DynamoDB has exactly one record for the Cognito `sub`; repeated processing does not create a second record or corrupt data. |
| **Actual result**      | Record number of Lambda invocations and observed record count.                                                                                      |
| **Status**             | `PASS`, `FAIL` or `BLOCKED`.                                                                                                                        |
| **Evidence**           | CloudWatch Logs and sanitized DynamoDB record.                                                                                                      |

> Cognito may retry Post Confirmation on errors. The Lambda must implement idempotent writes based on Cognito `sub`, not email.

---

#### AUTH-04 — Sign in with valid credentials

| Field                  | Content                                                                                                  |
|------------------------|----------------------------------------------------------------------------------------------------------|
| **Test ID**            | `AUTH-04`                                                                                                 |
| **Test name**          | Sign in with a confirmed account                                                                         |
| **Objective**          | Verify a confirmed user can sign in with valid credentials.                                              |
| **Prerequisites**      | Account is `CONFIRMED` and not disabled.                                                                 |
| **Steps**              | 1. Open sign-in page.<br>2. Enter correct email and password.<br>3. Click sign in.<br>4. Call a regular API after sign-in. |
| **Input data**         | Valid email and password.                                                                               |
| **Expected result**    | Cognito authenticates; frontend transitions to the application; token can be used to call an API that returns HTTP `200`. |
| **Actual result**      | Fill after execution.                                                                                      |
| **Status**             | `PASS`, `FAIL` or `BLOCKED`.                                                                              |
| **Evidence**           | Signed-in UI screenshot and HTTP `200` response, tokens redacted.                                        |

---

#### AUTH-05 — Sign in with wrong password

| Field                  | Content                                                                                                                      |
|------------------------|------------------------------------------------------------------------------------------------------------------------------|
| **Test ID**            | `AUTH-05`                                                                                                                     |
| **Test name**          | Deny sign-in when the password is incorrect                                                                                   |
| **Objective**          | Verify Cognito does not issue tokens for incorrect passwords.                                                                |
| **Prerequisites**      | User account is confirmed.                                                                                                    |
| **Steps**              | 1. Open sign-in page.<br>2. Enter correct email and wrong password.<br>3. Submit and observe response.                         |
| **Input data**         | Valid email and incorrect password.                                                                                           |
| **Expected result**    | Sign-in is denied; no token issued; frontend shows an appropriate message; API or frontend normalizes response to `400` or `401`. |
| **Actual result**      | Record observed HTTP status and message.                                                                                      |
| **Status**             | `PASS`, `FAIL` or `BLOCKED`.                                                                                                 |
| **Evidence**           | Screenshot of failure message or sanitized response.                                                                         |

---

#### AUTH-06 — Sign in with unconfirmed account

| Field                  | Content                                                                                         |
|------------------------|-------------------------------------------------------------------------------------------------|
| **Test ID**            | `AUTH-06`                                                                                        |
| **Test name**          | Deny sign-in for unconfirmed account                                                            |
| **Objective**          | Verify unconfirmed accounts cannot sign in.                                                     |
| **Prerequisites**      | Account has been registered but not confirmed.                                                  |
| **Steps**              | 1. Open sign-in page.<br>2. Submit credentials for the unconfirmed account.<br>3. Observe response. |
| **Input data**         | Correct email and password for unconfirmed account.                                            |
| **Expected result**    | Cognito denies authentication; no token issued; frontend indicates account confirmation is required. |
| **Actual result**      | Record HTTP status or message observed.                                                         |
| **Status**             | `PASS`, `FAIL` or `BLOCKED`.                                                                    |
| **Evidence**           | Cognito account status and screenshot.                                                          |

---

#### AUTH-07 — API call without token

| Field                  | Content                                                                                                             |
|------------------------|---------------------------------------------------------------------------------------------------------------------|
| **Test ID**            | `AUTH-07`                                                                                                            |
| **Test name**          | Reject requests without Authorization token                                                                         |
| **Objective**          | Verify API Gateway Authorizer protects endpoints requiring authentication.                                          |
| **Prerequisites**      | Endpoint under test is linked to an Authorizer.                                                                     |
| **Steps**              | 1. Prepare a request to a protected endpoint without `Authorization` header.<br>2. Send request.<br>3. Record status and response body. |
| **Input data**         | Request without Bearer token.                                                                                        |
| **Expected result**    | API Gateway rejects with HTTP `401`; backend Lambda not executed; no data modified.                                  |
| **Actual result**      | Record observed HTTP status and response.                                                                           |
| **Status**             | `PASS`, `FAIL` or `BLOCKED`.                                                                                        |
| **Evidence**           | HTTP `401` response and CloudWatch evidence showing no backend execution.                                          |

---

#### AUTH-08 — API call with invalid token

| Field                  | Content                                                                                                       |
|------------------------|---------------------------------------------------------------------------------------------------------------|
| **Test ID**            | `AUTH-08`                                                                                                      |
| **Test name**          | Reject invalid JWT                                                                                             |
| **Objective**          | Verify the Authorizer checks token format, signature, issuer and audience/client.                               |
| **Prerequisites**      | Endpoint protected by Authorizer.                                                                              |
| **Steps**              | 1. Create a tampered or incorrectly signed token.<br>2. Send request with `Authorization: Bearer <token>`.<br>3. Record response. |
| **Input data**         | Forged JWT or token not issued by the Cognito User Pool.                                                       |
| **Expected result**    | Authorizer rejects token; API returns HTTP `401`; backend not executed; system does not trust claims in forged token. |
| **Actual result**      | Record HTTP status and message.                                                                                |
| **Status**             | `PASS`, `FAIL` or `BLOCKED`.                                                                                   |
| **Evidence**           | HTTP `401` response; token omitted from evidence.                                                              |

---

#### AUTH-09 — API call with expired token

| Field                  | Content                                                                                      |
|------------------------|----------------------------------------------------------------------------------------------|
| **Test ID**            | `AUTH-09`                                                                                     |
| **Test name**          | Reject expired JWT                                                                            |
| **Objective**          | Verify the Authorizer checks the `exp` claim before allowing API access.                       |
| **Prerequisites**      | A previously valid token that has since expired.                                              |
| **Steps**              | 1. Prepare an expired token.<br>2. Send request to protected API.<br>3. Observe response and backend activation. |
| **Input data**         | Bearer token with `exp` earlier than current time.                                           |
| **Expected result**    | Authorizer rejects the request with HTTP `401`; backend not executed; data unchanged.          |
| **Actual result**      | Record HTTP status and observed result.                                                       |
| **Status**             | `PASS`, `FAIL` or `BLOCKED`.                                                                   |
| **Evidence**           | HTTP `401` response and sanitized logs.                                                        |

---

#### AUTH-10 — User calls regular API using verified identity

| Field                  | Content                                                                                                                           |
|------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Test ID**            | `AUTH-10`                                                                                                                         |
| **Test name**          | User accesses regular API using Cognito-verified claims                                                                         |
| **Objective**          | Verify backend uses the verified `sub` claim from the Authorizer as the current identity.                                         |
| **Prerequisites**      | User signed in; token valid; endpoint allows regular User access.                                                                 |
| **Steps**              | 1. Sign in as User.<br>2. Send a request to a regular API with a valid token.<br>3. Observe response.<br>4. Check CloudWatch Logs and data writes and match owner ID to `sub`. |
| **Input data**         | Valid user token and a valid business request.                                                                                   |
| **Expected result**    | API returns HTTP `200`; backend uses `sub` claim as identity; resources are accessed or created for the correct user; client must not declare identity. |
| **Actual result**      | Record HTTP status, returned data and identity matching results.                                                               |
| **Status**             | `PASS`, `FAIL` or `BLOCKED`.                                                                                                    |
| **Evidence**           | Sanitized HTTP response, CloudWatch Logs and related DynamoDB records.                                                          |

---

#### AUTH-11 — User denied Admin API access

| Field                  | Content                                                                                                                             |
|------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| **Test ID**            | `AUTH-11`                                                                                                                            |
| **Test name**          | Deny regular User access to Admin API                                                                                                 |
| **Objective**          | Verify an authenticated User who is not in Admin group cannot perform Admin operations.                                               |
| **Prerequisites**      | User signed in; User not in Cognito Group `Admin`; endpoint remains admin-only.                                                       |
| **Steps**              | 1. Sign in as User.<br>2. Send a valid request to Admin API.<br>3. Observe response and data state.                                   |
| **Input data**         | Valid User token and properly formatted admin request.                                                                                |
| **Expected result**    | API returns HTTP `403`; admin operation not performed; data unchanged.                                                                  |
| **Actual result**      | Record HTTP status and data state.                                                                                                    |
| **Status**             | `PASS`, `FAIL` or `BLOCKED`.                                                                                                          |
| **Evidence**           | HTTP `403` response, Cognito group status and DynamoDB unchanged.                                                                     |

---

#### AUTH-12 — Admin calls Admin API successfully

| Field                  | Content                                                                                                                  |
|------------------------|--------------------------------------------------------------------------------------------------------------------------|
| **Test ID**            | `AUTH-12`                                                                                                                 |
| **Test name**          | Admin accesses Admin API                                                                                                   |
| **Objective**          | Verify a user in Cognito Group `Admin` can perform administrative actions.                                                  |
| **Prerequisites**      | Admin account confirmed and in `Admin` group; admin endpoint deployed.                                                      |
| **Steps**              | 1. Sign in as Admin.<br>2. Send valid request to Admin API.<br>3. Observe response and data updates.                         |
| **Input data**         | Valid Admin token containing `cognito:groups` with `Admin`.                                                                 |
| **Expected result**    | Authorizer validates token; backend confirms rights from `cognito:groups`; API returns HTTP `200` or other success code; admin action applied. |
| **Actual result**      | Record HTTP status and observed data.                                                                                       |
| **Status**             | `PASS`, `FAIL` or `BLOCKED`.                                                                                               |
| **Evidence**           | Cognito group, successful response, CloudWatch Logs and relevant DynamoDB records.                                           |

---

#### AUTH-13 — Do not trust `userId` or `role` in request body

| Field                  | Content                                                                                                                                            |
|------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| **Test ID**            | `AUTH-13`                                                                                                                                         |
| **Test name**          | Prevent identity and role spoofing via request body                                                                                             |
| **Objective**          | Verify backend uses `sub` and `cognito:groups` from the verified token and does not trust `userId` or `role` supplied by the client.               |
| **Prerequisites**      | Regular User account A; knowledge of User B's ID; endpoint that creates or updates resources per user.                                           |
| **Steps**              | 1. Sign in as User A.<br>2. Send a request including `userId` of User B.<br>3. Check ownership of created or returned resource.<br>4. Send request with `role: Admin` in body and attempt Admin API.<br>5. Observe responses and data. |
| **Input data**         | Valid User A token; `userId` of User B; `role: Admin` in request body.                                                                           |
| **Expected result**    | Server ignores or rejects untrusted `userId` and `role`; User A cannot act on behalf of User B; Admin APIs return HTTP `403`; no unauthorized changes in DynamoDB. |
| **Actual result**      | Record HTTP status, stored owner ID and data state.                                                                                            |
| **Status**             | `PASS`, `FAIL` or `BLOCKED`.                                                                                                                    |
| **Evidence**           | Redacted request body, `403` responses, CloudWatch Logs and DynamoDB records proving the actual owner.                                         |

---

#### IAM Related Checks

In addition to the 13 test cases, verify IAM Execution Roles of relevant Lambdas:

- Post Confirmation Lambda should only write to the intended DynamoDB table.
- Lambdas must not have overly permissive DynamoDB access unless required.
- Lambdas should only write logs to configured CloudWatch Log groups.
- API Gateway must be allowed to invoke the intended Lambda functions only.
- Cognito should only invoke the correct Post Confirmation Lambda.
- Do not store AWS Access Key or Secret Access Key in source code or Lambda environment variables.
- Avoid `Action: "*"` and `Resource: "*"` in IAM policies unless justified.

Minimal permission checks can be validated by attempting an action outside the policy and verifying `AccessDenied` is returned, while legitimate flows continue to work.

#### Results Summary Table

| ID       | Test description                          | Expected HTTP status            | Status       |
|----------|-------------------------------------------|--------------------------------:|--------------|
| `AUTH-01`| User registration success                  | `200` or API-specific success   | Not executed |
| `AUTH-02`| Account confirmation success               | `200`                            | Not executed |
| `AUTH-03`| Post Confirmation and idempotency          | Success, no duplicate records    | Not executed |
| `AUTH-04`| Sign in with correct credentials           | `200`                            | Not executed |
| `AUTH-05`| Sign in wrong password                     | `400` or `401`                   | Not executed |
| `AUTH-06`| Sign in unconfirmed account                | `400` or `401`                   | Not executed |
| `AUTH-07`| API without token                          | `401`                            | Not executed |
| `AUTH-08`| API with invalid token                     | `401`                            | Not executed |
| `AUTH-09`| API with expired token                     | `401`                            | Not executed |
| `AUTH-10`| User calls regular API                     | `200`                            | Not executed |
| `AUTH-11`| User calls Admin API                       | `403`                            | Not executed |
| `AUTH-12`| Admin calls Admin API                      | `200` or appropriate success     | Not executed |
| `AUTH-13`| Forged `userId` or `role`                  | `403` or ignore forged input     | Not executed |

> HTTP statuses must follow the project API contract. If the frontend uses Cognito SDK directly, Cognito errors may not map to HTTP statuses; record Cognito errors and how the frontend translates them.

#### Test Evidence

Evidence for authentication and authorization tests includes:

- Registration and confirmation screenshots.
- Login success/failure screenshots.
- HTTP statuses `200`, `400`, `401`, `403`.
- Cognito User Pool account state.
- Cognito Group membership for User and Admin.
- CloudWatch Logs for Post Confirmation and API Lambdas.
- User records in DynamoDB.
- Before/after data for unauthorized action attempts.
- IAM policies with sensitive info removed.

Do not reveal passwords, tokens, keys or secrets in any evidence.

Test cases are `PASS` only when actual results match expected results and evidence exists. If Cognito, Authorizer, Lambda or DynamoDB components are not deployed, related test cases must be marked `BLOCKED`.
