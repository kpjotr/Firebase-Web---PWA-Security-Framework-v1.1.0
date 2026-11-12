# Firebase Web & PWA Security Framework

**Dokumentumcsomag:** 1.1.0  
**Kiadás dátuma:** 2026-11-03  
**Utolsó teljes felülvizsgálat (`last_reviewed`):** 2026-11-03  
**Következő kötelező felülvizsgálat (`next_full_audit_due`):** 2027-02-01 (90 naptári nap)  
**Nyelv:** magyar (`HU`) – tükörszinkronban az angol (`EN`) változattal (`CORE-LANG-001`)  
**Célplatform:** Firebase-alapú weboldalak, SPA-k és PWA-k; Node.js/Cloud Functions (v2); HTML, CSS, JavaScript, TypeScript, React  
**Elsődleges felhasználók:** AI-modellek, AI-kódoló és biztonsági felülvizsgáló ágensek, fejlesztők és biztonsági auditorok

---

## 1. A csomag rendeltetése (`CORE-SCOPE-001`)

Ez a csomag a korábbi `universal_firebase_pwa_security_skill_v3_2_2_en.md` (`LEGACY_SOURCE_v3_2_2_EN.md`) és az `1.0.0` keretrendszer továbbfejlesztett, moduláris, többügynökös (multi-agent) munkafolyamatokra és kontextusablak-védelemre optimalizált `v1.1.0` kiadása. A rendszer Firebase-központú marad; nem cél a Firebase-ről más backendplatformra való általánosítás.

A csomag célja, hogy a fejlesztés bármely fázisában – követelményfelmérés, architektúra, frontend, backend, integráció, tesztelés, üzembe helyezés és karbantartás – az AI a feladathoz szükséges biztonsági követelményeket progresszíven (Progressive Disclosure) aktiválja, végrehajtsa és bizonyítékalapon ellenőrizze.

---

## 2. Dokumentumtérkép és Betöltési Hierarchia (`CORE-LOAD-001`)

A keretrendszer **modell-, IDE- és ügynökfüggetlen használatra tervezett**: a normatív biztonsági követelmények nem kötődnek konkrét AI-platformhoz, azonban a fájlhozzáférés, az MCP/Skill-integráció és a munkafolyamat végrehajtása környezetfüggő lehet. A többfájlos rendszer elsődleges célja a kontextusablak védelme (a „Lost in the Middle” figyelemvesztés elkerülése) és a kódregresszió megelőzése.

