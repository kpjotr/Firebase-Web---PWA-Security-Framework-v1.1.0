---
name: identity-access-security
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
---

# 04 – Identity & Access Security

## 1. Hatókör (`IAM-SCOPE-001`)

Ez a modul (`Tier-1` szelektív doménmodul a `CORE-LOAD-001` szerint) a Firebase Authentication, az Identity Platform, az OAuth 2.0 / OpenID Connect (OIDC), az email/password hitelesítés, az MFA, a session lifecycle, a Custom Claims, a role/permission modell, az account recovery és az adminisztratív hozzáférés biztonsági követelményeit határozza meg.

Ezt a modult minden olyan projektnél kötelező betölteni, ahol felhasználói fiók, bejelentkezés, regisztráció, admin felület, szerepkör, védett erőforrás vagy külső identitásszolgáltató létezik.

---

## 2. Identitás és jogosultság szétválasztása (`IAM-SEP-001`)

A négy alapvető védelmi réteget tilos összekeverni vagy egymással helyettesíteni (`CORE-PRIN-007`, `CORE-PRIN-012`):
- **Authentication (Hitelesítés):** Ki a felhasználó? (pl. Firebase Authentication / Identity Platform).
- **Authorization (Jogosultságkezelés):** Mit tehet az adott hitelesített felhasználó az adott erőforrással? (Firestore/Storage Security Rules és Cloud Functions szerveroldali ellenőrzés).
- **App Check (Alkalmazás-hitelesítés):** A kérés várhatóan a regisztrált alkalmazáspéldányból érkezik-e? (Nem igazol felhasználói identitást és nem jogosultságkezelés!).
- **Google Cloud IAM:** A szerveroldali szolgáltatásfiók (Service Account) milyen Google Cloud / Firebase erőforrásokhoz férhet hozzá?

A sikeres bejelentkezés (`request.auth != null`) soha nem jelent automatikus hozzáférést minden rekordhoz!

---

## 3. Regisztráció, Onboarding és Kvóták (`IAM-LIFE-001`)

- Az **A (Admin-provisioned)**, **B (Invite-only)** vagy **C (Public signup)** onboarding modellt (`CORE-ONBOARD-001`) a projekt elején rögzíteni kell a `SECURITY_PROFILE.md`-ben, és ténylegesen konfigurálni kell a Firebase Console / Identity Platform felületen (`FIREBASE-AUTH-001`).
- Public signup (Model C) esetén kötelező a rate limit, az abuse-védelem, az App Check, az Email Enumeration Protection, a kötelező e-mail-verifikáció (`request.auth.token.email_verified == true`) és a minimális alapjog (zero-privilege default).
- Invite-only (Model B) esetén a meghívó token legyen kriptográfiailag véletlen, egyszer használatos, rövid élettartamú, célfelhasználóhoz/tenanthez kötött és szerveroldalon ellenőrzött (pl. `beforeUserCreated` blocking functionnel vagy tranzakciós meghívó-beváltással).
- **Identity Platform kvóták és díjazási határok (`FIREBASE-LIMIT-001`):**
  - **Spark csomag (`[PLAN QUOTA]`):** **3 000 DAU** napi aktív felhasználói korlát a Spark (no-cost) csomagban az érintett Identity Platform funkciókra (pl. blocking functions, TOTP MFA).
  - **Blaze csomag (`[PRICING / FREE TIER ALLOWANCE]`):** **50 000 MAU** díjmentes keret sztenderd providerekre és TOTP MFA-ra; szolgáltatói kategóriák szerint eltérő díjazással (az SMS MFA üzenetenként, a SAML/OIDC pedig külön díjköteles; kiadás előtt mindig ellenőrizd az aktuális hivatalos Firebase Pricing oldalt!).
- A fióktiltás, törlés és szerepkör-visszavonás kizárólag szerveroldali művelet lehet; a kliens soha nem módosíthat saját privilegizált mezőt.
- Az account recovery (jelszó-visszaállítási) folyamat biztonsági szintje nem lehet gyengébb, mint a normál hitelesítésé.

---

## 4. Jelszavak, hitelesítők és MFA (`IAM-CRED-001`)

