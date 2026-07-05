---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Cloud-Based Event Management Platform Workshop

## Deploying the serverless backend and running a demo application

This workshop introduces how to deploy and test the Event Management Platform, a system that supports the full process of organizing and managing events, built with a serverless architecture on AWS.

The Event Management Platform supports creating and managing events, categories, user profiles, event registration, digital ticket issuance, QR-based check-in, attendance tracking, certificate issuance, automated notifications and reminders, and an analytics dashboard. The target audience is technology workshops and seminars for the developer community, replacing manual workflows based on Google Forms and Excel, which are difficult to control in real time, require manual email sending, and lack reliable event-performance data.

The main goal of this workshop is to guide participants through deploying the backend using AWS serverless services, deploying the frontend via CloudFront, and testing the main business flows, including event management, ticket registration, check-in, certificate issuance, notifications/reminders, and analytics.

---

## Workshop scope

This workshop focuses on the AWS services and tools currently used in the project:

- Amazon Cognito for user authentication with JWT, used as the basis for extracting user claims in business Lambda functions.
- Amazon API Gateway to provide business routes, most of which are protected by the default Cognito Authorizer (some public read routes such as GET /events and GET /categories are marked Authorizer: NONE).
- AWS Lambda (.NET 8/C#) to handle backend business logic, consisting of 6 functions: EventFunction, UserProfileFunction, TicketFunction, AttendanceCertificateFunction, NotificationFunction, and AnalyticsFunction.
- Amazon DynamoDB to store application data in the tables EventManagementEvents, EventManagementCategories, EventManagementTickets, EventManagementAttendance, EventManagementUsers, and EventManagementNotificationLog.
- Amazon S3 to store event banners, user avatars, PDF certificates, and the frontend build files.
- Amazon SES to send certificates and notification emails to users (the sender address is configured via the SES_FROM_EMAIL environment variable and must be verified before use).
- Amazon EventBridge (default event bus) to process asynchronous notifications: TicketFunction publishes events to the default event bus with source eventmanagement.ticket and detail-type TicketRegistered; NotificationFunction listens through an EventBridge Rule.
- Amazon EventBridge Scheduler (rate(1 hour)) to periodically trigger reminder emails for upcoming events.
- Amazon CloudFront and Origin Access Control (OAC) to distribute the built React frontend from a fully private S3 bucket (no public access), currently using the default *.cloudfront.net domain.
- Amazon CloudWatch Logs to monitor and debug Lambda and API Gateway errors.
- AWS SAM to build and deploy the entire backend infrastructure through template.yaml.

The React frontend (TypeScript + Vite) is built as static assets and deployed through Amazon CloudFront, communicating with the backend via Axios and authenticating with Cognito.

---

## System architecture

The Event Management Platform uses a serverless architecture deployed in the AWS Singapore region: ap-southeast-1.

The React frontend communicates with Amazon Cognito for authentication and with Amazon API Gateway for backend business operations. API Gateway validates the JWT token issued by Cognito before routing protected requests to the corresponding Lambda function; a few public read routes (event list, categories) and the check-in/certificate routes are configured without requiring JWT. Lambda handles the main business logic (event management, user profiles, ticket registration, check-in, certificate generation, notifications, analytics) and reads/writes data to DynamoDB. S3 is used to store banners, avatars, and PDF certificates.

The notification flow is designed with two separate branches routed through Amazon EventBridge:

- Ticket registration notification: after a ticket is created successfully, TicketFunction publishes a TicketRegistered event to EventBridge; NotificationFunction listens and sends a confirmation email via SES.
- Reminder flow: an EventBridge Schedule runs every hour and triggers NotificationFunction to scan upcoming events and send reminder emails.

On the frontend side, the built React app is uploaded to a completely public-blocked S3 bucket, and CloudFront reads it through Origin Access Control (OAC) rather than direct S3 access. CloudFront handles 403/404 errors by returning index.html so React Router can manage client-side routing. The current setup uses the default AWS-provided domain (*.cloudfront.net) and does not yet use a custom domain or ACM certificate.

![Serverless Event Platform Architecture Diagram](/images/5-Workshop/5.1-Workshop-overview/system-architecture.jpg)
<p style="text-align: center;"><i>Figure 5: Architecture diagram and serverless data-flow on AWS.</i></p>

---

## Workshop sections

### [5.1 - Workshop Overview](5.1-Workshop-overview/)
This section introduces the Event Management Platform project, the workshop goals, the system architecture, and the main AWS services used.

### [5.2 - Environment Preparation](5.2-Prerequiste/)
This section lists the tools, accounts, access permissions, and local environment required before deploying the backend and running the application.

### [5.3 - Deploy Backend with AWS SAM](5.3-Deploy-backend/)
This section explains how to build and deploy the backend with AWS SAM and CloudFormation.

### [5.4 - Configure Authentication with Amazon Cognito](5.4-Configure-authentication/)
This section explains how Amazon Cognito is used for registration, sign-in, JWT token issuance, and User/Admin authorization.

### [5.5 - Event and Category Management](5.5-Event-management-flow/)
This section demonstrates the Admin flow for creating/editing/deleting events, managing categories, uploading banners via presigned URLs, and toggling event visibility.

### [5.6 - User Profile Management](5.6-User-profile-flow/)
This section demonstrates the flow for creating, viewing, and updating user profiles, along with avatar upload via presigned URL.

### [5.7 - Ticket Registration Flow](5.7-Registration-ticketing-flow/)
This section demonstrates ticket registration, the overbooking-prevention mechanism, and saving tickets with the CONFIRMED status.

### [5.8 - QR Check-in and PDF Certificate Flow](5.8-Checkin-certificate-flow/)
This section demonstrates QR-based check-in and the flow for creating and sending PDF certificates by email.

### [5.9 - Automated Notifications and Reminders](5.9-Notification-reminder-flow/)
This section demonstrates two notification flows: confirmation email via EventBridge when a ticket is registered successfully and periodic reminder emails via EventBridge Scheduler before the event.

### [5.10 - Dashboard and Event Analytics](5.10-Analytics-dashboard/)
This section shows how to view the overall dashboard and event-level analytics derived from Events, Tickets, Attendance, and Notification Log data.

### [5.11 - Deploy Frontend via CloudFront](5.11-Deploy-frontend-cloudfront/)
This section explains how to build the React frontend, upload it to a private S3 bucket, and deploy it through Amazon CloudFront with Origin Access Control.

### [5.12 - CloudWatch Monitoring and Resource Cleanup](5.12-Monitoring-and-cleanup/)
This section explains how to view Lambda/API Gateway logs with CloudWatch Logs and clean up DynamoDB, S3, CloudFront, and Lambda resources after the workshop.

---

## Expected outcomes

After completing this workshop, participants should be able to:

- Understand the serverless architecture of the Event Management Platform with 6 independent Lambda functions.
- Deploy the backend with AWS SAM and CloudFormation.
- Use Amazon Cognito for authentication and extracting user claims in Lambda.
- Test the event management, category, and banner upload flows.
- Test the user profile and avatar upload flows.
- Test the ticket registration flow, including the overbooking-prevention mechanism using DynamoDB conditions.
- Test the QR check-in flow and the PDF certificate generation/sending workflow via SES.
- Test the asynchronous notification flow through EventBridge when a ticket is registered and the periodic reminder flow through EventBridge Scheduler.
- View the event analytics dashboard.
- Deploy the React frontend and distribute it through Amazon CloudFront.
- View backend logs with CloudWatch Logs and clean up AWS resources after testing.

---

## Important notes

This workshop is designed for a development and demo environment based on the current project state.

Regarding security (IAM and Authorization): three routes under AttendanceCertificateFunction — POST /tickets/checkin, GET /tickets/{ticketId}, and GET /certificates-v2/{ticketId} — are currently declared with Auth: Authorizer: NONE in template.yaml, meaning they do not require a JWT token when called, unlike the other routes, which are generally protected by the Cognito Authorizer. This should be considered for future hardening with proper authentication or signed short-lived tokens.

The QR validation mechanism currently only checks whether a TicketId exists in the ticket table; it does not yet include digital signatures, expiration, or event/session-level restrictions.

The current template.yaml contains a CognitoUserPoolIdParam placeholder value that must be replaced with the real User Pool ID during deployment (sam deploy --parameter-overrides CognitoUserPoolIdParam=<PoolId>).

CloudFront currently uses the default domain provided by AWS (*.cloudfront.net) and is not yet configured with a custom domain. If a custom domain such as app.example.com is required later, an ACM certificate must be created in us-east-1 and configured through the CloudFront Aliases and ViewerCertificate settings.

The sender email address (SES_FROM_EMAIL) is environment-specific; for public reports or workshops, replace it with a placeholder rather than a real address.