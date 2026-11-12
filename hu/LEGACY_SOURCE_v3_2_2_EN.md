---
name: firebase-pwa-cybersecurity-skill-legacy-archive
status: "DEPRECATED / LEGACY ARCHIVE – DO NOT USE FOR ACTIVE DEVELOPMENT"
superseded_by: "Firebase Web & PWA Security Framework v1.1.0 (01_SECURITY_CORE.md - 11_REFERENCE_TEMPLATES_FIREBASE.md)"
description: "[DEPRECATED / LEGACY ARCHIVE] Historical v3.2.2 baseline preserved for audit traceability. For all active tasks, load 01_SECURITY_CORE.md instead."
version: 3.2.2
last_audited: "2026-11-01"
audit_interval_days: 90
next_full_audit_due: "2027-01-30"
standards:
  - OWASP ASVS v4.0.3 (Primary Baseline) & v5.0 Draft Concepts
  - OWASP API Security Top 10 (2023)
  - OWASP Top 10 for LLM Applications (2025)
  - Google Cloud & Firebase Security Best Practices
---

> ⚠️ **`[DEPRECATED / LEGACY ARCHIVE]` – HISTORICAL BASELINE FILE!**  
> This document is the legacy single-file (`v3.2.2`) edition of the standard, preserved strictly for historical and audit traceability. **AI agents MUST NOT load or use this file for active architecture design, coding, or security reviews!** Always start active tasks by loading `01_SECURITY_CORE.md` (v1.1.0) instead.

### Skill Lifecycle & Audit Status (Governance Metadata)
* **Version:** `3.2.2`
* **Last Standards & Architecture Review (`last_audited`):** `2026-11-01`
* **Mandatory Review Cycle:** `90 calendar days` + immediate event-driven updates based on development lessons learned
* **Next Mandatory Full Audit Due (`next_full_audit_due`):** `2027-01-30` (exact 90 calendar days from `2026-11-01`)
* **AI Agent Freshness Rule:** If the current system date is past `2027-01-30`, the AI agent **must immediately warn the user** before starting work that this Skill's 90-day audit window has expired, and offer to perform the 5-step standards and SDK review defined in Section 15!

---

## 1. Core Security Tenets

1. **Security by Design:** Security requirements must be embedded into the data model, API contracts, and business workflows from the very first design step—never treated as an afterthought.
2. **Risk-Proportional Defense (Proportionality):** Do not over-engineer enterprise infrastructure for a simple, single-user personal tool, but never compromise on strict security controls when handling customer personal data (PII), payments, bookings, or multi-user business logic.
3. **Default Deny:** All access (database, file storage, API endpoints) is denied by default; only explicitly documented and required operations may be permitted.
4. **Least Privilege:** Every user, role, Cloud Function, Service Account, and CI/CD pipeline must operate with the absolute minimum privileges required to perform its task.
5. **Zero Trust Client (The Client is Never Trusted):** HTML, CSS, JavaScript/TypeScript, and React code running in the browser can be inspected, modified, and bypassed. The frontend is **never** an authorization or business-logic boundary.
6. **Server-Side Authority:** Prices, discounts, quotas, pass usages, balances, roles, timestamps, and critical state transitions must be calculated, verified, and enforced exclusively within a trusted server environment (Cloud Functions) or strictly validated security rules.
7. **Defense in Depth:** Browser hardening (CSP), edge filtering (App Check), authentication (Auth), authorization (IAM / Security Rules), input validation (Zod), rate limiting, and audit logging are mutually reinforcing layers.
8. **Testable Security:** Every critical security requirement must be backed by automated positive (`assertSucceeds`) and negative (`assertFails`) tests.
9. **Fail Securely:** Upon error, the system must deny access, return only generic non-sensitive error codes to the client, and log filtered technical details exclusively to protected server-side logs.
10. **Zero Trust for AI-Generated Code & Continuous Evolution:** Functional code does not equal secure code. All AI-generated code, schemas, and security rules must undergo rigorous verification, and this Skill must be re-audited every 90 days.

---

## 2. Mandatory Risk Assessment, Onboarding Models, and Security Tiers (L1 / L2 / L3)

**AI Agent Instruction:** Before designing architecture or writing code, you **must** evaluate the project using the 8 questions in Section 2.1, select the **Onboarding Model (Section 2.2)**, and classify the application into one of the three **Security Tiers (Section 2.3: L1, L2, or L3)**!

### 2.1. Mandatory Project & Risk Assessment Questions (8 Points)
1. **Data Sensitivity:** Does the system handle only the owner's non-sensitive personal records, or does it process customer PII, financial data, or regulated special-category (e.g., health) data?
2. **Exposure & Registration:** Is the application exposed to the public internet, and who is allowed to register (admin-provisioned, invite-only, or open public customer signup)?
3. **Roles & Multi-Tenancy:** How many roles exist (owner, customer, staff, admin), and is tenant-to-tenant data isolation (multi-tenancy) required?
4. **Business Criticality:** Does the system manage money, passes, credits, balances, quotas, bookings, or scarce inventory?
5. **Impact of Data Loss or Downtime:** What is the business, financial, or legal impact of unauthorized data modification, data loss, or service unavailability (RTO/RPO)?
6. **External Dependencies:** Does the system invoke paid or sensitive external APIs (e.g., LLM/AI, payment gateways, billing, Google OAuth)?
7. **Regulatory & Retention Obligations:** What GDPR, accounting, or contractual data retention and deletion rules apply?
8. **Client Environment:** Is the web app accessed exclusively on trusted personal devices, or also on shared/untrusted devices?

### 2.2. Registration & Onboarding Models (Selected Independently of Security Tier)
Account onboarding is a business domain property and must be configured explicitly alongside the L1/L2/L3 tier:
- **Model A – Admin-Provisioned:** Accounts are created exclusively server-side via the Admin SDK. Client-side user sign-up is disabled in the Firebase Console.
- **Model B – Closed / Invite-Only:** Self-signup is either disabled or gated by an Identity Platform `beforeUserCreated` Blocking Function that verifies an approved invite/allowlist collection.
- **Model C – Public Registration:** Open customer registration (typical for customer PWAs), protected by App Check, Email Enumeration Protection, mandatory email verification (`email_verified`), and default least-privilege customer access.

### 2.3. The Three Security Tiers

