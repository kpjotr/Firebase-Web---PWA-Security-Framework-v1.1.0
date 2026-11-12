---
name: firebase-architecture-security
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
platform: Firebase / Google Cloud
---

# 02 – Firebase Architecture & Security

## 1. Hatókör és forrási alap (`FIREBASE-SCOPE-001`)

Ez a modul a korábbi Universal Firebase & Web/PWA Cybersecurity Skill v3.2.2 (`LEGACY_SOURCE_v3_2_2_EN.md`) és a v1.0.0 Firebase modul továbbfejlesztett `v1.1.0` kiadása. Megtartja a kockázatarányos L1/L2/L3 modellt, az A/B/C onboarding modelleket, valamint a Firestore Rules, App Check, Cloud Functions v2, idempotencia, ledger, PWA storage és audit-előírásokat, kiegészítve az ötkategóriás limit- és kvótakezeléssel (`FIREBASE-LIMIT-001`), a két külön tranzakció szigorú szétválasztásával (`FIREBASE-FN-003`, `FIREBASE-FN-004`) és az ellenőrzött referencia-kódsablonok (`11_REFERENCE_TEMPLATES_FIREBASE.md`) hivatkozásaival.

**Kötelező (`CORE-LOAD-001`):** A `01_SECURITY_CORE.md` + `02_FIREBASE_SECURITY.md` minden Firebase-projektben kötelező. A projekt csak az általa ténylegesen használt komponensek fejezeteit alkalmazza, de a komponens hiányát a `SECURITY_PROFILE.md` fájlban dokumentálja. Konkrét implementációkor (`firestore.rules`, Cloud Functions v2, PWA kijelentkezés, emulátortesztek) kötelező betölteni a Tier-2 szintű `11_REFERENCE_TEMPLATES_FIREBASE.md` modult is.

---

## 2. Firebase trust boundary és felelősség (`FIREBASE-BOUND-001`)

- A Firebase web config `apiKey` értéke nem titok; projektazonosító/hozzáférési konfigurációs adat. Korlátozni kell a releváns Google Cloud API-kra és HTTP referrerre, de ez nem jogosultságkezelés.
- A Firebase Authentication az identitást igazolja; az App Check a regisztrált app-kliens hitelességét/attestationjét segíti; egyik sem helyettesíti a Firestore Rules-t, a szerveroldali autorizációt vagy az IAM-et.
- A Firebase Admin SDK és a szerveroldali Google Cloud klienskönyvtárak **nem** a Firestore Security Rules alapján autorizálnak; a szerver hitelesítése IAM/ADC útján történik, és megkerüli a Security Rules szabályokat. Ezért backend kódon belül explicit jogosultság- és tulajdonjog-ellenőrzés, valamint külön handler-szintű autorizációs tesztelés (`REF-TEST-ADMIN-001`) kötelező.
- A kliens által küldött `uid`, `role`, `price`, `balance`, `remainingUses`, `isAdmin`, `createdAt` és üzleti állapot nem tekinthető hitelesnek.
- Minden környezethez külön Firebase projektet kell használni, legalább fejlesztői/staging és production elkülönítéssel, ha a projekt mérete és kockázata ezt indokolja; production adatok ne kerüljenek fejlesztői környezetbe.
- Publikus statikus marketingoldalakat és hitelesített webalkalmazásokat (SPA/PWA) külön projekten, külön aldomainen vagy külön Firebase Hosting targeten kell üzemeltetni. Mivel a szülődomainre beállított cookie-k megoszlanak az aldomainek között, bármilyen session cookie-t szigorúan **Host-Only cookie-ként** (`Domain` attribútum nélkül, `__Host-` prefixszel, `Secure`, `HttpOnly`, megfelelő `SameSite` beállítással) kell konfigurálni.

---

## 3. Kockázati besorolás és onboarding (`FIREBASE-TIER-001`)

A Core 4. fejezete (`CORE-TIER-001`, `CORE-TIER-L1-001`, `CORE-ONBOARD-001`) szerinti L1/L2/L3 és A/B/C modell kötelező. Firebase-specifikus minimum:

### L1 (`FIREBASE-TIER-001`)
- Szigorú Firestore Rules (`REF-RULES-BASE-001`): tulajdonjog (`isOwner`), mező-allowlist (`hasOnly`), adattípus, hossz- és tartománykorlát, immutable mezők (`userId`, `createdAt`).
- Kliensoldali CRUD csak nem kritikus, alacsony kockázatú saját adatra, felesleges Cloud Functions backend kényszerítése nélkül.
- Budget alert, rendszeres mentés/export és API-key restriction (HTTP Referrer + API scope).
- Minden adatbázis- és Storage-hozzáférés deny-by-default.
- Kötelező pozitív és negatív (`assertFails`) emulátoros Rules tesztek (`REF-TEST-RULES-001`).

### L2 (`FIREBASE-TIER-001`)
- Ügyféladatot (PII) vagy üzleti folyamatot (foglalás, bérlet, egyenleg, rendelés) kezelő rendszer alapértelmezett szintje.
- Minden L1 kontroll kötelező.
- App Check enforce (ReCAPTCHA Enterprise) a támogatott kliens-hozzáféréseknél, fokozatos rollout és monitoring mellett.
- Kritikus állapotváltozás kizárólag Cloud Functions v2 backendből (`REF-FN-CALLABLE-001`), atomikus Firestore tranzakcióval (`db.runTransaction`).
- A kritikus kollekciók (`passes`, `passLedger`, `bookings`, egyenlegek, szerepkörök) közvetlen kliensírása szigorúan tiltott (`allow create, update, delete: if false;`).
- Zod `.strict()` vagy egyenértékű szerveroldali séma-validáció, explicit authorization, külön tranzakciós rate limiting (`FIREBASE-FN-003`), SHA-256 `requestHash`-sel védett idempotencia és append-only `ledger` (`FIREBASE-FN-004`).
- Admin hozzáféréshez MFA és Custom Claims.
- Szerveroldali strukturált audit logging, riasztás, ütemezett mentés (kritikus állapotra PITR ajánlott) és restore-teszt.

### L3 (`FIREBASE-TIER-001`)
- L2 összes kontrollja kötelező.
- App Check Replay Protection (`consumeAppCheckToken: true`) a magas kockázatú, állapotmódosító callable műveletekre, ha a használt SDK és régió/konfiguráció támogatja.
- `consumeAppCheckToken: true` esetén a kliens kötelezően `{ limitedUseAppCheckTokens: true }` opcióval hívja a callable végpontot, és a futtató service account megkapja a `Firebase App Check Token Verifier` (`roles/firebaseappcheck.tokenVerifier`) IAM szerepkört (`FIREBASE-FN-002`).
- Skálázható per-UID és per-IP rate limiting (pl. Redis / Upstash / Memorystore); WAF / Google Cloud Armor az architektúrához illeszkedő HTTPS Load Balancer + Serverless NEG útvonalon.
- Függvényenként dedikált least-privilege IAM service accountok, kötelező Firestore PITR (7 napos helyreállítás), rendszeres DR/restore drill, szigorú nonce/hash alapú CSP, SIEM/Cloud Monitoring riasztások és független biztonsági vizsgálat.

---

## 4. Ötkategóriás Limit-, Kvóta- és Szabálykezelés (`FIREBASE-LIMIT-001`)

A modell-hallucinációk és a pontatlan korláthivatkozások elkerülése érdekében minden technikai korlátot és kvótát az alábbi **5 normatív kategória** egyikébe kell sorolni:

1. **`[PLATFORM HARD LIMIT]`** – A szolgáltató (Firebase / Google Cloud) által dokumentált és kikényszerített technikai platformkorlát.
2. **`[PLAN QUOTA]`** – Csomaghoz (pl. Spark) vagy szolgáltatási szinthez kötött működési kvóta.
3. **`[PRICING / FREE TIER ALLOWANCE]`** – Díjmentes használati keret vagy díjazási feltétel (amely túllépéskor számlázást indít, és a szolgáltató árlistája szerint változhat).
4. **`[FRAMEWORK DEFAULT]`** – A biztonsági keretrendszer által ajánlott alapértelmezés (nem platformlimit).
5. **`[PROJECT-SPECIFIC LIMIT]`** – Az adott alkalmazás üzleti és kapacitási követelménye (`SECURITY_PROFILE.md`).

