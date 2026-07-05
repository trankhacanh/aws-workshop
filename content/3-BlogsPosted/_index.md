---
title: "Blogs Posted"
date: 2026-07-03
weight: 3
chapter: false
pre: " <b> 3. </b> "
---


###  [Blog 1 - Building a Hybrid Multi-Tenant SaaS Architecture for Stateful Services on AWS](3.1-Blog1/)
This article introduces a Hybrid Architecture solution on AWS, combining the Isolation (Silo) and Shared (Pool) models for stateful services (such as Game Servers and real-time Chat). The system optimizes costs using the Pool model for small customers and guarantees performance for VIP/Enterprise customers through independent Silo partitions, utilizing DynamoDB as a dynamic "Routing Table" along with Amazon EKS and ElastiCache (Redis) to synchronize session states in real-time.

###  [Blog 2 - Building a Completely Serverless Notification, Analytics, and Deployment System on AWS](3.2-Blog2/)
This article shares the experience of deploying a Workshop management project using a completely Serverless architecture and Event-Driven thinking on AWS. The system utilizes Amazon EventBridge as a centralized event distribution hub to decouple components, combined with AWS Lambda and Amazon SES for automated email delivery; DynamoDB for aggregating Dashboard metrics; and CloudWatch for comprehensive monitoring and operations.

###  [Blog 3 - Deep Dive into AWS Lambda Concurrency – What Really Happens When Thousands of Requests Call a Lambda Simultaneously?](3.3-Blog3/)
This article explores the inner workings of AWS Lambda Concurrency and its auto-scaling mechanisms. It clarifies common misconceptions regarding thread allocation by highlighting the Execution Environment lifecycle. Additionally, the post differentiates between Cold Starts and Warm Starts, details scaling behaviors under sudden traffic spikes, and provides strategic insights into managing Reserved Concurrency and Provisioned Concurrency to mitigate throttling and optimize system performance.