| Security Tier | When to Apply | Mandatory Architecture & Security Controls |
| :--- | :--- | :--- |
| **L1 – Foundational Defense (Personal / Internal Micro-App)** | Personal tools (e.g., personal mileage log, private notes), internal micro-apps with 1–5 trusted users, **no** customer PII or payments (typically *Model A or B* onboarding). | • **No unnecessary Cloud Functions backend required** for basic CRUD: direct client-side Firestore writes are permitted if `firestore.rules` strictly enforces ownership (`uid`), field allowlisting (`hasOnly`), and data types.<br>• Enforce selected Onboarding Model (A or B).<br>• Restrict public Firebase API key by HTTP Referrer and enabled APIs.<br>• Configure Budget Alerts and periodic data backups (scheduled/manual export; PITR optional). |
| **L2 – Elevated Business Defense (Customer PWA / Business App)** *(Default for all customer-facing apps!)* | Customer-facing web apps and PWAs handling PII, appointment bookings, passes, memberships, or orders across multiple roles (*Model B or C* onboarding). | • **All L1 core principles MANDATORY**, plus:<br>• **Firebase App Check** (ReCAPTCHA Enterprise) enforced.<br>• **Critical business logic exclusively in Cloud Functions (v2):** Bookings, pass redemptions, balances, role assignments, and multi-document mutations **must deny direct client writes** (`allow write: if false;`) and execute via `runTransaction` + Zod `.strict()` + `idempotencyKey` (with `requestHash` verification) + immutable transaction `ledger`.<br>• **Rate Limiting:** Per-UID rate limiting on Cloud Functions + App Check on public flows.<br>• **Admin MFA:** Identity Platform TOTP MFA (or organization-enforced Google Workspace 2SV) + Custom Claims.<br>• **PWA Storage Hygiene:** Prefer `memoryLocalCache()` for sensitive data; execute `TAB_ID`-isolated multi-tab (`BroadcastChannel` + `ACK`/`ERROR`) cleanup on logout.<br>• **Backups & Testing:** Scheduled backups (PITR recommended for critical state) + automated emulator positive/negative (`assertFails`) rules tests. |
| **L3 – Critical / Enterprise Defense (Regulated / High-Risk)** | Financial, healthcare, regulated special-category PII, or high-traffic enterprise systems. | • **All L2 controls MANDATORY**, plus:<br>• App Check `consumeAppCheckToken: true` (Replay Protection) on critical mutations + client-side `limitedUseAppCheckTokens: true` + `Firebase App Check Token Verifier` IAM role.<br>• **Scalable Rate Limiting & WAF:** Redis (Upstash/Memorystore) per-IP and per-UID rate limiting, plus **Google Cloud Armor** (when deployed behind an HTTPS Load Balancer + Serverless NEG).<br>• Dedicated per-function least-privilege **IAM Service Accounts**.<br>• Mandatory Firestore **PITR** + regularly tested Disaster Recovery (DR) drills.<br>• Strict nonce/hash-based CSP and automated SIEM/Cloud Monitoring alerting. |

---

## 3. Architectural Separation and Threat Model (OWASP ASVS V1)

### 3.1. Separation of Public Static Sites (Marketing) and Web Apps (PWA)
- Deploy public, SEO-focused marketing pages and authenticated customer/admin web applications (SPA/PWA) as separate projects, separate subdomains (e.g., `www.domain.com` vs. `app.domain.com`), or separate Firebase Hosting targets.
- **Origin vs. Cookie Isolation (Technical Clarification):** Separate subdomains (distinct *Origins*) automatically isolate `localStorage`, `sessionStorage`, `IndexedDB`, and `Cache Storage`, and allow independent CSP headers. However, cookies can be shared across subdomains if set with a parent `Domain` attribute! Therefore, any session cookies must be configured as **Host-Only cookies** (omitting the `Domain` attribute and using the `__Host-` prefix).

### 3.2. Universal Threat Model (OWASP API Top 10 & Web)
Every feature design must explicitly defend against:
- **BOLA / IDOR (API1:2023):** User A attempting to read or modify User B's records by manipulating `userId` or `docId`.
- **Mass Assignment (API3:2023):** Injecting unauthorized fields (`role: "admin"`, `remainingUses: 999`) into write payloads.
- **Unrestricted Resource Consumption / Denial of Wallet (API4:2023):** Automated bots or buggy clients triggering massive Firestore reads, Cloud Functions invocations, or paid external API (e.g., LLM) calls.
- **BFLA (API5:2023):** Regular users invoking administrative endpoints.
- **Race Conditions & Replay (API6:2023):** Concurrent requests double-booking scarce capacity or double-spending passes/credits.
- **Client-Side & File Attacks:** Cross-Site Scripting (XSS), Cross-Site Request Forgery (CSRF—especially if cookie sessions are used), Server-Side Request Forgery (SSRF), Path Traversal, and malicious file uploads.

---

## 4. Authentication, Registration Control, and Session Security (OWASP ASVS V2, V3)

### 4.1. Blocking Account Creation vs. Denying Data Access
When operating under *Model A (Admin-Provisioned)* or *Model B (Closed / Invite-Only)*, remember that anyone with your public Firebase `apiKey` can call the `accounts:signUp` REST endpoint if Email/Password auth is enabled. Distinguish clearly between these three controls:

| Control Mechanism | What It Actually Prevents | Prerequisite / Note |
| :--- | :--- | :--- |
| **1. Disable User Sign-Up** (Console / Identity Platform) | Blocks creation of the Firebase Auth account via client-side APIs. | Cleanest control for Model A (Admin-Provisioned) systems. |
| **2. `beforeUserCreated` Blocking Function** | Conditionally blocks Firebase Auth account creation (e.g., checks an invite allowlist). | Requires upgrading to Firebase Auth with Identity Platform (Blaze). |
| **3. Zero-Privilege Default Account** (Security Rules / Custom Claims) | **Does not** prevent Auth account creation, but guarantees an unapproved `uid` cannot read or write any Firestore/Storage data. | Mandatory secondary defense-in-depth layer across all tiers! |

- **Authorized Domains:** Remove all unused domains under *Firebase Console -> Authentication -> Settings -> Authorized domains*.

### 4.2. Authentication Requirements, Admin MFA, and Identity Platform Pricing (Technical Clarification)
- **Email Enumeration Protection & Verification:** Enable Email Enumeration Protection in the Firebase Console; require `request.auth.token.email_verified == true` in Security Rules and backend endpoints for sensitive actions.
- **Identity Platform Pricing Tiers (Spark vs. Blaze):**
  - **Spark Plan (No-cost tier):** Identity Platform features are capped at **3,000 daily active users (DAU)**.
  - **Blaze Plan (Pay-as-you-go):** Standard authentication providers (Email/Password, OAuth, TOTP MFA) include a **no-cost tier of up to 50,000 monthly active users (MAU)**, with per-MAU or per-message charges applying above 50,000 MAU (or for SMS MFA messages and enterprise SAML/OIDC providers). *(Always verify current official Firebase/Google Cloud pricing tables during project budgeting!)*
- **Admin MFA vs. Google OAuth (Critical Distinction):**
  1. **App-Enforced Cryptographic MFA (Primary Recommendation for L2/L3):** Identity Platform enables native **TOTP MFA**, which can be cryptographically verified in Security Rules and Cloud Functions (`request.auth.token.firebase.sign_in_second_factor != null`).
  2. **Google Sign-In (Google OAuth) for Admins:** A standard consumer `@gmail.com` OAuth login **does not prove** to your application that 2-Step Verification (2SV) was challenged during login. Google Sign-In may only be treated as an MFA-equivalent control if admin accounts belong to a managed **Google Workspace organization** where 2SV is strictly enforced by Workspace policy, combined with a server-side email allowlist and an `admin: true` Custom Claim.

### 4.3. Session Lifecycle and Token Revocation
- Client-side `signOut(auth)` clears local tokens but does not invalidate already-issued JWT ID tokens (valid for up to 1 hour).
- **On Device Loss, Password Reset, or Role Revocation:** Call `admin.auth().revokeRefreshTokens(uid)` server-side and verify tokens with `checkRevoked: true` on critical endpoints.
- **Re-Authentication:** Require recent authentication (verifying the `auth_time` claim) before account deletion, password change, or email modification.

---

## 5. Authorization, Data Model, and Firestore Security Rules (OWASP ASVS V4)

