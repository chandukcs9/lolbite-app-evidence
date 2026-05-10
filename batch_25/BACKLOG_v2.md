# BACKLOG_v2 — frozen scope (composed batch_25)

**Composed:** 2026-05-10
**Source:** Deep fix loop (batches A–Q, commits `390d9305` → `4b2d1365`)
**Scope:** Items routed out of deep_fix mode because they exceeded the ≤50 LoC / ≤3 files / no-architectural-change gate, or because they require Brain / founder decisions that deep_fix mode cannot make unilaterally.

This list is FROZEN at composition time. New findings discovered during batch_26+ should be filed against a future BACKLOG_v3, not silently appended here.

---

## Summary: 24 items routed from deep fix loop

| Priority | Count |
|---|---|
| **P1 — must address before launch** | 7 |
| **P2 — address before public beta** | 11 |
| **P3 — post-beta nice-to-have** | 6 |
| **Total** | **24** |

| Category | Count |
|---|---|
| AI-features | 3 |
| Account | 1 |
| Discovery | 5 |
| Performance | 4 |
| Privacy | 1 |
| Profile | 4 |
| Search | 2 |
| Vibe-Lab | 4 |

Sources: 19 from `round-trips/CODE_AUDIT.md` (Plan agent, batch_24) + 4 from cycle-2 deep_fix static analysis + 1 from `round-trips/STUCK_LOG.md`.

---

## Items by priority

### P1 (must address before launch)

#### BL-20260510-01 — Tenor API key hardcoded in source [SECURITY]
- **Category:** Account
- **Source:** STUCK-01 (Tenor key triage during deep_fix)
- **Severity:** P1
- **Description:** `src/constants/config.js:46` ships a Google API key (AIzaSy… format) in committed source. Per CLAUDE.md §14 this is a hard-prohibited secret-in-source case.
- **Why-routed:** Founder must rotate the key on Google Cloud Console + add the new value to EAS env vars before CC can do the 1-line source replacement. Founder console action is non-delegable per CLAUDE.md §1.
- **Proposed-batch:** batch_26 (RED_BLOCK at start; founder unblocks, then 3-line follow-up)

#### BL-20260510-02 — recordSwipe has unreachable server-side match-listener branch
- **Category:** Discovery
- **Source:** CODE_AUDIT 1.5
- **Severity:** P1
- **Description:** `src/services/matchService.js:312-341` contains a server-side match-listener branch that is currently unreachable because no caller flips the corresponding flag. Either the branch is dead code that should be deleted, or the wire-up is the missing piece (depends on whether server-side match detection is part of the launch story).
- **Why-routed:** Architectural decision — Brain must call whether server-side detection is in scope for launch.
- **Proposed-batch:** batch_27 (Discovery feature scope decision)

#### BL-20260510-03 — HomeScreen empty-state masks errors as "no profiles found"
- **Category:** Discovery
- **Source:** CODE_AUDIT 6.2
- **Severity:** P1
- **Description:** When `loadProfiles` rejects (network down, Firestore rate-limit, permission denied), `HomeScreen` falls through to the same `<EmptyState>` UI used for legitimate zero-results. Users see "no profiles found" when the truth is "we couldn't load profiles."
- **Why-routed:** Needs new error-state UI + copy + retry CTA design from product/design.
- **Proposed-batch:** batch_28 (Discovery UX polish)

#### BL-20260510-04 — vibeScore + brainScore computed in parallel (waste)
- **Category:** Performance
- **Source:** CODE_AUDIT 4.3
- **Severity:** P1
- **Description:** `HomeScreen` + `matchService` compute the legacy 40/40/20 `vibeScore` AND the newer 5-component `brainScore` for every candidate, but only one is rendered. Doubles the per-profile compute cost for every queue load.
- **Why-routed:** >50 LoC across 2+ files; CLAUDE.md MEMORY.md still references the 40/40/20 algorithm so deletion needs Brain confirmation that brainScore has fully superseded it.
- **Proposed-batch:** batch_27 (combine with BL-02 Discovery scope review)

