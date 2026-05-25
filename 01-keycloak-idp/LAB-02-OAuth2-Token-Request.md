# Lab 02 — OAuth 2.0 Token Request with Keycloak

**Author:** Trisha Trafalgar (Trishsteezy)  
**Date:** May 2026  
**Difficulty:** Beginner - Intermediate  
**Time to Complete:** 1–2 hours  
**YouTube Walkthrough:** *(paste YouTube link here)*

---

## Objective

Use the OAuth 2.0 Resource Owner Password Credentials (ROPC) flow to request a real JWT access token from Keycloak. Decode and inspect the token to understand what data is embedded inside every enterprise authentication event.

---

## What is OAuth 2.0?

OAuth 2.0 is the industry-standard protocol for authorization. It allows applications to obtain tokens that prove a user's identity and permissions without ever exposing their password to the application itself.

### The 4 Main OAuth 2.0 Flows

| Flow | Use Case | When to Use |
|------|---------|------------|
| **Authorization Code** | Web apps with a backend server | Most secure — enterprise standard |
| **Client Credentials** | Machine to machine — no human involved | APIs, microservices, automation |
| **Resource Owner Password (ROPC)** | Direct username/password exchange | Testing only — never production |
| **Device Code** | Smart TVs, CLIs, limited input devices | IoT, command line tools |

> **In this lab we use ROPC (Resource Owner Password Credentials) because it is the simplest flow for testing and learning. In production, Authorization Code Flow with PKCE is always preferred.**

---

## Real World Context

Every time an employee logs into Salesforce, ServiceNow, or any enterprise app through SSO — this token exchange is happening behind the scenes in milliseconds. The application never sees the user's password. It only sees the token Keycloak issues after verifying the credentials.

As an IAM Engineer, understanding tokens is critical because:
- Token misconfiguration = security vulnerabilities
- Token contents determine what a user can access
- Token lifetimes affect both security and user experience
- Stolen tokens are how attackers maintain persistence after a breach

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Keycloak 26.6.2 | Identity Provider issuing tokens |
| Git Bash | Command line for curl requests |
| curl | HTTP client for making API requests |
| jwt.io | JWT token decoder and inspector |
| Postman (attempted) | API testing tool |

---

## Prerequisites

- Lab 01 completed (Keycloak running, iam-lab realm, testuser created)
- Keycloak running at `http://localhost:8080`
- Git Bash installed

---

## Step 1 — Understanding the Token Endpoint

Every Keycloak realm exposes a standard OpenID Connect token endpoint:

```
http://localhost:8080/realms/{realm-name}/protocol/openid-connect/token
```

For our lab:
```
http://localhost:8080/realms/iam-lab/protocol/openid-connect/token
```

This is the URL that applications call to exchange credentials for tokens. It is a POST request that accepts form-encoded data.

---

## Step 2 — Setting Up Postman (Attempted)

### What Happened
Attempted to use Postman desktop app to send the token request visually.

### Issue Encountered — Cloud Agent Error
```
Cloud agent error: cannot send request
```

### Why This Happened
Postman's default Cloud Agent routes requests through Postman's servers in the cloud. Requests to `localhost` cannot be routed through cloud servers because `localhost` only exists on your local machine.

### Fix Attempted — Desktop Agent
Downloaded and installed the Postman Desktop Agent to route requests locally instead of through the cloud. The Desktop Agent failed to stay open on this machine.

### Resolution
Switched to **curl** in Git Bash — a more powerful and industry-standard approach. In real enterprise environments, IAM engineers use curl constantly for testing and debugging token endpoints.

> **Lesson:** curl is a core tool for every IAM and security engineer. Being comfortable with curl means you can test any OAuth endpoint from any machine without needing a GUI tool.

---

## Step 3 — Configure the Client in Keycloak

Before requesting a token, the client (`my-test-app`) needed to be properly configured.

### Issue Encountered — Invalid Client Error
```json
{"error":"invalid_client","error_description":"Invalid client or Invalid client credentials"}
```

### Why This Happened
The client was configured as a **Confidential client** — meaning it requires a client secret in addition to user credentials. We had not provided the client secret in our request.