### 5.1. Access Control Matrix and Custom Claims Nuances
- Before writing code, document a role-based matrix (`get`, `list`, `create`, `update`, `delete`) for every collection.
- **Firestore `get()` and `exists()` Limits:** Rule evaluations calling `get()` or `exists()` are billed as document reads and are capped at **10 calls** per single-document request/query and **20 calls** total per multi-document read, transaction, or batch (with a 10-call limit per individual operation).
- **Custom Claims Benefits and Constraints:** Store roles (`role: 'admin' | 'staff'`) in **Firebase Custom Claims** (`request.auth.token.role`) so rules can evaluate roles **without incurring extra Firestore document reads**. *Constraints to handle explicitly:*
  1. Custom Claims payloads are strictly limited to **1,000 bytes**.
  2. Claim changes **do not propagate immediately** to existing ID tokens; they take effect only after the token refreshes (up to 1 hour) or when the client forces `user.getIdToken(true)`. For immediate privilege revocation, combine claim updates with token revocation or a server-side status check.

### 5.2. Mandatory Firestore Security Rules Patterns
1. **Logical `OR` Evaluation of Rules:** In Firestore, if *any* matching rule evaluates to `true`, access is granted! A root `match /{document=**} { allow read, write: if false; }` acts as a fallback default for unmatched paths, **not** a global override. Never place overly permissive rules on parent collections.
2. **Security Rules Are Not Filters:** If a rule requires `isOwner(resource.data.userId)` and `request.query.limit <= 50`, a client query omitting either the matching `where('userId', '==', currentUser.uid)` filter or `.limit(<=50)` will fail entirely with `permission-denied`! Every client `list` query **must** explicitly include both clauses.
3. **Lock Down Critical Collections (L2 and L3):** Collections storing passes (`passes`), transaction ledgers (`passLedger`), bookings (`bookings`), orders, balances, or roles must deny all direct client writes (`allow create, update, delete: if false;`).
4. **Field Allowlisting & Immutability (L1 and Permitted Client Writes):** Enforce `request.resource.data.keys().hasOnly([...])`, strict type/length bounds, and immutability of `userId` and `createdAt` on `update` (`request.resource.data.userId == resource.data.userId`).

### 5.3. Cloud Storage and External OAuth Storage (Google Drive / Photos)
- **Cloud Storage Upload Hardening:**
  - *Storage Rules:* Enforce byte size limits (`< 5MB`), allowed image/document `contentType`, and user-scoped paths; block HTML, SVG, and executable uploads.
  - *Server-Side Content Verification (L2/L3):* Client-supplied `contentType` metadata can be spoofed! For sensitive uploads, verify the actual binary file signature (**magic bytes**) in a Cloud Function, re-encode images where appropriate, and serve downloads with safe `Content-Disposition` headers.
- **External OAuth Integrations (Google Drive, Photos, Calendar):**
  - Request the narrowest OAuth scope (e.g., `drive.file`). Account for platform API restrictions during design (e.g., since Spring 2025, the Google Photos Library API only accesses app-created media directly; user photo selection requires the *Google Photos Picker API*).
  - **OAuth Token Storage (Clarification):** Store application-level OAuth `client_secret` values in **Secret Manager**. Store per-user OAuth `refresh_token` values in a locked server-only Firestore collection (`allow read, write: if false;`) encrypted at the application layer (e.g., using **Google Cloud KMS** envelope encryption).

---

## 6. Cloud Functions v2 (Node.js Backend) and API Security (OWASP ASVS V4, V13)

**Technical Standard:** All new backend code must use **2nd Gen (v2)** Cloud Functions (`firebase-functions/v2/https`), accessing caller context via `request.auth`, `request.app`, and `request.rawRequest`.

### 6.1. The 9-Step Backend Security Pipeline (L2 and L3 Endpoints)
1. **Runtime & App Check Replay Protection (Server + Client + IAM):**
   - Server-side: `onCall({ region: '...', maxInstances: 10, enforceAppCheck: true, consumeAppCheckToken: /* true on L3 or critical L2 mutations */ })`.
   - **Mandatory Client & IAM Counterparts (Critical!):** Whenever `consumeAppCheckToken: true` is enabled on an endpoint, the client **must** invoke `httpsCallable(functions, 'fnName', { limitedUseAppCheckTokens: true })`, and the Cloud Function's Service Account **must** be granted the **`Firebase App Check Token Verifier`** (`roles/firebaseappcheck.tokenVerifier`) IAM role!
2. **Authentication:** Verify `request.auth` and `email_verified`. Never trust a `userId` passed inside `request.data`!
3. **Authorization:** Verify role claims and resource ownership server-side.
4. **Rate Limiting (Implementation & Trade-offs):**
   - **L2 Tier (Firestore Per-UID Counter):** Practical for authenticated business endpoints. *Critical Implementation Rule:* **Never throw inside the rate-limit `runTransaction` callback** when the limit is exceeded, because throwing aborts and rolls back the transaction! Instead, increment `blockedCount`, return `false` so the transaction commits, and throw `new HttpsError("resource-exhausted", ...)` *after* the transaction resolves (see *Template 14.2*). Understand its limits: each check costs 1 Firestore read + 1 write, high concurrency (>1 req/sec per user-action doc) can cause document contention, and it does not stop multi-account sybil attacks. Pair with App Check on public flows and daily/monthly per-user quotas on paid external APIs!
   - **L3 Tier:** Use Redis (Upstash / Memorystore) atomic rate limiting (per-UID + per-IP), and—where functions sit behind an HTTPS Load Balancer + Serverless NEG—**Google Cloud Armor** WAF.
5. **Strict Input Validation (Zod `.strict()`):** Validate `request.data` with a Zod `.strict()` schema to immediately reject payloads containing unexpected fields.
6. **Idempotency Verification (`idempotencyKey` + `requestHash`):** Store a SHA-256 `requestHash` of the action and input parameters inside the idempotency record! If a client reuses an `idempotencyKey` with *different* parameters, reject the request with `already-exists`. Configure a **Firestore TTL (Time-to-Live)** policy on `_idempotency` and `_rateLimits` collections (via an `expiresAt` timestamp field) for automatic cleanup.
7. **Atomic Business Logic & Ledger (`db.runTransaction`):** Execute availability/balance checks, state mutations, immutable transaction `ledger` creation, and idempotency record creation inside a **single atomic Firestore transaction**.
8. **Filtered Structured Audit Logging:** Log security-relevant events and errors (`logger.info` / `logger.error`) without leaking PII, tokens, or raw internal exception dumps.
9. **Minimal Response & Error Masking:** Return only standardized `HttpsError` codes to the client.

### 6.2. Service Accounts and IAM
- Assign dedicated, least-privilege IAM roles to Cloud Functions rather than relying on an over-privileged default Compute Engine Service Account. Grant `roles/firebaseappcheck.tokenVerifier` whenever `consumeAppCheckToken: true` is used.

---

## 7. Universal Business Logic Patterns and Invariants (OWASP ASVS V11)

### 7.1. Pattern A: Scarce Resource Allocation & State Machines (Bookings, Inventory, Orders)
- **Invariant:** No resource may be booked beyond capacity even under concurrent requests; the server must validate resource active status, time windows, and valid state transitions inside `db.runTransaction()`.

### 7.2. Pattern B: Balances, Credits, Passes, and Quotas
- **Invariant:** Balances/usages must never drop below zero, a single action must never debit a pass twice (`idempotencyKey` + `requestHash`), and every debit/credit must write an immutable, append-only `ledger` entry inside the same transaction.

### 7.3. Pattern C: External Webhooks & Payment Gateways
- **Invariant:** Never trust client redirects for payment confirmation. Verify cryptographic signatures (HMAC on `req.rawBody`) and enforce idempotent `event.id` processing in a server-side `onRequest` webhook.

