# Lab 03 — Token Introspection & Revocation

**Author:** Trisha Trafalgar (Trishsteezy)  
**Date:** May 2026  
**Difficulty:** Intermediate  
**Time to Complete:** 1–2 hours  
**YouTube Walkthrough:** *(paste YouTube link here)*

---

## Objective

Simulate a real enterprise security incident response scenario using Keycloak's token introspection and session revocation capabilities. Learn how IAM engineers verify token validity and instantly terminate compromised user sessions across all connected applications.

---

## The Scenario — Real World Context

> **Security Alert:** An employee's laptop has been reported stolen. The employee had an active session with access to sensitive enterprise applications. As the IAM Engineer on call, you must immediately verify whether their tokens are still active and revoke all sessions before an attacker can use them.

This is one of the most critical IAM incident response procedures in any enterprise. Every minute a compromised session stays active is a minute an attacker has access to your systems.

---

## What is Token Introspection?

Token introspection is an OAuth 2.0 standard (RFC 7662) that allows a resource server or administrator to ask the Identity Provider one question:

**"Is this token still valid?"**

Keycloak responds with either:
- `{"active": true}` — the token is valid and can be used
- `{"active": false}` — the token is expired, revoked, or invalid

### Why It Matters
- Applications use introspection to validate tokens on every request
- Security teams use it during incident response to check if a stolen token is still usable
- It provides real-time token status — unlike JWT validation which only checks the signature and expiry

---

## What is Token Revocation?

Token revocation is the ability to instantly invalidate a token or session before it naturally expires. In Keycloak this can be done by:

1. **Revoking a specific token** — via the token revocation endpoint
2. **Terminating a user session** — via the admin console (kills all tokens for that session)
3. **Revoking all sessions for a user** — nuclear option for compromised accounts

### Why It Matters
Without revocation, stolen tokens remain valid until they naturally expire. A 30-minute access token gives an attacker a 30-minute window. Revocation closes that window instantly.

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Keycloak 26.6.2 | Identity Provider — token issuer and revoker |
| Git Bash | Command line for curl requests |
| curl | HTTP client for token requests and introspection |
| Keycloak Admin Console | Session management and revocation UI |

---

## Prerequisites

- Lab 01 complete (Keycloak running, iam-lab realm, testuser, roles, groups)
- Lab 02 complete (understanding of JWT tokens and OAuth 2.0 flows)
- Keycloak running at `http://localhost:8080`
- Client authentication enabled on `my-test-app`

---

## Step 1 — Enable Client Authentication

Token introspection requires the client to authenticate with Keycloak using a client secret. We needed to enable this on `my-test-app`.

**Steps:**
1. Go to `http://localhost:8080/admin`
2. Switch to **iam-lab** realm
3. Click **Clients** → **my-test-app**
4. Click **Settings** tab
5. Under **Capability config** — turn ON **Client authentication**
6. Click **Save**
7. Click **Credentials** tab → click **Regenerate** to generate a client secret
8. Copy the client secret

### Why Client Authentication Matters
When client authentication is OFF the client is **public** — any application can claim to be `my-test-app` without proving it. When it is ON the client must provide a secret to prove its identity before Keycloak will respond to introspection requests.

In production ALL clients that call sensitive endpoints like introspection must be confidential clients with rotating secrets.

---

## Step 2 — Request a Token with Client Secret

Now that client authentication is enabled the token request must include the client secret.

```bash
curl -s -X POST http://localhost:8080/realms/iam-lab/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password" \
  -d "client_id=my-test-app" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "username=testuser" \
  -d 'password=YourPassword'
```

Save the response to a file for easy access:

```bash
curl -s -X POST http://localhost:8080/realms/iam-lab/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password" \
  -d "client_id=my-test-app" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "username=testuser" \
  -d 'password=YourPassword' > token.json
```

---

## Step 3 — Token Introspection via curl (Attempted)

### The Introspection Endpoint

```
POST http://localhost:8080/realms/iam-lab/protocol/openid-connect/token/introspect
```

### The curl Command

```bash
curl -s -X POST http://localhost:8080/realms/iam-lab/protocol/openid-connect/token/introspect \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "token=YOUR_ACCESS_TOKEN" \
  -d "client_id=my-test-app" \
  -d "client_secret=YOUR_CLIENT_SECRET"
```

### Expected Response — Active Token
```json
{
  "active": true,
  "exp": 1779775862,
  "iat": 1779774062,
  "jti": "onrtro:5f6311e7-ebe5-3f08-4cce-e1297dd40cd7",
  "iss": "http://localhost:8080/realms/iam-lab",
  "sub": "d3f38892-addc-4f08-a1ea-929b66f024f8",
  "typ": "Bearer",
  "azp": "my-test-app",
  "realm_access": {
    "roles": ["user-role", "offline_access", "uma_authorization"]
  },
  "preferred_username": "testuser",
  "email": "testuser@iam-lab.local"
}
```

