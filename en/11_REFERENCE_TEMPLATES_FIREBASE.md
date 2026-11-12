---
name: firebase-reference-templates
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
tier_layer: Tier-2 (Implementation & Testing Reference Layer)
default_verification_level: DOCUMENTATION_REVIEWED
---

# 11 – Firebase Reference Implementations and Test Templates (`REF-MOD-001`)

## 1. Role, Loading Rules, and Verification Levels (`REF-MOD-001`, `REF-VERIFY-001`)

### 1.1. When Must This Module Be Loaded? (`CORE-LOAD-001`, `REF-MOD-001`)
This module is the **Tier-2 (Implementation Layer)** component of the framework. To protect the context window, do not load it during purely theoretical or high-level architectural planning; however, it **MUST be loaded** whenever an AI agent or developer:
1. writes or modifies `firestore.rules` or client-side Firestore queries (`REF-RULES-BASE-001`);
2. implements Cloud Functions v2 (`onCall`) transactional business endpoints, rate limiters, idempotency, or ledgers (`REF-FN-CALLABLE-001`);
3. implements PWA client-side logout, `BroadcastChannel` synchronization, or Firestore cache cleanup (`REF-PWA-LOGOUT-001`);
4. writes Firestore Security Rules emulator unit tests (`REF-TEST-RULES-001`);
5. writes server-side Admin SDK / Cloud Functions handler authorization (negative) tests (`REF-TEST-ADMIN-001`).

**Normative Warning (`REF-MOD-001`):** The code patterns in this module are not blind copy-paste snippets; they are **structured reference implementations (`REFERENCE / MINTA`)**. Before applying them, the agent MUST verify compatibility with the target project's actual data model, business invariants, and SDK versions pinned in `package-lock.json`!

---

### 1.2. Governance of Reference Implementation Verification Levels (`REF-VERIFY-001`)

A code template included in documentation does not constitute executed or production-tested code. Every reference template and every generated/adapted implementation MUST strictly distinguish among the following **5 verification levels (`Verification Level`)**:

| Verification Status (`Verification Level`) | Normative Meaning and Condition of Use |
| :--- | :--- |
| **`DOCUMENTATION_REVIEWED`** | Documentation and logical review completed; no code execution or emulator run occurred during document generation. *(This is the default, truthful status of all templates inside the `v1.1.0` documentation package!)* |
| **`STATIC_CHECKED`** | Actual static analysis, type-checking (`tsc`), or syntax linting was executed; does not prove runtime correctness. |
| **`EMULATOR_TESTED`** | The specified positive and negative tests were actually executed and passed inside the appropriate Firebase Emulator Suite environment. |
| **`PROJECT_TESTED`** | Verified inside a concrete target project with its actual SDKs, lockfile, and configuration in staging, production, or CI. |
| **`NOT_VERIFIED`** | No verification of any kind has been performed. |

**Strict Prohibition (`REF-VERIFY-001`):** Actual verification statuses, SDK versions, test environments, and dates (`Last Verified`) may only be populated based on real verification! AI-generated or template-derived code **never qualifies** for `STATIC_CHECKED`, `EMULATOR_TESTED`, or `PROJECT_TESTED` status on its own. When applying a template in a target project, the project must execute and evidence the `STATIC_CHECKED` -> `EMULATOR_TESTED` -> `PROJECT_TESTED` progression in its `CODER_TO_REVIEWER` / `REVIEWER_TO_PLANNER` handoff contracts.

---

## 2. Template 1: `REF-RULES-BASE-001` – Firestore Security Rules (v2) and Client Query Pattern

