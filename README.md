# Event-Driven Serverless File Processing on AWS

## Overview

This project implements an event-driven, serverless file processing architecture using AWS CloudFormation. When a file is uploaded to an Amazon S3 bucket, an S3 event notification triggers an AWS Lambda function deployed in a private subnet. The Lambda function processes the uploaded file, stores processing metadata in S3, and publishes a notification to Amazon SNS.

The infrastructure is provisioned using Infrastructure as Code (IaC) with a CloudFormation YAML template.

## Architecture

The architecture consists of the following workflow:

1. A user uploads a file to an Amazon S3 bucket.
2. S3 generates an `ObjectCreated` event.
3. The event invokes the Lambda function deployed in the private subnet of a VPC.
4. Lambda retrieves the uploaded object and generates a processing metadata log.
5. The metadata log is stored in the S3 bucket.
6. Lambda publishes a processing summary to an Amazon SNS topic.
7. SNS sends an email notification to the subscribed email address.

The VPC contains a public subnet, a private subnet, an Internet Gateway, a NAT Gateway, route tables, and an optional S3 Gateway VPC Endpoint.

## AWS Services Used

- **Amazon S3:** Stores uploaded files and processing metadata.
- **AWS Lambda:** Processes uploaded files in a private subnet.
- **Amazon SNS:** Sends email notifications about processing results.
- **Amazon VPC:** Provides network isolation for the Lambda function.
- **Internet Gateway:** Provides internet connectivity for the public subnet.
- **NAT Gateway:** Enables outbound internet access from the private subnet.
- **S3 Gateway VPC Endpoint:** Enables private connectivity to S3 through the VPC.
- **AWS IAM:** Controls Lambda execution permissions.
- **AWS CloudFormation:** Provisions and manages the infrastructure.

## Project Structure

```text
.
├── template.yaml
├── architecture-diagram.png
├── TA2-submission-report.pdf
└── README.md
```

| File | Description |
|---|---|
| `template.yaml` | CloudFormation template defining the AWS infrastructure and Lambda function. |
| `architecture-diagram.png` | Diagram illustrating the architecture and event flow. |
| `TA2-submission-report.pdf` | Report containing deployment and testing evidence. |
| `README.md` | Project overview, architecture, deployment instructions, and testing steps. |

## Infrastructure Components

### 1. Storage

The S3 bucket stores uploaded files and processing logs. Server-side encryption using S3-managed keys (SSE-S3) is enabled, and S3 Block Public Access settings are configured to prevent public access.

An S3 event notification invokes Lambda when an object is created.

### 2. Networking

The VPC contains public and private subnets.

- The public subnet is associated with a route table that routes internet-bound traffic through the Internet Gateway.
- The NAT Gateway resides in the public subnet.
- The private subnet routes outbound internet traffic through the NAT Gateway.
- The S3 Gateway VPC Endpoint provides private access to S3 through the private route table.

### 3. Lambda Function

The Lambda function runs in the private subnet and processes S3 event records. It retrieves the uploaded object's details, generates a JSON metadata log, stores the log in S3, and publishes a processing summary to SNS.

The Lambda execution role includes VPC execution permissions and the S3 and SNS permissions required by the function.

### 4. Notifications

An SNS topic receives processing summaries from Lambda. An email subscription delivers notifications to the configured recipient after the subscription is confirmed.

## Prerequisites

Before deploying the project, ensure that you have:

- An AWS account with permission to create the required resources.
- Access to the AWS Management Console.
- A valid email address for SNS notifications.
- The CloudFormation template (`template.yaml`).
- An AWS Region that supports the selected services and Lambda runtime.

## Deployment Steps

### Step 1: Open CloudFormation

Sign in to the AWS Management Console and open **AWS CloudFormation** in your selected Region.

### Step 2: Create a Stack

1. Select **Create stack**.
2. Choose **With new resources (standard)**.
3. Upload `template.yaml`.
4. Provide a stack name.
5. Enter the email address for SNS notifications.
6. Review the configuration and acknowledge IAM resource creation if prompted.
7. Submit the stack for deployment.

### Step 3: Verify Deployment

Monitor the stack's **Events** tab. Successful deployment is indicated by the `CREATE_COMPLETE` status.

If deployment fails, inspect the first `CREATE_FAILED` event and its status reason.

### Step 4: Confirm the SNS Subscription

Open the email inbox associated with the configured address and click the confirmation link sent by Amazon SNS.

## Testing the Application

### Step 1: Upload a Test File

1. Open the S3 bucket created by CloudFormation.
2. Upload a sample text file or another small test file.
3. If the template configures an upload prefix, upload the file under that prefix.

### Step 2: Verify Lambda Execution

Open the Lambda function in the AWS Console and inspect its monitoring information and Amazon CloudWatch logs. Verify that the function received the S3 event and processed the object successfully.

### Step 3: Verify the Processing Log

Return to the S3 bucket and locate the generated JSON metadata log. Verify that it contains information such as the source bucket, object key, file size, processing timestamp, and processing status.

### Step 4: Verify the Email Notification

Check the subscribed email inbox for the SNS notification containing the processing summary.

## Security Considerations

- Enable S3 Block Public Access.
- Use server-side encryption for stored objects.
- Grant Lambda only the IAM permissions needed for processing.
- Keep Lambda in the private subnet.
- Use the S3 Gateway VPC Endpoint for private S3 connectivity where configured.
- Avoid storing credentials or other sensitive information in source code.

## Cost Considerations

Some AWS resources, particularly NAT Gateways, incur charges while provisioned. Additional charges may apply to data transfer, Lambda execution, S3 storage, SNS notifications, and other services.

After completing testing and collecting the required screenshots, delete the CloudFormation stack if the resources are no longer needed. Verify that associated resources have been removed to avoid unexpected charges.

## Deliverables

- `template.yaml` — Infrastructure as Code template.
- `architecture-diagram.png` — AWS architecture diagram.
- `README.md` — Project documentation.

## Conclusion

This project demonstrates how AWS CloudFormation can automate the deployment of an event-driven serverless architecture. By integrating Amazon S3, AWS Lambda, Amazon SNS, and VPC networking, the solution enables automated file processing, metadata generation, and email-based notifications with infrastructure managed as code.
