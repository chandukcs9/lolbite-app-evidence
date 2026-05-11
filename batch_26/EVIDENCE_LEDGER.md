# batch_26 EVIDENCE_LEDGER

**Composed:** 2026-05-10
**Branch:** `claude/run-j-audit-apr27`
**HEAD at composition:** `2f916a29` (phase_3.5)
**Spec-of-record:** `round-trips/batch_26_brain_task_spec.md` (committed pre-execution per §18)
**Resume prompt:** RESUME batch_26 — PREREQS SATISFIED (founder unblock answers 1-7)

## Phase status summary

| Phase | Sub-task | Status | SHA | Evidence path |
|---|---|---|---|---|
| 1.1 | Read BACKLOG_v2 | ✅ | (carried from batch_25) | `docs/backlog/BACKLOG_v2.md` |
| 1.2 | Web research (5+ sources) | ✅ | `546b7560` | `docs/persona_test/BIO_RUBRIC.md#Research-sources-consulted` |
| 1.3 | BIO_RUBRIC.md | ✅ | `546b7560` | `docs/persona_test/BIO_RUBRIC.md` (144 lines) |
| 1.4 | 25 persona seed configs | ✅ | `546b7560` | `scripts/persona_test/seeds/PERSONAS.json` + `SCHEMA.md` |
| 1.5 | Gemini bio generation | ✅ | `1034b223` | `scripts/persona_test/seeds/BIOS.json` (25/25 ACCEPT) |
| 1.6 | Vibe Lab Deep Dive responses | ✅ | `a99bf1b6` | `scripts/persona_test/seeds/DEEP_DIVE_RESPONSES.json` (1000 answers) |
| 2 | Firestore persona seeding | ✅ | `3aa40f1a` | `scripts/persona_test/seeds/SEEDED_UIDS.json` + Firestore docs |
| 3 | Brain algorithm validation | ✅ | `5af17dbf` | `scripts/persona_test/seeds/BRAIN_VALIDATION_REPORT.md` + `MATCH_MATRIX.json` |
| 3.5 | Tenor → GIPHY migration | ✅ | `2f916a29` | `src/constants/config.js` + `src/screens/main/ChatScreen.js` |
| 4 | Function testing on S25U | ⏸️ DEFERRED | — | rc14 EAS build in upload phase; device tests blocked on build completion |
| 5 | AI features + Permissions + Lifecycle | ⏸️ DEFERRED | — | depends on Phase 4 |
| 6 | Edge cases + closeout | partial — closeout in progress | — | this ledger + round_trip md |

## V-verdicts (with specific evidence)

### V1 — Phase 1.5: 25 bios meet rubric threshold

**Evidence:** `scripts/persona_test/seeds/BIOS.json`
- 25 entries, all `status: "ACCEPT"`
- Aggregate: avg composite 4.52, avg 1943 chars
- Per-persona scores (sample, persona_01 Maya):
```json
{
  "specificity": 5,
  "voice": 5,
  "hook": 5,
  "length": 5,
  "anti_cliche": 5,
  "composite": 5.0,
  "len": 1690
}
```
- Self-scoring rubric: `scripts/persona_test/gen_bios.js#scoreBio` (lines 151-209)
- Acceptance threshold per `docs/persona_test/BIO_RUBRIC.md#Composite-scoring`: composite ≥4.0

**Verdict: PASS**

### V2 — Phase 1.6: All 25 personas have full Deep Dive answer sets

**Evidence:** `scripts/persona_test/seeds/DEEP_DIVE_RESPONSES.json`
- 25 personas, all `status: "ACCEPT"` (validated by `gen_deep_dive.js#validateTextAnswers`)
- 31 text answers + 8 choice + 1 slider = 40 answers per persona = 1000 total
- Sample (persona_01 Maya, q3 "Roman Empire"):
```
"wong kar-wai films. i watch 'in the mood for love' at least twice a month. the aesthetics, the longing... it gets me."
```
- Voice consistency confirmed: lowercase, specific named details, no banned phrases (validateTextAnswers `bannedHits=0` for all 25)

**Verdict: PASS**

