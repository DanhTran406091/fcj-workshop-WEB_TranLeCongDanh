---
title: "End-to-End Bidding Flow Testing"
date: 2026-08-03
weight: 6
chapter: false
pre: " <b> 5.5.6. </b> "
---

#### Objective

This testing section verifies the complete real-time bidding flow of the Live Auction system, from the moment a user submits a request on the frontend until the new bid is processed, stored, and broadcast to users who are currently watching the auction.

The flow to be tested is:

```text
Frontend
→ WebSocket API
→ la-ws-handler
→ SQS FIFO
→ la-bid-processor
→ DynamoDB
→ la-broadcast
→ WebSocket API
→ Frontend
```

The testing must not only confirm that the interface displays the new bid price, but must also prove that:

* The bid request is authenticated.
* The message is placed into the correct SQS FIFO queue.
* Duplicate messages do not create multiple bids.
* Requests are processed in the correct order within the same auction.
* Price updates are atomic.
* The highest bidder is identified correctly.
* Bid history is stored completely.
* Results are broadcast to the correct auction room.
* The frontend updates without requiring a page reload.
* Tokens, authentication information, or internal infrastructure data are not exposed.

---

#### Related Components

The components that must be tested include:

* Auction item detail page on the frontend.
* API Gateway WebSocket API.
* WebSocket route used for bidding, for example `place_bid`.
* Lambda `la-ws-handler`.
* Amazon SQS FIFO.
* Dead-letter queue, if configured.
* Lambda `la-bid-processor`.
* DynamoDB Auction or Current Bid Table.
* DynamoDB Bid History Table.
* Lambda `la-broadcast`.
* API Gateway Management API.
* CloudWatch Logs.
* JWT authentication mechanism or Amazon Cognito.
* Authorization mechanism controlling participation in the auction.

---

#### General Testing Preconditions

Before testing begins, the system must meet the following conditions:

* The WebSocket API has been deployed.
* The frontend is configured with the correct WebSocket URL for the testing environment.
* The bidding route is connected to `la-ws-handler`.
* `la-ws-handler` has permission to send messages to SQS FIFO.
* SQS FIFO is configured correctly.
* `la-bid-processor` is configured to receive messages from SQS.
* `la-bid-processor` has permission to read and update data in DynamoDB.
* `la-broadcast` has permission to call `execute-api:ManageConnections`.
* Tables for storing the current bid and bid history already exist.
* The DynamoDB Connections or Subscriptions Table contains room data.
* At least two valid user accounts are available.
* At least one account that does not belong to the auction or is not allowed to bid is available.
* At least one not-yet-started auction, one active auction, and one ended auction are available.
* CloudWatch Logs are enabled.
* The testing environment is separated from production.
* Permission is available to inspect SQS, DynamoDB, and CloudWatch Logs.
* The testing device clock is synchronized.

If any component in the flow has not yet been implemented, the related test case must be marked as `BLOCKED`.

---

#### Test Data

| Data                  | Description                                                                       |
| --------------------- | --------------------------------------------------------------------------------- |
| User A                | Valid User who is allowed to place bids                                           |
| User B                | Second valid User participating in the same auction                               |
| User C                | Valid User who does not belong to or is not allowed to participate in the auction |
| Anonymous User        | User who is not logged in                                                         |
| Auction Active        | Auction currently in `ACTIVE` state                                               |
| Auction Scheduled     | Auction not yet started, in `SCHEDULED` state                                     |
| Auction Ended         | Auction already ended, in `ENDED` state                                           |
| Current Price         | Current item price, for example `1,000,000 VND`                                   |
| Minimum Increment     | Minimum bid increment, for example `100,000 VND`                                  |
| Valid Bid             | Valid bid, for example `1,100,000 VND`                                            |
| Low Bid               | Bid equal to or lower than the current price                                      |
| Invalid Increment Bid | Bid higher than current price but not meeting the minimum increment               |
| Request ID            | Unique identifier for a bid request                                               |
| Client Message ID     | Client-generated ID used to support idempotency                                   |
| Room ID               | WebSocket room identifier for the item                                            |
| Auction ID            | Auction identifier                                                                |
| Item ID               | Auction item identifier                                                           |
| Expired Token         | Expired token                                                                     |
| Invalid Token         | Token with invalid signature or incorrect format                                  |

