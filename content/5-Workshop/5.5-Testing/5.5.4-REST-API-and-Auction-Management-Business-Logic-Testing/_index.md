---
title: "REST API and Auction Management Business Logic Testing"
date: 2026-08-03
weight: 4
chapter: false
pre: " <b> 5.5.4. </b> "
---

### REST API and Auction Management Business Logic Testing

#### Testing Objectives

This section tests the REST APIs and auction session management business logic implemented through Amazon API Gateway and the Business Logic Lambda.

The testing objectives include:

* Retrieve the list of auction sessions.
* View auction session and item details.
* Create and update auction sessions.
* Start and end auction sessions.
* Verify authorization between User and Admin roles.
* Validate input data.
* Verify auction session state transition rules.
* Verify the actual data written to DynamoDB.
* Verify API HTTP status codes and JSON structures.
* Verify that the frontend receives and displays results correctly.

The test cases are numbered from `API-01` to `API-12`.

A test case must not be marked as `PASS` simply because the Lambda function returns HTTP `200`. The team must verify all of the following:

1. API Gateway returns the correct HTTP status.
2. The response body has the correct JSON structure.
3. The Business Logic Lambda performs the correct business operation.
4. Data in DynamoDB is created or updated correctly.
5. The frontend receives and displays the correct result.
6. No data outside the requested scope is modified.

---

#### Testing Scope

The components to be tested include:

* User Frontend.
* Admin Frontend.
* Amazon API Gateway REST API.
* API Gateway Authorizer.
* Business Logic Lambda.
* Amazon DynamoDB.
* Amazon CloudWatch Logs.
* Cognito claims such as `sub` and `cognito:groups`.

---

#### General Testing Preconditions

Before executing the test cases, the system must meet the following conditions:

* The REST API has been deployed on API Gateway.
* Routes are connected to the correct Lambda Functions.
* APIs requiring authentication are protected by an Authorizer.
* The Business Logic Lambda has permission to read from or write to the correct DynamoDB tables.
* DynamoDB contains tables for storing auction sessions and items.
* The User Frontend and Admin Frontend can call the API.
* A confirmed User account is available.
* An Admin account belonging to the Cognito Group `Admin` is available.
* Valid Access Tokens are available for both User and Admin.
* CloudWatch Logs are enabled.
* Auction session data in the required states is available for testing.
* The testing environment is separated from production data.

---

#### Test Data

| Data                        | Description                                                           |
| --------------------------- | --------------------------------------------------------------------- |
| Valid Admin                 | Account belonging to Cognito Group `Admin`                            |
| Valid User                  | Account not belonging to the Admin group                              |
| `SCHEDULED` session         | Session that has been created but has not started                     |
| `ACTIVE` session            | Session currently in progress                                         |
| `ENDED` session             | Session that has ended                                                |
| `CANCELLED` session         | Session that has been cancelled, if supported by the system           |
| Valid Session ID            | ID of an existing session                                             |
| Non-existent Session ID     | Correctly formatted ID that does not exist in DynamoDB                |
| Valid Item ID               | ID of an existing item                                                |
| Non-existent Item ID        | Correctly formatted ID that does not exist in DynamoDB                |
| Valid session creation data | Includes name, start time, end time, and all required fields          |
| Invalid data                | Missing fields, incorrect data types, or violations of business rules |
| Valid time                  | `startTime` is earlier than `endTime`                                 |
| Invalid time                | `startTime` is greater than or equal to `endTime`                     |

Tokens, passwords, or sensitive information must not be included in the testing documentation or evidence.

---

#### HTTP Status Code Conventions

|                 HTTP status | Usage                                                                       |
| --------------------------: | --------------------------------------------------------------------------- |
|                    `200 OK` | Successfully retrieves data or updates business data                        |
|               `201 Created` | Successfully creates a new auction session                                  |
|           `400 Bad Request` | Request is missing fields, has invalid formatting, or violates input rules  |
|          `401 Unauthorized` | Token is missing or invalid                                                 |
|             `403 Forbidden` | User is authenticated but does not have permission to perform the operation |
|             `404 Not Found` | Auction session or item does not exist                                      |
|              `409 Conflict` | Current state does not allow the operation, or a business conflict occurs   |
| `500 Internal Server Error` | Unhandled internal system error                                             |

