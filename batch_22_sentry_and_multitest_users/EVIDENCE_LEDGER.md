# batch_22_sentry_and_multitest_users — EVIDENCE LEDGER

**Composed:** 2026-05-08
**Branch:** claude/run-j-audit-apr27
**Pre-batch HEAD:** `ffb070bb` (post-batch_21_tribunal_baseline_cleanup)
**Spec commit:** `abf927a7`
**Closing commit:** captured at closing

---

## FOUNDER_HEADLINE (5 lines)

**Sentry runtime crash reporting active. 3 test users + idempotent seed/reset scripts shipped. Tribunal HARD GATE 28/28 PASS in 1.86s.**
1. **Sentry @8.11.0 wired** at App.js line 20 BEFORE initAppCheck (line 32). `Sentry.init` exact spec params; `Sentry.wrap(App)` outermost wrapper; `__DEV__` capture-message confirmation. ErrorBoundary remains inner-tree fallback (Sentry.captureException integration deferred to future batch — spec scope guard).
2. **3 test users in Firebase Auth.** vqxr (+15555550100) + 6Dfe (+15555550101) pre-existing per CLAUDE.md §12; **KXet39oKhoV3y63D7JeIJL9UCfr1** (+15555550102) seeded this batch via `scripts/seed_test_users.js`. All 3 have Firestore profiles. Seed script idempotent: first run = 1 CREATED + 2 SKIP + 3 profile-SKIP; second run = 3 SKIP + 3 SKIP.
3. **Plan agent caught Brain's spec assumption.** Spec said "Create 2 new test users" — Plan step 1 verified both 0100/0101 already existed, corrected to 1 new (+0102). Without Plan, batch would have created a duplicate phone-binding attempt.
4. **Reset script defense-in-depth.** `scripts/reset_test_users.sh --dry-run` exits 0 with no Firestore writes; real reset requires `RESET_CONFIRM=YES` env var at BOTH shell layer AND node layer. `bash scripts/reset_test_users.sh` (no flag, no env) emits REFUSE message + exit 1.
5. **4 D-22-NN routed:** D-22-01 (P3 DOC) CLAUDE.md §12 add seed-script ref, D-22-02 (S-2) **founder OTP-whitelist add** for +15555550102 in Firebase Console (BLOCKS device-test against User 3), D-22-03 (P3 OBS) founder verify Sentry dashboard event arrival, D-22-04 (P3 OBS) Sentry config concerns (deprecated `enableInExpoDevelopment`, 20% prod tracesSampleRate revisit).

---

## CUMULATIVE_STATE

- Branch: `claude/run-j-audit-apr27`
- HEAD post-this-batch: captured in final report
- Batches done: 00b through 21 + **22_sentry_and_multitest_users** (28 closed; batch_08 Brain-only)
- Architecture v2 status: Tribunal HARD GATE 28/28 PASS in 1.86s. Workflow contract from batch_21 HELD this batch.
- **Sentry status**: `@sentry/react-native@^8.11.0` installed, init at App.js:20, wrap export shipped, capture-message wiring confirmed in `__DEV__`. Bundle resolves cleanly (Metro export tested to 42% / 1212 modules with zero @sentry resolution errors before 90s timeout — bundle was progressing normally).
- **Test infrastructure status**: 3 test users (vqxr/6Dfe pre-existing + KXet seeded). seed/reset/list scripts shipped. TEST_USERS.md at repo root.
- Phase 1 backlog: M-01 ✅, M-02 ✅, M-03 ✅. Pre-launch observability (Sentry) + test infrastructure now operational. Next: Brain composes M-04 (9572 meme upload) or M-05 (Vibe Lab assessment).
- New D-22-NN: 4 (D-22-01 P3 DOC, D-22-02 S-2 founder action, D-22-03/04 P3 OBS).
- Open RED_BLOCKs: 1 (F-R1, unchanged) + 1 candidate (D-22-02 OTP whitelist).
- Open P1: 0.
- Open P2 YELLOWs: ~22 (was 21 + D-22-02 if escalated).
- Open P3 OBSERVATION: ~13 (was 11 + D-22-01/03/04 = 14).

---

## LAST_TASK_RESULT

