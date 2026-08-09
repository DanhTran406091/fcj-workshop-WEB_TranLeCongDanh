### 5.5.5. WebSocket Connection and Real-Time Update Testing

#### Testing Objectives

This section tests the system's ability to establish WebSocket connections and deliver real-time data updates through:

* Amazon API Gateway WebSocket API.
* Lambda WebSocket Handler.
* The `$connect`, `$disconnect`, and `$default` routes.
* Routes for joining auction rooms and sending messages.
* The connection management table in Amazon DynamoDB.
* Lambda Broadcast.
* API Gateway Management API.
* Amazon CloudWatch Logs.
* The auction system frontend.

The test cases are numbered from `WS-01` to `WS-13`.

A WebSocket test case may only be marked as `PASS` when all of the following are verified:

1. The client receives the correct connection state or message.
2. The WebSocket Handler performs the correct business logic.
3. The Connection ID is stored, updated, or deleted correctly in DynamoDB.
4. Lambda Broadcast sends data to the correct users and auction room.
5. Failed or expired connections do not cause the entire broadcast process to fail.
6. Private auction-room data is not accidentally sent to users in other rooms.
7. Direct evidence is available from the browser, DynamoDB, or CloudWatch Logs.

---

#### Testing Scope

The components to be tested include:

* User Frontend.
* Amazon API Gateway WebSocket API.
* `$connect` route.
* `$disconnect` route.
* `$default` route.
* Routes for joining or leaving auction rooms.
* Lambda WebSocket Handler.
* Lambda Broadcast.
* API Gateway Management API.
* DynamoDB Connections Table.
* DynamoDB Auction Room or Subscription Table, if stored separately.
* Amazon CloudWatch Logs.
* Amazon Cognito or the system's WebSocket authentication mechanism.

---

#### General Testing Preconditions

Before testing begins, the system must meet the following conditions:

* The WebSocket API has been deployed on API Gateway.
* The WebSocket URL for the testing environment is configured on the frontend.
* The `$connect`, `$disconnect`, and `$default` routes are linked to the correct Lambda functions.
* Business routes such as `join_room`, `leave_room`, or `send_message` have been implemented if the architecture uses separate routes.
* Lambda has permission to read, write, and delete data in the DynamoDB connections table.
* Lambda Broadcast has permission to call `execute-api:ManageConnections`.
* CloudWatch Logs are enabled for the relevant Lambda functions.
* At least two valid User accounts are available.
* At least two different auction items or auction rooms are available.
* Two browser windows or two independent browser sessions can be opened.
* DynamoDB records can be inspected directly.
* The testing environment is separated from production data.
* The clock on the testing device is synchronized for log timestamp comparison.

If a required Lambda function, route, DynamoDB table, or frontend function has not yet been implemented, the corresponding test case must be marked as `BLOCKED`.

---

#### Test Data

| Data                  | Description                                                                    |
| --------------------- | ------------------------------------------------------------------------------ |
| User A                | Valid account participating in Auction Item A                                  |
| User B                | Valid account participating in the same Auction Item A                         |
| User C                | Valid account participating only in Auction Item B                             |
| Auction Item A        | Item with a valid WebSocket room                                               |
| Auction Item B        | Item different from Item A                                                     |
| Room A                | WebSocket room for Auction Item A                                              |
| Room B                | WebSocket room for Auction Item B                                              |
| Valid Connection ID   | Active Connection ID issued by API Gateway                                     |
| Expired Connection ID | Connection ID belonging to a client that has closed or lost its connection     |
| Valid message         | JSON containing the correct `action`, `roomId`, and required fields            |
| Invalid message       | Malformed JSON, missing fields, or unsupported action                          |
| Valid event           | Status update, bid update, viewer count update, or system notification         |
| Valid token           | Unexpired token belonging to a valid account                                   |
| Invalid token         | Token with an invalid signature, expired token, or incorrectly formatted token |

Access Tokens, ID Tokens, Refresh Tokens, and authentication headers must not appear in screenshots or testing reports.

---

#### Connection Status and Message Structure Conventions

