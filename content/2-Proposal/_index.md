---
title: "Proposal"
date: 2026-07-13
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# LIVE AUCTION PLATFORM ON AWS

## An online auction platform deployed on AWS Cloud

### 1. Executive Summary

**Live Auction** is an online auction platform that allows users to list products, follow auction sessions, and place bids in real time. The system aims to provide a transparent and convenient auction environment capable of serving multiple concurrent users.

The frontend is developed with **React/Vite**, while the backend uses **FastAPI and Python**. Application data is managed with **MySQL**. During the initial deployment phase, the team uses Amazon S3 to host the frontend and product images, Amazon EC2 to run the backend, Amazon RDS for MySQL to manage the database, and AWS Lambda for selected background tasks.

In addition to the initial deployment architecture, the team designed an extended AWS architecture to study real-time auction processing, high availability, data security, scalability, and disaster recovery.

### 2. Problem Statement

#### Current Problems

An online auction system must process multiple operations that may occur almost simultaneously, especially when several users place bids on the same item. Without proper processing, the following problems may occur:

- Bid information may not be updated promptly.
- Concurrent bids may be processed in the wrong order.
- Users may not receive the latest auction status.
- Product images and static resources may load slowly.
- The system may become unavailable when the server fails.
- Errors may be difficult to monitor and troubleshoot.
- User information and auction data may not be adequately protected.

Furthermore, deploying all components on a single server reduces scalability and creates a single point of failure.

#### Proposed Solution

The team proposes deploying the Live Auction system on AWS in multiple phases.

In the initial phase, the React/Vite frontend is built and deployed using **Amazon S3 Static Website Hosting**. The FastAPI backend is containerized with Docker and hosted on **Amazon EC2**. The local MySQL database is migrated from Docker to **Amazon RDS for MySQL**, while product images are stored in a separate S3 bucket. **AWS Lambda** is used for selected background or event-driven tasks.

In the extended architecture, the system may incorporate Amazon CloudFront, AWS WAF, Amazon API Gateway, Amazon Cognito, Amazon DynamoDB, Amazon SQS, Amazon EventBridge, and AWS monitoring services. This architecture is designed to process bids sequentially, improve scalability, strengthen security, and support multi-region disaster recovery.

#### Benefits

The proposed solution provides the following benefits:

- Users can participate in auctions remotely through a web browser.
- Auction information can be updated promptly.
- The frontend, backend, database, and image storage are separated.
- The system is less dependent on a single server.
- Resources can be scaled when the number of users increases.
- Security, monitoring, and data backup capabilities are improved.
- Team members gain practical experience deploying an application on AWS.

### 3. Solution Architecture

#### 3.1. Initial Deployment Architecture

The initial system architecture includes the following components:

1. Users access the Live Auction website through a web browser.
2. The React/Vite frontend is built into HTML, CSS, and JavaScript files.
3. The frontend files are deployed to Amazon S3.
4. The frontend sends requests to the FastAPI backend through REST APIs.
5. The backend is containerized with Docker and hosted on Amazon EC2.
6. The backend reads and writes data through Amazon RDS for MySQL.
7. Product images are stored in a separate Amazon S3 bucket.
8. AWS Lambda handles selected background or scheduled tasks.
9. Amazon CloudWatch supports resource monitoring and log collection.

This architecture is suitable for the current project scope because it is relatively straightforward to deploy, test, and maintain.

#### 3.2. Proposed Extended AWS Architecture

The following diagram illustrates the extended AWS architecture proposed for the Live Auction platform. It focuses on real-time auction processing, scalability, security, monitoring, high availability, and multi-region disaster recovery.

Click the diagram to open and view it at full size.

[![Proposed AWS architecture for the Live Auction platform](/images/2-Proposal/live-auction-proposed-architecture.svg)](/images/2-Proposal/live-auction-proposed-architecture.svg)

> **Note:** The diagram represents the proposed target architecture. Some advanced components have not yet been fully implemented in the current version of the project.