Production data must not be used during testing.

---

#### Bid Request Structure

Example message sent by the frontend:

```json
{
  "action": "place_bid",
  "requestId": "bid-request-001",
  "auctionId": "auction-active-001",
  "itemId": "item-001",
  "amount": 1100000
}
```

The frontend must not be allowed to send or independently determine:

```json
{
  "userId": "trusted-user-id",
  "role": "ADMIN",
  "currentPrice": 1000000,
  "isWinner": true
}
```

The bidder's identity must be obtained from a verified token or authentication context.

The current price, auction state, minimum increment, and highest bidder must be read from a trusted server-side data source.

---

#### Successful Result Structure

Example broadcast event for a successful bid:

```json
{
  "type": "BID_ACCEPTED",
  "requestId": "bid-request-001",
  "auctionId": "auction-active-001",
  "itemId": "item-001",
  "data": {
    "amount": 1100000,
    "highestBidderId": "masked-user-id",
    "bidSequence": 15
  },
  "timestamp": "2026-08-09T12:00:00Z"
}
```

#### Failed Result Structure

```json
{
  "type": "BID_REJECTED",
  "requestId": "bid-request-001",
  "error": {
    "code": "MINIMUM_INCREMENT_NOT_MET",
    "message": "The bid does not meet the minimum increment"
  },
  "timestamp": "2026-08-09T12:00:00Z"
}
```

The response must not contain:

* Access Token.
* ID Token.
* Refresh Token.
* AWS credentials.
* Unnecessary internal SQS message content.
* DynamoDB table names.
* Stack traces.
* Another user's Connection ID.
* Email addresses or unnecessary personal data.

---

### BID-01 — Valid Bid

| Field               | Content                                                                                                                                                                                                                                                                                                                                              |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `BID-01`                                                                                                                                                                                                                                                                                                                                             |
| **Test Name**       | User places a valid bid during an active auction                                                                                                                                                                                                                                                                                                     |
| **Objective**       | Verify that a valid bid request passes through the entire flow and is stored successfully.                                                                                                                                                                                                                                                           |
| **Preconditions**   | Auction Active is running; User A is logged in and has joined the correct room; current price is `1,000,000 VND`; minimum increment is `100,000 VND`.                                                                                                                                                                                                |
| **Test Steps**      | 1. User A opens the auction page.<br>2. Enter `1,100,000 VND`.<br>3. Submit the bid request.<br>4. Check the WebSocket frame.<br>5. Check `la-ws-handler` logs.<br>6. Verify that the message is sent to SQS FIFO.<br>7. Verify that `la-bid-processor` processes the message.<br>8. Check DynamoDB.<br>9. Check the broadcast message and frontend. |
| **Expected Result** | The request is accepted; the message is placed into SQS exactly once; the current price is updated to `1,100,000 VND`; User A becomes the highest bidder; one bid history record is created; users in the room receive `BID_ACCEPTED`; the interface updates without reloading.                                                                      |
| **Actual Result**   | Record the Request ID, Message ID, price before and after, highest bidder, and processing time.                                                                                                                                                                                                                                                      |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                                                                                        |
| **Evidence**        | WebSocket frames, CloudWatch Logs, SQS metrics, DynamoDB before and after, and frontend interface.                                                                                                                                                                                                                                                   |

---

### BID-02 — Bid Equal to or Lower Than the Current Price

| Field               | Content                                                                                                                                                                                                                                     |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `BID-02`                                                                                                                                                                                                                                    |
| **Test Name**       | Reject a bid that is not higher than the current price                                                                                                                                                                                      |
| **Objective**       | Verify that the system does not accept a bid equal to or lower than the current price.                                                                                                                                                      |
| **Preconditions**   | Current price is `1,100,000 VND`; auction is active.                                                                                                                                                                                        |
| **Test Steps**      | 1. Submit a bid of `1,100,000 VND`.<br>2. Submit another bid of `1,000,000 VND` using a different Request ID.<br>3. Check the result of each request.<br>4. Check DynamoDB and bid history.<br>5. Check broadcast activity.                 |
| **Expected Result** | Both requests are rejected with an error code such as `BID_NOT_HIGHER_THAN_CURRENT_PRICE`; current price remains unchanged; highest bidder remains unchanged; no successful bid history record is created; `BID_ACCEPTED` is not broadcast. |
| **Actual Result**   | Record each submitted amount, error code, and price after testing.                                                                                                                                                                          |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                               |
| **Evidence**        | Error frame, CloudWatch Logs, and unchanged DynamoDB data.                                                                                                                                                                                  |

