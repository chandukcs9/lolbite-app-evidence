# batch_15_M-01_DevBypass — EVIDENCE LEDGER

**Composed:** 2026-05-07
**Branch:** claude/run-j-audit-apr27
**Pre-batch HEAD:** `683c12ee` (post-batch_14_SKILL_INTEGRATION closing)
**Spec commit:** `39a74bfd`
**Closing commit:** captured at closing

---

## FOUNDER_HEADLINE (5 lines)

**M-01 DevBypass + dev_user_001 elimination — code change DONE, end-to-end device verify PARTIAL.**
1. **20 live `dev_user_001`/`DevBypassScreen`/`DEV_MOCK_USER`/`getDevBypass`/`devBypass` references in `src/**/*.js` reduced to ZERO** (final grep clean). 5 files modified, +12/−59 net lines, single semantic change: kill the bypass mechanism + scrub the dummy-data UID literal.
2. **DevBypass UI no longer reachable on any platform.** AppNavigator now hardcodes `initialRouteName="Splash"`; the mock `AuthContext.Provider` wrap is gone; the `?dev=true` URL-param check is gone. `SHOW_DEV_BUTTONS = false` literal preserved with M-01 comment locking the discipline.
3. **Plan agent decomposed M-01 into 7 atomic steps and caught two scope errors in Brain's spec** (no `src/screens/devbypass*` file exists — bypass was inline in AppNavigator; no `App.js` edit needed; ChatScreen.js was already clean). All 7 steps PASS per their mechanical acceptance criteria.
4. **Smoke L1 on S25U shows NO DevBypass UI** (app launches to MainTabs/Home — persisted Firebase Auth session from prior test). **CAVEAT:** installed binary is rc9 from batch_10 (pre-M-01) and fresh-auth path was not exercised because session persisted. Filed D-15-01 for end-to-end rc10 device verify in a follow-up batch.
5. **No crash, no AndroidRuntime errors, no FATALs in logcat.** Static eslint check on changed files: 0 errors, 6 pre-existing warnings (none introduced by M-01).

---

## CUMULATIVE_STATE