WebSocket connections do not use HTTP status codes for every message after the connection has been established. Therefore, results must be verified in two stages:

* Connection handshake stage: verify the HTTP status of the WebSocket handshake.
* Connected stage: verify WebSocket messages and connection state.

Common handshake outcomes include:

|                    Result | Meaning                                                    |
| ------------------------: | ---------------------------------------------------------- |
| `101 Switching Protocols` | WebSocket connection was established successfully          |
|        `401 Unauthorized` | Authentication information is missing or invalid           |
|           `403 Forbidden` | User is authenticated but not permitted to connect         |
|         Connection closed | API Gateway or Lambda rejects or terminates the connection |

A successful message should use a consistent structure, for example:

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

An error message should use a structure that the frontend can handle, for example:

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

Messages sent to clients must not contain:

* Access Token.
* ID Token.
* Refresh Token.
* AWS credentials.
* Another user's Connection ID.
* Stack traces.
* DynamoDB table names.
* Unnecessary internal infrastructure details.
* Another user's personal data.

---

#### WS-01 — User Successfully Connects to WebSocket

| Field               | Content                                                                                                                                                                                                                                                       |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `WS-01`                                                                                                                                                                                                                                                       |
| **Test Name**       | Valid User successfully establishes a WebSocket connection                                                                                                                                                                                                    |
| **Objective**       | Verify that an authenticated User can establish a connection to the API Gateway WebSocket API.                                                                                                                                                                |
| **Preconditions**   | The WebSocket API and `$connect` route have been deployed; User A has valid authentication information.                                                                                                                                                       |
| **Test Steps**      | 1. Log in as User A.<br>2. Open the Auction Item A detail page.<br>3. Open the Network tab and select the WebSocket filter.<br>4. Observe the WebSocket handshake.<br>5. Verify the connection state on the frontend.<br>6. Check the `$connect` Lambda logs. |
| **Input Data**      | Valid WebSocket URL and valid authentication information for User A.                                                                                                                                                                                          |
| **Expected Result** | The handshake returns `101 Switching Protocols`; the frontend changes to Connected or Live; the `$connect` Lambda is invoked exactly once; there is no continuous reconnect loop; logs do not contain the token.                                              |
| **Actual Result**   | Record the HTTP status, connection state, connection time, and actual Request ID.                                                                                                                                                                             |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                 |
| **Evidence**        | Network tab, Live status on the frontend, and CloudWatch Logs for `$connect`.                                                                                                                                                                                 |

---

#### WS-02 — Invalid Connection Is Rejected

| Field               | Content                                                                                                                                                                                                                                                         |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `WS-02`                                                                                                                                                                                                                                                         |
| **Test Name**       | Reject an invalid WebSocket connection                                                                                                                                                                                                                          |
| **Objective**       | Verify that a user without valid authentication information cannot establish a connection.                                                                                                                                                                      |
| **Preconditions**   | `$connect` performs authentication checks or an Authorizer has been configured.                                                                                                                                                                                 |
| **Test Steps**      | 1. Attempt to connect without authentication information.<br>2. Attempt to connect using an incorrectly formatted token.<br>3. Attempt to connect using an expired token.<br>4. Record the handshake result.<br>5. Check DynamoDB.<br>6. Check CloudWatch Logs. |
| **Input Data**      | No token, invalid token, or expired token.                                                                                                                                                                                                                      |
| **Expected Result** | The connection is rejected with `401`, `403`, or closed according to the system contract; the frontend does not display Live status; no active Connection ID is stored in DynamoDB; logs record the error code but do not contain the token value.              |
| **Actual Result**   | Record each type of test input and its corresponding result.                                                                                                                                                                                                    |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                   |
| **Evidence**        | Handshake result, absence of a valid connection record in DynamoDB, and CloudWatch Logs with sensitive data masked.                                                                                                                                             |

> If the token is transmitted through a query string, the team must verify that the token is not recorded in access logs, browser history, or evidence screenshots. A WebSocket URL containing a token must not be disclosed publicly.

---

#### WS-03 — `$connect` Event Stores the Connection ID