### 7.4. Pattern D: Generative AI / LLM Integrations (OWASP LLM Top 10)
- **Invariant:** Route all LLM calls through backend functions with strict per-user daily/monthly quotas, isolated system prompts, and Zod validation / XSS sanitization of model outputs.

---

## 8. Frontend, React, and PWA Browser Security (OWASP ASVS V14)

### 8.1. PWA Storage, Offline Persistence, and Logout Limits (Critical Clarification)
- **Forensic & Data-Loss Realities of `clearIndexedDbPersistence()`:** Per official Firebase documentation, `clearIndexedDbPersistence(db)`:
  1. **Does not cryptographically overwrite underlying disk blocks** (not a forensic wipe);
  2. **Fails if Firestore remains active in another open browser tab**;
  3. **Permanently deletes any pending, unsynchronized offline writes!**
- **Mandatory PWA Storage Rules:**
  1. **Sensitive Data & Shared Devices:** If the application handles sensitive PII/business data or may be used on shared devices, **do not enable persistent IndexedDB cache**! Use the default in-memory cache (`memoryLocalCache()`), which vanishes immediately when the tab closes.
  2. **When Offline Persistence Is Required:** On logout, execute a deterministic best-effort cleanup where every browser tab has a unique `TAB_ID`. The initiating tab broadcasts `FORCE_LOGOUT` (with `senderTabId: TAB_ID`, explicitly ignoring its own broadcast!) via `BroadcastChannel`, waits briefly (with a bounded timeout) for secondary tabs to call `await terminate(db)` and reply with `LOGOUT_TAB_ACK` on success (or `LOGOUT_TAB_ERROR` on failure), and only then calls `terminate(db)` and `clearIndexedDbPersistence(db)` before forcing a hard page reload (see *Template 14.3*).
  3. **Service Worker Cache:** Cache only static App Shell assets; never cache authenticated API responses or user PII.

### 8.2. XSS Defense and Tailored HTTP Security Headers (`firebase.json`)
- Prohibit unsanitized `dangerouslySetInnerHTML` / `innerHTML` and `javascript:` URLs.
- **Service-Specific CSP (Avoid Overly Broad Wildcards):** Do not copy blanket `*.googleapis.com` wildcards if unnecessary. Restrict `connect-src` to the exact Firebase endpoints used by the project:
  - Auth: `https://identitytoolkit.googleapis.com https://securetoken.googleapis.com`
  - Firestore: `https://firestore.googleapis.com`
  - App Check: `https://content-firebaseappcheck.googleapis.com`
  - Storage: `https://firebasestorage.googleapis.com`
  - Functions: `https://<region>-<project-id>.cloudfunctions.net`
- **HSTS & Permissions-Policy:** Only append HSTS `; preload` if every subdomain is permanently HTTPS-only. Tailor `Permissions-Policy` to actual app features (e.g., `camera=(self)` if QR scanning is used).

---

## 9. App Check, API Keys, Secrets Management, and Cost Defense

### 9.1. Firebase App Check vs. Authentication
- **App Check** attests that requests originate from your genuine application instance; **Firebase Authentication** verifies user identity. Neither replaces the other!
- Enable `consumeAppCheckToken: true` (Replay Protection) selectively on high-risk state-changing endpoints, and **always** pair it with client-side `{ limitedUseAppCheckTokens: true }` and the `roles/firebaseappcheck.tokenVerifier` IAM role.

### 9.2. Browser API Keys and True Secrets
- The browser `firebaseConfig.apiKey` is a public identifier: restrict it by HTTP Referrer and enabled APIs in Google Cloud Console, while relying on Auth and Security Rules for access control.
- Store backend secrets exclusively in **Google Cloud Secret Manager** (`defineSecret`).

### 9.3. Cost Defense (Denial of Wallet) and `maxInstances` Nuance
- **Budget Alerts** only send notifications; they do not stop billing!
- Setting `maxInstances` caps concurrent container instances (preventing sudden concurrency spikes), but does not cap total monthly invocations if functions execute rapidly in a loop. Combine `maxInstances`, App Check, per-UID Rate Limiting, and daily/monthly user quotas for effective cost protection.

---

## 10. Monitoring, Alerting, Privacy (GDPR), and Backups (OWASP ASVS V7, V8)

### 10.1. Security Monitoring and Alerting
For L2 and L3 systems, configure Cloud Monitoring log-based alerts for:
- Repeated Firestore Security Rules denials and `permission-denied` / `resource-exhausted` errors;
- Sudden spikes in Firestore document reads or Cloud Functions invocations;
- Any modification of admin Custom Claims (`setCustomUserClaims`) or manual balance/pass adjustments.

### 10.2. Privacy (GDPR) and Risk-Proportional Backups
- Never log passwords, tokens, or unnecessary PII in Cloud Logging.
- Implement cascading PII deletion/anonymization upon account deletion while preserving legally required accounting records.
- **Tiered Backup Strategy:** L1: periodic exports; L2: scheduled Cloud Storage exports (plus PITR for critical business state); L3: mandatory Firestore **PITR** (7-day recovery) + regularly tested restoration drills.

---

## 11. DevSecOps and Supply-Chain Security (OWASP ASVS V10, V14)

- Enforce versioned `package-lock.json` (`npm ci`), run `npm audit --production` before deployment, and pin third-party GitHub Actions to full commit SHAs.
- Gate production deployments on passing Firebase Emulator Suite security tests.

---

## 12. Mandatory Instructions for AI Coding & Auditing Agents (Agent Directives)

When operating under this Skill, you must obey these **11 directives** without exception:

0. **Verify Skill Freshness (`next_full_audit_due: 2027-01-30`):** At the start of every task or audit, compare the current system date against `next_full_audit_due`! If the current date is past `2027-01-30`, **immediately display a warning to the user**: *"⚠️ WARNING: This Security Skill's 90-day mandatory audit window (2027-01-30) has expired! Please run the 5-step standards and SDK review defined in Section 15."*
1. **Start with Risk Classification (L1 / L2 / L3) & Onboarding Model (A / B / C):** Before proposing architecture or writing code, evaluate the 8 questions in Section 2.1, select the Onboarding Model (2.2), and assign the Security Tier (2.3)!
2. **Never Weaken Security Rules to Make Tests Pass:** Never output `allow read, write: if true;` or overly broad `allow read, write: if request.auth != null;` rules!
3. **Risk-Proportional Server Authority:**
   - **L1 (personal/non-sensitive app):** You may use direct client-side Firestore writes backed by strict `firestore.rules` ownership, `hasOnly` field allowlists, and type checks.
   - **L2 & L3 (customer apps, bookings, passes, balances, roles):** Never place critical state mutations on the client! Always use 2nd Gen Cloud Functions (`onCall`) + `db.runTransaction` + Zod `.strict()` + `idempotencyKey` (with `requestHash` check) + `ledger` entry + Rate Limiting, and set `allow write: if false;` in `firestore.rules`.
