---
name: frontend-web-pwa-security
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
scope: HTML, CSS, JavaScript, TypeScript, React, browser, PWA
---

# 03 – Frontend Security: HTML, CSS, JavaScript, TypeScript, React és PWA

## 1. Cél és hatókör (`FRONT-SCOPE-001`)

A frontend nem biztonsági határ (`CORE-PRIN-005`), de elsődleges támadási felület. Ez a modul (`Tier-1` szelektív doménmodul a `CORE-LOAD-001` szerint) a böngészőben futó összes kódra, markupra, stílusra, kliensoldali adattárolásra és PWA-erőforrásra vonatkozik. Minden frontend- vagy statikus weboldal-feladatnál kötelező a `01_SECURITY_CORE.md` és (kódmódosítás esetén) a `08_AI_AGENT_CODING_SECURITY.md` modul mellett.

---

## 2. HTML és DOM (`FRONT-HTML-001`, `FRONT-DOMXSS-001`)

### 2.1. Kötelező HTML és attribútum szabályok (`FRONT-HTML-001`)
- A felhasználói vagy külső adatot alapértelmezetten szövegként jelenítsd meg, ne HTML-ként.
- Ne használj `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `document.write` vagy React `dangerouslySetInnerHTML` API-t nem megbízható bemenettel.
- Ha rich text/HTML megjelenítése üzletileg elkerülhetetlen, használj karbantartott, szigorúan konfigurált sanitizer könyvtárat (pl. DOMPurify), szigorú elem- és attribútum-allowlistet, és ahol támogatott, **Trusted Types** policy-t. A sanitizer önmagában nem teszi biztonságossá az összes kontextust.
- URL-t csak validálás és engedélyezett scheme/origin (`https:`, `mailto:`, `tel:`) ellenőrzése után használj navigációra vagy erőforrásbetöltésre. A `javascript:`, `vbscript:` és a nem várt `data:` URL-ek használata tilos (`MUST NOT`).
- Külső, új lapon (`target="_blank"`) nyíló linkeknél használj `rel="noopener noreferrer"` attribútumot, ha az `opener` kapcsolat nem szükséges.
- `iframe` esetén adj explicit `sandbox` attribútumot és minimális `allow` jogosultságot; az `allow-same-origin` + `allow-scripts` kombinációt csak indokolt, dokumentált esetben használd, mert azonos originen a beágyazott tartalom eltávolíthatja saját sandbox korlátozását.
- Ne bízz `data-*` attribútumba, hidden inputba, `disabled`/`readonly` mezőbe vagy kliensoldali validációba authorization célból.
- Form inputoknál mindig legyen szerveroldali (vagy L1 esetén Firestore Rules szintű) validáció is; a frontend validáció kizárólag UX célokat szolgál.

### 2.2. DOM XSS megelőzése (`FRONT-DOMXSS-001`)
- Kerüld a stringből kódot végrehajtó API-kat (`eval`, `new Function`, `setTimeout("...")`, `setInterval("...")`).
- Ne fűzz felhasználói adatot HTML-, JavaScript-, CSS- vagy URL-környezetbe kontextushelyes kódolás (output encoding) nélkül.
- Template engine vagy Markdown renderer használata esetén ellenőrizd és tiltsd le a nyers (raw) HTML támogatást, vagy alkalmazz utólagos sanitizálást.
- A DOM-ba kerülő adat forrását (`source`: pl. `location.search`, `location.hash`, `postMessage`, `localStorage`) és célját (`sink`) kockázat szerint vizsgáld.

---

## 3. CSS biztonság (`FRONT-CSS-001`)

A CSS önmagában nem jogosultsági réteg. Kötelező biztonsági ellenőrzési pontok:
- Külső CSS, `@import`, `url()` és fontforrás csak ismert, szükséges originről származhat.
- Ne injektálj nem megbízható értéket CSS stringbe, selectorba vagy `style` attribútumba.
- Ne használj CSS-t titkos adatok vagy DOM-attribútumok kiszivárogtatására alkalmas selector/URL mintákkal (CSS exfiltration); egy külső `url()` betöltése adatot szivárogtathat.
- Ne támaszkodj vizuális elrejtésre (`display: none`, `visibility: hidden`, `opacity: 0`, off-screen pozicionálás) jogosultságkezelésként: amit a felhasználó nem láthat, azt a szerver/adatbázis se küldje le a kliensnek!
- A CSP `style-src` lehetőleg nonce/hash alapú legyen; az `'unsafe-inline'` style csak indokolt, dokumentált kivételként használható.
- Külső fontoknál (pl. Google Fonts CDN) mérlegeld a privacy (IP-cím továbbítás), tracking, availability és supply-chain kockázatot; ahol indokolt, alkalmazz self-hostingot.

---

## 4. JavaScript és TypeScript (`FRONT-JSTS-001`)

- Minden kliensoldali kód és bundle nyilvánosnak tekintendő. Tilos benne secretet, privát API kulcsot, service account credentialt, signing key-t vagy admin tokent elhelyezni (`CORE-PRIN-005`).
- A `localStorage`, `sessionStorage`, `IndexedDB` és `Cache Storage` tartalma sikeres XSS vagy helyi eszközhozzáférés esetén teljes egészében kiolvasható. Ne tárolj bennük jelszót, hosszú életű titkot vagy szükségtelen érzékeny személyes adatot (PII).
- A Firebase Auth SDK tokenkezelését ne kerüld meg saját, nem ellenőrzött tokenrendszerrel.
- Külső ablakok közötti `window.postMessage` kommunikációnál mindig ellenőrizd az `event.origin`-t, az `event.source`-t, az üzenet sémáját és típusát; érzékeny adathoz soha ne használj `'*'` targetOrigint.
- `window.opener`, `BroadcastChannel`, `WebSocket` és Web/Service Worker üzeneteknél explicit trust boundary és runtime input validáció szükséges.
- Ne használj `Math.random()`-ot biztonsági tokenhez, meghívókódhoz vagy idempotencia-kulcshoz; kriptográfiai véletlenhez `crypto.getRandomValues()`, `crypto.randomUUID()` vagy szerveroldali CSPRNG használandó.
- Ne implementálj saját kriptográfiai algoritmust; használd a böngésző (`Web Crypto API`) vagy a Node.js beépített, auditált primitívjeit.
- A TypeScript típusellenőrzés fordítási idejű (compile-time), nem futásidejű (runtime) validáció! Külső vagy hálózati adatot futásidőben is validálj.
- Prototype pollution ellen kerüld az ellenőrizetlen rekurzív objektum-összefésülést (deep merge); használj mező-allowlistet, `Object.hasOwn` ellenőrzést vagy `Object.create(null)` struktúrákat.
- Production buildben tilos auth tokent, személyes adatot (PII), teljes request payloadot vagy session azonosítót a böngésző konzoljába (`console.log`) írni.

---

## 5. React-specifikus szabályok (`FRONT-REACT-001`)

- A JSX text node alapértelmezett escapingje fontos védelmi réteg, de nem teljes körű XSS-megoldás (pl. `href`, `src` attribútumokban vagy `dangerouslySetInnerHTML`-ben nem véd).
- A `dangerouslySetInnerHTML` használata alapértelmezetten tilos, kivéve dokumentált, sanitizált és lehetőség szerint Trusted Types-szel védett use case esetén.
- A React state, route guard, Context, komponens-láthatóság vagy kliensoldali role check **nem authorization** (`CORE-PRIN-012`). A szerver (Cloud Functions) és a Firebase Security Rules minden műveletet önállóan újra ellenőriz.
- Ne renderelj nem megbízható URL-t `href`, `src`, `action`, CSS vagy router navigation prop értékeként protokoll- és origin-validáció nélkül.
- React Server Components (RSC), SSR, hydration, router és framework-specifikus adatbetöltés esetén a szerver/kliens trust boundary-t külön modellezd, hogy szerveroldali titok vagy belső adatmodell ne szivárogjon át a kliensre szerializált props-ban.
- Soha ne tegyél titkot `VITE_*`, `REACT_APP_*`, `NEXT_PUBLIC_*` vagy más kliensoldali build-prefixű környezeti változóba!
- Az Error Boundary és a kliensoldali hibakezelés a felhasználónak csak általános hibaüzenetet mutasson; ne jelenítsen meg stack trace-t, secretet vagy belső rendszerkonfigurációt (`CORE-PRIN-009`).

---

## 6. CSP, HTTP security headers és böngészővédelem (`FRONT-CSP-001`, `FRONT-HDR-001`)

### 6.1. Content Security Policy (`FRONT-CSP-001`)
- Bevezetéskor kezdj `Content-Security-Policy-Report-Only` móddal, gyűjtsd össze a valós szükséges forrásokat, majd szigoríts kikényszerített (enforce) `Content-Security-Policy` módra.
- Kerüld az `'unsafe-inline'`, `'unsafe-eval'`, `*` és a túl széles wildcardok (pl. teljes `*.googleapis.com`) használatát a `script-src` és `connect-src` direktívákban.
- Preferáld a nonce/hash-alapú script policy-t, az `object-src 'none'`, `base-uri 'self'`, `frame-ancestors 'none'` (vagy `'self'`) és a szolgáltatás-specifikus `connect-src` értékeket.
- A Firebase végpontokat csak az adott projektben ténylegesen használt szolgáltatások és régiók alapján add hozzá a `connect-src`-hez:
  - Auth: `https://identitytoolkit.googleapis.com https://securetoken.googleapis.com`
  - Firestore: `https://firestore.googleapis.com`
  - App Check: `https://content-firebaseappcheck.googleapis.com`
  - Storage: `https://firebasestorage.googleapis.com`
  - Functions: `https://<region>-<project-id>.cloudfunctions.net` (vagy a v2 Cloud Run URL)
