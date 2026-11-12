---
name: firebase-architecture-security
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
platform: Firebase / Google Cloud
---

# 02 – Firebase Architecture & Security

## 1. Scope and Foundation (`FIREBASE-SCOPE-001`)

This module is the enhanced `v1.1.0` edition of the earlier Universal Firebase & Web/PWA Cybersecurity Skill v3.2.2 (`LEGACY_SOURCE_v3_2_2_EN.md`) and the v1.0.0 Firebase module. It retains the risk-proportional L1/L2/L3 model, Onboarding Models A/B/C, and all Firestore Rules, App Check, Cloud Functions v2, idempotency, ledger, PWA storage, and auditing controls, while adding the 5-category limit and quota governance (`FIREBASE-LIMIT-001`), the strict separation of rate-limit and business transactions (`FIREBASE-FN-003`, `FIREBASE-FN-004`), and links to the verified reference templates (`11_REFERENCE_TEMPLATES_FIREBASE.md`).

**Mandatory (`CORE-LOAD-001`):** `01_SECURITY_CORE.md` + `02_FIREBASE_SECURITY.md` are mandatory in every Firebase project. A project applies the sections corresponding to the components it actually uses, while documenting unused components in `SECURITY_PROFILE.md`. When writing concrete implementations (`firestore.rules`, Cloud Functions v2, PWA logout, emulator tests), loading the Tier-2 `11_REFERENCE_TEMPLATES_FIREBASE.md` module is also mandatory.

---

## 2. Firebase Trust Boundaries and Responsibilities (`FIREBASE-BOUND-001`)

- The browser `firebaseConfig.apiKey` is not a secret; it is a public project/configuration identifier. Restrict it in Google Cloud Console by allowed Google Cloud APIs and HTTP Referrers, but never treat it as an authorization boundary.
- Firebase Authentication proves user identity; Firebase App Check attests that requests originate from a registered client app instance; neither replaces Firestore/Storage Security Rules, server-side authorization, or IAM.
- The Firebase Admin SDK and server-side Google Cloud client libraries **do not** authorize via Firestore Security Rules; server authentication occurs via IAM/ADC and completely bypasses Security Rules. Therefore, explicit in-code authorization/ownership checks and dedicated handler-level negative authorization tests (`REF-TEST-ADMIN-001`) are mandatory in backend code.
- Client-supplied values for `uid`, `role`, `price`, `balance`, `remainingUses`, `isAdmin`, `createdAt`, or business status must never be trusted.
- Use separate Firebase projects for distinct environments (at least dev/staging vs. production separation where project size and risk warrant); never copy production customer data into development environments.
- Deploy public static marketing sites and authenticated web applications (SPA/PWA) on separate projects, separate subdomains, or separate Firebase Hosting targets. Because cookies scoped to a parent domain are shared across all subdomains, any session cookies must be configured strictly as **Host-Only cookies** (omitting the `Domain` attribute, using the `__Host-` prefix, `Secure`, `HttpOnly`, and appropriate `SameSite`).

---

## 3. Risk Classification and Onboarding (`FIREBASE-TIER-001`)

Applying the L1/L2/L3 tiers and Onboarding Models A/B/C defined in Core Section 4 (`CORE-TIER-001`, `CORE-TIER-L1-001`, `CORE-ONBOARD-001`) is mandatory. Firebase-specific minimums:

### L1 (`FIREBASE-TIER-001`)
- Strict Firestore Rules (`REF-RULES-BASE-001`): ownership check (`isOwner`), field allowlisting (`hasOnly`), data types, length and range bounds, and immutable fields (`userId`, `createdAt`).
- Direct client-side CRUD permitted only on non-critical, low-risk personal records without forcing unnecessary Cloud Functions backend infrastructure.
- Budget alerts, periodic data export/backup, and API-key restrictions (HTTP Referrer + API scope).
- Deny-by-default across all database and Storage paths.
- Mandatory positive and negative (`assertFails`) emulator Rules unit tests (`REF-TEST-RULES-001`).