| Field               | Content                                                                                                                                                                                                                                                                                                               |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `WS-03`                                                                                                                                                                                                                                                                                                               |
| **Test Name**       | Store connection information after successful `$connect`                                                                                                                                                                                                                                                              |
| **Objective**       | Verify that the `$connect` Lambda stores the Connection ID and user information correctly in DynamoDB.                                                                                                                                                                                                                |
| **Preconditions**   | User A can connect successfully; Lambda has permission to write to the Connections Table.                                                                                                                                                                                                                             |
| **Test Steps**      | 1. Record the table data before connecting.<br>2. User A opens the Auction Item A page.<br>3. Confirm that the connection succeeds.<br>4. Find the new record in DynamoDB.<br>5. Compare creation time, User ID, and Connection ID with CloudWatch Logs.<br>6. Verify the expiration attribute if the table uses TTL. |
| **Input Data**      | Valid connection from User A.                                                                                                                                                                                                                                                                                         |
| **Expected Result** | One connection record is created; the Connection ID is not empty; User ID is derived from verified identity information; `connectedAt` is recorded correctly; TTL is in the future if used; tokens are not stored in DynamoDB.                                                                                        |
| **Actual Result**   | Record the partially masked Connection ID, User ID, creation time, and TTL.                                                                                                                                                                                                                                           |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                         |
| **Evidence**        | DynamoDB record and CloudWatch Logs for `$connect`.                                                                                                                                                                                                                                                                   |

---

#### WS-04 — User Joins the Correct Auction Room

| Field               | Content                                                                                                                                                                                                                                                                                  |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `WS-04`                                                                                                                                                                                                                                                                                  |
| **Test Name**       | Join the WebSocket room for the correct auction item                                                                                                                                                                                                                                     |
| **Objective**       | Verify that User A is associated with Room A when opening Auction Item A.                                                                                                                                                                                                                |
| **Preconditions**   | User A is connected; Auction Item A exists; the room-joining route has been implemented.                                                                                                                                                                                                 |
| **Test Steps**      | 1. User A opens the Auction Item A page.<br>2. The frontend sends a `join_room` message if required by the architecture.<br>3. Verify the response message.<br>4. Check the connection record in DynamoDB.<br>5. Check the WebSocket Handler logs.<br>6. Publish a test event to Room A. |
| **Input Data**      | Room ID or Auction Item ID of Item A.                                                                                                                                                                                                                                                    |
| **Expected Result** | User A's connection is associated with Room A; Lambda verifies that Room A exists; the client receives a join confirmation; Room A events are sent to User A; duplicate subscriptions are not created for the same connection and room.                                                  |
| **Actual Result**   | Record the Room ID, received response, and DynamoDB data.                                                                                                                                                                                                                                |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                            |
| **Evidence**        | WebSocket frame, DynamoDB record, and CloudWatch Logs.                                                                                                                                                                                                                                   |

---

#### WS-05 — Two Users Join the Same Auction Item

| Field               | Content                                                                                                                                                                                                                                                                                            |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `WS-05`                                                                                                                                                                                                                                                                                            |
| **Test Name**       | Two Users join Room A                                                                                                                                                                                                                                                                              |
| **Objective**       | Verify that the system can manage multiple simultaneous connections in the same auction room.                                                                                                                                                                                                      |
| **Preconditions**   | User A and User B are available; two independent browser sessions can be opened.                                                                                                                                                                                                                   |
| **Test Steps**      | 1. Open Auction Item A as User A in the first window.<br>2. Open the same Item A as User B in the second window.<br>3. Confirm that both connections show Live status.<br>4. Verify viewer count if supported by the frontend.<br>5. Check DynamoDB records.<br>6. Publish a test event to Room A. |
| **Input Data**      | Two different Users joining the same Room A.                                                                                                                                                                                                                                                       |
| **Expected Result** | Two different Connection IDs are associated with Room A; viewer count is `2` if the function exists; both windows receive the Room A event; neither connection overwrites the other.                                                                                                               |
| **Actual Result**   | Record the number of connections, viewer count, and messages received in each window.                                                                                                                                                                                                              |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                      |
| **Evidence**        | Screenshots of both active windows, WebSocket frames, DynamoDB data, and broadcast logs.                                                                                                                                                                                                           |

