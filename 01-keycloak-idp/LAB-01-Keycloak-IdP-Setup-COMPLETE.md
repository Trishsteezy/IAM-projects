# Lab 01 — Keycloak Identity Provider Setup

**Author:** Trisha Trafalgar (Trishsteezy)  
**Date:** May 2026  
**Difficulty:** Beginner  
**Time to Complete:** 1–2 hours  
**YouTube Walkthrough:** *(paste YouTube link here)*

---

## Objective

Deploy and configure a self-hosted Identity Provider (IdP) using Keycloak on a local Windows 11 machine. This lab demonstrates how an enterprise Identity Provider works, how to manage realms, users, clients, roles, and groups, and how OpenID Connect (OIDC) authentication flows are structured.

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
                                         ↓
                                  [Roles & Policies]
```

---

## Step 1 — Install Java (Eclipse Temurin JDK)

Keycloak requires Java to run. Installed Eclipse Temurin JDK 25 from Adoptium.

**Download:** https://adoptium.net

**Installation settings:**
- Installed for all users
- Enabled: Set JAVA_HOME variable
- Enabled: JavaSoft registry keys
- Location: `C:\Program Files\Eclipse Adoptium\`

**Verify installation in Git Bash:**
```bash
java -version
```

**Expected output:**
```
openjdk version "25.0.3" 2026-04-21 LTS
OpenJDK Runtime Environment Temurin-25.0.3+9
OpenJDK 64-Bit Server VM Temurin-25.0.3+9
```

---

## Step 2 — Download Keycloak

Downloaded the Keycloak ZIP file from the official site.

**Download:** https://keycloak.org/downloads  
**Version:** Keycloak 26.6.2 (ZIP)  
**Extracted to:** `C:\Users\Tsteezy\Desktop\keycloak-26.6.2\`

---

## Step 3 — Start Keycloak in Development Mode

Navigated to the Keycloak directory in Git Bash and started the server.

```bash
cd /c/Users/Tsteezy/Desktop/keycloak-26.6.2/keycloak-26.6.2
bin/kc.bat start-dev
```

> **Note:** `start-dev` runs Keycloak in development mode with an in-memory H2 database. Do NOT use this configuration in production.

**Successful startup output:**
```
Keycloak 26.6.2 on JVM started in 11.277s
Listening on: http://localhost:8080
Profile dev activated
```

---

## Step 4 — Create Admin Account

Opened browser and navigated to:
```
http://localhost:8080
```

Created the initial admin account on the Keycloak welcome page:
- **Username:** admin
- **Password:** *(secure password)*

---

## Step 5 — Access the Admin Console

Clicked **Administration Console** and logged in with admin credentials.

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

---

## Step 6 — Create a New Realm

In a real enterprise, each business unit or application environment gets its own **realm** — an isolated identity domain.

**Steps:**
1. Click **Manage realms** in the left sidebar
2. Click **Create realm**
3. Set Realm name: `iam-lab`
4. Toggle **Enabled** to ON
5. Click **Create**

> **What is a Realm?** A realm is like a tenant or a company's identity space. All users, clients, roles, and groups within a realm are isolated from other realms. The master realm is Keycloak's admin realm — never use it for applications.

---

## Step 7 — Create a User

Inside the `iam-lab` realm, created a test user to simulate an employee identity.

**Steps:**
1. Click **Users** in the left sidebar
2. Click **Create new user**
3. Fill in:
   - **Username:** `testuser`
   - **Email:** `testuser@iam-lab.local`
   - **First name:** `Test`
   - **Last name:** `User`
   - Toggle **Email verified** ON
4. Click **Create**
5. Go to **Credentials** tab
6. Click **Set password**
7. Enter a password
8. Toggle **Temporary** to OFF
9. Click **Save**

> **Why toggle Temporary OFF?** When Temporary is ON, the user is forced to change their password on first login. For lab testing we turn it off for convenience. In production, Temporary ON is a security best practice for new accounts.

---

## Step 8 — Create a Client (Application)

A **client** in Keycloak represents an application that authenticates users through the IdP. Created a test client to simulate a web app using OIDC.

**Steps:**
1. Click **Clients** in the left sidebar
2. Click **Create client**
3. Fill in:
   - **Client type:** OpenID Connect
   - **Client ID:** `my-test-app`
4. Click **Next**
5. Enable **Standard flow** toggle ON
6. Click **Next**
7. Set **Valid redirect URIs:** `http://localhost:3000/*`
8. Click **Save**