| Megnevezés | Kategória | Érték és Pontos Szabály | Hivatalos Forrás & Ellenőrzési Státusz (`last_verified`) |
| :--- | :--- | :--- | :--- |
| **Custom Claims méret** | `[PLATFORM HARD LIMIT]` | Max. **1 000 byte** payload felhasználónként. | Firebase Auth Custom Claims Docs (`https://firebase.google.com/docs/auth/admin/custom-claims`) – Ellenőrizve: `2026-11-02` |
| **Firestore Rules `get()` / `exists()`** | `[PLATFORM HARD LIMIT]` | Max. **10 hívás** egyedi kérésenként/lekérdezésenként; max. **20 hívás** összesen multi-doc olvasás, tranzakció vagy batch esetén (azon belül műveletenként is max. 10). Minden hívás olvasásként számlázódik. | Firestore Rules Structure & Limits (`https://firebase.google.com/docs/firestore/security/rules-structure`) – Ellenőrizve: `2026-11-02` |
| **Identity Platform Spark DAU** | `[PLAN QUOTA]` | **3 000 DAU** napi aktív felhasználói korlát a Spark (no-cost) csomagban az érintett Identity Platform funkciókra. | Firebase Pricing (`https://firebase.google.com/pricing`) – Ellenőrizve: `2026-11-02` *(Kiadás előtt az aktuális árlista ellenőrizendő!)* |
| **Identity Platform Blaze MAU** | `[PRICING / FREE TIER ALLOWANCE]` | **50 000 MAU** díjmentes keret sztenderd providerekre és TOTP MFA-ra; szolgáltatói kategóriák szerint eltérő díjazással (pl. SMS MFA üzenetenként és SAML/OIDC külön díjköteles). | Firebase Pricing (`https://firebase.google.com/pricing`) – Ellenőrizve: `2026-11-02` *(Díjazási feltételek változhatnak!)* |
| **Firestore `list` query `.limit(50)`** | `[FRAMEWORK DEFAULT]` | `request.query.limit <= 50` a Rules-ban és `.limit(<=50)` a kliensben. Nem platformlimit és nem önálló költségvédelmi garancia; a projekt igényei szerint (`[PROJECT-SPECIFIC LIMIT]`) módosítható. | Keretrendszer belső ajánlás (`02_FIREBASE_SECURITY.md`, `FIREBASE-RULES-003`) |

---

## 5. Authentication és account lifecycle (`FIREBASE-AUTH-001`)

- Válassz A/B/C onboardingot (`CORE-ONBOARD-001`), és ténylegesen konfiguráld a Firebase Console / Identity Platform oldalon.
- **Fióklétrehozás blokkolása vs. Adathozzáférés megtagadása (`FIREBASE-AUTH-001`):** Ha az Email/Password hitelesítés be van kapcsolva, bárki, aki ismeri a nyilvános Firebase `apiKey` értéket, meghívhatja az `accounts:signUp` REST végpontot. Különböztesd meg a három kontrollt:

| Kontrollmechanizmus | Mit akadályoz meg ténylegesen? | Előfeltétel / Megjegyzés |
| :--- | :--- | :--- |
| **1. Disable User Sign-Up** (Console / Identity Platform) | Megakadályozza a Firebase Auth fiók létrehozását a kliensoldali API-kon keresztül. | A legtisztább kontroll Model A (Admin-provisioned) rendszerekhez. |
| **2. `beforeUserCreated` Blocking Function** | Feltételesen (pl. meghívó/allowlist alapján) blokkolja a Firebase Auth fiók létrehozását szerveroldalon. | Igényli a Firebase Auth with Identity Platform (Blaze) használatát. |
| **3. Zero-Privilege Default Account** (Security Rules / Custom Claims) | **Nem** akadályozza meg az Auth fiók létrejöttét, de garantálja, hogy a nem jóváhagyott `uid` semmilyen Firestore/Storage adatot nem tud olvasni vagy írni. | Kötelező másodlagos (Defense in Depth) védelmi réteg minden szinten (L1/L2/L3)! |