- A `frame-ancestors` és az `X-Frame-Options` viselkedését összehangoltan állítsd be és teszteld.
- A CSP önmagában nem helyettesíti az output encodingot, a sanitizálást vagy a dependency review-t (`CORE-PRIN-007`).

### 6.2. További HTTP biztonsági fejlécek (`FRONT-HDR-001`)
A `firebase.json` `hosting.headers` szekciójában konfiguráld:
- `Strict-Transport-Security` (HSTS): csak HTTPS-en; az `includeSubDomains` és `; preload` opciókat kizárólag akkor add hozzá, ha minden aldomain tartósan HTTPS-only.
- `X-Content-Type-Options: nosniff`.
- `Referrer-Policy`: legyen adatminimalizáló (pl. `strict-origin-when-cross-origin` vagy `same-origin`).
- `Permissions-Policy`: tiltsd le a nem használt böngésző-képességeket, és csak a ténylegesen szükségeseket engedélyezd (pl. `camera=(self), microphone=(), geolocation=()`).
- Cookie session használata esetén: `Secure`, `HttpOnly`, megfelelő `SameSite` (`Strict` vagy `Lax`), **Host-Only** (`Domain` attribútum elhagyása) és `__Host-` prefix (`FIREBASE-BOUND-001`). A Firebase kliens SDK tokenjei nem azonosak a szerveroldali cookie sessionnel.
- A CORS nem authorization; soha ne használj tetszőleges `Origin` visszatükrözést `Access-Control-Allow-Credentials: true` mellett.

