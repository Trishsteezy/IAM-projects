# Lab 02 — JWT Token Decoded: Findings, Analysis & Real World Importance

**Author:** Trisha Trafalgar (Trishsteezy)  
**Date:** May 2026  
**Lab:** OAuth 2.0 Token Request & JWT Inspection  
**YouTube Walkthrough:** *(paste YouTube link here)*

---

## What We Did

Successfully requested a real JWT access token from our self-hosted Keycloak Identity Provider using the OAuth 2.0 Resource Owner Password Credentials (ROPC) flow via curl in Git Bash. Decoded and inspected the token using jwt.io to understand exactly what data is embedded inside every enterprise authentication event.

---

## The Full Decoded JWT Payload

This is the actual token issued by our Keycloak iam-lab realm for testuser:

```json
{
  "exp": 1779741978,
  "iat": 1779741678,
  "jti": "onrtro:a3f8aa28-2047-91d1-e9a5-593dfc1431ea",
  "iss": "http://localhost:8080/realms/iam-lab",
  "aud": "account",
  "sub": "d3f38892-addc-4f08-a1ea-929b66f024f8",
  "typ": "Bearer",
  "azp": "my-test-app",
  "sid": "-zAmrWN_gAJEgEd86pGOvk4v",
  "acr": "1",
  "allowed-origins": [""],
  "realm_access": {
    "roles": [
      "default-roles-iam-lab",
      "user-role",
      "offline_access",
      "uma_authorization"
    ]
  },
  "resource_access": {
    "account": {
      "roles": [
        "manage-account",
        "manage-account-links",
        "view-profile"
      ]
    }
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

---

## Field by Field Analysis

### Identity Claims — WHO is this user?

| Field | Value | What It Means |
|-------|-------|--------------|
| `sub` | `d3f38892-addc-4f08-a1ea-929b66f024f8` | The unique subject ID — Keycloak's internal UUID for this user. This never changes even if the username changes. |
| `preferred_username` | `testuser` | The human-readable username |
| `name` | `test user` | Full display name |
| `given_name` | `test` | First name |
| `family_name` | `user` | Last name |
| `email` | `testuser@iam-lab.local` | Email address |
| `email_verified` | `true` | We verified this in Lab 01 when creating the user |

### Token Lifecycle Claims — WHEN is this token valid?

| Field | Value | What It Means |
|-------|-------|--------------|
| `iat` | 1779741678 | Issued At — Unix timestamp of when the token was created |
| `exp` | 1779741978 | Expiration — Unix timestamp when the token expires |
| `jti` | UUID | JWT ID — a unique identifier for this specific token instance |

> **Security Note:** The difference between `exp` and `iat` is 300 seconds — exactly 5 minutes. This is the access token lifetime we configured in Keycloak. Short lifetimes are a security best practice — a stolen token is only usable for 5 minutes before it expires.

### Trust Claims — WHO issued this token and WHO can use it?

| Field | Value | What It Means |
|-------|-------|--------------|
| `iss` | `http://localhost:8080/realms/iam-lab` | Issuer — our Keycloak IdP. Applications verify this matches the expected issuer before trusting the token. |
| `aud` | `account` | Audience — which service this token is intended for |
| `azp` | `my-test-app` | Authorized Party — which client application requested this token |
| `sid` | UUID | Session ID — links this token to a specific login session |

### Authorization Claims — WHAT can this user access?

| Field | Value | What It Means |
|-------|-------|--------------|
| `realm_access.roles` | `user-role`, `offline_access`, `uma_authorization`, `default-roles-iam-lab` | The roles assigned to this user at the realm level — these came directly from our role assignment in Lab 01 |
| `resource_access.account.roles` | `manage-account`, `manage-account-links`, `view-profile` | Client-level roles for the account management console |
| `scope` | `email profile` | What categories of user data this token has access to |
| `acr` | `1` | Authentication Context Class Reference — indicates the strength of authentication used (1 = password only) |

---

## Key Finding — Lab 01 Work Appears in Lab 02 Token

The most important discovery in this lab:

**The `user-role` we created and assigned in Lab 01 appears inside this JWT token.**

```json
"realm_access": {
  "roles": [
    "user-role"  ← This is the role we created and assigned in Lab 01
  ]
}
```

This proves the full IAM chain is working:

```
Lab 01: Created role → Assigned to user → Added user to group
              ↓
Lab 02: User authenticates → Keycloak issues token → Token contains role
              ↓
Real World: Application reads token → Grants access based on role
```

This is Role-Based Access Control (RBAC) working end to end.

---

## Errors Encountered and Fixed

### Error 1 — Cloud Agent Error in Postman
**Error:** `Cloud agent error: cannot send request`  
**Cause:** Postman's cloud agent cannot route requests to localhost — localhost only exists on your local machine, not in the cloud.  
**Fix:** Switched to curl in Git Bash — more powerful and the industry standard for API testing.  
**Lesson:** curl is a core tool for IAM and security engineers. Master it.

---

### Error 2 — Invalid Client
**Error:** `{"error":"invalid_client","error_description":"Invalid client or Invalid client credentials"}`  
**Cause:** Client was configured as Confidential — requiring a client secret we did not provide.  
**Fix:** Switched client to Public type and enabled Direct access grants in Keycloak client settings.  
**Lesson:** Public clients don't require secrets. Confidential clients do. Always use confidential clients in production.

