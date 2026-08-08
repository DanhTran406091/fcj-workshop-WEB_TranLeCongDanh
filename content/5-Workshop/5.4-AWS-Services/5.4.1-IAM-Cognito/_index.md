---
title: "AWS IAM and Amazon Cognito"
date: 2026-07-27
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

## Overview

The **Live Auction** system uses **AWS Identity and Access Management (IAM)** and **Amazon Cognito** for two different purposes:

* **AWS IAM** manages access permissions between AWS services.
* **Amazon Cognito** authenticates application accounts and supports the separation of User and Admin permissions.

IAM is not used to create application accounts for end users. Instead, User and Admin accounts are managed through an Amazon Cognito User Pool.

## Role of AWS IAM

AWS IAM provides Roles and Policies that allow AWS services to communicate with each other according to the **principle of least privilege**.

In the Live Auction system, IAM is used to:

* Allow AWS Lambda to write logs to Amazon CloudWatch.
* Allow Lambda to read and write data in Amazon DynamoDB.
* Allow Lambda to send and receive messages through Amazon SQS FIFO.
* Allow Lambda to manage API Gateway WebSocket connections.
* Allow API Gateway to invoke the corresponding Lambda Functions.
* Allow Amazon S3 and CloudFront to work together to distribute frontend content.
* Restrict each Lambda Function to only the resources required for its operation.
* Control access permissions between the components of the system.

## Roles and Policies in the System

Each group of Lambda Functions is assigned an IAM Role appropriate to its responsibilities.

| Component | Required permissions |
| --- | --- |
| **Business Logic Lambda** | Read and write data in DynamoDB and write logs to CloudWatch. |
| **Bid Processing Lambda** | Receive messages from SQS FIFO, update bid data in DynamoDB, and send results to the WebSocket API. |
| **WebSocket Lambda** | Store, retrieve, and remove Connection IDs in DynamoDB and manage WebSocket connections. |
| **API Gateway** | Invoke the configured Lambda Functions. |
| **CloudFront** | Access frontend content stored in S3 according to the distribution configuration. |

The IAM Roles and IAM Policies are declared in Terraform. When `terraform apply` is executed, Terraform automatically creates the Roles, attaches the required Policies, and configures trust relationships between AWS services.

## Checking IAM Roles on AWS Management Console

After the Terraform deployment is completed, the IAM Roles can be checked by following the steps below.

### Step 1: Open the IAM service

Sign in to **AWS Management Console**.

Enter the following keyword in the search bar:

```text
IAM
```

Select **IAM — Identity and Access Management** from the results.

### Step 2: Open the IAM Role list

From the navigation menu on the left, select:

```text
Access management → Roles
```

The Roles page displays all IAM Roles available in the AWS account.

### Step 3: Find the IAM Roles of the project

Enter the resource prefix of the system in the search box:

```text
la-
```

Verify the IAM Roles created by Terraform for the Lambda Functions and other related components.

<!--
SCREENSHOT INSTRUCTIONS:

1. Open IAM → Roles.
2. Enter "la-" in the search box.
3. Capture the list of IAM Roles belonging to the project.
4. Do not expose the AWS Account ID or other sensitive information.
5. Save the image as:
   iam-role-list.png
6. Place the image in:
   static/images/5-Workshop/5.4-AWS-services/5.4.1-IAM-Cognito/
-->

{{< figure
    src="/images/5-Workshop/5.4-AWS-services/5.4.1-IAM-Cognito/iam-role-list.png"
    title="Figure 5.4.1.1: IAM Roles of the Live Auction system"
    width="100%"
>}}

### Step 4: Check IAM Role permissions

Select an IAM Role assigned to a Lambda Function.

Open the **Permissions** tab and verify:

* The Policies attached to the Role.
* Permission to write logs to CloudWatch.
* Permission to access DynamoDB.
* Permission to access SQS FIFO if the Lambda Function processes queue messages.
* Permission to manage WebSocket connections if the Lambda Function sends real-time data.
* The resource scope allowed by each Policy.