- **Authorized Domains:** Távolíts el minden nem használt domaint a *Firebase Console -> Authentication -> Settings -> Authorized domains* listából.
- Invite-only (Model B) esetén az invite legyen egyszer használatos vagy korlátozott, lejáró, célfelhasználóhoz/tenanthez kötött; a szerver ellenőrizze.
- Public signup (Model C) esetén kötelező az email verification (`request.auth.token.email_verified == true`), az Email Enumeration Protection bekapcsolása, az abuse protection, az App Check és a minimális alapjog.
- **Custom Claims és Token Revocation:** Szerepkörök (`role: 'admin' | 'staff'`) tárolhatók Custom Claims-ben (max. 1 000 byte – `[PLATFORM HARD LIMIT]`), így a Rules extra Firestore `get()` olvasás nélkül ellenőrizheti őket. Mivel a claim-módosulás csak az ID token frissülésekor (max. 1 óra vagy kliens `user.getIdToken(true)` híváskor) lép életbe, jogosultság-visszavonáskor, eszközvesztéskor vagy jelszócserekor szerveroldali `admin.auth().revokeRefreshTokens(uid)` hívás és kritikus végpontokon `checkRevoked: true` / szerveroldali státuszellenőrzés kombinációja szükséges.
- **Admin MFA vs. Google Sign-In:** A sima fogyasztói (`@gmail.com`) Google OAuth bejelentkezés **nem bizonyítja** az alkalmazás számára, hogy a felhasználó 2SV/MFA ellenőrzésen esett át. A Google Sign-In csak akkor fogadható el MFA-egyenértékű kontrollként adminoknál, ha a fiókok menedzselt **Google Workspace szervezethez** tartoznak, ahol a 2SV szervezeti policy által kötelezően kikényszerített, szerveroldali e-mail allowlisttel és `admin: true` Custom Claimmel kombinálva. Egyéb esetben Identity Platform natív TOTP MFA (`request.auth.token.firebase.sign_in_second_factor != null`) alkalmazandó.

---

## 6. Firestore Security Rules (`FIREBASE-RULES-001` .. `FIREBASE-RULES-003`, `FIREBASE-ADMIN-001`)

### 6.1. Kötelező elvek (`FIREBASE-RULES-001`)
- Minden Rules fájl `rules_version = '2';` legyen.
- Default deny legyen az alap (`match /{document=**} { allow read, write: if false; }`).
- **Logikai `OR` kiértékelés:** Az egymást átfedő `allow` szabályok logikai `OR`-ként működnek: ha *bármelyik* illeszkedő szabály `true` értéket ad, a hozzáférés engedélyezett. A gyökérszintű catch-all deny csak a nem illeszkedő útvonalak alapértelmezése, **nem** globális deny override. Soha ne helyezz túl megengedő szabályt szülőkollekcióra.
- A `get()`/`exists()` hívásoknak szigorú platformlimitjei (10 / 20 hívás – `FIREBASE-LIMIT-001`) és olvasási költségei vannak.
- Ne támaszkodj csak dokumentumútvonalra, ha a lekérdezési modellhez explicit tulajdonos/tenant (`userId` / `tenantId`) mező szükséges.
- A `hasOnly` önmagában nem elég: minden kliens által írható mezőre típus-, méret- és tartománykorlát, valamint `update` esetén a tulajdonosi (`userId`) és létrehozási (`createdAt`) mezők immutabilitása (`request.resource.data.userId == resource.data.userId`) is kötelező (`REF-RULES-BASE-001`).

### 6.2. Kritikus kollekciók zárolása (`FIREBASE-RULES-002`)
L2 és L3 esetén a foglalás (`bookings`), készlet/időpont (`slots`), fizetési státusz, bérlet (`passes`), kredit, egyenleg, tranzakciós napló (`passLedger`), szerepkör és adminisztratív állapot kliensoldali írása **szigorúan tiltott** (`allow create, update, delete: if false;`). A kliens legfeljebb olyan kérést küldhet a Cloud Functions backendnek, amelynek minden előfeltételét és hatását a szerver tranzakcióban újra validálja.

### 6.3. Lekérdezések és a `.limit(50)` szerepének pontosítása (`FIREBASE-RULES-003`)
- **A Security Rules nem szűrő (Rules Are Not Filters):** Ha egy szabály megköveteli az `isOwner(resource.data.userId)` és a `request.query.limit <= 50` feltételt, akkor a kliens lekérdezésének is explicit módon tartalmaznia kell a `where('userId', '==', currentUid)` és a `.limit(<=50)` záradékot, különben a teljes lekérdezés `permission-denied` hibával elbukik.
- **Normatív pontosítás a `.limit(50)` szerepéről (`FIREBASE-RULES-003`):** A kliensoldali `.limit(50)` önmagában **nem biztonsági kontroll**, mert a kliens által küldött lekérdezés módosítható. A Firestore Security Rules nem általános lekérdezéslimitáló mechanizmus, hanem a lekérdezés által elérhető adatokra vonatkozó hozzáférési feltételeket (`request.query.limit <= 50`) érvényesíti. A `limit <= 50` egy `[FRAMEWORK DEFAULT]` ajánlás (`FIREBASE-LIMIT-001`), amelyet önálló jogosultsági vagy költségvédelmi garanciának tekinteni **tilos (`MUST NOT`)**; költséges vagy érzékeny lekérdezéseknél szerveroldali korlátozást, kvótát vagy rate limitet is alkalmazni kell.

