# MEMES_UPLOAD_PLAN — batch_23 Plan agent verbatim output

Composed: 2026-05-10
Plan agent verdict: **9572 source count doesn't match any single local dir.** Comprehensive investigation step required.

---

## ARCHITECTURAL RISKS

- **R1 (CRITICAL): 9,572 acceptance count likely does not match any single source.** Discovered PNG sources: `assets/memes/*` = 540 PNGs, `meme_factory/data/renders_final` = 3000, `renders_final_v1` = 3000, `renders_final_v2` = 8000. None equal 9572. The 9572 count likely comes from Firestore `memes/` doc count (sum of curated + scraped). Plan must HALT at INV-A and reconcile via INV-B before deciding which on-disk source maps to which Firestore docs.
- **R2 (CRITICAL): Two-schema Firestore split.** `probe-memes-collection.js` already documents Schema A (`isActive`+`url`+curated) and Schema B (`status`+`download_url`+Reddit-scraped). Schema B docs may reference `download_url` pointing to Reddit CDN, not Storage — uploading PNGs only solves Schema A. Schema B is likely JPGs not PNGs.
- **R3: File-extension mismatch.** Brain spec says PNG. `data/approved_memes/` is JPG (5318 jpg / 1023 png / 494 other). If Firestore Schema B references `.jpg` filenames, "9572 PNGs" is the wrong target.
- **R4: Bucket name discrepancy.** Acceptance says `lolbite-3ea7d.appspot.com`; existing `scripts/uploadMemes.js` uses `lolbite-3ea7d.firebasestorage.app`. Both names route to the same physical bucket on modern Firebase, but Admin SDK init must use the canonical one. Confirm with `admin.storage().bucket().name` before any upload.
- **R5: Storage path convention drift.** Existing script writes `memes/{category}/{filename}`. If Firestore docs (Schema B esp.) record a different `storagePath` field, mapping logic must derive paths from Firestore, not from local-dir convention.
- **R6: Admin SDK bypasses storage.rules write-block, but Firestore writes are scoped out (read-only).** The existing `uploadMemes.js` ALSO writes Firestore docs (`db.collection('memes').doc(id).set(...)`) — that violates this batch's "NEVER mutates Firestore" constraint. The new script MUST diverge from `uploadMemes.js` and skip the Firestore write step entirely.
- **R7: 9572-file upload at 10 concurrency over residential ISP could take 30–90 min and may trip Firebase quota. Idempotent checkpoint is mandatory.**

## INVESTIGATION FINDINGS (preliminary; full B23-P-1 required)

- **A. Source PNGs (preliminary):**
  - `assets/memes/{14 categories}/*.png` → 540. Naming: `{category}_NNNN.png`.
  - `meme_factory/data/renders_final/*.png` → 3000. Naming: `meme_NNNN.png`.
  - `meme_factory/data/renders_final_v1/*.png` → 3000.
  - `meme_factory/data/renders_final_v2/*.png` → 8000. Naming: `v4_NNNN.png`.
  - `data/approved_memes/*` → 6835 mixed (5318 jpg + 1023 png + 494 other). Reddit-id naming.
  - **None match 9572.**
- **B. Firestore schema** (from `scripts/probe-memes-collection.js`, not yet executed for THIS batch):
  - Two schemas. A: `isActive` + `url` + `storagePath` + `category`. B: `status` + `download_url` (no `storagePath`).
- **C. Storage rules** (`storage.rules`): `match /memes/{allPaths=**}` allows authenticated read; `allow write: if false` for clients. Admin SDK bypasses by default. Safe.
- **D. Existing Storage state**: Unknown until B23-P-1 lists `bucket.getFiles({prefix: 'memes/'})`.

## STEPS

### B23-P-1 — Investigation: Firestore schema, source files, existing Storage state
Author `scripts/b23_investigate.js`; capture bucket name, Firestore total, schema A/B counts, 5 random docs each schema, Storage prefix count + 10 sample paths; dump JSON + log; classify local source dirs.

### B23-P-2 — Source-to-Firestore mapping reconciliation (HARD GATE)
Build mapping `{docId, targetStoragePath, localSourcePath, exists, schema}`. If `mappable_with_local_file < 0.95 * firestoreTotal` → HALT escalate.

### B23-P-3 — Upload script `scripts/upload_memes_b23.js` (new; do NOT modify uploadMemes.js)
Reads mapping JSON → uploads localSourcePath → targetStoragePath. Concurrency 10 hand-rolled semaphore. Retry 3x exponential. Idempotent via checkpoint + bucket.exists(). NEVER touches Firestore. NEVER echoes SA. contentType inferred. NO makePublic.

### B23-P-4 — Verify script `scripts/verify_memes_b23.js` + `scripts/verify_memes.sh` wrapper
Counts + --spot-check N (signed URL fetch + magic bytes verify).

### B23-P-5 — Dry-run upload
node scripts/upload_memes_b23.js --dry-run. MAPPING_COUNT matches B23-P-2.

### B23-P-6 — Pre-upload state snapshot
Storage count baseline + Firestore 5-doc snapshot for immutability check.

### B23-P-7 — Live upload run
Concurrent upload, checkpoint resume capable. Exit 0 + report.

### B23-P-8 — Post-upload Storage count verify
Pre vs post diff documented.

### B23-P-9 — Spot-check 20 random uploads (HARD GATE)
20/20 PASS magic bytes + 200 OK.

### B23-P-10 — Firestore immutability re-verify
5 docs pre vs post diff = empty.

### B23-P-11 — Tribunal hard gate
28/28 PASS.

### B23-P-12 — Closure (ledger + tracking files + commit + push + Auto-Bridge)
Standard close per §6 + §12. NEVER force-push. NO SA in diff.