---

### BID-03 — Minimum Increment Not Met

| Field               | Content                                                                                                                                                                                                                                                          |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `BID-03`                                                                                                                                                                                                                                                         |
| **Test Name**       | Reject a bid that does not meet the minimum increment                                                                                                                                                                                                            |
| **Objective**       | Verify that a new bid must satisfy the minimum increment rule.                                                                                                                                                                                                   |
| **Preconditions**   | Current price is `1,100,000 VND`; minimum increment is `100,000 VND`.                                                                                                                                                                                            |
| **Test Steps**      | 1. User A submits `1,150,000 VND`.<br>2. Check the processing flow.<br>3. Check the response.<br>4. Check DynamoDB.<br>5. Check whether other users receive any update.                                                                                          |
| **Expected Result** | The request is rejected with `MINIMUM_INCREMENT_NOT_MET`; the server may return the minimum acceptable bid as `1,200,000 VND`; current price and highest bidder remain unchanged; no successful bid history record is created; no new price update is broadcast. |
| **Actual Result**   | Record the submitted amount, minimum acceptable bid, and error code.                                                                                                                                                                                             |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                    |
| **Evidence**        | WebSocket response, processor logs, and DynamoDB.                                                                                                                                                                                                                |

---

### BID-04 — Bid Placed Before Auction Starts

| Field               | Content                                                                                                                                                                                                                    |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `BID-04`                                                                                                                                                                                                                   |
| **Test Name**       | Reject a bid submitted before the auction start time                                                                                                                                                                       |
| **Objective**       | Verify that a `SCHEDULED` auction does not accept bids.                                                                                                                                                                    |
| **Preconditions**   | Auction Scheduled exists and the current time is earlier than `startTime`.                                                                                                                                                 |
| **Test Steps**      | 1. Open the auction that has not started.<br>2. Submit a numerically valid bid.<br>3. Check the response.<br>4. Check auction state and time in DynamoDB.<br>5. Check bid history.                                         |
| **Expected Result** | The request is rejected with `AUCTION_NOT_STARTED`; current price is not updated; highest bidder is not updated; no successful bid history record is created; frontend continues to show that the auction has not started. |
| **Actual Result**   | Record auction state, start time, server time, and error code.                                                                                                                                                             |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                              |
| **Evidence**        | WebSocket response, DynamoDB, and CloudWatch Logs.                                                                                                                                                                         |

---

### BID-05 — Bid Placed After Auction Ends

| Field               | Content                                                                                                                                                                                 |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `BID-05`                                                                                                                                                                                |
| **Test Name**       | Reject a bid after the auction has ended                                                                                                                                                |
| **Objective**       | Verify that an ended auction does not accept additional bids.                                                                                                                           |
| **Preconditions**   | Auction Ended is in `ENDED` state or the current time is later than `endTime`.                                                                                                          |
| **Test Steps**      | 1. Open the ended auction.<br>2. Submit a bid higher than the current price.<br>3. Check the response.<br>4. Check DynamoDB.<br>5. Check winner and bid history.                        |
| **Expected Result** | The request is rejected with `AUCTION_ENDED`; current price, winner, and highest bidder remain unchanged; no successful bid history record is created; `BID_ACCEPTED` is not broadcast. |
| **Actual Result**   | Record end time, server time, auction state, and error code.                                                                                                                            |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                           |
| **Evidence**        | Error frame, DynamoDB, and CloudWatch Logs.                                                                                                                                             |

---

### BID-06 — Invalid or Unauthenticated User