Permissions such as `AdministratorAccess` should not be assigned to Lambda Functions. Access permissions should be restricted to the resources required by each function.

<!--
SCREENSHOT INSTRUCTIONS:

1. Select an IAM Role assigned to a Lambda Function.
2. Open the Permissions tab.
3. Capture the list of Permission policies.
4. It is not necessary to capture the entire JSON Policy if it is too long.
5. Save the image as:
   iam-role-permissions.png
-->

{{< figure
    src="/images/5-Workshop/5.4-AWS-services/5.4.1-IAM-Cognito/iam-role-permissions.png"
    title="Figure 5.4.1.2: Policies attached to a Lambda IAM Role"
    width="100%"
>}}

### Step 5: Check the Trust Relationship

On the IAM Role details page, open the following tab:

```text
Trust relationships
```

Check which service is allowed to assume the Role.

For a Lambda Execution Role, the **Trusted entities** section must allow:

```text
lambda.amazonaws.com
```

The Trust Relationship ensures that only the specified AWS service can assume and use the IAM Role.

## Role of Amazon Cognito

Amazon Cognito is used as the account authentication service for both the **User Frontend** and the **Admin Frontend**.

Cognito is responsible for:

* User account registration.
* Sign-in and sign-out.
* Account verification.
* Password management.
* Issuing tokens after successful authentication.
* Storing basic account attributes.
* Supporting User and Admin role separation.
* Allowing API Gateway or the backend to verify the identity of the requester.

## Amazon Cognito Components

### Cognito User Pool

A Cognito User Pool is the account directory of the system.

The User Pool manages information such as:

* Username or email address.
* Password.
* Account confirmation status.
* Account attributes.
* Account role or group.
* Password policy.
* Registration and authentication processes.

The system uses a shared User Pool for both User and Admin accounts. Access permissions are separated based on the role or group assigned to each account.

### Cognito App Client

The App Client allows the frontend applications to connect to the Cognito User Pool.

After successful authentication, Cognito returns the required tokens, including:

* **ID Token:** Contains account identity information.
* **Access Token:** Used to authorize access.
* **Refresh Token:** Used to request new tokens after the current tokens expire.

The frontend includes the token in requests sent to API Gateway. The backend verifies the token before processing the requested operation.

{{% notice warning %}}
Do not include Client Secrets, Access Tokens, Refresh Tokens, or account credentials in screenshots or report content.
{{% /notice %}}

## Account Authentication Flow

The authentication flow is performed as follows:

1. A User or Admin enters their credentials on the corresponding frontend.
2. The frontend sends an authentication request to Amazon Cognito.
3. Cognito verifies the account and password stored in the User Pool.
4. If the credentials are valid, Cognito returns authentication tokens to the frontend.
5. The frontend stores the authentication information according to the application's mechanism.
6. When calling an API, the frontend includes an Access Token or ID Token in the request.
7. API Gateway or Lambda verifies the token and account role.
8. The request is processed only if the account has the required permission.
9. Administration APIs reject requests from regular User accounts.

## Checking the Cognito User Pool on AWS Management Console

### Step 1: Open Amazon Cognito

Enter the following keyword in the AWS Management Console search bar:

```text
Cognito
```

Select **Amazon Cognito**.

### Step 2: Open the User Pool list

From the Amazon Cognito interface, select:

```text
User pools
```

Verify the User Pool created by Terraform for the Live Auction system.

<!--
SCREENSHOT INSTRUCTIONS:

1. Open Amazon Cognito → User pools.
2. Capture the User Pool list.
3. Ensure that the User Pool name and status are visible.
4. Hide sensitive information if necessary.
5. Save the image as:
   cognito-user-pool.png
-->

