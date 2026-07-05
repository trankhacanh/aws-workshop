---
title: "Deploy Backend with AWS SAM"
date: 2026-06-29
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

---

This section explains how to build and deploy the Event Management Platform backend using AWS SAM and AWS CloudFormation.

The backend deployment creates all serverless resources: a Cognito Authorizer (referencing an existing User Pool), an API Gateway, 6 Lambda functions (.NET 8), 6 DynamoDB tables, 4 S3 buckets, EventBridge Rule and Scheduler, a CloudFront Distribution for the frontend, and the corresponding IAM roles and policies.

---

## Step 1: Open the backend folder

Open a terminal and switch to the backend folder:

```powershell
cd aws-event-management\BE
```

Check the main files and folders:

```powershell
dir
```

Make sure you have:

```text
EventManagement.Serverless.sln
template.yaml
src\
```

---

## Step 2: Restore backend dependencies

Because the backend is written in .NET, you should restore the .NET packages for the whole solution instead of using npm install:

```powershell
dotnet restore EventManagement.Serverless.sln
```

This restores the required NuGet packages for all 6 Lambda functions and the Shared project.

![restore](/images/5-Workshop/5.3-Deploy-backend/restore.jpg)

---

## Step 3: Review the SAM template

Open template.yaml in Visual Studio Code. This file defines the entire infrastructure, including:

| Resource group | Logical name in template.yaml |
|---|---|
| Cognito Authorizer | CognitoAuthorizer (references an existing User Pool through the CognitoUserPoolIdParam parameter) |
| Lambda functions | EventFunction, UserProfileFunction, TicketFunction, AttendanceCertificateFunction, NotificationFunction, AnalyticsFunction |
| DynamoDB tables | EventManagementEventsTable, EventManagementCategoriesTable, EventManagementTicketsTable, EventManagementAttendanceTable, EventManagementUsersTable, EventManagementNotificationLogTable |
| S3 buckets | EventManagementEventBannersBucket, EventManagementUserAvatarsBucket, EventManagementCertificatesBucket, FrontendBucket |
| EventBridge | Rule OnTicketRegistered (default event bus) + Schedule EventReminderSchedule (rate(1 hour)), declared directly in the NotificationFunction Events section |
| Frontend hosting | FrontendOAC, FrontendDistribution (CloudFront), FrontendBucketPolicy |

Unlike several other SAM examples, this project does not create a new Cognito User Pool; the CognitoAuthorizer references an existing User Pool through the CognitoUserPoolIdParam parameter, which must be passed in during deployment.

The backend is fully managed as Infrastructure as Code through AWS SAM and CloudFormation.

---

## Step 4: Build the backend

```powershell
sam build
```

SAM will build each .NET Lambda function with dotnet build underneath. If the build cache causes issues:

```powershell
sam build --no-cached
```

Expected result:

```text
Build Succeeded
```

---

## Step 5: Deploy the backend (first time — guided)

Because the repository does not include a pre-created samconfig.toml (often not committed because it contains environment-specific settings), the first deployment should be done in guided mode so SAM can ask a few questions and save the configuration:

```powershell
sam deploy --guided
```

SAM will prompt for:

```text
Stack Name: event-management-backend-dev
AWS Region: ap-southeast-1
Parameter CognitoUserPoolIdParam: <real Cognito User Pool ID>
Confirm changes before deploy: Y
Allow SAM CLI IAM role creation: Y
Save arguments to configuration file: Y
```

After this, samconfig.toml will be generated automatically, and subsequent deployments can use:

```powershell
sam deploy
```

or to avoid repeated confirmation prompts:

```powershell
sam deploy --no-confirm-changeset --no-fail-on-empty-changeset
```

---

## Step 6: Check the CloudFormation stack

`Open AWS Management Console → CloudFormation → Stacks` and find the newly created stack (for example, event-management-backend-dev).

Check that the stack status is CREATE_COMPLETE or UPDATE_COMPLETE.

![CloudFormation stack](/images/5-Workshop/5.3-Deploy-backend/CloudFormation.jpg)

If the stack fails, open the Events tab to see which resource failed during deployment.

---

## Step 7: Retrieve the outputs — API Gateway endpoint and CloudFront domain

After deployment succeeds, open the Outputs tab of the stack:

```text
FrontendCloudFrontDomain → CloudFront domain for accessing the frontend
FrontendBucketName → name of the S3 bucket containing the frontend build files
FrontendDistributionId → CloudFront Distribution ID (used for cache invalidation)
```

Also obtain the API Gateway endpoint from the Resources tab or from the output of the sam deploy command:

```text
https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev
```

This endpoint will be written into the frontend React .env file as VITE_API_BASE_URL.

The actual values will be shared directly during the demo.

---

## Step 8: Check the DynamoDB tables

Open `DynamoDB → Tables` and check that the 6 tables were created:

```text
EventManagementEvents
EventManagementCategories
EventManagementTickets
EventManagementAttendance
EventManagementUsers
EventManagementNotificationLog
```

All of them use BillingMode: PAY_PER_REQUEST (on-demand), which is suitable for workshop/demo environments because it avoids pre-sizing capacity and reduces cost when there is little or no traffic.

![DynamoDB tables](/images/5-Workshop/5.3-Deploy-backend/DynamoDB.jpg)

---

## Step 9: Check the S3 buckets

Open `Amazon S3 → Buckets` and verify the 4 buckets created by the stack:

| Bucket | Purpose |
|---|---|
| event-management-user-avatars-<region>-<account-id> | Stores user avatars |
| Banner bucket (auto-generated name) | Stores event banners |
| event-management-certificates-<account-id>-<region> | Stores PDF certificate files |
| event-management-frontend-<account-id>-<region> | Stores the React frontend build files and serves them through CloudFront |

The FrontendBucket is configured with full public access blocking (BlockPublicAcls/BlockPublicPolicy/IgnorePublicAcls/RestrictPublicBuckets: true) so only CloudFront can access it through Origin Access Control, not directly through the S3 URL.

![S3 buckets](/images/5-Workshop/5.3-Deploy-backend/Buckets.jpg)

---

## Step 10: Check the Cognito User Pool (reference, not creation)

Open Amazon Cognito → User pools and check that the correct User Pool was used as the value for CognitoUserPoolIdParam during deployment.

Note: this User Pool is not created by this stack; the backend only references its ARN to configure the CognitoAuthorizer for API Gateway.

![User Pool](/images/5-Workshop/5.3-Deploy-backend/UserPool.jpg)

---

## Step 11: Check Lambda functions and CloudWatch Logs

Open `AWS Lambda → Function`s and verify that the 6 functions were created: EventLambda, UserProfileLambda, TicketLambda, AttendanceCertificateLambda, NotificationLambda, AnalyticsLambda.

![Lambda functions](/images/5-Workshop/5.3-Deploy-backend/Functions.jpg)

Then open CloudWatch → Log groups and look for log groups named /aws/lambda/<function-name>. These logs are useful when debugging API, authentication, or business logic errors.

---

## Step 12: Check the CloudFront distribution

Open `Amazon CloudFront → Distributions` and verify the distribution created for the frontend.

Try accessing the CloudFront domain (in the form <id>.cloudfront.net). If the frontend has not yet been built and uploaded to the FrontendBucket, this step will not display the UI yet (this is covered in section 5.11).

![CloudFront distribution](/images/5-Workshop/5.3-Deploy-backend/CloudFront.jpg)

---

## Common deployment errors

### AWS credentials are not configured

```powershell
aws configure
aws sts get-caller-identity
```

### SAM build fails

```powershell
sam build --no-cached
```

Check the .NET SDK version:

```powershell
dotnet --version
```

### Deployment reports that CognitoUserPoolIdParam is missing

If you are not using --guided, pass the parameter directly:

```powershell
sam deploy --parameter-overrides CognitoUserPoolIdParam=<real Cognito User Pool ID>
```

### CloudFormation stack rolls back

Open CloudFormation → Stacks → <stack-name> → Events and read the first error event (usually the root cause), then fix the related resource or permission issue.

---

## Expected outcomes

After completing this section:

- The backend stack with 6 Lambda functions, 6 DynamoDB tables, 4 S3 buckets, and a CloudFront Distribution is deployed successfully.
- The API Gateway endpoint and CloudFront domain are ready to be configured for the frontend.
- The Cognito Authorizer is linked to the correct existing User Pool.
- The project is ready for authentication setup (5.4) and the following business flows.