---
name: api-integration-security
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
---

# 05 – API & Integration Security

## 1. Scope and Activation (`API-SCOPE-001`)

This module (`Tier-1` selective domain module under `CORE-LOAD-001`) is **mandatory** for every Cloud Functions v2 callable (`onCall`) and HTTP (`onRequest`) endpoint, REST API, webhook, external service, payment gateway, Google API, email service, or AI/LLM integration. When implementing concrete Firebase callable endpoints, also load `REF-FN-CALLABLE-001` and `REF-TEST-ADMIN-001` from `11_REFERENCE_TEMPLATES_FIREBASE.md`.

---

## 2. API Design (`API-DESIGN-001`)

- Every endpoint must have a documented purpose, owner, authentication and authorization model, strict input/output schema, rate limit, and structured logging plan.
- Strictly separate authentication (`request.auth`), application attestation (`request.app` / App Check), and authorization (ownership/role verification).
- Client-supplied values inside request payloads (`request.data` / `req.body`) for `userId`, `uid`, `role`, `tenantId`, `price`, `discount`, `balance`, `remainingUses`, or `status` are **never trusted** (`CORE-PRIN-005`, `CORE-PRIN-006`).
- On HTTP (`onRequest`) endpoints, allow only required HTTP methods (e.g., `POST`) and reject all others with `405 Method Not Allowed`.
- Enforce explicit `Content-Type` checks, request body size limits, timeouts, and `maxInstances` concurrency limits (`FIREBASE-FN-001`).
- Client-facing error responses must return only standardized error codes (e.g., `HttpsError`); never expose internal stack traces, database schemas, file paths, or secrets (`CORE-PRIN-009`).

---

## 3. Input Validation (`API-VAL-001`)

- Enforce server-side allowlist schema validation on every endpoint (e.g., **Zod `.strict()`** or equivalent – `REF-FN-CALLABLE-001`) that immediately rejects unknown/extra fields (Mass Assignment / OWASP API3:2023 defense).
- Validate field types, byte sizes, string lengths, array lengths, enum values, numeric bounds, formats (e.g., UUID), and business invariants.
- Perform input canonicalization (normalization) in a single well-defined processing layer; avoid multiple or inconsistent decoding passes between validation and execution.
- Apply targeted defenses against NoSQL/SQL injection, path traversal, command injection, template injection, and SSRF according to the downstream sink.
- Handle file and URL inputs under dedicated strict rules (`API-SSRF-001`, `FIREBASE-STORAGE-001`).

---

## 4. Authorization and BOLA / BFLA Defense (`API-AUTHZ-001`)

- **BOLA / IDOR (OWASP API1:2023) Defense:** On every record operation, verify resource ownership (`resource.userId === request.auth.uid`) and tenant scope server-side inside the transaction (`REF-FN-CALLABLE-001`).
- **BFLA (OWASP API5:2023) Defense:** Verify function-level and administrative permissions on the server, never solely via frontend routes or UI visibility.
- Require explicit admin claims/permissions and (for L2/L3) Admin MFA on admin-only endpoints (`IAM-ADMIN-001`).
- In bulk/batch operations, verify access permissions for **every individual record** in the batch.
- Never accept arbitrary Firestore document paths or collection names directly from client input!

---

## 5. Rate Limiting and Resource Consumption Defense (`API-RATE-001`)

- Rate-limit requests by UID, IP, tenant, endpoint, and cost where applicable (OWASP API4:2023 – Unrestricted Resource Consumption / Denial of Wallet defense).
- Apply dedicated, stricter limits to authentication, password reset, invitation redemption, email sending, booking, data export, search, and paid external/AI API endpoints.
- **Strict Separation of Two Transactions (`FIREBASE-FN-003`):** When using a Firestore-based per-UID rate limiter, execute rate limit verification and counter increments in a **standalone transaction prior to** the business transaction (`REF-FN-CALLABLE-001`). When the limit is exceeded, never throw inside the transaction callback: commit the incremented `blockedCount` first, then throw `resource-exhausted` *after* the transaction commits!
- **Handling Known Limitations:** Per-UID limits do not stop multi-account (sybil) attacks, and Firestore counters incur document contention and read/write costs under high concurrency (>1 req/sec/user-action document). On public or high-volume (L3) endpoints, combine App Check, IP-based rate limiting (Redis / Upstash / Memorystore), and WAF/Cloud Armor.
- Never rely solely on `maxInstances` or Budget Alerts as hard billing caps (`FIREBASE-FN-001`).
- Bound request body sizes, upload sizes, query page sizes (`limit`), execution timeouts, and external API quotas.

