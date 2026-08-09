### 5.5.1. Overview and Test Environment

#### Overview

After completing the deployment of AWS services in section **5.4**, the team performs system testing for the Live Auction to verify the end-to-end operation of the entire system in the AWS environment.

The testing process does not only validate the existence or individual configuration of resources, but focuses on verifying end-to-end business flows between the frontend, API Gateway, Lambda, Amazon SQS FIFO, DynamoDB, S3 and CloudFront.

Test results are recorded through:

- The User Frontend and Admin Frontend interfaces.
- HTTP responses from REST APIs.
- Messages received via WebSocket.
- Data stored in Amazon DynamoDB.
- Message states in Amazon SQS FIFO.
- Objects and object versions in Amazon S3.
- Lambda execution logs in Amazon CloudWatch Logs.
- Monitoring metrics on Amazon CloudWatch.

#### Test Objectives

The objectives of the system testing are:

- Verify the main functions of the Live Auction system work according to requirements.
- Verify authentication and authorization between regular users and administrators.
- Confirm REST APIs return correct data, HTTP status codes and error messages.
- Test WebSocket connectivity and real-time auction updates.
- Verify bid requests are enqueued into Amazon SQS FIFO and processed in order within the same message group.
- Check that auction session data, items and bid histories are correctly stored in Amazon DynamoDB.
- Confirm User Frontend and Admin Frontend are accessible through Amazon CloudFront.
- Test upload, storage and display of item images from Amazon S3.
- Test system behavior with invalid inputs, service errors and duplicate requests.
- Verify logs and monitoring metrics provide sufficient information for tracking and troubleshooting.
- Evaluate system readiness before production use.

#### Test Scope

The test scope includes the following functional groups:

1. Authentication and authorization via Amazon Cognito.
2. Business functions exposed via Amazon API Gateway REST.
3. Real-time data exchange through API Gateway WebSocket.
4. Business execution by AWS Lambda functions.
5. The end-to-end bidding flow of the Live Auction system.
6. Ordering and asynchronous processing using Amazon SQS FIFO.
7. Data integrity and consistency in Amazon DynamoDB.
8. Content storage and distribution via Amazon S3 and Amazon CloudFront.
9. Observability, logging and tracing using Amazon CloudWatch.
10. Error cases, invalid requests and unauthorized access behavior.
11. Operational availability of User Frontend and Admin Frontend on AWS.

Resource configuration checks via the AWS Management Console or Terraform are supplementary evidence only. System test results must be based on actual business flow behavior and generated data after tests are executed.

#### Architecture Under Test

The test architecture of the Live Auction system consists of two main flow groups.

##### REST API Flow

```text
User Frontend or Admin Frontend
        ↓
Amazon CloudFront
        ↓
Amazon API Gateway REST
        ↓
Amazon Cognito Authorizer
        ↓
AWS Lambda
        ↓
Amazon DynamoDB or Amazon S3
        ↓
Response to Frontend
```

This flow is used for functions such as:

- Registration and login.
- Listing auction sessions.
- Viewing session and item details.
- Managing auction sessions.
- Managing auction items.
- Uploading and retrieving images.
- Performing administrative functions.

##### WebSocket and Bidding Flow

```text
User Frontend
        ↓
API Gateway WebSocket
        ↓
Lambda la-ws-handler
        ↓
Amazon SQS FIFO
        ↓
Lambda la-bid-processor
        ↓
Amazon DynamoDB
        ↓
Lambda la-broadcast
        ↓
API Gateway WebSocket
        ↓
Update data on User Frontend
```

This flow is used to:

- Establish WebSocket connections.
- Join an auction room.
- Send bid requests.
- Process bid requests in order.
- Update current price and highest bidder.
- Persist bid history.
- Send results to users watching the auction session.

#### Test Environment

The system is tested on AWS infrastructure in the Region:

```text
Asia Pacific (Singapore) – ap-southeast-1
```

Test environment details are summarized as follows:

| Component                        | Test environment                                |
|----------------------------------|-------------------------------------------------|
| AWS Region                       | `ap-southeast-1`                                |
| User interface                   | User Frontend                                   |
| Admin interface                  | Admin Frontend                                  |
| Frontend distribution            | Amazon CloudFront                               |
| Frontend and image storage       | Amazon S3                                       |
| User authentication              | Amazon Cognito                                  |
| REST API                         | Amazon API Gateway REST                         |
| Real-time communication          | API Gateway WebSocket                           |
| Business processing              | AWS Lambda                                      |
| Bid queue processing             | Amazon SQS FIFO                                 |
| Data storage                     | Amazon DynamoDB                                 |
| Logging and monitoring           | Amazon CloudWatch                               |
| Test browser                     | Browser with JavaScript and WebSocket support   |
| Network connection               | Stable Internet connection                      |