- Branch: `claude/run-j-audit-apr27`
- HEAD post-this-batch: `<closing commit SHA>` — captured in final report
- Batches done: 00b, 01, 02, 02b, 02c, 03, 04a, 04b, 05, 05b, 05c, 06, 07, 09, 10, 09_VERIFY, 11_RECONCILE, 12_INVENTORY, 13_BACKLOG_v1, 14_SKILL_INTEGRATION, **15_M-01_DevBypass** (21 closed; batch_08 was Brain-only decision brief)
- Architecture v2: OPERATIONAL, augmented with first META execution
- Plan agent (decomposition primitive): VALIDATED in production use (caught 2 scope errors in Brain's spec)
- Device: SM-S938U1 / RFCY81H4F9Z, rc9 binary installed (pre-M-01); rc10 build deferred to follow-up
- Test user persisted session: present in app data (vqxr or 6DfeJ — uid not directly captured this batch; prior batches established these are the live whitelist users)
- M-01 status: **CODE_DONE_DEVICE_VERIFY_PARTIAL.** Source of truth at HEAD.
- Open RED_BLOCKs: 1 (F-R1 unchanged)
- Open P1: 0
- Open P2 YELLOWs: 19 (unchanged)
- Open P3 OBSERVATION: 4 (+D-15-01 below)

---

## LAST_TASK_RESULT

| TASK_ID | Status | Evidence |
|---|---|---|
| Pre-flight: §0/§0.6/§0.7/§5/§14 read | PASS | MASTER_PROCESS.md sections read via Read tool |
| Pre-flight: git status clean | PASS | `git status -sb` → only pre-existing untracked files |
| Pre-flight: spec commit + push | PASS | `39a74bfd batch_15_M-01_DevBypass spec` pushed |
| Pre-flight: ≥5 failure modes added | PASS | 8 additional FMs printed in chat before execution |
| DECOMPOSE: Plan agent invoked | PASS | Output captured verbatim in `M-01_PLAN.md` at root |
| DECOMPOSE: M-01_PLAN.md saved | PASS | File at root, 7 atomic steps + critical files + notes |
| M01-P-1 inventory snapshot baseline | PASS | Grep returned 20 hits in `src/`, all covered by Plan agent's pre-flight inventory; 0 orphan sites |
| M01-P-2 remove DEV_MOCK_USER + getDevBypass() | PASS-paired-with-P-3 | Definition removed; call sites in P-3 (per Plan agent's stated paired-acceptance) |
| M01-P-3 remove devBypass call sites + AuthContext.Provider wrap | PASS | `grep "devBypass\|devInitialRoute\|DEV_MOCK_USER\|AuthContext\.Provider"` returns 0 in AppNavigator.js; `initialRouteName="Splash"` literal; default `AuthContext` import dropped |
| M01-P-4 chatService.js dev_user_001 → DEV_LOCAL_UID | PASS | `grep "dev_user_001" src/services/chatService.js` → 0 matches |
| M01-P-5 userService.js + VibeLabDashboard comment | PASS | `grep -rn "dev_user_001" src/ --include="*.js"` → 0 matches; only AUDIT_RESULTS.md retains historical mentions (out of scope per Plan Step 5) |
| M01-P-6 SHOW_DEV_BUTTONS comment + literal false verified | PASS | config.js:9 reads `export const SHOW_DEV_BUTTONS = false;` (literal); M-01 comment above |
| M01-P-7 final grep + lint | PASS | `grep -rn "dev_user_001\|DevBypassScreen" src/ --include="*.js"` → 0; ESLint 0 errors, 6 pre-existing warnings |
| Gate: /security-audit subset | PASS | Check 5 (DEV_MODE config), Check 1 (secrets) — 0 new findings introduced; 4 pre-existing matches all false-positive substrings (`flask-outline`, `risk-taker`) or pre-existing FCM-token operations |
| Gate: /qa | NOT_APPLICABLE | gstack /qa is web-app QA via browse skill; this is RN code-only change; no dev server running per spec NEVER list. Static lint covers code-correctness gate. |
| Gate: /code-review (plugin) | NOT_APPLICABLE | Plugin requires `gh pr` against a PR; current work is direct-to-feature-branch. `git diff` reviewed manually below in EVIDENCE_LEDGER. |
| Gate: /secret-scan | PASS | Pre-commit hook runs `🔒 Running secret scan on staged files` on every commit this session — passed continuously. Will re-run on closing commit. |
| Smoke L1: ADB device check | PASS | `adb devices` → `RFCY81H4F9Z device` |
| Smoke L1: force-stop + cold start | PASS | `am start -n com.amovi.app/.MainActivity` returned `Starting: Intent`; `dumpsys window` shows `mCurrentFocus=Window{...com.amovi.app/com.amovi.app.MainActivity}` |
| Smoke L1: 30s screenshot | PARTIAL | Initial capture was black (screen asleep); woke device, re-captured at `screenshots/b15_smoke_30s.png` (216,050 bytes); shows MainTabs/Home with empty matches state — NOT the auth screen Brain's spec expected. |
| Smoke L1: no DevBypass UI | PASS | Screenshot shows Home + bottom tab bar (pizza/search/lab/chat/profile) + "no one on your wavelength yet" empty state. NO DevBypass-specific UI elements visible. |
| Smoke L1: auth screen visibility | NOT_VERIFIED | App landed at MainTabs because Firebase Auth session persisted from prior testing. Fresh-auth path not exercised this batch. Filed D-15-01. |
| Smoke L1: logcat clean | PASS | 348 lines pid-filtered; no FATAL/AndroidRuntime/exception. One warning: Firestore IndexedDB persistence falls back to memory cache (platform-known, not M-01-related) |
| Final META acceptance grep | PASS | `grep -rn "dev_user_001\|DevBypassScreen" src/ --include="*.js"` → 0 matches |

---

## EVIDENCE_LEDGER

### Files modified this batch (verified via git diff)

```
$ git diff --stat src/
 src/constants/config.js                      |  1 +
 src/navigation/AppNavigator.js               | 52 ++--------------------------
 src/screens/main/vibelab/VibeLabDashboard.js |  2 +-
 src/services/chatService.js                  | 14 ++++----
 src/services/userService.js                  |  2 +-
 5 files changed, 12 insertions(+), 59 deletions(-)
```

### Files created this batch

- `M-01_PLAN.md` (Plan agent output verbatim, ~5KB)
- `round-trips/batch_15_M-01_DevBypass_brain_task_spec.md` (committed at 39a74bfd)
- `round-trips/batch_15_M-01_DevBypass_evidence_ledger.md` (this file)
- `screenshots/b15_smoke_30s.png` (216,050 bytes)
- `logs/b15_smoke_logcat.txt` (348 lines)

### Inventory grep (M01-P-1 baseline)

```
src/AUDIT_RESULTS.md:471: ### MAJ-027: ControlRoom auth fallback to dev_user_001 in production
src/AUDIT_RESULTS.md:475: DESCRIPTION: `const currentUid = authUser?.uid || "dev_user_001"`...
src/AUDIT_RESULTS.md:644: ### MAJ-046: FullProfileView auth fallback to dev_user_001
src/AUDIT_RESULTS.md:648: DESCRIPTION: `const currentUid = authUser?.uid || "dev_user_001"` — same...
src/navigation/AppNavigator.js:115: const DEV_MOCK_USER = {
src/navigation/AppNavigator.js:121: function getDevBypass() {
src/navigation/AppNavigator.js:660: const devBypass = getDevBypass();
src/navigation/AppNavigator.js:662: devBypass ? null : undefined,
src/navigation/AppNavigator.js:667: if (!devBypass) return;
src/navigation/AppNavigator.js:670: }, [devBypass]);
src/navigation/AppNavigator.js:746: devBypass ? devInitialRoute || "MainTabs" : "Splash"
src/navigation/AppNavigator.js:806: if (devBypass) {
src/navigation/AppNavigator.js:810: user: DEV_MOCK_USER,
src/services/chatService.js:28-57: 7 dev_user_001 literals in DUMMY_CONVERSATIONS / DUMMY_MESSAGES
src/services/userService.js:45: id: "dev_user_001", // DEV_MODE
src/screens/main/vibelab/VibeLabDashboard.js:117: comment-only mention
```

### Final grep (M01-P-7 acceptance)

```
$ grep -rn "dev_user_001\|DevBypassScreen" src/ --include="*.js"
(0 matches)

$ grep -rn "dev_user_001\|DevBypassScreen" App.js
(0 matches)

$ grep -rn "dev_user_001" src/
src/AUDIT_RESULTS.md:471: ### MAJ-027: ControlRoom auth fallback to dev_user_001 in production
src/AUDIT_RESULTS.md:475: DESCRIPTION: ...
src/AUDIT_RESULTS.md:644: ### MAJ-046: FullProfileView auth fallback to dev_user_001
src/AUDIT_RESULTS.md:648: DESCRIPTION: ...
(4 matches in AUDIT_RESULTS.md — historical audit doc, out of scope per Plan Step 5)
```

### ESLint on changed files (M01-P-7)

```
src/screens/main/vibelab/VibeLabDashboard.js
   24:18  warning  'FONT_SIZES' is defined but never used        (PRE-EXISTING)
   24:30  warning  'RADIUS' is defined but never used            (PRE-EXISTING)
   24:38  warning  'SPACING' is defined but never used           (PRE-EXISTING)
   30:16  warning  'SCREEN_WIDTH' is assigned a value but never used  (PRE-EXISTING)
  178:9   warning  'calibrationLevel' is assigned a value but never used  (PRE-EXISTING)

src/services/userService.js
  15:10  warning  'DEV_MODE' is defined but never used  (PRE-EXISTING — DEV_MODE only referenced in comments now)

✖ 6 problems (0 errors, 6 warnings)
```

All 6 warnings are pre-existing — none introduced by M-01 edits.

### Smoke L1 device evidence

```
$ adb devices
RFCY81H4F9Z	device

$ adb shell pm list packages | grep amovi
package:com.amovi.app

$ adb shell am force-stop com.amovi.app
(no output)

$ adb shell am start -n com.amovi.app/.MainActivity
Starting: Intent { cmp=com.amovi.app/.MainActivity }

$ adb shell "dumpsys window | grep -E 'mCurrentFocus|mFocusedApp'"
mCurrentFocus=Window{af080ab u0 com.amovi.app/com.amovi.app.MainActivity}
mFocusedApp=ActivityRecord{92777406 u0 com.amovi.app/.MainActivity t222}

$ adb exec-out screencap -p > screenshots/b15_smoke_30s.png
(216,050 bytes)

$ adb shell pidof com.amovi.app
12276

$ adb logcat -d --pid=12276 > logs/b15_smoke_logcat.txt
(348 lines)

$ grep -iE "FATAL|AndroidRuntime|Exception|crash|error" logs/b15_smoke_logcat.txt
05-07 22:31:50.968 12276 12324 W ReactNativeJS: '[firebase/firestore] Firestore (12.9.0): Error using user provided cache. Falling back to memory cache: ... [code=unimplemented]: IndexedDB persistence is only available on platforms that support LocalStorage.'
(1 warning — platform-known fallback, not crash)
```

Multimodal screenshot review (per CLAUDE.md §3): Read tool inspected `screenshots/b15_smoke_30s.png`. Visible:
- Status bar: "10:32 GOOGL ... 100%" (battery 100%, signal strong)
- Top: "Amovi" wordmark in primary orange (capital A, no period — NOTE: this is Home screen, NOT in batch_10's wordmark fix scope which was 5 auth screens only; pre-existing rc9 state)
- Center: people-pair icon in orange circle
- Heading: "no one on your wavelength yet" + small "🔵" icon at end
- Subtitle: "switch up the energy. tweak filters, watch vibes roll in"
- CTA button: "ADJUST FILTERS" (orange filled, rounded)
- Bottom tab bar: pizza icon (left), search, flask/lab in center orange circle (highlighted), chat/speech bubble, profile (right)
- Visible screen is **Home tab in MainTabs**, NOT auth screen
- **Zero DevBypass UI elements visible** — no "Dev Bypass" button, no "Skip onboarding" link, no dev-mode chip, no `?dev=true` indicator

### CLOSURE_AUDIT (every item terminal-state per §7)

| Item | Terminal state | Required fields |
|---|---|---|
| `DEV_MOCK_USER` constant removed | CLOSED_GREEN | commit SHA in closing; verify `grep -n "DEV_MOCK_USER" src/` → 0 |
| `getDevBypass()` function removed | CLOSED_GREEN | commit SHA; verify `grep -n "getDevBypass" src/` → 0 |
| `devBypass` call sites removed | CLOSED_GREEN | commit SHA; verify `grep -n "devBypass" src/` → 0; `initialRouteName="Splash"` literal |
| `AuthContext.Provider` mock-wrap removed | CLOSED_GREEN | commit SHA; verify `grep -n "AuthContext\.Provider" src/` → 0 |
| Default `AuthContext` import dropped | CLOSED_GREEN | line 26 reads `import { useAuth } from "../context/AuthContext";` |
| `dev_user_001` literals in chatService.js | CLOSED_GREEN | 7 sites → `DEV_LOCAL_UID`; grep → 0 |
| `dev_user_001` literal in userService.js | CLOSED_GREEN | 1 site → `DEV_LOCAL_UID`; grep → 0 |
| VibeLabDashboard.js comment scrub | CLOSED_GREEN | line 117 rewritten to remove literal |
| `SHOW_DEV_BUTTONS = false` flag enforcement | CLOSED_DOCUMENTED | M-01 comment added above line 9; literal `false` preserved |
| `App.js` DevBypass route registration | CLOSED_DOCUMENTED | App.js never had a DevBypass route — Plan agent verified; `grep "DevBypass\|DEV_MOCK_USER\|dev_user_001" App.js` → 0 |
| `src/screens/devbypass*` file deletion | CLOSED_DOCUMENTED | No such file exists — Plan agent verified; bypass was inline in AppNavigator |
| `ChatScreen.js` cleanup | CLOSED_DOCUMENTED | File was already clean of `dev_user_001` literals — no edit needed; M-02 covers Chat refactor |
| Smoke L1 on S25U with rc9 binary | CLOSED_DOCUMENTED-WITH-CAVEAT | Screenshot + logcat captured; no DevBypass UI visible; auth path not fresh-tested due to persisted session; rc9 is pre-M-01 binary |
| End-to-end M-01 device verify on rc10 | ROUTED_NEXT_BATCH | D-15-01 below; routed to a follow-up batch (M-01b or first rebuild batch) |

GATE_STATUS: PASS-WITH-CAVEAT (smoke L1 verification gap on rc9 binary, full validation pending rc10 build).

### §4.5 GAP DISCOVERY (5-question scan)

**Q1: What did this batch ACTUALLY cover?**
- Removal of DevBypass mechanism from `src/navigation/AppNavigator.js` (definition + 3 call sites + mock-AuthContext wrap + import cleanup)
- Scrub of `dev_user_001` literal from 7 sites in `chatService.js` (DUMMY_CONVERSATIONS / DUMMY_MESSAGES) + 1 site in `userService.js` (DUMMY_USER_PROFILE.id) + 1 comment in `VibeLabDashboard.js`
- Annotation of `SHOW_DEV_BUTTONS` flag with M-01 discipline comment (literal `false` preserved)
- Static lint check on 5 changed files (0 errors)
- Subset of /security-audit (Check 1 secrets + Check 5 misconfig) — clean
- Pre-commit secret-scan via hook on closing commit
- Smoke L1 on S25U rc9: app launch + foreground activity check + screenshot + logcat — no crash, no DevBypass UI

**Q2: What did this batch TOUCH but not deeply test?**
- **D-15-01 [P2 OPEN]**: Fresh-auth flow on rc10 binary with M-01 changes. The installed rc9 doesn't carry M-01 changes; the persisted Firebase session prevented fresh-auth observation. Target: M-01b or first rebuild batch.
- **D-15-02 [P3 OPEN]**: VibeLabDashboard.js has 4 unused theme imports (FONT_SIZES/RADIUS/SPACING/SCREEN_WIDTH) — pre-existing lint warnings surfaced by ESLint run this batch. Target: design-cleanup batch.
- **D-15-03 [P3 OPEN]**: `userService.js:15` `DEV_MODE` import is now unused (after M-01 only references it in comments). Either remove the import or refactor a code site to use the flag explicitly. Target: M-02 (when ChatScreen baseline scaffold work touches services).
- **D-15-04 [P2 OPEN]**: `src/AUDIT_RESULTS.md` MAJ-027 + MAJ-046 reference `authUser?.uid || "dev_user_001"` in **ControlRoomScreen.js** and **FullProfileView.js** — but my live grep found 0 hits in those files. Either the audit doc is stale (those fallbacks were already removed), or the code is in a different file. Target: short investigation in M-02 prep.

**Q3: What new bug classes does this batch hint at?**
- **D-15-05 [P3 OBSERVATION]**: Multiple files import `DEV_MODE` and rely on it as a `__DEV__` alias. Future RN-version changes that affect `__DEV__` semantics could change DEV_MODE behavior in non-obvious ways. Target: codify in MEMORY.md / project doc.
- **D-15-06 [P3 OBSERVATION]**: `DEV_LOCAL_UID` placeholder string is now in 8 sites across chatService.js + userService.js. If a real user ever has UID `DEV_LOCAL_UID` (via Firebase Auth uid collision — extremely unlikely but theoretically possible), DUMMY data would be served. Target: low-priority polish; could prefix with non-Firebase-uid-shaped value like `__DEV_LOCAL_UID__`.

**Q4: What did this batch discover that wasn't on the upfront plan?**
- The bypass mechanism lives **inline in AppNavigator.js**, not in a separate `DevBypassScreen.js` file (Plan agent caught this).
- `App.js` has zero references to DevBypass — Brain's spec listed it in scope but it was unnecessary.
- `ChatScreen.js` was already clean of `dev_user_001` — Brain's spec listed it as scope-IN but only `chatService.js` and `userService.js` had the literals.
- A `rcscreensDevBypassScreen.js` file exists at the **repo root** (not under `src/`) — out of scope but worth noting for future cleanup. Target: D-15-07 below.

**Q5: What edge cases were partially tested?**
- **D-15-07 [P3 OPEN]**: `rcscreensDevBypassScreen.js` at repo root (not in `src/`). Plan agent flagged this as out of scope per the META's `src/` boundary, but if it's tracked in git it's part of the repo. Target: file-cleanup batch.
- **D-15-08 [P2 OPEN]**: Web `?dev=true` testing PATH was never observed broken in this batch — only the static code change was verified. A web build with the new code would confirm the bypass actually doesn't activate on `?dev=true`. Target: when web testing resumes (currently no web flow is in v1 scope).

### Routing decisions

- D-15-01 (P2 — rc10 device verify) → ROUTED_NEXT_BATCH (M-01b or first rebuild)
- D-15-02 (P3 — pre-existing lint warnings VibeLabDashboard) → ROUTED_NEXT_BATCH (design-cleanup, no specific target)
- D-15-03 (P3 — DEV_MODE unused import in userService.js) → ROUTED_NEXT_BATCH (M-02 prep)
- D-15-04 (P2 — AUDIT_RESULTS.md staleness re ControlRoom/FullProfileView fallback) → ROUTED_NEXT_BATCH (M-02 prep)
- D-15-05 (P3 OBSERVATION — DEV_MODE→__DEV__ alias risk) → backlog
- D-15-06 (P3 OBSERVATION — DEV_LOCAL_UID collision theoretical) → backlog
- D-15-07 (P3 — rcscreensDevBypassScreen.js at root) → ROUTED_NEXT_BATCH (file-cleanup)
- D-15-08 (P2 — web ?dev=true verification gap) → DEFERRED (no web in v1 scope)

---

## ASK

- Brain to compose **batch_16 M-02 ChatScreen real-user wiring** (next META in sequence per BACKLOG_PHASE_1.md). M-02 dependencies on M-01 are satisfied — DevBypass is gone, dev_user_001 literals scrubbed.
- Brain to decide on rc10 EAS build cadence: bundle M-01 + M-02 device verify together in M-02's closing batch (likely cheapest), or kick a dedicated M-01b rebuild batch first. Recommendation: bundle, since D-15-01 + M-02 testing share the rc10 build cost.
- D-15-04 (AUDIT_RESULTS.md ControlRoomScreen/FullProfileView audit-doc references with no live grep hits) deserves quick investigation — likely the audit doc is stale, but worth confirming before relying on it for M-02 scope.

---

## RETROSPECTIVE FAILURE MODES (≥13)

1. Plan agent misreads scope — Did not occur; agent expanded scope correctly (caught App.js no-op + ChatScreen.js already-clean + DevBypass-inline-in-AppNavigator).
2. atomic step acceptance ambiguity — Occurred at P-2/P-3 (paired acceptance); resolved by reading Plan agent's explicit note "those are removed in Step 3 of this same atomic step set."
3. Linter introduces new warnings — Did not occur; 6 warnings all pre-existing.
4. ESLint can't run (Node not on PATH) — Occurred; mitigated by direct `node.exe` path per MEMORY.md.
5. Smoke L1 against pre-M-01 binary — Occurred; logged D-15-01.
6. Persisted Firebase Auth session masks fresh-auth verification — Occurred; logged D-15-01.
7. Black-screen screenshot on first capture — Occurred (phone asleep); mitigated by KEYCODE_WAKEUP + retry.
8. ADB unauthorized — Did not occur; device was authorized.
9. App crash on launch after M-01 changes — Did not occur (no FATAL in logcat).
10. New unused-import warning from removed AuthContext default import — Did not occur; useState/useEffect still in use.
11. `/qa` slash command interactive blocker — Anticipated; documented as NOT_APPLICABLE rather than invoke.
12. `/code-review` plugin requires PR — Anticipated; documented as NOT_APPLICABLE; manual diff review captured here.
13. Pre-commit secret-scan trips on diff — Did not occur; will run on closing commit.
14. PowerShell `$_` mangling under Bash — Did not occur this batch (used Read/Edit/Grep tools instead of inline PowerShell where possible).
15. ChatScreen.js scope mismatch with Brain spec — Occurred; Plan agent surfaced (already clean); no edit needed.
16. App.js scope mismatch with Brain spec — Occurred; Plan agent surfaced (no DevBypass refs); no edit needed.
17. AUDIT_RESULTS.md staleness — Discovered (D-15-04); not blocking this batch.
18. rcscreensDevBypassScreen.js at root not in src/ — Discovered (D-15-07); out of scope for `src/`-bounded acceptance.

---

## END OF EVIDENCE LEDGER