### V3 — Phase 2: Firestore seeding writes verified by spot-readback

**Evidence:** `scripts/persona_test/seeds/SEEDED_UIDS.json` + Firestore readback
- 25 personas seeded; 10 to phone-auth uids (2 found existing, 8 created via admin.createUser); 15 to synthetic `b26_test_persona_NN` uids
- Spot-readback persona_01 Maya at uid `vqxr1eu9cNU3Awfe1DIYGfBORQn2`:
  - `name: "maya"`, `age: 24`, `gender: "woman"`, `lookingFor: "everyone"`
  - `__testPersona: "persona_01"`, `__batch: "batch_26"` markers present
  - `photos.length === 6` (picsum.photos URLs)
  - `bio.length === 1690`
  - `vibelab/deepDive` subcoll exists with 40 answers
- Spot-readback persona_05 Arjun at synthetic uid `b26_test_persona_05`:
  - same shape; bio 2244 chars; 40 deep dive answers

**Verdict: PASS**

### V4 — Phase 3: Brain algorithm validation produced concrete metrics + routed findings

**Evidence:** `scripts/persona_test/seeds/BRAIN_VALIDATION_REPORT.md`
- Aggregate accuracy:
  - Top-3 hit (≥2/3 overlap): 4/25 = **16.0%**
  - Top-3 partial (≥1/3 overlap): 15/25 = 60.0%
  - Bot-3 hit (≥2/3 overlap): 2/25 = 8.0%
  - Bot-3 partial (≥1/3 overlap): 17/25 = 68.0%
- Eligible pairs scored: 368 (gender×lookingFor filter applied)
- Discovery routing in `round-trips/batch_26/IN_FLIGHT_FIXES.md`:
  - **D-26-01 (P2):** 40/40/20 algorithm reads `memeReactions` + `foodSelections` + `location` only; PERSONAS.json predictedMatches were composed using personality+interests overlap. Decision required from Brain.
  - **D-26-02 (P3):** Algorithm has no anti-pattern modeling; bot-3 weakly signaled.

**Verdict: VALIDATED — data delivered, gap documented, routed**

### V5 — Phase 3.5: Tenor → GIPHY migration complete; zero remaining Tenor refs