### Fix — Switch to Public Client
1. Go to `http://localhost:8080`
2. Switch to **iam-lab** realm
3. Click **Clients** → **my-test-app**
4. Click **Settings** tab
5. Under **Capability config** — turn OFF **Client authentication**
6. Turn ON **Direct access grants** — required for ROPC flow
7. Click **Save**

### Public vs Confidential Clients Explained

| Type | Client Secret Required | Use Case |
|------|----------------------|---------|
| **Public** | No | Mobile apps, SPAs, testing |
| **Confidential** | Yes | Backend web apps, APIs |

> **Real World Note:** In production, confidential clients with rotating secrets are always preferred. Public clients are used for mobile and single-page applications where storing a secret securely is not possible.

---

## Step 4 — Request an Access Token Using curl

### The curl Command

```bash
curl -X POST http://localhost:8080/realms/iam-lab/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password" \
  -d "client_id=my-test-app" \
  -d "username=testuser" \
  -d 'password=YourPassword'
```

### Breaking Down the Command

| Part | What It Does |
|------|-------------|
| `curl -X POST` | Send an HTTP POST request |
| `http://localhost:8080/realms/iam-lab/protocol/openid-connect/token` | Keycloak token endpoint for iam-lab realm |
| `-H "Content-Type: application/x-www-form-urlencoded"` | Tell Keycloak we are sending form data |
| `-d "grant_type=password"` | Use Resource Owner Password flow |
| `-d "client_id=my-test-app"` | Identify which application is requesting the token |
| `-d "username=testuser"` | The user authenticating |
| `'password=YourPassword'` | The user's password (single quotes prevent bash from misreading special characters) |

---

## Step 5 — Troubleshooting Errors Encountered

### Error 1 — Invalid Client
```json
{"error":"invalid_client","error_description":"Invalid client or Invalid client credentials"}
```
**Cause:** Client authentication was enabled — client secret was required but not provided.  
**Fix:** Disabled client authentication (switched to public client) and enabled Direct access grants.

---

### Error 2 — Invalid User Credentials
```json
{"error":"invalid_grant","error_description":"Invalid user credentials"}
```
**Cause:** The password placeholder `yourpassword` was typed literally instead of the actual password.  
**Fix:** Replaced with the actual testuser password.

---

### Error 3 — Failed to Decode URL / IllegalArgumentException
```
java.lang.IllegalArgumentException: Failed to decode URL
```
**Cause:** The password contained a special character (such as `!`) that Git Bash interpreted as a bash history expansion command, corrupting the request.  
**Fix:** Wrapped the password value in single quotes instead of double quotes:
```bash
-d 'password=YourPassword!'
```

> **Important Lesson:** Special characters in passwords must be properly escaped or quoted when used in command line tools. This is a common issue IAM engineers encounter when scripting token requests. In production environments, passwords and secrets are pulled from secret vaults — never typed directly into terminals.

---

## Step 6 — Successful Token Response