| Field               | Content                                                                                                                                                                                                                                     |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `BID-06`                                                                                                                                                                                                                                    |
| **Test Name**       | Reject an unauthenticated bid request                                                                                                                                                                                                       |
| **Objective**       | Verify that only authenticated users can place bids.                                                                                                                                                                                        |
| **Preconditions**   | The system has a WebSocket authentication mechanism.                                                                                                                                                                                        |
| **Test Steps**      | 1. Attempt to bid without logging in.<br>2. Attempt to bid with an expired token.<br>3. Attempt to bid with a token having an invalid signature.<br>4. Attempt to send a fake `userId` in the message.<br>5. Check SQS, DynamoDB, and logs. |
| **Expected Result** | The connection or request is rejected with `UNAUTHENTICATED` or an equivalent code; client-provided `userId` is not trusted; no valid bid is created; current price is not updated; no valid business message is created in SQS.            |
| **Actual Result**   | Record token type and the corresponding connection result or error code.                                                                                                                                                                    |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                               |
| **Evidence**        | Handshake or error frame, absence of a valid bid in SQS, unchanged DynamoDB, and logs with tokens masked.                                                                                                                                   |

---

### BID-07 — User Is Not a Participant in the Auction

| Field               | Content                                                                                                                                                                                                              |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `BID-07`                                                                                                                                                                                                             |
| **Test Name**       | Reject a User who is not authorized to participate in the auction                                                                                                                                                    |
| **Objective**       | Verify that User C cannot bid in an auction in which they are not allowed to participate.                                                                                                                            |
| **Preconditions**   | User C is authenticated but has no membership, registration, or permission to participate in Auction Active.                                                                                                         |
| **Test Steps**      | 1. Log in as User C.<br>2. Submit a bid request to Auction Active.<br>3. Check the authorization result.<br>4. Check SQS and processor logs.<br>5. Check DynamoDB and broadcast activity.                            |
| **Expected Result** | The request is rejected with `NOT_AUCTION_PARTICIPANT` or `FORBIDDEN`; current price is not updated; no successful bid history record is created; highest bidder is unchanged; no successful bid event is broadcast. |
| **Actual Result**   | Record the masked User ID, Auction ID, and error code.                                                                                                                                                               |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                        |
| **Evidence**        | Error frame, authorization logs, and unchanged DynamoDB data.                                                                                                                                                        |

---

### BID-08 — Resubmitting the Same Bid Request

| Field               | Content                                                                                                                                                                                                                                                             |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `BID-08`                                                                                                                                                                                                                                                            |
| **Test Name**       | Handle idempotency and duplicate messages                                                                                                                                                                                                                           |
| **Objective**       | Verify that the same request does not create multiple bids when resent or redelivered by SQS.                                                                                                                                                                       |
| **Preconditions**   | A `requestId` or `clientMessageId` is available; an idempotency mechanism has been implemented.                                                                                                                                                                     |
| **Test Steps**      | 1. Submit a valid bid using `requestId=bid-request-008`.<br>2. Wait until the request is processed successfully.<br>3. Resubmit the exact same request.<br>4. If possible, simulate SQS redelivery.<br>5. Check bid history, current price, and broadcast activity. |
| **Expected Result** | Only one bid is applied; only one business history record is created; current price is updated only once; the duplicate request receives the previous result again or `DUPLICATE_REQUEST`; no duplicate business broadcast is created beyond the intended design.   |
| **Actual Result**   | Record the number of submissions, processor executions, bid history records, and broadcast events.                                                                                                                                                                  |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                       |
| **Evidence**        | Two sent frames, idempotency logs, SQS message attributes, DynamoDB, and broadcast logs.                                                                                                                                                                            |

> The system must not rely solely on SQS FIFO deduplication. `la-bid-processor` still requires an idempotency mechanism because messages may be redelivered after the visibility timeout expires or when Lambda completes processing but the batch completion has not yet been acknowledged.

---

### BID-09 — Multiple Users Place Bids Sequentially

| Field               | Content                                                                                                                                                                                                                                                         |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `BID-09`                                                                                                                                                                                                                                                        |
| **Test Name**       | Process a sequence of bids from multiple Users                                                                                                                                                                                                                  |
| **Objective**       | Verify that consecutive valid bids are processed in the correct order.                                                                                                                                                                                          |
| **Preconditions**   | User A and User B are watching the same auction; current price is `1,000,000 VND`; minimum increment is `100,000 VND`.                                                                                                                                          |
| **Test Steps**      | 1. User A bids `1,100,000 VND`.<br>2. Wait for a successful result.<br>3. User B bids `1,200,000 VND`.<br>4. User A bids `1,300,000 VND`.<br>5. Check message order in SQS FIFO.<br>6. Check DynamoDB and broadcast events.                                     |
| **Expected Result** | Bids are processed in the correct order within the same Auction/Item message group; prices become `1,100,000`, `1,200,000`, and `1,300,000 VND` in sequence; highest bidder changes A → B → A; bid history contains exactly three records in the correct order. |
| **Actual Result**   | Record sequence, User, amount, timestamp, and result of each bid.                                                                                                                                                                                               |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                   |
| **Evidence**        | WebSocket frames from both Users, logs, SQS attributes, and DynamoDB bid history.                                                                                                                                                                               |