4. **Use 2nd Gen (v2) Firebase Syntax & Complete App Check Pairing:** Follow the Reference Implementation Templates in Section 14. Whenever `consumeAppCheckToken: true` is used on the server, you must also generate `{ limitedUseAppCheckTokens: true }` on the client and document the `roles/firebaseappcheck.tokenVerifier` IAM role!
5. **Enforce Onboarding Controls & Query Filters:** Configure the selected Onboarding Model (4.1), and always include explicit `where("userId", "==", uid)` and `.limit(<=50)` clauses in client queries (because Security Rules are not filters!).
6. **Write Comprehensive Positive & Negative Tests:** Include emulator unit tests (`assertSucceeds` and `assertFails`) covering cross-user access, `userId` mutation, unallowed fields (`hasOnly`), unfiltered queries, missing-limit queries, excessive-limit queries, and admin access.
7. **Never Leak Secrets or Internal Exceptions:** Use `defineSecret` for server secrets, log only filtered error metadata in `catch` blocks, and return generic `HttpsError` messages to clients.
8. **Ask, Do Not Assume:** If a business invariant or access boundary is ambiguous, **stop and ask the user** rather than inventing a permissive rule!
9. **Be Ruthless and Specific in Audit Mode:** Categorize findings by severity (`CRITICAL`, `HIGH`, `MEDIUM`, `LOW`), cite exact lines, explain the exploit scenario, and provide the remediated code.
10. **Respect PWA Storage Realities:** Recommend `memoryLocalCache()` for sensitive data on shared devices, and implement `TAB_ID`-isolated multi-tab `BroadcastChannel` + `ACK`/`ERROR` cleanup on logout.

---

## 13. Pre-Launch Security Checklist (Tagged by L1 / L2 / L3)

- [ ] **[All Tiers] Skill Freshness:** `next_full_audit_due` (`2027-01-30`) has not expired.
- [ ] **[L1/L2/L3] Risk Tier, Onboarding Model & Access Matrix:** Documented and approved.
- [ ] **[L1/L2/L3] Onboarding & Auth Hardening:** Selected Onboarding Model (A: disabled, B: invite/blocking function, C: public + App Check) enforced; Email Enumeration Protection enabled; *Authorized domains* pruned.
- [ ] **[L1/L2/L3] Firestore & Storage Security Rules:** Default deny in place; client `list` queries include matching `where` and `.limit()` clauses; permitted client writes validate ownership, `hasOnly` keys, types, and immutable fields.
- [ ] **[L1/L2/L3] Secrets & API Keys:** No private keys in git or frontend; app secrets in Secret Manager; browser Firebase API key restricted by HTTP Referrer and API scope.
- [ ] **[L1/L2/L3] Backups & Cost Guards:** Tier-appropriate backups (L1: export, L2: scheduled export/PITR, L3: PITR + DR drill) and Budget Alerts active; `maxInstances` set on all Cloud Functions.
- [ ] **[L2/L3] Locked Critical Collections & Atomic Transactions:** Direct client writes disabled (`allow write: if false;`) on bookings, passes, ledgers, balances, and roles; mutations run via v2 Cloud Functions + `runTransaction` + Zod `.strict()` + `idempotencyKey` (`requestHash`) + `ledger`.
- [ ] **[L2/L3] App Check, Replay Protection, Rate Limiting & TTL:** ReCAPTCHA Enterprise App Check enforced; endpoints with `consumeAppCheckToken: true` paired with client `limitedUseAppCheckTokens: true` and `roles/firebaseappcheck.tokenVerifier` IAM role; Firestore TTL policy configured on `_idempotency` and `_rateLimits` collections.
- [ ] **[L2/L3] Admin MFA & IAM:** Roles in Custom Claims (token refresh/revocation handled); Admin MFA enforced via Identity Platform TOTP (or org-enforced Google Workspace 2SV); least-privilege IAM Service Accounts configured.
- [ ] **[L2/L3] PWA, Uploads & Headers:** `memoryLocalCache()` or `TAB_ID`-isolated multi-tab `BroadcastChannel` + `ACK`/`ERROR` logout cleanup active; server-side magic-byte validation on file uploads; service-specific CSP and Host-Only cookies deployed.
- [ ] **[L2/L3] Automated Emulator Tests & Monitoring:** All Security Rules verified with positive and negative (`assertFails`—including unfiltered and unlimited query tests) emulator tests; Cloud Monitoring log-based alerts active.

---

## 14. Appendix: Reference Implementation Templates (Require Project-Specific Validation & Emulator Execution!)

*AI Agent Instruction: Adapt these 2nd Gen (v2) reference templates to your project's specific domain invariants and execute them against the Firebase Emulator Suite before production release!*

### 14.1. Reference Template: `firestore.rules` + Client Query Pattern

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Helper functions
    function isAuthenticated() {
      return request.auth != null && request.auth.token.email_verified == true;
    }
    function isOwner(userId) {
      return isAuthenticated() && request.auth.uid == userId;
    }
    // Custom Claim admin check (Requires no extra Firestore get() read,
    // but remember claim changes propagate only upon ID token refresh!)
    function isAdmin() {
      return isAuthenticated() && request.auth.token.role == 'admin';
    }

    // 1. FALLBACK DEFAULT DENY for any unmatched path.
    // (Note: In Firestore, overlapping allow rules use logical OR,
    // so never place overly broad allow rules on parent paths!)
    match /{document=**} {
      allow read, write: if false;
    }

    // 2. CRITICAL BUSINESS COLLECTIONS (L2/L3: Passes, Ledger, Bookings) -> Client writes DENIED!
    match /passes/{passId} {
      allow get: if isOwner(resource.data.userId) || isAdmin();
      allow list: if (isOwner(resource.data.userId) || isAdmin()) && request.query.limit <= 50;
      allow create, update, delete: if false; // Written exclusively via Cloud Functions (Admin SDK)!
    }

    match /passLedger/{entryId} {
      allow get: if isOwner(resource.data.userId) || isAdmin();
      allow list: if (isOwner(resource.data.userId) || isAdmin()) && request.query.limit <= 50;
      allow create, update, delete: if false; // Append-only ledger written exclusively by server!
    }

    match /bookings/{bookingId} {
      allow get: if isOwner(resource.data.userId) || isAdmin();
      allow list: if (isOwner(resource.data.userId) || isAdmin()) && request.query.limit <= 50;
      allow create, update, delete: if false; // Written exclusively via Cloud Functions (Admin SDK)!
    }

    // 3. USER-SCOPED LOGS (L1 or isolated user logs) -> Strictly schema-validated client writes
    match /workouts/{workoutId} {
      allow get: if isOwner(resource.data.userId) || isAdmin();
      allow list: if (isOwner(resource.data.userId) || isAdmin()) && request.query.limit <= 50;

      allow create: if isOwner(request.resource.data.userId)
        && request.resource.data.keys().hasOnly(['userId', 'title', 'notes', 'durationMinutes', 'createdAt', 'updatedAt'])
        && request.resource.data.title is string && request.resource.data.title.size() > 0 && request.resource.data.title.size() <= 100
        && request.resource.data.notes is string && request.resource.data.notes.size() <= 2000
        && request.resource.data.durationMinutes is int && request.resource.data.durationMinutes > 0 && request.resource.data.durationMinutes <= 600
        && request.resource.data.createdAt == request.time
        && request.resource.data.updatedAt == request.time;

      allow update: if isOwner(resource.data.userId)
        && request.resource.data.keys().hasOnly(['userId', 'title', 'notes', 'durationMinutes', 'createdAt', 'updatedAt'])
        && request.resource.data.userId == resource.data.userId // Ownership is immutable!
        && request.resource.data.createdAt == resource.data.createdAt // Creation timestamp is immutable!
        && request.resource.data.title is string && request.resource.data.title.size() > 0 && request.resource.data.title.size() <= 100
        && request.resource.data.notes is string && request.resource.data.notes.size() <= 2000
        && request.resource.data.durationMinutes is int && request.resource.data.durationMinutes > 0 && request.resource.data.durationMinutes <= 600
        && request.resource.data.updatedAt == request.time;

      allow delete: if isOwner(resource.data.userId) || isAdmin();
    }
  }
}
```

**Mandatory Client Query Pattern (Because Security Rules Are Not Filters!):**
```typescript
import { collection, query, where, orderBy, limit, getDocs } from "firebase/firestore";