---

## 6. Idempotency, Transactional Concurrency, and Ledgers (`API-IDEMP-001`)

- Require an `idempotencyKey` parameter on every non-idempotent state-mutating operation (bookings, pass deductions, orders, payments, balance adjustments) (`FIREBASE-FN-004`, `FIREBASE-IDEMP-001`, `REF-FN-CALLABLE-001`).
- Scope idempotency records to the user and action (`_idempotency/${uid}_${idempotencyKey}`) and store a **SHA-256 `requestHash`** of the validated input parameters.
- For the same `idempotencyKey` + **identical `requestHash`**, return the stored result (`deduplicated: true`) without re-executing the deduction.
- For the same `idempotencyKey` + **different `requestHash`**, reject the request (`already-exists` / `failed-precondition`).
- Commit domain state changes, the **append-only `ledger`** entry, and the `_idempotency` record inside a **single atomic `db.runTransaction` operation** (`FIREBASE-FN-004`).
- **No External Side Effects Inside Transactions:** Because Firestore transaction callbacks may re-execute on contention, **never (`MUST NOT`)** perform external HTTP/API calls, email sending, or payment gateway mutations inside a transaction callback! Use an outbox/event pattern or idempotent post-processing instead.
- Design webhook and queue consumers to be idempotent, as delivery is typically at-least-once.

---

## 7. Webhook Security (`API-WEBHOOK-001`)

- For payment or third-party webhooks (`onRequest`), verify cryptographic signatures (e.g., HMAC) per vendor specifications against the **raw request body (`req.rawBody`)**!
- Enforce a timestamp tolerance window (replay defense), atomic `event.id` deduplication, and idempotent processing.
- Store dedicated webhook signing secrets in Secret Manager (`defineSecret`) and support key rotation.
- Never trust client-side redirect URLs or query parameters as proof of payment!
- Never execute business logic or database mutations before webhook signature verification succeeds.
- Acknowledge webhooks promptly within the provider's timeout window and process heavy workloads asynchronously/idempotently.

---

## 8. CORS and CSRF (`API-CORS-001`)

- CORS is solely a browser cross-origin read policy; **it never replaces authentication or authorization** (`CORE-PRIN-012`).
- Maintain an exact `Access-Control-Allow-Origin` allowlist; when `Access-Control-Allow-Credentials: true` is used, `*` wildcards are forbidden and reflecting arbitrary incoming `Origin` headers is **strictly prohibited**.
- For cookie-based sessions, enforce CSRF tokens, `SameSite=Strict` (or `Lax`), the `__Host-` prefix, and server-side `Origin`/`Referer` verification.
- For Bearer token (Authorization header / Firebase Callable SDK) APIs, classic CSRF risk differs, but XSS defense remains critical.

---

## 9. SSRF and Server-Side URL Fetching (`API-SSRF-001`)

