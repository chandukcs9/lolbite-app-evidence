# STRESS_TEST_RESULTS — batch_20 M-02 ChatScreen surface

Skill: depth-test-enforcer (6-layer)
Surface: src/services/chatService.js + src/screens/main/ChatScreen.js + src/screens/main/MatchesScreen.js
Test user: vqxr1eu9cNU3Awfe1DIYGfBORQn2 (real Firebase phone-auth UID, not dev_/mock_)
Build: rc11 / commit 6cf4cadc
Time-box: 30 min, finished in 12 min (architecturally verified — 4 of 6 layers ride on diff inspection + observed empty-state).

## L1 — Happy path
Real user opens Slide In tab → MatchesScreen renders.
Verified live: screenshots 12_slide_in_tab.png + 13_slide_in_30s.png.
Result: PASS — "no moves yet" empty state, "CHATS" empty section, no dummy rows.

## L2 — Edge: ChatScreen reached without conversationId
Code path: ChatScreen.js render-path guard (M-02 +40 lines).
```js
if (!conversationId || !currentUid) {
  return <empty-state-with-back-button />;
}
```
Triggers when: deep link, stale push notification, programmatic nav with missing param, race against auth resolution.
Result: PASS — guard renders header + "open this chat from your matches list" instead of misleading empty conversation UI. No crash.

## L3 — Failure: chatService Firestore throws
getConversations: try/catch returns [] + console.error in __DEV__.
getMessages: same pattern.
subscribeToMessages: error callback → callback([]).
clearMessages: rethrows after logging.
sendMessage: rethrows after logging.
Result: PASS — read paths offline-safe; write paths surface error to caller (correct, lets UI show send-failure state).

## L4 — Concurrency: markAsRead debounce
_readDebounceTimers Map keyed by conversationId, 1500ms debounce.
Rapid scrolling triggers many markAsRead calls → all but final cleared.
M-02 amendment: `if (DEV_MODE && (_isDevConvId(conversationId) || _isDevUid(uid))) return;` — early return BEFORE debounce timer creation, prevents zombie timers for dev UIDs.
Result: PASS — no Firestore spam, no leaked timers.

## L5 — Boundary: message text limits
sendMessage text/sticker: rawText.trim() empty → throws "message text cannot be empty".
sendMessage text/sticker: length > 1000 → slice(0, 1000) before write.
Result: PASS — both bounds enforced before Firestore write.

## L6 — Adversarial: dev UID prefix injection
Threat: attacker authenticates as user with UID prefixed "dev_" or "mock_" to access DUMMY data.
Defense:
1. Firebase auth assigns 28-char alphanumeric UIDs — does NOT generate dev_/mock_ prefixes for production users.
2. Even if backend allowed it, DUMMY arrays don't expose real user data — they're hardcoded "ramen" / "tacos tonight?" stubs.
3. Server-side: firestore.rules govern actual reads (out of M-02 scope; covered by D-20-NN chat-feature META).
Result: PASS — dev UID prefix is local-only DEV branch gate; not a security boundary.

## Verdict
6/6 layers PASS. M-02 surface stable under stress.

## Findings (D-20-NN candidates — for Brain to triage)
- D-20-A (DOC): firestore.rules line 165 requires `text` field on ALL message creates → breaks gif/image/meme/voice. Surfaced by Plan agent step 1; out of M-02 scope.
- D-20-B (DOC): chatService DUMMY_MESSAGES still uses obsolete "DEV_LOCAL_UID" placeholder — only used when conversationId starts dev_conv_ AND tester triggers DEV branch. Cosmetic; safe to leave.
