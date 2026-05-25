# Lab 01 — Keycloak Identity Provider Setup

**Author:** Trisha Trafalgar (Trishsteezy)  
**Date:** May 2026  
**Difficulty:** Beginner  
**Time to Complete:** 1–2 hours  
**YouTube Walkthrough:** *(link coming soon)*

---

## Objective

Deploy and configure a self-hosted Identity Provider (IdP) using Keycloak on a local Windows 11 machine. This lab demonstrates how an enterprise Identity Provider works, how to manage realms, users, and clients, and how OpenID Connect (OIDC) authentication flows are structured.

---

## What is Keycloak?

Keycloak is an open-source Identity and Access Management solution used by enterprises to manage authentication and authorization. It supports industry-standard protocols including OAuth 2.0, OpenID Connect (OIDC), and SAML 2.0. Running your own Keycloak instance simulates what a real enterprise IdP looks like — the same concepts used in Okta, Microsoft Entra ID, and Ping Identity.

---

## Tools & Technologies Used

| Tool | Version | Purpose |
|------|---------|---------|
| Keycloak | 26.6.2 | Identity Provider |
| Eclipse Temurin JDK | 25.0.3 | Java runtime for Keycloak |
| Windows 11 | — | Host operating system |
| Git Bash | 2.54.0 | Terminal / command line |
| Chrome / Edge | — | Browser for admin console |

---

## Prerequisites

- Windows 11 machine with 8GB+ RAM
- Git Bash installed (git-scm.com)
- Internet connection for downloads
- Admin rights on your machine

---

## Architecture Overview

```
[Browser] → [Keycloak IdP :8080] → [Realm: iam-lab]
                                         ↓
                              [Clients / Applications]
                                         ↓
                                [Users & Groups]
```

---

## Step 1 — Install Java (Eclipse Temurin JDK)

Keycloak requires Java to run. We installed Eclipse Temurin JDK 25 from Adoptium.

**Download:** https://adoptium.net

**Installation settings:**
- Installed for all users
- Enabled: Set JAVA_HOME variable
- Enabled: JavaSoft registry keys
- Location: `C:\Program Files\Eclipse Adoptium\`

**Verify installation:**
```bash
java -version
```

**Expected output:**
```
openjdk version "25.0.3" 2026-04-21 LTS
OpenJDK Runtime Environment Temurin-25.0.3+9
OpenJDK 64-Bit Server VM Temurin-25.0.3+9
```

📸 *Screenshot: screenshots/01-java-version-confirmed.png*

---

## Step 2 — Download Keycloak

Downloaded the Keycloak ZIP file from the official site.

**Download:** https://keycloak.org/downloads

**Version downloaded:** Keycloak 26.6.2 (ZIP)

**Extracted to:** `C:\Users\Tsteezy\Desktop\keycloak-26.6.2\`

📸 *Screenshot: screenshots/02-keycloak-extracted.png*

---

## Step 3 — Start Keycloak in Development Mode

Navigated to the Keycloak directory in Git Bash and started the server.

```bash
cd /c/Users/Tsteezy/Desktop/keycloak-26.6.2/keycloak-26.6.2
bin/kc.bat start-dev
```

> **Note:** `start-dev` runs Keycloak in development mode with an in-memory H2 database. Do NOT use this configuration in production.

**What to look for in the output:**
```
Keycloak 26.6.2 on JVM started in 11.277s
Listening on: http://localhost:8080
Profile dev activated
```

📸 *Screenshot: screenshots/03-keycloak-running.png*

---

## Step 4 — Create Admin Account

Opened browser and navigated to:
```
http://localhost:8080
```

On the welcome page, created the initial admin account:
- **Username:** admin
- **Password:** *(secure password set)*

📸 *Screenshot: screenshots/04-keycloak-welcome-page.png*  
📸 *Screenshot: screenshots/05-admin-account-created.png*

---

## Step 5 — Access the Admin Console

Clicked **Administration Console** on the welcome page and logged in with admin credentials.

**Admin Console URL:**
```
http://localhost:8080/admin
```

Successfully accessed the Keycloak dashboard showing the default **master** realm with 6 built-in clients:
- account (OpenID Connect)
- account-console (OpenID Connect)
- admin-cli (OpenID Connect)
- broker (OpenID Connect)
- master-realm
- security-admin-console

📸 *Screenshot: screenshots/06-admin-console-clients.png*

---

## Step 6 — Create a New Realm

In a real enterprise, each business unit or application environment gets its own **realm** — an isolated identity domain.

**Steps:**
1. Click **Manage realms** in the left sidebar
2. Click **Create realm**
3. Set Realm name: `iam-lab`
4. Toggle **Enabled** to ON
5. Click **Create**

📸 *Screenshot: screenshots/07-create-realm.png*  
📸 *Screenshot: screenshots/08-iam-lab-realm-created.png*

---

## Step 7 — Create a User

Inside the `iam-lab` realm, created a test user to simulate an employee identity.

**Steps:**
1. Click **Users** in the left sidebar
2. Click **Create new user**
3. Fill in:
   - **Username:** testuser
   - **Email:** testuser@iam-lab.local
   - **First name:** Test
   - **Last name:** User
   - Toggle **Email verified** ON
4. Click **Create**
5. Go to **Credentials** tab → Set password → Toggle **Temporary** OFF

📸 *Screenshot: screenshots/09-create-user.png*  
📸 *Screenshot: screenshots/10-user-credentials-set.png*

---

## Step 8 — Create a Client (Application)

A **client** in Keycloak represents an application that will authenticate users through the IdP. We created a test client to simulate an app using OIDC.

**Steps:**
1. Click **Clients** in the left sidebar
2. Click **Create client**
3. Fill in:
   - **Client type:** OpenID Connect
   - **Client ID:** my-test-app
4. Click **Next**
5. Enable **Standard flow** (Authorization Code Flow)
6. Click **Next**
7. Set **Valid redirect URIs:** `http://localhost:3000/*`
8. Click **Save**

