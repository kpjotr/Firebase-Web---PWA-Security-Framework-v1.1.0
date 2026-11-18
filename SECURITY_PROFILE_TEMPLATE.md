# SECURITY_PROFILE.md – Project Security Profile Template (v1.1.0)

**Framework Version:** `1.1.0`  
**Project:**  
**Repository:**  
**Date:**  
**Owner (Project Lead):**  
**Planning / Coding / Reviewing AI Model(s):**  

---

## 0. L1 Lightweight Mode Option (`CORE-TIER-L1-001`)

> **Guidance:** If the project is unambiguously classified as **L1 (Foundational Defense)** based on the Section 4.1 risk assessment (personal or small internal tool, no customer PII, no payments/passes/bookings), you may complete the **Compact L1 Profile (10–15 lines)** below instead of the full L2/L3 template.
> **Mandatory Condition:** Authorization and ownership checks (`isOwner`), secret protection, input validation (`hasOnly`, type/size bounds, immutability), and negative security tests (`assertFails`) MUST NOT be omitted even in L1, and test evidence must remain traceable inside the repository!

### [OPTIONAL] Compact L1 Profile Block (Complete only for L1 projects)
- **Tier & Onboarding:** `L1` (Lightweight Mode) | Onboarding: `A` / `B`
- **Rationale (why L1):** 
- **Affected Collections & Access Rule (`isOwner` + `hasOnly`):** 
- **Client Query Constraints (`where` + `.limit(<=50)`):** 
- **Secret Management & API-Key Restrictions (HTTP Referrer + API scope):** 
- **Cost Defense & Backups (Budget Alert + export):** 
- **Mandatory Negative Tests Location & Result (`assertFails`):** 
- **Project Freshness (`package-lock.json` / SDK versions verified):** 

*(For L2 and L3 projects, completing Sections 1–6 in full is MANDATORY!)*

---

## 1. Technology Profile and Project Freshness (`CORE-FRESH-001`)
- **Firebase project(s) (dev / staging / prod separation):**
- **Firebase Client SDK / Admin SDK versions (per `package-lock.json`):**
- **Cloud Functions runtime and SDK version (v2):**
- **Validation library (e.g., `zod` version):**
- **Frontend framework / build tool:**
- **Node.js runtime (`engines` / `.nvmrc`):**
- **Hosting / domain / subdomain separation:**
- **External APIs and integrations:**
- **CI/CD pipeline:**
- **Project Freshness verification date (`Project Freshness`):**

## 2. Risk Classification (`CORE-ASSESS-001`, `CORE-TIER-001`, `CORE-ONBOARD-001`)
- **Tier:** `L1` / `L2` / `L3`
- **Rationale:**
- **Data Categories (PII, financial, special-category regulated data):**
- **Public Exposure:**
- **Registration (Onboarding) Model:** `A` (Admin-provisioned) / `B` (Invite-only) / `C` (Public signup)
- **Roles and Custom Claims Plan:**
- **Tenant Isolation:**
- **Business-Critical Operations (bookings, passes, balances, inventory):**
- **RTO / RPO Targets:**
- **Client Environment (trusted personal device vs. shared device -> PWA cache decision):**
- **Project-Specific Limits (`[PROJECT-SPECIFIC LIMIT]` – `FIREBASE-LIMIT-001`):**

## 3. Data Protection & Privacy (`07_DATA_PROTECTION_PRIVACY.md`)
- **Data Controller:**
- **Purposes and Legal Bases (GDPR Art. 6 / Art. 9):**
- **Retention Schedule (per data category):**
- **Data Processors and Regions:**
- **Cross-Border Data Transfers (outside EU/EEA):**
- **Data Subject Rights and Cascading Deletion Workflow (Auth, Firestore, Storage, Backups):**
- **DPIA Necessity (per GDPR Art. 35):**

## 4. Activated Modules and Reference Templates (`CORE-LOAD-001`)
- [x] `01_SECURITY_CORE.md` (Tier-0 – Mandatory)
- [ ] `08_AI_AGENT_CODING_SECURITY.md` (Tier-0 – Mandatory when coding/modifying)
- [ ] `02_FIREBASE_SECURITY.md` (Tier-1)
- [ ] `03_FRONTEND_SECURITY.md` (Tier-1)
- [ ] `04_IDENTITY_ACCESS_SECURITY.md` (Tier-1)
- [ ] `05_API_INTEGRATION_SECURITY.md` (Tier-1)
- [ ] `06_DEVSECOPS_SUPPLY_CHAIN.md` (Tier-1)
- [ ] `07_DATA_PROTECTION_PRIVACY.md` (Tier-1)
- [ ] `09_SECURITY_TESTING_THREAT_MODELING.md` (Tier-1)
- [ ] `10_SOURCE_REGISTER.md` (Tier-1)
- [ ] `11_REFERENCE_TEMPLATES_FIREBASE.md` (Tier-2 – Selected templates: `REF-RULES-BASE-001` / `REF-FN-CALLABLE-001` / `REF-PWA-LOGOUT-001` / `REF-TEST-RULES-001` / `REF-TEST-ADMIN-001`)

**Rationale and Omitted Modules:**

## 5. Security Decisions / ADRs and Risk Acceptance (`CORE-RISK-001`)
| ID | Requirement ID | Decision / Deviation | Rationale & Compensating Control | Risk | Human Approver | Review Date |
|---|---|---|---|---|---|---|
| ADR-001 | | | | | | |

## 6. Mandatory Security Evidence (`CORE-EVID-001`)
- **Threat model:**
- **Access matrix (`get`, `list`, `create`, `update`, `delete`):**
- **Firestore & Storage Rules tests (`REF-TEST-RULES-001`):**
- **Cloud Functions & Admin SDK authz tests (`REF-TEST-ADMIN-001`):**
- **Frontend & PWA security tests:**
- **SCA / SAST / secret scan:**
- **Backup / restore evidence:**
- **Release gate record:**