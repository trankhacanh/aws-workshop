---
title: "Week 11 Worklog"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 Objectives

* Complete the deployment of the frontend application on AWS using Amazon S3 and Amazon CloudFront.
* Configure related AWS services, including Amazon Cognito, CloudFront, and frontend environment variables, to ensure the application runs correctly in the production environment.
* Set up system monitoring with Amazon CloudWatch, perform comprehensive system testing, and finalize the source code and project documentation before completing the project.

### Tasks Completed This Week

| Day | Task | Start Date | Completion Date | Reference |
| --- | --- | --- | --- | --- |
| Monday | - Designed and added **Amazon S3 Bucket** and **Amazon CloudFront Distribution** resources (with **Origin Access Control**) to the AWS SAM `template.yaml` file for hosting and delivering the frontend application. | 29/06/2026 | 29/06/2026 | |
| Tuesday | - Updated the **Amazon Cognito App Client** configuration, including the Allowed Callback URLs and Sign-out URLs.<br>- Updated frontend environment variables to use the production CloudFront domain. | 30/06/2026 | 30/06/2026 | |
| Wednesday | - Reviewed and fixed remaining TypeScript compilation errors that prevented the frontend from building.<br>- Successfully built the frontend application.<br>- Deployed the application to Amazon S3 and performed an Amazon CloudFront cache invalidation to publish the latest version. | 01/07/2026 | 01/07/2026 | |
| Thursday | - Configured **Amazon CloudWatch Logs** and **CloudWatch Alarms** to monitor AWS Lambda functions.<br>- Reviewed runtime logs across the system to verify deployment health and identify potential issues. | 02/07/2026 | 02/07/2026 | |
| Friday | - Performed end-to-end testing of all implemented business workflows.<br>- Cleaned up and optimized the source code.<br>- Consolidated the system architecture, implementation details, and project outcomes for inclusion in the final project report. | 03/07/2026 | 03/07/2026 | |

### Week 11 Achievements

* Successfully designed and provisioned **Amazon S3 Bucket** and **Amazon CloudFront Distribution** resources (using **Origin Access Control**) in the AWS SAM `template.yaml`, completing the infrastructure required to host and distribute the frontend application.
* Updated the **Amazon Cognito App Client** configuration, including Callback URLs, Sign-out URLs, and frontend environment variables, ensuring that user authentication worked correctly in the production environment.
* Resolved all remaining TypeScript compilation issues, successfully built the frontend application, deployed it to Amazon S3, and refreshed the Amazon CloudFront cache to serve the latest version.
* Configured **Amazon CloudWatch Logs** and **CloudWatch Alarms** to monitor AWS Lambda functions, inspect runtime logs, and support post-deployment troubleshooting.
* Successfully completed end-to-end testing of all major system workflows, including user authentication, event management, ticket registration, email notifications, attendance tracking, certificate generation, and analytics reporting.
* Cleaned up and optimized the project source code, documented the overall system architecture, summarized the technologies used and the implementation results, and finalized the project report upon completion of the development phase.