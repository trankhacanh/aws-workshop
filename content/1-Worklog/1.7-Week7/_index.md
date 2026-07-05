---
title: "Week 7 Worklog"
date: 2026-06-01
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives

* Study the core AWS services used in the Serverless project, including AWS Lambda, AWS SAM, Amazon API Gateway, Amazon Cognito, Amazon DynamoDB, and Amazon SES.
* Understand the overall system architecture and how AWS services interact to process business workflows.
* Become familiar with the project's source code structure and the responsibilities of each Lambda module in preparation for feature development and maintenance in the following weeks.

### Tasks Completed This Week

| Day | Task | Start Date | Completion Date | Reference |
| --- | --- | --- | --- | --- |
| Monday | **Project Technology Research:**<br>- Studied AWS Lambda fundamentals, including the execution model, cold starts, resource limits (memory and timeout), and the .NET 8 runtime.<br>- Learned the lifecycle of an AWS Lambda function and how to implement a basic Lambda handler.<br>- Studied AWS Serverless Application Model (AWS SAM), including the structure of `template.yaml` and the syntax for `Resources`, `Globals`, and `Events`. | 01/06/2026 | 01/06/2026 | |
| Tuesday | - Studied Amazon API Gateway, including REST APIs, Lambda Proxy Integration, and CORS configuration.<br>- Reviewed Amazon Cognito documentation, including User Pools, App Clients, and API Gateway Authorizers for authentication and authorization. | 02/06/2026 | 02/06/2026 | |
| Wednesday | - Studied Amazon DynamoDB concepts and architecture.<br>- Learned how to perform common DynamoDB operations using the AWS SDK for .NET, including `PutItem`, `GetItem`, and `Query`. | 03/06/2026 | 03/06/2026 | |
| Thursday | - Studied the architecture and source code of the **EventLambda** module.<br>- Learned how Amazon Simple Email Service (Amazon SES) is used for email delivery within the project. | 04/06/2026 | 04/06/2026 | |
| Friday | - Reviewed the overall system architecture to better understand data storage, event-driven workflows, and service interactions.<br>- Analyzed the four Lambda modules developed by the team:<br>&emsp;+ **EventLambda**<br>&emsp;+ **UserProfileLambda** (assigned to Thịnh)<br>&emsp;+ **RegistrationTicketLambda** (assigned to Tâm)<br>&emsp;+ **AttendanceCertificateLambda** (assigned to Thư Kỳ)<br>- Studied the project's coding conventions, shared project structure, and common AWS resources such as Amazon DynamoDB tables and Amazon S3 buckets. | 05/06/2026 | 05/06/2026 | |

### Week 7 Achievements

* Developed a solid understanding of AWS Lambda, including its execution lifecycle, handlers, cold starts, resource limitations, and the .NET 8 runtime.
* Learned AWS SAM and understood the structure of `template.yaml` as well as how serverless resources and events are defined.
* Studied Amazon API Gateway, including Lambda Proxy Integration, CORS configuration, and API integration with AWS Lambda.
* Learned how Amazon Cognito manages authentication through User Pools, App Clients, and API Gateway Authorizers.
* Gained practical knowledge of Amazon DynamoDB and its basic operations using the AWS SDK for .NET, including `PutItem`, `GetItem`, and `Query`.
* Understood how Amazon SES is integrated into the system to send emails.
* Analyzed the overall serverless architecture, including the event-driven workflow, data flow, and interactions among AWS services.
* Reviewed the project's Lambda modules, source code organization, shared Amazon DynamoDB tables, and Amazon S3 resources, establishing a solid foundation for participating in feature development in the following weeks.