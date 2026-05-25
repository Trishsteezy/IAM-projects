# Tools & Applications Reference — Classified Builds

**Author:** Trisha Trafalgar (Trishsteezy)  
**Last Updated:** May 2026  
**Purpose:** Master reference of every tool and application used across all labs — what it is, why it is used, and how it fits into real world IAM and cybersecurity work.

---

## How to Read This Document

Every tool listed here was used hands-on in a lab. Each entry includes:
- **What it is** — plain English explanation
- **Why IAM engineers use it** — real world context
- **How we used it** — specific lab usage
- **Where to get it** — official download link
- **Lab first used** — which lab introduced it

---

## Identity & Access Management Tools

---

### Keycloak
**Category:** Identity Provider (IdP)  
**Version Used:** 26.6.2  
**Download:** https://keycloak.org/downloads  
**Lab First Used:** Lab 01

#### What it is
Keycloak is an open-source Identity and Access Management solution. It acts as a centralized Identity Provider — the system that verifies who you are and issues tokens that prove your identity to other applications.

#### Why IAM Engineers Use It
Keycloak implements the same concepts used in enterprise IAM platforms like Okta, Microsoft Entra ID, Ping Identity, and ForgeRock. Learning Keycloak deeply means understanding how ALL enterprise IdPs work at their core. It supports OAuth 2.0, OpenID Connect (OIDC), and SAML 2.0 — the three protocols that power every modern enterprise SSO system.

#### Real World Use Cases
- Centralizing authentication for all enterprise applications
- Implementing SSO so employees log in once and access everything
- Managing user identities, roles, and groups at scale
- Federating with Active Directory, Okta, or other external IdPs
- Enforcing MFA policies across the organization

#### How We Used It
- Deployed locally in development mode on Windows 11
- Created the `iam-lab` realm as an isolated identity domain
- Created users, roles, and groups
- Registered `my-test-app` as an OIDC client
- Issued real JWT access tokens via the token endpoint
- Configured public client with Direct access grants for testing

#### Key Commands
```bash
# Start Keycloak in development mode
cd /c/Users/Tsteezy/Desktop/keycloak-26.6.2/keycloak-26.6.2
bin/kc.bat start-dev

# Access Admin Console
http://localhost:8080/admin

# Token Endpoint
http://localhost:8080/realms/iam-lab/protocol/openid-connect/token
```

---

## Development & Version Control Tools

---

### Git
**Category:** Version Control  
**Version Used:** 2.54.0  
**Download:** https://git-scm.com  
**Lab First Used:** Lab 01 (Setup)

#### What it is
Git is the industry-standard version control system. It tracks every change made to files over time, allowing you to save snapshots of your work, collaborate with others, and maintain a full history of everything you've done.

#### Why IAM/Security Engineers Use It
- Tracking infrastructure and configuration changes
- Documenting lab work with a timestamped history
- Collaborating on security scripts and automation
- Maintaining IaC (Infrastructure as Code) for identity systems
- Building a public portfolio that proves real hands-on experience

#### How We Used It
- Installed Git Bash on Windows 11
- Configured global username and email
- Generated SSH keys for secure GitHub authentication
- Cloned, committed, and pushed lab documentation to GitHub
- Used `.gitignore` to prevent large video files from being pushed

#### Key Commands
```bash
git config --global user.name "Trishsteezy"
git config --global user.email "email@gmail.com"
git add .
git commit -m "description of what you did"
git push
git clone git@github.com:Trishsteezy/IAM-projects.git
```

---

### Git Bash
**Category:** Terminal / Command Line Interface  
**Version Used:** 2.54.0 (included with Git)  
**Download:** https://git-scm.com  
**Lab First Used:** Lab 01 (Setup)

#### What it is
Git Bash is a terminal emulator for Windows that provides a Unix-like command line environment. It gives Windows users access to bash commands, SSH, curl, and other tools that security and IAM engineers use daily.

#### Why IAM/Security Engineers Use It
Most enterprise IAM tools, APIs, and security scripts are designed for Unix/Linux environments. Git Bash bridges that gap on Windows, allowing you to run the same commands you would on a Linux server without needing a full Linux installation.

#### How We Used It
- Running Keycloak start commands
- Making curl requests to Keycloak token endpoints
- Managing Git repositories
- Running SSH key generation
- Navigating the file system and organizing lab files

---

### GitHub
**Category:** Code & Documentation Hosting  
**URL:** https://github.com/Trishsteezy/IAM-projects  
**Lab First Used:** Lab 01 (Setup)

