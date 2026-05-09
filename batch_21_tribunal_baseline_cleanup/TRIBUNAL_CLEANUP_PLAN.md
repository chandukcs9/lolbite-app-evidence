# TRIBUNAL_CLEANUP_PLAN — batch_21 Plan agent verbatim output

Composed: 2026-05-08
Plan agent input: 9 tribunal-debt items
Plan agent verdict: **Item 1 (T2-12 EAS) is STALE — already PASSES per b20 output. eas.json line 4 already has `"appVersionSource": "remote"`.** Effective scope: 8 items + 1 verify-only.

---

## ARCHITECTURAL RISKS

- **eas.json T2-12 already passes per b20 output** (line 20: `✅ T2-12: appVersionSource = remote`). The brain spec lists T2-12 as item 1, but file already has `"appVersionSource": "remote"` at line 4. Plan must verify-only, not edit. If we edit anyway we risk introducing JSON drift. Founder context: this means the actual fail count is 8 (T1-1, T1-5, T1-9, T2-1, T2-8, T2-9, T2-11, T3-6) — matches b20 output. Item 1 in spec is stale.
- **T1-1 source location surprise**: the only buzzword match in `src/` is the literal token `leverage` inside `highest-leverage` at `aiProfileService.js:62`. The full Gemini prompt is 40+ lines; changing wording risks degrading model output quality. Replacement must be semantically equivalent ("biggest" / "most-impactful" / "top") and not change the prompt's behavior.
- **PhotoUploadScreen "looking for" is in a voice-prompt pool** (line 54), not a header/CTA. Removing breaks the random-pick logic if the array length isn't preserved (line 81 picks `shuffled[0]` and `shuffled[1]` — fine if length stays ≥2). Replacement is safer than deletion. Spec says "rephrase, relocate, or remove" — rephrase is lowest risk.
- **Tribunal grep refinements use bash quoting + `git grep` pathspecs** — a malformed pathspec silently matches nothing (false PASS) instead of erroring. Each refined check needs a sanity test with deliberately-injected match that proves the check still trips when it should. Without that, we ship a "working" tribunal that no longer detects regressions.
- **T1-5/T1-7/T1-8/T1-10 currently use `grep -v "//\|comment"`** — that pattern is a single regex with `\|` alternation; under busybox/macOS BSD grep this means literal `\|`. On git-bash/MSYS GNU grep it's alternation. Branch already runs on Windows git-bash so it works, but switching pattern style mid-script is a foot-gun — keep pattern style consistent across siblings.
- **`grep -rn` vs `git grep`**: T1-5/T1-9/T1-1 use `grep -rn` (recursive disk scan, no pathspec syntax); T2-8/9/11 use `git grep` (tracked files only, supports `:!` pathspecs). Pathspec-based exclusion only works with `git grep`. Refining T1-9 to add `:!*.md` requires switching it from `grep -rn` to `git grep`, which changes scoping (untracked files no longer matched). Acceptable for this repo (working tree is clean for source paths).
- **T3-6 widening to 3..6**: range loosening masks future regressions. Better fix: filter to actual flow steps by scoping to `Basics|Preferences|PhotoUpload` filenames. Plan picks a hybrid: scope by filename pattern AND keep a wider sanity bound.
- **Untracked working tree (huge list of `.txt`, `.png`, `project_notes/`)**: `git grep` ignores these (good), but `grep -rn` walks them. T1-1/T1-5/T1-9 use `grep -rn` against `src/` only, so untracked top-level junk doesn't affect them. Confirmed safe.
- **No tribunal regression-evidence captured**: spec mandates "Capture full output" post-run, but doesn't mandate diff against b20. If a previously-passing check (e.g., T2-3 Warm Ember) silently regresses while we're patching T1-5, we won't notice. Add a diff step.
- **AUDIT_RESULTS.md is at `src/AUDIT_RESULTS.md`** (not repo root). T1-9 currently `grep -rn` over `src/` so the `:!*.md` pathspec won't apply unless we switch to `git grep -- 'src/' ':!*.md'`. Pathspec scoping interaction matters.
- **D-20-01 BACKLOG amend** is doc-only and not gated by any tribunal check. Not adding tribunal coverage means the amend is unverifiable except by `grep -F` against the new sentence. Acceptance must be string-match on the exact appended phrase.

## STEPS

### B21-P-1 — Pre-flight: capture current state of all 9 target files
ACTION: Run `git rev-parse HEAD` → expect `753123bf...`. Verify all 9 paths exist. Record b20 fail set: T1-1, T1-5, T1-9, T2-1, T2-8, T2-9, T2-11, T3-6.
ACCEPTANCE: All 9 paths exist; HEAD prints `753123bf...`; captured fail set matches b20.
ON_FAILURE: HALT, file D-21-01.

### B21-P-2 — Pre-flight: capture exact b20 baseline grep counts
ACTION: Re-run each currently-failing tribunal check; record numeric count for each.
ACCEPTANCE: All 8 counts >0. Counts match b20 output where applicable (T1-5=2, T2-8=39, T2-9=5, T2-11=8, T3-6=6, T1-1=1, T1-9=1, T2-1=1).
ON_FAILURE: HALT, file D-21-02.

