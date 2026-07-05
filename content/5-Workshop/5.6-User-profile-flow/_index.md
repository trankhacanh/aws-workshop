---
title: "User Profile Management"
date: 2026-06-29
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

---

This section walks through the demo flow for initializing, viewing, and updating a user profile, as well as uploading an avatar. This business logic is handled by UserProfileFunction (UserProfileLambda), which reads/writes to the EventManagementUsers table and the EventManagementUserAvatarsBucket S3 bucket.

Since this module has a short and clear data flow, this section will demonstrate it directly using **PowerShell (Invoke-RestMethod)** calling the API Gateway directly instead of going through the UI — this approach shows the actual request/response data more clearly at each step, and does not depend on the Frontend interface.

---

## User Profile Management Flow

1. After the first login (email or Google), the Frontend calls POST /profile/init — the Lambda checks whether the user already exists in EventManagementUsers; if not, it creates a new profile with default Role and Status; if it already exists, it only updates LastLoginAt and re-syncs Role based on the latest cognito:groups claim.
2. The User views their current profile via GET /profile/me.
3. The User updates their name and avatar via PUT /profile/me.
4. Before updating the avatar, the User calls GET /profile/avatar-upload-url to get a presigned URL, uploads the image directly to S3 (using the same 15-minute presigned URL mechanism as the event banner in section 5.5), then sends the AvatarUrl in PUT /profile/me.

The user profile schema (UserProfileDto) includes:

```text
UserId, Email, FullName, AvatarUrl, Role (USER | ADMIN), Status (ACTIVE | INACTIVE | BLOCKED), CreatedAt, UpdatedAt, LastLoginAt
```

All 4 routes above require a valid JWT via the default Cognito Authorizer — there is no public route in this module, since a profile is always tied to the identity of the logged-in user.

---

## Step 1: Prepare a Token for Testing

Get the ID Token from the login session (you can get it from the DevTools Network tab after logging in as described in section 5.4, or from fetchAuthSession()), then save it into a PowerShell variable:

```powershell
$token = "<paste ID Token here>"
$baseUrl = "https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev"
$headers = @{ Authorization = "Bearer $token" }
```

---

## Step 2: Test Profile Initialization (InitProfile)

```powershell
Invoke-RestMethod -Uri "$baseUrl/profile/init" -Method Post -Headers $headers
```

Expected result (first call):

```json
{
  "isNewUser": true,
  "profile": {
    "userId": "...",
    "email": "...",
    "fullName": "...",
    "role": "USER",
    "status": "ACTIVE",
    "createdAt": "...",
    "lastLoginAt": "..."
  }
}
```

Call it a second time, and confirm that isNewUser returns false and lastLoginAt is updated with a new timestamp.

<!-- > 📌 **Image suggestion:** Capture the PowerShell terminal showing two consecutive calls, clearly showing isNewUser: true then isNewUser: false — evidence that the new/existing user distinction logic works correctly. -->

---

## Step 3: Test Retrieving Profile Information (GetProfile)

```powershell
Invoke-RestMethod -Uri "$baseUrl/profile/me" -Method Get -Headers $headers
```

Check that the response returns all the fields listed in the schema above.

---

## Step 4: Test Getting a Presigned URL and Uploading an Avatar

```powershell
$uploadInfo = Invoke-RestMethod -Uri "$baseUrl/profile/avatar-upload-url?contentType=image/jpeg" -Method Get -Headers $headers
$uploadInfo
```

Expected response:

```json
{
  "uploadUrl": "https://<bucket>.s3.ap-southeast-1.amazonaws.com/...",
  "avatarUrl": "https://<bucket>.s3.ap-southeast-1.amazonaws.com/..."
}
```

Upload the actual image file directly to uploadUrl using PUT (no token needed, since the presigned URL already contains temporary access permission):

```powershell
Invoke-WebRequest -Uri $uploadInfo.uploadUrl -Method Put -InFile ".\avatar-test.jpg" -ContentType "image/jpeg"
```

<!-- > 📌 **Image suggestion:** Capture the terminal showing the `uploadUrl`/`avatarUrl` response, and capture the S3 Console showing the newly uploaded image file appearing in the avatar bucket — no need to capture the Frontend UI. -->

---

## Step 5: Test Updating the Profile (UpdateProfile)

```powershell
$body = @{
    fullName  = "Nguyen Van Test"
    avatarUrl = $uploadInfo.avatarUrl
} | ConvertTo-Json

Invoke-RestMethod -Uri "$baseUrl/profile/me" -Method Put -Headers $headers -Body $body -ContentType "application/json"
```

Call GET /profile/me again to confirm that fullName and avatarUrl have been updated correctly.

---

## Step 6: Check the Data in DynamoDB

Open DynamoDB > Tables > EventManagementUsers > Explore table items, and check that the test user's item has all the updated fields, and that Role correctly reflects ADMIN if the user belongs to the Admins group.

<!-- > 📌 **Image suggestion:** Capture the item in the DynamoDB Console — compare it directly against the JSON response from Step 5 to prove the data was written correctly. -->

---

## Step 7: Test the Case With No Token (Verify Route Protection)

```powershell
try {
    Invoke-RestMethod -Uri "$baseUrl/profile/me" -Method Get
} catch {
    $_.Exception.Response.StatusCode.value__
}
```

Expected result: 401 Unauthorized, confirming that all routes in the User Profile module are correctly protected by the Cognito Authorizer as designed — unlike the public routes in the Event Management module (section 5.5).

---

## Expected Outcome

After completing this section, the practitioner should be able to:

- Test the profile initialization flow and correctly distinguish new/existing users (isNewUser).
- Test the flow for viewing and updating a user profile via direct API requests.
- Test the avatar upload mechanism via presigned URL, confirming the file appears correctly in S3.
- Confirm that all routes in this module require a valid JWT (returning 401 when the token is missing).
- Cross-check the data returned from the API against the actual data stored in DynamoDB.
- Have profile data (especially FullName, Email) ready to sync into the e-ticket in the ticket registration flow (section 5.7).