> The project may use `400` instead of `409` for invalid state transitions. However, the entire API must follow a consistent contract, and this behavior must be clearly documented in the API documentation.

---

#### General JSON Structures to Verify

For successful responses, the API should use a consistent structure, for example:

```json
{
  "data": {},
  "message": "Operation completed successfully",
  "requestId": "example-request-id"
}
```

For failed responses, the API should return a structure that can be handled properly:

```json
{
  "error": {
    "code": "AUCTION_SESSION_NOT_FOUND",
    "message": "Auction session was not found"
  },
  "requestId": "example-request-id"
}
```

The following information must not be returned to the client:

* Stack traces.
* Internal source code paths.
* AWS credentials.
* Table names or unnecessary infrastructure information.
* Token contents.
* Error details that could expose the internal system structure.

The actual structure may differ from the examples above, but it must remain consistent across endpoints.

---

#### API-01 — Retrieve Auction Session List

| Field               | Content                                                                                                                                                                                                                                                                    |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `API-01`                                                                                                                                                                                                                                                                   |
| **Test Name**       | Retrieve auction session list                                                                                                                                                                                                                                              |
| **Objective**       | Verify that the API returns the auction session list from DynamoDB and that the frontend displays the data correctly.                                                                                                                                                      |
| **Preconditions**   | DynamoDB contains multiple auction sessions in different states; the list retrieval endpoint has been deployed.                                                                                                                                                            |
| **Test Steps**      | 1. Record the existing sessions in DynamoDB.<br>2. Open the auction session list page.<br>3. Send a request to retrieve the session list.<br>4. Record the HTTP status and JSON response.<br>5. Compare the API data with DynamoDB.<br>6. Verify the list on the frontend. |
| **Input Data**      | Pagination parameters, status filters, or sorting options if supported by the API.                                                                                                                                                                                         |
| **Expected Result** | The API returns HTTP `200`; the response contains the session list using the correct structure; the data matches DynamoDB; filtering, pagination, and sorting work correctly; the frontend displays the correct name, status, and time for each session.                   |
| **Actual Result**   | Record the number of returned records, HTTP status, response structure, and actual frontend display result.                                                                                                                                                                |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                              |
| **Evidence**        | HTTP request/response, corresponding DynamoDB data, and a screenshot of the frontend list.                                                                                                                                                                                 |

---

#### API-02 — View Auction Session or Item Details

| Field               | Content                                                                                                                                                                                                                                                     |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `API-02`                                                                                                                                                                                                                                                    |
| **Test Name**       | View auction session and item details                                                                                                                                                                                                                       |
| **Objective**       | Verify that the API returns the correct details for the requested resource.                                                                                                                                                                                 |
| **Preconditions**   | Valid auction sessions and items exist in DynamoDB.                                                                                                                                                                                                         |
| **Test Steps**      | 1. Select an existing session or item.<br>2. Send a request to retrieve details by ID.<br>3. Record the HTTP status and JSON response.<br>4. Compare the fields with DynamoDB.<br>5. Open the detail page on the frontend.<br>6. Verify the displayed data. |
| **Input Data**      | Valid Session ID or Item ID.                                                                                                                                                                                                                                |
| **Expected Result** | The API returns HTTP `200`; the ID in the response matches the requested ID; fields such as name, description, status, time, price, and related items match DynamoDB; the frontend displays the correct data.                                               |
| **Actual Result**   | Record the HTTP status, ID, compared fields, and frontend result.                                                                                                                                                                                           |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                               |
| **Evidence**        | Response JSON, DynamoDB record, and screenshot of the detail page.                                                                                                                                                                                          |

---

#### API-03 — Admin Successfully Creates an Auction Session

