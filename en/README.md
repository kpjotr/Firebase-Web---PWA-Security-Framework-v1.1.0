# Firebase Web & PWA Security Framework

**Document Package Version:** 1.1.0  
**Release Date:** 2026-11-03  
**Last Full Review (`last_reviewed`):** 2026-11-03  
**Next Mandatory Full Audit Due (`next_full_audit_due`):** 2027-02-01 (90 calendar days)  
**Language:** English (`EN`) – in 100% mirror sync with the Hungarian (`HU`) edition (`CORE-LANG-001`)  
**Target Platform:** Firebase-based websites, SPAs, and PWAs; Node.js/Cloud Functions (v2); HTML, CSS, JavaScript, TypeScript, React  
**Primary Users:** AI models, AI coding and security reviewing agents, software developers, and security auditors

---

## 1. Purpose of the Package (`CORE-SCOPE-001`)

This package is the modular, multi-agent-ready, context-window-optimized `v1.1.0` evolution of the earlier `universal_firebase_pwa_security_skill_v3_2_2_en.md` (`LEGACY_SOURCE_v3_2_2_EN.md`) and the `1.0.0` framework. The system remains strictly Firebase-centric; it is not intended to be generalized away from Firebase to other backend platforms.

The goal of this package is to ensure that at any stage of development—requirements gathering, architecture design, frontend, backend, external integrations, testing, deployment, and maintenance—the AI agent progressively activates (Progressive Disclosure), enforces, and verifies the security requirements relevant to the task based on verifiable evidence.

---

## 2. Document Map and Loading Hierarchy (`CORE-LOAD-001`)

This framework is **designed for model-, IDE-, and agent-agnostic use**: its normative security requirements are not tied to any specific AI platform, although file access, MCP/Skill integration, and workflow execution may be environment-dependent. The primary goal of the multi-file architecture is context window protection (preventing "Lost in the Middle" attention degradation) and preventing code regression.

