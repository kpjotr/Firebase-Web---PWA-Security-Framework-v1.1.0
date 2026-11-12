---
name: api-integration-security
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
---

# 05 – API & Integration Security

## 1. Hatókör és aktiválás (`API-SCOPE-001`)

Ez a modul (`Tier-1` szelektív doménmodul a `CORE-LOAD-001` szerint) minden Cloud Functions v2 callable (`onCall`) és HTTP (`onRequest`) végpont, REST API, webhook, külső szolgáltatás, fizetési kapu (payment gateway), Google API, e-mail szolgáltatás vagy AI/LLM integráció esetén **kötelező**. Konkrét Firebase callable végpont implementálásakor a `11_REFERENCE_TEMPLATES_FIREBASE.md` (`REF-FN-CALLABLE-001` és `REF-TEST-ADMIN-001`) sablonjait is be kell tölteni.

---

## 2. API tervezés (`API-DESIGN-001`)

- Minden végpontnak legyen dokumentált célja, felelőse, autentikációs és autorizációs modellje, strict input/output sémája, rate limitje és strukturált naplózási terve.
- Szigorúan különítsd el az autentikációt (`request.auth`), az alkalmazás-attestationt (`request.app` / App Check) és az autorizációt (tulajdonosi/szerepkör ellenőrzés).
- A kliens által a request payloadban (`request.data` / `req.body`) küldött `userId`, `uid`, `role`, `tenantId`, `price`, `discount`, `balance`, `remainingUses` vagy `status` mező **soha nem megbízható** (`CORE-PRIN-005`, `CORE-PRIN-006`).
- HTTP (`onRequest`) végpontoknál csak a ténylegesen szükséges HTTP metódusokat (pl. `POST`) engedélyezd; minden más metódust `405 Method Not Allowed` válasszal utasíts el.
- Alkalmazz explicit `Content-Type` ellenőrzést, kérésméret-korlátot (body size limit), timeoutot és `maxInstances` konkurencia-korlátot (`FIREBASE-FN-001`).
- A kliensnek visszaadott hibaüzenet csak szabványos hibakódot (pl. `HttpsError`) tartalmazhat; tilos belső stack trace-t, adatbázis-sémát, útvonalat vagy titkot felfedni (`CORE-PRIN-009`).

---

## 3. Input validáció (`API-VAL-001`)

- Minden végponton kötelező a szerveroldali allowlist alapú séma-validáció (pl. **Zod `.strict()`** vagy azzal egyenértékű validator – `REF-FN-CALLABLE-001`), amely azonnal elutasítja az ismeretlen/extra mezőket (Mass Assignment / OWASP API3:2023 védelem).
- Ellenőrizd minden mező típusát, méretét, stringhosszát, tömbhosszát, enum-értékkészletét, számérték-tartományát, formátumát (pl. UUID) és üzleti invariánsait.
- A bemenet canonicalizálását (normalizálását) egyértelműen meghatározott feldolgozási rétegben végezd; kerüld a többszörös vagy eltérő dekódolást a validáció és a felhasználás között.
- Alkalmazz célzott védelmet NoSQL/SQL injection, path traversal, command injection, template injection és SSRF ellen az érintett sink szerint.
- A fájl- és URL-bemeneteket külön szigorított szabályok (`API-SSRF-001`, `FIREBASE-STORAGE-001`) szerint kezeld.

---

## 4. Authorization és BOLA / BFLA védelem (`API-AUTHZ-001`)

- **BOLA / IDOR (OWASP API1:2023) elleni védelem:** Minden rekordműveletnél szerveroldalon (tranzakción belül) ellenőrizd a resource ownershipet (`resource.userId === request.auth.uid`) és a tenant scope-ot (`REF-FN-CALLABLE-001`).
- **BFLA (OWASP API5:2023) elleni védelem:** Funkciószintű és adminisztratív jogosultságot mindig a szerveren ellenőrizz, soha ne csak a frontend route-on vagy UI-ban.
- Admin-only végpontnál követelj meg explicit admin claimet/jogosultságot és L2/L3 szinten Admin MFA-t (`IAM-ADMIN-001`).
- Tömeges (batch/bulk) műveletnél a csomag **minden egyes rekordjára** külön ellenőrizd a hozzáférési jogosultságot.
- Soha ne fogadj el tetszőleges Firestore dokumentum-útvonalat vagy kollekciónevet közvetlenül kliens inputból!