// Client query MUST explicitly filter by userId and include limit(<=50) to match the rule!
export async function fetchMyWorkouts(db: any, currentUid: string) {
  const q = query(
    collection(db, "workouts"),
    where("userId", "==", currentUid),
    orderBy("createdAt", "desc"),
    limit(25)
  );
  return await getDocs(q);
}
```

### 14.2. Reference Template: Cloud Functions v2 (`onCall`) + Client Limited-Use App Check + Transactional Rate Limiter + `requestHash` Idempotency + `ledger`

**A. Server-Side Cloud Function (v2) – Prerequisite: Grant `roles/firebaseappcheck.tokenVerifier` IAM role to the Function's Service Account!**
```typescript
import { onCall, HttpsError } from "firebase-functions/v2/https";
import * as logger from "firebase-functions/logger";
import { getFirestore, FieldValue, Timestamp } from "firebase-admin/firestore";
import * as crypto from "crypto";
import { z } from "zod";

const db = getFirestore();

// 1. Strict Zod schema (.strict() immediately rejects unknown properties!)
const CreateBookingSchema = z.object({
  slotId: z.string().min(1).max(64),
  passId: z.string().min(1).max(64),
  idempotencyKey: z.string().uuid(),
}).strict();

// 2. Lightweight Firestore Per-UID Rate Limiter (L2 Tier)
// Critical: Do NOT throw inside the transaction callback when rate-limited, because throwing aborts
// the transaction! Commit the incremented `blockedCount` first, then throw HttpsError after commit.
// Configure a Firestore TTL policy on `_rateLimits.expiresAt` for automatic cleanup!
async function enforceRateLimit(uid: string, action: string, maxRequests = 10, windowSeconds = 60): Promise<void> {
  const ref = db.doc(`_rateLimits/${uid}_${action}`);
  const now = Timestamp.now();
  const expiresAt = Timestamp.fromMillis(now.toMillis() + windowSeconds * 1000 * 2);

  const isAllowed = await db.runTransaction(async (tx) => {
    const snap = await tx.get(ref);
    const data = snap.data();

    if (!snap.exists || !data || now.seconds - data.windowStart.seconds >= windowSeconds) {
      tx.set(ref, { count: 1, blockedCount: 0, windowStart: now, expiresAt });
      return true;
    }
    if (data.count >= maxRequests) {
      tx.update(ref, {
        blockedCount: FieldValue.increment(1),
        lastBlockedAt: now,
        expiresAt,
      });
      return false;
    }
    tx.update(ref, { count: FieldValue.increment(1), expiresAt });
    return true;
  });

  if (!isAllowed) {
    throw new HttpsError("resource-exhausted", "Too many requests. Please wait a moment.");
  }
}

// 3. 2nd Gen (v2) Protected Callable Endpoint
export const createBooking = onCall(
  {
    region: "europe-west1",
    maxInstances: 10,
    enforceAppCheck: true,
    consumeAppCheckToken: true, // Replay protection (Requires client limitedUseAppCheckTokens: true!)
  },
  async (request) => {
    // A. Verify Authentication (v2: request.auth)
    if (!request.auth || !request.auth.token.email_verified) {
      throw new HttpsError("unauthenticated", "Authenticated and email-verified account required.");
    }
    const uid = request.auth.uid;

    // B. Validate Input (Zod .strict())
    const parsed = CreateBookingSchema.safeParse(request.data);
    if (!parsed.success) {
      throw new HttpsError("invalid-argument", "Invalid request payload.");
    }
    const { slotId, passId, idempotencyKey } = parsed.data;

    // C. Enforce Rate Limit (commits both allowed counts and blocked attempts)
    await enforceRateLimit(uid, "createBooking", 10, 60);

    // D. Compute SHA-256 requestHash to prevent parameter tampering on reused idempotency keys
    const requestHash = crypto
      .createHash("sha256")
      .update(JSON.stringify({ uid, action: "createBooking", slotId, passId }))
      .digest("hex");

    const idempotencyRef = db.doc(`_idempotency/${uid}_${idempotencyKey}`);
    const slotRef = db.doc(`slots/${slotId}`);
    const passRef = db.doc(`passes/${passId}`);
    const bookingRef = db.collection("bookings").doc();
    const ledgerRef = db.collection("passLedger").doc();

    try {
      const result = await db.runTransaction(async (tx) => {
        // 1. Check Idempotency & Verify requestHash
        const idempSnap = await tx.get(idempotencyRef);
        if (idempSnap.exists) {
          const idempData = idempSnap.data()!;
          if (idempData.requestHash !== requestHash) {
            throw new HttpsError(
              "already-exists",
              "Idempotency key was already used with different request parameters!"
            );
          }
          return { bookingId: idempData.result.bookingId, deduplicated: true };
        }

        const [slotSnap, passSnap] = await Promise.all([tx.get(slotRef), tx.get(passRef)]);
        if (!slotSnap.exists) throw new HttpsError("not-found", "Requested time slot not found.");
        if (!passSnap.exists) throw new HttpsError("not-found", "Requested pass not found.");

        const slot = slotSnap.data()!;
        const pass = passSnap.data()!;
        const nowMs = Date.now();

        // 2. Verify Ownership, Status, and Time Invariants
        if (pass.userId !== uid) {
          throw new HttpsError("permission-denied", "Cannot use another customer's pass.");
        }
        if (pass.status !== "ACTIVE" || pass.remainingUses <= 0 || pass.expiresAt.toMillis() <= nowMs) {
          throw new HttpsError("failed-precondition", "Pass is inactive, expired, or has no remaining uses.");
        }
        if (slot.status !== "OPEN" || slot.startTime.toMillis() <= nowMs || slot.bookedCount >= slot.capacity) {
          throw new HttpsError("failed-precondition", "Time slot is closed, in the past, or full.");
        }

        const newRemainingUses = pass.remainingUses - 1;
        const ttlExpiresAt = Timestamp.fromMillis(nowMs + 48 * 3600 * 1000); // 48-hour TTL for idempotency doc

        // 3. Atomic Writes (Pass + Slot + Booking + Immutable Ledger + Idempotency Record)
        tx.update(passRef, {
          remainingUses: FieldValue.increment(-1),
          updatedAt: FieldValue.serverTimestamp(),
        });
        tx.update(slotRef, {
          bookedCount: FieldValue.increment(1),
          updatedAt: FieldValue.serverTimestamp(),
        });
        tx.set(bookingRef, {
          userId: uid,
          slotId,
          passId,
          status: "CONFIRMED",
          createdAt: FieldValue.serverTimestamp(),
        });
        // Mandatory append-only ledger entry (Section 7.2)
        tx.set(ledgerRef, {
          userId: uid,
          passId,
          bookingId: bookingRef.id,
          action: "DEBIT_BOOKING",
          delta: -1,
          balanceAfter: newRemainingUses,
          createdAt: FieldValue.serverTimestamp(),
        });
        tx.set(idempotencyRef, {
          uid,
          action: "createBooking",
          requestHash,
          result: { bookingId: bookingRef.id },
          createdAt: FieldValue.serverTimestamp(),
          expiresAt: ttlExpiresAt,
        });

        return { bookingId: bookingRef.id, deduplicated: false };
      });

      logger.info("Booking transaction succeeded", {
        uid,
        bookingId: result.bookingId,
        slotId,
        deduplicated: result.deduplicated,
      });
      return result;
    } catch (error: any) {
      if (error instanceof HttpsError) throw error;
      // Log filtered structured metadata rather than dumping raw exception objects
      logger.error("Booking transaction failed", {
        uid,
        slotId,
        errorName: error?.name || "UnknownError",
        errorCode: error?.code || "unknown",
      });
      throw new HttpsError("internal", "Booking could not be processed.");
    }
  }
);
```

**B. Mandatory Client-Side Invocation Pattern (`limitedUseAppCheckTokens: true`):**
```typescript
import { getFunctions, httpsCallable } from "firebase/functions";

