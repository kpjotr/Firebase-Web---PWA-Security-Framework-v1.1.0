---
name: security-testing-threat-modeling
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
---

# 09 – Security Testing & Threat Modeling

## 1. Hatókör és aktiválás (`TEST-SCOPE-001`)

Ez a modul (`Tier-1` szelektív doménmodul a `CORE-LOAD-001` szerint) **kötelező**:
- projekttervezéskor és architektúra-kialakításkor;
- új adatfolyam, szerepkör, API-végpont, fájlfeltöltés, hitelesítési mód vagy külső integráció bevezetésekor;
- jelentős architektúra- vagy jogosultságváltozáskor;
- minden L2 és L3 production release előtt (és L1-ben a jogosultsági/Rules tesztekhez a `CORE-TIER-L1-001` szerint);
- biztonsági incidens vagy új sérülékenység után;
- az időszakos (90 napos) biztonsági audit részeként (`CORE-FRESH-001`).

Konkrét Firebase Security Rules és Cloud Functions (Admin SDK) tesztek írásakor kötelező betölteni a `11_REFERENCE_TEMPLATES_FIREBASE.md` modul `REF-TEST-RULES-001` és `REF-TEST-ADMIN-001` sablonjait is.

---

## 2. Fenyegetésmodellezés – Threat Modeling (`TEST-THREAT-001`, `TEST-STRIDE-001`)

### 2.1. Bizalmi határok (Trust Boundaries) és komponensek (`TEST-THREAT-001`)
Térképezd fel és rögzítsd a `SECURITY_PROFILE.md`-ben:
- a felhasználói szerepköröket (anonim, regisztrált ügyfél, staff, admin) és a tenant-határokat;
- a böngésző/PWA kliensoldali környezetet (Zero Trust Client – `CORE-PRIN-005`);
- a Firebase Authentication / Identity Platform határt;
- a Firestore és Cloud Storage közvetlen klienselérési határt (`firestore.rules`, `storage.rules`);
- a Cloud Functions v2 szerveroldali végrehajtási határt (amely Admin SDK-val megkerüli a Rules-t – `FIREBASE-ADMIN-001`);
- a Google Cloud Service Account / IAM és Secret Manager határt;
- a külső API-kat, fizetési webhookokat, OAuth-szolgáltatókat és AI/LLM végpontokat;
- a CI/CD pipeline-t, a git repositoryt és az AI-kódoló ágensek munkakörnyezetét (`08_AI_AGENT_CODING_SECURITY.md`).

### 2.2. STRIDE és OWASP fenyegetési kategóriák (`TEST-STRIDE-001`)
Minden kritikus adatfolyamot és végpontot vizsgálj meg a **STRIDE** modell szerint:
- **Spoofing (Megszemélyesítés):** identitás, e-mail-verifikáció, App Check token vagy webhook-aláírás hamisítása;
- **Tampering (Manipuláció):** adat, tulajdonosi mező (`userId`), egyenleg, bérlet vagy idempotencia-paraméter (`requestHash`) jogosulatlan módosítása;
- **Repudiation (Letagadhatóság):** kritikus egyenleg-, bérlet- vagy jogosultságmódosítás nyomon követhetetlen végrehajtása (ezért kötelező az append-only `passLedger` és az audit log);
- **Information Disclosure (Adatszivárgás):** BOLA/IDOR olvasás, szűrő nélküli lekérdezés, kliensoldali PWA cache-szivárgás megosztott eszközön, stack trace vagy PII naplózása;
- **Denial of Service / Denial of Wallet:** korlátlan lekérdezés, hiányzó rate limit, konkurens dupla foglalás (race condition), költséges Cloud Functions vagy LLM hívások elárasztása;
- **Elevation of Privilege (Jogosultságemelés):** Mass Assignment (`isAdmin: true`, `role: 'admin'`), BFLA admin végponton, vagy Admin SDK jogosultság-ellenőrzés hiánya.

Kiegészítésként alkalmazd az **OWASP ASVS 5.0.0**, az **OWASP API Security Top 10 (2023)** és az **OWASP Top 10 for LLM Applications (2025)** kategóriáit.

---

## 3. Biztonsági követelmény- és tesztmátrix (`TEST-MATRIX-001`)