---

#### WS-06 — User Sends a Valid Message

| Field               | Content                                                                                                                                                                                                                                                                                              |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `WS-06`                                                                                                                                                                                                                                                                                              |
| **Test Name**       | Process a valid WebSocket message                                                                                                                                                                                                                                                                    |
| **Objective**       | Verify that the WebSocket Handler receives, validates, and processes a valid message correctly.                                                                                                                                                                                                      |
| **Preconditions**   | User A is connected and has joined Room A.                                                                                                                                                                                                                                                           |
| **Test Steps**      | 1. User A sends a message containing a supported action.<br>2. Check the sent frame.<br>3. Check the server response.<br>4. Check CloudWatch Logs.<br>5. If the message triggers a broadcast, verify the result for User B.<br>6. If the message changes data, verify the corresponding source data. |
| **Input Data**      | Valid JSON with the correct `action`, `roomId`, and required fields.                                                                                                                                                                                                                                 |
| **Expected Result** | The Handler correctly reads the action; verifies the User and Room; performs the business operation once; the client receives an ACK or appropriate result; related Users receive the correct event; the client cannot determine its own identity or permissions using message data.                 |
| **Actual Result**   | Record the message type, response, and actual broadcast result.                                                                                                                                                                                                                                      |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                        |
| **Evidence**        | WebSocket frames with sensitive data masked, CloudWatch Logs, and related data.                                                                                                                                                                                                                      |

---

#### WS-07 — Invalid Message Format Is Rejected

| Field               | Content                                                                                                                                                                                                                                                                                            |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `WS-07`                                                                                                                                                                                                                                                                                            |
| **Test Name**       | Reject an invalid WebSocket message                                                                                                                                                                                                                                                                |
| **Objective**       | Verify that Lambda safely handles malformed JSON, missing fields, or unsupported actions.                                                                                                                                                                                                          |
| **Preconditions**   | User A is connected through WebSocket.                                                                                                                                                                                                                                                             |
| **Test Steps**      | 1. Send a non-JSON string.<br>2. Send JSON without `action`.<br>3. Send an unsupported action.<br>4. Send a message without `roomId` when required.<br>5. Send a field with an incorrect data type.<br>6. Verify the response, logs, and data after each attempt.                                  |
| **Input Data**      | Syntactically invalid messages or messages that violate the schema.                                                                                                                                                                                                                                |
| **Expected Result** | The server returns a consistently structured error message; no business operation is performed; invalid messages are not broadcast; no unintended data changes occur; Lambda does not encounter an unhandled error; the connection may remain open or be closed according to the defined contract. |
| **Actual Result**   | Record each test message, error code, and connection state after the error.                                                                                                                                                                                                                        |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                      |
| **Evidence**        | Sent and received frames, unchanged DynamoDB or business data, and CloudWatch Logs.                                                                                                                                                                                                                |

---

#### WS-08 — Auction Status Is Broadcast to All Participants

| Field               | Content                                                                                                                                                                                                                                                                                |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `WS-08`                                                                                                                                                                                                                                                                                |
| **Test Name**       | Broadcast status updates to all members of Room A                                                                                                                                                                                                                                      |
| **Objective**       | Verify that Lambda Broadcast sends auction status updates to every active connection in the correct room.                                                                                                                                                                              |
| **Preconditions**   | User A and User B are participating in Room A; Lambda Broadcast and the Management API are ready.                                                                                                                                                                                      |
| **Test Steps**      | 1. Open two windows on Auction Item A.<br>2. Perform an operation that changes the status or creates a valid event.<br>3. Record the event publication time.<br>4. Observe the message in both windows.<br>5. Verify that the interface is updated.<br>6. Check Lambda Broadcast logs. |
| **Input Data**      | An event such as `AUCTION_STATUS_UPDATED`, `BID_UPDATED`, or `VIEWER_COUNT_UPDATED`.                                                                                                                                                                                                   |
| **Expected Result** | Both User A and User B receive the same event type, Room ID, and new data; the frontend updates without requiring a page reload; duplicate messages are not sent beyond the designed number; logs show the total number of target connections and successful and failed sends.         |
| **Actual Result**   | Record the message received in each window and the observed latency.                                                                                                                                                                                                                   |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                          |
| **Evidence**        | Screenshots of both windows, WebSocket messages, and Lambda Broadcast CloudWatch Logs.                                                                                                                                                                                                 |