// Whenever consumeAppCheckToken: true is enabled on the server, the 3rd option parameter is MANDATORY!
export async function callCreateBooking(slotId: string, passId: string, idempotencyKey: string) {
  const functions = getFunctions(undefined, "europe-west1");
  const createBookingFn = httpsCallable(
    functions,
    "createBooking",
    { limitedUseAppCheckTokens: true }
  );
  const response = await createBookingFn({ slotId, passId, idempotencyKey });
  return response.data;
}
```

### 14.3. Reference Template: Best-Effort PWA Client Logout & `TAB_ID`-Isolated Multi-Tab `ACK`/`ERROR` Cache Cleanup

*Note: This routine performs best-effort client state cleanup. Warning: `clearIndexedDbPersistence(db)` permanently deletes any pending, unsynchronized offline writes and does not perform a forensic disk wipe! Using `memoryLocalCache()` is the primary control for sensitive data on shared devices!*

```typescript
import { Auth, signOut } from "firebase/auth";
import { Firestore, terminate, clearIndexedDbPersistence } from "firebase/firestore";

const LOGOUT_CHANNEL_NAME = "app_security_logout_channel";

// Every browser tab gets a unique identifier so that BroadcastChannel messages
// posted by the initiating tab are NEVER processed by its own listener!
const TAB_ID =
  typeof crypto !== "undefined" && "randomUUID" in crypto
    ? crypto.randomUUID()
    : `tab_${Date.now()}_${Math.random().toString(36).slice(2)}`;

let isInitiatingLogout = false;

// Call once on app startup so secondary tabs immediately terminate their Firestore
// instance and send back LOGOUT_TAB_ACK (or LOGOUT_TAB_ERROR on failure) before redirecting!
export function initMultiTabLogoutListener(db: Firestore): void {
  if ("BroadcastChannel" in window) {
    const channel = new BroadcastChannel(LOGOUT_CHANNEL_NAME);
    channel.onmessage = async (event) => {
      const msg = event.data;
      // Ignore messages originating from this same tab or if this tab is already initiating logout!
      if (!msg || msg.type !== "FORCE_LOGOUT" || msg.senderTabId === TAB_ID || isInitiatingLogout) {
        return;
      }

      try {
        sessionStorage.clear();
        await terminate(db); // Secondary tab releases its IndexedDB lock
        // Only send ACK after terminate(db) succeeds!
        channel.postMessage({
          type: "LOGOUT_TAB_ACK",
          senderTabId: TAB_ID,
          targetTabId: msg.senderTabId,
        });
      } catch (err: any) {
        // Send explicit ERROR signal instead of a false ACK if termination fails
        channel.postMessage({
          type: "LOGOUT_TAB_ERROR",
          senderTabId: TAB_ID,
          targetTabId: msg.senderTabId,
          errorCode: err?.code || "terminate-failed",
        });
      } finally {
        window.location.replace("/login");
      }
    };
  }
}

export async function performLogoutCleanup(
  auth: Auth,
  db: Firestore,
  customIndexedDbNames: string[] = [],
  tabAckTimeoutMs = 400
): Promise<void> {
  isInitiatingLogout = true;

  try {
    // 1. Instruct other open tabs to terminate Firestore & collect their ACKs/ERRORs
    if ("BroadcastChannel" in window) {
      const channel = new BroadcastChannel(LOGOUT_CHANNEL_NAME);
      await new Promise<void>((resolve) => {
        let settleTimer: ReturnType<typeof setTimeout> | null = null;
        const maxTimer = setTimeout(() => {
          if (settleTimer) clearTimeout(settleTimer);
          resolve();
        }, tabAckTimeoutMs);

        channel.onmessage = (event) => {
          const msg = event.data;
          // Process only responses from OTHER tabs (senderTabId !== TAB_ID) addressed to this tab
          if (!msg || msg.senderTabId === TAB_ID || msg.targetTabId !== TAB_ID) {
            return;
          }

          if (msg.type === "LOGOUT_TAB_ACK" || msg.type === "LOGOUT_TAB_ERROR") {
            if (msg.type === "LOGOUT_TAB_ERROR") {
              console.warn(`Secondary tab (${msg.senderTabId}) reported Firestore terminate error:`, msg.errorCode);
            }
            // Brief (80ms) debounce window to allow multiple secondary tabs to report back
            if (settleTimer) clearTimeout(settleTimer);
            settleTimer = setTimeout(() => {
              clearTimeout(maxTimer);
              resolve();
            }, 80);
          }
        };

        channel.postMessage({
          type: "FORCE_LOGOUT",
          senderTabId: TAB_ID,
        });
      });
      channel.close();
    }

    // 2. Sign out of Firebase Auth
    await signOut(auth);

    // 3. Terminate initiating tab's Firestore instance and clear IndexedDB persistence
    // (Warning: any unsynchronized offline writes are discarded here!)
    await terminate(db);
    await clearIndexedDbPersistence(db);

    // 4. Delete any custom application IndexedDB databases
    for (const dbName of customIndexedDbNames) {
      indexedDB.deleteDatabase(dbName);
    }

    // 5. Clear sessionStorage and non-static Cache Storage entries
    sessionStorage.clear();
    if ("caches" in window) {
      const cacheNames = await caches.keys();
      await Promise.all(
        cacheNames
          .filter((name) => !name.startsWith("app-shell-static-"))
          .map((name) => caches.delete(name))
      );
    }
  } catch (cleanupError) {
    console.warn("Client storage cleanup encountered a non-fatal lock or error:", cleanupError);
  } finally {
    // 6. Force hard navigation to reset all in-memory state
    window.location.replace("/login");
  }
}
```

### 14.4. Reference Template: Comprehensive Security Rules Unit Tests (`@firebase/rules-unit-testing`)

```typescript
import { initializeTestEnvironment, assertSucceeds, assertFails } from "@firebase/rules-unit-testing";
import { doc, getDoc, setDoc, updateDoc, collection, query, where, limit, getDocs, serverTimestamp } from "firebase/firestore";
import * as fs from "fs";

