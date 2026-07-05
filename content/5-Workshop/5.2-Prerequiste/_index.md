---
title: "Environment Preparation"
date: 2026-06-29
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

---

Before starting the workshop, you need to prepare an AWS account, local tools, source code, and project configuration.

---

## AWS account requirements

You need an AWS account with permissions to create and manage the following resources:

- Amazon Cognito User Pool
- Amazon API Gateway
- AWS Lambda
- Amazon DynamoDB
- Amazon S3
- Amazon SES
- Amazon EventBridge (default event bus + Scheduler)
- Amazon CloudFront
- Amazon CloudWatch Logs
- AWS CloudFormation stacks
- IAM roles and policies created by AWS SAM

The project is deployed in the AWS region:

```text
ap-southeast-1
```

![AWS Singapore Region](/images/5-Workshop/5.2-Prerequisite/Region.jpg)

---

## Tools to install

Install the following tools on your machine:

| Tool | Purpose |
|---|---|
| AWS CLI | Configure AWS credentials and interact with AWS services |
| AWS SAM CLI | Build and deploy the serverless backend |
| .NET SDK 8.0 | Build and run the backend Lambda code (.NET 8/C#) |
| Node.js (LTS) and npm | Install dependencies and run the React frontend (Vite) |
| Git | Clone and manage the source code |
| Visual Studio / Visual Studio Code | Edit and manage project files |

---

## Verify AWS CLI

Open a terminal and run:

```powershell
aws --version
```

Then configure AWS credentials:

```powershell
aws configure
```

Enter the required values:

```text
AWS Access Key ID
AWS Secret Access Key
Default region name: ap-southeast-1
Default output format: json
```

Verify the current AWS identity:

```powershell
aws sts get-caller-identity
```

If the command returns an AWS account ID and user ARN, AWS CLI is configured successfully.

---

## Verify AWS SAM CLI

```powershell
sam --version
```

If SAM CLI is installed correctly, the terminal will output its version.

---

## Verify .NET SDK

```powershell
dotnet --version
```

The result should start with 8.x, matching the TargetFramework net8.0 declared in the .csproj files of the Lambda functions.

---

## Verify Node.js and npm

```powershell
node -v
npm -v
```

The frontend uses Vite as the build tool and React 19.

---

## Project source code

The project source code is organized into two main parts in the same repository:

```text
aws-event-management/
├── BE/
│   ├── EventManagement.Serverless.sln
│   ├── template.yaml
│   └── src/
│       ├── Functions/
│       └── Shared/
└── FE/
    ├── package.json
    ├── vite.config.ts
    └── src/
```

---

## Backend configuration files

The BE/ directory should contain:

```text
BE/
├── EventManagement.Serverless.sln
├── template.yaml
└── src/
    ├── Functions/
    │   ├── EventManagement.EventLambda/
    │   ├── EventManagement.UserProfileLambda/
    │   ├── EventManagement.RegistrationTicketLambda/
    │   ├── EventManagement.AttendanceCertificateLambda/
    │   ├── EventManagement.NotificationLambda/
    │   └── EventManagement.AnalyticsLambda/
    └── Shared/
```

Important files and folders:

| File / Folder | Purpose |
|---|---|
| template.yaml | Defines all AWS resources to deploy with SAM (6 Lambda functions, DynamoDB, S3, Cognito Authorizer, CloudFront, and more) |
| EventManagement.Serverless.sln | Main solution containing all Lambda projects |
| src/Functions/ | Source code for each Lambda function |
| src/Shared/ | Shared services, repositories, DTOs, validators, and constants used across the Lambda functions |

---

## Frontend configuration

The React frontend uses Vite environment variables to connect to the AWS backend that has been deployed, for example in FE/.env:

```env
VITE_API_BASE_URL=<API Gateway endpoint>
VITE_AWS_REGION=ap-southeast-1
VITE_COGNITO_USER_POOL_ID=<Cognito User Pool ID>
VITE_COGNITO_CLIENT_ID=<Cognito App Client ID>
```

The frontend uses AWS Amplify (aws-amplify and @aws-amplify/ui-react) together with cognitoAuthService.ts for authentication and axiosInstance.ts to centralize API calls (base URL, JWT interception).

The values in .env must match the deployed AWS backend.

---

## Install frontend dependencies

The React frontend (there is no separate admin web app; Admin and User share one SPA, differentiated by src/pages/admin and src/pages/user):

```powershell
cd aws-event-management/FE
npm install
```

Key libraries in package.json:

| Library | Role |
|---|---|
| aws-amplify, @aws-amplify/ui-react | Integrates authentication with Amazon Cognito |
| antd, @ant-design/icons, tailwindcss | UI components and styling |
| @reduxjs/toolkit, react-redux | Global state management |
| axios | Calls the API through API Gateway |
| html5-qrcode, qrcode.react | Scans QR codes (check-in) and generates QR codes (tickets) |
| recharts | Draws charts for the Dashboard/Analytics page |
| react-router-dom | Navigation between User/Admin/Public pages |

---

## Important notes

Before starting the workshop, confirm that the AWS account has permissions for the full list of services above, and configure AWS Budget alerts to avoid unexpected costs, especially for CloudFront and SES in production-style usage.

template.yaml currently accepts CognitoUserPoolIdParam as a deployment parameter instead of hard-coding it, so you need the real User Pool ID ready before running sam deploy.

Amazon SES runs in Sandbox mode by default and can only send emails to verified addresses. Verify the sender address (SES_FROM_EMAIL) and the test recipient addresses in the SES Console before testing notification/certificate flows.