#### BL-20260510-05 — MatchesScreen FlatList nested inside ScrollView (perf anti-pattern)
- **Category:** Performance
- **Source:** CODE_AUDIT 7.1
- **Severity:** P1
- **Description:** `MatchesScreen` renders a FlatList inside a ScrollView, which defeats virtualization — the entire matches list mounts at once. As real users accumulate matches the screen will jank and OOM on low-RAM devices (Layer 5/6 trigger).
- **Why-routed:** Full restructure — needs to extract sections + render via a single FlatList with section headers, or split into separate scroll regions. >50 LoC across navigation + list components.
- **Proposed-batch:** batch_29 (Performance pre-beta pass)

#### BL-20260510-06 — BlockListScreen surfaces Firebase errors as raw strings
- **Category:** Privacy
- **Source:** CODE_AUDIT 6.1
- **Severity:** P1
- **Description:** When the block/unblock Firestore mutation fails, the user sees an unstyled error toast with the raw Firebase error message. Privacy-adjacent flows must degrade gracefully.
- **Why-routed:** Needs new error-UX state + copy decision + retry CTA design.
- **Proposed-batch:** batch_28 (with BL-03; bundled UX polish)

#### BL-20260510-07 — `languages` field schema migration gap
- **Category:** Profile
- **Source:** CODE_AUDIT 10.1
- **Severity:** P1
- **Description:** Newer onboarding writes `languages: string[]` but legacy users have no field, and several scorers `.includes()` directly on the array without an undefined guard. First match-cycle for legacy users will throw or score 0 unfairly.
- **Why-routed:** Schema migration — Brain must decide between (a) backfill via Cloud Function, (b) defensive `?.includes` callsites, or (c) Firestore rule + write-on-read pattern.
- **Proposed-batch:** batch_27 (combine with BL-02 / BL-04 Discovery scope review)

---

### P2 (address before public beta)

#### BL-20260510-08 — aiClient has no rate-limit UX
- **Category:** AI-features
- **Source:** CODE_AUDIT 9.1
- **Severity:** P2
- **Description:** Gemini calls under load can return 429s with retry-after headers, but `aiClient.js` surfaces them as generic errors. Users see "AI is unavailable" with no signal that retrying in 30s would work.
- **Why-routed:** API-shape change rippling to all callers + new UX state.
- **Proposed-batch:** batch_30 (AI features hardening)

#### BL-20260510-09 — aiClient cache-key collisions on prompt variants
- **Category:** AI-features
- **Source:** CODE_AUDIT 9.2
- **Severity:** P2
- **Description:** Cache key is `prompt.slice(0, 64)` — long prompts that share a 64-char prefix collide and return wrong cached completions.
- **Why-routed:** Requires hash impl choice (sha256 vs murmur vs FNV) + cache-eviction strategy.
- **Proposed-batch:** batch_30 (with BL-08 / BL-10 AI bundle)

#### BL-20260510-10 — aiClient retries reuse identical prompt
- **Category:** AI-features
- **Source:** CODE_AUDIT 9.3
- **Severity:** P2
- **Description:** On Gemini transient error, retry uses the same prompt verbatim. Best practice is mild prompt variation on retry to dodge model-side identical-input deduping. Prompt-engineering decision.
- **Why-routed:** Brain must approve the retry-prompt-mutation strategy.
- **Proposed-batch:** batch_30 (with BL-08 / BL-09 AI bundle)

#### BL-20260510-11 — MainTabs.js orphan (176 LoC)
- **Category:** Discovery
- **Source:** CODE_AUDIT 2.3
- **Severity:** P2
- **Description:** `src/navigation/MainTabs.js` defines a `TAB_CONFIG` array but is unused — `AppNavigator.js` inlines all tabs directly. Either delete (cosmetic) or refactor `AppNavigator` to consume the config (architectural).
- **Why-routed:** Architectural decision — Brain calls delete vs adopt.
- **Proposed-batch:** batch_28 (Discovery refactor)

