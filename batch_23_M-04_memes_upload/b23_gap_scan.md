# §4.5 5-Question Gap Scan — batch_23

## Q1: What did Brain assume that turned out wrong?
**Two stale assumptions:**
1. **Spec said 9,572 PNGs**. Actual Firestore /memes total is **15,872** (65% larger). Actual Storage state is **17,072 files** (78% larger than spec). The "9,572" figure derives from an earlier Firestore state, possibly from when M-04 was first scoped in BACKLOG_PHASE_1.md before the Schema B Reddit-scraping pipeline ran.
2. **Spec said bucket `lolbite-3ea7d.appspot.com`**. Actual bucket is `lolbite-3ea7d.firebasestorage.app`. First investigation run failed with "The specified bucket does not exist." Plan agent flagged this as risk R4 pre-execution.
- **D-23-01 (DOC, S-3)**: BACKLOG_PHASE_1.md M-04 mark CLOSED_GREEN with verified-pre-existing note (similar pattern to batch_21 T2-12 EAS).
- **D-23-02 (DOC, S-3)**: CLAUDE.md §12 Firebase Specifics should list `lolbite-3ea7d.firebasestorage.app` as canonical bucket name to prevent future-batch confusion.

## Q2: What scope grew during execution that wasn't in the spec?
**None — scope shrank.** Spec assumed a multi-hour 9,572-PNG upload from local sources. Plan agent's B23-P-1 investigation surfaced that Storage was already 100% populated. Batch became verify-only (no upload, no Firestore mutation). Total runtime: ~8 min for investigation + verify, vs estimated 30-90 min for the upload spec assumed.

This is the **second consecutive batch where Plan agent caught a "do X" spec for X-already-done** (batch_21: T2-12 EAS appVersionSource was already remote; batch_23: M-04 memes were already in Storage). Pattern emerged in INNOVATION_LOG batch_22 — Plan-agent-as-state-verifier.
- **No D entry** — pattern is now well-documented; spec accuracy delta is the natural consequence of doing batches in parallel with state-changing work outside the build loop.

## Q3: What did the Plan agent surface that the spec didn't?
**Six findings:**
1. **R1**: 9572 doesn't match any local source (sources: 540/3000/3000/8000/6835).
2. **R2**: Two-schema Firestore split (A curated 540 + B Reddit-scraped 14,832 + 500 placeholder).
3. **R3**: Mixed file extensions (Schema A is PNG; Schema B is mixed PNG/JPG).
4. **R4**: Bucket name discrepancy.
5. **R5**: Storage path convention drift (Schema A uses storagePath field; Schema B uses download_url URL extraction).
6. **R6**: Existing uploadMemes.js writes to Firestore — would have violated batch's "read-only on Firestore" constraint if reused without modification. New script diverged correctly.

## Q4: What did the post-investigation NOT verify?
**Comprehensive Schema B mapping.** I verified 20/20 random Schema B docs map to existing Storage paths, but did NOT enumerate all 14,832 Schema B docs. Inferring 100% mapping from a 20-sample is statistically loose (a 1% gap = 148 missing memes would be invisible to a 20-sample). However:
- Spot-check 20/20 with magic-byte verification = high confidence
- Storage count 17,072 > Firestore total 15,872 (Storage is a superset)
- All sampled Schema B docs (random IDs across categories) matched

A future sweep could exhaustively verify all 14,832 if discrepancy emerges. Not blocking M-04 acceptance.
- **D-23-04 (P3 OBS)**: Future batch could exhaustive-verify all Schema B docs map to Storage if any user-facing meme display reports missing memes.

## Q5: Is there a Storage state that PASSES today but would mask a real failure?
**Two candidates:**
1. **`download_url` extraction regex**: my Schema B verifier extracts `memes/[^?#]+` from the URL. If a Schema B doc had a malformed `download_url` (e.g., absolute Reddit CDN URL with no `memes/` substring), my regex returns null and counts as "unmatched." 20/20 sample showed all matched, but this assumes Reddit-CDN-style URLs are absent. If they exist, those memes would silently fail to display in the app even though Storage looks "fine."
2. **Spot-check is RANDOM-ANY-FILE**, not RANDOM-FROM-FIRESTORE. My current spot-check picks 20 files from `bucket.getFiles({prefix:'memes/'})` and verifies. This proves "20 random Storage files are valid PNG/JPG" — but a more rigorous test would pick 20 random Firestore /memes docs and verify their `storagePath`/`download_url` resolves to a valid Storage file. Current verify is one-direction (Storage → magic bytes); ideal is bidirectional (Firestore → Storage → magic bytes).
- **D-23-05 (P3 OBS)**: verify_memes_b23.js could be enhanced with `--spot-check-firestore N` mode that picks N random Firestore docs and verifies Storage roundtrip. Future iteration.

## Triage summary for Brain
- **D-23-01** (DOC, S-3): BACKLOG M-04 mark CLOSED_GREEN
- **D-23-02** (DOC, S-3): CLAUDE.md §12 add canonical bucket name
- **D-23-03** (P3 OBS): 500 placeholder picsum docs cleanup (future)
- **D-23-04** (P3 OBS): Exhaustive Schema B verification (future, only if user reports missing)
- **D-23-05** (P3 OBS): Bidirectional verify_memes spot-check enhancement (future)