---

## 7. PWA, Service Worker és Kijelentkezési Korlátok (`FRONT-PWA-001`)

A PWA kliensoldali tárolás és kijelentkezés implementálásakor kötelező betölteni a `11_REFERENCE_TEMPLATES_FIREBASE.md` modul `REF-PWA-LOGOUT-001` sablonját, és betartani a `FIREBASE-PWA-001` szabályait:
- A Service Worker kizárólag a szükséges statikus App Shell asseteket cache-elje; személyes, hitelesített vagy dinamikus API-választ tilos cache-elnie explicit indok és kontroll nélkül.
- Alkalmazz cache-verziózást (`app-shell-static-vX`) és invalidációt; kijelentkezéskor töröld az összes nem statikus cache-t.
- Cache poisoning és cross-user cache leakage ellen a cache key nem hagyhatja figyelmen kívül a jogosultsági kontextust.
- A Service Worker üzeneteit (`message` event) és `fetch` eseményeit nem megbízható bemenetként kezeld; csak ismert `message.type` és origin/client ellenőrzés után hajts végre műveletet.
- A Service Worker update folyamatánál gondoskodj arról, hogy egy sebezhető régi klienskód ne maradjon indokolatlanul aktív.
- Offline queue használata esetén dokumentáld a `SECURITY_PROFILE.md`-ben, milyen művelet maradhat pending állapotban, hogyan történik az újraküldés és az idempotencia, valamint mi történik a szinkronizálatlan írásokkal kijelentkezéskor.
- **Az 5 kötelező PWA kijelentkezési és cache-korlát (`FIREBASE-PWA-001`, `REF-PWA-LOGOUT-001`):**
  1. Logoutkor minden elérhető fülön érvényteleníteni kell a felhasználói munkamenethez tartozó alkalmazásszintű állapotot.
  2. A `clearIndexedDbPersistence(db)` **nem forenzikus lemezfelülírás**, hibát adhat, ha más fülön aktív a Firestore, és **véglegesen eldobja a még nem szinkronizált offline írásokat**.
  3. Érzékeny adat és megosztott eszköz esetén részesítsd előnyben a `memoryLocalCache()` használatát, de dokumentáld, hogy ez sem jelent teljes adattörlési garanciát.
  4. A logout nem garantálja a korábban már megjelenített/megszerzett adat fizikai eltávolítását a böngészőből.
  5. Globális tab-regiszter nélkül a `BroadcastChannel` + `TAB_ID` + `LOGOUT_TAB_ACK` / `LOGOUT_TAB_ERROR` + timeout protokoll **best-effort koordináció** marad.