### 6.4. Admin SDK megkerülés védelme (`FIREBASE-ADMIN-001`)
A Firestore és Storage Security Rules **nem védik** az Admin SDK-val végzett szerveroldali műveleteket. A backendben kötelező:
- autentikálni a hívót (`request.auth` és `email_verified`);
- ellenőrizni a szerepkört és a resource ownershipet (`pass.userId === request.auth.uid`);
- strict sémával validálni a teljes inputot (`zod` `.strict()`);
- csak a szükséges dokumentumokat olvasni/írni;
- minimalizálni a service account IAM-jogosultságait;
- közvetlenül tesztelni a szerveroldali handler autorizációs logikáját negatív tesztekkel (`REF-TEST-ADMIN-001`).

---

## 7. Cloud Functions v2 és a Két Tranzakció Szétválasztása (`FIREBASE-FN-001` .. `FIREBASE-FN-004`)

### 7.1. Általános Cloud Functions v2 követelmények (`FIREBASE-FN-001`)
- Új backendnél a támogatott 2nd gen (`firebase-functions/v2`) API-kat használd (`request.auth`, `request.app`, `request.data`, `request.rawRequest`), és mindig ellenőrizd a projekt `package-lock.json` fájljában telepített SDK-verziót.
- `onCall` és `onRequest` között tudatosan válassz: Firebase SDK kliensekhez `onCall`, külső/fizetési webhookhoz `onRequest` (HMAC aláírás-ellenőrzéssel a `req.rawBody`-n).
- Minden függvény kapjon explicit régiót (`region`), timeoutot, memory/CPU beállítást és `maxInstances` korlátot. A `maxInstances` a konkurens példányokat korlátozza, nem havi költségplafon; a Budget Alert pedig értesítést küld, nem állítja le automatikusan a számlázást.
- Bemeneti adatot strict schema (pl. Zod `.strict()`) validálja, amely azonnal elutasítja az ismeretlen/extra mezőket (Mass Assignment védelem).

### 7.2. App Check és Replay Protection kockázatalapú alkalmazása (`FIREBASE-FN-002`)
- Az `enforceAppCheck: true` L2 és L3 szinten minden támogatott callable végponton alapértelmezetten kötelező.
- A `consumeAppCheckToken: true` (Replay Protection) **nem minden callable végpontra automatikusan kötelező**, mivel extra ellenőrzési round-trippel és késleltetéssel jár. Kockázatalapon, az **L3-as vagy különösen érzékeny/költséges L2 állapotmódosító végpontokon** kell alkalmazni.
- Amikor egy végponton a `consumeAppCheckToken: true` be van kapcsolva, kötelező:
  1. kliensoldalon a `{ limitedUseAppCheckTokens: true }` opció átadása a `httpsCallable` hívásban;
  2. szerveroldalon a futtató Service Account számára a `Firebase App Check Token Verifier` (`roles/firebaseappcheck.tokenVerifier`) IAM szerepkör biztosítása.

### 7.3. Rate Limiter Tranzakció és korlátai (`FIREBASE-FN-003`)
- Az `enforceRateLimit` ellenőrzést **önálló tranzakcióban**, az üzleti (pl. foglalási) tranzakció **előtt** kell végrehajtani (`REF-FN-CALLABLE-001`).
- **Kritikus implementációs szabály:** Soha ne dobj kivételt (`throw new HttpsError(...)`) a rate-limit `db.runTransaction` callbackjén belül, amikor a limitet túllépték, mert a kivétel megszakítja és visszagörgeti (rollback) a tranzakciót! Ehelyett növeld a `blockedCount` mezőt, frissítsd az `expiresAt` (TTL) mezőt, térj vissza `false` értékkel, hogy a tranzakció sikeresen commitoljon, és a tranzakció **után** dobj `resource-exhausted` hibát.
- **Ismert korlátozás:** A Firestore-alapú per-UID számláló kis és közepes forgalomra alkalmas, de magas konkurencia (>1 kérés/mp/user-action dokumentum) esetén document contentiont és tranzakciós költséget (1 olvasás + 1 írás ellenőrzésenként) okoz, valamint önmagában nem véd a többfiókos (sybil) támadások ellen. Publikus vagy nagy terhelésű (L3) végpontokon kombináld App Checkkel, kvótákkal és Redis/Cloud Armor alapú védelemmel.

