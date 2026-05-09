# §4.5 5-Question Gap Scan — batch_21

## Q1: What did Brain assume that turned out wrong?
**Brain assumed T2-12 EAS was a fail.** Plan agent step 1 found b20 output line 20 already showed `✅ T2-12: appVersionSource = remote` and eas.json line 4 already had the correct value. M-03 was effectively already-shipped — possibly during batch_07 EAS reset when the workflow was first stabilized. Plan agent saved the batch from a no-op edit that would have introduced JSON drift.
- **D-21-01 (DOC, S-3)**: BACKLOG_PHASE_1.md M-03 entry should be marked CLOSED_GREEN with reference to `eas.json:4`. Currently M-03 is described as still-open.

## Q2: What scope grew during execution that wasn't in the spec?
**None.** Plan agent's 16 steps mapped exactly to the 9 spec items (8 fixes + 1 verify-only + 4 process steps). Zero unplanned edits. `git diff --stat`: 4 files, +11/-11.

## Q3: What did the Plan agent surface that the spec didn't?
**Two findings:**
1. **Item 1 was stale** — Plan caught T2-12 already passing in b20 (above).
2. **T3-6 hybrid approach** — spec offered range loosening 3..6 OR flow-count refinement; Plan picked filename-scope to 3 named files which is more precise than range and FAIL-fast on rename regressions.
3. **Adversarial gate-correctness sanity check** — Plan added P-15 "diff against b20 for silent PASS→FAIL transitions" to defend against grep refinements over-correcting. Caught 0 silent regressions; but the discipline now lives in the playbook.
- **D-21-02 (META, S-3)**: Adopt P-15 diff-discipline as standing rule for any future tribunal grep refinement batch.

## Q4: What did the post-fix tribunal show that wasn't visible before?
**Tribunal is now a trustworthy gate.** Pre-batch: 8 FAILs of which 7 were false-positives (grep pattern issues) + 1 was real (D-19-04). Post-batch: 28/28 PASS in 1.86s, every FAIL would be a real violation. From batch_22 forward, tribunal can be invoked as a hard gate (exit code matters, FAILs are blocking) rather than a baseline (exit code can be 1, FAILs route as findings). This changes the workflow contract.
- **No D entry** — this is the intended outcome.

## Q5: Is there a tribunal check that PASSES today but would silently miss a regression class?
**Two candidates worth observing:**
1. **T1-5 comment exclusion (`grep -v "//\|/\*"`)** — if a developer writes `// looking for` AND a real string `"looking for"` on the same line, the line is excluded. Risk is minor (one-line two-purpose lines are unusual).
2. **T2-1 anchor-based detection** — if someone writes `import {X, TYPOGRAPHY, Y} from "./theme"` on a continuation line (e.g., `^  TYPOGRAPHY,$` after a multi-line import), the regex `import.*TYPOGRAPHY` may not match because there's no `import` on that line. ESLint catches this independently, so blast radius is limited.
- **D-21-03 (P3 OBS)**: Both edge cases noted; tribunal-refinement-v2 META if needed in future.

## Triage summary for Brain
- **D-21-01** (DOC, S-3): M-03 BACKLOG entry mark CLOSED_GREEN
- **D-21-02** (META, S-3): Adopt P-15 diff-discipline standing rule
- **D-21-03** (P3 OBS): T1-5 + T2-1 multi-line edge cases