📸 *Screenshot: screenshots/11-create-client.png*  
📸 *Screenshot: screenshots/12-client-created.png*

---

## Step 9 — Test the Login Flow

Tested authentication by accessing the account console for the `iam-lab` realm.

**URL:**
```
http://localhost:8080/realms/iam-lab/account
```

Logged in with the testuser credentials to verify the full authentication flow works end to end.

📸 *Screenshot: screenshots/13-login-flow-test.png*  
📸 *Screenshot: screenshots/14-user-logged-in.png*

---

## Key Concepts Learned

| Concept | Definition |
|---------|-----------|
| **Realm** | An isolated identity domain — like a company's identity space |
| **Client** | An application that authenticates users through Keycloak |
| **User** | An identity managed by the IdP |
| **OpenID Connect (OIDC)** | The protocol used for authentication |
| **Authorization Code Flow** | The most secure OAuth 2.0 flow for web apps |
| **IdP (Identity Provider)** | The system that verifies who you are |

---

## Real World Application

This lab simulates what enterprise IAM engineers do when:
- Onboarding a new application to SSO
- Managing user identities centrally
- Implementing Zero Trust — verify every user, every time
- Replacing password-based auth with token-based auth

Tools like **Okta**, **Microsoft Entra ID**, and **Ping Identity** work on the same principles as Keycloak — just at enterprise scale.

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `java: command not found` | Close and reopen Git Bash after Java install |
| `bin/kc.bat: No such file` | Make sure you're in the correct nested folder |
| Port 8080 already in use | Run `netstat -ano | findstr :8080` to find and kill the process |
| Can't access localhost:8080 | Make sure Keycloak is still running in Git Bash |

---

## Next Steps

- [ ] Lab 02 — OAuth 2.0 Authorization Code Flow with Postman
- [ ] Lab 03 — Microsoft Entra ID SSO Configuration
- [ ] Lab 04 — SAML 2.0 Integration
- [ ] Lab 05 — SCIM 2.0 User Provisioning

---

## Resources

- [Keycloak Official Docs](https://www.keycloak.org/documentation)
- [OAuth 2.0 Explained](https://oauth.net/2/)
- [OpenID Connect Explained](https://openid.net/connect/)
- [IDPro Body of Knowledge](https://idpro.org/body-of-knowledge/)

---

*Part of the [Classified Builds](https://github.com/Trishsteezy/IAM-projects) portfolio — building real IAM experience in public.*
