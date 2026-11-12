---
name: security-source-register
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
---

# 10 – Source Register, Szabvány-, Limit- és Dokumentációjegyzék (`SRC-SCOPE-001`)

**Dokumentum felülvizsgálat dátuma (`last_reviewed`):** 2026-11-03  
**Következő kötelező 90 napos felülvizsgálat (`next_full_audit_due`):** 2027-02-01  

**Normatív Forrásfrissességi Szabály (`CORE-FRESH-001`):** A jelen jegyzékben szereplő `last_verified` dátumok kizárólag a ténylegesen ellenőrzött hivatalos dokumentációkat tükrözik. Tilos minden hivatkozásnak automatikusan vagy fiktív módon új ellenőrzési dátumot adni valós felülvizsgálat nélkül! A linkek, kvóták, díjazási feltételek és SDK-verziók idővel változhatnak; projektindítás, architektúra-döntés és 90 napos audit előtt ellenőrizd újra az érintett forrásokat.

---

## 1. Elsődleges webalkalmazás-biztonsági szabványok (`SRC-WEB-001`)

| Forrás | Aktuális státusz az ellenőrzéskor | Ellenőrzés dátuma (`last_verified`) | Használat a keretrendszerben | Hivatalos URL |
| :--- | :--- | :--- | :--- | :--- |
| **OWASP ASVS** | **5.0.0 stabil** (kiadva: 2025-05-30) | `2026-11-02` | Elsődleges webalkalmazás biztonsági követelmény- és verifikációs alap (`01`–`09` modulok) | https://asvs.owasp.org/ és https://owasp.org/www-project-application-security-verification-standard/ |
| **OWASP WSTG** | **4.2 stabil** (5.0 fejlesztés alatt) | `2026-11-02` | Webes biztonsági tesztelési módszertan és tesztesetek (`09_SECURITY_TESTING_THREAT_MODELING.md`) | https://owasp.github.io/www-project-web-security-testing-guide/stable/ és https://owasp.org/projects/web-security-testing-guide |
| **OWASP API Security Top 10** | **2023 kiadás** (stabil) | `2026-11-02` | API-specifikus kockázati kategóriák: BOLA, BFLA, Mass Assignment, Resource Consumption, SSRF (`02`, `05`, `09`) | https://owasp.org/API-Security/ |
| **OWASP Top 10 for LLM Applications** | **2025 kiadás** (stabil) | `2026-11-02` | Prompt injection, sensitive disclosure, excessive agency (`05_API_INTEGRATION_SECURITY.md`, `08_AI_AGENT_CODING_SECURITY.md`) | https://owasp.org/www-project-top-10-for-large-language-model-applications/ és https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/ |
| **OWASP SAMM** | Aktuális stabil kiadás | `2026-11-02` | Szervezeti secure SDLC érettségi referencia (`09`) | https://owaspsamm.org/ |

---

## 2. NIST és kiberbiztonsági keretrendszerek (`SRC-NIST-001`)

| Forrás | Aktuális státusz és Hatókör | Ellenőrzés dátuma (`last_verified`) | Hivatalos URL |
| :--- | :--- | :--- | :--- |
| **NIST CSF 2.0** | Stabil keretrendszer | `2026-11-02` | https://www.nist.gov/cyberframework |
| **NIST SP 800-218 (SSDF 1.1)** | **Final (stabil baseline)** a biztonságos szoftverfejlesztési életciklushoz (`06`, `08`) | `2026-11-02` | https://csrc.nist.gov/pubs/sp/800/218/final |
| **NIST SP 800-218 Rev. 1 (SSDF 1.2)** | 2025-12-17-én *draft* státuszban szerepelt a publikációs listán; **draft, nem stabil baseline!** | `2026-11-02` | https://csrc.nist.gov/projects/ssdf/publications |
| **NIST SP 800-218A** | **Final (2024-07-26)** – AI és dual-use foundation model fejlesztési profil; AI coding agent használatára kiegészítő informatív referencia (`05`, `08`) | `2026-11-02` | https://csrc.nist.gov/pubs/sp/800/218/a/final |
| **NIST SP 800-63-4** | **Final (2025-07)** – Digital Identity Guidelines (felváltotta a 800-63-3 kiadást; `04_IDENTITY_ACCESS_SECURITY.md`) | `2026-11-02` | https://csrc.nist.gov/pubs/sp/800/63/4/final |
| **NIST AI RMF** | AI Risk Management Framework (`08`) | `2026-11-02` | https://www.nist.gov/itl/ai-risk-management-framework |

