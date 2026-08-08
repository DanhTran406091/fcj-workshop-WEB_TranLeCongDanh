---
title: "Worklog Week 6"
date: 2026-07-27
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Duration:

**From July 27, 2026 to July 31, 2026**

### Objectives for Week 6:

* Prepare the AWS deployment environment.
* Apply Terraform to manage infrastructure as code.
* Create the Infrastructure directory structure.
* Deploy the main serverless architecture components to AWS.
* Integrate the application with the deployed AWS services.
* Document the implementation process for the Workshop chapter.

### Tasks completed:

| Day | Date | Tasks | Reference |
| --- | --- | --- | --- |
| Monday | 27/07/2026 | Prepared the deployment environment; installed and checked AWS CLI and Terraform; configured AWS credentials and the working Region. | <https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli> |
| Tuesday | 28/07/2026 | Created the Infrastructure directory structure; defined the AWS Provider, input variables, resources, and output values in Terraform. | <https://developer.hashicorp.com/terraform/language> |
| Wednesday | 29/07/2026 | Ran `terraform init` and `terraform plan`; validated the configuration, resolved errors, and reviewed the resources to be created. | <https://developer.hashicorp.com/terraform/cli/commands/init> |
| Thursday | 30/07/2026 | Ran `terraform apply` to deploy AWS resources; reviewed IAM, Cognito, S3, CloudFront, Lambda, API Gateway, DynamoDB, and SQS FIFO. | <https://developer.hashicorp.com/terraform/cli/commands/apply> |
| Friday | 31/07/2026 | Integrated the frontend and processing functions with the deployed AWS resources; documented commands, deployment steps, and results for the Workshop. | Team source code and deployment documentation |

### Results achieved in Week 6:

* Installed and verified the required development tools:

  * AWS CLI.
  * Terraform.
  * Git.
  * Node.js and npm.
  * Python and the required libraries.

* Configured the working environment with the team’s AWS account.
* Created the Infrastructure directory for managing Terraform code.
* Defined the main Terraform components:

  * AWS Provider.
  * Input variables.
  * Local values.
  * Resources.
  * Data sources.
  * Output values.

* Performed the infrastructure deployment workflow:

  * `terraform init`.
  * `terraform validate`.
  * `terraform plan`.
  * `terraform apply`.
  * Resource verification through the AWS Management Console.

* Performed the initial deployment of the following AWS services:

  * **AWS IAM:** roles and permissions for AWS services.
  * **Amazon Cognito:** user registration, authentication, and management.
  * **Amazon S3:** frontend and static resource storage.
  * **Amazon CloudFront:** frontend content distribution.
  * **AWS Lambda:** serverless business logic execution.
  * **Amazon API Gateway:** REST API endpoints.
  * **API Gateway WebSocket:** real-time connections and updates.
  * **Amazon DynamoDB:** auction and WebSocket connection data.
  * **Amazon SQS FIFO:** ordered bid request processing.

* Established the initial connection between the frontend and AWS APIs.
* Retrieved Terraform outputs such as API endpoints, WebSocket endpoints, and the CloudFront address.
* Documented the preparation, configuration, and deployment steps for Chapter 5 – Workshop.
* Identified configuration and integration issues requiring further work in Week 7.