{{< figure
    src="/images/5-Workshop/5.4-AWS-services/5.4.1-IAM-Cognito/cognito-user-pool.png"
    title="Figure 5.4.1.3: Cognito User Pool of the Live Auction system"
    width="100%"
>}}

### Step 3: Check the sign-in configuration

Select the User Pool of the system and verify:

* User Pool name.
* User Pool ID.
* AWS Region.
* Sign-in method.
* Attributes used for authentication.
* Password policy.
* Self-registration status.
* Email verification mechanism.
* User groups if Cognito Groups are used.

The full User Pool ID may be hidden in the report to avoid publicly exposing resource identifiers.

### Step 4: Check the App Client

On the User Pool details page, open:

```text
Applications → App clients
```

Verify the App Client used by the frontend applications for registration and authentication.

The following information should be checked:

* App Client name.
* Client ID.
* Enabled authentication flows.
* Token validity duration.
* Callback URLs and sign-out URLs, if used.
* Whether the App Client configuration is appropriate for a frontend application.

<!--
SCREENSHOT INSTRUCTIONS:

1. Open User Pool → Applications → App clients.
2. Capture the App Client list.
3. Hide the Client ID if necessary.
4. Do not capture or expose the Client Secret.
5. Save the image as:
   cognito-app-client.png
-->

{{< figure
    src="/images/5-Workshop/5.4-AWS-services/5.4.1-IAM-Cognito/cognito-app-client.png"
    title="Figure 5.4.1.4: App Client configured for the Live Auction system"
    width="100%"
>}}

### Step 5: Check User and Admin accounts

From the User Pool, open:

```text
User management → Users
```

Check the accounts created in the system.

The following information should be verified:

* Username.
* Email address.
* Account confirmation status.
* Account activation status.
* Account creation date.
* Account role or group.

If Cognito Groups are used, open:

```text
User management → Groups
```

Verify the corresponding groups, such as:

```text
User
Admin
```

An administrator account must be assigned the appropriate Admin permission before it can access the administration features.

<!--
SCREENSHOT INSTRUCTIONS:

1. Open User management → Users.
2. Capture part of the account list.
3. Hide email addresses, usernames, or personal information if necessary.
4. Do not capture tokens or sign-in credentials.
5. Save the image as:
   cognito-users.png
-->

{{< figure
    src="/images/5-Workshop/5.4-AWS-services/5.4.1-IAM-Cognito/cognito-users.png"
    title="Figure 5.4.1.5: Accounts managed by Amazon Cognito"
    width="100%"
>}}

## Testing the Authentication Functions

After checking the AWS configuration, the authentication flow is tested directly on both frontend applications.

### Testing a User account

1. Open the User Frontend.
2. Create a new account.
3. Complete the account verification process if required.
4. Sign in using the newly created account.
5. Verify access to personal profile information.
6. Verify the ability to view and create auction sessions.
7. Confirm that the User account cannot access administration functions.

### Testing an Admin account

1. Open the Admin Frontend.
2. Sign in using an Admin account.
3. Test the user account management function.
4. Test the product category management function.
5. Test the auction approval function.
6. Test the function for creating additional Admin accounts.
7. Confirm that administration APIs only accept accounts with the Admin role.

## Results

After completing the deployment and verification process:

* IAM Roles and IAM Policies were successfully created by Terraform.
* Lambda Functions were assigned permissions appropriate to their responsibilities.
* Access between AWS services was controlled according to the principle of least privilege.
* Lambda Functions were able to write logs to Amazon CloudWatch.
* The required Lambda Functions were able to access DynamoDB, SQS FIFO, and the WebSocket API.
* The Cognito User Pool was successfully created and operated correctly.
* The App Client was configured for frontend registration and authentication.
* Users were able to create accounts and sign in to the User Frontend.
* Administrators were able to sign in to the Admin Frontend.
* User and Admin permissions were separated when accessing system functions.
* Administration APIs were protected from accounts without the required permissions.
* The authentication and authorization components were ready for integration with the remaining AWS services.