### L2 (`FIREBASE-TIER-001`)
- Default tier for all applications handling customer PII or business workflows (bookings, passes, balances, memberships, orders).
- All L1 controls are mandatory.
- App Check enforced (ReCAPTCHA Enterprise) across supported client services following staged monitoring/rollout.
- Critical state mutations executed exclusively via Cloud Functions v2 (`REF-FN-CALLABLE-001`) using atomic Firestore transactions (`db.runTransaction`).
- Direct client writes to critical collections (`passes`, `passLedger`, `bookings`, balances, roles) are strictly prohibited (`allow create, update, delete: if false;`).
- Server-side schema validation (Zod `.strict()` or equivalent), explicit authorization, standalone transactional rate limiting (`FIREBASE-FN-003`), SHA-256 `requestHash`-verified idempotency, and append-only `ledger` (`FIREBASE-FN-004`).
- Admin MFA and Custom Claims for administrative access.
- Server-side structured audit logging, log-based alerting, scheduled backups (PITR recommended for critical state), and restore testing.

### L3 (`FIREBASE-TIER-001`)
- All L2 controls are mandatory.
- App Check Replay Protection (`consumeAppCheckToken: true`) on high-risk state-mutating callable endpoints where supported by the SDK and region.
- Whenever `consumeAppCheckToken: true` is used, the client MUST invoke the callable with `{ limitedUseAppCheckTokens: true }`, and the executing service account MUST hold the `Firebase App Check Token Verifier` (`roles/firebaseappcheck.tokenVerifier`) IAM role (`FIREBASE-FN-002`).
- Scalable per-UID and per-IP rate limiting (e.g., Redis / Upstash / Memorystore); WAF / Google Cloud Armor when deployed behind an HTTPS Load Balancer + Serverless NEG architecture.
- Dedicated per-function least-privilege IAM service accounts, mandatory Firestore PITR (7-day recovery), regular DR/restore drills, strict nonce/hash-based CSP, SIEM/Cloud Monitoring alerts, and independent security assessment.

---

## 4. Five-Category Limit, Quota, and Rule Governance (`FIREBASE-LIMIT-001`)

To prevent model hallucinations and inaccurate limit claims, every technical constraint and quota MUST be classified into one of the following **5 normative categories**:

1. **`[PLATFORM HARD LIMIT]`** – A technical platform limit documented and enforced by the provider (Firebase / Google Cloud).
2. **`[PLAN QUOTA]`** – An operational quota tied to a specific billing plan (e.g., Spark) or service tier.
3. **`[PRICING / FREE TIER ALLOWANCE]`** – A no-cost usage allowance or pricing threshold (which triggers billing when exceeded and may change per vendor pricing tables).
4. **`[FRAMEWORK DEFAULT]`** – A recommended default defined by this security framework (not a platform hard limit).
5. **`[PROJECT-SPECIFIC LIMIT]`** – A business or capacity limit specific to the target application (`SECURITY_PROFILE.md`).

| Limit / Quota Name | Category | Value and Exact Rule | Official Source & Verification Status (`last_verified`) |
| :--- | :--- | :--- | :--- |
| **Custom Claims Size** | `[PLATFORM HARD LIMIT]` | Max. **1,000 bytes** payload per user. | Firebase Auth Custom Claims Docs (`https://firebase.google.com/docs/auth/admin/custom-claims`) – Verified: `2026-11-02` |
| **Firestore Rules `get()` / `exists()`** | `[PLATFORM HARD LIMIT]` | Max. **10 calls** per single-document request/query; max. **20 calls** total per multi-doc read, transaction, or batch (with max. 10 per individual operation). Each call is billed as a read. | Firestore Rules Structure & Limits (`https://firebase.google.com/docs/firestore/security/rules-structure`) – Verified: `2026-11-02` |
| **Identity Platform Spark DAU** | `[PLAN QUOTA]` | **3,000 DAU** daily active user cap on the Spark (no-cost) plan for affected Identity Platform features. | Firebase Pricing (`https://firebase.google.com/pricing`) – Verified: `2026-11-02` *(Verify current pricing table before release!)* |
| **Identity Platform Blaze MAU** | `[PRICING / FREE TIER ALLOWANCE]` | **50,000 MAU** no-cost tier for standard providers and TOTP MFA; differentiated pricing across provider categories (e.g., SMS MFA per-message charges and SAML/OIDC billed separately). | Firebase Pricing (`https://firebase.google.com/pricing`) – Verified: `2026-11-02` *(Pricing terms may change!)* |
| **Firestore `list` query `.limit(50)`** | `[FRAMEWORK DEFAULT]` | `request.query.limit <= 50` in Rules and `.limit(<=50)` on the client. Not a platform limit and not a standalone cost-protection guarantee; adjustable per project (`[PROJECT-SPECIFIC LIMIT]`). | Internal Framework Default (`02_FIREBASE_SECURITY.md`, `FIREBASE-RULES-003`) |

