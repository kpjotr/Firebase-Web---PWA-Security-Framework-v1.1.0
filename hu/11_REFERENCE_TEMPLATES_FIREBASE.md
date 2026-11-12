---
name: firebase-reference-templates
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
tier_layer: Tier-2 (Implementation & Testing Reference Layer)
default_verification_level: DOCUMENTATION_REVIEWED
---

# 11 – Firebase Referencia-Implementációk és Tesztsablonok (`REF-MOD-001`)

## 1. Szerep, Betöltési Szabály és Ellenőrzési Szintek (`REF-MOD-001`, `REF-VERIFY-001`)

### 1.1. Mikor kell betölteni ezt a modult? (`CORE-LOAD-001`, `REF-MOD-001`)
Ez a modul a keretrendszer **Tier-2 (Implementációs réteg)** szintű eleme. Tisztán elméleti vagy magas szintű architekturális tervezésnél nem kell betölteni (a kontextusablak kímélése érdekében), azonban **kötelezően betöltendő**, amikor az AI-ágens vagy fejlesztő:
1. `firestore.rules` szabályokat vagy kliensoldali Firestore lekérdezéseket ír/módosít (`REF-RULES-BASE-001`);
2. Cloud Functions v2 (`onCall`) tranzakciós üzleti végpontot, rate limitert, idempotenciát vagy ledgert implementál (`REF-FN-CALLABLE-001`);
3. PWA kliensoldali kijelentkezést, `BroadcastChannel` szinkronizációt vagy Firestore cache-kezelést készít (`REF-PWA-LOGOUT-001`);
4. Firestore Security Rules emulátoros egységteszteket ír (`REF-TEST-RULES-001`);
5. Szerveroldali Admin SDK / Cloud Functions handler jogosultsági (negatív) teszteket készít (`REF-TEST-ADMIN-001`).

**Normatív figyelmeztetés (`REF-MOD-001`):** Az itt szereplő kódminták nem vakon bemásolható snippetek, hanem **strukturált referencia-implementációk (`REFERENCE / MINTA`)**. Alkalmazás előtt az ágensnek mindig ellenőriznie kell a célprojekt tényleges adatmodelljével, üzleti invariánsaival és a `package-lock.json` fájlban rögzített SDK-verziókkal való kompatibilitást!

---

### 1.2. A Referencia-Implementációk Ellenőrzési Szintjeinek Szabályozása (`REF-VERIFY-001`)

A dokumentumba illesztett kódminta önmagában nem minősül futtatott vagy élesben tesztelt kódnak. Minden referenciasablonnál és minden generált/átvett implementációnál kötelező az alábbi **5 ellenőrzési szint (`Verification Level`)** szigorú megkülönböztetése:

| Ellenőrzési Státusz (`Verification Level`) | Normatív Jelentés és Alkalmazási Feltétel |
| :--- | :--- |
| **`DOCUMENTATION_REVIEWED`** | Dokumentációs és logikai áttekintés történt; futtatás vagy emulátoros végrehajtás a generálás során nem történt. *(A `v1.1.0` dokumentumcsomagban szereplő sablonok alapértelmezett, valós státusza!)* |
| **`STATIC_CHECKED`** | Valós statikus elemzés, típusellenőrzés (`tsc`) vagy szintaktikai linter futtatás megtörtént; ez nem bizonyítja a futásidejű helyességet. |
| **`EMULATOR_TESTED`** | A megfelelő Firebase Emulator Suite környezetben a megjelölt pozitív és negatív tesztek ténylegesen lefutottak és sikeresek voltak. |
| **`PROJECT_TESTED`** | A minta konkrét célprojektben, annak tényleges SDK-, lockfile- és konfigurációs környezetével éles/staging vagy CI környezetben ellenőrzésre került. |
| **`NOT_VERIFIED`** | Semmilyen ellenőrzés nem történt. |

**Szigorú tilalom (`REF-VERIFY-001`):** A tényleges státuszt, verziókat, tesztkörnyezetet és dátumot (`Last Verified`) kizárólag valós ellenőrzés alapján szabad kitölteni! Az AI által generált vagy e modulból átvett kódminta önmagában **nem jogosít** `STATIC_CHECKED`, `EMULATOR_TESTED` vagy `PROJECT_TESTED` státuszra. A célprojektben való alkalmazáskor a projektnek kell elvégeznie és a `CODER_TO_REVIEWER` / `REVIEWER_TO_PLANNER` szerződésekben bizonyítékkal igazolnia a `STATIC_CHECKED` -> `EMULATOR_TESTED` -> `PROJECT_TESTED` validációt.

---

## 2. Sablon 1: `REF-RULES-BASE-001` – Firestore Security Rules (v2) és Kliens Query Minta

### Kötelező 7-elemű Metaadat-fejléc
1. **Template ID:** `REF-RULES-BASE-001`
2. **Applicability (`Alkalmazási feltételek`):** L1, L2 és L3 Firebase projektek. Tartalmazza a globális fallback default deny szabályt, az L1 kliens által írható saját naplókollekciót (`workouts`), valamint az L2/L3 kliensírás elől teljesen zárolt üzleti kollekciókat (`passes`, `passLedger`, `bookings`).
3. **Target SDK Baseline (`Kompatibilitási referencia-alapvonal`):** Firestore Security Rules `rules_version = '2'`, kliensoldalon `firebase@^11.x` (Modular Web SDK). *Nem automatikus verziókényszer; a célprojekt `package-lock.json` fájljával egyeztetendő!*
4. **Security Invariants (`Biztonsági invariánsok`):**
   - Minden nem illeszkedő útvonal alapértelmezetten tiltott (`FIREBASE-RULES-001`).
   - Kritikus üzleti kollekciók (`passes`, `passLedger`, `bookings`) közvetlen kliensoldali létrehozása, módosítása és törlése tiltott (`FIREBASE-RULES-002`).
   - Kliens által írható dokumentumoknál (`workouts`) kötelező a tulajdonosi egyezés (`isOwner`), a `hasOnly` mező-allowlist, a szigorú típus- és méretellenőrzés, a szerveroldali időbélyeg (`request.time`), valamint `update` esetén a `userId` és `createdAt` mezők megváltoztathatatlansága (immutabilitása).
   - Listázó (`list`) lekérdezéseknél kötelező a tulajdonosi/admin jogosultság és a `request.query.limit <= 50` (`[FRAMEWORK DEFAULT]` – `FIREBASE-RULES-003`).
