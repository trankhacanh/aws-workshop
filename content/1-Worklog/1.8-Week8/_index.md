---
title: "Week 8 Worklog"
date: 2026-06-08
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 Objectives

* Study AWS services used for event processing and application deployment, including Amazon EventBridge, Amazon S3, Amazon CloudFront, AWS IAM, and Amazon CloudWatch.
* Begin developing the **NotificationLambda** module by integrating Amazon SES and Amazon DynamoDB for email delivery and notification logging.
* Complete the notification workflow, deploy the solution to AWS, and perform end-to-end testing of the email notification process.

### Tasks Completed This Week

| Day | Task | Start Date | Completion Date | Reference |
| --- | --- | --- | --- | --- |
| Monday | - Studied Amazon EventBridge, including Event Buses, Rules, Pattern Matching, and EventBridge Scheduler (Rate/Cron).<br>- Simulated the end-to-end ticket registration workflow using mock data to understand the event-driven architecture. | 08/06/2026 | 08/06/2026 | |
| Tuesday | - Reviewed Amazon S3, including Static Website Hosting, Bucket Policies, and Block Public Access.<br>- Studied Amazon CloudFront for future frontend deployment.<br>- Learned AWS IAM concepts, including Roles, Policies, and the Principle of Least Privilege.<br>- Studied Amazon CloudWatch, including Log Groups, Alarms, and Metric Filters. | 09/06/2026 | 09/06/2026 | |
| Wednesday | - Designed the **EventManagementNotificationLog** table in Amazon DynamoDB.<br>- Updated the `template.yaml` file to add the **NotificationFunction**, including its Lambda configuration, IAM Role, and required IAM Policies.<br>- Implemented **NotificationRepository**, **NotificationLogDynamoMapper**, and **SesEmailService** to send emails via Amazon SES and store notification logs in DynamoDB. | 10/06/2026 | 10/06/2026 | |
| Thursday | - Implemented `Function.cs` for **NotificationLambda**.<br>- Added support for three trigger types:<br>&emsp;+ API Gateway (retrieve notification history).<br>&emsp;+ Amazon EventBridge Rule (registration confirmation emails).<br>&emsp;+ EventBridge Scheduler (event reminder emails). | 11/06/2026 | 11/06/2026 | |
| Friday | - Deployed **NotificationLambda** to AWS for testing.<br>- Debugged and resolved a runtime issue caused by an incorrect DynamoDB primary key attribute name.<br>- Successfully completed end-to-end testing of the registration confirmation email workflow. | 12/06/2026 | 12/06/2026 | |
| Saturday | - Attended a company technical event and participated in discussions about real-world AWS application development and cloud engineering best practices. | 13/06/2026 | 13/06/2026 | |

### Week 8 Achievements

* Studied Amazon EventBridge, including Event Buses, Rules, Pattern Matching, and EventBridge Scheduler (Rate/Cron), and simulated an event-driven ticket registration workflow using mock data.
* Gained a better understanding of Amazon S3, Amazon CloudFront, AWS IAM, and Amazon CloudWatch, including their roles in storage, content delivery, access control, and monitoring.
* Designed the **EventManagementNotificationLog** table in Amazon DynamoDB and updated the AWS SAM `template.yaml` file to provision resources for **NotificationLambda**.
* Developed the **NotificationRepository**, **NotificationLogDynamoMapper**, and **SesEmailService** components to send emails via Amazon SES and store notification history in DynamoDB.
* Completed the implementation of **NotificationLambda**, supporting three trigger types: API Gateway, Amazon EventBridge Rule, and EventBridge Scheduler.
* Successfully deployed **NotificationLambda** to AWS, identified and resolved a runtime issue related to the DynamoDB primary key, and validated the complete registration confirmation email workflow through end-to-end testing.
* Participated in a company technical event, gaining practical insights into AWS-based application development and cloud engineering practices.