Minden L2 és L3 kritikus követelményhez rögzíts követhetőségi mátrixot:
- `Requirement ID` (pl. `FIREBASE-RULES-002`, `FIREBASE-FN-003`, `FIREBASE-FN-004`);
- modul és normatív forrás;
- érintett komponens és fájl;
- kivédett fenyegetés;
- implementált kontroll;
- pozitív teszt hivatkozása;
- negatív teszt (`assertFails` / `HttpsError`) hivatkozása;
- tényleges ellenőrzési szint (`Verification Level` – `REF-VERIFY-001`) és teszteredmény;
- felelős és fennmaradó (residual) kockázat.

---

## 4. Firebase-specifikus biztonsági tesztelés (`TEST-FB-001`, `TEST-NEG-001`)

### 4.1. Kettős Firebase tesztelési kötelezettség (`TEST-FB-001`)
1. **Firestore és Storage Security Rules egységtesztek (`REF-TEST-RULES-001`):** Firebase Local Emulator Suite + `@firebase/rules-unit-testing` segítségével. A Firebase Console *Rules Playground* szimulációja **nem helyettesíti** a verziókövetett, CI-ben futtatható automatizált emulátorteszteket!
2. **Szerveroldali Admin SDK Handler Autorizációs és Tranzakciós tesztek (`REF-TEST-ADMIN-001`):** Mivel a Firebase Admin SDK teljes egészében megkerüli a Security Rules szabályokat (`FIREBASE-ADMIN-001`), minden Cloud Functions v2 (`onCall`/`onRequest`) handlert külön, közvetlen negatív autorizációs, validációs, rate-limit és idempotencia tesztekkel kell ellenőrizni az emulátor ellen!
- A tesztkörnyezet mindig külön emulátor vagy izolált demo projekt (`demo-*`) legyen; **tilos production adatbázist vagy éles ügyféladatot használni teszteléshez!**

### 4.2. Kötelező negatív tesztpéldák (`TEST-NEG-001`)
Egyetlen jogosultsági vagy üzleti funkció sem kaphat `PASS` minősítést, ha hiányoznak a releváns negatív (`assertFails` / elutasítási) tesztek:
- [ ] anonim (hitelesítetlen) felhasználó hozzáférési kísérlete;
- [ ] nem verifikált e-mail című (`email_verified: false`) felhasználó kísérlete;
- [ ] másik felhasználó dokumentumának olvasása vagy módosítása (BOLA / IDOR);
- [ ] másik tenant dokumentumának elérése;
- [ ] a megváltoztathatatlan `userId`, `tenantId` vagy `createdAt` mező átírási kísérlete `update` során;
- [ ] nem engedélyezett extra mező (`isAdmin: true`, `role: 'admin'`) beszúrása (`hasOnly` és `Zod .strict()` teszt);
- [ ] érvénytelen adattípus vagy tartományon kívüli érték (pl. negatív időtartam/összeg);
- [ ] `list` lekérdezés hiányzó tulajdonosi (`where('userId', '==', uid)`) szűrővel (`unfilteredQuery` -> `assertFails`);
- [ ] `list` lekérdezés hiányzó `.limit()` záradékkal (`noLimitQuery` -> `assertFails`) vagy a megengedett limitet (`50`) túllépő értékkel (`excessiveQuery` -> `assertFails`);
- [ ] közvetlen kliensoldali írási kísérlet zárolt kritikus kollekcióra (`passes`, `passLedger`, `bookings` -> `assertFails`);
- [ ] normál felhasználó által meghívott adminisztratív Cloud Function (BFLA);
- [ ] rate limit túllépése (`resource-exhausted`), valamint annak ellenőrzése, hogy az 1. (rate-limit) tranzakcióban növelt számláló akkor is megmarad, ha a 2. (üzleti) tranzakció üzleti hiba miatt elbukik (`FIREBASE-FN-003`, `REF-TEST-ADMIN-001`);
- [ ] azonos `idempotencyKey` újraküldése eltérő/manipulált bemeneti paraméterekkel (`requestHash` mismatch -> `already-exists` elutasítás – `FIREBASE-FN-004`, `REF-TEST-ADMIN-001`);
- [ ] hamis aláírású vagy újrajátszott (replayed) webhook elutasítása;
- [ ] túlméretes vagy hamis MIME/magic-byte-tal rendelkező Storage fájlfeltöltés elutasítása.