#### What it is
GitHub is a web-based platform for hosting Git repositories. It makes your code and documentation publicly accessible, tracks your contributions over time, and serves as the primary portfolio platform for technical professionals.

#### Why IAM/Security Engineers Use It
Hiring managers and technical recruiters in cybersecurity and IAM go directly to GitHub before anything else. A well-maintained GitHub portfolio with real lab documentation, configs, and code is stronger evidence of skill than any certification alone.

#### How We Used It
- Created the `IAM-projects` public repository
- Set up SSH authentication for secure pushing
- Organized labs into separate folders with full documentation
- Published professional README files visible to anyone
- Maintained a `.gitignore` to keep the repo clean

#### Repository Structure
```
IAM-projects/
├── 01-keycloak-idp/
│   ├── screenshots/
│   ├── LAB-01-Keycloak-IdP-Setup-COMPLETE.md
│   └── Keycloak-SME-Guide.md
├── 02-oauth2-lab/
│   ├── screenshots/
│   ├── LAB-02-OAuth2-Token-Request.md
│   └── LAB-02-JWT-Decoded-Analysis.md
├── .gitignore
└── README.md
```

---

## API & Testing Tools

---

### curl
**Category:** HTTP Client / API Testing  
**Version Used:** Built into Git Bash  
**Lab First Used:** Lab 02

#### What it is
curl is a command-line tool for making HTTP requests. It sends requests to APIs and displays the raw responses — like a browser but for APIs and web services.

#### Why IAM/Security Engineers Use It
curl is the most widely used tool for testing OAuth 2.0 and OIDC endpoints in enterprise environments. It is available on every operating system, requires no installation beyond Git Bash on Windows, and gives complete control over every aspect of an HTTP request. Security engineers use curl to:
- Test token endpoints during IAM configuration
- Debug authentication failures
- Verify that OAuth flows work correctly
- Script automated token requests for testing
- Inspect raw API responses

#### How We Used It
- Sent POST requests to the Keycloak token endpoint
- Requested real JWT access tokens using the ROPC flow
- Diagnosed and fixed authentication errors by reading raw responses
- Learned to handle special characters in passwords using single quotes

#### Key curl Command for Token Request
```bash
curl -s -X POST http://localhost:8080/realms/iam-lab/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password" \
  -d "client_id=my-test-app" \
  -d "username=testuser" \
  -d 'password=YourPassword'
```

---

### Postman
**Category:** API Testing GUI  
**Version Used:** Latest  
**Download:** https://postman.com  
**Lab First Used:** Lab 02 (Attempted)

#### What it is
Postman is a graphical interface for making API requests and testing endpoints. It provides a visual way to build, send, and inspect HTTP requests without using a command line.

#### Why IAM/Security Engineers Use It
Postman is widely used in enterprise environments for:
- Testing OAuth 2.0 and OIDC flows visually
- Building collections of API requests for repeated testing
- Collaborating on API test suites with development teams
- Documenting API behavior with request/response examples
- Automating API tests in CI/CD pipelines

#### How We Used It
- Installed Postman desktop application
- Created a workspace for Keycloak OAuth 2.0 testing
- Encountered Cloud Agent limitations with localhost requests
- Switched to curl as a more reliable alternative for local testing

#### Issue Encountered
Postman's Cloud Agent cannot route requests to `localhost` because localhost only exists on your local machine. The Desktop Agent also had stability issues on this machine. Resolution: Used curl in Git Bash instead — a more powerful approach that is standard in enterprise IAM work.

#### When to Use Postman vs curl

| Situation | Use |
|-----------|-----|
| Visual testing and exploration | Postman |
| Remote servers and cloud endpoints | Postman |
| Local development (localhost) | curl |
| Scripting and automation | curl |
| Sharing test collections with teams | Postman |
| Quick one-off API tests | curl |

---

## Token Analysis Tools

---

### jwt.io
**Category:** JWT Token Decoder / Inspector  
**URL:** https://jwt.io  
**Lab First Used:** Lab 02

#### What it is
jwt.io is a web-based tool for decoding, inspecting, and verifying JSON Web Tokens (JWTs). Paste a token and it instantly shows the decoded header, payload, and signature in human-readable format.

#### Why IAM/Security Engineers Use It
JWTs are the currency of modern IAM — they carry identity, roles, and permissions across every enterprise system. IAM engineers use jwt.io to:
- Inspect token contents during configuration and testing
- Verify that roles and claims appear correctly after Keycloak changes
- Debug authentication failures by checking token expiry and issuer
- Validate token structure during application onboarding
- Investigate security incidents by inspecting tokens from logs

