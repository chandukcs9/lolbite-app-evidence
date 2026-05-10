# §4.5 5-Question Gap Scan — batch_24

## Q1: What did Brain assume that turned out wrong?
**Two assumptions corrected during execution:**
1. **Spec assumed `python3` would be on PATH**. Windows Git-Bash on this host has `python` (3.14.2 at /c/.../Python314) but no `python3` symlink. Pre-flight created `/c/Users/chand/bin/python3` shim wrapping `python.exe`.
2. **Spec didn't mention MSYS path mangling**. Default Git-Bash behavior converts `/sdcard/screen.png` to `C:/Program Files/Git/sdcard/screen.png` on adb commands — silently breaks screencap + pull. Fix: `export MSYS_NO_PATHCONV=1` at automation.sh top.
- **D-24-04 (P3 DOC)**: CLAUDE.md §11 (Device Specifics) should add: "Windows Git-Bash adb scripts MUST set MSYS_NO_PATHCONV=1 OR the path conversion mangles `/sdcard/*` device paths."

## Q2: What scope grew during execution that wasn't in the spec?
**Two minor additions:**
1. python3 shim creation (necessary for find_by_text/find_by_desc helpers).
2. MSYS_NO_PATHCONV export in automation.sh.
Neither expanded charter scope; both are framework-level fixes captured before kick-off.
- **No D entry** — framework hardening within scope.

## Q3: What did the Plan agent surface that the spec didn't?
**Plan agent ran in read-only mode** (system prompt prohibited Write tool) and produced full charter contents inline. CC wrote all 17 files. Plan agent did NOT add architectural risks beyond following the reference pattern — task was mechanical extension, not new design.
- **No D entry** — appropriate Plan-agent role for templated work.

## Q4: What does the live smoke check NOT verify?
**Charters 11-25 are unproven against real UI.** Smoke check only ran through ~6 charters before batch close. Charters 11-25 may have similar text-mismatch issues to charters 03/04/06 (MISS warnings on UI text that doesn't exist). These are NOT framework failures; they're charter-content findings batch_25 will triage by reviewing screenshots + UI dumps.

Specifically unverified:
- Multi-user flow (charter 25) — depends on D-22-02 OTP whitelist for +15555550102 (founder action pending; charter targets +15555550101 which IS whitelisted, so should work but unverified)
- Vibe Lab calibration screens (16-19) — actual entry text unknown
- Stress rapid (24) — concurrency behavior under 30 taps in 10s untested
- **D-24-05 (P3 OBS)**: batch_25 must triage charter results not just count PASS/FAIL — many "FAILs" will be "framework worked, app surface differs from charter heuristic guess" and need text-update fixes, not framework fixes.

## Q5: Is the framework architecture flexible enough for batch_25's iteration?
**Yes, with one caveat.** Charters are independent files; updating one doesn't require runner changes. tap_text/find_by_text helpers are reusable. The caveat: charters lack a recovery hook. If charter N corrupts state (e.g., logs out unexpectedly), charters N+1, N+2, etc. inherit that state and may fail cascadingly. Runner does `force-stop` on TIMEOUT only, not on FAIL. Future improvement: add `restart_fresh` between certain charter groups (e.g., between auth-state-dependent and auth-independent groups).
- **D-24-06 (P3 OBS)**: Add inter-charter state-recovery hook in runner.sh for batch_26+ if cascading failures observed in batch_25 results.

## Triage summary for Brain
- D-24-01 (P3 BUG): charter 02 fatal=$grep-c quoting fix (cosmetic)
- D-24-02 (P2 GAP): bottom-nav tab text-label vs icon-label discovery
- D-24-03 (P3 OBS): runner.log duplicate-entry deduplication
- D-24-04 (P3 DOC): CLAUDE.md §11 add MSYS_NO_PATHCONV note
- D-24-05 (P3 OBS): batch_25 triage discipline (framework vs charter-content failures)
- D-24-06 (P3 OBS): inter-charter state-recovery hook (defer to batch_26+)