| Field               | Content                                                                                                                                                                                                                                                                                                            |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Test Case ID**    | `API-03`                                                                                                                                                                                                                                                                                                           |
| **Test Name**       | Create an auction session using an Admin account                                                                                                                                                                                                                                                                   |
| **Objective**       | Verify that an Admin can create a new auction session and that the data is stored correctly.                                                                                                                                                                                                                       |
| **Preconditions**   | Admin is logged in; the token contains the `cognito:groups` claim with `Admin`; the session creation endpoint has been deployed.                                                                                                                                                                                   |
| **Test Steps**      | 1. Record the data before testing.<br>2. Log in as Admin.<br>3. Enter all required session data on the Admin Frontend.<br>4. Send the session creation request.<br>5. Verify the HTTP status and response.<br>6. Find the newly created record in DynamoDB.<br>7. Reopen the session list or session details page. |
| **Input Data**      | Session name, description, `startTime`, `endTime`, and all valid required fields.                                                                                                                                                                                                                                  |
| **Expected Result** | The API returns HTTP `201`; the response contains a new ID; the session is stored exactly once in DynamoDB; the initial status is `SCHEDULED` or the status specified by the contract; `createdBy` is taken from the Admin's `sub` claim; the frontend displays the new session.                                   |
| **Actual Result**   | Record the HTTP status, generated ID, status, and actual DynamoDB data.                                                                                                                                                                                                                                            |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                      |
| **Evidence**        | Request with token masked, `201` response, DynamoDB record, CloudWatch Logs, and Admin Frontend screenshot.                                                                                                                                                                                                        |

---

#### API-04 — Admin Updates an Auction Session

| Field               | Content                                                                                                                                                                                                                                                    |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `API-04`                                                                                                                                                                                                                                                   |
| **Test Name**       | Update auction session information                                                                                                                                                                                                                         |
| **Objective**       | Verify that an Admin can update a session while it is in an editable state.                                                                                                                                                                                |
| **Preconditions**   | A session exists in a state that allows editing; Admin is logged in.                                                                                                                                                                                       |
| **Test Steps**      | 1. Record the session data before the update.<br>2. Modify allowed fields, such as the name or time.<br>3. Send the update request.<br>4. Verify the response.<br>5. Read the DynamoDB record again.<br>6. Reload the session detail page on the frontend. |
| **Input Data**      | Valid Session ID and valid update fields.                                                                                                                                                                                                                  |
| **Expected Result** | The API returns HTTP `200`; only requested fields are updated; unrelated fields remain unchanged; `updatedAt` is updated if the system uses this field; DynamoDB and the frontend display the new values.                                                  |
| **Actual Result**   | Record the data before and after the update, HTTP status, and displayed result.                                                                                                                                                                            |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                              |
| **Evidence**        | Response, DynamoDB records before and after the update, CloudWatch Logs, and frontend screenshot.                                                                                                                                                          |

---

#### API-05 — Start an Auction Session

| Field               | Content                                                                                                                                                                                                                                                                                                    |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `API-05`                                                                                                                                                                                                                                                                                                   |
| **Test Name**       | Transition a session from `SCHEDULED` to `ACTIVE`                                                                                                                                                                                                                                                          |
| **Objective**       | Verify that a session can only be started when all required conditions are satisfied.                                                                                                                                                                                                                      |
| **Preconditions**   | The session is currently `SCHEDULED`; required data and items are available; Admin is logged in.                                                                                                                                                                                                           |
| **Test Steps**      | 1. Verify the initial status in DynamoDB.<br>2. Send the request to start the session.<br>3. Record the HTTP status and response.<br>4. Read the session from DynamoDB again.<br>5. Verify the status on the User Frontend and Admin Frontend.<br>6. Try accessing functions available to active sessions. |
| **Input Data**      | Session ID of a `SCHEDULED` session.                                                                                                                                                                                                                                                                       |
| **Expected Result** | The API returns HTTP `200`; the status changes to `ACTIVE`; the actual start time is recorded if applicable; the session is updated only once; the frontend displays the session as active and enables appropriate operations.                                                                             |
| **Actual Result**   | Record the status before and after the operation, update time, and frontend result.                                                                                                                                                                                                                        |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                              |
| **Evidence**        | API response, DynamoDB records before and after the operation, CloudWatch Logs, and screenshot showing `ACTIVE` status.                                                                                                                                                                                    |

