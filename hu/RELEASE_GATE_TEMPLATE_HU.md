# Release Security Gate & Documentation Release Gate (v1.1.0)

**Projekt / verzió / commit (`reviewed_commit_or_diff_id`):**  
**Feladatazonosító (`task_id`):**  
**Dátum:**  
**Tier:** `L1` / `L2` / `L3`  
**Reviewer (AI-modell / Emberi ellenőr):**  
**Ellenőrzés függetlensége (`review_independence` – `CORE-REV-001`):** `INDEPENDENT_MODEL` / `SEPARATE_AGENT_INSTANCE` / `SELF_REVIEWED_L1_ONLY`  
**Kapcsolódó Handoff szerződés:** `REVIEWER_TO_PLANNER`  

---

## 0. L1 Lightweight Zárójelentés Opció (`CORE-TIER-L1-001`)
> *Ha a projekt L1 Lightweight Mode-ban fut, a zárójelentés tömöríthető az alábbi 4 sorra, feltéve, hogy a teszteredmények a repositoryban visszakereshetők:*
> 1. **Módosított fájlok és commit/diff ID:**
> 2. **Érvényesített biztonsági kontrollok (`isOwner`, `hasOnly`, titokvédelem):**
> 3. **Lefuttatott pozitív és negatív (`assertFails`) tesztek és parancsuk:**
> 4. **Státusz (`PASS` / `CONDITIONAL_PASS` / `FAIL` / `NOT_VERIFIED`) és `review_independence`:**

*(L2 és L3 projektek esetén az 1–7. fejezetek teljes kitöltése KÖTELEZŐ!)*

---

## 1. Kód, Fájlzárolás és Függőségek (`06_DEVSECOPS_SUPPLY_CHAIN.md`, `CORE-CONC-001`)
- [ ] Exkluzív fájlzárolás (`TECHNICAL_LOCK` / `COORDINATION_RESERVATION`) betartva, `lock_released: true`
- [ ] Párhuzamos módosítások (merge/rebase) után minden teszt újra lefutott az egyesített kódbázison
- [ ] lint / typecheck / build sikeres
- [ ] unit / integration tesztek sikeresek
- [ ] lockfile (`package-lock.json`) és Node/runtime verzió rögzített és ellenőrzött (`Project Freshness`)
- [ ] dependency / SCA vizsgálat (`npm audit`) triage-olva
- [ ] secret scan tiszta (nincs titok gitben, frontendben vagy logban)
- [ ] SAST eredmény triage-olva

## 2. Firebase és Backend (`02_FIREBASE_SECURITY.md`, `11_REFERENCE_TEMPLATES_FIREBASE.md`)
- [ ] Firestore Rules (`rules_version = '2';`) default deny, pozitív és negatív emulátortesztekkel (`REF-TEST-RULES-001`) ellenőrizve (`EMULATOR_TESTED` / `PROJECT_TESTED`)
- [ ] Storage Rules tesztelt (és L2/L3 esetén szerveroldali magic-byte validáció aktív), ha használatban van
- [ ] Admin SDK authorization külön, handler-szintű negatív tesztekkel (`REF-TEST-ADMIN-001`) tesztelt
- [ ] App Check konfiguráció (és L3 / kritikus L2 esetén `consumeAppCheckToken: true` + kliens `limitedUseAppCheckTokens: true` + `roles/firebaseappcheck.tokenVerifier` IAM szerepkör) ellenőrzött
- [ ] Callable/HTTP authz, Zod `.strict()` validáció, külön tranzakciós rate limit (`FIREBASE-FN-003`) és atomi üzleti tranzakció + `requestHash` idempotencia + append-only `ledger` (`FIREBASE-FN-004`) tesztelt
- [ ] IAM least-privilege service accountok és Secret Manager ellenőrzött
- [ ] Cloud Monitoring riasztások és Budget Alert ellenőrzött

## 3. Frontend és PWA (`03_FRONTEND_SECURITY.md`)
- [ ] XSS / DOM XSS kontrollok ellenőrizve (nincs unsanitized `dangerouslySetInnerHTML` / `innerHTML`)
- [ ] Szolgáltatás-specifikus CSP és HTTP security headers (`X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`, HSTS, Host-Only cookie) ellenőrizve
- [ ] Autholt cache, offline queue és PWA kijelentkezés (`memoryLocalCache()` vagy `TAB_ID`-izolált `BroadcastChannel` + `LOGOUT_TAB_ACK` / `LOGOUT_TAB_ERROR` best-effort cleanup – `REF-PWA-LOGOUT-001`) ellenőrizve
- [ ] Token / PII nincs kliensoldali konzollogban vagy publikus bundle-ben
- [ ] Külső script és SRI / self-hosting döntés dokumentált

## 4. Adatvédelem (`07_DATA_PROTECTION_PRIVACY.md`)
- [ ] Adatkezelési cél, jogalap és retention táblázat ellenőrzött (TTL nincs üzleti/számviteli ledgerre alkalmazva)
- [ ] PII log redaction ellenőrzött
- [ ] Külső adatfeldolgozók és AI/LLM adatminimalizálás ellenőrzött
- [ ] Törlési és backup/restore folyamat összehangolt

## 5. Nyitott findingek (`REVIEWER_TO_PLANNER.findings`)
| ID | Severity (`CRITICAL`/`HIGH`/`MEDIUM`/`LOW`) | Követelmény ID | Fájl | Leírás | Kompenzáló kontroll | Felelős | Határidő | Jóváhagyás |
|---|---|---|---|---|---|---|---|---|