5. **Required Tests (`Kötelező pozitív és negatív tesztek`):** Lásd `REF-TEST-RULES-001` (saját és admin olvasás, jogosulatlan olvasás/írás, `userId` módosítás, `hasOnly` megsértése, szűrő nélküli `unfilteredQuery`, limit nélküli `noLimitQuery` és limitet túllépő `excessiveQuery` elutasítása).
6. **Known Limitations (`Ismert korlátozások`):**
   - A Firestore-ban az egymást átfedő `allow` szabályok logikai `OR`-ként értékelődnek ki; egy szülőútvonalon elhelyezett tág engedély felülírja a gyermekútvonal tiltását.
   - A kliensoldali `.limit(50)` önmagában **nem biztonsági kontroll**, mert a kliens által küldött lekérdezés módosítható. A Firestore Security Rules nem általános lekérdezéslimitáló mechanizmus, hanem a lekérdezés által elérhető adatokra vonatkozó hozzáférési feltételeket (`request.query.limit <= 50`) érvényesíti. A `limit <= 50` egy `[FRAMEWORK DEFAULT]` ajánlás, amelyet önálló jogosultsági vagy költségvédelmi garanciának tekinteni tilos; költséges vagy érzékeny lekérdezéseknél szerveroldali korlátozást, kvótát vagy rate limitet is alkalmazni kell (`FIREBASE-RULES-003`).
   - A Rules **nem védi** a Firebase Admin SDK-val végzett szerveroldali műveleteket (`FIREBASE-ADMIN-001`).
7. **Verification Level & Last Verified (`Ellenőrzési szint és dátum`):** `DOCUMENTATION_REVIEWED` | Utolsó dokumentációs felülvizsgálat: `2026-11-03` *(Célprojektben emulátoros futtatás szükséges az `EMULATOR_TESTED` szinthez!)*

### Referencia-kód (`firestore.rules` és Kliens Lekérdezés)

~~~javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Segédfüggvények
    function isAuthenticated() {
      return request.auth != null && request.auth.token.email_verified == true;
    }
    function isOwner(userId) {
      return isAuthenticated() && request.auth.uid == userId;
    }
    // Custom Claim alapú admin ellenőrzés (Nem igényel extra Firestore get() olvasást,
    // de a claim-változás csak az ID token frissülésekor lép életbe! Max. 1000 byte [PLATFORM HARD LIMIT])
    function isAdmin() {
      return isAuthenticated() && request.auth.token.role == 'admin';
    }

    // 1. FALLBACK DEFAULT DENY minden nem illeszkedő útvonalra (FIREBASE-RULES-001).
    // (Figyelem: az átfedő allow szabályok logikai OR-ként működnek;
    // soha ne tegyél túl tág allow szabályt szülőkollekcióra!)
    match /{document=**} {
      allow read, write: if false;
    }

    // 2. KRITIKUS ÜZLETI KOLLEKCIÓK (L2/L3: Passes, Ledger, Bookings) -> Kliensírás TILTVA! (FIREBASE-RULES-002)
    match /passes/{passId} {
      allow get: if isOwner(resource.data.userId) || isAdmin();
      // request.query.limit <= 50: [FRAMEWORK DEFAULT] (FIREBASE-RULES-003)
      allow list: if (isOwner(resource.data.userId) || isAdmin()) && request.query.limit <= 50;
      allow create, update, delete: if false; // Kizárólag Cloud Functions v2 (Admin SDK) írhatja!
    }

    match /passLedger/{entryId} {
      allow get: if isOwner(resource.data.userId) || isAdmin();
      allow list: if (isOwner(resource.data.userId) || isAdmin()) && request.query.limit <= 50;
      allow create, update, delete: if false; // Append-only ledger, kizárólag szerver írhatja!
    }

    match /bookings/{bookingId} {
      allow get: if isOwner(resource.data.userId) || isAdmin();
      allow list: if (isOwner(resource.data.userId) || isAdmin()) && request.query.limit <= 50;
      allow create, update, delete: if false; // Kizárólag Cloud Functions v2 (Admin SDK) írhatja!
    }

    // 3. FELHASZNÁLÓI SAJÁT NAPLÓ (L1 vagy izolált saját napló) -> Szigorúan séma-validált kliensírás
    match /workouts/{workoutId} {
      allow get: if isOwner(resource.data.userId) || isAdmin();
      allow list: if (isOwner(resource.data.userId) || isAdmin()) && request.query.limit <= 50;

      allow create: if isOwner(request.resource.data.userId)
        && request.resource.data.keys().hasOnly(['userId', 'title', 'notes', 'durationMinutes', 'createdAt', 'updatedAt'])
        && request.resource.data.title is string && request.resource.data.title.size() > 0 && request.resource.data.title.size() <= 100
        && request.resource.data.notes is string && request.resource.data.notes.size() <= 2000
        && request.resource.data.durationMinutes is int && request.resource.data.durationMinutes > 0 && request.resource.data.durationMinutes <= 600
        && request.resource.data.createdAt == request.time
        && request.resource.data.updatedAt == request.time;

      allow update: if isOwner(resource.data.userId)
        && request.resource.data.keys().hasOnly(['userId', 'title', 'notes', 'durationMinutes', 'createdAt', 'updatedAt'])
        && request.resource.data.userId == resource.data.userId // A tulajdonos (userId) megváltoztathatatlan!
        && request.resource.data.createdAt == resource.data.createdAt // A létrehozás ideje megváltoztathatatlan!
        && request.resource.data.title is string && request.resource.data.title.size() > 0 && request.resource.data.title.size() <= 100
        && request.resource.data.notes is string && request.resource.data.notes.size() <= 2000
        && request.resource.data.durationMinutes is int && request.resource.data.durationMinutes > 0 && request.resource.data.durationMinutes <= 600
        && request.resource.data.updatedAt == request.time;

      allow delete: if isOwner(resource.data.userId) || isAdmin();
    }
  }
}
~~~

**Kötelező Kliensoldali Lekérdezési Minta (Mivel a Security Rules nem szűrő!):**
~~~typescript
import { Firestore, collection, query, where, orderBy, limit, getDocs } from "firebase/firestore";

