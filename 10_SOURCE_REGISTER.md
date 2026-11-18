---
name: security-source-register
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
---

# 10 – Source Register, Standards, Limits, and Documentation Catalog (`SRC-SCOPE-001`)

**Document Review Date (`last_reviewed`):** 2026-11-03  
**Next Mandatory 90-Day Audit Due (`next_full_audit_due`):** 2027-02-01  

**Normative Source Freshness Rule (`CORE-FRESH-001`):** The `last_verified` dates recorded in this register reflect exclusively official documentation that was actually verified on that date. Automatically or fictitiously assigning new verification dates to unverified references is strictly prohibited! URLs, quotas, pricing terms, and SDK versions evolve over time; re-verify affected sources prior to project kickoff, architectural decisions, and 90-day audits.

---

## 1. Primary Web Application Security Standards (`SRC-WEB-001`)

| Source | Status at Verification | Verification Date (`last_verified`) | Usage in Framework | Official URL |
| :--- | :--- | :--- | :--- | :--- |
| **OWASP ASVS** | **5.0.0 Stable** (released 2025-05-30) | `2026-11-02` | Primary web application security requirement and verification baseline (Modules `01`–`09`) | https://asvs.owasp.org/ and https://owasp.org/www-project-application-security-verification-standard/ |
| **OWASP WSTG** | **4.2 Stable** (5.0 in development) | `2026-11-02` | Web security testing methodology and test cases (`09_SECURITY_TESTING_THREAT_MODELING.md`) | https://owasp.github.io/www-project-web-security-testing-guide/stable/ and https://owasp.org/projects/web-security-testing-guide |
| **OWASP API Security Top 10** | **2023 Edition** (Stable) | `2026-11-02` | API risk categories: BOLA, BFLA, Mass Assignment, Resource Consumption, SSRF (`02`, `05`, `09`) | https://owasp.org/API-Security/ |
| **OWASP Top 10 for LLM Applications** | **2025 Edition** (Stable) | `2026-11-02` | Prompt injection, sensitive disclosure, excessive agency (`05_API_INTEGRATION_SECURITY.md`, `08_AI_AGENT_CODING_SECURITY.md`) | https://owasp.org/www-project-top-10-for-large-language-model-applications/ and https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/ |
| **OWASP SAMM** | Current Stable Release | `2026-11-02` | Organizational secure SDLC maturity reference (`09`) | https://owaspsamm.org/ |

---

## 2. NIST and Cybersecurity Frameworks (`SRC-NIST-001`)

| Source | Status and Scope | Verification Date (`last_verified`) | Official URL |
| :--- | :--- | :--- | :--- |
| **NIST CSF 2.0** | Stable Framework | `2026-11-02` | https://www.nist.gov/cyberframework |
| **NIST SP 800-218 (SSDF 1.1)** | **Final (Stable Baseline)** for secure software development lifecycle (`06`, `08`) | `2026-11-02` | https://csrc.nist.gov/pubs/sp/800/218/final |
| **NIST SP 800-218 Rev. 1 (SSDF 1.2)** | Listed as *Draft* (2025-12-17) on NIST publications page; **draft, not a final baseline!** | `2026-11-02` | https://csrc.nist.gov/projects/ssdf/publications |
| **NIST SP 800-218A** | **Final (2024-07-26)** – AI and dual-use foundation model development profile; supplementary informative reference for AI coding agents (`05`, `08`) | `2026-11-02` | https://csrc.nist.gov/pubs/sp/800/218/a/final |
| **NIST SP 800-63-4** | **Final (2025-07)** – Digital Identity Guidelines (superseded 800-63-3; `04_IDENTITY_ACCESS_SECURITY.md`) | `2026-11-02` | https://csrc.nist.gov/pubs/sp/800/63/4/final |
| **NIST AI RMF** | AI Risk Management Framework (`08`) | `2026-11-02` | https://www.nist.gov/itl/ai-risk-management-framework |

---

## 3. ISO/IEC Standards (`SRC-ISO-001`)

Informative references depending on project risk and contractual context:
- **ISO/IEC 27001:2022** – Information security management systems (ISMS) requirements.
- **ISO/IEC 27002:2022** – Information security controls.
- **ISO/IEC 27034 series** – Application security.
- **ISO/IEC 27701** – Privacy information management (PIMS); verify current edition.
- **Official Catalog:** https://www.iso.org/standards.html (`last_verified: 2026-11-02`)