---

## 3. ISO/IEC szabványok (`SRC-ISO-001`)

A projekt kockázatától és szerződéses környezetétől függően tájékoztató jelleggel:
- **ISO/IEC 27001:2022** – Információbiztonsági irányítási rendszer (ISMS) követelményei.
- **ISO/IEC 27002:2022** – Információbiztonsági kontrollok.
- **ISO/IEC 27034 sorozat** – Alkalmazásbiztonság (Application Security).
- **ISO/IEC 27701** – Adatvédelmi irányítási rendszer (PIMS), aktuális kiadás ellenőrzendő.
- **Hivatalos katalógus:** https://www.iso.org/standards.html (`last_verified: 2026-11-02`)

**Normatív megjegyzés (`CORE-DOCGATE-001`):** Az ISO-szabványok teljes szövege licencelt. Ez a keretrendszer nem idéz vagy reprodukál teljes ISO-követelményeket, és nem állít ISO-megfelelést.

---

## 4. Firebase, Google Cloud és Platformlimitek Hivatalos Forrásai (`SRC-FB-001`, `FIREBASE-LIMIT-001`)

| Téma / Platformlimit | Kategória (`FIREBASE-LIMIT-001`) | Ellenőrzés dátuma (`last_verified`) | Hivatalos Dokumentáció URL |
| :--- | :--- | :--- | :--- |
| **Firebase Security Checklist** | Általános Firebase biztonsági alapelvek | `2026-11-02` | https://firebase.google.com/support/guides/security-checklist |
| **Custom Claims 1 000 byte limit** | `[PLATFORM HARD LIMIT]` – Max. 1 000 byte payload / user | `2026-11-02` | https://firebase.google.com/docs/auth/admin/custom-claims |
| **Firestore Rules `get()` / `exists()` limitek** | `[PLATFORM HARD LIMIT]` – Max. 10 hívás / single request; max. 20 / multi-doc, tx, batch | `2026-11-02` | https://firebase.google.com/docs/firestore/security/rules-structure |
| **Identity Platform Spark (3k DAU) & Blaze (50k MAU)** | `[PLAN QUOTA]` (Spark 3 000 DAU) és `[PRICING / FREE TIER ALLOWANCE]` (Blaze 50 000 MAU) | `2026-11-02` | https://firebase.google.com/pricing |
| **Firebase App Check & Replay Protection (`consumeAppCheckToken`)** | Cloud Functions v2 App Check, `limitedUseAppCheckTokens` és `roles/firebaseappcheck.tokenVerifier` | `2026-11-02` | https://firebase.google.com/docs/app-check/cloud-functions és https://firebase.google.com/docs/app-check |
| **Firestore Security Rules & Secure Queries** | Rules Are Not Filters, logikai `OR` kiértékelés | `2026-11-02` | https://firebase.google.com/docs/firestore/security/get-started és https://firebase.google.com/docs/firestore/security/rules-query |
| **Firestore Offline Persistence & `clearIndexedDbPersistence()`** | Forenzikus és offline adatvesztési korlátok, `terminate(db)`, `memoryLocalCache()` | `2026-11-02` | https://firebase.google.com/docs/firestore/manage-data/enable-offline |
| **Firebase Emulator Suite & Rules Unit Testing** | `@firebase/rules-unit-testing` (`assertSucceeds`, `assertFails`) | `2026-11-02` | https://firebase.google.com/docs/emulator-suite és https://firebase.google.com/docs/rules/unit-tests |
| **Firebase Auth & Multi-Factor Authentication (MFA)** | TOTP MFA, session lifecycle, token revocation | `2026-11-02` | https://firebase.google.com/docs/auth és https://firebase.google.com/docs/auth/web/multi-factor |
| **Cloud Functions for Firebase (2nd Gen / v2)** | `firebase-functions/v2/https` (`onCall`, `onRequest`, `maxInstances`) | `2026-11-02` | https://firebase.google.com/docs/functions |
| **Cloud Storage Security Rules** | Fájlméret, `contentType`, user-scoped path | `2026-11-02` | https://firebase.google.com/docs/storage/security |
| **Firebase Hosting** | Security headers, CSP, target/subdomain szeparáció | `2026-11-02` | https://firebase.google.com/docs/hosting |
| **Google Cloud IAM, Secret Manager, Logging & Monitoring** | Service account least privilege, `defineSecret`, audit logs, alerts | `2026-11-02` | https://cloud.google.com/iam/docs , https://cloud.google.com/secret-manager/docs , https://cloud.google.com/logging/docs , https://cloud.google.com/monitoring/docs |

