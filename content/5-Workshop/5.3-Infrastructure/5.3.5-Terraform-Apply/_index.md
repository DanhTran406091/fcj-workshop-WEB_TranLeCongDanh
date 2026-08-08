---
title: "Deploy the Infrastructure"
date: 2026-07-13
weight: 5
chapter: false
pre: "<b>5.3.5. </b>"
---


## Introduction

After reviewing and approving the deployment plan, the team creates the AWS resources using the `terraform apply` command.

This command applies the changes defined in the Terraform plan, including creating, updating, replacing, or deleting resources so that the actual AWS infrastructure matches the desired configuration.

In the Live Auction system, the infrastructure is divided into independent modules. These modules are deployed according to their dependency order to ensure that the foundational resources are available before the dependent components are created.

The deployment order is:

1. Identity.
2. Data.
3. Messaging.
4. Compute.
5. API.
6. Edge.

---

## Review the Saved Plan

Before deployment, check the `tfplan` file in the current module:

```powershell
Get-ChildItem
```

Review the saved plan:

```powershell
terraform show tfplan
```

The team should carefully review:

* The number of resources to be created.
* Existing resources to be updated.
* Resources to be deleted or replaced.
* Resource names and AWS Regions.
* IAM Roles and IAM Policies.
* Expected Output values.

{{% notice warning %}}
Do not continue if the plan contains unexpected resource deletions or replacements. Review the configuration and generate a new `terraform plan`.
{{% /notice %}}

---

## Run Terraform Apply

If the deployment plan was saved using:

```powershell
terraform plan -out="tfplan"
```

apply the exact saved plan:

```powershell
terraform apply "tfplan"
```

When a saved `tfplan` file is used, Terraform does not request confirmation because the plan has already been generated and reviewed.

If a saved plan is not used, run:

```powershell
terraform apply
```

Terraform displays the proposed changes and requests confirmation:

```text
Do you want to perform these actions?

  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```

Enter:

```text
yes
```

to begin the deployment.

{{% notice info %}}
The recommended workflow is to use `terraform plan -out="tfplan"` followed by `terraform apply "tfplan"`. This ensures that Terraform applies only the changes previously reviewed by the team.
{{% /notice %}}

---

## Deploy the Identity Module

The Identity module is deployed first because the remaining modules require IAM Roles, IAM Policies, and user authentication resources.

Navigate to the module directory:

```powershell
cd infra\03-identity
```

Review the saved plan:

```powershell
terraform show tfplan
```

Deploy the module:

```powershell
terraform apply "tfplan"
```

This module creates the following resources:

* Amazon Cognito User Pool.
* Amazon Cognito User Pool Client.
* Required IAM Roles.
* IAM Policies for Lambda and other AWS services.
* User authentication and authorization configuration.

When the deployment is successful, Terraform displays:

```text
Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
```

The actual number of resources may vary depending on the project configuration.

<!--
SCREENSHOT INSTRUCTIONS:
1. Run terraform apply "tfplan" in infra/03-identity.
2. Wait for the deployment to finish.
3. Capture the final Terminal output containing:
   Apply complete! Resources: ... added, ... changed, ... destroyed.
4. Save the image as:
static/images/5-Workshop/5.3-Infrastructure/terraform-apply-identity.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-apply-identity.png" alt="Deploy the Identity module" width="90%">
    <figcaption style="text-align: center;">
        <b>Figure 5.3.18.</b> Terraform deployment result for the Identity module.
    </figcaption>
</figure>

---

## Deploy the Data Module

After the Identity module is complete, navigate to the Data module:

```powershell
cd ..\04-data
```

If a plan has not been generated or the configuration has changed, run:

```powershell
terraform init
terraform validate
terraform plan -out="tfplan"
```

Deploy the module:

```powershell
terraform apply "tfplan"
```

The Data module creates Amazon DynamoDB tables for storing:

* Product information.
* Auction information.
* Auction status.
* Bid history.
* WebSocket connection information.
* Other application data.

After the deployment, review the Output values:

```powershell
terraform output
```

<!--
SCREENSHOT INSTRUCTIONS:
1. Deploy the 04-data module.
2. Capture the Apply complete result or the terraform output result.
3. Do not include sensitive data.
4. Save the image as:
static/images/5-Workshop/5.3-Infrastructure/terraform-apply-data.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-apply-data.png" alt="Deploy the Data module" width="90%">
    <figcaption style="text-align: center;">
        <b>Figure 5.3.19.</b> Deployment result for the Data module.
    </figcaption>
</figure>

---

## Deploy the Messaging Module

Navigate to the Messaging module:

```powershell
cd ..\05-messaging
```

Validate, plan, and deploy the module:

```powershell
terraform init
terraform validate
terraform plan -out="tfplan"
terraform apply "tfplan"
```

This module creates the messaging resources:

* Amazon SQS FIFO Queue.
* Dead-letter queue, if configured.
* Queue Policy.
* Permissions for sending and receiving messages.
* Configuration for processing bid requests in order.

Review the Output values:

```powershell
terraform output
```

The Outputs may include:

* SQS Queue URL.
* SQS Queue ARN.
* Dead-letter Queue ARN.

---

## Deploy the Compute Module

Navigate to the Compute module:

```powershell
cd ..\06-compute
```

Initialize and validate the configuration if necessary:

```powershell
terraform init
terraform validate
terraform plan -out="tfplan"
```

Deploy the module:

```powershell
terraform apply "tfplan"
```

The Compute module deploys the business logic components:

* AWS Lambda Functions.
* IAM Roles for Lambda.
* Environment Variables.
* Lambda Permissions.
* Event Source Mappings between Amazon SQS and AWS Lambda.
* Functions used by the REST API and WebSocket API.

During deployment, the Lambda source code is packaged and uploaded to AWS. Terraform configures the Runtime, Handler, memory, timeout, and environment variables for each function.

<!--
SCREENSHOT INSTRUCTIONS:
1. Deploy the 06-compute module.
2. Capture the output showing the aws_lambda_function resources.
3. Include the Apply complete result.
4. Save the image as:
static/images/5-Workshop/5.3-Infrastructure/terraform-apply-compute.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-apply-compute.png" alt="Deploy the Compute module" width="90%">
    <figcaption style="text-align: center;">
        <b>Figure 5.3.20.</b> Deploying the system's AWS Lambda Functions.
    </figcaption>
</figure>

---

## Deploy the API Module

After the Lambda Functions are available, navigate to the API module:

```powershell
cd ..\07-api
```

Run:

```powershell
terraform init
terraform validate
terraform plan -out="tfplan"
terraform apply "tfplan"
```

The API module deploys:

* Amazon API Gateway.
* REST API.
* API Routes and Methods.
* Lambda Integrations.
* Cognito Authorizer.
* API Gateway WebSocket.
* WebSocket Routes such as `$connect`, `$disconnect`, and `$default`.
* Deployment Stages.
* Permissions allowing API Gateway to invoke Lambda.

After deployment, display the API endpoints:

```powershell
terraform output
```

The result may contain:

```text
rest_api_endpoint      = "https://example.execute-api.ap-southeast-1.amazonaws.com/dev"
websocket_api_endpoint = "wss://example.execute-api.ap-southeast-1.amazonaws.com/dev"
```

{{% notice warning %}}
The endpoints above are examples only. Use the endpoints returned by Terraform in the actual deployment environment.
{{% /notice %}}

<!--
SCREENSHOT INSTRUCTIONS:
1. Run terraform output after deploying the 07-api module.
2. Capture the REST API endpoint and WebSocket endpoint.
3. Partially hide the API ID if necessary.
4. Save the image as:
static/images/5-Workshop/5.3-Infrastructure/terraform-api-outputs.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-api-outputs.png" alt="Terraform API Outputs" width="85%">
    <figcaption style="text-align: center;">
        <b>Figure 5.3.21.</b> REST API and WebSocket endpoints after deployment.
    </figcaption>
</figure>

---

## Deploy the Edge Module

The Edge module is deployed after the API and backend components are available.

Navigate to the module:

```powershell
cd ..\09-edge
```

Validate, plan, and deploy the module:

```powershell
terraform init
terraform validate
terraform plan -out="tfplan"
terraform apply "tfplan"
```

The Edge module deploys:

* S3 Bucket for the frontend.
* S3 Bucket Policy.
* Amazon CloudFront Distribution.
* Origin Access Control.
* Default Root Object configuration.
* Static content distribution configuration.
* Output values for S3 and CloudFront.

After the deployment, review the Outputs:

```powershell
terraform output
```

Example:

```text
frontend_bucket_name   = "live-auction-frontend-example"
cloudfront_domain_name = "example.cloudfront.net"
```

<!--
SCREENSHOT INSTRUCTIONS:
1. Run terraform output in the 09-edge module.
2. Capture the S3 Bucket name and CloudFront domain.
3. Save the image as:
static/images/5-Workshop/5.3-Infrastructure/terraform-edge-outputs.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-edge-outputs.png" alt="Terraform Edge Outputs" width="85%">
    <figcaption style="text-align: center;">
        <b>Figure 5.3.22.</b> S3 and CloudFront Output values after deployment.
    </figcaption>