---

## 5. Authentication and Account Lifecycle (`FIREBASE-AUTH-001`)

- Select Onboarding Model A, B, or C (`CORE-ONBOARD-001`) and configure it in the Firebase Console / Identity Platform settings.
- **Blocking Account Creation vs. Denying Data Access (`FIREBASE-AUTH-001`):** Whenever Email/Password auth is enabled, anyone possessing your public Firebase `apiKey` can invoke the `accounts:signUp` REST endpoint. Distinguish clearly among these three controls:

| Control Mechanism | What It Actually Prevents | Prerequisite / Note |
| :--- | :--- | :--- |
| **1. Disable User Sign-Up** (Console / Identity Platform) | Blocks creation of the Firebase Auth account via client-side APIs. | Cleanest control for Model A (Admin-provisioned) systems. |
| **2. `beforeUserCreated` Blocking Function** | Conditionally blocks Firebase Auth account creation server-side (e.g., checks an invite allowlist). | Requires upgrading to Firebase Auth with Identity Platform (Blaze). |
| **3. Zero-Privilege Default Account** (Security Rules / Custom Claims) | **Does not** prevent Auth account creation, but guarantees an unapproved `uid` cannot read or write any Firestore/Storage data. | Mandatory secondary (Defense in Depth) layer across all tiers (L1/L2/L3)! |

- **Authorized Domains:** Remove all unused domains under *Firebase Console -> Authentication -> Settings -> Authorized domains*.
- In Invite-only (Model B) systems, invitations must be single-use or strictly bounded, expiring, bound to the target user/tenant, and verified server-side.
- In Public signup (Model C) systems, email verification (`request.auth.token.email_verified == true`), Email Enumeration Protection, abuse protection, App Check, and zero-privilege default access are mandatory.
- **Custom Claims and Token Revocation:** Roles (`role: 'admin' | 'staff'`) may be stored in Custom Claims (max. 1,000 bytes – `[PLATFORM HARD LIMIT]`) so Rules can evaluate roles without extra Firestore `get()` reads. Because claim updates take effect only when the ID token refreshes (up to 1 hour, or when the client forces `user.getIdToken(true)`), privilege revocation, device loss, or password reset requires calling `admin.auth().revokeRefreshTokens(uid)` server-side combined with `checkRevoked: true` or a server-side/Rules active status check.
- **Admin MFA vs. Google Sign-In:** A standard consumer (`@gmail.com`) Google OAuth login **does not prove** to the relying application that 2-Step Verification (2SV/MFA) was challenged. Google Sign-In may only be accepted as an MFA-equivalent control for admins if admin accounts belong to a managed **Google Workspace organization** where 2SV is strictly enforced by organizational policy, combined with a server-side email allowlist and an `admin: true` Custom Claim. Otherwise, native Identity Platform TOTP MFA (`request.auth.token.firebase.sign_in_second_factor != null`) is mandatory.

---

## 6. Firestore Security Rules (`FIREBASE-RULES-001` .. `FIREBASE-RULES-003`, `FIREBASE-ADMIN-001`)

