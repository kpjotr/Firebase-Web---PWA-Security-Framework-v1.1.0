---
name: security-testing-threat-modeling
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
---

# 09 – Security Testing & Threat Modeling

## 1. Scope and Activation (`TEST-SCOPE-001`)

This module (`Tier-1` selective domain module under `CORE-LOAD-001`) is **mandatory**:
- during project planning and architecture design;
- when introducing new data flows, roles, API endpoints, file uploads, authentication methods, or external integrations;
- during major architectural or authorization changes;
- prior to every L2 and L3 production release (and in L1 for authorization/Rules testing per `CORE-TIER-L1-001`);
- following any security incident or newly discovered vulnerability;
- as part of periodic (90-day) security audits (`CORE-FRESH-001`).

When writing concrete Firebase Security Rules and Cloud Functions (Admin SDK) tests, loading `REF-TEST-RULES-001` and `REF-TEST-ADMIN-001` from `11_REFERENCE_TEMPLATES_FIREBASE.md` is also mandatory.

---

## 2. Threat Modeling (`TEST-THREAT-001`, `TEST-STRIDE-001`)

### 2.1. Trust Boundaries and Components (`TEST-THREAT-001`)
Map and record in `SECURITY_PROFILE.md`:
- user roles (anonymous, registered customer, staff, admin) and tenant boundaries;
- browser/PWA client environment (Zero Trust Client – `CORE-PRIN-005`);
- Firebase Authentication / Identity Platform boundary;
- direct client access boundary to Firestore and Cloud Storage (`firestore.rules`, `storage.rules`);
- Cloud Functions v2 server execution boundary (which bypasses Rules via the Admin SDK – `FIREBASE-ADMIN-001`);
- Google Cloud Service Account / IAM and Secret Manager boundary;
- external APIs, payment webhooks, OAuth providers, and AI/LLM endpoints;
- CI/CD pipelines, git repositories, and AI coding agent workspaces (`08_AI_AGENT_CODING_SECURITY.md`).

### 2.2. STRIDE and OWASP Threat Categories (`TEST-STRIDE-001`)
Evaluate every critical data flow and endpoint using **STRIDE**:
- **Spoofing:** forging identity, email verification, App Check tokens, or webhook signatures;
- **Tampering:** unauthorized modification of records, ownership fields (`userId`), balances, passes, or idempotency parameters (`requestHash`);
- **Repudiation:** untraceable balance, pass, or privilege mutations (mitigated via the append-only `passLedger` and structured audit logs);
- **Information Disclosure:** BOLA/IDOR reads, unfiltered list queries, client-side PWA cache leakage on shared devices, stack trace or PII logging;
- **Denial of Service / Denial of Wallet:** unbounded queries, missing rate limits, concurrent double-booking race conditions, flooding expensive Cloud Functions or LLM APIs;
- **Elevation of Privilege:** Mass Assignment (`isAdmin: true`, `role: 'admin'`), BFLA on admin endpoints, or missing Admin SDK authorization checks.

Supplement STRIDE with **OWASP ASVS 5.0.0**, **OWASP API Security Top 10 (2023)**, and **OWASP Top 10 for LLM Applications (2025)** categories.

---

## 3. Security Requirement and Test Traceability Matrix (`TEST-MATRIX-001`)

Maintain a traceability matrix for all L2 and L3 critical requirements:
- `Requirement ID` (e.g., `FIREBASE-RULES-002`, `FIREBASE-FN-003`, `FIREBASE-FN-004`);
- module and normative source;
- affected component and file;
- mitigated threat;
- implemented control;
- positive test reference;
- negative test (`assertFails` / `HttpsError`) reference;
- actual verification level (`Verification Level` – `REF-VERIFY-001`) and test result;
- owner and residual risk.

---

## 4. Firebase-Specific Security Testing (`TEST-FB-001`, `TEST-NEG-001`)

### 4.1. Dual Firebase Testing Obligation (`TEST-FB-001`)
1. **Firestore and Storage Security Rules Unit Tests (`REF-TEST-RULES-001`):** Execute using Firebase Local Emulator Suite + `@firebase/rules-unit-testing`. Firebase Console *Rules Playground* simulations **do not replace** version-controlled, CI-automated emulator unit tests!
2. **Server-Side Admin SDK Handler Authorization and Transaction Tests (`REF-TEST-ADMIN-001`):** Because the Firebase Admin SDK completely bypasses Security Rules (`FIREBASE-ADMIN-001`), every Cloud Functions v2 (`onCall`/`onRequest`) handler must be tested directly against the emulator with dedicated negative authorization, validation, rate-limit, and idempotency tests!
- Always run tests against local emulators or an isolated demo project (`demo-*`); **never run security tests against production databases or live customer data!**