---

### BID-10 — Highest Bidder Is Updated Correctly

| Field               | Content                                                                                                                                                                                                                                               |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `BID-10`                                                                                                                                                                                                                                              |
| **Test Name**       | Correctly update the highest bidder                                                                                                                                                                                                                   |
| **Objective**       | Verify that the highest bidder always corresponds to the highest valid accepted bid.                                                                                                                                                                  |
| **Preconditions**   | Multiple bids have been submitted by User A and User B.                                                                                                                                                                                               |
| **Test Steps**      | 1. Record the initial highest bidder.<br>2. User A submits a valid bid.<br>3. Check the highest bidder.<br>4. User B submits a higher bid.<br>5. Check the highest bidder again.<br>6. Submit an invalid bid from User A.<br>7. Check the final data. |
| **Expected Result** | The highest bidder changes only when a new bid is accepted; a rejected bid does not change the highest bidder; `highestBidAmount`, `highestBidderId`, and bid history all refer to the same result.                                                   |
| **Actual Result**   | Record the highest bidder and amount after each step.                                                                                                                                                                                                 |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                         |
| **Evidence**        | DynamoDB before and after, processor logs, and broadcast messages.                                                                                                                                                                                    |

---

### BID-11 — Bid History Is Stored Correctly

| Field               | Content                                                                                                                                                                                                                                                                        |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Test Case ID**    | `BID-11`                                                                                                                                                                                                                                                                       |
| **Test Name**       | Store complete and accurate bid history                                                                                                                                                                                                                                        |
| **Objective**       | Verify that each successful bid creates one traceable history record.                                                                                                                                                                                                          |
| **Preconditions**   | At least three successful bids exist in the same auction.                                                                                                                                                                                                                      |
| **Test Steps**      | 1. Perform several valid bids.<br>2. Query history by Auction ID or Item ID.<br>3. Verify User ID, amount, timestamp, Request ID, and sequence.<br>4. Verify display order.<br>5. Compare with processor logs and current price.                                               |
| **Expected Result** | Each accepted bid has exactly one record; amount and User ID are correct; history ordering is correct; Request IDs are not duplicated except for idempotent requests; the latest bid matches current price and highest bidder; rejected bids do not appear as successful bids. |
| **Actual Result**   | Record number of bids submitted, accepted bids, and history records.                                                                                                                                                                                                           |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                  |
| **Evidence**        | DynamoDB query results, CloudWatch Logs, and frontend-displayed data.                                                                                                                                                                                                          |

---

### BID-12 — Broadcast to All Watching Users

| Field               | Content                                                                                                                                                                                                                                    |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Test Case ID**    | `BID-12`                                                                                                                                                                                                                                   |
| **Test Name**       | Broadcast bidding results to the correct room                                                                                                                                                                                              |
| **Objective**       | Verify that every active connection in the correct room receives the bid update result.                                                                                                                                                    |
| **Preconditions**   | User A and User B are watching Room A; User C is watching Room B.                                                                                                                                                                          |
| **Test Steps**      | 1. Open Room A as User A and User B.<br>2. Open Room B as User C.<br>3. User A submits a valid bid in Room A.<br>4. Observe messages in all three windows.<br>5. Check `la-broadcast` logs.<br>6. Check the queried connection list.       |
| **Expected Result** | User A and User B receive `BID_ACCEPTED` for Room A; User C does not receive the event; the message contains the correct Auction ID, Item ID, amount, and sequence; failure of one connection does not cause the entire broadcast to fail. |
| **Actual Result**   | Record which messages were received or not received by each User.                                                                                                                                                                          |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                              |
| **Evidence**        | Three browser windows, WebSocket frames, DynamoDB subscriptions, and broadcast logs.                                                                                                                                                       |

---

### BID-13 — Frontend Updates Price Without Reloading the Page