### Mandatory 7-Element Metadata Header
1. **Template ID:** `REF-RULES-BASE-001`
2. **Applicability:** L1, L2, and L3 Firebase projects. Includes the global fallback default deny rule, an L1 client-writable personal log collection (`workouts`), and L2/L3 business collections locked against direct client writes (`passes`, `passLedger`, `bookings`).
3. **Target SDK Baseline:** Firestore Security Rules `rules_version = '2'`, client-side `firebase@^11.x` (Modular Web SDK). *Not an automatic version lock; reconcile with the target project's `package-lock.json`!*
4. **Security Invariants:**
   - All unmatched paths are denied by default (`FIREBASE-RULES-001`).
   - Direct client-side creation, modification, and deletion of critical business collections (`passes`, `passLedger`, `bookings`) are prohibited (`FIREBASE-RULES-002`).
   - Client-writable documents (`workouts`) enforce ownership matching (`isOwner`), `hasOnly` field allowlisting, strict type and size bounds, server timestamps (`request.time`), and immutability of `userId` and `createdAt` on `update`.
   - `list` queries require owner/admin authorization and `request.query.limit <= 50` (`[FRAMEWORK DEFAULT]` – `FIREBASE-RULES-003`).
5. **Required Tests:** See `REF-TEST-RULES-001` (owner and admin reads, unauthorized read/write rejection, `userId` mutation rejection, `hasOnly` violation rejection, and rejection of `unfilteredQuery`, `noLimitQuery`, and `excessiveQuery`).
6. **Known Limitations:**
   - Overlapping `allow` rules in Firestore evaluate with logical `OR`; an overly broad permission on a parent path overrides child denials.
   - Client-side `.limit(50)` is **not a security control** in itself because client queries can be modified. Firestore Security Rules is not a general query-limiting engine; it enforces access conditions (`request.query.limit <= 50`) on data reachable by the query. `limit <= 50` is a `[FRAMEWORK DEFAULT]` recommendation that must never be treated as a standalone authorization or cost-protection guarantee; expensive or sensitive queries require server-side restrictions, quotas, or rate limits (`FIREBASE-RULES-003`).
   - Security Rules **do not protect** server-side operations performed via the Firebase Admin SDK (`FIREBASE-ADMIN-001`).
7. **Verification Level & Last Verified:** `DOCUMENTATION_REVIEWED` | Last Documentation Review: `2026-11-03` *(Requires emulator execution in the target project to achieve `EMULATOR_TESTED`!)*

### Reference Code (`firestore.rules` and Client Query)

~~~javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Helper functions
    function isAuthenticated() {
      return request.auth != null && request.auth.token.email_verified == true;
    }
    function isOwner(userId) {
      return isAuthenticated() && request.auth.uid == userId;
    }
    // Custom Claim admin check (Requires no extra Firestore get() read,
    // but remember claim changes propagate only upon ID token refresh! Max. 1,000 bytes [PLATFORM HARD LIMIT])
    function isAdmin() {
      return isAuthenticated() && request.auth.token.role == 'admin';
    }

    // 1. FALLBACK DEFAULT DENY for any unmatched path (FIREBASE-RULES-001).
    // (Warning: overlapping allow rules evaluate with logical OR;
    // never place overly broad allow rules on parent paths!)
    match /{document=**} {
      allow read, write: if false;
    }

    // 2. CRITICAL BUSINESS COLLECTIONS (L2/L3: Passes, Ledger, Bookings) -> Client writes DENIED! (FIREBASE-RULES-002)
    match /passes/{passId} {
      allow get: if isOwner(resource.data.userId) || isAdmin();
      // request.query.limit <= 50: [FRAMEWORK DEFAULT] (FIREBASE-RULES-003)
      allow list: if (isOwner(resource.data.userId) || isAdmin()) && request.query.limit <= 50;
      allow create, update, delete: if false; // Written exclusively via Cloud Functions v2 (Admin SDK)!
    }

    match /passLedger/{entryId} {
      allow get: if isOwner(resource.data.userId) || isAdmin();
      allow list: if (isOwner(resource.data.userId) || isAdmin()) && request.query.limit <= 50;
      allow create, update, delete: if false; // Append-only ledger written exclusively by server!
    }

    match /bookings/{bookingId} {
      allow get: if isOwner(resource.data.userId) || isAdmin();
      allow list: if (isOwner(resource.data.userId) || isAdmin()) && request.query.limit <= 50;
      allow create, update, delete: if false; // Written exclusively via Cloud Functions v2 (Admin SDK)!
    }

    // 3. USER-SCOPED LOGS (L1 or isolated user logs) -> Strictly schema-validated client writes
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
        && request.resource.data.userId == resource.data.userId // Ownership (userId) is immutable!
        && request.resource.data.createdAt == resource.data.createdAt // Creation timestamp is immutable!
        && request.resource.data.title is string && request.resource.data.title.size() > 0 && request.resource.data.title.size() <= 100
        && request.resource.data.notes is string && request.resource.data.notes.size() <= 2000
        && request.resource.data.durationMinutes is int && request.resource.data.durationMinutes > 0 && request.resource.data.durationMinutes <= 600
        && request.resource.data.updatedAt == request.time;

      allow delete: if isOwner(resource.data.userId) || isAdmin();
    }
  }
}
~~~