---

#### WS-09 — A User Leaves the Page or Disconnects

| Field               | Content                                                                                                                                                                                                                                                            |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Test Case ID**    | `WS-09`                                                                                                                                                                                                                                                            |
| **Test Name**       | User leaves the auction room                                                                                                                                                                                                                                       |
| **Objective**       | Verify that the system handles a User closing the page, navigating away, or losing the connection.                                                                                                                                                                 |
| **Preconditions**   | User A and User B are both participating in Room A.                                                                                                                                                                                                                |
| **Test Steps**      | 1. Confirm that both Users are connected.<br>2. Close User B's tab or navigate away from Item A.<br>3. Observe the state in User A's window.<br>4. Verify viewer count or room-leave notification if available.<br>5. Check CloudWatch Logs.<br>6. Check DynamoDB. |
| **Input Data**      | User B closes the tab, navigates away, or loses network connectivity.                                                                                                                                                                                              |
| **Expected Result** | User B is no longer treated as an active member of Room A; viewer count decreases from `2` to `1` if supported; User A receives the corresponding update event; User A's connection remains unaffected.                                                            |
| **Actual Result**   | Record both clients' states, viewer count, and the time at which the disconnection is detected.                                                                                                                                                                    |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                      |
| **Evidence**        | Screenshots before and after leaving, User A's frame, DynamoDB data, and CloudWatch Logs.                                                                                                                                                                          |

> When a device suddenly loses network connectivity, `$disconnect` may not be processed immediately. If the system uses heartbeat or TTL, the maximum time required to detect and clean up the connection should be clearly documented.

---

#### WS-10 — `$disconnect` Deletes or Deactivates the Connection

| Field               | Content                                                                                                                                                                                                                                                                                |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `WS-10`                                                                                                                                                                                                                                                                                |
| **Test Name**       | Clean up the connection record when `$disconnect` occurs                                                                                                                                                                                                                               |
| **Objective**       | Verify that the `$disconnect` Lambda deletes or marks the disconnected Connection ID as inactive.                                                                                                                                                                                      |
| **Preconditions**   | User B has an active Connection ID stored in DynamoDB.                                                                                                                                                                                                                                 |
| **Test Steps**      | 1. Record User B's connection record before disconnecting.<br>2. Close User B's WebSocket connection.<br>3. Check the `$disconnect` logs.<br>4. Read the DynamoDB record again.<br>5. Perform the next broadcast to Room A.<br>6. Verify the connection list used by Lambda Broadcast. |
| **Input Data**      | User B's active Connection ID.                                                                                                                                                                                                                                                         |
| **Expected Result** | `$disconnect` receives the correct Connection ID; the record is deleted or marked inactive according to the design; User B is no longer included in the broadcast list; cleanup is idempotent and does not fail if executed again.                                                     |
| **Actual Result**   | Record the connection record state before and after `$disconnect`.                                                                                                                                                                                                                     |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                          |
| **Evidence**        | `$disconnect` CloudWatch Logs, DynamoDB records before and after the operation, and logs from the next broadcast.                                                                                                                                                                      |

---

#### WS-11 — Expired Connection Does Not Cause the Entire Broadcast to Fail