### Expected Response — Expired or Revoked Token
```json
{"active": false}
```

---

## Issues Encountered & Fixed

### Issue 1 — Unauthorized Client Error
**Error:**
```json
{"error":"unauthorized_client","error_description":"Invalid client or Invalid client credentials"}
```
**Cause:** Client authentication was enabled but the curl command did not include the client secret.  
**Fix:** Added `client_secret` parameter to every curl request.  
**Lesson:** Once client authentication is enabled on a confidential client, ALL requests from that client must include the client secret.

---

### Issue 2 — Token Returning `active: false` Immediately
**Error:** `{"active": false}` even on freshly issued tokens.  
**Cause:** Multiple causes investigated:
- Token was expiring before introspection ran (5-minute lifetime)
- Token.json file contained an old expired token from a previous session
- Git Bash on Windows was corrupting the token variable when passed through `$TOKEN`

**Fix Attempts:**
- Increased Access Token Lifespan to 30 minutes in Realm settings → Tokens tab
- Decoded token manually using base64 to verify contents
- Confirmed token was genuinely valid by checking `exp` vs current Unix timestamp (`date +%s`)
- Tried multiple variable extraction methods

**Root Cause Identified:** Git Bash on Windows has known compatibility issues with passing large strings through environment variables to curl. The token was being truncated or corrupted in transit.

**Lesson:** In real enterprise environments token introspection scripts run on Linux servers where this issue does not exist. On Windows, use the Keycloak Admin Console for session management or use PowerShell instead of Git Bash for curl commands.

---

### Issue 3 — Token Lifetime Not Updating
**Symptom:** Even after changing Access Token Lifespan to 30 minutes, `expires_in` still showed 300 (5 minutes) in token responses.  
**Cause:** The token.json file was cached from before the change. New tokens were being requested but old token.json was being read.  
**Fix:** Always get a fresh token after changing realm settings.  
**Lesson:** Token lifetime changes only apply to newly issued tokens — existing tokens keep their original lifetime.

---

### Issue 4 — Client Secret Regenerated Mid-Lab
**Symptom:** Authentication started failing after previously working.  
**Cause:** Client secret was accidentally regenerated in Keycloak, invalidating the old secret used in curl commands.  
**Fix:** Updated all curl commands with the new client secret.  
**Lesson:** Client secret rotation is a security best practice — but it must be coordinated. When you rotate a secret in production, all applications using the old secret will fail until updated. This is why secret management tools like HashiCorp Vault are essential.

---

## Step 4 — Token Revocation via Admin Console

When curl-based introspection had compatibility issues on Windows, we used the Keycloak Admin Console to demonstrate session revocation — which is the primary method used in real enterprise incident response.

### Steps to Revoke a Session

1. Go to `http://localhost:8080/admin`
2. Switch to **iam-lab** realm
3. Click **Sessions** in the left sidebar
4. Find the active session for **testuser**
5. Click **Sign out** next to the session

**Result:** testuser's session was immediately terminated. All tokens associated with that session became invalid instantly.

### What Happens When You Revoke a Session

```
Before Revocation:
testuser session → active tokens → access to all connected apps

After Revocation:
testuser session → TERMINATED → all tokens invalid → no access
```

Any application that tries to use the revoked token will receive `{"active": false}` on introspection — or a 401 Unauthorized error on protected API calls.

---

## Step 5 — Verify Revocation Worked

After revoking the session, ran introspection again:

```bash
TOKEN=$(grep -o '"access_token":"[^"]*' token.json | cut -c17-)
curl -s -X POST http://localhost:8080/realms/iam-lab/protocol/openid-connect/token/introspect \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "token=$TOKEN&client_id=my-test-app&client_secret=YOUR_CLIENT_SECRET"
```

**Result:** `{"active": false}`

Token confirmed dead. ✅

---

## Decoding the Token to Verify Contents

To verify the token was genuinely valid before revocation we decoded it using base64:

```bash
cat token.json | grep -o '"access_token":"[^"]*' | cut -c17- | cut -d'.' -f2 | base64 -d 2>/dev/null
```

**Output confirmed:**
```json
{
  "exp": 1779775862,
  "iat": 1779774062,
  "preferred_username": "testuser",
  "realm_access": {
    "roles": ["user-role"]
  }
}
```

Current Unix timestamp: `1779774127` — confirmed the token was valid (between iat and exp) at time of revocation.

---

## Key Concepts Learned