| Field               | Content                                                                                                                                                                                                                                                                       |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | `BID-13`                                                                                                                                                                                                                                                                      |
| **Test Name**       | Real-time interface update                                                                                                                                                                                                                                                    |
| **Objective**       | Verify that the frontend processes broadcast events and immediately updates displayed data.                                                                                                                                                                                   |
| **Preconditions**   | User A and User B have the same page open; WebSocket is in Connected or Live state.                                                                                                                                                                                           |
| **Test Steps**      | 1. Record the displayed price in both windows.<br>2. User A submits a valid bid.<br>3. Do not reload the page.<br>4. Observe the new price in both windows.<br>5. Check highest bidder, bid history, and interface notification.<br>6. Check browser console and Network tab. |
| **Expected Result** | Both windows display the new price without reloading; highest bidder and bid history are updated according to the design; no duplicate UI entry is created; no JavaScript errors occur; displayed price matches DynamoDB and the broadcast message.                           |
| **Actual Result**   | Record price before, price after, update time, and UI state.                                                                                                                                                                                                                  |
| **Status**          | `PASS`, `FAIL`, or `BLOCKED`.                                                                                                                                                                                                                                                 |
| **Evidence**        | Video or before-and-after screenshots, WebSocket frame, browser console, and DynamoDB.                                                                                                                                                                                        |

---

### Data Verification Matrix After Each Request Type

| Scenario                        | Current Price    | Highest Bidder   | Bid History                    | Successful Broadcast                 |
| ------------------------------- | ---------------- | ---------------- | ------------------------------ | ------------------------------------ |
| Valid bid                       | Must update      | Must update      | Add exactly one record         | Yes                                  |
| Bid equal to/lower than current | No change        | No change        | No successful bid record added | No                                   |
| Minimum increment not met       | No change        | No change        | No successful bid record added | No                                   |
| Auction not started             | No change        | No change        | No successful bid record added | No                                   |
| Auction ended                   | No change        | No change        | No successful bid record added | No                                   |
| Unauthenticated User            | No change        | No change        | No record added                | No                                   |
| Unauthorized User               | No change        | No change        | No successful bid record added | No                                   |
| Duplicate request               | Update only once | Update only once | Only one record                | No duplicate broadcast beyond design |

---

### SQS FIFO Verification Requirements

SQS FIFO must be verified against the following criteria:

* Messages are sent to the correct queue.
* `MessageGroupId` is determined by Auction ID or Item ID.
* Bids for the same item are processed in order.
* A single shared `MessageGroupId` should not be used for the entire system if this would cause unrelated auctions to block one another.
* `MessageDeduplicationId` or content-based deduplication is configured correctly.
* Messages contain a Request ID to support tracing and idempotency.
* Messages do not contain access tokens.
* Messages do not trust User IDs supplied by the client.
* User ID in internal messages must come from verified authentication context.
* Failed messages are retried according to configuration.
* Messages that exceed the allowed number of processing attempts are moved to a DLQ if configured.
* Messages are not deleted before business processing completes.
* Partial batch failure handling is implemented if Lambda consumes messages in batches.

---

### DynamoDB Concurrent Update Verification Requirements

`la-bid-processor` must use conditional writes, transactions, or an equivalent concurrency-control mechanism.

At minimum, the following update conditions must be verified:

* The auction is still in `ACTIVE` state.
* Server time is still within the auction period.
* The new bid is higher than the current price.
* The new bid meets the minimum increment.
* The version or current price has not already been changed by another request.
* The Request ID has not already been processed.

The system should not be implemented as:

```text
Read current price
→ validate in Lambda
→ perform unconditional update
```

This approach may allow a lower bid to overwrite a higher bid when two Lambda executions run concurrently.

Updating the current price and writing bid history must remain consistent. If one operation succeeds while the other fails, the system must use a transaction or a clearly defined recovery mechanism.

---

### CloudWatch Logs Verification Requirements

Logs must contain sufficient information for tracing:

* Request ID.
* Lambda Request ID.
* WebSocket route key.
* Auction ID.
* Item ID.
* Verified User ID.
* Bid amount.
* SQS Message ID.
* Message Group ID.
* Receive count.
* Idempotency result.
* Price before and after update.
* Conditional write result.
* Broadcast event type.
* Number of target connections.
* Number of successful and failed broadcasts.
* Processing time for each component.
* Error code when a bid is rejected.