---

## 5. Frontend és PWA biztonsági tesztelés (`TEST-FRONT-001`)

- DOM XSS, reflected és stored XSS tesztelés minden dinamikus megjelenítési ponton.
- URL scheme injection (`javascript:`, `data:`) és open redirect tesztelés.
- Szolgáltatás-specifikus CSP és HTTP biztonsági fejlécek (`firebase.json`) ellenőrzése.
- `iframe` sandbox, clickjacking és `postMessage` origin-validáció tesztelése.
- **PWA kijelentkezés és cache-izoláció tesztelése (`REF-PWA-LOGOUT-001`):**
  - Többtabos `BroadcastChannel` + `TAB_ID` kijelentkezés tesztelése (self-echo kizárása, `LOGOUT_TAB_ACK` / `LOGOUT_TAB_ERROR` és timeout működése);
  - Service Worker cache vizsgálata (hitelesített válasz vagy PII nem maradhat a Cache Storage-ban);
  - `localStorage`, `sessionStorage` és `IndexedDB` ellenőrzése kijelentkezés után.

---

## 6. Tesztelési rétegek és a 4+1 Ellenőrzési Szint (`TEST-LAYERS-001`, `REF-VERIFY-001`)

### 6.1. A 7 tesztelési réteg (`TEST-LAYERS-001`)
1. **Unit test:** séma-validáció (`Zod .strict()`), üzleti invariánsok, segédfüggvények.
2. **Emulator / Integration test:** Firebase Local Emulator Suite (`firestore.rules`, `storage.rules`, Cloud Functions v2 Admin SDK handlerek, Auth).
3. **E2E test:** böngészőn keresztüli teljes folyamatok különböző szerepkörökkel és több böngészőfüllel.
4. **SAST / SCA / Secret scan:** statikus kódelemzés, függőség- és titokvizsgálat.
5. **DAST:** staging környezet végpontjainak és fejléceinek dinamikus vizsgálata.
6. **Manual Security Review (`CORE-REV-001`):** üzleti logika, jogosultsági határok, race condition és abuse case-ek független felülvizsgálata.
7. **Penetration test:** L3 vagy magas kockázatú, illetve szabályozott rendszereknél független szakértői penetrációs teszt.

*(Az automatizált szkennerek soha nem helyettesítik az üzleti logika és az autorizáció célzott vizsgálatát!)*

### 6.2. Az Ellenőrzési Szintek Kötelező Jelölése (`REF-VERIFY-001`)
Minden kódsablonnál, tesztjelentésben és handoff-szerződésben kötelező valósan megjelölni a ténylegesen elvégzett ellenőrzési szintet:
- **`DOCUMENTATION_REVIEWED`:** Csak dokumentációs/logikai áttekintés történt, kód nem futott (a `11_REFERENCE_TEMPLATES_FIREBASE.md` sablonjainak alapállapota).
- **`STATIC_CHECKED`:** Típusellenőrzés (`tsc`) és linter lefutott, de futásidejű teszt nem.
- **`EMULATOR_TESTED`:** A Firebase Emulator Suite-ban a pozitív és negatív tesztek ténylegesen lefutottak és sikeresek voltak.
- **`PROJECT_TESTED`:** A célprojekt teljes környezetével (CI/staging) ellenőrizve.
- **`NOT_VERIFIED`:** Nem történt ellenőrzés.

---

## 7. OWASP ASVS 5.0.0 alkalmazása (`TEST-ASVS-001`)

- Az **OWASP ASVS 5.0.0** stabil kiadás a keretrendszer elsődleges webalkalmazás-verifikációs referenciája (`10_SOURCE_REGISTER.md`).
- A projektnek a `SECURITY_PROFILE.md`-ben kell meghatároznia a célzott ASVS szintet és az alkalmazandó követelményeket.
- **Tilos alaptalan megfelelőségi állítást tenni:** Ne állítsd, hogy egy projekt „OWASP ASVS compliant”, ha nem történt követelményenkénti tételes verifikáció és bizonyítékrögzítés.

---

## 8. Release Gate, Reviewer Függetlenség és Dokumentációs Kapu (`TEST-GATE-001`, `CORE-REV-001`, `CORE-DOCGATE-001`)