**Mandatory Client-Side Query Pattern (Because Security Rules Are Not Filters!):**
~~~typescript
import { Firestore, collection, query, where, orderBy, limit, getDocs } from "firebase/firestore";

// Client query MUST explicitly filter by userId and include limit(<=50) to match the rule,
// otherwise Firestore Rules rejects the entire query with permission-denied!
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

## 3. Template 2: `REF-FN-CALLABLE-001` – Cloud Functions v2 (`onCall`) with Two Separate Transactions

### Mandatory 7-Element Metadata Header
1. **Template ID:** `REF-FN-CALLABLE-001`
2. **Applicability:** All business mutations in L2 and L3 systems managing scarce capacity (`slots`), balances/passes (`passes`), bookings (`bookings`), or multi-document state transitions.
3. **Target SDK Baseline:** `firebase-functions@^6.x` (2nd Gen / v2 `onCall`), `firebase-admin@^13.x`, `zod@^3.x`, client-side `firebase@^11.x`. *Reconcile with the target project's `package-lock.json`!*
4. **Security Invariants:**
   - Authenticated caller with verified email (`request.auth` and `email_verified === true`); `uid` is derived exclusively from the verified auth token (`FIREBASE-ADMIN-001`).
   - Strict input validation (`Zod .strict()`) rejecting any unknown fields.
   - **Transaction 1 (`enforceRateLimit` – `FIREBASE-FN-003`):** Executes in a standalone transaction *prior to* the business transaction; on limit exhaustion, increments and commits `blockedCount`, then throws `resource-exhausted` *after* the transaction commits.
   - **Transaction 2 (Business / Booking Transaction – `FIREBASE-FN-004`):** Checks and mutates the idempotency record (`_idempotency` with SHA-256 `requestHash`), pass (`passes`), slot (`slots`), booking (`bookings`), and append-only transaction ledger (`passLedger`) inside a **single atomic `db.runTransaction`**.
   - **Separated Transaction Responsibilities:** If Transaction 2 (business) fails due to a business precondition error (e.g., expired pass or full slot), the request quota consumed in Transaction 1 (rate limiter) intentionally **does not roll back**.