| Concept | Definition |
|---------|-----------|
| **Token Introspection** | Asking the IdP in real time whether a token is still valid |
| **Token Revocation** | Instantly invalidating a token or session before natural expiry |
| **active: true** | Token is valid — user is authenticated and authorized |
| **active: false** | Token is expired, revoked, or invalid |
| **Session Termination** | Killing all tokens associated with a login session |
| **Client Secret** | Password for a confidential client — required for introspection |
| **Token Lifetime** | How long a token is valid — shorter = more secure |
| **Unix Timestamp** | How computers represent time — seconds since January 1, 1970 |

---

## Real World Application

### Incident Response Scenario
When a security team receives an alert about a compromised account:

1. **Identify** — find the user's active sessions in the IdP
2. **Verify** — use token introspection to confirm tokens are still active
3. **Revoke** — terminate all sessions immediately
4. **Confirm** — verify tokens now return `active: false`
5. **Document** — log the incident with timestamps for compliance

This entire process can be completed in under 60 seconds in Keycloak — limiting the attacker's window of access.

### Why Every IAM Engineer Must Know This
- **Compliance** — SOX, HIPAA, FedRAMP all require the ability to immediately terminate access
- **Incident Response** — compromised credentials are the #1 attack vector
- **Zero Trust** — never trust, always verify — introspection enables continuous verification
- **Audit Trail** — every session termination is logged in Keycloak events

### Enterprise Tools That Use These Concepts
- **Okta** — Session Management and Universal Logout
- **Microsoft Entra ID** — Continuous Access Evaluation (CAE)
- **Ping Identity** — Token Mediation Service
- **CrowdStrike** — Identity Threat Detection and Response (ITDR)

---

## Technical Notes — Git Bash Compatibility

A significant portion of this lab involved troubleshooting Git Bash on Windows compatibility with curl and large token strings. Key findings:

- Git Bash environment variables can corrupt large strings (1300+ characters)
- The `--data-urlencode` flag behaves differently on Windows vs Linux
- Token files saved with `> token.json` work more reliably than variables
- In production environments, token introspection scripts always run on Linux

**Recommendation for Windows users:** Use Windows Subsystem for Linux (WSL) for more reliable curl behavior with large payloads, or use the Keycloak Admin Console for session management.

---

## Troubleshooting Reference

| Issue | Cause | Fix |
|-------|-------|-----|
| `unauthorized_client` | Missing client secret | Add `client_secret` to request |
| `active: false` immediately | Expired token or corrupted variable | Get fresh token, use token file |
| Token lifetime not changing | Reading cached token.json | Delete token.json and get fresh token |
| Client secret stopped working | Secret was regenerated | Update all commands with new secret |
| Variable shows only "token" | Variable cleared between sessions | Re-run TOKEN= command in same session |
| curl returns wrong result | Git Bash string corruption | Use Admin Console for revocation on Windows |

---

## Commands Reference

### Get Token with Client Secret
```bash
curl -s -X POST http://localhost:8080/realms/iam-lab/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password" \
  -d "client_id=my-test-app" \
  -d "client_secret=YOUR_SECRET" \
  -d "username=testuser" \
  -d 'password=YourPassword' > token.json
```

### Extract and Verify Token
```bash
TOKEN=$(grep -o '"access_token":"[^"]*' token.json | cut -c17-)
echo $TOKEN | wc -c
```

### Decode Token Without jwt.io
```bash
cat token.json | grep -o '"access_token":"[^"]*' | cut -c17- | cut -d'.' -f2 | base64 -d 2>/dev/null
```

### Check Current Unix Timestamp
```bash
date +%s
```

### Introspect Token
```bash
curl -s -X POST http://localhost:8080/realms/iam-lab/protocol/openid-connect/token/introspect \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "token=YOUR_TOKEN" \
  -d "client_id=my-test-app" \
  -d "client_secret=YOUR_SECRET"
```

### Revoke via Admin Console
```
http://localhost:8080/admin → iam-lab realm → Sessions → Sign out
```

---

## Next Steps

- [ ] Lab 04 — MFA Setup with TOTP (Google Authenticator)
- [ ] Lab 05 — Microsoft Entra ID SSO Configuration
- [ ] Lab 06 — Active Directory / LDAP Federation
- [ ] Lab 07 — Custom Authentication Flows

---

## Resources

- [OAuth 2.0 Token Introspection RFC 7662](https://datatracker.ietf.org/doc/html/rfc7662)
- [OAuth 2.0 Token Revocation RFC 7009](https://datatracker.ietf.org/doc/html/rfc7009)
- [Keycloak Session Management Docs](https://www.keycloak.org/docs/latest/server_admin/#managing-user-sessions)
- [NIST 800-63B — Session Management](https://pages.nist.gov/800-63-3/sp800-63b.html)

---

*Part of the [Classified Builds](https://github.com/Trishsteezy/IAM-projects) portfolio — building real IAM expertise in public.*
