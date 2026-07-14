---
name: google-one-tap-auth
description: Implement or review Google One Tap authentication on web applications. Use when asked to add Google One Tap, integrate Google Identity Services (GIS), connect One Tap with Supabase, Firebase, Auth.js/NextAuth, Clerk, or a custom backend, troubleshoot One Tap issues, or migrate from legacy Google Sign-In.
metadata:
  author: Rohit Lodhi
  version: "1.0.0"
  argument-hint: <project-or-files>
---

# Google One Tap Authentication

Implement Google's modern **One Tap Sign-In** using **Google Identity Services (GIS)**.

This skill covers both:

- Authentication providers (Supabase, Firebase Auth, Auth.js/NextAuth, Clerk, etc.)
- Custom authentication backends

Always use the latest Google Identity Services SDK.

---

# Overview

Google One Tap allows users to sign in with a single click without leaving the current page.

Unlike traditional OAuth redirects, One Tap displays a floating account picker.

Typical flow:

```
User
   ↓
Google One Tap
   ↓
Google Identity Services
   ↓
ID Token (JWT)
   ↓
Authentication Provider / Backend
   ↓
Application Session
```

One Tap only authenticates the user.

Your authentication provider or backend is responsible for creating the application's session.

---

# Architecture

There are two supported architectures.

---

# Option 1 — Authentication Provider (Recommended)

Use this whenever the project already uses an authentication provider.

Examples:

- Supabase
- Firebase Auth
- Auth.js (NextAuth)
- Clerk
- Auth0
- Cognito
- Better Auth

Flow:

```
User
   ↓
Google One Tap
   ↓
Google returns ID Token
   ↓
Provider verifies token
   ↓
Provider creates session
   ↓
User authenticated
```

The application should **never manually verify Google tokens** in this architecture.

The provider is responsible for:

- JWT verification
- Google public key validation
- Token expiration
- User creation
- Session creation
- Refresh tokens (if supported)

---

## Example — Supabase

After receiving the Google credential:

```ts
await supabase.auth.signInWithIdToken({
  provider: "google",
  token: credential,
})
```

Do **not** call Google's verification endpoint manually.

Supabase performs all validation.

---

## Example — Firebase

Authenticate using Google's credential through Firebase Authentication.

Do not manually decode or verify the JWT.

---

## Example — Auth.js / NextAuth

Configure Google as an authentication provider.

Allow Auth.js to manage:

- OAuth flow
- Sessions
- Cookies
- Token refresh

---

## Example — Clerk

Use Clerk's Google authentication strategy.

Avoid implementing custom verification.

---

# Option 2 — Custom Backend

Use this only when no authentication provider exists.

Flow:

```
User
   ↓
Google One Tap
   ↓
Google returns JWT
   ↓
Frontend
   ↓
POST JWT
   ↓
Backend
   ↓
Verify JWT
   ↓
Find/Create User
   ↓
Create Session
   ↓
Return Cookie/JWT
```

The backend must verify the ID token.

Never trust the frontend.

---

## Backend Responsibilities

The backend should:

- Verify Google signature
- Validate audience (Client ID)
- Validate issuer
- Validate expiration
- Read user profile
- Find existing user
- Create user if needed
- Issue application session

Never use the Google ID token as your application's authentication token.

Always issue your own session.

---

# Frontend Responsibilities

The frontend should:

- Load Google Identity Services
- Initialize One Tap once
- Handle credential callback
- Pass credential to provider/backend
- Avoid duplicate initialization
- Respect dismissal
- Avoid showing prompt when authenticated

The frontend should never:

- Verify JWT
- Decode JWT for authentication
- Store Google ID token permanently
- Treat Google ID token as an application session

---

# Google Identity Services

Always use:

```
https://accounts.google.com/gsi/client
```

Do not use the deprecated Google Sign-In library.

---

# Initialization Guidelines

Initialize only once.

Avoid duplicate initialization caused by:

- React Strict Mode
- Multiple layouts
- Multiple providers
- Re-render loops

One Google client should exist per application.

---

# Prompt Display Rules

Display One Tap only when:

- User is logged out
- Google determines user is eligible
- Browser allows One Tap
- User has not recently dismissed the prompt

Do not force the prompt.

Google manages frequency.

---

# Session Management

Authentication providers:

Use the provider's existing session management.

Do not create parallel authentication systems.

Custom backend:

Issue:

- Secure cookies
- HTTP-only cookies
- JWT
- Session IDs

according to the application's existing authentication architecture.

---

# Security

Never:

- Trust frontend identity
- Skip backend verification (custom backend)
- Store Google ID token long-term
- Expose Client Secret
- Use ID token as API authorization

Always:

- Use HTTPS
- Verify audience
- Verify issuer
- Validate expiration
- Use secure cookies
- Use CSRF protection where applicable

---

# Common Issues

## One Tap never appears

Possible causes:

- User already signed in
- Browser suppression
- Third-party cookie restrictions
- User dismissed prompt
- HTTP instead of HTTPS
- Invalid Client ID
- Duplicate initialization

---

## origin_mismatch

Check:

Authorized JavaScript Origins

Example:

```
http://localhost:3000
https://example.com
```

---

## popup_closed

The user dismissed One Tap.

Treat as a normal cancellation.

---

## invalid_client

Usually indicates:

- Wrong Client ID
- OAuth Client not configured
- Incorrect Google Cloud project

---

## Multiple prompts

Usually caused by:

- React Strict Mode
- Multiple initializations
- Rendering the component multiple times

Initialize GIS only once.

---

# Best Practices

✔ Use Google Identity Services

✔ Keep One Tap separate from session management

✔ Let authentication providers manage verification

✔ Verify JWT only on custom backends

✔ Use HTTPS

✔ Handle dismissal gracefully

✔ Initialize once

✔ Show only for logged-out users

✔ Keep implementation modular

✔ Follow provider-specific authentication flows

---

# Review Checklist

When reviewing an implementation, verify:

- Latest Google Identity Services SDK is used
- Deprecated Google Sign-In APIs are absent
- One Tap initializes once
- No duplicate prompts
- Prompt only shown when logged out
- HTTPS requirements satisfied
- Correct Google Client ID configured
- Authentication provider is used correctly
- Custom backend verifies JWT properly
- No frontend JWT verification
- No Client Secret exposed
- Existing session management reused
- Proper error handling implemented
- React Strict Mode handled correctly
- Mobile and desktop compatibility verified
- Security best practices followed

---

# Usage

When a user provides project files:

1. Detect the authentication architecture.
2. Determine whether an authentication provider or custom backend is used.
3. Apply the appropriate implementation strategy.
4. Review authentication flow.
5. Report architectural issues.
6. Suggest improvements following current Google Identity Services best practices.
