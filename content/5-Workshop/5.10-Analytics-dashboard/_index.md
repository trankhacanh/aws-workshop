---
title: "Dashboard & Event Performance Analytics"
date: 2026-06-29
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

---

This section walks through viewing the overview dashboard and the analytics for individual events, aggregating data from 4 tables: Events, Tickets, Attendance, and Notification Log, along with the S3 bucket that stores certificates.

This business logic is handled by AnalyticsFunction, which only has read permissions on all related resources — no write permissions, consistent with its role as a purely aggregation/reporting module.

---

## Overview Dashboard Flow

The overview dashboard route returns system-wide aggregate metrics:

| Metric | Calculation Source |
|---|---|
| Total events | Count of all items in the Events table |
| Tickets issued | Count of all items in the Tickets table |
| Confirmed | Number of tickets with a status other than Cancelled |
| Attendees | Count of all items in the Attendance table |
| Average attendance rate | Calculated from the check-in ratio over confirmed tickets |
| Certificates issued | Count of actual files in S3 under the certificates/ prefix — counts the actual PDF files rather than relying on a status flag in DynamoDB |
| Emails sent / failed | Counted by Sent / Failed status in the Notification Log table |

Worth noting: **"Certificates issued" is counted by directly listing objects in S3**, rather than counting via a status flag in DynamoDB — this ensures the number accurately reflects the actual PDF files that exist, avoiding discrepancies if there's a logging error during the certificate creation step.

---

## Per-Event Analytics Flow

The per-event analytics route returns detailed metrics for a specific event (passing in an eventId):

| Metric | Calculation Source |
|---|---|
| Total registrations | Count of tickets by EventId |
| Confirmed | Number of the event's tickets with a status other than Cancelled |
| Attended | Direct Query on the Attendance table by EventId (Partition Key — uses Query instead of Scan for performance optimization) |
| Attendance rate | (Number of check-ins / Number confirmed) × 100, rounded to 1 decimal place |

---

## Step 1: Test Retrieving the Overview Dashboard

```powershell
Invoke-RestMethod -Uri "$baseUrl/admin/analytics/dashboard" -Method Get -Headers $headers
```

Expected response:

```json
{
  "totalEvents": "<Total number of events created>",
  "totalRegistrations": "<Total number of tickets issued>",
  "confirmedRegistrations": "<Number of confirmed tickets>",
  "totalCheckIns": "<Total number of check-ins>",
  "averageAttendanceRate": "<Average attendance rate, %>",
  "emailSentCount": "<Number of emails sent successfully>",
  "emailFailedCount": "<Number of emails that failed to send>",
  "certificatesIssuedCount": "<Number of actual certificate PDF files in S3>"
}
```

![Dashboard](/images/5-Workshop/5.10-Analytics-dashboard/Dashboard.jpg)

---

## Step 2: Test Per-Event Analytics

```powershell
$eventId = "<EventId you want to view in detail>"
Invoke-RestMethod -Uri "$baseUrl/admin/analytics/events/$eventId" -Method Get -Headers $headers
```

Expected response:

```json
{
  "eventId": "<EventId passed in>",
  "eventTitle": "<Corresponding event title>",
  "totalRegistrations": "<Total number of tickets for the event>",
  "confirmedCount": "<Number of confirmed tickets>",
  "checkInCount": "<Actual number of check-ins>",
  "attendanceRate": "<Attendance rate, %>"
}
```

![DashboardEvent](/images/5-Workshop/5.10-Analytics-dashboard/DashboardEvent.jpg)

---

## Step 3: Cross-Check the Metrics Against Raw Data

To confirm the aggregate metrics are accurate, cross-check them against the data already verified in previous sections:

1. The "Tickets issued" count from Step 1 should exactly match the number of items in the Tickets table on DynamoDB (already reviewed in section 5.7).
2. The "Certificates issued" count should exactly match the number of files in the certificates S3 bucket, under the certificates/ folder (already reviewed in section 5.8).
3. The "Emails sent / failed" counts should match the number of Sent / Failed status records in the Notification Log table on DynamoDB (already reviewed in section 5.9).

---

## Expected Outcome

After completing this section, the practitioner should be able to:

- View the overview dashboard with system-wide aggregate metrics.
- View detailed statistics for a specific event.
- Understand how "Certificates issued" is counted directly from S3 rather than relying on a DynamoDB status flag.
- Understand how the attendance rate is calculated based on actual check-ins relative to confirmed tickets.
- Cross-check and confirm that the dashboard metrics exactly match the raw data in DynamoDB and S3 already verified in previous sections.
- This is the final data-summary section of the entire system — completing it means the entire core business lifecycle, from event creation to performance analytics, has been fully tested.