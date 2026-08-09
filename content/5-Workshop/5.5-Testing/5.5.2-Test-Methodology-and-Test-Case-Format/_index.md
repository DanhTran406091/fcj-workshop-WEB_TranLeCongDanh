---
title: "Test Methodology and Test Case Format"
date: 2026-08-03
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
---
### Test Methodology and Test Case Format

#### Testing Method

The team performs system testing using a black-box approach, combined with verification of data and logs on AWS services.

For each test case, the team provides input data and performs actions through:

- User Frontend.
- Admin Frontend.
- REST API.
- WebSocket connections.
- AWS Management Console.
- API testing tools such as Postman or `curl`.
- Load testing tools if available.

Actual results are then compared against expected results to determine the test case status.

The testing procedure follows this sequence:

```text
Prepare environment and test data
1. Verify prerequisites
2. Execute test steps
3. Observe results on frontend or API
4. Check data and logs on AWS
5. Compare with expected results
6. Record status
7. Save test evidence
```

Evaluation is not based solely on frontend display. Depending on the test case, the team also verifies:

- HTTP status and response body of REST APIs.
- Messages sent and received via WebSocket.
- Records created or updated in Amazon DynamoDB.
- Message state in Amazon SQS FIFO.
- CloudWatch Logs of AWS Lambda.
- CloudWatch Metrics for Lambda, API Gateway, DynamoDB and SQS.
- Object metadata or versions in Amazon S3.
- Content distributed through Amazon CloudFront.
- User and group state in Amazon Cognito.

#### Testing Principles

The testing process adheres to the following principles:

1. Each test case focuses on a single behavior or condition.
2. Test cases must define clear prerequisites and input data.
3. Steps must be detailed enough for another team member to reproduce.
4. Expected results must be measurable or observable.
5. Actual results must be recorded based on real system behavior.
6. A test case is marked `PASS` only when actual results match expectations.
7. If results differ, mark `FAIL` and describe observed errors.
8. If testing is not possible due to missing features, resources or dependencies, mark `BLOCKED`.
9. A `PASS` test case must include appropriate evidence.
10. Do not change expected results after execution just to mark a test `PASS`.
11. Data created by prior test cases must not bias subsequent tests.
12. Do not include tokens, passwords or AWS secret keys in reports.

#### Test Case Classification

Test cases are grouped by system components and given a prefix:

| Prefix    | Test group                           | Example       |
|-----------|--------------------------------------|---------------|
| `AUTH`    | Authentication and authorization     | `AUTH-01`     |
| `API`     | REST API and auction management      | `API-01`      |
| `WS`      | WebSocket and real-time updates      | `WS-01`       |
| `BID`     | End-to-end bidding flow              | `BID-01`      |
| `FIFO`    | SQS FIFO ordering and processing     | `FIFO-01`     |
| `DB`      | DynamoDB and data integrity          | `DB-01`       |
| `STORAGE` | Amazon S3 and CloudFront             | `STORAGE-01`  |
| `RECOVERY`| Error handling and recovery          | `RECOVERY-01` |
| `PERF`    | Performance and concurrency          | `PERF-01`     |
| `SEC`     | System security                      | `SEC-01`      |

Test IDs must be unique across the test documentation. The ID structure is:

```text
<PREFIX>-<SEQUENCE>
```

Examples:

```text
AUTH-01
API-03
WS-05
BID-07
FIFO-02
SEC-04
```

#### Test Case Format

Each test case is recorded using the following fields:

| Field                  | Description                                                                                                      |
|------------------------|------------------------------------------------------------------------------------------------------------------|
| **Test ID**            | Unique test case ID, e.g. `AUTH-01`.                                                                            |
| **Test name**          | Short descriptive name of the test case.                                                                         |
| **Objective**          | The function, behavior or condition the test case verifies.                                                      |
| **Prerequisites**      | Accounts, data, configuration and system state required before testing.                                          |
| **Steps**              | Ordered actions to execute the test case.                                                                        |
| **Input data**         | Tokens, accounts, passwords, session IDs, item IDs, bid amounts or request payloads.                             |
| **Expected result**    | The correct system behavior after execution.                                                                     |
| **Actual result**      | Observed result on frontend, API or AWS services.                                                               |
| **Status**             | `PASS`, `FAIL` or `BLOCKED`.                                                                                     |
| **Evidence**           | Screenshots, API responses, WebSocket messages, CloudWatch Logs, Metrics or AWS data.                            |

#### Example Recorded Test Case

##### AUTH-01 — Successful User sign-in

| Field                  | Content                                                                                                                |
|------------------------|------------------------------------------------------------------------------------------------------------------------|
| **Test ID**            | `AUTH-01`                                                                                                              |
| **Test name**          | Sign in with a valid User account                                                                                      |
| **Objective**          | Verify a confirmed user can sign in to the User Frontend via Amazon Cognito.                                           |
| **Prerequisites**      | Test email does not already exist in the Cognito User Pool; User Frontend can reach Cognito.                            |
| **Steps**              | 1. Open User Frontend. 2. Go to sign-in page. 3. Enter valid email and password. 4. Click Sign in. 5. Observe UI result. |
| **Input data**         | Email and password of the test User.                                                                                     |
| **Expected result**    | Cognito authenticates successfully; user is redirected to main page; frontend shows signed-in state.                    |
| **Actual result**      | Fill after executing the test.                                                                                         |
| **Status**             | `PASS`, `FAIL` or `BLOCKED`.                                                                                           |
| **Evidence**           | Screenshot of signed-in UI and sanitized logs.                                                                          |

