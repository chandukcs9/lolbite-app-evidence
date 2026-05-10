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

### V6 — Phase 4: rc14 build initiated but device tests deferred

**Evidence:**
- EAS build kicked at 2026-05-10T~14:55 with `eas build --profile preview --platform android --non-interactive`
- Build log: `round-trips/batch_26/PHASE_4_RC14_BUILD.log`
- Last log line: "Uploading to EAS Build (0 / 1.2 GB)" — local upload phase, slow on this connection
- EAS env updated with new vars (created via `eas env:create`):
  - `EXPO_PUBLIC_GIPHY_API_KEY` → preview env (sensitive visibility)
  - `EXPO_PUBLIC_GEMINI_API_KEY` → preview env (sensitive visibility)
- Status: in-flight upload; build will queue once upload completes

**Verdict: DEFERRED — rc14 build artifact not yet available; Phase 4 device tests to resume in next session once APK lands**

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
