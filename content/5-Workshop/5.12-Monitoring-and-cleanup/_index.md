---
title: "CloudWatch Monitoring and Resource Cleanup"
date: 2026-06-29
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

---

This section walks through viewing logs and monitoring the system's operational metrics using Amazon CloudWatch (Logs, Metrics), and also lists the steps for cleaning up all the resources created during the workshop to avoid unexpected costs after it ends.

The system's Notification business logic is handled by a single C# Lambda function called **NotificationFunction** — but it is triggered by two independent EventBridge sources, without going through SNS/SQS:

1. **Reminder flow**: a schedule called **EventReminderSchedule** (built with EventBridge Scheduler, running once every hour) triggers NotificationFunction directly. The Lambda scans the Events and Tickets tables to find upcoming events, calls SES to send emails, then logs the result to the notification log table.
2. **Ticket registration notification flow**: after the ticket registration function successfully creates a ticket, it publishes an event named **TicketRegistered** to a custom Event Bus. A rule called **OnTicketRegistered** listens for exactly this event and asynchronously triggers NotificationFunction, which calls SES to send an email and then logs the result to the same notification log table.

The EventBridge rule, the EventBridge schedule, and the notification log table are all declared in the project's infrastructure template (template.yaml), managed as code alongside the rest of the system's resources.

The monitoring demo will mainly use the AWS Console — specifically CloudWatch Logs Insights — since it's the most intuitive tool for reading Lambda logs. The cleanup section will use the AWS CLI combined with the SAM CLI to ensure resources are deleted in the correct order.

---

## Part A — CloudWatch Monitoring

### A.1. Log Group for Each Lambda

CloudWatch automatically creates a Log Group for every Lambda function, named after the function itself (following the pattern /aws/lambda/function-name). In this project, there's one log group for each of the following functions:

- Event management
- Ticket registration
- Attendance & certificates
- User profile
- Notifications — this one is special, because it records invocations coming from both sources: the hourly reminder schedule and the ticket-registration event.

On the CloudWatch Console, open **Log groups**, then select the one you want to inspect. Each time the Lambda "cold starts", a new Log Stream is created underneath that log group, so you may see several streams inside one log group.

<!-- > 📌 **Image suggestion:** Capture the list of Log Groups in the CloudWatch Console, showing the names of all the project's Lambda functions. -->

### A.2. Using CloudWatch Logs Insights for Quick Searches

Instead of opening each log stream one by one, CloudWatch offers a built-in search tool called Logs Insights that lets you filter across all the logs in a log group at once, similar to searching text in a document — you don't need to know any special query language to use it.

To use it: open CloudWatch, go to **Logs → Logs Insights**, pick the log group(s) you want to search from the dropdown, choose a time range (e.g. "Last 1 hour"), type the word or phrase you're looking for, then click **Run query**. A few useful searches for this project:

- **Find registrations rejected because the event was full**: select the ticket registration function's log group and search for the phrase that appears in that error message (e.g. "Sold out"). Sort by most recent first and limit the results to the last 20 matches so the list stays readable.
- **Find errors across several functions at once**: you can select multiple log groups at the same time in the same search box, then search for the word "Exception" to see every failure across the whole system within the chosen time range.
- **Tell the two Notification triggers apart**: since NotificationFunction is triggered by two different sources, you can separate them by searching for different keywords — search for "TicketRegistered" to see only the emails sent right after someone registers for a ticket, or search for "reminder" (or the Vietnamese phrase for "upcoming") to see only the emails sent by the hourly schedule.

The Console's Logs Insights editor already comes with sample searches you can pick from a dropdown and adjust by simply swapping in your own keyword — there's no need to memorize any special syntax to get useful results.

<!-- > 📌 **Image suggestion:** Capture the results of the two searches above, clearly showing the separation between the logs of the reminder flow and the logs of the ticket-registration flow. -->

### A.3. Monitoring Lambda, API Gateway, DynamoDB, and EventBridge Metrics

On CloudWatch, under **Metrics → All metrics**, the key metrics to monitor include:

- **Lambda**: number of invocations, errors, duration, and throttles — for each function, with particular attention to NotificationFunction since it's the convergence point of both EventBridge flows.
- **API Gateway**: request count, 4XX errors, 5XX errors, and latency — for each API/stage.
- **DynamoDB**: consumed write capacity and throttled requests — watched on the Tickets/Events tables (write contention when ticket registration opens) and on the notification log table (written to continuously by both notification flows).
- **The EventBridge rule** that listens for ticket registrations: invocation count, failed invocations, and throttled rules. An unusual rise in failed invocations means the ticket-registered event isn't reaching NotificationFunction.
- **The EventBridge schedule** that sends reminders: invocation attempt count and target error count. A target error count above zero means the schedule ran on time but failed to invoke NotificationFunction (either a permissions error or an SES error).

You can build a Dashboard that combines all the metrics above into a single screen to monitor during an end-to-end demo (register a ticket → receive a confirmation email → wait for the scheduled time → receive a reminder email), instead of opening multiple separate tabs.

