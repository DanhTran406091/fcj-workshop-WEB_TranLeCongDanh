---
title: "Worklog Week 7"
date: 2026-08-03
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Duration:

**From August 3, 2026 to August 7, 2026**

### Objectives for Week 7:

* Test the complete Live Auction system deployed on AWS.
* Verify the connection between the frontend and backend services.
* Test user registration, authentication, and authorization.
* Test auction functionality and real-time updates.
* Identify and resolve outstanding issues.
* Review the Terraform configuration, IAM permissions, and AWS resources.
* Conduct a final system review before completing the project.

### Tasks completed:

| Day | Date | Tasks | Reference |
| --- | --- | --- | --- |
| Monday | 03/08/2026 | Prepared system test cases; tested the user interface, navigation, and frontend access through Amazon CloudFront. | Team testing documentation |
| Tuesday | 04/08/2026 | Tested user registration, authentication, login, and authorization through Amazon Cognito; reviewed IAM Role permissions. | <https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools.html> |
| Wednesday | 05/08/2026 | Tested REST APIs and AWS Lambda functions; verified product and auction management, input validation, and data storage in DynamoDB. | <https://docs.aws.amazon.com/lambda/latest/dg/testing-guide.html> |
| Thursday | 06/08/2026 | Tested WebSocket connections and real-time bidding; verified the ordered processing of bid requests through Amazon SQS FIFO. | <https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-websocket-api.html> |
| Friday | 07/08/2026 | Fixed issues discovered during testing; reviewed Terraform resources, performed the final system walkthrough, and evaluated project completion. | Team source code and deployment documentation |

### Results achieved in Week 7:

* Prepared test cases for the main system features.
* Verified frontend access through Amazon CloudFront.
* Tested the main user authentication flows:

  * User registration.
  * Account confirmation.
  * Login.
  * Logout.
  * Password recovery.
  * Access control for protected features.

* Tested the core system features:

  * Displaying the product list.
  * Viewing product details.
  * Creating and updating products.
  * Creating and managing auctions.
  * Joining an auction.
  * Submitting bids.
  * Viewing auction status and bid history.

* Verified the connections between:

  * Frontend and Amazon Cognito.
  * Frontend and Amazon API Gateway.
  * API Gateway and AWS Lambda.
  * AWS Lambda and Amazon DynamoDB.
  * AWS Lambda and Amazon SQS FIFO.
  * API Gateway WebSocket and connected users.

* Verified real-time auction status updates.
* Verified the bid request processing flow through SQS FIFO.
* Identified and resolved several types of issues:

  * Environment variable configuration errors.
  * CORS errors when the frontend called APIs.
  * Permission errors between AWS services.
  * Lambda input processing errors.
  * WebSocket connection and disconnection errors.
  * User-interface status display errors.
  * Terraform resource configuration errors.

* Reviewed IAM Policies according to the principle of least privilege.
* Reviewed AWS resource status and system logs.
* Completed the main system walkthrough from user authentication to auction participation and bidding.
* Identified the remaining documentation and submission tasks for Week 8.