### 6.1. Mandatory Rules Principles (`FIREBASE-RULES-001`)
- All Rules files MUST specify `rules_version = '2';`.
- Default deny is mandatory (`match /{document=**} { allow read, write: if false; }`).
- **Logical `OR` Evaluation:** Overlapping `allow` rules evaluate with logical `OR`: if *any* matching rule evaluates to `true`, access is granted. A root catch-all deny serves only as a fallback for unmatched paths, **not** a global deny override. Never place overly permissive rules on parent collections.
- `get()` and `exists()` calls incur document read billing and strict platform limits (10 / 20 calls – `FIREBASE-LIMIT-001`).
- Do not rely solely on document path hierarchy if your query model requires explicit owner/tenant (`userId` / `tenantId`) fields.
- `hasOnly` alone is insufficient: enforce strict type, length, and numeric range bounds on every client-writable field, plus immutability of ownership (`userId`) and creation (`createdAt`) fields on `update` (`request.resource.data.userId == resource.data.userId`) (`REF-RULES-BASE-001`).

### 6.2. Locking Critical Collections (`FIREBASE-RULES-002`)
In L2 and L3 systems, direct client-side writes to bookings (`bookings`), capacity slots (`slots`), payment states, passes (`passes`), credits, balances, transaction ledgers (`passLedger`), roles, and administrative state are **strictly prohibited** (`allow create, update, delete: if false;`). Clients may only submit requests to the Cloud Functions backend, which re-validates all preconditions inside an atomic transaction.

### 6.3. Queries and Clarification of the `.limit(50)` Rule (`FIREBASE-RULES-003`)
- **Security Rules Are Not Filters:** If a rule requires `isOwner(resource.data.userId)` and `request.query.limit <= 50`, every client `list` query must explicitly include both `where('userId', '==', currentUid)` and `.limit(<=50)`, or the entire query fails with `permission-denied`.
- **Normative Clarification on `.limit(50)` (`FIREBASE-RULES-003`):** Client-side `.limit(50)` is **not a security control** in itself because client queries can be modified by an attacker. Firestore Security Rules is not a general query-limiting engine; rather, it enforces access conditions (`request.query.limit <= 50`) on the data reachable by a query. `limit <= 50` is a `[FRAMEWORK DEFAULT]` recommendation (`FIREBASE-LIMIT-001`) that **MUST NOT** be treated as a standalone authorization or cost-protection guarantee; expensive or sensitive queries must also be protected via server-side restrictions, quotas, or rate limits.

### 6.4. Protecting Against Admin SDK Bypass (`FIREBASE-ADMIN-001`)
Firestore and Storage Security Rules **do not protect** server-side operations executed via the Firebase Admin SDK. Backend code MUST:
- authenticate the caller (`request.auth` and `email_verified`);
- verify role claims and resource ownership (`pass.userId === request.auth.uid`);
- validate the entire input payload against a strict schema (`zod` `.strict()`);
- read and write only the minimum required documents;
- minimize the service account's IAM permissions;
- directly test server-side handler authorization logic using negative unit/emulator tests (`REF-TEST-ADMIN-001`).

---

## 7. Cloud Functions v2 and Separation of Two Transactions (`FIREBASE-FN-001` .. `FIREBASE-FN-004`)

### 7.1. General Cloud Functions v2 Requirements (`FIREBASE-FN-001`)
- Use supported 2nd gen (`firebase-functions/v2`) APIs (`request.auth`, `request.app`, `request.data`, `request.rawRequest`) for all new backend code, and always verify installed SDK versions against the project's `package-lock.json`.
- Choose consciously between `onCall` and `onRequest`: use `onCall` for Firebase SDK client calls, and `onRequest` for external/payment webhooks (verifying HMAC signatures against `req.rawBody`).
- Configure explicit `region`, timeout, memory/CPU bounds, and `maxInstances` on every function. `maxInstances` caps concurrent container instances, not monthly invocation billing; Budget Alerts send notifications but do not automatically stop billing.
- Validate all incoming payloads with a strict schema (e.g., Zod `.strict()`) that immediately rejects unknown/extra fields (Mass Assignment defense).