// A kliens lekérdezésnek EXPLICIT tartalmaznia kell a userId szűrőt és a limit(<=50) záradékot,
// különben a Firestore Rules a teljes lekérdezést permission-denied hibával elutasítja!
export async function fetchMyWorkouts(db: Firestore, currentUid: string) {
  const q = query(
    collection(db, "workouts"),
    where("userId", "==", currentUid),
    orderBy("createdAt", "desc"),
    limit(25)
  );
  return await getDocs(q);
}
~~~

---

## 3. Sablon 2: `REF-FN-CALLABLE-001` – Cloud Functions v2 (`onCall`) Két Külön Tranzakcióval

### Kötelező 7-elemű Metaadat-fejléc
1. **Template ID:** `REF-FN-CALLABLE-001`
2. **Applicability (`Alkalmazási feltételek`):** L2 és L3 rendszerek minden olyan üzleti mutációja, amely szűkös kapacitást (`slots`), egyenleget/bérletet (`passes`), foglalást (`bookings`) vagy többdoku-mentumos állapotváltozást kezel.
3. **Target SDK Baseline (`Kompatibilitási referencia-alapvonal`):** `firebase-functions@^6.x` (2nd Gen / v2 `onCall`), `firebase-admin@^13.x`, `zod@^3.x`, kliensoldalon `firebase@^11.x`. *A célprojekt `package-lock.json` fájljával egyeztetendő!*
4. **Security Invariants (`Biztonsági invariánsok`):**
   - Hitelesített és verifikált e-mail című hívó (`request.auth` és `email_verified === true`); a `uid` kizárólag a hitelesített tokenből származhat (`FIREBASE-ADMIN-001`).
   - Strict input validáció (`Zod .strict()`), amely minden extra mezőt elutasít.
   - **1. Tranzakció (`enforceRateLimit` – `FIREBASE-FN-003`):** Önálló tranzakcióban, az üzleti tranzakció *előtt* fut; túllépés esetén növeli és commitolja a `blockedCount` mezőt, majd a tranzakció *után* dob `resource-exhausted` hibát.
   - **2. Tranzakció (Üzleti / Foglalási tranzakció – `FIREBASE-FN-004`):** Egyetlen atomi `db.runTransaction` műveletben ellenőrzi és módosítja az idempotencia-rekordot (`_idempotency` SHA-256 `requestHash`-sel), a bérletet (`passes`), az időpontot (`slots`), a foglalást (`bookings`) és az append-only tranzakciós naplót (`passLedger`).
   - **Szétválasztott tranzakciós felelősség:** Ha a 2. (üzleti) tranzakció üzleti előfeltétel-hiba (pl. lejárt bérlet vagy betelt időpont) miatt elbukik, az 1. (rate limiter) tranzakcióban elhasznált kéréskvóta szándékosan **nem gördül vissza**.
5. **Required Tests (`Kötelező pozitív és negatív tesztek`):** Lásd `REF-TEST-ADMIN-001` (hitelesítetlen hívás, nem verifikált e-mail, BOLA más felhasználó `passId`-jával, rate-limit megőrzése üzleti hiba esetén, `requestHash` mismatch elutasítása).
6. **Known Limitations (`Ismert korlátozások`):**
   - A `consumeAppCheckToken: true` (Replay Protection) extra ellenőrzési round-trippel és késleltetéssel jár; L3-on és különösen érzékeny L2 végpontokon alkalmazandó, és megköveteli a kliensoldali `{ limitedUseAppCheckTokens: true }` hívást, valamint a szerveroldali `roles/firebaseappcheck.tokenVerifier` IAM szerepkört (`FIREBASE-FN-002`).
   - A Firestore-alapú per-UID rate limiter (`_rateLimits`) kis és közepes forgalomra alkalmas, de magas konkurencia (>1 kérés/mp/user-action dokumentum) esetén document contentiont és tranzakciós költséget okoz, valamint nem véd a többfiókos (sybil) támadások ellen (`FIREBASE-FN-003`).
   - A `_rateLimits` és `_idempotency` kollekciókra az `expiresAt` mező alapján Firestore TTL policy-t kell konfigurálni; a `passLedger` kollekcióra tilos TTL-t beállítani (`FIREBASE-IDEMP-001`).
7. **Verification Level & Last Verified (`Ellenőrzési szint és dátum`):** `DOCUMENTATION_REVIEWED` | Utolsó dokumentációs felülvizsgálat: `2026-11-03`

### Referencia-kód (Szerveroldali Cloud Function v2 és Kliensoldali Meghívás)

~~~typescript
import { onCall, HttpsError, CallableRequest } from "firebase-functions/v2/https";
import * as logger from "firebase-functions/logger";
import { getFirestore, FieldValue, Timestamp } from "firebase-admin/firestore";
import * as crypto from "crypto";
import { z } from "zod";

const db = getFirestore();

// 1. Strict Zod séma (.strict() azonnal elutasítja a nem várt/extra mezőket!)
export const CreateBookingSchema = z.object({
  slotId: z.string().min(1).max(64),
  passId: z.string().min(1).max(64),
  idempotencyKey: z.string().uuid(),
}).strict();

// 2. ELSŐ TRANZAKCIÓ: Firestore Per-UID Rate Limiter (FIREBASE-FN-003)
// Kritikus szabály: TILOS kivételt dobni a tranzakció callbackjén belül limit-túllépéskor,
// mert a throw visszagörgeti (rollback) a tranzakciót! Előbb commitold a növelt blockedCount
// értéket, és a tranzakció UTÁN dobj HttpsError("resource-exhausted") hibát.
// Konfigurálj Firestore TTL policy-t a `_rateLimits.expiresAt` mezőre!
export async function enforceRateLimit(
  uid: string,
  action: string,
  maxRequests = 10,
  windowSeconds = 60
): Promise<void> {
  const ref = db.doc(`_rateLimits/${uid}_${action}`);
  const now = Timestamp.now();
  const expiresAt = Timestamp.fromMillis(now.toMillis() + windowSeconds * 1000 * 2);

  const isAllowed = await db.runTransaction(async (tx) => {
    const snap = await tx.get(ref);
    const data = snap.data();

    if (!snap.exists || !data || now.seconds - data.windowStart.seconds >= windowSeconds) {
      tx.set(ref, { count: 1, blockedCount: 0, windowStart: now, expiresAt });
      return true;
    }
    if (data.count >= maxRequests) {
      tx.update(ref, {
        blockedCount: FieldValue.increment(1),
        lastBlockedAt: now,
        expiresAt,
      });
      return false;
    }
    tx.update(ref, { count: FieldValue.increment(1), expiresAt });
    return true;
  });

  if (!isAllowed) {
    throw new HttpsError("resource-exhausted", "Too many requests. Please wait before retrying.");
  }
}