| Field               | Content                                                                                                                                                                                                                                                                                       |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `WS-11`                                                                                                                                                                                                                                                                                       |
| **Test Name**       | Isolate failure caused by an expired connection                                                                                                                                                                                                                                               |
| **Objective**       | Verify that an invalid Connection ID does not prevent Lambda from sending data to other active connections.                                                                                                                                                                                   |
| **Preconditions**   | Room A contains at least one valid connection and one expired or non-existent Connection ID.                                                                                                                                                                                                  |
| **Test Steps**      | 1. Keep User A connected.<br>2. Create a condition where User B's outdated record remains in DynamoDB.<br>3. Publish an event to Room A.<br>4. Observe the message received by User A.<br>5. Verify the Lambda Broadcast result.<br>6. Check error logs for the outdated Connection ID.       |
| **Input Data**      | One valid Connection ID and one expired Connection ID in the same Room A.                                                                                                                                                                                                                     |
| **Expected Result** | User A still receives the event; Lambda handles errors individually for each Connection ID; one failed send does not terminate the whole broadcast loop; logs show at least one successful and one failed send; Lambda does not fail the entire operation because of one outdated connection. |
| **Actual Result**   | Record the number of target connections, successful sends, failed sends, and the result observed by User A.                                                                                                                                                                                   |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                 |
| **Evidence**        | User A's message, CloudWatch Logs, and the expired Connection ID record.                                                                                                                                                                                                                      |

---

#### WS-12 — `GoneException` Is Handled and Removed from DynamoDB

| Field               | Content                                                                                                                                                                                                                                                                                            |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `WS-12`                                                                                                                                                                                                                                                                                            |
| **Test Name**       | Clean up an outdated connection when the Management API returns `GoneException`                                                                                                                                                                                                                    |
| **Objective**       | Verify that Lambda Broadcast recognizes HTTP `410 Gone` and removes the invalid Connection ID.                                                                                                                                                                                                     |
| **Preconditions**   | DynamoDB still contains a Connection ID belonging to a closed connection; Lambda Broadcast has permission to delete or update the record.                                                                                                                                                          |
| **Test Steps**      | 1. Identify an expired Connection ID.<br>2. Confirm that the record still exists before the broadcast.<br>3. Trigger Lambda Broadcast.<br>4. Check logs for the `postToConnection` call.<br>5. Verify that `GoneException` is caught.<br>6. Read DynamoDB again.<br>7. Trigger a second broadcast. |
| **Input Data**      | Connection ID that no longer exists in API Gateway but still exists in DynamoDB.                                                                                                                                                                                                                   |
| **Expected Result** | The Management API returns an error equivalent to `410 Gone`; Lambda catches the error without terminating the entire broadcast; the outdated connection record is deleted or deactivated; the next broadcast no longer attempts to send to that Connection ID.                                    |
| **Actual Result**   | Record the error code, cleanup action, and record state after processing.                                                                                                                                                                                                                          |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                      |
| **Evidence**        | CloudWatch Logs containing `GoneException` or `410`, DynamoDB before and after cleanup, and logs from the next broadcast.                                                                                                                                                                          |

> Not every Management API error should be treated as an expired connection. Only errors that confirm the connection no longer exists, such as `GoneException`, should be used as a reason to delete the record.

---

#### WS-13 — Users Outside the Room Do Not Receive Private Room Data

| Field               | Content                                                                                                                                                                                                                                                                                                               |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `WS-13`                                                                                                                                                                                                                                                                                                               |
| **Test Name**       | Isolate data between auction rooms                                                                                                                                                                                                                                                                                    |
| **Objective**       | Verify that events from Room A are sent only to connections belonging to Room A.                                                                                                                                                                                                                                      |
| **Preconditions**   | User A and User B are in Room A; User C is in Room B.                                                                                                                                                                                                                                                                 |
| **Test Steps**      | 1. Open Room A as User A and User B.<br>2. Open Room B as User C.<br>3. Confirm that all three connections are active.<br>4. Publish an event belonging only to Room A.<br>5. Verify messages in all three windows.<br>6. Check the DynamoDB query used by Lambda Broadcast.<br>7. Repeat using an event from Room B. |
| **Input Data**      | Event specific to Auction Item A and event specific to Auction Item B.                                                                                                                                                                                                                                                |
| **Expected Result** | User A and User B receive the Room A event; User C does not receive it; User C only receives Room B events; Lambda queries connections using the correct Room ID; bid data, status, or private auction-session information is not sent across rooms.                                                                  |
| **Actual Result**   | Record which messages were or were not received in each window.                                                                                                                                                                                                                                                       |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                         |
| **Evidence**        | Screenshots of three windows or WebSocket frames, broadcast query/logs, and subscription data in DynamoDB.                                                                                                                                                                                                            |