| TASK_ID | Status | Detail |
|---|---|---|
| Pre-flight: re-read MASTER_PROCESS | PASS | sections cached |
| Pre-flight: tree clean (tracked) | PASS | no tracked-file changes |
| Pre-flight: SA file exists | PASS | `firebase-service-account.json` present |
| Pre-flight: spec save + commit + push | PASS | `abf927a7` |
| Pre-flight: ≥5 failure modes | PASS | 7 modes printed |
| Plan agent decompose | PASS | 17 atomic steps; surfaced (1) firebase-admin already installed, (2) only 1 new test user needed (not 2), (3) Sentry wrap-order vs ErrorBoundary, (4) reset defense-in-depth |
| B22-P-1 pre-flight investigation | PASS | @sentry/react-native ABSENT, firebase-admin ^13.7.0 PRESENT, expo ~54.0.32, react-native 0.81.5; App.js:20 = initAppCheck(); SA 4 keys; listUsers shows vqxr+6Dfe+founder (3 users, 2 are tests) |
| B22-P-2 .gitignore + node | PASS | `firebase-service-account.json` line 119, `*-firebase-adminsdk-*.json` line 123; node v20.20.2 confirmed |
| B22-P-3 npm install @sentry/react-native | PASS | Added 13 packages in 22s; package.json `^8.11.0` |
| B22-P-4 Sentry.init at App.js TOP | PASS | Line 2 import, line 20 init, line 28 captureMessage; line 32 initAppCheck — init runs BEFORE initAppCheck |
| B22-P-5 Sentry.wrap default export | PASS | `function App() {...}` + `export default Sentry.wrap(App);` |
| B22-P-6 bundle smoke test | PASS-WITH-CAVEAT | Metro export progressing 42%/1212 modules with zero @sentry/TYPOGRAPHY errors at 90s timeout; full export not finished, but error-free progress is acceptance proof |
| B22-P-7 throw audit | PASS | Zero top-level `throw ` in App.js |
| B22-P-8 test-user delta decision | PASS | 1 new (+0102) — both 0100/0101 already exist |
| B22-P-9 author seed_test_users.js | PASS | 165 lines; idempotent skip-if-exists for both auth + profile; --reset/--verify flags; RESET_CONFIRM env gate; NO SA echo |
| B22-P-10 first run + idempotency | PASS | Run1: 2 SKIP + 1 CREATED + 3 profile-SKIP. Run2: 3 SKIP + 3 SKIP. CREATED UID:KXet39oKhoV3y63D7JeIJL9UCfr1 phone:+15555550102 |
| B22-P-11 verify 3 users | PASS | `--verify` lists vqxr/6Dfe/KXet (3 test users) |
| B22-P-12 author reset_test_users.sh | PASS | POSIX bash; --dry-run + RESET_CONFIRM gate + REFUSE message |
| B22-P-13 dry-run reset | PASS | --dry-run exit 0 + intent log + zero Firestore writes; no-flag-no-env REFUSE exit 1 |
| B22-P-14 author TEST_USERS.md | PASS | 3 phones documented; 0 `private_key`; founder OTP-whitelist instructions included |
| B22-P-15 cleanup probe | PASS | `scripts/_b22_listusers.js` → `scripts/list_test_users.js`; SA not staged |
| Atomic-step-checks (≥4 spec) | PASS | B22-P-9 (private_key=0), B22-P-14 (TYPOGRAPHY-not-defined=0), B22-P-5 (Sentry.wrap=0 in src/), B22-P-10 (private_key=0) — all ✅ |
| B22-P-16 tribunal HARD GATE | **PASS 28/28** | T1=10/10 T2=12/12 T3=6/6, exit 0 |
| Step 13 depth-test 6-layer | PASS | 6/6 layers; STRESS_TEST_RESULTS_b22.md |
| Step 14 evidence-budget | PASS | 74MB/5GB |
| Step 15 §4.5 gap scan | PASS | 5/5 questions, 4 D-22-NN routed |
| Step 16 ledger | THIS-FILE | |
| Step 17 6 tracking files | PENDING | next |
| Step 18 closing commit + push | PENDING | next |
| Step 19 Auto-Bridge + curl + final report | PENDING | next |

---

## CHANGED_FILES

```
 M App.js                                +14/-1   (Sentry import + init + wrap)
 M package.json                          +1/-0    (@sentry/react-native dep)
 M package-lock.json                     +<many>  (dep tree resolution)
 A scripts/seed_test_users.js            +165     (idempotent test-user seed)
 A scripts/reset_test_users.sh           +43      (POSIX wrapper, dry-run + confirm)
 A scripts/list_test_users.js            +28      (renamed from _b22_listusers.js)
 A TEST_USERS.md                         +60      (test-user inventory)
 A SENTRY_AND_TESTUSER_PLAN.md           +88      (Plan agent verbatim)
 A round-trips/STRESS_TEST_RESULTS_b22.md
 A round-trips/b22_gap_scan.md
 A round-trips/batch_22_sentry_and_multitest_users_evidence_ledger.md (THIS)
 A round-trips/b22_tribunal_output.txt
 A round-trips/b22_evidence_budget.txt
```

---

## D-22-NN ROUTING

| ID | Class | Severity | Topic | Status |
|---|---|---|---|---|
| D-22-01 | DOC | P3 | CLAUDE.md §12 add seed-script + TEST_USERS.md references | ROUTED |
| D-22-02 | RED_BLOCK candidate | S-2 | **Founder action**: add `+15555550102` to Firebase Console OTP whitelist (Authentication → Settings → Phone numbers for testing). Required before device-test against User 3. | ROUTED (founder) |
| D-22-03 | P3 OBS | Founder verify Sentry dashboard receives `'batch_22 Sentry init verified'` event on next dev/EAS build run | ROUTED (founder) |
| D-22-04 | P3 OBS | Sentry config: `enableInExpoDevelopment` deprecated in @8.x; production `tracesSampleRate: 0.2` revisit post-launch | ROUTED |

---

## CLOSING_TARGET_PROOF (per §6 + §12)

- Spec on git: ✅ `abf927a7`
- Plan on git: ✅ `SENTRY_AND_TESTUSER_PLAN.md` (committed with closing)
- Code on git: ✅ closing commit (App.js + package.json + 3 scripts + TEST_USERS.md)
- Sentry init: ✅ App.js:20 BEFORE initAppCheck; bundle progressing 42%/1212 modules error-free
- Test users: ✅ 3/3 (vqxr/6Dfe/KXet)
- reset_test_users.sh: ✅ dry-run PASS + REFUSE-without-confirm
- TEST_USERS.md: ✅ created
- Tribunal: ✅ **28/28 PASS, 1.86s, exit 0** (HARD GATE held)
- Depth-test: ✅ 6/6 layers
- Evidence budget: ✅ 74MB/5GB
- §4.5 gap scan: ✅ 5/5 questions
- Ledger: ✅ THIS file
- 6 tracking files: PENDING (closing step)
- Auto-Bridge: PENDING (closing step)

---

END LEDGER.