// 3. Handler-logika kiemelése a közvetlen egységtesztelhetőséghez (REF-TEST-ADMIN-001)
export async function handleCreateBooking(request: CallableRequest<unknown>) {
  // A. Hitelesítés ellenőrzése (v2: request.auth)
  if (!request.auth || request.auth.token.email_verified !== true) {
    throw new HttpsError("unauthenticated", "Authenticated and email-verified account required.");
  }
  const uid = request.auth.uid;

  // B. Bemenet validálása (Zod .strict())
  const parsed = CreateBookingSchema.safeParse(request.data);
  if (!parsed.success) {
    throw new HttpsError("invalid-argument", "Invalid request payload.");
  }
  const { slotId, passId, idempotencyKey } = parsed.data;

  // C. 1. TRANZAKCIÓ: Rate Limit érvényesítése (független az üzleti tranzakciótól!)
  await enforceRateLimit(uid, "createBooking", 10, 60);

  // D. SHA-256 requestHash számítása a paraméter-manipuláció megakadályozására
  const requestHash = crypto
    .createHash("sha256")
    .update(JSON.stringify({ uid, action: "createBooking", slotId, passId }))
    .digest("hex");

  const idempotencyRef = db.doc(`_idempotency/${uid}_${idempotencyKey}`);
  const slotRef = db.doc(`slots/${slotId}`);
  const passRef = db.doc(`passes/${passId}`);
  const bookingRef = db.collection("bookings").doc();
  const ledgerRef = db.collection("passLedger").doc();

  try {
    // E. 2. TRANZAKCIÓ: Atomi Üzleti (Foglalási) Tranzakció (FIREBASE-FN-004)
    const result = await db.runTransaction(async (tx) => {
      // 1. Idempotencia és requestHash ellenőrzése
      const idempSnap = await tx.get(idempotencyRef);
      if (idempSnap.exists) {
        const idempData = idempSnap.data()!;
        if (idempData.requestHash !== requestHash) {
          throw new HttpsError(
            "already-exists",
            "Idempotency key was already used with different request parameters!"
          );
        }
        return { bookingId: idempData.result.bookingId, deduplicated: true };
      }

      const [slotSnap, passSnap] = await Promise.all([tx.get(slotRef), tx.get(passRef)]);
      if (!slotSnap.exists) throw new HttpsError("not-found", "Requested time slot not found.");
      if (!passSnap.exists) throw new HttpsError("not-found", "Requested pass not found.");

      const slot = slotSnap.data()!;
      const pass = passSnap.data()!;
      const nowMs = Date.now();

      // 2. Tulajdonjog (BOLA védelem), státusz és időbeli invariánsok ellenőrzése
      if (pass.userId !== uid) {
        throw new HttpsError("permission-denied", "Cannot use another customer's pass.");
      }
      if (pass.status !== "ACTIVE" || pass.remainingUses <= 0 || pass.expiresAt.toMillis() <= nowMs) {
        throw new HttpsError("failed-precondition", "Pass is inactive, expired, or has no remaining uses.");
      }
      if (slot.status !== "OPEN" || slot.startTime.toMillis() <= nowMs || slot.bookedCount >= slot.capacity) {
        throw new HttpsError("failed-precondition", "Time slot is closed, in the past, or full.");
      }

      const newRemainingUses = pass.remainingUses - 1;
      const ttlExpiresAt = Timestamp.fromMillis(nowMs + 48 * 3600 * 1000); // 48 órás TTL az idempotencia-rekordra

      // 3. Atomi írások (Pass + Slot + Booking + Append-Only Ledger + Idempotency Record)
      tx.update(passRef, {
        remainingUses: FieldValue.increment(-1),
        updatedAt: FieldValue.serverTimestamp(),
      });
      tx.update(slotRef, {
        bookedCount: FieldValue.increment(1),
        updatedAt: FieldValue.serverTimestamp(),
      });
      tx.set(bookingRef, {
        userId: uid,
        slotId,
        passId,
        status: "CONFIRMED",
        createdAt: FieldValue.serverTimestamp(),
      });
      // Kötelező append-only ledger bejegyzés (TTL nélkül!)
      tx.set(ledgerRef, {
        userId: uid,
        passId,
        bookingId: bookingRef.id,
        action: "DEBIT_BOOKING",
        delta: -1,
        balanceAfter: newRemainingUses,
        createdAt: FieldValue.serverTimestamp(),
      });
      tx.set(idempotencyRef, {
        uid,
        action: "createBooking",
        requestHash,
        result: { bookingId: bookingRef.id },
        createdAt: FieldValue.serverTimestamp(),
        expiresAt: ttlExpiresAt,
      });

      return { bookingId: bookingRef.id, deduplicated: false };
    });

    logger.info("Booking transaction succeeded", {
      uid,
      bookingId: result.bookingId,
      slotId,
      deduplicated: result.deduplicated,
    });
    return result;
  } catch (error: any) {
    if (error instanceof HttpsError) throw error;
    // Szűrt strukturált hibametaadat naplózása nyers exception dump helyett
    logger.error("Booking transaction failed", {
      uid,
      slotId,
      errorName: error?.name || "UnknownError",
      errorCode: error?.code || "unknown",
    });
    throw new HttpsError("internal", "Booking could not be processed.");
  }
}

// 4. 2nd Gen (v2) Védett Callable Végpont Regisztrációja
// Előfeltétel consumeAppCheckToken: true esetén: a Service Account rendelkezzen
// a `roles/firebaseappcheck.tokenVerifier` IAM szerepkörrel! (FIREBASE-FN-002)
export const createBooking = onCall(
  {
    region: "europe-west1",
    maxInstances: 10,
    enforceAppCheck: true,
    consumeAppCheckToken: true, // Kockázatalapon L3 és kritikus L2 mutációkon (kliens limitedUseAppCheckTokens: true szükséges!)
  },
  handleCreateBooking
);
~~~