#### 3.3. General System Flow

The proposed architecture operates according to the following flow:

1. A user accesses the system through its domain name.
2. Amazon Route 53 routes the request to Amazon CloudFront.
3. AWS WAF inspects and filters potentially invalid requests.
4. CloudFront distributes the frontend stored in Amazon S3.
5. Users authenticate before accessing protected functions.
6. REST APIs process account, product, and auction operations.
7. WebSocket connections deliver new bids and auction updates to connected users.
8. Bid requests are placed in a queue for sequential processing.
9. Auction data is stored in an appropriate database.
10. Product images and audit records are stored in Amazon S3.
11. Amazon CloudWatch and AWS CloudTrail monitor system activities.
12. If the primary region becomes unavailable, Route 53 can route traffic to the standby region.

### 4. Proposed AWS Services

#### Amazon Route 53

Amazon Route 53 manages the system domain name and routes users to the application. In the extended architecture, Route 53 can perform health checks and redirect traffic to a standby region when necessary.

#### Amazon CloudFront

Amazon CloudFront distributes HTML, CSS, JavaScript, and image files through a global content delivery network. It reduces website loading time and decreases the number of requests sent directly to Amazon S3.

#### AWS WAF

AWS WAF filters suspicious requests, applies rate-based rules, and protects the web application from common web attacks.

#### Amazon S3

Amazon S3 is used to:

- Host the built React/Vite frontend.
- Store product images.
- Store logs or audit records.
- Replicate important data to a standby region when required.

#### Amazon EC2

During the initial deployment phase, Amazon EC2 hosts the FastAPI backend. The backend is packaged as a Docker container to provide a consistent runtime environment.

#### Amazon ECS and AWS Fargate

In the extended architecture, the backend can be migrated from EC2 to Amazon ECS with AWS Fargate. This approach simplifies container management and supports horizontal scaling based on application demand.

#### Amazon ECR

Amazon ECR stores the backend Docker images. A deployment pipeline can build a new image, push it to ECR, and update the running backend service.

#### Elastic Load Balancing

An Application Load Balancer distributes requests across backend containers, performs health checks, and reduces dependency on a single server.

#### Amazon API Gateway

Amazon API Gateway provides endpoints for REST and WebSocket APIs. REST APIs process business operations, while WebSocket APIs deliver real-time auction information to connected users.

#### AWS Lambda

AWS Lambda can be used to:

- Process selected independent APIs.
- Verify and update auction status.
- Process bid commands.
- Close auction sessions at their scheduled end time.
- Send notifications to users.
- Execute scheduled or event-driven background tasks.

#### Amazon RDS for MySQL

In the initial implementation, Amazon RDS for MySQL stores relational data such as:

- User accounts.
- Product information.
- Auction sessions.
- Bid history.
- Payment and transaction information.

Amazon RDS reduces database administration effort and supports automated data backup.

#### Amazon Aurora

In a target architecture requiring greater scalability and multi-region disaster recovery, Amazon Aurora may be evaluated as a replacement for the initial relational database. It is an extended architectural component and is not the database currently used by the application.

#### Amazon DynamoDB

Amazon DynamoDB is proposed for high-frequency and low-latency data such as:

- Current auction states.
- Bid events.
- Wallet hold information.
- Active WebSocket connections.

DynamoDB Streams can trigger subsequent processing tasks whenever records are changed.

#### Amazon SQS FIFO

Amazon SQS FIFO is proposed for receiving bid requests and processing them in order. It helps prevent inconsistencies when multiple users place bids within a short period.

#### Amazon EventBridge

Amazon EventBridge can schedule auction start and end times and route system events to the appropriate processing components.

#### Amazon Kinesis Data Streams

Amazon Kinesis Data Streams can receive auction event streams for near-real-time analytics, reporting, and operational monitoring.

#### Amazon Cognito

Amazon Cognito is proposed for user registration, authentication, and authorization in the extended architecture. The current application version continues to use JWT authentication managed by the FastAPI backend.