### 7.2. Risk-Based Application of App Check and Replay Protection (`FIREBASE-FN-002`)
- `enforceAppCheck: true` is mandatory by default on all supported callable endpoints at L2 and L3 tiers.
- `consumeAppCheckToken: true` (Replay Protection) is **not automatically mandatory on every callable endpoint**, because it introduces an extra verification round-trip and latency. Apply it on a risk basis to **L3 endpoints and particularly sensitive/costly L2 state-mutating endpoints**.
- Whenever `consumeAppCheckToken: true` is enabled on an endpoint, you MUST:
  1. pass `{ limitedUseAppCheckTokens: true }` in the client-side `httpsCallable` invocation;
  2. grant the `Firebase App Check Token Verifier` (`roles/firebaseappcheck.tokenVerifier`) IAM role to the executing Cloud Function Service Account.

### 7.3. Rate Limiter Transaction and Its Limitations (`FIREBASE-FN-003`)
- Execute `enforceRateLimit` in a **standalone transaction prior to** the business (e.g., booking) transaction (`REF-FN-CALLABLE-001`).
- **Critical Implementation Rule:** Never throw an exception (`throw new HttpsError(...)`) inside the rate-limit `db.runTransaction` callback when the limit is exceeded, because throwing aborts and rolls back the transaction! Instead, increment `blockedCount`, update `expiresAt` (TTL), return `false` so the transaction commits, and throw `HttpsError("resource-exhausted", ...)` *after* the transaction resolves.
- **Known Limitation:** A Firestore-based per-UID counter is suitable for low-to-moderate traffic, but under high concurrency (>1 req/sec per user-action document) it causes document contention and transaction read/write costs, and does not stop multi-account (sybil) attacks. On public or high-volume (L3) endpoints, combine with App Check, user quotas, and Redis/Cloud Armor rate limiting.

### 7.4. Atomic Business Transaction and Distinct Transaction Responsibilities (`FIREBASE-FN-004`)
- Execute all business state mutations—checking and incrementing slot capacity (`slots`), verifying and deducting pass usage (`passes`), creating the booking (`bookings`), writing the append-only transaction ledger entry (`passLedger`), and recording the SHA-256 `requestHash`-protected idempotency record (`_idempotency`)—inside a **single atomic `db.runTransaction` operation** (`REF-FN-CALLABLE-001`).
- **Distinct Transaction Responsibilities:** The responsibilities of the rate limiter transaction (`FIREBASE-FN-003`) and the business transaction (`FIREBASE-FN-004`) are strictly separated. The rate limiter state (consumed request quota) intentionally **does not roll back** if the subsequent business transaction fails due to a business precondition error (e.g., full slot or expired pass).
- Because Firestore transaction callbacks may retry on contention, **never** perform external API calls, email sending, or non-idempotent side effects inside the transaction callback.

---

## 8. Idempotency and Append-Only Ledger (`FIREBASE-IDEMP-001`)

For critical state mutations (`REF-FN-CALLABLE-001`):
- `idempotencyKey` must have sufficient entropy (e.g., UUID) and be scoped to the caller and action (`_idempotency/${uid}_${idempotencyKey}`).
- Store `uid`, `action`, `requestHash` (SHA-256 digest of the action and validated input parameters), `result`, `createdAt`, and `expiresAt`.
- On a retry with the same `idempotencyKey` + **identical `requestHash`**, return the stored result (`deduplicated: true`) without repeating the mutation.
- On a retry with the same `idempotencyKey` + **different `requestHash`** (parameter tampering), reject the request (`already-exists` / `failed-precondition`).
- The `passLedger` / transaction log must be **append-only**, closed to client writes (`allow create, update, delete: if false;`), with `delta` and `balanceAfter` calculated by the server inside the same atomic transaction.
- **Firestore TTL Rule:** Automatic deletion policies (TTL on `expiresAt`) may only be configured on technical `_idempotency` and `_rateLimits` collections; configuring automatic TTL deletion on legally, financially, or operationally required `ledger` collections is **strictly prohibited (`MUST NOT`)**!
- A SHA-256 `requestHash` does not replace authorization, input canonicalization, or proper key scoping.