#### Status Conventions

| Status   | Recording rules                                                                                         |
|----------|--------------------------------------------------------------------------------------------------------|
| `PASS`   | Actual result matches expected result and evidence exists.                                             |
| `FAIL`   | Actual result differs, system error occurs or data is incorrect.                                        |
| `BLOCKED`| Test cannot run due to missing components, configuration, data or dependencies.                         |

When a test case is `FAIL`, additionally record:

- The step where the failure occurred.
- Time of failure.
- Observed error message.
- HTTP status if related to REST API.
- Request ID or correlation ID if available.
- Related Lambda function.
- CloudWatch Log Group or Log Stream.
- Impact to the system.
- Suggested mitigation or bug ticket.

When a test case is `BLOCKED`, the record should include:

- Missing component.
- Dependent feature not implemented.
- Configuration not deployed.
- Access or permission missing.
- Conditions required before retry.

#### Actual Result Recording Rules

The **Actual result** field must describe observed behavior and must not duplicate the **Expected result** if the test was not executed.

Appropriate example:

```text
API returned HTTP 200. Response contains sessionId, itemId,
currentPrice and status = ACTIVE. Corresponding data exists
in the Auctions table in DynamoDB.
```

Inappropriate example:

```text
Works correctly.
```

If a test fails, include specific error details:

```text
API returned HTTP 500 instead of HTTP 400 when bidAmount was missing.
CloudWatch Logs show KeyError in Lambda la-ws-handler.
```

#### Evidence Collection Rules

Evidence must directly relate to the test objective. Depending on the case, the team may use one or more of the following:

| Evidence type            | Usage case                                                              |
|--------------------------|-------------------------------------------------------------------------|
| Frontend screenshot      | Demonstrate user-facing functionality.                                 |
| Admin screenshot         | Demonstrate admin functions and authorization.                          |
| HTTP request/response    | Prove REST API returned expected status and data.                       |
| WebSocket message        | Prove real-time messages were sent and received.                        |
| CloudWatch Logs          | Show Lambda trigger and business logic processing.                      |
| CloudWatch Metrics       | Show request counts, errors, latency or message throughput.             |
| DynamoDB item            | Prove data was created or updated correctly.                             |
| SQS Metrics              | Prove messages were sent, received and deleted from the queue.          |
| DLQ contents             | Show messages that failed and moved to the Dead-letter Queue.           |
| S3 object                | Prove file was uploaded to the correct bucket.                          |
| CloudFront response      | Prove frontend or static content was served successfully.               |

Each image should include a caption and the related test case ID. Avoid using a single image for multiple test cases unless it clearly demonstrates each case.

#### Test Data Management

To enable reproducibility, prepare the following test data before execution:

- Confirmed User accounts.
- Admin accounts in the correct group.
- Regular Users without Admin privileges.
- Auction sessions in `SCHEDULED`, `ACTIVE` and `ENDED` states.
- Items with starting prices and minimum increments.
- Valid and invalid resource IDs.
- Valid and invalid bid amounts.
- Valid and invalid image files and sizes.
- Active WebSocket connections.
- Duplicate messages to test idempotency where applicable.

Test data must be isolated from production. After testing, remove or mark test data if no longer needed.

#### Order of Test Groups

Execute test groups in the following dependency order:

1. Environment and endpoint checks.
2. Authentication and authorization tests.
3. REST API tests.
4. DynamoDB data validation.
5. WebSocket connectivity tests.
6. End-to-end bidding flow tests.
7. SQS FIFO and concurrency processing tests.
8. S3 and CloudFront tests.
9. Error handling and recovery tests.
10. Security tests.
11. Performance tests.
12. Results consolidation.

If authentication tests fail, dependent test cases that require valid tokens may be marked `BLOCKED`. Similarly, if WebSocket or SQS FIFO are not working, the end-to-end bidding flow cannot be validated.

#### Re-testing After Fixes

When a test case is `FAIL`, follow this process:

```text
Record issue
→ Diagnose cause
→ Fix
→ Deploy new release
→ Re-run the failed test case
→ Verify related functionality
→ Update results and evidence
```

Retesting must re-execute the test case from the beginning. Do not change the outcome from `FAIL` to `PASS` solely because code changes were made without re-running the test.

In addition to re-running the failed case, perform regression tests on related functionality to ensure the fix does not introduce regressions.

#### Test Evidence Security

Do not show in documentation or evidence:

- Account passwords.
- Access Token, ID Token or Refresh Token.
- `Authorization` header.
- AWS Access Key ID or Secret Access Key.
- AWS Session Token.
- Cognito Client Secret.
- Login cookies or session data.
- `.env` contents.
- Presigned URLs that are still valid.
- Unnecessary personal data.

If requests or logs contain sensitive data, redact or remove them before inclusion in reports. Keep only the information required to prove the test case outcome.
