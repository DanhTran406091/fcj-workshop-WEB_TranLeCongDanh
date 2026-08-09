---
title : "Preparation"
date : 2026-07-13
weight : 2 
chapter : false
pre : " <b> 5.2. </b> "
---

---

# Environment Preparation

## Introduction

Before deploying the **Live Auction** system on **Amazon Web Services (AWS)**, it is necessary to prepare the development environment and the required deployment tools. Proper preparation ensures that the infrastructure can be provisioned consistently using **Terraform** and allows all team members to work within the same environment.

In this workshop, several tools are used for source code management, application development, infrastructure provisioning, and interaction with AWS services.

---

## Software and Tools

Before getting started, prepare the following software and tools:

- An AWS account with the required permissions.
- Git.
- AWS CLI.
- Terraform.
- Docker Desktop.
- Node.js.
- Python 3.
- Visual Studio Code (or an equivalent IDE).

---

## Preparing the Source Code

Clone the project source code from the Git repository:

```bash
git clone <repository-url>
cd Live-Auction
```

After cloning the repository, the main project structure is as follows:

```text
backend/
frontend/
admin-frontend/
infra/
docker-compose.yml
```

Where:

- **backend/**: FastAPI backend source code.
- **frontend/**: User web application.
- **admin-frontend/**: Administration web application.
- **infra/**: Terraform configuration used to provision AWS infrastructure.

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.2-Preparation/project-structure.png" alt="Project Structure">
    <figcaption style="text-align: center;">
        <b>Figure 5.2.1.</b> Main project directory structure.
    </figcaption>
</figure>

---

## Installing and Configuring AWS CLI

After installing AWS CLI, verify the installation using:

```bash
aws --version
```

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.2-Preparation/aws-version.png" alt="AWS CLI Version">
    <figcaption style="text-align: center;">
        <b>Figure 5.2.2.</b> Verifying the installed AWS CLI version.
    </figcaption>
</figure>

Next, configure the AWS credentials:

```bash
aws configure
```

Provide the following information when prompted:

- AWS Access Key ID
- AWS Secret Access Key
- Default Region
- Default Output Format

Once completed, AWS CLI stores the credentials locally so that Terraform can authenticate and provision AWS resources.

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.2-Preparation/aws-configure.png" alt="AWS Configure">
    <figcaption style="text-align: center;">
        <b>Figure 5.2.3.</b> Configuring AWS CLI using the <code>aws configure</code> command.
    </figcaption>
</figure>

---

## Installing Terraform

Verify the Terraform installation by running:

```bash
terraform version
```

If Terraform is installed successfully, the current version information will be displayed.

Terraform will be used throughout the following sections to provision and manage the AWS infrastructure for the Live Auction system.

<figure style="text-align: center;">
    <img src="/images/5-Workshop/5.2-Preparation/terraform-version.png" alt="Terraform Version">
    <figcaption style="text-align: center;">
        <b>Figure 5.2.4.</b> Verifying the installed Terraform version.
    </figcaption>
</figure>

---

## Result

After completing the preparation steps above, the development environment is ready for provisioning the AWS infrastructure using Terraform.

The next section demonstrates how the infrastructure is initialized and deployed using Terraform.