---

## 5. Rate limiting és erőforrás-védelem (`API-RATE-001`)

- Korlátozd a hívási gyakoriságot UID, IP, tenant, végpont és költség szerint, ahol releváns (OWASP API4:2023 – Unrestricted Resource Consumption / Denial of Wallet elleni védelem).
- Alkalmazz külön, szigorúbb limitet a hitelesítési, jelszó-visszaállítási, meghívó-beváltási, e-mail-küldési, foglalási, adatexportálási, keresési és fizetős külső/AI API-t hívó műveletekre.
- **Két külön tranzakció szigorú szétválasztása (`FIREBASE-FN-003`):** Firestore-alapú per-UID rate limiter használatakor a rate limit ellenőrzését és a számláló növelését **önálló tranzakcióban**, az üzleti tranzakció **előtt** kell végrehajtani (`REF-FN-CALLABLE-001`). Limit-túllépéskor tilos a tranzakció callbackjén belül kivételt dobni: előbb commitold a növelt `blockedCount` értéket, és a tranzakció *után* dobj `resource-exhausted` hibát!
- **Ismert korlátok kezelése:** A per-UID limit önmagában nem véd a többfiókos (sybil) támadások ellen, a Firestore számláló pedig magas konkurencia (>1 kérés/mp/user-action dokumentum) esetén document contentiont és tranzakciós költséget okoz. Publikus vagy nagy terhelésű (L3) végpontoknál kombináld App Checkkel, IP-alapú limittel (Redis / Upstash / Memorystore) és WAF/Cloud Armor védelemmel.
- Ne támaszkodj kizárólag a `maxInstances` beállításra vagy a Budget Alertre költségplafonként (`FIREBASE-FN-001`).
- Korlátozd a request body méretét, a feltöltési méretet, a lekérdezési elemszámot (pagination), a futási időt és a külső API-kvótát.

---

## 6. Idempotencia, tranzakciós konkurencia és ledger (`API-IDEMP-001`)

- Minden nem idempotens, állapotmódosító mutációhoz (foglalás, bérletlevonás, rendelés, fizetés, egyenlegmódosítás) követelj meg `idempotencyKey` paramétert (`FIREBASE-FN-004`, `FIREBASE-IDEMP-001`, `REF-FN-CALLABLE-001`).
- Az idempotencia-rekordot kösd a felhasználóhoz és a művelethez (`_idempotency/${uid}_${idempotencyKey}`), és tárold benne a bemeneti paraméterek **SHA-256 `requestHash`** lenyomatát.
- Azonos `idempotencyKey` + **azonos `requestHash`** esetén add vissza a korábbi sikeres eredményt (`deduplicated: true`) újabb levonás nélkül.
- Azonos `idempotencyKey` + **eltérő `requestHash`** esetén utasítsd el a kérést (`already-exists` / `failed-precondition`).
- A domain-állapotmódosítás, az **append-only `ledger`** bejegyzés és az `_idempotency` rekord írása **egyetlen atomi `db.runTransaction` műveletben** történjen (`FIREBASE-FN-004`).
- **Tilos külső mellékhatás tranzakción belül:** Mivel a Firestore tranzakció callbackje konkurencia esetén többször újrafuthat (retry), **tilos (`MUST NOT`)** benne külső HTTP/API-hívást, e-mail-küldést vagy fizetési gateway hívást végezni! Ezeket outbox/event patternnel vagy idempotens utófeldolgozással kezeld.
- A webhook- és queue-fogyasztók (consumers) mindig legyenek idempotensek, mert a kézbesítés legalább egyszeri (at-least-once) modellben ismétlődhet.

---

## 7. Webhook biztonság (`API-WEBHOOK-001`)

