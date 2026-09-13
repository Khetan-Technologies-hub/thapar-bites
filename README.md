# Thapar Bites 🍔

Food delivery from campus food courts to hostels at **Thapar Institute**.

Students order from any participating campus food court; each food court delivers to hostels with its own staff. The app is deliberately **campus-scoped** — only Thapar students can order (verified by college email), delivery is only to on-campus hostels, and the vendors are the campus food courts themselves.

> **One-line vision:** *Order from any campus food court, get it delivered to your hostel — without leaving your room or standing in a queue.*

---

## Platforms

| Platform | Audience | Tech |
|---|---|---|
| Android + iOS | Students (primary) | Kotlin Multiplatform + Compose Multiplatform |
| Web | Vendors (dashboard, primary) + students | Lightweight web app on Firebase |

---

## Tech stack at a glance

- **Shared logic:** Kotlin Multiplatform (KMP) — models, order state machine, validation, Firebase access.
- **Mobile UI:** Compose Multiplatform (Android + iOS).
- **Backend:** Firebase — Auth, Firestore, Cloud Functions, Cloud Messaging (FCM), Storage, Hosting.
- **Firebase from KMP:** GitLive Firebase Kotlin SDK (`dev.gitlive:firebase-*`).
- **Payments:** Razorpay (UPI/online) + Cash on Delivery, configured **per food court**.
- **Auth:** Google Sign-In restricted to the `thapar.edu` domain; roles via Firebase custom claims.

---

## Documentation

| Doc | What it covers |
|---|---|
| [docs/PRD.md](docs/PRD.md) | Product requirements — vision, scope, users, decisions. **Start here.** |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | System architecture, modules, request/data flow, security model. |
| [docs/DATA_MODEL.md](docs/DATA_MODEL.md) | Firestore collections, document shapes, indexes, security rules. |
| [docs/FUNCTIONS.md](docs/FUNCTIONS.md) | Cloud Functions — the server-side contract (orders, payments, notifications). |
| [docs/SETUP.md](docs/SETUP.md) | Local dev environment setup for mobile, web, and Firebase. |
| [docs/ROADMAP.md](docs/ROADMAP.md) | Phases from pilot to full campus rollout, and what's deferred. |

---

## Core concept: three roles

- **Student** — browse courts, order, pay, track, manage saved hostel addresses.
- **Vendor** (food court operator) — manage menu & settings, accept orders, advance delivery status.
- **Admin** (platform operator) — onboard vendors, oversee all orders, resolve disputes.

## The order loop

```
Placed → Accepted → Preparing → Out for delivery → Delivered
```

Every transition pushes a real-time update and a notification to the relevant party.

---

## Project status

🚧 **Pre-build / design phase.** PRD and architecture documentation are complete; scaffolding is the next step. See [docs/ROADMAP.md](docs/ROADMAP.md).

## Repository layout (planned)

```
thapar-bites/
├── shared/            # KMP shared module (models, logic, Firebase access)
├── androidApp/        # Android entry point (Compose MP)
├── iosApp/            # iOS entry point (Compose MP)
├── web/               # Vendor + student web app
├── functions/         # Firebase Cloud Functions
├── firebase/          # Firestore rules, indexes, Firebase config
└── docs/              # This documentation set
```

---

## Contributing / building this out

This project follows a ticket-driven workflow:

1. **PRD** ([docs/PRD.md](docs/PRD.md)) is the source of truth for *what* and *why*.
2. **`/draft-architecture`** generates `CLAUDE.md` — the coding rules and conventions.
3. **`/draft-ticket <thing>`** creates focused, buildable tickets that reference the PRD.

_Generated with [Claude Code](https://claude.com/claude-code)_
