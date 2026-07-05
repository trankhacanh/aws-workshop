---
title: "Week 10 Worklog"
date: 2026-06-22
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 Objectives

* Complete the event-driven communication architecture between system modules using Amazon EventBridge.
* Gain a deeper understanding of the **RegistrationTicketLambda** module and the ticket registration workflow to better understand how the serverless modules interact.
* Complete the backend deployment on AWS and prepare the environment for frontend deployment and project completion.
* Organize the internship documentation, create the report template, and consolidate the knowledge gained throughout the project.

### Tasks Completed This Week

| Day | Task | Start Date | Completion Date | Reference |
| --- | --- | --- | --- | --- |
| Monday | - Refactored the notification workflow by replacing direct AWS Lambda invocation with **Amazon EventBridge PutEvents**.<br>- Standardized the event payload format across modules to improve scalability and reduce tight coupling between services. | 22/06/2026 | 22/06/2026 | |
| Tuesday | - Studied the architecture and source code of the **RegistrationTicketLambda** module.<br>- Learned how ticket information is stored in Amazon DynamoDB.<br>- Studied the use of a **Global Secondary Index (GSI)** on **UserId**.<br>- Analyzed the atomic update mechanism used to prevent duplicate ticket registrations.<br>- Learned how registration events are published through Amazon EventBridge after successful ticket registration. | 23/06/2026 | 23/06/2026 | |
| Wednesday | - Reviewed the overall AWS architecture of the project.<br>- Analyzed the end-to-end data flow and interactions among AWS services to gain a deeper understanding of the serverless application architecture. | 24/06/2026 | 24/06/2026 | |
| Thursday | - Prepared the internship report template.<br>- Organized project documentation and summarized the research topics, implementation process, and project outcomes. | 25/06/2026 | 25/06/2026 | |
| Friday | - Deployed the backend application to AWS.<br>- Identified and resolved deployment issues.<br>- Verified that the backend APIs were functioning correctly in preparation for deploying the frontend application to Amazon S3. | 26/06/2026 | 26/06/2026 | |
| Saturday | - Attended a company technical event and exchanged project development experiences with team members while learning additional best practices for AWS Serverless application development. | 27/06/2026 | 27/06/2026 | |

### Week 10 Achievements

* Successfully migrated the notification workflow from direct AWS Lambda invocation to an **Amazon EventBridge** event-driven architecture using **PutEvents**, while standardizing event payloads across modules to improve scalability and reduce service coupling.
* Gained an in-depth understanding of the **RegistrationTicketLambda** module, including Amazon DynamoDB data modeling, the use of a **Global Secondary Index (GSI)** on **UserId**, atomic update operations to prevent duplicate registrations, and event publishing through Amazon EventBridge.
* Analyzed the complete AWS architecture of the project and gained a clear understanding of how AWS Lambda, Amazon API Gateway, Amazon EventBridge, Amazon DynamoDB, Amazon S3, Amazon SES, and Amazon Cognito interact within the serverless architecture.
* Created the internship report template and organized the knowledge, implementation activities, and achievements accumulated throughout the internship.
* Successfully deployed the backend application to AWS, resolved deployment issues, and ensured that all APIs were stable and ready for the frontend deployment to Amazon S3.
* Participated in a company technical event, exchanged development experiences with team members, and gained additional practical knowledge of AWS services and Serverless architecture.