---

#### API-06 — End an Auction Session

| Field               | Content                                                                                                                                                                                                                                                                                   |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `API-06`                                                                                                                                                                                                                                                                                  |
| **Test Name**       | Transition a session from `ACTIVE` to `ENDED`                                                                                                                                                                                                                                             |
| **Objective**       | Verify that an active session can be ended and that the system prevents operations that are no longer valid.                                                                                                                                                                              |
| **Preconditions**   | The session is currently `ACTIVE`; Admin is logged in, or the automatic ending mechanism is ready.                                                                                                                                                                                        |
| **Test Steps**      | 1. Verify the initial status.<br>2. Send the request to end the session.<br>3. Record the response.<br>4. Read the data from DynamoDB again.<br>5. Reload the session page on the frontend.<br>6. Attempt an operation allowed only while the session is `ACTIVE`, such as placing a bid. |
| **Input Data**      | Session ID of an `ACTIVE` session.                                                                                                                                                                                                                                                        |
| **Expected Result** | The API returns HTTP `200`; the status changes to `ENDED`; the end time is recorded; the frontend displays the session as ended; the system no longer accepts operations restricted to `ACTIVE` sessions.                                                                                 |
| **Actual Result**   | Record the status, end time, response, and result of attempting an operation after the session ends.                                                                                                                                                                                      |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                             |
| **Evidence**        | Response, DynamoDB before and after the operation, CloudWatch Logs, and frontend screenshot.                                                                                                                                                                                              |

---

#### API-07 — User Is Not Allowed to Create or Modify Auction Sessions

| Field               | Content                                                                                                                                                                                                                                               |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `API-07`                                                                                                                                                                                                                                              |
| **Test Name**       | Reject User attempts to perform session administration operations                                                                                                                                                                                     |
| **Objective**       | Verify that a regular User cannot create, update, start, or end auction sessions.                                                                                                                                                                     |
| **Preconditions**   | User is logged in but does not belong to the Cognito Group `Admin`; administrative endpoints are available for testing.                                                                                                                               |
| **Test Steps**      | 1. Log in as User.<br>2. Send a request to create a session.<br>3. Send a request to update an existing session.<br>4. If applicable, attempt to start or end a session.<br>5. Verify the HTTP status.<br>6. Verify DynamoDB data after each request. |
| **Input Data**      | Valid User token and a syntactically valid request.                                                                                                                                                                                                   |
| **Expected Result** | Administrative APIs return HTTP `403`; the Business Logic Lambda does not perform unauthorized changes; no new session is created; existing sessions are not updated; the frontend does not display or allow use of Admin functions.                  |
| **Actual Result**   | Record the status of each request and the resulting data state after testing.                                                                                                                                                                         |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                         |
| **Evidence**        | `403` response, unchanged DynamoDB data, and User interface screenshot.                                                                                                                                                                               |

> Hiding administrative buttons on the frontend does not replace backend authorization checks. The test case must send requests directly to the API using a User token.

---

#### API-08 — Missing or Invalid Input Data

| Field               | Content                                                                                                                                                                                                                                                                                                                                                       |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `API-08`                                                                                                                                                                                                                                                                                                                                                      |
| **Test Name**       | Verify request validation                                                                                                                                                                                                                                                                                                                                     |
| **Objective**       | Verify that the API rejects requests with missing required fields, incorrect data types, or violations of data rules.                                                                                                                                                                                                                                         |
| **Preconditions**   | Admin is logged in and has permission to call the create or update session APIs.                                                                                                                                                                                                                                                                              |
| **Test Steps**      | 1. Send a request without a session name.<br>2. Send a request without `startTime` or `endTime`.<br>3. Send a request with an invalid time format.<br>4. Send a request where `startTime` is greater than or equal to `endTime`.<br>5. Send a request containing fields with incorrect data types.<br>6. Verify the response and DynamoDB after each request. |
| **Input Data**      | Requests with missing fields or invalid data.                                                                                                                                                                                                                                                                                                                 |
| **Expected Result** | The API returns HTTP `400`; the response identifies the invalid field or rule; no partial data is created or updated in DynamoDB; the frontend displays an understandable validation message.                                                                                                                                                                 |
| **Actual Result**   | Record each data set, HTTP status, error code, and DynamoDB state.                                                                                                                                                                                                                                                                                            |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                                                                 |
| **Evidence**        | Error requests/responses, unchanged DynamoDB data, and frontend validation screenshot.                                                                                                                                                                                                                                                                        |