---

### Error 3 — Invalid User Credentials
**Error:** `{"error":"invalid_grant","error_description":"Invalid user credentials"}`  
**Cause:** The literal text `yourpassword` was used instead of the actual password.  
**Fix:** Replaced placeholder with actual testuser password.  
**Lesson:** Always verify credentials before debugging deeper issues.

---

### Error 4 — Failed to Decode URL
**Error:** `java.lang.IllegalArgumentException: Failed to decode URL`  
**Cause:** Password contained special characters that Git Bash interpreted as bash commands, corrupting the request before it reached Keycloak.  
**Fix:** Wrapped the password value in single quotes instead of double quotes.  
```bash
# Wrong - double quotes let bash interpret special characters
-d "password=MyPass!"

# Correct - single quotes prevent bash interpretation
-d 'password=MyPass!'
```
**Lesson:** In production, passwords and secrets are NEVER typed into terminals. They are pulled from secret management tools like HashiCorp Vault, AWS Secrets Manager, or Azure Key Vault. This prevents both the bash interpretation issue and prevents secrets from appearing in command history logs.

---

### Error 5 — URL Breaking Across Lines
**Error:** `{"error":"HTTP 405 Method Not Allowed"}` and `bash: n: command not found`  
**Cause:** The curl command was being pasted with line breaks, splitting the URL across two lines — `toke` on one line and `n` on the next.  
**Fix:** Pasted the entire command on a single line with no line breaks.  
**Lesson:** When copying multi-line commands from documentation into a terminal, always verify the URL is complete and unbroken.

---

## Why This Lab Matters — Real World Importance

### 1. This is What SSO Looks Like Behind the Scenes
Every time an employee logs into Salesforce, Workday, ServiceNow, or any enterprise application through Single Sign-On — this exact token exchange is happening in milliseconds. The user never sees it. The application validates the token and knows exactly who the user is and what they can access.

### 2. Tokens Are the Target
Tokens are what attackers want. A stolen access token gives an attacker:
- The user's identity
- The user's roles and permissions
- Access to every application that trusts the token
- All without ever knowing the user's password

This is why token lifetime (5 minutes in our lab) and token revocation are critical security controls.

### 3. Roles in Tokens = Authorization Decisions
The `realm_access.roles` field inside the token is how applications make access decisions without calling the database on every request. The application reads the token, sees `user-role`, and knows what this user can and cannot do. No additional lookup required.

### 4. The `sub` Claim is the True Identity
The `preferred_username` can change. The email can change. But the `sub` UUID never changes. In enterprise systems, the `sub` is the immutable identifier used to link a user across all systems — even if they change their name or email.

### 5. Audit Trail Starts Here
The `jti` (JWT ID) and `sid` (Session ID) in the token are what security teams use during incident response to:
- Track exactly which tokens were issued
- Identify which sessions were active during a breach
- Correlate token usage across systems in a SIEM

### 6. The `acr` Claim Drives MFA Policy
The `acr: 1` in our token means password-only authentication. In enterprise Zero Trust environments, applications check the `acr` value and reject tokens with weak authentication levels. If `acr` is too low, the application forces re-authentication with MFA. This is how step-up authentication works.

### 7. Short Token Lifetimes Are Non-Negotiable
Our token expires in 5 minutes. This is intentional. If an attacker steals a token:
- They have a maximum 5-minute window to use it
- After expiry, the refresh token (30 minutes) must be used
- After the refresh token expires, full re-authentication is required

In a breach scenario this limits the blast radius significantly.

---

## What IAM Engineers Do With This Knowledge Daily

- **Debug authentication failures** by requesting tokens manually and inspecting claims
- **Verify role assignments** by checking token contents after configuration changes
- **Configure token lifetimes** based on application security requirements and compliance mandates
- **Implement token validation** in applications to verify issuer, audience, and signature
- **Set up token revocation** so stolen tokens can be invalidated immediately
- **Monitor token usage** through Keycloak events and SIEM integration
- **Design MFA policies** using the `acr` claim for step-up authentication

---

## Screenshots

| Screenshot | Description |
|-----------|-------------|
| `screenshots/jwt-decoded-payload.png` | Full decoded JWT payload showing all claims |
| `screenshots/curl-token-response.png` | Successful curl token request response |
| `screenshots/keycloak-client-settings.png` | Client configuration — public client with direct access grants |

---

## Next Steps

- [ ] Lab 03 — Token Introspection — asking Keycloak if a token is still valid
- [ ] Lab 04 — Token Revocation — invalidating tokens instantly
- [ ] Lab 05 — Client Credentials Flow — machine to machine authentication
- [ ] Lab 06 — MFA Setup with TOTP and WebAuthn
- [ ] Lab 07 — Microsoft Entra ID SSO Configuration

---

## Resources

- [JWT Decoder — jwt.io](https://jwt.io)
- [OAuth 2.0 RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749)
- [JWT RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519)
- [Keycloak Token Documentation](https://www.keycloak.org/docs/latest/securing_apps/)
- [NIST 800-63B — Authentication Guidelines](https://pages.nist.gov/800-63-3/sp800-63b.html)

---

*Part of the [Classified Builds](https://github.com/Trishsteezy/IAM-projects) portfolio — building real IAM expertise in public.*
