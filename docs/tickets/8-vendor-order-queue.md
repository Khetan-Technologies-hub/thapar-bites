**Brief:** #5

## 📖 Story / Why
The vendor side of the loop: a live queue of incoming orders that never gets missed, and one-tap progress through to delivery. A busy counter needs the next action to always be obvious.

## 🧭 Context
**Platform: web only, by design** — the vendor works on a laptop/desktop browser (confirmed product decision). No vendor mobile app in v1.

Vendor dashboard (React + TS) + Firestore listeners, calling `transitionOrder` (5a). Vendor sees only their own court's orders (rules from 5a/#2). Orders appear only **after** the 30s hold (5b). Delivery is closed by the vendor/staff (Brief #5).

## 🔑 Access & prerequisites
- Order core (5a) and vendor catalog/dashboard (#3) built.
- A provisioned vendor account with a court that has orders (or seeded emulator orders).

## ✅ Scope / What to build
- [ ] Live order queue (real-time) with a **loud, unmissable new-order alert** (sound + visual).
- [ ] Accept / reject a new order (reject triggers refund path in 5d for paid orders).
- [ ] Advance status: Accepted → Preparing → Out for delivery → **Delivered**, each via `transitionOrder`.
- [ ] Order detail: items, quantities, total, delivery address, payment method + status.
- [ ] Loading / empty / error states; disabled controls during in-flight transitions.

## 🎯 Acceptance Criteria
- [ ] New (post-hold) orders appear in the vendor's queue in real time with an alert.
- [ ] Vendor can accept/reject, and advance an accepted order through to Delivered; illegal transitions are impossible (server-validated).
- [ ] A vendor never sees another court's orders.
- [ ] Status changes reflect on the student side within seconds (via 5e once built; here verified in the data).

## 🖼️ UI standards
UI ticket — apply `templates/ui-standards.md`. Load-bearing: light + dark; responsive with desktop reflow + min width; truncation; loading/empty/error/disabled; no hardcoded strings; design-system tokens. New-order alert must be reliable. Match the provided design exactly (ask the PM for assets at `/start-ticket`).

## 🚫 Out of scope
- Student checkout/hold (5b). Razorpay/refunds (5d). Student tracking/notifications (5e). Sales reporting (later).

## 🔗 Dependencies
- Tickets 5a and #3.

## 📚 References
- PRD §5.2, `docs/ARCHITECTURE.md` §4/§6, `docs/FUNCTIONS.md` (`transitionOrder`), Brief #5.

## 🤖 Kickoff prompt
```
/start-ticket <this-issue-number>
```
