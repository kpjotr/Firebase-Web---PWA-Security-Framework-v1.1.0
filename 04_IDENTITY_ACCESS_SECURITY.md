---
name: identity-access-security
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
---

# 04 – Identity & Access Security

## 1. Scope (`IAM-SCOPE-001`)

This module (`Tier-1` selective domain module under `CORE-LOAD-001`) defines security requirements for Firebase Authentication, Identity Platform, OAuth 2.0 / OpenID Connect (OIDC), email/password authentication, MFA, session lifecycle, Custom Claims, role/permission models, account recovery, and administrative access.

Loading this module is mandatory in every project featuring user accounts, login, registration, admin interfaces, roles, protected resources, or external identity providers.

---

## 2. Separation of Identity and Authorization (`IAM-SEP-001`)

Never conflate or substitute these four distinct defense layers (`CORE-PRIN-007`, `CORE-PRIN-012`):
- **Authentication:** Who is the user? (e.g., Firebase Authentication / Identity Platform).
- **Authorization:** What is the authenticated user permitted to do with a specific resource? (Firestore/Storage Security Rules and Cloud Functions server-side checks).
- **App Check:** Does the request likely originate from the registered application instance? (Does not prove user identity and is not authorization!).
- **Google Cloud IAM:** What Google Cloud / Firebase operations is a server-side Service Account permitted to perform?

A successful login (`request.auth != null`) never grants automatic access to all records!

---

## 3. Registration, Onboarding, and Quotas (`IAM-LIFE-001`)

- Record Onboarding Model **A (Admin-provisioned)**, **B (Invite-only)**, or **C (Public signup)** (`CORE-ONBOARD-001`) at project kickoff in `SECURITY_PROFILE.md` and configure it in the Firebase Console / Identity Platform settings (`FIREBASE-AUTH-001`).
- For Public signup (Model C), enforce rate limiting, abuse defense, App Check, Email Enumeration Protection, mandatory email verification (`request.auth.token.email_verified == true`), and zero-privilege default access.
- For Invite-only (Model B), invitation tokens must be cryptographically random, single-use, short-lived, bound to the target user/tenant, and verified server-side (e.g., via a `beforeUserCreated` blocking function or transactional invite redemption).
- **Identity Platform Quotas and Pricing Allowances (`FIREBASE-LIMIT-001`):**
  - **Spark Plan (`[PLAN QUOTA]`):** **3,000 DAU** daily active user limit on the Spark (no-cost) plan for affected Identity Platform features (e.g., blocking functions, TOTP MFA).
  - **Blaze Plan (`[PRICING / FREE TIER ALLOWANCE]`):** **50,000 MAU** no-cost allowance for standard providers and TOTP MFA; differentiated pricing across provider categories (SMS MFA incurs per-message charges and SAML/OIDC is billed separately; always check current official Firebase Pricing before release!).
- Account suspension, deletion, and role revocation must be server-side operations; clients must never modify their own privileged fields.
- Account recovery (password reset) flows must never be weaker than primary authentication.

---

## 4. Passwords, Authenticators, and MFA (`IAM-CRED-001`)

- Never store plaintext or reversible passwords in your own database (Firestore).
- When using Firebase Authentication credentials, never implement a parallel custom password database.
- If handling custom secondary secrets/PINs is unavoidable, design according to **NIST SP 800-63-4** and the **OWASP Authentication Cheat Sheet**: strong secret rules, breached password screening, memory-hard hashing (Argon2id) with unique salts, strict rate limiting, and secure recovery.
- Do not force arbitrary periodic password rotation without evidence or suspicion of compromise (per NIST SP 800-63-4).
- **Multi-Factor Authentication (MFA):** Enforce MFA based on risk and role; **MFA is MANDATORY (`MUST`) for all administrative and privileged accounts in L2 and L3 systems**.
- TOTP MFA, passkeys/WebAuthn, and SMS MFA provide different assurance and phishing-resistance levels; do not treat them as equivalent, and avoid SMS MFA where TOTP or WebAuthn is feasible.

---

## 5. OAuth 2.0 and OpenID Connect (`IAM-OAUTH-001`)

- Use **Authorization Code Flow + PKCE** for public clients (SPAs/PWAs) per **RFC 9700 (OAuth 2.0 Security Best Current Practice)**.
- Enforce `state` (CSRF protection) and `nonce` (replay protection) parameters per protocol and SDK requirements.
- Restrict Redirect URIs to an exact allowlist without wildcards.
- Request the narrowest necessary OAuth `scope` (e.g., `drive.file` instead of full Drive access – `FIREBASE-STORAGE-001`).
- Delegate client-side access/refresh token storage to the official platform SDK; never log tokens or send them to analytics.
- Never expose application-level OAuth `client_secret` values to the frontend; store them in Secret Manager, and store per-user external OAuth `refresh_token` values in a locked server-only Firestore collection encrypted at the application layer (e.g., via Cloud KMS – `FIREBASE-STORAGE-001`).
- OIDC ID tokens are for authentication, not general-purpose external API access tokens.
- During server-side token validation, always verify `iss` (issuer), `aud` (audience), cryptographic signature, `exp` (expiration), `nonce`/`state`, and allowed algorithm (`alg`) using the official vendor library.

---

## 6. Firebase Auth Tokens and Custom Claims (`IAM-CLAIMS-001`)

