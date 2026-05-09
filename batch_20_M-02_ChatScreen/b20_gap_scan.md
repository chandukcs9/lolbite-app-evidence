# §4.5 5-Question Gap Scan — batch_20

## Q1: What did Brain assume that turned out wrong?
**Brain assumed Slide In tab routes to ChatScreen.js.** Plan agent step 1 verified Slide In → MatchesScreen.js (conversation list); ChatScreen.js is the conversation-detail view, only reachable by tapping a chat row. Plan agent's Interpretation 2 surfaced this — without Plan, the batch would have exec'd against the wrong screen.
- **D-20-01 (DOC, S-3)**: BACKLOG_PHASE_1.md M-02 description should be amended: "Slide In tab → MatchesScreen (conversation list); ChatScreen reached via row tap."

## Q2: What scope grew during execution that wasn't in the spec?
**MatchesScreen.js dead-code removal (-124 lines).** Spec named "ChatScreen real-user wiring" — MatchesScreen wasn't mentioned. But Plan agent surfaced DUMMY_NEW_BITES + DUMMY_CONVERSATIONS as the actual M-02 dummy-data leak, since MatchesScreen renders before any ChatScreen tap. Removed under M-02's "no dummy data" acceptance.
- **No D entry** — scope creep was correct given user-visible surface; in-scope per acceptance criteria.

## Q3: What did the Plan agent surface that the spec didn't?
**Three findings:**
1. **THE actual M-02 hazard** — bare `if (DEV_MODE)` short-circuits in chatService.js return DUMMY regardless of UID; in __DEV__ builds real users see dummy data. Fixed via _isDevUid/_isDevConvId helpers.
2. **firestore.rules schema bug** — line 165 requires text field on ALL message creates → breaks gif/image/meme/voice writes. Out of M-02 scope.
3. **ChatScreen render-path gap** — no guard for missing conversationId (deep link, stale notification). Added 40-line guard.
- **D-20-02 (BUG, S-2)**: firestore.rules text-field requirement breaks non-text message types. Routed to D-20-NN chat-feature META — Brain composes when chat features wire up.

## Q4: What did the device verify show that the diff couldn't?
**Empty CHATS section + "no moves yet" rendered correctly.** Diff inspection alone cannot prove the empty state because:
- Loading skeleton might never resolve.
- Firestore offline-safe `return []` could be masking an auth permission error.
- React Native rendering bugs are diff-invisible.
Live verify on rc11 with vqxr1eu9cNU3Awfe1DIYGfBORQn2 (real Firebase phone-auth UID, not dev_/mock_) confirmed: PID stable across 30s, logcat clean (no FATAL/Exception in pid=27076 stream), tab nav works, empty state renders with branded copy.

## Q5: What pre-existing technical debt did tribunal surface that nobody is tracking?
Tribunal 8 FAILs — none from M-02 surface (verified by grep). Each needs a D entry:
- **T1-1**: 1 corporate buzzword — likely D-19-03 leftover (PARTIAL FIX)
- **T1-5**: "looking for" phrase in onboarding (2) — branding drift, NEW gap
- **T1-9**: Match-phrase used (1) — should be "Move" per amovi-dev-workflow
- **T2-1**: TYPOGRAPHY imported (1) — CRASH risk per CLAUDE.md memory; CRITICAL
- **T2-8**: npx in code (39) — process-enforcer rule violation, broad
- **T2-9**: EXPO_OFFLINE in code (5) — process-enforcer rule violation
- **T2-11**: DevBypassScreen ref (8) — D-15-04 cleanup leftover
- **T3-6**: 6 onboarding screens — D-19-04 PhotoUpload known
- **D-20-03 (META, S-2)**: Open tribunal-debt META to track each FAIL → individual D entry → close one per batch.

## Triage summary for Brain
- **D-20-01** (DOC, S-3): BACKLOG M-02 description amend
- **D-20-02** (BUG, S-2): firestore.rules text-field schema bug
- **D-20-03** (META, S-2): tribunal-debt cleanup META
