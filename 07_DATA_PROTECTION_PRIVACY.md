---
name: data-protection-privacy
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
jurisdiction: EU / Hungary
canonical_language_note: "CORE-LANG-001: For EU/Hungarian statutory and data protection (GDPR/NAIH) concepts, the Hungarian (HU) text is canonical."
---

# 07 – Data Protection & Privacy

## 1. Scope, Legal Framework, and Canonical Language (`PRIV-SCOPE-001`)

This module (`Tier-1` selective domain module under `CORE-LOAD-001`) is **mandatory** for every project that processes personal data (PII) or collects technical telemetry (e.g., IP addresses, device identifiers, session metadata) capable of being linked to a natural person. For operations within the EU and Hungary, the primary legal framework is **Regulation (EU) 2016/679 (GDPR)**, supplemented by **EDPB** guidelines and applicable Hungarian law (including **NAIH** regulatory practice).

- **Canonical Language Rule (`CORE-LANG-001`):** For EU and Hungarian statutory and data protection (GDPR/NAIH) terminology, the Hungarian (`HU`) edition of this document is canonical.
- **Legal Disclaimer (`CORE-DOCGATE-001`):** This module provides technical and software-architectural implementation guidance; it does not constitute legal advice or formal GDPR compliance certification. Finalizing legal bases, controller/processor roles, retention schedules, and privacy notices requires a Data Protection Officer (DPO) or legal counsel where appropriate.

---

## 2. Data Inventory and Data Flows (`PRIV-INV-001`)

In every project handling personal data, record (within `SECURITY_PROFILE.md` or a dedicated data inventory):
- data fields and data categories processed;
- the specific purpose and GDPR Article 6 (or Article 9 for special-category data) legal basis;
- categories of data subjects (e.g., customers, staff, administrators, children);
- data sources and recipients;
- Firebase / Google Cloud storage locations and regions (e.g., `europe-west1` / `eur3`);
- roles and server-side Service Accounts with access;
- retention periods and concrete deletion/anonymization mechanisms;
- transfers to external processors/vendors;
- applied security controls and access logging.

On the data-flow diagram, map the browser/PWA, Firebase Auth, Firestore, Cloud Storage, Cloud Functions, external APIs, analytics services, and administrative tools.

---

## 3. Enforcing GDPR Principles in Software Architecture (`PRIV-GDPR-001`)

Embed the principles of GDPR Article 5 directly into the system architecture and data model:
1. **Lawfulness, fairness and transparency**.
2. **Purpose limitation**.
3. **Data minimisation**.
4. **Accuracy**.
5. **Storage limitation**.
6. **Integrity and confidentiality**.
7. **Accountability**.

Never collect data fields "just in case they are useful later." Every Firestore field and Storage path must have a documented purpose, legal basis, and retention period.

---

## 4. Data Protection by Design and by Default (`PRIV-PBD-001`)

Per GDPR Article 25 and EDPB Guidelines 4/2019:
- Collect and store the minimum amount of personal data by default (`CORE-PRIN-011`).
- Configure new user profiles and records to be private by default with minimal privileges (`FIREBASE-AUTH-001`).
- Activate non-essential analytics or marketing trackers only on a valid legal basis and (where required) after prior, affirmative consent.
- Make privacy settings clear and ensure consent can be withdrawn as easily as it was given.
- **Never use live customer data** during development, emulator testing, or CI/CD; use synthetic or irreversibly anonymized test fixtures!
- Process special-category personal data (e.g., health data under GDPR Art. 9) only with an explicit legal basis, elevated security tier evaluation (L2/L3 assessment per `CORE-TIER-001`), and appropriate safeguards.
- Evaluate the necessity of a **Data Protection Impact Assessment (DPIA – GDPR Art. 35)** in `SECURITY_PROFILE.md` based on actual risk factors; never automatically claim either that every project requires a DPIA or that none do.

---

## 5. Data Security and Logging Prohibitions (`PRIV-SEC-001`)

- Enforce TLS/HTTPS for all network transit; alongside provider encryption at rest, apply application-layer envelope encryption (e.g., Google Cloud KMS – `FIREBASE-STORAGE-001`) for sensitive tokens or special-category data.
- Manage encryption keys via Secret Manager / Cloud KMS with key rotation and access logging.
- **PII Logging Prohibition (`FIREBASE-IAM-001`):** Personal data (names, emails, phone numbers, physical addresses, health notes), passwords, session identifiers, or tokens **must never appear** in application logs (`Cloud Logging`), browser consoles, crash reports, analytics events, or AI prompts! When logging errors, record only internal technical identifiers (`uid` – where required for security auditing) and filtered error codes (`errorCode`).
- Log all bulk data exports and administrative queries.
- Treat access to backups as full personal data access subject to strict IAM controls.

---

## 6. Data Subject Rights, Cascading Deletion, and PWA Limitations (`PRIV-RIGHTS-001`)

Depending on the applicable legal basis and statutory retention exemptions, the system must support requests for access, rectification, erasure ("right to be forgotten"), restriction of processing, data portability, and objection:
- **Cascading Deletion vs. Statutory Retention:** Upon account deletion, inspect all data stores: Firebase Auth, Firestore collections, Cloud Storage files, external email/marketing providers, and caches. However, isolate and restrict access to transaction/invoicing records that must be retained by law (e.g., accounting regulations), deleting or anonymizing only non-mandatory personal profile fields.
- **Aligning Backups and Deletion:** Deleting records from the live database does not immediately overwrite archived backups or PITR snapshots. Document backup retention windows and the procedure for re-applying pending deletion requests following a disaster recovery restore.
- **Anonymization vs. Pseudonymization:** Pseudonymized data (e.g., linked to a `uid` or hash) remains personal data under the GDPR! Data is anonymized only if re-identifying the natural person is reasonably impossible by any means.
- **PWA Cache Realities (`FIREBASE-PWA-001`):** Client-side `clearIndexedDbPersistence(db)` and cache cleanup (`REF-PWA-LOGOUT-001`) **do not constitute forensic data erasure**. For shared devices and sensitive PII, use `memoryLocalCache()` instead of persistent client caching.

