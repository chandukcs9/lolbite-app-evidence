# SENTRY_AND_TESTUSER_PLAN — batch_22 Plan agent verbatim output

Composed: 2026-05-08
Plan agent verdict:
- **firebase-admin already at ^13.7.0** — no install needed for Part B.
- **Both vqxr (+0100) AND 6Dfe (+0101) test users already exist** per CLAUDE.md §12 — only 1 NEW user needed (+15555550102), not 2 as spec assumed.
- 17 atomic steps total. Sentry DSN public-by-design (Sentry docs).

---

## ARCHITECTURAL RISKS

- **Sentry.wrap vs ErrorBoundary order**: `Sentry.wrap(App)` instruments the root and forwards errors to Sentry, but the existing `ErrorBoundary` (src/components/ErrorBoundary.js) currently only `console.error`s in `componentDidCatch`. After wrap, ErrorBoundary still catches first (closer to children) and renders fallback UI; Sentry's wrap auto-tags native crashes + JS errors not caught by boundaries. Must verify ErrorBoundary still owns the user-visible fallback. If desired, ErrorBoundary's `componentDidCatch` can later add `Sentry.captureException(error)` — but that's out of scope for this batch (memory rule: no scope creep).
- **App.js side-effect ordering**: Line 20 already runs `initAppCheck()` at module-load. Sentry.init MUST be placed at TOP (before `initAppCheck()` import-side-effects + before any other module that could throw on import). Per spec, Sentry.init goes at App.js TOP.
- **TYPOGRAPHY landmine**: Sentry.init body must not transitively touch the constants/theme TYPOGRAPHY export. The wrapper code we add only imports `@sentry/react-native` — safe. ErrorBoundary already imports COLORS (not TYPOGRAPHY), unchanged.
- **Native module + Expo SDK 54 compatibility**: `@sentry/react-native` requires native code. Project is Expo 54 with `expo-dev-client` (line 66) so dev client supports custom native. But Metro/JS-only `expo start --web` won't have the native bridge — Sentry has a JS fallback; `enableInExpoDevelopment: true` ensures init in dev. Risk: prebuild/EAS may need re-run; this batch installs the package only, not a full rebuild. Tribunal must validate Metro bundle + bundle size.
- **`Sentry.wrap(App)` re-exports a wrapped component**: Default export shape changes. Confirmed App is `export default function App()` — wrapping returns a forwardRef-compatible component. Should be transparent to AppEntry.
- **DSN in source**: DSN strings are public-by-design for Sentry but still committed. Acceptable per Sentry docs; no secret leak.
- **firebase-service-account.json is at repo root** (per ls). CLAUDE.md §12 says "DO NOT commit to ANY repo" — verify .gitignore includes it before any add. NEVER `git add` it.
- **Phone-auth UID rebinding**: CLAUDE.md §12 maps `+15555550100`→vqxr, `+15555550101`→6Dfe. Creating phone `+15555550102` via Admin SDK will allocate NEW UID only if phone isn't already bound. Idempotent skip-if-phone-exists logic required.
- **Test user count drift**: Spec says "3 test users target". Two pre-existing (vqxr, 6Dfe). Need to add ONE new (+15555550102) — NOT two. Plan reflects this.
- **`reset_test_users.sh` on Windows**: Repo is on Windows (PowerShell shell). The `.sh` script will require Git-Bash/WSL to run locally. Keep POSIX-portable.
- **Node binary location**: Bash session here reports `node: command not found`, but `package.json` scripts use `node` and CLAUDE.md says Node 20.20.2 required. Execution agent will need to invoke node from PowerShell or proper PATH; planning step assumes runtime env has node.

## STEPS

### B22-P-1 — Pre-flight investigation (READ-ONLY)
Verify package.json deps, App.js init structure, SA JSON keys, existing test users.

### B22-P-2 — Verify .gitignore covers SA + node availability
Both SA filenames matched; node 20.x in PATH.

### B22-P-3 — Install @sentry/react-native (manual, no wizard)
`npm install @sentry/react-native --save --no-audit --no-fund`. Forbidden: `npx`, wizard.

### B22-P-4 — Wire Sentry.init at App.js TOP
Insert before `initAppCheck()` (line 20). Exact init params per spec.

### B22-P-5 — Wrap default export with Sentry.wrap
`function App() {...}` → `export default Sentry.wrap(App);`

### B22-P-6 — Smoke-test bundle (Metro JS load)
Direct binary path; capture stderr; check for module-resolution + TYPOGRAPHY errors.

### B22-P-7 — Confirm Sentry.init runs before any throw site
Static analysis: all `throw` in App.js inside function bodies; init line < initAppCheck call.

### B22-P-8 — Decide test-user delta from P-1 listUsers output
1 new (+0102) given both 0100/0101 exist.

### B22-P-9 — Author scripts/seed_test_users.js
Idempotent (skip-if-phone-exists); modeled on scripts/batch07_run2_seed_test_user.js.

### B22-P-10 — Run seed script once + idempotency re-run
First run: 2 SKIP + 1 CREATED. Second run: 3 SKIP.

### B22-P-11 — Verify 3 users via Admin SDK list
Phones 0100/0101/0102 all present.

### B22-P-12 — Author scripts/reset_test_users.sh (POSIX, --dry-run)
Bash shebang. Real reset requires `RESET_CONFIRM=YES`.

### B22-P-13 — Dry-run reset to validate
Exit 0. "DRY-RUN" + "would delete" present. NO real Firestore calls.

### B22-P-14 — Author TEST_USERS.md
Table: UID | Phone | Display Name | Source. NO secrets.

### B22-P-15 — Cleanup probe script
Rename `scripts/_b22_listusers.js` → `scripts/list_test_users.js`. Verify SA not staged.

### B22-P-16 — Tribunal hard gate
28/28 PASS required.

### B22-P-17 — Closure (commit + push, NO force)
Stage 7 files explicitly. SA absent from diff.
