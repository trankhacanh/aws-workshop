---
title: "Workshop Overview"
date: 2026-06-29
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

---

## Introduction

This workshop introduces how to deploy and test the Event Management Platform, a system that supports the full lifecycle of organizing and managing events, built with a serverless architecture on AWS.

The real-world problem this project solves is that many event organizers still rely on Google Forms, Excel sheets, and manual emails for registration and coordination. This makes it hard to control actual attendance, manually send confirmations, prevent ticket fraud, and evaluate event performance after the event. The Event Management Platform digitizes the entire process on a serverless platform.

The Event Management Platform is designed for two primary roles:

- User (Attendee): uses the web application to view events, register, receive digital tickets, scan QR codes at the event, and download certificates after completion.
- Admin: manages events (create/edit/delete), categories, banners, tracks registration and check-in counts in real time, and views the analytics dashboard.

The workshop focuses on deploying the entire serverless backend infrastructure (6 Lambda functions) with AWS SAM, deploying the frontend through CloudFront, and running a real demo of the application.



---

## Workshop objectives

After completing this workshop, participants should be able to:

- Understand the serverless architecture of the Event Management Platform with 6 independent Lambda functions.
- Deploy the backend with AWS SAM and manage resources through AWS CloudFormation.
- Use Amazon Cognito for authentication and issue JWT tokens.
- Use API Gateway to provide backend APIs and distinguish between public and protected routes.
- Use Lambda (.NET 8) to implement business logic for event management, user profiles, ticket registration, check-in, certificate issuance, notifications, and data analytics.
- Use DynamoDB to store application data.
- Use S3 to store event banners, user avatars, PDF certificates, and frontend build assets.
- Use SES to send certificates and notification emails.
- Use EventBridge (default event bus + Scheduler) for asynchronous notifications and automatic reminders instead of direct calls or SNS/SQS.
- Deploy the React frontend through Amazon CloudFront with Origin Access Control (OAC).
- View backend logs in CloudWatch Logs.
- Test the main end-to-end business flows for both users and admins.

---

## AWS services and tools used

This workshop uses the following AWS services and tools:

| Service / Tool | Purpose |
|---|---|
| Amazon Cognito | User authentication, JWT issuance, User/Admin authorization |
| Amazon API Gateway | Backend API exposure; most routes are protected by the default Cognito Authorizer, while some public read routes are configured separately |
| AWS Lambda (.NET 8) | Runs backend business logic through 6 functions: EventFunction, UserProfileFunction, TicketFunction, AttendanceCertificateFunction, NotificationFunction, AnalyticsFunction |
| Amazon DynamoDB | Stores events, categories, tickets, attendance, users, and notification logs |
| Amazon S3 | Stores event banners, user avatars, PDF certificates, and frontend build assets |
| Amazon SES | Sends participation certificates and notifications/reminders |
| Amazon EventBridge (Default Event Bus + Scheduler) | Handles asynchronous notifications when tickets are registered and sends periodic reminders before events |
| Amazon CloudFront + OAC | Distributes the React frontend from a private S3 bucket without public access |
| Amazon CloudWatch Logs | Monitors Lambda/API logs and supports debugging |
| AWS SAM | Builds and deploys the serverless backend |
| AWS CloudFormation | Manages AWS resources via stack deployment |

![Serverless Event Platform Architecture Diagram](/images/5-Workshop/5.1-Workshop-overview/system-architecture.jpg)

---

## Services outside the workshop scope

The following services are not included in the current workshop scope:

| Service | Status |
|---|---|
| AWS Amplify Hosting | Not used; CloudFront + S3 is used instead |
| Amazon EC2 | Not used |
| Amazon VPC | Not used |
| Amazon RDS | Not used |
| Amazon SNS / SQS | Not used for notification flows; EventBridge is used instead |

---

## Current project technology and structure

The project source code is divided into two main parts:

- Backend (BE): serverless architecture on AWS Lambda (.NET 8/C#), using AWS SAM (template.yaml) as Infrastructure as Code, organized into src/Functions (6 Lambda functions) and src/Shared (shared services, repositories, DTOs, validators, constants).
- Frontend (FE): React 18+ with TypeScript, built with Vite, organized by feature folders such as components/events, components/tickets, components/certificates, components/notifications, and uses Axios for API calls and Cognito for authentication.

| ![Image 1](/images/5-Workshop/5.1-Workshop-overview/BE.jpg) | ![Image 2](/images/5-Workshop/5.1-Workshop-overview/FE.jpg) |
|---|---|

---

## Deployment information

The current backend deployment configuration is as follows:

| Item | Value |
|---|---|
| AWS Region | ap-southeast-1 |
| CloudFormation Stack | aws-event-management |
| API Gateway Endpoint | https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev |
| Cognito User Pool ID | ap-southeast-1_XXXXXXXXX |
| CloudFront Domain | <distribution-id>.cloudfront.net |

The real identifiers (Pool ID, endpoint, domain) are managed through deployment parameters and environment variables and are not committed to source control; they will be provided directly during the demo.

---

## Main demo flow

The workshop demo follows this flow:

1. Admin signs in and creates a category and a new event with a banner.
2. The user signs up or signs in through Cognito.
3. The user views the public event list and selects an event to join.
4. The user submits a ticket registration request; the system checks the remaining slots and creates a CONFIRMED ticket.
5. The system publishes a TicketRegistered event to EventBridge; NotificationFunction sends a confirmation email via SES.
6. The user views their personal ticket through GET /my-tickets.
7. As the event approaches, the EventBridge Scheduler triggers NotificationFunction to scan and send reminder emails.
8. At the event venue, the user presents the QR code (containing the TicketId) for check-in.
9. After check-in, the user requests a certificate; the system generates a PDF, uploads it to S3, and sends it via SES.
10. Admin views the analytics dashboard: registration counts, attendance rate, and notification logs for each event.

---

## Expected outcomes

After the workshop, the backend with 6 Lambda functions will be deployed successfully to AWS, and the React frontend will be built and distributed through CloudFront so it can connect to the deployed backend.

Participants should be able to test the main flows of the Event Management Platform: event management, ticket registration, QR-based check-in, certificate issuance, automatic notifications/reminders through EventBridge, and event-performance analytics.