**Kötelező Kliensoldali Meghívási Minta (`limitedUseAppCheckTokens: true`):**
~~~typescript
import { getFunctions, httpsCallable } from "firebase/functions";

// Amikor a szerveren a consumeAppCheckToken: true aktív, a 3. opciós paraméter KÖTELEZŐ!
export async function callCreateBooking(slotId: string, passId: string, idempotencyKey: string) {
  const functions = getFunctions(undefined, "europe-west1");
  const createBookingFn = httpsCallable(
    functions,
    "createBooking",
    { limitedUseAppCheckTokens: true }
  );
  const response = await createBookingFn({ slotId, passId, idempotencyKey });
  return response.data;
}
~~~

---

## 4. Sablon 3: `REF-PWA-LOGOUT-001` – Best-Effort PWA Kijelentkezés és `TAB_ID`-Izolált Többtabos Cache-Takarítás

### Kötelező 7-elemű Metaadat-fejléc
1. **Template ID:** `REF-PWA-LOGOUT-001`
2. **Applicability (`Alkalmazási feltételek`):** Minden olyan L1/L2/L3 SPA és PWA alkalmazás, ahol a felhasználó kijelentkezhet, különösen ha több böngészőfül lehet nyitva, vagy ha offline Firestore persistence használatban van.
3. **Target SDK Baseline (`Kompatibilitási referencia-alapvonal`):** `firebase@^11.x` (`firebase/auth`, `firebase/firestore`), modern böngésző `BroadcastChannel` és `CacheStorage` API. *A célprojekt `package-lock.json` fájljával egyeztetendő!*
4. **Security Invariants (`Biztonsági invariánsok`):**
   - Minden böngészőfül egyedi `TAB_ID` azonosítót kap, így a kijelentkezést kezdeményező fül soha nem dolgozza fel a saját maga által küldött `FORCE_LOGOUT` üzenetet (self-echo védelem).
   - A másodlagos fülek először meghívják az `await terminate(db)` függvényt az IndexedDB lock feloldására, és csak ezután küldenek `LOGOUT_TAB_ACK` (vagy hiba esetén `LOGOUT_TAB_ERROR`) üzenetet.
   - Kijelentkezéskor törlődik a `sessionStorage` és a nem statikus (`app-shell-static-` prefixen kívüli) Service Worker Cache Storage.
5. **Required Tests (`Kötelező pozitív és negatív tesztek`):**
   - Pozitív teszt: több nyitott fül esetén a `FORCE_LOGOUT` üzenetre a másodlagos fülek `LOGOUT_TAB_ACK`-val válaszolnak, a `terminate(db)` és a `signOut(auth)` lefut, majd minden fül `/login`-ra navigál.
   - Negatív teszt: a kezdeményező fül figyelmen kívül hagyja a saját `senderTabId === TAB_ID` üzenetét; ha egy másodlagos fül `LOGOUT_TAB_ERROR`-t küld vagy nem válaszol a `tabAckTimeoutMs` határidőn belül, a kijelentkezés nem akad el végtelen ciklusban, hanem a timeout után végrehajtja a `signOut(auth)`-ot és a kemény oldal-újratöltést.
6. **Known Limitations (`Ismert korlátozások – Kötelezően dokumentálandó!`, `FIREBASE-PWA-001`):**
   1. Logoutkor minden elérhető fülön érvényteleníteni kell a felhasználói munkamenethez tartozó alkalmazásszintű állapotot.
   2. A tartósan tárolt érzékeny adatokat külön kell kezelni; a `clearIndexedDbPersistence()` **nem tekinthető forenzikus lemezfelülírásnak**, és **véglegesen eldobja a még nem szinkronizált offline írásokat**.
   3. A `memoryLocalCache()` csökkenti a tartós Firestore-cache kockázatát megosztott eszközökön, de **nem jelent teljes adattörlési garanciát**, és nem helyettesíti az alkalmazásszintű adatvédelmi intézkedéseket.
   4. A logout nem garantálhatja egy korábban már megjelenített vagy megszerzett adat visszavonását vagy a böngészőből való teljes fizikai eltávolítását.
   5. Globális tab-regiszter nélkül a `BroadcastChannel` + `TAB_ID` + `LOGOUT_TAB_ACK` / `LOGOUT_TAB_ERROR` + timeout protokoll **best-effort koordináció** marad.
7. **Verification Level & Last Verified (`Ellenőrzési szint és dátum`):** `DOCUMENTATION_REVIEWED` | Utolsó dokumentációs felülvizsgálat: `2026-11-03`

### Referencia-kód (`logout.ts`)

~~~typescript
import { Auth, signOut } from "firebase/auth";
import { Firestore, terminate, clearIndexedDbPersistence } from "firebase/firestore";

const LOGOUT_CHANNEL_NAME = "app_security_logout_channel";

// Minden böngészőfül egyedi azonosítót kap, hogy a kezdeményező fül saját BroadcastChannel
// példányai SOHA ne dolgozzák fel a saját maguk által küldött FORCE_LOGOUT üzenetet!
const TAB_ID =
  typeof crypto !== "undefined" && "randomUUID" in crypto
    ? crypto.randomUUID()
    : `tab_${Date.now()}_${Math.random().toString(36).slice(2)}`;

let isInitiatingLogout = false;

// Alkalmazás indításakor egyszer meghívandó, hogy a másodlagos fülek azonnal leállítsák
// a Firestore példányukat és LOGOUT_TAB_ACK (vagy hiba esetén LOGOUT_TAB_ERROR) választ küldjenek!
export function initMultiTabLogoutListener(db: Firestore): void {
  if (typeof window !== "undefined" && "BroadcastChannel" in window) {
    const channel = new BroadcastChannel(LOGOUT_CHANNEL_NAME);
    channel.onmessage = async (event) => {
      const msg = event.data;
      if (!msg || msg.type !== "FORCE_LOGOUT" || msg.senderTabId === TAB_ID || isInitiatingLogout) {
        return;
      }

      try {
        sessionStorage.clear();
        await terminate(db); // A másodlagos fül felszabadítja az IndexedDB zárolást
        channel.postMessage({
          type: "LOGOUT_TAB_ACK",
          senderTabId: TAB_ID,
          targetTabId: msg.senderTabId,
        });
      } catch (err: any) {
        channel.postMessage({
          type: "LOGOUT_TAB_ERROR",
          senderTabId: TAB_ID,
          targetTabId: msg.senderTabId,
          errorCode: err?.code || "terminate-failed",
        });
      } finally {
        channel.close();
        window.location.replace("/login");
      }
    };
  }
}