#### Amazon CloudWatch and AWS CloudTrail

- **Amazon CloudWatch** collects logs and metrics and provides operational alarms.
- **AWS CloudTrail** records administrative activities performed within the AWS account.

#### AWS IAM, AWS KMS, and AWS Secrets Manager

- **AWS IAM** applies permissions according to the principle of least privilege.
- **AWS KMS** manages encryption keys.
- **AWS Secrets Manager** stores sensitive information such as database credentials and application secrets.

### 5. System Component Design

#### Frontend

The frontend is developed using React/Vite and provides interfaces for:

- User registration and login.
- Browsing available products.
- Viewing auction details.
- Placing bids and following the current price.
- Listing products.
- Managing personal information.
- Administering the system.

After the build process, the application generates a `dist/` directory. The files in this directory are uploaded to Amazon S3 and delivered to users.

#### Backend

The FastAPI backend provides APIs for:

- User authentication.
- Account management.
- Product management.
- Auction session management.
- Bid validation and processing.
- Product image management.
- Notification management.
- Administrative functions.

The backend is packaged with Docker to maintain a consistent runtime environment between local development and AWS.

#### Database

During local development, the data is managed using MySQL in Docker. When deployed to AWS, the database is migrated to Amazon RDS for MySQL.

The backend connects to RDS using configuration stored in environment variables or AWS Secrets Manager. The database Security Group only allows connections from the backend.

#### Image Storage

Product images are not stored directly in the relational database. The backend uploads images to Amazon S3 and stores their object keys or URLs in the database. This approach reduces database size and supports scalable object storage.

#### Real-Time Auction Processing

In the initial version, the backend validates bid requests and updates the relational database. In the extended architecture, bid requests can be sent to an SQS FIFO queue to preserve their processing order.

A WebSocket API delivers new prices and auction status updates to connected browsers, allowing users to receive updates without reloading the entire page.

### 6. Technical Implementation Plan

#### Phase 1: Analysis and Design

- Analyze the requirements of the auction platform.
- Define buyer, seller, and administrator roles.
- Design the relational database.
- Design the source-code directory structure.
- Create the proposed AWS architecture diagram.

#### Phase 2: Application Development

- Develop the frontend with React/Vite.
- Develop REST APIs with FastAPI.
- Configure MySQL using Docker.
- Implement JWT authentication.
- Develop product and auction functions.
- Integrate image storage.

#### Phase 3: Initial AWS Deployment

- Build the frontend using `npm run build`.
- Upload the `dist/` directory to Amazon S3.
- Enable Static Website Hosting for the frontend.
- Package the backend with Docker.
- Deploy the backend to Amazon EC2.
- Create an Amazon RDS for MySQL database.
- Configure connectivity between EC2 and RDS.
- Create an S3 bucket for product images.
- Configure IAM roles and Security Groups.
- Test the connectivity among all components.

#### Phase 4: Testing

- Test user registration and login.
- Test product creation.
- Test bid placement.
- Test access permissions for each user role.
- Test image upload and display.
- Test concurrent bid requests.
- Review AWS logs and resource costs.

#### Phase 5: Future Extension

- Integrate Amazon CloudFront and AWS WAF.
- Research WebSocket API implementation.
- Process bid requests through Amazon SQS FIFO.
- Evaluate DynamoDB for real-time auction states.
- Migrate the backend container to Amazon ECS with AWS Fargate.
- Build an automated CI/CD pipeline.
- Evaluate multi-region disaster recovery.

### 7. Technical Requirements

#### Development Technologies

- Frontend: React, Vite, TypeScript or JavaScript.
- Backend: Python and FastAPI.
- Database: MySQL.
- Containerization: Docker and Docker Compose.
- Source control: Git and GitHub.
- Architecture design: diagrams.net.

#### Security Requirements

