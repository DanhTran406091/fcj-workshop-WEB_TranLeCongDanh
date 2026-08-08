---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Building and Deploying the Live Auction System on AWS

#### Overview

**Live Auction** is an online auction platform that enables users to register accounts, monitor auction sessions, and place bids in real time.

In this workshop, our team presents the process of building and deploying the **Live Auction** system on **Amazon Web Services (AWS)** using a serverless architecture. The entire cloud infrastructure is provisioned and managed through **Terraform (Infrastructure as Code)**, allowing AWS resources to be created, configured, and maintained in an automated and consistent manner.

After the infrastructure is deployed, the system leverages multiple AWS services, including **Amazon S3**, **Amazon CloudFront**, **Amazon Cognito**, **AWS Lambda**, **Amazon API Gateway**, **Amazon DynamoDB**, and **Amazon SQS FIFO**, to provide a scalable, secure, and real-time online auction platform.

This workshop focuses on preparing the deployment environment, provisioning cloud infrastructure with Terraform, verifying the deployed AWS resources, testing the system, and evaluating the deployment results.

#### Contents

1. [Project Overview and Deployment Architecture](5.1-Overview/)
2. [Environment Preparation](5.2-Preparation/)
3. [Infrastructure Deployment with Terraform](5.3-Infrastructure/)
4. [Deployed AWS Services](5.4-AWS-Services/)
5. [System Testing](5.5-Testing/)
6. [Results and Conclusion](5.6-Conclusion/)