- Fizetési vagy külső szolgáltatói webhook (`onRequest`) esetén kötelező a kriptográfiai aláírás (pl. HMAC) ellenőrzése a szolgáltató hivatalos előírása szerint, a **nyers kérés-törzs (`req.rawBody`)** alapján!
- Érvényesíts időbélyeg/replay ablakot (timestamp tolerance), `event.id` alapú atomi deduplikációt és idempotens feldolgozást.
- Használj külön webhook signing secretet Secret Managerből (`defineSecret`), és támogasd a kulcsrotációt.
- Soha ne fogadj el sikeres fizetési státuszt kliensoldali redirect URL vagy query paraméter alapján!
- Tilos bármilyen üzleti feldolgozást vagy adatbázis-módosítást végezni a webhook aláírás-ellenőrzése előtt.
- Válaszolj gyorsan a szolgáltató timeout-elvárásának megfelelően, a hosszabb feldolgozást pedig aszinkron/idempotens módon végezd.

---

## 8. CORS és CSRF (`API-CORS-001`)

- A CORS kizárólag böngészőoldali cross-origin olvasási szabály; **nem helyettesíti az autentikációt vagy az autorizációt** (`CORE-PRIN-012`).
- A CORS `Access-Control-Allow-Origin` allowlist legyen pontos; `Access-Control-Allow-Credentials: true` mellett a `*` wildcard használata tilos és böngésző által is blokkolt, de a bejövő `Origin` fejléc ellenőrzés nélküli visszatükrözése is **szigorúan tilos**.
- Cookie-alapú session használatakor alkalmazz CSRF tokent, `SameSite=Strict` (vagy `Lax`) attribútumot, `__Host-` prefixet és szerveroldali `Origin`/`Referer` fejléc-ellenőrzést.
- Bearer token (Authorization header / Firebase Callable SDK) alapú API-knál a klasszikus CSRF kockázat eltérő, de az XSS elleni védelem továbbra is kritikus.

---

## 9. SSRF és szerveroldali URL-lekérés (`API-SSRF-001`)