**Normative Note (`CORE-DOCGATE-001`):** Full texts of ISO standards are licensed. This framework does not reproduce licensed ISO text and makes no claim of ISO certification.

---

## 4. Official Sources for Firebase, Google Cloud, and Platform Limits (`SRC-FB-001`, `FIREBASE-LIMIT-001`)

| Topic / Platform Limit | Category (`FIREBASE-LIMIT-001`) | Verification Date (`last_verified`) | Official Documentation URL |
| :--- | :--- | :--- | :--- |
| **Firebase Security Checklist** | General Firebase security principles | `2026-11-02` | https://firebase.google.com/support/guides/security-checklist |
| **Custom Claims 1,000-Byte Limit** | `[PLATFORM HARD LIMIT]` – Max. 1,000 bytes payload / user | `2026-11-02` | https://firebase.google.com/docs/auth/admin/custom-claims |
| **Firestore Rules `get()` / `exists()` Limits** | `[PLATFORM HARD LIMIT]` – Max. 10 calls / single request; max. 20 / multi-doc, tx, batch | `2026-11-02` | https://firebase.google.com/docs/firestore/security/rules-structure |
| **Identity Platform Spark (3k DAU) & Blaze (50k MAU)** | `[PLAN QUOTA]` (Spark 3,000 DAU) and `[PRICING / FREE TIER ALLOWANCE]` (Blaze 50,000 MAU) | `2026-11-02` | https://firebase.google.com/pricing |
| **Firebase App Check & Replay Protection (`consumeAppCheckToken`)** | Cloud Functions v2 App Check, `limitedUseAppCheckTokens`, and `roles/firebaseappcheck.tokenVerifier` | `2026-11-02` | https://firebase.google.com/docs/app-check/cloud-functions and https://firebase.google.com/docs/app-check |
| **Firestore Security Rules & Secure Queries** | Rules Are Not Filters, logical `OR` evaluation | `2026-11-02` | https://firebase.google.com/docs/firestore/security/get-started and https://firebase.google.com/docs/firestore/security/rules-query |
| **Firestore Offline Persistence & `clearIndexedDbPersistence()`** | Forensic and offline data-loss limitations, `terminate(db)`, `memoryLocalCache()` | `2026-11-02` | https://firebase.google.com/docs/firestore/manage-data/enable-offline |
| **Firebase Emulator Suite & Rules Unit Testing** | `@firebase/rules-unit-testing` (`assertSucceeds`, `assertFails`) | `2026-11-02` | https://firebase.google.com/docs/emulator-suite and https://firebase.google.com/docs/rules/unit-tests |
| **Firebase Auth & Multi-Factor Authentication (MFA)** | TOTP MFA, session lifecycle, token revocation | `2026-11-02` | https://firebase.google.com/docs/auth and https://firebase.google.com/docs/auth/web/multi-factor |
| **Cloud Functions for Firebase (2nd Gen / v2)** | `firebase-functions/v2/https` (`onCall`, `onRequest`, `maxInstances`) | `2026-11-02` | https://firebase.google.com/docs/functions |
| **Cloud Storage Security Rules** | File size, `contentType`, user-scoped paths | `2026-11-02` | https://firebase.google.com/docs/storage/security |
| **Firebase Hosting** | Security headers, CSP, target/subdomain separation | `2026-11-02` | https://firebase.google.com/docs/hosting |
| **Google Cloud IAM, Secret Manager, Logging & Monitoring** | Service account least privilege, `defineSecret`, audit logs, alerts | `2026-11-02` | https://cloud.google.com/iam/docs , https://cloud.google.com/secret-manager/docs , https://cloud.google.com/logging/docs , https://cloud.google.com/monitoring/docs |

---

## 5. Frontend, Browser, Language, and Node.js Runtime (`SRC-FRONT-001`)

