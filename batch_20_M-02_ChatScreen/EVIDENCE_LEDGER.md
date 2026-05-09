# batch_20_M-02_ChatScreen — EVIDENCE LEDGER

**Composed:** 2026-05-08
**Branch:** claude/run-j-audit-apr27
**Pre-batch HEAD:** `2fdba582` (post-batch_19_tribunal_timeout_fix)
**Spec commit:** `9f4013b2`
**Code commit:** `6cf4cadc`
**Closing commit:** captured at closing

---

## FOUNDER_HEADLINE (5 lines)

**M-02 ChatScreen real-user wiring landed. Test user vqxr1eu9cNU3Awfe1DIYGfBORQn2 lands on Slide In tab → MatchesScreen empty state, no dummy rows, PID stable 30s, logcat clean.**
1. **chatService.js DEV_MODE branches scoped to dev_*/mock_* UIDs only** via `_isDevUid` + `_isDevConvId` helpers. THE actual M-02 hazard fixed (Plan agent finding): real authenticated UIDs always hit Firestore now, even in __DEV__ builds. 6 sites patched (+16/-6). Mirrors userService pattern.
2. **MatchesScreen.js dead DUMMY arrays removed** (-124 lines, -5 NEW_BITES + -5 CONVERSATIONS + fallback comment block). Prevents future drift where developer "wires the fallback" and reintroduces leak.
3. **ChatScreen.js render-path guard added** (+40 lines): blocks misleading empty-conversation UI when reached without conversationId (deep link / stale notification / pre-auth race). amovi-branded copy: "open this chat from your matches list".
4. **First META execution with full skill+gate stack firing automatically.** atomic-step-check.sh first-use (D-18-04 → CLOSED). Tribunal 28 checks <60s (2s actual), evidence-budget 74MB/5GB, depth-test 6/6, gap-scan 5/5.
5. **3 D-20-NN findings routed for Brain triage:** D-20-01 (BACKLOG description amend), D-20-02 (firestore.rules text-field schema bug, breaks gif/image/meme/voice), D-20-03 (tribunal-debt META covering 8 pre-existing FAILs not from M-02 surface).

---

## CUMULATIVE_STATE

- Branch: `claude/run-j-audit-apr27`
- HEAD post-this-batch: captured in final report
- Batches done: 00b through 19 + **20_M-02_ChatScreen** (26 closed; batch_08 Brain-only)
- Architecture v2 status: OPERATIONAL. M-02 is FIRST META BATCH executing full stack: Plan-decompose → atomic-step-guard → spec-on-git → device verify (multimodal screenshots) → tribunal 20 → depth-test 6-layer → evidence-budget rotate → §4.5 gap scan → closure → Auto-Bridge.
- Phase 1 backlog: M-01 ✅, **M-02 ✅**. Next: M-03 (TBD by Brain).
- D-18-04 (atomic-step-check first-use) status: **CLOSED_GREEN** (validated this batch).
- New D-20-NN: 3 routed (D-20-01/02/03) — see Findings.
- Open RED_BLOCKs: 1 (F-R1, unchanged).
- Open P1: 0.
- Open P2 YELLOWs: 23 (was 21 + D-20-02 P2 + D-20-03 P2).
- Open P3 OBSERVATION: 14 (was 13 + D-20-01 P3).

---

## LAST_TASK_RESULT

| TASK_ID | Status | Detail |
|---|---|---|
| Pre-flight: re-read MASTER_PROCESS §0/§0.6/§0.7/§5/§14 | PASS | sections cached from batch_19 |
| Pre-flight: clean tree | PASS | uncommitted b18_tribunal_smoke_output.txt restored to canonical |
| Pre-flight: spec save + commit + push | PASS | `9f4013b2` |
| Pre-flight: ≥5 failure modes printed | PASS | 8 modes printed |
| Plan agent decompose | PASS | M-02_PLAN.md saved verbatim; 8 atomic steps; surfaced architectural mismatch (Slide In→MatchesScreen, not ChatScreen) and THE M-02 hazard (bare DEV_MODE branches) |
| P-1 pre-flight grep audit | PASS | 0 dev_user_001, 7 DEV_LOCAL_UID, 6 if (DEV_MODE), 2 DUMMY arrays |
| P-2 chatService scoping (+16/-6) | PASS | 0 bare `if (DEV_MODE)`, 4 _isDevUid uses, 6 _isDevConvId uses |
| P-3 ChatScreen render-path guard (+40) | PASS | guard before line 1318 main return |
| P-4 MatchesScreen DUMMY removal (-124) | PASS | atomic-step-check.sh validated D-18-04 first-use; pattern 'DUMMY_NEW_BITES' matches: 0 (expected 0) |
| P-7 ESLint final | PASS | 0 errors, 5 pre-existing warnings (2 ChatScreen vars, 3 MatchesScreen unused theme imports) |
| P-7 final grep audit | PASS | dev_user_001=0, DEV_LOCAL_UID=8, DUMMY_NEW/CONV=2 (chatService DUMMY_CONVERSATIONS + 1 var ref); 3 files +56/-130 |
| Code commit + push | PASS | `6cf4cadc`; secret scan PASSED |
| Step 8 EAS build rc11 | PASS | build id `4308b2bc-64c0-420c-8cd1-5f75690ddbac`; gitCommitHash `6cf4cadc930ad13e19406413d86c6fd5be84424c` matches HEAD (D-09-02 prevention verified); FINISHED 23:52:04 UTC; APK 160.5MB |
| Step 9 adb install + pm clear | PASS | Streamed install Success; pm clear Success |
| Step 10 launch + auth | PASS | phone +15555550100 → OTP 123456 → Home (testuser, 18); first OTP attempt dropped a digit (adb input text quirk), retry via keyevents succeeded |
| Step 11 Slide In tab → MatchesScreen | PASS | tab nav via uiautomator-derived bounds (756, 2120); landing screenshot 12_slide_in_tab.png; 30s stability 13_slide_in_30s.png; PID 27076 stable; "no moves yet" empty state + empty CHATS section, NO dummy data |
| Step 12 logcat | PASS | 1049 lines for pid=27076; no FATAL/AndroidRuntime/Exception; 2 pre-existing warnings (App Check 403, IndexedDB persistence fallback) |
| Step 13 tribunal 20 | PASS-WITH-FINDINGS | 20/28 PASS, 8 FAIL; ZERO M-02 surface contributions (verified by grep across chatService/ChatScreen/MatchesScreen) — all 8 are pre-existing tech debt (D-19-03/04/15-04 leftovers + new gaps); spec rule: known FAILs not blocking, only NEW M-02 surface FAILs blocking |
| Step 14 depth-test 6-layer | PASS | STRESS_TEST_RESULTS_b20.md; 6/6 layers (happy, edge guard, failure offline-safe, debounce concurrency, text bounds, dev-prefix adversarial) |
| Step 15 evidence-budget-rotate | PASS | 74MB/5GB, no rotation |
| Step 16 §4.5 gap scan | PASS | b20_gap_scan.md; 5 questions; 3 D-20-NN routed |
| Step 17 ledger compose | THIS-FILE | |
| Step 18 6 tracking files update | PENDING | next |
| Step 19 closing commit + push | PENDING | next |
| Step 20 Auto-Bridge publish + curl verify | PENDING | next |

