---
title: "Worklog Week 4"
date: 2026-07-13
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Duration:

**From July 13, 2026 to July 17, 2026**

### Objectives for Week 4:

* Select and define the scope of the team project.
* Analyze the online auction problem and system requirements.
* Identify the core features of the Live Auction platform.
* Research an AWS architecture suitable for real-time processing.
* Select the AWS services expected to be used in the project.
* Prepare the project proposal and implementation plan.

### Tasks completed:

| Day | Date | Tasks | Reference |
| --- | --- | --- | --- |
| Monday | 13/07/2026 | Discussed and selected the **Live Auction Platform on AWS** project; identified its initial objectives and scope. | Project requirements |
| Tuesday | 14/07/2026 | Analyzed the online auction problem; identified user roles and core features such as registration, authentication, product management, auction creation, and bidding. | Team requirement analysis |
| Wednesday | 15/07/2026 | Analyzed issues involving real-time price updates, concurrent bids, bid ordering, system availability, and scalability. | <https://aws.amazon.com/architecture/> |
| Thursday | 16/07/2026 | Researched the proposed architecture and the roles of Amazon S3, CloudFront, Cognito, API Gateway, Lambda, DynamoDB, and SQS FIFO. | <https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html> |
| Friday | 17/07/2026 | Prepared the proposed architecture diagram; selected Terraform as the infrastructure deployment tool, assigned tasks, and completed the project proposal. | <https://developer.hashicorp.com/terraform/tutorials/aws-get-started> |

### Results achieved in Week 4:

* Selected the **Live Auction Platform on AWS** as the team project.
* Defined the main objectives and scope of the project.
* Identified the primary capabilities of the platform:

  * User account management.
  * Auction product management.
  * Auction session creation and management.
  * Bid request processing.
  * Real-time auction status updates.
  * Scalability based on user traffic.

* Identified the main problems that the system must address:

  * Multiple users may place bids at the same time.
  * Bid requests must be processed in the correct order.
  * Auction status must be updated promptly.
  * User information and system resources must be protected.
  * The system must be scalable and maintainable.

* Completed the initial proposed architecture.
* Identified the expected roles of the AWS services:

  * **Amazon Cognito:** user authentication and management.
  * **Amazon S3:** frontend and static resource storage.
  * **Amazon CloudFront:** content distribution.
  * **Amazon API Gateway:** API request handling.
  * **API Gateway WebSocket:** real-time auction updates.
  * **AWS Lambda:** serverless business logic processing.
  * **Amazon DynamoDB:** low-latency and real-time data storage.
  * **Amazon SQS FIFO:** ordered bid request processing.
  * **AWS IAM:** access control between AWS services.

* Selected Terraform to manage and deploy AWS infrastructure as code.
* Completed the proposal as the foundation for the next stages.