### 4.2. Mandatory Negative Test Cases (`TEST-NEG-001`)
No authorization or business feature may receive a `PASS` status if relevant negative (`assertFails` / rejection) tests are missing:
- [ ] anonymous (unauthenticated) access attempt;
- [ ] unverified email (`email_verified: false`) access attempt;
- [ ] reading or modifying another user's document (BOLA / IDOR);
- [ ] accessing another tenant's document;
- [ ] attempting to mutate immutable `userId`, `tenantId`, or `createdAt` fields during `update`;
- [ ] injecting unallowed extra fields (`isAdmin: true`, `role: 'admin'`) (`hasOnly` and `Zod .strict()` tests);
- [ ] invalid data types or out-of-range values (e.g., negative duration/amount);
- [ ] `list` query missing the required owner filter (`where('userId', '==', uid)`) (`unfilteredQuery` -> `assertFails`);
- [ ] `list` query missing the `.limit()` clause (`noLimitQuery` -> `assertFails`) or exceeding the max limit (`50`) (`excessiveQuery` -> `assertFails`);
- [ ] direct client-side write attempt to locked critical collections (`passes`, `passLedger`, `bookings` -> `assertFails`);
- [ ] regular user invoking an administrative Cloud Function (BFLA);
- [ ] rate limit exhaustion (`resource-exhausted`), plus verifying that the counter increment committed in Transaction 1 (rate limit) persists even when Transaction 2 (business) fails a precondition (`FIREBASE-FN-003`, `REF-TEST-ADMIN-001`);
- [ ] reusing an `idempotencyKey` with modified/tampered input parameters (`requestHash` mismatch -> `already-exists` rejection – `FIREBASE-FN-004`, `REF-TEST-ADMIN-001`);
- [ ] forged signature or replayed webhook rejection;
- [ ] oversized or spoofed MIME/magic-byte Storage file upload rejection.

---

## 5. Frontend and PWA Security Testing (`TEST-FRONT-001`)

- DOM XSS, reflected XSS, and stored XSS testing across all dynamic rendering sinks.
- URL scheme injection (`javascript:`, `data:`) and open redirect testing.
- Verification of service-specific CSP and HTTP security headers (`firebase.json`).
- `iframe` sandbox, clickjacking, and `postMessage` origin validation testing.
- **PWA Logout and Cache Isolation Testing (`REF-PWA-LOGOUT-001`):**
  - Multi-tab `BroadcastChannel` + `TAB_ID` logout testing (verifying self-echo suppression, `LOGOUT_TAB_ACK` / `LOGOUT_TAB_ERROR`, and timeout fallback);
  - Service Worker cache inspection (verifying zero authenticated responses or PII remain in Cache Storage);
  - `localStorage`, `sessionStorage`, and `IndexedDB` inspection after logout.

---

## 6. Testing Layers and the 4+1 Verification Levels (`TEST-LAYERS-001`, `REF-VERIFY-001`)

### 6.1. The 7 Testing Layers (`TEST-LAYERS-001`)
1. **Unit tests:** schema validation (`Zod .strict()`), business invariants, helper functions.
2. **Emulator / Integration tests:** Firebase Local Emulator Suite (`firestore.rules`, `storage.rules`, Cloud Functions v2 Admin SDK handlers, Auth).
3. **E2E tests:** full browser user journeys across multiple roles and multiple tabs.
4. **SAST / SCA / Secret scans:** static code analysis, dependency auditing, and secret detection.
5. **DAST:** dynamic scanning of staging endpoints and security headers.
6. **Manual Security Review (`CORE-REV-001`):** independent review of business logic, access control boundaries, race conditions, and abuse cases.
7. **Penetration testing:** independent expert penetration testing for L3, high-risk, or regulated systems.

*(Automated scanners never replace targeted manual review of business logic and authorization!)*