5. **Required Tests:** See `REF-TEST-ADMIN-001` (unauthenticated call, unverified email, BOLA using another user's `passId`, rate-limit persistence across business errors, `requestHash` mismatch rejection).
6. **Known Limitations:**
   - `consumeAppCheckToken: true` (Replay Protection) adds an extra verification round-trip and latency; apply it on L3 and high-risk L2 mutation endpoints, paired with client `{ limitedUseAppCheckTokens: true }` and server `roles/firebaseappcheck.tokenVerifier` IAM role (`FIREBASE-FN-002`).
   - The Firestore per-UID rate limiter (`_rateLimits`) is suitable for low-to-moderate traffic, but under high concurrency (>1 req/sec/user-action document) it causes document contention and read/write costs, and does not stop multi-account (sybil) attacks (`FIREBASE-FN-003`).
   - Configure Firestore TTL policies on `_rateLimits.expiresAt` and `_idempotency.expiresAt`; never configure TTL on `passLedger` (`FIREBASE-IDEMP-001`).
7. **Verification Level & Last Verified:** `DOCUMENTATION_REVIEWED` | Last Documentation Review: `2026-11-03`

### Reference Code (Server-Side Cloud Function v2 and Client-Side Invocation)

~~~typescript
import { onCall, HttpsError, CallableRequest } from "firebase-functions/v2/https";
import * as logger from "firebase-functions/logger";
import { getFirestore, FieldValue, Timestamp } from "firebase-admin/firestore";
import * as crypto from "crypto";
import { z } from "zod";

const db = getFirestore();

// 1. Strict Zod schema (.strict() immediately rejects unexpected/extra properties!)
export const CreateBookingSchema = z.object({
  slotId: z.string().min(1).max(64),
  passId: z.string().min(1).max(64),
  idempotencyKey: z.string().uuid(),
}).strict();

// 2. TRANSACTION 1: Standalone Firestore Per-UID Rate Limiter (FIREBASE-FN-003)
// Critical rule: NEVER throw inside the transaction callback when rate-limited, because throwing
// aborts and rolls back the transaction! Commit the incremented blockedCount first, then throw
// HttpsError("resource-exhausted") AFTER the transaction resolves.
// Configure a Firestore TTL policy on `_rateLimits.expiresAt`!
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

// 3. Extracted Handler Logic for Direct Unit/Emulator Testing (REF-TEST-ADMIN-001)
export async function handleCreateBooking(request: CallableRequest<unknown>) {
  // A. Verify Authentication (v2: request.auth)
  if (!request.auth || request.auth.token.email_verified !== true) {
    throw new HttpsError("unauthenticated", "Authenticated and email-verified account required.");
  }
  const uid = request.auth.uid;

  // B. Validate Input (Zod .strict())
  const parsed = CreateBookingSchema.safeParse(request.data);
  if (!parsed.success) {
    throw new HttpsError("invalid-argument", "Invalid request payload.");
  }
  const { slotId, passId, idempotencyKey } = parsed.data;

  // C. TRANSACTION 1: Enforce Rate Limit (independent of the business transaction!)
  await enforceRateLimit(uid, "createBooking", 10, 60);

  // D. Compute SHA-256 requestHash to prevent parameter tampering on reused idempotency keys
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
    // E. TRANSACTION 2: Atomic Business (Booking) Transaction (FIREBASE-FN-004)
    const result = await db.runTransaction(async (tx) => {
      // 1. Check Idempotency & Verify requestHash
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

      // 2. Verify Ownership (BOLA defense), Status, and Time Invariants
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
      const ttlExpiresAt = Timestamp.fromMillis(nowMs + 48 * 3600 * 1000); // 48-hour TTL for idempotency record

      // 3. Atomic Writes (Pass + Slot + Booking + Append-Only Ledger + Idempotency Record)
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
      // Mandatory append-only ledger entry (NEVER set TTL on passLedger!)
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
    // Log filtered structured metadata rather than dumping raw exception objects
    logger.error("Booking transaction failed", {
      uid,
      slotId,
      errorName: error?.name || "UnknownError",
      errorCode: error?.code || "unknown",
    });
    throw new HttpsError("internal", "Booking could not be processed.");
  }
}

// 4. 2nd Gen (v2) Protected Callable Endpoint Registration
// Prerequisite when consumeAppCheckToken: true is enabled: grant
// `roles/firebaseappcheck.tokenVerifier` IAM role to the Function's Service Account! (FIREBASE-FN-002)
export const createBooking = onCall(
  {
    region: "europe-west1",
    maxInstances: 10,
    enforceAppCheck: true,
    consumeAppCheckToken: true, // Risk-based on L3 and critical L2 mutations (requires client limitedUseAppCheckTokens: true!)
  },
  handleCreateBooking
);
~~~

**Mandatory Client-Side Invocation Pattern (`limitedUseAppCheckTokens: true`):**
~~~typescript
import { getFunctions, httpsCallable } from "firebase/functions";

// Whenever consumeAppCheckToken: true is enabled on the server, the 3rd options parameter is MANDATORY!
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

## 4. Template 3: `REF-PWA-LOGOUT-001` – Best-Effort PWA Logout and `TAB_ID`-Isolated Multi-Tab Cache Cleanup

### Mandatory 7-Element Metadata Header
1. **Template ID:** `REF-PWA-LOGOUT-001`
2. **Applicability:** All L1/L2/L3 SPA and PWA applications supporting user logout, especially where multiple browser tabs may be open or offline Firestore persistence is used.
3. **Target SDK Baseline:** `firebase@^11.x` (`firebase/auth`, `firebase/firestore`), modern browser `BroadcastChannel` and `CacheStorage` APIs. *Reconcile with the target project's `package-lock.json`!*
4. **Security Invariants:**
   - Every browser tab generates a unique `TAB_ID` so the tab initiating logout never processes its own `FORCE_LOGOUT` broadcast (self-echo prevention).
   - Secondary tabs first call `await terminate(db)` to release their IndexedDB locks and only then reply with `LOGOUT_TAB_ACK` (or `LOGOUT_TAB_ERROR` on failure).
   - Logout clears `sessionStorage` and all non-static (`!name.startsWith("app-shell-static-")`) Service Worker Cache Storage entries.
5. **Required Tests:**
   - Positive test: with multiple open tabs, secondary tabs respond to `FORCE_LOGOUT` with `LOGOUT_TAB_ACK`, `terminate(db)` and `signOut(auth)` execute, and all tabs navigate to `/login`.
   - Negative test: the initiating tab ignores messages where `senderTabId === TAB_ID`; if a secondary tab replies with `LOGOUT_TAB_ERROR` or fails to respond within `tabAckTimeoutMs`, logout does not hang indefinitely and proceeds to `signOut(auth)` and hard page reload after the timeout.
6. **Known Limitations (Mandatory to Document! – `FIREBASE-PWA-001`):**
   1. Upon logout, application-level state belonging to the user session must be invalidated across all reachable tabs.
   2. Persistently stored sensitive data requires separate handling; `clearIndexedDbPersistence()` **is not a forensic disk wipe** and **permanently discards any pending, unsynchronized offline writes**.
   3. `memoryLocalCache()` reduces persistent Firestore cache exposure on shared devices, but **does not guarantee complete data erasure** and does not replace application-level privacy controls.
   4. Logout cannot guarantee the revocation or complete physical removal of data that was already displayed or accessed in the browser.
   5. Without a global browser tab registry, the `BroadcastChannel` + `TAB_ID` + `LOGOUT_TAB_ACK` / `LOGOUT_TAB_ERROR` + timeout protocol remains **best-effort coordination**.
7. **Verification Level & Last Verified:** `DOCUMENTATION_REVIEWED` | Last Documentation Review: `2026-11-03`

### Reference Code (`logout.ts`)

~~~typescript
import { Auth, signOut } from "firebase/auth";
import { Firestore, terminate, clearIndexedDbPersistence } from "firebase/firestore";

const LOGOUT_CHANNEL_NAME = "app_security_logout_channel";

// Every browser tab gets a unique identifier so that BroadcastChannel instances
// inside the initiating tab NEVER process their own FORCE_LOGOUT broadcast!
const TAB_ID =
  typeof crypto !== "undefined" && "randomUUID" in crypto
    ? crypto.randomUUID()
    : `tab_${Date.now()}_${Math.random().toString(36).slice(2)}`;

let isInitiatingLogout = false;

// Call once on app startup so secondary tabs immediately terminate their Firestore
// instance and reply with LOGOUT_TAB_ACK (or LOGOUT_TAB_ERROR on failure)!
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
        await terminate(db); // Secondary tab releases its IndexedDB lock
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
    // 1. Notify secondary tabs and collect ACK/ERROR responses (best-effort coordination)
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

    // 2. Sign out of Firebase Authentication
    await signOut(auth);

    // 3. Terminate initiating tab's Firestore instance and clear IndexedDB persistence
    // (Warning: pending unsynchronized offline writes are discarded here, and this is not a forensic wipe!)
    await terminate(db);
    await clearIndexedDbPersistence(db);

    // 4. Delete custom application IndexedDB databases
    for (const dbName of customIndexedDbNames) {
      indexedDB.deleteDatabase(dbName);
    }

    // 5. Clear sessionStorage and non-static Cache Storage entries
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
    // 6. Force hard navigation to reset all in-memory state
    window.location.replace("/login");
  }
}
~~~