- Soha ne tárolj saját adatbázisban (Firestore) plaintext vagy visszafejthető jelszót.
- Firebase Authentication által kezelt credential esetén tilos párhuzamos saját jelszóadatbázist implementálni.
- Ha egyedi hitelesítő vagy másodlagos PIN/kód kezelése elkerülhetetlen, a **NIST SP 800-63-4** és az **OWASP Authentication Cheat Sheet** alapján tervezz: erős jelszó/kód követelmény, kompromittált jelszavak szűrése (breached password screening), memóriaköltséges hash algoritmus (Argon2id) egyedi salttal, szigorú rate limit és biztonságos visszaállítás.
- Ne kényszeríts indokolatlan periodikus jelszócserét kompromittálódás gyanúja vagy incidens nélkül (NIST SP 800-63-4 előírás).
- **Multi-Factor Authentication (MFA):** Kockázat és szerepkör szerint kötelezővé kell tenni, de **legalább minden adminisztratív és privilegizált hozzáférésnél (L2/L3) KÖTELEZŐ (`MUST`)**.
- A TOTP MFA, a passkey/WebAuthn és az SMS MFA eltérő biztonsági (assurance) és adathalászat-ellenállási (phishing-resistance) szintet nyújt; ne kezeld őket egyenértékűnek, és kerüld az SMS MFA-t ott, ahol TOTP vagy WebAuthn használható.

---

## 5. OAuth 2.0 és OpenID Connect (`IAM-OAUTH-001`)

- Public client (SPA/PWA) esetén használj **Authorization Code Flow + PKCE** protokollt (**RFC 9700 – OAuth 2.0 Security Best Current Practice**).
- Alkalmazz `state` (CSRF védelem) és `nonce` (replay védelem) paramétereket az adott protokoll és SDK előírásai szerint.
- A Redirect URI legyen pontos allowlist, bármiféle wildcard nélkül.
- A kért OAuth `scope` legyen a feladathoz szükséges legszűkebb (pl. `drive.file` a teljes Drive hozzáférés helyett – `FIREBASE-STORAGE-001`).
- Az access és refresh tokenek kliensoldali kezelését bízd a hivatalos platform SDK-ra; tilos őket logolni vagy analitikába küldeni.
- Az alkalmazásszintű OAuth `client_secret` soha nem kerülhet frontendbe; tárold Secret Managerben, a felhasználónkénti külső OAuth `refresh_token`-eket pedig zárolt szerveroldali kollekcióban, alkalmazásszintű (pl. Cloud KMS) titkosítással (`FIREBASE-STORAGE-001`).
- Az OIDC ID token a felhasználó azonosítására szolgál, nem általános külső API access tokenként.
- Szerveroldali token-validáció során mindig ellenőrizd az `iss` (issuer), `aud` (audience), kriptográfiai aláírás, `exp` (lejárat), `nonce`/`state` és az elfogadott algoritmus (`alg`) követelményeit a hivatalos szolgáltatói könyvtárral.

---

## 6. Firebase Auth tokenek és Custom Claims (`IAM-CLAIMS-001`)

- **Méretkorlát (`FIREBASE-LIMIT-001`):** A Custom Claims payload mérete legfeljebb **1 000 byte** felhasználónként (`[PLATFORM HARD LIMIT]`). Csak kis méretű, ritkán változó jogosultsági adatot (pl. `role: 'admin' | 'staff'`, `tenantId`) tárolj benne; tilos teljes felhasználói profilt Custom Claims-be írni.
- **Propagációs késleltetés:** A Custom Claims szerveroldali frissítése (`setCustomUserClaims`) **nem jelenik meg azonnal** a már kiadott (legfeljebb 1 óráig érvényes) ID tokenekben; csak új token kérésekor (`user.getIdToken(true)`) lép életbe.
- Jogosultság vagy admin szerepkör azonnali visszavonásakor ne várj a token természetes lejáratára: kombináld a claim-frissítést a refresh tokenek visszavonásával (`admin.auth().revokeRefreshTokens(uid)`), a `checkRevoked: true` ellenőrzéssel vagy egy szerveroldali/Rules oldali aktív státuszellenőrzéssel (`FIREBASE-AUTH-001`).
- A kliens által request payloadban küldött `role` vagy `isAdmin` érték soha nem megbízható; a Rules-ban a `request.auth.token.role`, a Cloud Functions-ben pedig a hitelesített `request.auth.token` használandó.

---

## 7. Session életciklus és kijelentkezés (`IAM-SESS-001`)

- Különböztesd meg a kliensoldali `signOut(auth)` hívást, a szerveroldali refresh token visszavonást (`revokeRefreshTokens`), a JWT ID token érvényességi idejét (1 óra) és a szerveroldali session cookie-t (`__Host-` prefixű Host-Only cookie).
- Érzékeny fiókműveletek (jelszócsere, e-mail-cím módosítása, fióktörlés, MFA módosítás, admin szerepkör-változtatás) előtt követelj meg friss hitelesítést (**recent authentication / step-up auth** az `auth_time` claim ellenőrzésével).
- Megosztott eszközön ne használj hosszú életű tartós (persistent) munkamenetet vagy tartós IndexedDB cache-t indokolatlanul (`memoryLocalCache()` preferált).
- A többtabos PWA kijelentkezést és cache-takarítást a `02_FIREBASE_SECURITY.md` (`FIREBASE-PWA-001`) és a `11_REFERENCE_TEMPLATES_FIREBASE.md` (`REF-PWA-LOGOUT-001`) szerinti `TAB_ID`-izolált `BroadcastChannel` best-effort protokollal valósítsd meg, és soha ne dokumentáld forenzikus adattörlésként!