---

## 5. Frontend, böngésző, nyelv és Node.js runtime (`SRC-FRONT-001`)

| Forrás | Ellenőrzés dátuma (`last_verified`) | Hivatalos URL |
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
| **Node.js Security Best Practices & Releases** | `2026-11-02` | https://nodejs.org/en/learn/getting-started/security-best-practices és https://nodejs.org/en/about/previous-releases |
| **GitHub Actions Security Hardening** | `2026-11-02` | https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions |
| **npm `ci` & `audit` CLI Docs** | `2026-11-02` | https://docs.npmjs.com/cli/commands/npm-ci és https://docs.npmjs.com/cli/commands/npm-audit |
| **OpenSSF Scorecard** | `2026-11-02` | https://scorecard.dev/ |

---

## 6. Identity, OAuth 2.0, OIDC és OWASP Cheat Sheets (`SRC-IAM-001`)

| Forrás | Ellenőrzés dátuma (`last_verified`) | Hivatalos URL |
| :--- | :--- | :--- |
| **OAuth 2.0 Security Best Current Practice (RFC 9700)** | `2026-11-02` | https://www.rfc-editor.org/rfc/rfc9700 |
| **OpenID Connect Core 1.0** | `2026-11-02` | https://openid.net/specs/openid-connect-core-1_0.html |
| **OWASP Authentication Cheat Sheet** | `2026-11-02` | https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html |
| **OWASP Authorization Cheat Sheet** | `2026-11-02` | https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html |
| **OWASP REST Security Cheat Sheet** | `2026-11-02` | https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html |
| **OWASP SSRF Prevention Cheat Sheet** | `2026-11-02` | https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html |

---

## 7. EU és magyar adatvédelmi források (`SRC-PRIV-001`)

| Forrás | Ellenőrzés dátuma (`last_verified`) | Hivatalos URL |
| :--- | :--- | :--- |
| **GDPR – (EU) 2016/679 Rendelet** | `2026-11-02` | https://eur-lex.europa.eu/eli/reg/2016/679/oj |
| **EDPB Guidelines 4/2019 on Article 25 (Data Protection by Design and by Default)** | `2026-11-02` | https://www.edpb.europa.eu/our-work-tools/our-documents/guidelines/guidelines-42019-article-25-data-protection-design-and_en |
| **EDPB 2026 Summary of Data Protection by Design and by Default** | `2026-11-02` | https://www.edpb.europa.eu/system/files/2026-02/edpb-summary-gdpr-data-protection-design-default_en.pdf |
| **ENISA (European Union Agency for Cybersecurity)** | `2026-11-02` | https://www.enisa.europa.eu/ |
| **NAIH (Nemzeti Adatvédelmi és Információszabadság Hatóság)** | `2026-11-02` | https://www.naih.hu/ |

---

## 8. Verziókezelési és karbantartási szabály (`SRC-VER-001`)

1. **Stabil baseline elsőbbsége:** Mindig a stabil, véglegesített szabványkiadás legyen az elsődleges normatív baseline (pl. OWASP ASVS 5.0.0, NIST SP 800-218 v1.1 Final, NIST SP 800-63-4 Final, RFC 9700).
2. **Draft és `latest` elhatárolása:** A draft, preview, beta, dev és `latest` tartalom csak előzetes tájékozódásra használható; production követelményként csak kifejezett szakmai indoklással alkalmazható.
3. **Projektfrissesség (`Project Freshness` – `CORE-FRESH-001`):** A konkrét SDK API-kat és típusokat mindig a célprojekt `package-lock.json` fájljában telepített verziók szerint kell ellenőrizni.
4. **Valós ellenőrzési naplózás:** Új (90 napos vagy eseményvezérelt) auditkor rögzítsd a ténylegesen felülvizsgált források hozzáférési dátumát (`last_verified`), verzióját, státuszát és a releváns delta-változásokat. Nem ellenőrzött forrás dátumát tilos átírni!