---
name: devsecops-supply-chain-security
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
---

# 06 – DevSecOps & Software Supply Chain Security

## 1. Scope (`DEVSEC-SCOPE-001`)

This module (`Tier-1` selective domain module under `CORE-LOAD-001`) governs security across the repository, Git, GitHub, npm, Node.js runtime, build tooling, GitHub Actions, Firebase CLI, deployments, artifacts, secrets, dependencies, and release pipelines. Activate it for every task affecting code, packages (`package.json` / `package-lock.json`), builds, CI/CD, or infrastructure configuration (`firebase.json`, Rules).

---

## 2. Repository Hygiene and File Locking (`DEVSEC-REPO-001`)

- Protect the `main` / production branch (branch protection), requiring pull request reviews and passing CI status checks.
- Trigger production deployments exclusively from verified, clean git branches/tags and passing CI/emulator test runs.
- **Multi-Agent and Concurrent Work (`CORE-CONC-001`):** During parallel development, enforce the 6-step exclusive file-locking protocol (`TECHNICAL_LOCK` or `COORDINATION_RESERVATION`) defined in Core Section 7.4. Protected security files (`firestore.rules`, `storage.rules`, `firebase.json`, `package.json`, `package-lock.json`, IAM/CI workflows, auth middleware) may be modified by at most one agent at a time (`concurrency_lock: "EXCLUSIVE"`). Following any parallel branch merge or rebase, re-execute all security and emulator tests against the combined codebase!
- Never commit secrets, credentials, `.env` files, service account JSON keys, database backups, or personal data (PII) to git.
- Maintain strict `.gitignore` (and `.npmignore` when publishing packages) rules; always run `npm pack --dry-run` or `npm publish --dry-run` prior to package publication.
- **Compromised Secret Rule:** Any secret or key that ever enters git history MUST be treated as **immediately compromised**: revoke and rotate the secret immediately; rewriting git history alone is insufficient!
- Use a `CODEOWNERS` file for critical security files where team size warrants.

---

## 3. Node.js, npm, and Project Freshness (`DEVSEC-NODE-001`)

- Use an actively supported Node.js LTS release; keep the runtime version aligned across `package.json` (`engines`), `.nvmrc`/`.node-version`, `firebase.json` (Functions runtime), and CI/CD pipelines (`CORE-FRESH-001` – `Project Freshness`).
- Commit the lockfile (`package-lock.json`) to version control; in CI/CD and reproducible builds, always run **`npm ci`** instead of `npm install`.
- Review direct and transitive dependencies; merge automated dependency updates (e.g., Dependabot/Renovate) only after passing tests and review.
- Run `npm audit` (and `npm audit --omit=dev` / `--production`) and triage findings by severity, exploitability, reachability, and fix availability; never ignore findings blindly.
- Before installing an unfamiliar package, inspect its maintainer, official repository, release history, download metrics, and requested permissions (defending against typosquatting, dependency confusion, lockfile poisoning, and malicious packages).
- Manage `install` / `postinstall` script risks; consider `--ignore-scripts` where dependency functionality permits.
- Never enable or expose the Node.js inspector/debug port (`--inspect`) in production environments.

---

## 4. CI/CD Pipeline Security (`DEVSEC-CICD-001`)

- Operate CI tokens under least privilege; set explicit minimal `permissions:` blocks in GitHub Actions workflows (defaulting to `contents: read`).
- Pin third-party GitHub Actions to **full (40-character) commit SHAs**, as version tags (`@v3`) can be mutated after publication.
- Prefer **OIDC / Workload Identity Federation** over long-lived cloud service account JSON keys for Google Cloud / Firebase deployments.
- Store secrets in environment-scoped, deployment-protected secret stores.
- Make linting, type-checking (`tsc`), unit/integration tests, Firebase Emulator Rules/Admin tests (`firebase emulators:exec`), and security scans mandatory blocking gates in the deployment pipeline.
- Never allow pull requests from external forks (e.g., via misconfigured `pull_request_target`) to access production secrets or write permissions!
- Retain build logs, release tags, and deployment IDs for audit traceability.

---

## 5. Security Scanning (`DEVSEC-SCAN-001`)

