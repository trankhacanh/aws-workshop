---
title: "Configure Authentication with Amazon Cognito"
date: 2026-06-29
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

---

This section explains how the Event Management Platform uses Amazon Cognito for user authentication and User/Admin authorization.

The system supports two sign-in methods: direct sign-up/sign-in with email and password, and Google sign-in through Cognito Hosted UI (Federated Identity Provider). The Cognito User Pool is referenced in the backend through the CognitoUserPoolIdParam parameter in template.yaml, which is used to configure the Cognito Authorizer for protected API routes.

---

## Email/password authentication flow

1. The user registers with email, password, and full name (phone number is optional) through Amazon Cognito (using aws-amplify and the signUp function).
2. Cognito sends a confirmation code to the registered email address.
3. The user confirms the code with confirmSignUp.
4. The user signs in with signIn; Cognito returns Access Token and ID Token.
5. The frontend calls userProfileService.initProfile() so the backend creates/synchronizes the user profile in DynamoDB.
6. Every subsequent API request uses axiosInstance.ts to retrieve the current ID Token (fetchAuthSession) and attach it to the Authorization: Bearer <idToken> header.
7. API Gateway uses the Cognito Authorizer (COGNITO_USER_POOLS) to validate the ID Token before forwarding the request to the corresponding Lambda function.
8. Lambda reads the claims already decoded by API Gateway from request.RequestContext.Authorizer.Claims; it does not verify JWTs itself in the Lambda code.

---

## Google sign-in flow (Federated Sign-In)

In addition to email/password login, the system supports Google sign-in through Cognito Hosted UI using OAuth 2.0 Authorization Code Grant:

1. The user clicks “Continue with Google” on the sign-in page; the frontend calls signInWithRedirect({ provider: 'Google' }) from Amplify.
2. The browser is redirected to Cognito Hosted UI and then to the Google login page.
3. After successful authentication with Google, Cognito receives the authorization code, exchanges it for tokens, and redirects the user back to the frontend at the callback route (/auth/callback), handled by the AuthCallback component.
4. Amplify Hub emits a signInWithRedirect event; AuthContext.tsx listens to this event and, when the sign-in succeeds, calls initProfile() to synchronize the profile for a newly signed-in Google user.
5. From that point onwards, a Google user is handled the same way as an email-based user: they have an ID Token and call APIs through axiosInstance.ts.

The OAuth configuration is declared centrally in main.tsx when Amplify is initialized:

```ts
Amplify.configure({
  Auth: {
    Cognito: {
      userPoolId: import.meta.env.VITE_COGNITO_USER_POOL_ID,
      userPoolClientId: import.meta.env.VITE_COGNITO_CLIENT_ID,
      loginWith: {
        oauth: {
          domain: import.meta.env.VITE_COGNITO_OAUTH_DOMAIN,
          scopes: ['openid', 'email', 'profile', 'aws.cognito.signin.user.admin'],
          redirectSignIn: [import.meta.env.VITE_COGNITO_REDIRECT_SIGN_IN],
          redirectSignOut: [import.meta.env.VITE_COGNITO_REDIRECT_SIGN_OUT],
          responseType: 'code',
        }
      }
    },
  },
});
```

Add the following environment variables to the frontend .env (in addition to those already described in section 5.2):

```env
VITE_COGNITO_CLIENT_ID=<Cognito App Client ID>
VITE_COGNITO_OAUTH_DOMAIN=<Cognito Hosted UI domain>
VITE_COGNITO_REDIRECT_SIGN_IN=<callback URL after login, e.g. http://localhost:5173/auth/callback>
VITE_COGNITO_REDIRECT_SIGN_OUT=<URL after logout>
```

| ![Image 1](/images/5-Workshop/5.4-Configure-authentication/Login.jpg) | ![Image 2](/images/5-Workshop/5.4-Configure-authentication/Logingg.jpg) |
|---|---|

---

## A noteworthy customization: dynamic “Remember Me”

