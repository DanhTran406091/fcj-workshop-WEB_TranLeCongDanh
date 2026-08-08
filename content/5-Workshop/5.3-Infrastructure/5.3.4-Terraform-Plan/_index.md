---
title: "Review the Deployment Plan"
date: 2026-07-13
weight: 4
chapter: false
pre: "<b>5.3.4. </b>"
---

## Introduction

After initializing the Terraform environment, the team validates the configuration and prepares the deployment plan using the `terraform plan` command.

This command analyzes the Terraform configuration files, reads the current infrastructure state, and compares it with the desired configuration. The result shows which resources will be created, updated, replaced, or deleted before the changes are deployed to AWS.

Reviewing the deployment plan helps the team detect configuration errors, invalid permissions, and unexpected changes before they affect the running infrastructure.

---

## Check the Terraform Formatting

Before generating a deployment plan, check the formatting of the Terraform files:

```powershell
terraform fmt -check
```

If any files do not follow the standard format, run:

```powershell
terraform fmt
```

The `terraform fmt` command automatically standardizes indentation, spacing, and the presentation of Terraform configuration blocks.

After formatting the files, verify them again:

```powershell
terraform fmt -check
```

If the command does not return an error, the configuration files follow the expected format.

---

## Validate the Terraform Configuration

Inside an initialized module directory, run:

```powershell
terraform validate
```

The `terraform validate` command checks:

* The syntax of the `.tf` files.
* Variable names and data types.
* Required resource properties.
* References between resources.
* Provider and module configuration.
* The internal consistency of the Terraform configuration.

When the configuration is valid, Terraform displays:

```text
Success! The configuration is valid.
```

<!--
SCREENSHOT INSTRUCTIONS:
1. Open the Terminal in infra/03-identity.
2. Run: terraform validate
3. Capture the result containing:
   Success! The configuration is valid.
4. Save the image as:
static/images/5-Workshop/5.3-Infrastructure/terraform-validate-success.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-validate-success.png" alt="Successful Terraform validation" width="80%">
    <figcaption style="text-align: center;">
        <b>Figure 5.3.15.</b> Successfully validating the Terraform configuration with <code>terraform validate</code>.
    </figcaption>
</figure>

{{% notice warning %}}
The `terraform validate` command only checks the syntax and internal consistency of the configuration. It does not confirm that the AWS account has sufficient permissions to create the resources.
{{% /notice %}}

---

## Generate the Deployment Plan

After the configuration has been validated, run:

```powershell
terraform plan
```

Terraform performs the following operations:

1. Reads the configuration files in the module.
2. Reads the input variable values.
3. Connects to the Terraform Backend.
4. Reads the current Terraform State.
5. Retrieves the current resource state from AWS.
6. Compares the current state with the desired configuration.
7. Displays the proposed infrastructure changes.

Terraform uses the following symbols in the plan result:

| Symbol | Meaning |
| --- | --- |
| `+` | A resource will be created. |
| `~` | A resource will be updated in place. |
| `-` | A resource will be deleted. |
| `-/+` | A resource will be deleted and recreated. |
| `<=` | Data will be read from a Data Source. |

Example:

```text
Terraform will perform the following actions:

  # aws_cognito_user_pool.live_auction will be created
  + resource "aws_cognito_user_pool" "live_auction" {
      + id   = (known after apply)
      + name = "live-auction-user-pool"
    }

Plan: 3 to add, 0 to change, 0 to destroy.
```

The summary means:

* `3 to add`: three resources will be created.
* `0 to change`: no existing resources will be updated.
* `0 to destroy`: no resources will be deleted.

<!--
SCREENSHOT INSTRUCTIONS:
1. Run terraform plan in the 03-identity module.
2. Scroll to the bottom of the output.
3. Capture the summary:
   Plan: ... to add, ... to change, ... to destroy.
4. Do not include an Access Key, Secret Key, or sensitive data.
5. Save the image as:
static/images/5-Workshop/5.3-Infrastructure/terraform-plan-summary.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-plan-summary.png" alt="Terraform Plan result" width="90%">
    <figcaption style="text-align: center;">
        <b>Figure 5.3.16.</b> Deployment plan result for the Identity module.
    </figcaption>
</figure>

---

## Use an Input Variable File

If the module uses a separate variable file, provide it to the `terraform plan` command:

```powershell
terraform plan -var-file="terraform.tfvars"
```

Example content of `terraform.tfvars`:

```hcl
aws_region   = "ap-southeast-1"
environment  = "dev"
project_name = "live-auction"
```

A variable file separates configuration values from the main Terraform source code and supports multiple deployment environments.

Examples:

```text
terraform.dev.tfvars
terraform.staging.tfvars
terraform.prod.tfvars
```

To generate a plan for the development environment:

```powershell
terraform plan -var-file="terraform.dev.tfvars"
```

{{% notice warning %}}
Do not store an Access Key, Secret Access Key, password, or other sensitive information in a `.tfvars` file committed to Git. Sensitive values should be provided through environment variables or an appropriate secrets management service.
{{% /notice %}}

---

## Save the Deployment Plan

After reviewing the result, save the deployment plan to a `tfplan` file:

```powershell
terraform plan -out="tfplan"
```

The `-out` option saves the exact plan generated by Terraform. This file can be used with `terraform apply` to ensure that Terraform applies only the changes reviewed by the team.

Review the saved plan:

```powershell
terraform show tfplan
```

Export the plan in JSON format:

```powershell
terraform show -json tfplan
```

The complete workflow is:

```powershell
terraform validate
terraform plan -out="tfplan"
terraform show tfplan
```