- The AWS root account must not be used for daily operations.
- Each team member must receive only the required permissions.
- Access keys and passwords must not be committed to GitHub.
- Database credentials must be stored in environment variables or Secrets Manager.
- S3 bucket access policies must be configured appropriately.
- The RDS Security Group must not allow unrestricted public access.
- The backend must validate user tokens and roles.
- HTTPS should be used when the application is placed into production.

### 8. Roadmap and Milestones

- **Week 1:** Become familiar with the working environment and learn the fundamentals of AWS.
- **Week 2:** Study commonly used AWS services.
- **Week 3:** Practise using AWS Management Console and learn how AWS costs are generated.
- **Week 4:** Analyze requirements and design the system architecture.
- **Week 5:** Develop and integrate the main application functions.
- **Week 6:** Deploy the system components to AWS.
- **Week 7:** Test, troubleshoot, and conduct the final project rehearsal.
- **Week 8:** Complete the project, workshop, and internship report.

### 9. Budget Estimation

The actual cost depends on the selected AWS Region, traffic volume, data storage, and resource operating time. Services that may generate costs include:

| Service | Purpose | Main Cost Factors |
| --- | --- | --- |
| Amazon S3 | Frontend and image storage | Storage, requests, and data transfer |
| Amazon EC2 | FastAPI backend | Instance type and operating time |
| Amazon RDS | MySQL database | Instance type, storage, and backups |
| AWS Lambda | Background tasks | Invocations and processing duration |
| Amazon CloudFront | Content delivery | Outbound data transfer |
| API Gateway | REST and WebSocket APIs | Requests and connection duration |
| Amazon CloudWatch | Logs and monitoring | Log volume and custom metrics |
| Amazon Route 53 | Domain and routing | Hosted zones and DNS queries |

Within the scope of a student project, the team prioritizes small resource configurations, removes unused resources, and uses AWS Budgets to monitor costs.

An accurate estimate will be prepared using the [AWS Pricing Calculator](https://calculator.aws/) after the final configuration and operating duration of each resource have been determined.

### 10. Risk Assessment

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Multiple users place bids concurrently | High | Use database transactions, locking, or SQS FIFO |
| EC2 backend failure | High | Use health checks, backups, and multiple instances when required |
| Database connectivity failure | High | Use RDS backups, connectivity monitoring, and Multi-AZ when required |
| Product images cannot be accessed | Medium | Review S3 policies, CORS configuration, and object keys |
| AWS credentials are exposed | High | Use IAM roles, Secrets Manager, and never commit credentials |
| AWS costs exceed the budget | Medium | Use AWS Budgets, billing alerts, and remove unused resources |
| WebSocket connections are interrupted | Medium | Implement reconnection and connection-state management |
| Primary AWS Region becomes unavailable | High | Use Route 53 failover and a standby region |
| Bids are processed in the wrong order | High | Use an FIFO queue and auction-state validation |

### 11. Contingency Plan

- Perform regular backups of the RDS database.
- Enable versioning for important S3 buckets.
- Store stable Docker images in Amazon ECR.
- Monitor application errors through CloudWatch.
- Prepare procedures to restore the backend from a Docker image.
- Avoid making unverified changes directly in the running environment.
- Evaluate Infrastructure as Code using Terraform or AWS CloudFormation.
- Establish a standby region when high availability becomes necessary.

### 12. Expected Outcomes

After completion, the system is expected to:

- Provide an auction website accessible through the Internet.
- Allow users to register, log in, and manage their accounts.
- Allow sellers to list products and create auction sessions.
- Allow buyers to view products and place bids.
- Store structured data in Amazon RDS.
- Store product images in Amazon S3.
- Deploy both frontend and backend components on AWS.
- Monitor system activities and errors.
- Establish a foundation for future real-time auction capabilities.
- Improve the team members’ software development, teamwork, system deployment, and AWS skills.

The extended architecture shown in the diagram represents the long-term development direction. The initial version prioritizes implementing the core functions and establishing a stable deployment with Amazon S3, Amazon EC2, Amazon RDS, and AWS Lambda before integrating more advanced services.