---

## 5. Template 4: `REF-TEST-RULES-001` – Firestore Security Rules Unit Test Template (`@firebase/rules-unit-testing`)

### Mandatory 7-Element Metadata Header
1. **Template ID:** `REF-TEST-RULES-001`
2. **Applicability:** All L1, L2, and L3 Firebase projects using Cloud Firestore.
3. **Target SDK Baseline:** `@firebase/rules-unit-testing@^3.x` / `^4.x`, `firebase@^11.x`, Firebase Local Emulator Suite. *Reconcile with the target project's `package-lock.json`!*
4. **Security Invariants:**
   - Verifies that locked business collections (`passes`) are readable by owner and admin, unreadable by other users (BOLA), and non-writable even by the owner.
   - Verifies BOLA defense, `userId` immutability, `hasOnly` field allowlisting, type/range checks, and `list` query filter/limit requirements on client-writable collections (`workouts`).
5. **Required Tests:**
   - Positive (`assertSucceeds`): owner/admin read, valid owner document creation, `list` query with matching owner filter and `.limit(<=50)`.
   - Negative (`assertFails`): cross-user read/update, direct client write to `passes`, mutating `userId` on `update`, injecting unallowed field (`isAdmin: true`), out-of-range value (`durationMinutes: -10`), `unfilteredQuery`, `noLimitQuery`, and `excessiveQuery`.
