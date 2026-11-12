# Release Security Gate & Documentation Release Gate (v1.1.0)

**Project / Version / Commit (`reviewed_commit_or_diff_id`):**  
**Task ID (`task_id`):**  
**Date:**  
**Tier:** `L1` / `L2` / `L3`  
**Reviewer (AI Model / Human Auditor):**  
**Review Independence (`review_independence` – `CORE-REV-001`):** `INDEPENDENT_MODEL` / `SEPARATE_AGENT_INSTANCE` / `SELF_REVIEWED_L1_ONLY`  
**Associated Handoff Contract:** `REVIEWER_TO_PLANNER`  

---

## 0. L1 Lightweight Final Report Option (`CORE-TIER-L1-001`)
> *If the project operates in L1 Lightweight Mode, the task release report may be condensed into the 4 lines below, provided test results remain traceable in the repository:*
> 1. **Modified files and commit/diff ID:**
> 2. **Enforced security controls (`isOwner`, `hasOnly`, secret protection):**
> 3. **Executed positive and negative (`assertFails`) tests and command:**
> 4. **Status (`PASS` / `CONDITIONAL_PASS` / `FAIL` / `NOT_VERIFIED`) and `review_independence`:**

*(For L2 and L3 projects, completing Sections 1–7 in full is MANDATORY!)*

---

## 1. Code, File Locking, and Dependencies (`06_DEVSECOPS_SUPPLY_CHAIN.md`, `CORE-CONC-001`)
- [ ] Exclusive file locking (`TECHNICAL_LOCK` / `COORDINATION_RESERVATION`) respected, `lock_released: true`
- [ ] All tests re-executed against the combined codebase following any parallel merge/rebase
- [ ] lint / typecheck / build succeeded
- [ ] unit / integration tests succeeded
- [ ] lockfile (`package-lock.json`) and Node/runtime version pinned and verified (`Project Freshness`)
- [ ] dependency / SCA scan (`npm audit`) triaged
- [ ] secret scan clean (zero secrets in git, frontend bundle, or logs)
- [ ] SAST results triaged

## 2. Firebase and Backend (`02_FIREBASE_SECURITY.md`, `11_REFERENCE_TEMPLATES_FIREBASE.md`)
- [ ] Firestore Rules (`rules_version = '2';`) default deny, verified with positive and negative emulator tests (`REF-TEST-RULES-001`) (`EMULATOR_TESTED` / `PROJECT_TESTED`)
- [ ] Storage Rules tested (and server-side magic-byte validation active for L2/L3), if Storage is used
- [ ] Admin SDK authorization separately tested via handler-level negative tests (`REF-TEST-ADMIN-001`)
- [ ] App Check configuration (and for L3 / critical L2: `consumeAppCheckToken: true` + client `limitedUseAppCheckTokens: true` + `roles/firebaseappcheck.tokenVerifier` IAM role) verified
- [ ] Callable/HTTP authz, Zod `.strict()` validation, standalone transactional rate limit (`FIREBASE-FN-003`), and atomic business transaction + `requestHash` idempotency + append-only `ledger` (`FIREBASE-FN-004`) tested
- [ ] IAM least-privilege service accounts and Secret Manager verified
- [ ] Cloud Monitoring alerts and Budget Alerts verified

## 3. Frontend and PWA (`03_FRONTEND_SECURITY.md`)
- [ ] XSS / DOM XSS controls verified (no unsanitized `dangerouslySetInnerHTML` / `innerHTML`)
- [ ] Service-specific CSP and HTTP security headers (`X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`, HSTS, Host-Only cookies) verified
- [ ] Authenticated cache, offline queue, and PWA logout (`memoryLocalCache()` or `TAB_ID`-isolated `BroadcastChannel` + `LOGOUT_TAB_ACK` / `LOGOUT_TAB_ERROR` best-effort cleanup – `REF-PWA-LOGOUT-001`) verified
- [ ] Zero tokens or PII in client console logs or public bundles
- [ ] External scripts and SRI / self-hosting decisions documented

## 4. Data Protection & Privacy (`07_DATA_PROTECTION_PRIVACY.md`)
- [ ] Processing purposes, legal bases, and retention schedule verified (no Firestore TTL applied to business/accounting ledgers)
- [ ] PII log redaction verified
- [ ] External processors and AI/LLM data minimization verified
- [ ] Cascading deletion and backup/restore workflows aligned

