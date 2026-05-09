# TEST_USERS — Amovi Test User Inventory

**Updated:** 2026-05-08 (batch_22)
**Firebase project:** `lolbite-3ea7d` (per CLAUDE.md §12)
**Seed script:** `scripts/seed_test_users.js` (idempotent; node)
**Reset script:** `scripts/reset_test_users.sh` (POSIX bash; supports `--dry-run`)
**List script:** `scripts/list_test_users.js` (read-only Auth listing)

## Test users (3)

| # | UID | Phone | Display Name | Source |
|---|---|---|---|---|
| 1 | `vqxr1eu9cNU3Awfe1DIYGfBORQn2` | `+15555550100` | (auth-only) | Pre-existing per CLAUDE.md §12 (batch_05+) |
| 2 | `6DfeJAzO2PZNyDcGOWo90I61g7X2` | `+15555550101` | (auth-only) | Pre-existing per CLAUDE.md §12 (batch_06+) |
| 3 | `KXet39oKhoV3y63D7JeIJL9UCfr1` | `+15555550102` | Amovi Test User 3 | **Seeded batch_22** via `scripts/seed_test_users.js` |

## OTP convention

Firebase test phone numbers (configured in `whitelist`) bypass real SMS delivery. Test users authenticate with:

| Phone | OTP code |
|---|---|
| `+15555550100` | (any 6-digit; tests historically used `123456`) |
| `+15555550101` | (any 6-digit; tests historically used `654321`) |
| `+15555550102` | (newly registered; needs OTP code added to Firebase Console whitelist) |

> Note: User 3 was created via Admin SDK `createUser({phoneNumber, displayName})` which registers the phone, but **OTP whitelist entries are managed in Firebase Console at Authentication → Settings → Phone numbers for testing**. Founder action may be required to add an OTP for `+15555550102` if device-test against this user is needed.

## Seed/reset workflow

**Idempotent seed (safe to re-run):**
```
node scripts/seed_test_users.js
```
- For each phone: skip-if-exists; create-if-missing.
- For each users/<uid> Firestore profile: skip-if-exists; write-if-missing.

**Verify only (no mutations):**
```
node scripts/seed_test_users.js --verify
```

**Reset (destructive — wipes profiles + re-seeds):**
```
RESET_CONFIRM=YES bash scripts/reset_test_users.sh
# OR for preview only:
bash scripts/reset_test_users.sh --dry-run
```

## Default seed profile

Each user 1/2/3 gets this Firestore `users/<uid>` doc on fresh seed (only written if absent):

```
{
  onboardingComplete: true,
  name: "amovi test user N",
  age: 25,
  gender: "f",
  lookingFor: "long term",
  location: { city: "Hyderabad", state: "Telangana", country: "India", lat: 17.385, lng: 78.486 },
  photos: ["test-url-1", "test-url-2"],
  bio: "batch_22 test fixture — automated seed",
  updatedAt: <serverTimestamp>
}
```

## Service account

`firebase-service-account.json` lives at repo root. **NEVER committed** (covered by `.gitignore` lines 119, 123). Required by `firebase-admin` initialization in seed/list scripts. If missing, scripts fail with module-not-found.

## Founder reference

- Authentication console: https://console.firebase.google.com/project/lolbite-3ea7d/authentication/users
- Firestore console: https://console.firebase.google.com/project/lolbite-3ea7d/firestore/data/~2Fusers
- Phone numbers for testing (OTP whitelist): https://console.firebase.google.com/project/lolbite-3ea7d/authentication/providers