6. **Known Limitations:**
   - Tests only `firestore.rules`; **does not test** server-side authorization inside Cloud Functions (Admin SDK) endpoints (which requires `REF-TEST-ADMIN-001`!).
7. **Verification Level & Last Verified:** `DOCUMENTATION_REVIEWED` | Last Documentation Review: `2026-11-03`

### Reference Code (`tests/firestore.rules.spec.ts`)

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

  // Seed initial state bypassing Security Rules (Admin SDK equivalent)
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

  // 1. LOCKED CRITICAL COLLECTION TESTS (passes)
  await assertSucceeds(getDoc(doc(aliceDb, "passes/pass_1"))); // Owner can read
  await assertSucceeds(getDoc(doc(adminDb, "passes/pass_1"))); // Admin can read
  await assertFails(getDoc(doc(malloryDb, "passes/pass_1"))); // Other user CANNOT read (BOLA)
  await assertFails(setDoc(doc(aliceDb, "passes/pass_1"), { userId: "alice_uid", remainingUses: 999 })); // Owner CANNOT write!

  // 2. CLIENT-WRITABLE COLLECTION TESTS (workouts)
  // A. Cross-user update attempt -> MUST FAIL
  await assertFails(updateDoc(doc(malloryDb, "workouts/w_1"), { title: "Hacked" }));

  // B. Mutating immutable ownership (userId) on update -> MUST FAIL
  await assertFails(updateDoc(doc(aliceDb, "workouts/w_1"), { userId: "mallory_uid", updatedAt: serverTimestamp() }));

  // C. Injecting unallowed extra field (isAdmin: true) -> MUST FAIL (hasOnly check)
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

  // D. Invalid range (durationMinutes: -10) -> MUST FAIL
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

  // 3. QUERY LIMIT & FILTER TESTS (allow list - Security Rules are not filters!)
  // A. Valid query with matching userId filter and limit(20) -> MUST SUCCEED
  const validQuery = query(collection(aliceDb, "workouts"), where("userId", "==", "alice_uid"), limit(20));
  await assertSucceeds(getDocs(validQuery));

  // B. Missing userId filter -> MUST FAIL
  const unfilteredQuery = query(collection(aliceDb, "workouts"), limit(20));
  await assertFails(getDocs(unfilteredQuery));

  // C. Missing limit() clause -> MUST FAIL
  const noLimitQuery = query(collection(aliceDb, "workouts"), where("userId", "==", "alice_uid"));
  await assertFails(getDocs(noLimitQuery));

  // D. Query exceeding max limit (limit(100) > 50) -> MUST FAIL
  const excessiveQuery = query(collection(aliceDb, "workouts"), where("userId", "==", "alice_uid"), limit(100));
  await assertFails(getDocs(excessiveQuery));

  await testEnv.cleanup();
}
~~~