---

#### API-09 — Resource ID Does Not Exist

| Field               | Content                                                                                                                                                                                                                              |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Test Case ID**    | `API-09`                                                                                                                                                                                                                             |
| **Test Name**       | Access or update a non-existent resource                                                                                                                                                                                             |
| **Objective**       | Verify that the API correctly handles a Session ID or Item ID that does not exist.                                                                                                                                                   |
| **Preconditions**   | A correctly formatted ID that does not exist in DynamoDB is available.                                                                                                                                                               |
| **Test Steps**      | 1. Call the detail API using a non-existent ID.<br>2. Call the update API using that ID.<br>3. If applicable, attempt to start or end a session using that ID.<br>4. Record the response.<br>5. Verify DynamoDB and CloudWatch Logs. |
| **Input Data**      | Non-existent Session ID or Item ID.                                                                                                                                                                                                  |
| **Expected Result** | The API returns HTTP `404`; the response contains a consistent error code and message; no unintended resource is created; no data is modified; the frontend displays a resource-not-found message.                                   |
| **Actual Result**   | Record the HTTP status, error code, and data state.                                                                                                                                                                                  |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                        |
| **Evidence**        | `404` response, DynamoDB lookup result, and frontend screenshot.                                                                                                                                                                     |

---

#### API-10 — Session State Does Not Allow the Operation

| Field               | Content                                                                                                                                                                                                                                                                                    |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Test Case ID**    | `API-10`                                                                                                                                                                                                                                                                                   |
| **Test Name**       | Reject invalid state transitions or operations                                                                                                                                                                                                                                             |
| **Objective**       | Verify that the Business Logic Lambda correctly enforces the auction session state machine.                                                                                                                                                                                                |
| **Preconditions**   | Sessions exist in `SCHEDULED`, `ACTIVE`, and `ENDED` states.                                                                                                                                                                                                                               |
| **Test Steps**      | 1. Attempt to directly end a `SCHEDULED` session if this flow is not allowed.<br>2. Attempt to start an `ACTIVE` session again.<br>3. Attempt to start or edit an `ENDED` session.<br>4. Attempt to end an `ENDED` session again.<br>5. Verify the response and data after each operation. |
| **Input Data**      | Session ID and an operation that is invalid for the current state.                                                                                                                                                                                                                         |
| **Expected Result** | The API returns HTTP `409` or `400` according to the API contract; the response indicates that the current state does not permit the operation; the session state and data remain unchanged; the frontend displays an appropriate message.                                                 |
| **Actual Result**   | Record the initial state, operation, HTTP status, and state after the request.                                                                                                                                                                                                             |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                              |
| **Evidence**        | Business error response, DynamoDB before and after the operation, CloudWatch Logs, and frontend screenshot.                                                                                                                                                                                |

Valid state transitions should follow the project's state machine, for example:

```text
SCHEDULED → ACTIVE → ENDED
```

The following transitions are not allowed:

```text
ENDED → ACTIVE
ACTIVE → SCHEDULED
ENDED → SCHEDULED
```

If the project supports the `CANCELLED` state, all related state transitions must be explicitly defined and tested according to the business contract.

---

#### API-11 — Verify DynamoDB Data Consistency