---

## 8. Fájlfeltöltés és letöltés a frontendről (`FRONT-FILE-001`)

- Az `<input type="file" accept="...">` attribútum és a kliensoldali MIME-ellenőrzés csak UX; a szerveroldali és `storage.rules` validáció kötelező (`FIREBASE-STORAGE-001`).
- Érvényesíts méret-, kiterjesztés-, MIME- és (L2/L3 szinten szerveroldali) magic-byte ellenőrzést; a veszélyes formátumokat (HTML, SVG, végrehajtható fájlok) zárd ki.
- Soha ne jeleníts meg felhasználó által feltöltött fájlt aktív HTML/SVG-ként ugyanazon originről.
- A `URL.createObjectURL()` által létrehozott Blob URL-eket `URL.revokeObjectURL()` hívással szabadítsd fel, amikor már nincs rájuk szükség.
- A letöltött fájlnév ne legyen alkalmas path traversalra vagy fejléc-injekcióra; szerveroldalon normalizáld.
- Ne használj `data:` URL-t nagy vagy érzékeny payloadhoz indokolatlanul.

---

## 9. Külső erőforrások és Subresource Integrity (`FRONT-SRI-001`)

- Külső (harmadik féltől származó) scriptet lehetőleg ne használj; ha elkerülhetetlen, alkalmazz rögzített verziót, megbízható origint, **Subresource Integrity (SRI)** `integrity` hasht és megfelelő `crossorigin` attribútumot.
- Az SRI nem alkalmazható minden dinamikus CDN- vagy ES-modulbetöltési mintára; ne állítsd, hogy minden külső dependency védett SRI-vel, ha ez technikailag nem áll fenn.
- Részesítsd előnyben a kritikus frontend-függőségek és fontok self-hostingját, és ellenőrizd a csomag kiadóját (`06_DEVSECOPS_SUPPLY_CHAIN.md`).
- A third-party analytics és tag manager scriptek teljes JavaScript-kódvégrehajtási jogosultságot kapnak az adott originen; bevezetésük előtt külön biztonsági és adatvédelmi (`07_DATA_PROTECTION_PRIVACY.md`) kockázatértékelés kötelező.

---

## 10. Frontend ellenőrzőlista (`FRONT-CHK-001`)

- [ ] nincs frontend secret vagy privát kulcs a kódban, `VITE_*`/`REACT_APP_*`/`NEXT_PUBLIC_*` változóban vagy bundle-ben (`FRONT-JSTS-001`, `FRONT-REACT-001`);
- [ ] nincs nem megbízható bemenetet fogadó `innerHTML` / `dangerouslySetInnerHTML` (`FRONT-HTML-001`);
- [ ] minden dinamikus URL és `postMessage` origin/séma ellenőrzött (`FRONT-HTML-001`, `FRONT-JSTS-001`);
- [ ] szolgáltatás-specifikus CSP és HTTP security headers (`firebase.json`) konfigurálva és tesztelve (`FRONT-CSP-001`, `FRONT-HDR-001`);
- [ ] hitelesített API-válasz vagy PII nem kerül Service Worker cache-be (`FRONT-PWA-001`);
- [ ] kliensoldali storage (`localStorage`/`IndexedDB`) tartalma minimalizált; PWA kijelentkezés követi a `REF-PWA-LOGOUT-001` best-effort protokollt (`FRONT-PWA-001`);
- [ ] a React route guardok mögött valódi szerveroldali / Firestore Rules jogosultság-ellenőrzés áll (`FRONT-REACT-001`);
- [ ] a külső scriptek és SRI / self-hosting döntések felülvizsgáltak (`FRONT-SRI-001`);
- [ ] a fájlfeltöltéseket a szerver és a Storage Rules validálja (`FRONT-FILE-001`);
- [ ] az XSS / DOM XSS tesztek lefutottak (`09_SECURITY_TESTING_THREAT_MODELING.md`).

---

## 11. Hivatalos hivatkozások (`FRONT-REF-001`)

A pontos verzióstátuszt és ellenőrzési dátumokat (`last_verified`) lásd a `10_SOURCE_REGISTER.md` modulban:
- React DOM Common Components & Props: https://react.dev/reference/react-dom/components/common
- MDN Content Security Policy (CSP): https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
- MDN Trusted Types API: https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API
- MDN Subresource Integrity (SRI): https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity
- MDN Service Worker API: https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API
- WHATWG HTML Living Standard: https://html.spec.whatwg.org/
- ECMAScript Specification: https://tc39.es/ecma262/