---

#### Event Distribution Matrix to Verify

| User                 | Joined Room          | Room A Event                            | Room B Event                   |
| -------------------- | -------------------- | --------------------------------------- | ------------------------------ |
| User A               | Room A               | Must receive                            | Must not receive               |
| User B               | Room A               | Must receive                            | Must not receive               |
| User C               | Room B               | Must not receive                        | Must receive                   |
| Expired connection   | Room A               | Send fails and connection is cleaned up | Not applicable                 |
| User who left Room A | No longer subscribed | Must not receive                        | Receives only if joined Room B |

---

#### DynamoDB Connections Table Verification Requirements

For the connection management table, verify the following:

* Connection ID is obtained from `requestContext.connectionId`.
* User ID is obtained from verified identity information.
* Do not trust `userId`, `role`, or `connectionId` supplied by the client in a message.
* Each connection is associated with the correct User.
* Each subscription is associated with the correct Room ID or Auction Item ID.
* Two tabs opened by the same User may have two different Connection IDs.
* Closed connections are deleted or deactivated.
* TTL is configured correctly if automatic cleanup is used.
* Access Tokens, ID Tokens, and Refresh Tokens are not stored.
* One user's connection does not overwrite another user's connection.
* Duplicate subscriptions do not exist unless intentionally designed.
* `GoneException` results in cleanup of the corresponding Connection ID.
* Broadcast queries only the connections belonging to the target room.
* Failure of one connection does not prevent updates from reaching remaining connections.

If the table uses a single-table design, the team must verify the partition key, sort key, and indexes used to query connections by Room ID.

---

#### Lambda Broadcast Verification Requirements

Lambda Broadcast must be tested against the following criteria:

* Receives the correct Room ID and event type.
* Queries the correct list of connections for the room.
* Does not broadcast solely based on a list of Connection IDs supplied by the client.
* Sends data using the correct WebSocket API endpoint and stage.
* Sends valid JSON messages.
* Does not send sensitive data.
* Continues processing when sending to one connection fails.
* Catches and handles `GoneException`.
* Cleans up expired Connection IDs.
* Records the number of target connections.
* Records the number of successful and failed sends.
* Does not log complete tokens or unnecessary personal data.
* Includes a mechanism to limit message size.
* Does not send duplicate events beyond the intended design.
* Does not allow an error in one room to affect another room.

---

#### CloudWatch Logs Verification Requirements

CloudWatch Logs should contain sufficient information for tracing, including:

* Request ID.
* Route key such as `$connect`, `$disconnect`, `$default`, or `join_room`.
* Partially masked Connection ID when included in reports.
* Verified User ID or Cognito `sub`.
* Room ID or Auction Item ID.
* Message type or event type.
* Number of target connections.
* Number of successful sends.
* Number of failed sends.
* Error codes such as `INVALID_MESSAGE_FORMAT` or `GONE_CONNECTION`.
* Processing time.
* Result of cleaning up outdated connection records.

The logs must not contain:

* Access Token.
* ID Token.
* Refresh Token.
* Headers or query parameters containing authentication information.
* Passwords.
* AWS Access Key ID.
* AWS Secret Access Key.
* AWS Session Token.
* Entire WebSocket messages if they contain sensitive data.
* Stack traces in responses sent to the client.

---

#### Result Summary Table

