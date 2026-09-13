# CLAUDE.md — Thapar Bites

> Read on every ticket. The architecture and the non-negotiable rules. When in doubt, [docs/PRD.md](docs/PRD.md) is the source of truth for *what/why*; this file is *how*.

## What this project is

Thapar Bites is a campus food-delivery app: Thapar students order from campus food courts and get delivery to their hostel; each food court delivers with its own staff. Kotlin Multiplatform (shared logic) + Compose Multiplatform (Android/iOS) + a React web app (vendor dashboard), all on Firebase. Campus-scoped by design — students verified by `thapar.edu`, delivery only to hostels, vendors are the campus courts.

## Stack (locked — do not substitute without a PRD decision)

| Layer | Choice |
|---|---|
| Shared logic | Kotlin Multiplatform (`shared/`) |
| Mobile UI | Compose Multiplatform (`androidApp/`, `iosApp/`) |
| Presentation | **MVVM** — shared ViewModels expose `StateFlow`; UI observes |
| DI | **Koin** |
| Backend | Firebase — Auth, Firestore, Cloud Functions, FCM, Storage, Hosting |
| Firebase from KMP | GitLive Firebase Kotlin SDK (`dev.gitlive:firebase-*`) |
| Cloud Functions | **TypeScript** (`functions/`) |
| Web | **React + TypeScript** (`web/`) on the Firebase JS SDK |
| Payments | Razorpay (UPI) + COD, configured per court |

## Architecture

```
shared/         KMP: models · order state machine · validation · repositories (Firebase) · ViewModels
androidApp/     Compose MP entry (thin — UI only)
iosApp/         Compose MP entry (thin — UI only)
web/            React + TS vendor dashboard (+ student web)
functions/      Cloud Functions (TypeScript) — server-authoritative logic
firebase/       firestore.rules · firestore.indexes.json · firebase.json
docs/           PRD, architecture, data model, functions, setup, roadmap
```

**Layering (strict):**
- **Models / domain** — plain Kotlin data classes + enums. The **order state machine** and **validation** (min-order, operating hours, cart integrity) live here and nowhere else.
- **Repositories** — the *only* place that talks to Firebase. Expose `suspend` functions and `Flow`s. UI and ViewModels never touch Firestore/Auth directly.
- **ViewModels** (shared) — hold UI state as `StateFlow`, call repositories. No Firebase types leak into the UI.
- **UI** (Compose MP / React) — dumb: render state, emit events. No business logic.

**Data flow:** UI → ViewModel → Repository → Firebase. Real-time via Firestore listeners surfaced as `Flow`. See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Key rules (non-negotiable)

1. **The server owns money and order state.** Order totals, minimum-order enforcement, allowed payment methods, and every status transition are validated/performed by **Cloud Functions**. Clients send *requests*; they never write authoritative fields (`total`, `subtotal`, `status`, `paymentStatus`) directly. Firestore rules must deny such client writes.
2. **Money is integer paise.** Never floats, anywhere — Kotlin, TS, or Firestore. ₹1 = 100.
3. **One source of truth for order transitions** — the state machine in `shared/`. `transitionOrder` validates against it. No client marks its own order forward illegally.
4. **Price snapshotting** — orders store item prices at placement; later menu edits never change past orders.
5. **Roles come from the auth token's custom claim**, never from a client-writable `users` field.
6. **Repositories are the Firebase boundary.** No Firebase SDK calls in ViewModels or UI.
7. **Never validate only on the client.** Client checks are UX; the server re-validates everything.
8. **Campus timezone is Asia/Kolkata** for all operating-hours logic; evaluate server-side.

## Security

- Auth: Google Sign-In restricted to `thapar.edu` (students); admin-provisioned email/password + custom claims for vendors/admins.
- Firestore security rules enforce the role matrix in [docs/DATA_MODEL.md](docs/DATA_MODEL.md). Least privilege; a client never trusts another client's writes.
- **No secrets in the repo or in this file.** Razorpay keys, webhook secret, and service accounts live in Firebase secrets/config. `.gitignore` excludes `google-services.json`, `GoogleService-Info.plist`, web `.env*`, `*serviceAccount*.json`. Use Razorpay **test mode** and Firebase **emulators** for all local work.
- Webhooks (`razorpayWebhook`) verify signatures and are idempotent.

## Conventions

- Kotlin: official style; `camelCase` members, `PascalCase` types; packages under `com.thaparbites.*`. Enums for `role`/`status`/`paymentMethod`/`paymentStatus` (defined once in `shared/`, mirrored in TS/rules).
- Compose UI is stateless where possible; hoist state to ViewModels.
- TypeScript (functions & web): strict mode on; throw typed `HttpsError` with codes clients can map to messages.
- Tests: business logic (state machine, validation) unit-tested in `shared/` (`kotlin.test`, Turbine for Flows); Cloud Functions tested against the Firebase emulator. New logic ships with tests.
- Commits/PRs follow the ticket (see below).

## Anti-patterns — do NOT

- Compute or trust order totals / minimums on the client as final.
- Call Firestore/Auth from UI or ViewModels.
- Add a delivery fee or platform commission (v1 is free delivery, no cut — see PRD Decision Log).
- Introduce student-runner/gig delivery, GPS tracking, or ratings — all deferred (PRD §7).
- Commit config/secret files.
- Use floating point for money.

## How we work (ticket workflow)

This project is ticket-driven (Humble Task Force). Process detail lives in `docs/PROCESS.md` (created by `/setup-tickets`).

- `/draft-ticket <thing>` — draft a ticket that references this file + the PRD.
- `/start-ticket` — build a ticket (reads context, plans, then codes).
- `/handoff` — produce the handoff report from the real git diff.
- `/manager-review` — review the PR against the ticket's acceptance criteria.

Every change maps to a ticket. Tickets follow the phases in [docs/ROADMAP.md](docs/ROADMAP.md).

## References

- [docs/PRD.md](docs/PRD.md) — product spec + decision log (source of truth)
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) — system architecture & flows
- [docs/DATA_MODEL.md](docs/DATA_MODEL.md) — Firestore schema, indexes, security-rule matrix
- [docs/FUNCTIONS.md](docs/FUNCTIONS.md) — Cloud Functions contract
- [docs/SETUP.md](docs/SETUP.md) — local dev setup
- [docs/ROADMAP.md](docs/ROADMAP.md) — build phases
- **Open decisions** to confirm before payment/auth work: PRD §7 **D1** (payment settlement) and **D6** (Thapar Google Workspace).
