# §4.5 5-Question Gap Scan — batch_22

## Q1: What did Brain assume that turned out wrong?
**Brain assumed 2 NEW test users needed.** Plan agent step 1 verified Firebase Auth state and CLAUDE.md §12: vqxr (+0100) AND 6Dfe (+0101) BOTH already exist. Only 1 NEW user needed (+0102). Spec said "Create 2 new test users" — Plan corrected to 1 (with docs explaining the 3-user target was 2 existing + 1 new).
- **D-22-01 (DOC, S-3)**: CLAUDE.md §12 lists test users vqxr/6Dfe but doesn't reference the seed script — should add reference to `scripts/seed_test_users.js` and `TEST_USERS.md` for future-batch discoverability.

## Q2: What scope grew during execution that wasn't in the spec?
**OTP whitelist gap surfaced.** New user +15555550102 was created via Admin SDK but Firebase test-phone OTP whitelist is configured in Firebase Console (not Admin SDK). Future device-test against User 3 will fail at OTP unless founder adds whitelist entry. Documented in TEST_USERS.md but not actionable in this batch.
- **D-22-02 (RED_BLOCK candidate, S-2)**: Founder must add `+15555550102` → OTP code in Firebase Console at Authentication → Settings → Phone numbers for testing. Without this, User 3 cannot authenticate via the app on a device. Not blocking batch_22 (Admin SDK seed is what spec required) but BLOCKS device-test using User 3.

## Q3: What did the Plan agent surface that the spec didn't?
**Three findings:**
1. **firebase-admin already at ^13.7.0** — no install needed for Part B. Spec assumed install might be needed.
2. **Sentry render-tree wrap order** — Plan agent flagged that Sentry.wrap is the OUTERMOST wrapper; ErrorBoundary remains the inner-tree fallback. Documented in STRESS_TEST L2 with no scope creep (didn't add Sentry.captureException to ErrorBoundary's componentDidCatch — left for future batch).
3. **Defense-in-depth on reset script** — Plan added RESET_CONFIRM env var check at BOTH shell layer (reset_test_users.sh) AND node layer (seed_test_users.js --reset). Single layer would have been sufficient per spec; double layer is intentional belt-and-suspenders.
- **No D entry** — Plan agent's surfaces all became code that shipped or documented decisions.

## Q4: What did device verify NOT show that should be checked later?
**Sentry dev-event verification couldn't be done in CC.** `Sentry.captureMessage('batch_22 Sentry init verified')` runs in `__DEV__` mode and dispatches to the Sentry dashboard. CC cannot log into Sentry web console (no creds). The init-code path runs cleanly (zero bundle errors during partial Metro export at 42% / 1212 modules), and Sentry @8.11.0 is well-documented to dispatch on init when DSN+debug:true. Founder verification step: log into Sentry dashboard, confirm event arrives in development environment.
- **D-22-03 (P3 OBS)**: Founder action — verify Sentry dashboard receives `'batch_22 Sentry init verified'` event when next dev/EAS build runs the app. Out-of-band action.

## Q5: Is there a configuration that PASSES today but would silently miss a real failure?
**Two candidates worth observing:**
1. **`enableInExpoDevelopment: true`** — known-deprecated in Sentry RN @8.x (replaced by `_experiments.enableSpotlight`). Currently silently ignored in @8.11.0 (no warning at init time). Production builds skip dev-mode init anyway via `__DEV__` checks elsewhere. Risk: future Sentry SDK upgrade may surface this as an error.
2. **`tracesSampleRate: 0.2` in production** — 20% sample. If production crash volume is high, 80% of crashes produce no trace. Acceptable for v1 launch (cost control); revisit after first 1000 production users.
- **D-22-04 (P3 OBS)**: Both config concerns noted; revisit at next Sentry SDK upgrade or post-launch cost review.

## Triage summary for Brain
- **D-22-01** (DOC, S-3): CLAUDE.md §12 add seed-script references
- **D-22-02** (S-2): Founder OTP whitelist add for +15555550102
- **D-22-03** (P3 OBS): Founder verify Sentry dashboard event arrival
- **D-22-04** (P3 OBS): Sentry config concerns (deprecated flag, low sample rate)