### 7.4. Atomi Üzleti Tranzakció és a két tranzakció eltérő felelőssége (`FIREBASE-FN-004`)
- Az üzleti állapotváltozást – pl. az időpont-kapacitás ellenőrzését és növelését (`slots`), a bérlet érvényességének ellenőrzését és levonását (`passes`), a foglalás létrehozását (`bookings`), az append-only tranzakciós naplóbejegyzést (`passLedger`) és a SHA-256 `requestHash`-sel védett idempotencia-rekordot (`_idempotency`) – **egyetlen atomi `db.runTransaction` műveletben** kell végrehajtani (`REF-FN-CALLABLE-001`).
- **Eltérő tranzakciós felelősség:** A rate limiter tranzakció (`FIREBASE-FN-003`) és az üzleti tranzakció (`FIREBASE-FN-004`) felelőssége szigorúan szétválasztott. A rate limiter állapota (az elhasznált kéréskvóta) szándékosan **nem gördül vissza**, ha a későbbi üzleti tranzakció üzleti előfeltétel-hiba (pl. betelt időpont vagy lejárt bérlet) miatt elutasításra kerül.
- A tranzakció callback újrafuttatható (retry), ezért **tilos** benne külső API-hívást, e-mail-küldést vagy nem idempotens mellékhatást végezni.

---

## 8. Idempotencia és append-only ledger (`FIREBASE-IDEMP-001`)

Kritikus mutációs műveletnél (`REF-FN-CALLABLE-001`):
- Az `idempotencyKey` legyen kellően véletlenszerű (pl. UUID) és a felhasználóhoz/művelethez kötött (`_idempotency/${uid}_${idempotencyKey}`).
- Tárold a `uid`, `action`, `requestHash` (a művelet és a bemeneti paraméterek SHA-256 lenyomata), `result`, `createdAt` és `expiresAt` mezőket.
- Azonos `idempotencyKey` + **azonos `requestHash`** esetén ne végezd el újra a levonást, hanem add vissza az eltárolt eredményt (`deduplicated: true`).
- Azonos `idempotencyKey` + **eltérő `requestHash`** (paraméter-manipuláció) esetén utasítsd el a kérést (`already-exists` / `failed-precondition`).
- A `passLedger` / tranzakciós napló legyen **append-only**, kliens számára írás tiltva (`allow create, update, delete: if false;`), az egyenlegváltozást (`delta`) és az új egyenleget (`balanceAfter`) a szerver számítsa ugyanabban a tranzakcióban.
- **Firestore TTL szabály:** Automatikus törlést (TTL policy az `expiresAt` mezőn) kizárólag technikai `_idempotency` és `_rateLimits` kollekciókra szabad beállítani; jogi, számviteli vagy üzleti megőrzésű `ledger` kollekcióra **tilos** automatikus TTL törlést alkalmazni!
- A SHA-256 `requestHash` nem helyettesíti az autorizációt, az input canonicalizálását vagy a kulcsok megfelelő scope-olását.

---

## 9. App Check és API-key védelem (`FIREBASE-APPCHK-001`)

- Az App Check a kliens/alkalmazáspéldány eredetét ellenőrző védelmi réteg, nem user authentication és nem authorization.
- Enforcement bevezetése előtt mérd fel az érvényes és nem ellenőrzött forgalmat monitorozási módban, majd rollout után kapcsold be az enforce módot.
- A publikus Firebase `apiKey`-t a Google Cloud Console-ban HTTP Referrer és engedélyezett API scope szerint korlátozni kell, de tilos titokként vagy jogosultsági határként kezelni.

---

## 10. Cloud Storage és külső OAuth tokenek (`FIREBASE-STORAGE-001`)