CloudFront, API Gateway and WebSocket endpoints must be obtained from the actual deployment environment. When reporting, the team may redact parts of these addresses if not required to be public.

#### AWS Services Involved in Testing

| Service                      | Role in testing process                                                                 |
|-----------------------------|------------------------------------------------------------------------------------------|
| **Amazon Cognito**          | User authentication, token issuance and managing User/Admin groups.                      |
| **Amazon API Gateway REST** | Accept HTTP requests from User and Admin frontends.                                      |
| **API Gateway WebSocket**   | Maintain connections and transmit auction data in real time.                             |
| **AWS Lambda**              | Handle additional authentication, auction logic, bidding and broadcasting.                |
| **Amazon DynamoDB**         | Store users, auction sessions, items, WebSocket connections and bid history.             |
| **Amazon SQS FIFO**         | Receive and preserve order of bid requests within the same Message Group.                |
| **Amazon S3**               | Store User Frontend, Admin Frontend and auction item images.                              |
| **Amazon CloudFront**       | Distribute frontend and static content from private S3 buckets.                           |
| **Amazon CloudWatch**       | Store logs, track metrics and assist in tracing system behavior.                          |

#### Interfaces Under Test

The system has two main interfaces:

##### User Frontend

Used to test user-facing functions including:

- Registration and login.
- Viewing auction session lists.
- Viewing item details.
- Joining auctions.
- Establishing WebSocket connections.
- Sending bid requests.
- Receiving real-time price updates.
- Viewing item images and status.

##### Admin Frontend

Used to test administrative functions including:

- Admin login.
- Creating and updating auction sessions.
- Managing auction items.
- Managing item images.
- Starting or ending auction sessions as permitted.
- Accessing admin-only REST APIs.
- Verifying that a User account cannot perform Admin functions.

#### Test Accounts

Testing uses at least two account types:

| Account type | Purpose                                                                 |
|--------------|-------------------------------------------------------------------------|
| **User**     | Test login, view sessions, join WebSocket and place bids.              |
| **Admin**    | Test create, update and manage sessions or auction items.              |

In addition to the two valid account types, some tests may use:

- Unconfirmed accounts.
- Accounts with incorrect passwords.
- Accounts not in the Admin group.
- Invalid tokens.
- Expired tokens.
- Requests without tokens.

Test accounts must be created specifically for testing. Do not use personal or sensitive accounts.

#### Test Result Conventions

Each test case is evaluated with one of three statuses:

| Status     | Meaning                                                                                               |
|------------|-------------------------------------------------------------------------------------------------------|
| `PASS`     | Actual result fully matches the expected result.                                                      |
| `FAIL`     | Actual result does not match expected result or the system throws an error.                          |
| `BLOCKED`  | Test cannot be executed or completed due to missing resources, data, configuration or dependencies.    |

A test case is `PASS` only when there is supporting evidence, such as:

- Screenshots of the UI.
- HTTP status and response bodies.
- WebSocket messages.
- CloudWatch Logs.
- CloudWatch Metrics.
- Records in Amazon DynamoDB.
- Message state in Amazon SQS.
- Object metadata or versions in Amazon S3.

If a feature is not implemented or not configured, the test case must be marked `BLOCKED`. Do not mark `PASS` based on design or expected behavior alone.

#### Security Rules for Evidence

When capturing screenshots or collecting evidence, do not reveal sensitive information, including:

- Access Token.
- ID Token.
- Refresh Token.
- AWS Access Key ID.
- AWS Secret Access Key.
- AWS Session Token.
- User passwords.
- Cognito Client Secret.
- Login cookies.
- Authorization header.
- Presigned URLs that allow access.
- Environment variable contents that contain credentials.

Before including images or logs in reports, the team must:

1. Review all visible content.
2. Redact or remove tokens, passwords and credentials.
3. Do not capture `.env` files with real data.
4. Avoid including full request headers containing `Authorization`.
5. Avoid exposing sensitive CloudWatch Logs in reports.
6. Keep only the information necessary to prove test results.

Information such as API endpoints, Cognito User Pool IDs or App Client IDs are not always secret, but the team should limit exposure when documents are shared outside the project scope.