#### BL-20260510-12 — obfuscateDistance returns string in some paths, number in others
- **Category:** Discovery
- **Source:** CODE_AUDIT 10.4
- **Severity:** P2
- **Description:** `matchService.obfuscateDistance` returns `"~5km"` (string) in display paths but a numeric value when called from `HomeScreen` filter logic. Type inconsistency leads to silent comparison bugs (`"~5km" > 10` is false but not for the right reason).
- **Why-routed:** API-shape change rippling across matchService + HomeScreen + display.
- **Proposed-batch:** batch_29 (combine with BL-05 perf pass; both touch matchService)

#### BL-20260510-13 — Sequential awaits in match/conversation hot paths
- **Category:** Performance
- **Source:** CODE_AUDIT 7.2
- **Severity:** P2
- **Description:** `matchService` + `chatService` use `for (const x of arr) await fn(x)` patterns where `Promise.all(arr.map(fn))` would parallelize. N round-trips become 1.
- **Why-routed:** Rewrites flow control across multiple services; needs careful audit for ordering dependencies.
- **Proposed-batch:** batch_29 (Performance pre-beta pass)

#### BL-20260510-14 — HomeScreen profiles.map allocation per render
- **Category:** Performance
- **Source:** CODE_AUDIT 7.3
- **Severity:** P2
- **Description:** `HomeScreen` rebuilds `profiles.map(p => decorate(p))` on every render even when `profiles` reference is stable. Wraps with `useMemo` is the obvious fix; intertwined with swipe animation state, so >50 LoC restructure.
- **Why-routed:** Restructure intertwined with `react-native-deck-swiper` ref handling.
- **Proposed-batch:** batch_29 (Performance pre-beta pass)

#### BL-20260510-15 — DUMMY_USER_PROFILE legacy in userService
- **Category:** Profile
- **Source:** CODE_AUDIT 2.4
- **Severity:** P2
- **Description:** `src/services/userService.js:42-111` carries DUMMY_USER_PROFILE remnants from earlier dev mode. Confirmed unused at the call site Plan checked but `dev_*` / `mock_*` prefix users may still consume it.
- **Why-routed:** Deletion needs verification across all `dev_*` / `mock_*` user code paths.
- **Proposed-batch:** batch_28 (with BL-11 cleanup)

#### BL-20260510-16 — getUserProfile minimal-stub semantics inconsistent
- **Category:** Profile
- **Source:** CODE_AUDIT 10.2
- **Severity:** P2
- **Description:** `userService.getUserProfile` returns a minimal stub `{uid, name?}` for not-yet-onboarded users, but callers across services treat absence-of-fields differently (some default, some throw, some skip). Needs a consistent "minimal" contract.
- **Why-routed:** API contract change rippling to many services.
- **Proposed-batch:** batch_28 (Profile contract pass)

#### BL-20260510-17 — FONT_SIZES vs V2_TYPE_SCALE coexistence
- **Category:** Vibe-Lab
- **Source:** CODE_AUDIT 5.4
- **Severity:** P2
- **Description:** `theme.js:411` exports both legacy `FONT_SIZES` and the newer `V2_TYPE_SCALE`. Two systems live side-by-side; new screens already mix them.
- **Why-routed:** Architectural rationalization — Brain calls which is canonical, then mass replace.
- **Proposed-batch:** batch_30 (Design system v2 pass; combines with BL-23 / BL-24)

#### BL-20260510-18 — WarmEmber.js orphan component (176 LoC)
- **Category:** Vibe-Lab
- **Source:** Cycle-2 deep_fix-A
- **Severity:** P2
- **Description:** `src/components/WarmEmber.js` has 0 importers across `src/` (only self-references). Pure deletion (low risk).
- **Why-routed:** 176-line file delete exceeds the ≤50 LoC scope gate; spec-compliant route is BACKLOG.
- **Proposed-batch:** batch_28 (with BL-11 / BL-15 cleanup batch)

---

### P3 (post-beta nice-to-have)

#### BL-20260510-19 — `||` falsy-guard age in aiSearchService
- **Category:** Search
- **Source:** CODE_AUDIT 3.4
- **Severity:** P3
- **Description:** `src/services/aiSearchService.js:331` uses `user.age || 18` — if a user had `age: 0` (impossible by 18+ enforcement, but theoretical) the guard would silently substitute 18.
- **Why-routed:** Theoretical edge only; not impacting any real user. Filed for completeness.
- **Proposed-batch:** DEFERRED post-launch