> **What is Standard Flow?** Standard flow is the OAuth 2.0 Authorization Code Flow — the most secure flow for web applications. The user logs in, gets an authorization code, which is exchanged for an access token. This is how Okta, Azure AD, and every enterprise SSO system works.

---

## Step 9 — Test the Login Flow

Tested authentication by accessing the account console for the `iam-lab` realm.

**URL:**
```
http://localhost:8080/realms/iam-lab/account
```

Logged in with testuser credentials. Successfully authenticated — confirming the full IdP flow works end to end.

---

## Step 10 — Create Realm Roles

Roles define what a user is allowed to do. Created 3 roles to simulate a real enterprise access structure.

**Steps:**
1. Click **Realm roles** in the left sidebar
2. Click **Create role**
3. Created the following roles:

| Role Name | Purpose |
|-----------|---------|
| `admin-role` | Full administrative access |
| `user-role` | Standard user access |
| `read-only-role` | View-only access |

> **RBAC in action:** Role-Based Access Control (RBAC) is the most common access control model in enterprise IAM. Instead of assigning permissions directly to users, you assign roles — and roles carry the permissions. This makes managing access at scale much easier.

---

## Step 11 — Assign Role to User

Assigned the `user-role` to testuser to control their level of access.

**Steps:**
1. Click **Users** in the left sidebar
2. Click on **testuser**
3. Click the **Role mapping** tab
4. Click **Assign role**
5. Select **user-role**
6. Click **Assign**

---

## Step 12 — Create a Group

Groups allow you to manage access for many users at once. Any user added to a group automatically inherits the group's roles and permissions — this is how enterprise IAM scales to thousands of users.

**Steps:**
1. Click **Groups** in the left sidebar
2. Click **Create group**
3. Name it: `iam-team`
4. Click **Create**
5. Click on **iam-team** group
6. Click **Role mapping** tab
7. Click **Assign role**
8. Select **user-role**
9. Click **Assign**

> **Why use Groups?** Imagine onboarding 500 new employees to the same department. Instead of assigning roles to each person individually, you add them all to a group — they instantly inherit all the right permissions. This is standard practice in Okta, Active Directory, and Entra ID.

---

## Step 13 — Add User to Group

Added testuser to the `iam-team` group, giving them group-based access in addition to their directly assigned role.

**Steps:**
1. Click **Users** in the left sidebar
2. Click on **testuser**
3. Click the **Groups** tab
4. Click **Join Group**
5. Select **iam-team**
6. Click **Join**

---

## Lab Complete — What Was Built

| Component | Value |
|-----------|-------|
| Identity Provider | Keycloak 26.6.2 |
| Realm | iam-lab |
| Client | my-test-app (OIDC) |
| User | testuser |
| Roles | admin-role, user-role, read-only-role |
| Group | iam-team |
| Auth Flow | Authorization Code Flow (OIDC) |

---

## Key Concepts Learned

| Concept | Definition |
|---------|-----------|
| **Realm** | An isolated identity domain |
| **Client** | An application that authenticates through the IdP |
| **User** | An identity managed by the IdP |
| **Role** | A set of permissions assigned to users |
| **Group** | A collection of users sharing the same roles |
| **OIDC** | OpenID Connect — the authentication protocol |
| **Authorization Code Flow** | Most secure OAuth 2.0 flow for web apps |
| **RBAC** | Role-Based Access Control — permissions via roles |

---

## Real World Application

This lab simulates what enterprise IAM engineers do when:
- Onboarding a new application to SSO
- Managing user identities centrally
- Implementing RBAC for access control
- Replacing password-based auth with token-based auth
- Managing users at scale through groups

Tools like **Okta**, **Microsoft Entra ID**, and **Ping Identity** work on the same principles as Keycloak — just at enterprise scale with thousands of users and applications.

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `java: command not found` | Close and reopen Git Bash after Java install |
| `bin/kc.bat: No such file` | Make sure you are in the correct nested folder |
| Port 8080 already in use | Run `netstat -ano \| findstr :8080` to find and kill the process |
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
