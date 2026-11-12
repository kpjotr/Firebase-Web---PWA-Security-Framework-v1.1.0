---
name: devsecops-supply-chain-security
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
---

# 06 – DevSecOps & Software Supply Chain Security

## 1. Hatókör (`DEVSEC-SCOPE-001`)

Ez a modul (`Tier-1` szelektív doménmodul a `CORE-LOAD-001` szerint) a repository, a Git, a GitHub, az npm, a Node.js runtime, a build tooling, a GitHub Actions, a Firebase CLI, a deploy, az artifact-kezelés, a titkok (secrets), a függőségek és a release folyamat biztonsági szabályait rögzíti. Minden olyan feladatnál kötelező aktiválni, amely kódot, csomagot (`package.json` / `package-lock.json`), buildet, CI/CD-t vagy infrastruktúra-konfigurációt (`firebase.json`, Rules) érint.

---

## 2. Repository hygiene és Fájlzárolás (`DEVSEC-REPO-001`)

- Védd a `main` / production ágat (branch protection), követelj meg pull request review-t és zöld CI status checkeket.
- Production deploy kizárólag ellenőrzött, tiszta git branch/tag és sikeres CI/emulátorteszt eredmény alapján indítható.
- **Többügynökös és párhuzamos munkavégzés (`CORE-CONC-001`):** Párhuzamos fejlesztésnél kötelező a Core 7.4. pontjában rögzített 6 lépéses exkluzív fájlzárolási protokoll (`TECHNICAL_LOCK` vagy `COORDINATION_RESERVATION`) betartása. Védett biztonsági fájlokat (`firestore.rules`, `storage.rules`, `firebase.json`, `package.json`, `package-lock.json`, IAM/CI workflow-k, auth middleware) egy időben csak egyetlen ágens módosíthat (`concurrency_lock: "EXCLUSIVE"`). Párhuzamos ágak összefésülése (`merge` / `rebase`) után kötelező az összes biztonsági és emulátoros teszt újrafuttatása az egyesített kódbázison!
- Titkok (secrets), credentialök, `.env` fájlok, service account JSON kulcsok, adatbázis-mentések (backups) és személyes adatok (PII) soha nem kerülhetnek gitbe.
- Állíts be szigorú `.gitignore` (és csomagpublikálásnál `.npmignore`) szabályokat; npm publish előtt mindig futtass `npm pack --dry-run` vagy `npm publish --dry-run` ellenőrzést.
- **Kompromittált titok szabálya:** Ha egy titok vagy kulcs valaha bekerült a git historyba, azt azonnal **kompromittáltnak kell tekinteni**: a titkot azonnal vissza kell vonni és le kell cserélni (revoke & rotate); a git history átírása önmagában nem elégséges!
- Alkalmazz `CODEOWNERS` fájlt a kritikus biztonsági fájlokhoz, ahol a csapatméret ezt indokolja.

---

## 3. Node.js, npm és Projektfrissesség (`DEVSEC-NODE-001`)

- Használj aktívan támogatott Node.js LTS verziót; a runtime verzióját tartsd szinkronban a `package.json` (`engines`), az `.nvmrc`/`.node-version`, a `firebase.json` (Functions runtime) és a CI/CD pipeline között (`CORE-FRESH-001` – `Project Freshness`).
- A lockfile (`package-lock.json`) mindig legyen verziókövetve (commitolva); CI/CD-ben és reprodukálható buildeknél `npm install` helyett kötelező az **`npm ci`** használata.
- Ellenőrizd a közvetlen és tranzitív függőségeket; automatikus dependency update (pl. Dependabot/Renovate) csak sikeres teszt és review után merge-ölhető.
- Futtass `npm audit` (illetve `npm audit --omit=dev` / `--production`) vizsgálatot, és az eredményt értékeld súlyosság (severity), kihasználhatóság (exploitability), elérhetőség (reachability) és javítási elérhetőség szerint; ne hagyd figyelmen kívül vakon.
- Ismeretlen vagy új csomag telepítése előtt vizsgáld meg a csomag karbantartóját (maintainer), a hivatalos repositoryt, a kiadási történetet, a letöltési mintát és a kért jogosultságokat (typosquatting, dependency confusion, lockfile poisoning és malicious package megelőzése).
- Kezeld az `install` / `postinstall` scriptek kockázatát; ahol a függőségek működése megengedi, mérlegeld az `--ignore-scripts` használatát.
- A Node.js inspector/debug port (`--inspect`) production környezetben soha nem lehet engedélyezve vagy nyilvánosan elérhető.