### 6.2. Mandatory Labeling of Verification Levels (`REF-VERIFY-001`)
Every code template, test report, and handoff contract MUST truthfully state the actual verification level achieved:
- **`DOCUMENTATION_REVIEWED`:** Documentation and logical review only; no code execution occurred (the baseline status of templates in `11_REFERENCE_TEMPLATES_FIREBASE.md`).
- **`STATIC_CHECKED`:** Type-checking (`tsc`) and linting executed, but no runtime tests run.
- **`EMULATOR_TESTED`:** Positive and negative tests actually executed and passed in the Firebase Emulator Suite.
- **`PROJECT_TESTED`:** Verified within the target project's full environment (CI/staging).
- **`NOT_VERIFIED`:** No verification performed.

---

## 7. Applying OWASP ASVS 5.0.0 (`TEST-ASVS-001`)

- The stable **OWASP ASVS 5.0.0** release is the primary web application verification baseline for this framework (`10_SOURCE_REGISTER.md`).
- Each project defines its target ASVS level and applicable requirements in `SECURITY_PROFILE.md`.
- **Prohibition of Unsubstantiated Compliance Claims:** Never claim a project is "OWASP ASVS compliant" without item-by-item requirement verification backed by concrete evidence.

---

## 8. Release Gate, Reviewer Independence, and Documentation Gate (`TEST-GATE-001`, `CORE-REV-001`, `CORE-DOCGATE-001`)

Use `RELEASE_GATE_TEMPLATE.md` and the `REVIEWER_TO_PLANNER` handoff contract (`CORE-HANDOFF-001`):

1. **Application Release `FAIL` (Release Blocked) When:**
   - any `CRITICAL` or `HIGH` exploitable security finding remains open;
   - Firestore/Storage Rules or server-side Admin SDK authz negative tests were not executed or failed;
   - any secret was committed to git, bundled into the frontend, or written to logs;
   - a critical business endpoint operates without server-side authorization, atomic transactions, or idempotency;
   - security tests were not re-executed against the combined codebase after a parallel merge/rebase (`CORE-CONC-001`);
   - any security test or control was disabled without documented rationale and human approval.
2. **Security Reviewer Independence (`CORE-REV-001`):** The Reviewer operates in a separate phase, independently inspects `git diff` and test outputs (`independently_executed_tests`), must not self-approve at L2/L3 tiers, and records `review_independence` (`INDEPENDENT_MODEL` / `SEPARATE_AGENT_INSTANCE` / `SELF_REVIEWED_L1_ONLY`).
3. **Scope Limitation of `DOCUMENTATION_RELEASE_GATE: PASS` (`CORE-DOCGATE-001`):** A `DOCUMENTATION_RELEASE_GATE: PASS` issued for the documentation package certifies solely documentation completeness, referential integrity, and HU/EN parity; **it does not certify** application security, project implementation correctness, completion of a full security audit or penetration test, production readiness, or legal/regulatory compliance!

---

## 9. Minimum Structure of an Audit Report (`TEST-AUDIT-001`)

Every security review or audit report must include:
1. Scope (`scope`), inspected files (`inspected_files`), and environment;
2. Inspected versions and exact git commit/diff identifier (`reviewed_commit_or_diff_id`);
3. Applied standards, loaded modules (`01`–`11`), and methodology;
4. Actually executed tests (`independently_executed_tests`), tools, and deferred tests;
5. Discovered `findings` categorized by severity (`CRITICAL`, `HIGH`, `MEDIUM`, `LOW`) and `requirement_id`;
6. Reproduction steps / exploit scenario and business impact;
7. Concrete remediation tasks (`rework_tasks`) and remediated code;
8. Retest results;
9. Assessment limitations and `residual_risks`;
10. Verification level (`Verification Level`: `DOCUMENTATION_REVIEWED` / `STATIC_CHECKED` / `EMULATOR_TESTED` / `PROJECT_TESTED`), Reviewer independence (`review_independence`), and `final_status` (`PASS` / `CONDITIONAL_PASS` / `FAIL` / `NOT_VERIFIED`).

---

## 10. Official Sources (`TEST-REF-001`)

For exact version status and verification dates (`last_verified`), see `10_SOURCE_REGISTER.md`:
- OWASP Application Security Verification Standard (ASVS 5.0.0): https://asvs.owasp.org/
- OWASP Web Security Testing Guide (WSTG v4.2 Stable): https://owasp.github.io/www-project-web-security-testing-guide/stable/
- OWASP WSTG Project: https://owasp.org/projects/web-security-testing-guide
- Firebase Local Emulator Suite: https://firebase.google.com/docs/emulator-suite
- Firebase Security Rules Unit Testing: https://firebase.google.com/docs/rules/unit-tests
- OWASP SAMM: https://owaspsamm.org/