| File | Loading Tier | Content | When is loading mandatory? |
| :--- | :--- | :--- | :--- |
| `01_SECURITY_CORE.md` | **Tier-0 (Mandatory Core)** | Central controller: project assessment, L1/L2/L3 tiers, L1 Lightweight Mode, module selection, priority rules, 3-level freshness gate, 3-way multi-agent handoff, Reviewer independence, 6-step file locking, HU/EN sync, documentation release gate | Mandatory to load **first** before **every** security-relevant task (design, analysis, coding, testing, auditing) |
| `08_AI_AGENT_CODING_SECURITY.md` | **Tier-0 (Coding/Modifying Core)** | Prompt injection, untrusted repository files, clarified `HALT & ESCALATE`, tool usage, permissions, multi-agent locking, human-in-the-loop approval | Mandatory for every agent that writes, modifies, or generates code/configuration, executes commands, or proposes concrete code changes *(purely theoretical/architectural analysis agents do not need to load the full coding module)* |
| `02_FIREBASE_SECURITY.md` | **Tier-1 (Selective Domain)** | Firebase Auth, Firestore, Storage, Functions v2, Hosting, App Check, IAM, Rules, PWA persistence, 5-category limit and quota management | In every Firebase project; load sections corresponding to affected components |
| `03_FRONTEND_SECURITY.md` | **Tier-1 (Selective Domain)** | HTML, CSS, JS/TS, React, DOM, browser APIs, PWA, Service Worker, CSP | For every frontend, static site, or PWA task |
| `04_IDENTITY_ACCESS_SECURITY.md` | **Tier-1 (Selective Domain)** | Authentication, MFA, OAuth/OIDC, session lifecycle, tokens, roles, account lifecycle | Whenever login, registration, authorization, roles, or user accounts are involved |
| `05_API_INTEGRATION_SECURITY.md` | **Tier-1 (Selective Domain)** | Callable/HTTP APIs, webhooks, external APIs, SSRF, CORS, validation, abuse/quotas, LLM APIs | For every API endpoint or external integration |
| `06_DEVSECOPS_SUPPLY_CHAIN.md` | **Tier-1 (Selective Domain)** | Git, npm, lockfiles, CI/CD, builds, secrets, dependencies, release pipeline | For every repository, package, build, CI/CD, or deployment task |
| `07_DATA_PROTECTION_PRIVACY.md` | **Tier-1 (Selective Domain)** | GDPR, data minimization, retention, deletion, encryption, logging, data breach handling | Whenever personal data (PII), customer data, analytics, file uploads, or data retention are involved |
| `09_SECURITY_TESTING_THREAT_MODELING.md` | **Tier-1 (Selective Domain)** | Threat modeling, ASVS 5.0.0 verification, test plans, SAST/DAST, emulators, Admin SDK authz tests, pentesting | During design, major changes, pre-release checks, and security audits |
| `10_SOURCE_REGISTER.md` | **Tier-1 (Selective Domain)** | Official and normative sources, verified dates (`last_verified`), version status, platform limits | At project kickoff, architectural decisions, audits, and 90-day reviews |
| `11_REFERENCE_TEMPLATES_FIREBASE.md` | **Tier-2 (Implementation Layer)** | 5 structured Firebase v2 reference implementations (`firestore.rules`, `onCall` with two separate transactions, PWA `BroadcastChannel` + `TAB_ID` logout, Rules unit tests, Admin SDK handler authz tests) with 7-element metadata headers and explicit `Verification Level` | Load **exclusively** when implementing or unit/emulator-testing concrete Firebase code (`firestore.rules`, Cloud Functions v2, PWA logout/persistence) |
| `SECURITY_PROFILE_TEMPLATE.md` | **Template** | Project security profile template (with full L2/L3 and compact L1 Lightweight Mode options) | At project initialization and architectural changes |
| `RELEASE_GATE_TEMPLATE.md` | **Template** | Release security gate and `DOCUMENTATION_RELEASE_GATE` checklist template | Before every release, PR closure, and documentation package release |
| `handoff_contracts.schema.json` | **JSON Schema** | Machine-validatable JSON Schema (Draft 2020-12) for `PLANNER_TO_CODER`, `CODER_TO_REVIEWER`, and `REVIEWER_TO_PLANNER` handoff contracts | During automated or structured validation of multi-agent workflows |
| `LEGACY_SOURCE_v3_2_2_EN.md` | **Legacy Archive** | Historical archive of the earlier single-file v3.2.2 English skill (`DEPRECATED / LEGACY ARCHIVE`) | For historical audit traceability only; use modules `01`–`11` for all active work! |

---

## 3. How to Use

1. Place the complete package inside the project repository or the AI agent's accessible policy directory.
2. The agent MUST always read `01_SECURITY_CORE.md` first and verify the Three-Level Freshness Gate (`CORE-FRESH-001`).
3. Following the Core, the agent performs the project risk assessment and creates (or updates) the project-specific `SECURITY_PROFILE.md` decision record (for L1 apps, the proportional L1 Lightweight Mode – `CORE-TIER-L1-001` – may be selected).
4. Based on the Core's module loading hierarchy (`CORE-LOAD-001`), the agent loads only the modules required for the task:
   - When writing or modifying code/configs, `01_SECURITY_CORE.md` and `08_AI_AGENT_CODING_SECURITY.md` are always mandatory (Tier-0).
   - For standard Firebase web/PWA development, `02_FIREBASE_SECURITY.md` and `03_FRONTEND_SECURITY.md` are active by default (Tier-1).
   - When writing concrete Firebase rules, Cloud Functions v2 endpoints, PWA logout flows, or emulator tests, `11_REFERENCE_TEMPLATES_FIREBASE.md` must also be loaded (Tier-2).
5. In multi-agent workflows (`Planner -> Coder -> Security Reviewer -> Planner`), enforcing the three-way handoff contracts (`CORE-HANDOFF-001`), Reviewer independence (`CORE-REV-001`), and the 6-step exclusive file-locking protocol (`CORE-CONC-001`) is mandatory.
6. Every security requirement must be backed by verifiable evidence: automated test output, configuration, code location, CI result, independent review, or documented human-approved risk acceptance.
7. Prior to release, execute the verification defined in `09_SECURITY_TESTING_THREAT_MODELING.md` and `RELEASE_GATE_TEMPLATE.md`.

