# SECURITY_PROFILE.md – Projektbiztonsági Profil Sablon (v1.1.0)

**Keretrendszer verzió:** `1.1.0`  
**Projekt:**  
**Repository:**  
**Dátum:**  
**Felelős (Projektgazda):**  
**Tervező / Kódoló / Ellenőrző AI-modell(ek):**  

---

## 0. L1 Lightweight Mode Választás (`CORE-TIER-L1-001`)

> **Útmutató:** Ha a projekt a 4.1. kockázati felmérés alapján egyértelműen **L1 (Alapvédelem)** besorolású (személyes vagy szűk belső eszköz, nincs ügyfél-PII, nincs fizetés/bérlet/foglalás), választható az alábbi **Kompakt L1 Profil (10–15 sor)** a teljes L2/L3 sablon helyett.
> **Kötelező feltétel:** A jogosultság- és tulajdonosi ellenőrzés (`isOwner`), a titkok védelme, a bemenet-validáció (`hasOnly`, típus/méret, immutabilitás) és a negatív biztonsági tesztek (`assertFails`) L1-ben SEM hagyhatók el, és a tesztbizonyítékoknak a repositoryban visszakereshetőknek kell lenniük!

### [OPCIONÁLIS] Kompakt L1 Profil Blokkminta (Csak L1 esetén kitöltendő)
- **Tier & Onboarding:** `L1` (Lightweight Mode) | Onboarding: `A` / `B`
- **Indoklás (miért L1):** 
- **Érintett kollekciók és hozzáférési szabály (`isOwner` + `hasOnly`):** 
- **Kliens query korlátok (`where` + `.limit(<=50)`):** 
- **Titokkezelés és API-key korlátozás (HTTP Referrer + API scope):** 
- **Költségvédelem és Mentés (Budget Alert + export):** 
- **Kötelező negatív tesztek helye és eredménye (`assertFails`):** 
- **Projektfrissesség (`package-lock.json` / SDK verziók ellenőrizve):** 

*(L2 és L3 projektek esetén az alábbi 1–6. fejezetek teljes kitöltése KÖTELEZŐ!)*

---

## 1. Technológiai profil és Projektfrissesség (`CORE-FRESH-001`)
- **Firebase project(s) (dev / staging / prod elkülönítés):**
- **Firebase Client SDK / Admin SDK verziók (`package-lock.json` alapján):**
- **Cloud Functions runtime és SDK verzió (v2):**
- **Validációs könyvtár (pl. `zod` verzió):**
- **Frontend framework / build tool:**
- **Node.js runtime (`engines` / `.nvmrc`):**
- **Hosting / domain / aldomain-szeparáció:**
- **Külső API-k és integrációk:**
- **CI/CD pipeline:**
- **Projektfrissesség ellenőrzésének dátuma (`Project Freshness`):**

## 2. Kockázati besorolás (`CORE-ASSESS-001`, `CORE-TIER-001`, `CORE-ONBOARD-001`)
- **Tier:** `L1` / `L2` / `L3`
- **Indoklás:**
- **Adatkategóriák (PII, pénzügyi, különleges kategóriájú adat):**
- **Public exposure:**
- **Regisztrációs (Onboarding) modell:** `A` (Admin-provisioned) / `B` (Invite-only) / `C` (Public signup)
- **Szerepkörök és Custom Claims terv:**
- **Tenant isolation:**
- **Üzleti kritikus műveletek (foglalás, bérlet, egyenleg, készlet):**
- **RTO / RPO követelmény:**
- **Kliens környezet (megbízható saját eszköz vs. megosztott eszköz -> PWA cache döntés):**
- **Projektspecifikus limitek (`[PROJECT-SPECIFIC LIMIT]` – `FIREBASE-LIMIT-001`):**

## 3. Adatvédelem (`07_DATA_PROTECTION_PRIVACY.md`)
- **Adatkezelő:**
- **Célok és jogalapok:**
- **Retention (megőrzési idők adatcsoportonként):**
- **Adatfeldolgozók és régiók:**
- **Adattovábbítás (EU/EGT-n kívülre):**
- **Érintetti jogok és kaszkádolt törlési folyamat (Auth, Firestore, Storage, Backup):**
- **DPIA szükségessége (GDPR 35. cikk alapján):**

## 4. Aktivált modulok és Referencia-Sablonok (`CORE-LOAD-001`)
- [x] `01_SECURITY_CORE.md` (Tier-0 – Kötelező)
- [ ] `08_AI_AGENT_CODING_SECURITY.md` (Tier-0 – Kódolásnál/módosításnál kötelező)
- [ ] `02_FIREBASE_SECURITY.md` (Tier-1)
- [ ] `03_FRONTEND_SECURITY.md` (Tier-1)
- [ ] `04_IDENTITY_ACCESS_SECURITY.md` (Tier-1)
- [ ] `05_API_INTEGRATION_SECURITY.md` (Tier-1)
- [ ] `06_DEVSECOPS_SUPPLY_CHAIN.md` (Tier-1)
- [ ] `07_DATA_PROTECTION_PRIVACY.md` (Tier-1)
- [ ] `09_SECURITY_TESTING_THREAT_MODELING.md` (Tier-1)
- [ ] `10_SOURCE_REGISTER.md` (Tier-1)
- [ ] `11_REFERENCE_TEMPLATES_FIREBASE.md` (Tier-2 – Kiválasztott sablonok: `REF-RULES-BASE-001` / `REF-FN-CALLABLE-001` / `REF-PWA-LOGOUT-001` / `REF-TEST-RULES-001` / `REF-TEST-ADMIN-001`)

**Indoklás és kihagyott modulok:**

## 5. Biztonsági döntések / ADR és Kockázatelfogadás (`CORE-RISK-001`)
| ID | Követelmény ID | Döntés / Eltérés | Indok és Kompenzáló Kontroll | Kockázat | Emberi Jóváhagyó | Felülvizsgálat Dátuma |
|---|---|---|---|---|---|---|
| ADR-001 | | | | | | |

## 6. Kötelező bizonyítékok (`CORE-EVID-001`)
- **Threat model:**
- **Access matrix (`get`, `list`, `create`, `update`, `delete`):**
- **Firestore & Storage Rules tests (`REF-TEST-RULES-001`):**
- **Cloud Functions & Admin SDK authz tests (`REF-TEST-ADMIN-001`):**
- **Frontend & PWA security tests:**
- **SCA / SAST / secret scan:**
- **Backup / restore evidence:**
- **Release gate jegyzőkönyv:**