Használd a `RELEASE_GATE_TEMPLATE.md` sablont és a `REVIEWER_TO_PLANNER` handoff-szerződést (`CORE-HANDOFF-001`):

1. **Alkalmazás Release `FAIL` (kiadás megtagadása), ha:**
   - bármilyen `CRITICAL` vagy `HIGH` súlyosságú, kihasználható biztonsági hiba nyitott;
   - a Firestore/Storage Rules vagy a szerveroldali Admin SDK authz negatív tesztek nem futottak le vagy megbuktak;
   - titok (secret) került a git repositoryba, a frontend bundle-be vagy a logokba;
   - kritikus üzleti végpont szerveroldali autorizáció, tranzakció vagy idempotencia nélkül működik;
   - párhuzamos módosítások (merge/rebase) után nem futtatták újra a biztonsági teszteket az egyesített kódbázison (`CORE-CONC-001`);
   - biztonsági tesztet vagy kontrollt magyarázat és emberi jóváhagyás nélkül kikapcsoltak.
2. **Security Reviewer függetlensége (`CORE-REV-001`):** A Reviewer külön fázisban dolgozik, önállóan ellenőrzi a `git diff`-et és a tesztkimeneteket (`independently_executed_tests`), L2/L3 szinten nem alkalmazhat önjóváhagyást, és rögzíti a `review_independence` (`INDEPENDENT_MODEL` / `SEPARATE_AGENT_INSTANCE` / `SELF_REVIEWED_L1_ONLY`) értéket.
3. **A `DOCUMENTATION_RELEASE_GATE: PASS` hatókör-korlátozása (`CORE-DOCGATE-001`):** A dokumentumcsomagra kiadott `DOCUMENTATION_RELEASE_GATE: PASS` kizárólag a dokumentáció teljességét, hivatkozási integritását és HU/EN szinkronját igazolja; **nem igazolja** az alkalmazás biztonságát, a projektimplementáció helyességét, teljes körű biztonsági audit vagy penetrációs teszt elvégzését, a production readiness állapotot vagy a jogi/szabályozási megfelelést!

---

## 9. Auditjelentés minimális szerkezete (`TEST-AUDIT-001`)

Minden biztonsági felülvizsgálati vagy auditjelentésnek tartalmaznia kell:
1. Hatókör (`scope`), vizsgált fájlok (`inspected_files`) és környezet;
2. Vizsgált verziók és pontos git commit/diff azonosító (`reviewed_commit_or_diff_id`);
3. Alkalmazott szabványok, betöltött modulok (`01`–`11`) és módszertan;
4. Ténylegesen lefuttatott tesztek (`independently_executed_tests`), eszközök és elhalasztott tesztek;
5. Feltárt hibák (`findings`) súlyosság (`CRITICAL`, `HIGH`, `MEDIUM`, `LOW`) és követelményazonosító (`requirement_id`) szerint;
6. Reprodukciós lépések / támadási forgatókönyv és üzleti hatás;
7. Konkrét javítási feladatok (`rework_tasks`) és javasolt kód;
8. Retest (újratesztelés) eredménye;
9. Vizsgálati korlátozások és fennmaradó kockázatok (`residual_risks`);
10. Az ellenőrzés típusa (`Verification Level`: `DOCUMENTATION_REVIEWED` / `STATIC_CHECKED` / `EMULATOR_TESTED` / `PROJECT_TESTED`), a Reviewer függetlensége (`review_independence`) és a végső státusz (`PASS` / `CONDITIONAL_PASS` / `FAIL` / `NOT_VERIFIED`).

---

## 10. Hivatalos források (`TEST-REF-001`)

A pontos verzióstátuszt és ellenőrzési dátumokat (`last_verified`) lásd a `10_SOURCE_REGISTER.md` modulban:
- OWASP Application Security Verification Standard (ASVS 5.0.0): https://asvs.owasp.org/
- OWASP Web Security Testing Guide (WSTG v4.2 Stable): https://owasp.github.io/www-project-web-security-testing-guide/stable/
- OWASP WSTG Project: https://owasp.org/projects/web-security-testing-guide
- Firebase Local Emulator Suite: https://firebase.google.com/docs/emulator-suite
- Firebase Security Rules Unit Testing: https://firebase.google.com/docs/rules/unit-tests
- OWASP SAMM: https://owaspsamm.org/