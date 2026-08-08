---
title: "Worklog Week 5"
date: 2026-07-20
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Duration:

**From July 20, 2026 to July 24, 2026**

### Objectives for Week 5:

* Begin developing the core features of the Live Auction system.
* Build the user interface and basic management features.
* Develop APIs and business logic for the system.
* Design data structures for products, auctions, and bids.
* Research methods for integrating the application with AWS services.
* Track progress and integrate source code from team members.

### Tasks completed:

| Day | Date | Tasks | Reference |
| --- | --- | --- | --- |
| Monday | 20/07/2026 | Finalized the source-code structure, development conventions, and task assignments; prepared frontend and backend development environments. | Team technical documentation |
| Tuesday | 21/07/2026 | Developed the React/Vite user interface, including authentication, product listing, product details, and auction listing pages. | <https://react.dev/learn> |
| Wednesday | 22/07/2026 | Developed APIs and business logic for user, product, and auction management; validated input data and API responses. | <https://fastapi.tiangolo.com/tutorial/> |
| Thursday | 23/07/2026 | Designed data structures for users, products, auctions, and bid history; researched how these structures could be migrated to Amazon DynamoDB. | <https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html> |
| Friday | 24/07/2026 | Integrated the completed features; tested the flow between the frontend and APIs; documented errors and features requiring further development. | Team source code and testing documentation |

### Results achieved in Week 5:

* Created the initial source-code structure for the Live Auction system.
* Established the development and source-code integration workflow.
* Completed the initial interfaces for several core features:

  * User registration and authentication.
  * Product listing.
  * Product details.
  * Auction listing.
  * Auction product management.
  * Auction status monitoring.

* Developed initial APIs for:

  * User management.
  * Product management.
  * Auction management.
  * Bid request processing.
  * Bid history retrieval.

* Designed the main data entities:

  * User information.
  * Product information.
  * Auction information.
  * Auction status.
  * Bid prices and bid history.

* Established the initial connection between the frontend and backend APIs.
* Tested the basic request flow from the user interface to the business logic.
* Researched how backend functions could be migrated to AWS Lambda.
* Researched how real-time auction data could be stored in Amazon DynamoDB.
* Identified the need for API Gateway WebSocket to deliver real-time updates.
* Documented the issues and unfinished features for the following week.