#### How We Used It
- Decoded our first real JWT token issued by Keycloak
- Inspected all claims — identity, roles, expiry, issuer, audience
- Verified that the `user-role` from Lab 01 appeared in the token
- Confirmed the token lifetime was 300 seconds (5 minutes)
- Understood the connection between Lab 01 configuration and Lab 02 token output

#### What We Found in Our Decoded Token
```json
{
  "iss": "http://localhost:8080/realms/iam-lab",
  "realm_access": {
    "roles": ["user-role", "offline_access", "uma_authorization"]
  },
  "preferred_username": "testuser",
  "email": "testuser@iam-lab.local",
  "exp": 1779741978
}
```

---

## Runtime & Infrastructure Tools

---

### Eclipse Temurin JDK (Java)
**Category:** Java Runtime Environment  
**Version Used:** 25.0.3 LTS  
**Download:** https://adoptium.net  
**Lab First Used:** Lab 01

#### What it is
Eclipse Temurin is an open-source distribution of the Java Development Kit (JDK) maintained by the Eclipse Foundation. Keycloak is a Java-based application and requires a JDK to run.

#### Why IAM Engineers Need It
Many enterprise IAM tools are Java-based — Keycloak, ForgeRock, Apache Directory Server, and others. Understanding Java runtime requirements is part of deploying and maintaining IAM infrastructure.

#### How We Used It
- Installed Temurin JDK 25 from adoptium.net
- Configured JAVA_HOME environment variable
- Verified installation with `java -version`
- Used as the runtime for Keycloak 26.6.2

#### Installation Notes
- Installed for all users on Windows 11
- Enabled Set JAVA_HOME variable during installation
- Enabled JavaSoft registry keys
- Must close and reopen Git Bash after installation for PATH to update

---

## Operating System & Platform

---

### Windows 11
**Category:** Operating System  
**Device:** Dell Latitude 5480 (12th Gen Intel i5-1245U, 16GB RAM, 477GB SSD)  
**Lab First Used:** All Labs

#### Relevance to IAM Work
Windows 11 is the dominant enterprise desktop operating system. IAM engineers working in enterprise environments must be comfortable configuring, troubleshooting, and deploying IAM tools on Windows. Key Windows IAM concepts include:
- Active Directory integration
- Windows Hello for Business (FIDO2)
- Kerberos authentication
- Group Policy for access control
- Windows Credential Manager

---

## Upcoming Tools (Planned for Future Labs)

| Tool | Category | Lab Planned | Purpose |
|------|---------|------------|---------|
| **Postman** | API Testing | Lab 03 | Token introspection and revocation |
| **Microsoft Entra ID** | Cloud IdP | Lab 04 | Enterprise Azure AD SSO |
| **Active Directory** | Directory Service | Lab 05 | LDAP/AD federation with Keycloak |
| **HashiCorp Vault** | Secrets Management | Lab 06 | Secure secret storage and token management |
| **OBS Studio** | Screen Recording | Documentation | Professional lab video recording |
| **Docker** | Containerization | Lab 08 | Running Keycloak in containers |
| **Wireshark** | Network Analysis | Lab 09 | Inspecting OAuth traffic at the packet level |
| **Splunk / Elastic** | SIEM | Lab 10 | Sending Keycloak events to a security monitoring platform |

---

## Tool Summary Table

| Tool | Category | Version | Status | Purpose |
|------|---------|---------|--------|---------|
| Keycloak | Identity Provider | 26.6.2 | ✅ Active | Core IdP for all labs |
| Git | Version Control | 2.54.0 | ✅ Active | Tracking and pushing lab work |
| Git Bash | Terminal | 2.54.0 | ✅ Active | Command line on Windows |
| GitHub | Code Hosting | — | ✅ Active | Portfolio and documentation |
| curl | HTTP Client | Built-in | ✅ Active | API and token endpoint testing |
| Postman | API GUI | Latest | ⚠️ Limited | Visual API testing (localhost issues) |
| jwt.io | Token Inspector | Web | ✅ Active | Decoding and inspecting JWTs |
| Eclipse Temurin JDK | Java Runtime | 25.0.3 | ✅ Active | Required for Keycloak |
| Windows 11 | OS | — | ✅ Active | Host operating system |

---

*Part of the [Classified Builds](https://github.com/Trishsteezy/IAM-projects) portfolio — building real IAM expertise in public.*