Proportionally to the project's risk tier (`L1`/`L2`/`L3`), run:
- **SAST:** CodeQL, ESLint security rules, or an equivalent static analyzer.
- **Dependency / SCA:** `npm audit` and/or a dedicated Software Composition Analysis tool.
- **Secret scanning:** GitHub secret scanning, gitleaks, or equivalent pre-commit/CI checks.
- **IaC / Config scanning:** inspection of `firestore.rules`, `storage.rules`, `firebase.json` Hosting headers, GitHub Actions workflows, and Google Cloud IAM bindings.
- **DAST:** dynamic scanning against staging environments covering authenticated and unauthenticated routes.
- **SBOM:** CycloneDX or SPDX Software Bill of Materials generation for L2/L3 or contractual supply-chain requirements.
- Triage all scan findings item-by-item; justify false positives in `RELEASE_GATE.md` rather than silently suppressing them.

---

## 6. Build and Frontend Artifacts (`DEVSEC-BUILD-001`)

- Disable public publication of JavaScript **source maps** in production, or enable them only after a conscious decision verifying they expose no internal business secrets.
- Treat every environment variable bundled into client-side code as **public**.
- Code minification and obfuscation are **not security controls** (`CORE-PRIN-012`).
- Build artifacts in a reproducible, controlled environment and never hand-edit artifacts after build/deployment.
- Keep `firebase.json`, `.firebaserc`, `firestore.rules`, `firestore.indexes.json`, `storage.rules`, and Cloud Functions configuration under version control (git).

---

## 7. Secrets Management (`DEVSEC-SEC-001`)

- Store all server-side secrets in Google Cloud Secret Manager (`defineSecret`) or the platform's protected secret store (`FIREBASE-IAM-001`).
- Never commit `.env` files, API tokens, private keys, service account JSON files, or OAuth `client_secret` values to git.
- Maintain a documented procedure for secret rotation and immediate revocation.
- Enforce log redaction and error report filtering so environment variables or secrets never leak into logs upon failure.
- Ensure build-time secrets never leak into public frontend bundles.
- **AI Agent Rule (`08_AI_AGENT_CODING_SECURITY.md`):** Never paste production secrets into an AI prompt! If an automated tool requires a secret, use approved environment/Secret Manager injection rather than chat text.

---

## 8. Minimum Release Pipeline (`DEVSEC-PIPE-001`)

Before any L2/L3 production release, follow this 11-step pipeline sequence:
1. Dependency and lockfile integrity check (`npm ci`);
2. Lint + static type-checking (`tsc` -> `STATIC_CHECKED`);
3. Unit and integration tests;
4. Firestore/Storage Rules and Admin SDK handler emulator tests (`REF-TEST-RULES-001`, `REF-TEST-ADMIN-001` -> `EMULATOR_TESTED`);
5. SAST / SCA (`npm audit`) / secret scan;
6. Reproducible production build;
7. Staging deployment;
8. Smoke tests + DAST / HTTP security header and CSP verification (`PROJECT_TESTED`);
9. Independent Security Review (`CORE-REV-001`) and human approval (`RELEASE_GATE.md`);
10. Production deployment;
11. Post-deploy monitoring and immediate rollback readiness.

---

## 9. Node.js Runtime Specifics (`DEVSEC-RUNTIME-001`)

- Never execute untrusted input via `eval`, `new Function`, the `vm` module, or shell-spawning `child_process.exec`.
- If spawning a child process is unavoidable, use `execFile` / `spawn` with `shell: false` and a strictly validated argument array (never string concatenation!).
- Normalize filesystem paths (`path.resolve`) and enforce confinement within the designated root directory (path traversal defense).
- Defend against prototype pollution and ReDoS (Regular Expression Denial of Service) vulnerabilities.
- Verify HTTP parser and timeout settings against current Node.js security best practices.

---

## 10. Official Sources (`DEVSEC-REF-001`)

For exact version status and verification dates (`last_verified`), see `10_SOURCE_REGISTER.md`:
- Node.js Security Best Practices: https://nodejs.org/en/learn/getting-started/security-best-practices
- NIST SSDF SP 800-218 (v1.1 Final): https://csrc.nist.gov/pubs/sp/800/218/final
- NIST SSDF Publications: https://csrc.nist.gov/projects/ssdf/publications
- GitHub Actions Security Hardening: https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions
- npm audit Docs: https://docs.npmjs.com/cli/commands/npm-audit
- npm ci Docs: https://docs.npmjs.com/cli/commands/npm-ci
- OpenSSF Scorecard: https://scorecard.dev/