---

## 8. Authorization modell (`IAM-AUTHZ-001`)

- Minden projekthez készíts explicit szerepkör–jogosultság–erőforrás mátrixot (`get`, `list`, `create`, `update`, `delete` és callable műveletek) a `SECURITY_PROFILE.md`-ben.
- A szerepköralapú hozzáférés (RBAC) mellett mindig ellenőrizd a rekordtulajdonost (`resource.data.userId == request.auth.uid`) és a tenant-hatókört (`tenantId`) is (ABAC / BOLA védelem).
- Minden API és callable művelet ellenőrizze a hívó jogosultságát szerveroldalon (`REF-FN-CALLABLE-001`).
- Soha ne támaszkodj frontend route-elrejtésre, disabled gombra vagy kliensoldali role checkre.
- Alkalmazz Deny by Default elvet (`CORE-PRIN-003`).
- BOLA / IDOR megelőzése: a kliens által megadott `docId` vagy `passId` erőforrás-azonosítót szerveroldalon tranzakcióban mindig kösd a hitelesített `request.auth.uid`-hoz (`pass.userId === request.auth.uid`).

---

## 9. Admin és privilegizált hozzáférés (`IAM-ADMIN-001`)

- Az adminisztratív fiókok különösen védettek; L2 és L3 szinten az **Admin MFA kötelező**.
- Tilos közös (shared) admin fiókot használni; minden adminisztrátornak egyedi, auditálható fiókkal kell rendelkeznie.
- Az adminisztratív műveletek legyenek strukturáltan naplózottak (`Cloud Logging`) és riasztással bekötöttek (`FIREBASE-IAM-001`).
- A gépi `Service Account` identitásokat és az emberi admin identitásokat szigorúan külön kell kezelni.
- **Google Sign-In vs. Admin MFA (`FIREBASE-AUTH-001`):** Fogyasztói `@gmail.com` Google OAuth bejelentkezés önmagában **nem bizonyít** 2SV/MFA meglétet! Google Sign-In csak akkor fogadható el admin MFA-ként, ha a fiók menedzselt **Google Workspace** szervezethez tartozik kötelezően kikényszerített 2SV policy-vel, szerveroldali e-mail allowlisttel és `admin: true` Custom Claimmel. Egyéb esetben Identity Platform TOTP MFA (`request.auth.token.firebase.sign_in_second_factor != null`) kötelező.

---

## 10. Kötelező Identity & Access tesztek (`IAM-TEST-001`)

Minden releváns funkcióhoz pozitív és negatív (`assertFails` / `HttpsError`) tesztekkel (`REF-TEST-RULES-001`, `REF-TEST-ADMIN-001`) kell igazolni az alábbi eseteket:
- hitelesítetlen (anonim) felhasználó elutasítása;
- nem verifikált e-mail című (`email_verified: false`) felhasználó elutasítása;
- normál felhasználó hozzáférése saját erőforrásához (pozitív);
- másik felhasználó erőforrás-azonosítójával (`userId` / `passId`) történő hozzáférési vagy módosítási kísérlet (BOLA -> elutasítás);
- másik tenant erőforrásához való hozzáférési kísérlet (elutasítás);
- lejárt vagy érvénytelen token elutasítása;
- visszavont (revoked) refresh token vagy visszavont szerepkör kezelése;
- hiányzó MFA vagy hiányzó step-up authentication elutasítása privilegizált műveletnél;
- meghívó (invite) újrafelhasználásának vagy lejárt meghívónak az elutasítása;
- admin végpont meghívása normál felhasználói tokennel (BFLA -> elutasítás);
- account recovery folyamat visszaélési (rate limit / enumeration) tesztje.

---

## 11. Hivatalos források (`IAM-REF-001`)

A pontos verzióstátuszt és ellenőrzési dátumokat (`last_verified`) lásd a `10_SOURCE_REGISTER.md` modulban:
- NIST SP 800-63-4 Digital Identity Guidelines: https://csrc.nist.gov/pubs/sp/800/63/4/final
- Firebase Authentication Docs: https://firebase.google.com/docs/auth
- Firebase Custom Claims Docs: https://firebase.google.com/docs/auth/admin/custom-claims
- Firebase Multi-Factor Authentication (MFA): https://firebase.google.com/docs/auth/web/multi-factor
- OAuth 2.0 Security Best Current Practice (RFC 9700): https://www.rfc-editor.org/rfc/rfc9700
- OpenID Connect Core 1.0: https://openid.net/specs/openid-connect-core-1_0.html
- OWASP Authentication Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- OWASP Authorization Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html