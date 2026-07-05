---
title: "Event & Category Management"
date: 2026-06-29
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

---

This section walks through the demo flow for the Admin managing events and categories — a mandatory step for creating the base data that Users need before they can register for tickets in the sections that follow.

This entire business logic is handled by EventFunction (EventLambda), which reads/writes to the two tables EventManagementEvents and EventManagementCategories, along with the EventManagementEventBannersBucket S3 bucket used to store banners.

---

## Event Management Flow

1. Admin logs in (already holding a valid JWT, belonging to the Admins group — see section 5.4).
2. Admin calls GET /admin/events/banner-upload-url to get a presigned URL, then uploads the banner file directly to S3 from the browser (not through Lambda).
3. Admin calls POST /admin/events to create a new event, including the BannerUrl just uploaded, title, description, location, start/end time, category, speaker, prerequisites, required tools, and maximum number of slots (MaxSlots).
4. Admin can view the list of admin events (GET /admin/events), view details (GET /admin/events/{eventId}), or update an event (PUT /admin/events/{eventId}).
5. Admin toggles event visibility via PATCH /admin/events/{eventId}/visibility — flipping the IsVisible flag without deleting the event.
6. Users (no login required) view the list of public events via GET /events and details via GET /events/{eventId} — both routes are declared with Authorizer: NONE.

Note: the system **has no route for deleting an event** (DeleteEvent) — an event can only be hidden via the IsVisible flag, preserving the entire ticket/attendance history associated with it instead of hard-deleting the data.

---

## Banner Upload Mechanism via Presigned URL

Instead of letting the Frontend upload files directly through Lambda (which costs processing time and is limited by the API Gateway payload size), the system uses a **presigned URL** mechanism:

1. The Frontend calls GET /admin/events/banner-upload-url?contentType=<mime-type>.
2. EventService.GenerateBannerUploadUrlAsync generates a unique fileKey, determines the file extension (jpg, webp, or png by default) based on the contentType, then creates a presigned URL valid for **15 minutes** to PUT directly to S3.
3. The Frontend uses this URL to upload the file straight to S3, bypassing the backend.
4. Once the upload is complete, the Frontend sends the BannerUrl (derived from fileKey) as part of the POST /admin/events or PUT /admin/events/{eventId} request.

When returning the banner for the client to display, the system generates another presigned URL valid for **24 hours** to read the image (ResolveBannerAccessUrl), rather than making the bucket public — consistent with how CloudFront/OAC is handled on the Frontend side.

---

## Automatic Event Status Calculation Mechanism

The event status (Status) is not only set manually by the Admin but is also automatically recalculated via EventStatusHelper.ApplyEffectiveStatus, based on comparing EndTime with the current time — an event that has passed its EndTime will automatically be treated as ended (IsEnded) when returned to the client, even if the Status value stored in DynamoDB has not yet been updated.

Because of this mechanism, SetEventVisibilityAsync will **reject** any request to enable visibility (IsVisible = true) for an event that has already ended, preventing the Admin from accidentally re-displaying a past event.

---

## Category Management Flow

Alongside events, the Admin manages categories through separate routes:

- GET /categories — public, no login required (Authorizer: NONE).
- GET /admin/categories, POST /admin/categories — view the list and create new categories.
- PUT /admin/categories/{categoryId} — update a category.
- DELETE /admin/categories/{categoryId} — delete a category (unlike events, categories **do** have a real delete route).

---

## Step 1: Test the Event Creation Flow via Admin UI

1. Log in with an account belonging to the Admins group.
2. Go to the event administration page and create a new category first (if one doesn't already exist).
3. Create a new event: enter title, description, location, time, select category, upload banner, enter the maximum number of slots.
4. Save the event, then check that it appears in the admin list.

![Event](/images/5-Workshop/5.5-Event-management-flow/Event.jpg)


---

## Step 2: Check the Data in DynamoDB

`Open DynamoDB > Tables > EventManagementEvents > Explore table items`, and check that the newly created item has all the fields: EventId, Title, Status, BannerUrl, MaxSlots, RegisteredCount (initialized to 0), IsVisible.

![Dynamo](/images/5-Workshop/5.5-Event-management-flow/Dynamo.jpg)
---

## Step 3: Test Toggling Event Visibility

1. From the admin page, click "Hide event" for the event just created.
2. Confirm that the event no longer appears on the public list page (GET /events) but is still present in the admin list (GET /admin/events).
3. Re-enable visibility and confirm the event reappears on the public page.

| ![Image 1](/images/5-Workshop/5.5-Event-management-flow/EventUnHiden.jpg) | ![Image 2](/images/5-Workshop/5.5-Event-management-flow/EventHiden.jpg) |
|---|---|

---

## Step 4: Test the Public Route Without Login

1. Open the browser in incognito mode (not logged in).
2. Access the event list page and confirm you can still view the list of events where IsVisible = true.
3. Open the DevTools Network tab and confirm that the GET /events request has **no** Authorization header but still returns 200 OK.

---

## Expected Outcome

After completing this section, the practitioner should be able to:

- Create, list, and update events with full information via the Admin API.
- Understand and test the banner upload mechanism via presigned URL, without going through Lambda.
- Understand the automatic event status calculation mechanism (ApplyEffectiveStatus) based on EndTime.
- Test the event visibility toggle flow (IsVisible) and confirm the correct behavior of rejecting re-visibility for ended events.
- Manage event categories (create, update, delete) via separate admin routes.
- Confirm that the public read routes (GET /events, GET /categories) work correctly without a JWT.
- Have event data ready for use in the ticket registration flow in section 5.7.