---

## 4. CI/CD pipeline biztonság (`DEVSEC-CICD-001`)

- A CI tokenek működjenek a legkisebb jogosultság (`least privilege`) elve szerint; GitHub Actions workflow-kban a `permissions:` blokkot explicit módon állítsd a szükséges minimumra (alapértelmezetten `contents: read`).
- A harmadik féltől származó GitHub Action-öket **teljes (40 karakteres) commit SHA-ra pineld**, mert a verziótagek (`@v3`) utólag átmozgathatók.
- Felhő-hitelesítéshez (Google Cloud / Firebase deploy) hosszú életű service account JSON kulcsok helyett részesítsd előnyben az **OIDC / Workload Identity Federation** megoldást.
- A titkokat környezet-specifikus (environment-scoped) és deployment-védett secret store-ban tárold.
- A lint, typecheck (`tsc`), unit/integrációs tesztek, Firebase Emulator Rules/Admin tesztek (`firebase emulators:exec`) és security scanek legyenek a deploy gate kötelező, blokkoló lépései.
- Soha ne engedd, hogy külső forkból érkező pull request (`pull_request_target` helytelen használata) hozzáférjen production titkokhoz vagy írási jogosultsághoz!
- Őrizd meg a build logokat, a release tageket és a deploy azonosítókat az elszámoltathatóság érdekében.

---

## 5. Security scanning (`DEVSEC-SCAN-001`)

A projekt kockázati szintjéhez (`L1`/`L2`/`L3`) igazítva alkalmazd az alábbi vizsgálatokat:
- **SAST:** CodeQL, ESLint security szabályok vagy megfelelő statikus kódelemző.
- **Dependency / SCA:** `npm audit` és/vagy dedikált Software Composition Analysis eszköz.
- **Secret scanning:** GitHub secret scanning, gitleaks vagy hasonló pre-commit/CI ellenőrzés.
- **IaC / Config scanning:** `firestore.rules`, `storage.rules`, `firebase.json` Hosting fejlécek, GitHub Actions workflow-k és Google Cloud IAM jogosultságok ellenőrzése.
- **DAST:** Staging környezeten végzett dinamikus vizsgálat hitelesített és nem hitelesített útvonalakon.
- **SBOM:** CycloneDX vagy SPDX formátumú Software Bill of Materials előállítása L2/L3 vagy szerződéses ellátásilánc-igény esetén.
- A scan-eredményeket tételesen triage-old; a false positive találatokat szakmailag indokold a `RELEASE_GATE.md`-ben, ne csak némítsd el (suppress) őket.

---

## 6. Build és frontend artifactok (`DEVSEC-BUILD-001`)

- Production környezetben a JavaScript **source map** fájlok nyilvános publikálását tiltsd le, vagy csak tudatos döntés és annak ellenőrzése után engedélyezd, hogy nem tartalmaznak-e belső üzleti titkot.
- Minden olyan környezeti változó, amely bekerül a kliensoldali buildbe, **nyilvánosnak minősül**.
- A kódminifikálás (minification) és az obfuszkáció **nem biztonsági kontroll** (`CORE-PRIN-012`).
- Az artifactokat reprodukálható, kontrollált környezetben buildeld, és build/deploy után soha ne módosítsd őket kézzel.
- A `firebase.json`, `.firebaserc`, `firestore.rules`, `firestore.indexes.json`, `storage.rules` és a Cloud Functions konfiguráció mindig legyen verziókövetés (git) alatt.