The system currently has no CloudWatch Alarms configured (for example, an alarm for Lambda errors, for a spike in 5XX errors, or for the reminder schedule's target error count rising above zero). This is a missing item that should be noted in the recommendations section of the report.

<!-- > 📌 **Image suggestion:** Capture a self-built CloudWatch Dashboard combining Lambda invocations/errors, API Gateway request count/4XX/5XX, and EventBridge rule/schedule invocations/failures while testing the registration + reminder flow. -->

---

## Part B — Resource Cleanup

Because the EventBridge rule, the EventBridge schedule, and the notification log table are all declared in the infrastructure template, these resources will be automatically deleted together in Step B.3 below — no separate manual deletion is needed for them. The steps below must be performed in the correct order to avoid dependency-related deletion errors (for example, CloudFormation can't delete an S3 bucket that still contains files).

### B.1. Empty the S3 Buckets Before Deleting the Stack

CloudFormation and SAM won't automatically delete an S3 bucket that still contains files. Each bucket needs to be emptied manually first:

```powershell
aws s3 rm s3://event-management-frontend-<suffix> --recursive
aws s3 rm s3://<EventManagementEventBannersBucket-name> --recursive
aws s3 rm s3://<EventManagementCertificatesBucket-name> --recursive
aws s3 rm s3://<EventManagementUserAvatarsBucket-name> --recursive
```

### B.2. Delete the CloudFront Distribution

CloudFront cannot be deleted immediately while it's turned on or while it's still in a "Deploying" state. It must be disabled first, and you then need to wait for its status to change to "Deployed" before it can actually be deleted:

```powershell
aws cloudfront get-distribution-config --id <distribution-id> > dist-config.json
# In dist-config.json, change "Enabled": true to "Enabled": false, keeping the ETag unchanged

aws cloudfront update-distribution `
  --id <distribution-id> `
  --distribution-config file://dist-config.json `
  --if-match <ETag-from-get-distribution-config>

# Once the status changes to "Deployed" (check via get-distribution), it can then be deleted:
aws cloudfront delete-distribution --id <distribution-id> --if-match <latest-ETag>
```

<!-- > 📌 **Image suggestion:** Capture the CloudFront Console at the moment the Distribution's status changes from "Deploying" to "Disabled/Deployed", meeting the conditions for deletion. -->

### B.3. Delete the SAM Stack (Lambda, API Gateway, DynamoDB, EventBridge Rule/Schedule, Related IAM Roles)

```powershell
sam delete --stack-name event-management-stack --region ap-southeast-1
```

This command deletes the CloudFormation stack along with everything defined in the infrastructure template, including:

- All the Lambda functions (event management, ticket registration, attendance & certificates, user profile, notifications, etc.).
- API Gateway.
- All the DynamoDB tables: events, categories, tickets, attendance, users, and the notification log.
- The EventBridge rule and the EventBridge schedule described earlier.
- The associated IAM roles and policies (including the permissions those resources needed to publish events, run on a schedule, and invoke Lambda functions).
- The S3 buckets (these can only be deleted if they were already emptied in step B.1).

If the delete command reports an error due to a duplicated resource identifier in the template, go to the CloudFormation Console, open the stack, and delete it manually there, resolving each resource that reports a deletion failure one at a time. For the reminder schedule specifically, if it gets stuck failing to delete, check whether it's still actively running or has an execution in progress before attempting to delete it again.

### B.4. Delete the Cognito User Pool

```powershell
aws cognito-idp delete-user-pool --user-pool-id <user-pool-id>
```

The User Pool needs to be deleted with this separate command, since it was created manually in section 5.4 rather than through the infrastructure template — so it won't be removed automatically along with the rest of the stack.

### B.5. Delete the Frontend Bucket (After It's Already Been Emptied in B.1)

```powershell
aws s3api delete-bucket --bucket event-management-frontend-<suffix> --region ap-southeast-1
```

### B.6. Verify No Resources Remain

- **CloudFormation Console**: confirm the stack has reached a fully deleted status.
- **S3 Console**: confirm none of the project's buckets remain.
- **CloudFront Console**: confirm the Distribution has disappeared from the list.
- **DynamoDB Console**: confirm none of the project's tables remain, including the notification log table.
- **EventBridge Console**: confirm the Schedules section no longer contains the reminder schedule, and the Rules section no longer contains the ticket-registration rule.
- **Cognito Console**: confirm the User Pool has been deleted.
- **Billing Console → Cost Explorer**: check that no new costs are being incurred after cleanup — pay particular attention to the reminder schedule, since if it's forgotten, it will keep triggering the Notification function every hour indefinitely.

<!-- > 📌 **Image suggestion:** Capture the CloudFormation Console, S3 Console, and EventBridge Console (Schedules + Rules sections) all showing an empty state with no remaining project resources — as proof that cleanup is fully complete. -->

---

## Expected Outcome

After completing this section, the practitioner should be able to:

- Read and search Lambda logs using CloudWatch Logs Insights to support debugging, distinguishing between the reminder-triggered logs and the ticket-registration-triggered logs on the same Notification function.
- Understand the key groups of metrics to monitor for a serverless system, including the dedicated metrics for the EventBridge rule and the EventBridge schedule.
- Correctly identify the missing item (CloudWatch Alarms) to note in the report's recommendations section.
- Perform a full cleanup of all created AWS resources in the correct dependency order, confirming that the EventBridge rule, the schedule, and the notification log table are handled automatically by the stack deletion.
- Verify proof of complete cleanup on each relevant service console, including EventBridge.