export async function performLogoutCleanup(
  auth: Auth,
  db: Firestore,
  customIndexedDbNames: string[] = [],
  tabAckTimeoutMs = 400
): Promise<void> {
  isInitiatingLogout = true;

  try {
    // 1. Másodlagos fülek értesítése és ACK/ERROR válaszok begyűjtése (best-effort koordináció)
    if (typeof window !== "undefined" && "BroadcastChannel" in window) {
      const channel = new BroadcastChannel(LOGOUT_CHANNEL_NAME);
      await new Promise<void>((resolve) => {
        let settleTimer: ReturnType<typeof setTimeout> | null = null;
        const maxTimer = setTimeout(() => {
          if (settleTimer) clearTimeout(settleTimer);
          resolve();
        }, tabAckTimeoutMs);

        channel.onmessage = (event) => {
          const msg = event.data;
          if (!msg || msg.senderTabId === TAB_ID || msg.targetTabId !== TAB_ID) {
            return;
          }

          if (msg.type === "LOGOUT_TAB_ACK" || msg.type === "LOGOUT_TAB_ERROR") {
            if (msg.type === "LOGOUT_TAB_ERROR") {
              console.warn(`Secondary tab (${msg.senderTabId}) reported Firestore terminate error:`, msg.errorCode);
            }
            if (settleTimer) clearTimeout(settleTimer);
            settleTimer = setTimeout(() => {
              clearTimeout(maxTimer);
              resolve();
            }, 80);
          }
        };

        channel.postMessage({
          type: "FORCE_LOGOUT",
          senderTabId: TAB_ID,
        });
      });
      channel.close();
    }

    // 2. Kijelentkezés a Firebase Authenticationből
    await signOut(auth);

    // 3. Kezdeményező fül Firestore példányának leállítása és az IndexedDB persistence törlése
    // (Figyelem: a még nem szinkronizált offline írások itt elvesznek, és ez nem forenzikus törlés!)
    await terminate(db);
    await clearIndexedDbPersistence(db);

    // 4. Egyedi alkalmazásspecifikus IndexedDB adatbázisok törlése
    for (const dbName of customIndexedDbNames) {
      indexedDB.deleteDatabase(dbName);
    }

    // 5. sessionStorage és nem statikus Cache Storage bejegyzések törlése
    sessionStorage.clear();
    if ("caches" in window) {
      const cacheNames = await caches.keys();
      await Promise.all(
        cacheNames
          .filter((name) => !name.startsWith("app-shell-static-"))
          .map((name) => caches.delete(name))
      );
    }
  } catch (cleanupError) {
    console.warn("Client storage cleanup encountered a non-fatal lock or error:", cleanupError);
  } finally {
    // 6. Kemény navigáció a memóriabeli állapot teljes alaphelyzetbe állításához
    window.location.replace("/login");
  }
}
~~~

---

## 5. Sablon 4: `REF-TEST-RULES-001` – Firestore Security Rules Egységteszt Sablon (`@firebase/rules-unit-testing`)

### Kötelező 7-elemű Metaadat-fejléc
1. **Template ID:** `REF-TEST-RULES-001`
2. **Applicability (`Alkalmazási feltételek`):** Minden L1, L2 és L3 Firebase projekt, amely Firestore adatbázist használ.
3. **Target SDK Baseline (`Kompatibilitási referencia-alapvonal`):** `@firebase/rules-unit-testing@^3.x` / `^4.x`, `firebase@^11.x`, Firebase Local Emulator Suite. *A célprojekt `package-lock.json` fájljával egyeztetendő!*
4. **Security Invariants (`Biztonsági invariánsok`):**
   - Igazolja, hogy a zárolt üzleti kollekciók (`passes`) saját tulajdonos és admin által olvashatók, de más felhasználó által nem olvashatók, és még a tulajdonos által sem írhatók.
   - Igazolja a kliens által írható kollekciók (`workouts`) BOLA-védelmét, a `userId` immutabilitását, a `hasOnly` mezőszűrést, a típus/tartomány-ellenőrzést, valamint a `list` lekérdezések szűrő- és limitkövetelményeit.
5. **Required Tests (`Kötelező pozitív és negatív tesztek`):**
   - Pozitív (`assertSucceeds`): tulajdonosi/admin olvasás, szabályos saját létrehozás, szűrőt és limitet (`<=50`) tartalmazó `list` lekérdezés.
   - Negatív (`assertFails`): idegen dokumentum olvasása/módosítása, közvetlen kliensírás `passes` kollekcióra, `userId` átírása `update` során, nem engedélyezett mező (`isAdmin: true`) beszúrása, érvénytelen tartomány (`durationMinutes: -10`), szűrő nélküli (`unfilteredQuery`), limit nélküli (`noLimitQuery`) és limitet túllépő (`excessiveQuery`) lekérdezés.
6. **Known Limitations (`Ismert korlátozások`):**
   - Ez a teszt kizárólag a `firestore.rules` szabályokat teszteli; **nem teszteli** a Cloud Functions (Admin SDK) végpontok szerveroldali jogosultság-ellenőrzését (ahhoz a `REF-TEST-ADMIN-001` sablon kötelező!).
7. **Verification Level & Last Verified (`Ellenőrzési szint és dátum`):** `DOCUMENTATION_REVIEWED` | Utolsó dokumentációs felülvizsgálat: `2026-11-03`

### Referencia-kód (`tests/firestore.rules.spec.ts`)

~~~typescript
import { initializeTestEnvironment, assertSucceeds, assertFails } from "@firebase/rules-unit-testing";
import { doc, getDoc, setDoc, updateDoc, collection, query, where, limit, getDocs, serverTimestamp } from "firebase/firestore";
import * as fs from "fs";