## 6. Tesztbizonyíték és Ellenőrzési Szint (`REF-VERIFY-001`)
- **Sablonok/kód tényleges ellenőrzési szintje (`Verification Level`):** `DOCUMENTATION_REVIEWED` / `STATIC_CHECKED` / `EMULATOR_TESTED` / `PROJECT_TESTED` / `NOT_VERIFIED`
- **Tesztek ténylegesen lefutottak (parancs és eredmény):**
- **Reviewer által önállóan újrafuttatott tesztek (`independently_executed_tests`):**
- **Tesztkörnyezet:**
- **Eszközök / SDK verziók:**
- **Nem futott vagy elhalasztott tesztek (`not_applicable_or_deferred_tests`):**
- **Ismert korlátozások és maradék kockázatok (`residual_risks`):**

## 7. Alkalmazás / Feladat Release Döntés (`CORE-HANDOFF-001`)
- [ ] `PASS` *(Minden kötelező ellenőrzés és pozitív/negatív teszt ténylegesen lefutott és sikeres)*
- [ ] `CONDITIONAL_PASS` *(Csak nem kritikus, dokumentált elem halasztva; kritikus hiba nincs)*
- [ ] `FAIL` *(Kritikus/high hiba nyitott, vagy kötelező negatív teszt hiányzik/megbukott)*
- [ ] `NOT_VERIFIED` *(Tesztek nem futottak le / nincs elégséges bizonyíték)*
- **Indoklás:**
- **Emberi jóváhagyó:**

---

## 8. Dokumentumcsomag-Kiadási Kapu (`DOCUMENTATION_RELEASE_GATE` – `CORE-DOCGATE-001`)

*(Kizárólag a biztonsági keretrendszer dokumentumcsomagjának kiadásakor vagy frissítésekor töltendő ki!)*

> **FONTOS HATÓKÖR-KORLÁTOZÁS (`CORE-DOCGATE-001`):** A `DOCUMENTATION_RELEASE_GATE: PASS` eredmény **kizárólag azt igazolja**, hogy a vizsgált dokumentumcsomag megfelel az előírt dokumentációs teljességi, szerkezeti, hivatkozási, HU/EN szinkronizációs és belső konzisztenciakövetelményeknek.
> **NEM IGAZOLJA:** (1) a teljes Firebase-alkalmazás biztonságát; (2) a konkrét projektimplementáció helyességét; (3) teljes körű biztonsági audit elvégzését; (4) penetrációs teszt elvégzését; (5) production readiness állapotot; (6) jogi vagy szabályozási (pl. GDPR, ISO, ASVS) megfelelőségi tanúsítást.

- [ ] **1. Modul-teljesség:** A `README.md`-ben felsorolt összes modul (`01`–`11`, sablonok, JSON Schema) fizikailag létezik HU és EN nyelven.
- [ ] **2. Hivatkozási integritás:** Minden hivatkozott fájlnév és követelményazonosító ténylegesen létezik a célfájlokban.
- [ ] **3. HU/EN ID-egyezőség:** A magyar és az angol fájlok követelményazonosítói 100%-ban megegyeznek.
- [ ] **4. HU/EN tartalmi és strukturális összhang:** A két nyelvi változat fejezetszámozása, logikai struktúrája és normatív ereje összhangban van.
- [ ] **5. Referencia-kódmodul teljessége és ellenőrzési szintje:** A `11_REFERENCE_TEMPLATES_FIREBASE.md` mind az öt előírt sablont tartalmazza a 7-elemű metaadat-fejléccel és a valós `DOCUMENTATION_REVIEWED` ellenőrzési szinttel.
- [ ] **6. Háromirányú Handoff YAML és JSON Schema érvényesség:** Mindhárom átadási szerződés (`PLANNER_TO_CODER`, `CODER_TO_REVIEWER`, `REVIEWER_TO_PLANNER`) dokumentált és megfelel a JSON Schema definíciónak.
- [ ] **7. Sablon-konzisztencia:** A `SECURITY_PROFILE_TEMPLATE.md` és a `RELEASE_GATE_TEMPLATE.md` megfelel a Core v1.1.0 előírásainak.
- [ ] **8. Legacy v3.2.2 elhatárolás:** A `v3.2.2` legacy forrásfájl egyértelműen történeti előzményként (`DEPRECATED / LEGACY ARCHIVE`) van megjelölve.
- [ ] **9. Forrásjegyzék lefedettség:** A `10_SOURCE_REGISTER.md` minden kritikus technikai állításhoz és platformlimithez tartalmazza a hivatalos URL-t.
- [ ] **10. Valós ellenőrzési dátumok:** A források ellenőrzési dátumai tényleges ellenőrzést tükröznek.
- [ ] **11. Belső kereszthivatkozások:** Nincs hiányzó vagy hibás belső fejezet-/fájlhivatkozás.
- [ ] **12. Csomag-sértetlenség és Hatókör-nyilatkozat:** A `DOCUMENTATION_RELEASE_GATE: PASS` hatókör-korlátozása szerepel a dokumentációban és a zárójelentésben.

**Dokumentációs Kiadási Kapu Eredménye:** `DOCUMENTATION_RELEASE_GATE: PASS` / `FAIL` / `NOT_VERIFIED`