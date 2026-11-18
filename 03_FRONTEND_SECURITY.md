---
name: frontend-web-pwa-security
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
scope: HTML, CSS, JavaScript, TypeScript, React, browser, PWA
---

# 03 – Frontend Security: HTML, CSS, JavaScript, TypeScript, React, and PWA

## 1. Purpose and Scope (`FRONT-SCOPE-001`)

The frontend is never a security boundary (`CORE-PRIN-005`), but it is a primary attack surface. This module (`Tier-1` selective domain module under `CORE-LOAD-001`) governs all code, markup, styles, client-side data storage, and PWA assets running in the browser. It is mandatory for every frontend or static website task alongside `01_SECURITY_CORE.md` and (when modifying code) `08_AI_AGENT_CODING_SECURITY.md`.

---

## 2. HTML and DOM (`FRONT-HTML-001`, `FRONT-DOMXSS-001`)

### 2.1. Mandatory HTML and Attribute Rules (`FRONT-HTML-001`)
- Render user-supplied or external data as plain text by default, never as raw HTML.
- Never use `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `document.write`, or React `dangerouslySetInnerHTML` with untrusted input.
- If rendering rich text/HTML is an unavoidable business requirement, use an actively maintained, strictly configured sanitizer library (e.g., DOMPurify), enforce a strict element and attribute allowlist, and pair it with a **Trusted Types** policy where supported. Remember that a sanitizer alone does not make every injection context safe.
- Validate URLs and enforce an explicit scheme/origin allowlist (`https:`, `mailto:`, `tel:`) before using them for navigation or resource loading. `javascript:`, `vbscript:`, and unexpected `data:` URLs are strictly prohibited (`MUST NOT`).
- On external links opening in a new tab (`target="_blank"`), include `rel="noopener noreferrer"` whenever `window.opener` access is not required.
- For `iframe` elements, apply an explicit `sandbox` attribute and minimal `allow` permissions; never combine `allow-same-origin` + `allow-scripts` unless explicitly justified and documented, because same-origin embedded content can remove its own sandbox attribute.
- Never rely on `data-*` attributes, hidden inputs, `disabled`/`readonly` fields, or client-side validation for authorization.
- Every form input must be backed by server-side (or for L1, Firestore Rules) validation; frontend validation is strictly for UX.

### 2.2. Preventing DOM XSS (`FRONT-DOMXSS-001`)
- Avoid APIs that evaluate strings as code (`eval`, `new Function`, `setTimeout("...")`, `setInterval("...")`).
- Never concatenate user data into HTML, JavaScript, CSS, or URL contexts without context-appropriate output encoding.
- When using template engines or Markdown renderers, verify and disable raw HTML support or enforce post-render sanitization.
- Inspect all data flows entering the DOM from untrusted sources (`location.search`, `location.hash`, `postMessage`, `localStorage`) to dangerous sinks.

---

## 3. CSS Security (`FRONT-CSS-001`)

CSS is not an access control layer. Mandatory security checks:
- External stylesheets, `@import`, `url()`, and font resources may only be loaded from known, required origins.
- Never inject untrusted values into CSS strings, selectors, or inline `style` attributes.
- Prevent CSS data exfiltration patterns (selector/attribute matching combined with external `url()` requests).
- Never rely on visual hiding (`display: none`, `visibility: hidden`, `opacity: 0`, off-screen positioning) as authorization: data a user is not authorized to see must never be sent to the client by the server/database!
- Prefer nonce- or hash-based CSP `style-src` directives; `'unsafe-inline'` styles may only be used as a documented exception.
- For external web fonts (e.g., Google Fonts CDN), evaluate privacy (IP address transmission), tracking, availability, and supply-chain risks; self-host fonts where appropriate.

---

## 4. JavaScript and TypeScript (`FRONT-JSTS-001`)

- All client-side code and bundles are public. Placing secrets, private API keys, service account credentials, signing keys, or admin tokens in frontend code is strictly prohibited (`CORE-PRIN-005`).
- Contents of `localStorage`, `sessionStorage`, `IndexedDB`, and `Cache Storage` are fully readable upon a successful XSS attack or local device access. Never store passwords, long-lived secrets, or unnecessary sensitive personal data (PII) in them.
- Do not bypass the Firebase Auth SDK's built-in token management with custom, unverified token storage schemes.
- For cross-window `window.postMessage` communication, always verify `event.origin`, `event.source`, message schema, and message type; never use `'*'` as `targetOrigin` for sensitive data.
- Enforce explicit trust boundaries and runtime input validation on `window.opener`, `BroadcastChannel`, `WebSocket`, and Web/Service Worker messages.
- Never use `Math.random()` for security tokens, invitation codes, or idempotency keys; use `crypto.getRandomValues()`, `crypto.randomUUID()`, or a server-side CSPRNG.
- Never implement custom cryptographic algorithms; use audited built-in browser (`Web Crypto API`) or Node.js primitives.
- TypeScript types provide compile-time checking only, not runtime validation! Validate external or network data at runtime.
- Prevent prototype pollution by avoiding unchecked recursive object deep merges; use field allowlists, `Object.hasOwn`, or `Object.create(null)`.
- In production builds, never log auth tokens, personal data (PII), full request payloads, or session identifiers to the browser console (`console.log`).

---

## 5. React-Specific Rules (`FRONT-REACT-001`)

- Default JSX text node escaping is a helpful defense layer, but not a complete XSS solution (it does not protect `href`/`src` attributes or `dangerouslySetInnerHTML`).
- `dangerouslySetInnerHTML` is prohibited by default, except for documented, sanitized use cases protected by Trusted Types where supported.
- React state, route guards, Context, component visibility, or client-side role checks are **not authorization** (`CORE-PRIN-012`). The server (Cloud Functions) and Firebase Security Rules must independently verify every operation.
- Never render untrusted URLs into `href`, `src`, `action`, CSS, or router navigation props without protocol and origin validation.
- When using React Server Components (RSC), SSR, hydration, routers, or framework data loaders, explicitly model the server/client trust boundary so server secrets or internal data models are never serialized into client props.
- Never place secrets into `VITE_*`, `REACT_APP_*`, `NEXT_PUBLIC_*`, or any other client-bundled environment variable!
- Error Boundaries and client error handlers must display only generic user-facing messages, never stack traces, secrets, or internal configuration (`CORE-PRIN-009`).

---

## 6. CSP, HTTP Security Headers, and Browser Hardening (`FRONT-CSP-001`, `FRONT-HDR-001`)

### 6.1. Content Security Policy (`FRONT-CSP-001`)
- Start in `Content-Security-Policy-Report-Only` mode to inventory legitimate sources, then transition to enforced `Content-Security-Policy` mode.
- Avoid `'unsafe-inline'`, `'unsafe-eval'`, `*`, and overly broad wildcards (such as blanket `*.googleapis.com`) in `script-src` and `connect-src`.
- Prefer nonce/hash-based script policies, `object-src 'none'`, `base-uri 'self'`, `frame-ancestors 'none'` (or `'self'`), and service-specific `connect-src` allowlists.
- Add only the exact Firebase endpoints used by the project to `connect-src`:
  - Auth: `https://identitytoolkit.googleapis.com https://securetoken.googleapis.com`
  - Firestore: `https://firestore.googleapis.com`
  - App Check: `https://content-firebaseappcheck.googleapis.com`
  - Storage: `https://firebasestorage.googleapis.com`
  - Functions: `https://<region>-<project-id>.cloudfunctions.net` (or the v2 Cloud Run URL)