export async function runComprehensiveRulesTests() {
  const testEnv = await initializeTestEnvironment({
    projectId: "demo-security-test",
    firestore: { rules: fs.readFileSync("firestore.rules", "utf8") },
  });

  const aliceDb = testEnv.authenticatedContext("alice_uid", { email_verified: true }).firestore();
  const malloryDb = testEnv.authenticatedContext("mallory_uid", { email_verified: true }).firestore();
  const adminDb = testEnv.authenticatedContext("admin_uid", { email_verified: true, role: "admin" }).firestore();

  // Kezdeti állapot feltöltése a Security Rules megkerülésével (Admin SDK megfelelője)
  await testEnv.withSecurityRulesDisabled(async (context) => {
    const adminFs = context.firestore();
    await setDoc(doc(adminFs, "passes/pass_1"), { userId: "alice_uid", remainingUses: 5 });
    await setDoc(doc(adminFs, "workouts/w_1"), {
      userId: "alice_uid",
      title: "Morning Session",
      notes: "Felt great",
      durationMinutes: 45,
      createdAt: new Date(),
      updatedAt: new Date(),
    });
  });

  // 1. ZÁROLT KRITIKUS KOLLEKCIÓ TESZTEK (passes)
  await assertSucceeds(getDoc(doc(aliceDb, "passes/pass_1"))); // Tulajdonos olvashatja
  await assertSucceeds(getDoc(doc(adminDb, "passes/pass_1"))); // Admin olvashatja
  await assertFails(getDoc(doc(malloryDb, "passes/pass_1"))); // Más felhasználó NEM olvashatja (BOLA)
  await assertFails(setDoc(doc(aliceDb, "passes/pass_1"), { userId: "alice_uid", remainingUses: 999 })); // Tulajdonos SEM írhatja!

  // 2. KLIENS ÁLTAL ÍRHATÓ KOLLEKCIÓ TESZTEK (workouts)
  // A. Másik felhasználó dokumentumának módosítása -> MUST FAIL
  await assertFails(updateDoc(doc(malloryDb, "workouts/w_1"), { title: "Hacked" }));

  // B. Megváltoztathatatlan tulajdonos (userId) átírása update során -> MUST FAIL
  await assertFails(updateDoc(doc(aliceDb, "workouts/w_1"), { userId: "mallory_uid", updatedAt: serverTimestamp() }));

  // C. Nem engedélyezett extra mező (isAdmin: true) beszúrása -> MUST FAIL (hasOnly ellenőrzés)
  await assertFails(
    setDoc(doc(aliceDb, "workouts/w_2"), {
      userId: "alice_uid",
      title: "Test",
      notes: "",
      durationMinutes: 30,
      isAdmin: true,
      createdAt: serverTimestamp(),
      updatedAt: serverTimestamp(),
    })
  );

  // D. Érvénytelen tartomány (durationMinutes: -10) -> MUST FAIL
  await assertFails(
    setDoc(doc(aliceDb, "workouts/w_3"), {
      userId: "alice_uid",
      title: "Test",
      notes: "",
      durationMinutes: -10,
      createdAt: serverTimestamp(),
      updatedAt: serverTimestamp(),
    })
  );

  // 3. LEKÉRDEZÉSI LIMIT ÉS SZŰRŐ TESZTEK (allow list - Security Rules are not filters!)
  // A. Szabályos lekérdezés userId szűrővel és limit(20) záradékkal -> MUST SUCCEED
  const validQuery = query(collection(aliceDb, "workouts"), where("userId", "==", "alice_uid"), limit(20));
  await assertSucceeds(getDocs(validQuery));

  // B. Hiányzó userId szűrő -> MUST FAIL
  const unfilteredQuery = query(collection(aliceDb, "workouts"), limit(20));
  await assertFails(getDocs(unfilteredQuery));

  // C. Hiányzó limit() záradék -> MUST FAIL
  const noLimitQuery = query(collection(aliceDb, "workouts"), where("userId", "==", "alice_uid"));
  await assertFails(getDocs(noLimitQuery));

  // D. Maximális limitet (50) túllépő lekérdezés (limit(100)) -> MUST FAIL
  const excessiveQuery = query(collection(aliceDb, "workouts"), where("userId", "==", "alice_uid"), limit(100));
  await assertFails(getDocs(excessiveQuery));

  await testEnv.cleanup();
}
~~~

---

## 6. Sablon 5: `REF-TEST-ADMIN-001` – Szerveroldali Admin SDK Handler Autorizációs és Tranzakciós Teszt Sablon

### Kötelező 7-elemű Metaadat-fejléc
1. **Template ID:** `REF-TEST-ADMIN-001`
2. **Applicability (`Alkalmazási feltételek`):** Minden L2 és L3 projekt, ahol Cloud Functions v2 (vagy más Admin SDK-t használó szerveroldali komponens) adatbázis-olvasást vagy -módosítást végez.
3. **Target SDK Baseline (`Kompatibilitási referencia-alapvonal`):** `firebase-admin@^13.x`, `firebase-functions@^6.x`, Node.js tesztfuttató (pl. `vitest`, `jest` vagy `node:test`) a Firestore Emulator ellen futtatva (`FIRESTORE_EMULATOR_HOST`). *A célprojekt `package-lock.json` fájljával egyeztetendő!*
4. **Security Invariants (`Biztonsági invariánsok`):**
   - Mivel a Firebase Admin SDK megkerüli a `firestore.rules` szabályokat (`FIREBASE-ADMIN-001`), ez a teszt közvetlenül a szerveroldali handler (`handleCreateBooking`) jogosultsági, validációs, rate-limit és idempotencia-invariánsait igazolja.
   - Igazolja a két külön tranzakció (`FIREBASE-FN-003` és `FIREBASE-FN-004`) eltérő felelősségét: ha az üzleti tranzakció előfeltétel-hiba miatt elbukik, a rate limiter számlálója (`_rateLimits`) akkor is megőrzi a növelt értéket.
