## 📖 Story / Why
The platform operator (you) needs a way to actually run Thapar Bites: onboard food courts, create their accounts, grant roles, and oversee orders. Nothing in the current backlog does this — #2 built the `setUserRole` function but there's no way to *use* it. Without this, you can't set up your first pilot vendor. This is the operator's control room.

## 🧭 Context
**Platform: web only, by design** — the admin works on a laptop/desktop browser. React + TS admin console + Firestore, admin-only (`admin` custom claim). Uses `setUserRole` (#2) and writes `courts` (`docs/DATA_MODEL.md` §2). Read-only oversight of the top-level `orders` collection.

**First-admin bootstrap (chicken-and-egg):** `setUserRole` is admin-only, so the *first* admin claim must be set out-of-band via a one-time Admin SDK script (custom claims can't be set from the Firebase console UI). This ticket includes that script + documents it in `docs/SETUP.md`.

## 🔑 Access & prerequisites
- Auth & roles (#2) built, including the `setUserRole` callable.
- Ability to run a one-time Admin SDK script against the project to seed the first admin (uses the Functions/Admin credentials; no service-account key committed).

## ✅ Scope / What to build
- [ ] Admin sign-in → admin console; non-admins are denied (rules + claim check).
- [ ] **First-admin bootstrap script** (one-time, Admin SDK) + a short `docs/SETUP.md` note on running it.
- [ ] **Onboard a court:** create a `courts/{courtId}` doc and assign an `ownerUid`; grant that user the `vendor` claim via `setUserRole`. (Vendor then manages their court via #3.)
- [ ] **Roles:** grant / revoke `vendor` and `admin` claims for a user.
- [ ] **Court visibility:** toggle a court live/offline (only live courts appear to students in #4).
- [ ] **Orders oversight:** read-only view of all orders across courts, with a basic status/court filter (for dispute triage).
- [ ] Loading / empty / error states.

## 🎯 Acceptance Criteria
- [ ] The first admin can be bootstrapped and log into the console; a non-admin cannot access it (verified against rules).
- [ ] An admin creates a court, assigns a vendor, and that vendor can then log into their dashboard (#3) and manage only that court.
- [ ] An admin can grant and revoke `vendor`/`admin` roles.
- [ ] Toggling a court offline removes it from the student court list (#4); toggling it on restores it.
- [ ] An admin can view orders across all courts; a vendor/student still cannot.

## 🖼️ UI standards
UI ticket — apply `templates/ui-standards.md`. Load-bearing: light + dark; responsive with desktop reflow + min width; correct truncation; loading/empty/error/disabled; no hardcoded strings; design-system tokens. Match the provided design exactly (ask the PM at `/start-ticket` if not attached).

## 🚫 Out of scope
- Rich analytics/reporting (beyond the order list) and a formal dispute-resolution workflow — disputes handled manually in v1. Vendor/admin mobile apps (web-only). Vendor sales view (separate ticket, deferred).

## 🔗 Dependencies
- Tickets #1 (scaffolding) and #2 (auth & `setUserRole`).

## 📚 References
- PRD §5.3 (Admin), `docs/FUNCTIONS.md` (`setUserRole`), `docs/DATA_MODEL.md` §2 (`courts`) & §5 (rules), `docs/SETUP.md` (bootstrap note to add).

## 🤖 Kickoff prompt
```
/start-ticket <this-issue-number>
```