- Align and test `frame-ancestors` and `X-Frame-Options` so they do not conflict.
- CSP does not replace output encoding, sanitization, or dependency review (`CORE-PRIN-007`).

### 6.2. Additional HTTP Security Headers (`FRONT-HDR-001`)
Configure the following in `firebase.json` under `hosting.headers`:
- `Strict-Transport-Security` (HSTS): HTTPS-only; append `includeSubDomains` and `; preload` only if every subdomain is permanently HTTPS-only.
- `X-Content-Type-Options: nosniff`.
- `Referrer-Policy`: use a data-minimizing policy (e.g., `strict-origin-when-cross-origin` or `same-origin`).
- `Permissions-Policy`: disable unused browser features and allow only required ones (e.g., `camera=(self), microphone=(), geolocation=()`).
- If cookie sessions are used: enforce `Secure`, `HttpOnly`, appropriate `SameSite` (`Strict` or `Lax`), **Host-Only** (omitting the `Domain` attribute), and the `__Host-` prefix (`FIREBASE-BOUND-001`). Note that Firebase client SDK tokens are distinct from server-side session cookies.
- CORS is not authorization; never reflect arbitrary incoming `Origin` headers alongside `Access-Control-Allow-Credentials: true`.

---

## 7. PWA, Service Worker, and Logout Limitations (`FRONT-PWA-001`)