| Fájl | Betöltési Réteg | Tartalom | Mikor kötelező betölteni? |
| :--- | :--- | :--- | :--- |
| `01_SECURITY_CORE.md` | **Tier-0 (Kötelező mag)** | Központi vezérlő: projektfelmérés, L1/L2/L3 szintek, L1 Lightweight Mode, modulválasztás, prioritás, háromszintű frissességi kapu, háromirányú multi-agent handoff, Reviewer függetlenség, 6 lépéses fájlzárolás, HU/EN szinkron, dokumentációs kiadási kapu | **Minden** biztonsági szempontból releváns feladatnál (tervezés, elemzés, kódolás, tesztelés, audit) elsőként kötelező |
| `08_AI_AGENT_CODING_SECURITY.md` | **Tier-0 (Kódoló/Módosító mag)** | Prompt injection, nem megbízható projektfájlok, pontosított `HALT & ESCALATE`, eszközhasználat, jogosultságok, multi-agent zárolás, emberi jóváhagyás | Minden olyan ágensnél kötelező, amely kódot vagy konfigurációt ír, módosít, generál, parancsot futtat vagy konkrét kódmódosítást javasol *(tisztán elméleti/architekturális elemzésnél nem kötelező a teljes modult betölteni)* |
| `02_FIREBASE_SECURITY.md` | **Tier-1 (Szelektív domén)** | Firebase Auth, Firestore, Storage, Functions v2, Hosting, App Check, IAM, Rules, PWA-persistence, 5-kategóriás limit- és kvótakezelés | Minden Firebase-projektben; az érintett komponensek szerint |
| `03_FRONTEND_SECURITY.md` | **Tier-1 (Szelektív domén)** | HTML, CSS, JS/TS, React, DOM, böngésző API-k, PWA, Service Worker, CSP | Minden frontend- vagy PWA-feladatnál |
| `04_IDENTITY_ACCESS_SECURITY.md` | **Tier-1 (Szelektív domén)** | Hitelesítés, MFA, OAuth/OIDC, session, tokenek, szerepkörök, fiókéletciklus | Bármilyen bejelentkezés, regisztráció, jogosultság vagy felhasználói fiók esetén |
| `05_API_INTEGRATION_SECURITY.md` | **Tier-1 (Szelektív domén)** | Callable/HTTP API, webhook, külső API, SSRF, CORS, validáció, abuse és quota, LLM API-k | Minden API-végpontnál vagy külső integrációnál |
| `06_DEVSECOPS_SUPPLY_CHAIN.md` | **Tier-1 (Szelektív domén)** | Git, npm, lockfile, CI/CD, build, secrets, függőségek, release | Minden repository-, csomag-, build- vagy deploy-feladatnál |
| `07_DATA_PROTECTION_PRIVACY.md` | **Tier-1 (Szelektív domén)** | GDPR, adatminimalizálás, megőrzés, törlés, titkosítás, naplózás, adatvédelmi incidens | Személyes adat, ügyféladat, analitika, fájlfeltöltés vagy adatmegőrzés esetén |
| `09_SECURITY_TESTING_THREAT_MODELING.md` | **Tier-1 (Szelektív domén)** | Threat modeling, ASVS 5.0.0 verifikáció, tesztterv, SAST/DAST, emulator, Admin SDK authz tesztek, pentest | Tervezéskor, jelentős módosításkor, release előtt és auditnál |
| `10_SOURCE_REGISTER.md` | **Tier-1 (Szelektív domén)** | Hivatalos és normatív források, valós ellenőrzési dátumok (`last_verified`), verzióstátusz, platformlimitek | Új projekt indulásakor, architektúra-döntésnél, auditnál és 90 napos frissítéskor |
| `11_REFERENCE_TEMPLATES_FIREBASE.md` | **Tier-2 (Implementációs réteg)** | 5 strukturált Firebase v2 referencia-implementáció (`firestore.rules`, `onCall` két külön tranzakcióval, PWA `BroadcastChannel` + `TAB_ID` logout, Rules unit teszt, Admin SDK handler authz teszt) 7-elemű metaadat-fejléccel és `Verification Level` jelöléssel | **Kizárólag akkor töltendő be**, ha konkrét Firebase-implementáció (`firestore.rules`, Cloud Functions v2, PWA logout/persistence) vagy annak emulátoros/egységtesztelése történik |
| `SECURITY_PROFILE_TEMPLATE.md` | **Sablon** | Projektbiztonsági profil sablon (teljes L2/L3 és kompakt L1 Lightweight Mode opcióval) | Projekt indításakor és architektúra-változáskor |
| `RELEASE_GATE_TEMPLATE.md` | **Sablon** | Kiadási biztonsági kapu és `DOCUMENTATION_RELEASE_GATE` ellenőrzőlista sablon |Minden release, PR lezárás és dokumentációs csomagkiadás előtt |
| `handoff_contracts.schema.json` | **JSON Schema** | Géppel validálható JSON Schema (Draft 2020-12) a `PLANNER_TO_CODER`, `CODER_TO_REVIEWER` és `REVIEWER_TO_PLANNER` átadási szerződésekhez | Multi-agent munkafolyamatok gépi vagy strukturált validálásakor |
| `LEGACY_SOURCE_v3_2_2_EN.md` | **Legacy Archívum** | A korábbi egyfájlos v3.2.2 angol nyelvű skill történeti archívuma (`DEPRECATED / LEGACY ARCHIVE`) | Történeti visszakereséshez; aktív fejlesztéshez a `01`–`11` modulok használandók! |

---

## 3. Használati mód

1. A teljes csomagot helyezd el a projekt repository-jában vagy az AI-ágens által elérhető szabályzatkönyvtárban.
2. Az ágens elsőként mindig az `01_SECURITY_CORE.md` fájlt olvassa el, és ellenőrzi a háromszintű frissességi kaput (`CORE-FRESH-001`).
3. A Core alapján az ágens elvégzi a projektfelmérést és létrehozza (vagy frissíti) a `SECURITY_PROFILE.md` projekt-specifikus döntési lapot (L1 esetén választható az arányos L1 Lightweight Mode – `CORE-TIER-L1-001`).
4. A Core modulbetöltési hierarchiája (`CORE-LOAD-001`) alapján az ágens csak a feladathoz szükséges modulokat tölti be:
   - Kódírásnál és módosításnál a `01_SECURITY_CORE.md` és a `08_AI_AGENT_CODING_SECURITY.md` mindig kötelező (Tier-0).
   - Szokásos Firebase web/PWA fejlesztésnél a `02_FIREBASE_SECURITY.md` és a `03_FRONTEND_SECURITY.md` alapértelmezett (Tier-1).
   - Konkrét Firebase szabályok, Cloud Functions v2 végpontok, PWA kijelentkezés vagy emulátoros tesztek írásakor be kell tölteni a `11_REFERENCE_TEMPLATES_FIREBASE.md` modult is (Tier-2).