---

## 9. App Check and API-Key Protection (`FIREBASE-APPCHK-001`)

- App Check is an application-origin attestation layer, not user authentication and not authorization.
- Monitor valid and unverified traffic metrics before enabling enforcement.
- Restrict the public browser Firebase `apiKey` by HTTP Referrer and enabled API scope in Google Cloud Console, but never treat it as a secret or access control mechanism.

---

## 10. Cloud Storage and External OAuth Tokens (`FIREBASE-STORAGE-001`)

- Scope Storage paths by user or tenant; enforce authentication, authorization, maximum byte size (e.g., `< 5 MB`), and allowed `contentType` in `storage.rules`.
- Client-supplied MIME types (`contentType`) can be spoofed. For sensitive uploads in L2/L3 systems, enforce server-side **magic-byte (binary signature)** verification, malware scanning where appropriate, image re-encoding, and safe `Content-Disposition` headers.
- Block uploads of SVG, HTML, scripts, and executable content, or never serve them as active content from the same origin.
- Treat Firebase Storage download URLs/tokens as bearer access credentials.
- **Storing External OAuth Tokens (e.g., Google Drive, Photos, Calendar):** Request the narrowest OAuth scope. Store application-level `client_secret` values in Secret Manager. Store per-user OAuth `refresh_token` values in a server-only Firestore collection (`allow read, write: if false;`) encrypted at the application layer (e.g., using Google Cloud KMS envelope encryption).

---

## 11. PWA, Firestore Persistence, and Logout Limitations (`FIREBASE-PWA-001`)

When designing and documenting PWA client-side storage and logout flows (`REF-PWA-LOGOUT-001`), the following **5 normative rules and limitations** MUST be enforced:

1. **Invalidating Session State:** Upon logout, application-level state belonging to the user session (`sessionStorage`, in-memory state, non-static Cache Storage) must be invalidated across all reachable browser tabs.
2. **Realities of `clearIndexedDbPersistence()`:** Persistently stored sensitive data requires explicit handling; `clearIndexedDbPersistence(db)` **is not a forensic disk wipe**, fails if Firestore remains active in another open tab, and **permanently discards any pending, unsynchronized offline writes**.
3. **Role of `memoryLocalCache()`:** For sensitive personal/business data or shared devices, default to in-memory caching (`memoryLocalCache()`) instead of persistent IndexedDB. While `memoryLocalCache()` reduces persistent Firestore cache exposure on shared devices, it **does not guarantee complete data erasure** and does not replace application-level privacy controls.
4. **Previously Displayed Data Limitation:** Logout cannot guarantee the revocation or complete physical removal of data that was already displayed or accessed in the browser.
5. **Best-Effort Multi-Tab Coordination (`BroadcastChannel` + `TAB_ID`):** Because `BroadcastChannel` delivers messages across same-origin instances (meaning an initiating tab's own secondary listener could receive its own message), every tab must generate a unique `TAB_ID`, ignore its own broadcasts (`senderTabId === TAB_ID`), and secondary tabs must send `LOGOUT_TAB_ACK` (or `LOGOUT_TAB_ERROR` on failure) only after `await terminate(db)` completes. **Important Limitation:** Without a global browser tab registry, the `BroadcastChannel` + `TAB_ID` + `LOGOUT_TAB_ACK` / `LOGOUT_TAB_ERROR` + timeout protocol remains **best-effort coordination** and must never be documented as guaranteed erasure.
- Service Workers may cache only static App Shell assets; caching authenticated API responses or personal data by default is prohibited.

---

## 12. IAM, Secrets, and Audit Logging (`FIREBASE-IAM-001`)

- Run Cloud Functions with dedicated or separated least-privilege service accounts rather than an over-privileged default Compute Engine service account.
- Store backend secrets exclusively in Google Cloud Secret Manager (`defineSecret`); never commit them in `.env` files to git or include them in frontend bundles, source maps, or logs.
- Prefer ADC and Workload Identity Federation over long-lived service account JSON key files.
- Structured logs (`logger.info` / `logger.error`) must contain only necessary security events and filtered error metadata (`errorName`, `errorCode`). Logging passwords, tokens, session IDs, full request bodies, raw exception dumps, payment data, or unnecessary PII is prohibited.
- In L2/L3 systems, configure Cloud Monitoring log-based alerts for: repeated Firestore Rules `permission-denied` and `resource-exhausted` errors, sudden spikes in document reads or function invocations, and any modification of admin Custom Claims (`setCustomUserClaims`) or manual balance/pass adjustments.

---

## 13. Backups, Recovery, and Cost Defense (`FIREBASE-BACKUP-001`)

- Select backup frequency based on Tier and RPO/RTO targets (L1: periodic exports; L2: scheduled Cloud Storage exports + PITR for critical state; L3: mandatory Firestore PITR + regular restore drills).
- A backup strategy is incomplete without a documented restoration test (restore drill).
- Firestore PITR, managed exports, and Cloud Storage backups are not interchangeable; their retention, cost, and recovery mechanics differ.
- Mitigate Denial-of-Wallet risks by combining `maxInstances`, App Check, per-UID/per-IP rate limiting, daily/monthly user quotas, and Budget Alerts.

---

## 14. Firebase Release Checklist (`FIREBASE-CHK-001`)

- [ ] L1/L2/L3 tier and A/B/C onboarding model documented (`SECURITY_PROFILE.md`).
- [ ] Auth providers, *Authorized domains*, signup restrictions, and `email_verified` verified (`FIREBASE-AUTH-001`).
- [ ] App Check monitored/enforced; endpoints with `consumeAppCheckToken: true` paired with client `limitedUseAppCheckTokens: true` and `roles/firebaseappcheck.tokenVerifier` IAM role (`FIREBASE-FN-002`).
- [ ] Firestore and Storage Rules enforce default deny, `hasOnly`, type/size bounds, and immutability, verified by positive/negative emulator tests (`REF-TEST-RULES-001`) (`FIREBASE-RULES-001`..`003`).
- [ ] Admin SDK access verified at IAM and server handler levels via dedicated negative tests (`REF-TEST-ADMIN-001`) (`FIREBASE-ADMIN-001`).
- [ ] Critical business mutations separate the rate-limit transaction (`FIREBASE-FN-003`) from the atomic business + `requestHash` idempotency + append-only `ledger` transaction (`FIREBASE-FN-004`).
- [ ] Firestore TTL enabled only on `_idempotency` and `_rateLimits` collections (`FIREBASE-IDEMP-001`).
- [ ] Secret Manager, least-privilege service accounts, and PII-free structured logging verified (`FIREBASE-IAM-001`).
- [ ] PWA `memoryLocalCache()` or `TAB_ID`-isolated best-effort multi-tab logout (`REF-PWA-LOGOUT-001`) tested (`FIREBASE-PWA-001`).
- [ ] Backup/restore and Cloud Monitoring/Budget alerts verified (`FIREBASE-BACKUP-001`).

---

## 15. Official Sources

For exact URLs, verification dates (`last_verified`), and version status, see `10_SOURCE_REGISTER.md`:
- Firebase Security Checklist: https://firebase.google.com/support/guides/security-checklist
- Firebase App Check for Cloud Functions: https://firebase.google.com/docs/app-check/cloud-functions
- Firestore Security Rules Get Started: https://firebase.google.com/docs/firestore/security/get-started
- Firestore Rules Structure & Limits: https://firebase.google.com/docs/firestore/security/rules-structure
- Secure Firestore Queries: https://firebase.google.com/docs/firestore/security/rules-query
- Firebase Custom Claims: https://firebase.google.com/docs/auth/admin/custom-claims
- Firebase Pricing: https://firebase.google.com/pricing
- Firebase Local Emulator Suite & Rules Unit Testing: https://firebase.google.com/docs/rules/unit-tests