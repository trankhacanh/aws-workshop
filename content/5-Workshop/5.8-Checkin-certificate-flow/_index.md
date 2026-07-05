---
title: "QR Check-In Flow & PDF Certificate Issuance"
date: 2026-06-29
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

---

This section demonstrates the ticket check-in flow via QR code, and the flow for generating and emailing a PDF certificate after a successful check-in.

This entire business logic is handled by AttendanceCertificateFunction, which reads/writes to the Tickets and Attendance tables, stores PDF files in the certificates S3 bucket, and sends emails via Amazon SES.

All 3 routes in this module (check in a ticket, view ticket details, retrieve a certificate) require no JWT — suitable for scanning QR codes at a check-in counter, where the scanning device doesn't need to log in again.

---

## Check-In Flow

1. The scanning device (or any client) sends a check-in request with the ticketId taken from the QR code on the ticket.
2. The Lambda looks up the ticket by TicketId and checks its status:
   - If the status is already Checked_in → returns a 409 Conflict error, preventing a duplicate check-in.
   - If the status doesn't belong to the valid group (Confirmed, Success, Pending_checkin, Pending) → returns a 400 Bad Request error.
3. The Lambda also checks the Attendance table to see whether the ticket already has a check-in record for that specific eventId — a second, independent verification layer separate from the status field on the ticket.
4. If valid, a new record is written to the Attendance table (including the check-in time and check-in method), and the ticket's status is updated to Checked_in.

---

## Certificate Generation and Sending Flow

1. The client sends a request to retrieve a certificate for a given ticketId.
2. The Lambda checks whether the ticket has already been checked in (based on the Attendance table) — if not, it returns the error "You haven't checked in yet, so the certificate isn't available."
3. If already checked in, the Lambda generates a certificateId in the format CERT- followed by the first 8 characters of the TicketId in uppercase, and creates a PDF file using the QuestPDF library (Community License) with fields such as: full name, event name, location, and issue date.
4. The PDF file is uploaded to S3 at the path certificates/{ticketId}.pdf, after which the Lambda generates a presigned URL valid for 15 minutes to download the file.
5. The Lambda sends an email via Amazon SES to the ticket holder's email address, attaching the presigned URL just generated.

One notable detail: if the ticket is missing some snapshot information (full name, event title, location), the Lambda automatically falls back to default values ("Participant", "AWS Event Management Workshop", "HUTECH") instead of leaving them blank or throwing an error — this keeps the certificate issuance flow from breaking due to missing secondary data.

---

## Step 1: Prepare Test Data

Use the ticketId of a ticket successfully registered in section 5.7:

```powershell
$baseUrl = "https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev"
$ticketId = "<TicketId from section 5.7>"
```

Note: no Authorization header is needed for the requests in this section, since all 3 routes require no JWT.

---

## Step 2: Test a Successful Check-In

```powershell
$body = @{ ticketId = $ticketId; method = "QR" } | ConvertTo-Json
Invoke-RestMethod -Uri "$baseUrl/tickets/checkin" -Method Post -Body $body -ContentType "application/json"
```

Expected response:

```json
{
  "success": true,
  "message": "Check-in successful.",
  "ticketId": "<Ticket's TicketId — taken from the registration response in section 5.7>",
  "eventId": "<EventId of the corresponding event>",
  "eventTitle": "<Event title, snapshotted at registration time>",
  "userId": "<UserId of the account that registered the ticket — matches the sub in the JWT>",
  "userEmail": "<Email of the registered account, used to receive the certificate email in Step 6>",
  "userFullName": "<Full name synced from the user profile at registration time>",
  "checkInAt": "<Actual check-in time, UTC ISO 8601 format, e.g. 2026-07-05T07:20:11Z>",
  "checkInMethod": "QR",
  "status": "CHECKED_IN"
}
```