---

## 6. Template 5: `REF-TEST-ADMIN-001` – Server-Side Admin SDK Handler Authorization and Transaction Test Template

### Mandatory 7-Element Metadata Header
1. **Template ID:** `REF-TEST-ADMIN-001`
2. **Applicability:** All L2 and L3 projects where Cloud Functions v2 (or any server-side component using the Admin SDK) reads or mutates database state.
3. **Target SDK Baseline:** `firebase-admin@^13.x`, `firebase-functions@^6.x`, Node.js test runner (`vitest`, `jest`, or `node:test`) executed against the Firestore Emulator (`FIRESTORE_EMULATOR_HOST`). *Reconcile with the target project's `package-lock.json`!*
4. **Security Invariants:**
   - Because the Firebase Admin SDK bypasses `firestore.rules` (`FIREBASE-ADMIN-001`), this test directly verifies the server-side handler's (`handleCreateBooking`) authorization, input validation, rate-limit, and idempotency invariants.
   - Verifies the separated responsibilities of the two transactions (`FIREBASE-FN-003` and `FIREBASE-FN-004`): when the business transaction fails due to a precondition/authorization error, the rate limiter counter (`_rateLimits`) still retains its committed increment.
5. **Required Tests:**
   - Negative 1: Unauthenticated (`auth: undefined`) call rejected (`unauthenticated`).
   - Negative 2: Unverified email (`email_verified: false`) rejected (`unauthenticated`).
   - Negative 3: Attempting to use another user's pass (`passId`) rejected (BOLA -> `permission-denied`), while the victim's pass balance remains untouched AND the attacker's rate-limit counter increments!
   - Positive + Negative 4 (Idempotency & `requestHash`): Retrying with the same `idempotencyKey` and identical parameters returns a deduplicated response (`deduplicated: true`, no double deduction); retrying with the same `idempotencyKey` and **modified parameters (`slotId`)** is rejected with `already-exists`.
6. **Known Limitations:**
   - Invoking the handler function directly does not exercise transport-level App Check token cryptography inside the v2 `onCall` wrapper; verify App Check enforcement via configuration review and integration tests.
7. **Verification Level & Last Verified:** `DOCUMENTATION_REVIEWED` | Last Documentation Review: `2026-11-03`

### Reference Code (`tests/booking.admin.spec.ts`)

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

  // Seed initial test state in the Firestore Emulator
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

  // 1. NEGATIVE TEST: Unauthenticated call -> unauthenticated
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

  // 2. NEGATIVE TEST: Unverified email address -> unauthenticated
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

  // 3. NEGATIVE TEST (BOLA + Separated Transaction Responsibilities):
  // Mallory attempts to redeem Alice's pass -> permission-denied
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

  // Verify: Alice's pass remains untouched (3 uses), BUT Mallory's rate-limit counter
  // (Transaction 1) committed and did NOT roll back when Transaction 2 failed!
  const alicePassAfterBola = (await db.doc("passes/pass_alice").get()).data()!;
  assert.equal(alicePassAfterBola.remainingUses, 3);
  const malloryRateLimit = (await db.doc("_rateLimits/mallory_uid_createBooking").get()).data()!;
  assert.equal(malloryRateLimit.count, 1);

  // 4. POSITIVE TEST + IDEMPOTENCY DEDUPLICATION:
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

  // Same idempotencyKey + identical parameters -> deduplicated response, no double deduction!
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
  assert.equal(alicePassAfterValid.remainingUses, 2); // Deducted only once!

  // 5. NEGATIVE TEST: Same idempotencyKey + DIFFERENT parameters (requestHash mismatch) -> already-exists
  await expectHttpsError(
    () =>
      handleCreateBooking({
        auth: { uid: "alice_uid", token: { email_verified: true } },
        data: {
          slotId: "slot_open_2", // Tampered slotId with reused idempotencyKey!
          passId: "pass_alice",
          idempotencyKey: validIdempKey,
        },
      } as any),
    "already-exists"
  );
}
~~~