## 5. Open Findings (`REVIEWER_TO_PLANNER.findings`)
| ID | Severity (`CRITICAL`/`HIGH`/`MEDIUM`/`LOW`) | Requirement ID | File | Description | Compensating Control | Owner | Due Date | Approval |
|---|---|---|---|---|---|---|---|---|

## 6. Verification Evidence and Verification Level (`REF-VERIFY-001`)
- **Actual Verification Level of Templates/Code (`Verification Level`):** `DOCUMENTATION_REVIEWED` / `STATIC_CHECKED` / `EMULATOR_TESTED` / `PROJECT_TESTED` / `NOT_VERIFIED`
- **Tests Actually Executed (command and output):**
- **Tests Independently Re-Executed by Reviewer (`independently_executed_tests`):**
- **Test Environment:**
- **Tools / SDK Versions:**
- **Not Applicable or Deferred Tests (`not_applicable_or_deferred_tests`):**
- **Known Limitations and Residual Risks (`residual_risks`):**

## 7. Application / Task Release Decision (`CORE-HANDOFF-001`)
- [ ] `PASS` *(All mandatory checks and positive/negative tests actually executed and passed)*
- [ ] `CONDITIONAL_PASS` *(Only non-critical, documented items deferred; zero critical flaws)*
- [ ] `FAIL` *(Critical/high exploitable flaw open, or mandatory negative test missing/failed)*
- [ ] `NOT_VERIFIED` *(Tests were not executed / insufficient verifiable evidence)*
- **Rationale:**
- **Human Approver:**

---

## 8. Documentation Package Release Gate (`DOCUMENTATION_RELEASE_GATE` – `CORE-DOCGATE-001`)

*(Complete exclusively when releasing or updating the security framework documentation package itself!)*

> **IMPORTANT SCOPE LIMITATION (`CORE-DOCGATE-001`):** A `DOCUMENTATION_RELEASE_GATE: PASS` result **solely certifies** that the inspected documentation package satisfies the prescribed documentation completeness, structural, referential, HU/EN synchronization, and internal consistency requirements.
> **IT DOES NOT CERTIFY:** (1) the overall security of the Firebase application; (2) the correctness of a specific project implementation; (3) the completion of a comprehensive security audit; (4) the completion of a penetration test; (5) production readiness status; (6) legal or regulatory compliance certification (e.g., GDPR, ISO, ASVS).

- [ ] **1. Module Completeness:** All modules listed in `README.md` (`01`–`11`, templates, JSON Schema) physically exist in both HU and EN editions.
- [ ] **2. Referential Integrity:** Every referenced filename and requirement identifier actually exists in the target files.
- [ ] **3. HU/EN ID Parity:** Requirement identifiers in the Hungarian and English files match 100%.
- [ ] **4. HU/EN Content and Structural Alignment:** Section numbering, logical structure, and normative force match across both language editions.
- [ ] **5. Reference Code Module Completeness & Verification Level:** `11_REFERENCE_TEMPLATES_FIREBASE.md` contains all five prescribed templates with the 7-element metadata header and truthful `DOCUMENTATION_REVIEWED` verification level.
- [ ] **6. Three-Way Handoff YAML and JSON Schema Validity:** All three handoff contracts (`PLANNER_TO_CODER`, `CODER_TO_REVIEWER`, `REVIEWER_TO_PLANNER`) are documented and conform to the JSON Schema definition.
- [ ] **7. Template Consistency:** `SECURITY_PROFILE_TEMPLATE.md` and `RELEASE_GATE_TEMPLATE.md` comply with Core v1.1.0 prescriptions.
- [ ] **8. Legacy v3.2.2 Separation:** The `v3.2.2` legacy source file is clearly marked as a historical archive (`DEPRECATED / LEGACY ARCHIVE`).
- [ ] **9. Source Register Coverage:** `10_SOURCE_REGISTER.md` contains official URLs for every critical technical claim and platform limit.
- [ ] **10. Truthful Verification Dates:** Source verification dates reflect actual verification.
- [ ] **11. Internal Cross-References:** No broken or missing internal section or file references exist.
- [ ] **12. Package Integrity & Scope Declaration:** The `DOCUMENTATION_RELEASE_GATE: PASS` scope limitation is stated in both the documentation and the final release report.

**Documentation Release Gate Result:** `DOCUMENTATION_RELEASE_GATE: PASS` / `FAIL` / `NOT_VERIFIED`