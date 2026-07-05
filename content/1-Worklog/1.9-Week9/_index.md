---
title: "Week 9 Worklog"
date: 2026-06-15
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Week 9 Objectives

* Complete and test the event reminder email workflow using Amazon EventBridge Scheduler.
* Develop the **Analytics** module to collect and aggregate system statistics from Amazon DynamoDB and Amazon S3.
* Build the **Analytics Dashboard** on the frontend and integrate it with the analytics APIs to help administrators monitor the system.

### Tasks Completed This Week

| Day | Task | Start Date | Completion Date | Reference |
| --- | --- | --- | --- | --- |
| Monday | - Tested the event reminder email workflow using **Amazon EventBridge Scheduler** by simulating scheduled events.<br>- Verified that the system correctly retrieved event data and successfully delivered reminder emails to registered users. | 15/06/2026 | 15/06/2026 | |
| Tuesday | - Analyzed the data structures of the **Ticket** and **Attendance** tables from other team members' modules.<br>- Studied the relationships between the data models.<br>- Designed the **AnalyticsRepository** to aggregate system statistics, including total events, registrations, and attendee check-ins. | 16/06/2026 | 16/06/2026 | |
| Wednesday | - Implemented **AnalyticsRepository**.<br>- Collected statistics from Amazon DynamoDB using **Scan** and **Query** operations.<br>- Counted the number of generated certificates by listing objects stored in the Amazon S3 bucket. | 17/06/2026 | 17/06/2026 | |
| Thursday | - Implemented `Function.cs` for **AnalyticsLambda**.<br>- Updated `template.yaml` with environment variables and the required **DynamoDBReadPolicy** and **S3ReadPolicy** permissions.<br>- Deployed the Lambda function to AWS and tested the Dashboard APIs. | 18/06/2026 | 18/06/2026 | |
| Friday | - Developed the **AnalyticsPage** using React.<br>- Integrated the Dashboard API and event-specific analytics APIs.<br>- Updated the application's routing configuration and administrator navigation menu to provide access to the analytics features. | 19/06/2026 | 19/06/2026 | |
| Saturday | - Attended a company technical event and discussed the system development progress with project members while gaining additional practical experience in AWS Serverless application development. | 20/06/2026 | 20/06/2026 | |

### Week 9 Achievements

* Successfully tested the event reminder email workflow by simulating **Amazon EventBridge Scheduler** events and verified that the system retrieved the correct data and delivered reminder emails successfully.
* Analyzed the data structures of the **Ticket** and **Attendance** tables, identified their relationships, and designed the **AnalyticsRepository** to support reporting and analytics features.
* Implemented **AnalyticsRepository** to collect statistics such as the total number of events, registrations, attendee check-ins from Amazon DynamoDB, and issued certificates from Amazon S3.
* Completed the implementation of **AnalyticsLambda**, updated the AWS SAM `template.yaml` with the required environment variables and IAM permissions, successfully deployed the function to AWS, and verified the Dashboard APIs.
* Developed the **AnalyticsPage** using React, integrated both the Dashboard API and event-specific analytics APIs, and updated the application's routing and administrator menu to support the new analytics functionality.
* Participated in a company technical event, exchanged development experiences with project members, and gained additional practical knowledge of building Serverless applications on AWS.