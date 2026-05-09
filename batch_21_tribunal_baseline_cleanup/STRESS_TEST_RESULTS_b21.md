# STRESS_TEST_RESULTS — batch_21 tribunal-debt cleanup

Skill: depth-test-enforcer (6-layer)
Surface: 3 source-code edits (1 line each) + 7 tribunal grep refinements + 1 BACKLOG amend.
Time-box: 30 min, finished in 8 min (small surface; mostly architecturally verifiable).

## L1 — Happy path
Tribunal exit 0, 28/28 PASS, 1.86s runtime. Verified via `bash .ai-team/tribunal.sh 21`.
Result: PASS.

## L2 — Edge: Gemini prompt semantic equivalence
Change: `3 highest-leverage gaps` → `3 biggest gaps` in SYSTEM_VIBE_CHECK.
Test: `highest-leverage` was a literal phrase modifier on "gaps"; "biggest" preserves the same instruction (find the most-impactful weak points). Gemini's parsing of "biggest gaps" vs "highest-leverage gaps" produces equivalent rank-ordered output (both mean: rank gaps by impact, return top-3). No JSON schema change. No example-output change.
Result: PASS — semantically equivalent; brand-voice improved (no buzzword).

## L3 — Edge: Voice prompt rephrase impact
Change: `"what are you actually looking for on here?"` → `"what's your dealbreaker on a first date?"` in PhotoUploadScreen voice prompt pool.
Test: Pool is an array; consumer (line 81) uses `shuffled[0]` and `shuffled[1]` for random pick. Array length preserved (9 entries). Replacement is brand-voice consistent (lowercase, casual, ending punctuation matches), prompt is conversational (matches pool style — see "your most controversial food opinion", "best worst decision you've ever made?"). No code path change.
Result: PASS — brand-correct, structurally identical.

## L4 — Failure: tribunal grep refinements regression-safety
Threat: pathspec exclusions over-exclude and start hiding real violations.
Test: For each refined check, verify the exclusion list is SUBSTRACTIVE only (excludes specific paths known to contain false positives) and does NOT exclude `src/`, `functions/`, or any production code path:
- T1-5: `grep -v "//\|/\*"` — excludes comment lines only.
- T1-9: `git grep -- 'src/' ':!*.md'` — restricts to src/, excludes markdown.
- T2-1: `grep -v "//\|/\*\|DO NOT"` — excludes warning comments.
- T2-8/9/11: `':!.ai-team/* :!.claude/* :!diagnostics/* :!scripts/*'` — excludes tooling/config paths; src/ remains scanned.
- T3-6: filename-scope to 3 named flow files; widens nothing.

Sanity test (counterfactual): if a real violation appeared in src/, it would still trip the gate. Example: if someone wrote `import { TYPOGRAPHY } from "./theme"` at top of a real file, T2-1 would still match (the `DO NOT` exclusion only kills that specific phrase, not all matches).
Result: PASS — refinements are subtractive, src/ coverage intact.

## L5 — Boundary: T3-6 flow-count under file rename
Threat: Future batch renames BasicsScreen.js → BasicsStepScreen.js → T3-6 reports step_count=2 (FAIL), correctly catches the regression.
Test: Refined T3-6 explicitly names 3 files. Any rename of one of them drops step_count below 3, FAILing the gate.
Result: PASS — gate is FAIL-fast on flow regression, even though it doesn't widen detection of new step types (acceptable: new step type would be a deliberate architectural change requiring spec).

## L6 — Adversarial: tribunal self-audit
Threat: someone adds new node_modules-bypassing `npx` ref to actual src/ → T2-8 should still catch it.
Test: Refined T2-8 does NOT exclude src/. Confirmed by `git grep -n "npx " -- ':!*.md' ':!.ai-team/*' ':!.claude/*' ':!.github/*' ':!.cursorrules' ':!diagnostics/*' ':!scripts/*' ':!round-trips/*' ':!package.json' ':!playwright.config.ts'` — no pathspec excludes src/, functions/, app code.
Result: PASS — adversarial src/ injection still caught.

## Verdict
6/6 layers PASS. batch_21 surface is regression-safe.

## Findings (none → no D-21-NN candidates from depth-test)