After fixing all errors, Keycloak returned a successful token response:

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 300,
  "refresh_expires_in": 1800,
  "refresh_token": "eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "not-before-policy": 0,
  "session_state": "Rf9iGiXuz2tfEC0s_C8222JV",
  "scope": "email profile"
}
```

### Breaking Down the Token Response

| Field | Value | What It Means |
|-------|-------|--------------|
| `access_token` | Long JWT string | The token the app uses to prove the user's identity |
| `expires_in` | 300 (5 minutes) | Access token expires in 5 minutes — short for security |
| `refresh_expires_in` | 1800 (30 minutes) | Refresh token valid for 30 minutes |
| `refresh_token` | Long JWT string | Used to get a new access token without re-logging in |
| `token_type` | Bearer | How the token is sent — in the Authorization header |
| `session_state` | UUID | Unique identifier for this login session |
| `scope` | email profile | What user data this token has access to |

---

## Step 7 — Decode and Inspect the JWT Token

### What is a JWT?

A JWT (JSON Web Token) has 3 parts separated by dots:

```
header.payload.signature
eyJhbGci...  .  eyJleHAi...  .  GW5VCTf5...
```

| Part | Contents | Purpose |
|------|---------|---------|
| **Header** | Algorithm used to sign the token | Tells the app how to verify the signature |
| **Payload** | User data, roles, expiry | The actual claims about the user |
| **Signature** | Cryptographic hash | Proves the token was issued by Keycloak and not tampered with |

### Decoding the Token

Go to **https://jwt.io** and paste the access token into the Encoded box.

### What Was Inside the Decoded Token

```json
{
  "exp": 1779740853,
  "iat": 1779740553,
  "jti": "onrtro:f31a28b1-2a78-f18d-f941-883497ed8d16",
  "iss": "http://localhost:8080/realms/iam-lab",
  "aud": "account",
  "sub": "d3f38892-addc-4f08-a1ea-929b66f024f8",
  "typ": "Bearer",
  "azp": "my-test-app",
  "sid": "Rf9iGiXuz2tfEC0s_C8222JV",
  "realm_access": {
    "roles": [
      "default-roles-iam-lab",
      "user-role",
      "offline_access",
      "uma_authorization"
    ]
  },
  "scope": "email profile",
  "email_verified": true,
  "name": "test user",
  "preferred_username": "testuser",
  "given_name": "test",
  "family_name": "user",
  "email": "testuser@iam-lab.local"
}
```

### Breaking Down the Token Claims

| Claim | Value | What It Means |
|-------|-------|--------------|
| `exp` | Unix timestamp | When the token expires |
| `iat` | Unix timestamp | When the token was issued |
| `iss` | `http://localhost:8080/realms/iam-lab` | Who issued the token — our Keycloak IdP |
| `sub` | UUID | The unique ID of the authenticated user |
| `azp` | `my-test-app` | Which application requested the token |
| `realm_access.roles` | `user-role`, etc. | The roles assigned to this user |
| `preferred_username` | `testuser` | The username |
| `email` | `testuser@iam-lab.local` | The user's email |
| `email_verified` | `true` | Whether the email has been verified |

---

## Key Concepts Learned

| Concept | Definition |
|---------|-----------|
| **OAuth 2.0** | Authorization framework — how apps get tokens |
| **ROPC Flow** | Direct username/password exchange for tokens — testing only |
| **JWT** | JSON Web Token — the format of the access token |
| **Bearer Token** | A token that grants access to whoever holds it |
| **Claims** | Data embedded inside a JWT token |
| **Token Expiry** | Access tokens expire quickly for security |
| **Refresh Token** | Gets new access tokens without re-login |
| **Public Client** | Client that does not require a secret |
| **Confidential Client** | Client that requires a secret — production standard |

---

## Real World Application

In enterprise environments this token exchange happens automatically when:
- An employee opens Salesforce and is automatically logged in via SSO
- A microservice calls another microservice's API using a service account token
- A mobile app authenticates a user and stores a refresh token for persistent login
- A CI/CD pipeline authenticates to deploy code using client credentials

As an IAM Engineer you will:
- Configure token lifetimes based on security requirements
- Debug token issues when applications fail to authenticate
- Inspect tokens to verify roles and claims are correct
- Set up token revocation for incident response
- Audit token usage through Keycloak events

---

## Troubleshooting Reference

| Error | Cause | Fix |
|-------|-------|-----|
| `invalid_client` | Client secret required but not provided | Disable client authentication or provide client secret |
| `invalid_grant` | Wrong username or password | Verify credentials are correct |
| `Failed to decode URL` | Special character in password breaking bash | Wrap password in single quotes |
| `Cloud agent error` | Postman cloud cannot reach localhost | Use Desktop Agent or switch to curl |
| `unauthorized_client` | Direct access grants not enabled | Enable Direct access grants in client settings |

---

## Next Steps

- [ ] Lab 03 — Inspecting and Validating Tokens with Token Introspection
- [ ] Lab 04 — OAuth 2.0 Client Credentials Flow (Machine to Machine)
- [ ] Lab 05 — MFA Setup with TOTP and WebAuthn
- [ ] Lab 06 — Microsoft Entra ID SSO Configuration

---

## Resources

- [OAuth 2.0 RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749)
- [JWT Decoder](https://jwt.io)
- [Keycloak Token Endpoint Docs](https://www.keycloak.org/docs/latest/securing_apps/)
- [OAuth 2.0 Flows Explained](https://auth0.com/docs/get-started/authentication-and-authorization-flow)

---

*Part of the [Classified Builds](https://github.com/Trishsteezy/IAM-projects) portfolio — building real IAM expertise in public.*