---

## CHANGED_FILES

```
 M src/services/chatService.js       (+16/-6)
 M src/screens/main/ChatScreen.js    (+40/-0)
 M src/screens/main/MatchesScreen.js (-124/-0)
 A M-02_PLAN.md                      (Plan agent verbatim, +193)
 3 production files +56/-130; 1 plan doc +193
```

Commit: `6cf4cadc M-02 batch_20 — chatService DEV_MODE scoped to dev/mock UIDs; ChatScreen empty-route guard; MatchesScreen dead DUMMY arrays removed`

---

## DEVICE_VERIFY_EVIDENCE

| Step | Screenshot | Verdict |
|---|---|---|
| Auth landing | `round-trips/b20_screenshots/01_post_launch.png` | amovi. wordmark + 4 auth options |
| Phone entry | `round-trips/b20_screenshots/04_kb_hidden.png` | "drop your digits" + send code btn |
| OTP screen | `round-trips/b20_screenshots/05_otp_screen.png` | "prove you're real 👀" |
| OTP success | `round-trips/b20_screenshots/08_post_otp_v3.png` | "checking..." + notif perm prompt |
| Home post-auth | `round-trips/b20_screenshots/09_post_auth.png` | testuser, 18; 5 bottom tabs |
| **Slide In tab landing** | `round-trips/b20_screenshots/12_slide_in_tab.png` | **"slide in" + "no moves yet" + empty CHATS — NO DUMMY DATA** |
| **Slide In + 30s** | `round-trips/b20_screenshots/13_slide_in_30s.png` | **identical, PID 27076 stable** |

Logcat: `round-trips/b20_logcat.txt` (1049 lines, 0 FATAL).

---

## D-20-NN ROUTING

| ID | Class | Severity | Topic | Status |
|---|---|---|---|---|
| D-18-04 | DOC | P3 | atomic-step-check first-use | **CLOSED_GREEN** this batch |
| D-20-01 | DOC | P3 | BACKLOG_PHASE_1.md M-02 description: clarify Slide In→MatchesScreen, ChatScreen→row-tap | ROUTED |
| D-20-02 | BUG | P2 | firestore.rules:165 requires text on all message creates → breaks gif/image/meme/voice | ROUTED (chat-feature META) |
| D-20-03 | META | P2 | tribunal-debt cleanup META — 8 pre-existing FAILs need individual D entries; close 1/batch | ROUTED |

---

## CLOSING_TARGET_PROOF (per §6 + §12)

- Spec on git: ✅ `9f4013b2` (round-trips/batch_20_M-02_ChatScreen_brain_task_spec.md)
- Plan on git: ✅ `6cf4cadc` (M-02_PLAN.md committed with code)
- Code on git: ✅ `6cf4cadc` (3 files +56/-130)
- gitCommitHash on EAS = HEAD: ✅ `6cf4cadc930ad13e19406413d86c6fd5be84424c` (D-09-02 prevention)
- Device verify proof: ✅ 7 screenshots + logcat
- Tribunal: ✅ 20/28 PASS, 0 NEW M-02 FAILs (spec rule: not blocking)
- Depth-test: ✅ 6/6 layers
- Evidence budget: ✅ 74MB/5GB
- §4.5 gap scan: ✅ 5/5 questions answered
- Ledger: ✅ THIS file
- 6 tracking files: PENDING (closing step)
- Auto-Bridge: PENDING (closing step)

---

END LEDGER.