Instead of always saving tokens to localStorage (persistent) or always using sessionStorage (cleared when the tab closes), the project implements a custom KeyValueStorageInterface for Amplify (dynamicAuthStorage in main.tsx) that decides where to store tokens based on the user’s “Remember Me” selection at sign-in time:

- If the user chooses “Remember Me,” the token is stored in localStorage (persistent login across sessions).
- If not, the token is stored in sessionStorage (cleared when the browser is closed).

---

## Step 1: Open the Amazon Cognito User Pool

Open AWS Management Console → Amazon Cognito → User pools and select the User Pool used by the system.

---

## Step 2: Review the User Pool configuration

| Item | Current configuration |
|---|---|
| Sign-in method | Sign in with email |
| Federated Identity Provider | Google (via Cognito Hosted UI) |
| App Client | Public client (no client secret) |
| Hosted UI Domain | Domain in the form <prefix>.auth.ap-southeast-1.amazoncognito.com |
| Token | Access Token + ID Token; the frontend uses the ID Token for API calls |

---

## Step 3: Review the Google Identity Provider configuration

Open Amazon Cognito → User pools → Sign-in experience → Federated identity provider sign-in and verify that the Google provider is configured with the Client ID and Client Secret obtained from Google Cloud Console (OAuth Consent Screen + Credentials).

![Google sign-in configuration](/images/5-Workshop/5.4-Configure-authentication/sign-inGoogle.jpg)

Then check App integration → App client → Hosted UI to ensure:
- Google is enabled for the App Client being used.
- Allowed callback URLs and Allowed sign-out URLs match the frontend values of VITE_COGNITO_REDIRECT_SIGN_IN and VITE_COGNITO_REDIRECT_SIGN_OUT.

![Google sign-in configuration details](/images/5-Workshop/5.4-Configure-authentication/sign-inGoogle1.jpg)

---

## Step 4: Review Cognito user groups

The Event Management Platform uses the Cognito group Admins to distinguish Admin roles.

Open Amazon Cognito → User pools → Groups and verify the Admins group exists. Users belonging to this group will have a cognito:groups claim containing "Admins" when they sign in, for both email/password and Google sign-in.

![User Pool Admin group](/images/5-Workshop/5.4-Configure-authentication/UserPoolAdmin.jpg)

---

## Step 5: How the backend reads claims and determines Admin permissions

After API Gateway validates the JWT successfully, the claims are passed into request.RequestContext.Authorizer.Claims:

```text
sub → UserId
email → Email
name → FullName
cognito:groups → string such as "[Admins]" or "[Admins, Organizers]"
```

```csharp
public bool IsAdmin => Groups.Contains("Admins");
```

The role (IsAdmin) is synchronized into DynamoDB in the EventManagementUsers table each time POST /profile/init is called, for both standard email sign-up users and Google users.

---

## Step 6: Test the authentication flow

1. Register a new account with email, confirm the code, and sign in.
2. Test the sign-in flow with the “Continue with Google” button and confirm that the redirect through Google works and returns successfully to the frontend.
3. Open DevTools (Network tab) and verify that API requests carry the Authorization: Bearer <idToken> header for both authentication methods.
4. In the Cognito Console, add one of the users to the Admins group, sign out and sign in again, and call GET /profile/me to confirm the role is returned as Admin.

![Profile view](/images/5-Workshop/5.4-Configure-authentication/Profileme.jpg)

---

## Expected outcomes

After completing this section, participants should be able to:

- Register and sign in successfully with email/password through Amazon Cognito.
- Sign in successfully with a Google account through Cognito Hosted UI (Federated Identity Provider).
- Confirm that the ID Token is attached correctly to the Authorization header of API requests for both login methods.
- Add a user to the Cognito group Admins and verify that the cognito:groups claim reflects the role correctly when calling GET /profile/me.
- Understand how the backend extracts claims from API Gateway (sub, email, name, cognito:groups) without verifying JWTs directly in Lambda.
- Understand the dynamic “Remember Me” mechanism, which stores tokens in localStorage or sessionStorage based on the user’s choice.
- Be ready for the following business sections (Event Management, Ticket, Check-in, and so on), which all depend on a valid JWT and correctly established role claims.