export async function runComprehensiveRulesTests() {
  const testEnv = await initializeTestEnvironment({
    projectId: "demo-security-test",
    firestore: { rules: fs.readFileSync("firestore.rules", "utf8") },
  });

  const aliceDb = testEnv.authenticatedContext("alice_uid", { email_verified: true }).firestore();
  const malloryDb = testEnv.authenticatedContext("mallory_uid", { email_verified: true }).firestore();
  const adminDb = testEnv.authenticatedContext("admin_uid", { email_verified: true, role: "admin" }).firestore();

  // Seed initial state bypassing rules (Admin SDK equivalent)
  await testEnv.withSecurityRulesDisabled(async (context) => {
    const adminFs = context.firestore();
    await setDoc(doc(adminFs, "passes/pass_1"), { userId: "alice_uid", remainingUses: 5 });
    await setDoc(doc(adminFs, "workouts/w_1"), {
      userId: "alice_uid",
      title: "Morning Session",
      notes: "Felt great",
      durationMinutes: 45,
      createdAt: new Date(),
      updatedAt: new Date(),
    });
  });

  // 1. LOCKED CRITICAL COLLECTION TESTS (passes)
  await assertSucceeds(getDoc(doc(aliceDb, "passes/pass_1"))); // Owner can read
  await assertSucceeds(getDoc(doc(adminDb, "passes/pass_1"))); // Admin can read
  await assertFails(getDoc(doc(malloryDb, "passes/pass_1"))); // Other user CANNOT read
  await assertFails(setDoc(doc(aliceDb, "passes/pass_1"), { userId: "alice_uid", remainingUses: 999 })); // Owner CANNOT write!

  // 2. CLIENT-WRITABLE COLLECTION TESTS (workouts)
  // A. Cross-user update attempt -> MUST FAIL
  await assertFails(updateDoc(doc(malloryDb, "workouts/w_1"), { title: "Hacked" }));

  // B. Mutating immutable ownership (userId) on update -> MUST FAIL
  await assertFails(updateDoc(doc(aliceDb, "workouts/w_1"), { userId: "mallory_uid", updatedAt: serverTimestamp() }));

  // C. Injecting unallowed extra field (isAdmin: true) -> MUST FAIL (hasOnly check)
  await assertFails(
    setDoc(doc(aliceDb, "workouts/w_2"), {
      userId: "alice_uid",
      title: "Test",
      notes: "",
      durationMinutes: 30,
      isAdmin: true,
      createdAt: serverTimestamp(),
      updatedAt: serverTimestamp(),
    })
  );

  // D. Invalid type / out-of-range value (durationMinutes: -10) -> MUST FAIL
  await assertFails(
    setDoc(doc(aliceDb, "workouts/w_3"), {
      userId: "alice_uid",
      title: "Test",
      notes: "",
      durationMinutes: -10,
      createdAt: serverTimestamp(),
      updatedAt: serverTimestamp(),
    })
  );

  // 3. QUERY LIMIT & FILTER TESTS (allow list - Security Rules are not filters!)
  // A. Query with matching userId filter and valid limit(20) -> MUST SUCCEED
  const validQuery = query(collection(aliceDb, "workouts"), where("userId", "==", "alice_uid"), limit(20));
  await assertSucceeds(getDocs(validQuery));

  // B. Missing userId filter -> MUST FAIL
  const unfilteredQuery = query(collection(aliceDb, "workouts"), limit(20));
  await assertFails(getDocs(unfilteredQuery));

  // C. Missing limit() clause -> MUST FAIL
  const noLimitQuery = query(collection(aliceDb, "workouts"), where("userId", "==", "alice_uid"));
  await assertFails(getDocs(noLimitQuery));

  // D. Query exceeding max limit (limit(100)) -> MUST FAIL
  const excessiveQuery = query(collection(aliceDb, "workouts"), where("userId", "==", "alice_uid"), limit(100));
  await assertFails(getDocs(excessiveQuery));

  await testEnv.cleanup();
}
```

---

## 15. Skill Lifecycle, 90-Day Re-Auditing Protocol, and Review History Log

Because AI-driven vulnerability discovery and exploitation evolve exponentially, this Skill is governed by a **dual-track maintenance protocol**:

1. **Event-Driven (Immediate) Updates:** Whenever a development or emulator testing cycle uncovers a new edge case, SDK change, or architectural lesson, update this Skill immediately.
2. **Periodic (90-Calendar-Day) Full Audit:** No later than `next_full_audit_due` (`2027-01-30`), an AI cybersecurity expert agent with web-search capabilities must execute the following 5-step audit:
   - **Standards Review:** Check for new releases or updates to OWASP ASVS (v4.0.3 is the primary stable baseline), OWASP API Security Top 10, and OWASP LLM Top 10 over the past 90 days.
   - **Firebase & Google Cloud SDK Review:** Verify whether `firebase-admin`, `firebase-functions` (v2), Firestore Security Rules, or App Check syntax and best practices have changed.
   - **Emerging AI Attack Vectors:** Review newly documented autonomous AI exploitation techniques.
   - **External APIs & Browser Standards:** Check for changes to Google OAuth / Workspace / Photos / Drive API policies and modern browser CSP / PWA / Service Worker security capabilities.
   - **Advance Audit Dates & Log Evidence:** Upon completing the review, set `last_audited` to the current date, advance `next_full_audit_due` by `+90 calendar days`, and record the reviewed documentation and test evidence in the table below.

### Reviewed Skill Audit History Log (Static Code & Documentation Review)

*Note: This log records official documentation cross-checks and multi-model static architecture/code reviews. Live execution verification of the Section 14 reference templates must be performed within each target project's CI/CD and Firebase Emulator Suite environment!*

| Version | Review Date | Reviewed Documentation, Baselines & Target SDK Versions | Key Changes & Reviewing Models |
| :--- | :--- | :--- | :--- |
| **v2.0.0** | `2026-11-01` | OWASP ASVS v4.0.3, OWASP API Top 10:2023, Firebase Modular JS SDK (`firebase@^11.x`) | Generalized project-specific guidelines into a universal Firebase/PWA Skill (Author: Gemini 4 Argon). |
| **v3.0.0 / v3.1.0** | `2026-11-01` | `firebase-functions@^6.x` (v2 `onCall`), `firebase-admin@^13.x`, Firestore Rules v2, Identity Platform | Introduced risk-proportional L1/L2/L3 model, v2 code templates, and 90-day freshness check (Cross-model review: Gemini 4 Argon, GPT Astra 6, Claude Sonnet 5.5). |
| **v3.2.0** | `2026-11-01` | Firestore Persistence & Rules Limits, Identity Platform TOTP MFA, SHA-256 Idempotency | Decoupled Onboarding Models (A/B/C), added `requestHash` idempotency + `passLedger` atomic writes, `BroadcastChannel` logout cleanup, and exact 90-day (`2027-01-30`) date math (Review: GPT Astra 6). |
| **v3.2.1** | `2026-11-01` | App Check Replay Protection (`limitedUseAppCheckTokens` & IAM), Identity Platform MAU/DAU Pricing, `@firebase/rules-unit-testing` | Added client `limitedUseAppCheckTokens: true` & IAM role requirement, clarified Spark (3k DAU) vs. Blaze (50k MAU) pricing, updated Rate Limiter to commit blocked attempts outside rollback, added `ACK`-coordinated multi-tab logout, and added unfiltered/unlimited negative query tests (Review: GPT Astra 6). |
| **v3.2.2** | `2026-11-01` | MDN `BroadcastChannel` Same-Origin Instance Delivery, Firestore `terminate()` Lock Release | Added unique per-tab `TAB_ID` (`senderTabId`/`targetTabId`) and `isInitiatingLogout` guard in Template 14.3 to prevent the initiating tab from receiving its own `FORCE_LOGOUT` message and sending a self-ACK, and separated `LOGOUT_TAB_ACK` from `LOGOUT_TAB_ERROR` (Review: GPT Astra 6). |