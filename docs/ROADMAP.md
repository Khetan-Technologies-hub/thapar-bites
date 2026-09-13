# Roadmap — Thapar Bites

> Phased path from nothing to a full campus rollout. The PRD's v1 = **Phases 0–3** below. Deferred items are called out explicitly and tracked as Open Decisions in [PRD.md](PRD.md) §7.

---

## Phase 0 — Foundation (scaffolding)

- KMP project with `shared`, `androidApp`, `iosApp`, `web`, `functions`, `firebase` modules.
- Firebase project (dev), emulators wired up, config hygiene in place.
- Shared domain models + order state machine + enums.
- CI skeleton (build + lint).

**Exit:** the empty apps build and run against the emulator on all three platforms.

---

## Phase 1 — Identity & roles

- Google Sign-In restricted to `thapar.edu`; `onUserCreate` sets default `student` claim and creates the user doc.
- Admin-provisioned vendor/admin accounts via `setUserRole`.
- Firestore security rules v1 (role-based matrix).

**Exit:** a student can log in and see an (empty) court list; a vendor can log into the dashboard.

---

## Phase 2 — Catalog & vendor management

- Vendor dashboard: menu CRUD, court settings (hours, open/closed, accepted payment methods, min order value), image upload to Storage.
- Student app: browse courts (open/closed, hours) and menus.

**Exit:** vendors can fully set up a court; students can browse a real menu.

---

## Phase 3 — The order loop (core of v1)

- Cart + checkout with min-order and payment-method validation.
- `placeOrder` (server-authoritative), COD path first.
- Razorpay UPI integration + `razorpayWebhook`.
- Vendor order queue with accept/reject and status transitions (`transitionOrder`).
- Live status (Firestore listeners) + FCM notifications (`onOrderStatusChange`).
- Student order history + reorder.

**Exit — this is shippable v1:** a student orders from a real court, pays (UPI or COD), and the vendor delivers, with live status and notifications throughout.

---

## Phase 4 — Pilot hardening (1–2 courts live)

- Onboard the first one or two food courts for real.
- Monitoring/Crashlytics, error surfaces, edge-case polish.
- Basic vendor sales view (today's orders + totals).
- Operational runbook for onboarding a court.

**Exit:** a real pilot running on campus with live orders.

---

## Phase 5+ — Post-v1 enhancements (deferred)

Tracked as Open Decisions in the PRD; pull in based on pilot learnings:

- **Ratings & reviews** (D5).
- **Dedicated delivery-staff logins** (D3).
- **Live GPS tracking** of the delivery person (D4).
- **Monetization:** delivery fees or per-order commission + settlement/reporting.
- **Scheduled / pre-orders**, promotions/coupons, wallet.
- **Payment aggregator model** if the platform later needs to hold funds (D1) — significant compliance work.
- Roll out to **all campus courts**.

---

## Sequencing notes

- Phases are mostly sequential; Phase 2 (catalog) and early Phase 3 (cart UI) can overlap.
- **Unblockers:** confirm PRD **D1** (payment settlement) and **D6** (Thapar Google Workspace) before Phase 1/3 payment + auth work.
- Each phase should map to a small set of tickets drafted via `/draft-ticket`, each referencing the PRD.
