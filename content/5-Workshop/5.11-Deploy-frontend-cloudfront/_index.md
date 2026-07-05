---
title: "Deploy Frontend via CloudFront"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

---

This section explains how to build the React frontend, upload it to a private S3 bucket, and distribute it through Amazon CloudFront with Origin Access Control (OAC). The goal is to make the frontend available through CloudFront while preventing direct public access to the S3 bucket.

> The current deployment uses the default CloudFront domain provided by AWS (for example, <distribution-id>.cloudfront.net), and it does not yet use a custom domain or ACM certificate. The custom-domain step is treated as a future extension and is not part of the current workshop scope.

Because this is an infrastructure deployment step rather than a business API call, the demo uses AWS CLI together with the AWS Console.

---

## Frontend deployment flow

1. Build the React project into a static output folder (dist/).
2. Create a dedicated S3 bucket for the frontend, enable Block Public Access, and do not enable public static website hosting because access will go through CloudFront rather than directly to S3.
3. Create a CloudFront Distribution with the S3 bucket as the origin and configure Origin Access Control so that CloudFront has permission to read objects from S3 but users cannot access S3 directly.
4. Update the bucket policy so that only the CloudFront service principal is allowed to read objects from the bucket.
5. Configure the Default Root Object as index.html and set Custom Error Responses so that React Router client-side routes return index.html with HTTP 200 for 403/404 errors.
6. Upload the build to S3 and create an Invalidation on CloudFront after each deployment to clear the CDN cache.
7. Access the application through the CloudFront domain to verify the full flow, including API Gateway and Cognito integration configured in the previous sections.

---

## Step 1: Build the frontend

In the FE/ directory, configure the environment variables to point to the deployed API Gateway and Cognito (for example, in .env.production):

```env
VITE_API_BASE_URL=https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev
VITE_COGNITO_USER_POOL_ID=<user-pool-id>
VITE_COGNITO_CLIENT_ID=<app-client-id>
VITE_COGNITO_REGION=ap-southeast-1
```

Build for production:

```powershell
npm install
npm run build
```

Result: the dist/ directory contains index.html and the bundled assets folder (JS/CSS), which will be uploaded to S3.

---

## Step 2: Create an S3 bucket for the frontend (private)

```powershell
aws s3api create-bucket `
  --bucket event-management-frontend-<suffix> `
  --region ap-southeast-1 `
  --create-bucket-configuration LocationConstraint=ap-southeast-1

aws s3api put-public-access-block `
  --bucket event-management-frontend-<suffix> `
  --public-access-block-configuration BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
```

Note: this bucket does not enable Static Website Hosting. CloudFront accesses the bucket through its REST endpoint (bucket.s3.amazonaws.com), not through the website endpoint.

---

## Step 3: Create a CloudFront distribution with Origin Access Control

In the CloudFront Console:

1. Create Distribution → Origin domain: select the S3 bucket created above (use the REST endpoint, not the website endpoint).
2. In Origin access, choose Origin access control settings (recommended) → Create control setting (keep the default “Sign requests”).
3. Default root object: index.html.
4. Viewer protocol policy: Redirect HTTP to HTTPS.
5. Alternate domain name (CNAME): leave empty — the current system uses the default CloudFront domain rather than a custom domain (see Step 7 for the future extension).
6. After creation, CloudFront will show a warning asking you to update the bucket policy; click Copy policy.

Update the Bucket Policy in the S3 Console → Permissions → Bucket Policy with the policy CloudFront provides, similar to:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipal",
      "Effect": "Allow",
      "Principal": { "Service": "cloudfront.amazonaws.com" },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::event-management-frontend-<suffix>/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::<account-id>:distribution/<distribution-id>"
        }
      }
    }
  ]
}
```

---

## Step 4: Configure Custom Error Responses for React Router

Because React Router handles client-side routing, when the user refreshes a nested route such as /events/123, S3 would return 403/404 because that object does not exist. CloudFront should return index.html for those errors:

In Distribution → Error pages → Create custom error response:

| HTTP Error Code | Customize Error Response | Response Page Path | HTTP Response Code |
|---|---|---|---|
| 403 | Yes | /index.html | 200 |
| 404 | Yes | /index.html | 200 |

---

## Step 5: Upload the build and create an invalidation

```powershell
aws s3 sync ./dist s3://event-management-frontend-<suffix> --delete

aws cloudfront create-invalidation `
  --distribution-id <distribution-id> `
  --paths "/*"
```

- --delete ensures old objects are removed from the bucket.
- create-invalidation is required after each deployment; otherwise CloudFront may continue serving cached content from the previous deployment.

---

## Step 6: Test access through CloudFront

Open the CloudFront default domain in the browser: https://<distribution-domain>

1. Verify that the home page loads and calls the public GET /events route correctly.
2. Try signing in through Cognito and confirm that navigation and protected routes such as Profile and Registration work correctly.
3. Open a nested route like /my-tickets and refresh the page; confirm that it does not show a blank screen due to 403/404 errors thanks to the custom error response.
4. Try accessing the direct S3 URL (https://event-management-frontend-<suffix>.s3.amazonaws.com/index.html) and confirm it is denied (AccessDenied), proving the bucket is private and only CloudFront can read it.

---

## Step 7 (Future extension — not yet implemented): Attach a custom domain

The current system does not yet use a custom domain; it uses the default CloudFront domain. If a custom domain such as app.example.com is required later, the following steps are needed:

1. Create an ACM certificate in the us-east-1 region (required for CloudFront).
2. In the CloudFront Distribution, update Alternate domain names (CNAMEs) and the custom SSL certificate in ViewerCertificate.
3. Create a DNS record pointing the custom domain to the CloudFront distribution (Alias if using Route 53, or CNAME otherwise).
4. Test the full flow again using the custom domain instead of the default one.

This is a recommended future enhancement, not part of the current workshop implementation.

---

## Expected outcomes

After completing this section, participants should be able to:

- Build and deploy a production version of the frontend to S3.
- Confirm that the frontend bucket is completely private and cannot be accessed directly from the outside.
- Confirm that CloudFront is the only access point and uses OAC to retrieve content from S3.
- Confirm that 403/404 errors return index.html correctly for React Router client-side routing.
- Understand the deployment cycle: build → s3 sync → create-invalidation.
- Understand that the current system uses the default CloudFront domain and know the steps required to attach a custom domain later (ACM in us-east-1 + Aliases + ViewerCertificate).