- A Storage path legyen user-/tenant-scope-olt; a `storage.rules` ellenőrizze az autentikációt, jogosultságot, fájlméretet (pl. `< 5 MB`) és a megengedett `contentType`-ot.
- A kliens által megadott MIME-type (`contentType`) könnyen hamisítható. L2/L3 esetén érzékeny feltöltéseknél szerveroldali **magic-byte (bináris fájlszignatúra)** ellenőrzés, szükség szerint malware scanning, képfeldolgozás/újrakódolás és biztonságos `Content-Disposition` fejléc kötelező.
- SVG, HTML, script és végrehajtható tartalom feltöltését tiltsd, vagy soha ne szolgáld ki ugyanazon originről aktív tartalomként.
- A letöltési URL-eket (download tokens) kezelj úgy, mint egy bearer hozzáférési tokent.
- **Külső OAuth tokenek tárolása (pl. Google Drive, Photos, Calendar):** A legszűkebb OAuth scope-ot kérd. Az alkalmazásszintű `client_secret` Secret Managerben tárolandó. A felhasználónkénti OAuth `refresh_token` értékeket kliens elől zárolt (`allow read, write: if false;`) szerveroldali Firestore kollekcióban, alkalmazásszinten (pl. Google Cloud KMS envelope encryptionnel) titkosítva kell tárolni.

---

## 11. PWA, Firestore persistence és kijelentkezési korlátok (`FIREBASE-PWA-001`)

A PWA kliensoldali adattárolás és kijelentkezés (`REF-PWA-LOGOUT-001`) tervezésekor és dokumentálásakor az alábbi **5 normatív szabályt és korlátot** kötelező érvényesíteni:

1. **Munkamenet-állapot érvénytelenítése:** Kijelentkezéskor (logout) minden elérhető böngészőfülön érvényteleníteni kell a felhasználói munkamenethez tartozó alkalmazásszintű állapotot (`sessionStorage`, memóriabeli state, nem statikus Cache Storage).
2. **A `clearIndexedDbPersistence()` valós korlátai:** A tartósan tárolt érzékeny adatokat külön kell kezelni; a `clearIndexedDbPersistence(db)` **nem tekinthető forenzikus lemezfelülírásnak**, hibát dob, ha a Firestore egy másik nyitott fülön még aktív, és **véglegesen eldobja a még nem szinkronizált offline írásokat (pending writes)**.
3. **A `memoryLocalCache()` szerepe:** Érzékeny személyes/üzleti adatok és megosztott eszközök esetén alapértelmezetten memóriabeli cache (`memoryLocalCache()`) használandó tartós IndexedDB helyett. A `memoryLocalCache()` csökkenti a tartós Firestore-cache kockázatát megosztott eszközökön, de **nem jelent teljes adattörlési garanciát**, és nem helyettesíti az alkalmazásszintű adatvédelmi intézkedéseket.
4. **Korábban megjelenített adatok korlátja:** A logout nem garantálhatja egy korábban már megjelenített vagy megszerzett adat visszavonását, illetve a böngészőből/eszközről való teljes fizikai eltávolítását.
5. **Best-effort többtabos koordináció (`BroadcastChannel` + `TAB_ID`):** Mivel a `BroadcastChannel` azonos origin példányai között üzen (és a küldő fül saját másodlagos listener példánya is megkaphatná az üzenetet), minden fülnek egyedi `TAB_ID` azonosítót kell használnia, saját üzenetét (`senderTabId === TAB_ID`) figyelmen kívül kell hagynia, és a másodlagos füleknek csak a sikeres `await terminate(db)` után szabad `LOGOUT_TAB_ACK` (hiba esetén `LOGOUT_TAB_ERROR`) választ küldeniük. **Fontos korlátozás:** Globális tab-regiszter nélkül a `BroadcastChannel` + `TAB_ID` + `LOGOUT_TAB_ACK` / `LOGOUT_TAB_ERROR` + timeout protokoll **best-effort koordináció** marad, és tilos garantált törlésként dokumentálni.
- A Service Worker kizárólag a statikus App Shell asseteket cache-elheti; hitelesített API választ vagy személyes adatot alapértelmezetten tilos cache-elnie.

---

## 12. IAM, secrets és naplózás (`FIREBASE-IAM-001`)