---

## 7. Data Retention and the Firestore TTL Rule (`PRIV-RET-001`)

Define a retention schedule in `SECURITY_PROFILE.md` covering every data group:
- active user accounts and profile records;
- inactive accounts (automated notification and deletion cycle);
- transactional, pass, booking, and accounting records (`passLedger`, invoices);
- security and audit logs (`Cloud Logging`);
- backups and exports;
- marketing and analytics data;
- customer support communications.

**Normative Firestore TTL Prohibition (`FIREBASE-IDEMP-001`):** Firestore TTL (Time-to-Live) automatic deletion is a technical cleanup utility (for `_idempotency` and `_rateLimits` collections), not a standalone data retention policy! It is **STRICTLY PROHIBITED (`MUST NOT`)** to configure Firestore TTL on business, financial, or legally required audit ledgers (`passLedger`, billing records) whose retention is mandated by law or business accountability.

---

## 8. Analytics, Cookies, and Client-Side Trackers (`PRIV-COOKIE-001`)

- Evaluate cookies, `localStorage`, SDK identifiers, QR/marketing funnels, and analytics events as complete data flows, not merely as a "cookie banner" checkbox.
- Document the purpose, lifetime, and any third-party data sharing for every client storage item.
- Where consent is required, it must be prior, freely given, specific, informed, unambiguous, and revocable at any time.
- **Prohibition of False Claims:** Never state in a privacy notice or documentation that a site "collects no personal or technical data" when Firebase Hosting, Cloud Functions, Cloud Logging, App Check (reCAPTCHA), or external assets process IP addresses and request metadata!
- Even "cookie-free" first-party analytics is not automatically exempt from needing a legal basis; evaluate the actual data processed (e.g., hashed identifiers, referral/source parameters) under GDPR Article 6 (e.g., legitimate interest vs. consent).

---

## 9. Personal Data Breach Management (`PRIV-INC-001`)

- Maintain a documented process for detecting, containing, and escalating security and personal data breaches.
- Preserve forensic evidence and logs (`Cloud Logging`) during an incident while minimizing further copying of compromised personal data.
- Assess whether a personal data breach under GDPR Article 4(12) has occurred (breach of confidentiality, integrity, or availability), which data subjects and categories are affected, and the risk to natural persons' rights and freedoms.
- Fulfill supervisory authority notification (in Hungary, reporting to **NAIH within 72 hours** of becoming aware – GDPR Art. 33) and high-risk data subject notification (GDPR Art. 34) based on actual facts and human/legal decision-making; an AI agent must never autonomously submit regulatory notifications!
- Perform a root cause analysis after every incident and update security controls accordingly.

---

## 10. External Data Processors and AI Providers (`PRIV-PROC-001`)

- For every external vendor (Google Cloud/Firebase, email provider, payment gateway, storage/backup server, AI/LLM provider), verify processor status, Data Processing Agreements (DPAs), sub-processors, data center regions (EU/EEA), cross-border transfer safeguards, and retention terms.
- Send only minimal, strictly necessary data to AI / LLM services (`API-LLM-001`).
- Verify the AI provider's data usage (opt-out of model training!), retention, and data residency terms.
- Redact or pseudonymize personal data in prompts and uploaded files wherever feasible.

---

## 11. Privacy Checklist (`PRIV-CHK-001`)

- [ ] data inventory and data-flow diagram completed in `SECURITY_PROFILE.md` (`PRIV-INV-001`);
- [ ] purpose, legal basis, and retention period defined for every data group (`PRIV-GDPR-001`, `PRIV-RET-001`);
- [ ] data minimization and private defaults enforced (`PRIV-PBD-001`);
- [ ] Firestore TTL is not configured on business/accounting `ledger` collections (`PRIV-RET-001`);
- [ ] data processors, regions, and cross-border transfers reviewed (`PRIV-PROC-001`);
- [ ] data subject rights and cascading deletion + backup workflows aligned (`PRIV-RIGHTS-001`);
- [ ] PII redaction verified across logs, analytics, and AI prompts (`PRIV-SEC-001`);
- [ ] DPIA necessity evaluated (`PRIV-PBD-001`);
- [ ] breach response and NAIH escalation workflow established (`PRIV-INC-001`);
- [ ] privacy notice and consent flows reviewed by a human/legal owner (`PRIV-COOKIE-001`).

---

## 12. Official Sources (`PRIV-REF-001`)

For exact version status and verification dates (`last_verified`), see `10_SOURCE_REGISTER.md`:
- GDPR (EU) 2016/679: https://eur-lex.europa.eu/eli/reg/2016/679/oj
- EDPB Guidelines 4/2019 on Article 25 (Data Protection by Design and by Default): https://www.edpb.europa.eu/our-work-tools/our-documents/guidelines/guidelines-42019-article-25-data-protection-design-and_en
- EDPB 2026 Summary of Data Protection by Design and by Default: https://www.edpb.europa.eu/system/files/2026-02/edpb-summary-gdpr-data-protection-design-default_en.pdf
- ENISA: https://www.enisa.europa.eu/
- Hungarian National Authority for Data Protection and Freedom of Information (NAIH): https://www.naih.hu/