Whenever the backend fetches a URL supplied or influenced by client input (OWASP API7:2023 – Server-Side Request Forgery):
- Enforce a strict destination domain allowlist whenever possible.
- Allow only `https:` and standard ports.
- After DNS resolution, block loopback (`127.0.0.0/8`, `::1`), private (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`), link-local (`169.254.0.0/16`, `fe80::/10`), **cloud metadata (`169.254.169.254`, `metadata.google.internal`)**, and all internal IPv4/IPv6 addresses!
- Defend against DNS rebinding and HTTP redirect following: disable redirects (`redirect: 'error'` / `manual`) or re-validate every redirect target at the DNS/IP level.
- Enforce strict response size caps, expected MIME types, and network timeouts.

---

## 10. External API and OAuth Integrations (`API-EXT-001`)

- Request the narrowest required OAuth scope, store API secrets in Secret Manager, and manage tokens server-side.
- Store per-user OAuth `refresh_token` values in a client-locked (`allow read, write: if false;`) Firestore collection encrypted at the application layer (e.g., via Cloud KMS – `FIREBASE-STORAGE-001`).
- Support token rotation, revocation, and account disconnect flows.
- Treat all external API responses as untrusted input and validate them against a schema!
- Enforce explicit timeouts, exponential backoff retries, circuit breakers, quotas, and cost caps.

---

## 11. AI / LLM API Integrations (`API-LLM-001`)

When integrating generative AI / LLM APIs:
- Invoke LLM providers exclusively from the backend (Cloud Functions v2); store API keys in Secret Manager (`defineSecret`).
- Enforce strict per-user / per-tenant daily and monthly quotas, token budgets, rate limits, max input/output lengths, and billing alerts (Denial of Wallet defense).
- Data minimization (`07_DATA_PROTECTION_PRIVACY.md`): never send unnecessary personal data (PII) or another user's business data to the model.
- **Prompt Injection and Excessive Agency (OWASP LLM Top 10 2025) Defense:** Model outputs are never trusted or authoritative; they must never make standalone authorization decisions or trigger privileged actions without deterministic server-side authorization.
- Validate model outputs against a strict schema (e.g., Zod) and sanitize/encode them before rendering in the browser (XSS defense).
- Treat RAG-retrieved or database-sourced documents as untrusted data, clearly separated from system instructions in the prompt.
- *Standards Note:* **OWASP LLM Top 10 (2025)** and **NIST SP 800-218A** serve as informative references; the latter primarily targets AI model development lifecycles and is not a standalone agent-runtime standard (`10_SOURCE_REGISTER.md`).

---

## 12. Mandatory API Test Checklist (`API-TEST-001`)

Verify every endpoint and integration with positive and negative (`REF-TEST-ADMIN-001`) tests:
- [ ] anonymous call, expired/invalid token, or unverified email rejection;
- [ ] cross-user or cross-tenant resource ID rejection (BOLA);
- [ ] regular user calling an admin endpoint rejected (BFLA);
- [ ] extra fields (Mass Assignment), invalid types, or oversized payloads rejected (`Zod .strict()`);
- [ ] rate limit exhaustion (`resource-exhausted`) and verification that Transaction 1 (rate limit) commits its increment even when Transaction 2 (business) fails a precondition (`FIREBASE-FN-003`);
- [ ] idempotent retry with same `idempotencyKey` + identical `requestHash` (`deduplicated: true`, no double deduction), and rejection when parameters differ (`requestHash` mismatch -> `already-exists`) (`FIREBASE-FN-004`);
- [ ] forged signature or replayed webhook rejection (`API-WEBHOOK-001`);
- [ ] SSRF internal IP / metadata IP (`169.254.169.254`) and redirect rejection (`API-SSRF-001`);
- [ ] CORS and CSRF enforcement verified (`API-CORS-001`);
- [ ] external API / LLM timeout and quota enforcement;
- [ ] verification that error responses and logs contain zero secrets, stack traces, or raw PII.

---

## 13. Official Sources (`API-REF-001`)

For exact version status and verification dates (`last_verified`), see `10_SOURCE_REGISTER.md`:
- OWASP API Security Top 10 (2023): https://owasp.org/API-Security/
- OWASP REST Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
- OWASP SSRF Prevention Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html
- Firebase Cloud Functions Docs: https://firebase.google.com/docs/functions
- OWASP Top 10 for LLM Applications (2025): https://owasp.org/www-project-top-10-for-large-language-model-applications/
- NIST SP 800-218A: https://csrc.nist.gov/pubs/sp/800/218/a/final