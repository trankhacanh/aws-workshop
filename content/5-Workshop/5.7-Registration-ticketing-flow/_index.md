---
title: "Ticket Registration Flow"
date: 2026-06-29
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

---

This section demonstrates the ticket registration flow, the overbooking-prevention mechanism using DynamoDB conditional expressions, and publishing the TicketRegistered event to EventBridge to trigger asynchronous notifications (covered in detail in section 5.9).

This business logic is handled by TicketFunction, which reads/writes to the Tickets table, reads/increments a counter in the Events table, and reads profile information from the Users table to sync into the ticket.

---

## Ticket Registration Flow

1. The User sends a ticket registration request (with an eventId) along with a valid JWT.
2. TicketService checks whether the User has already registered for this event (querying by UserId via a GSI, matching against EventId) — if already registered, it returns a business logic error.
3. The system performs a conditional update to increment the event's RegisteredCount:

```csharp
UpdateExpression = "SET #regCount = #regCount + :inc",
ConditionExpression = "#regCount < #maxSlots"
```

4. If the condition is not met (i.e., MaxSlots has already been reached), DynamoDB automatically throws a ConditionalCheckFailedException — this is an atomic mechanism that prevents race conditions when multiple users register at the same time, without needing manual locking at the application layer.
5. If the condition passes, the system creates a new ticket with status Confirmed, snapshotting the event information (event name, start time, location, category) and User information (email, full name) at the moment of registration — so the ticket still displays the correct information even if the event or user profile changes later.
6. After the ticket is successfully created, the Lambda publishes an event to EventBridge (source eventmanagement.ticket, detail-type TicketRegistered) to trigger NotificationFunction, which sends a confirmation email asynchronously. This operation is wrapped in its own separate try/catch — if publishing the event fails, only a warning is logged; it does not roll back or fail the ticket registration request.
7. The User views their own tickets via the personal tickets route.

---

## UI Behavior Based on Registration Status

On the event detail page, the action button changes depending on two fields returned from the API — isFull and the registration status (isAlreadyRegistered) — with no other intermediate "pending" state:

| Status | UI Display |
|---|---|
| Not registered, slots available (isFull: false) | "Register for event" button |
| Successfully registered (isAlreadyRegistered: true) | Button changes to "View my ticket", re-registration disabled |
| Event is full (isFull: true) | Shows "Registration slots full" message, registration button hidden |
| Event has ended / been canceled | Shows the corresponding status message, registration button hidden |

After a successful call to the ticket registration request, the Frontend reloads the event data to immediately update the latest registered count/full status, ensuring the button state changes instantly without needing to reload the page.

---

## Step 1: Prepare the Token and Test Environment Variables

```powershell
$token = "<paste ID Token>"
$baseUrl = "https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev"
$headers = @{ Authorization = "Bearer $token" }
$eventId = "<EventId from section 5.5>"
```

---

## Step 2: Test Successful Ticket Registration

```powershell
Invoke-RestMethod -Uri "$baseUrl/events/$eventId/register" -Method Post -Headers $headers
```

Actual response returned:

```json
{
  "ticketId": "8f2c1a4e-7b3d-4e91-9c5a-1d6f0b8a2e3c",
  "eventId": "4573b866-0a75-4453-9346-655c416fd97e",
  "userId": "a1b2c3d4-5678-90ab-cdef-1234567890ab",
  "eventTitle": "AWS Serverless Event Management Workshop 2026",
  "eventStartTime": "2026-07-05T06:03:00.000Z",
  "eventLocation": "Cloud Lab Room – 4th Floor, Building A, University of Information Technology",
  "eventCategory": "AI PROMT",
  "userEmail": "user.test@example.com",
  "userFullName": "Nguyen Van Test",
  "status": "CONFIRMED",
  "createdAt": "2026-07-05T06:10:22.000Z"
}
```

<!-- > 📌 **Image suggestion:** Capture two side-by-side UI screenshots — before registering (the "Register for event" button) and after registering (the button changed to "View my ticket") — directly illustrating the UI behavior described above. -->

---

## Step 3: Test Blocking Duplicate Registration

Call the exact same request from Step 2 a second time:

```powershell
try {
    Invoke-RestMethod -Uri "$baseUrl/events/$eventId/register" -Method Post -Headers $headers
} catch {
    $_.Exception.Response.StatusCode.value__
    $_.ErrorDetails.Message
}
```

Expected result: a business logic error such as "You have already registered a ticket for this event!". On the UI, this scenario can't actually happen because the button already switched to "View my ticket" right after the first registration — this second test request can only be reproduced by calling the API directly.

---

## Step 4: Test the Overbooking-Prevention Mechanism

If the test event has a small maximum slot count (e.g., = 1), use several different accounts/tokens to register consecutively until the slot limit is exceeded:

```powershell
$headers2 = @{ Authorization = "Bearer <another user's token>" }
try {
    Invoke-RestMethod -Uri "$baseUrl/events/$eventId/register" -Method Post -Headers $headers2
} catch {
    $_.ErrorDetails.Message
}
```

Expected result once slots are full: an error such as "This event has reached its seating capacity (Sold out)!" — originating from DynamoDB's ConditionalCheckFailedException error. On the UI, at this point the full-status flag becomes true, and the registration button is replaced with a "Registration slots full" message for the remaining users who haven't registered.

<!-- > 📌 **Image suggestion:** Capture the event UI at this point showing "Registration slots full", along with a CloudWatch Logs screenshot showing the corresponding ConditionalCheckFailedException log line. -->

---

## Step 5: Check the Data in DynamoDB

Open DynamoDB → Tables → the Tickets table → Explore table items, and check that the newly created ticket has the full data snapshot (event name, user's full name, etc.) and a status of Confirmed.

Next, open DynamoDB → Tables → the Events table, and check that the event's registered count has increased correctly to match the number of successful ticket registrations, and that it reaches the maximum slot count at the point when the UI shows "Registration slots full".

![FullFe](/images/5-Workshop/5.7-Registration-ticketing-flow/FullSlotFe.jpg)

![FullAWS](/images/5-Workshop/5.7-Registration-ticketing-flow/FullSlot.jpg)

---

## Step 6: Check That the Event Was Published to EventBridge

Open Amazon EventBridge → Rules, and find the rule that listens for the ticket registration event (on the default event bus). Go to the rule's Monitoring tab and check that the Invocations chart has increased by the exact number of ticket registrations just tested.

![Event](/images/5-Workshop/5.7-Registration-ticketing-flow/Rule.jpg)

---

## Step 7: Test Viewing Personal Tickets

```powershell
Invoke-RestMethod -Uri "$baseUrl/my-tickets" -Method Get -Headers $headers
```

Confirm the response returns the list of tickets, including the ticket registered in Step 2 — this is the same data displayed when clicking the "View my ticket" button in the UI.

![Tikcet](/images/5-Workshop/5.7-Registration-ticketing-flow/MyTicket.jpg)

---

## Expected Outcome

After completing this section, the practitioner should be able to:

- Successfully register a ticket and receive the correct snapshot data in the response.
- Confirm that the UI button transitions correctly based on the full-status and already-registered status (Register → View ticket, or → Registration slots full).
- Confirm the mechanism that blocks duplicate registration for the same event.
- Confirm that the overbooking-prevention mechanism works correctly via the ConditionalCheckFailedException error.
- Cross-check data between the API response and the actual items in the two DynamoDB tables (Tickets, Events).
- Confirm that the ticket registration event was successfully published to EventBridge via the Rule's Monitoring chart.
- Review the list of personal tickets via the personal tickets route.
- Have ticket data ready for the check-in and certification flow in section 5.8.