| Field               | Content                                                                                                                                                                                                                                                                                                                              |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Test Case ID**    | `API-11`                                                                                                                                                                                                                                                                                                                             |
| **Test Name**       | Verify data after session management operations                                                                                                                                                                                                                                                                                      |
| **Objective**       | Verify that the actual data in DynamoDB is consistent with the request, response, and business rules.                                                                                                                                                                                                                                |
| **Preconditions**   | At least one create, update, or session state transition operation can be performed.                                                                                                                                                                                                                                                 |
| **Test Steps**      | 1. Record data before the operation.<br>2. Perform the operation through the REST API.<br>3. Record the response.<br>4. Read the corresponding DynamoDB record directly.<br>5. Compare the ID, status, time, creator, and related attributes.<br>6. Verify the number of records.<br>7. Call the read API again to confirm the data. |
| **Input Data**      | A valid request to create, update, start, or end a session.                                                                                                                                                                                                                                                                          |
| **Expected Result** | DynamoDB contains exactly the data accepted by the business operation; there are no duplicate records; the partition key and sort key are correct; the state is correct; `createdBy` or `updatedBy` is derived from a verified claim; unrelated attributes are not lost; the read API returns the latest data.                       |
| **Actual Result**   | Record the data before and after the operation and the fields that were compared.                                                                                                                                                                                                                                                    |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                                        |
| **Evidence**        | Request/response and DynamoDB records before and after the operation.                                                                                                                                                                                                                                                                |

If the API returns success but DynamoDB does not contain the data, stores incorrect data, or the frontend continues to display outdated data, the test case must be marked as `FAIL`.

---

#### API-12 — Verify API Contract and Frontend Results

| Field               | Content                                                                                                                                                                                                                                                                                                                                                                     |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `API-12`                                                                                                                                                                                                                                                                                                                                                                    |
| **Test Name**       | Verify HTTP status, JSON response, and frontend integration                                                                                                                                                                                                                                                                                                                 |
| **Objective**       | Verify that the frontend can reliably consume API responses and correctly display business operation results.                                                                                                                                                                                                                                                               |
| **Preconditions**   | The User Frontend and Admin Frontend are integrated with the REST API.                                                                                                                                                                                                                                                                                                      |
| **Test Steps**      | 1. Perform a successful data retrieval request.<br>2. Perform a successful create or update request.<br>3. Perform a request that fails validation.<br>4. Perform a request for a non-existent resource.<br>5. Perform an unauthorized request.<br>6. Verify the HTTP status, `Content-Type`, and JSON of each response.<br>7. Verify how the frontend handles each result. |
| **Input Data**      | Requests representing successful, invalid data, not-found, and insufficient-permission scenarios.                                                                                                                                                                                                                                                                           |
| **Expected Result** | The API returns the appropriate status such as `200`, `201`, `400`, `403`, `404`, or `409`; `Content-Type` is `application/json`; JSON field names and data types are consistent; the frontend displays updated data after successful operations and appropriate messages after failures; no JavaScript errors occur because of an invalid response structure.              |
| **Actual Result**   | Record the HTTP status, JSON structure, and displayed result for each case.                                                                                                                                                                                                                                                                                                 |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                                                                               |
| **Evidence**        | Network tab, request/response, frontend screenshots, and console logs with sensitive data removed.                                                                                                                                                                                                                                                                          |

---

#### State Transition Matrix to Verify

| Current State | Operation                  | Expected State | Result   |
| ------------- | -------------------------- | -------------- | -------- |
| `SCHEDULED`   | Update valid information   | `SCHEDULED`    | Allowed  |
| `SCHEDULED`   | Start session              | `ACTIVE`       | Allowed  |
| `ACTIVE`      | Start again                | No change      | Rejected |
| `ACTIVE`      | End session                | `ENDED`        | Allowed  |
| `ACTIVE`      | Change back to `SCHEDULED` | No change      | Rejected |
| `ENDED`       | Start again                | No change      | Rejected |
| `ENDED`       | End again                  | No change      | Rejected |
| `ENDED`       | Edit locked business data  | No change      | Rejected |

The matrix above must be adjusted if the project's official business contract defines additional states or operations.

---

#### DynamoDB Verification Requirements

For each API that modifies data, the team must verify:

* The partition key and sort key match the table design.
* The ID in the response matches the ID in DynamoDB.
* Data is created only once.
* The status is updated correctly.
* `createdAt` does not change during an update.
* `updatedAt` changes after a successful update.
* `createdBy` or `updatedBy` is derived from the `sub` claim.
* `userId` or `role` from the request body is not treated as trusted data.
* No partial update occurs when the business operation fails.
* No unrelated attributes are deleted.
* DynamoDB data types match the design.
* Data retrieved again through the API matches the stored data.