Ha a backendnek kliens által megadott vagy befolyásolt URL-t kell lekérnie (OWASP API7:2023 – Server-Side Request Forgery):
- Alkalmazz szigorú cél-domain allowlistet, amikor csak lehetséges.
- Tiltsd a csak `https:`-től eltérő protokollokat és a nem szabványos portokat.
- DNS-feloldás után blokkold a loopback (`127.0.0.0/8`, `::1`), privát (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`), link-local (`169.254.0.0/16`, `fe80::/10`), **cloud metadata (`169.254.169.254`, `metadata.google.internal`)** és minden belső hálózati IPv4/IPv6 címet!
- Védj a DNS rebinding és a HTTP redirect követés kockázata ellen: a redirecteket tiltsd le (`redirect: 'error'` / `manual`), vagy minden egyes átirányítási célállomást validálj újra DNS/IP szinten.
- Korlátozd a válasz méretét, a várt MIME-típust és a hálózati timeoutot.

---

## 10. Külső API és OAuth integráció (`API-EXT-001`)

- Kérd a legkisebb szükséges OAuth scope-ot, használj külön API kulcsot/secretet Secret Managerből, és a tokenkezelést tartsd szerveroldalon.
- A felhasználónkénti OAuth `refresh_token`-eket tárold kliens elől elzárt (`allow read, write: if false;`) Firestore kollekcióban, alkalmazásszinten (pl. Cloud KMS-sel) titkosítva (`FIREBASE-STORAGE-001`).
- Biztosíts token-rotációt, visszavonást (revocation) és fiókleválasztási (disconnect) folyamatot.
- A külső API-tól érkező választ mindig nem megbízható (untrusted) bemenetként kezeld és sémával validáld!
- Alkalmazz explicit timeoutot, exponenciális backoff retry-t, circuit breakert, kvótát és költségkorlátot.

---

## 11. AI / LLM API integrációk (`API-LLM-001`)

Ha az alkalmazás generatív AI / LLM API-t használ:
- Kizárólag a backendből (Cloud Functions v2) hívd az LLM szolgáltatót; az API kulcsot tárold Secret Managerben (`defineSecret`).
- Érvényesíts szigorú per-user / per-tenant napi és havi kvótát, token-budgetet, rate limitet, maximális input/output hosszt és költségriasztást (Denial of Wallet védelem).
- Adatminimalizálás (`07_DATA_PROTECTION_PRIVACY.md`): ne küldj szükségtelen személyes adatot (PII) vagy más felhasználó üzleti adatát a modellnek.
- **Prompt Injection és Excessive Agency (OWASP LLM Top 10 2025) elleni védelem:** A modell kimenete soha nem tekinthető hitelesnek vagy biztonságosnak; nem hozhat önálló jogosultsági döntést, és nem indíthat privilegizált műveletet szerveroldali determinisztikus authorization nélkül.
- A modell kimenetét validáld strict sémával (pl. Zod), és megjelenítés előtt kezeld nem megbízható szövegként (XSS sanitization / output encoding).
- A RAG-ból vagy adatbázisból beemelt dokumentumokat kezeld nem megbízható adatként, és a promptban egyértelműen különítsd el a rendszerutasításoktól.
- *Megjegyzés a szabványokhoz:* Az **OWASP LLM Top 10 (2025)** és a **NIST SP 800-218A** informatív forrásként alkalmazandó; az utóbbi elsősorban AI-modell fejlesztési életciklusra készült, ezért önmagában nem tekinthető agent-runtime szabványnak (`10_SOURCE_REGISTER.md`).

---

## 12. Kötelező API tesztlista (`API-TEST-001`)

Minden végpontot és integrációt pozitív és negatív (`REF-TEST-ADMIN-001`) tesztekkel kell ellenőrizni:
- [ ] anonim hívás, lejárt/rossz token vagy nem verifikált e-mail elutasítása;
- [ ] másik felhasználó vagy másik tenant erőforrás-azonosítójával történő hívás elutasítása (BOLA);
- [ ] normál felhasználó által hívott admin végpont elutasítása (BFLA);
- [ ] extra mezőket tartalmazó (Mass Assignment), hibás típusú vagy túlméretes input elutasítása (`Zod .strict()`);
- [ ] rate limit túllépés (`resource-exhausted`) és annak igazolása, hogy az 1. (rate-limit) tranzakció állapota megmarad akkor is, ha a 2. (üzleti) tranzakció előfeltétel-hiba miatt elbukik (`FIREBASE-FN-003`);
- [ ] idempotens újrahívás azonos `idempotencyKey` + azonos `requestHash` mellett (`deduplicated: true`, nincs dupla levonás), valamint eltérő paraméterrel (`requestHash` mismatch -> `already-exists` elutasítás) (`FIREBASE-FN-004`);
- [ ] hamis aláírású vagy ismételt (replayed) webhook elutasítása (`API-WEBHOOK-001`);
- [ ] SSRF belső IP / metadata IP (`169.254.169.254`) és redirect elutasítása (`API-SSRF-001`);
- [ ] CORS és CSRF szabályok ellenőrzése (`API-CORS-001`);
- [ ] külső API / LLM timeout és kvóta-kikényszerítés;
- [ ] annak igazolása, hogy sem a hibaüzenetbe, sem a logokba nem kerül secret, stack trace vagy nyers PII.

---

## 13. Hivatalos források (`API-REF-001`)

A pontos verzióstátuszt és ellenőrzési dátumokat (`last_verified`) lásd a `10_SOURCE_REGISTER.md` modulban:
- OWASP API Security Top 10 (2023): https://owasp.org/API-Security/
- OWASP REST Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
- OWASP SSRF Prevention Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html
- Firebase Cloud Functions Docs: https://firebase.google.com/docs/functions
- OWASP Top 10 for LLM Applications (2025): https://owasp.org/www-project-top-10-for-large-language-model-applications/
- NIST SP 800-218A: https://csrc.nist.gov/pubs/sp/800/218/a/final