The success, message, checkInMethod, and status fields are always fixed as shown above for every test account. The remaining fields (ticketId, eventId, eventTitle, userId, userEmail, userFullName, checkInAt) will differ depending on the ticket and account being tested — when capturing real screenshots, these are the exact values that need to match the ticket created in section 5.7.

![Checkin](/images/5-Workshop/5.8-Checkin-certificate-flow/Checkin.jpg)

---

## Step 3: Test Blocking a Duplicate Check-In

Call the exact same request from Step 2 a second time:

```powershell
try {
    Invoke-RestMethod -Uri "$baseUrl/tickets/checkin" -Method Post -Body $body -ContentType "application/json"
} catch {
    $_.Exception.Response.StatusCode.value__
    $_.ErrorDetails.Message
}
```

Expected result: a 409 Conflict error with the message "This ticket has already been checked in. No need to check in again."

---

## Step 4: Test Checking In With an Invalid ticketId

```powershell
$badBody = @{ ticketId = "invalid-ticket-id"; method = "QR" } | ConvertTo-Json
try {
    Invoke-RestMethod -Uri "$baseUrl/tickets/checkin" -Method Post -Body $badBody -ContentType "application/json"
} catch {
    $_.Exception.Response.StatusCode.value__
}
```

Expected result: a 404 Not Found error with the message "Ticket does not exist."

---

## Step 5: Check the Attendance Data in DynamoDB

Open DynamoDB → Tables → the Attendance table → Explore table items, and check that the newly created record has a complete check-in time and a check-in method of QR.

Next, open DynamoDB → Tables → the Tickets table, and check that the corresponding ticket's status has changed to Checked_in.

---

## Step 6: Test Generating and Sending the Certificate

```powershell
Invoke-RestMethod -Uri "$baseUrl/certificates-v2/$ticketId" -Method Get
```

Expected response:

```json
{
  "success": true,
  "message": "Certificate created successfully.",
  "ticketId": "8f2c1a4e-7b3d-4e91-9c5a-1d6f0b8a2e3c",
  "certificateId": "CERT-8F2C1A4E",
  "downloadUrl": "https://<bucket>.s3.ap-southeast-1.amazonaws.com/certificates/8f2c1a4e-....pdf?X-Amz-..."
}
```

![certificates](/images/5-Workshop/5.8-Checkin-certificate-flow/certificates.jpg)

---

## Step 7: Test Blocking Certificate Generation Before Check-In

Use a different ticketId (registered but not yet checked in):

```powershell
Invoke-RestMethod -Uri "$baseUrl/certificates-v2/<ticketId-not-checked-in>" -Method Get
```

Expected result: the error "You haven't checked in yet, so the certificate isn't available."

---

## Step 8: Check the PDF File in S3 and the Sent Email

Open Amazon S3 → Buckets → the project's certificates bucket, and check that the file certificates/ticketId.pdf now exists.

Open CloudWatch → Log groups → the log group for AttendanceCertificateFunction, find the log stream from the test time, and check for log lines recording the outgoing email and a successful SES call status.

![S3](/images/5-Workshop/5.8-Checkin-certificate-flow/S3bucketcertificates.jpg)
![ClouldWatch](/images/5-Workshop/5.8-Checkin-certificate-flow/CloudWatch.jpg)
![Mail](/images/5-Workshop/5.8-Checkin-certificate-flow/Mail.jpg)

---

## Expected Outcome

After completing this section, the practitioner should be able to:

- Successfully check in a ticket using a TicketId and confirm the Attendance data is recorded correctly.
- Confirm the mechanisms that block duplicate check-ins (409 Conflict) and check-ins for non-existent tickets (404 Not Found).
- Successfully generate a PDF certificate for a checked-in ticket and download the file via a presigned URL.
- Confirm the mechanism that blocks certificate generation for a ticket that hasn't been checked in.
- Verify the actual PDF file in S3 and confirm via logs that the email was sent through SES.
- Understand the default-value fallback mechanism used when a ticket is missing secondary snapshot information.
- Have the Attendance data ready for the analytics dashboard in section 5.10.