#### BL-20260510-20 — SearchScreen unmanaged setTimeout
- **Category:** Search
- **Source:** Cycle-2 deep_fix-B
- **Severity:** P3
- **Description:** `src/screens/main/SearchScreen.js:324` uses `setTimeout(() => handleSearch(searchQuery), 100)` with no `clearTimeout` on unmount — would log a setState-on-unmounted warning if the user navigates away within 100ms.
- **Why-routed:** 100ms window too narrow to reliably hit; cosmetic warning, not user-impacting.
- **Proposed-batch:** DEFERRED post-launch (or fold into batch_30 Search polish)

#### BL-20260510-21 — backgroundLight token off the Warm Ember palette
- **Category:** Vibe-Lab
- **Source:** CODE_AUDIT 5.2
- **Severity:** P3
- **Description:** `src/constants/theme.js:47` defines `backgroundLight` outside the canonical Warm Ember palette. Needs full repo grep to confirm zero refs before deletion.
- **Why-routed:** Audit + cleanup; visual harmless if unused.
- **Proposed-batch:** batch_30 (Design system v2 pass)

#### BL-20260510-22 — Multiple off-palette colors in theme.js
- **Category:** Vibe-Lab
- **Source:** CODE_AUDIT 5.3
- **Severity:** P3
- **Description:** `theme.js:74-77, 81-83, etc.` — several legacy colors that are not Warm Ember tokens. Refactor across many call sites.
- **Why-routed:** Brand-vibe pass, not user-impacting in current screens but blocks design-system v2 ratification.
- **Proposed-batch:** batch_30 (Design system v2 pass)

#### BL-20260510-23 — Hardcoded fontSize across 759 sites
- **Category:** Vibe-Lab
- **Source:** Cycle-2 deep_fix-C
- **Severity:** P3
- **Description:** `grep -c "fontSize: [0-9]"` across `src/screens/` + `src/components/` returns 759 — all bypass `theme.js` `FONT_SIZES` / `V2_TYPE_SCALE` tokens.
- **Why-routed:** Too broad; needs design-system v2 migration pattern (codemod) before hand-edits.
- **Proposed-batch:** batch_30 (Design system v2 pass; combines with BL-17)

#### BL-20260510-24 — Spacing grid violations (10+ sites)
- **Category:** Vibe-Lab
- **Source:** Cycle-2 deep_fix-D
- **Severity:** P3
- **Description:** `paddingVertical` / `paddingHorizontal` values not on the design 4/8/16/24 grid; ~10+ confirmed sites.
- **Why-routed:** Too broad without a design audit pass + codemod.
- **Proposed-batch:** batch_30 (Design system v2 pass)

---

## Provenance

- **Plan agent CODE_AUDIT** (`round-trips/CODE_AUDIT.md`, batch_24): produced 24 findings — 16 fixed in batches A–I, 8 routed BACKLOG (1.5, 2.3, 2.4, 3.4, 4.3, 5.2, 5.3, 5.4, 6.1, 6.2, 7.1, 7.2, 7.3, 9.1, 9.2, 9.3, 10.1, 10.2, 10.4 → 19 distinct after counting sub-findings; 9.x bundle had Sentry portion fixed via Batch J leaving 9.1/9.2/9.3 as routed).
- **Cycle-2 static analysis** (deep_fix mode, post Plan-self-audit): surfaced 4 additional items — WarmEmber orphan, SearchScreen setTimeout, hardcoded fontSize population, spacing grid violations.
- **STUCK_LOG.md** (Tenor key, RED_BLOCK candidate): 1 item routed for founder console action.

19 + 4 + 1 = 24.

## Acceptance criteria (for batches that close BACKLOG_v2 items)

When a batch closes a BL-20260510-NN item:
1. Reference the BL ID in the commit message and EVIDENCE_LEDGER.
2. Update this file by appending `**STATUS:** CLOSED — batch_NN — <commit-sha>` to the item's entry.
3. Note in the closing-batch round-trip that the BL item was a BACKLOG_v2 candidate.

This file is FROZEN to additions. All new findings → BACKLOG_v3.