If the system uses multiple tables or multiple items for one business operation, the team must verify all related data, not only the primary item.

---

#### CloudWatch Logs Verification Requirements

CloudWatch Logs should support request tracing but must not contain sensitive information.

Information that should be recorded includes:

* Request ID.
* Lambda Function invoked.
* Business operation name.
* Resource ID.
* Verified User `sub`.
* State before and after the business operation.
* Success result or error code.
* Processing time.

The following must not be written to logs:

* Access Token.
* ID Token.
* Refresh Token.
* `Authorization` header.
* Password.
* AWS credentials.
* Unnecessary complete personal information.

---

#### Result Summary Table

| ID       | Test Content                     | Expected HTTP Status | DynamoDB Verification | Frontend Verification | Status     |
| -------- | -------------------------------- | -------------------: | --------------------- | --------------------- | ---------- |
| `API-01` | Retrieve session list            |                `200` | Yes                   | Yes                   | Not tested |
| `API-02` | View session or item details     |                `200` | Yes                   | Yes                   | Not tested |
| `API-03` | Admin creates session            |                `201` | Yes                   | Yes                   | Not tested |
| `API-04` | Admin updates session            |                `200` | Yes                   | Yes                   | Not tested |
| `API-05` | Start session                    |                `200` | Yes                   | Yes                   | Not tested |
| `API-06` | End session                      |                `200` | Yes                   | Yes                   | Not tested |
| `API-07` | User creates or modifies session |                `403` | Must remain unchanged | Yes                   | Not tested |
| `API-08` | Invalid input data               |                `400` | Must remain unchanged | Yes                   | Not tested |
| `API-09` | Resource does not exist          |                `404` | Must remain unchanged | Yes                   | Not tested |
| `API-10` | State does not allow operation   |       `409` or `400` | Must remain unchanged | Yes                   | Not tested |
| `API-11` | DynamoDB consistency             | Depends on operation | Required              | Read data again       | Not tested |
| `API-12` | HTTP status, JSON, and frontend  |  Depends on scenario | When changes occur    | Required              | Not tested |

---

#### Testing Evidence

Evidence for REST API and auction management business logic testing includes:

* User Frontend screenshots.
* Admin Frontend screenshots.
* HTTP requests and responses.
* HTTP status for each test case.
* Response header `Content-Type`.
* DynamoDB records before and after business operations.
* CloudWatch Logs from the Business Logic Lambda.
* API Gateway Logs, if enabled.
* Browser Network tab.
* Frontend console logs if errors occur.
* Screenshots of session status before and after starting or ending the session.
* Request ID for cross-referencing between API Gateway and Lambda.

Each piece of evidence should clearly indicate the corresponding test case ID, for example:

```text
Figure 5.5.4.1: HTTP 201 response for test case API-03
Figure 5.5.4.2: DynamoDB record created after test case API-03
Figure 5.5.4.3: New auction session displayed on the Admin Frontend
```

---

#### Evaluation Criteria

A test case may only be marked as `PASS` when all of the following conditions are satisfied:

* The HTTP status matches the API contract.
* The response JSON has the correct structure and data types.
* The Business Logic Lambda performs the correct business operation.
* DynamoDB contains the correct data or remains unchanged when the request is rejected.
* The frontend receives and displays the correct result.
* No unintended data changes occur.
* Direct evidence of the result is available.

A test case must be marked as `FAIL` in any of the following situations:

* The API returns HTTP `200`, but the data is not stored.
* The API reports success but stores the wrong state.
* DynamoDB is updated even though the request is rejected.
* A User can perform an operation intended for an Admin.
* The API returns an incorrect HTTP status code.
* The response JSON is missing required fields or contains incorrect data types.
* The frontend does not display new data after a successful API request.
* Lambda returns an error, but API Gateway converts it into a successful response.
* A create operation generates multiple duplicate records.

A test case is marked as `BLOCKED` when the endpoint, Lambda, DynamoDB table, frontend, Authorizer, or dependent data has not yet been implemented.