- **Size Limit (`FIREBASE-LIMIT-001`):** Custom Claims payloads are capped at **1,000 bytes** per user (`[PLATFORM HARD LIMIT]`). Store only small, infrequently changing authorization attributes (e.g., `role: 'admin' | 'staff'`, `tenantId`); never store full user profiles in Custom Claims.
- **Propagation Delay:** Server-side Custom Claims updates (`setCustomUserClaims`) **do not propagate immediately** to already-issued ID tokens (valid for up to 1 hour); they take effect only when a fresh token is issued (`user.getIdToken(true)`).
- When immediate privilege or admin role revocation is required, do not wait for natural token expiration: combine claim updates with refresh token revocation (`admin.auth().revokeRefreshTokens(uid)`), `checkRevoked: true` verification, or a server-side/Rules active status check (`FIREBASE-AUTH-001`).
- Never trust `role` or `isAdmin` values passed inside client request payloads; evaluate `request.auth.token.role` in Rules and verified `request.auth.token` claims in Cloud Functions.

---

## 7. Session Lifecycle and Logout (`IAM-SESS-001`)

- Distinguish clearly among client-side `signOut(auth)`, server-side refresh token revocation (`revokeRefreshTokens`), JWT ID token lifetime (1 hour), and server-side session cookies (`__Host-` prefixed Host-Only cookies).
- Require recent authentication (**step-up auth** verifying the `auth_time` claim) before sensitive account actions (password change, email change, account deletion, MFA modification, admin role assignment).
- On shared devices, avoid long-lived persistent sessions or persistent IndexedDB caches (`memoryLocalCache()` is preferred).
- Implement multi-tab PWA logout and cache cleanup using the `TAB_ID`-isolated `BroadcastChannel` best-effort protocol defined in `02_FIREBASE_SECURITY.md` (`FIREBASE-PWA-001`) and `11_REFERENCE_TEMPLATES_FIREBASE.md` (`REF-PWA-LOGOUT-001`), and never document it as forensic erasure!

---

## 8. Authorization Model (`IAM-AUTHZ-001`)

- Document an explicit role–permission–resource matrix (`get`, `list`, `create`, `update`, `delete`, and callable operations) in `SECURITY_PROFILE.md`.
- Alongside role-based access control (RBAC), always verify resource ownership (`resource.data.userId == request.auth.uid`) and tenant scope (`tenantId`) (ABAC / BOLA defense).
- Every API and callable endpoint must verify caller permissions server-side (`REF-FN-CALLABLE-001`).
- Never rely on hidden frontend routes, disabled buttons, or client-side role checks.
- Enforce Deny by Default (`CORE-PRIN-003`).
- Prevent BOLA / IDOR: bind every client-supplied resource ID (`docId`, `passId`) to the authenticated `request.auth.uid` inside a server-side transaction (`pass.userId === request.auth.uid`).

---

## 9. Admin and Privileged Access (`IAM-ADMIN-001`)

- Protect administrative accounts rigorously; **Admin MFA is mandatory** at L2 and L3 tiers.
- Prohibit shared admin accounts; every administrator must use an individual, auditable account.
- Log all administrative operations using structured audit logs (`Cloud Logging`) wired to monitoring alerts (`FIREBASE-IAM-001`).
- Strictly separate machine `Service Account` identities from human administrator identities.
- **Google Sign-In vs. Admin MFA (`FIREBASE-AUTH-001`):** Consumer `@gmail.com` Google OAuth login **does not prove** 2SV/MFA execution! Google Sign-In may only be accepted as admin MFA if accounts belong to a managed **Google Workspace** organization with mandatory policy-enforced 2SV, combined with a server-side email allowlist and an `admin: true` Custom Claim. Otherwise, Identity Platform TOTP MFA (`request.auth.token.firebase.sign_in_second_factor != null`) is mandatory.

---

## 10. Mandatory Identity & Access Tests (`IAM-TEST-001`)

Verify every relevant feature with positive and negative (`assertFails` / `HttpsError`) tests (`REF-TEST-RULES-001`, `REF-TEST-ADMIN-001`) covering:
- unauthenticated (anonymous) caller rejection;
- unverified email (`email_verified: false`) caller rejection;
- regular user accessing their own resource (positive);
- attempting to read or mutate another user's resource (`userId` / `passId` BOLA -> rejection);
- cross-tenant resource access attempt (rejection);
- expired or invalid token rejection;
- revoked refresh token or revoked role handling;
- missing MFA or missing step-up authentication rejection on privileged actions;
- invitation token reuse or expired invitation rejection;
- invoking an admin endpoint with a regular user token (BFLA -> rejection);
- account recovery abuse (rate limiting / enumeration) testing.

---

## 11. Official Sources (`IAM-REF-001`)

For exact version status and verification dates (`last_verified`), see `10_SOURCE_REGISTER.md`:
- NIST SP 800-63-4 Digital Identity Guidelines: https://csrc.nist.gov/pubs/sp/800/63/4/final
- Firebase Authentication Docs: https://firebase.google.com/docs/auth
- Firebase Custom Claims Docs: https://firebase.google.com/docs/auth/admin/custom-claims
- Firebase Multi-Factor Authentication (MFA): https://firebase.google.com/docs/auth/web/multi-factor
- OAuth 2.0 Security Best Current Practice (RFC 9700): https://www.rfc-editor.org/rfc/rfc9700
- OpenID Connect Core 1.0: https://openid.net/specs/openid-connect-core-1_0.html
- OWASP Authentication Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- OWASP Authorization Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html