| Source | Verification Date (`last_verified`) | Official URL |
| :--- | :--- | :--- |
| **WHATWG HTML Living Standard** | `2026-11-02` | https://html.spec.whatwg.org/ |
| **WHATWG Fetch Standard** | `2026-11-02` | https://fetch.spec.whatwg.org/ |
| **ECMAScript Specification (TC39)** | `2026-11-02` | https://tc39.es/ecma262/ |
| **MDN Web Security** | `2026-11-02` | https://developer.mozilla.org/en-US/docs/Web/Security |
| **MDN Content Security Policy (CSP)** | `2026-11-02` | https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP |
| **MDN Trusted Types API** | `2026-11-02` | https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API |
| **MDN Subresource Integrity (SRI)** | `2026-11-02` | https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity |
| **MDN Service Worker API** | `2026-11-02` | https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API |
| **MDN BroadcastChannel API** | `2026-11-01` | https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel |
| **React DOM Common Components (`dangerouslySetInnerHTML`)** | `2026-11-02` | https://react.dev/reference/react-dom/components/common |
| **Node.js Security Best Practices & Releases** | `2026-11-02` | https://nodejs.org/en/learn/getting-started/security-best-practices and https://nodejs.org/en/about/previous-releases |
| **GitHub Actions Security Hardening** | `2026-11-02` | https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions |
| **npm `ci` & `audit` CLI Docs** | `2026-11-02` | https://docs.npmjs.com/cli/commands/npm-ci and https://docs.npmjs.com/cli/commands/npm-audit |
| **OpenSSF Scorecard** | `2026-11-02` | https://scorecard.dev/ |

---

## 6. Identity, OAuth 2.0, OIDC, and OWASP Cheat Sheets (`SRC-IAM-001`)

| Source | Verification Date (`last_verified`) | Official URL |
| :--- | :--- | :--- |
| **OAuth 2.0 Security Best Current Practice (RFC 9700)** | `2026-11-02` | https://www.rfc-editor.org/rfc/rfc9700 |
| **OpenID Connect Core 1.0** | `2026-11-02` | https://openid.net/specs/openid-connect-core-1_0.html |
| **OWASP Authentication Cheat Sheet** | `2026-11-02` | https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html |
| **OWASP Authorization Cheat Sheet** | `2026-11-02` | https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html |
| **OWASP REST Security Cheat Sheet** | `2026-11-02` | https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html |
| **OWASP SSRF Prevention Cheat Sheet** | `2026-11-02` | https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html |

---

## 7. EU and Hungarian Data Protection Sources (`SRC-PRIV-001`)

| Source | Verification Date (`last_verified`) | Official URL |
| :--- | :--- | :--- |
| **GDPR – Regulation (EU) 2016/679** | `2026-11-02` | https://eur-lex.europa.eu/eli/reg/2016/679/oj |
| **EDPB Guidelines 4/2019 on Article 25 (Data Protection by Design and by Default)** | `2026-11-02` | https://www.edpb.europa.eu/our-work-tools/our-documents/guidelines/guidelines-42019-article-25-data-protection-design-and_en |
| **EDPB 2026 Summary of Data Protection by Design and by Default** | `2026-11-02` | https://www.edpb.europa.eu/system/files/2026-02/edpb-summary-gdpr-data-protection-design-default_en.pdf |
| **ENISA (European Union Agency for Cybersecurity)** | `2026-11-02` | https://www.enisa.europa.eu/ |
| **NAIH (Hungarian National Authority for Data Protection and Freedom of Information)** | `2026-11-02` | https://www.naih.hu/ |

---

## 8. Versioning and Maintenance Policy (`SRC-VER-001`)

1. **Primacy of Stable Baselines:** Always use stable, finalized standard releases as the primary normative baseline (e.g., OWASP ASVS 5.0.0, NIST SP 800-218 v1.1 Final, NIST SP 800-63-4 Final, RFC 9700).
2. **Separation of Draft and `latest` Content:** Draft, preview, beta, dev, and `latest` materials may be used for preliminary awareness only; adopt them as production requirements only with explicit technical justification.
3. **Project Freshness (`CORE-FRESH-001`):** Always verify concrete SDK APIs and TypeScript types against the versions pinned in the target project's `package-lock.json`.
4. **Truthful Verification Logging:** During each 90-day or event-driven audit, record the access date (`last_verified`), version, status, and relevant delta changes exclusively for sources that were actually re-verified. Never overwrite verification dates on uninspected sources!