<!--
SCREENSHOT INSTRUCTIONS:
1. Run: terraform plan -out="tfplan"
2. Open the module directory in VS Code Explorer.
3. Capture the directory showing the generated tfplan file.
4. Save the image as:
static/images/5-Workshop/5.3-Infrastructure/terraform-plan-file.png
-->

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.3-Infrastructure/terraform-plan-file.png" alt="Saved Terraform plan file" width="65%">
    <figcaption style="text-align: center;">
        <b>Figure 5.3.17.</b> The <code>tfplan</code> file generated after saving the deployment plan.
    </figcaption>
</figure>

{{% notice info %}}
The `tfplan` file uses a binary format and should not be edited manually. It may contain infrastructure configuration information and should not be committed to a public source-code repository.
{{% /notice %}}

---

## Generate a Plan for Each Module

Because the infrastructure is divided into multiple modules, the team runs `terraform plan` according to the dependency order of the architecture.

### Identity Module

Navigate to the Identity module:

```powershell
cd infra\03-identity
```

Validate the configuration and generate the plan:

```powershell
terraform validate
terraform plan -out="tfplan"
```

This module prepares the deployment plan for:

* AWS IAM.
* Amazon Cognito.
* Roles and Policies used for authentication and authorization.

### Data Module

```powershell
cd ..\04-data
terraform validate
terraform plan -out="tfplan"
```

The Data module prepares the plan for Amazon DynamoDB tables used to store:

* Product data.
* Auction information.
* Bid history.
* Auction status.
* WebSocket connection information.

### Messaging Module

```powershell
cd ..\05-messaging
terraform validate
terraform plan -out="tfplan"
```

The Messaging module prepares the plan for:

* Amazon SQS FIFO.
* A dead-letter queue, if configured.
* Queue Policies and related access permissions.

### Compute Module

```powershell
cd ..\06-compute
terraform validate
terraform plan -out="tfplan"
```

The Compute module prepares the plan for:

* AWS Lambda Functions.
* Lambda Layers, if used.
* IAM Roles for Lambda.
* Environment Variables.
* Event Source Mappings between Lambda and SQS.

### API Module

```powershell
cd ..\07-api
terraform validate
terraform plan -out="tfplan"
```

The API module prepares the plan for:

* Amazon API Gateway.
* REST API routes.
* API Gateway WebSocket.
* Lambda Integrations.
* Authorizers and deployment Stages.

### Edge Module

```powershell
cd ..\09-edge
terraform validate
terraform plan -out="tfplan"
```

The Edge module prepares the plan for:

* Amazon S3.
* Amazon CloudFront.
* Origin Access Control.
* Bucket Policies.
* Frontend distribution configuration.

---

## Review the Plan Before Deployment

Before accepting the deployment plan, the team reviews the following information.

### Resource Names

The names of S3 Buckets, DynamoDB tables, Lambda Functions, APIs, and Queues must follow the naming conventions of the project.

### AWS Region

Verify that the resources will be created in the expected Region:

```text
ap-southeast-1
```

### IAM Permissions

IAM Policies should follow the principle of least privilege and provide only the permissions required by each service.

### Deleted or Replaced Resources

If the plan contains:

```text
Plan: 0 to add, 0 to change, 1 to destroy.
```

or the following symbol:

```text
-/+
```

the team must review the change carefully before continuing. Replacing a resource may interrupt the system or cause data loss if it is not configured correctly.

### Output Values

Review the expected Output values, including:

* Cognito User Pool ID.
* Cognito Client ID.
* DynamoDB table names.
* SQS Queue URL.
* REST API endpoint.
* WebSocket endpoint.
* S3 Bucket name.
* CloudFront domain name.

---

## A Plan with No Changes

If the current infrastructure already matches the Terraform configuration, Terraform displays:

```text
No changes. Your infrastructure matches the configuration.
```

This message indicates that:

* The Terraform State matches the configuration.
* No resources need to be created.
* No resources need to be updated.
* No resources need to be deleted.

This result commonly appears when `terraform plan` is executed after a successful deployment without any subsequent configuration changes.

---

## Common Errors

### Terraform has not been initialized

Error message:

```text
Backend initialization required
```

Initialize the module:

```powershell
terraform init
```

If the Backend configuration has changed:

```powershell
terraform init -reconfigure
```

### A required variable is missing

Error message:

```text
No value for required variable
```

Provide the value directly:

```powershell
terraform plan -var="project_name=live-auction"
```

Alternatively, use a variable file:

```powershell
terraform plan -var-file="terraform.tfvars"
```

### Insufficient AWS permissions

Error message:

```text
AccessDenied
```

Verify the current AWS identity:

```powershell
aws sts get-caller-identity
```

Then review the corresponding IAM Policy.

### A resource already exists

If a resource was created manually but is not recorded in the Terraform State, Terraform may report an error when the plan is applied.

Review the existing AWS resource and consider importing it into the Terraform State with `terraform import` instead of creating it again.

### The Terraform State is locked

When another process is updating the State, Terraform may report a State lock error.

Do not manually remove the lock until confirming that the other Terraform process has finished. Multiple team members applying changes to the same module simultaneously can cause State conflicts.

---

## Result

After completing the planning process:

* The Terraform files were formatted with `terraform fmt`.
* Each module configuration was validated with `terraform validate`.
* Terraform compared the desired configuration with the current infrastructure state.
* The team reviewed the number of resources to create, update, replace, or delete.
* The deployment plan for each module was saved to a `tfplan` file.
* Potentially disruptive infrastructure changes were reviewed before deployment.
* The modules were ready for deployment using `terraform apply`.