5. Többügynökös (`Planner -> Coder -> Security Reviewer -> Planner`) munkafolyamat esetén kötelező a háromirányú handoff-szerződések (`CORE-HANDOFF-001`), a Reviewer függetlenség (`CORE-REV-001`) és a 6 lépéses exkluzív fájlzárolási protokoll (`CORE-CONC-001`) alkalmazása.
6. Minden biztonsági követelményhez legyen ellenőrizhető bizonyíték: teszt, konfiguráció, kódhely, CI eredmény, független review vagy dokumentált, ember által jóváhagyott kockázatelfogadás.
7. Release előtt a `09_SECURITY_TESTING_THREAT_MODELING.md` és a `RELEASE_GATE_TEMPLATE.md` szerinti ellenőrzést végre kell hajtani.

---

## 4. Fontos korlátozások és a `DOCUMENTATION_RELEASE_GATE` hatóköre (`CORE-DOCGATE-001`)

- A szabályzat nem helyettesíti a projekt konkrét fenyegetésmodelljét, jogi/adatvédelmi értékelését vagy szakértői penetrációs tesztjét.
- A szabványra hivatkozás nem bizonyít megfelelést. Megfelelőségi állítás csak az alkalmazandó követelmények tételes ellenőrzésével és bizonyítékokkal tehető.
- A `11_REFERENCE_TEMPLATES_FIREBASE.md` modulban található referencia-kód szemléltető referencia-implementáció (`REFERENCE / MINTA`), amely alapértelmezetten `DOCUMENTATION_REVIEWED` státuszú (`REF-VERIFY-001`). Nem univerzálisan biztonságos, változtatás és emulátoros tesztelés nélkül élesbe állítható termékkód.
- A Firebase Admin SDK a Firestore Security Rules szabályait megkerüli; a szerveroldali hozzáférést IAM-mel, kódbeli autorizációval és külön handler-szintű negatív tesztekkel (`REF-TEST-ADMIN-001`) kell védeni.
- A szabványok és SDK-k változnak. A forrásregisztert (`10_SOURCE_REGISTER.md`) és a projektben használt verziókat (`package-lock.json`) minden nagyobb kiadásnál és 90 naponta ellenőrizni kell.

> **FONTOS HATÓKÖR-KORLÁTOZÁS (`CORE-DOCGATE-001`):** A `DOCUMENTATION_RELEASE_GATE: PASS` eredmény **kizárólag azt igazolja**, hogy a vizsgált dokumentumcsomag megfelel az előírt dokumentációs teljességi, szerkezeti, hivatkozási, HU/EN szinkronizációs és belső konzisztenciakövetelményeknek.
>
> **A `DOCUMENTATION_RELEASE_GATE: PASS` ÖNMAGÁBAN NEM IGAZOLJA:**
> 1. a teljes Firebase-alkalmazás biztonságát;
> 2. a konkrét projektimplementáció helyességét;
> 3. teljes körű biztonsági audit elvégzését;
> 4. penetrációs teszt elvégzését;
> 5. production readiness (élesítési érettségi) állapotot;
> 6. jogi vagy szabályozási (pl. GDPR, ISO, ASVS) megfelelőségi tanúsítást.

---

## 5. Eredet, HU/EN szinkron és verziótörténet (`CORE-LANG-001`)

- **Eredet:** A korábbi Firebase/PWA Security Skill v3.2.2 (`LEGACY_SOURCE_v3_2_2_EN.md`) a Firebase-modul és a referencia-implementációk szakmai kiindulópontja. A korábbi v3.2.2 dokumentum ASVS 4.0.3 hivatkozását az új csomag az **OWASP ASVS 5.0.0** stabil kiadására frissítette, miközben a v3.2.2 kiforrott 2nd Gen (v2) kódsablonjait a `11_REFERENCE_TEMPLATES_FIREBASE.md` modulba emelte át szigorú metaadat- és ellenőrzésiszint-keretezéssel (`REF-MOD-001`, `REF-VERIFY-001`).
- **HU/EN tükörszinkron (`CORE-LANG-001`):** Minden normatív követelményhez, tesztelváráshoz és referenciasablonhoz egyedi, nyelvfüggetlen azonosító (`ID`) tartozik. A magyar (`HU`) és angol (`EN`) dokumentumváltozatban ugyanazoknak az ID-knak és logikai struktúráknak kell szerepelniük. EU/magyar jogszabályi és adatvédelmi (GDPR/NAIH) fogalmaknál a magyar (`HU`), technikai/API-nevek, kódminták, HTTP fejlécek és OWASP/NIST szakkifejezések esetén az angol (`EN`) változat és a hivatalos gyártói dokumentáció az irányadó.

---

## 6. A csomag fájljai (`v1.1.0`)

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
- `LEGACY_SOURCE_v3_2_2_EN.md` *(történeti előzmény / `DEPRECATED / LEGACY ARCHIVE`)*