### B21-P-3 — Verify eas.json T2-12 already PASSES (item 1 is stale)
ACTION: `grep -q '"appVersionSource".*"remote"' eas.json && echo PASS || echo FAIL`. No edit.
ACCEPTANCE: Prints `PASS`. NO file modification.
ON_FAILURE: edit eas.json (keep cli object), validate JSON.

### B21-P-4 — Fix T1-1: replace `highest-leverage` in aiProfileService.js
ACTION: Replace `3 highest-leverage gaps` → `3 biggest gaps`. No other edits.
ACCEPTANCE: T1-1 grep returns 0.
ON_FAILURE: enumerate other matches, HALT if >3.

### B21-P-5 — Fix D-19-04: rephrase "looking for" in PhotoUploadScreen voice prompt
ACTION: Replace `"what are you actually looking for on here?"` → `"what's your dealbreaker on a first date?"`. No other edits.
ACCEPTANCE: `grep -n "looking for" src/screens/onboarding/PhotoUploadScreen.js` = 0.
ON_FAILURE: byte-level Read + retry; alt-copy `"the most underrated red flag is..."`.

### B21-P-6 — Fix D-19-05: refine T1-5 grep to exclude comment lines
ACTION: Tribunal line 36: append ` | grep -v "//\|/\*"` before `wc -l`.
ACCEPTANCE: T1-5 returns 0.
ON_FAILURE: enumerate residual.

### B21-P-7 — Fix D-19-06: refine T1-9 grep with `:!*.md` symmetry + switch to `git grep`
ACTION: Tribunal line 52: replace `grep -rn` with `git grep -n` + pathspec `'src/' ':!*.md'`.
ACCEPTANCE: T1-9 returns 0.
ON_FAILURE: fall back to `--include` filtering.

### B21-P-8 — Fix D-19-07: refine T2-1 grep to exclude warning comments
ACTION: Tribunal line 65: anchor `^import.*TYPOGRAPHY\|^.*from.*TYPOGRAPHY` + exclude `DO NOT`.
ACCEPTANCE: T2-1 returns 0.
ON_FAILURE: append `crash-prone` exclusion.

### B21-P-9 — Fix D-19-08: add pathspec exclusions to T2-8 (npx)
ACTION: Tribunal line 92: add `:!.ai-team/* :!.claude/* :!.github/* :!.cursorrules :!diagnostics/* :!scripts/* :!round-trips/* :!package.json :!playwright.config.ts`.
ACCEPTANCE: T2-8 returns 0.
ON_FAILURE: enumerate residual; add specific pathspec.

### B21-P-10 — Fix D-19-08: add pathspec exclusions to T2-9 (EXPO_OFFLINE)
ACTION: Tribunal line 96: same exclusion set as P-9 minus package.json/playwright.
ACCEPTANCE: T2-9 returns 0.
ON_FAILURE: residual src/ match → NEW bug → HALT.

### B21-P-11 — Fix D-19-08: add pathspec exclusions to T2-11 (DevBypassScreen)
ACTION: Tribunal line 104: add `:!.ai-team/* :!.claude/* :!diagnostics/* :!scripts/*`.
ACCEPTANCE: T2-11 returns 0.
ON_FAILURE: residual src/ match → M-01 regression → HALT.

### B21-P-12 — Fix D-19-09: refine T3-6 to count flow-step screens by name
ACTION: Tribunal line 141: scope `find` to `BasicsScreen.js`, `PreferencesScreen.js`, `PhotoUploadScreen.js` only.
ACCEPTANCE: step_count=3, T3-6 PASS.
ON_FAILURE: missing flow file → real regression → HALT.

### B21-P-13 — Fix D-20-01: amend M-02 description in BACKLOG_PHASE_1.md
ACTION: BACKLOG line 23 (M-02 strategic intent), append: ` Slide In tab routes to MatchesScreen.js (conversation list); ChatScreen.js is the conversation-detail view, reachable only via row tap. chatService DEV_MODE branches scoped to dev_*/mock_*/dev_conv_ prefixes only.`
ACCEPTANCE: `grep -F "Slide In tab routes to MatchesScreen.js"` = 1 match.
ON_FAILURE: smart-quote substitution → byte-level Read + retry.

### B21-P-14 — Run full tribunal post-fix
ACTION: `bash .ai-team/tribunal.sh 21 | tee round-trips/b21_tribunal_output.txt`.
ACCEPTANCE: Exit 0. `TOTAL: 28 / 28 (0 FAIL)`. <60s.
ON_FAILURE: per-FAIL D-21-NN; NEW regression → revert.

### B21-P-15 — Diff-check for silent regressions of previously-passing checks
ACTION: `diff <(grep "✅\|❌" b20) <(grep "✅\|❌" b21)` — only ❌→✅ transitions allowed.
ACCEPTANCE: Zero PASS→FAIL transitions.
ON_FAILURE: revert culprit step.

### B21-P-16 — Capture per-step git diff stat for evidence
ACTION: `git diff --stat HEAD`.
ACCEPTANCE: 4-5 files modified; total churn <20 lines.
ON_FAILURE: revert excess.