**Evidence:**
- `src/constants/config.js:45-49` — env-loaded GIPHY key replaces hardcoded Tenor key
- `src/screens/main/ChatScreen.js:157-158` — `GIPHY_API_BASE = "https://api.giphy.com/v1/gifs"`
- `src/screens/main/ChatScreen.js:580-602` — search uses `?api_key=` and parses `data[].images.fixed_height`
- `src/screens/main/ChatScreen.js:2068-2075` — "powered by GIPHY" attribution badge in picker header (GIPHY TOS requirement: https://developers.giphy.com/branding-guidelines/)
- `grep -rn "tenor\|TENOR" src/` returns 0 hits
- `npx eslint` on changed files: 0 errors (only pre-existing warnings)
- `@babel/parser` validates both files OK
- `docs/backlog/BACKLOG_v2.md` BL-20260510-01 marked `MIGRATED` with founder follow-up note (revoke old key on Google Cloud Console)

**Verdict: PASS — code merged; founder must still revoke the old Tenor key on Google Cloud Console (was committed to public companion git history pre-batch_26)**

### V6 — Phase 4: rc14 build complete; Layer 1 + Layer 3 partial-pass on device

**Evidence:**
- EAS build `3a1603ea-c668-4f99-9a96-8a4da2ea1432` FINISHED at 2026-05-10T15:32:37; commit `2f916a293b3712a464ee9778725cd2c0c5b9519c` (= local HEAD `2f916a29` with the Phase 3.5 GIPHY migration). APK URL: `https://expo.dev/artifacts/eas/8b1Z8hyGMyRZX7imgNnV8y.apk` (173 MB).
- Build profile: `preview`. Distribution: internal. Version: `1.1.0-rc9` (D-07-01 carry-forward — appVersionSource=remote auto-version held). Fingerprint: `d68ec00e678353b0ec2ec48ff8d26987d4c6366c`.
- `adb -s RFCY81H4F9Z install -r /tmp/rc14.apk` → "Success". `dumpsys package com.amovi.app` confirms `lastUpdateTime=2026-05-10 15:33:59`.
- **CAVEAT — env vars not in this APK:** rc14 was queued BEFORE `eas env:create` added `EXPO_PUBLIC_GIPHY_API_KEY` and `EXPO_PUBLIC_GEMINI_API_KEY` to the `preview` env. EAS env values bind at queue time. Build log line "No environment variables with visibility 'Plain text' and 'Sensitive' found for the 'preview' environment on EAS." confirms. The migration code is in the APK; only the GIPHY key is empty (degraded behavior: GIF picker calls return 401 → fallback empty list; "powered by GIPHY" badge still renders since it's static UI). rc15 with env vars in place is the next-session ask.

#### Layer 1 SMOKE — partial pass

**Component 1 (Cold start → Landing screen → screenshot auth):** ✅ PASS
- Force-stop + monkey-launch + 8s wait + screenshot.
- `screenshots/b26_01_coldstart.png` shows: amovi. wordmark (italic lowercase amber, batch_10 D-06-01 fix rendering correctly), "where memes meet matches" tagline, 4 auth buttons (Continue with phone / Google / Instagram / Snapchat), legal links, "been here before? sign in", **ZERO DevBypass UI elements** (M-01 closure held).

**Component 11 (Logcat aggregate, 0 FATAL):** ✅ PASS
- `adb logcat -d -t 500 *:E | grep -E "FATAL|AndroidRuntime|Exception|com.amovi"` returns 0 hits across full Phase 4 partial session.
- Only system errors found: `WifiStaIfaceAidlImpl getCachedScanData` Samsung wifi driver chatter (not app-related).

**Component 2 (Auth Phone OTP — phone entry → OTP screen):** ✅ PARTIAL PASS
- Tap "Continue with phone" → PhoneEntryScreen renders with brand voice ("drop your digits 📱", "we'll slide into your texts with a code", "no cap, we don't spam ❤️"), country code default `+1`, encrypted-and-never-shared trust note. Screenshot: `b26_05_phone_entry.png`.
- Type `5555550100` → digits visible in input field. Screenshot: `b26_07_phone_typed2.png`.
- Tap "send code" → OTPScreen renders with "prove you're real 👀" header, masked phone `+15•••••00`, 6-digit input boxes, 27s resend timer, "we promise the code is on its way" subtitle. Screenshot: `b26_09_otp_screen.png`.
- OTP submission via ADB blocked by single-digit-input auto-advance timing. Filed as **D-26-03 (P3)** — OTP single-digit input boxes interact badly with `adb shell input text` (keystrokes dropped during box transitions). Doesn't impact human users.

#### Layer 3 — Design QA on 3 traversed screens

**Landing screen** (`b26_01_coldstart.png`):
- Wordmark "amovi." (lowercase + period, italic, amber #FFAA00) — correct per batch_10 D-06-01
- Dark theme (void / surface tokens)
- 4 auth pill buttons with brand-correct iconography (phone icon, Google G, Instagram gradient, Snapchat ghost)
- Touch targets visually ≥48dp
- ZERO DevBypass UI (M-01 closure held)

**PhoneEntryScreen** (`b26_05_phone_entry.png`):
- Wordmark consistent
- Brand voice copy ("drop your digits 📱", "no cap, we don't spam ❤️")
- amber-bordered input on focus
- "encrypted & never shared" trust note with shield icon
- Carousel progress dots (1 of 3 active)
- Marketing-tag carousel below ("memes > pickup lines | food compatibility > zodiac signs | first 100 testers · McKinney/Dallas")

**OTPScreen** (`b26_09_otp_screen.png`):
- Wordmark consistent
- Header "prove you're real 👀"
- Phone masking `+15•••••00` (correct PII redaction)
- 6 input boxes with amber focus indicator
- Countdown timer with clock icon
- "didn't get it? resend" surfaces after countdown expires
- "device verified" green-shield badge for previously-verified device

**Verdict: PARTIAL PASS** — rc14 successfully installed; Layer 1 + Layer 3 partial validation completed on device; Layer 2 + Layer 4-8 deferred to next session with rc15 (env-var-complete build) for full GIF API testing. Filed D-26-03 as new finding.

## Layered nuclear test results

### Layer 1 (SMOKE)
**Status:** Not executed — rc14 build in upload phase; the currently-installed rc9 (versionName=1.1.0-rc9, lastUpdate 2026-05-10 06:37) does not contain the Phase 3.5 GIPHY migration, so smoke testing on rc9 doesn't validate the batch's actual code change.

### Layer 2-8
**Status:** Pending Phase 4 build completion. Test plan composed; awaits APK install on RFCY81H4F9Z.

### Static checks (executed)
- ESLint clean on all touched files
- `@babel/parser` syntactic validation OK
- `grep -rn "tenor\|TENOR" src/` returns 0 hits (migration completeness)
- Firestore spot-readback confirmed seed shape

## ≥13 prospective + retrospective failure modes considered

1. **Gemini API daily quota** — RECURRED: gemini-2.5-flash hit 20 RPD free-tier cap mid-Phase 1.5; mitigated by switch to gemini-2.5-flash-lite then gemini-3.1-flash-lite for tail.
2. **Gemini-2.5-flash thinking-mode token consumption** — OCCURRED: probe bio came back at 225 chars vs 1500 target; root cause was thinking-mode burning output tokens; mitigated by `thinkingConfig.thinkingBudget: 0`.
3. **Field-name drift between PERSONAS.json schema and matchService.js** — OCCURRED: `memeMarathonPattern` vs `memeReactions`; if undetected, brain memeScore stays at 50 default. Caught during Phase 3 prep; fixed in seed (IFF-26-01) before commit.
4. **Free-tier Gemini 2.0 Flash limit:0** — OCCURRED: spec said "use 2.0 Flash" but project free tier returned `limit: 0`. Switched to 2.5/3.1 lines.
5. **Dummy-or-real Auth user collision** — RISK: writing to `+15555550100..0109` could overwrite real user docs. Mitigation: all docs marked `__batch: "batch_26"` for safe selective teardown; CLAUDE.md §12 confirmed phones are dedicated test slots.
6. **Photo source rate-limit / API key cost** — AVOIDED: rejected Unsplash API in favor of `picsum.photos/seed/...` deterministic URLs (no key, no rate limit).
7. **Pre-commit secret scanner false-positive on documentation** — OCCURRED: BACKLOG_v2 update cited the old Tenor key literally; pre-commit hook caught it; redacted to "AIza…CYQ" form.
8. **EAS build env var pipeline gap** — OCCURRED: `EXPO_PUBLIC_GIPHY_API_KEY` was missing from EAS env when rc14 was kicked; added mid-upload. Build retry needed for env to propagate.
9. **GIPHY TOS attribution gap** — AVOIDED: badge added in picker header; cited in PR + IN_FLIGHT_FIXES.
10. **GIPHY response schema mismatch** — MITIGATED: response parser refactored from `media_formats.tinygif.url` to `images.fixed_height.url` per GIPHY API v1 docs.
11. **Firestore docs ineligible due to lookingFor filter on test data** — DOCUMENTED: validate_brain.js applies eligibility filter; PERSONAS.json predictedMatches were composed without it; some "predicted" pairs are unreachable. Listed as a Phase 3 limitation.
12. **EAS upload size = 1.2GB blocking build completion** — OCCURRED: working tree has heavy untracked .png/.txt/.log artifacts; .easignore covered most but not all. Phase 4 deferred to next session.
13. **rc9 currently on device lacks GIPHY change** — RISK: tempting to "test" GIF on rc9, but that exercises old Tenor code, not the actual Phase 3.5 migration. Resisted.
14. **Brain validation thresholds (≥2 overlap = "hit") may be too strict** — DOCUMENTED: alternative metric (≥1 overlap = partial) shows 60% top-3 partial alignment; finer-grained threshold tuning is in scope for future batch.
15. **Service account JSON in working tree** — VERIFIED: `firebase-service-account.json` exists at repo root, not committed (`.gitignore` covers it; CLAUDE.md §14 strict prohibition).
16. **Synthetic uids may collide with real onboarding uids** — RISK: `b26_test_persona_NN` is unlikely format collision but checked for uniqueness via `__batch=batch_26` filter in teardown.
17. **Gemini deterministic vs probabilistic regen** — DOCUMENTED: bio gen used 3-attempt retry with rubric-driven revise prompt; Deep Dive used responseMimeType=json for parse-stability.
18. **Time-to-quota-reset UX** — OCCURRED: Phase 1.5 burned through gemini-2.5-flash 20 RPD cap in ~16 calls; switched models rather than waiting for daily reset.

## Anti-fabrication discipline log (per CLAUDE.md §2)

- All commits independently verified via `git log` and `git show --stat` (e.g., HEAD `2f916a29` content fence shows the migration delta).
- All Firestore writes verified by Admin SDK readback (V3 spot-checks).
- All bio scores generated by deterministic algorithm (`gen_bios.js#scoreBio`), not LLM-judged.
- BRAIN_VALIDATION_REPORT.md and MATCH_MATRIX.json contain raw computed values, not narrative summaries.
- No hollow V-verdicts: every claim above ties to a specific path/line/json key/SHA.

## Discoveries filed (D-26-NN)

- **D-26-01 (P2):** Brain algorithm signal-set vs predictedMatches composition basis mismatch. Top-3 hit 16%. Routed to Brain in IN_FLIGHT_FIXES + this ledger.
- **D-26-02 (P3):** Algorithm has no dealbreaker / anti-pattern modeling for false-positive bottom-3 suppression. Routed.

## In-flight fixes landed (IFF-26-NN)

- **IFF-26-01:** Phase 2 — added canonical `memeReactions` field alongside `memeMarathonPattern` in seed_to_firestore.js#buildUserDoc so brain scoring works (matchService.js reads `memeReactions`). +6 LoC, 1 file. Tribunal skipped (data-write fix only).

## Founder follow-up actions (RED_BLOCK candidates)

- **Revoke old Tenor key:** the AIza…CYQ value previously at `src/constants/config.js:46` is in pre-batch_26 git history of the private repo + may be in the public companion repo. Must rotate on Google Cloud Console even though source no longer ships it.
- **Phase 4 device test resumption:** once rc14 EAS build finishes uploading + queuing + completing (~25-30 min more), CC needs to resume `adb install` + L1-L8 layered nuclear test. The build SHA + APK URL can be retrieved via `eas build:list --limit=1`.

---

## ============================================================
## OVERNIGHT RESUMPTION 2026-05-11 — Phase 4.5/4.6 → 5 → 6
## ============================================================

**Resume HEAD:** `e9...` (post Phase 3.5 commit `2f916a29`)
**Closeout HEAD:** `3dcad7fb` (pre-rc20-validation, awaiting device confirm)
**Time window:** 2026-05-11 ~01:15 → ~02:15 CDT

### Headline outcome — D-26-07 RESOLVED

D-26-07 (chat-tab-empty regression) is NOT a query/data/auth/cache/rules bug. It is a **viewport misdiagnosis**: the Theo conversation row WAS rendering correctly all along, just below the initial scroll fold of the chat tab, hidden behind the NEW VIBES empty state vertical bloat ("no moves yet" → "head home and start swiping" → "START SWIPING" button → "want ghost AI" link). Four prior CC sessions in this batch concluded "empty" without scrolling, leading to IFF-26-03/-04/-05 against a non-existent root cause. The DIAG-26-07 instrumentation (IFF-26-04) + the IFF-26-07 logger context fix finally surfaced the actual data — `matchesSize=1 convosSize=1 blockedSize=0` — proving queries work. Visual inspection after scrolling confirmed the row exists. Fix is 1-line guard suppressing the empty state when chats exist (IFF-26-10).

### V-verdicts (overnight phase)

**V13 — IFF-26-06 OTP code corrected (commit `aa75a4e9`)**
- BEFORE CLAUDE.md line 81: `OTP 555555` (Firebase docs example).
- AFTER (founder-verified actual codes): `OTP 123456` (with full slot mapping).
- Validation: rc18 OTP `123456` → screenshot `b26_102_rc18_post_otp_correct.png` shows "checking..." + notification permission dialog (post-auth) → rc18 reached Home/Swipe as Maya for the first time in this run.
- Verdict: PASS.

**V14 — IFF-26-07 loggerServiceV2 context fix (commit `7f65390f`)**
- BEFORE: `context: data ? { detail: String(data) } : {}` → literal `"[object Object]"` for any plain object.
- AFTER: `_safeSerialize(data)` helper using `JSON.parse(JSON.stringify(data))`.
- Validation: rc19 /errors readback (`scripts/persona_test/post_rc19_diag.js`) shows real values: `{"uid":"vqxr1eu9cNU3Awfe1DIYGfBORQn2","size":1}`.
- Verdict: PASS.

**V15 — IFF-26-08 HomeScreen brand mark + lowercase buttons (commit `e85bef9c`)**
- BEFORE: `<Text style={styles.logo}>Amovi</Text>` × 2 in HomeScreen; `START SWIPING` in MatchesScreen empty mini; `START SWIPING PEOPLE` in VibeResultScreen CTA.
- AFTER: `<AmoviMark size={28} />` × 2 (canonical italic Plus Jakarta ExtraBold + period); `start swiping` lowercase × 2.
- Validation: rc20 device pending.
- Verdict: PASS (code), validation pending.

**V16 — IFF-26-09 VibeLab modules lowercase + Search apply button (commit `d2a4369c`)**
- BEFORE: `FoodPicker / MemeCalibration / QuickFire` in modules array (AppNavigator.js); `textTransform: "uppercase"` on SearchScreen `applyButtonText`.
- AFTER: `food picker / meme calibration / quick fire`; uppercase transform removed.
- Validation: rc20 device pending.
- Verdict: PASS (code), validation pending.

**V17 — IFF-26-10 D-26-07 root cause fix + DIAG-26-07 trace cleanup (commit `3dcad7fb`)**
- Evidence chain:
  - DIAG-26-07 trace dump after rc19: `matchesSize=1 convosSize=1 blockedSize=0` with canonical uid `vqxr1eu9cNU3Awfe1DIYGfBORQn2`.
  - Visual evidence: screenshot `b26_125_rc19_chat_state.png` (initial chat viewport — empty above fold) vs screenshot `b26_126_rc19_chat_scrolled.png` (after scroll — Theo row rendered with `hey 👋` + 3h + unread badge 1).
- Fix: 1 line in MatchesScreen.js empty-state guard adds clause `!(activeTab === "active" && filteredConversations.length > 0)`.
- Cleanup: 6 DIAG-26-07 log.error calls + unused log import removed.
- Validation: rc20 device pending.
- Verdict: PASS (code), validation pending.

### Lifecycle / AI / adversarial PASS (rc18 device evidence)

| Layer | Test | Verdict | Evidence |
|---|---|---|---|
| L1 #5 | Main nav 5 tabs render | PASS | screenshots b26_103/115/117/104/105 |
| L1 #8 | Force-stop + relaunch (persistent session) | PASS | b26_109 |
| L1 #9 | Background + foreground 30s | PASS | b26_106 + b26_107 (identical state) |
| L1 #10 | Network disconnect (wifi + data off) | PASS | b26_112 (Gen Z offline banner) |
| L1 #11 | Logcat aggregate (71k lines) | PASS | 0 AndroidRuntime FATAL for amovi |
| L8 | 10x rapid chat tab tap | PASS | No crash; /errors recorded 10 entries |
| Phase 5 AI | amoviAI profile coach | PASS | b26_114 (3 real tips rendered) |

### Hypothesis disambiguation matrix (D-26-07)

13 hypotheses chased and DISPROVEN before viewport explanation found. Per §3 of this ledger overnight section.

### Additional D-NN-NN filings (overnight)

- **D-26-07 (RESOLVED via IFF-26-10):** viewport misdiagnosis on chat tab — NEW VIBES empty state pushed CHATS below fold. Closed by IFF-26-10. Rec: future device tests must include explicit scroll-down on any screen showing an empty section header.
- **D-26-08 (P3 OPEN, deferred):** loggerServiceV2 `_safeSerialize` regression risk — JSON-roundtrip drops `undefined`, functions, BigInt; cycles caught with fallback. For 99% of log.error contexts this is fine, but edge cases (Maps, Sets, Promises) collapse to empty objects. Not blocking; track as low-priority refinement.
- **D-26-09 (P3 OPEN, deferred):** /errors collection compound query (`tag in [...] orderBy createdAt`) requires a missing index. Diagnostic-only impact (admin SDK from CC); worked around with client-side filter. Fix: add the index to firestore.indexes.json + deploy, OR keep client-side filter.

### IFFs landed overnight

- **IFF-26-06** OTP code fix (CLAUDE.md doc) — `aa75a4e9`
- **IFF-26-07** Logger context serialization — `7f65390f`
- **IFF-26-08** HomeScreen brand mark + button case — `e85bef9c`
- **IFF-26-09** VibeLab module + Search apply button — `d2a4369c`
- **IFF-26-10** D-26-07 root cause + diag cleanup — `3dcad7fb`

### Anti-fabrication discipline log (overnight)

- All claims tied to specific commit SHAs (verified via `git log --oneline`) or screenshot paths (verified via Read tool on PNG content).
- /errors trace evidence pulled via `scripts/persona_test/post_rc19_diag.js` with stdout captured in this ledger verbatim.
- Visual claim "Theo row rendered after scroll" backed by screenshot `b26_126_rc19_chat_scrolled.png` viewed directly in this session.
- No hollow V-verdicts: every overnight V-verdict references file:line OR screenshot path OR /errors timestamp+context.

### Founder follow-up after rc20 validates

- After rc20 device validation confirms IFF-26-10/08/09 render correctly:
  - Update OPEN_ITEMS_TRACKER.md — flip D-26-07 to CLOSED_GREEN.
  - Update BATCH_DISCOVERY_LOG.md — add overnight session entries.
  - Update CUMULATIVE_STATE.md, SHIP_GATE.md, DAY_FOUNDER_DIGEST.md, INNOVATION_LOG.md.
  - Auto-Bridge publish to `chandukcs9/lolbite-app-evidence` companion repo per CLAUDE.md §9.
  - Update WAKE_BRIEF.md → "READY FOR /ultrareview".
  - HALT for founder.


---

## §7 RC20 DEVICE VALIDATION (2026-05-11 ~02:50)

All 4 post-rc19 IFFs validated on rc20 (HEAD `3dcad7fb`, build ID `b1d527eb-abbd-40cb-a606-82427a4b631f`, APK 166MB).

| IFF | Validation evidence | Verdict |
|---|---|---|
| IFF-26-08 HomeScreen brand mark | screenshots/b26_130_rc20_post_auth.png shows top header "amovi." italic-lowercase-period (was "Amovi" Title Case on rc18) | PASS |
| IFF-26-09 Search apply button | screenshots/b26_132_rc20_search_validation.png shows "lock it in" lowercase (was "LOCK IT IN" uppercase via textTransform on rc18) | PASS |
| IFF-26-09 VibeLab modules | screenshots/b26_133_rc20_vibelab_validation.png shows "food picker / meme calibration / deep dive / quick fire" lowercase (was CamelCase on rc18) | PASS |
| IFF-26-10 D-26-07 viewport fix | screenshots/b26_131_rc20_chat_tab_validation.png shows Theo conv row ABOVE the fold immediately after chat-tab tap ("theo / hey 👋 / 4h / unread 1") — NEW VIBES empty state correctly suppressed because CHATS has entries | PASS |

**D-26-07 FULLY RESOLVED.** Chat tab now displays conversations at the top of the active tab view, no scrolling required to see them. The 600px of vertical bloat from NEW VIBES empty state ("no moves yet" + "START SWIPING" + "want ghost AI") that previously hid them is suppressed when CHATS has entries.