| ID      | Test Content                    | Main Expected Result                          | DynamoDB Verification | Status     |
| ------- | ------------------------------- | --------------------------------------------- | --------------------- | ---------- |
| `WS-01` | User connects successfully      | Handshake `101`, frontend displays Live       | Yes                   | Not tested |
| `WS-02` | Invalid connection              | Rejected, no active connection                | Yes                   | Not tested |
| `WS-03` | `$connect` stores Connection ID | Connection record is created correctly        | Required              | Not tested |
| `WS-04` | User joins correct room         | Connection is associated with correct Room ID | Required              | Not tested |
| `WS-05` | Two Users join one item         | Both connections receive the event            | Required              | Not tested |
| `WS-06` | Send valid message              | Handler processes and responds correctly      | When data changes     | Not tested |
| `WS-07` | Invalid message format          | Rejected, no broadcast                        | Must remain unchanged | Not tested |
| `WS-08` | Broadcast status                | All room members receive the update           | Yes                   | Not tested |
| `WS-09` | User leaves page                | No longer treated as active connection        | Yes                   | Not tested |
| `WS-10` | `$disconnect` cleans connection | Delete or deactivate Connection ID            | Required              | Not tested |
| `WS-11` | Expired connection exists       | Valid connections still receive data          | Yes                   | Not tested |
| `WS-12` | Handle `GoneException`          | Old connection is removed from the table      | Required              | Not tested |
| `WS-13` | Room isolation                  | Users outside the room do not receive data    | Yes                   | Not tested |

---

#### Testing Evidence

Evidence for WebSocket testing includes:

* Two or three browser windows running simultaneously.
* Network tab showing the WebSocket handshake.
* Sent and received WebSocket frames.
* Live or Connected state on the frontend.
* Viewer count before and after Users join or leave the page.
* CloudWatch Logs for `$connect`.
* CloudWatch Logs for `$disconnect`.
* CloudWatch Logs for the WebSocket Handler.
* CloudWatch Logs for Lambda Broadcast.
* Connection ID records in DynamoDB.
* Subscription records linking Connection IDs and Room IDs.
* DynamoDB data before and after disconnection.
* Logs showing `GoneException` or HTTP `410`.
* Messages received in each browser window.
* Request ID used to correlate API Gateway and Lambda logs.

Each piece of evidence should clearly identify the corresponding test case, for example:

```text
Figure 5.5.5.1: Successful WebSocket handshake for test case WS-01
Figure 5.5.5.2: Connection ID stored in DynamoDB after WS-03
Figure 5.5.5.3: Two windows receiving the Room A event in WS-08
Figure 5.5.5.4: GoneException log and connection cleanup result in WS-12
Figure 5.5.5.5: User C does not receive the Room A event in WS-13
```

The following information must be masked in evidence screenshots:

* Tokens.
* Authentication information contained in the WebSocket URL.
* Authentication headers.
* AWS account ID if it does not need to be publicly disclosed.
* Full Connection ID if the document is shared publicly.
* Personal information unrelated to the test.

---

#### Evaluation Criteria

A test case may only be marked as `PASS` when:

* The actual result matches the expected result.
* The client receives the correct message or is correctly rejected.
* Connection IDs are stored and cleaned up correctly.
* Subscriptions belong to the correct auction room.
* Broadcast sends the correct content to the correct users in the correct room.
* Users outside the room do not receive the data.
* Expired connections do not cause the entire broadcast to fail.
* `GoneException` is handled and the outdated Connection ID is cleaned up.
* No unintended data changes occur.
* Direct evidence is available.

A test case must be marked as `FAIL` when:

* The frontend displays Live even though the Connection ID was not stored.
* `$connect` succeeds using an invalid token.
* One User's Connection ID overwrites another User's connection.
* A User joins Room A but is stored as belonging to Room B.
* A User in the room does not receive a valid event.
* A User who does not belong to the room receives private room data.
* An invalidly formatted message is still processed or broadcast.
* One expired connection causes Lambda to stop the entire broadcast.
* `GoneException` occurs but the outdated record is not cleaned up.
* `$disconnect` deletes the wrong connection.
* The frontend requires a page reload to receive data that is designed to be real-time.
* Logs or evidence expose tokens or authentication information.

A test case is marked as `BLOCKED` when:

* The WebSocket API has not been deployed.
* The route under test has not been configured.
* The Lambda Handler or Lambda Broadcast does not exist.
* The connections table or Room ID index has not been created.
* The frontend has not been integrated with WebSocket.
* Permission to inspect DynamoDB or CloudWatch Logs is unavailable.
* The corresponding functionality has not yet been implemented.