</figure>

---

## Review Terraform Outputs

After deploying each module, display its Output values:

```powershell
terraform output
```

Display a specific Output:

```powershell
terraform output rest_api_endpoint
```

Export the Output values as JSON:

```powershell
terraform output -json
```

Output values are used to:

* Configure the frontend.
* Configure Lambda environment variables.
* Connect the application to API Gateway.
* Establish WebSocket connections.
* Verify resource names and ARNs.
* Transfer information between modules.

If an Output is marked as sensitive, Terraform does not display its value in the normal output.

{{% notice warning %}}
Do not publicly disclose Outputs containing tokens, passwords, credentials, or other sensitive information.
{{% /notice %}}

---

## Review the Terraform State

After deployment, list the resources managed by Terraform:

```powershell
terraform state list
```

Example:

```text
aws_cognito_user_pool.live_auction
aws_cognito_user_pool_client.web_client
aws_iam_role.lambda_execution_role
```

Display the details of a resource:

```powershell
terraform state show aws_cognito_user_pool.live_auction
```

The `terraform state list` command confirms that the deployed resources have been recorded in the Terraform State.

{{% notice warning %}}
Do not edit the Terraform State file manually. Incorrect State modifications may prevent Terraform from managing the infrastructure accurately.
{{% /notice %}}

---

## Verify Infrastructure Synchronization

After `terraform apply` is complete, run:

```powershell
terraform plan
```

If the infrastructure matches the configuration, Terraform displays:

```text
No changes. Your infrastructure matches the configuration.
```

This confirms that:

* The AWS resources were created according to the configuration.
* The Terraform State was updated.
* No pending infrastructure changes remain.
* The Terraform source code and actual infrastructure are synchronized.

<!--
SCREENSHOT INSTRUCTIONS:
1. After a successful apply, run terraform plan again.
2. Capture:
   No changes. Your infrastructure matches the configuration.
3. Save the image as:
static/images/5-Workshop/5.3-Infrastructure/terraform-no-changes.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-no-changes.png" alt="Terraform reports no changes" width="90%">
    <figcaption style="text-align: center;">
        <b>Figure 5.3.23.</b> Terraform confirms that the infrastructure matches the configuration.
    </figcaption>
</figure>

---

## Common Errors

### The saved plan is no longer valid

Error message:

```text
Saved plan is stale
```

Generate a new plan:

```powershell
terraform plan -out="tfplan"
```

Apply the new plan:

```powershell
terraform apply "tfplan"
```

### Insufficient permissions

Error message:

```text
AccessDenied
```

Verify the current AWS identity:

```powershell
aws sts get-caller-identity
```

Then review the IAM Policy assigned to the deployment identity.

### The resource already exists

If a resource was created manually, Terraform may report a duplicate-name error.

The team should:

* Review the existing AWS resource.
* Rename the new resource if appropriate.
* Or import the existing resource into the Terraform State using `terraform import`.

### Module dependency error

If a module requires an Output from another module that has not been deployed, the deployment may fail.

Deploy the modules in the required order:

```text
Identity → Data → Messaging → Compute → API → Edge
```

### Lambda package error

Check:

* The source-code path.
* Handler name.
* Runtime.
* Required dependencies.
* ZIP file structure.
* The `source_code_hash` value.

### CloudFront is not immediately available

After deployment, CloudFront may require some time to reach the `Deployed` status. Wait for the distribution process to finish before testing the frontend.

---

## Destroy Resources in a Test Environment

To remove test resources, first review the destruction plan:

```powershell
terraform plan -destroy
```

After carefully reviewing the plan, run:

```powershell
terraform destroy
```

Terraform requests confirmation:

```text
yes
```

{{% notice danger %}}
`terraform destroy` deletes the resources managed by the current module. Do not run this command in an active environment without backing up data and confirming the exact scope of the operation.
{{% /notice %}}

---

## Result

After completing the deployment process:

* The Terraform modules were applied in the correct dependency order.
* IAM Roles, IAM Policies, and Amazon Cognito were created.
* Amazon DynamoDB tables were deployed.
* Amazon SQS FIFO was configured for ordered bid request processing.
* AWS Lambda Functions were deployed.
* REST API and WebSocket API were created using Amazon API Gateway.
* Amazon S3 and Amazon CloudFront were configured for the frontend.
* The required Output values were retrieved from Terraform.
* The Terraform State was updated with the deployed resources.
* The post-deployment `terraform plan` confirmed that no pending changes remained.
* The infrastructure was ready for deployment verification.