5. **Required Tests (`Kötelező pozitív és negatív tesztek`):**
   - Negatív 1: Hitelesítetlen (`auth: undefined`) hívás elutasítása (`unauthenticated`).
   - Negatív 2: Nem verifikált e-mail (`email_verified: false`) elutasítása (`unauthenticated`).
   - Negatív 3: Másik felhasználó bérletével (`passId`) történő visszaélési kísérlet (BOLA -> `permission-denied`), miközben a bérlet egyenlege változatlan marad, de a rate-limit számláló növekszik!
   - Pozitív + Negatív 4 (Idempotencia és `requestHash`): Ugyanazzal az `idempotencyKey`-jel és azonos paraméterekkel történő újrahívás deduplikált választ ad (`deduplicated: true`, nincs dupla levonás); azonban ugyanazzal az `idempotencyKey`-jel és **módosított paraméterrel (`slotId`)** történő hívás `already-exists` hibával elutasításra kerül.
6. **Known Limitations (`Ismert korlátozások`):**
   - A handler közvetlen meghívása a v2 `onCall` keretrendszeren belüli transzport-szintű App Check token-kriptográfiát nem váltja ki; az App Check enforcement beállítását konfigurációs ellenőrzéssel és integrációs teszttel is igazolni kell.
7. **Verification Level & Last Verified (`Ellenőrzési szint és dátum`):** `DOCUMENTATION_REVIEWED` | Utolsó dokumentációs felülvizsgálat: `2026-11-03`

### Referencia-kód (`tests/booking.admin.spec.ts`)

~~~typescript
import * as assert from "node:assert/strict";
import { getFirestore, Timestamp } from "firebase-admin/firestore";
import { handleCreateBooking } from "../functions/src/booking";

const db = getFirestore();

async function expectHttpsError(fn: () => Promise<unknown>, expectedCode: string) {
  let threw = false;
  try {
    await fn();
  } catch (err: any) {
    threw = true;
    assert.equal(err?.code, expectedCode, `Expected HttpsError code '${expectedCode}', got '${err?.code}'`);
  }
  assert.equal(threw, true, `Expected function to throw HttpsError('${expectedCode}'), but it succeeded.`);
}

export async function runAdminHandlerAuthzTests() {
  const nowMs = Date.now();

  // Kezdeti tesztállapot létrehozása az emulátorban
  await db.doc("slots/slot_open_1").set({
    status: "OPEN",
    capacity: 5,
    bookedCount: 0,
    startTime: Timestamp.fromMillis(nowMs + 3600_000),
  });
  await db.doc("slots/slot_open_2").set({
    status: "OPEN",
    capacity: 5,
    bookedCount: 0,
    startTime: Timestamp.fromMillis(nowMs + 7200_000),
  });
  await db.doc("passes/pass_alice").set({
    userId: "alice_uid",
    status: "ACTIVE",
    remainingUses: 3,
    expiresAt: Timestamp.fromMillis(nowMs + 86400_000),
  });

  // 1. NEGATÍV TESZT: Hitelesítetlen hívás -> unauthenticated
  await expectHttpsError(
    () =>
      handleCreateBooking({
        auth: undefined,
        data: {
          slotId: "slot_open_1",
          passId: "pass_alice",
          idempotencyKey: "11111111-1111-4111-8111-111111111111",
        },
      } as any),
    "unauthenticated"
  );

  // 2. NEGATÍV TESZT: Nem verifikált e-mail cím -> unauthenticated
  await expectHttpsError(
    () =>
      handleCreateBooking({
        auth: { uid: "alice_uid", token: { email_verified: false } },
        data: {
          slotId: "slot_open_1",
          passId: "pass_alice",
          idempotencyKey: "11111111-1111-4111-8111-111111111111",
        },
      } as any),
    "unauthenticated"
  );

  // 3. NEGATÍV TESZT (BOLA + Két külön tranzakció felelőssége):
  // Mallory megpróbálja felhasználni Alice bérletét -> permission-denied
  await expectHttpsError(
    () =>
      handleCreateBooking({
        auth: { uid: "mallory_uid", token: { email_verified: true } },
        data: {
          slotId: "slot_open_1",
          passId: "pass_alice",
          idempotencyKey: "22222222-2222-4222-8222-222222222222",
        },
      } as any),
    "permission-denied"
  );

  // Ellenőrzés: Alice bérlete érintetlen maradt (3 alkalom), DE Mallory rate-limit számlálója
  // (1. tranzakció) commitolt és nem gördült vissza a 2. tranzakció hibája miatt!
  const alicePassAfterBola = (await db.doc("passes/pass_alice").get()).data()!;
  assert.equal(alicePassAfterBola.remainingUses, 3);
  const malloryRateLimit = (await db.doc("_rateLimits/mallory_uid_createBooking").get()).data()!;
  assert.equal(malloryRateLimit.count, 1);

  // 4. POZITÍV TESZT + IDEMPOTENCIA DEDUPLIKÁCIÓ:
  const validIdempKey = "33333333-3333-4333-8333-333333333333";
  const firstCall = await handleCreateBooking({
    auth: { uid: "alice_uid", token: { email_verified: true } },
    data: {
      slotId: "slot_open_1",
      passId: "pass_alice",
      idempotencyKey: validIdempKey,
    },
  } as any);
  assert.equal(firstCall.deduplicated, false);

  // Azonos idempotencyKey + azonos paraméterek -> deduplikált válasz, nincs dupla levonás!
  const duplicateCall = await handleCreateBooking({
    auth: { uid: "alice_uid", token: { email_verified: true } },
    data: {
      slotId: "slot_open_1",
      passId: "pass_alice",
      idempotencyKey: validIdempKey,
    },
  } as any);
  assert.equal(duplicateCall.deduplicated, true);
  assert.equal(duplicateCall.bookingId, firstCall.bookingId);

  const alicePassAfterValid = (await db.doc("passes/pass_alice").get()).data()!;
  assert.equal(alicePassAfterValid.remainingUses, 2); // Csak egyszer vonódott le!

  // 5. NEGATÍV TESZT: Azonos idempotencyKey + ELTÉRŐ paraméter (requestHash mismatch) -> already-exists
  await expectHttpsError(
    () =>
      handleCreateBooking({
        auth: { uid: "alice_uid", token: { email_verified: true } },
        data: {
          slotId: "slot_open_2", // Manipulált slotId ugyanazzal az idempotencyKey-jel!
          passId: "pass_alice",
          idempotencyKey: validIdempKey,
        },
      } as any),
    "already-exists"
  );
}
~~~