---

## 7. Titokkezelés (`DEVSEC-SEC-001`)

- Minden szerveroldali titkot Google Cloud Secret Managerben (`defineSecret`) vagy a platform védett secret store-jában tárolj (`FIREBASE-IAM-001`).
- Tilos `.env` fájlt, API tokent, privát kulcsot, service account JSON-t vagy OAuth `client_secret`-et gitbe commitolni.
- Legyen dokumentált eljárás a titkok rotációjára (rotation) és azonnali visszavonására (revocation).
- Gondoskodj a log redactionről és a hibajelentések szűréséről, hogy környezeti változók vagy titkok hiba esetén se kerüljenek naplóba.
- Build-time titok nem szivároghat át a publikus frontend bundle-be.
- **AI-ágens szabály (`08_AI_AGENT_CODING_SECURITY.md`):** Soha ne másolj éles titkot AI promptba! Ha egy automatizált eszköznek titokra van szüksége, használj jóváhagyott környezeti/Secret Manager injektálást, ne csevegési szöveget.

---

## 8. Release pipeline minimum (`DEVSEC-PIPE-001`)

Minden L2/L3 éles kiadás (release) előtt az alábbi 11 lépéses pipeline-sorrend az irányadó:
1. Dependency és lockfile integritás-ellenőrzés (`npm ci`);
2. Lint + statikus típusellenőrzés (`tsc` -> `STATIC_CHECKED`);
3. Unit és integrációs tesztek;
4. Firestore/Storage Rules és Admin SDK handler emulátortesztek (`REF-TEST-RULES-001`, `REF-TEST-ADMIN-001` -> `EMULATOR_TESTED`);
5. SAST / SCA (`npm audit`) / secret scan;
6. Reprodukálható production build;
7. Staging deploy;
8. Smoke teszt + DAST / HTTP security header és CSP ellenőrzés (`PROJECT_TESTED`);
9. Független Security Review (`CORE-REV-001`) és emberi jóváhagyás (`RELEASE_GATE.md`);
10. Production deploy;
11. Post-deploy monitoring és azonnali rollback-készenlét.

---

## 9. Node.js futásidejű sajátosságok (`DEVSEC-RUNTIME-001`)

- Soha ne futtass nem megbízható bemenetet `eval`, `new Function`, `vm` modul vagy shell-t nyitó `child_process.exec` segítségével.
- Ha gyermekfolyamat indítása elkerülhetetlen, használd az `execFile` / `spawn` API-t `shell: false` beállítással és szigorúan validált argumentum-tömbbel (soha ne string-összefűzéssel!).
- A fájlrendszer-útvonalakat normalizáld (`path.resolve`) és kényszerítsd a kijelölt gyökérkönyvtár alá (path traversal védelem).
- Védj a prototype pollution és a ReDoS (Regular Expression Denial of Service) sebezhetőségek ellen.
- A HTTP parser és timeout beállításokat a Node.js aktuális biztonsági útmutatója alapján ellenőrizd.

---

## 10. Hivatalos források (`DEVSEC-REF-001`)

A pontos verzióstátuszt és ellenőrzési dátumokat (`last_verified`) lásd a `10_SOURCE_REGISTER.md` modulban:
- Node.js Security Best Practices: https://nodejs.org/en/learn/getting-started/security-best-practices
- NIST SSDF SP 800-218 (v1.1 Final): https://csrc.nist.gov/pubs/sp/800/218/final
- NIST SSDF Publications: https://csrc.nist.gov/projects/ssdf/publications
- GitHub Actions Security Hardening: https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions
- npm audit Docs: https://docs.npmjs.com/cli/commands/npm-audit
- npm ci Docs: https://docs.npmjs.com/cli/commands/npm-ci
- OpenSSF Scorecard: https://scorecard.dev/