Logs must not contain:

* Access Token.
* ID Token.
* Refresh Token.
* Authorization header.
* Passwords.
* AWS credentials.
* Complete personal data.
* Stack traces in responses sent to the frontend.

---

### Result Summary Table

| ID       | Test Content                       | Main Expected Result                           | Status     |
| -------- | ---------------------------------- | ---------------------------------------------- | ---------- |
| `BID-01` | Valid bid                          | Price, highest bidder, and history are updated | Not tested |
| `BID-02` | Bid equal to or lower than current | Rejected, data remains unchanged               | Not tested |
| `BID-03` | Minimum increment not met          | Rejected with the appropriate error            | Not tested |
| `BID-04` | Auction not started                | Bid is not accepted                            | Not tested |
| `BID-05` | Auction ended                      | Bid is not accepted                            | Not tested |
| `BID-06` | Invalid User                       | Authentication is rejected                     | Not tested |
| `BID-07` | User not part of auction           | Authorization is rejected                      | Not tested |
| `BID-08` | Resubmit same request              | Processed only once                            | Not tested |
| `BID-09` | Multiple Users bid sequentially    | Processed in correct order                     | Not tested |
| `BID-10` | Update highest bidder              | Highest bidder always remains correct          | Not tested |
| `BID-11` | Store bid history                  | Complete, correct, and non-duplicated          | Not tested |
| `BID-12` | Broadcast result                   | Correct User, correct Room                     | Not tested |
| `BID-13` | Update frontend                    | New price displayed without reload             | Not tested |

---

### Testing Evidence

Evidence should include:

* WebSocket frame sending the bid request.
* WebSocket frame returning the result.
* Two or three browser windows running simultaneously.
* Displayed price before and after bidding.
* Bid history on the frontend.
* CloudWatch Logs for `la-ws-handler`.
* SQS message metadata with sensitive information masked.
* CloudWatch Logs for `la-bid-processor`.
* DynamoDB current price before and after.
* DynamoDB highest bidder before and after.
* Bid history records.
* CloudWatch Logs for `la-broadcast`.
* Result of duplicate-request handling.
* Conditional write result under concurrency.
* Request ID used to correlate the complete flow.

Example naming convention:

```text
Figure 5.5.6.1: User A submits a valid bid request in BID-01
Figure 5.5.6.2: Message processed by la-bid-processor
Figure 5.5.6.3: Price and highest bidder updated in DynamoDB
Figure 5.5.6.4: Two Users receive the BID_ACCEPTED event in BID-12
Figure 5.5.6.5: Frontend displays the new price without requiring a page reload
Figure 5.5.6.6: Duplicate request creates only one bid history record in BID-08
```

---

### Evaluation Criteria

A test case may only be marked as `PASS` when:

* The actual result matches the expected result.
* A valid request passes through the complete flow.
* An invalid request does not modify data.
* Current price is updated correctly.
* Highest bidder is identified correctly.
* Bid history is stored correctly and contains no duplicates.
* Duplicate requests are applied only once.
* Bid ordering within the same auction is preserved.
* Broadcast is sent only to the correct room.
* The frontend updates without requiring a page reload.
* Logs and evidence contain no tokens or sensitive data.

A test case must be marked as `FAIL` when:

* An invalid bid still changes the price.
* A valid bid is not stored.
* A lower bid overwrites a higher bid.
* Highest bidder does not match the highest accepted bid.
* One duplicate request creates two history records.
* Bid history and current price are inconsistent.
* An auction that has not started or has already ended still accepts bids.
* An unauthenticated or unauthorized User can place a bid.
* A User outside the room receives data belonging to another room.
* Failure of one connection causes the entire broadcast to fail.
* The frontend only updates after the page is reloaded.
* Logs or evidence expose tokens.

A test case is marked as `BLOCKED` when:

* The bidding route has not been implemented.
* `la-ws-handler` does not yet send messages to SQS.
* SQS FIFO does not exist.
* `la-bid-processor` is not connected to SQS.
* DynamoDB does not yet have a structure for storing the current bid or bid history.
* The idempotency mechanism has not been implemented.
* `la-broadcast` does not exist.
* The frontend does not yet handle bid events.
* Permission is unavailable to inspect SQS, DynamoDB, or CloudWatch Logs.