- A Cloud Functions dedikált vagy elkülönített, minimális jogosultságú (least-privilege) service accounttal fusson; ne támaszkodj túl tág jogosultságú alapértelmezett Compute Engine service accountra.
- A backend titkokat kizárólag Google Cloud Secret Managerben (`defineSecret`) tárold; tilos őket `.env` fájlban gitbe commitolni, frontend bundle-be, source mapbe vagy logba helyezni.
- Hosszú életű service account JSON kulcsfájlok helyett használj ADC-t és Workload Identity Federationt.
- A strukturált naplózás (`logger.info` / `logger.error`) csak szükséges eseményt és szűrt hibametaadatot (`errorName`, `errorCode`) tartalmazzon. Jelszó, token, session ID, teljes request body, nyers exception dump, fizetési adat és szükségtelen PII naplózása tilos.
- L2/L3 szinten állíts be Cloud Monitoring log-alapú riasztást: ismétlődő Firestore Rules `permission-denied` és `resource-exhausted` hibákra, hirtelen olvasási/hívási kiugrásokra, valamint admin Custom Claims (`setCustomUserClaims`) vagy manuális egyenleg/bérlet módosításokra.

---

## 13. Mentés, helyreállítás és költségvédelem (`FIREBASE-BACKUP-001`)

- A backup stratégiát a Tier és az RPO/RTO alapján válaszd meg (L1: rendszeres export; L2: ütemezett Cloud Storage export + kritikus állapotra PITR; L3: kötelező Firestore PITR + rendszeres restore drill).
- A backup nem tekinthető működőnek dokumentált visszaállítási teszt (restore test) nélkül.
- A PITR, a Firestore export és a Cloud Storage backup nem egymás szinonimái; megőrzésük, költségük és visszaállítási módjuk eltér.
- A Denial-of-Wallet kockázatot `maxInstances`, App Check, per-UID/per-IP rate limit, napi/havi felhasználói kvóták és Budget Alert együttes alkalmazásával csökkentsd.

---

## 14. Firebase release ellenőrzőlista (`FIREBASE-CHK-001`)

- [ ] L1/L2/L3 szint és A/B/C onboarding modell dokumentálva (`SECURITY_PROFILE.md`).
- [ ] Auth providerek, *Authorized domains*, signup korlátozás és `email_verified` ellenőrizve (`FIREBASE-AUTH-001`).
- [ ] App Check monitorozva/enforce-olva; `consumeAppCheckToken: true` esetén kliens `limitedUseAppCheckTokens: true` és `roles/firebaseappcheck.tokenVerifier` IAM szerepkör beállítva (`FIREBASE-FN-002`).
- [ ] Firestore és Storage Rules default deny, `hasOnly`, típus/méret és immutabilitás érvényesítve, emulátoros pozitív/negatív tesztekkel (`REF-TEST-RULES-001`) igazolva (`FIREBASE-RULES-001`..`003`).
- [ ] Admin SDK hozzáférés IAM és szerverkód szinten külön negatív tesztekkel (`REF-TEST-ADMIN-001`) ellenőrizve (`FIREBASE-ADMIN-001`).
- [ ] Kritikus üzleti műveletben a rate-limit tranzakció (`FIREBASE-FN-003`) és az atomi üzleti + `requestHash` idempotencia + append-only `ledger` tranzakció (`FIREBASE-FN-004`) szétválasztva és tesztelve.
- [ ] Firestore TTL csak `_idempotency` és `_rateLimits` kollekciókon aktív (`FIREBASE-IDEMP-001`).
- [ ] Secret Manager, least-privilege service account és PII-mentes strukturált logolás ellenőrizve (`FIREBASE-IAM-001`).
- [ ] PWA `memoryLocalCache()` vagy `TAB_ID`-izolált best-effort többtabos logout (`REF-PWA-LOGOUT-001`) tesztelve (`FIREBASE-PWA-001`).
- [ ] Backup/restore és Cloud Monitoring/Budget riasztások ellenőrizve (`FIREBASE-BACKUP-001`).

---

## 15. Hivatalos források

A pontos URL-eket, ellenőrzési dátumokat (`last_verified`) és verzióstátuszt lásd a `10_SOURCE_REGISTER.md` modulban:
- Firebase Security Checklist: https://firebase.google.com/support/guides/security-checklist
- Firebase App Check for Cloud Functions: https://firebase.google.com/docs/app-check/cloud-functions
- Firestore Security Rules Get Started: https://firebase.google.com/docs/firestore/security/get-started
- Firestore Rules Structure & Limits: https://firebase.google.com/docs/firestore/security/rules-structure
- Secure Firestore Queries: https://firebase.google.com/docs/firestore/security/rules-query
- Firebase Custom Claims: https://firebase.google.com/docs/auth/admin/custom-claims
- Firebase Pricing: https://firebase.google.com/pricing
- Firebase Local Emulator Suite & Rules Unit Testing: https://firebase.google.com/docs/rules/unit-tests
~~~