When implementing PWA client-side storage and logout, loading `REF-PWA-LOGOUT-001` from `11_REFERENCE_TEMPLATES_FIREBASE.md` and enforcing `FIREBASE-PWA-001` is mandatory:
- Service Workers may cache only required static App Shell assets; never cache personal, authenticated, or dynamic API responses without explicit justification and access controls.
- Implement cache versioning (`app-shell-static-vX`) and invalidation; delete all non-static caches upon logout.
- Prevent cache poisoning and cross-user cache leakage by never ignoring authorization context in cache keys.
- Treat Service Worker `message` and `fetch` events as untrusted input; act only after verifying known `message.type` and origin/client identity.
- Ensure the Service Worker update lifecycle prevents vulnerable legacy client code from remaining active indefinitely.
- When an offline mutation queue is used, document in `SECURITY_PROFILE.md` which operations may remain pending, how retries and idempotency work, and how unsynchronized writes are handled on logout.
- **The 5 Mandatory PWA Logout and Cache Limitations (`FIREBASE-PWA-001`, `REF-PWA-LOGOUT-001`):**
  1. Upon logout, invalidate application-level state belonging to the user session across all reachable tabs.
  2. `clearIndexedDbPersistence(db)` **is not a forensic disk wipe**, fails if Firestore is active in another open tab, and **permanently discards pending, unsynchronized offline writes**.
  3. Prefer `memoryLocalCache()` for sensitive data on shared devices, while documenting that it does not guarantee complete data erasure either.
  4. Logout cannot guarantee the physical removal of data that was already displayed or accessed in the browser.
  5. Without a global tab registry, the `BroadcastChannel` + `TAB_ID` + `LOGOUT_TAB_ACK` / `LOGOUT_TAB_ERROR` + timeout protocol remains **best-effort coordination**.

---

## 8. File Uploads and Downloads from the Frontend (`FRONT-FILE-001`)

- The `<input type="file" accept="...">` attribute and client-side MIME checks are UX helpers only; server-side and `storage.rules` validation are mandatory (`FIREBASE-STORAGE-001`).
- Enforce size, extension, MIME, and (at L2/L3 tiers, server-side) magic-byte verification; block dangerous formats (HTML, SVG, executables).
- Never render user uploads as active HTML/SVG on the same origin.
- Revoke Blob URLs created via `URL.createObjectURL()` using `URL.revokeObjectURL()` as soon as they are no longer needed.
- Normalize downloaded filenames server-side to prevent path traversal and header injection.
- Avoid using `data:` URLs for large or sensitive payloads.

---

## 9. External Resources and Subresource Integrity (`FRONT-SRI-001`)

- Avoid third-party external scripts whenever possible; when unavoidable, pin exact versions, use trusted origins, enforce **Subresource Integrity (SRI)** `integrity` hashes, and set `crossorigin` appropriately.
- SRI cannot be applied to all dynamic CDN or ES module loading patterns; never claim all external dependencies are SRI-protected when technically untrue.
- Prefer self-hosting critical frontend dependencies and fonts, and vet package publishers (`06_DEVSECOPS_SUPPLY_CHAIN.md`).
- Third-party analytics and tag manager scripts receive full JavaScript execution privileges on the origin; perform a dedicated security and privacy (`07_DATA_PROTECTION_PRIVACY.md`) risk assessment before adoption.

---

## 10. Frontend Checklist (`FRONT-CHK-001`)

- [ ] zero frontend secrets or private keys in source code, `VITE_*`/`REACT_APP_*`/`NEXT_PUBLIC_*` variables, or bundles (`FRONT-JSTS-001`, `FRONT-REACT-001`);
- [ ] zero untrusted inputs passed to `innerHTML` / `dangerouslySetInnerHTML` (`FRONT-HTML-001`);
- [ ] all dynamic URLs and `postMessage` origins/schemas validated (`FRONT-HTML-001`, `FRONT-JSTS-001`);
- [ ] service-specific CSP and HTTP security headers (`firebase.json`) configured and tested (`FRONT-CSP-001`, `FRONT-HDR-001`);
- [ ] authenticated API responses and PII excluded from Service Worker cache (`FRONT-PWA-001`);
- [ ] client-side storage (`localStorage`/`IndexedDB`) minimized; PWA logout follows the `REF-PWA-LOGOUT-001` best-effort protocol (`FRONT-PWA-001`);
- [ ] React route guards backed by genuine server-side / Firestore Rules authorization (`FRONT-REACT-001`);
- [ ] external scripts and SRI / self-hosting decisions reviewed (`FRONT-SRI-001`);
- [ ] file uploads validated by Storage Rules and server-side checks (`FRONT-FILE-001`);
- [ ] XSS / DOM XSS tests executed (`09_SECURITY_TESTING_THREAT_MODELING.md`).

---

## 11. Official References (`FRONT-REF-001`)

For exact version status and verification dates (`last_verified`), see `10_SOURCE_REGISTER.md`:
- React DOM Common Components & Props: https://react.dev/reference/react-dom/components/common
- MDN Content Security Policy (CSP): https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
- MDN Trusted Types API: https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API
- MDN Subresource Integrity (SRI): https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity
- MDN Service Worker API: https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API
- WHATWG HTML Living Standard: https://html.spec.whatwg.org/
- ECMAScript Specification: https://tc39.es/ecma262/