---

## 4. Important Limitations and Scope of `DOCUMENTATION_RELEASE_GATE` (`CORE-DOCGATE-001`)

- This framework does not replace a project's specific threat model, legal/privacy assessment, or expert penetration test.
- Referencing a standard does not prove compliance. Compliance claims may only be made through item-by-item verification of applicable requirements backed by concrete evidence.
- Reference code in `11_REFERENCE_TEMPLATES_FIREBASE.md` is an illustrative reference implementation (`REFERENCE / MINTA`) carrying a default `DOCUMENTATION_REVIEWED` status (`REF-VERIFY-001`). It is not universally secure drop-in production code without project adaptation and emulator testing.
- The Firebase Admin SDK bypasses Firestore and Storage Security Rules; server-side access must be protected via IAM, explicit in-code authorization, and dedicated handler-level negative tests (`REF-TEST-ADMIN-001`).
- Standards and SDKs evolve. Verify `10_SOURCE_REGISTER.md` and project lockfile versions (`package-lock.json`) at every major release and every 90 days.

> **IMPORTANT SCOPE LIMITATION (`CORE-DOCGATE-001`):** A `DOCUMENTATION_RELEASE_GATE: PASS` result **solely certifies** that the inspected documentation package satisfies the prescribed documentation completeness, structural, referential, HU/EN synchronization, and internal consistency requirements.
>
> **A `DOCUMENTATION_RELEASE_GATE: PASS` DOES NOT IN ITSELF CERTIFY:**
> 1. the overall security of the Firebase application;
> 2. the correctness of a specific project implementation;
> 3. the completion of a comprehensive security audit;
> 4. the completion of a penetration test;
> 5. production readiness status;
> 6. legal or regulatory compliance certification (e.g., GDPR, ISO, ASVS).

---

## 5. Origin, HU/EN Synchronization, and Version History (`CORE-LANG-001`)

- **Origin:** The earlier Firebase/PWA Security Skill v3.2.2 (`LEGACY_SOURCE_v3_2_2_EN.md`) serves as the technical foundation for the Firebase module and reference templates. While v3.2.2 referenced ASVS 4.0.3, this package updates the primary verification baseline to the stable **OWASP ASVS 5.0.0** release, while elevating the battle-tested 2nd Gen (v2) code templates from v3.2.2 into `11_REFERENCE_TEMPLATES_FIREBASE.md` with strict metadata and verification-level governance (`REF-MOD-001`, `REF-VERIFY-001`).
- **HU/EN Mirror Synchronization (`CORE-LANG-001`):** Every normative requirement, test expectation, and reference template has a unique, language-independent identifier (`ID`). Both the Hungarian (`HU`) and English (`EN`) editions contain the exact same IDs and logical structure. For EU/Hungarian legal and data protection (GDPR/NAIH) concepts, the Hungarian (`HU`) text is canonical; for technical/API names, code templates, HTTP headers, and OWASP/NIST terminology, the English (`EN`) edition and official vendor documentation are authoritative.

---

## 6. Package Files (`v1.1.0`)

- `README.md`
- `01_SECURITY_CORE.md`
- `02_FIREBASE_SECURITY.md`
- `03_FRONTEND_SECURITY.md`
- `04_IDENTITY_ACCESS_SECURITY.md`
- `05_API_INTEGRATION_SECURITY.md`
- `06_DEVSECOPS_SUPPLY_CHAIN.md`
- `07_DATA_PROTECTION_PRIVACY.md`
- `08_AI_AGENT_CODING_SECURITY.md`
- `09_SECURITY_TESTING_THREAT_MODELING.md`
- `10_SOURCE_REGISTER.md`
- `11_REFERENCE_TEMPLATES_FIREBASE.md`
- `SECURITY_PROFILE_TEMPLATE.md`
- `RELEASE_GATE_TEMPLATE.md`
- `handoff_contracts.schema.json`
- `LEGACY_SOURCE_v3_2_2_EN.md` *(historical baseline / `DEPRECATED / LEGACY ARCHIVE`)*